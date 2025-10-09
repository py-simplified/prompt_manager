from __future__ import annotations

import re
from typing import Iterable, List, Sequence, Tuple

from docx import Document


def _is_markdown_table_line(line: str) -> bool:
    stripped = line.strip()
    return stripped.startswith("|") and stripped.endswith("|") and "|" in stripped[1:-1]


def _parse_markdown_table(lines: Sequence[str]) -> List[List[str]]:
    rows: List[List[str]] = []
    for raw_line in lines:
        stripped = raw_line.strip().strip("|")
        columns = [col.strip() for col in stripped.split("|")]
        rows.append(columns)

    def is_divider(row: List[str]) -> bool:
        return all(re.fullmatch(r":?-{3,}:?", col.replace(" ", "")) for col in row)

    filtered_rows = [row for row in rows if not is_divider(row)]
    column_count = len(filtered_rows[0]) if filtered_rows else 0
    normalized_rows = [row if len(row) == column_count else row + [""] * (column_count - len(row)) for row in filtered_rows]
    return normalized_rows


def create_docx(results: Iterable[Tuple[str, str]]) -> Document:
    """
    Build a docx Document containing the generated content for each artifact.
    """
    document = Document()
    compiled_results = list(results)

    for index, (artifact, content) in enumerate(compiled_results):
        document.add_heading(str(artifact), level=1)
        lines = content.splitlines() if content else [""]

        index = 0
        while index < len(lines):
            stripped = lines[index].strip()
            if not stripped:
                document.add_paragraph("")
                index += 1
                continue

            if _is_markdown_table_line(lines[index]):
                table_lines = []
                while index < len(lines) and _is_markdown_table_line(lines[index]):
                    table_lines.append(lines[index])
                    index += 1
                table_data = _parse_markdown_table(table_lines)
                if table_data:
                    table = document.add_table(rows=len(table_data), cols=len(table_data[0]))
                    table.style = "Table Grid"
                    for row_idx, row_data in enumerate(table_data):
                        for col_idx, cell_text in enumerate(row_data):
                            cell = table.cell(row_idx, col_idx)
                            cell.text = cell_text
                            if row_idx == 0:
                                for paragraph in cell.paragraphs:
                                    for run in paragraph.runs:
                                        run.bold = True
                    document.add_paragraph("")
                continue

            if stripped.startswith("- "):
                document.add_paragraph(stripped[2:], style="List Bullet")
            elif re.match(r"\d+\.\s+", stripped):
                numbered_text = re.sub(r"^\d+\.\s+", "", stripped)
                document.add_paragraph(numbered_text, style="List Number")
            else:
                document.add_paragraph(stripped)
            index += 1

        if index < len(compiled_results) - 1:
            document.add_page_break()

    return document
