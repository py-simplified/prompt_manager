from __future__ import annotations

import re
import re
from io import BytesIO
from typing import Iterable, List, Sequence, Tuple, Union

from reportlab.lib import colors
from reportlab.lib.pagesizes import LETTER
from reportlab.lib.styles import ParagraphStyle, getSampleStyleSheet
from reportlab.platypus import PageBreak, Paragraph, SimpleDocTemplate, Spacer, Table, TableStyle


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


def create_pdf(results: Iterable[Tuple[str, str]], output_path: Union[str, BytesIO]) -> None:
    """
    Write the generated content into a PDF file at the given path or file-like object.
    """
    styles = getSampleStyleSheet()
    heading_style = styles["Heading1"]
    body_style = styles["BodyText"]
    bullet_style = styles.get("Bullet") or ParagraphStyle(
        name="Bullet",
        parent=body_style,
        leftIndent=18,
        bulletIndent=9,
    )

    doc = SimpleDocTemplate(output_path, pagesize=LETTER)
    available_width = doc.width
    story = []
    compiled_results = list(results)

    for index, (artifact, content) in enumerate(compiled_results):
        story.append(Paragraph(str(artifact), heading_style))
        lines = content.splitlines() if content else [""]

        idx = 0
        while idx < len(lines):
            stripped = lines[idx].strip()
            if not stripped:
                story.append(Spacer(1, 12))
                idx += 1
                continue

            if _is_markdown_table_line(lines[idx]):
                table_lines = []
                while idx < len(lines) and _is_markdown_table_line(lines[idx]):
                    table_lines.append(lines[idx])
                    idx += 1
                table_data = _parse_markdown_table(table_lines)
                if table_data:
                    header_style = ParagraphStyle(
                        name="TableHeader",
                        parent=body_style,
                        fontName="Helvetica-Bold",
                        alignment=0,
                    )
                    table_cells: List[List[Paragraph]] = []
                    for row_idx, row_data in enumerate(table_data):
                        row_cells = []
                        for cell_text in row_data:
                            cell_style = header_style if row_idx == 0 else body_style
                            row_cells.append(Paragraph(cell_text, cell_style))
                        table_cells.append(row_cells)
                    num_cols = len(table_cells[0])
                    col_widths = [available_width / num_cols] * num_cols if num_cols else None
                    table = Table(table_cells, colWidths=col_widths, repeatRows=1, hAlign="LEFT")
                    table.setStyle(
                        TableStyle(
                            [
                                ("BACKGROUND", (0, 0), (-1, 0), colors.lightgrey),
                                ("TEXTCOLOR", (0, 0), (-1, 0), colors.black),
                                ("ALIGN", (0, 0), (-1, -1), "LEFT"),
                                ("FONTNAME", (0, 0), (-1, 0), "Helvetica-Bold"),
                                ("FONTSIZE", (0, 0), (-1, -1), 10),
                                ("BOTTOMPADDING", (0, 0), (-1, 0), 6),
                                ("GRID", (0, 0), (-1, -1), 0.5, colors.grey),
                                ("VALIGN", (0, 0), (-1, -1), "TOP"),
                            ]
                        )
                    )
                    story.append(table)
                    story.append(Spacer(1, 12))
                continue

            if stripped.startswith("- "):
                story.append(Paragraph(stripped[2:], bullet_style))
            elif re.match(r"\d+\.\s+", stripped):
                story.append(Paragraph(stripped, body_style))
            else:
                story.append(Paragraph(stripped, body_style))
            idx += 1

        if index < len(compiled_results) - 1:
            story.append(PageBreak())

    doc.build(story)
