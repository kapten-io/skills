---
name: kapten
description: Explains Kapten's resource model and operates Kapten through its CLI. Use when reasoning about Kapten organizations, projects, environments, clusters, components, applications, jobs, databases, or pipelines, or when inspecting or changing Kapten resources.
compatibility: Requires the kapten CLI for live resource operations; authentication and network access may be required.
---

# Kapten

Use Kapten's domain hierarchy to choose the right resource, then let the installed CLI describe its current command surface and schemas.

## Choose the relevant context

- Read [the concept model](references/concepts.md) when the task involves resource ownership, hierarchy, deployment, runs, authorization scope, or translating the user's intent into Kapten resources.
- Read [the CLI workflow](references/cli.md) before executing, composing, or troubleshooting `kapten` commands.
- For a conceptual question that does not require live state, answer from the concept model without requiring CLI access.

## Operate through discovery

Do not rely on a memorized operation list: the CLI is generated from Kapten's evolving OpenAPI contract.

1. Run `kapten --help` to discover current resource groups and global controls.
2. Run `kapten <group> --help` to discover that group's operations.
3. Run `kapten <group> <operation> --help` before composing a request. Treat its positional arguments, examples, request schema, required fields, enums, and response schema as authoritative.
4. Resolve required parent and resource IDs from live list/get output; do not substitute names where help requires IDs.
5. Execute with the intended context. Prefer `-o json` when subsequent reasoning or shell processing depends on exact fields.
6. After a mutation, verify the resulting resource with a read command discovered through help. Report the affected resource name and ID.

When the user has explicitly requested a state change, carry it through discovery, execution, and verification. Do not stop merely because the exact operation was not known before inspecting help.

## Guardrails

- Confirm the active organization/context and target hierarchy before changing or deleting resources. A Kapten context selects an identity, organization, and API endpoint; it is not a Project or Environment.
- Follow the live request schema rather than guessing body fields. Quote inline bodies so the shell does not expand punctuation or wildcards.
- Use default human output for interactive explanation and explicit JSON for automation. Do not scrape decorative tables when JSON is available.
- Treat authorization failures as server decisions. Do not work around them by changing identifiers or contexts unless the user requested that context.
- Never expose access, refresh, or identity tokens. The CLI manages credentials; use its auth commands rather than extracting its credential store.
- For unexpected HTTP behavior, reproduce once with verbose output and diagnose from status, response body, target URL, and context. Do not leave verbose headers enabled by default.

## Completion

A live operation is complete when the command succeeds, its result is verified through a discovered read path, and the response belongs to the intended organization hierarchy. If it fails, report the command shape, context name (never tokens), HTTP status or transport error, and the next actionable correction.
