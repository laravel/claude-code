# Laravel Agents & Skills

A collection of agents and skills for PHP / Laravel development, available as plugins for [Claude Code](https://code.claude.com) and [Cursor](https://cursor.com), with support for [OpenCode](https://opencode.ai).

## Plugins

### laravel-simplifier

An agent that reviews recently modified PHP / Laravel code and refines it for clarity, consistency, and maintainability — without changing functionality.

- Applies Laravel conventions and PSR-12 standards
- Reduces unnecessary complexity and nesting
- Improves naming and readability
- Focuses on recently modified code by default

### laravel-cloud

A skill for deploying and managing applications on [Laravel Cloud](https://cloud.laravel.com) using the `cloud` CLI.

- Guides deployment workflows (first deploy, existing apps, environment setup)
- Manages infrastructure resources (databases, caches, domains, buckets)
- Includes operational checklists for multi-step tasks
- Follows the CRUD command patterns of the Cloud CLI

> The skill content is sourced from [`laravel/cloud-cli`](https://github.com/laravel/cloud-cli/tree/main/skills/deploying-laravel-cloud) via a sparse-checkout submodule.

## Installation

### Claude Code

```
/plugin marketplace add laravel/agent-skills
/plugin install laravel-simplifier@laravel
/plugin install laravel-cloud@laravel
```

### OpenCode

Copy the plugin directories into your global `~/.config/opencode/` folder:

**laravel-simplifier**

```bash
mkdir -p ~/.config/opencode/agents
cp -r path/to/agent-skills/laravel-simplifier/.opencode/agents/. ~/.config/opencode/agents/
```

**laravel-cloud**

```bash
mkdir -p ~/.config/opencode/skills
cp -r path/to/agent-skills/laravel-cloud/.opencode/skills/. ~/.config/opencode/skills/
```

Or place them in your project's `.opencode/` directory for project-scoped access:

```bash
mkdir -p .opencode/agents .opencode/skills
cp -r path/to/agent-skills/laravel-simplifier/.opencode/agents/laravel-simplifier.md .opencode/agents/
cp -r path/to/agent-skills/laravel-cloud/.opencode/skills/deploying-laravel-cloud .opencode/skills/
```

## Usage

**laravel-simplifier** — invoke the agent after a coding session:

```
> Review recent changes using the laravel-simplifier agent
```

**laravel-cloud** — triggers automatically when you ask about deploying or managing Laravel Cloud resources:

```
> Deploy my app to Laravel Cloud
> Set up a database and cache for the staging environment
```

## Development

After cloning, initialize the submodule to pull the Laravel Cloud skill:

```bash
git clone --recurse-submodules https://github.com/laravel/agent-skills.git
```

Or if already cloned:

```bash
git submodule update --init
```

The submodule uses sparse checkout — only the `skills/deploying-laravel-cloud` directory is checked out from `laravel/cloud-cli`.
