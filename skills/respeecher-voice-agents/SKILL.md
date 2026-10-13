---
name: respeecher-voice-agents
description: Use Respeecher as the text-to-speech provider in a voice-agent platform — LiveKit Agents, Pipecat, Ultravox or VAPI. Use whenever the user wants to set up, configure or debug Respeecher voices in one of these platforms, or asks how to add Respeecher TTS to a realtime voice agent.
---

# Respeecher in voice-agent platforms

Respeecher's Space API provides realtime TTS. Get a key at https://space.respeecher.com/api-keys (sent as `X-API-Key`; see the `respeecher-space-tts` skill for endpoints and limits). Integration docs: https://space.respeecher.com/docs/space/integrations/<livekit|pipecat|ultravox|vapi>.md

Common rules:
- Voices are licensed from professional voice actors. **There is no voice cloning**, in any integration. Never suggest cloning a voice here.
- List the voices available to the account first (`GET https://api.respeecher.com/v1/public/tts/<model>/voices`) and use a voice ID from that same model. Models: `en-rt` (English), `ua-rt`, `ar-rt`, `de-rt`, `pt-br-rt`.
- Max 5 concurrent generations per account (1 on free trial). Many simultaneous calls will hit this.
- Don't paste real API keys into files that get committed; use the platform's secret storage or env vars.

## LiveKit Agents

Official plugin: `pip install livekit-plugins-respeecher` — a TTS node that drops into any LiveKit Agents pipeline.
Follow the plugin README for the exact class names and options: https://github.com/respeecher/livekit-plugins-respeecher#readme (read it rather than guessing the API).

## Pipecat

Official plugin: `pip install pipecat-respeecher` — a TTS service that drops into any Pipecat pipeline.
README: https://github.com/respeecher/pipecat-respeecher#pipecat-respeecher-real-time-tts-integration

## Ultravox (bring-your-own TTS, via the SSE endpoint)

1. In https://app.ultravox.ai/voices choose **Import voice**. Do **not** use "Create custom" — that opens Ultravox's cloning panel, which defaults to ElevenLabs and can't be used with Respeecher.
2. Paste this JSON, replacing `<API_KEY>`:

```json
{
  "url": "https://api.respeecher.com/v1/public/tts/en-rt/tts/sse",
  "headers": { "X-Api-Key": "<API_KEY>", "Content-Type": "application/json" },
  "body": {
    "transcript": "{text}",
    "voice": { "id": "samantha" },
    "output_format": { "encoding": "pcm_s16le" }
  },
  "responseSampleRate": 22050,
  "jsonAudioFieldPath": "data",
  "responseMimeType": "application/jsonl"
}
```

3. Use the Test button, then save. Change `en-rt` and the voice ID for other models/voices; `voice` may also hold `sampling_params`.

## VAPI (custom TTS endpoint)

1. Create a custom credential at https://dashboard.vapi.ai/settings/integrations/custom-credential: type **Bearer Token**, token = Respeecher API key, header name `X-Api-Key`, **Include Bearer Prefix: No**. Note the credential **ID** (not the name).
2. PATCH the assistant with the VAPI private key (not the Respeecher key):

```bash
curl -X PATCH https://api.vapi.ai/assistant/<ASSISTANT_ID> \
  -H "Authorization: Bearer <VAPI_API_KEY>" -H "Content-Type: application/json" \
  -d '{
    "voice": {
      "provider": "custom-voice",
      "server": {
        "url": "https://api.respeecher.com/v1/public/tts/en-rt/tts/vapi",
        "credentialId": "<CREDENTIAL_ID>",
        "timeoutSeconds": 30,
        "headers": { "X-Voice-JSON": "{\"id\": \"vikram\"}" }
      }
    }
  }'
```

3. To switch model or voice, change `en-rt` in the URL or the JSON string in `X-Voice-JSON` (same shape as the `voice` object in other TTS endpoints, can include `sampling_params`).

Known VAPI quirks (from the docs):
- The graphical assistant builder may display the custom voice wrongly; calls still work. To use the builder, temporarily revert to a built-in voice, then re-run the PATCH.
- `firstMessageMode: assistant-speaks-first` may misbehave with custom TTS; try `assistant-speaks-first-with-model-generated-message` or `assistant-waits-for-user`.

## Voice-agent text tips

Agent LLM output is spoken verbatim, so instruct the agent to avoid markdown, emoji and symbols, to write numbers and acronyms the way they should be spoken, and to keep sentences short. See the `respeecher-tts` skill for the rules (and for Ukrainian stress marks).
