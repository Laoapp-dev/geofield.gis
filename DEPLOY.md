# 🗺️ GeoField Map — Deployment & Build Guide

> **GIS Fieldwork App**: GPS tracking · Vector drawing · Waypoints · Export GPX/GeoJSON/KML

---

## 📦 What's Included

| Component | Description |
|---|---|
| `app/` | Android APK source (Jetpack Compose + Leaflet WebView) |
| `webapp/` | Standalone web app (GitHub Pages ready) |
| `.github/workflows/` | Auto-deploy GitHub Actions |

---

## 🌐 Part 1: Deploy to GitHub Pages

### Step 1 — Push to GitHub

```bash
git init
git add .
git commit -m "feat: GeoField Map initial release"
git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPO.git
git push -u origin main
```

### Step 2 — Enable GitHub Pages

1. Go to your repo → **Settings** → **Pages**
2. Under **Source**, select **GitHub Actions**
3. Click **Save**

### Step 3 — Done! ✅

The GitHub Action (`.github/workflows/deploy-pages.yml`) will automatically:
- Detect pushes to `main`/`master`
- Upload the `webapp/` folder as the Pages site
- Deploy to: `https://YOUR_USERNAME.github.io/YOUR_REPO/`

> **Note**: GPS tracking requires HTTPS. GitHub Pages provides HTTPS by default ✅

---

## 📱 Part 2: Build & Install the APK

### Prerequisites

- [Android Studio Ladybug](https://developer.android.com/studio) (2024.2.1+)
- Java 11 or newer (bundled with Android Studio)
- Android SDK 24+ (minSdk) and SDK 36 (target)

### Step 1 — Set up environment

Create a `.env` file in the project root:

```
GEMINI_API_KEY=your_actual_gemini_api_key_here
```

> Get a free key at: https://aistudio.google.com/app/apikey

### Step 2 — Open in Android Studio

1. Open Android Studio
2. **File → Open** → select the `geofield-map` folder
3. Wait for Gradle sync to complete (first time takes 2-3 min)

### Step 3 — Build Debug APK

```
Build → Build Bundle(s)/APK(s) → Build APK(s)
```

Or via terminal:

```powershell
.\gradlew.bat assembleDebug
```

APK output: `app/build/outputs/apk/debug/app-debug.apk`

### Step 4 — Install on Device

**Option A: Direct USB Install**
```powershell
adb install app\build\outputs\apk\debug\app-debug.apk
```

**Option B: Copy APK to phone**
- Copy `app-debug.apk` to your phone
- Enable "Install unknown apps" in phone settings
- Tap the APK file to install

**Option C: Run directly from Android Studio**
- Connect phone via USB (enable USB debugging)
- Click the green **▶ Run** button

### Step 5 — Build Release APK (for distribution)

1. Generate a keystore (one-time):
```powershell
keytool -genkey -v -keystore my-upload-key.jks -keyalg RSA -keysize 2048 -validity 10000 -alias upload
```

2. Set environment variables:
```powershell
$env:KEYSTORE_PATH = "C:\path\to\my-upload-key.jks"
$env:STORE_PASSWORD = "your_store_password"
$env:KEY_PASSWORD = "your_key_password"
```

3. Build:
```powershell
.\gradlew.bat assembleRelease
```

Output: `app/build/outputs/apk/release/app-release.apk`

---

## 🗂️ Project Structure

```
geofield-map/
├── .github/
│   └── workflows/
│       └── deploy-pages.yml       ← Auto GitHub Pages deploy
├── app/
│   └── src/main/
│       ├── AndroidManifest.xml
│       ├── assets/
│       │   ├── leaflet_map.html   ← Leaflet map (WebView)
│       │   ├── leaflet.js
│       │   └── leaflet.css
│       └── java/com/example/
│           ├── MainActivity.kt
│           ├── ui/
│           │   ├── MainMapScreen.kt
│           │   ├── MapWebView.kt  ← Fixed: race condition + isMapReady guard
│           │   ├── GeoViewModel.kt
│           │   ├── GeminiAssistant.kt  ← Fixed: model name
│           │   └── OnboardingScreen.kt
│           └── data/
│               ├── AppDatabase.kt
│               ├── Entities.kt
│               └── GeoRepository.kt
├── webapp/
│   ├── index.html                 ← Web app entry point
│   ├── app.js                     ← Full GIS logic
│   ├── style.css                  ← Premium dark UI
│   ├── leaflet.js                 ← Leaflet (local copy)
│   ├── leaflet.css
│   ├── manifest.json              ← PWA installable
│   └── sw.js                      ← Service Worker (offline)
├── .env.example
├── build.gradle.kts
└── gradle/
    └── libs.versions.toml
```

---

## 🔑 Fixes Applied

| # | File | Fix |
|---|---|---|
| 1 | `app/build.gradle.kts` | Fixed `compileSdk` DSL syntax |
| 2 | `app/build.gradle.kts` | Removed invalid debug `signingConfig` |
| 3 | `gradle/libs.versions.toml` | KSP version `2.2.10-1.0.29` (matches Kotlin) |
| 4 | `gradle/libs.versions.toml` | Updated `composeBom` to `2025.01.00` |
| 5 | `gradle.properties` | Added `kotlin.jvm.target=11` |
| 6 | `GeminiAssistant.kt` | Fixed model name: `gemini-1.5-flash` |
| 7 | `MapWebView.kt` | Fixed race condition with `isMapReady` guard |
| 8 | `MapWebView.kt` | Suppressed deprecated API warnings |
| 9 | `leaflet_map.html` | Added missing `@keyframes pulse` CSS |
| 10 | `leaflet_map.html` | Fixed `onMapReady()` with retry loop |
| 11 | `AndroidManifest.xml` | Added `hardwareAccelerated=true` |

---

## 🌟 Web App Features

- 🗺️ **Leaflet map** — Satellite / Streets / Terrain basemaps
- 📍 **GPS tracking** — Real-time location via browser Geolocation API
- ✏️ **Vector drawing** — Waypoints, polylines, polygons with live measurements
- 🔍 **Geocoding search** — Nominatim OpenStreetMap search
- 📤 **Export** — GeoJSON, GPX, KML, CSV
- 📂 **Import** — GeoJSON paste import
- 💾 **Persistence** — localStorage (survives page refresh)
- 📦 **PWA** — Installable as app, offline shell cached

---

## ❓ Troubleshooting

**Gradle sync fails?**
→ Check Android Studio: File → Invalidate Caches → Restart

**GPS not working in web app?**
→ Must be served over HTTPS (GitHub Pages ✅) or localhost

**Gemini AI not responding?**
→ Add your API key to `.env`: `GEMINI_API_KEY=your_key`

**APK install blocked?**
→ Enable "Install from unknown sources" in Android Settings → Security
