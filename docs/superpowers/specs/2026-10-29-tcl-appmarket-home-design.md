# TCL App Market Home Shortcut

## Intent

Keep TCL App Market (`com.tcl.appmarket2`) visible on the CleanYourTV home screen for the TCL Thunderbird 75-inch TV, without changing the stock launcher, package installer, or the App Market package itself.

## Approach

The TV's App Market declares a normal `LAUNCHER` entry point but no `LEANBACK_LAUNCHER` entry point, while CleanYourTV's default home list starts from TV apps. Add the package to the launcher's `library_apps` preload list so it is explicitly included in the home tiles. This is preferred over enabling the full app library because it changes only the requested app and keeps the home screen uncluttered.

## Constraints and rollback

Build the launcher through the user's GitHub fork and GitHub Actions, install the resulting APK over ADB, and leave `com.tcl.cyberui`, `com.android.packageinstaller`, and `com.tcl.appmarket2` enabled. Rollback is to restore the previous launcher APK or select the stock Home activity; no TV system package is removed or disabled.

## Verification

Confirm the GitHub Actions build succeeds, the APK installs, the App Market package remains enabled, and the CleanYourTV home screen resolves and displays the App Market tile.
