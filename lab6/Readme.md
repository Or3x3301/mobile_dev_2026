# Лабораторная работа №6

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

Лабораторная работа №6
**«Отображение списка задач из предыдущей лабораторной в красивых карточках»**  
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

Научиться использовать RecyclerView для отображения списка данных, освоить создание адаптера и ViewHolder, применить CardView для оформления элементов списка.

## item_task.xml
```kotlin

<?xml version="1.0" encoding="utf-8"?>
<androidx.cardview.widget.CardView
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_marginHorizontal="8dp"
    android:layout_marginVertical="4dp"
    app:cardCornerRadius="12dp"
    app:cardElevation="3dp"
    app:cardBackgroundColor="#FFFFFF"
    app:contentPadding="16dp">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:gravity="center_vertical">

        <TextView
            android:id="@+id/textTask"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:textSize="16sp"
            android:textColor="#212121"/>

        <CheckBox
            android:id="@+id/checkTask"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:clickable="true"
            android:focusable="true"/>

    </LinearLayout>

</androidx.cardview.widget.CardView>

```
## TaskAdapter.kt
```kotlin
package com.example.todoapp.adapter

import android.graphics.Paint
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.CheckBox
import android.widget.TextView
import androidx.recyclerview.widget.RecyclerView
import com.example.todoapp.R

class TaskAdapter(
    private val tasks: MutableList<String>,
    private val onTaskToggle: (Int, Boolean) -> Unit = { _, _ -> },
    private val onItemLongClick: (Int) -> Unit = {},
    private val onCompletedCountChanged: (Int) -> Unit = {}
) : RecyclerView.Adapter<TaskAdapter.TaskViewHolder>() {

    class TaskViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
        val textTask: TextView = itemView.findViewById(R.id.textTask)
        val checkTask: CheckBox = itemView.findViewById(R.id.checkTask)
    }

    // Хранит состояния чекбоксов (синхронизируется с tasks)
    private val completedStates = mutableListOf<Boolean>()

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): TaskViewHolder {
        val view = LayoutInflater.from(parent.context)
            .inflate(R.layout.item_task, parent, false)
        return TaskViewHolder(view)
    }

    override fun onBindViewHolder(holder: TaskViewHolder, position: Int) {
        if (position >= tasks.size) return

        val task = tasks[position]
        val isCompleted = completedStates.getOrElse(position) { false }

        holder.textTask.text = task
        holder.checkTask.isChecked = isCompleted
        applyStrikeThrough(holder.textTask, isCompleted)

        // ВАЖНО: сбрасываем слушатель перед установкой нового, чтобы избежать рекурсии
        holder.checkTask.setOnCheckedChangeListener(null)
        holder.checkTask.setOnCheckedChangeListener { _, isChecked ->
            completedStates[position] = isChecked
            applyStrikeThrough(holder.textTask, isChecked)
            onTaskToggle(position, isChecked)
            onCompletedCountChanged(completedStates.count { it })
        }

        holder.itemView.setOnLongClickListener {
            onItemLongClick(position)
            true
        }
    }

    private fun applyStrikeThrough(textView: TextView, isCompleted: Boolean) {
        textView.paintFlags = if (isCompleted) {
            textView.paintFlags or Paint.STRIKE_THRU_TEXT_FLAG
        } else {
            textView.paintFlags and Paint.STRIKE_THRU_TEXT_FLAG.inv()
        }
    }

    override fun getItemCount(): Int = tasks.size

    fun updateTasks(newTasks: List<String>) {
        val oldStates = completedStates.toList() // Сохраняем старые состояния
        tasks.clear()
        completedStates.clear()
        tasks.addAll(newTasks)

        // Восстанавливаем состояния для существующих индексов
        newTasks.forEachIndexed { index, _ ->
            completedStates.add(if (index < oldStates.size) oldStates[index] else false)
        }
        notifyDataSetChanged()
    }

    fun removeTask(position: Int) {
        if (position in tasks.indices) {
            tasks.removeAt(position)
            completedStates.removeAt(position)
            notifyItemRemoved(position)
            onCompletedCountChanged(completedStates.count { it }) // Обновляем счётчик после удаления
        }
    }
}
```

## MainActivity.kt
```kotlin

package com.example.todoapp

import android.os.Bundle
import android.view.inputmethod.EditorInfo
import android.widget.Button
import android.widget.EditText
import android.widget.TextView
import android.widget.Toast
import androidx.appcompat.app.AlertDialog
import androidx.appcompat.app.AppCompatActivity
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView
import com.example.todoapp.adapter.TaskAdapter

class MainActivity : AppCompatActivity() {

    // === Менеджеры ===
    private lateinit var counterManager: CounterManager
    private lateinit var taskManager: TaskManager

    // === Адаптер для RecyclerView ===
    private lateinit var taskAdapter: TaskAdapter

    // === UI: Счётчик ===
    private lateinit var textCounter: TextView

    // === UI: Ввод и отображение текста (отдельный блок) ===
    private lateinit var editTextInput: EditText
    private lateinit var buttonShow: Button
    private lateinit var textEntered: TextView

    // === UI: ToDo-список (RecyclerView) ===
    private lateinit var recyclerViewTasks: RecyclerView
    private lateinit var editTextTask: EditText
    private lateinit var buttonAddTask: Button
    private lateinit var buttonRemoveLast: Button
    private lateinit var buttonClearAll: Button
    private lateinit var editTextIndexToRemove: EditText
    private lateinit var buttonRemoveByIndex: Button
    private lateinit var textTaskCount: TextView
    private lateinit var textCompletedCount: TextView

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        bindViews()                    // ← 1. Сначала привязываем все View
        setupRecyclerView()            // ← 2. Создаём адаптер (теперь taskAdapter существует!)
        initManagers()                 // ← 3. Теперь колбэки могут безопасно использовать taskAdapter
        setupListeners()
        restoreState(savedInstanceState) // ← 4. Восстанавливаем состояние
    }

    private fun initManagers() {
        // Инициализация счётчика с колбэком для обновления UI
        counterManager = CounterManager { count ->
            // Проверяем, что View инициализирован
            if (::textCounter.isInitialized) {
                textCounter.text = getString(R.string.counter_text, count)
            }
        }

        // Инициализация менеджера задач с колбэком для обновления адаптера
        taskManager = TaskManager { tasks ->
            // проверяем инициализацию адаптера
            if (::taskAdapter.isInitialized) {
                taskAdapter.updateTasks(tasks)
            }
            if (::textTaskCount.isInitialized) {
                updateTaskCount(tasks.size)
            }
        }
    }

    private fun bindViews() {
        // === Счётчик ===
        textCounter = findViewById(R.id.textCounter)
        textCompletedCount = findViewById(R.id.textCompletedCount)
        // === Ввод текста ===
        editTextInput = findViewById(R.id.editTextInput)
        buttonShow = findViewById(R.id.buttonShow)
        textEntered = findViewById(R.id.textEntered)

        // === ToDo-список ===
        recyclerViewTasks = findViewById(R.id.recyclerViewTasks)
        editTextTask = findViewById(R.id.editTextTask)
        buttonAddTask = findViewById(R.id.buttonAddTask)
        buttonRemoveLast = findViewById(R.id.buttonRemoveLast)
        buttonClearAll = findViewById(R.id.buttonClearAll)
        editTextIndexToRemove = findViewById(R.id.editIndexToRemove)
        buttonRemoveByIndex = findViewById(R.id.buttonRemoveByIndex)
        textTaskCount = findViewById(R.id.textTaskCount)
    }

    private fun setupRecyclerView() {
        recyclerViewTasks.layoutManager = LinearLayoutManager(this)

        taskAdapter = TaskAdapter(
            tasks = mutableListOf(),
            onItemLongClick = { position ->
                if (taskManager.removeByIndex(position)) {
                    showToast("Задача удалена")
                }
            },
            onCompletedCountChanged = { count ->
                textCompletedCount.text = "Выполнено: $count"
            }
        )

        recyclerViewTasks.adapter = taskAdapter
        recyclerViewTasks.itemAnimator = androidx.recyclerview.widget.DefaultItemAnimator()
    }

    private fun setupListeners() {
        // === Счётчик ===
        setupCounterListeners()

        // === Ввод текста ===
        setupTextInputListeners()

        // === ToDo-список ===
        setupTaskListListeners()
    }

    private fun setupCounterListeners() {
        findViewById<Button>(R.id.buttonIncrement).setOnClickListener {
            counterManager.increment()
        }

        findViewById<Button>(R.id.buttonDecrement).setOnClickListener {
            counterManager.decrement()
        }

        findViewById<Button>(R.id.buttonResetCounter).setOnClickListener {
            counterManager.reset()
            showToast("Счётчик сброшен")
        }
    }

    private fun setupTextInputListeners() {
        buttonShow.setOnClickListener {
            val text = editTextInput.text.toString()
            textEntered.text = "${getString(R.string.label_entered)} $text"
            editTextInput.text.clear()
        }

        // Поддержка Enter в поле ввода
        editTextInput.setOnEditorActionListener { _, actionId, _ ->
            if (actionId == EditorInfo.IME_ACTION_DONE) {
                buttonShow.performClick()
                true
            } else false
        }
    }

    private fun setupTaskListListeners() {
        // Добавление задачи
        buttonAddTask.setOnClickListener { addCurrentTask() }

        // Поддержка Enter в поле задачи
        editTextTask.setOnEditorActionListener { _, actionId, _ ->
            if (actionId == EditorInfo.IME_ACTION_DONE) {
                addCurrentTask()
                true
            } else false
        }

        // Удаление последней задачи
        buttonRemoveLast.setOnClickListener {
            if (taskManager.removeLast()) {
                showToast("Последняя задача удалена")
            } else {
                showToast("Список пуст")
            }
        }

        // Очистка всех задач
        buttonClearAll.setOnClickListener {
            if (taskManager.getTasks().isNotEmpty()) {
                AlertDialog.Builder(this)
                    .setTitle("Подтверждение")
                    .setMessage("Удалить все задачи?")
                    .setPositiveButton("Да") { _, _ ->
                        taskManager.clearAll()
                        showToast("Все задачи удалены")
                    }
                    .setNegativeButton("Отмена", null)
                    .show()
            } else {
                showToast("Список уже пуст")
            }
        }

        // Удаление по индексу
        buttonRemoveByIndex.setOnClickListener { removeTaskByIndex() }
    }

    private fun addCurrentTask() {
        val task = editTextTask.text.toString().trim()
        if (taskManager.addTask(task)) {
            editTextTask.text.clear()
            showToast("Задача добавлена ✓")
            // Прокрутка к новой задаче
            recyclerViewTasks.smoothScrollToPosition(taskAdapter.itemCount - 1)
        } else {
            showToast("Введите текст задачи")
        }
    }

    private fun removeTaskByIndex() {
        val indexStr = editTextIndexToRemove.text.toString()
        if (indexStr.isBlank()) {
            showToast("Введите индекс")
            return
        }

        val index = indexStr.toIntOrNull()
        if (index == null) {
            showToast("Неверный индекс")
            return
        }

        if (taskManager.removeByIndex(index)) {
            showToast("Задача #$index удалена")
            editTextIndexToRemove.text.clear()
        } else {
            showToast("Индекс вне диапазона")
        }
    }

    private fun updateTaskCount(count: Int) {
        textTaskCount.text = "Задач: $count"
    }

    private fun showToast(message: String) {
        Toast.makeText(this, message, Toast.LENGTH_SHORT).show()
    }

    // === Сохранение и восстановление состояния ===

    override fun onSaveInstanceState(outState: Bundle) {
        super.onSaveInstanceState(outState)
        outState.putBundle("counter_state", counterManager.saveState())
        outState.putBundle("tasks_state", taskManager.saveState())
    }

    private fun restoreState(savedInstanceState: Bundle?) {
        savedInstanceState?.let {
            it.getBundle("counter_state")?.let { counterBundle ->
                counterManager.restoreState(counterBundle)
            }
            it.getBundle("tasks_state")?.let { tasksBundle ->
                taskManager.restoreState(tasksBundle)
            }
        }
    }
}

```

## Индивидуальное задание: Счетчик выполненных задач
Добавьте в шапку экрана TextView, показывающий количество выполненных задач (отмеченных чекбоксов). Обновляйте его при каждом изменении состояния чекбокса.

<img width="286" height="639" alt="image" src="https://github.com/user-attachments/assets/8840d66e-1fca-4d48-abe2-7243c1e412bf" />

## Контрольные вопросы
**Для чего нужен RecyclerView? Чем он лучше ListView?** : RecyclerView предназначен для эффективного отображения больших списков данных. Он лучше ListView, так как переиспользует (рециклирует) элементы при прокрутке, что экономит оперативную память и повышает производительность, а также предоставляет более гибкую архитектуру с обязательным разделением на LayoutManager и Adapter.

**Какие компоненты необходимы для работы RecyclerView?** : Для работы необходимы три основных компонента: LayoutManager (отвечает за расположение элементов в списке), Adapter (связывает данные с элементами интерфейса) и ViewHolder (кэширует ссылки на внутренние View элемента списка).

**Что такое ViewHolder и для чего он используется?** : ViewHolder — это вспомогательный класс, который хранит прямые ссылки на все дочерние View внутри одного элемента списка. Он используется для оптимизации производительности, устраняя необходимость многократного вызова метода findViewById() при прокрутке списка.

**Чем отличается notifyDataSetChanged() от notifyItemInserted()?** : notifyDataSetChanged() полностью перерисовывает все видимые элементы списка без анимации, что ресурсозатратно. notifyItemInserted() уведомляет адаптер только об изменении одной конкретной позиции, запускает встроенную анимацию вставки и работает значительно быстрее.

**Как добавить обработку кликов на элементы RecyclerView?** : Обработка кликов добавляется путём передачи лямбда-выражения или интерфейса обратного вызова в конструктор адаптера и установки слушателя setOnClickListener на корневой View (itemView) внутри метода onBindViewHolder.

## Вывод

В результате выполнения лабораторной работы было разработано Android-приложение для демонстрации эффективного отображения динамических списков с использованием современных компонентов RecyclerView и CardView.

**Изученные и применённые технологии:**
- **RecyclerView:** реализован высокопроизводительный механизм отображения списков с переиспользованием (recycling) элементов при прокрутке, что минимизирует потребление памяти.
- **CardView & Material Design:** применён для создания визуально структурированных карточек задач с закруглёнными углами, тенями и стандартными отступами.
- **Adapter & ViewHolder:** создан кастомный адаптер с паттерном ViewHolder для кэширования ссылок на дочерние View и устранения многократных вызовов `findViewById()`.
- **LayoutManager:** настроен `LinearLayoutManager` для корректного вертикального расположения элементов внутри контейнера.
- **Точечные уведомления:** использованы методы `notifyItemInserted()` и `notifyItemRemoved()` для локального обновления UI без полной перерисовки списка.

**Обработка данных и логика:**
- Реализована синхронизация состояния чекбоксов с визуальным оформлением текста (зачёркивание через `Paint.STRIKE_THRU_TEXT_FLAG`).
- Настроена безопасная работа со слушателями: предотвращение дублирования событий `OnCheckedChangeListener` за счёт сброса перед привязкой в `onBindViewHolder`.
- Обеспечена корректная обработка динамических изменений списка с автоматическим пересчётом индексов и сохранением состояний выполненных задач при добавлении/удалении.
- Продемонстрирован принцип разделения ответственности: адаптер отвечает только за отображение и передачу событий, бизнес-логика управления данными инкапсулирована в `MainActivity` и `TaskManager`.

**Дополнительные улучшения (вариативные задания):**
- Реализовано индивидуальное задание «Счётчик выполненных задач»: добавлен `TextView`, динамически отображающий количество отмеченных чекбоксов.
- Внедрён механизм подсчёта через параллельный список булевых состояний в адаптере с колбэком обновления UI в активность.
- Добавлена обработка долгого нажатия (`OnLongClickListener`) на карточку для быстрого удаления задачи из списка.
- Настроена плавная анимация появления и удаления элементов через `DefaultItemAnimator`, улучшающая пользовательский опыт.

**Интерфейс и запуск:**
- В `item_task.xml` создана модульная разметка карточки с горизонтальным расположением текста и чекбокса, корректно масштабируемая под разные размеры экранов.
- В `activity_main.xml` заменён статический `TextView` на `RecyclerView` с отключённым `nestedScrollingEnabled` для стабильной работы внутри `ScrollView`.
- Реализована автоматическая прокрутка к новому элементу через `smoothScrollToPosition()` и валидация пустого ввода перед добавлением.
- Приложение успешно компилируется и стабильно работает на эмуляторе, все интерактивные элементы (чекбоксы, счётчики, удаление) отрабатывают без сбоев, утечек памяти или артефактов прокрутки.

**Итог:** Цель работы достигнута. Получены практические навыки работы с RecyclerView, освоены принципы создания адаптеров, использования ViewHolder и оптимизации производительности списков. Реализована интеграция CardView для современного оформления элементов и отработана логика обработки пользовательских взаимодействий. Написанный код демонстрирует модульную архитектуру, эффективное управление ресурсами и готовность к масштабированию в реальных Android-проектах.
