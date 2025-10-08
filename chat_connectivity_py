from __future__ import annotations

import os
import uuid
from typing import Any

import requests

try:
    from openai import OpenAI
except ImportError:  # pragma: no cover
    OpenAI = None  # type: ignore

OPENAI_CHAT_URL = "https://api.openai.com/v1/chat/completions"


def chat_complete(
    provider: str,
    prompt: str,
    max_tokens: int = 800,
    user_id: str = "DocGenUser",
    **kwargs: Any,
) -> str:
    """
    Execute a chat completion request against either OpenAI or an internal endpoint.
    """

    if provider == "openai":
        return _openai_chat_complete(prompt, max_tokens=max_tokens, user_id=user_id, **kwargs)

    if provider == "internal":
        return _internal_chat_complete(prompt, max_tokens=max_tokens, user_id=user_id, **kwargs)

    raise ValueError(f"Unsupported provider: {provider}")


def _openai_chat_complete(prompt: str, max_tokens: int, user_id: str, **kwargs: Any) -> str:
    api_key = kwargs.get("api_key") or os.getenv("OPENAI_API_KEY")
    if not api_key:
        raise ValueError("OpenAI API key is required for provider 'openai'.")

    model = kwargs.get("model") or "gpt-3.5-turbo"
    messages = [{"role": "user", "content": prompt}]

    if OpenAI is not None:
        client = OpenAI(api_key=api_key)
        response = client.chat.completions.create(
            model=model,
            messages=messages,
            max_tokens=max_tokens,
            user=user_id,
        )
        if not response.choices:
            raise ValueError("OpenAI response did not include any choices.")
        return response.choices[0].message.content or ""

    headers = {
        "Authorization": f"Bearer {api_key}",
        "Content-Type": "application/json",
    }
    payload = {
        "model": model,
        "messages": messages,
        "max_tokens": max_tokens,
        "user": user_id,
    }
    timeout = kwargs.get("timeout", 60)
    response = requests.post(OPENAI_CHAT_URL, headers=headers, json=payload, timeout=timeout)
    response.raise_for_status()
    data = response.json()
    try:
        return data["choices"][0]["message"]["content"]
    except (KeyError, IndexError, TypeError) as exc:
        raise ValueError("OpenAI response missing choices[0].message.content") from exc


def _internal_chat_complete(prompt: str, max_tokens: int, user_id: str, **kwargs: Any) -> str:
    endpoint_url = kwargs.get("endpoint_url")
    access_token = kwargs.get("access_token")
    if not endpoint_url:
        raise ValueError("Internal provider requires an endpoint URL.")
    if not access_token:
        raise ValueError("Internal provider requires an access token.")

    model = kwargs.get("model") or "doc-gen-model"
    correlation_id = kwargs.get("correlation_id") or str(uuid.uuid4())
    timeout = kwargs.get("timeout", 60)

    headers = {
        "AMToken": access_token,
        "Token_Type": "SESSION_TOKEN",
        "x-correlation-id": correlation_id,
        "Content-Type": "application/json",
    }
    payload = {
        "model": model,
        "messages": [{"role": "user", "content": prompt}],
        "max_tokens": max_tokens,
        "user": user_id,
    }

    response = requests.post(endpoint_url, headers=headers, json=payload, timeout=timeout)
    response.raise_for_status()
    data = response.json()
    try:
        return data["choices"][0]["message"]["content"]
    except (KeyError, IndexError, TypeError) as exc:
        raise ValueError("Internal chat service response missing choices[0].message.content") from exc
