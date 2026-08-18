---
name: conceptual-naming-review
description: Review recently written or changed code for ambiguous, overloaded, inconsistent, or conceptually misleading names. Build a concept map, propose precise repository-aware renames, and apply them only after user approval. Use toward the end of feature development or when the user asks for naming clarity, terminology consistency, a naming audit, or help distinguishing similar concepts.
---

# Conceptual Naming Review

Perform a semantic naming pass over a completed or nearly completed change. The
goal is not stylistic uniformity. The goal is that a reader can reliably infer
which domain concept each important name represents and distinguish it from
neighboring concepts.

Typical failures include:

- two similar names for different concepts, such as `reserved_bytes` and
  `reserve`;
- several names for the same concept;
- a name that omits a decisive dimension such as units, scope, direction,
  ownership, lifecycle, or state;
- a broad name for a narrow value, or a narrow name for a broad value;
- action, state, capacity, request, limit, allocation, and result concepts that
  share an overloaded root without clear qualifiers.

## Interaction contract

- Review first. Do not edit code until the user approves a concrete rename
  proposal.
- Treat naming as a domain-modeling problem, not a thesaurus or linting exercise.
- Prefer the smallest set of renames that removes real ambiguity.
- Do not rename merely to satisfy personal taste or generic style advice.
- Follow established repository and language conventions unless those
  conventions are the source of the ambiguity.
- Preserve public and persisted compatibility unless the user explicitly
  approves a breaking change or a migration strategy.
- Never claim that a name is ambiguous without explaining which concepts a
  reader could reasonably confuse.
- If the reviewed names are already clear, say so and make no proposal.

Invoke as:

```text
/conceptual-naming-review
/conceptual-naming-review --base main
/conceptual-naming-review --working-tree
/conceptual-naming-review path/to/subsystem
```

## Phase 1: Establish scope

Infer the review scope from the request:

1. Use an explicit path, pull request, commit, branch, or comparison base when
   provided.
2. `--working-tree` means staged, unstaged, and relevant untracked files.
3. On a feature branch, default to changes since the merge-base with the
   repository's default branch.
4. In an active development conversation, include code created or materially
   changed during the session.
5. If no meaningful scope can be inferred, ask one focused question using the
   host's structured question UI when available.

Collect the complete change set and enough unchanged context to understand it.
Include relevant types, callers, callees, tests, schemas, configuration,
documentation, and external boundaries. Exclude generated, vendored, and
mechanical files from direct review, but account for them when a rename would
require regeneration.

## Phase 2: Build a concept map

Before judging individual identifiers, reconstruct the vocabulary of the
changed behavior. Trace data and control flow rather than reading names in
isolation.

For each important concept, record the dimensions that actually distinguish it:

- domain meaning and invariant;
- role, such as request, limit, capacity, allocation, reservation, lease,
  measurement, state, event, or result;
- unit or representation;
- scope and owner;
- lifecycle or time horizon;
- direction or source and destination;
- singularity, plurality, and cardinality;
- mutability and state transitions;
- whether it is desired, attempted, accepted, current, cumulative, or remaining.

Create a compact internal concept map connecting identifiers to concepts and
code locations. Include names across variables, fields, parameters, functions,
types, enum members, commands, events, configuration keys, serialized fields,
database columns, tests, and documentation.

Do not assume two names are equivalent because they share a root. Do not assume
two concepts differ because the current code uses different words. Determine
their behavior and invariants.

## Phase 3: Diagnose naming risks

Look for high-signal problems:

### Different concepts that look alike

- near-synonyms, shared roots, or singular/plural variants used for distinct
  meanings;
- a bare noun next to a qualified form, where the bare form could mean either;
- values distinguished only by subtle tense or grammatical form;
- related values whose names omit their decisive dimension.

### Same concept named differently

- terminology drift across layers, files, tests, configuration, and docs;
- aliases that suggest a semantic distinction that does not exist;
- abbreviations or legacy terms used inconsistently.

### Names that misstate behavior

- names that imply ownership, units, mutability, ordering, or guarantees the
  code does not have;
- verbs that conceal side effects or nouns that actually perform actions;
- `is_`, `has_`, or `can_` booleans whose truth condition does not match the
  predicate;
- collection names that obscure element type or cardinality;
- names such as `data`, `value`, `item`, `info`, `manager`, `handler`, `process`,
  `temp`, or `result` where local context does not make the concept obvious.

### Missing relational clarity

- requested versus granted;
- configured versus effective;
- minimum versus maximum;
- total versus available versus remaining;
- current versus cumulative;
- source versus destination;
- local versus remote;
- logical versus physical;
- policy versus per-operation input;
- definition versus instance;
- identifier versus object.

Only report a concern when it creates a plausible misunderstanding, increases
the chance of incorrect use, or obscures an important domain distinction.

## Phase 4: Design candidate names

Derive names from the concept map and the repository's existing vocabulary.
Good candidate names should:

- identify the domain concept before implementation detail;
- encode only dimensions needed to distinguish nearby concepts;
- use the same word for the same concept throughout the relevant scope;
- use visibly different words or qualifiers for different concepts;
- include units when the type or surrounding API does not make them obvious;
- remain concise enough to read naturally at call sites;
- form coherent pairs or families, such as `requested_lease_bytes`,
  `granted_lease_bytes`, and `minimum_free_bytes`.

Evaluate candidates at declarations and representative call sites. A locally
descriptive name can still be poor if expressions become misleading or
needlessly repetitive.

Search for established domain terms before inventing new vocabulary. Prefer a
project's precise term over a generic industry term, unless the project uses the
existing term inconsistently.

## Phase 5: Present the review

Present a short concept summary followed by a proposal table:

| Current name | Actual concept | Ambiguity or mismatch | Proposed name | Compatibility |
|---|---|---|---|---|
| `reserve` | Bytes requested for one lease attempt | Confusable with the persistent reserve policy | `requested_lease_bytes` | Internal |
| `reserved_bytes` | Minimum bytes kept free by policy | Sounds like bytes already allocated | `minimum_free_bytes` | Serialized config key; migration needed |

For every proposal:

- cite representative declarations and usages;
- explain the competing interpretation, not just that the name "could be
  clearer";
- state whether the rename is internal, public API, serialized, persisted,
  user-facing, or cross-repository;
- mention related names that must change together;
- identify uncertain domain intent explicitly.

Group mutually dependent renames into one proposal. Separate required clarity
fixes from optional refinements. Omit low-value suggestions.

End by asking the user to approve, reject, or revise the proposals. Do not edit
in the same response.

## Phase 6: Apply approved renames

After approval:

1. Re-read affected files in case they changed during review.
2. Use language-aware rename or refactoring tools when available.
3. Update all code references, tests, fixtures, examples, comments, and
   documentation that use the concept.
4. Update serialized names, configuration keys, database columns, telemetry,
   command-line flags, or public APIs only when included in the approval.
5. When compatibility must be preserved, keep the external name at the boundary
   and use the clearer name internally, or implement the repository's normal
   deprecation or migration pattern.
6. Regenerate derived artifacts using existing repository tooling rather than
   editing generated files manually.
7. Search for stale uses of the old terminology and inspect each remaining
   occurrence.

Do not mix unrelated cleanup into the rename.

## Phase 7: Validate semantic consistency

Run the smallest existing tests, type checks, builds, or linters that cover the
affected code. Then inspect representative flows using the new vocabulary:

- declarations and call sites read consistently;
- related concepts remain visibly distinct;
- the new terms match runtime behavior and invariants;
- external compatibility or migration behavior is intact;
- no old name remains accidentally in code, tests, docs, or user-facing text.

If validation reveals that a proposed name was based on a mistaken concept map,
stop and explain the discrepancy rather than forcing the rename through.

## Phase 8: Finish

Summarize:

- the concepts clarified;
- the approved renames applied;
- any external names intentionally retained for compatibility;
- unresolved terminology questions or deferred breaking changes.

Do not pad the result with names reviewed but left unchanged unless their
retention is important to the user's decision.
