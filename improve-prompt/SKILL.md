---
name: improve-prompt
description: Use when user asks to improve, revise, sharpen, or copy a prompt to clipboard — especially vague feature requests or bug reports that lack technical specificity. Triggered by "improve this prompt", "revise this", "copy to clipboard".
---

# Improve Prompt

## Overview

Transform a rough user prompt into a clear, behavior-focused one. Read codebase context to understand the problem, but keep the output high-level — no file names, line numbers, or internal implementation details in the final prompt. Always copy to clipboard. Never implement.

## Flags

**`--how`** — Prescriptive mode. Includes a soft hypothesis about why the problem might be happening and hints at the general area to investigate — but still no file names, line numbers, or specific API names. Use when you want to nudge the implementer toward the right area.

**`--what`** (default) — Descriptive mode. Current behavior, expected behavior, and enough context to understand the gap. No solutions, no hypotheses. Let the model reason freely.

If no flag is given, default to `--what`.

## Modes

**Standard mode:** User provides a rough prompt. Improve it.

**Follow-up mode:** User provides a previous prompt + a summary of what was already implemented + what they want next. This happens when they cleared their session and need the new prompt to be self-contained for a fresh context.

## Process

1. **Check for follow-up context** — if the user provided a previous prompt and/or a summary of changes already made, this is follow-up mode. Go to step 2a. Otherwise skip to 2b.

   **2a. Follow-up mode:**
   - Read the current state of the relevant files to understand what was actually implemented
   - The output prompt must open with a **"Current state"** section (2-5 bullet points) describing what already exists in the code — enough for a fresh session to understand the baseline without reading the whole file
   - Then add the new request below it

   **2b. Standard mode:**
   - Read relevant code first — grep/read the files involved. Identify the exact views, models, functions, and APIs touched.

2. **Separate concerns** — if the prompt has 2+ independent issues, split into numbered sections with bold headings.

3. **Flag behavioral constraints** — note high-level facts the implementer must know upfront, but frame them as behaviors, not implementation details (e.g. "categories are static — adding new ones requires a structural change, not just data").

4. **Write the output based on the flag:**

   **`--how` output:** Describe current behavior, expected behavior, and add a soft hypothesis about why it might be happening — phrased as "I suspect this is because…" or "this might be related to…" to nudge without prescribing. Stay high-level. No file names, no line numbers, no API names.

   **`--what` output:** Describe each problem precisely — what is currently happening, what the expected behavior is, and enough context to understand the gap. No solutions, no hypotheses, no implementation details. The goal is a prompt that gives a model full understanding of the problem with zero bias toward a particular fix.

   **Both modes — hard rules for the output:**
   - No file paths or line numbers
   - No function/class/struct names from the codebase
   - No specific API or framework method names
   - Frame everything in terms of user-visible behavior
   - The codebase reading you did is for YOUR understanding — it should not leak into the output

5. **Copy to clipboard** — always end by running:
   ```bash
   cat > /tmp/prompt_out.txt << 'EOF'
   ...
   EOF
   pbcopy < /tmp/prompt_out.txt
   ```

6. **Prompt feedback** — after copying, print a short "Prompt feedback" section (NOT in the clipboard, just in the chat). Be honest and direct — tell the user what was weak about their original prompt and give one actionable tip they can apply next time. Examples of things to call out:
   - Vague descriptions ("it doesn't work" — what doesn't work?)
   - Missing current vs expected behavior
   - Mixing multiple issues without separating them
   - Not mentioning the conditions that trigger the bug (e.g. light mode only)
   - Burying the actual problem in filler words
   - Not specifying what "fix" means to them

   Keep it to 2-3 sentences max. Tone: honest friend, not lecturer.

7. **Stop** — do not make any code changes.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Rephrasing without reading code | Always read relevant files first — but to inform yourself, not to quote in output |
| Implementing after copying | Stop after `pbcopy`. Say "Copied." and wait for next instruction |
| Bundling multiple issues in one paragraph | Use numbered sections with bold headings |
| Leaking file names, line numbers, or API names into the output | The output is behavioral only. Code context is for your understanding, not the prompt |
| Using `--what` but sneaking in solution hints or hypotheses | `--what` = describe the problem only. Zero solutions, zero guesses |
| Using `--how` but being too specific about implementation | Soft hypothesis only — "I suspect…" not "the issue is in X.swift line 42" |
| Forgetting the clipboard step | Always run the `pbcopy` command — it's the whole point |
| Asking clarifying questions before copying | Read the code, infer context, write the prompt. Ask only if truly blocked |
| Follow-up with no current-state section | Fresh sessions have no memory. Always open with "Current state" in follow-up mode |
