# 05 — Claude Code (CLI + VS Code extension)

I use Claude Code from the terminal and through the official VS Code
extension. Both are pointed at DeepSeek's Anthropic-compatible endpoint, using
the same model settings.

> **Never commit the real API key.** This chapter uses
> `<YOUR-DEEPSEEK-API-KEY>`. The real values live only in
> `~/.config/fish/config.fish` and `~/.claude/settings.json`, outside this
> repository. Restrict both files with `chmod 600`, and rotate the key if it
> has ever been shared or committed.

## 1. Install the CLI

```bash
curl -fsSL https://claude.ai/install.sh | bash
claude --version
```

If `claude` is not found afterwards, make sure `~/.local/bin` is on `PATH` and
open a new shell.

## 2. Fish configuration for the CLI

Edit the fish config — no `sudo`; this is your own file:

```bash
micro ~/.config/fish/config.fish
```

Add:

```fish
set -gx ANTHROPIC_BASE_URL https://api.deepseek.com/anthropic
set -gx ANTHROPIC_AUTH_TOKEN "<YOUR-DEEPSEEK-API-KEY>"
set -gx ANTHROPIC_MODEL "deepseek-flash[1m]"
set -gx ANTHROPIC_DEFAULT_OPUS_MODEL "deepseek-flash[1m]"
set -gx ANTHROPIC_DEFAULT_SONNET_MODEL "deepseek-flash[1m]"
set -gx ANTHROPIC_DEFAULT_HAIKU_MODEL "deepseek-flash"
set -gx CLAUDE_CODE_SUBAGENT_MODEL "deepseek-flash"
set -gx CLAUDE_CODE_EFFORT_LEVEL "max"
set -gx CLAUDE_CODE_AUTO_COMPACT_WINDOW "786432"
```

Reload:

```fish
source ~/.config/fish/config.fish
```

`ANTHROPIC_AUTH_TOKEN` is the variable DeepSeek expects for the Bearer token;
`ANTHROPIC_API_KEY` is not the right one for this endpoint. If the file already
has content, append these lines rather than replacing it.

A cleaner option is to put the token lines in
`~/.config/fish/conf.d/deepseek.fish` (Fish loads that directory
automatically), set `chmod 600` on it, and keep `config.fish` free of secrets.

## 3. Shared settings file

The VS Code extension does not reliably inherit shell environment variables,
so it reads `~/.claude/settings.json`:

```bash
mkdir -p ~/.claude
micro ~/.claude/settings.json
chmod 600 ~/.claude/settings.json
```

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://api.deepseek.com/anthropic",
    "ANTHROPIC_AUTH_TOKEN": "<YOUR-DEEPSEEK-API-KEY>",
    "ANTHROPIC_MODEL": "deepseek-flash[1m]",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "deepseek-flash[1m]",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "deepseek-flash[1m]",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "deepseek-flash",
    "CLAUDE_CODE_SUBAGENT_MODEL": "deepseek-flash",
    "CLAUDE_CODE_EFFORT_LEVEL": "max",
    "CLAUDE_CODE_AUTO_COMPACT_WINDOW": "786432"
  }
}
```

If `~/.claude/settings.json` already exists, merge the `"env"` object into it
instead of replacing the file. The key is now stored in two places — Fish and
this file — so update both when rotating it.

## 4. Test the CLI

```bash
cd /path/to/your-project
claude
```

Ask "What model are you?" to confirm the DeepSeek endpoint responds.

## 5. VS Code extension

Install the official extension:

```bash
code --install-extension anthropic.claude-code
```

Or use the Extensions view (`Ctrl+Shift+X`) and search for **Claude Code**.
Restart VS Code completely after installing.

To skip the Anthropic login prompt, add this to VS Code's **User Settings
(JSON)** (`Ctrl+Shift+P` → "Preferences: Open User Settings (JSON)"):

```json
{
  "claudeCode.disableLoginPrompt": true
}
```

This is a VS Code setting, so it belongs in VS Code's `settings.json`, not in
`~/.claude/settings.json`. Merge it into the existing object if necessary.
Restart VS Code again, then open the Claude Code panel.

If the extension still prompts for login, check that this setting is in the
user settings file, that `~/.claude/settings.json` is valid JSON, and that VS
Code was fully restarted.
