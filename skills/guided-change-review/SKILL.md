---
name: guided-change-review
description: Interactively explain a branch, pull request, commit, or working-tree diff one major file at a time. Order files and changed code by data and command flow from the clearest pedagogical entry point, pause after every file for questions and comments, and continue only when the user explicitly asks to proceed. Use when the user asks to understand, walk through, review together, or explain a substantial set of code changes.
---

# Guided Change Review

Help the user understand an existing change set through a paced, conversational
walkthrough. This is an explanation workflow, not a defect-finding code review.

The defining behavior is:

1. Analyze the whole change set before presenting it.
2. Derive a teaching order from runtime, data, or command flow.
3. Present exactly one major file at a time.
4. Pause after every file.
5. Answer questions without silently advancing.
6. Continue only when the user explicitly says to continue.

Invoke as:

```text
/guided-change-review
/guided-change-review --base main
/guided-change-review --pr 1234
/guided-change-review --commit <sha>
/guided-change-review --working-tree
```

## Non-negotiable interaction contract

- Remain read-only unless the user separately and explicitly asks for an edit.
- Never dump explanations for multiple files into one response.
- Never advance because the current file "seems finished."
- Advance only on an unambiguous instruction such as `next`, `continue`, or
  `skip this file`.
- Questions, corrections, and comments keep the walkthrough on the current
  file unless the user also explicitly asks to advance.
- Do not delegate the conversation or pacing to a subagent.
- Do not overwhelm the user with every changed line. Explain the important
  behavior and use unchanged context only when it is needed to understand it.
- Distinguish observed facts from inferred intent. Label uncertainty plainly.
- Respect repository instructions and content-exclusion policies.

## Phase 1: Establish the change set

Infer the comparison target from the request:

1. An explicit PR, commit, branch, or base always wins.
2. `--working-tree` means staged, unstaged, and relevant untracked files.
3. In a feature branch, default to its merge-base with the repository's default
   branch.
4. If no meaningful comparison can be inferred, ask one focused question using
   the host's structured question UI when available.

Collect, without modifying the worktree:

- repository and branch identity;
- merge-base or requested comparison range;
- changed file names and statuses;
- diff statistics;
- complete diff hunks;
- renamed, generated, binary, lock, vendored, and deleted files;
- repository-local guidance that applies to changed paths.

Do not begin teaching from `git diff` order. That order is incidental.

## Phase 2: Build a change map

Analyze enough unchanged context to understand how the changed code is reached
and what it affects. Prefer semantic code intelligence when available, then
language-aware search, then text search.

For each changed file, identify:

- changed symbols and their callers;
- imports, exports, registrations, routes, handlers, commands, or event edges;
- inputs, outputs, state mutations, persistence, and external calls;
- configuration or schema dependencies;
- tests that demonstrate the changed behavior;
- whether the file is an entry point, orchestration layer, domain implementation,
  boundary adapter, test, configuration, generated artifact, or incidental
  support file.

Construct a compact flow graph. Typical shapes include:

- request -> route -> handler -> domain logic -> persistence -> response;
- command -> argument parsing -> orchestration -> operation -> output;
- event -> subscriber -> state transition -> emitted effect;
- UI action -> state update -> API call -> backend -> rendered result;
- public API -> implementation -> dependency boundary -> returned contract;
- configuration -> registration -> runtime selection -> behavior.

When several entry points are possible, choose the one that best explains why
the change exists, not necessarily the process's literal startup file.

## Phase 3: Choose the pedagogical order

Classify files:

- **Major**: necessary to understand the behavior or design of the change.
- **Supporting**: tests, configuration, fixtures, adapters, or documentation
  that confirm or enable the major flow.
- **Mechanical**: generated output, lockfiles, formatting-only changes, or
  repetitive migrations.

Order major files by causal comprehension:

1. the clearest externally visible or conceptual entry point;
2. orchestration and dispatch;
3. core behavior and state transitions;
4. boundaries, integrations, and persistence;
5. tests that reveal contracts or edge cases;
6. configuration and supporting changes.

Break this order when another sequence is more teachable. For example, a small
domain type may need to precede its caller if every later concept depends on it.
Explain such deviations briefly.

Within each file, independently derive an explanation order. Follow control or
data flow across changed symbols rather than automatically proceeding top to
bottom. Constructors, helpers, and types may be introduced before their textual
position when that makes the runtime path clearer.

## Phase 4: Start the walkthrough

Before the first file, give only:

- a two- or three-sentence thesis of the change;
- a compact end-to-end flow;
- the numbered major-file itinerary with one short reason per file;
- a short note identifying supporting or mechanical files that will not receive
  full treatment unless requested.

Then present the first file in the same response. Do not require a redundant
confirmation when the user's request to begin is already clear.

If a file or diff viewer/editor canvas is available, open or focus the current
file there. The chat explanation must still stand on its own.

## Phase 5: Present one file

Use this structure, adapting detail to the file:

```text
File 2 of 7: path/to/file
Why this file comes now

Role in the change

Walkthrough
1. Symbol or changed region (current line/range)
   - What reaches it.
   - What changed from the base version.
   - How data/control moves through it.
   - Important decisions, side effects, and failure behavior.
2. Next symbol or region in execution order
   ...

How this connects
What this file receives from the preceding flow and passes to later files.

Points worth retaining
A small number of design choices, assumptions, or subtleties.
```

For each important changed region:

- anchor the explanation to a symbol and current line or range;
- describe before-versus-after semantics, not merely syntax;
- identify inputs, transformations, outputs, and ownership;
- explain non-obvious error, cancellation, concurrency, security, or lifecycle
  behavior when relevant;
- connect the region to the overall change thesis;
- mention tests that establish its contract, without teaching a future test file
  prematurely.

Avoid:

- paraphrasing every line;
- repeating the raw diff;
- presenting speculative author motivation as fact;
- turning the walkthrough into a list of review findings;
- fully explaining future files before their turn.

End after one file with a clear pause, for example:

```text
Paused on `path/to/file`. I will stay with this file for questions or comments;
say `next` when you want to continue.
```

This is a status statement, not a request that requires a forced choice. Let the
user respond naturally.

## Phase 6: Handle discussion

While paused:

- Answer questions at the depth requested.
- Inspect additional unchanged context when needed for a correct answer.
- If the answer depends on a future file, give the minimum bridge needed and
  record the full topic for that file.
- Incorporate user corrections into the walkthrough model.
- Record design concerns, unresolved questions, and conclusions the user wants
  to retain.
- Stay on the current file after answering.

Interpret navigation commands as follows:

- `next` or `continue`: mark the current file discussed and present exactly the
  next file.
- `back`: return to the previously discussed file.
- `skip`: mark the current file skipped and present exactly the next file.
- `jump <path or number>`: move to that file and note that the causal order was
  intentionally interrupted.
- `overview`: restate the itinerary and progress without presenting a new file.
- `stop`: preserve a compact checkpoint and end the walkthrough.

If the user asks a question and says `next` in the same message, answer the
question first and then present exactly one next file.

## Phase 7: Maintain resumable state

Maintain a compact ledger throughout the conversation:

- comparison target and merge-base;
- change thesis;
- ordered major files and reasons;
- current file and current index;
- discussed, skipped, and remaining files;
- supporting/mechanical files;
- deferred cross-file questions;
- user corrections and conclusions;
- unresolved uncertainties.

Use session-scoped scratch storage when the host provides it. Never add state
files to the repository. If persistent scratch storage is unavailable, keep the
ledger concise enough to survive conversation summarization and restate it when
the user requests `overview` or `stop`.

Before resuming after a long interruption, verify that the comparison target
and relevant file versions have not changed. If they changed, explain the drift
and rebuild the affected part of the itinerary.

## Phase 8: Finish

After the final major file, provide:

- the complete command/data flow in one compact narrative;
- the principal design choices and changed contracts;
- unresolved questions or risks raised during discussion;
- the user's recorded conclusions;
- the list of supporting/mechanical files not discussed in depth.

Do not manufacture a defect verdict or approval. If the user wants correctness
review, security review, edits, or tests, treat that as a separate follow-up
task with the appropriate workflow.
