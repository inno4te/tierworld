# The Berean School — Android App

A Christian apologetics school focused on engaging Islamic theology.  
Wraps the full single-file HTML course as a native Android APK using Capacitor.

---

## App Login Credentials

| Field | Value |
|---|---|
| Username | `tierworld` |
| Password | `tierw0rld` |

---

## Getting the APK (No Setup Required)

Every time you push to the `main` branch, GitHub Actions builds a fresh APK automatically.

1. Go to your repository on GitHub
2. Click the **Actions** tab
3. Click the most recent **"Build Berean School APK"** workflow run
4. Scroll to the bottom — click **berean-school-debug-apk** to download the zip
5. Unzip it to get `app-debug.apk`
6. Transfer to your Android device and install (see below)

### Installing on Android
1. Transfer `app-debug.apk` to your Android phone (email, Google Drive, USB cable)
2. On your phone: **Settings → Security (or Privacy) → Install unknown apps**
3. Enable installation from whichever source you used (Files, Gmail, etc.)
4. Tap the APK file → Install

---

## Creating a Release (for versioned, permanently-linked APKs)

Tag a commit and GitHub will attach the APK to a formal Release:

```bash
git tag v1.0.0
git push origin v1.0.0
```

Then find it under **Releases** on your GitHub repo page — permanent, sharable download link.

---

## Repository Structure

```
berean-app/
├── .github/
│   └── workflows/
│       └── build-apk.yml        ← GitHub Actions CI — builds the APK
├── public/
│   └── index.html               ← The complete Berean School site (3,000+ lines)
├── capacitor.config.json        ← App ID, name, webDir, plugin settings
├── package.json                 ← npm dependencies (Capacitor)
├── .gitignore
└── README.md
```

---

## How It Works

1. `public/index.html` is the entire school — all 39 modules, the Reference Library, login gate, quiz logic, and progress tracking are embedded in this single file.
2. **Capacitor** wraps it in a native Android WebView, giving it a proper app icon, splash screen, and APK packaging.
3. **GitHub Actions** installs Capacitor, runs `cap sync android`, then calls `gradlew assembleDebug` to produce the APK — no local Android Studio or Java setup required on your end.

---

## Making Changes to the Course Content

1. Edit `public/index.html` with your changes (or replace it entirely with a new version from Claude)
2. Commit and push to `main`:
   ```bash
   git add public/index.html
   git commit -m "Update course content"
   git push
   ```
3. GitHub Actions automatically builds a new APK within ~5-8 minutes
4. Download from the Actions tab as above

---

## Customising the App

### App Icon and Name
Edit `capacitor.config.json`:
```json
{
  "appId": "com.inno4te.bereanschool",
  "appName": "The Berean School",
  ...
}
```
- `appId` — must be a unique reverse-domain identifier (used by the Play Store if you ever publish)
- `appName` — the name shown under the icon on Android

### Splash Screen Colour
The background colour on launch (`#1a1a2e` — dark navy, matching the school's dark theme) is set in `capacitor.config.json` under `plugins.SplashScreen.backgroundColor`.

### Building a Signed (Release) APK
The APK built by this workflow is a **debug APK** — installable directly but not publishable to the Google Play Store. To publish to the Play Store you need a **signed release APK**:

1. Generate a keystore:
   ```bash
   keytool -genkey -v -keystore berean-school.keystore \
     -alias berean -keyalg RSA -keysize 2048 -validity 10000
   ```
2. Add the keystore to GitHub Secrets:
   - `KEYSTORE_BASE64` (base64-encoded keystore file)
   - `KEY_ALIAS`
   - `KEY_PASSWORD`
   - `STORE_PASSWORD`
3. Modify the workflow to run `assembleRelease` and sign with the keystore

For personal/ministry distribution (sideloading), the debug APK from this workflow is fully sufficient.

---

## If the Build Fails

| Symptom | Fix |
|---|---|
| `ANDROID_SDK_ROOT not set` | The `android-actions/setup-android@v3` step should handle this — check it ran |
| `gradlew: Permission denied` | The workflow has `chmod +x android/gradlew` — ensure that step ran |
| `cap add android` hangs | Usually a Node version issue — workflow pins Node 20, which is correct |
| Artifact not appearing | Check the Actions tab for error logs — expand each step |

---

## Progress Logging (Google Apps Script Backend)

If you want quiz responses and module progress logged to a Google Sheet:

1. Deploy your `Code.gs` as a Google Apps Script Web App
2. Paste the `/exec` URL into `public/index.html` at the `CONFIG.SCRIPT_URL` line:
   ```javascript
   const CONFIG = {
     SCRIPT_URL: "https://script.google.com/macros/s/YOUR_ID_HERE/exec"
   };
   ```
3. Rebuild the APK by pushing the updated `index.html`

Without the URL, the app works perfectly — progress just isn't logged externally.
