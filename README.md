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
- **Switch HUD** — with 2+ favorites, every cycle flashes a floating row of the set centered on screen (~700 ms, active layout highlighted). Appears instantly — keyboard-initiated actions are never animated — and never steals focus
- **Favorites palette** — borderless panel centered on the active screen: flags and checkboxes on every row, `↑`/`↓` or hover to navigate, `Space`/`1–9` or a click toggles, `Enter`, **Apply** or the palette key applies, `Esc`/**Cancel** cancels. Scrolls when the list is long
- **Add/remove sources in place** — the palette's *All Input Sources* section lists everything installed: `＋` enables a source, `−` disables it — the current source is switched away first and the last usable one can't be removed. A `−` on a favorites row disables that source outright, dropping it from the cycle. Changes land in the palette and menu within a second, no restart. The section collapses via its `▸` header and remembers the state. IME modes macOS only manages in System Settings stay dimmed; the footer's *Open Keyboard Settings…* button covers them
- **Swappable triggers** — menu toggle exchanges the roles: Caps Lock cycles, Right Shift opens the palette. Persisted across restarts
- **IME-safe** — input methods (Korean `2-Set`, Japanese, Chinese…) are listed alongside layouts; every switch is verified ~100 ms later and retried with a mode-aware fallback, working around the known `TISSelectInputSource` CJKV flakiness
- **Two dedicated triggers** — Right Shift → `LANG1`, Caps Lock → `LANG2`, remapped through the system `UserKeyMapping` HID property via direct IOKit calls as soon as the app has Accessibility. Until 2 favorites are picked, both keys open the palette. Left Shift and every other key are untouched, except the かな/英数 keys of JIS keyboards (see Troubleshooting)
- **Clean teardown** — the app adds and removes only its own two pairs in `UserKeyMapping`, so key mappings you set with `hidutil` stay in place. If you had mapped Right Shift or Caps Lock yourself, that mapping comes back when the app quits normally, unless another program changed the key mapping while the app ran. The remap is removed on quit (from the menu, `kill` or `killall`), when permissions are lost, and at launch
- **Live source sync** — debounced TIS notifications plus a cross-check against the `AppleEnabledInputSources` backing store keep the palette and menu in sync with System Settings, even where TIS's per-process snapshot lags
- **Native & lightweight** — universal app for Apple Silicon and Intel Macs, background agent, no Dock icon, no helper processes, ~2 MB
- **Menu bar control** — status icon (⚠️ needs permissions / ⚙️ fewer than 2 usable favorites, both trigger keys open the palette / ⌨️ active), favorites checklist with flag icons, ● on the live layout, launch-at-login toggle

## Quick start

Download the latest build from [Releases](https://github.com/alanbigulov/my-keyboard-Switcher-releases/releases), copy it to `/Applications`, then:

```bash
xattr -cr /Applications/MyKeyboardSwitcher.app   # clear Gatekeeper quarantine
open /Applications/MyKeyboardSwitcher.app
```

1. Grant **Accessibility** permission when prompted (System Settings → Privacy & Security → Accessibility → enable `MyKeyboardSwitcher`)
2. Press **Caps Lock** or **Right Shift** to open the palette (or use the menu bar icon), check 2–3 favorites and press `Enter`
3. Press **Right Shift** — done, it just cycles

> [!NOTE]
> Updating from 1.2.8 or earlier? macOS asks once whether to open a version that "differs from previously opened versions". Choose **Open Anyway**, then toggle Accessibility off and on once in System Settings. Later updates keep the permission.

> [!IMPORTANT]
> Upgrading from ≤1.2.0? The app was renamed **CapsLockSwitcher → MyKeyboardSwitcher** and the bundle ID changed to `com.doasync.MyKeyboardSwitcher`. macOS treats it as a new app: grant Accessibility to the new entry, remove the old `CapsLockSwitcher.app`, and re-enable **Launch on Startup** if you used it.

## Usage

| Action | How |
|---|---|
| Cycle favorites forward | **Right Shift** (HUD shows the set when 2+ are picked) |
| Open / close palette | **Caps Lock** (closing applies, like `Enter`); before 2 favorites are picked, **Right Shift** too |
| Navigate palette | `↑` / `↓` or hover |
| Toggle favorite | `Space`, `1`–`9`, or click the row |
| Enable/disable an input source | `＋` / `−` (or `Space`/click) in *All Input Sources* |
| Disable a source from favorites | `−` on its row in the favorites section |
| Collapse/expand *All Input Sources* | click the `▾`/`▸` section header |
| Apply / cancel | `Enter` or the palette key / `Esc`, or the **Apply** / **Cancel** buttons |
| Manage sources the app can't (IME modes) | palette → **Open Keyboard Settings…** |
| Swap the two trigger roles | status icon → **Swap Trigger Keys** |
| Same palette, from the menu | status icon → **Edit Favorites…** |
| Enable at login | status icon → **Launch on Startup** |

Cycling is **forward only** (A → B → C → A). If the active layout isn't in
favorites, the first press jumps to favorite #1. Removing the current layout
from favorites keeps you on it until the next cycle.

## How it works

- `IOHIDEventSystemClientCopyProperty` reads `UserKeyMapping`, and `IOHIDEventSystemClientSetProperty` writes it back with both trigger pairs (`RightShift→LANG1`, `CapsLock→LANG2`) added and every other pair kept. On exit the app removes its pairs and restores any pair of yours for those two keys that it replaced
- While the app has Accessibility it reads `UserKeyMapping` back every 3 seconds and whenever its menu or palette opens. If another program replaced the array, the app puts its two pairs back
- `CGEventTap` intercepts the remapped key codes, consumes them, and routes the two keys to cycle / palette. Until 2 favorites are picked, both go to the palette. **Swap Trigger Keys** only flips the routing after that, the remap itself is constant
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

- **Not switching** — check the icon: ⚠️ means permissions were revoked (toggle them off/on); ⚙️ means fewer than 2 favorites are usable, so Caps Lock and Right Shift both open the palette until you check 2–3; ⌨️ means active. Ensure 2+ favorites are checked. A favorite whose layout was removed in System Settings → Keyboard → Input Sources no longer counts and shows unchecked: check another one, or enable it again with `＋`.
- **Layout missing from the list** — enable it with `＋` in the palette's *All Input Sources* section, or via **Open Keyboard Settings…** → Text Input → Input Sources. Both layouts and input methods appear.
- **A second copy does nothing** — only one copy of the app runs at a time. A second copy, started from another folder, with `open -n` or from Xcode, quits at launch and leaves the running copy alone. To switch copies, quit the running one from its menu first.
- **Caps Lock doesn't capitalize, Right Shift isn't a Shift** — while the app has Accessibility they are its trigger keys, also before you pick favorites. They work normally again as soon as the app quits or loses permissions.
- **かな / 英数 (JIS) or 한/영 / 한자 (Korean) keys stop working** — these keys send `LANG1` and `LANG2`, the same codes the app turns Right Shift and Caps Lock into. While the app has Accessibility it takes them over: before 2 favorites are picked both open the palette, after that they cycle and open the palette like Right Shift and Caps Lock. Switch input modes from the menu bar input menu or with `Control`-`Space` meanwhile, or quit the app.
- **Right Shift or Caps Lock still remapped after the app is gone** — this happens only after a crash, a Force Quit from Activity Monitor, `kill -9`, or a stop from a debugger, because the app gets no chance to clean up. Launch the app again, or restart the Mac. To clear the remap from Terminal, run `hidutil property --set '{"UserKeyMapping":[]}'`. That command also clears any key mappings you set with `hidutil` yourself, so run your own `hidutil` command again afterwards.
- **IME flickers / doesn't switch** — switch verification retries automatically; a beep means it still failed, and the HUD then highlights the layout you are still on — please open an issue with the layout name. To record what the app saw, run `log stream --level debug --predicate 'subsystem == "com.doasync.MyKeyboardSwitcher"'` in Terminal, press your switch key a few times, stop it with `Control`-`C`, and attach the output to the issue. The log names input sources and app states, never the keys you press.

## Acknowledgments

MyKeyboardSwitcher is a fork of [CapsLockSwitcher](https://github.com/doasync/CapsLockSwitcher) by [@doasync](https://github.com/doasync) — and was itself distributed as CapsLockSwitcher until v1.2.1. Big thanks to the original author — the core idea (remapping a trigger key through `UserKeyMapping` and selecting input sources directly via Text Input Source Services) and the codebase this project started from are his work.
