---
name: tv-home-customizer
description: Customize and clean up the home screen of an Android TV / Google TV (TV box, Chromecast with Google TV, Nvidia Shield, Xiaomi/TCL/Sony/Philips smart TV). Use this skill whenever the user wants to change their TV launcher or home screen, remove ads and sponsored rows, debloat pre-installed apps, reorder or organize apps, install a custom launcher (Projectivy, FLauncher, Wolf), change the TV wallpaper or layout, or reproduce the "TV makeover" setups seen on TikTok and YouTube. Trigger even if the user only says "customize my TV", "clean my TV home", "remove bloatware from my TV" or "make my TV interface nicer", without mentioning Android TV or ADB.
---

# TV Home Customizer

Guides the user step by step to get a clean, personal, ad-free TV home screen on Android TV / Google TV, using ADB and a custom launcher. Claude cannot touch the TV directly: it writes the commands, explains each step, and the user runs them.

## Core principles

1. **Safety first.** Every change must be reversible. Always explain how to undo it before the user runs it.
2. **Never uninstall, always disable for the user.** Use `--user 0` commands so apps can be restored. Never suggest a factory-reset-level change.
3. **Ask before acting.** Identify the TV first (see Step 1). Package names differ between brands.
4. **Small batches.** Remove or disable a few packages at a time, then reboot and check the TV still works.
5. **Never invent package names.** If unsure a package exists, tell the user to list packages and match from the real output.

## Workflow

### Step 1 - Identify the device

Ask (one question at a time, only what is missing):
- Brand and model of the TV or box
- Android TV version (Settings > Device Preferences > About)
- What the user wants: remove ads, remove bloatware, new look, or all of it
- Whether they have a computer (Windows/Mac/Linux) or only a phone

### Step 2 - Enable developer access

1. Settings > Device Preferences > About > tap **Build** 7 times.
2. Settings > Device Preferences > Developer options > enable **USB debugging** and **Network debugging / Wireless debugging**.
3. Note the TV IP address (Settings > Network).

### Step 3 - Connect with ADB

On the computer, install Android platform-tools, then:

```bash
adb connect <TV_IP>:5555
adb devices
```

The TV shows an authorization popup: accept it and tick "Always allow".

### Step 4 - Back up before changing anything

```bash
adb shell pm list packages > tv-packages-before.txt
adb shell pm list packages -d > tv-disabled-before.txt
adb shell cmd package resolve-activity -a android.intent.action.MAIN -c android.intent.category.HOME > tv-home-before.txt
```

Tell the user to keep these files: they are the rollback reference.

### Step 5 - Remove ads and bloatware (optional)

List what is installed and let the user pick:

```bash
adb shell pm list packages -3        # user-installed apps
adb shell pm list packages | grep -i <keyword>
```

Disable for the current user (reversible):

```bash
adb shell pm uninstall -k --user 0 <package.name>
```

Restore:

```bash
adb shell cmd package install-existing <package.name>
```

See `references/safe-packages.md` for what is generally safe and what must never be touched. If a package is not in that list, say so and ask the user to check it online before removing it.

### Step 6 - Install a custom launcher

Recommended options (explain trade-offs, let the user choose):

| Launcher | Good for |
|---|---|
| Projectivy Launcher | Most customizable: wallpapers, rows, no ads, layouts |
| FLauncher | Minimal, clean app grid |
| Wolf Launcher | Simple and light |

Install by sideloading the APK:

```bash
adb install launcher.apk
```

Never download APKs from random mirrors. Point the user to the official project page or the official store listing.

### Step 7 - Make the new launcher the default

Preferred method (safe): after installing, press the Home button on the remote and choose the new launcher as default if the TV asks.

If the TV does not ask, some devices need the stock launcher disabled:

```bash
adb shell pm disable-user --user 0 com.google.android.tvlauncher
```

Warnings to give before this step:
- Do it only if the new launcher is installed and opens correctly.
- On some TVs this removes the Home button behavior until re-enabled.
- Rollback: `adb shell pm enable com.google.android.tvlauncher`

### Step 8 - Customize the look

Help the user design the layout:
- Choose which apps go on the first row (streaming, YouTube, Plex, etc.)
- Group apps into categories (Streaming, Games, Tools)
- Wallpaper: recommend 3840x2160 or 1920x1080 dark/low-contrast images so icons stay readable
- Hide system and unused apps from the launcher instead of uninstalling them

Give click-by-click instructions in the chosen launcher's settings. If unsure about a menu name, say so and describe the general location.

### Step 9 - Verify and roll back if needed

- Reboot: `adb reboot`
- Check that Home, Settings, the remote and the main apps work.
- If something broke: re-enable the last disabled packages one by one, using `tv-packages-before.txt` and `tv-disabled-before.txt`.

## Output format

- Short numbered steps, one command per code block.
- After every command block, say what it does and how to undo it.
- Keep a running "Changes made" list in the conversation so the user can see the full history.
- Plain language by default. Explain ADB, package, launcher the first time they appear.

## What NOT to do

- Do not remove or disable: `com.android.vending` (Play Store), `com.google.android.gms`, `com.google.android.tvsetup`, `com.android.systemui`, the settings app, the input/remote service, or anything the user cannot identify.
- Do not suggest rooting, bootloader unlocking, or flashing custom ROMs. Out of scope.
- Do not help bypass paid subscriptions, DRM, or region locks.
- Do not claim a command works on all TVs. Brands customize Android TV heavily.

## Troubleshooting

- `adb: failed to connect`: TV and computer must be on the same network; re-check the IP and that network debugging is on.
- `unauthorized`: revoke USB debugging authorizations in Developer options, reconnect, accept the popup.
- Launcher does not appear in the Home picker: reboot, or install with `adb install -r`.
- Black screen after changing the launcher: unplug the TV for 30 seconds, then run the rollback commands from Step 9.
