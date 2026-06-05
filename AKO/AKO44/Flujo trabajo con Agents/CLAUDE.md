# CLAUDE.md

@AGENTS.md

## Claude Code notes

Claude Code loads this file as project memory. The line `@AGENTS.md` imports the shared agent instructions from the repository root so Claude and Codex can follow the same rules.

When working on a documented task:

1. `AGENTS.md` is auto-loaded via the `@AGENTS.md` import above — no need to re-read it manually.
2. Read the task document under `docs/ai/`.
3. Work only on the requested phase.
4. Update the task document after the phase.
5. Do not move to the next phase without explicit confirmation.

If Claude appears not to follow these instructions, run `/memory` and verify that this `CLAUDE.md` file is loaded.
