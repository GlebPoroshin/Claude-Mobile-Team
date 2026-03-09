---
name: mvi-viewmodel
description: Создание MVI ViewModel на базе SharedViewModel<S, E, A> с State/Event/Action sealed classes, handleEvent, updateState, sendAction. Для KMP и нативного Android.
globs: ["**/*.kt"]
autoContext: true
---

# MVI ViewModel Skill (SharedViewModel)

Создаёт ViewModel по паттерну MVI с полным набором файлов на базе `SharedViewModel<S : UiState, E : UiEvent, A : UiAction>`.
Все правила MVI — в `agents/_shared-rules.md`.

## Использование

Когда нужно создать новый ViewModel для фичи, используй этот шаблон.

## Файлы для создания

Для фичи `{Feature}` создай 4 файла:

### 1. `{Feature}State.kt`
```kotlin
sealed class {Feature}State : UiState {
    data object Loading : {Feature}State()
    data class Content(
        // data fields
    ) : {Feature}State()
    data class Error(val message: String?) : {Feature}State()
}
```

### 2. `{Feature}Event.kt`
```kotlin
sealed class {Feature}Event : UiEvent {
    data object OnCreate : {Feature}Event()
    // user actions с префиксом On
}
```

### 3. `{Feature}Action.kt`
```kotlin
sealed class {Feature}Action : UiAction {
    // side-effects: навигация, snackbar, etc.
}
```

### 4. `{Feature}ViewModel.kt`
```kotlin
class {Feature}ViewModel(
    private val someUseCase: SomeUseCase
) : SharedViewModel<{Feature}State, {Feature}Event, {Feature}Action>(
    initialState = {Feature}State.Loading
) {

    override suspend fun handleEvent(event: {Feature}Event) {
        when (event) {
            is {Feature}Event.OnCreate -> loadData()
        }
    }

    private fun loadData() {
        viewModelScope.launch {
            runCatching { someUseCase() }
                .onSuccess { data ->
                    updateState { {Feature}State.Content(/* data */) }
                }
                .onFailure { error ->
                    updateState { {Feature}State.Error(error.message) }
                }
        }
    }
}
```

## SharedViewModel API (уже есть в проекте, НЕ создавать)

```kotlin
abstract class SharedViewModel<S : UiState, E : UiEvent, A : UiAction>(initialState: S) : ViewModel() {
    val viewState: StateFlow<S>          // подписка из UI
    val viewAction: SharedFlow<A>        // одноразовые side-effects
    fun onEvent(event: E)                // вызывается из UI
    protected abstract suspend fun handleEvent(event: E)  // реализуется в каждом VM
    protected fun updateState(reducer: S.() -> S)          // обновить state
    protected fun sendAction(action: A)                    // отправить action
    protected val currentState: S                          // текущий state
}
```

## Marker Interfaces (уже есть в проекте, НЕ создавать)

```kotlin
interface UiState
interface UiEvent
interface UiAction
```

## Правила

- **State — sealed class** (Loading/Content/Error), реализует `UiState`. НЕ data class с boolean-флагами!
- **Event — sealed class**, реализует `UiEvent`, события от UI с префиксом `On`
- **Action — sealed class**, реализует `UiAction`, side-effects для UI
- **ViewModel** наследует `SharedViewModel`, переопределяет `handleEvent`
- `when` по event → отдельный private метод на каждый event
- Используй `updateState {}`, `sendAction()`, `currentState`
- Используй `runCatching` для обработки ошибок
- Никаких других публичных методов кроме унаследованных
- Каждый файл отдельно
- `viewState` (НЕ `state`), `viewAction` (НЕ `actions`)
