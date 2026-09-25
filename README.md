# Catallaxy

> Distributed AI observation for your coding sessions.

**Status:** Closed beta · invite-only · macOS

Catallaxy is a macOS app that runs a panel of passive AI observers alongside your Claude Code, Codex, Cowork, or ChatGPT Work sessions. The observers read the transcript as it grows and speak up when something is worth saying; they never touch the work. Everything else about the product lives on the site: what it is and who it's for at [catallaxy.app](https://catallaxy.app), and the questions people ask, from what you need to run it to what leaves your machine, on the [FAQ](https://catallaxy.app/faq).

**This repository is the issue tracker.** It does not contain the source code, which is closed during the beta. What it holds: this README, the bug-report template, and the third-party license inventory.

---

## Contents

- [Reporting bugs](#reporting-bugs)
- [Installing](#installing)
- [Uninstalling](#uninstalling)
- [Where Catallaxy stores its data](#where-catallaxy-stores-its-data)
- [The hooks Catallaxy installs](#the-hooks-catallaxy-installs)
- [License and terms](#license-and-terms)
- [Third-party software](#third-party-software)
- [Acknowledgments](#acknowledgments)

---

## Reporting bugs

Click the bug button in Catallaxy's left-hand sidebar, or use **Help → Report Issue** in the menu bar. Either opens your browser at this repository's new-issue form with the app version filled in. You write the rest: what happened, the steps to reproduce, your macOS version, and which observer model was involved.

The button sends nothing on its own. Catallaxy never collects transcripts, observer responses, personas, lenses, configuration, or tokens through this path; whatever you paste into the issue is what arrives. The form asks for your log file (**Session → Reveal Logs**), which records app events and the paths of the projects Catallaxy watched. **Issues in this repository are public.** Look through anything before you attach it, and leave out whatever you wouldn't post publicly.

For anything that isn't a bug, use the [contact page](https://catallaxy.app/contact).

---

## Installing

You need an Apple Silicon Mac on macOS 14 or newer.

1. Once your request is approved, you receive an email with a link to your license key. The page asks you to confirm before it shows the key, so a mail scanner or a link preview can't reveal it. The link works for 24 hours; the key it shows is yours for good, so save it somewhere durable.
2. The same page offers the installer. Download the `.dmg`, open it, drag **Catallaxy** to `/Applications`, and eject the image.
3. Launch Catallaxy. The first launch asks for your license key, then walks you through connecting your session tool and summoning a first council.

The build is signed with an Apple Developer ID and notarized by Apple, so Gatekeeper should not need any workaround. If it does, please file a bug. You can confirm both from the terminal: `spctl --assess --verbose /Applications/Catallaxy.app` reports Gatekeeper's verdict, and `xcrun stapler validate /Applications/Catallaxy.app` confirms the notarization staple. (`codesign --verify` checks the local signature only and does not contact Apple, so it is not by itself proof of notarization.)

Lost your link? [Ask for a new one](https://catallaxy.app/retrieve).

---

## Uninstalling

1. In Catallaxy, choose **Session → Prepare for Uninstall…**. It removes Catallaxy's entries from your Claude Code settings. (The same menu item restores them if you change your mind before step 3.)
2. Quit the app and move `Catallaxy.app` from `/Applications` to the Trash.
3. Remove the data directory: `rm -rf ~/.catallaxy/`
4. Optional leftovers, each harmless on its own: the backup Catallaxy took before its first change to your Claude Code settings, at `~/.claude/settings.json.catallaxy-backup`; the observer agent definition it wrote for Antigravity, at `~/.gemini/config/agents/catallaxy-observer/`; any `observer-protocol.md` you accepted into a project (a plain text file, yours to keep or delete); and the observers' own conversation histories, which live wherever each vendor's tool keeps its sessions (Claude Code's under `~/.claude/projects/`, Codex's under `~/.codex/sessions/`, Antigravity's under `~/.gemini/antigravity-cli/`).

---

## Where Catallaxy stores its data

Everything Catallaxy itself persists is under `~/.catallaxy/`:

| Path | What it is |
|------|-----------|
| `config.toml` | Operational config: bindings, rooms, summon configurations, app settings, your license key, and the hook token. Hand-editable for troubleshooting. |
| `.hook-token` | A copy of the hook token, read by the relay script below. |
| `bin/hook-relay.sh` | The script Claude Code's hooks run. It holds the token so your Claude Code settings don't. |
| `personas/<name>/` | One directory per persona: `persona.toml`, images, version history, and an Illustrator's showcase gallery. |
| `lenses/<name>/` | One directory per lens: `lens.toml`, `prompt.md`, an optional icon, and any payload files. |
| `packages/<name>/` | Package drafts you author and receipts for packages you install. |
| `holding-pen/` | Sessions Catallaxy noticed but you haven't placed in a room yet. |
| `agy-sandbox/<observer>/` | Per-observer state for Gemini observers: the dispatch logs and the settings tree the observer runs under. |
| `usage-tally-<fingerprint>.json` | The usage tally: counts only, every counter listed in the Terms. |
| `freehold-state.json` | UI-only state, such as dismissed Freehold notifications. Deletable without breaking the app. |
| `dispatch-trace.jsonl` | Optional debug trace, created only if `dispatch_trace = true` in `config.toml`. Size-capped. |

`config.toml` and `.hook-token` carry your installation's secrets: the license key and the hook token. They never leave your machine.

---

## The hooks Catallaxy installs

Catallaxy learns about your Claude Code sessions, when one starts, when you send a prompt, when a turn finishes, through Claude Code's hook system. The entries it installs are command hooks that run `~/.catallaxy/bin/hook-relay.sh`, which forwards each event to a local-only server on `127.0.0.1:19191`, authenticated with your installation's token. The token lives in the relay script and under `~/.catallaxy/`, never in your Claude Code settings file.

Hooks are installed at the **user level**, in `~/.claude/settings.json`, never per project. This is a deliberate security choice: project-level hooks committed to git have been the vector for reported Claude Code CVEs (`CVE-2025-59536`, `CVE-2026-21852`), and Catallaxy wants no part of that surface. The installer is append-only and keeps your existing hooks; it writes a backup to `~/.claude/settings.json.catallaxy-backup` before its first change, recognizes its own entries so it never duplicates them, and sweeps entries left by earlier releases. You can turn the connection off and on in Settings ("Connected to Claude Code"), or remove the entries with **Session → Prepare for Uninstall…**. While the connection is on, Catallaxy re-adds its entries at launch if they've gone missing.

Cowork, Codex, and ChatGPT Work sessions need no hooks: Catallaxy reads the session files those tools write on your Mac.

The first time a room is tied to a project folder, Catallaxy offers to add `observer-protocol.md` to it. It's a short plain-text file that tells your coding agent how to handle pasted observer alerts, and it ends by asking the agent to emit a marker when a turn is done, which is how Catallaxy times its reviews. Decline and nothing breaks; Catallaxy falls back to an idle timer.

---

## License and terms

This section is the closed-beta license between Catallaxy ("we", "us", "I") and you ("the licensee"). It's written in plain language to be readable, but the terms are operative — by installing or running Catallaxy, you agree to them.

The full [Terms of Service](https://catallaxy.app/terms) that you accepted at download time are the binding version. This README provides a plain-language summary for your reference; where the two diverge, the Terms of Service govern.

### License grant

We grant you a personal, non-exclusive, non-transferable, revocable license to install and run Catallaxy on devices you own or control, up to the per-device limit associated with your license key, for the duration of the closed beta. The license is yours, not transferable to anyone else.

### What you can do

- Run Catallaxy for any lawful purpose, personal or commercial.
- Author personas, lenses, and packages, and keep them locally on your machine.
- Publish content you authored to the Freehold marketplace, subject to the [Freehold marketplace terms](#freehold-marketplace) below.
- Discuss your experience with Catallaxy publicly.

### What you can't do

- **Redistribute the application binary.** The `.dmg`, the `.app`, the underlying executables — none of these may be passed to anyone else, posted publicly, mirrored, repackaged, or otherwise distributed. Each license key is bound to a specific person and a specific small number of devices.
- **Share license keys or access links.** Both are credentials. We can revoke them if they leak.
- **Reverse-engineer, decompile, or disassemble** the application beyond what local laws explicitly permit despite this restriction.
- **Sublicense, sell, rent, or lease** Catallaxy or any portion of it.
- **Use the Catallaxy name or logo** to imply endorsement of, or affiliation with, any product, service, or content that we have not authorized.

### Beta software — no warranty

Catallaxy is provided **AS IS** and **AS AVAILABLE**, without warranty of any kind, express or implied, including (but not limited to) warranties of merchantability, fitness for a particular purpose, non-infringement, accuracy, or availability. Beta software contains defects we have not yet found. You accept that risk by using it.

### Limitation of liability

To the maximum extent permitted by law, our total cumulative liability arising from or relating to your use of Catallaxy is limited to the greater of (a) the amount you paid us for the software (during closed beta, this is zero) or (b) USD $50. We are not liable for indirect, incidental, special, consequential, or punitive damages, including but not limited to lost profits, lost data, or business interruption.

This limitation applies regardless of the legal theory and even if we have been advised of the possibility of such damages.

### Privacy

Catallaxy is local-first by design: your transcripts, observer responses, and code never reach any infrastructure we operate. What the app sends to `catallaxy.app` and what the backend keeps is enumerated in the [Terms](https://catallaxy.app/terms), section 6, and answered in plain language on the [FAQ](https://catallaxy.app/faq#q-servers). If you find a gap between those pages and what the app does, [file a bug](#reporting-bugs); we treat privacy claims as load-bearing.

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

We may update these terms during the closed beta. The effective date on the [Terms of Service](https://catallaxy.app/terms) is authoritative. Material changes will be communicated to invited testers directly.

### Choice of law

This license is governed by the laws of the State of Texas, USA, without regard to conflict-of-laws principles. Any dispute arising from or relating to this license shall be brought exclusively in the state or federal courts located in Texas.

### Source code

The Catallaxy source is closed during the beta; this repository holds no code. The application binary is signed with our Apple Developer ID and notarized by Apple (how to check: [Installing](#installing)).

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

Catallaxy exists because tools like **Claude Code**, **Codex**, and **Antigravity** exist. Each of those is the product of people who chose to ship a command-line experience for AI-assisted work, which is what makes the observer pattern possible at all.

This product also showcases a lot of my own personal adoration of many cultural icons from my formative years (looking at you, Arthur!). My use of these references is entirely for educational and entertainment purposes, and with love and gratitude. If the owners of any characters that resemble the ones I have in this app have a problem with that, just send me a note through the [contact page](https://catallaxy.app/contact) and I'll remove them.
