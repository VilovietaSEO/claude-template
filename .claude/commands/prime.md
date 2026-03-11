---
name: prime-phoenix
description: Load full context for a Thorbit Phoenix work session — reads manifest, system architecture, schema, and routes to get immediately up to speed.
allowed-tools: Read
---

You are starting a Thorbit Phoenix work session. Read the following files in order and internalize everything before responding. Do not summarize them back to the user — just confirm you are ready and state the current build phase and what is next based on the MANIFEST.

**Step 1 — Project state (what's built, what's next):**
Read: /Users/vilovieta/Desktop/production-factory/planner-workflows/thorbit-phoenix/MANIFEST.md

**Step 2 — System architecture (stack, rules, directory structure):**
Read: /Users/vilovieta/Downloads/thorbit-phoenix/CLAUDE.md
Read: /Users/vilovieta/Downloads/thorbit-phoenix/docs/DESIGN-SYSTEM.md

**Step 3 — Data model:**
Read: /Users/vilovieta/Downloads/thorbit-phoenix/docs/DATABASE.md

**Step 4 — Routes and topology:**
Read: /Users/vilovieta/Downloads/thorbit-phoenix/docs/ROUTES.md
Read: /Users/vilovieta/Downloads/thorbit-phoenix/docs/TOPOLOGY.md

**Step 5 — Runtime constraints (Vercel timeouts, Neon rules, API limits, state machines):**
Read: /Users/vilovieta/Downloads/thorbit-phoenix/docs/CONSTRAINTS.md

**Step 6 - How to work and Develop**
Read: /Users/vilovieta/Desktop/production-factory/planner-workflows/thorbit-phoenix/how-to-plan-features.md
Every plan requires a folder with the nouns file, verbs file, state file, integration file, collab and permission model. This means that every time we begin working after priming, we investigate and first document nouns. Then we move to document Verbs, etc.

We always get primed, then plan, then execute.

## Important Note on Codebase

`thorbit-portal` is the example of proper code architectural design and best practices. It is not the codebase containing our actual designs or tools, though we can be inspired by tools in it.

`thorbit-karl` is the old codebase that actually contains the information on what was built.

`thorbit-phoenix` is the resurrected version of KARL and it will use the better design, better code, and improved structure.

We want to reuse our functions and use the same naming conventions. We don't want to bloat our code. Remember this.


**Step 7 — Testing architecture (how to run and add tests):**
Read: /Users/vilovieta/Downloads/thorbit-phoenix/docs/TESTING.md

**CRITICAL TESTING RULE — read before writing any test:**
- Unit / Integration / Live tests → run locally, no server required (or server in background)
- **Mode B workflows (claude CLI subprocess / Vercel Sandbox)** → MUST be tested via `tests/sandbox/` using a real Vercel Sandbox. Do NOT attempt to spawn the `claude` CLI locally for these — use `launchWorkflow()` + poll DB. `PHOENIX_API_URL` must be the deployed Vercel URL (`https://preview-skybreaker-overland.vercel.app`). Do NOT use ngrok or localhost — the sandbox cannot reach localhost. Guard with `SANDBOX=1`.
- Before writing a new sandbox test: run `scripts/test-sandbox.mjs` first (6-step sanity check)
- Sandbox tests take 30min–2h. Run manually, never in CI.
- See `tests/helpers/sandbox-harness.ts` for `pollJobUntilTerminal()`, `getWorkflowEvents()`, `getWorkflowSpans()`

**Only Read If Working on EICS architecture (Tier 2 service — read if working on EICS):**
Read: /Users/vilovieta/Downloads/thorbit-phoenix/docs/EICS-ARCHITECTURE.md

After reading all files, respond with:
1. Current build phase (based on MANIFEST.md status tracker)
2. The next 3 concrete things to build
3. Any open questions or blockers you notice
4. Confirm you are ready to work

Keep the response short — this is a status brief, not a recap.

