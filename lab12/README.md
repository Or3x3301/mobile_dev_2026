# Лабораторная работа №12

<div align="center">

**МИНИСТЕРСТВО НАУКИ И ВЫСШЕГО ОБРАЗОВАНИЯ РОССИЙСКОЙ ФЕДЕРАЦИИ**  
**ФЕДЕРАЛЬНОЕ ГОСУДАРСТВЕННОЕ БЮДЖЕТНОЕ ОБРАЗОВАТЕЛЬНОЕ УЧРЕЖДЕНИЕ ВЫСШЕГО ОБРАЗОВАНИЯ**  
**«САХАЛИНСКИЙ ГОСУДАРСТВЕННЫЙ УНИВЕРСИТЕТ»**

<br>
<br>

Институт естественных наук и техносферной безопасности  
Кафедра информатики  
**Чегодаев Артем Сергеевич**

<br>
<br>
<br>
<br>

Лабораторная работа №12
**«Выполнение длительных операций (симуляция загрузки) с использованием viewModelScope»**  
01.03.02 Прикладная математика и информатика  
3 Курс

<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>

<div align="right">
Научный руководитель<br>
Соболев Евгений Игоревич
</div>

<br>
<br>
<br>

г. Южно-Сахалинск  
2026 г.

</div>

---

## Цель Работы

Научиться выполнять длительные операции в фоновом потоке с использованием корутин и `viewModelScope`, управлять состоянием загрузки в UI, реализовать имитацию загрузки данных и обработку ошибок.

## UiState.kt
```kotlin
package com.example.todoapp.ui

import com.example.todoapp.database.TaskEntity

sealed class TasksUiState {
    object Loading : TasksUiState()
    data class Success(val tasks: List<TaskEntity>) : TasksUiState()
    data class Error(val message: String) : TasksUiState()
}
```

## TaskRepository.kt
```kotlin
package com.example.todoapp.data.repository

import com.example.todoapp.database.TaskEntity
import kotlinx.coroutines.flow.Flow

interface TaskRepository {
    fun getAllTasks(): Flow<List<TaskEntity>>
    suspend fun addTask(title: String)
    suspend fun deleteTask(task: TaskEntity)
    suspend fun deleteTaskById(id: Long)
    suspend fun updateTask(task: TaskEntity)
    suspend fun toggleTaskCompletion(task: TaskEntity, isCompleted: Boolean)
    suspend fun deleteAllTasks()
    suspend fun getTasksOnce(): List<TaskEntity>
}
```

## TaskRepositoryImpl.kt
```kotlin
package com.example.todoapp.data.repository

import com.example.todoapp.database.TaskDao
import com.example.todoapp.database.TaskEntity
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.first

class TaskRepositoryImpl(
    private val taskDao: TaskDao
) : TaskRepository {

    override fun getAllTasks(): Flow<List<TaskEntity>> = taskDao.getAllTasks()

    override suspend fun addTask(title: String) {
        val task = TaskEntity(title = title)
        taskDao.insertTask(task)
    }

    override suspend fun deleteTask(task: TaskEntity) {
        taskDao.deleteTask(task)
    }

    override suspend fun deleteTaskById(id: Long) {
        taskDao.deleteTaskById(id)
    }

    override suspend fun updateTask(task: TaskEntity) {
        taskDao.updateTask(task)
    }

    override suspend fun toggleTaskCompletion(task: TaskEntity, isCompleted: Boolean) {
        val updatedTask = task.copy(isCompleted = isCompleted)
        taskDao.updateTask(updatedTask)
    }

    override suspend fun deleteAllTasks() {
        taskDao.deleteAll()
    }

    override suspend fun getTasksOnce(): List<TaskEntity> {
        return taskDao.getAllTasks().first()
    }
}
```

## MainViewModel.kt
```kotlin
package com.example.todoapp.ui.theme

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.example.todoapp.data.repository.TaskRepository
import com.example.todoapp.database.TaskEntity
import com.example.todoapp.ui.TasksUiState
import kotlinx.coroutines.delay
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.launch

class MainViewModel(
    private val repository: TaskRepository
) : ViewModel() {

    private val _uiState = MutableStateFlow<TasksUiState>(TasksUiState.Loading)
    val uiState: StateFlow<TasksUiState> = _uiState.asStateFlow()

    init {
        loadTasks()
    }

    fun loadTasks() {
        viewModelScope.launch {
            _uiState.value = TasksUiState.Loading
            try {
                delay(2000)
                val tasks = repository.getTasksOnce()
                _uiState.value = TasksUiState.Success(tasks)
            } catch (e: Exception) {
                _uiState.value = TasksUiState.Error(e.message ?: "Ошибка загрузки")
            }
        }
    }

    fun addTask(title: String) {
        viewModelScope.launch {
            repository.addTask(title)
            loadTasks()
        }
    }

    fun deleteTask(task: TaskEntity) {
        viewModelScope.launch {
            repository.deleteTask(task)
            loadTasks()
        }
    }

    fun toggleTaskCompletion(task: TaskEntity, isCompleted: Boolean) {
        viewModelScope.launch {
            repository.toggleTaskCompletion(task, isCompleted)
            loadTasks()
        }
    }

    fun refresh() {
        loadTasks()
    }
}
```

## activity_main.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.swiperefreshlayout.widget.SwipeRefreshLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/swipeRefresh"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <androidx.recyclerview.widget.RecyclerView
            android:id="@+id/recyclerViewTasks"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:visibility="gone"/>

        <ProgressBar
            android:id="@+id/progressBar"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:visibility="gone"/>

        <TextView
            android:id="@+id/textError"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:text="Ошибка загрузки"
            android:visibility="gone"
            android:padding="16dp"
            android:gravity="center"/>

    </FrameLayout>

</androidx.swiperefreshlayout.widget.SwipeRefreshLayout>
```

## MainActivity.kt
```kotlin
package com.example.todoapp

import android.app.Activity
import android.content.Intent
import android.os.Bundle
import android.view.View
import android.widget.EditText
import android.widget.ProgressBar
import android.widget.TextView
import android.widget.Toast
import androidx.activity.result.contract.ActivityResultContracts
import androidx.activity.viewModels
import androidx.appcompat.app.AppCompatActivity
import androidx.lifecycle.Lifecycle
import androidx.lifecycle.lifecycleScope
import androidx.lifecycle.repeatOnLifecycle
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView
import androidx.swiperefreshlayout.widget.SwipeRefreshLayout
import com.example.todoapp.database.AppDatabase
import com.example.todoapp.data.repository.TaskRepositoryImpl
import com.example.todoapp.ui.TasksUiState
import com.example.todoapp.ui.theme.MainViewModel
import com.example.todoapp.ui.theme.MainViewModelFactory
import kotlinx.coroutines.launch

class MainActivity : AppCompatActivity() {

    private val database by lazy { AppDatabase.getInstance(this) }
    private val repository by lazy { TaskRepositoryImpl(database.taskDao()) }
    private val viewModel: MainViewModel by viewModels {
        MainViewModelFactory(repository)
    }
    private lateinit var adapter: TaskAdapter
    private lateinit var swipeRefresh: SwipeRefreshLayout
    private lateinit var recyclerView: RecyclerView
    private lateinit var progressBar: ProgressBar
    private lateinit var textError: TextView

    private val detailLauncher = registerForActivityResult(
        ActivityResultContracts.StartActivityForResult()
    ) { result ->
        if (result.resultCode == Activity.RESULT_OK) {
            val deletedId = result.data?.getLongExtra("deleted_task_id", -1L) ?: -1L
            if (deletedId != -1L) {
                viewModel.deleteTaskById(deletedId)
            }
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        swipeRefresh = findViewById(R.id.swipeRefresh)
        recyclerView = findViewById(R.id.recyclerViewTasks)
        progressBar = findViewById(R.id.progressBar)
        textError = findViewById(R.id.textError)
        val editTextTask = findViewById<EditText>(R.id.editTextTask)
        val buttonAddTask = findViewById<Button>(R.id.buttonAddTask)

        recyclerView.layoutManager = LinearLayoutManager(this)
        adapter = TaskAdapter(
            tasks = emptyList(),
            onItemClick = { task ->
                val intent = Intent(this, DetailActivity::class.java)
                intent.putExtra("task_text", task.title)
                intent.putExtra("task_id", task.id)
                detailLauncher.launch(intent)
            },
            onItemLongClick = { task ->
                viewModel.deleteTask(task)
                Toast.makeText(this, "Задача удалена", Toast.LENGTH_SHORT).show()
            },
            onCheckChange = { task, isChecked ->
                viewModel.toggleTaskCompletion(task, isChecked)
            }
        )
        recyclerView.adapter = adapter

        swipeRefresh.setOnRefreshListener {
            viewModel.refresh()
        }

        lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.uiState.collect { state ->
                    when (state) {
                        is TasksUiState.Loading -> {
                            recyclerView.visibility = View.GONE
                            progressBar.visibility = View.VISIBLE
                            textError.visibility = View.GONE
                            swipeRefresh.isRefreshing = false
                        }
                        is TasksUiState.Success -> {
                            recyclerView.visibility = View.VISIBLE
                            progressBar.visibility = View.GONE
                            textError.visibility = View.GONE
                            adapter.updateData(state.tasks)
                            swipeRefresh.isRefreshing = false
                        }
                        is TasksUiState.Error -> {
                            recyclerView.visibility = View.GONE
                            progressBar.visibility = View.GONE
                            textError.visibility = View.VISIBLE
                            textError.text = state.message
                            swipeRefresh.isRefreshing = false
                        }
                    }
                }
            }
        }

        buttonAddTask.setOnClickListener {
            val task = editTextTask.text.toString()
            if (task.isNotBlank()) {
                viewModel.addTask(task)
                editTextTask.text.clear()
            } else {
                Toast.makeText(this, "Введите задачу", Toast.LENGTH_SHORT).show()
            }
        }
    }
}
```

## Индивидуальное задание: Pull-to-refresh
Реализована возможность обновления списка задач свайпом вниз с использованием `SwipeRefreshLayout`. В `activity_main.xml` корневой контейнер заменён на `SwipeRefreshLayout`, внутри которого размещён `FrameLayout` с переключением между `RecyclerView`, `ProgressBar` и `TextView` ошибки. В `MainActivity` установлен слушатель `setOnRefreshListener`, вызывающий `viewModel.refresh()`. Индикатор обновления управляется через `swipeRefresh.isRefreshing = false` после получения нового состояния из `viewModel.uiState`. Пользователь может обновить список в любой момент без перезапуска приложения.



<img width="384" height="847" alt="image" src="https://github.com/user-attachments/assets/61df8ab4-1a79-400a-a80b-016d3aa1572c" />


## Ответы на контрольные вопросы
1. Длительные операции нельзя выполнять в главном потоке, потому что это блокирует отрисовку UI и обработку пользовательских событий, что приводит к зависанию приложения и ошибке `ANR` (Application Not Responding).
2. `viewModelScope` — это область корутин, привязанная к жизненному циклу `ViewModel`. Все корутины, запущенные в этой области, автоматически отменяются при очистке `ViewModel`, что предотвращает утечки памяти и фоновую работу после уничтожения экрана.
3. `sealed class` позволяет представить все возможные состояния UI в одном типе, обеспечивая типобезопасность и исчерпывающую обработку через `when`. Это исключает недопустимые комбинации состояний и упрощает тестирование.
4. Задержка имитируется функцией `delay(milliseconds)` из `kotlinx.coroutines`, которая приостанавливает выполнение корутины без блокировки потока.
5. Ошибки обрабатываются через конструкцию `try-catch` внутри корутины. При перехвате исключения состояние `UiState.Error` эмитится в `StateFlow`, что позволяет UI отобразить сообщение об ошибке.

## Вывод

В результате выполнения лабораторной работы в приложение `TodoApp` была добавлена поддержка асинхронных длительных операций с управлением состоянием загрузки и обработки ошибок.

**Изученные и применённые технологии:**
- **viewModelScope:** все длительные операции выполняются в корутинах, привязанных к жизненному циклу ViewModel, что гарантирует безопасную отмену при уничтожении экрана.
- **Sealed class для состояний:** `TasksUiState` представляет три состояния экрана (Loading, Success, Error), обеспечивая типобезопасную обработку в UI.
- **StateFlow для реактивного UI:** состояние экрана передаётся в UI через `StateFlow`, который автоматически уведомляет подписчиков об изменениях.
- **SwipeRefreshLayout:** реализовано обновление списка свайпом вниз, улучшающее пользовательский опыт.

**Архитектура и управление состоянием:**
- ViewModel инкапсулирует логику загрузки данных, симуляции задержки и обработки исключений.
- Репозиторий предоставляет метод `getTasksOnce()` для однократного получения данных из `Flow` через `first()`.
- UI реагирует на изменения `uiState` через `collect` в `lifecycleScope`, переключая видимость элементов и обновляя адаптер.
- Индикатор загрузки управляется централизованно: `ProgressBar` показывается только в состоянии `Loading`.

**Индивидуальное задание («Pull-to-refresh»):**
- В `activity_main.xml` корневой контейнер заменён на `SwipeRefreshLayout`.
- В `MainActivity` установлен обработчик `setOnRefreshListener`, вызывающий `viewModel.refresh()`.
- Состояние `isRefreshing` сбрасывается после получения нового состояния из `viewModel.uiState`.
- Пользователь может обновить список в любой момент, получая визуальную обратную связь через стандартный индикатор.

**Интерфейс и запуск:**
- При старте приложения отображается `ProgressBar` в течение 2 секунд, затем список задач.
- Свайп вниз запускает повторную загрузку с индикатором обновления.
- При ошибке (например, сбой в репозитории) отображается сообщение с описанием проблемы.
- Все операции выполняются асинхронно, UI остаётся отзывчивым.

**Итог:** Цель работы достигнута. Получены практические навыки выполнения длительных операций в `viewModelScope`, управления состоянием UI через `sealed class` и `StateFlow`, а также реализации паттерна обновления через свайп. Архитектура приложения стала более устойчивой к ошибкам и удобной для пользователя.
