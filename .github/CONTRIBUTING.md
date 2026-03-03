# Contributing

Contributions that improve the orchestration protocol, add failure scenarios, or sharpen the worked examples are welcome.

## What to Contribute

**Good contributions:**
- Corrections to protocol steps that cause incorrect or ambiguous behavior in real sessions
- New failure scenarios based on observed Agent Teams failures (with root cause + resolution + anti-patterns)
- Improvements to the `references/example.md` that make the worked example clearer or more representative
- Clarifications to the Teammate Prompt Template or Synthesis Protocol

**Not in scope:**
- Changes that require Agent Teams features not yet available in Claude Code
- Additions that conflict with the minimum viable team principle or exception-driven communication model

## How to Contribute

1. **Open an issue first** for anything beyond a small typo fix. Describe the problem and your proposed solution before writing content.

2. **Fork and branch** from `main`.

3. **Make focused changes.** Each PR should address one concern. Mixing a bug fix with a new failure scenario makes review harder.

4. **Validate against real usage.** If you are changing orchestration instructions, test the updated skill in an actual Claude Code Agent Teams session. Document what you tested in the PR.

5. **Keep SKILL.md internally consistent.** Every section of the skill file is read by the model during execution. Contradictions between phases, the Teammate Prompt Template, and the Synthesis Protocol cause real failures. Cross-check your changes against all other sections.

6. **Update references if needed.** If your change affects observable behavior shown in `references/example.md`, update the example. If you are fixing a failure mode, add or update the corresponding entry in `references/failure-scenarios.md`.

## PR Checklist

- [ ] Change is focused on a single concern
- [ ] SKILL.md is internally consistent after the change
- [ ] Tested in a real Agent Teams session (or clearly marked as untested)
- [ ] References updated if behavior changed
- [ ] Issue linked in PR description

## Style Notes

- Use second person ("The lead calls...") for protocol instructions in SKILL.md
- Use present tense for facts, imperative for actions
- Anti-patterns sections use bold labels: `**DO NOT:**` followed by the problematic behavior in a blockquote
- Failure scenario entries follow the structure: Situation → Root Cause → Resolution (numbered steps) → Anti-patterns
