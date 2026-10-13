# Respeecher agent skills

Skills for AI coding agents (Claude Code, Cursor, Codex, ...) that help you work with Respeecher.

## Install

```bash
npx skills add respeecher/agent-skills
```

Install a single skill:

```bash
npx skills add respeecher/agent-skills --skill respeecher-tts
```

## Skills

| Skill | What it does |
| --- | --- |
| [`respeecher-tts`](skills/respeecher-tts/SKILL.md) | Rewrites text so it is pronounced correctly by Respeecher TTS: numbers as words, acronyms, phonetic spelling, pauses, and stress marks for the Ukrainian model. Based on the [TTS input tips](https://space.respeecher.com/docs/space/tts-input-tips). |
| [`respeecher-space-tts`](skills/respeecher-space-tts/SKILL.md) | Integrate the Space API: choose Bytes, SSE or WebSocket, authenticate, pick a language model, stream audio, and stay within limits. |
| [`respeecher-voice-agents`](skills/respeecher-voice-agents/SKILL.md) | Use Respeecher as the TTS provider in LiveKit, Pipecat, Ultravox or VAPI. |
| [`respeecher-tts-tuning`](skills/respeecher-tts-tuning/SKILL.md) | Tune sound with sampling parameters (temperature, top_p, seed, penalties). |

## Claude.ai / Claude Desktop

Zip the skill folder (`skills/respeecher-tts`) and upload it under Settings → Capabilities → Skills.
