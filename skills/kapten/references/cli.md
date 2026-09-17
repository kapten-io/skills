# Kapten CLI workflow

Read this reference before running, composing, or diagnosing `kapten` commands.

## Discover before composing

The installed CLI embeds an OpenAPI contract. Its help is the source of truth for the installed version; do not copy an operation catalog into plans or scripts.

```bash
kapten --help
kapten <group> --help
kapten <group> <operation> --help
```

Use each level for a different decision:

- Root help shows current resource groups, global flags, and representative examples.
- Group help shows current operations in that group.
- Operation help shows positional arguments, an input example, the request schema, required fields marked with `*`, field descriptions and enums, response schemas, and operation-specific flags.
- Add `--help-all` when ordinary help hides inherited Restish controls.

If a command you expected is absent, do not synthesize it from API conventions. Report that it is not exposed by the installed CLI or inspect adjacent groups through help.

## Select authentication context

Inspect contexts before acting when the target Organization is not already clear:

```bash
kapten auth list
```

Use the active context by default. Use `--context <name>` for one command when the user identifies another existing context. Use the CLI's `auth login`, `auth use`, and `auth logout` help when authentication state must change.

An OAuth discovery error points to the issuer; a connection-refused API request points to the API endpoint or service. They are different paths even when one context stores both.

## Resolve the hierarchy

Start at the nearest known parent and list downward. Capture IDs from JSON when another command needs them:

```bash
kapten <parent-group> list -o json
```

Read the operation's usage line to preserve argument order. Environment and component-family commands commonly need Project and Environment IDs before the resource ID. Do not assume an ID alone is sufficient.

## Build request bodies from help

Before create or update:

```bash
kapten <group> <operation> --help
```

Build only fields accepted by the displayed request schema. Supply every required field, preserve enum spelling and units, and use the displayed example as a shape rather than blindly copying its placeholder values.

For a small body, pass one quoted JSON or compact YAML-style argument following the help example. For a large or generated body, prefer JSON on stdin. If operation help exposes request validation, use it before a consequential mutation.

Never guess schema fields from a similar resource: Application, Job, and each Database engine have different configuration unions and requirements.

## Choose output deliberately

Default output is designed for humans: list pagination is unwrapped into tables, useful columns are prioritized, and ordinary responses omit HTTP headers.

Use JSON whenever exact structure matters:

```bash
kapten <group> <operation> ... -o json
```

This is the preferred mode for `jq`, scripts, comparisons, and capturing IDs. Keep error output intact when reporting a failure; the default formatter includes the public API message and HTTP status.

For protocol diagnosis, add `-v` to show request/response headers. Add `-vv` only when TLS details are relevant. Headers can contain operational metadata, so include only the necessary diagnosis in the final report and never expose authorization values.

## Mutate and verify

For create, update, or delete:

1. Confirm the selected context and parent hierarchy.
2. Read operation help and construct the request from its schema.
3. Execute the requested mutation.
4. Capture the returned ID, or retain the target ID for deletion.
5. Discover and run the corresponding get/list path to verify resulting state. After deletion, verify absence or the API's documented deleted state.

Do not treat a zero exit alone as sufficient when a follow-up read is available.

## Diagnose failures

Keep failures at the layer where they occurred:

- Discovery failure: inspect the configured issuer and its advertised OAuth endpoints.
- Authentication failure: inspect context selection and login/refresh state without reading raw credential files.
- Connection refused or DNS failure: inspect the API endpoint and service reachability.
- `401`: refresh/login may be required; do not substitute another identity silently.
- `403`: the server denied the operation in this Organization/resource scope.
- `404`: recheck endpoint version, argument order, parent IDs, and target ID.
- `409`: inspect the domain conflict in the response and choose a user-approved correction.
- `5xx`: retry only when safe, then use `-v` to capture the request target and response status for diagnosis.

When reporting a failure, include the command shape with sensitive values omitted, context name, HTTP status or transport error, and the next concrete action.
