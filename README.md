# Review Buddy

A Hermes skill for progressive, context-aware, human-in-the-loop code review.

## What it is

Traditional review tools throw reviewers into a giant diff and expect them to reconstruct intent, architecture, and risk all at once. Review Buddy turns review into a guided workflow:

1. frame the review target
2. gather the missing context
3. split the change into meaningful chunks
4. review chunk by chunk with the right lenses
5. consolidate findings into a clear verdict

This repo packages the skill definition, reusable templates, and concrete examples.

## Repo structure

- `SKILL.md` — the main Review Buddy skill
- `references/review-templates.md` — reusable output/comment templates
- `examples/sample-pr/` — a worked example showing how Review Buddy would review a realistic PR

## Intended use

Best for:

- PR review in unfamiliar codebases
- reviewing Claude Code / Codex / other AI-generated changes
- large or mixed diffs
- reviewers who need AI to bring in surrounding context instead of only summarizing the diff

## Example workflow

A good Review Buddy session typically looks like this:

1. **Review brief** — identify change type, intent, hotspots, and order
2. **Context bootstrap** — read surrounding code, contracts, and tests
3. **Chunk plan** — split the change into meaningful review units
4. **Guided review** — inspect one chunk at a time with the right lenses
5. **Final summary** — consolidate blocking issues, questions, and praise

See `examples/sample-pr/` for a realistic worked example.

## Installation / use in Hermes

If you want to install this as a local skill, point Hermes at the `SKILL.md` in this repo or copy the folder into your Hermes skills directory.

## License

MIT
