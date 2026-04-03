# Discord Channel for Claude Code — Multi-Project Fork

Fork of the official `discord@claude-plugins-official` plugin with **per-project namespace resolution**. Allows multiple Claude Code projects to use different Discord bots simultaneously.

## What's different

The official plugin hardcodes `DISCORD_STATE_DIR` in its `.mcp.json`, so every project shares the same bot token and access config. This fork adds `resolveStateDir()` which:

1. Looks up the parent process (Claude Code) working directory
2. Walks up directories looking for a `.discord-ns` file
3. Reads the channel namespace from it → resolves to `~/.claude/channels/<namespace>/`
4. Falls back to `~/.claude/channels/discord/` if no file found

## Setup

### 1. Add the marketplace

In `~/.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "bf-discord": {
      "source": {
        "source": "github",
        "repo": "Bettor-Faster/multiagent-discord-coworker"
      }
    }
  },
  "enabledPlugins": {
    "discord@bf-discord": true
  }
}
```

### 2. Disable the official plugin

Remove `"discord@claude-plugins-official": true` from `enabledPlugins` if present.

### 3. Create `.discord-ns` in each project root

```bash
# Project A
echo "discord-project-a" > ~/WorkProjects/project-a/.discord-ns

# Project B
echo "discord-project-b" > ~/WorkProjects/project-b/.discord-ns
```

### 4. Set up each channel

```bash
mkdir -p ~/.claude/channels/discord-project-a
echo "DISCORD_BOT_TOKEN=your-token-here" > ~/.claude/channels/discord-project-a/.env
chmod 600 ~/.claude/channels/discord-project-a/.env
```

Then use `/discord:access` to configure access control for each channel.

## License

Apache-2.0 (same as upstream)
