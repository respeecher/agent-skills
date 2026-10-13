---
name: respeecher-space-tts
description: Integrate Respeecher's Space API for text-to-speech — pick the endpoint (Bytes, SSE, WebSocket), authenticate, list voices, choose a language model, generate and stream audio, and handle limits. Use whenever the user wants to call Respeecher TTS from code or curl, build streaming/realtime speech, uses the Respeecher Python or TypeScript SDK, or hits errors, truncated audio or concurrency limits with the Respeecher API.
---

# Respeecher Space API (text-to-speech)

Docs index for agents: https://space.respeecher.com/docs/llms.txt (append `.md` to any docs page URL for clean Markdown). Check the live page before relying on exact schemas or limits below.

## Step 0: is this the right API?

Respeecher has two separate APIs. Keys are **not** interchangeable.

| | Space API (this skill) | Marketplace API |
| --- | --- | --- |
| For | Text → speech, realtime streaming | Speech → speech, projects/orders |
| Base URL | `https://api.respeecher.com` | `https://gateway.respeecher.com` |
| Auth header | `X-API-Key` | `api-key` |
| Key from | https://space.respeecher.com/api-keys | https://marketplace.respeecher.com/account |

"Turn this text into audio" → Space API. Speech-to-speech conversion → Marketplace API (not covered here).

## Setup

- Create an API key at https://space.respeecher.com/api-keys. Send it as `X-API-Key` on every request, **including the WebSocket handshake**. Read it from an environment variable; never hardcode it.
- SDKs: `pip install respeecher` (`from respeecher import Respeecher`), or `npm install @respeecher/respeecher-js` (`import { RespeecherClient } from "@respeecher/respeecher-js"`). Usage examples: the live-API tests in https://github.com/respeecher/respeecher-python and https://github.com/respeecher/respeecher-js.

## Choose a model (language)

The model is part of the base URL: `https://api.respeecher.com/v1/public/tts/<model>`. Each has one primary language and limited support for short passages in others.

| Language | `<model>` |
| --- | --- |
| English (SDK default) | `en-rt` |
| Ukrainian | `ua-rt` |
| Arabic | `ar-rt` |
| German | `de-rt` |
| Brazilian Portuguese | `pt-br-rt` |

Voice IDs differ per model, so list voices from the same model you will generate with. Python SDK: `Respeecher(environment=RespeecherEnvironment.PUBLIC_UA_RT)`.

## List voices

```bash
curl https://api.respeecher.com/v1/public/tts/en-rt/voices -H "X-API-Key: $RESPEECHER_API_KEY"
```

Returns `[{id, full_name?, gender?, accent?, age?, sampling_params?}]`. Use `id` as `voice.id`. There is **no voice cloning and no public cloning API**; custom voices are by request via support@respeecher.com. Never suggest cloning.

## Choose an endpoint

| Endpoint | Use when | Notes |
| --- | --- | --- |
| **Bytes** `POST …/tts/bytes` | Simple, offline, ≤ ~5,000 characters | Returns a WAV (16-bit PCM). `output_format` takes only `sample_rate`. Not for latency-sensitive use. |
| **SSE** `POST …/tts/sse` | Streaming over plain HTTP, long text | JSONL stream; no input streaming. |
| **WebSocket** `wss://…/tts/websocket` | Realtime, voice agents, best latency | Many concurrent generations per connection, input and output streaming. |

Request body (all three): `{"transcript": "...", "voice": {"id": "samantha", "sampling_params": {...}?}, "output_format": {...}?}`. Default sample rate is 22050 Hz.

### Bytes

```python
import os, requests

r = requests.post(
    "https://api.respeecher.com/v1/public/tts/en-rt/tts/bytes",
    headers={"X-API-Key": os.environ["RESPEECHER_API_KEY"]},
    json={"transcript": "Hello, World!", "voice": {"id": "samantha"}},
    timeout=130,  # server-side limit is 120 s
)
r.raise_for_status()
open("result.wav", "wb").write(r.content)
```

### SSE

Each line is JSON: `{"type":"chunk","data":"<base64 raw PCM>"}` or `{"type":"error","error":"...","status_code":N,"context_id"?}`. Set `output_format.encoding` explicitly (`pcm_f32le`, `pcm_s16le`, `pcm_mulaw`) and decode with that encoding and the sample rate. Check for `error` events; HTTP 200 does not guarantee success mid-stream. To save a file, decode and concatenate chunks, then write a WAV (the quickstart does this with numpy + soundfile).

### WebSocket

Handshake with `X-API-Key`. Messages:

- Client → server, generate: `{"transcript": "...", "voice": {"id": "..."}, "context_id": "c1", "continue": false, "output_format": {...}?}`
- Client → server, cancel: `{"context_id": "c1", "cancel": true}` (already generated but unstreamed audio is still charged)
- Server → client: `{"type":"chunk","data":"<base64 PCM>","context_id"}`, `{"type":"done","context_id"}`, `{"type":"error","error","status_code","context_id"?}`

Best practices (from the WebSockets Guide):
- Open the connection before the first request to avoid first-generation latency; **reuse** it.
- Use a different `context_id` per generation and route incoming audio by `context_id`.
- To stream text in pieces, send chunks with `"continue": true` on all but the last; this keeps prosody consistent.
- Reopen the connection periodically (not guaranteed to live past 12 hours); close it when idle (idle connections may be closed server-side).

## Limits (verify at https://space.respeecher.com/docs/space/limits)

- 5 concurrent TTS generations per account across all endpoints (1 on free trial). Queue or throttle client-side.
- Bytes: ~5,000 characters per call and 120 s timeout; responses over 100 MB are **silently truncated**. Use SSE/WebSocket, or split the text, for anything longer.

## Preparing text

Pronunciation and pacing are controlled through the text itself. Use the `respeecher-tts` skill (or https://space.respeecher.com/docs/space/tts-input-tips) before sending scripts, especially numbers, acronyms, and Ukrainian stress marks.

## Tuning the sound

Sampling parameters (temperature, seed, …) are covered by the `respeecher-tts-tuning` skill.

## Troubleshooting

- 401/403: wrong header name or a Marketplace key (`api-key`) used on Space. Use `X-API-Key` with a Space key.
- Voice not found: the voice ID belongs to a different model than the base URL.
- Audio cut off with Bytes: text too long or response over 100 MB; switch to SSE/WebSocket.
- Rate/limit errors: more than 5 concurrent generations.
- Garbled or noisy streamed audio: the decoder's encoding or sample rate doesn't match `output_format`.
