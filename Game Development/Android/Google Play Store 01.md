# Complete Android Studio project template for wrapping your Easel game in an Android app.

This includes:
- A basic `MainActivity.kt` file.
- A simple layout (`activity_main.xml`) with a WebView.
- The required manifest setup.
- Instructions on how to customize the **app icon** and add a **splash screen** (optional).

---

## 📁 Folder Structure of the Project

Here’s what the key files look like:

```
MyEaselGameApp/
├── app/
│   ├── src/main/
│   │   ├── java/ (or kotlin/)
│   │   │   └── com/example/myeaselgameapp/
│   │   │       └── MainActivity.kt
│   │   ├── res/
│   │   │   ├── layout/
│   │   │   │   └── activity_main.xml
│   │   │   ├── mipmap/
│   │   │   │   ├── ic_launcher.png (mdpi)
│   │   │   │   ├── ic_launcher_xhdpi.png
│   │   │   │   └── ... (other densities)
│   │   │   └── values/
│   │   │       └── strings.xml
│   │   └── AndroidManifest.xml
│   └── build.gradle (Module-level)
├── build.gradle (Project-level)
└── settings.gradle
```

---

## ✅ 1. MainActivity.kt (Kotlin)

```kotlin
package com.example.myeaselgameapp

import android.os.Bundle
import android.webkit.WebView
import androidx.appcompat.app.AppCompatActivity

class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val webView = WebView(this)
        setContentView(webView)

        // Enable JavaScript if needed
        webView.settings.javaScriptEnabled = true

        // Load your Easel game URL
        webView.loadUrl("https://easel.games/@yourname/yourgame")
    }
}
```

> Replace `"https://easel.games/@yourname/yourgame"` with your actual Easel game link.

---

## ✅ 2. activity_main.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<WebView xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/webview"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

---

## ✅ 3. AndroidManifest.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.myeaselgameapp">

    <!-- Internet permission is required -->
    <uses-permission android:name="android.permission.INTERNET"/>

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/AppTheme">
        <activity
            android:name=".MainActivity"
            android:configChanges="orientation|keyboardHidden|screenSize">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```

---

## ✅ 4. strings.xml

```xml
<resources>
    <string name="app_name">My Easel Game</string>
</resources>
```

---

## 🎨 Optional: Customize App Icon

### Step-by-step:
1. Create or download a square PNG image (512x512 pixels recommended).
2. Resize it into different sizes for each density folder:
   - `mdpi`: 48x48
   - `hdpi`: 72x72
   - `xhdpi`: 96x96
   - `xxhdpi`: 144x144
   - `xxxhdpi`: 192x192
3. Place them in:
   ```
   app/src/main/res/mipmap/
   ```
   Example:
   - `mipmap-hdpi/ic_launcher.png`
   - `mipmap-xhdpi/ic_launcher.png`
   - etc.

Make sure all are named exactly: `ic_launcher.png`.

---

## ⏱️ Optional: Add a Splash Screen

You can show a splash screen while the WebView loads.

### 1. Create a new layout file: `res/layout/activity_splash.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center"
    android:background="@color/white"
    android:orientation="vertical">

    <ImageView
        android:id="@+id/splash_logo"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:src="@drawable/splash_logo" />

</LinearLayout>
```

> You’ll need to add a `splash_logo.png` to `res/drawable`.

---

### 2. Update `MainActivity.kt` to Show Splash First

```kotlin
package com.example.myeaselgameapp

import android.content.Intent
import android.os.Bundle
import android.os.Handler
import android.os.Looper
import androidx.appcompat.app.AppCompatActivity

class SplashActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_splash)

        // Show splash screen for 2 seconds
        Handler(Looper.getMainLooper()).postDelayed({
            startActivity(Intent(this, MainActivity::class.java))
            finish()
        }, 2000)
    }
}
```

Then rename your original `MainActivity.kt` to `GameActivity.kt`, and update its class name accordingly.

Update `AndroidManifest.xml`:

```xml
<activity
    android:name=".SplashActivity"
    android:label="@string/app_name">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
<activity android:name=".GameActivity" />
```

---

## 🧪 How to Build the App

1. Open the project in **Android Studio**.
2. Click **Sync Now** if prompted.
3. Connect a device or use the emulator.
4. Click **Run** ▶️ to test the app.
5. Once satisfied, go to **Build > Generate Signed Bundle / APK**.
6. Choose **Android App Bundle (.aab)** format (recommended for Google Play).
7. Follow the steps to create a keystore and sign your app.

---

## 🚀 Upload to Google Play Store

Once you have your `.aab` file:
1. Go to [Google Play Console](https://play.google.com/console).
2. Create a new app.
3. Fill out the store listing:
   - Title
   - Short description
   - Long description
   - Screenshots
   - Category
   - Content rating
   - Pricing
4. Upload your `.aab` under **Release management > App releases**.
5. Review and publish!

---

## 📦 Download the Template (Optional)

If you'd prefer to download this as a ZIP file to import directly into Android Studio, I can generate that too — just let me know!

---

## 🙌 Next:
- Generating the full ZIP project
- Customizing the splash screen logo or app icon
- Hosting your Easel game locally instead of loading from a URL
