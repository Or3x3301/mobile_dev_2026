# Лабораторная работа №3

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

Лабораторная работа №3 
**«Реализация списка объектов с фильтрацией с использованием .map, .filter, .sortedBy»**  
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

Изучить функциональные методы обработки коллекций в Kotlin (filter, map, sortedBy) на примере списка объектов и вывести результаты в интерфейс Android-приложения.

## Product
```kotlin

package com.example.lab3.models

data class Product(
    val name: String,
    val category: String,
    val price: Double,
    val inStock: Boolean
)

```
## MainActivity
```kotlin
package com.example.lab3

import android.os.Bundle
import android.widget.TextView
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.padding
import androidx.compose.material3.Scaffold
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.tooling.preview.Preview
import com.example.lab3.models.Movie
import com.example.lab3.models.Product
import com.example.lab3.ui.theme.Lab3Theme

class MainActivity : ComponentActivity() {
    private fun getProducts(): List<Product> {
        return listOf(
            Product("Ноутбук", "Электроника", 75000.0, true),
            Product("Мышь", "Электроника", 1500.0, true),
            Product("Книга 'Котлин'", "Книги", 1200.0, false),
            Product("Флешка 64GB", "Электроника", 2000.0, true),
            Product("Блокнот", "Канцелярия", 300.0, true),
            Product("Ручка", "Канцелярия", 50.0, false),
            Product("Монитор", "Электроника", 25000.0, true)
        )
    }

    private fun getMovies(): List<Movie> {
        return listOf(
            Movie("Побег из Шоушенка", "Драма", 9.3, 1994),
            Movie("Крёстный отец", "Драма", 9.2, 1972),
            Movie("Тёмный рыцарь", "Боевик", 9.0, 2008),
            Movie("Начало", "Фантастика", 8.8, 2010),
            Movie("Интерстеллар", "Фантастика", 8.6, 2014),
            Movie("Форрест Гамп", "Драма", 8.8, 1994),
            Movie("Матрица", "Фантастика", 8.7, 1999),
            Movie("Бойцовский клуб", "Драма", 8.8, 1999),
            Movie("Джокер", "Драма", 8.4, 2019),
            Movie("Мстители: Финал", "Боевик", 8.4, 2019),
            Movie("Сияние", "Ужасы", 8.4, 1980),
            Movie("Титаник", "Драма", 7.9, 1997)  // рейтинг < 8.0, не попадёт в фильтр
        )
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val products = getProducts()

        // Исходный список
        val originalText = products.joinToString("\n") {
            "${it.name} – ${it.price} руб. (${if (it.inStock) "в наличии" else "нет"})"
        }
        findViewById<TextView>(R.id.textOriginal).text = originalText

        // Фильтр: только в наличии
        val inStockProducts = products.filter { it.inStock }
        val inStockText = inStockProducts.joinToString("\n") {
            "${it.name} – ${it.price} руб."
        }
        findViewById<TextView>(R.id.textInStock).text = inStockText

        //  Цепочка: Электроника + в наличии → сортировка по цене → форматирование
        val electronicsSorted = products
            .filter { it.category == "Электроника" && it.inStock }  // фильтрация
            .sortedBy { it.price }                                   // сортировка по возрастанию
            .map { "${it.name} – ${it.price} руб." }                 // преобразование в строку

        findViewById<TextView>(R.id.textSorted).text = electronicsSorted.joinToString("\n")

        // Товары дешевле 2000 руб, отсортированные по названию
        val cheapProducts = products
            .filter { it.price < 2000 && it.inStock }
            .sortedBy { it.name }
            .map { "${it.name}: ${it.price} руб." }

        findViewById<TextView>(R.id.textCheap).text = cheapProducts.joinToString("\n")

        val movies = getMovies()

        val topMovies = movies
            .filter { it.rating > 8.0 }                    // фильтрация по рейтингу
            .sortedBy { it.year }                          // сортировка по году (старые → новые)
            .map { "${it.title} (${it.year}) — ${it.rating}" } // форматирование

        findViewById<TextView>(R.id.textMovies).text = topMovies.joinToString("\n")
    }
}

@Composable
fun Greeting(name: String, modifier: Modifier = Modifier) {
    Text(
        text = "Hello $name!",
        modifier = modifier
    )
}

@Preview(showBackground = true)
@Composable
fun GreetingPreview() {
    Lab3Theme {
        Greeting("Android")
    }
}

```

## Индивидуальное задание: Класс "Студент"
Список фильмов (название, жанр, рейтинг, год выпуска).

Показать фильмы с рейтингом выше 8.0.
Отсортировать их по году выпуска.
Вывести список названий и рейтингов.

<img width="383" height="855" alt="image" src="https://github.com/user-attachments/assets/fda38056-93f3-44b5-b853-bdc809b024c5" />


## Ответы на контрольные вопросы
**Что возвращает filter?**

- Новый список, исходный не изменяется (принцип неизменяемости).

**Разница sortedBy и sortedByDescending?**

- Первый сортирует по возрастанию, второй — по убыванию.

**Как объединить условия в filter?**

- Через && (И) или || (ИЛИ):
- filter { it.price > 100 && it.inStock }

**Для чего map? Пример:**

- Преобразует элементы коллекции:
- products.map { it.name } → список только с названиями.

**Что такое joinToString?**

- Функция, которая объединяет элементы коллекции в одну строку с разделителем:
- listOf("A", "B").joinToString(" | ") → "A | B"

## Вывод

В результате выполнения лабораторной работы было разработано Android-приложение для демонстрации функциональной обработки списков объектов с использованием встроенных методов Kotlin.

**Изученные и применённые технологии:**
- **Data class:** реализована модель `Movie` для хранения данных о фильме (название, жанр, рейтинг, год выпуска).
- **Функциональные методы коллекций:** применены `.filter`, `.sortedBy` и `.map` для выборки, сортировки и преобразования данных.
- **Цепочки вызовов (Chaining):** настроена последовательная обработка данных без создания лишних промежуточных переменных.
- **Лямбда-выражения:** использованы анонимные функции для передачи условий фильтрации и правил сортировки (в т.ч. с неявным параметром `it`).
- **Форматирование вывода:** применён метод `.joinToString()` для преобразования коллекции строк в единый текстовый блок с переносами.

**Обработка коллекций и логика:**
- Реализована фильтрация списка по условию `rating > 8.0`, корректно исключающая фильмы с низким рейтингом.
- Выполнена сортировка отфильтрованной выборки по году выпуска в порядке возрастания.
- Осуществлено преобразование объектов `Movie` в человекочитаемые строки с указанием названия и рейтинга.
- Продемонстрирован принцип иммутабельности: исходный список не изменяется, каждая операция возвращает новую коллекцию.

**Индивидуальное задание («Список фильмов»):**
- Создан тестовый набор данных из 12 фильмов различных жанров и годов выпуска.
- Настроена цепочка обработки: `filter { it.rating > 8.0 } → sortedBy { it.year } → map { ... }`.
- Реализован вывод отфильтрованных и отсортированных данных в интерфейс приложения.
- Проверена корректность работы при модификации условий (например, добавление фильтра по жанру или использование `sortedByDescending`).

**Интерфейс и запуск:**
- В `activity_main.xml` создан вертикальный интерфейс с `ScrollView` и `TextView` для безопасного отображения длинных списков без обрезки.
- Результаты обработки данных успешно привязаны к UI-компонентам через `findViewById`.
- Приложение корректно компилируется и запускается на эмуляторе, все блоки вывода заполняются ожидаемыми данными.

**Итог:** Цель работы достигнута. Получены практические навыки работы с коллекциями в Kotlin, освоены методы функциональной обработки данных (`.filter`, `.map`, `.sortedBy`) и их интеграция в Android-приложение. Написанный код демонстрирует принципы декларативного, читаемого и эффективного программирования.
