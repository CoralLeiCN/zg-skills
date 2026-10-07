# Optimize Prompt: specification

This document records the constraints and limitations for the purpose described
in [intent.md](intent.md). The operational instructions live in
[SKILL.md](../../plugins/optimize-prompt/skills/optimize-prompt/SKILL.md).

## Operating constraints

- Analyze the supplied prompt and context without executing its task unless the
  user explicitly requests both optimization and execution.
- Preserve objectives, domain facts, variables, placeholders, hard constraints,
  and examples that encode requirements. Do not invent facts, permissions, tools,
  sources, business rules, or success claims.
- Preserve message roles and instruction authority. Do not promote lower-priority
  content or flatten a role-separated bundle in a way that changes its meaning.
- Distinguish API configuration from prompt text. Preserve relevant action and
  approval boundaries; include an autonomy policy only when the task needs one.
- Use the distilled GPT-5.6 guidance without requiring network access. Consult
  current guidance when requested or when freshness materially affects the work.

## Audit and output requirements

- Recover the intended outcome, context, constraints, tools, authorized actions,
  and output contract before rewriting.
- Audit instruction and information conflicts, sequence and scope problems,
  incompatible outputs, tool and evidence gaps, priority ambiguity, duplication,
  and drift in accumulated context. Distinguish missing information from a
  contradiction.
- Resolve conflicts only through a supported priority or scope rule. Later
  placement alone does not establish precedence. Ask for the smallest material
  decision when no supported resolution exists.
- Unless another format is requested, return a copy-ready optimized prompt, a
  conflict and context audit, material assumptions or missing information, and
  at most four bullets explaining meaningful changes.
- Label conflicts `Resolved` or `Unresolved`; label context findings `Duplicate`,
  `Drift`, or `Justified repetition`. State `None found` where applicable.
- If an unresolved conflict prevents a definitive rewrite, provide a clearly
  provisional prompt only when useful, or ask the focused question before
  finalizing.

## Limitations

Internal consistency review does not establish external factual accuracy.
Externally unverified claims must remain labeled, and unavailable evidence must
not be presented as checked.

The audit covers the supplied prompt and available context. It cannot account
for undisclosed application instructions, tool behavior, or later conversation
changes. Improvements are not a guarantee of downstream model behavior or a
substitute for testing the prompt in its intended application.

Shortening is a preference, not a fixed success criterion. Missing boundaries or
requirements may justify a longer prompt, and different tasks may require
different structures.
