# Лабораторная работа №8

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

Лабораторная работа №8
**«Перенос логики списка задач из Activity в ViewModel. Использование StateFlow для хранения состояния»**  
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

Изучить архитектурный компонент `ViewModel`, научиться выносить логику и состояние UI из `Activity`, использовать `StateFlow` для реактивного обновления данных, обеспечить сохранение состояния при изменении конфигурации экрана.

## MainViewModel.kt
```kotlin
package com.example.todoapp

import androidx.lifecycle.ViewModel
import kotlinx.coroutines.flow.MutableSharedFlow
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.SharedFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asSharedFlow
import kotlinx.coroutines.flow.asStateFlow

class MainViewModel : ViewModel() {
    private val _tasks = MutableStateFlow<List<String>>(emptyList())
    val tasks: StateFlow<List<String>> = _tasks.asStateFlow()

    // SharedFlow для одноразовых UI-событий (Индивидуальное задание)
    private val _notificationFlow = MutableSharedFlow<String>(extraBufferCapacity = 1)
    val notificationFlow: SharedFlow<String> = _notificationFlow.asSharedFlow()

    fun addTask(task: String) {
        if (task.isBlank()) {
            _notificationFlow.tryEmit("Ошибка: задача не может быть пустой")
            return
        }
        val currentList = _tasks.value.toMutableList()
        currentList.add(task)
        _tasks.value = currentList
        _notificationFlow.tryEmit("Задача добавлена")
    }

    fun deleteTask(index: Int) {
        val currentList = _tasks.value.toMutableList()
        if (index in currentList.indices) {
            currentList.removeAt(index)
            _tasks.value = currentList
            _notificationFlow.tryEmit("Задача удалена")
        }
    }

    fun updateTask(index: Int, newText: String) {
        val currentList = _tasks.value.toMutableList()
        if (index in currentList.indices) {
            currentList[index] = newText
            _tasks.value = currentList
            _notificationFlow.tryEmit("Задача обновлена")
        }
    }

    fun loadTestData() {
        _tasks.value = listOf(
            "Купить продукты",
            "Сделать ДЗ по Android",
            "Позвонить маме",
            "Записаться к врачу"
        )
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
import kotlinx.coroutines.launch

class MainActivity : AppCompatActivity() {

    private val viewModel: MainViewModel by viewModels()
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
            onItemClick = { position ->
                val taskText = viewModel.tasks.value[position]
                val intent = Intent(this, DetailActivity::class.java)
                intent.putExtra("task_text", taskText)
                startActivity(intent)
            },
            onItemLongClick = { position ->
                viewModel.deleteTask(position)
            }
        )
        recyclerView.adapter = adapter

        // Подписка на StateFlow списка задач
        lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.tasks.collect { tasks ->
                    adapter.updateData(tasks)
                }
            }
        }

        // Подписка на SharedFlow уведомлений
        lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.notificationFlow.collect { message ->
                    Toast.makeText(this@MainActivity, message, Toast.LENGTH_SHORT).show()
                }
            }
        }

        buttonAddTask.setOnClickListener {
            val task = editTextTask.text.toString()
            viewModel.addTask(task)
            if (task.isNotBlank()) editTextTask.text.clear()
        }

        if (viewModel.tasks.value.isEmpty()) {
            viewModel.loadTestData()
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

class TaskAdapter(
    private var tasks: List<String>,
    private val onItemClick: (Int) -> Unit,
    private val onItemLongClick: (Int) -> Unit
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
        holder.textTask.text = task

        holder.itemView.setOnClickListener { onItemClick(position) }
        holder.itemView.setOnLongClickListener {
            onItemLongClick(position)
            true
        }

        holder.checkTask.setOnCheckedChangeListener { _, isChecked ->
            if (isChecked) {
                holder.textTask.paintFlags = holder.textTask.paintFlags or android.graphics.Paint.STRIKE_THRU_TEXT_FLAG
            } else {
                holder.textTask.paintFlags = holder.textTask.paintFlags and android.graphics.Paint.STRIKE_THRU_TEXT_FLAG.inv()
            }
        }
    }

    override fun getItemCount(): Int = tasks.size

    fun updateData(newTasks: List<String>) {
        tasks = newTasks
        notifyDataSetChanged()
    }
}
```

## Индивидуальное задание: SharedFlow для уведомлений
В `MainViewModel` добавлен `MutableSharedFlow<String>` с `extraBufferCapacity = 1` для отправки одноразовых событий. Все сообщения об успехе или ошибке («Задача добавлена», «Ошибка: задача не может быть пустой», «Задача удалена») эмитируются через `tryEmit()`. В `MainActivity` реализована подписка через `lifecycleScope.launch` + `repeatOnLifecycle(Lifecycle.State.STARTED)`, которая безопасно показывает `Toast` только когда активность находится в активном состоянии. Прямые вызовы `Toast` из UI-слоя удалены, логика уведомлений полностью инкапсулирована в ViewModel.


<img width="390" height="853" alt="image" src="https://github.com/user-attachments/assets/5cb6da63-66cd-4963-98f7-6f6fb68e1f51" />

## Ответы на контрольные вопросы
1. `ViewModel` хранит данные, связанные с UI, и переживает изменения конфигурации (поворот экрана, смена языка). Это предотвращает потерю состояния и избавляет от необходимости сохранять данные вручную через `onSaveInstanceState`.
2. `StateFlow` – это hot-поток из Kotlin Coroutines, который всегда хранит последнее значение и требует явного указания начального значения. Отличается от `LiveData` нативной поддержкой корутин, потоковых операторов и отсутствием привязки к Android-жизненному циклу на уровне API. Предпочтителен в современных Kotlin-first проектах.
3. `lifecycleScope` предоставляет `CoroutineScope`, привязанный к жизненному циклу `Activity`/`Fragment`. `repeatOnLifecycle` безопасно запускает и останавливает сбор потока при переходе между состояниями (например, `STARTED`), предотвращая утечки и фоновую работу UI.
4. Данные в `StateFlow` обновляются присваиванием нового значения свойству `value` изменяемого потока: `_tasks.value = newList`. Для списков рекомендуется создавать новую копию (`toMutableList()`), чтобы триггернуть уведомление подписчиков.
5. Вынос логики в `ViewModel` устраняет зависимость от Android-компонентов (`Activity`, `Context`, `View`). Это позволяет писать чистые unit-тесты на JVM без эмулятора, проверять бизнес-логику изолированно и упрощает рефакторинг.

## Вывод

В результате выполнения лабораторной работы архитектура приложения `TodoApp` была переведена на современный подход с использованием `ViewModel` и Kotlin Flow. Логика управления данными полностью отделена от UI-слоя, обеспечено автоматическое сохранение состояния при повороте экрана.

**Изученные и применённые технологии:**
- **ViewModel:** создан класс `MainViewModel`, инкапсулирующий список задач и операции добавления/удаления. Использован делегат `by viewModels()` для корректного жизненного цикла.
- **StateFlow:** реализован паттерн приватного `_tasks: MutableStateFlow` и публичного `tasks: StateFlow`. UI подписывается на поток и реактивно обновляет адаптер.
- **Coroutines & Lifecycle:** применены `lifecycleScope` и `repeatOnLifecycle(Lifecycle.State.STARTED)` для безопасного сбора потоков без утечек памяти и фоновых обновлений UI.
- **Рефакторинг адаптера:** `TaskAdapter` переведён на работу с неизменяемым `List<String>` и методом `updateData()`, что соответствует принципу однонаправленного потока данных.

**Индивидуальное задание («SharedFlow для уведомлений»):**
- В `MainViewModel` внедрён `MutableSharedFlow<String>` с буфером для мгновенной доставки событий.
- Валидация ввода и генерация сообщений перенесены в ViewModel. UI-слой только отображает результат.
- Подписка на `notificationFlow` реализована через отдельный `lifecycleScope.launch`, гарантирующий показ `Toast` только в активном состоянии экрана.
- Достигнута полная изоляция UI-событий от логики данных, код стал тестируемым и предсказуемым.

**Интерфейс и запуск:**
- `MainActivity.kt` очищен от прямой манипуляции списком и ручных `Toast`. Все действия делегированы в `viewModel`.
- При повороте экрана `ViewModel` не пересоздаётся, `StateFlow` сохраняет последний список, адаптер мгновенно восстанавливает отображение.
- Приложение успешно компилируется, тестовые данные загружаются при первом старте, добавление и удаление задач работает реактивно, уведомления отображаются корректно.

**Итог:** Цель работы достигнута. Получены практические навыки построения архитектуры Android-приложений с разделением ответственности, использования `StateFlow`/`SharedFlow` для управления состоянием и событиями, а также безопасной работы с корутинами в жизненном цикле `Activity`. Реализованный код соответствует современным рекомендациям Google и готов к масштабированию.
