# Overall Architecture

The app follows the **MVVM architecture** (Model–View–ViewModel) combined with **UDF** (Unidirectional Data Flow). This is the recommended architecture for Android applications using Jetpack Compose.

- **Data flows downward**: from network layer → repository → ViewModel → UI
- **Events flow upward**: user actions are sent to the ViewModel which updates state
- The UI layer only reads state, it never writes directly to data sources

---

## Layers and responsibilities

### Data layer (`data/`)

#### `network/`
Contains the API clients that communicate directly with external services:

- **`FrostApi`** fetches wind and rain data from the Norwegian Meteorological Institute's Frost API. Uses Ktor with HTTP Basic Auth (client ID and secret stored in `local.properties` and exposed via `BuildConfig`).
- **`MapboxGeocodingApi`** performs address search and reverse geocoding via the Mapbox Geocoding API.

The network layer is the only place in the app that knows about concrete URLs and API structures. The rest of the app only interacts with the return values.

#### `model/`
Contains pure data classes with no logic:

- **`FrostModelsV0` / `FrostRainIdfModels`** model the JSON response from the Frost API
- **`PropertyModel`** represents a selected property with coordinates and address
- **`BeaufortScale`** enum for the Beaufort scale with colors and boundaries per level
- **`WindRarity`** enum for classifying abnormally high wind measurements
- **`MeasureModels`** data classes and enums for climate measures

#### `repository/`
Repositories fetch data from the network layer, process it and return domain objects ready for use in the ViewModel. No Compose or Android UI dependencies exist here.

- **`FrostRepository`** analyses wind gust and mean wind data, calculates IDF rain data and fetches historical rain observations
- **`MeasureRepository`** reads climate measures from local JSON files in `assets/`

---

### ViewModels

All ViewModels extend `AndroidViewModel`, which provides access to the `Application` context. This is necessary to retrieve string resources and support localisation without holding a reference to an Activity. State is exposed as `StateFlow` and coroutines run in `viewModelScope`.

| ViewModel | Responsibility |
|---|---|
| `FrostViewModel` | Fetches and holds wind and rain data for the selected location |
| `MapboxViewModel` | Manages the map, address search, geocoding and selected property |
| `MeasureViewModel` | Loads and filters climate measures |
| `InfoViewModel` | Delivers questions and answers to the FAQ screen |

State is modelled as **sealed classes** with four states:

```kotlin
sealed class WindGustState {
    data object Empty   : WindGustState()
    data object Loading : WindGustState()
    data class Success(val data: WindGustCardData) : WindGustState()
    data class Error(val message: String) : WindGustState()
}
```

This ensures the UI always knows exactly which state it is in, and `when` expressions become exhaustive and enforce handling of all cases.

---

### UI layer (`ui/`)

The entire UI is built with **Jetpack Compose**. Each screen is a `@Composable` function that receives a ViewModel and a `NavController`. The UI observes state with `collectAsStateWithLifecycle()`, which ensures the observation is lifecycle-aware and does not leak resources when the app is in the background.

Reusable components such as `HistoricalTrendButton`, `ShowMeasuresButton` and `ThresholdRow` reside in `ui/components/` and are shared across screens to avoid code duplication.

---

### Navigation

The app uses **Navigation Compose** with a single `NavHost` defined in `AppNavGraph.kt`. All routes are defined in `Screen.kt` as a sealed class. Coordinates are passed as string arguments in the route so that the selected location is available on destination screens. The app has a single Activity (`MainActivity`); all navigation happens within Compose.

---

## Object-oriented principles

### Low coupling
- The UI layer has no knowledge of the network layer; it only communicates with the ViewModel
- The ViewModel has no knowledge of Compose; it only exposes `StateFlow`
- The repository has no knowledge of the ViewModel; it returns pure domain objects
- The network classes (`FrostApi`, `MapboxGeocodingApi`) are independent of each other

### High cohesion
- Each class has a single clear responsibility: `FrostApi` talks to the Frost API, `FrostRepository` analyses and processes the data, `FrostViewModel` holds and exposes state to the UI
- Reusable UI components are collected in `ui/components/` and contain no business logic
- Theme colours and typography are centralised in `ui/theme/` and used consistently via the `AppColors` object throughout the app

---

## Design patterns

### MVVM (Model–View–ViewModel)
MVVM separates the UI from business logic. The ViewModel survives configuration changes such as screen rotation and keeps state stable. The UI only renders what the ViewModel exposes — it makes no independent decisions about data.

### UDF (Unidirectional Data Flow)
State is always updated through the ViewModel. The user clicks → the UI calls a function in the ViewModel → the ViewModel updates the `StateFlow` → the UI recomposes automatically. This one-way flow makes it straightforward to trace state changes and debug.

### Repository pattern
Separates data fetching from business logic and the UI. Makes it easy to swap out or extend data sources without changing the ViewModel or UI.

### Sealed class state pattern
All asynchronous operations are modelled with `Empty / Loading / Success / Error`. This eliminates invalid states and makes `when` expressions exhaustive.

---

## Technologies

| Technology | Usage |
|---|---|
| Jetpack Compose | Declarative UI |
| Material 3 for Compose | Material Design components |
| Navigation Compose | Navigation between screens |
| AndroidX ViewModel + Lifecycle | MVVM and lifecycle management |
| Kotlin Coroutines + StateFlow | Asynchronous programming and state |
| Ktor Client | HTTP calls to Frost API and Mapbox |
| Kotlinx Serialization | JSON parsing |
| Mapbox Maps SDK v11 | Interactive map |
| Mapbox Geocoding API | Address search and reverse geocoding |
| Vico | Line charts for historical data |
| AndroidX Splash Screen | Splash screen on app startup |

### External APIs
- **Frost API (met.no)** — historical wind and rain data from Norwegian weather stations. Requires client ID and secret in `local.properties` (not version controlled).
- **Mapbox** — maps and geocoding. Requires a Mapbox token in `local.properties`.

---

## API level and Android version

| Setting | Value |
|---|---|
| `minSdk` | 26 (Android 8.0 Oreo) |
| `targetSdk` | 36 |
| `compileSdk` | 36 |

**Why minSdk 26?**
Android 8.0 (Oreo) was chosen as the minimum requirement because it provides access to modern APIs such as `ConnectivityManager.NetworkCallback` for network monitoring, it covers over 95% of active Android devices, and Jetpack Compose and Mapbox Maps SDK v11 work best from API 26 and above. It also avoids the need for many backwards-compatibility workarounds that would unnecessarily complicate the codebase.

---

## Code style and conventions

- **Kotlin** is used consistently throughout the project
- State is always exposed as `StateFlow`, never as `MutableStateFlow` directly to the UI
- Strings displayed in the UI are fetched from `strings.xml` (and `strings_en.xml` for English); no hardcoded strings in code
- Colours and styles are fetched from `AppColors` and `MaterialTheme`; no hardcoded colour values in composables
- Comments are written in English and explain *why*, not *what*
- Sealed classes are used for all UI state that can have multiple states
