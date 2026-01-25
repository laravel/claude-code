# Yii2 Claude Code Plugins

A collection of Claude Code plugins tailored for PHP / Yii2 development.

## Plugins

### yii2-simplifier

A code simplification agent that refines PHP / Yii2 code for clarity, consistency, and maintainability while preserving functionality.

**Features:**

- Applies common Yii2 conventions and standards
- Reduces unnecessary complexity and nesting
- Improves readability through clear naming
- Focuses on recently modified code by default
- Preserves all original functionality

## Installation

Add the marketplace and install the plugin:

```
/plugin marketplace add yii2/claude-code
```
```
/plugin install yii2-simplifier@yii2
```

## Usage

Once installed, tell Claude Code to use the agent after a long coding session:

```
> Review recent changes using the yii2-simplifier agent
```
