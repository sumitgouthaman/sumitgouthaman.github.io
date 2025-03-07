+++
title = 'Leaky Abstractions: Multimodal messages in LangChain'
summary = 'LangGraph is a framework I appreciate for its ability to abstract away the underlying LLM provider. However, when it comes to multimodal messages, the underlying implementation details leak through.'
date = 2025-03-03T22:22:50-08:00
+++

LangChain's [`BaseChatModel`](https://python.langchain.com/api_reference/core/language_models/langchain_core.language_models.chat_models.BaseChatModel.html) is a good abstraction. Typically, you can implement your components to use a BaseChatModel without worrying too much about the underlying LLM model (Claude, Gemini, etc.).

However, as I discovered [recently]({{< relref "extracting-structured-data-from-pdfs.md" >}}) when working with multimodal inputs, this abstraction becomes quite leaky when dealing with anything other than simple text messages.

It appears there is no standardized pattern across LLM providers for how to pass non-text inputs (images, PDFs, audio, etc.) to the model. One must refer to each model's documentation to determine the correct input format.

For example, here's how Claude expects PDF chunks to be defined in the input ([reference](https://docs.anthropic.com/en/docs/build-with-claude/pdf-support)):

```
 {
    "type": "document",
    "source": {
        "type": "base64",
        "media_type": "application/pdf",
        "data": <base64_encoded_data>,
    }
 }
```

And here's how Gemini expects it ([reference](https://cloud.google.com/vertex-ai/generative-ai/docs/multimodal/document-understanding)):

```
{
    "type": "media",
    "mime_type": "application/pdf",
    "data": <base64_encoded_data>,
}
```

These are minor differences, but they cause problems when implementing a tool or node intended for reuse or sharing. You are now forced to restrict it to only work with `BaseChatModel` instances backed by specific LLMs. The only workaround I've found is to do something like this:

```pdf
class MyTool:
    """Do something fancy."""

    def __init__(self, llm: BaseChatModel):
        self.llm = llm

    def execute(self, file_path: str | os.PathLike) -> List[SomeResult]:
        """Does some analysis on the file.

        Args:
            file_path: The path to the file.
        """

        with open(file_path, "rb") as f:
            file_base64 = base64.b64encode(f.read()).decode("utf-8")

        pdf_content: dict = {}
        if "gemini" in self.llm.model:
            pdf_content = {
                "type": "media",
                "mime_type": "application/pdf",
                "data": file_base64,
            }
        elif "claude" in self.llm.model:
            pdf_content = {
                "type": "document",
                "source": {
                    "type": "base64",
                    "media_type": "application/pdf",
                    "data": file_base64,
                },
            }
        else:
            raise ValueError(
                f"Unsupported model: {llm.model}, multi-modal message format needs to be implemented for new/unsupported models."
            )

        response_dict = self.llm.with_structured_output(
            SomeResult, include_raw=True
        ).invoke(
            [
                SystemMessage(content=SYSTEM_PROMPT),
                HumanMessage(
                    content=[
                        pdf_content,
                        {
                            "type": "text",
                            "text": "Return results for this file based on the instructions.",
                        },
                    ]
                ),
            ]
        )

        if not response_dict["parsed"]:
            print("raw:\n", response_dict["raw"])
            raise RuntimeError(
                f"There was an error parsing: {response_dict['parsing_error']}"
            )

        return response_dict["parsed"]
```

As you can see, this is not an ideal solution.
