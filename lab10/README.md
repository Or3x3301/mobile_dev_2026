# Лабораторная работа №10

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

Лабораторная работа №10
**«Интеграция Room в проект. Сохранение списка задач в БД»**  
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

Изучить основы работы с Room Database — официальной библиотекой для работы с SQLite в Android. Научиться создавать Entity, DAO, Database, интегрировать Room с ViewModel и корутинами, обеспечить сохранение списка задач между сессиями приложения.

## TaskEntity.kt
```kotlin
package com.example.todoapp.database

import androidx.room.Entity
import androidx.room.PrimaryKey

@Entity(tableName = "tasks")
data class TaskEntity(
    @PrimaryKey(autoGenerate = true)
    val id: Long = 0,
    val title: String,
    val isCompleted: Boolean = false,
    val createdTime: Long = System.currentTimeMillis()
)
```

## TaskDao.kt
```kotlin
package com.example.todoapp.database

import androidx.room.*
import kotlinx.coroutines.flow.Flow

@Dao
interface TaskDao {

    @Query("SELECT * FROM tasks ORDER BY isCompleted ASC, createdTime DESC")
    fun getAllTasks(): Flow<List<TaskEntity>>

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertTask(task: TaskEntity)

    @Update
    suspend fun updateTask(task: TaskEntity)

    @Delete
    suspend fun deleteTask(task: TaskEntity)

    @Query("DELETE FROM tasks")
    suspend fun deleteAll()

    @Query("SELECT * FROM tasks WHERE id = :id")
    suspend fun getTaskById(id: Long): TaskEntity?
}
```

## AppDatabase.kt
```kotlin
package com.example.todoapp.database

import android.content.Context
import androidx.room.Database
import androidx.room.Room
import androidx.room.RoomDatabase

@Database(
    entities = [TaskEntity::class],
    version = 1,
    exportSchema = false
)
abstract class AppDatabase : RoomDatabase() {
    abstract fun taskDao(): TaskDao

    companion object {
        @Volatile
        private var INSTANCE: AppDatabase? = null

        fun getInstance(context: Context): AppDatabase {
            return INSTANCE ?: synchronized(this) {
                val instance = Room.databaseBuilder(
                    context.applicationContext,
                    AppDatabase::class.java,
                    "todo_database"
                )
                    .fallbackToDestructiveMigration()
                    .build()
                INSTANCE = instance
                instance
            }
        }
    }
}
```

## MainViewModel.kt
```kotlin
package com.example.todoapp.ui.theme

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.example.todoapp.database.AppDatabase
import com.example.todoapp.database.TaskEntity
import kotlinx.coroutines.flow.SharingStarted
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.stateIn
import kotlinx.coroutines.launch

class MainViewModel(
    private val database: AppDatabase
) : ViewModel() {

    private val taskDao = database.taskDao()

    val tasks: StateFlow<List<TaskEntity>> = taskDao.getAllTasks()
        .stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5000),
            initialValue = emptyList()
        )

    fun addTask(title: String) {
        viewModelScope.launch {
            val task = TaskEntity(title = title)
            taskDao.insertTask(task)
        }
    }

    fun deleteTask(task: TaskEntity) {
        viewModelScope.launch {
            taskDao.deleteTask(task)
        }
    }

    fun updateTask(task: TaskEntity) {
        viewModelScope.launch {
            taskDao.updateTask(task)
        }
    }

    fun toggleTaskCompletion(task: TaskEntity, isCompleted: Boolean) {
        viewModelScope.launch {
            val updatedTask = task.copy(isCompleted = isCompleted)
            taskDao.updateTask(updatedTask)
        }
    }

    fun deleteAllTasks() {
        viewModelScope.launch {
            taskDao.deleteAll()
        }
    }
}
```

## MainViewModelFactory.kt
```kotlin
package com.example.todoapp

import androidx.lifecycle.ViewModel
import androidx.lifecycle.ViewModelProvider
import com.example.todoapp.database.AppDatabase
import com.example.todoapp.ui.theme.MainViewModel

class MainViewModelFactory(
    private val database: AppDatabase
) : ViewModelProvider.Factory {
    override fun <T : ViewModel> create(modelClass: Class<T>): T {
        if (modelClass.isAssignableFrom(MainViewModel::class.java)) {
            @Suppress("UNCHECKED_CAST")
            return MainViewModel(database) as T
        }
        throw IllegalArgumentException("Unknown ViewModel class")
    }
}
```

## MainActivity.kt
```kotlin
package com.example.todoapp

import android.content.Intent
import android.os.Bundle
import android.widget.Button
import android.widget.EditText
import android.widget.Toast
import androidx.activity.viewModels
import androidx.appcompat.app.AppCompatActivity
import androidx.lifecycle.Lifecycle
import androidx.lifecycle.lifecycleScope
import androidx.lifecycle.repeatOnLifecycle
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView
import com.example.todoapp.database.AppDatabase
import com.example.todoapp.ui.theme.MainViewModel
import com.example.todoapp.ui.theme.MainViewModelFactory
import kotlinx.coroutines.launch

class MainActivity : AppCompatActivity() {

    private val database by lazy { AppDatabase.getInstance(this) }
    private val viewModel: MainViewModel by viewModels {
        MainViewModelFactory(database)
    }
    private lateinit var adapter: TaskAdapter

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
                startActivity(intent)
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

## TaskAdapter.kt
```kotlin
package com.example.todoapp

import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.CheckBox
import android.widget.TextView
import androidx.recyclerview.widget.RecyclerView
import com.example.todoapp.database.TaskEntity

class TaskAdapter(
    private var tasks: List<TaskEntity>,
    private val onItemClick: (TaskEntity) -> Unit,
    private val onItemLongClick: (TaskEntity) -> Unit,
    private val onCheckChange: (TaskEntity, Boolean) -> Unit
) : RecyclerView.Adapter<TaskAdapter.TaskViewHolder>() {

    class TaskViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
        val textTask: TextView = itemView.findViewById(R.id.textTask)
        val checkTask: CheckBox = itemView.findViewById(R.id.checkTask)
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): TaskViewHolder {
        val view = LayoutInflater.from(parent.context)
            .inflate(R.layout.item_task, parent, false)
        return TaskViewHolder(view)
    }

    override fun onBindViewHolder(holder: TaskViewHolder, position: Int) {
        val task = tasks[position]
        holder.textTask.text = task.title
        holder.checkTask.isChecked = task.isCompleted

        holder.itemView.setOnClickListener {
            val pos = holder.bindingAdapterPosition
            if (pos != RecyclerView.NO_POSITION) onItemClick(tasks[pos])
        }

        holder.itemView.setOnLongClickListener {
            val pos = holder.bindingAdapterPosition
            if (pos != RecyclerView.NO_POSITION) onItemLongClick(tasks[pos])
            true
        }

        holder.checkTask.setOnCheckedChangeListener { _, isChecked ->
            onCheckChange(task, isChecked)
        }
    }

    override fun getItemCount(): Int = tasks.size

    fun updateData(newTasks: List<TaskEntity>) {
        tasks = newTasks
        notifyDataSetChanged()
    }
}
```

## Индивидуальное задание: Сортировка задач
Реализована сортировка задач в `TaskDao` через SQL-запрос: `ORDER BY isCompleted ASC, createdTime DESC`. Это обеспечивает отображение невыполненных задач (`isCompleted = false`) выше выполненных, а внутри каждой группы — задачи с более поздней датой создания отображаются первыми. Сортировка выполняется на уровне базы данных, что эффективно по производительности и не требует дополнительной обработки в ViewModel. При добавлении новой задачи она автоматически попадает на нужную позицию в списке благодаря реактивному `Flow`.


<img width="309" height="688" alt="image" src="https://github.com/user-attachments/assets/d7bb3008-93e6-4934-877d-d24522f7845d" />


## Ответы на контрольные вопросы
1. **Room** — это библиотека-обёртка над SQLite, предоставляющая абстракцию, валидацию запросов на этапе компиляции и интеграцию с корутинами/Flow. Решает проблемы шаблонного кода, ошибок в SQL и ручной работы с курсорами.
2. Три компонента: **Entity** (класс-таблица), **DAO** (интерфейс с методами доступа к данным), **Database** (абстрактный класс-точка входа, связывающий Entity и DAO).
3. Методы DAO, изменяющие данные, объявляются `suspend`, чтобы их можно было безопасно вызывать из корутин без блокировки основного потока. Room автоматически выполняет их в фоновом потоке.
4. **Flow** — это реактивный поток данных из Kotlin Coroutines. Room возвращает `Flow<List<T>>`, который автоматически эмитит новые значения при изменении таблицы, что позволяет UI реагировать на изменения без ручного обновления.
5. Room использует аннотационный процессор (`kapt`), который анализирует `@Query` на этапе компиляции и проверяет корректность SQL-синтаксиса и имён таблиц/полей. Ошибки отображаются как обычные ошибки компиляции.
6. Паттерн Singleton гарантирует, что в приложении будет только один экземпляр базы данных. Это предотвращает утечки ресурсов, конфликты транзакций и обеспечивает консистентность данных.

## Вывод

В результате выполнения лабораторной работы архитектура приложения `TodoApp` была переведена на использование локальной базы данных Room. Данные задач теперь сохраняются в SQLite и восстанавливаются между сессиями работы приложения.

**Изученные и применённые технологии:**
- **Room Database:** реализованы три основных компонента — `TaskEntity` (таблица), `TaskDao` (интерфейс доступа), `AppDatabase` (синглтон-фабрика базы данных).
- **Аннотации Room:** использованы `@Entity`, `@PrimaryKey`, `@Dao`, `@Query`, `@Insert`, `@Update`, `@Delete` для декларативного описания схемы и операций.
- **Корутины и Flow:** методы DAO объявлены как `suspend`, запросы возвращают `Flow<List<TaskEntity>>`, что обеспечивает асинхронную реактивную работу без блокировки UI.
- **Интеграция с ViewModel:** `MainViewModel` получает экземпляр `AppDatabase` через фабрику, подписывается на `Flow` через `stateIn()`, UI обновляется автоматически при изменении данных.

**Архитектура и сохранение состояния:**
- Данные хранятся в файле `todo_database` в приватной директории приложения.
- При каждом запуске `AppDatabase.getInstance()` возвращает один и тот же экземпляр (Singleton), что гарантирует консистентность.
- `fallbackToDestructiveMigration()` позволяет безопасно пересоздавать таблицу при изменении схемы во время разработки.
- Все операции записи выполняются в `viewModelScope`, что обеспечивает отмену при уничтожении ViewModel и предотвращает утечки.

**Индивидуальное задание («Сортировка задач»):**
- В `TaskDao` реализован SQL-запрос с двойной сортировкой: `ORDER BY isCompleted ASC, createdTime DESC`.
- Невыполненные задачи (`false < true`) автоматически отображаются выше выполненных.
- Внутри групп задачи сортируются по убыванию времени создания — новые сверху.
- Сортировка выполняется на уровне СУБД, что эффективно и не требует дополнительной логики в Kotlin-коде.

**Интерфейс и запуск:**
- `TaskAdapter` обновлён для работы с `TaskEntity`: отображает заголовок, чекбокс, корректно обрабатывает клики через `bindingAdapterPosition`.
- `MainActivity` инициализирует базу данных через `by lazy`, передаёт её в фабрику ViewModel.
- Приложение успешно компилируется, задачи добавляются/удаляются/обновляются с мгновенным отображением, данные сохраняются после перезапуска, сортировка работает корректно.

**Итог:** Цель работы достигнута. Получены практические навыки интеграции Room в Android-приложение, работы с аннотациями, корутинами и реактивными потоками. Реализованный код соответствует современным рекомендациям по архитектуре (MVVM + Repository-подобный подход через DAO) и готов к расширению функционала.
