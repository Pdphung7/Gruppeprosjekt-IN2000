# Application - VærSikret
# 📦 How to run the app
---
1. Download and install [Android Studio](https://developer.android.com/studio/install)
2. Download the project folder via GitHub
3. Open the project in Android Studio [Guide](https://developer.android.com/studio/projects/create-project#ImportAProject)
4. Create a `local.properties`file in the root of the project if one does not already exist, and add the following:

```
sdk.dir=YOUR_ANDROID_SDK_PATH
MAPBOX_TOKEN=YOUR_MAPBOX_TOKEN
FROST_CLIENT_ID=YOUR_FROST_CLIENT_ID
FROST_CLIENT_SECRET=YOUR_FROST_CLIENT_SECRET
```

- `sdk.dir` is generated automatically by Android Studio
- `MAPBOX_TOKEN` can be obtained by creating an account at [mapbox.com](https://www.mapbox.com/)
- `FROST_CLIENT_ID` and `FROST_CLIENT_SECRET` can be obtained by registering at [frost.met.no](https://frost.met.no/)

5. Set up an emulator or connect an Android device and run the app ([Guide](https://developer.android.com/studio/run/emulator#get-started))

> If an error occurs, run Gradle Sync and press `Build > Clean Project`

# 📚 Libraries
- [Jetpack Compose](https://developer.android.com/develop/ui/compose/documentation), for building the user interface declaratively using `@Composable` functions

- [Material 3 for Compose](https://developer.android.com/develop/ui/compose/designsystems/material3), for pre-built Material Design components in Compose

- [Navigation Compose](https://developer.android.com/develop/ui/compose/navigation), for navigation between screens in the app

- [AndroidX Lifecycle og ViewModel](https://developer.android.com/topic/libraries/architecture/viewmodel), for managing UI state and logic that should persist across recomposition and configuration changes

- [Kotlin Coroutines og Flow](https://kotlinlang.org/docs/coroutines-overview.html), for asynchronous programming and state observation, used among other things for API calls

- [Ktor Client](https://ktor.io/docs/client-create-new-application.html), for making HTTP requests to external APIs

- [Kotlinx Serialization](https://kotlinlang.org/docs/serialization.html), for converting JSON data to Kotlin objects and vice versa

- [Mapbox Maps SDK for Android](https://docs.mapbox.com/android/maps/guides/), for displaying maps in the app and allowing the user to interact with them

- [Mapbox GeoJSON](https://docs.mapbox.com/help/glossary/geojson/), for representing geographic points and coordinates

- [AndroidX Activity Compose](https://developer.android.com/reference/kotlin/androidx/activity/compose/package-summary), for connecting Android activities with Jetpack Compose

- [AndroidX Core / Core KTX](https://developer.android.com/jetpack/androidx/releases/core), for additional AndroidX utility functions and cross-version Android compatibility

- [Vico](https://guide.vico.patrykandpatrick.com/), for rendering interactive line charts in Compose, used to display historical wind and rain data

- [AndroidX Splash Screen](https://developer.android.com/develop/ui/views/launch/splash-screen), for displaying a splash screen on app startup

- [JUnit](https://junit.org/junit4/), for writing and running unit tests

- [Robolectric](https://robolectric.org/), for running Android tests on the JVM without needing an emulator or physical device

- [Espresso](https://developer.android.com/training/testing/espresso), for instrumented UI tests run on an Android device or emulator
