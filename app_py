from __future__ import annotations

import os
from io import BytesIO
from typing import List, Tuple

import streamlit as st
from dotenv import load_dotenv

from chat_connectivity import chat_complete
from utils.docx_utils import create_docx
from utils.file_utils import read_prompts, read_requirement
from utils.pdf_utils import create_pdf

load_dotenv()

st.set_page_config(page_title="Prompt Document Generator", layout="wide")
if "generation_results" not in st.session_state:
    st.session_state["generation_results"] = []

st.title("Prompt Document Generator")
st.markdown(
    "Upload a requirement document and a prompt template file to generate structured outputs. "
    "Each prompt will be executed with the requirement content to create the final documents."
)

with st.sidebar:
    st.header("Generation Settings")
    provider = st.radio("Provider", options=["openai", "internal"], index=0)
    max_tokens = st.number_input("Max tokens", min_value=100, max_value=2000, value=800, step=50)

    if provider == "openai":
        model_name = st.text_input("Model name", value="gpt-3.5-turbo", key="openai_model")
        st.caption("Leave the API key blank to use the OPENAI_API_KEY value from your environment.")
        openai_key_input = st.text_input("OpenAI API Key", type="password", key="openai_api_key")
        internal_token = ""
        internal_endpoint = ""
    else:
        model_name = st.text_input("Model name", value="doc-gen-model", key="internal_model")
        internal_token = st.text_input("AP Token", type="password", key="internal_token")
        internal_endpoint = st.text_input("Internal Endpoint URL", key="internal_endpoint")
        openai_key_input = ""

requirement_file = st.file_uploader(
    "Requirement document (.txt, .md, .pdf, .docx)",
    type=["txt", "md", "pdf", "docx"],
    accept_multiple_files=False,
)
prompt_file = st.file_uploader(
    "Prompt definitions (.csv or .xlsx with Artifact and Prompt columns)",
    type=["csv", "xlsx"],
    accept_multiple_files=False,
)

generate_clicked = st.button("Generate Outputs", disabled=not requirement_file or not prompt_file)

results: List[Tuple[str, str]] = list(st.session_state.get("generation_results", []))

if generate_clicked:
    if not requirement_file:
        st.error("Please upload a requirement document before generating outputs.")
    elif not prompt_file:
        st.error("Please upload a prompt file before generating outputs.")
    else:
        try:
            requirement_text = read_requirement(requirement_file)
        except ValueError as exc:
            st.error(str(exc))
            st.stop()

        try:
            prompts_df = read_prompts(prompt_file)
        except ValueError as exc:
            st.error(str(exc))
            st.stop()

        if prompts_df.empty:
            st.warning("The prompt file is empty. Add at least one prompt to continue.")
            st.stop()

        provider_kwargs = {"model": model_name}
        if provider == "openai":
            api_key = openai_key_input.strip() or os.getenv("OPENAI_API_KEY")
            if not api_key:
                st.error("Please provide an OpenAI API key via the input field or environment variable.")
                st.stop()
            provider_kwargs["api_key"] = api_key
        else:
            if not internal_token:
                st.error("Please provide an AP Token for the internal provider.")
                st.stop()
            if not internal_endpoint:
                st.error("Please provide the Internal Endpoint URL for the internal provider.")
                st.stop()
            provider_kwargs["access_token"] = internal_token
            provider_kwargs["endpoint_url"] = internal_endpoint

        with st.spinner("Generating content..."):
            new_results: List[Tuple[str, str]] = []
            for row in prompts_df.itertuples(index=False):
                artifact = str(row.Artifact)
                prompt_template = str(row.Prompt)
                composed_prompt = f"{prompt_template}\n\nRequirement:\n{requirement_text}"
                try:
                    response_text = chat_complete(
                        provider=provider,
                        prompt=composed_prompt,
                        max_tokens=max_tokens,
                        user_id="DocGenUser",
                        **provider_kwargs,
                    )
                except Exception as exc:  # pragma: no cover - surfaced to user
                    st.error(f"Failed to generate output for '{artifact}': {exc}")
                    st.stop()
                new_results.append((artifact, response_text.strip()))

        results = new_results
        st.session_state["generation_results"] = new_results
        st.success("Generation complete.")

if results:
    for artifact, content in results:
        st.subheader(artifact)
        st.markdown(content or "_No content returned._")

    docx_document = create_docx(results)
    docx_buffer = BytesIO()
    docx_document.save(docx_buffer)
    docx_buffer.seek(0)

    pdf_buffer = BytesIO()
    create_pdf(results, pdf_buffer)
    pdf_buffer.seek(0)

    col_docx, col_pdf = st.columns(2)
    with col_docx:
        st.download_button(
            "Download combined DOCX",
            data=docx_buffer.getvalue(),
            file_name="prompt_outputs.docx",
            mime="application/vnd.openxmlformats-officedocument.wordprocessingml.document",
        )
    with col_pdf:
        st.download_button(
            "Download combined PDF",
            data=pdf_buffer.getvalue(),
            file_name="prompt_outputs.pdf",
            mime="application/pdf",
        )
