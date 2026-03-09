# Shared Rules (referenced by all agents)

## Clean Architecture (STRICT)

```
DataSource (сеть/БД/in-memory)
    ↓
Repository (агрегация DataSource, маппинг data→domain)
    ↓
UseCase (бизнес-логика, один invoke)
    ↓
ViewModel (MVI: State + Events + Actions)
```

### Dependency Rules
- **DataSource** зависит ТОЛЬКО от: Ktor, SqlDelight, MultiplatformSettings, платформенных API (EncryptedSharedPreferences, Keychain через iosMain и т.д.)
- **DataSource** НЕ зависит от другого DataSource
- **Repository** зависит ТОЛЬКО от DataSource
- **Repository** НЕ зависит от другого Repository
- **Repository** маппит сетевые/локальные модели в доменные
- **UseCase** зависит ТОЛЬКО от **интерфейса** Repository (НЕ от реализации) и других UseCase при необходимости
- **UseCase** имеет ОДИН публичный метод `operator fun invoke`
- **ViewModel** зависит ТОЛЬКО от UseCase (и фасадов типа AnalyticsFacade)

### Repository Interface Pattern
```kotlin
// api/FeatureRepository.kt — интерфейс
interface FeatureRepository {
    suspend fun getItems(): List<Item>
}

// data/FeatureRepositoryImpl.kt — реализация
class FeatureRepositoryImpl(
    private val remoteDataSource: FeatureRemoteDataSource
) : FeatureRepository {
    override suspend fun getItems(): List<Item> {
        return remoteDataSource.getItems().map { it.toDomain() }
    }
}

// UseCase ВСЕГДА зависит от интерфейса:
class GetItemsUseCase(
    private val repository: FeatureRepository  // интерфейс, НЕ Impl!
)
```

### Module Structure
```
common/feature/{name}/
  api/              # Доменные модели + интерфейсы UseCase/Repository
  data/             # Реализация интерфейсов, DataSource, сетевые модели, маппинг
  presentation/     # ViewModel (MVI), State, Event, Action, UI-модели
  di/               # DI module
```

Если фича добавляется в существующий модуль — не создавай новый, расширяй.

## MVI ViewModel (SharedViewModel base)

Все ViewModel наследуются от `SharedViewModel<S : UiState, E : UiEvent, A : UiAction>`:

```kotlin
class FeatureViewModel(
    private val someUseCase: SomeUseCase
) : SharedViewModel<FeatureState, FeatureEvent, FeatureAction>(
    initialState = FeatureState.Loading
) {

    override suspend fun handleEvent(event: FeatureEvent) {
        when (event) {
            is FeatureEvent.OnCreate -> loadData()
            is FeatureEvent.OnRetry -> loadData()
            is FeatureEvent.OnItemClick -> sendAction(FeatureAction.OpenDetail(event.id))
        }
    }

    private fun loadData() {
        viewModelScope.launch {
            runCatching { someUseCase() }
                .onSuccess { items -> updateState { FeatureState.Content(items = items) } }
                .onFailure { error -> updateState { FeatureState.Error(error.message) } }
        }
    }
}
```

### SharedViewModel API (уже есть в проекте, НЕ создавать)

Публичный API (доступен из UI):
- `viewState: StateFlow<S>` — состояние для UI (НЕ `state`)
- `viewAction: SharedFlow<A>` — одноразовые side-effects (НЕ `actions`)
- `onEvent(event: E)` — публичный метод, вызывает handleEvent

Protected API (доступен только внутри ViewModel):
- `handleEvent(event: E)` — abstract suspend, переопределяется в каждом ViewModel
- `updateState(reducer: S.() -> S)` — обновляет state
- `sendAction(action: A)` — отправляет action
- `currentState: S` — текущий state (для чтения внутри VM, НЕ для UI)

### State / Event / Action Rules
- **State** — sealed class, реализует маркер `UiState`. Варианты: Loading / Content / Error. НЕ data class с boolean-флагами!
- **Event** — sealed class, реализует маркер `UiEvent`, события от UI с префиксом `On`
- **Action** — sealed class, реализует маркер `UiAction`, side-effects (навигация, snackbar)
- `when` по sealed class → отдельный private метод на каждый event
- Никаких других публичных методов кроме унаследованных от SharedViewModel
- Каждый в ОТДЕЛЬНОМ файле

### State / Event / Action Examples
```kotlin
sealed class FeatureState : UiState {
    data object Loading : FeatureState()
    data class Content(val items: List<Item>) : FeatureState()
    data class Error(val message: String?) : FeatureState()
}

sealed class FeatureEvent : UiEvent {
    data object OnCreate : FeatureEvent()
    data object OnRetry : FeatureEvent()
    data class OnItemClick(val id: String) : FeatureEvent()
}

sealed class FeatureAction : UiAction {
    data class OpenDetail(val id: String) : FeatureAction()
}
```

### Marker Interfaces
Проверь через `ast-index search "UiState"` — если есть в проекте, НЕ создавай.
Если проект новый и маркеров нет — создай в `common/core/` или `common/presentation/base/`:
```kotlin
interface UiState
interface UiEvent
interface UiAction
```

## Models

- **Сетевые**: `@Serializable`, суффикс `Dto` или `Response`/`Request`, живут в `data/models/`
- **Доменные**: чистые data class, без аннотаций, живут в `api/models/`
- **UI модели**: при необходимости, живут в `presentation/`
- **Маппинг**: extension-функции `Dto.toDomain()`, живут в `data/mappers/`
- Каждый класс в ОТДЕЛЬНОМ файле

## File Structure

- Один класс = один файл
- Sealed class/interface = отдельный файл
- Data class = отдельный файл
- Нет god-файлов (>600 строк — warning, >1000 — blocker)

## Git Rules

- Тикет извлекается из имени ветки: `ABCD-1234` из `ABCD-1234-description`
- Коммит: `[ABCD-1234] Short message` — ТОЛЬКО заголовок, без body
- Атомарные коммиты: один логический блок = один коммит
- Запустить `detekt` и `ktlint` перед коммитом
- НИКОГДА не пушить
- НИКОГДА не упоминать AI/Claude в коммитах, коде, комментариях

## Search Rules

- ВСЕГДА использовать `ast-index` ПЕРВЫМ для поиска по коду
- grep/Grep ТОЛЬКО если ast-index вернул пустой результат или нужен regex/string literals

## Unit Testing (Developer Responsibility)

Каждый разработчик пишет unit-тесты для своего кода. Unit-тесты — ответственность того, кто реализует фичу.

### Правила
- Unit-тест на КАЖДЫЙ новый/изменённый UseCase — без исключений
- **AAA паттерн**: Arrange → Act → Assert
- **Именование**: `` `should <expected> when <condition>` ``
- **MockK** для мокирования зависимостей
- **coEvery / coVerify** для suspend-функций
- **runBlocking** для запуска suspend в тестах
- **Turbine** для тестирования Flow (`useCase().test { ... }`)
- Тесты в `commonTest` (KMP/CMP) или `test` (нативный Android)
- Покрыть: happy path, error cases, edge cases
- Один тест = одно поведение

### Шаблон теста UseCase
```kotlin
class GetItemsUseCaseTest {

    private val repository: FeatureRepository = mockk()
    private val useCase = GetItemsUseCase(repository)

    @Test
    fun `should return items when repository returns data`() {
        // Arrange
        val expected = listOf(
            Item(id = "1", name = "Item 1", price = 10.0),
            Item(id = "2", name = "Item 2", price = 20.0)
        )
        coEvery { repository.getItems() } returns expected

        // Act
        val result = runBlocking { useCase() }

        // Assert
        assertEquals(expected, result)
        coVerify(exactly = 1) { repository.getItems() }
    }

    @Test
    fun `should throw exception when repository fails`() {
        // Arrange
        coEvery { repository.getItems() } throws IOException("Network error")

        // Act & Assert
        assertFailsWith<IOException> {
            runBlocking { useCase() }
        }
    }
}
```

### Тестирование Flow (Turbine)
```kotlin
@Test
fun `should emit items from repository flow`() = runBlocking {
    // Arrange
    val items = listOf(Item(id = "1", name = "Test", price = 10.0))
    every { repository.observeItems() } returns flowOf(items)

    // Act & Assert
    useCase().test {
        assertEquals(items, awaitItem())
        awaitComplete()
    }
}
```

### Что тестировать
- Корректный возврат данных при нормальном выполнении
- Обработка ошибок (исключения от Repository)
- Бизнес-логика (фильтрация, сортировка, трансформация)
- Вызов правильных методов Repository с правильными параметрами
- Edge cases: пустые списки, null значения, граничные условия

### Что НЕ тестировать
- ViewModel (пока)
- DataSource напрямую
- Маппинг моделей (если тривиальный)

## Error Handling

- Если `ast-index` не проиндексирован — запусти `ast-index rebuild`
- Если Gradle wrapper отсутствует — сообщи пользователю
- Если detekt/ktlint не настроены — пропусти с warning, сообщи пользователю
- Если сборка не проходит — попробуй диагностировать, при невозможности сообщи пользователю
