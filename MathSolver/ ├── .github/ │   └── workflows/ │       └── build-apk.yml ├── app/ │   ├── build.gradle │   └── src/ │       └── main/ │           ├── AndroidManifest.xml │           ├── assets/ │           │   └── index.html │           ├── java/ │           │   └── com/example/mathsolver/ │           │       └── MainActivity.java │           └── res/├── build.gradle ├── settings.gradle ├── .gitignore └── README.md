plugins {
    id 'com.android.application' version '8.7.0' apply false
}
pluginManagement {
    repositories {
        google()
        mavenCentral()
        gradlePluginPortal()
    }
}

dependencyResolutionManagement {
    repositoriesMode.set(
        RepositoriesMode.FAIL_ON_PROJECT_REPOS
    )

    repositories {
        google()
        mavenCentral()
    }
}

rootProject.name = "MathSolver"
include ':app'
plugins {
    id 'com.android.application'
}

android {
    namespace 'com.example.mathsolver'
    compileSdk 35

    defaultConfig {
        applicationId 'com.example.mathsolver'

        minSdk 23
        targetSdk 35

        versionCode 5
        versionName '5.0'
    }

    compileOptions {
        sourceCompatibility JavaVersion.VERSION_17
        targetCompatibility JavaVersion.VERSION_17
    }
}
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <uses-permission android:name="android.permission.INTERNET"/>

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:roundIcon="@mipmap/ic_launcher"
        android:label="MathSolver + Русский"
        android:theme="@android:style/Theme.Material.Light.NoActionBar">

        <activity
            android:name=".MainActivity"
            android:exported="true">

            <intent-filter>

                <action
                    android:name="android.intent.action.MAIN"/>

                <category
                    android:name="android.intent.category.LAUNCHER"/>

            </intent-filter>

        </activity>

    </application>

</manifest>
package com.example.mathsolver;

import android.app.Activity;
import android.content.Intent;
import android.net.Uri;
import android.os.Bundle;

import android.webkit.ValueCallback;
import android.webkit.WebChromeClient;
import android.webkit.WebSettings;
import android.webkit.WebView;
import android.webkit.WebViewClient;

public class MainActivity extends Activity {

    private static final int PICK = 1001;

    private WebView webView;

    private ValueCallback<Uri[]> fileCallback;


    @Override
    protected void onCreate(Bundle savedInstanceState) {

        super.onCreate(savedInstanceState);


        webView = new WebView(this);

        setContentView(webView);


        WebSettings settings =
                webView.getSettings();


        settings.setJavaScriptEnabled(true);

        settings.setDomStorageEnabled(true);

        settings.setAllowFileAccess(true);

        settings.setAllowContentAccess(true);

        settings.setDatabaseEnabled(true);


        settings.setMixedContentMode(
                WebSettings.MIXED_CONTENT_ALWAYS_ALLOW
        );


        webView.setWebViewClient(
                new WebViewClient()
        );


        webView.setWebChromeClient(

            new WebChromeClient() {

                @Override
                public boolean onShowFileChooser(
                        WebView webView,
                        ValueCallback<Uri[]> callback,
                        FileChooserParams params
                ) {

                    if (fileCallback != null) {

                        fileCallback.onReceiveValue(null);

                    }


                    fileCallback = callback;


                    Intent intent =
                            new Intent(
                                Intent.ACTION_GET_CONTENT
                            );


                    intent.addCategory(
                            Intent.CATEGORY_OPENABLE
                    );


                    intent.setType("image/*");


                    startActivityForResult(

                        Intent.createChooser(
                            intent,
                            "Выберите фото задачи"
                        ),

                        PICK
                    );


                    return true;

                }

            }

        );


        webView.loadUrl(
                "file:///android_asset/index.html"
        );

    }


    @Override
    protected void onActivityResult(
            int requestCode,
            int resultCode,
            Intent data
    ) {

        super.onActivityResult(
                requestCode,
                resultCode,
                data
        );


        if (requestCode == PICK) {

            if (fileCallback == null) {
                return;
            }


            Uri[] result = null;


            if (
                resultCode == RESULT_OK
                &&
                data != null
                &&
                data.getData() != null
            ) {

                result =
                        new Uri[]{
                            data.getData()
                        };

            }


            fileCallback.onReceiveValue(result);

            fileCallback = null;

        }

    }


    @Override
    public void onBackPressed() {

        if (
            webView != null
            &&
            webView.canGoBack()
        ) {

            webView.goBack();

        } else {

            super.onBackPressed();

        }

    }

}
name: Build MathSolver Full APK

on:
  push:
    branches: [ main ]

  workflow_dispatch:

jobs:

  build:

    runs-on: ubuntu-latest

    steps:

      - name: Checkout
        uses: actions/checkout@v4


      - name: Set up Java 17
        uses: actions/setup-java@v4

        with:
          distribution: temurin
          java-version: '17'


      - name: Set up Gradle 8.9
        uses: gradle/actions/setup-gradle@v4

        with:
          gradle-version: '8.9'


      - name: Build APK
        run: |
          gradle assembleDebug --stacktrace


      - name: Rename APK
        run: |
          cp \
          app/build/outputs/apk/debug/app-debug.apk \
          MathSolver-Full.apk


      - name: Upload APK
        uses: actions/upload-artifact@v4

        with:
          name: MathSolver-Full-APK
          path: MathSolver-Full.apk
          if-no-files-found: error
