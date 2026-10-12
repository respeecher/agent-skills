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

## Claude.ai / Claude Desktop

Zip the skill folder (`skills/respeecher-tts`) and upload it under Settings → Capabilities → Skills.
