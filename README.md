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

## Repo layout

```text
review-buddy/
├── SKILL.md
├── README.md
├── LICENSE
├── .gitignore
├── references/
│   └── review-templates.md
└── examples/
    └── sample-pr/
        ├── README.md
        ├── pr-description.md
        ├── review-brief.md
        ├── context-cards.md
        ├── chunk-plan.md
        ├── guided-review.md
        └── final-summary.md
```

## Installation / use in Hermes

### Option 1: Use as a local linked skill

This repo is already suitable for a linked/local-development workflow where Hermes reads the `SKILL.md` from this project directory via your local skill path.

### Option 2: Copy into Hermes skills manually

Copy the project into a Hermes skill directory such as:

```text
~/.hermes/skills/software-development/review-buddy/
```

and make sure `SKILL.md` sits at the root of that folder.

### Option 3: Install from a raw `SKILL.md`

If you host the raw `SKILL.md` somewhere accessible, Hermes can install skills from a direct URL.

## What makes this skill different

Review Buddy is not primarily a GitHub API skill and not a one-shot code-review summarizer.

It is designed to:

- help a reviewer build context before judging code
- split large changes into meaningful chunks
- apply the right review lenses to each chunk
- keep the human reviewer in control of the verdict
- work especially well for AI-generated or unfamiliar changes

## Examples

The `examples/sample-pr/` directory demonstrates the intended rhythm of the skill:

- orient the reviewer first
- pull surrounding context second
- decompose the change third
- review one chunk at a time
- summarize only after the chunked review is complete

## Contributing / editing

If you edit this repo:

- keep `SKILL.md` as the source of truth
- keep examples aligned with the skill's recommended review rhythm
- prefer additive reference material under `references/` and worked examples under `examples/`
- preserve valid frontmatter at the top of `SKILL.md`

## License

MIT
