# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A Claude Code plugin marketplace maintained by Chancetop. Provides a centralized catalog of plugins that extend Claude Code with custom commands, agents, hooks, and MCP servers.

## Marketplace Configuration

The marketplace is defined in `.claude-plugin/marketplace.json`:
- **Name**: `chancetop`
- **Owner**: Chancetop

Users add this marketplace with:
```shell
/plugin marketplace add chancetop-com/claude-code-plugins
```

## Creating a New Plugin

1. Create plugin directory at repository root:
   ```
   plugin-name/
     .claude-plugin/
       plugin.json      # Required manifest
     commands/          # Optional: .md files for slash commands
     agents/            # Optional: .md files for specialized agents
     skills/            # Optional: SKILL.md files for reusable prompts
   ```

2. Create `plugin-name/.claude-plugin/plugin.json`:
   ```json
   {
     "name": "plugin-name",
     "description": "Plugin description",
     "version": "0.0.1",
     "author": {
       "name": "Team Name"
     }
   }
   ```

3. Register in `.claude-plugin/marketplace.json` by adding to the `plugins` array:
   ```json
   {
     "plugins": [
       {
         "name": "plugin-name",
         "path": "./plugin-name"
       }
     ]
   }
   ```

4. Hooks and MCP servers are defined directly in `plugin.json` rather than as separate files.

## Validation

```shell
claude plugin validate .
```

Or within Claude Code:
```shell
/plugin validate .
```

## Testing Locally

```shell
/plugin marketplace add ./path/to/claude-code-plugins
/plugin install plugin-name@chancetop
```
