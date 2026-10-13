---
name: respeecher-tts-tuning
description: Tune how Respeecher TTS sounds using sampling parameters (temperature, top_p, top_k, min_p, seed, penalties). Use whenever the user wants speech to be more stable, more expressive, reproducible, less repetitive or glitchy, asks what a Respeecher sampling parameter does, or wants to override a voice's default parameters.
---

# Tuning Respeecher TTS with sampling parameters

Sampling parameters shape the generated audio. Every voice ships with tuned defaults; you override only what you need, per request, in the `voice` object. Guide: https://space.respeecher.com/docs/space/api/tts/sampling-params-guide

First rule out text problems: mispronunciations, odd stress and pacing are usually fixed by changing the *text* (see the `respeecher-tts` skill), not the parameters.

## Where to set them

1. Read the voice's defaults: `GET https://api.respeecher.com/v1/public/tts/<model>/voices` (header `X-API-Key`). Each voice has `sampling_params`.
2. Override in the request. Unspecified parameters keep that voice's defaults:

```json
{
  "transcript": "Hello, World!",
  "voice": { "id": "samantha", "sampling_params": { "temperature": 0.22 } }
}
```

Works on Bytes, SSE and WebSocket (and inside the VAPI `X-Voice-JSON` header / Ultravox `voice` body).

## Parameters and valid ranges

| Parameter | Effect | Range |
| --- | --- | --- |
| `temperature` | Randomness. Lower = more stable and monotone; higher = more expressive but riskier. | ≥ 0 |
| `top_p` | Sample from the smallest token set above this cumulative probability. Lower = more focused. | (0, 1] |
| `top_k` | Consider only the k most likely tokens. Lower = more focused. `-1` = all tokens. | -1 or > 0 |
| `min_p` | Minimum probability threshold. Higher = fewer tokens considered. | [0, 1] |
| `repetition_penalty` | Penalizes tokens already generated; more for likely tokens. | [1, 2] |
| `frequency_penalty` | Penalizes by how often a token (phoneme) has appeared; stops overused sounds. | [0, 2] |
| `presence_penalty` | Penalizes tokens that have appeared at all, regardless of likelihood. | [0, 2] |
| `seed` | Same parameters **including a non-null seed** give identical output. | integer |

Out-of-range values are rejected, so clamp them.

## Procedure

1. Fetch the voice's defaults and record them as the baseline.
2. Fix a `seed` so comparisons are fair; change **one** parameter at a time. The docs say the right set is largely trial and error, so keep steps small.
3. Typical moves:
   - Too flat or robotic → raise `temperature` slightly.
   - Unstable, glitchy, wandering delivery → lower `temperature` (and/or `top_p`/`top_k`).
   - Repeated sounds, stutter, looping → raise `frequency_penalty` or `repetition_penalty`.
   - Need the exact same take again → pass the same `seed` and parameters.
4. When happy, remove the fixed `seed` for production unless reproducibility is needed. Keep only the overrides that differ from the defaults.

These are starting points, not guarantees; the effect varies by voice and model, so verify by listening. Don't invent "recommended values" for a voice; derive them from that voice's defaults.

## Pitfalls

- Overriding all parameters hides future improvements to a voice's defaults; override minimally.
- Defaults differ per voice (for example `temperature` 0.6 vs 0.7), so don't copy one voice's values to another.
- Changing `seed` changes the take even with identical text.
