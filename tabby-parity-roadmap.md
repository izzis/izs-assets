# Tabby Desktop Parity Roadmap — ssh-client-android

> Working notes for the Android client, published for the public roadmap page.
> Source: direct audit of `tabby-*` (except `tabby-android`) vs the actual
> Android code. Per-item status verified with grep/read, not from
> ARCHITECTURE.md alone. Nothing here is promised — priorities change and
> some tiers may never ship.

## Tier S — small, high impact, do first

1. ~~**`skipBanner` actually works.** Toggle exists but says "(desktop only)"
   (`ProfileEditScreen.kt:1047`), the connector never reads it (grep
   `skipBanner` in `core/ssh` only hits the `SshConnector.kt:36` comment).
   Desktop: `tabby-ssh/src/session/ssh.ts:442-446`.~~ ✅ `23a36fa`
2. ~~**`keepaliveCountMax` watchdog.** Stored + default 10
   (`SshDefaults.kt:17`), acknowledged stored-only (`SshConnector.kt:271-273`).
   Desktop: `ssh.ts:436-437`.~~ ✅ `4d33681`
3. **Per-profile `$TERM` option.** Desktop:
   `sshProfileSettings...pug:237-245`. Android hardcodes `xterm-256color`
   (`SshConnector.kt:385`).
4. ~~**Username prompt when blank ("Ask every time").** Desktop:
   `ssh.ts:461-464` (PromptModal). Android defaults to `root`
   (`TabbyModels.kt:60`, `SshConnector.kt:253`).~~ ✅ `4c8ad8c`
5. ~~**`behaviorOnSessionEnd`** (auto/keep/reconnect/close + press-any-key).
   Desktop: `editProfileModal...pug:71-81`,
   `connectableTerminalTab...ts:67-94`.~~ ✅ `34a1bfa`
6. ~~**Auto-upload when `sync.auto` is on.** Desktop uploads on every config
   change (`configSync.service.ts:32-39`). Android only auto-*downloads*
   (`SyncRepository.kt:564-588`); upload is always manual
   (`ConfigSyncScreen.kt:114`).~~ ✅ `0b9dda7`
7. **Terminal bell (audible/visual).** Desktop:
   `terminalSettingsTab...pug:174-219`, `bell.ogg`,
   `baseTerminalTab...ts:439-447`. Android: BEL is dropped
   (`TerminalEmulator.kt:340`).
8. **Per-profile backspace mode** (Input tab). Desktop:
   `inputProcessingSettings...pug`, `inputProcessing.ts`. Android always DEL.
9. ~~**Clear terminal.** Desktop: hotkey/action `clear`
    (`tabby-terminal/src/hotkeys.ts:48-51`, `frontend.clear()`).~~ ✅ `1046e28`

## Tier A — high value, medium effort

10. **Keyboard-interactive transport + challenge UI.** Desktop:
    `ssh.ts:253-291`, `keyboardInteractiveAuthPanel`. Android narrows to
    password (`SshConnector.kt:302-310`) — 2FA/OTP servers can't connect.
11. **Search-in-buffer + persistent options** (regex/whole-word/case,
    `searchRegexAlwaysEnabled`). Desktop: `searchPanel.component.ts`,
    `config.ts:53-58`. Android: none.
12. **OSC 52 (remote copy) + OSC 1337 cwd + `rememberCwd` + "Copy current
    path".** Desktop: `oscProcessing.ts`,
    `sshProfileSettings...pug:231-235`, `tabContextMenu.ts:68-73`. Android:
    OSC swallowed, no cwd concept (grep `cwd` outside SFTP = zero).
13. **Auto-sudo-password.** `tabby-auto-sudo-password/src/decorator.ts`
    (19 languages + sudo-rs, hint + needs Enter — design already safe). Port
    `AutoSudoPasswordMiddleware` to the reader pump.
14. **SFTP: mkdir + delete + filter + download/upload folder.** Desktop:
    `sftpPanel...ts:197-315`, `sftpContextMenu.ts:24-76`. Android explicitly
    out-of-scope (`SftpTransfer.kt:21`), sheet has no filter/folder.
15. **Jump host chains** (`connectVia` exists in sshj). Desktop: pug `:60-64`.
    Android: throws `SshConnector.kt:246-250`.
16. **HTTP CONNECT proxy.** Desktop: pug `:83-97`. Android: throws (same lines).
17. **Dynamic (SOCKS) forwarding.** Desktop:
    `sshPortForwardingConfig...pug:65-76`; the Android model already knows the
    `Dynamic` type (`TabbyModels.kt:94`) but connect refuses
    (`PortForwarding.kt:54-57`).
18. **Profile type `telnet`** (plain TCP + emulator + existing login scripts).
    Desktop: `tabby-telnet`. Android: only SSH can connect
    (`RawConfigStore.kt:499`).
19. **Ports dialog while the session is live** (add/remove forwards without
    reconnect). Desktop: `sshPortForwardingModal`, `sshTab...pug:32-38`.
    Android only at connect.

## Tier B — medium value, medium-to-large effort

20. **OpenSSH `~/.ssh/config` importer + static `ssh-profiles.yaml` + auto
    key discovery** (`tabby-electron/src/sshImporters.ts`). Android only
    imports full YAML (`ConfigFileScreen.kt`).
21. ~~**Duplicate profile, hide/blacklist, per-type & per-group defaults,
    default-profile-for-new-tabs.** Desktop: `profilesSettingsTab...pug`,
    `profiles.service.ts:379-414,580-585`. Android: duplicate (editor
    copy-mode) + hide (`profileBlacklist`, synced) shipped; the remainder
    is dropped (single connectable type, no local shell).~~ ✅ `ef924aa`
22. **Group editor** (name, parent, icon, color + per-group defaults).
    Desktop: `editProfileGroupModal...pug`. Android: rename + reparent +
    delete (members ungrouped, children to top level) shipped; icon,
    color + per-group defaults still open.
23. **Vault secret browser** (show/rename/replace/export/delete). Desktop:
    `vaultSettingsTab...pug:30-69`. Android deliberately simple
    (`VaultSettingsScreen.kt:37-42`).
24. **SFTP rename + chmod** (`sftp.ts:99-108`). Android: none.
25. **SFTP "Copy full path" + "Edit locally" + watch-reupload.**
    Desktop: `tabby-electron/src/sftpContextMenu.ts`.
26. **Export terminal buffer to file.** Desktop:
    `tabby-electron/src/terminalContextMenu.ts`. Android copy only.
27. **`%` progress detection.** Desktop: `baseTerminalTab...ts:522-533`,
    `detectProgress`. Android: none.
28. **Wide-char/CJK (Unicode11).** Desktop: `xtermFrontend.ts:205-206`.
    Android: wide char = 1 cell (`TerminalEmulator.kt:18-19`) → CJK text skews.
29. **Profile type `serial` (USB-OTG)** + baud/databits/parity/RTSCTS/
    XON-XOFF/slow-send. Desktop: `serialProfileSettings...pug`. Needs
    usb-serial lib + permission.
30. **Dynamic tab title + `disableDynamicTitle` + clear-on-connect.**
    Android title = profile name only (`TerminalScreen.kt:272-273`); no
    dynamic title (`titleChange` = zero).
31. **Light color scheme + auto/dark/light mode + palette
    generate/harmonious.** Desktop: `colorSchemeSettingsTab...pug:1-74`.
    Android dark-only (`ColorScheme.kt:11-12`).
32. **Tab management**: rename/pin/duplicate tab, save-as-profile,
    close-other/left/right, reopen last tab. Desktop: `tabContextMenu.ts`
    (core+terminal). Android: close + close-all only.
33. **Notify-when-done / notify-on-activity per tab.** Desktop:
    `tabContextMenu.ts:173-243`. Android only underline + session-lost notif.
34. **Tab recovery (`recoverTabs`, default ON).** Desktop:
    `configDefaults.yaml:42`, `recoveryProvider.ts`,
    `connectableTerminalTab...ts:106-113`. Realistic on phones: reopen
    as *disconnected* tabs + reconnect button.
35. **Profile icon picker.** Desktop: `editProfileModal...pug:34-52` (FA).
    `ProfileEditScreen.kt` has no icon field. Note: FA names can't render
    1:1 — needs a Material/emoji mapping.
36. **Config file editable + view defaults.** Desktop:
    `settingsTab...pug:141-176`. Android read-only + import (low priority).
37. **Render polish**: font weight/ligatures/fallback/linePadding/min-contrast/
    bold-bright/wordSeparators/scrollOnInput. Desktop:
    `appearanceSettingsTab...pug`, `xtermFrontend.ts:700-721`. Android:
    font+size+cursor only.

## Tier C — niche or heavy, later

38. **Zmodem rz/sz** (`features/zmodem.ts`, 347 lines + confirm dialog).
    Needs protocol port + SAF.
39. **Sixel/images** (`config sixel`, `ImageAddon`). Custom renderer with no
    image pipeline.
40. **Full mouse reporting** (clicks in tmux/vim). Already tracks
    `?1000/1002/1003/1006` but ignores them
    (`TerminalEmulator.kt:16-19,562-570`).
41. **Split panes + multifocus broadcast.** Small phone screen; low value
    except broadcast.
42. **Stream processing modes** (local-echo/readline/hex, hexdump, newline
    mapping) — relevant mostly once serial/telnet exist.
43. **Language settings/i18n.** Desktop dozens of locales
    (`settingsTab...pug:62-74`); Android English-only by design.
44. **On-device local shell** (`tabby-local`; repo has `termux-app/` but
    unused). Needs local PTY + binary — a project of its own.

## Tier F — impossible / irrelevant on Android

- `proxyCommand` (needs helper binary `ssh -W`), X11 forwarding (no X
  server), agentForward/auth-agent/Pageant/pipe (no ssh-agent),
  WinSCP launch (Windows-only), plugin manager (no plugin runtime),
  physical-editor hotkeys, chrome window (vibrancy/opacity/frame/docking/
  tray/touchbar/shell-integration/ConPTY/WSL/UAC-admin/COMSPEC/GPU-toggle),
  auto-update/DevTools, Hyper themes, debug state, `preventAccidentalTabClosure`
  (web-only), built-in local/WSL profiles, drag-drop upload, CLI args.

## Audit coverage (all read)

- `tabby-terminal`: config defaults, terminal/appearance/color-scheme/input/
  stream/login-scripts/search pug+ts, xtermFrontend, base/connectable tab,
  tabContextMenu, zmodem, osc/input/login/debug middleware, hotkeys, settings.
- `tabby-ssh`: profile settings pug, ssh/sftp settings tab, sftp panel+context,
  KI panel, `session/ssh.ts`, `session/sftp.ts`, port-forward modal,
  recoveryProvider, hotkeys, profiles, settings registration.
- `tabby-settings`: window/profiles/edit-profile/edit-group/configSync/vault/
  hotkeys/settings-tab/plugins.
- `tabby-core`: root configDefaults, hotkeys, tabContextMenu, commands.
- `tabby-local` / `tabby-serial` / `tabby-telnet`, `tabby-linkifier`,
  `tabby-auto-sudo-password`, `tabby-electron` (context menus, sshImporters,
  updater/dock/touchbar), `tabby-web`, `tabby-uac` (C++ Windows-only),
  `tabby-community-color-schemes` (already parity: 102 built-in).

## Effort map (repo-pattern estimates: pure-JVM + MINA tests)

- **XS (1 file, hours):** 1 skipBanner works · 3 per-profile `$TERM` ·
  8 backspace mode · 9 clear terminal · 7 visual bell (+audible, 1 small file) ·
  4 username prompt · 26 export buffer to file · 27 `%` progress detection ·
  35 profile icon picker (UI only).
- **S (2–4 files, ~1 day):** 2 keepaliveCountMax watchdog · 5
  behaviorOnSessionEnd · 6 auto-upload when auto is on · 14 SFTP
  rename/chmod/delete/mkdir · 25 SFTP copy-full-path + edit-locally ·
  30 dynamic tab title · 31 light scheme + auto mode · 32 reconnect/zoom
  explicit per tab.
- **M (small subsystem, days + tests):** 10 keyboard-interactive ·
  11 search-in-buffer · 12 OSC 52/1337 + rememberCwd · 13 auto-sudo-password ·
  14 SFTP folder up/download · 15 jump host · 16 HTTP proxy · 17 Dynamic
  forwarding · 18 telnet · 19 live Ports dialog · 20 OpenSSH/static importer ·
  21 profile duplicate/hide/defaults · 22 group editor · 23 vault browser ·
  34 tab recovery (reopen disconnected) · 32 rename/pin/duplicate tab +
  save-as-profile · 33 done/activity notifications · 36 editable config file ·
  37 render polish.
- **L (large, ~1–2 weeks):** 29 serial USB-OTG · 38 Zmodem · 40 full mouse
  reporting · 28 wide-char CJK · 43 i18n · 41 split panes + multifocus.
- **XL (own project / impossible):** 44 local shell + all of Tier F.

## Version map (semver; on-demand releases per tag, `RELEASE.md`)

- **1.2.0 — Tier S batch (1–9):** skipBanner · keepaliveCountMax · `$TERM` ·
  username prompt · behaviorOnSessionEnd · auto-upload · bell · backspace mode ·
  clear terminal. No new YAML schema.
- **1.2.x patch (per item, anytime):** 1.2.1 = SFTP rename/chmod (24);
  1.2.2 = profile icon picker (35); 1.2.3 = export buffer (26) + progress (27).
- **1.3.0 — Tier A batch:** auto-upload (6) · keyboard-interactive (10) ·
  search (11) · OSC/cwd (12) · auto-sudo (13) · SFTP ops (14) · jump host (15) ·
  HTTP proxy (16) · Dynamic forwarding (17) · telnet (18) · live Ports (19).
- **1.4.0+ — Tier B/C:** importer (20) · profile/group/vault (21–23) ·
  advanced SFTP (24–25) · buffer/progress (26–27) · CJK/serial/title/scheme/tab/
  notif/recovery/icon/config-file/render (28–37).

Practical rule: no new permissions, no YAML/sync schema changes, covered by
JVM unit tests → patch-worthy; otherwise minor.

## Done (tier list above untouched; cross-outs logged here)

- [x] **#1 `skipBanner` actually works** — `23a36fa`. Protocol banner
      (`UserAuth.getBanner()`, e.g. `/etc/issue.net`) shows as the first
      service line unless the profile skips it. motd (`Welcome to Zorin`
      etc.) deliberately NOT touched — post-login shell bytes, same as
      Tabby. "(desktop only)" label removed.
- [x] **#2 `keepaliveCountMax` watchdog** — `4d33681`. Bonus find during
      implementation: the post-construction provider assignment never took
      effect (production ran on HEARTBEAT, not IGNORE as the comment
      claimed); provider now fixed pre-construction + countMax wired to
      `KeepAliveRunner`. `KeepaliveTest` 2 tests.
- [x] **#4 Username prompt when blank** — `4c8ad8c` (including fixup: the
      answer is kept for the tab lifetime, not one-shot — without it the
      prompt appeared 3x per login: username → trust → username →
      password → username). Editor: raw-aware init (explicit blank shows
      empty + "Ask every time", missing key still shows `root`), blank
      may be saved, password row hides when user is empty, old secrets
      orphaned (auto-migration removed — Tabby orphans too). Cancel →
      "Username required" error card (deliberate divergence from Tabby's
      null-propagation). `UsernamePromptTest` 5 tests + blank parse in
      `ProfileFieldsTest`.
- [x] **#6 Auto-upload when auto is on** — `0b9dda7`. Tick uploads
      locally-dirty configs (SHA-256 baseline stamped after every completed
      up/download/import); both-sides-dirty pauses with toast + Settings dot
      + resolve card instead of overwriting. Same commit also fixed the
      silent target no-save (detached-copy write-back). `SyncTargetTest` 4
      tests + `AutoSyncDecisionTest` 5 tests.
- Non-roadmap (side work, not Tabby parity): hold-to-repeat extra keys +
  Half width (`1d12d50`), `#607d8b` swatch removal (amended in).
- **#22 Group editor (partial)** — `1ed65a3`. Rename + reparent
  (cycle-guarded) + delete (`deleteProfiles:false`: members ungrouped,
  children rise to top level) via the shared plaintext/encrypted
  `updateTerminalSection` path. `GroupEditTest` 6 tests. Icon, color +
  per-group defaults still open.
- [x] **#21 Duplicate + hide** — `ef924aa`. Duplicate opens the editor
      pre-filled (`copy:<id>` mode, Save creates, Back cancels); Hide
      writes the synced `profileBlacklist` (desktop honors it natively),
      filtered from the home tree + new-tab picker, with a collapsed
      Hidden section as the unhide path. `ProfileBlacklistTest` 2 tests.
      Remainder (per-type/per-group defaults, default-for-new-tabs)
      closed as not important: one connectable type needs no type
      defaults, and with no local shell the picker stays the new-tab flow.
- [x] **#9 Clear terminal** — `1046e28`. `TerminalEmulator.clear()`
      (grid + scrollback emptied, cursor home, session alive) behind a
      "Clear" item in the shared ⋮ menu (all five anchors). SSH-only like
      desktop's non-Windows path: no Ctrl+L sent, the shell redraws nothing.
- [x] **#5 `behaviorOnSessionEnd`** — `34a1bfa`. Model + synced YAML
      (unknown values fall back to `auto`, auto omitted on write) + editor
      dropdown (Advanced > Session) + `recentInputs` (desktop-capped last
      32 chars) + four-way branch on session end: close/explicit-auto
      destroy the tab, reconnect redials at once, keep/auto-keep keep the
      failed card plus the "Press any key to reconnect" line (first keypress
      reconnects; input stays enabled while the offer stands). Deliberate
      divergences: keep still shows the failed card (the auto-connect effect
      needs `failed != null`, otherwise keep would self-connect), auto keeps
      the one-shot self-heal (Android kills connections on app switch —
      desktop-exact silence would strand every backgrounded tab), no
      `isDisconnectedByHand` (`ShellSession.close` suppresses `onDied`, so
      the hook never sees manual disconnects).
- [x] **Keep-alive interval (pre-existing, not a numbered item).** Model +
      YAML + editor (`keepaliveInterval` ms / `keepaliveCountMax`, defaults
      5000/10 = desktop `profiles.ts:23-24`) + SSHJ `KeepAliveRunner`
      (`SshConnector.kt:267-280,353-360`, wired by `4d33681` together with
      the #2 watchdog + `KeepaliveTest`). Wiring verified against a live
      in-test MINA server; heartbeat-over-idle-NAT and background/Doze
      survival still need on-device manual testing. Micro-divergence:
      `1500ms` truncates to 1s here vs `Math.round` to 2s on desktop;
      sub-second values coerce to 1s on both sides. Background survival is
      covered separately by `SessionService` (foreground service + opt-in
      wake lock); OEM task killers remain outside any app's control.
