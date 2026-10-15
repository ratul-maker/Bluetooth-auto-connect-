# Earbuds Auto-Connect

A small, offline Android app (plain Java, **no third-party libraries**) that looks after one
pair of already-paired Bluetooth earbuds: reconnect attempts, honest battery readings, low-battery
alerts, battery/connection history, a Quick Settings tile, audio diagnostics and a self-check screen.

- Built for **Android 11 (API 30)**, installs on Android 8.0+ (`minSdk 26`, `targetSdk 30`)
- Needs only: Bluetooth, foreground service, run-at-startup. **No internet, location, microphone, contacts or notification permission.**
- No account, ads, analytics or server. Everything is stored on the phone.

## Get the APK (this project does not contain a pre-built APK)

An APK has to be compiled with the Android SDK. Pick one route:

### A. GitHub Actions (recommended, free, reliable)
1. Create a **private** GitHub repository and upload everything in this folder (keep the folder structure,
   including `.github/workflows/build-apk.yml`).
2. Open the repo's **Actions** tab. The *Build APK* workflow runs on every push (or press *Run workflow*).
3. When it turns green, open the repo's **Releases** page on your phone and tap `EarbudsAutoConnect.apk`.
   (The same file is also under the run's *Artifacts*.)

### B. Phone-only, no folder upload (one file + one pasted workflow)
1. Create a private GitHub repo. Upload the single file `EarbudsAutoConnect-project.zip`.
2. *Add file -> Create new file*, name it `.github/workflows/build-from-zip.yml`, paste the contents of
   `phone-only/build-from-zip.yml`, and commit.
3. *Actions -> Build APK from zip -> Run workflow*. Download from *Releases* as above.

### C. Netlify (best effort, untested)
Netlify normally hosts websites; it does not build Android apps. `netlify.toml` + `tools/netlify_build.sh`
try to download a JDK and the Android SDK inside Netlify's build container, build the APK, and publish it
next to `public/index.html` as a download page. If that build fails, use A or B.

### D. Android Studio
Open the folder, *Build -> Build APK(s)* (release is already configured to sign with the bundled key).

## Install
Open the downloaded `.apk` on the phone, allow "install unknown apps" for your browser/Files app if asked,
tap Install. Then *Choose earbuds* and pick the already-paired earbuds.

The release is signed with a personal key kept in `signing/` so every build can update the previous one.
Keep the repo **private** and do not publish this app to a store with that key.

## What it does (and honestly cannot do)
| Feature | Reality on Android 11, non-rooted |
|---|---|
| Choose earbuds | Lists only already-paired devices; never scans or pairs |
| Connection status | Read from Android's A2DP (music) and hands-free (calls) state, never assumed |
| Reconnect | Android has **no public API to start a headset connection**. "Connect now"/Auto-Connect send a best-effort Bluetooth LE request, then **verify the real state**. Classic-only earbuds: the app says it can't and just waits |
| Retries | At most 4 automatic tries (20 s, 1 min, 3 min, 10 min), then pauses until something changes or you tap Connect now |
| Disconnect | Not possible via public API - not offered |
| Battery | One value, **only if the earbuds report it** (Android's system report, headset +IPHONEACCEV report in 10% steps, or the standard BLE battery service when exactly one exists). Otherwise: "Battery information unavailable for this device" |
| Left / right / case battery | Not available through public APIs - never shown, never guessed |
| Alerts | One alert per low episode, re-armed only after recovery; never on missing data |
| History | Optional, real readings only, no playtime prediction |
| Background | Foreground service + notification (Connect now / Open app / Stop). XOS/Android may still pause apps - see Settings > Keep it running |
| Tile | "Earbuds" Quick Settings tile showing real state |

## Testing status (read this)
**Verified in the build environment**
- All 78 classes compile with `javac` against the real Android API 34 and API 30 `android.jar` (0 errors).
- 57 plain-Java checks of the pure logic pass (`tools/run_logic_tests.sh`): retry schedule is bounded and
  terminates, alert de-duplication/re-arm, invalid readings ignored, `+IPHONEACCEV` parsing, history sessions.
- All XML is well-formed; every manifest class and resource reference exists.

**NOT verified (could not be, here)**
- The Gradle/AGP build itself, resource compilation, APK signing, installation, launch.
- Any run on a phone or emulator: screen layout/appearance, dark mode, the tile, notifications.
- Any real Bluetooth behaviour: reconnection, battery reports, background survival, XOS behaviour.
Treat the first install as a test and use *Diagnostics -> Copy report* if something looks wrong.

## Layout
`app/src/main/java/com/btearbuds/autoconnect/`
- `bt/` Bluetooth: `BtController` (the only place that talks to Android Bluetooth), `GattProbe`, `AudioRoute`, `Compat`, `Diagnostics`, `StatusText`
- `logic/` pure Java rules with tests: `RetryPolicy`, `BatteryAlertEngine`, `VendorBatteryParser`, `InsightCalculator`
- `service/` monitoring foreground service, notifications, boot receiver, Quick Settings tile
- `data/` local settings and SQLite history
- `ui/` screens built in code (no layout XML, no UI libraries)
