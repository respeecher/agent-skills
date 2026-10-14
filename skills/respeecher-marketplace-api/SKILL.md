---
name: respeecher-marketplace-api
description: Use Respeecher's Marketplace API for speech-to-speech (STS) voice conversion and project-based workflows — authenticate, create projects and folders, upload recordings, calibrate pitch, pick voices/accents, create conversion orders, poll and download results, export projects. Use whenever the user wants to convert existing audio to another Respeecher voice or accent by API, automate batch conversions, or call gateway.respeecher.com.
---

# Respeecher Marketplace API (speech-to-speech)

Docs: https://space.respeecher.com/docs/marketplace/api-overview.md and https://space.respeecher.com/docs/marketplace/endpoints-reference.md (full schemas; read them rather than guessing fields). Index for agents: https://space.respeecher.com/docs/llms.txt

## Step 0: is this the right API?

- Converting **existing audio** from one voice/accent to another, or working in projects/folders/recordings → **this skill**.
- Generating speech from **text**, especially realtime or voice agents → use the **Space API** (`respeecher-space-tts` skill). It is faster and has SDKs. The Marketplace API can also do TTS, but only as a project-based workflow.
- The two APIs use **different keys and headers**; they are not interchangeable.

| | Marketplace API | Space API |
| --- | --- | --- |
| Base URL | `https://gateway.respeecher.com` | `https://api.respeecher.com` |
| Auth header | `api-key` | `X-API-Key` |
| Key from | https://marketplace.respeecher.com/account | https://space.respeecher.com/api-keys |

## Authentication

Preferred: **API key**. Generate it at https://marketplace.respeecher.com/account (choose an expiry), then send it on every authenticated request:

```
api-key: <your key>
```

Keys can also be created/deleted via `POST /api/api-key` (body `{"expires_at": "<ISO 8601>"}`) and `DELETE /api/api-key`. Read the key from an environment variable; never commit it.

Alternative: session cookie. `POST /api/login` with `{"email","password"}` returns a `session_id` cookie and `csrf_token`; send the cookie plus `X-Csrf-Token` on later requests. Prefer API keys for scripts.

## Workflow (speech-to-speech)

All paths are under `https://gateway.respeecher.com`.

1. **Project**: `POST /api/projects` `{"name": "...", "owner": "<user-or-group-id>"}`. The user ID comes from the login response; `models` is no longer used.
2. **Folder**: `POST /api/folders` `{"name": "...", "project_id": "..."}` (both required).
3. **Original recording**: `POST /api/recordings`, `multipart/form-data` with `data` (wav, ogg, mp3 or flac), `parent_folder_id`, `microphone` (use `file` when uploading a file) and optional `label`.
4. **Calibration** (optional but advisable for speech-to-speech): `POST /api/calibration`, multipart `name` (unique) + `data`. It takes a few minutes; check `GET /api/calibration` until `state` is no longer `processing`. Calibration measures the source speaker's mean pitch so the model can shift pitch correctly. If skipped, automatic calibration is used. Set the default with `PUT /api/calibration/{id}/enable` or `POST /api/calibration/auto`. Not needed for TTS-only conversions.
5. **Pick a voice and accent**: `GET /api/v2/voices` (filters: `limit` default 25, `offset`, `sort`, `direction`, `visibility`, `species`, `gender`, `age_group`, `pitch_group`, `nationality`). Each voice lists `accents` (used for STS) and `narration_styles` (used for TTS). `GET /api/v2/voices/search?name=` finds a voice by name. Check `available` and the tiers against the account.
6. **Order**: `POST /api/v2/orders`:

```json
{
  "original_id": "<recording id>",
  "conversions": [
    { "voice_id": "...", "accent_id": "...", "semitones_correction": 0, "label": "..." }
  ],
  "calibration_id": "...",
  "use_calibration": true
}
```

   - `accent_id`, `semitones_correction`, `label` are optional. Final pitch = your voice's pitch + the voice's default shift + `semitones_correction`; keep it inside the target voice's natural range for quality.
   - `use_calibration: true` + `calibration_id` selects a specific calibration; otherwise the account's enabled calibration is used.
   - Retry a failed order with `POST /api/v2/orders/retry/{recording_id}`.
7. **Wait for completion**: list conversions with `GET /api/recordings/conversions?original_id=...` (or `GET /api/recordings/{id}`) and poll until `state` is `done`; failures carry `error`. Poll politely (see rate limits).
8. **Download**: when `state` is `done`, `url` (e.g. `/storage/<id>.wav`) is the file: `GET https://gateway.respeecher.com<url>` with the same auth.
9. **Export**: `GET /api/projects/{project_id}/export?starred_only=false` returns a `.zip` (`Content-Disposition` has the filename). For one original's conversions: `GET /api/recordings/export?original_id=...&starred_only=...`.

## Text-to-speech through the Marketplace API

Only when the project-based workflow is wanted:
1. Create the recording from text: `POST /api/v2/recordings/tts` `{"parent_folder_id", "text", "text_language"?, "label"?}`. `text_language` (ISO 639) is auto-detected if omitted. (`POST /api/recordings/tts` is **deprecated**.)
2. Create the order with a `narration_style_id` per conversion (**required for TTS**; without it the result sounds robotic). Narration styles come from the voice object.

For preparing the text itself, see the `respeecher-tts` skill.

## Other endpoints worth knowing

`GET /api/credits` (credits left), `GET /api/stats` (usage), recordings list/update (`PATCH /api/recordings/{id}` with `label`, `starred`), folder and project rename/delete, notes (`/api/notes`), `GET /api/tts-voices` (legacy TTS voices). **Delete endpoints are destructive**; confirm with the user before calling them on real projects.

## Rate limits

Per user, per 300 seconds: `fast` 500 requests, `slow` 100 requests. Slow endpoints include `POST /api/recordings`, `POST /api/recordings/tts`, `POST /api/calibration` and conversion orders. On HTTP 429 wait for the `Retry-After` header (seconds) before retrying. Limits can be raised on request.

## Pro Tools

Avid Pro Tools users can convert in-DAW with the AudioSuite plugin (beta; needs a Marketplace plan that includes speech-to-speech) — see https://space.respeecher.com/docs/marketplace/pro-tools-plugin. No API work is needed for that.

## Python

The docs point to an official Python client for the Marketplace TTS API: https://github.com/respeecher/respeecher_tts. Check its README before hand-rolling requests in Python.

## Troubleshooting

- 401: wrong header (`X-API-Key` instead of `api-key`), a Space key, or an expired key.
- 429: slow/fast limit hit; honor `Retry-After`.
- Robotic TTS result: missing `narration_style_id`.
- Poor pitch or quality: add or enable a calibration; adjust `semitones_correction` within the voice's range.
- Upload rejected: only wav, ogg, mp3 and flac are supported.
