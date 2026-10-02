# MyKeyboardSwitcher — Releases

Binary releases of **MyKeyboardSwitcher**, a macOS menu-bar app for instant cycling through your favorite keyboard layouts: **Right Shift** cycles, **Caps Lock** opens the favorites palette (flags, checkboxes, mouse or keyboard).

Source code lives in a private repository — this repo only distributes builds.

## Install

1. Download `MyKeyboardSwitcher-<version>.dmg` from the [latest release](../../releases/latest)
2. Open the DMG and drag `MyKeyboardSwitcher.app` to **Applications**
3. Clear the Gatekeeper quarantine (the app is ad-hoc signed):

   ```bash
   xattr -cr /Applications/MyKeyboardSwitcher.app
   ```

4. Launch it and grant **Accessibility** when prompted (Privacy & Security → Accessibility)

## Thanks

MyKeyboardSwitcher is a fork of [CapsLockSwitcher](https://github.com/doasync/CapsLockSwitcher) by [@doasync](https://github.com/doasync).
