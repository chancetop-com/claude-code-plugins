# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Claude Code plugin marketplace maintained by Chancetop. It provides a centralized catalog of plugins that extend Claude Code with custom commands, agents, hooks, and MCP servers.

## Repository Structure

```
.claude-plugin/
  marketplace.json        # Marketplace catalog defining available plugins
core-ng/                  # Plugin for core-ng
  .claude-plugin/
    plugin.json           # Plugin manifest
  skills/                 # Reusable skills
  CHANGELOG.md            # Plugin changelog
```

## Marketplace Configuration

The marketplace is defined in `.claude-plugin/marketplace.json` with:
- **Name**: `chancetop`
- **Owner**: Chancetop
- **Plugin root**: Root of this repository

Users can add this marketplace with:
```shell
/plugin marketplace add chancetop-com/claude-code-plugins
```

## Plugin Development Workflow

### Creating a New Plugin

1. Create plugin directory structure:
   ```bash
   mkdir -p plugin-name/{.claude-plugin,commands,agents,skills,hooks,mcp-servers}
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

3. Add plugin to `.claude-plugin/marketplace.json` in the `plugins` array

4. Test locally:
   ```shell
   /plugin marketplace add ./path/to/claude-code-plugins
   /plugin install plugin-name@chancetop
   ```

### Plugin Components

- **Commands** (`.md` files in `commands/`): Custom slash commands
- **Agents** (`.md` files in `agents/`): Specialized AI agents for complex tasks
- **Skills** (`SKILL.md` files in `skills/`): Reusable prompt templates
- **Hooks** (defined in `plugin.json`): Event-triggered automations
- **MCP Servers** (defined in `plugin.json`): External service integrations

### Validation

Validate marketplace and plugin configuration:
```shell
claude plugin validate .
```

Or from within Claude Code:
```shell
/plugin validate .
```

## Current Plugins

### core-ng
core-ng development tools and workflows. Install with:
```shell
/plugin install core-ng@chancetop
```
