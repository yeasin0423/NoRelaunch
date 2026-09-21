# AGENTS.md - NoRelaunch

## Project
- Xposed/LSPosed module forked from `YifePlayte/NoRelaunch`. Single-module (`app`), Kotlin + Compose + Miuix 0.4.7 + Haze. Hook: `hook/MainHook.kt:39` scans `hook/hooks/singlepackage` -> `Android.kt:NoRelaunch` hooks `ActivityRecord.shouldRelaunchLocked`. UI entry `activity/MainActivity.kt`.

## Toolchain (verified from `app/build.gradle.kts:10-51`, `build.gradle.kts`, `gradle-wrapper.properties`)
- `compileSdk 36`, `targetSdk 36`, `minSdk 33` (forked from upstream 34 for Android 13 - `app/build.gradle.kts:16`). Keep `compileSdk 36`.
- AGP `8.13.1`, Gradle `8.13`, Kotlin `2.2.0`, `jvmToolchain(21)`, `android.useAndroidX=true`.
- `androidResources.additionalParameters += --allow-reserved-package-id --package-id 0x45` - do not remove.

## Build
- `./gradlew assembleDebug` / `./gradlew assembleRelease` (`app/build.gradle.kts:21-27` renames output to `NoRelaunch-<version>.apk`). Release: `isMinifyEnabled=true`, `isShrinkResources=true` + `proguard-rules.pro`.
- Local SDK not required - CI builds on GitHub. If building locally needs JDK 21 and platform 36.
- No test/lint suite.

## Android 13 Backport Gotcha
- `NoRelaunch.kt:3-8` imports 3 API-34 constants (`CONFIG_FONT_WEIGHT_ADJUSTMENT`, `CONFIG_GRAMMATICAL_GENDER`, `CONFIG_ASSETS_PATHS`). They are `public static final int` inlined at compile time against `compileSdk 36`; harmless on A13 (`shouldRelaunchLocked` just never sees those bits). Do not revert `minSdk` to 34.
- No other API-34 usage found (`LoadPackageParam.kt:20` is `SDK_INT` only). `enableEdgeToEdge()` is via AndroidX.

## CI (`.github/workflows/build.yml`)
- `runs-on: ubuntu-latest`, `actions/setup-java@v4` (temurin 21, gradle cache), `gradle/actions/setup-gradle@v4`, `assembleDebug` + `assembleRelease`, `actions/upload-artifact@v4`. No `android-actions/setup-android` (its `sdkmanager tools` package is removed; runner image already has `cmdline-tools 16.0` + AGP auto-downloads `compileSdk 36`).
- Triggers: `push`/`pull_request` on `main`/`master` + `workflow_dispatch`. Check Actions tab after `git push origin main`.

## Git
- Remote `origin https://github.com/yeasin0423/NoRelaunch.git`. Keep commits atomic (e.g., minSdk fix separate from CI). Push with `git push origin main`; workflow builds APKs automatically.
