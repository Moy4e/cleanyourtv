# TCL App Market Home Shortcut Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Make TCL App Market (`com.tcl.appmarket2`) appear as a normal tile on the CleanYourTV home screen for the user's TCL Thunderbird TV.

**Architecture:** Use the launcher's existing `library_apps` preload mechanism to include the App Market package even though its APK exposes only a normal `LAUNCHER` activity. Build the launcher through the user's GitHub fork, then install the resulting APK through the already-tested temporary PackageInstaller bypass that restores the installer immediately.

**Tech Stack:** Android XML resources, Gradle/Android build, GitHub Actions, ADB over Wi-Fi, repository `tools/tv.py` helpers.

**Spec:** `docs/superpowers/specs/2026-10-29-tcl-appmarket-home-design.md`

## Global Constraints

- Keep `com.tcl.appmarket2` installed and enabled.
- Keep `com.tcl.cyberui` and `com.android.packageinstaller` enabled.
- Do not disable or remove any additional TV system package.
- The change must be reversible by selecting the stock Home activity or installing the prior launcher APK.

## Review Focus

- A package with `LAUNCHER` but no `LEANBACK_LAUNCHER` entry must still appear on Home; verify the App Market tile is present.
- The GitHub build must produce a successful release APK; verify the Actions run and artifact.
- The temporary install path must restore PackageInstaller if installation fails; verify its enabled state after installation.
- Existing disabled-package state must remain unchanged; compare the exact disabled package list before and after.
- Home must still resolve to `com.gathin.tvhome/.MainActivity`; verify with `cmd package resolve-activity`.

### Task 1: Preload TCL App Market in the launcher

**Files:**
- Modify: `launcher/app/src/main/res/values/config.xml` in the `library_apps` string array
- Test: GitHub Actions `Launcher build and unit tests` job

**Interfaces:**
- Consumes: the existing `library_apps` configuration read by `Config.load()` and `AppAdapter.reload()`.
- Produces: a release launcher whose Home tile query explicitly includes `com.tcl.appmarket2`.

- [x] **Step 1: Add the exact package name**

  Add one item, `com.tcl.appmarket2`, to the existing `library_apps` array. Do not enable the full app library or alter `hidden_apps`, `dock_apps`, or package disable lists.

- [x] **Step 2: Commit and push the configuration change**

  Commit the config and accompanying design/plan documents with a focused message, then push `main` to the user's fork so GitHub Actions builds the forked source.

- [x] **Step 3: Run the online build and verify it**

  Dispatch the fork's CI workflow and wait for completion. Expected: the launcher build/unit-test job succeeds and an `app-release.apk` artifact is available.

### Task 2: Install and verify on the TV

**Files:**
- Use: `../install_t982_launcher.py` and the downloaded CI artifact
- Test: TV package state, Home resolver, launcher screenshot, and App Market launch

**Interfaces:**
- Consumes: the successful APK artifact from Task 1.
- Produces: the updated launcher active on `<tv ip>` with the original installer and stock launcher still recoverable.

- [x] **Step 1: Transfer and install the artifact**

  Push the APK to `/data/local/tmp/tvhome.apk` and run the existing helper, which temporarily disables only `com.android.packageinstaller`, installs the APK, and restores the installer in a `finally` block.

- [x] **Step 2: Verify package safety and Home resolution**

  Confirm `com.tcl.appmarket2`, `com.android.packageinstaller`, and `com.tcl.cyberui` are enabled; confirm the seven previously disabled packages are unchanged; confirm Home resolves to `com.gathin.tvhome/.MainActivity`.

- [x] **Step 3: Verify the user-visible result**

  Press Home, capture a screenshot, and open the App Market tile. Expected: the tile is visible and the App Market starts normally.

- [x] **Step 4: Commit the completed change**

  Record the implementation commit and verification evidence in the final handoff, without deleting the rollback artifacts or disabling ADB while the user is still testing.
