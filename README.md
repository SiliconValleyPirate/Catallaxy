# Catallaxy

> Distributed AI observation for your coding sessions.

**Status:** Closed beta · invite-only · macOS only
**Version:** 0.66.1 · **Document last updated:** 2026-04-27

Catallaxy runs passive AI observer agents alongside an active coding session. While you work with Claude Code, observers watch the full transcript in real time and surface their thoughts when they detect issues of sufficient degree. Each observer runs its own connection to the a model provider of your choice. You don't speak to them directly (not yet, anyway). They watch, and they speak up when it matters.

This README is the canonical place to learn what Catallaxy is, how to install it, and what it does to your machine. If you're an invited beta tester, you've already received an invite code and a download link. If you're interested, sign up at catallaxy.app.

---

## Contents

- [What Catallaxy is](#what-catallaxy-is)
- [Requirements](#requirements)
- [Installing](#installing)
- [Your first observer](#your-first-observer)
- [What you should know before running it](#what-you-should-know-before-running-it)
- [Reporting bugs](#reporting-bugs)
- [Roadmap](#roadmap)
- [License and terms](#license-and-terms)
- [Third-party software](#third-party-software)
- [Acknowledgments](#acknowledgments)

---

## What Catallaxy is

Catallaxy is a macOS app that runs *next to* your AI coding tools, not in front of them. It observes your active Claude Code sessions through Anthropic's hook system, accumulates the session transcript, and dispatches that transcript to one or more observer agents — long-running CLI sessions of your choice (Claude Code, Codex, or Gemini CLI, at the moment) — that you've configured with their own personalities and instructions.

The observers don't drive your work. They watch, and they push back when they think they should. A senior-engineer observer might flag a security smell. A test-discipline observer might note that you just touched a file that has no tests. A scope observer might warn that the change you're making is bigger than the task you described. You can keep coding while they think; reviews arrive as in-app notifications you read on your own time.

Each observer runs locally, on your machine, using *your* subscription for whichever LLM they're configured against. Your transcripts, observer responses, and your code, never get sent to any Catallaxy backend. The only things it sends to its backend are license validation, update checks, and (if you choose to use it) interactions with the Freehold marketplace where personas and lenses can be published, downloaded, etc.

The product takes its name from F.A. Hayek's term for the order produced by distributed individual knowledge — many specialized observers, each watching from a different angle, each producing a bit of knowledge that collectively creates a far more intelligent outcome.

And yes, Catallaxy takes inspiration from Planescape, Star Trek, and many other sources too!

---

## Requirements

| Requirement | Detail |
|------------|--------|
| **Operating system** | macOS 13 Ventura or newer (Apple Silicon recommended) |
| **Architecture** | Apple Silicon (`arm64`). |
| **Claude Code** | Installed and on your `PATH`. Catallaxy hooks into Claude Code; without it, there's nothing to observe. |
| **At least one observer CLI** | One or more of: Claude Code (`claude`), Codex CLI (`codex`), Gemini CLI (`gemini`). Each must be authenticated with the corresponding provider before Catallaxy can use it. |

---

## Installing

You'll have received an invite link of the form:

```
https://catallaxy.app/download/<your-invite-code>
```

The page shows your license key once and lets you download the installer. Save the license key somewhere durable — you'll need it on first launch.

1. Download the `.dmg`.
2. Open it, drag **Catallaxy** to `/Applications`.
3. Eject the `.dmg`.
4. Launch Catallaxy from `/Applications` or Spotlight.

The first launch will prompt you for your license key. After validation, the app guides you through a brief onboarding — including a one-time injection of the [observer dispatch signal snippet](#a-snippet-in-your-projects-claudemd) into your Claude Code project's `CLAUDE.md` (you'll be asked before this happens).

If macOS shows a Gatekeeper warning, the build is signed and notarized; you should not need to use a "right-click → Open" workaround. If you do, please file a bug. https://github.com/SiliconValleyPirate/Catallaxy/issues

### Uninstalling

To remove Catallaxy completely:

1. Quit the app.
2. Move `Catallaxy.app` from `/Applications` to the Trash.
3. Remove your data directory: `rm -rf ~/.catallaxy/`
4. Remove Catallaxy's hook entries from `~/.claude/settings.json`. The installer left a backup at `~/.claude/settings.json.catallaxy-backup` if you want to restore the pre-install state.
5. Remove the snippet that onboarding added to your project's `CLAUDE.md` files (search for `## Observer Dispatch Signal` followed by a paragraph mentioning the `⌁` character).

---

## Your first observer

The full workflow takes a few minutes:

1. **Enter your license key.** This validates against `catallaxy.app` and registers your device. See [What it reads, what stays local, what crosses the network](#what-it-reads-what-stays-local-what-crosses-the-network) below for the full payload.
2. **Connect to Claude Code.** The Settings → Integration toggle installs Catallaxy's hooks into your user-level Claude Code config (`~/.claude/settings.json`). Append-only — your existing hooks are preserved. A backup is taken before the first modification.
3. **Pick or create a persona.** Personas are the creative identities that observers wear: a name, a voice, a knowledge focus, a calibration of when to speak up. Catallaxy ships with a starter pack you can install from Freehold, or you can author your own in the Scriptorium.
4. **Drop a persona into a room.** Rooms are persistent observation contexts (typically per-project). Drag a summon configuration into a room from the left-hand sidebar; an observer is born.
5. **Open a Claude Code session in that project.** Catallaxy detects it through the hook system, routes the session to the matching room, and the observer begins watching. You can use your terminal or the Claude Code app's "code" area.
6. **Wait for a review.** Observers can be dispatched manually by pressing "consult". They'll read up to the last thing that was said in your Claude Code transcript. They can also be set to "automatic" to consult with all observers in your room session automatically whenever Claude Code finishes responding in your coding session.

You can run multiple observers in the same room — they'll review in parallel, each with their own perspective.

---

## What you should know before running it

### Your subscription, your usage

Catallaxy does not provide LLM access. Every observer in every room is a separate CLI process that runs against *your* subscription with the relevant provider — your Anthropic subscription for Claude Code observers, your OpenAI subscription for Codex CLI observers, your Google subscription for Gemini CLI observers.

That means **observer reviews consume your subscription's usage allowance**, the same way running the CLI directly does. We never see your usage and never set those allowances. Heavy use — many observers running often — will exhaust your allowance faster than using the CLI directly would. If your subscription has rate windows ("X messages per Y hours") or credit pools, observers count against them.

Note: this describes the default-supported subscription-based adapters. If you customize a binding to invoke a different tool (for example, one that uses a paid API key directly), that observer's costs follow whatever that tool charges per call.

We are not responsible for any allowance you consume on third-party provider subscriptions. See [License and terms](#license-and-terms) for the formal version.

### What it reads, what stays local, what crosses the network

Catallaxy reads your Claude Code session transcripts in order to dispatch them to observers (to send the request to Anthropic, OpenAI, or Google). Those transcripts may contain anything that appeared in your session: source code, terminal output, file contents, conversation history, secrets that scrolled past in the terminal.

Before a transcript ever gets sent out to a model, Catallaxy runs a credential scanner that redacts known credential patterns (API keys, PEM private keys, common token prefixes). This is defense-in-depth, but good security and key hygiene starts at your own Claude Code terminal.

**The desktop app makes two kinds of network calls - the ones it sends over to the models, and the ones it sends over to catallaxy.app. These are the only ones we send over to Catallaxy's backend:**

- **License validation** when you enter your license key during onboarding (`POST catallaxy.app/api/v1/license/validate`). Payload: license key, a UUID we generate to identify your device, the product name, app version, OS, and architecture. *Note: re-validation on every subsequent launch ships in a near-term update.*
- **Update check** on launch (`GET catallaxy.app/api/v1/version/check`). Payload: current version, platform, architecture. *Note: auto-update is not yet wired in this build; the check endpoint is in place but the desktop integration ships in a near-term update.*
- **Freehold marketplace** requests when you initiate them, plus a few automatic checks: browse, install, publish, recommend, comment, profile (read/edit/avatar), default-package bootstrap on first run, and update checks for content you've installed. Payload: only what's required for that action — your bearer token (the license key) for authentication, your text and any image/video files for publishing actions.
- **Bug reports** — these are not Catallaxy network calls. The bug-report button opens your default browser to the GitHub issue tracker with the app version pre-filled in the issue title. No Catallaxy backend involvement.

**The desktop app does not transmit the following to Catallaxy's backend:**

- Your transcripts, ever.
- Observer responses, ever.
- Persona, lens, or package content, *unless you publish it to Freehold*. Drafts and locally-installed copies stay local.
- Configuration or hook tokens.
- Telemetry, analytics, or implicit usage data of any kind.

If we are wrong about any of this, [please file a bug](#reporting-bugs) — we treat privacy claims as load-bearing and want to fix any drift between what we say here and what the app does.

### A snippet in your project's `CLAUDE.md`

Catallaxy's review timing - when set to "automatic" mode - depends on knowing when your AI tool has finished a unit of work. The signal we use is a single character (`⌁`, U+2301 "electric arrow") emitted at the end of a complete response. Without that signal, Catallaxy falls back to a 30-second idle timer, which is functional but introduces latency into every review.

During onboarding, Catallaxy can append a short section to your project's `CLAUDE.md` that asks the AI to emit `⌁` at end-of-turn. The full snippet is short and human-readable; you'll see exactly what's being added before agreeing to it.

If you decline, Catallaxy still works — reviews just dispatch via the timer. You can also add the snippet to other projects manually any time.

### The hooks Catallaxy installs

Catallaxy uses Claude Code's hook system to learn when sessions start, when prompts are submitted, when tools complete, and when Claude stops. The hooks point at a local-only HTTP server (`127.0.0.1:19191`) authenticated by a per-install shared secret. The secret lives in your config (`~/.catallaxy/config.toml`), is mirrored to `~/.catallaxy/.hook-token` for the command-relay path, and is embedded in the HTTP hook entries the installer writes into `~/.claude/settings.json`. None of those files leave your machine.

Hooks are installed at the **user level** (`~/.claude/settings.json`), never per-project. This is a deliberate security choice: project-level hooks committed to git have been the vector for at least two reported CVEs against Claude Code (`CVE-2025-59536`, `CVE-2026-21852`), and we want no part of that surface area.

The installer is append-only. Your existing hooks are preserved. Before the first modification, a backup is written to `~/.claude/settings.json.catallaxy-backup`. On launch, the installer detects its own existing entries (by URL or command marker) and never duplicates them.

You can disable auto-install via the **Settings → Integration** toggle. You can also remove the hooks manually — see [Uninstalling](#uninstalling).

### Where Catallaxy stores its data

Everything Catallaxy persists is under `~/.catallaxy/`. Concrete contents:

| Path | What it is |
|------|-----------|
| `config.toml` | Operational config: bindings, rooms, summon configurations, app settings. Hand-editable for troubleshooting. |
| `personas/<name>/` | One subdirectory per persona, each with `persona.toml` (creative fields), images, and version history. |
| `lenses/<name>/` | One subdirectory per lens, with `lens.toml`, `prompt.md`, an optional icon, and any payload files. |
| `packages/<name>/` | Authored package drafts and installed package receipts. |
| `holding-pen/` | Sessions Catallaxy detected but couldn't yet route to a room. You assign or dismiss them from the in-app badge. |
| `freehold-state.json` | UI-only state (e.g., dismissed Freehold notifications). Deletable without breaking the app. |
| `.hook-token` | A copy of the hook authentication secret, used by the command-relay path. The primary copy lives in `config.toml`. |
| `dispatch-trace.jsonl` | Optional debug trace. Created only if `dispatch_trace = true` in `config.toml`. Capped at 2 MB. |

`config.toml` carries your installation's secret material (license key, hook token). It and the related local files (the `.hook-token` mirror, the hook entries embedded in `~/.claude/settings.json`) are local-only — Catallaxy never transmits them.

---

## Reporting bugs

Click the bug icon in Catallaxy's left sidebar (next to **Pause All**), or use **Help → Report Issue** from the menu bar. Either action opens your default browser to the Catallaxy issue tracker on GitHub, with the app version pre-filled in the title and in the issue form's structured `app-version` field. You fill in what happened, the steps to reproduce, your OS version, and which observer adapter (if any) was involved.

The bug-report button does not transmit anything from Catallaxy itself — it just opens a URL. Nothing about your transcripts, observer responses, personas, lenses, configurations, or hook tokens reaches us through this path. If you want to share a snippet of an observer response or a transcript fragment in support of a bug report, copy and paste it manually into the issue body; we don't auto-collect it.

For questions, suggestions, or anything that isn't a bug: please email us. The address is on `catallaxy.app`.

---

## Roadmap

Some upcoming features:

- **Browser Extension:** a Chrome extension is planned post-beta to bring observers to writing tasks outside the CLI.
- **More Models to Choose From:** Grok is the next one we have planned.
- **Catallaxy-specific Memory System:** Specifically of the 'narrative and reflection' type.
- **Meta-Observer** Helps you pick the right observer for what's currently happening. Can offer feedback on the observers themselves directly.
- **Bring-Your-Own API Key:** For users who want to run the models on their API keys instead of their subscription access.

The shape is subject to change based on what testers tell us.

---

## License and terms

This section is the closed-beta license between Catallaxy ("we", "us", "I") and you ("the licensee"). It's written in plain language to be readable, but the terms are operative — by installing or running Catallaxy, you agree to them.

The full Terms of Service that you accepted at download time are the binding version. This README provides a plain-language summary for your reference; where the two diverge, the Terms of Service govern. The Terms of Service are available at `catallaxy.app`.

### License grant

We grant you a personal, non-exclusive, non-transferable, revocable license to install and run Catallaxy on devices you own or control, up to the per-device limit associated with your license key, for the duration of the closed beta. The license is yours, not transferable to anyone else.

### What you can do

- Run Catallaxy for any lawful purpose, personal or commercial.
- Author personas, lenses, and packages, and keep them locally on your machine.
- Publish content you authored to the Freehold marketplace, subject to the [Freehold marketplace terms](#freehold-marketplace) below.
- Discuss your experience with Catallaxy publicly.

### What you can't do

- **Redistribute the application binary.** The `.dmg`, the `.app`, the underlying executables — none of these may be passed to anyone else, posted publicly, mirrored, repackaged, or otherwise distributed. Each invite and license key is bound to a specific person and a specific small number of devices.
- **Share license keys or invite codes.** Both are credentials. We can revoke them if they leak.
- **Reverse-engineer, decompile, or disassemble** the application beyond what local laws explicitly permit despite this restriction.
- **Sublicense, sell, rent, or lease** Catallaxy or any portion of it.
- **Use the Catallaxy name or logo** to imply endorsement of, or affiliation with, any product, service, or content that we have not authorized.

### Beta software — no warranty

Catallaxy is provided **AS IS** and **AS AVAILABLE**, without warranty of any kind, express or implied, including (but not limited to) warranties of merchantability, fitness for a particular purpose, non-infringement, accuracy, or availability. Beta software contains defects we have not yet found. You accept that risk by using it.

### Limitation of liability

To the maximum extent permitted by law, our total cumulative liability arising from or relating to your use of Catallaxy is limited to the greater of (a) the amount you paid us for the software (during closed beta, this is zero) or (b) USD $50. We are not liable for indirect, incidental, special, consequential, or punitive damages, including but not limited to lost profits, lost data, or business interruption.

This limitation applies regardless of the legal theory and even if we have been advised of the possibility of such damages.

### Privacy

Catallaxy is local-first by design. Concretely:

- **What we receive from your desktop:** license validation payloads (license key, a per-install device UUID, app version, platform, architecture, OS string) and version-check payloads (current version, platform, architecture).
- **What we receive when you use Freehold:** the content you choose to publish, your downloads, your recommends and comments, and your profile fields. All of this requires explicit user actions on your end.
- **What we never receive:** your transcripts, your observer responses, your personas/lenses/packages unless you publish them, your config files, your hook token, your code, or any telemetry/analytics about how you use the app.
- **What we store:** license records, installation records (one per device), phone-home log entries (used for license auditing, with hashed IP addresses, pruned after 90 days), and Freehold publications you author. We do not have your transcripts to store.

If you submit a bug report, you do so by opening your browser to GitHub and filling in a form yourself. Catallaxy itself transmits nothing for bug reports. Anything you choose to paste into the issue body is what we receive.

### Freehold marketplace

If you publish a persona, lens, or package to Freehold:

- You represent that you have the right to publish the content (you authored it, or it's appropriately licensed for redistribution).
- You grant us a non-exclusive, royalty-free, worldwide license to host it, serve it to other users, and display it in marketplace listings, for as long as you keep it published.
- You grant other Catallaxy users a non-exclusive, royalty-free license to download it, install it locally, and use it for their own observation purposes.
- You retain authorship and may withdraw your publication at any time. Withdrawn publications are hidden from browse and your stall, but URLs already in the wild remain resolvable — withdrawal is not a takedown.
- We may remove or restore publications at our discretion in service of marketplace quality and safety, especially during pre-beta where the marketplace is curated rather than open.

You agree not to publish content that is illegal, infringes others' rights, contains malware, exfiltrates user data, or is designed to cause harm. We can revoke your publishing privileges if you do.

### Trademark

"Catallaxy" and the Catallaxy logo are trademarks of the developer (formal registration is post-beta). Don't use the name or logo to suggest your work is associated with, endorsed by, or affiliated with us unless we've said so in writing.

### Termination

This license terminates automatically if you breach its terms or if your license key is revoked. The closed-beta license also terminates when the closed-beta period ends; we'll communicate replacement terms before that happens. On termination, you must stop using Catallaxy and remove it from your devices. The privacy provisions, the warranty disclaimer, and the limitation of liability survive termination.

### Changes to these terms

We may update these terms during the closed beta. The version date at the top of this document is authoritative. Material changes will be communicated to invited testers directly.

### Choice of law

This license is governed by the laws of the State of Texas, USA, without regard to conflict-of-laws principles. Any dispute arising from or relating to this license shall be brought exclusively in the state or federal courts located in Texas.

### Source code

This GitHub repository is the public-facing issue tracker for the Catallaxy beta. **It does not contain the source code.** The Catallaxy source is closed during the beta. The application binary is signed with our Apple Developer ID and notarized by Apple. You can confirm both from the terminal: `spctl --assess --verbose /Applications/Catallaxy.app` reports overall Gatekeeper acceptance, and `xcrun stapler validate /Applications/Catallaxy.app` confirms the notarization staple specifically. (`codesign --verify` checks the local signature only and does not contact Apple, so it is not by itself proof of notarization.)

---

## Third-party software

Catallaxy is built on substantial open-source work. A complete list of the libraries we use, their versions, and their license texts is at [`THIRD-PARTY-LICENSES.md`](THIRD-PARTY-LICENSES.md).

The biggest contributors include:

- **[Tauri](https://tauri.app)** (Apache 2.0 / MIT) — the application framework.
- **[Svelte](https://svelte.dev)** (MIT) — the UI runtime.
- **[Rust](https://www.rust-lang.org)** and the Rust ecosystem (various permissive licenses) — the desktop logic.
- **[Elixir](https://elixir-lang.org/)** and the Elixir ecosystem (various permissive licenses) — the backend logic.

We're grateful for all of it. None of these projects endorse Catallaxy; we're standing on their shoulders.

---

## Acknowledgments

Catallaxy exists because tools like **Claude Code**, **Codex CLI**, and **Gemini CLI** exist. Each of those is the product of people who chose to ship a CLI experience for AI-assisted coding, which is what makes the observer pattern possible at all.

This product also showcases a lot of my own personal adoration of many cultural icons from my formative years (looking at you, Arthur!). My use of these references is entirely for educational and entertainment purposes, and with love and gratitude. If the owners of any characters that resemble the ones I have in this app have a problem with that, just send me a note (the address is on `catallaxy.app`) and I'll remove them.

---

## Document version

This README is a living document. We update it with every meaningful change to the product or to these terms.

| Field | Value |
|------|-------|
| Document version | 0.3.0 (observer-round revisions: TOC fix, copy/notarization corrections, hook-token accuracy, network-surface completeness, ToS pointer, subscription-claim qualifier) |
| Last updated | 2026-04-27 |
| Catallaxy version at last update | 0.66.1 |
| Canonical location | `https://github.com/SiliconValleyPirate/Catallaxy/blob/main/README.md` |

If you're reading a copy of this document somewhere else, the canonical version on GitHub may be more recent.
