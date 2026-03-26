# Catallaxy

> Last updated: 2026-03-26. This is a trimmed-down README — the full version with detailed requirements, install guide, and documentation lives at [catallaxy.app/readme](https://catallaxy.app/readme).

Distributed AI observation for Claude Code sessions.

Catallaxy runs AI observer agents alongside your active coding session. Observers watch the full transcript in real time and surface observations when they detect issues — wrong assumptions, bugs, security concerns, architectural missteps, or missed alternatives.

> **Beta** — This software is in closed beta. Expect rough edges. Your feedback shapes what ships.

## Important:

Catallaxy orchestrates observer agents over your CLI subscription to their respective providers (Google Gemini, OpenAI Codex, Anthropic Claude). Each review cycle generates usage on your accounts. Monitor your provider dashboards for cost visibility.

## Requirements

- macOS (only verified on Tahoe 26+)
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed — this is what Catallaxy observes
- **Optional but recommended:** [Gemini CLI](https://github.com/google-gemini/gemini-cli) and/or [Codex CLI](https://github.com/openai/codex) installed. Model diversity gives you a diversity of perspectives, but you can run all observers on Claude if you prefer.

## Quick start

1. Install Catallaxy from your invite download link
2. Launch the app — it hooks on to Claude Code sessions automatically
3. Your first Claude Code prompt activates the observation room
4. Observers review your session and surface observations in the app

## Uninstall

1. Quit Catallaxy
2. Move Catallaxy.app to Trash
3. Remove the data directory: `rm -rf ~/.catallaxy`
4. Remove Claude Code hooks (if installed):
   - Check `~/.claude/settings.json` for Catallaxy hook entries and remove them

## Feedback & issues

Found a bug or have a suggestion? [Open an issue](../../issues/new/choose) — I check this regularly.
