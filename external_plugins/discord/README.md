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

### 5. Launch Claude Code with the channel flag

```bash
claude --dangerously-load-development-channels plugin:discord@bf-discord
```

> **Important:** Do NOT combine `--channels` and `--dangerously-load-development-channels` for the same plugin. Using both creates duplicate channel entries where the non-dev entry shadows the dev one, causing the allowlist gate to silently reject channel notifications. Use only `--dangerously-load-development-channels`.

You can also pass `DISCORD_STATE_DIR` explicitly to bypass the `.discord-ns` lookup:

```bash
DISCORD_STATE_DIR=~/.claude/channels/discord-project-a claude --dangerously-load-development-channels plugin:discord@bf-discord
```

### Verify

Once launched, run `/discord:configure` in your session to check the connection status, or DM your bot — you should see the message appear inline.

## How namespace resolution works

When the plugin starts, `resolveStateDir()` resolves the state directory in this order:

1. `DISCORD_STATE_DIR` env var (if set, used directly)
2. Walk up from the Claude Code process's working directory looking for a `.discord-ns` file — read the namespace from it → `~/.claude/channels/<namespace>/`
3. Fall back to `~/.claude/channels/discord/`

## License

Apache-2.0 (same as upstream)
