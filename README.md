# Review Buddy

A Hermes skill for progressive, context-aware, human-in-the-loop code review.

## What it is

Traditional review tools throw reviewers into a giant diff and expect them to reconstruct intent, architecture, and risk all at once. Review Buddy turns review into a guided workflow:

1. frame the review target
2. gather the missing context
3. split the change into meaningful chunks
4. review chunk by chunk with the right lenses
5. consolidate findings into a clear verdict

This repo currently contains the skill definition and supporting templates.

## Contents

- `SKILL.md` — the main Review Buddy skill
- `references/review-templates.md` — reusable output/comment templates

## Intended use

Best for:

- PR review in unfamiliar codebases
- reviewing Claude Code / Codex / other AI-generated changes
- large or mixed diffs
- reviewers who need AI to bring in surrounding context instead of only summarizing the diff

## License

MIT
