+++
title = 'Extracting structured data from PDFs like a pro'
summary = 'How I extract useful structured data from messy brokerage statements.'
date = 2025-03-01
tags = ['llms', 'gen-ai']
+++

One of my biggest annoyances with financial institutions is the lack of a standardized format for exporting details of stocks and other holdings in a brokerage account.

You can typically export a PDF from your broker, but that's not machine-friendly. Some brokers also provide a CSV, but it's often messy, with inconsistent headers and merged cells, making it difficult to parse correctly.

It turns out, the current generation of foundational models are exceptionally good at PDF parsing. Below is a Langchain-based implementation I developed to parse statements from my broker.

### Prerequisites

1.  You'll need an API key from Anthropic (or OpenAI or Gemini).
    -   The code below assumes Anthropic.
    -   Set it as an environment variable: `export ANTHROPIC_API_KEY=[YOUR_API_KEY]`

    **NOTE**: If you use Gemini without a paid plan, the terms of use allow Google to use the data provided for training the LLM. For sensitive data make sure you are using a API key with billing enabled.

1.  You'll need the following packages:

    ```sh
    python3 -m venv .venv
    source .venv/bin/activate
    pip install langgraph langchain_anthropic pydantic tabulate
    ```

### Models

These are the Pydantic models we will use when interacting with the LLM.

```python
# models.py

from typing import List
from pydantic import BaseModel, Field
from enum import Enum

class FinancialInstitution(str, Enum):
    CHASE = "Chase"
    FIDELITY = "Fidelity"
    SCHWAB = "Schwab"

class PositionType(str, Enum):
    STOCK = "Stock"
    MUTUAL_FUND = "Mutual Fund"
    ETF = "Exchange Traded Fund"
    OTHER = "Other"

class Position(BaseModel):
    """Represents holdings of a particular stock, mutual fund, ETF, etc."""
    type: PositionType = Field(description="The type of position.")
    symbol: str = Field(description="The stock ticker symbol.")
    description: str = Field(description="The description or name.")
    quantity: float = Field(description="The quantity held.")
    price: float = Field(description="The current per-unit price.")

class InvestmentAccount(BaseModel):
    """Information about a particular brokerage account."""
    account_number: str = Field(description="The account number.")
    name: str = Field(description="The name or nickname of the account.")
    institution: FinancialInstitution = Field(description="The financial institution.")
    stock_holdings: List[Position] = Field(description="The stock holdings in the account.")
    cash: float = Field(description="Amount of cash available in the account to invest.")
```

### Parser

The actual parser is straightforward to implement:

```python
# statement_parser.py

import os
import base64
from typing import List
from langchain_core.language_models import BaseChatModel
from langchain_core.messages import SystemMessage, HumanMessage
from pydantic import BaseModel, Field

from models import InvestmentAccount

SYSTEM_PROMPT = """
You are tasked with parsing a PDF document and extracting a list of brokerage accounts mentioned in the document, along with the stocks held in each account.
Your goal is to identify and list all unique accounts referenced in the document.

To complete this task, follow these steps:

1. Carefully read through the entire PDF content.
2. Look for mentions of accounts. These may be identified by:
   - Account names (e.g., "Savings Account", "Checking Account")
   - Account numbers (usually a series of digits, possibly with hyphens)
   - References to specific financial products that imply an account (e.g., "401(k)", "IRA")
   - Sections or headers that group account information
3. Within each account, look for a table of stocks held in that account.
   - Categorize money market funds as uninvested cash.
   - Do not consider unvested RSUs or stock grants.
4. Only consider accounts that contain stocks, mutual funds or ETFs.
5. As you identify accounts, keep a running list. Make sure to only include unique accounts—if an account is mentioned multiple times, only list it once.
6. Pay attention to context. Some words like "account" might be used in a general sense and not refer to a specific account. Use your judgment to determine if a mention truly refers to a distinct account.
7. If you encounter any ambiguities or uncertainties about whether something should be considered an account or if it's unclear which account a stock is in, make a note of it.
8. After you've parsed the entire document, review your list to ensure all entries are unique and relevant.

Remember, your task is to extract accounts and stock holdings mentioned in the document, not to analyze or interpret the content beyond this scope. Focus on identifying and listing the accounts accurately and comprehensively.
"""


class ParseAccountsOutput(BaseModel):
    """Result of parsing accounts and stock holdings in a PDF file."""
    accounts: List[InvestmentAccount] = Field(description="Accounts in the file.")
    errors: List[str] = Field(description="Optional errors/information about any ambiguities or uncertainties.")


class StatementProcessor:
    """Processes financial statements."""

    def __init__(self, llm: BaseChatModel):
        """Initializes the statement processor.

        Args:
            llm: The language model to use for processing.
        """
        self.llm = llm

    def parse_statement(self, file_path: str | os.PathLike) -> List[InvestmentAccount]:
        """Parses a financial statement.

        Args:
            file_path: The path to the financial statement.
        """

        if not os.path.exists(file_path):
            raise FileNotFoundError(f"File not found: {file_path}")

        with open(file_path, "rb") as f:
            file_base64 = base64.b64encode(f.read()).decode("utf-8")

        response_dict = self.llm.with_structured_output(ParseAccountsOutput, include_raw=True).invoke(
            [
                SystemMessage(content=SYSTEM_PROMPT),
                HumanMessage(content=[
                    # TODO: This is not model-agnostic!
                    {
                        "type": "document",
                        "source": {
                            "type": "base64",
                            "media_type": "application/pdf",
                            "data": file_base64
                        }
                    },
                    {
                        "type": "text",
                        "text": "Identify the accounts and stock holdings in this file."
                    }
                ])
            ]
        )

        if not response_dict["parsed"]:
            print("raw:\n", response_dict["raw"])
            raise RuntimeError(f"There was an error parsing: {response_dict['parsing_error']}")

        return response_dict["parsed"]
```

### Testing

Here's a snippet you can use to run it against any PDF file:

```python
# main.py

from langchain_anthropic import ChatAnthropic
from statement_processor import StatementProcessor
from tabulate import tabulate


if __name__ == "__main__":
    # Assumes ANTHROPIC_API_KEY is set.
    llm = ChatAnthropic(
                model="claude-3-7-sonnet-20250219",
                temperature=0,
                max_tokens=4096,
                timeout=None,
                max_retries=0
            )
    processor = StatementProcessor(llm)
    file_path = input("Enter path of file to process: ")
    try:
        response = processor.parse_statement(file_path)
        for account in response.accounts:
            print(f"Account Name: {account.name}")
            print(f"Account Number: {account.account_number}")
            print(f"Institution: {account.institution}")
            headers = ["#", "Type", "Symbol", "Description", "Quantity", "Price"]
            table_data = []
            for i, holding in enumerate(account.stock_holdings):
                table_data.append([i+1, holding.type, holding.symbol, holding.description, holding.quantity, holding.price])
            print(tabulate(table_data, headers=headers, tablefmt="grid"))
            print(f"Cash: {account.cash}")
            print("-" * 40)

        print("Errors:")
        if response.errors:
          for error in response.errors:
            print(error)
        else:
          print("No Errors Found.")
        print("-" * 40)

    except FileNotFoundError as e:
      print(e)
    except RuntimeError as e:
      print(e)
```

### Result

Here's a sample result (read data has been replaced / removed):

```
Enter path of file to process: /path/to/statement.pdf
Account Name: JohnDoe - INDIVIDUAL - TOD
Account Number: 12345
Institution: FinancialInstitution.FIDELITY
+-----+--------------------------+----------+-----------------------------------------------+------------+---------+
|   # | Type                     | Symbol   | Description                                   |   Quantity |   Price |
+=====+==========================+==========+===============================================+============+=========+
|   1 | PositionType.STOCK       | AMZN     | AMAZON.COM INC                                |     XX     |  XXX.XX |
+-----+--------------------------+----------+-----------------------------------------------+------------+---------+
|   2 | PositionType.STOCK       | AAPL     | APPLE INC                                     |     XX     |  XXX    |
+-----+--------------------------+----------+-----------------------------------------------+------------+---------+
|   3 | PositionType.STOCK       | BRKB     | BERKSHIRE HATHAWAY INC COM USD0.0033 CLASS B  |     XX     |  XXX.XX |
+-----+--------------------------+----------+-----------------------------------------------+------------+---------+
|   6 | PositionType.STOCK       | MSFT     | MICROSOFT CORP                                |     XX     |  XXX.XX |
+-----+--------------------------+----------+-----------------------------------------------+------------+---------+
|   7 | PositionType.STOCK       | SHOP     | SHOPIFY INC COM NPV CL A                      |     XX     |  XXX.X  |
+-----+--------------------------+----------+-----------------------------------------------+------------+---------+
|  10 | PositionType.MUTUAL_FUND | FNILX    | FIDELITY ZERO LARGE CAP INDEX FUND            |    XXX.XX  |   XX.XX |
+-----+--------------------------+----------+-----------------------------------------------+------------+---------+
|  11 | PositionType.MUTUAL_FUND | FZIPX    | FIDELITY ZERO EXTND MARKET INDEX FUND         |    XXX.XX  |   XX.XX |
+-----+--------------------------+----------+-----------------------------------------------+------------+---------+
|  14 | PositionType.ETF         | SCHA     | SCHWAB STRATEGIC TR US SMALL-CAP ETF          |    XXX     |   XX.XX |
+-----+--------------------------+----------+-----------------------------------------------+------------+---------+
|  15 | PositionType.ETF         | VYM      | VANGUARD WHITEHALL FDS HIGH DIV YLD           |    XXX.XXX |  XXX.XX |
+-----+--------------------------+----------+-----------------------------------------------+------------+---------+
|  19 | PositionType.ETF         | VTI      | VANGUARD INDEX FDS VANGUARD TOTAL STK MKT ETF |    XXX     |  XXX.X  |
+-----+--------------------------+----------+-----------------------------------------------+------------+---------+
|  20 | PositionType.ETF         | BND      | VANGUARD BD INDEX FDS TOTAL BND MRKT          |     XX     |   XX.XX |
+-----+--------------------------+----------+-----------------------------------------------+------------+---------+
Cash: XXX.XX
----------------------------------------
Account Name: JohnDoe - INDIVIDUAL - TOD
Account Number: 12346
Institution: FinancialInstitution.FIDELITY
+-----+--------+----------+---------------+------------+---------+
| #   | Type   | Symbol   | Description   | Quantity   | Price   |
+=====+========+==========+===============+============+=========+
+-----+--------+----------+---------------+------------+---------+
Cash: X.X
----------------------------------------
Account Name: JohnDoe - HEALTH SAVINGS ACCOUNT
Account Number: 12347
Institution: FinancialInstitution.FIDELITY
+-----+------------------+----------+-----------------------------------------------+------------+---------+
|   # | Type             | Symbol   | Description                                   |   Quantity |   Price |
+=====+==================+==========+===============================================+============+=========+
|   1 | PositionType.ETF | VTI      | VANGUARD INDEX FDS VANGUARD TOTAL STK MKT ETF |          X |   XXX.X |
+-----+------------------+----------+-----------------------------------------------+------------+---------+
Cash: XXXX.X
----------------------------------------
Errors:
The second account (12346) appears to be empty with no cash or stock holdings. It's mentioned in the document but has no assets listed.
The document also mentions XYZ RSU (Restricted Stock Units) with X,XXX unvested units valued at $XXX,XXX.XX, but these are not held in a brokerage account - they're listed under 'Stock Plans' as other holdings.
----------------------------------------
```

### Cost

So how much does this cost? That depends on how big your PDF is.

Datapoint:
- The largest statement I got from one of my brokers ended up consuming around **45K input tokens** and the **output consumed 2K tokens**.
- The current pricing for Claude Sonnet is $3 per 1M input tokens and $15 per 1M output tokens.

Because I'm being lazy, let's use an LLM to calculate the total cost per PDF (using `claude-3-7-sonnet-20250219`):

```
Input Cost:
Cost per input token: $3 / 1,000,000 tokens = $0.000003 per token
Cost of 50,000 input tokens: $0.000003/token * 50,000 tokens = $0.15

Output Cost:
Cost per output token: $15 / 1,000,000 tokens = $0.000015 per token
Cost of 2,000 output tokens: $0.000015/token * 2,000 tokens = $0.03

Total Cost:
Total cost: $0.15 (input) + $0.03 (output) = $0.18
```

So, processing a single PDF is pretty cheap. It will be 50% cheaper if you use [batch processing](https://docs.anthropic.com/en/docs/build-with-claude/batch-processing).

Another way to cut cost would be to use a cheaper model like `gemini-2.0-flash` ($0.10/$0.40 per 1M input/output tokens). In my experience, Gemini performs quite well on PDF parsing tasks.

Pricing for `gemini-2.0-flash`:

```
Input Cost:
  ($0.10 / 1,000,000) * 50,000 = $0.005

Output Cost:
  ($0.40 / 1,000,000) * 2,000 = $0.0008

Total Cost:
  $0.005 + $0.0008 = $0.0058
```

I did try using `gemini-2.0-flash-lite`, but the results were not good.