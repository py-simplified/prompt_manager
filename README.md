# PromptDocGen_L1

Streamlit application that turns a requirement document and a set of prompts into ready-to-download DOCX and PDF deliverables.

## Features
- Upload a single requirement file in `.txt`, `.md`, `.pdf`, or `.docx` format.
- Upload prompt templates from `.csv` or `.xlsx` with `Artifact` and `Prompt` columns.
- Toggle between `openai` and `internal` chat providers with appropriate credential handling.
- Preview generated outputs in the browser and download combined DOCX and PDF files.

## Getting Started

### Prerequisites
- Python 3.9 or higher
- pip

### Installation
```bash
pip install -r requirements.txt
```

### Environment Variables
Copy `.env.example` to `.env` and fill in the values you need:
```
OPENAI_API_KEY=your-openai-api-key
INTERNAL_ACCESS_TOKEN=your-internal-access-token
INTERNAL_CHAT_URL=https://internal-chat-endpoint.example.com
```

`OPENAI_API_KEY` is used automatically when you select the OpenAI provider unless you enter a key in the UI. The internal credential values are convenience references; you can also paste them directly into the sidebar inputs.

### Run the App
```bash
streamlit run app.py
```

## Usage
1. Upload a requirement document.
2. Upload a prompt file with the required columns.
3. Choose the chat provider:
   - **openai**: supply an API key or rely on `OPENAI_API_KEY` from the environment.
   - **internal**: provide an AP Token and Internal Endpoint URL. The request sends `AMToken`, `Token_Type="SESSION_TOKEN"`, and a generated `x-correlation-id` header.
4. Click **Generate Outputs** to run each prompt against the requirement text. Each call is formatted as:
   ```
   {Prompt}

   Requirement:
   {<extracted requirement text>}
   ```
5. Review the generated sections and download the combined DOCX or PDF.

## Sample Data
- `sample_data/sample_requirement.txt`: example requirement text.
- `sample_data/sample_prompts.csv`: prompt templates for Requirement Summary, User Stories, and Test Cases.

These files let you try the app without connecting to a live provider by mocking responses in `chat_connectivity.py`.
