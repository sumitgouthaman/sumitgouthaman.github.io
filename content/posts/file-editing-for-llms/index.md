+++
title = 'The hidden sophistication behind how AI agents edit files'
date = 2025-10-11T20:02:42-07:00
tags = ['llms', 'agents', 'tool-use']
+++

I was recently using Q CLI to implement a new feature and noticed it struggling in an unexpected way. It had managed to implement the feature incredibly quickly in 4-5 iterations but was struggling when it came to updating all the tests impacted by the change. In particular, it was repeatedly running into issues editing large text files either because the string it was trying to replace wasn't present or was present in more than one location in the file. I had seen this happen before: typically it would try in vain for a while and eventually either give up or use command-line tools to edit the file.

As a human, it was incredibly obvious to me what should happen. In this particular case, the agent should just have used a "find and replace all" strategy to replace multiple instances. Was such a tool available to the agent? I was curious. I made a mental note to look into this later.

This weekend I finally got around to digging into the source code for Q CLI and Gemini CLI to see how they both approach file editing. Turns out, Gemini CLI does a bunch of smart things to make file editing mistakes very forgiving for the LLM. Also, it turns out there is enough of a market here that startups have appeared that specialize in building a file editing tool for your AI agent!

## What makes file editing tricky for LLMs?

I've actually struggled with this problem myself when experimenting with an agent that upgrades source code. Back then, I was using the [filesystem](https://python.langchain.com/docs/integrations/tools/filesystem/) tools in LangGraph which are pretty rudimentary and simply replace or append to the file's content. I also remember experimenting with a custom tool implementation that allowed search-and-replace on a file's content. The simple approaches often failed in predictable ways:

1. The LLM generates some search pattern that doesn't match the file's content because of whitespace issues (probably due to incorrect indentation).
2. It generates some pattern that is present multiple times making it ambiguous which one to replace.
3. It generated replacement code that was indented incorrectly for languages sensitive to indentation.
4. It tries to replace the entire file or huge chunks of the file, but rather than return the entire content to write, it had portions like `// This section remains the same`.

All these seem pretty predictable given how LLMs work.

## How do coding assistants implement file editing?

Q CLI and Gemini CLI are the two coding agents I use regularly. Thankfully, they are both open-source and I can check how they implement their tools. The one minor wrinkle is that I am only barely familiar with TypeScript (which Gemini CLI is written in) and not at all familiar with Rust (which Q CLI is written in). This is where I was able to leverage Gemini CLI itself to go through its own code (which I cloned locally) and help me understand the nuances of its implementation. Same with Q CLI.

To understand how these assistants handle file editing, let's first examine their approach to reading files—a crucial first step in the process.

### Reading from files

**Q CLI**

The `fs_read` [(source code)](https://github.com/aws/amazon-q-developer-cli/blob/main/crates/chat-cli/src/cli/chat/tools/fs_read.rs) tool has 4 modes:

- Line: Read lines from a file
- Directory: List directory contents
- Search: Search for patterns in files
- Image: Read and process images

The line mode allows reading only snippets of the file by providing a `start_line` and `end_line`.

**Gemini CLI**

The `read-file` [(source code)](https://github.com/google-gemini/gemini-cli/blob/main/packages/core/src/tools/read-file.ts) tool here is similar, but different in a subtle way.

The LLM agent can invoke this tool with the following parameters:

| Parameter       | Type     | Required | Description                                                                                       |
| :-------------- | :------- | :------- | :------------------------------------------------------------------------------------------------ |
| `absolute_path` | `string` | Yes      | The absolute path to the file that needs to be read.                                              |
| `offset`        | `number` | No       | An optional 0-based line number to start reading from. Useful for paginating through large files. |
| `limit`         | `number` | No       | An optional parameter to specify the maximum number of lines to read.                             |

Rather than have the LLM calculate and figure out the `end_line`, here it can just specify how many lines it wants. Maybe that's simpler?

Also, the output of the tool prints out some surrounding preamble to help the agent understand that the content is truncated:

```
IMPORTANT: The file content has been truncated.
Status: Showing lines 1-500 of 2500 total lines.
Action: To read more of the file, you can use the 'offset' and 'limit' parameters in a subsequent 'read_file' call. For example, to read the next section of the file, use offset: 501.

--- FILE CONTENT (truncated) ---
... (first 500 lines of file content) ...
```

### Editing files

**Q CLI**

The `fs_write` [(source code)](https://github.com/aws/amazon-q-developer-cli/blob/main/crates/chat-cli/src/cli/chat/tools/fs_write.rs) has 4 modes: create, str_replace, insert and append.

Focusing in on `str_replace` (since that's what you would use for most edits), I see the parameters:

- `path: String`: The path to the file to modify.
- `old_str: String`: The exact string to be replaced.
- `new_str: String`: The new string to substitute.
- `summary: Option<String>`: An optional description of the change.

Looking at the actual implementation, it is a very simple find & replace. In case the `old_str` appears multiple times or 0 times, it returns an error to the LLM.

**Gemini CLI**

Gemini seems to have 2 tools for editing files.

There is a `write-file` [(source code)](https://github.com/google-gemini/gemini-cli/blob/main/packages/core/src/tools/write-file.ts) tool with parameters:

| Parameter   | Type     | Required | Description                                          |
| :---------- | :------- | :------- | :--------------------------------------------------- |
| `file_path` | `string` | Yes      | The absolute path of the file to write to.           |
| `content`   | `string` | Yes      | The full content that should be written to the file. |

But even more interesting is the `smart-edit` / `replace` tool [(source code)](https://github.com/google-gemini/gemini-cli/blob/main/packages/core/src/tools/smart-edit.ts):

| Parameter     | Type     | Required | Description                                                                                                                   |
| :------------ | :------- | :------- | :---------------------------------------------------------------------------------------------------------------------------- |
| `file_path`   | `string` | Yes      | The absolute path to the file to be modified.                                                                                 |
| `instruction` | `string` | Yes      | A clear, high-level instruction explaining the purpose of the change. This is crucial for the tool's self-correction feature. |
| `old_string`  | `string` | Yes      | The exact, literal text to be replaced. It should include several lines of context to ensure it's a unique match.             |
| `new_string`  | `string` | Yes      | The exact, literal text to replace `old_string` with.                                                                         |

While it may appear similar to Q CLI's tool, notice the `instruction` field. That hints at something more complicated happening under the covers.

Digging into the code, I see that it implements a multi-step strategy to find the `old_str`:

1.  **Exact Match**: First, it attempts a direct, literal string match.
2.  **Flexible Match**: If the exact match fails, it tries a more lenient match that ignores leading/trailing whitespace on each line, which helps overcome minor formatting differences.
3.  **Regex Match**: As a final attempt, it constructs a regular expression from the `old_string` that is flexible with respect to the amount of whitespace between words or tokens.

It also uses the `instruction` for LLM powered self-correction: If all search strategies fail, the tool uses the provided `instruction` to make another LLM call. It asks the second LLM to "fix" the original `old_string` and `new_string` to better match the actual file content, and then re-attempts the replacement.

### Which one is better?

Compared to the bare-bones "find and replace", Gemini has a more sophisticated tool that tries to use different strategies to fix any mistakes in the LLM's input to the tool.

It seems like improving file editing tools is a low-hanging fruit that could improve the performance of any agent that needs to edit files.

The tools in Gemini CLI seem to follow some of the same best-practices that Anthropic recommends in their excellent guide: https://www.anthropic.com/engineering/writing-tools-for-agents.

## More sophisticated file-editing tools

While I was digging into this, I realized that there are more sophisticated editing approaches out there.

I came across this startup: https://www.morphllm.com, which claims 98% first-pass accuracy (v/s 84% of search-replace), 50% fewer tokens, 12x faster time-to-merge.

I was curious about how that was possible. I didn't get time to try it out myself, but [this](https://www.morphllm.com/comparisons/morph-vs-claude-search-replace) post on their website seems to explain how it works.

- Instead of having to generate old and new version of the code, it only asks the agent to generate the new code + some instructions explaining what is being changed.
- Within the tool call, you call their API and pass the instruction + new code + existing file content and it returns the updated file content.
- On their end, they use their embedding model to retrieve relevant parts of the code and a re-ranking model to sort the snippets. Then they use a fine-tunes Llama 70b model to actually apply the edits.
- They claim cost savings because your main model is producing fewer tokens (since it doesn't have to generate old code). But, you have to add the cost of calling Morph itself (fast model is $0.8 / 1M input tokens and $1.2 / 1M output tokens). For comparison, Claude API pricing is $3 / 1M input tokens and $15 / 1M output tokens.

I hadn't heard of this before, but another post on their website sent me to a [deleted post](https://web.archive.org/web/20240823050616/https://www.cursor.com/blog/instant-apply) from Cursor (I've definitely heard of that!), that describes how Cursor uses a similar approach for their file editing tool.

## Conclusion

What looks fairly simple on the surface is actually quite a bit more sophisticated when you dig into the details. There is a lot of hype that surrounds new model releases, but I never see much discussion about how these subtle differences in tool implementations can change the performance and perceived efficacy of these coding agents.
