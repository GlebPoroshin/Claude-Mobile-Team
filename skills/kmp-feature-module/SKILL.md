---
name: kmp-feature-module
description: Скаффолдинг feature-модуля KMP с api/data/presentation структурой. Clean Architecture, все слои, DI, маппинг.
globs: ["**/*.kt", "**/build.gradle.kts"]
autoContext: true
---

# KMP Feature Module Skill

Создаёт полную структуру feature-модуля по Clean Architecture.
Все архитектурные правила — в `agents/_shared-rules.md`.

## Использование

Когда создаётся новая фича, используй эту структуру.

## Модульная структура

```
common/feature/{feature-name}/
  api/
    models/
      {Feature}.kt                    # Доменная модель (чистый data class)
    {Feature}Repository.kt            # Интерфейс Repository
  data/
    models/
      {Feature}Dto.kt                 # Сетевая модель (@Serializable)
    mappers/
      {Feature}Mapper.kt              # Маппинг Dto → Domain
    {Feature}RemoteDataSource.kt      # Сетевой источник (Ktor)
    {Feature}LocalDataSource.kt       # Локальный источник (SqlDelight/Settings)
    {Feature}RepositoryImpl.kt        # Реализация Repository
  presentation/
    {Feature}ViewModel.kt             # MVI ViewModel
    {Feature}State.kt                 # UI State (sealed class)
    {Feature}Event.kt                 # UI Events (sealed class)
    {Feature}Action.kt                # Side-effects (sealed class)
  di/
    {Feature}Module.kt                # DI module (Koin или Kodein)
```

## Слои и зависимости

```
api/ (контракты)
  ├── Доменные модели (чистые data class, без аннотаций)
  └── Интерфейсы Repository

data/ (реализация)
  ├── Сетевые модели (@Serializable, суффикс Dto/Response/Request)
  ├── Маппинг data → domain
  ├── DataSource (Ktor, SqlDelight, MultiplatformSettings)
  └── RepositoryImpl (агрегатор DataSource, маппинг)

presentation/ (UI логика)
  ├── ViewModel (MVI: SharedViewModel<State, Event, Action>)
  ├── State (sealed class: Loading / Content / Error)
  ├── Event (sealed class)
  └── Action (sealed class)
```

## Правила зависимостей

- DataSource → только Ktor / SqlDelight / MultiplatformSettings
- DataSource ← НЕ зависит от → DataSource
- Repository → только DataSource
- Repository ← НЕ зависит от → Repository
- UseCase → только **интерфейс** Repository (не Impl)
- ViewModel → только UseCase

## DI модуль

### Koin
```kotlin
val {feature}Module = module {
    factory { {Feature}RemoteDataSource(get()) }
    factory { {Feature}LocalDataSource(get()) }
    factory<{Feature}Repository> { {Feature}RepositoryImpl(get(), get()) }
    factory { Get{Feature}UseCase(get()) }
    viewModel { {Feature}ViewModel(get()) }
}
```

### Kodein
```kotlin
val {feature}Module = DI.Module("{feature}") {
    bindProvider { {Feature}RemoteDataSource(instance()) }
    bindProvider { {Feature}LocalDataSource(instance()) }
    bindProvider<{Feature}Repository> { {Feature}RepositoryImpl(instance(), instance()) }
    bindProvider { Get{Feature}UseCase(instance()) }
}
```

## Checklist после создания

- [ ] Каждый класс в отдельном файле
- [ ] Доменные модели без аннотаций сериализации
- [ ] Сетевые модели с @Serializable
- [ ] Маппинг между слоями
- [ ] UseCase зависит от интерфейса Repository, не от Impl
- [ ] State/Event/Action — sealed class (не sealed interface, не data class)
- [ ] DI модуль создан и зарегистрирован
- [ ] Зависимости между слоями корректны
