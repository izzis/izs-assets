# Tabby Parity Roadmap — ssh-client-android

> Working notes for the Android client, published for the public roadmap page.
> Source: direct audit of `tabby-*` (except `tabby-android`) vs the actual
> Android code. Per-item status verified with grep/read, not from
> ARCHITECTURE.md alone. Nothing here is promised — priorities change and
> some tiers may never ship.
> Last audited against izs-ssh-android `0fa26ff` (2026-09-22).

## Tier S — small, high impact, do first

1. ~~**`skipBanner` actually works** (Tabby `ssh.ts:442-446`).~~ ✅ `23a36fa`
2. ~~**`keepaliveCountMax` watchdog** (Tabby `ssh.ts:436-437`).~~ ✅ `4d33681`
3. **Per-profile `$TERM` option.** Desktop:
   `sshProfileSettings...pug:237-245`. Android hardcodes `xterm-256color`
   (`SshConnector.kt:405`, `TerminalScreen.kt:177`).
4. ~~**Username prompt when blank ("Ask every time")** (Tabby
   `ssh.ts:461-464`).~~ ✅ `4c8ad8c` + blank-vs-null fixup `a2be47b`
5. ~~**`behaviorOnSessionEnd`** (auto/keep/reconnect/close + press-any-key).~~
   ✅ `34a1bfa`
6. ~~**Auto-upload when `sync.auto` is on** (Tabby
   `configSync.service.ts:32-39`).~~ ✅ `0b9dda7`
7. **Terminal bell (audible/visual).** Desktop:
   `terminalSettingsTab...pug:174-219`, `bell.ogg`,
   `baseTerminalTab...ts:439-447`. Android: BEL is dropped
   (`TerminalEmulator.kt:355`, GROUND-state `\u0007` → `Unit`).
8. **Per-profile backspace mode** (Input tab). Desktop:
   `inputProcessingSettings...pug`, `inputProcessing.ts`. Android always DEL.
9. ~~**Clear terminal** (Tabby `hotkeys.ts:48-51`, `frontend.clear()`).~~
   ✅ `1046e28`

## Tier A — high value, medium effort

10. **Keyboard-interactive transport + challenge UI.** Desktop:
    `ssh.ts:253-291`, `keyboardInteractiveAuthPanel`. Android narrows to
    password (auth-selection block in `SshConnector.kt`) — 2FA/OTP servers
    can't connect.
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
14. **SFTP sheet parity (UI-grounded).** What Tabby actually shows —
    toolbar: Refresh · Filter · Create directory · Upload files · Upload
    folder (`sftpPanel.component.pug:20-38`); rows with icon · size ·
    modified date · mode string, tap file = download; right-click menu =
    Create directory / Download(-directory) / Delete
    (`sftpContextMenu.ts:24-69`; Electron adds Copy full path + Edit
    locally + watch-reupload). Android already: list + download/upload
    files (SAF) · filter · breadcrumb · auto-refresh after upload · size
    column · long-press copy full path (`42547d1`, `06ac71e`). Still open,
    all practical on Android: Create directory (mkdir + dialog), Delete
    (confirm + recursive dir), Upload folder (SAF tree + recursive
    mkdir/upload), Download folder (SAF tree + recursive download).
    Rename/chmod/mkdir/delete beyond the sheet stay terminal-side. Minor
    display gaps, low priority: no manual Refresh button, no
    modified-date/mode columns, no "Go up" row (breadcrumb covers it).
15. **Jump host chains** (`connectVia` exists in sshj). Desktop: pug `:60-64`.
    Android: throws (`SshConnector.kt:252-254`).
16. **HTTP CONNECT proxy.** Desktop: pug `:83-97`. Android: throws (same lines).
17. **Dynamic (SOCKS) forwarding.** Desktop:
    `sshPortForwardingConfig...pug:65-76`; the Android model already knows the
    `Dynamic` type (`TabbyModels.kt:94`) but connect refuses
    (`PortForwarding.kt:54-57`).
18. **Profile type `telnet`** (plain TCP + emulator + existing login scripts).
    Desktop: `tabby-telnet`. Android: only SSH can connect
    (`RawConfigStore.kt:479`).
19. **Ports dialog while the session is live** (add/remove forwards without
    reconnect). Desktop: `sshPortForwardingModal`, `sshTab...pug:32-38`.
    Android only at connect.

## Tier B — medium value, medium-to-large effort

20. **OpenSSH `~/.ssh/config` importer + static `ssh-profiles.yaml` + auto
    key discovery** (`tabby-electron/src/sshImporters.ts`). Android only
    imports full YAML (`ConfigFileScreen.kt`).
21. ~~**Duplicate profile + hide/blacklist** (Tabby `profilesSettingsTab`,
    `profiles.service.ts`).~~ ✅ `ef924aa` — remainder (per-type/per-group
    defaults, default-for-new-tabs) closed as not applicable: one
    connectable type, no local shell.
22. **Group editor** (name, parent, icon, color + per-group defaults).
    Desktop: `editProfileGroupModal...pug`. Android: rename + reparent +
    delete (members ungrouped, children to top level) shipped (`1ed65a3`);
    ungrouped profiles render in a synthetic top-level Ungrouped folder
    (desktop `profileTree` parity, UI-only, `08db905`); icon,
    color + per-group defaults still open.
23. **Vault secret browser** (show/rename/replace/export/delete). Desktop:
    `vaultSettingsTab...pug:30-69`. Android deliberately simple
    (`VaultSettingsScreen.kt:37-42`).
24. **SFTP "Edit locally" + watch-reupload.** Desktop (Electron-only
    provider): `tabby-electron/src/sftpContextMenu.ts` (temp file +
    `fs.watch` + re-upload on change). "Copy full path" from the same
    provider already shipped on Android (`06ac71e`: long-press); "Edit
    locally" still open (needs SAF temp + change observer).
25. **Export terminal buffer to file.** Desktop:
    `tabby-electron/src/terminalContextMenu.ts`. Android copy only.
26. **`%` progress detection.** Desktop: `baseTerminalTab...ts:522-533`,
    `detectProgress`. Android: none.
27. **Wide-char/CJK (Unicode11).** Desktop: `xtermFrontend.ts:205-206`.
    Android: wide char = 1 cell (`TerminalEmulator.kt:18-19`) → CJK text skews.
28. **Profile type `serial` (USB-OTG)** + baud/databits/parity/RTSCTS/
    XON-XOFF/slow-send. Desktop: `serialProfileSettings...pug`. Needs
    usb-serial lib + permission.
29. **Dynamic tab title + `disableDynamicTitle` + clear-on-connect.**
    Android title = profile name only; no dynamic title (`titleChange` = zero).
30. **Light color scheme + auto/dark/light mode + palette
    generate/harmonious.** Desktop: `colorSchemeSettingsTab...pug:1-74`.
    Android dark-only.
31. **Tab management**: rename/pin/duplicate tab, save-as-profile,
    close-other/left/right, reopen last tab. Desktop: `tabContextMenu.ts`
    (core+terminal). Android: close + close-all only.
32. **Notify-when-done / notify-on-activity per tab.** Desktop:
    `tabContextMenu.ts:173-243`. Android only underline + session-lost notif.
33. **Tab recovery (`recoverTabs`, default ON).** Desktop:
    `configDefaults.yaml:42`, `recoveryProvider.ts`,
    `connectableTerminalTab...ts:106-113`. Realistic on phones: reopen
    as *disconnected* tabs + reconnect button.
34. **Profile icon picker.** Desktop: `editProfileModal...pug:34-52` (FA).
    `ProfileEditScreen.kt` has no icon field; the value itself round-trips
    (`TabbyModels.icon`, `RawConfigStore`) and the home list renders a
    desktop-style icon tinted with the profile color
    (`ProfileListScreen.kt:1257`). Note: FA names can't render
    1:1 — needs a Material/emoji mapping.
35. **Config file editable + view defaults.** Desktop:
    `settingsTab...pug:141-176`. Android read-only + import (low priority).
36. **Render polish**: font weight/ligatures/fallback/linePadding/min-contrast/
    bold-bright/wordSeparators/scrollOnInput. Desktop:
    `appearanceSettingsTab...pug`, `xtermFrontend.ts:700-721`. Android:
    font+size+cursor only.

## Tier C — niche or heavy, later

37. **Zmodem rz/sz** (`features/zmodem.ts`, 347 lines + confirm dialog).
    Needs protocol port + SAF.
38. **Sixel/images** (`config sixel`, `ImageAddon`). Custom renderer with no
    image pipeline.
39. **Full mouse reporting** (clicks in tmux/vim). Already tracks
    `?1000/1002/1003/1006` but ignores them (`TerminalEmulator.kt:16-19`).
40. **Split panes + multifocus broadcast.** Small phone screen; low value
    except broadcast.
41. **Stream processing modes** (local-echo/readline/hex, hexdump, newline
    mapping) — relevant mostly once serial/telnet exist.
42. **Language settings/i18n.** Desktop dozens of locales
    (`settingsTab...pug:62-74`); Android English-only by design.
43. **On-device local shell** (`tabby-local`; repo has `termux-app/` but
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

## Effort map (remaining work only; pure-JVM + MINA tests)

- **XS (1 file, hours):** 3 per-profile `$TERM` · 8 backspace mode ·
  7 visual bell (+audible, 1 small file) · 25 export buffer to file ·
  26 `%` progress detection · 34 profile icon picker (UI only).
- **S (2–4 files, ~1 day):** 14 SFTP create-directory + delete ·
  24 SFTP edit-locally · 29 dynamic tab title · 30 light scheme + auto mode ·
  31 per-tab reconnect/zoom + rename/pin/duplicate + save-as-profile.
- **M (small subsystem, days + tests):** 10 keyboard-interactive ·
  11 search-in-buffer · 12 OSC 52/1337 + rememberCwd · 13 auto-sudo-password ·
  14 SFTP upload/download folder · 15 jump host · 16 HTTP proxy ·
  17 Dynamic forwarding · 18 telnet · 19 live Ports dialog ·
  20 OpenSSH/static importer · 22 group editor (icon/color/defaults) ·
  23 vault browser · 33 tab recovery (reopen disconnected) ·
  32 done/activity notifications · 35 editable config file ·
  36 render polish.
- **L (large, ~1–2 weeks):** 28 serial USB-OTG · 37 Zmodem · 39 full mouse
  reporting · 27 wide-char CJK · 42 i18n · 40 split panes + multifocus.
- **XL (own project / impossible):** 43 local shell + all of Tier F.

## Version map (semver; on-demand releases per tag, `RELEASE.md`)

- **1.2.x — Tier S, mostly shipped:** 1.2.1 = encrypted-storage switch
  (Tink) + keepalive watchdog (`4d33681`) + username prompt (`4c8ad8c`);
  1.2.2 = auto-upload (`0b9dda7`) + duplicate/hide (`ef924aa`) + group
  editor (`1ed65a3`).
- **1.3.0 (current, `19bebfd`) — Tier S remainder + SFTP parity slices:**
  `behaviorOnSessionEnd` (`34a1bfa`), clear terminal (`1046e28`),
  username blank-vs-null fixup (`a2be47b`), SFTP filter (`42547d1`) +
  copy-path (`06ac71e`), Ungrouped folder (`08db905`), clipboard parity
  (`182bd3f`, non-roadmap Tabby parity). Tier S still open after 1.3.0:
  per-profile `$TERM` (3), bell (7), backspace mode (8) — no new YAML schema
  for any of the three.
- **1.3.x patch (per item, anytime):** profile icon picker (34); export
  buffer (25) + progress (26).
- **1.4.0+ — Tier A batch (still fully open):** keyboard-interactive (10) ·
  search (11) · OSC/cwd (12) · auto-sudo (13) · SFTP create-directory/
  delete/upload/download folder (14 remainder) · jump host (15) ·
  HTTP proxy (16) · Dynamic forwarding (17) · telnet (18) · live Ports (19).
  Plus Tier B/C: importer (20) · group/vault (22–23) · edit-locally/
  buffer/progress (24–26) · CJK/serial/title/scheme/tab/notif/recovery/
  icon/config-file/render (27–36).

Practical rule: no new permissions, no YAML/sync schema changes, covered by
JVM unit tests → patch-worthy; otherwise minor.

## Done (shipped; tier list shows one-line cross-outs, full notes here)

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
      `ProfileFieldsTest`. Follow-up `a2be47b`: blank-vs-null parity with
      Tabby (explicit blank = ask every time, missing key = `root`
      default) + `ProfileFieldsTest`/`RawRoundTripTest` coverage.
- [x] **#6 Auto-upload when auto is on** — `0b9dda7`. Tick uploads
      locally-dirty configs (SHA-256 baseline stamped after every completed
      up/download/import); both-sides-dirty pauses with toast + Settings dot
      + resolve card instead of overwriting. Same commit also fixed the
      silent target no-save (detached-copy write-back). `SyncTargetTest` 4
      tests + `AutoSyncDecisionTest` 5 tests.
- [x] **Non-roadmap (side work, not Tabby parity)** — hold-to-repeat extra
      keys + half width (`1d12d50`), `#607d8b` swatch removal (amended in).
- [x] **#22 Group editor (partial)** — `1ed65a3`. Rename + reparent
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
- [x] **#14 (partial) SFTP filter** — `42547d1`. Tabby `sftpPanel` parity:
      hide/show filter box in the sheet handle zone (shared
      `CompactFilterField` in `AnchoredSheet.kt`), cleared on navigate,
      case-insensitive substring match, empty-match row text. Create-directory
      / delete / folder up/download still open.
- [x] **#24 (partial) SFTP "Copy full path"** — `06ac71e`. Long-press any
      row (file or folder, offline-safe) copies the full remote path via
      the platform clipboard + Toast; tap still navigates into folders
      (guarded: needs a live session). "Edit locally" + watch-reupload
      still open.
- [x] **#22 (follow-up) Ungrouped folder** — `08db905`. Desktop
      `profileTree` parity, UI-only: ungrouped profiles render in a
      synthetic Ungrouped folder pinned at the TOP (not a flat bottom
      list), with sticky header + expand/collapse + count, in normal and
      search views; not manageable (no pencil, `onManageGroup` guard);
      editor "No group" shows as Ungrouped (placeholder parity).
      Domain/sync/vault untouched.
- [x] **Non-roadmap (Tabby parity, unnumbered): clipboard parity** —
      `182bd3f` (4 synced `terminal.*` keys with delete-on-default,
      bracketed-paste `?2004` tracking + paste funnel with fold/replace/
      strip/trim + multiline warn; `ClipboardParityTest` 192 lines).
      Side work: faster encrypted-config saves via RAM caches
      (`d71b455`, no format change), launcher icon (`10afc22`), shared
      half/full sheet + 70% default (`42547d1`/`06ac71e` infra), docs now
      call the reference "Tabby" not "desktop" (`0fa26ff`).
