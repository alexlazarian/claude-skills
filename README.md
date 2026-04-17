# claude-skills

Custom skills for [Claude Code](https://docs.anthropic.com/en/docs/claude-code).

## Skills

### code-review

Entropy-reducing code review. Runs 9 independent review layers as isolated subagents — architecture, data flow, testability, security, correctness, test coverage, performance, observability, and hygiene. Each layer gets a fresh context and reviews through only its assigned lens. Favors deletion, consolidation, and simplification over additive fixes.

### review-synthesis

Blind validation pipeline for multiple code review outputs. Takes N review files, strips reviewer identity, verifies every finding against actual code, filters hallucinations, and produces a single prioritized report with signal-to-noise stats. Three-stage pipeline: pre-processor → blind synthesizer → reporter.

### improve-prompt

Transforms rough user prompts into clear, behavior-focused ones. Reads codebase context to understand the problem, keeps output high-level (no file names or implementation details), and copies to clipboard. Two modes: `--what` (descriptive, default) and `--how` (soft hypothesis about the problem area).

### writing-for-interfaces

Voice-first framework for writing, reviewing, and rewriting interface copy — buttons, errors, empty states, onboarding, CLI output, notifications. Establishes voice before applying principles (Purpose, Anticipation, Context, Empathy), then craft rules. Encodes a precedence chain: clarity > voice > craft rules. Patterns reference covers alerts, destructive actions, accessibility labels, and more.

## Usage: review-synthesis

The synthesis skill expects review outputs as `.md` files in a shared directory. After any review agent finishes (code-review, PR review, etc.), save its output with this prompt:

```
write the full review output as markdown to /tmp/reviews/review-{N}.md using the Write tool.
Create the /tmp/reviews/ directory first if it doesn't exist (use Bash: mkdir -p /tmp/reviews)
```

Replace `{N}` with 1, 2, 3... for each reviewer. Then synthesize:

```
synthesize the reviews in /tmp/reviews
```

To start fresh between runs:

```
rm -rf /tmp/reviews
```

## Setup

These skills are designed to be symlinked into `~/.claude/skills/`:

```bash
ln -s /path/to/claude-skills/code-review ~/.claude/skills/code-review
ln -s /path/to/claude-skills/review-synthesis ~/.claude/skills/review-synthesis
```

## Credits

- [vhpoet/.agents](https://github.com/vhpoet/.agents) — the code-review skill is based on Vahe Hovhannisyan's code-review skill
- [andrewgleave/skills](https://github.com/andrewgleave/skills) — writing-for-interfaces is a fork of Andrew Gleave's skill of the same name
- [superpowers](https://github.com/obra/superpowers) — foundational skill framework. Several skills here reference superpowers skills like `receiving-code-review` and `dispatching-parallel-agents`.

## Adding a new skill

1. Create a directory with a `SKILL.md` file
2. Symlink it into `~/.claude/skills/`
3. Claude picks it up automatically
