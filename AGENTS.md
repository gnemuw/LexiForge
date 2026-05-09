# LexiForge Agent Guide

LexiForge is an AI-assisted Obsidian vault for exam vocabulary learning. Agents
generate readable Markdown articles from vocabulary libraries, mark target words
inside the article, and update the library so future sessions can cover new
words and review older words.

This file is the project-level contract. Follow it when working in this vault,
regardless of which coding or writing agent is being used.

## Directory Contract

- `Libraries/`: vocabulary libraries. Each Markdown file is one library.
  Example: `Libraries/PET.md`.
- `Articals/`: generated learning articles. Keep this spelling for now because
  the current vault already uses it. Example: `Articals/PET/`.
- `AGENTS.md`: this operating guide.

Do not create a parallel `Articles/` directory unless the user explicitly asks
for a migration. If a migration happens later, update this guide and move all
existing generated articles in one controlled change.

## Vocabulary Table Schema

Libraries are Markdown tables with these columns:

| Column | Meaning |
| :--- | :--- |
| `单词` | Vocabulary entry, including part of speech or phrase notes when present. |
| `状态` | Learning state. Use the allowed values below. |
| `上次日期` | Last date this word appeared in a generated article, in `YYYY-MM-DD` format. |
| `出现次数` | Number of generated articles that have included this word. |
| `出现文章` | Obsidian links to generated articles that included this word. |

Allowed `状态` values:

- Empty: never selected before. Treat as `new`.
- `seen`: selected in at least one generated article, not confirmed mastered.
- `review`: selected again for spaced repetition.
- `mastered`: user has checked or explicitly confirmed mastery.
- `ignored`: do not select unless the user explicitly asks.

When updating a row:

- Preserve the original `单词` text exactly.
- Use `YYYY-MM-DD` for dates.
- Increment `出现次数` by 1. If it is empty, set it to `1`.
- Append the generated article link to `出现文章`.
- Do not reorder the table.
- Do not rewrite unrelated rows.

## Word Selection

When the user asks for a new learning article, determine:

- library name, defaulting to the only obvious library if there is one;
- target word count, defaulting to `10` if the user does not specify;
- article type, defaulting to a story if the user does not specify;
- language and difficulty, matching the user's request when provided.

Selection priority:

1. Empty-state words that have never appeared before.
2. Review-due words whose `上次日期` is old enough for spaced repetition.
3. Previously seen words with low `出现次数`.
4. `mastered` words only when the user asks for review of mastered words.
5. Never select `ignored` words unless explicitly requested.

Use this simple spaced repetition schedule until the project has a script:

| Previous appearances | Due after |
| :--- | :--- |
| 1 | 1 day |
| 2 | 2 days |
| 3 | 4 days |
| 4 | 7 days |
| 5 | 15 days |
| 6+ | 30 days |

If there are enough never-seen words, include mostly new words but allow a small
review mix when due words exist. A practical default is 70-80% new words and
20-30% due review words.

## Article Format

Generated articles must be Markdown files under:

```text
Articals/<LibraryName>/
```

Use a stable filename:

```text
YYYY-MM-DD-<short-topic-or-type>.md
```

If the filename already exists, append `-2`, `-3`, and so on.

Each article must start with YAML frontmatter:

```yaml
---
library: PET
created: 2026-05-10
article_type: story
target_count: 10
target_words:
  - ability (n)
  - abroad (adv)
---
```

Required article sections:

1. Title.
2. Main article body.
3. `## Vocabulary Checklist`.

In the main body:

- Every selected target word must appear at least once.
- The first meaningful occurrence of each target word must be bolded with
  Markdown `**...**`.
- For phrases, bold the full phrase when natural.
- For inflected forms, prefer the base entry when natural. If an inflected form
  is needed, keep it clearly connected to the selected word.
- Do not bold non-target words just because they are in the same library.

Checklist format:

```markdown
## Vocabulary Checklist

- [ ] ability (n)
- [ ] abroad (adv)
```

The checklist is for the learner's mastery confirmation. Generating an article
does not automatically mean the word is mastered.

## Library Update After Article Generation

After writing the article, update the source library rows for the selected
words only.

For each selected word:

- `状态`: set to `seen` if it was empty; keep `mastered` only if the user has
  already confirmed mastery; use `review` when the word was selected because it
  was due for spaced repetition.
- `上次日期`: set to the article creation date.
- `出现次数`: increment by 1.
- `出现文章`: append an Obsidian link such as
  `[[Articals/PET/2026-05-10-market-story|2026-05-10 market story]]`.

If a user later checks items in an article checklist and asks the agent to sync
progress, update those checked words to `mastered`. Leave unchecked words as
`seen` or `review`.

## Validation Checklist

Before finishing any article-generation task, verify:

- The article file exists in the expected `Articals/<LibraryName>/` directory.
- YAML frontmatter includes the library, creation date, article type, count, and
  selected words.
- Every selected word is present in the article body.
- Every selected word has at least one bolded occurrence.
- The checklist contains exactly the selected words.
- The library table still has the same header and row order.
- Only selected word rows were changed.
- Dates use `YYYY-MM-DD`.
- `出现次数` values are numeric.

If validation fails, fix the files before responding.

## Future Hardening

The current Markdown table is human-friendly but fragile for repeated automated
updates. When the workflow becomes frequent, add a small script that:

- parses the library table;
- selects target words according to the schedule above;
- writes or validates article metadata;
- updates only selected rows;
- reports changed rows before saving.

Until that script exists, make small, careful edits and avoid broad formatting
rewrites of large library files.
