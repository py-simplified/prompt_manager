from __future__ import annotations

from io import BytesIO
from pathlib import Path
from typing import IO

import pandas as pd
from docx import Document
from PyPDF2 import PdfReader

SUPPORTED_REQUIREMENT_TYPES = {".txt", ".md", ".pdf", ".docx"}
SUPPORTED_PROMPT_TYPES = {".csv", ".xlsx"}


def _reset_stream(buffer: IO[bytes]) -> None:
    if hasattr(buffer, "seek"):
        buffer.seek(0)


def read_requirement(uploaded_file) -> str:
    """
    Read requirement content from an uploaded file and return text.
    Supports .txt, .md, .pdf, and .docx files.
    """
    filename = getattr(uploaded_file, "name", "") or getattr(uploaded_file, "filename", "")
    extension = Path(filename).suffix.lower()
    if extension not in SUPPORTED_REQUIREMENT_TYPES:
        raise ValueError(f"Unsupported requirement file type: {extension or 'unknown'}")

    _reset_stream(uploaded_file)
    raw_bytes = uploaded_file.read()
    _reset_stream(uploaded_file)

    if extension in {".txt", ".md"}:
        try:
            return raw_bytes.decode("utf-8")
        except UnicodeDecodeError as exc:
            raise ValueError("Requirement file must be UTF-8 encoded.") from exc

    if extension == ".docx":
        document = Document(BytesIO(raw_bytes))
        paragraphs = [para.text for para in document.paragraphs if para.text]
        return "\n".join(paragraphs).strip()

    if extension == ".pdf":
        reader = PdfReader(BytesIO(raw_bytes))
        pages = []
        for page in reader.pages:
            text = page.extract_text() or ""
            pages.append(text)
        return "\n".join(pages).strip()

    raise ValueError(f"Unsupported requirement file type: {extension}")  # pragma: no cover


def read_prompts(uploaded_file) -> pd.DataFrame:
    """
    Read prompts from a CSV or XLSX file and return a DataFrame with columns Artifact and Prompt.
    """
    filename = getattr(uploaded_file, "name", "") or getattr(uploaded_file, "filename", "")
    extension = Path(filename).suffix.lower()
    if extension not in SUPPORTED_PROMPT_TYPES:
        raise ValueError(f"Unsupported prompt file type: {extension or 'unknown'}")

    _reset_stream(uploaded_file)

    if extension == ".csv":
        df = pd.read_csv(uploaded_file)
    else:
        df = pd.read_excel(uploaded_file)

    expected_columns = {"Artifact", "Prompt"}
    if not expected_columns.issubset(df.columns):
        raise ValueError("Prompt file must contain 'Artifact' and 'Prompt' columns.")

    # Return only the required columns in a consistent order.
    return df.loc[:, ["Artifact", "Prompt"]]
