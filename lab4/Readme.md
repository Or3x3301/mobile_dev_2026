# Лабораторная работа №4

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

Лабораторная работа №4
**«Верстка экрана профиля пользователя (аватар, имя, кнопка «Редактировать»)»**  
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

Освоить создание пользовательского интерфейса в Android с использованием ConstraintLayout, изучить основные компоненты: ImageView, TextView, Button. Научиться работать с ресурсами (строки, цвета, размеры) и обрабатывать нажатия кнопок.

## activity_main.xml
```kotlin

<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/gray_light"
    tools:context=".MainActivity">

    <ImageView
        android:id="@+id/imageAvatar"
        android:layout_width="@dimen/avatar_size"
        android:layout_height="@dimen/avatar_size"
        android:src="@drawable/ic_profile"
        android:contentDescription="@string/profile_name"
        android:scaleType="centerInside"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        android:layout_marginTop="@dimen/margin_normal" />

    <TextView
        android:id="@+id/textName"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/profile_name"
        android:textSize="@dimen/text_size_name"
        android:textColor="@color/black"
        android:textStyle="bold"
        app:layout_constraintTop_toBottomOf="@id/imageAvatar"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        android:layout_marginTop="@dimen/margin_small" />

    <TextView
        android:id="@+id/textStatus"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/profile_status"
        android:textSize="@dimen/text_size_status"
        android:textColor="@color/purple_500"
        app:layout_constraintTop_toBottomOf="@id/textName"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        android:layout_marginTop="@dimen/margin_small" />

    <LinearLayout
        android:id="@+id/layoutContacts"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:gravity="center"
        android:layout_marginTop="@dimen/margin_normal"
        app:layout_constraintTop_toBottomOf="@id/textStatus"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent">

        <LinearLayout
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            android:gravity="center_vertical"
            android:layout_marginEnd="@dimen/margin_normal">

            <ImageView
                android:layout_width="@dimen/icon_size"
                android:layout_height="@dimen/icon_size"
                android:src="@drawable/ic_phone"
                android:contentDescription="Phone"
                android:layout_marginEnd="@dimen/margin_small"
                app:tint="@color/purple_500" />

            <TextView
                android:id="@+id/textPhone"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@string/contact_phone"
                android:textSize="@dimen/text_size_contact"
                android:textColor="@color/black" />
        </LinearLayout>

        <LinearLayout
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            android:gravity="center_vertical">

            <ImageView
                android:layout_width="@dimen/icon_size"
                android:layout_height="@dimen/icon_size"
                android:src="@drawable/ic_email"
                android:contentDescription="Email"
                android:layout_marginEnd="@dimen/margin_small"
                app:tint="@color/purple_500" />

            <TextView
                android:id="@+id/textEmail"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@string/contact_email"
                android:textSize="@dimen/text_size_contact"
                android:textColor="@color/black" />
        </LinearLayout>

    </LinearLayout>

    <Button
        android:id="@+id/buttonEdit"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/button_edit"
        android:backgroundTint="@color/purple_200"
        app:cornerRadius="@dimen/button_corner_radius"
        app:layout_constraintTop_toBottomOf="@id/layoutContacts"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        android:layout_marginTop="@dimen/margin_normal"/>

    <Button
        android:id="@+id/buttonBack"
        style="@style/Widget.MaterialComponents.Button.TextButton"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/button_back"
        android:textColor="@color/purple_500"
        app:layout_constraintTop_toBottomOf="@id/buttonEdit"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        android:layout_marginTop="@dimen/margin_small"/>

</androidx.constraintlayout.widget.ConstraintLayout>

```
## MainActivity.kt
```kotlin
package com.example.profileapp

import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import android.widget.Button
import android.widget.Toast

class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val buttonEdit = findViewById<Button>(R.id.buttonEdit)
        buttonEdit.setOnClickListener {
            Toast.makeText(this, R.string.toast_message, Toast.LENGTH_SHORT).show()
        }

        val buttonBack = findViewById<Button>(R.id.buttonBack)
        buttonBack.setOnClickListener {
            Toast.makeText(this, R.string.toast_back, Toast.LENGTH_SHORT).show()
        }
    }
}
```

## Индивидуальное задание: Профиль с контактами
Добавьте под статусом информацию о телефоне и email (два TextView с иконками). Используйте LinearLayout горизонтальный внутри ConstraintLayout.

<img width="322" height="717" alt="image" src="https://github.com/user-attachments/assets/919e8548-f4ed-49ad-8cb3-718dc14faf58" />



## Ответы на контрольные вопросы
1. ConstraintLayout используется для гибкого позиционирования элементов интерфейса относительно друг друга или границ родительского контейнера. Преимущества перед LinearLayout: плоская иерархия виджетов (отсутствует глубокая вложенность), высокая производительность рендеринга, возможность задавать сложные связи, пропорции и смещения без использования дополнительных ViewGroup.

2. Атрибуты `app:layout_constraint...` задают ограничения (привязки) конкретных сторон виджета к сторонам родительского контейнера или к идентификаторам других виджетов, определяя точное положение и поведение элемента при изменении размеров экрана.

3. Размеры выносятся в файл `res/values/dimens.xml`, цвета — в `res/values/colors.xml`. Это необходимо для централизованного управления стилями, повторного использования значений, упрощения поддержки кода, быстрой смены тем и адаптации под разные плотности экранов.

4. Обработчик клика добавляется методом `setOnClickListener`, передавая ему лямбда-выражение или объект `View.OnClickListener`: `button.setOnClickListener { /* действия */ }`.

5. Обработчик добавляется аналогично: `imageView.setOnClickListener { /* действия */ }`. В XML-разметке к элементу `ImageView` обязательно нужно добавить атрибуты `android:clickable="true"` и `android:focusable="true"`.

## Вывод

В результате выполнения лабораторной работы было разработано Android-приложение с экраном профиля пользователя, содержащим аватар, имя, статус, контактные данные и интерактивные кнопки управления.

**Изученные и применённые технологии:**
- **ConstraintLayout:** использован как корневой контейнер для создания плоской иерархии интерфейса и гибкого позиционирования элементов относительно друг друга.
- **Система ресурсов Android:** текстовые строки вынесены в `strings.xml`, цветовая палитра — в `colors.xml`, размеры и отступы — в `dimens.xml`.
- **Базовые View-компоненты:** реализованы `ImageView` для отображения аватара, `TextView` для текстовой информации и `Button` для выполнения действий.
- **Kotlin и Android SDK:** применены `findViewById` для связывания кода с XML-разметкой и `setOnClickListener` для обработки пользовательских взаимодействий.

**Верстка и архитектура интерфейса:**
- Настроены привязки (`app:layout_constraint...`) для всех виджетов, обеспечивающие корректное центрирование и адаптивность на разных размерах экранов.
- Реализована вертикальная структура макета: аватар → имя → статус → контакты → кнопки действий.
- Обеспечена поддерживаемость кода за счёт отказа от хардкода и централизованного управления стилями через ресурсные файлы.
- Добавлена обработка нажатий, вызывающая системные уведомления (`Toast`) для обратной связи с пользователем.

**Индивидуальное задание («Профиль с контактами»):**
- Под статусом профиля размещён горизонтальный `LinearLayout`, содержащий информацию о номере телефона и электронной почте.
- Для каждого контакта добавлены векторные иконки (`ic_phone`, `ic_email`) и текстовые метки с выравниванием по вертикальному центру.
- Реализована визуальная группировка данных с использованием вложенных контейнеров и атрибутов `gravity`/`orientation`.
- Проверено корректное отображение контактов при изменении параметров экрана и масштабировании шрифтов.

**Интерфейс и запуск:**
- В `activity_main.xml` полностью настроены ограничения всех виджетов относительно родительского контейнера и соседних элементов, исключены ошибки позиционирования.
- В `MainActivity.kt` реализована инициализация компонентов и привязка логики кликов без конфликтов импортов и ошибок компиляции.
- Приложение успешно компилируется и запускается на эмуляторе, все элементы интерфейса отображаются согласно заданию, кнопки реагируют на нажатия выводом `Toast`.

**Итог:** Цель работы достигнута. Получены практические навыки создания пользовательского интерфейса в Android с использованием ConstraintLayout, работы с ресурсами проекта и обработки событий взаимодействия. Написанный код демонстрирует принципы модульной верстки, повторного использования стилей и корректной интеграции UI-компонентов с логикой приложения на Kotlin.
