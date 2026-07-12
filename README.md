```markdown
# 📱 Termux Android App Builder

Build and install Android APKs entirely from **Termux** — no Android Studio, no computer, no Gradle, just your phone and the command line.

This project provides a complete, minimal build pipeline that compiles Android applications directly on your device. It starts with a simple Hello World app to demonstrate the entire process from source code to installable APK.

---

## 🎯 Why This Exists

Android development traditionally requires Android Studio, Gradle, and a desktop computer. This project proves that with just Termux and a few command-line tools, you can:

- Write Java code on your phone
- Compile Android resources
- Build DEX bytecode
- Package and sign APKs
- Install directly on your device

No cloud services, no remote build servers, no desktop required. Everything happens locally on your Android device.

---

## 📋 Prerequisites

| Requirement | How to Get It |
|-------------|---------------|
| **Termux** | Install from [F-Droid](https://f-droid.org/packages/com.termux/) (NOT Play Store — that version is outdated) |
| **Java JDK 21** | `pkg install openjdk-21` |
| **Storage Access** | `termux-setup-storage` (required to copy APK to shared storage) |
| **Internet** | Only needed during `setup.sh` to download android.jar |

---

## 🚀 Quick Start

### Step 1: Clone the Repository

```bash
git clone https://github.com/0-Hackibit/termux-android-app-builder.git
cd termux-android-app-builder
```

### Step 2: Run Setup

This installs all build tools and downloads the Android SDK platform:

```bash
bash setup.sh
```

**What happens during setup:**
- Installs `aapt2`, `dx`, `apksigner`, and `zip` via `pkg`
- Downloads `android.jar` (API 33, ~25MB) to `~/android-sdk/`
- Generates a debug keystore at `~/.debug.keystore`
- Creates necessary directory structure

### Step 3: Grant Storage Permission

```bash
termux-setup-storage
```

This allows copying the built APK to your phone's shared storage for installation.

### Step 4: Build

```bash
bash build.sh
```

A successful build outputs something like:

```
=== Cleaning build artifacts ===
=== Step 1: Compile resources ===
=== Step 2: Link resources ===
=== Step 3: Compile Java ===
=== Step 4: DEX ===
=== Step 5: Package APK ===
=== Step 6: Sign ===

=== BUILD SUCCESSFUL ===
```

### Step 5: Install

Your APK is at `build/apk/app.apk`. Install it:

```bash
cp build/apk/app.apk ~/storage/shared/
termux-open ~/storage/shared/app.apk
```

Or navigate to your file manager and tap the APK to install.

---

## 🔧 The Build Pipeline Explained

Unlike Gradle-based builds, this pipeline runs each step manually. Here's exactly what happens:

### Step 1: Resource Compilation (`aapt2 compile`)
```
res/layout/activity_main.xml  ──→  build/compiled_res/layout_activity_main.xml.flat
res/values/strings.xml        ──→  build/compiled_res/values_strings.arsc.flat
```
Converts human-readable XML resources into binary `.flat` files.

### Step 2: Resource Linking (`aapt2 link`)
```
*.flat files + AndroidManifest.xml  ──→  build/apk/app-unaligned.apk + build/gen/R.java
```
Links compiled resources, generates the `R.java` file with resource IDs, and creates an unsigned APK skeleton.

### Step 3: Java Compilation (`javac`)
```
src/*.java + gen/R.java  ──→  build/obj/*.class
```
Compiles Java source code to JVM bytecode using the Android framework JAR.

### Step 4: DEX Conversion (`dx`)
```
build/obj/*.class  ──→  build/classes.dex
```
Converts standard JVM bytecode to Dalvik Executable format that Android understands.

### Step 5: APK Packaging (`zip`)
```
app-unaligned.apk + classes.dex  ──→  app.apk (unaligned)
```
Injects the DEX file into the APK.

### Step 6: Signing (`apksigner`)
```
app.apk  ──→  app.apk (signed with debug keystore)
```
Signs the APK so Android will install it.

---

## 📁 Project Structure

```
termux-android-app-builder/
│
├── AndroidManifest.xml              # App configuration (package name, permissions, activities)
├── build.sh                         # Complete build script (all 6 steps)
├── setup.sh                         # One-time environment setup
├── README.md                        # This file
│
├── res/                             # Android resources
│   ├── layout/
│   │   └── activity_main.xml        # UI layout definition
│   └── values/
│       └── strings.xml              # String constants
│
├── src/                             # Java source code
│   └── com/
│       └── example/
│           └── hello/
│               └── MainActivity.java # Main app code
│
└── build/                           # Build output (created by build.sh)
    ├── apk/
    │   └── app.apk                  # Final installable APK
    ├── compiled_res/                # Intermediate .flat files
    ├── gen/                         # Generated R.java
    └── obj/                         # Compiled .class files
```

---

## 📝 Understanding the Hello World App

### MainActivity.java
```java
package com.example.hello;

import android.app.Activity;
import android.os.Bundle;
import android.widget.TextView;

public class MainActivity extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        TextView greeting = findViewById(R.id.greeting);
        greeting.setText("Hello from Termux!");
    }
}
```

### activity_main.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:gravity="center">
    
    <TextView
        android:id="@+id/greeting"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="24sp"
        android:text="Loading..." />
        
</LinearLayout>
```

---

## 🔨 Customizing for Your Own App

### Change the App Name
Edit `res/values/strings.xml`:
```xml
<string name="app_name">Your App Name</string>
```

### Change the Package Name
1. Update `package` attribute in `AndroidManifest.xml`
2. Rename directories under `src/` to match
3. Update the `javac` command in `build.sh`

### Add More Activities
1. Create new `.java` files in `src/`
2. Declare them in `AndroidManifest.xml`
3. They'll be compiled automatically (build.sh finds all `.java` files)

### Add Permissions
Edit `AndroidManifest.xml`:
```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.CAMERA" />
```

---

## ⚠️ Limitations

### Technical Limitations

| Limitation | Why | Workaround |
|------------|-----|------------|
| **Java 8 only** | `dx` tool can't handle Java 9+ bytecode (`invokedynamic`) | Use anonymous classes instead of lambdas, avoid `var` keyword |
| **No Gradle/Maven** | No dependency management | Copy JAR files manually or write framework-only code |
| **No AndroidX** | Requires Gradle for artifact resolution | Use legacy `android.app.*` and `android.widget.*` APIs |
| **No ProGuard/R8** | Minification tools not installed | Accept larger APK size (~50KB for Hello World) |
| **No Jetpack Compose** | Requires Kotlin + Gradle plugin | Use XML layouts with `LinearLayout`, `RelativeLayout`, etc. |
| **No Multi-DEX** | Complex to set up manually | Keep app under 64K method limit |
| **Debug builds only** | Signed with debug keystore | Can't publish to Play Store, but fine for personal use |
| **API 33 target only** | Downloads single `android.jar` | Change `setup.sh` to download different API version |

### Practical Limitations

- **Slow compilation** — Phone CPUs are slower than desktop
- **Limited editing** — No IDE features like autocomplete or refactoring
- **Manual error handling** — No linting or static analysis
- **No emulator** — Must install directly on device to test
- **Battery drain** — Compilation uses significant CPU

---

## 💡 Tips for Development

### Editing Code on Your Phone
- Use `nano` or `vim` in Termux for quick edits
- Use a code editor app like **Acode** or **DroidEdit** for better experience
- Connect a Bluetooth keyboard for serious coding

### Debugging
- View runtime logs: `logcat | grep -E "AndroidRuntime|System.err"`
- Check build errors carefully — error messages are your friend
- Test incrementally — build often to catch issues early

### Adding External JARs
1. Download JAR to a `libs/` directory
2. Add to classpath in `build.sh`:
   ```bash
   javac -classpath "libs/mylib.jar:$ANDROID_JAR" ...
   ```
3. Include in DEX conversion:
   ```bash
   dx --dex --output=classes.dex build/obj libs/mylib.jar
   ```

---

## 📊 Build Performance

Tested on a mid-range Android device (Snapdragon 778G, 8GB RAM):

| Step | Time |
|------|------|
| Resource compile | ~1 second |
| Resource link | ~1 second |
| Java compile | ~3 seconds |
| DEX conversion | ~2 seconds |
| Packaging + Signing | ~1 second |
| **Total** | **~8 seconds** |

First build is slower due to JVM warmup. Subsequent builds are faster.

---

## 🐛 Common Issues

### "android.jar not found"
Run `bash setup.sh` first to download the SDK.

### "cannot find symbol"
Make sure your Java files are in the correct package directory matching your `AndroidManifest.xml`.

### "invokedynamic requires --min-sdk-version >= 26"
You're using a lambda expression. Replace with an anonymous class:
```java
// Don't do this:
button.setOnClickListener(v -> doSomething());

// Do this instead:
button.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        doSomething();
    }
});
```

### "INSTALL_FAILED_UPDATE_INCOMPATIBLE"
Uninstall the existing app first, or increment `versionCode` in `AndroidManifest.xml`.

---

## 🔐 Security Notes

- The debug keystore uses well-known passwords (`android`/`android`) — don't use for production
- APKs are signed with debug certificates, valid for 10,000 days
- No code signing or verification beyond Android's built-in checks

---

## 🙏 Credits

- **Termux** — The terminal emulator that makes this possible
- **Android Open Source Project** — For the SDK and build tools
- **OpenJDK** — Java runtime for ARM Android devices

---

## 🔗 Resources

- [Android Developer Documentation](https://developer.android.com/docs)
- [Termux Wiki](https://wiki.termux.com/)
- [AAPT2 Documentation](https://developer.android.com/studio/command-line/aapt2)
- [D8/DX Documentation](https://developer.android.com/studio/command-line/d8)
```
