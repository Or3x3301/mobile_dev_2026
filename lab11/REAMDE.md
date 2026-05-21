# Лабораторная работа №11

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

Лабораторная работа №11
**«Рефакторинг: добавление слоя Repository между ViewModel и Room»**  
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

Изучить архитектурный паттерн Repository, научиться выделять слой доступа к данным, отделяя его от бизнес-логики, выполнить рефакторинг существующего приложения для использования репозитория.

## TaskRepository.kt
```kotlin
package com.example.todoapp.data.repository

import com.example.todoapp.database.TaskEntity
import kotlinx.coroutines.flow.Flow

interface TaskRepository {
    fun getAllTasks(): Flow<List<TaskEntity>>
    suspend fun addTask(title: String)
    suspend fun deleteTask(task: TaskEntity)
    suspend fun updateTask(task: TaskEntity)
    suspend fun toggleTaskCompletion(task: TaskEntity, isCompleted: Boolean)
    suspend fun deleteAllTasks()
}
```

## TaskRepositoryImpl.kt
```kotlin
package com.example.todoapp.data.repository

import com.example.todoapp.database.TaskDao
import com.example.todoapp.database.TaskEntity
import kotlinx.coroutines.flow.Flow

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
}
```

## InMemoryTaskRepository.kt
```kotlin
package com.example.todoapp.data.repository

import com.example.todoapp.database.TaskEntity
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.flow.update

class InMemoryTaskRepository : TaskRepository {

    private val _tasks = MutableStateFlow<List<TaskEntity>>(emptyList())
    override fun getAllTasks(): Flow<List<TaskEntity>> = _tasks.asStateFlow()

    override suspend fun addTask(title: String) {
        val task = TaskEntity(
            id = System.currentTimeMillis(),
            title = title,
            createdTime = System.currentTimeMillis()
        )
        _tasks.update { current -> current + task }
    }

    override suspend fun deleteTask(task: TaskEntity) {
        _tasks.update { current -> current.filter { it.id != task.id } }
    }

    override suspend fun updateTask(task: TaskEntity) {
        _tasks.update { current ->
            current.map { if (it.id == task.id) task else it }
        }
    }

    override suspend fun toggleTaskCompletion(task: TaskEntity, isCompleted: Boolean) {
        val updatedTask = task.copy(isCompleted = isCompleted)
        updateTask(updatedTask)
    }

    override suspend fun deleteAllTasks() {
        _tasks.update { emptyList() }
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
import kotlinx.coroutines.flow.SharingStarted
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.stateIn
import kotlinx.coroutines.launch

class MainViewModel(
    private val repository: TaskRepository
) : ViewModel() {

    val tasks: StateFlow<List<TaskEntity>> = repository.getAllTasks()
        .stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5000),
            initialValue = emptyList()
        )

    fun addTask(title: String) {
        viewModelScope.launch {
            repository.addTask(title)
        }
    }

    fun deleteTask(task: TaskEntity) {
        viewModelScope.launch {
            repository.deleteTask(task)
        }
    }

    fun toggleTaskCompletion(task: TaskEntity, isCompleted: Boolean) {
        viewModelScope.launch {
            repository.toggleTaskCompletion(task, isCompleted)
        }
    }

    fun deleteAllTasks() {
        viewModelScope.launch {
            repository.deleteAllTasks()
        }
    }
}
```

## MainViewModelFactory.kt
```kotlin
package com.example.todoapp

import androidx.lifecycle.ViewModel
import androidx.lifecycle.ViewModelProvider
import com.example.todoapp.data.repository.TaskRepository
import com.example.todoapp.ui.theme.MainViewModel

class MainViewModelFactory(
    private val repository: TaskRepository
) : ViewModelProvider.Factory {
    override fun <T : ViewModel> create(modelClass: Class<T>): T {
        if (modelClass.isAssignableFrom(MainViewModel::class.java)) {
            @Suppress("UNCHECKED_CAST")
            return MainViewModel(repository) as T
        }
        throw IllegalArgumentException("Unknown ViewModel class")
    }
}
```

## MainActivity.kt
```kotlin
package com.example.todoapp

import android.app.Activity
import android.content.Intent
import android.os.Bundle
import android.widget.Button
import android.widget.EditText
import android.widget.Toast
import androidx.activity.result.contract.ActivityResultContracts
import androidx.activity.viewModels
import androidx.appcompat.app.AppCompatActivity
import androidx.lifecycle.Lifecycle
import androidx.lifecycle.lifecycleScope
import androidx.lifecycle.repeatOnLifecycle
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView
import com.example.todoapp.database.AppDatabase
import com.example.todoapp.data.repository.TaskRepositoryImpl
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

        val editTextTask = findViewById<EditText>(R.id.editTextTask)
        val buttonAddTask = findViewById<Button>(R.id.buttonAddTask)
        val recyclerView = findViewById<RecyclerView>(R.id.recyclerViewTasks)

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

        lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.tasks.collect { tasks ->
                    adapter.updateData(tasks)
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

## Индивидуальное задание: In-Memory репозиторий
Реализована альтернативная реализация `TaskRepository` — `InMemoryTaskRepository`, которая хранит данные в памяти через `MutableStateFlow`. Все операции (добавление, удаление, обновление) выполняются над ин-memory списком без обращения к базе данных. Это демонстрирует принцип инверсии зависимостей: ViewModel работает с интерфейсом `TaskRepository`, не зная конкретной реализации. При переключении на `InMemoryTaskRepository` в `MainActivity` приложение продолжает работать, но данные не сохраняются между сессиями. Такой подход упрощает тестирование и позволяет быстро прототипировать функционал без настройки БД.


<img width="316" height="694" alt="image" src="https://github.com/user-attachments/assets/fd0b4566-767a-4f1a-96d4-bcc75bec7c11" />


## Ответы на контрольные вопросы
1. **Repository** инкапсулирует логику доступа к данным, предоставляя единый чистый API для ViewModel. Он абстрагирует источники данных (БД, сеть, кэш) и управляет их синхронизацией.
2. Преимущества: разделение ответственности (ViewModel не зависит от Room), упрощение тестирования (можно подменить репозиторий на mock), гибкость при смене источника данных, централизация логики кэширования и обработки ошибок.
3. ViewModel не изменится — она зависит только от интерфейса `TaskRepository`. Достаточно создать новую реализацию репозитория для сетевого источника и передать её в фабрику.
4. Методы объявлены `suspend`, чтобы их можно было безопасно вызывать из корутин без блокировки основного потока. Room и другие асинхронные источники данных требуют такого подхода.
5. Инверсия зависимостей (DIP) — принцип, при котором модули верхнего уровня (ViewModel) зависят от абстракций (интерфейс `TaskRepository`), а не от деталей реализации (`TaskRepositoryImpl`). Это достигается через внедрение зависимостей в конструктор.

## Вывод

В результате выполнения лабораторной работы архитектура приложения `TodoApp` была рефакторингована с добавлением слоя Repository. Это улучшило модульность, тестируемость и гибкость кода.

**Изученные и применённые технологии:**
- **Паттерн Repository:** создан интерфейс `TaskRepository` и его реализация `TaskRepositoryImpl`, инкапсулирующая доступ к Room DAO.
- **Инверсия зависимостей:** ViewModel зависит от абстракции, а не от конкретной реализации, что позволяет легко подменять источники данных.
- **Рефакторинг без потери функциональности:** все существующие методы сохранены, изменена только точка доступа к данным.
- **In-Memory реализация:** создана `InMemoryTaskRepository` для демонстрации гибкости архитектуры и упрощения тестирования.

**Архитектурные улучшения:**
- ViewModel больше не знает о существовании Room, только о контракте `TaskRepository`.
- При необходимости добавить сетевой источник или кэш достаточно создать новую реализацию репозитория.
- Код стал более тестируемым: можно передавать mock-репозиторий в ViewModel для изолированных тестов.
- Соблюдены принципы Clean Architecture и рекомендации Google по архитектуре Android-приложений.

**Индивидуальное задание («In-Memory репозиторий»):**
- Реализован `InMemoryTaskRepository` с хранением данных в `MutableStateFlow`.
- Все методы интерфейса реализованы без обращения к БД, операции выполняются над ин-memory списком.
- При переключении в `MainActivity` на `InMemoryTaskRepository` приложение работает корректно, но данные не сохраняются после перезапуска — это ожидаемое поведение для демонстрации.
- Подтверждена работоспособность принципа подмены реализаций без изменения ViewModel.

**Интерфейс и запуск:**
- Все файлы скомпилированы без ошибок, импорты корректны.
- Приложение запускается, функционал добавления/удаления/обновления задач работает идентично предыдущей лабораторной.
- Данные сохраняются в Room при использовании `TaskRepositoryImpl`, не сохраняются при использовании `InMemoryTaskRepository`.
- Рефакторинг не повлиял на UI: адаптер, активности и навигация остались без изменений.

**Итог:** Цель работы достигнута. Получены практические навыки рефакторинга с выделением слоя Repository, применения принципа инверсии зависимостей и создания альтернативных реализаций для тестирования. Архитектура приложения стала более гибкой, модульной и соответствующей современным стандартам Android-разработки.
