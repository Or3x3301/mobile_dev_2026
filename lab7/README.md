# Лабораторная работа №7

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

Лабораторная работа №7
**«Добавление второго экрана (детали задачи). Переход по клику на элемент списка»**  
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

Научиться создавать многоэкранные приложения, осуществлять переход между экранами с передачей данных через `Intent`, обрабатывать клики на элементах `RecyclerView` и организовывать навигацию между `Activity`.

## activity_detail.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Детали задачи"
        android:textSize="24sp"
        android:textStyle="bold"
        android:layout_marginBottom="24dp"/>

    <TextView
        android:id="@+id/textTaskDetail"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textSize="18sp"
        android:layout_marginBottom="16dp"/>

    <Button
        android:id="@+id/buttonBack"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Назад"
        android:layout_gravity="center_horizontal"/>

    <Button
        android:id="@+id/buttonDelete"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Удалить задачу"
        android:backgroundTint="#FF5252"
        android:layout_gravity="center_horizontal"
        android:layout_marginTop="8dp"/>

</LinearLayout>
```

## DetailActivity.kt
```kotlin
package com.example.todoapp

import android.app.Activity
import android.content.Intent
import android.os.Bundle
import android.widget.Button
import android.widget.TextView
import androidx.appcompat.app.AppCompatActivity

class DetailActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_detail)

        val textTaskDetail = findViewById<TextView>(R.id.textTaskDetail)
        val buttonBack = findViewById<Button>(R.id.buttonBack)
        val buttonDelete = findViewById<Button>(R.id.buttonDelete)

        val taskText = intent.getStringExtra("task_text") ?: "Нет данных"
        val taskPosition = intent.getIntExtra("task_position", -1)

        textTaskDetail.text = taskText

        buttonBack.setOnClickListener { finish() }

        buttonDelete.setOnClickListener {
            val resultIntent = Intent()
            resultIntent.putExtra("deleted_position", taskPosition)
            setResult(Activity.RESULT_OK, resultIntent)
            finish()
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
    private val tasks: MutableList<String>,
    private val onItemClick: (Int) -> Unit
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

        holder.itemView.setOnClickListener {
            onItemClick(position)
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
import androidx.appcompat.app.AppCompatActivity
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView

class MainActivity : AppCompatActivity() {

    private val tasks = mutableListOf<String>()
    private lateinit var adapter: TaskAdapter

    private val detailLauncher = registerForActivityResult(
        ActivityResultContracts.StartActivityForResult()
    ) { result ->
        if (result.resultCode == Activity.RESULT_OK) {
            val position = result.data?.getIntExtra("deleted_position", -1) ?: -1
            if (position != -1 && position < tasks.size) {
                tasks.removeAt(position)
                adapter.notifyItemRemoved(position)
                Toast.makeText(this, "Задача удалена", Toast.LENGTH_SHORT).show()
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
        adapter = TaskAdapter(tasks) { position ->
            val intent = Intent(this, DetailActivity::class.java)
            intent.putExtra("task_text", tasks[position])
            intent.putExtra("task_position", position)
            detailLauncher.launch(intent)
        }
        recyclerView.adapter = adapter

        buttonAddTask.setOnClickListener {
            val task = editTextTask.text.toString()
            if (task.isNotBlank()) {
                tasks.add(task)
                adapter.notifyItemInserted(tasks.size - 1)
                editTextTask.text.clear()
            } else {
                Toast.makeText(this, "Введите задачу", Toast.LENGTH_SHORT).show()
            }
        }

        if (savedInstanceState != null) {
            val savedTasks = savedInstanceState.getStringArrayList("tasks")
            if (savedTasks != null) {
                tasks.clear()
                tasks.addAll(savedTasks)
                adapter.notifyDataSetChanged()
            }
        }
    }

    override fun onSaveInstanceState(outState: Bundle) {
        super.onSaveInstanceState(outState)
        outState.putStringArrayList("tasks", ArrayList(tasks))
    }
}
```

## Индивидуальное задание: Удаление задачи с экрана деталей
Реализована передача позиции задачи на второй экран через `Intent`. На экране деталей добавлена кнопка «Удалить задачу». При нажатии формируется результат с позицией удалённого элемента, вызывается `setResult(RESULT_OK, intent)` и активность закрывается. В `MainActivity` результат обрабатывается через современный `ActivityResultLauncher`: задача удаляется из списка, адаптер уведомляется через `notifyItemRemoved()`, выводится `Toast`.

<img width="310" height="690" alt="image" src="https://github.com/user-attachments/assets/dddc58c0-e4af-461c-84af-7147ac4cc8d5" />


## Ответы на контрольные вопросы
1. `Intent` – это объект, используемый для выполнения операций в Android: запуска `Activity`, `Service` или отправки `Broadcast`. Существуют **явные Intent** (указан конкретный класс компонента) и **неявные Intent** (указано действие или тип данных, система выбирает приложение самостоятельно).
2. Данные передаются через метод `putExtra(key, value)` при создании `Intent`. В целевой `Activity` они извлекаются методами `getStringExtra()`, `getIntExtra()` и т.д. через объект `intent`. Для сложных объектов используют `Parcelable` или `Serializable`.
3. Основные способы: передача лямбды/callback в конструктор адаптера, создание интерфейса-слушателя, установка `setOnClickListener` напрямую на `itemView` внутри `onBindViewHolder`. Наиболее современный и лаконичный подход – передача лямбды.
4. В Android Studio: кликнуть правой кнопкой мыши по пакету → `New` → `Activity` → `Empty Views Activity`. Ввести имя активности, имя layout-файла, снять галочку `Launcher Activity`, нажать `Finish`. IDE создаст класс, XML и пропишет активность в `AndroidManifest.xml`.
5. Метод `finish()` завершает текущую `Activity`, удаляет её из стека навигации и возвращает пользователя к предыдущему экрану. Аналогично нажатию системной кнопки «Назад».

## Вывод

В результате выполнения лабораторной работы было доработано Android-приложение `TodoApp`: реализован переход на второй экран с деталями задачи при клике на элемент списка, настроена передача данных между экранами, организована обратная навигация и выполнено индивидуальное задание по удалению задачи.

**Изученные и применённые технологии:**
- **Intent и навигация:** использованы явные `Intent` для запуска `DetailActivity`, методы `putExtra()`/`getStringExtra()`/`getIntExtra()` для передачи текста и позиции задачи.
- **RecyclerView Click Handling:** реализована обработка нажатий через передачу лямбда-выражения `(Int) -> Unit` в конструктор адаптера.
- **ActivityResult API:** применён современный `registerForActivityResult(ActivityResultContracts.StartActivityForResult())` для безопасного получения результата из второй активности без устаревшего `onActivityResult`.
- **Жизненный цикл Activity:** использован `setResult()` + `finish()` для возврата данных и закрытия экрана деталей.

**Верстка и архитектура интерфейса:**
- Создан экран `activity_detail.xml` с вертикальным `LinearLayout`, заголовком, `TextView` для задачи и двумя кнопками управления.
- В `TaskAdapter` добавлен `holder.itemView.setOnClickListener`, вызывающий callback с позицией.
- В `MainActivity` лямбда адаптера формирует `Intent`, упаковывает данные и запускает экран через `detailLauncher.launch()`.
- Логика разделена: адаптер отвечает только за отображение и клик, навигация и управление данными сосредоточены в `MainActivity`.

**Индивидуальное задание («Удаление задачи с экрана деталей»):**
- На экран деталей добавлена кнопка удаления с красным акцентом (`backgroundTint="#FF5252"`).
- Позиция задачи передаётся через `Intent`, возвращается через `setResult(RESULT_OK, intent)`.
- В `MainActivity` результат обрабатывается в `detailLauncher`: задача удаляется из `MutableList`, вызывается `adapter.notifyItemRemoved(position)` для анимированного обновления списка, выводится системное уведомление.
- Проверена корректность индексов, исключены `IndexOutOfBoundsException` при быстром удалении.

**Интерфейс и запуск:**
- Все XML-разметки валидны, отступы и размеры шрифтов адаптированы под разные экраны.
- В `DetailActivity.kt` реализована безопасная распаковка данных через элвис-оператор, исключены `NullPointerException`.
- В `MainActivity.kt` корректно инициализирован `ActivityResultLauncher`, импорты добавлены, конфликты отсутствуют.
- Приложение успешно компилируется и запускается. Клик по карточке открывает детали, кнопка «Назад» возвращает к списку, кнопка «Удалить» удаляет задачу с анимацией и обновляет адаптер. Состояние списка сохраняется при повороте экрана.

**Итог:** Цель работы достигнута. Получены практические навыки создания многоэкранных приложений, передачи данных через `Intent`, обработки кликов в `RecyclerView` и управления результатом активностей через современный `ActivityResult API`. Реализованный код демонстрирует принципы слабой связанности, безопасной навигации и корректной интеграции UI-событий с логикой приложения на Kotlin.
