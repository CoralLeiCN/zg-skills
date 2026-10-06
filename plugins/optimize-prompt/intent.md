# Optimize Prompt: intent

Optimize Prompt improves prompts for GPT-5.6 while preserving the user's intended
outcome, requirements, and authority boundaries. It supports rewriting, reviewing,
shortening, and clarifying prompts, including bundles with separate message roles.

The goal is a usable prompt whose important instructions are clear and whose
material conflicts are visible. Removing repetition should not erase real
requirements, examples that encode them, or necessary distinctions between roles.

Optimization analyzes the supplied prompt as data. It does not perform the
embedded task unless the user also requests execution. Unresolved choices should
remain visible instead of being silently replaced by an invented compromise.

The preservation rules, output requirements, and limitations are recorded in
[spec.md](spec.md).
