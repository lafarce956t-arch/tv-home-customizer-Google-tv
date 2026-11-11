# Package safety guide (Android TV / Google TV)

Always list the real packages on the user's TV first (`adb shell pm list packages`). Names vary by brand and Android version. Treat everything below as a starting point, not a guarantee.

## Never touch

| Package | Why |
|---|---|
| `com.android.vending` | Play Store |
| `com.google.android.gms` | Google Play services, many apps depend on it |
| `com.google.android.tvsetup` | Initial setup and account flow |
| `com.android.systemui` | System interface |
| `com.android.settings` / `com.android.tv.settings` | Settings |
| `com.google.android.inputmethod.*` / remote services | Remote and keyboard input |
| Anything starting with `com.android.providers.*` | System data providers |

## Usually the launcher itself (handle with care)

| Package | Note |
|---|---|
| `com.google.android.tvlauncher` | Stock Android TV launcher. Disable only after a custom launcher works. |
| `com.google.android.apps.tv.launcherx` | Google TV launcher. Same rule. |

## Often considered removable (verify on the user's TV first)

These are examples of commonly disabled categories, not exact package lists:
- Pre-installed third-party streaming promo apps from the TV brand
- Brand "store" or "recommendation" apps
- Demo or retail mode apps
- Brand-specific voice assistant promos

For each one: ask the user what the app is, check that it is not needed for the TV's remote, input switching or updates, then disable it with `--user 0`.

## Rule of thumb

If you cannot say what a package does, do not remove it. Disable in small batches, reboot, test, continue.
