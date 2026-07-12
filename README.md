# 📱 Termux Android App Builder

Build and install Android APKs entirely from **Termux** — no Android Studio, no computer, just your phone.

This is a minimal Hello World app that demonstrates the complete build pipeline: resource compilation, Java compilation, DEX conversion, APK packaging, and signing.

---

## 📋 Prerequisites

- Android device with [Termux](https://f-droid.org/packages/com.termux/) installed
- Java JDK: `pkg install openjdk-21`

---

## 🚀 Quick Start

```bash
# Clone the repo
git clone https://github.com/0-Hackibit/termux-android-app-builder.git
cd termux-android-app-builder

# Install build tools and download android.jar
bash setup.sh

# Grant storage access (needed once for installing APKs)
termux-setup-storage

# Build the APK
bash build.sh

# Install on device
# Your APK file (app.apk) will be in your phone storage after a successful build.
```
