---
name: vocab-daily
description: Generate and save daily CELPIP vocabulary study sheets from the local vocabulary files, quiz the user from saved day sheets, or guide the user through a saved day word by word with adaptive explanations and persisted notes.
---

# Vocab Daily

Use this skill for three related jobs:
- generate a daily CELPIP vocabulary study sheet and save it under the project
- quiz the user from an existing saved day sheet
- guide the user through a saved day word by word, adapt to the user's self-rating, and persist the learning notes back into the saved day sheet

## Files To Use

Read these local files:
- `vocabulary/400_words.md`
- `vocabulary/200_vocab.md`
- `vocabulary/200_phrasal_verbs.md`

Use this file as the style reference:
- `vocabulary/notes.md`

Save generated study sheets here:
- `vocabulary/daily_study/day_XX.md`

## Primary Mode: Generate Day N

When the user asks for a daily vocab sheet, determine the day number `N` and generate one saved file for that day.

Selection rules:
- From `400_words.md`, use items `(N-1)*40 + 1` through `N*40`
- From `200_vocab.md`, use items `(N-1)*20 + 1` through `N*20`
- From `200_phrasal_verbs.md`, use items `(N-1)*20 + 1` through `N*20`

Examples:
- Day 1: `1-40`, `1-20`, `1-20`
- Day 2: `41-80`, `21-40`, `21-40`
- Day 10: `361-400`, `181-200`, `181-200`

If numbering is irregular:
- Preserve source order
- Extract the next valid entries until the quota for that section is filled
- Do not reorder entries

## Study Sheet Format

Write the output in Markdown.

Use this section structure:
- `# Day XX CELPIP Vocabulary`
- `## From 400_words.md`
- `## From 200_vocab.md`
- `## From 200_phrasal_verbs.md`

For each entry, include only:
- word or phrase
- short meaning in simple English
- one natural CELPIP-style example sentence
- one brief usage note only when needed

Keep each entry brief. The sheet is for fast daily study, not deep dictionary-style explanation.

Preferred entry format:

```md
1. **word / phrase**: short meaning
  - Explanation: short meaning and usage in simple English.
  - 中文意思: short Chinese support for the meaning.
  - 中文语感: short Chinese support for nuance or usage when helpful.
  - Usage note: only when needed.
  - Examples:
    - one natural CELPIP-style sentence.
```

When an entry later gets guided-study notes, extend the same entry instead of creating a separate document.

## Style Rules

Follow `vocabulary/notes.md` for usage style.

Prefer:
- natural and practical English
- CELPIP-friendly topics such as daily life, workplace, housing, school, services, travel, and opinions
- clear sentence patterns and useful connectors when appropriate

Avoid:
- awkward “advanced” wording
- overly academic or literary examples
- long explanations
- fake idiomaticity

When improving or selecting examples:
- prioritize natural usage over difficulty
- preserve the core meaning of the word
- make examples useful for speaking and writing, not just recognition

## File Handling

Before saving a day sheet:
- ensure `vocabulary/daily_study/` exists

Save as:
- Day 1 -> `vocabulary/daily_study/day_01.md`
- Day 2 -> `vocabulary/daily_study/day_02.md`

Use zero-padded day numbers.

If the target day file already exists:
- overwrite it only if the user explicitly asks to regenerate or update it
- otherwise, read the existing file and continue from it

## Secondary Mode: Quiz From Saved Sheet

If the user asks for a quiz:
- read the requested file from `vocabulary/daily_study/day_XX.md`
- generate the quiz live from that saved file
- do not save quiz files

Good quiz types:
- meaning recall
- reverse meaning recall
- example gap-fill
- "use this word in a CELPIP-style sentence"

When quizzing:
- rely on the saved day sheet instead of going back to the original source lists unless the saved file is missing

## Tertiary Mode: Guided Study From Saved Sheet

Use this mode when the user wants to study the daily words one by one, discuss each word interactively, rate confidence, or save explanations/examples while learning.

Workflow:
- determine the requested day number
- read `vocabulary/daily_study/day_XX.md`
- if the file is missing, generate it first using the primary mode, then continue
- study one entry at a time in saved-file order
- do not jump ahead or dump multiple words at once unless the user explicitly asks

For each word:
1. prompt the word or phrase first
2. ask the user for a confidence rating from `0` to `3`
3. adapt the response based on the rating
4. give the explanation, nuance, and example sentences together in one response, with explanation and nuance before examples
5. include a short Chinese section for the meaning and nuance in the live teaching response
6. continue refining only if the user says they still need help
7. persist the explanation and useful examples into the day markdown before moving to the next word

Chinese support rules:
- add `中文意思:` after the main English meaning/explanation
- add `中文语感:` after the nuance or usage distinction when nuance is given
- keep both Chinese parts brief, natural, and practical
- use Chinese only as learning support; the main teaching should still be English-first
- for rating `3`, Chinese can be minimal and only included if it helps clarify a subtle distinction

Rating behavior:
- `0`: assume the user does not know the word; give a very simple meaning, plain-language explanation, a brief `中文意思`, common context, and one easy contrast or synonym if helpful
- `1`: assume partial recognition; give a concise explanation focused on usage, a brief `中文意思`, and a common context
- `2`: assume the user mostly knows it; focus on nuance, collocations, when to choose this word over a simpler one, and add a brief `中文语感`
- `3`: assume the user has mastered it; keep the explanation minimal, give only one example, and save the word directly once the user confirms they are good

Conversation pattern:
- first turn for a word: show only the word or phrase and ask for the `0-3` rating
- second turn: provide the explanation matched to the rating, then the nuance, then the examples; include the short Chinese support for meaning and nuance when applicable
- for rating `3`, do not expand into a longer teaching loop; one example is enough before saving
- for rating `0`, `1`, or `2`, once that teaching response has been shown, the user may either respond in natural language or type `1` as a shortcut meaning "save this word and go to the next one"
- if the user still feels unsure, give one more simpler explanation or one more example, not a full essay
- once the user feels good, mark the word complete and move to the next entry

Input handling:
- at the start of a word, interpret `0`, `1`, `2`, `3` as the confidence rating only
- after the explanation-plus-examples response has been shown for a `0`, `1`, or `2` word, interpret a standalone `1` as "save and continue to the next word"
- before that teaching response is shown, do not treat `1` as "save and continue"; it only means the confidence rating if the agent is still at the initial rating prompt

Persistence rules:
- update the same `day_XX.md` file after each completed word, not only at the end
- preserve existing sections and source order
- do not remove the original short meaning/example from the generated sheet
- append guided-study notes under the current entry
- if guided-study notes already exist for that entry, update them instead of duplicating them

Use this structure under each studied entry:

```md
1. **word / phrase (3)**: short meaning
  - Explanation: the clearest explanation that helped the user.
  - 中文意思: short Chinese support for the meaning.
  - 中文语感: short Chinese support for nuance or usage when helpful.
  - Examples:
    - original sheet example.
  - Usage note: only when needed.
```

Persistence content:
- save the final helpful explanation, not every intermediate draft
- save only the best `1-3` example sentences that actually helped
- keep wording concise and practical
- when the user changes their confidence during study, persist the latest rating directly after the word, for example `blurry (3)`
- keep all example sentences under the same `Examples` field; do not split them into `Example` and `Extra examples`
- save the Chinese support lines by default when guided-study responses included them
- save `中文意思` for every completed word studied with Chinese support
- save `中文语感` when a nuance, contrast, or usage distinction was taught and it adds real learning value
- keep the Chinese lines short and practical; do not turn the sheet into a long bilingual dictionary
- when both explanation/nuance and examples are present, persist the explanation and Chinese support lines before the `Examples` section
- number the word entries within each section as `1. 2. 3. ...`; do not number the internal notes under each word

Teaching style:
- use simple English first
- add short Chinese support only for meaning and nuance, not for every sentence
- present explanation and nuance before examples in both the live response and saved notes
- keep examples practical and CELPIP-friendly
- prefer everyday contexts: work, school, housing, services, travel, opinions, scheduling, and problem solving
- avoid dictionary-heavy or overly academic explanations
- do not overwhelm the user with long blocks of text for a single word
- if the word is a phrasal verb, explain the base idea and then the natural use in a sentence

Word roots and derivatives:
- when useful, include a short `Word root` or `Word family` note as part of the teaching response and save it into the markdown
- prefer roots or derived forms that add real learning value, such as a core noun/adjective/verb relationship or a related form with distinct real-world usage
- good examples: `rigor -> rigorous`, `rehearse -> rehearsal`
- do not include transparent adverbs such as `ridiculous -> ridiculously`
- do not include inflectional variations such as `ridicule / ridiculed / ridiculing`
- do not include transparent informational derivatives whose meaning is fully predictable and adds no new usage value
- only include a root or derivative when it helps the user remember the word, understand nuance, or recognize a common related form

Session continuity:
- if the user stops mid-day and later resumes, continue from the next unfinished word when possible
- treat an entry as finished when it already has a saved rating in the word line plus explanation and examples, unless the user asks to revisit it
- if the user asks to revisit a finished word, update the existing notes rather than duplicating them

## Boundaries

This skill does not handle:
- review scheduling
- spaced repetition systems
- advanced automatic progress tracking beyond the saved learning notes unless the user requests it separately
- full CELPIP scoring or feedback outside vocabulary-focused tasks

Keep the workflow simple: generate a day sheet, save it, quiz from it, or guide the user through it one word at a time and persist the useful learning notes.
