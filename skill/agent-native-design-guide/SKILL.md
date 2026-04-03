---
name: agent-native-design-guide
description: >
  This skill should be used when the user is designing software, tools, CLIs, or
  Skills meant to be operated by AI agents rather than humans. It provides a
  decision framework and design principles for Agent-Native software architecture.
  Trigger on 'design for agents', 'agent-native', 'agent-friendly CLI',
  'make this tool agent-compatible', 'CLI for agents', 'Skill architecture',
  'agent-first design', 'how should agents use this tool',
  'convert to agent-native', 'tool design principles'.
---

# Agent-Native Design Guide

Agent-Native software design guide -- providing design principles and architecture patterns for building software whose primary users are AI agents, not humans.

When the user is designing, building, or refactoring a tool/CLI/Skill for agent consumption, use this guide to inform design decisions. The core thesis: agents perceive through text, act through tools, and compose atomic capabilities into emergent workflows.

## When to Use This Guide

Apply this guide when:
- Designing a new CLI tool that agents will invoke
- Refactoring an existing tool to be agent-friendly
- Choosing between CLI+Skill, MCP, or other protocols
- Deciding architecture for a tool with both agent and human users
- Writing SKILL.md for a new capability

Do NOT apply this guide when:
- Building pure human-facing UI with no agent interaction
- Working on internal business logic that agents don't touch
- The tool is a one-off script not intended for reuse

## Decision Framework

### 1. Does this tool need Agent-Native design?

```
Is the tool invoked by an AI agent?
  No  --> Standard CLI/API design is sufficient
  Yes --> Continue

Is the agent the PRIMARY user?
  Yes --> Full Agent-Native design (all principles apply)
  No  --> Dual-mode design (add --json + --help, keep human defaults)
```

### 2. Which protocol?

Default to CLI+Skill. Escalate only when needed:

```
Start with CLI + --json output + input validation     (P0, covers all platforms)
  |
Add SKILL.md + --help --json                          (P1, discoverability)
  |
Add --dry-run + --fields                              (P2, safety + efficiency)
  |
Complex nested params? Shell escaping painful?
  Yes --> Add MCP Server surface on top of CLI
  No  --> CLI+Skill is sufficient
```

Core rule: one implementation, two surfaces. CLI and MCP share the same schema and business logic. MCP complements CLI, never replaces it.

### 3. Which architecture?

Match architecture complexity to the tool's actual needs:

| Complexity | Architecture | Control Layer | Data Layer |
|------------|-------------|---------------|------------|
| Simple | Single CLI | CLI + `--json` | Filesystem |
| Medium | CLI + Service | CLI + SKILL.md (+ optional MCP) | Filesystem + lightweight DB |
| Complex | Platform | CLI/MCP + backend agent | Backend API + DB |

The three-layer separation pattern applies at every level: **Control Layer** (agent entry -- structured interface), **Presentation Layer** (human entry -- visual rendering), **Data Layer** (shared kernel -- single source of truth). Both entries share one data layer. Read `references/architecture-patterns.md` for detailed architecture guidance.

## Ten Design Principles -- Quick Reference

### Core Principles (the "why")

| # | Principle | One-line Definition |
|---|-----------|-------------------|
| C1 | Text is the Interface | Agents perceive and act through structured text. JSON, Markdown, CLI output are their native media |
| C2 | Atomic & Composable | Expose minimal independent units. Agents compose them into emergent capabilities |
| C3 | Parity | Everything a human can do via UI, an agent must be able to do via tools |
| C4 | Security Boundaries First | Declare permissions, enforce boundaries. Freedom within boundaries beats per-action confirmation |

### Practice Principles (the "how")

| # | Principle | One-line Definition |
|---|-----------|-------------------|
| P1 | Discoverability | Agents discover capabilities via `--help`, schema, SKILL.md -- not by guessing |
| P2 | Determinism | Same input, same output. Stable formats, predictable state changes |
| P3 | Output is Product | Output format design matters more than feature design. Cut verbosity before adding features |
| P4 | Gotchas-Driven Iteration | A Skill's real value is in accumulated edge cases, not Day 1 documentation |
| P5 | Progressive Disclosure | Use filesystem as context engineering -- SKILL.md as concise hub, details in references/ |
| P6 | Constrain Intent, Not Steps | Define goals and boundaries, let the agent choose the method |

Each principle includes boundary conditions -- situations where it does not fully apply. Read `references/design-principles.md` for full definitions, rationale, examples, and boundary conditions.

## Key Anti-Patterns

Avoid these three most common Agent-Native design mistakes:

**1. Workflow-as-Tool** (violates C2)
Packaging multi-step decision logic into a single tool (e.g., `classify_and_organize_files()`). The agent cannot intervene at intermediate steps, losing its reasoning advantage.
- Fix: expose atomic operations (`read_file`, `move_file`, `tag_file`), let the agent compose them.

**2. Build-First-Expose-Later** (violates C3)
Developing a complete GUI product, then retroactively "exposing APIs for agents." This typically results in partial capability access.
- Fix: design the tool layer and UI layer in parallel from the start, sharing the same data layer.

**3. Ignoring Output Design** (violates P3)
Correct functionality but verbose, unstable output formats. The agent spends more tokens parsing irrelevant information than doing useful work. Empirical data: structured output optimization reduces agent parsing time by 5-9x.
- Fix: implement `--json` with a standard envelope `{success, data, metadata, error}`. Support `--fields` for selective output.

## CLI Design Quick Checklist

When implementing or reviewing a CLI tool for agent use, verify:

- [ ] `--json` flag for structured output (standard envelope: `{success, data, metadata, error}`)
- [ ] `--help` includes parameter types, defaults, constraints, and examples
- [ ] Command naming follows `<tool> <resource> <action>` pattern
- [ ] Input validation at CLI entry point (path normalization, schema validation, reject dangerous characters)
- [ ] Error responses include error code (machine-readable) + message + recovery suggestion
- [ ] Exit codes follow semantic convention (0=success, 1=general error, 2=usage/argument error, 3=resource not found, 4=permission denied, 10=dry-run preview)
- [ ] `--dry-run` for destructive operations
- [ ] `--no-interactive` mode: suppress prompts, pagers, and confirmation dialogs (lesson: AWS CLI v2 defaulting to `less` pager broke thousands of CI pipelines in 2019)
- [ ] `--fields` for output field masking (context window protection)

Working code examples: `examples/cli-json-output.py` (JSON envelope) and `examples/cli-help-design.py` (agent-friendly `--help`).

## Security Essentials

Agent security has a fundamental tension: agents must read external data to work, but external data may contain malicious instructions (prompt injection). Three principles for tool developers:

1. **Validate all input as adversarial** -- path traversal checks, parameter schema validation, reject shell metacharacters. Use `subprocess.run([list])` never `os.popen(string)`.
2. **Declare minimal permissions** -- specify exactly what the tool needs (filesystem paths, network domains, executables). Not more, not less. Like Android permissions but finer-grained.
3. **Design for sandbox execution** -- do not assume full host access. Confine file operations to working directory, declare required network domains. If the tool cannot run in a sandbox, explain why in SKILL.md.

Permission levels for reference: L0 read-only (auto-allow) -> L1 scoped write (confirm once) -> L2 shell execution (whitelist or confirm) -> L3 network/credentials (always confirm) -> L4 irreversible operations (confirm + dry-run required).

## Reference Files

Consult these when deeper guidance is needed:

### References

- **`references/design-principles.md`** -- Full definitions of 10 principles with rationale, positive examples, boundary conditions, anti-pattern table, and applicability spectrum. Read when making design trade-off decisions or when a principle's boundary conditions matter.
- **`references/architecture-patterns.md`** -- Three-layer architecture definition, protocol selection matrix (CLI+Skill vs MCP vs A2A vs OpenAPI), complexity-graded architecture choices (simple/medium/complex), presentation layer evolution roadmap P0-P3. Read when choosing architecture for a new tool or evaluating protocol options.

### Examples

- **`examples/cli-json-output.py`** -- Runnable Python example demonstrating the standard JSON envelope, `--fields` masking, `--quiet` mode, and exit code conventions. Run with `uv run python examples/cli-json-output.py list --json`.
- **`examples/cli-help-design.py`** -- Runnable Python example demonstrating dual-mode `--help` (human text / agent JSON) and `schema` sub-command for machine-readable JSON Schema introspection. Run with `uv run python examples/cli-help-design.py --help`.

## Gotchas

<!-- This section grows over time as agents encounter edge cases. Each time an agent hits a problem while applying this guide, add a line here. -->

(No gotchas recorded yet -- append as they are discovered in practice.)
