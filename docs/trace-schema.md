# Execution Trace Schema Evolution Policy

This document defines how execution trace JSON files evolve over time.
It exists so the trace exporter, the compare/replay tooling, and the
documentation all follow the same compatibility rules.

## Current schema

The current trace format is the `ExecutionTrace` structure in
[`src/compare/trace.rs`](../src/compare/trace.rs). It is serialized as
plain JSON and currently relies on serde defaults for optional fields.

Current top-level fields include:

- `label`
- `contract`
- `function`
- `args`
- `storage`
- `budget`
- `return_value`
- `call_sequence`
- `events`

## Compatibility rules

- Prefer additive changes.
- New fields should be optional and use `#[serde(default)]` so older
  traces continue to load.
- Existing fields should keep the same meaning and type whenever
  possible.
- If a field must change shape, add a new field instead of reusing the
  old one unless the old data can be losslessly normalized.
- Preserve deterministic ordering for maps and collections used in
  comparison output.

## Versioning policy

- The trace JSON is intentionally treated as a stable, evolving schema.
- The repository does not currently embed a schema version in exported
  trace files.
- If a future change requires a breaking schema update, introduce an
  explicit version field and support both the previous and new formats
  during the transition.
- Any versioned change must update:
  - the exporter
  - the loader used by `compare` and `replay`
  - example traces in the documentation
  - regression tests that read or write traces

## Migration policy

- Breaking changes should ship with a migration path or conversion
  guidance.
- Keep the previous format readable for at least one release when a
  version bump is introduced.
- Prefer one-way upgrades in the exporter and backward-compatible
  parsing in the loader over ad-hoc manual conversions.
- If a migration is unavoidable, document:
  - the old and new schema shapes
  - how to regenerate traces
  - whether old traces are still accepted

## Contributor checklist

Before merging a trace schema change:

1. Update `src/compare/trace.rs`.
2. Update `docs/doc/compare.md`.
3. Update the README trace example if the user-facing shape changed.
4. Add or update tests for loading old and new trace files.
5. Confirm the compare/replay commands still accept existing traces.
