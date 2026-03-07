# Architecture: <Feature Name>

## Module Structure
```
common/feature/{name}/
  api/
    models/
    {Feature}Repository.kt
  data/
    models/
    mappers/
    {Feature}RemoteDataSource.kt
    {Feature}LocalDataSource.kt
    {Feature}RepositoryImpl.kt
  presentation/
    {Feature}ViewModel.kt
    {Feature}State.kt
    {Feature}Event.kt
    {Feature}Action.kt
  di/
    {Feature}Module.kt
```

## Interfaces (api/)

```kotlin
interface FeatureRepository {
    suspend fun getItems(): List<Item>
}
```

## Models

### Network Models (data/)
```kotlin
@Serializable
data class ItemDto(...)
```

### Domain Models (api/)
```kotlin
data class Item(...)
```

### UI Models (presentation/)
<!-- Если нужны отдельные UI модели -->

### Mapping Strategy
```kotlin
fun ItemDto.toDomain() = Item(...)
```

## Data Flow

```
API (Ktor) → RemoteDataSource → Repository → UseCase → ViewModel → UI
                                     ↑
LocalDataSource (SqlDelight) ────────┘
```

## ViewModel Contract (SharedViewModel)

```kotlin
// State — sealed class, реализует UiState
sealed class FeatureState : UiState {
    data object Loading : FeatureState()
    data class Content(...) : FeatureState()
    data class Error(val message: String?) : FeatureState()
}

// Events — sealed class, реализует UiEvent
sealed class FeatureEvent : UiEvent {
    data object OnCreate : FeatureEvent()
    // ...
}

// Actions — sealed class, реализует UiAction
sealed class FeatureAction : UiAction {
    // ...
}

// ViewModel наследует SharedViewModel
class FeatureViewModel(...) : SharedViewModel<FeatureState, FeatureEvent, FeatureAction>(
    initialState = FeatureState.Loading
)
```

## Platform-Specific Parts

### Android
- UI: Jetpack Compose
- Навигация: {Cicerone / Navigation 3}

### iOS
- UI: {SwiftUI / UIKit}
- Навигация: стандартная

### expect/actual (если есть)
```kotlin
expect class PlatformSpecific { ... }
```

## DI Configuration

```kotlin
// {Koin / Kodein} module
val featureModule = module { ... }
```

## Navigation Integration

Как фича встраивается в навигацию проекта.

## File List

Полный список файлов для создания:
1. `path/to/File1.kt`
2. `path/to/File2.kt`

## Decisions & Rationale

| Решение | Обоснование |
|---------|-------------|
| Решение 1 | Почему так |
| Решение 2 | Почему так |
