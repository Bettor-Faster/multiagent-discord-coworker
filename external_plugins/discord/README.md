# Discord Channel for Claude Code — Multi-Project Fork

Fork of the official `discord@claude-plugins-official` plugin that supports **per-project bot isolation** via the `DISCORD_STATE_DIR` environment variable. Each project can use its own bot token and access config.

## What's different

The official plugin defaults to `~/.claude/channels/discord/` for all projects. This fork is identical except it's distributed as a separate marketplace, so you can run it alongside the official plugin or point each project to a different state directory using `DISCORD_STATE_DIR`.

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

### 3. Create a Discord bot

1. Go to [Discord Developer Portal](https://discord.com/developers/applications) → New Application
2. Bot tab → Reset Token (copy it, shown only once)
3. Enable **MESSAGE_CONTENT** privileged intent
4. Turn off **Public Bot** to lock it down
5. Invite to your server with `bot` scope + `Send Messages`, `Read Message History`, `Add Reactions` permissions

### 4. Set up the channel state directory

```bash
mkdir -p ~/.claude/channels/my-project
echo "DISCORD_BOT_TOKEN=your-token-here" > ~/.claude/channels/my-project/.env
chmod 600 ~/.claude/channels/my-project/.env
```

### 5. Launch Claude Code

```bash
DISCORD_STATE_DIR=~/.claude/channels/my-project claude --dangerously-load-development-channels plugin:discord@bf-discord
```

> **Important:** Do NOT combine `--channels` and `--dangerously-load-development-channels` for the same plugin. Using both creates duplicate channel entries where the non-dev entry shadows the dev one, causing channel notifications to be silently dropped. Use only `--dangerously-load-development-channels`.

### 6. Configure access

Run `/discord:access` in the session, then DM your bot on Discord. It replies with a pairing code. Approve with:

```
/discord:access pair <code>
```

After pairing, lock it down:

```
/discord:access policy allowlist
```

### Verify

DM your bot — the message should appear inline in your Claude Code session.

## Multi-project usage

Each project gets its own state directory with a separate bot token and access config:

```bash
# Project A
DISCORD_STATE_DIR=~/.claude/channels/discord-project-a claude --dangerously-load-development-channels plugin:discord@bf-discord

# Project B
DISCORD_STATE_DIR=~/.claude/channels/discord-project-b claude --dangerously-load-development-channels plugin:discord@bf-discord
```

Without `DISCORD_STATE_DIR`, the plugin falls back to `~/.claude/channels/discord/`.

## License

Apache-2.0 (same as upstream)
