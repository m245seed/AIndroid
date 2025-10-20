# AIndroid - Android Compose Project

This is an Android Studio project with an Empty Compose Activity setup.

## Project Structure

```
AIndroid/
├── app/
│   ├── src/
│   │   └── main/
│   │       ├── java/com/example/aindroid/
│   │       │   ├── MainActivity.kt
│   │       │   └── ui/theme/
│   │       │       ├── Color.kt
│   │       │       ├── Theme.kt
│   │       │       └── Type.kt
│   │       ├── res/
│   │       │   ├── drawable/
│   │       │   ├── mipmap-*/
│   │       │   └── values/
│   │       └── AndroidManifest.xml
│   └── build.gradle.kts
├── gradle/
├── build.gradle.kts
├── settings.gradle.kts
└── gradle.properties
```

## Features

### Compose Enabled
The project has Jetpack Compose enabled in `app/build.gradle.kts`:
```kotlin
buildFeatures {
    compose = true
}
```

### Dependencies

The following dependencies have been added as requested:

1. **androidx.documentfile:documentfile:1.0.1**
   - Provides access to documents and directories using the Storage Access Framework

2. **org.jetbrains.kotlinx:kotlinx-serialization-json:1.6.3**
   - Kotlin multiplatform / multi-format serialization library

3. **com.squareup.okhttp3:okhttp:4.12.0**
   - HTTP client library for making network requests

### MainActivity.kt

The main activity is set up with a basic Compose UI:
- Empty Compose Activity template
- Greeting composable function
- Material 3 theme (AIndroidTheme)
- Preview support

## Building the Project

To build the project, run:
```bash
./gradlew build
```

To run on an emulator or device:
```bash
./gradlew installDebug
```

## Requirements

- Android Studio Hedgehog | 2023.1.1 or newer
- JDK 8 or higher
- Android SDK with API 34
- Kotlin 1.9.20

## Configuration

- **Package name**: com.example.aindroid
- **Min SDK**: 24 (Android 7.0 Nougat)
- **Target SDK**: 34 (Android 14)
- **Compile SDK**: 34

## Gradle

- **Gradle version**: 8.2
- **Android Gradle Plugin**: 8.1.4
- **Kotlin version**: 1.9.20
