---
name: deploying-laravel-cloud
description: "Deploys and manages Laravel applications on Laravel Cloud using the `cloud` CLI. Use when the user wants to deploy an app, ship to cloud, create/manage environments, databases, caches, domains, instances, background processes, check billing/usage/spend, or any Laravel Cloud infrastructure. Triggers on deploy, ship, cloud management, environment setup, database provisioning, billing/usage queries, and similar cloud operations."
---
# Deploying with Laravel Cloud CLI

## Setup

```sh
composer global require laravel/cloud-cli
cloud auth -n
```

`cloud auth` opens a browser. Where that isn't possible, set `LARAVEL_CLOUD_TOKEN` in the environment — it overrides any saved token and writes nothing to disk. To save a token instead: `cloud auth:token --add --token=<token> -n`, or pipe it: `echo "$TOKEN" | cloud auth:token --add -n`.

## Commands

Commands follow a CRUD pattern: `resource:list`, `resource:get`, `resource:create`, `resource:update`, `resource:delete`.

Available resources: `application`, `environment`, `instance`, `database-cluster`, `database`, `cache`, `bucket`, `domain`, `websocket-cluster`, `background-process`, `secret`, `command`, `deployment`.

Some resources have additional commands (e.g., `domain:verify`, `database:open`, `instance:sizes`, `cache:types`). Discover these via `cloud -h`.

Never hardcode command signatures. Always run `cloud <command> -h` to discover options at runtime.

## CLI Flags

Always add `-n` to every command — prevents the CLI from hanging.
Never use `-q` or `--silent` — they suppress all output.

Flag combos per operation:
- Read (`:list`, `:get`) → `--json -n`
- Create (`:create`) → `--json -n`
- Update (`:update`) → `--json -n --force`
- Delete (`:delete`) → `-n --force` (no `--json`)
- Environment variables → `-n --force`
- Deploy/ship → `-n` with all options passed explicitly (no `--json`)

## Deployment Workflow

Determine the task and follow the matching path:

First deploy? → `cloud ship -n` (discover options via `cloud ship -h`)

Existing app? →
```sh
cloud deploy {app_name} {environment} -n --open
cloud deploy:monitor -n
```

Environment variables? → `cloud environment:variables -n --force`

Secrets? → `echo "$VALUE" | cloud secret:create --name=NAME --json -n` then `cloud environment-secret:attach`

Provision infrastructure? → `cloud <resource>:create --json -n`

Custom domain? → `cloud domain:create --json -n` then `cloud domain:verify -n`

For multi-step operations, see [reference/checklists.md](reference/checklists.md).

Not sure what the user needs? → ask them before running anything.

## When a Command Fails

1. Read the error output
2. Check resource status with `:list --json -n` or `:get --json -n`
3. Auth error? → `cloud auth -n`
4. Fix the issue, re-run the command
5. If the same error repeats after one fix, stop and ask the user

Always run `cloud deploy:monitor -n` after every deploy. If it fails, show the user what went wrong before attempting a fix.

## Subagent Delegation

Delegate high-output operations to subagents (using the Task tool) to keep the main context window small. Only the summary comes back — verbose output stays in the subagent's context.

Delegate these to a subagent:
- `cloud deploy:monitor -n` — deployment logs can be very long
- `cloud deployment:get --json -n` — full deployment details
- `cloud <resource>:list --json -n` — listing many resources produces large JSON
- Fetching docs from https://cloud.laravel.com/docs/llms.txt via `WebFetch`

Keep in the main context:
- Short commands like `:create`, `:delete`, `:update` — output is small
- `cloud deploy -n` — you need the deployment ID immediately
- Any command where you need the result for the next step right away

## Rules

Follow exact steps:
- Flag selection — always use the documented combos above
- Deploy sequence — deploy then monitor, never skip monitoring
- Destructive commands — always confirm with user first, show the command and wait for approval
- Error loop — diagnose, fix once, ask user if it fails again

Use your judgment:
- Instance sizes, regions, cluster types — ask the user if not specified
- Which resources to provision — based on what the user describes
- Order of provisioning — no strict sequence required
- How to present output — summarize, show raw, or extract fields based on context

## Secrets

Encrypted values shared across the organization and attached to environments. The CLI encrypts the value locally, so plaintext never reaches the API.

```sh
cloud secret:list --json -n
echo "$VALUE" | cloud secret:create --name=STRIPE_KEY --json -n
cloud environment-secret:attach {environment} {secretId} -n
cloud environment-secret:list {environment} --json -n
```

Pipe the value in. `--value=` works, but leaves the plaintext in shell history and the process list.

`secret:update`, `secret:delete`, and `environment-secret:attach` take secret IDs, not names — names are not unique. Read IDs from `cloud secret:list --json -n`.

There is no `secret:get`, and no way to detach a secret from a single environment. `secret:delete` removes it and detaches it everywhere.

Secrets need the `sodium` PHP extension. No other command does.

## Remote Access

### Tinker (>= v0.2.0)

Run PHP code directly in a Cloud environment:

```sh
cloud tinker {environment} --code='Your PHP code here' --timeout=60 -n
```

- `--code` — PHP code to execute (required in non-interactive mode)
- `--timeout` — max seconds to wait for output (default: 60)

The code must explicitly output results using `echo`, `dump`, or similar — expressions alone produce no output.

Always pass `--code` and `-n` to avoid interactive prompts.

### Remote Commands

Run shell commands on a Cloud environment:

```sh
cloud command:run {environment} --cmd='your command here' -n
```

- `--cmd` — the command to run (required in non-interactive mode)
- `--no-monitor` — skip real-time output streaming
- `--copy-output` — copy output to clipboard

Review past commands:

- `cloud command:list {environment} --json -n` — list command history
- `cloud command:get {commandId} --json -n` — get details and output of a specific command

Delegate `command:run` to a subagent when output may be long.

## Billing & Usage

View billing and usage for the current organization:

```sh
cloud usage --json -n
```

- `--period=current|previous|1|2|3` — billing period (default `current`; `1`/`2`/`3` are N periods back, max 3). Anything else errors out.
- `--environment=<id>` — filter usage to a single environment
- `--detailed` — include per-application, per-resource, and per-add-on breakdowns
- `--json` — machine-readable output (always pair with `-n`)

Common queries:

- Current spend: `cloud usage --json -n | jq '.currentSpendCents'`
- Last month's bill: `cloud usage --period=previous --json -n`
- One environment, full breakdown: `cloud usage --environment=<id> --detailed --json -n`

All amounts are in cents. Keys are camelCase at every level (e.g. `currentSpendCents`, `bandwidth.allowanceBytes`, `databases[].totalCents`, `applications[].totalCostCents`).

Delegate `--detailed --json` to a subagent — the payload includes every database, cache, bucket, websocket, and application and can get large.

## Config

1. Environment: `LARAVEL_CLOUD_TOKEN` — an API token, taking precedence over any saved one (empty counts as unset)
2. Global: `~/.config/cloud/config.json` — auth tokens and preferences
3. Repo-local: `.cloud/config.json` — app and environment defaults (set by `cloud repo:config {application} -n`)
4. CLI arguments override both config files

Pass the application to `repo:config` — without it the command has to ask, and under `-n` it fails when the organization has more than one application. Deploy commands don't need these defaults; pass the application and environment to them directly.

Multiple organizations means multiple stored API tokens. Every command reads `organization_id` from `.cloud/config.json` to pick one; if it isn't set, they fail. Set it with `cloud repo:config {application} --organization=<id|name|slug> -n`.

`LARAVEL_CLOUD_TOKEN` holds one token, so it picks the organization on its own. Naming a different one with `--organization` fails rather than falling back to the token's organization.

## Documentation

Laravel Cloud Docs: https://cloud.laravel.com/docs/llms.txt

When the user asks how something works or needs an explanation of a Laravel Cloud feature, fetch the docs from the URL above using `WebFetch` and use it to provide accurate answers.

## When Stuck

- Fetch https://cloud.laravel.com/docs/llms.txt for official documentation
- Run `cloud <command> -h` for any command's options
- Run `cloud -h` to discover commands
