<div align="center">

<img src="MyKeyboardSwitcher/Assets.xcassets/AppIcon.appiconset/MyKeyboardSwitcher_512x512x32.png" width="128" alt="MyKeyboardSwitcher icon">

# MyKeyboardSwitcher

[![macOS](https://img.shields.io/badge/macOS-13.0+-blue.svg)](https://www.apple.com/macos)
[![Release](https://img.shields.io/github/v/release/alanbigulov/my-keyboard-Switcher-releases.svg)](https://github.com/alanbigulov/my-keyboard-Switcher-releases/releases)
[![License](https://img.shields.io/badge/License-MIT-brightgreen.svg)](LICENSE)

**Instant cycling through your favorite keyboard layouts — with on-screen feedback.**

Press **Right Shift** to cycle through a pair or trio of layouts you picked —
a HUD flashes the set right on screen. Press **Caps Lock** to change that
selection — palette, flags, checkboxes, mouse or keyboard.

</div>

## Why?

The native macOS input switcher cycles through *every* enabled layout —
including ones you rarely use — adds a noticeable delay, and shows nothing
when it switches. MyKeyboardSwitcher takes a different approach:

- You choose **2–3 favorite input sources** (layouts *and* IMEs like Korean, Japanese, Chinese)
- **Right Shift** cycles forward through just those — in ~10–70 ms
- A **floating HUD** shows the whole set and the layout you just landed on
- **Caps Lock** opens a palette where you re-pick the set in seconds
- Everything else stays reachable through the system input menu, as usual

## Features

- **Instant switching** — a `CGEventTap` consumes the trigger key and selects the next favorite via Text Input Source Services directly
- **Switch HUD** — with 3 favorites, every cycle flashes a floating row of the set centered on screen (~700 ms, active layout highlighted). Appears instantly — keyboard-initiated actions are never animated — and never steals focus
- **Favorites palette** — borderless panel centered on the active screen: flags and checkboxes on every row, `↑`/`↓` or hover to navigate, `Space`/`1–9` or a click toggles, `Enter`/**Apply** applies, `Esc`/**Cancel** cancels. Scrolls when the list is long
- **Add/remove languages in place** — the palette's "＋ Add/Remove Language…" button opens System Settings → Keyboard; changes land in the open palette and the menu within a second, no restart
- **Swappable triggers** — menu toggle exchanges the roles: Caps Lock cycles, Right Shift opens the palette. Persisted across restarts
- **IME-safe** — input methods (Korean `2-Set`, Japanese, Chinese…) are listed alongside layouts; every switch is verified ~100 ms later and retried with a mode-aware fallback, working around the known `TISSelectInputSource` CJKV flakiness
- **Two dedicated triggers** — Right Shift → `LANG1`, Caps Lock → `LANG2`, remapped through the system `UserKeyMapping` HID property via direct IOKit calls. Left Shift and every other key are untouched
- **Clean teardown** — both remaps are reverted synchronously on quit, permission loss, and launch, so your keys never stay remapped after the app exits
- **Live source sync** — debounced TIS notifications plus a cross-check against the `AppleEnabledInputSources` backing store keep the palette and menu in sync with System Settings, even where TIS's per-process snapshot lags
- **Native & lightweight** — background agent, no Dock icon, no helper processes, ~1 MB
- **Menu bar control** — status icon (⚠️ needs permissions / ⌨️ active), favorites checklist with flag icons, ● on the live layout, launch-at-login toggle

## Quick start

Download the latest build from [Releases](https://github.com/alanbigulov/my-keyboard-Switcher-releases/releases), copy it to `/Applications`, then:

```bash
xattr -cr /Applications/MyKeyboardSwitcher.app   # clear Gatekeeper quarantine
open /Applications/MyKeyboardSwitcher.app
```

1. Grant **Accessibility** permission when prompted (System Settings → Privacy & Security → Accessibility → enable `MyKeyboardSwitcher`)
2. Press **Caps Lock** (or use the menu bar icon) and check 2–3 favorites
3. Press **Right Shift** — done, it just cycles

> [!NOTE]
> Updating to a new build? The ad-hoc signature changes, so macOS revokes Accessibility. Toggle the permission off/on once in System Settings.

> [!IMPORTANT]
> Upgrading from ≤1.2.0? The app was renamed **CapsLockSwitcher → MyKeyboardSwitcher** and the bundle ID changed to `com.doasync.MyKeyboardSwitcher`. macOS treats it as a new app: grant Accessibility to the new entry, remove the old `CapsLockSwitcher.app`, and re-enable **Launch on Startup** if you used it.

## Usage

| Action | How |
|---|---|
| Cycle favorites forward | **Right Shift** (HUD shows the set when 3 are picked) |
| Open / close palette | **Caps Lock** |
| Navigate palette | `↑` / `↓` or hover |
| Toggle favorite | `Space`, `1`–`9`, or click the row |
| Apply / cancel | `Enter` / `Esc`, or the **Apply** / **Cancel** buttons |
| Add/remove a layout in macOS | palette → **＋ Add/Remove Language…** |
| Swap the two trigger roles | status icon → **Swap Trigger Keys** |
| Same palette, from the menu | status icon → **Edit Favorites…** |
| Enable at login | status icon → **Launch on Startup** |

Cycling is **forward only** (A → B → C → A). If the active layout isn't in
favorites, the first press jumps to favorite #1. Removing the current layout
from favorites keeps you on it until the next cycle.

## How it works

- `IOHIDEventSystemClientSetProperty` writes both trigger pairs (`RightShift→LANG1`, `CapsLock→LANG2`) into `UserKeyMapping` atomically — and writes identity mappings back on exit
- `CGEventTap` intercepts the remapped key codes, consumes them, and routes the two keys to cycle / palette — **Swap Trigger Keys** only flips that routing table, the remap itself is constant
- `TISSelectInputSource` performs the switch; a delayed check confirms the current source changed, with one retry and an input-mode fallback for IMEs
- Favorites persist as an ordered `favoriteSourceIDs` array in `UserDefaults` (migrates automatically from the v1.0 two-slot model)
- Source lists come from `TISCreateInputSourceList` cross-checked against the `AppleEnabledInputSources` plist — TIS's per-process snapshot can lag removals of IME input modes for minutes without posting a notification, so membership is verified against the backing store
- Debounced TIS change notifications plus a delayed recheck keep the menu — and an open palette — in sync when input sources are edited in System Settings

## Permissions explained

The app needs **Accessibility** to observe the two trigger keys system-wide via
the event tap. It checks only whether the pressed key is a trigger — it does
not log keystrokes and sends nothing anywhere. This is the standard permission
every input-monitoring tool requires.

## Troubleshooting

- **Not switching** — check the icon: ⚠️ means permissions were revoked (toggle them off/on); ⌨️ means active. Ensure 2+ favorites are checked and still enabled in System Settings → Keyboard → Input Sources.
- **Layout missing from the list** — add it via the palette's **＋ Add/Remove Language…** button or System Settings → Keyboard → Text Input → Input Sources. Both layouts and input methods appear.
- **Caps Lock doesn't capitalize** — while the app is Active it opens the palette instead. It works normally again the moment the app quits or loses permissions.
- **IME flickers / doesn't switch** — switch verification retries automatically; a beep means it still failed — please open an issue with the layout name.

## Acknowledgments

MyKeyboardSwitcher is a fork of [CapsLockSwitcher](https://github.com/doasync/CapsLockSwitcher) by [@doasync](https://github.com/doasync) — and was itself distributed as CapsLockSwitcher until v1.2.1. Big thanks to the original author — the core idea (remapping a trigger key through `UserKeyMapping` and selecting input sources directly via Text Input Source Services) and the codebase this project started from are his work.
