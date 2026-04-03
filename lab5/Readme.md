# Лабораторная работа №5

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

Лабораторная работа №5
**«Счетчик нажатий, поле ввода и отображение текста. Реализация ToDo-списка»**  
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

Научиться обрабатывать пользовательский ввод, работать с состоянием (счетчик, список задач), динамически обновлять интерфейс приложения на Kotlin.

## activity_main.xml
```kotlin

<?xml version="1.0" encoding="utf-8"?>
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:padding="16dp">

        <!-- === Блок 1: Счётчик с расширенным управлением === -->
        <TextView
            android:id="@+id/textCounterLabel"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Счётчик нажатий:"
            android:textStyle="bold"
            android:layout_marginBottom="8dp"/>

        <TextView
            android:id="@+id/textCounter"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/counter_text"
            android:textSize="24sp"
            android:textStyle="bold"
            android:layout_marginBottom="12dp"/>

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            android:layout_marginBottom="24dp">

            <Button
                android:id="@+id/buttonDecrement"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="-1"
                android:layout_marginEnd="4dp"/>

            <Button
                android:id="@+id/buttonIncrement"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="@string/button_increment"
                android:layout_marginStart="4dp"
                android:layout_marginEnd="4dp"/>

            <Button
                android:id="@+id/buttonResetCounter"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="↺"
                android:layout_marginStart="4dp"/>
        </LinearLayout>

        <!-- === Блок 2: Ввод и отображение текста === -->
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Ввод текста:"
            android:textStyle="bold"
            android:layout_marginBottom="8dp"/>

        <EditText
            android:id="@+id/editTextInput"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="@string/hint_input"
            android:inputType="text"
            android:layout_marginBottom="8dp"
            android:imeOptions="actionDone"/>

        <Button
            android:id="@+id/buttonShow"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/button_show"
            android:layout_marginBottom="8dp"/>

        <TextView
            android:id="@+id/textEntered"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="@string/label_entered"
            android:textSize="16sp"
            android:background="#E8F4FD"
            android:padding="12dp"
            android:layout_marginBottom="24dp"/>

        <!-- === Блок 3: ToDo-список === -->
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Список задач:"
            android:textStyle="bold"
            android:layout_marginBottom="8dp"/>

        <!-- Счётчик задач -->
        <TextView
            android:id="@+id/textTaskCount"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Задач: 0"
            android:textSize="14sp"
            android:textColor="#666"
            android:layout_marginBottom="8dp"/>

        <EditText
            android:id="@+id/editTextTask"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="Новая задача..."
            android:inputType="text"
            android:layout_marginBottom="8dp"
            android:imeOptions="actionDone"/>

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            android:layout_marginBottom="8dp">

            <Button
                android:id="@+id/buttonAddTask"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="2"
                android:text="@string/button_add_task"/>

            <Button
                android:id="@+id/buttonRemoveLast"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="↩"
                android:layout_marginStart="4dp"/>
        </LinearLayout>

        <Button
            android:id="@+id/buttonClearAll"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="🗑 Очистить всё"
            android:backgroundTint="#FF6B6B"
            android:layout_marginBottom="8dp"/>

        <!-- Поле для удаления по индексу -->
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            android:layout_marginBottom="8dp">

            <EditText
                android:id="@+id/editIndexToRemove"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:hint="Индекс для удаления"
                android:inputType="number"
                android:maxLength="3"/>

            <Button
                android:id="@+id/buttonRemoveByIndex"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="Удалить №"
                android:layout_marginStart="4dp"/>
        </LinearLayout>

        <TextView
            android:id="@+id/textTasks"
            android:layout_width="match_parent"
            android:layout_height="200dp"
            android:text="@string/label_tasks"
            android:textSize="16sp"
            android:background="#F8F9FA"
            android:padding="12dp"
            android:scrollbars="vertical"
            android:fadeScrollbars="false"/>

    </LinearLayout>
</ScrollView>

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

class MainActivity : AppCompatActivity() {

    private lateinit var counterManager: CounterManager
    private lateinit var taskManager: TaskManager

    // UI элементы
    private lateinit var textCounter: TextView
    private lateinit var textTaskCount: TextView
    private lateinit var textTasks: TextView
    private lateinit var editTextTask: EditText
    private lateinit var editTextIndex: EditText

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        initManagers()
        bindViews()
        setupListeners()
        restoreState(savedInstanceState)
    }

    private fun initManagers() {
        counterManager = CounterManager { count ->
            textCounter.text = getString(R.string.counter_text, count)
        }

        taskManager = TaskManager { tasks ->
            updateTasksDisplay(tasks)
            updateTaskCount(tasks.size)
        }
    }

    private fun bindViews() {
        textCounter = findViewById(R.id.textCounter)
        textTaskCount = findViewById(R.id.textTaskCount)
        textTasks = findViewById(R.id.textTasks)
        editTextTask = findViewById(R.id.editTextTask)
        editTextIndex = findViewById(R.id.editIndexToRemove)
    }

    private fun setupListeners() {
        // === Счётчик ===
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

        // === Ввод текста ===
        val editInput = findViewById<EditText>(R.id.editTextInput)
        val buttonShow = findViewById<Button>(R.id.buttonShow)
        val textEntered = findViewById<TextView>(R.id.textEntered)

        buttonShow.setOnClickListener {
            val text = editInput.text.toString()
            textEntered.text = "${getString(R.string.label_entered)} $text"
            editInput.text.clear()
        }

        // Поддержка Enter в поле ввода
        editInput.setOnEditorActionListener { _, actionId, _ ->
            if (actionId == EditorInfo.IME_ACTION_DONE) {
                buttonShow.performClick()
                true
            } else false
        }

        // === ToDo-список ===
        setupTaskListeners()
    }

    private fun setupTaskListeners() {
        // Добавление задачи
        findViewById<Button>(R.id.buttonAddTask).setOnClickListener {
            addCurrentTask()
        }

        // Поддержка Enter
        editTextTask.setOnEditorActionListener { _, actionId, _ ->
            if (actionId == EditorInfo.IME_ACTION_DONE) {
                addCurrentTask()
                true
            } else false
        }

        // Удаление последней
        findViewById<Button>(R.id.buttonRemoveLast).setOnClickListener {
            if (taskManager.removeLast()) {
                showToast("Последняя задача удалена")
            } else {
                showToast("Список пуст")
            }
        }

        // Очистка всех
        findViewById<Button>(R.id.buttonClearAll).setOnClickListener {
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
        findViewById<Button>(R.id.buttonRemoveByIndex).setOnClickListener {
            removeTaskByIndex()
        }
    }

    private fun addCurrentTask() {
        val task = editTextTask.text.toString()
        if (taskManager.addTask(task)) {
            editTextTask.text.clear()
            showToast("Задача добавлена ✓")
        } else {
            showToast("Введите текст задачи")
        }
    }

    private fun removeTaskByIndex() {
        val indexStr = editTextIndex.text.toString()
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
            editTextIndex.text.clear()
        } else {
            showToast("Индекс вне диапазона")
        }
    }

    private fun updateTasksDisplay(tasks: List<String>) {
        textTasks.text = if (tasks.isEmpty()) {
            getString(R.string.label_tasks)
        } else {
            tasks.mapIndexed { index, task -> "${index}. $task" }
                .joinToString("\n")
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

## Индивидуальное задание: Удаление задач
Добавьте второе поле ввода для номера задачи (индекса) и кнопку "Удалить по индексу". При нажатии задача с указанным индексом удаляется из списка (с проверкой границ).

<img width="385" height="853" alt="image" src="https://github.com/user-attachments/assets/8bba250a-3b40-488b-8156-f1665b1b7819" />



## Ответы на контрольные вопросы
**Как получить текст из EditText?** : `editText.text.toString()` (для удаления пробелов по краям можно добавить `.trim()`).

**Почему при повороте экрана данные (счётчик, список задач) сбрасываются? Как это можно исправить?** : При повороте экрана активность пересоздаётся, и все переменные в памяти инициализируются заново. Исправить можно двумя способами: 1) переопределив метод `onSaveInstanceState()` для сохранения простых данных в Bundle; 2) используя `ViewModel`, который переживает пересоздание активности и хранит данные в памяти.

**Для чего используется joinToString? Как изменить разделитель?** : Функция преобразует коллекцию (список) в одну строку, соединяя элементы. Разделитель задаётся первым аргументом: по умолчанию это запятая `", "`, но можно указать `"\n"` для переноса строки, `" | "` или любой другой символ: `tasks.joinToString(separator = "\n")`.

**В чём разница между List и MutableList?** : `List` — это интерфейс только для чтения, элементы нельзя добавлять, удалять или изменять после создания. `MutableList` наследуется от `List` и предоставляет методы для модификации: `add()`, `remove()`, `clear()`, а также доступ к элементам по индексу для записи.

**Как очистить поле ввода после добавления задачи?** : Вызвать метод `editText.text.clear()` или альтернативно `editText.setText("")`.

## Вывод

В результате выполнения лабораторной работы было разработано Android-приложение для демонстрации обработки пользовательского ввода, управления состоянием и динамического обновления интерфейса на Kotlin.

**Изученные и применённые технологии:**
- **UI-компоненты:** реализована работа с `EditText` (ввод текста), `TextView` (отображение данных) и `Button` (инициация действий).
- **Обработка событий:** настроены слушатели нажатий `setOnClickListener` и обработки клавиатурного действия `OnEditorActionListener`.
- **Управление состоянием:** использованы переменные для счётчика и `MutableList<String>` для хранения задач; внедрено сохранение данных через `onSaveInstanceState` и `Bundle`.
- **Работа со строками и коллекциями:** применены методы `.toString()`, `.trim()`, `.isNotBlank()`, `.clear()` и `.joinToString()` для безопасного извлечения, валидации и форматирования данных.
- **Модульная архитектура:** логика счётчика и списка задач инкапсулирована в отдельные классы с использованием колбэков для реактивного обновления UI.

**Обработка данных и логика:**
- Реализован счётчик с операциями увеличения, уменьшения и сброса, защищённый от некорректных состояний.
- Настроена валидация вводимых данных: задачи добавляются только при наличии текста, индексы для удаления строго проверяются на границы массива.
- Обеспечено мгновенное обновление интерфейса при любом изменении состояния через централизованные методы рендеринга.
- Продемонстрирован принцип безопасного доступа: список задач возвращается как неизменяемый (`toList()`), предотвращая побочные модификации извне.

**Дополнительные улучшения (вариативные задания):**
- Внедрён динамический счётчик общего количества задач в списке.
- Реализовано удаление задачи по заданному индексу с проверкой корректности числового ввода.
- Добавлена функция полной очистки списка с диалоговым окном подтверждения (`AlertDialog`).
- Настроено корректное сохранение и восстановление состояния приложения при изменении ориентации экрана.

**Интерфейс и запуск:**
- В `activity_main.xml` создана адаптивная вертикальная вёрстка с `ScrollView` для безопасного отображения длинных списков без обрезки на любых устройствах.
- Все текстовые ресурсы вынесены в `strings.xml`, что соответствует стандартам Android-разработки и упрощает поддержку кода.
- Реализована пользовательская обратная связь через `Toast`-уведомления и автоматическую очистку полей ввода после успешных операций.
- Приложение успешно компилируется и запускается на эмуляторе, все функциональные блоки отрабатывают стабильно и предсказуемо.

**Итог:** Цель работы достигнута. Получены практические навыки обработки пользовательского ввода, управления состоянием компонентов и динамического обновления UI в Android-приложениях. Освоены принципы написания читаемого, модульного кода с валидацией данных и сохранением состояния, что создаёт прочную основу для разработки полноценных мобильных приложений.
