# Money — native Android build (Capacitor)

This is your expense tracker wrapped as a **native Android app**. The web app is bundled inside the
APK (in `www/`), so it isn't tied to any hosted URL and works fully offline. Your data is stored on
the device in the app's own WebView storage.

It's already scaffolded with **Capacitor 8** — app name **Money**, package **`com.money.app`**,
launcher icons and splash screen generated. You mainly need to install the build tools, then open
and build.

---

## 1. Install the tools (one time)

- **Node.js 20+ LTS** — https://nodejs.org (verify with `node -v`)
- **Android Studio Otter (2025.2.1) or newer** — https://developer.android.com/studio
  - The required **JDK 21 is bundled with Android Studio** — you do *not* need to install Java separately.
  - On first launch, let the setup wizard install the **Android SDK**. Then open
    **Tools → SDK Manager** and make sure an **SDK Platform for API 24 or higher** and the
    **Android SDK Build-Tools** are installed.

## 2. Restore dependencies

This project ships without `node_modules` to keep it small. From this folder:

```bash
npm install
```

(If you ever change files in `www/`, run `npx cap sync` afterward to copy them into the Android project.)

## 3. Open in Android Studio

```bash
npx cap open android
```

(or just open the `android/` folder in Android Studio). Let Gradle finish syncing the first time —
it may download Gradle/AGP components, which needs internet and a few minutes.

## 4. Build & run

**Run on a phone or emulator:**
- Plug in an Android phone with **USB debugging** on (Settings → Developer options), or start an
  emulator (Device Manager → create an API 24+ device).
- Click the green **Run ▶** button. The app installs and launches.

**Build an installable APK file (to share/sideload):**
- Menu **Build → Build App Bundle(s) / APK(s) → Build APK(s)**.
- When it finishes, click **locate** in the notification. The file is at:
  `android/app/build/outputs/apk/debug/app-debug.apk`
- Copy it to a phone and open it (enable "Install unknown apps" for your Files app when prompted).

A debug APK is perfect for installing on your own phone. It is **not** for the Play Store — for that
you need a signed release build (next section).

## 5. Signed release build (Play Store or distributable APK)

1. Menu **Build → Generate Signed App Bundle / APK**.
2. Choose **Android App Bundle** (`.aab`, required by Google Play) or **APK** (for direct sideloading).
3. **Create a new keystore** when prompted, pick a path, and set passwords.
   ⚠️ **Back up this keystore file and its passwords and keep them forever.** You can only push
   updates to the same app listing if you sign with the same key. Losing it means you can't update.
4. Select the **release** build variant and finish.
   - AAB output: `android/app/build/outputs/bundle/release/app-release.aab`
   - APK output: `android/app/build/outputs/apk/release/app-release.apk`
5. Upload the `.aab` in the Google Play Console (one-time $25 developer account), or share the
   `.apk` directly.

---

## Updating the app later

The web app lives in `www/index.html`. To change it:

1. Edit `www/index.html` (or replace it with a newer `Money.html`).
2. `npx cap sync`
3. Rebuild in Android Studio.

## Changing name, package id, or icons

- **Name / package id:** edit `capacitor.config.json` (`appName`, `appId`). Changing `appId` after a
  Play release is not allowed, so decide before publishing. After changing, the cleanest path is to
  delete `android/` and run `npx cap add android` again.
- **Icons / splash:** replace the source images in `assets/` (`icon.png`, `icon-foreground.png`,
  `icon-background.png`, `splash.png`, `splash-dark.png`) and run:
  ```bash
  npm install -D @capacitor/assets
  npx capacitor-assets generate --android
  ```

## Your data
Your transactions are stored in the phone's **native app storage** (via the Capacitor Preferences
plugin), not in the WebView cache — so clearing the browser/WebView cache won't touch them, and they
persist across app restarts and updates. If the app finds data from an earlier build in the WebView's
localStorage, it migrates it into native storage automatically on first launch.

**Back up data** (Settings) writes a JSON file and opens the Android share sheet, so you can save it to
Files, Google Drive, etc. — a real copy in storage you control. **Restore** reads a backup file back in.

Note: an OS-level "Clear storage" on the app, or uninstalling, still removes the app's data — that's
true of any app. The share-based backup is your safety copy for those cases and for moving phones.

## Troubleshooting
- **"SDK location not found":** open the project in Android Studio once; it writes
  `android/local.properties` with your SDK path. Or set `ANDROID_HOME`.
- **Gradle/JDK errors:** use the JDK bundled with Android Studio (Settings → Build Tools → Gradle →
  Gradle JDK → select the embedded JDK 21). Run **Tools → AGP Upgrade Assistant** if prompted.
- **Gradle sync needs internet** the first time to fetch dependencies; after that it's cached.

## Requirements summary (current as of Capacitor 8)
- Node.js 20+ LTS
- Android Studio Otter 2025.2.1+ (bundles JDK 21)
- Android SDK Platform API 24+ and Build-Tools
- minSdk 24 (Android 7+), target/compile SDK 36 — already set in `android/variables.gradle`
