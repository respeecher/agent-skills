---
name: respeecher-tts
description: Prepare, rewrite or check text for Respeecher TTS (text-to-speech) so it is pronounced correctly — phonetic respelling, numbers as words, acronyms, punctuation for pacing, sentence splitting, symbol removal, and stress marks for the Ukrainian model. Use whenever the user wants to write, clean up, or fix a script/text/voiceover for Respeecher or TTS synthesis, asks why a word is mispronounced, or asks to "make this TTS-ready".
---

# Respeecher TTS text preparation

The model reads plain text. Everything you control about pronunciation, stress and rhythm goes
through the text itself, so rewrite the input using the rules below. Never change the meaning,
facts or tone — only the *form* the model sees.

Source of truth: https://space.respeecher.com/docs/space/tts-input-tips

## Rules

| Rule | Applies to |
| --- | --- |
| 1. Phonetic spelling | All models |
| 2. Acronyms in capitals | All models |
| 3. Numbers as words | All models |
| 4. Punctuation for pauses | All models |
| 5. Split long sentences | All models **except Ukrainian** |
| 6. Avoid special symbols | All models |
| 7. Stress marks | **Ukrainian only** |

### 1. Phonetic spelling for words read wrong
If a word (foreign name, brand, loanword, jargon) is mispronounced, replace it with a phonetic
spelling in the script of the output language.

- `Aoife` → `ee-fah`.
- Ukrainian: write the sound in Cyrillic, e.g. `Nginx` → `Енджинікс`, `Leicester` → `Лестер`.
- Keep the original spelling in your explanation to the user so they know what was substituted.

### 2. Acronyms in capitals
If an acronym isn't read correctly, write it in capital letters, separate the letters with
periods, or both.

- `brb` → `BRB` or `B.R.B.`; `нбу` → `НБУ` or `Н.Б.У.`
- Only touch acronyms that are likely to be misread. Short forms like `т.д.`, `напр.`, `e.g.`,
  `Dr.` are safest spelled out (`так далі`, `наприклад`, `for example`, `Doctor`).

### 3. Numbers as words
Write numbers in words rather than numerals — this gives more stable results. This includes
money, dates, years, percentages, ordinals, phone numbers and units.

- `My dad's yacht cost $1,000,000.` → `My dad's yacht cost one million dollars.`
- Ukrainian: agree case and gender — `у дві тисячі двадцять шостому році`, `двадцять одна гривня`,
  `п'ять відсотків`.
- Expand symbols: `%` → відсотків/percent, `$` → доларів/dollars, `№` → номер, `°C` → градусів
  Цельсія, `км/год` → кілометрів на годину.

### 4. Punctuation for pauses
Commas, dashes and ellipses change the rhythm and add natural pauses.

- `I get where you're coming from, but I... um... don't know what to say.`
- Add them where the speaker would naturally pause or hesitate; don't pepper every sentence.

### 5. Split long sentences (not Ukrainian)
Long or complex sentences are harder to read clearly. Split them into shorter ones.

- `He arrived late because the train was delayed and stuck outside the station for over an hour.`
  → `He arrived late because the train was delayed. It was stuck outside the station for over an hour.`
- **Skip this rule for the Ukrainian model** — keep the original sentence structure.

### 6. Avoid special symbols
Replace symbols like `( )`, `@`, `#`, `"`, `$` with words, or rephrase without them.

- `He works (as does his brother) at the bank.` → `He works at the bank. His brother works there too.`
- `Our #1 priority is safety.` → `Our number one priority is safety.`
- Apply the same idea to brackets, slashes (`і/або` → `і або`), bullet symbols, emoji, URLs
  (spell out or drop) and markdown formatting.

### 7. Stress marks (Ukrainian model only)
Stress marks control where the stress falls in a word. Useful for homographs and names the model
might otherwise guess wrong. **Do not add stress marks for any other language or model.**

Place the mark **immediately before the vowel** that should be stressed. Three equivalent forms:

- Plus sign: `йог+урт` (default choice — easiest to read and copy)
- Stress token: `йог<stress>урт`
- Unicode combining acute accent (U+0301) after the vowel: `йогу́рт`

Pick one form and use it consistently within a text.

- Mark homographs (`з+амок`/`зам+ок`, `м+ука`/`мук+а`, `д+орога`/`дорог+а`, `+атлас`/`атл+ас`),
  rare words, proper names, surnames, toponyms, and any word the user reports as mis-stressed.
  Don't mark every word.
- If a single mark isn't honored, repeat it: `йог++урт` or `йог<stress><stress>урт`. Repeats only
  increase how strongly the stress is enforced; they never add a secondary stress. Start with one
  and add more only if needed. **No more than three repeats.**
- Never use a stress mark on a word that already has one.

## Workflow

1. Detect the language(s) of the text and which model it targets (stress marks and sentence
   splitting depend on it; if unknown and the text is Ukrainian, assume the Ukrainian model).
2. Apply rules in this order: symbols/markup (6) → split sentences (5, non-Ukrainian only) →
   numbers and symbols (3) → acronyms (2) → phonetic respelling (1) → stress marks (7, Ukrainian
   only) → pacing punctuation (4).
3. Output the **TTS-ready text in a single code block** (so `+` marks and spacing survive
   copy-paste).
4. Below it, list the non-obvious changes briefly (stresses added, words respelled, numbers
   expanded), so the user can revert any they disagree with.
5. If the user reports a specific mispronunciation, fix only that (stress → mark, repeating if it
   didn't take; otherwise phonetic respelling or acronym dots) and don't rewrite the rest.
