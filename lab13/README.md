# Лабораторная работа №13

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

Лабораторная работа №13
**«Создание простого API клиента. Запрос списка постов с jsonplaceholder.typicode.com»**  
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

Научиться выполнять сетевые запросы в Android-приложении с использованием библиотеки Retrofit и корутин, обрабатывать ответы сервера, парсить JSON-данные и отображать их в RecyclerView.

## Post.kt
```kotlin
package com.example.postsapp.models

import android.os.Parcelable
import kotlinx.parcelize.Parcelize

@Parcelize
data class Post(
    val id: Int,
    val title: String,
    val body: String
) : Parcelable
```

## ApiService.kt
```kotlin
package com.example.postsapp.api

import com.example.postsapp.models.Post
import retrofit2.http.GET

interface ApiService {
    @GET("posts")
    suspend fun getPosts(): List<Post>
}
```

## RetrofitClient.kt
```kotlin
package com.example.postsapp.api

import retrofit2.Retrofit
import retrofit2.converter.gson.GsonConverterFactory

object RetrofitClient {
    private const val BASE_URL = "https://jsonplaceholder.typicode.com/"

    private val retrofit = Retrofit.Builder()
        .baseUrl(BASE_URL)
        .addConverterFactory(GsonConverterFactory.create())
        .build()

    val apiService: ApiService = retrofit.create(ApiService::class.java)
}
```

## PostsRepository.kt
```kotlin
package com.example.postsapp.repositories

import com.example.postsapp.api.RetrofitClient
import com.example.postsapp.models.Post
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.withContext

class PostsRepository {
    private val apiService = RetrofitClient.apiService

    suspend fun getPosts(): List<Post> = withContext(Dispatchers.IO) {
        apiService.getPosts()
    }
}
```

## PostsViewModel.kt
```kotlin
package com.example.postsapp.viewmodels

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.example.postsapp.models.Post
import com.example.postsapp.repositories.PostsRepository
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.launch

sealed class PostsUiState {
    object Loading : PostsUiState()
    data class Success(val posts: List<Post>) : PostsUiState()
    data class Error(val message: String) : PostsUiState()
}

class PostsViewModel : ViewModel() {
    private val repository = PostsRepository()
    
    private val _uiState = MutableStateFlow<PostsUiState>(PostsUiState.Loading)
    val uiState: StateFlow<PostsUiState> = _uiState.asStateFlow()

    init {
        loadPosts()
    }

    fun loadPosts() {
        viewModelScope.launch {
            _uiState.value = PostsUiState.Loading
            try {
                val posts = repository.getPosts()
                _uiState.value = PostsUiState.Success(posts)
            } catch (e: Exception) {
                _uiState.value = PostsUiState.Error(e.message ?: "Unknown error")
            }
        }
    }
}
```

## PostsAdapter.kt
```kotlin
package com.example.postsapp.adapters

import android.view.LayoutInflater
import android.view.ViewGroup
import androidx.recyclerview.widget.RecyclerView
import com.example.postsapp.databinding.ItemPostBinding
import com.example.postsapp.models.Post

class PostsAdapter(
    private val onItemClick: (Post) -> Unit
) : RecyclerView.Adapter<PostsAdapter.PostViewHolder>() {
    
    private var posts = emptyList<Post>()
    
    fun submitList(newPosts: List<Post>) {
        posts = newPosts
        notifyDataSetChanged()
    }
    
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): PostViewHolder {
        val binding = ItemPostBinding.inflate(LayoutInflater.from(parent.context), parent, false)
        return PostViewHolder(binding)
    }
    
    override fun onBindViewHolder(holder: PostViewHolder, position: Int) {
        holder.bind(posts[position])
    }
    
    override fun getItemCount() = posts.size
    
    inner class PostViewHolder(private val binding: ItemPostBinding) : 
        RecyclerView.ViewHolder(binding.root) {
        
        fun bind(post: Post) {
            binding.textPostId.text = "ID: ${post.id}"
            binding.textPostTitle.text = post.title
            binding.textPostBody.text = post.body
            binding.root.setOnClickListener { onItemClick(post) }
        }
    }
}
```

## activity_main.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <Button
        android:id="@+id/buttonRefresh"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Обновить"/>

    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <androidx.recyclerview.widget.RecyclerView
            android:id="@+id/recyclerViewPosts"
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
            android:visibility="gone"/>

    </FrameLayout>

</LinearLayout>
```

## item_post.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.cardview.widget.CardView
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_margin="8dp"
    app:cardCornerRadius="8dp"
    app:cardElevation="4dp">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:padding="16dp">

        <TextView
            android:id="@+id/textPostId"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="ID: "
            android:textStyle="bold"
            android:textSize="14sp"/>

        <TextView
            android:id="@+id/textPostTitle"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Title"
            android:textSize="18sp"
            android:textStyle="bold"
            android:layout_marginTop="4dp"/>

        <TextView
            android:id="@+id/textPostBody"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Body"
            android:textSize="14sp"
            android:layout_marginTop="8dp"/>

    </LinearLayout>
</androidx.cardview.widget.CardView>
```

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
        android:id="@+id/textDetailTitle"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textSize="20sp"
        android:textStyle="bold"
        android:layout_marginBottom="16dp"/>

    <TextView
        android:id="@+id/textDetailBody"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textSize="16sp"/>

</LinearLayout>
```

## DetailActivity.kt
```kotlin
package com.example.postsapp

import android.os.Bundle
import android.widget.TextView
import androidx.appcompat.app.AppCompatActivity
import com.example.postsapp.models.Post

class DetailActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_detail)

        val textDetailTitle = findViewById<TextView>(R.id.textDetailTitle)
        val textDetailBody = findViewById<TextView>(R.id.textDetailBody)

        val post = intent.getParcelableExtra<Post>("post")
        textDetailTitle.text = post?.title
        textDetailBody.text = post?.body
    }
}
```

## MainActivity.kt
```kotlin
package com.example.postsapp

import android.content.Intent
import android.os.Bundle
import android.widget.Button
import android.widget.ProgressBar
import android.widget.TextView
import androidx.activity.viewModels
import androidx.appcompat.app.AppCompatActivity
import androidx.lifecycle.Lifecycle
import androidx.lifecycle.lifecycleScope
import androidx.lifecycle.repeatOnLifecycle
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView
import com.example.postsapp.adapters.PostsAdapter
import com.example.postsapp.models.Post
import com.example.postsapp.viewmodels.PostsUiState
import com.example.postsapp.viewmodels.PostsViewModel
import kotlinx.coroutines.launch

class MainActivity : AppCompatActivity() {

    private val viewModel: PostsViewModel by viewModels()
    private lateinit var adapter: PostsAdapter
    private lateinit var recyclerView: RecyclerView
    private lateinit var progressBar: ProgressBar
    private lateinit var textError: TextView
    private lateinit var buttonRefresh: Button

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        recyclerView = findViewById(R.id.recyclerViewPosts)
        progressBar = findViewById(R.id.progressBar)
        textError = findViewById(R.id.textError)
        buttonRefresh = findViewById(R.id.buttonRefresh)

        setupRecyclerView()
        observeUiState()

        buttonRefresh.setOnClickListener {
            viewModel.loadPosts()
        }
    }

    private fun setupRecyclerView() {
        adapter = PostsAdapter { post ->
            val intent = Intent(this, DetailActivity::class.java)
            intent.putExtra("post", post)
            startActivity(intent)
        }
        recyclerView.layoutManager = LinearLayoutManager(this)
        recyclerView.adapter = adapter
    }

    private fun observeUiState() {
        lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.uiState.collect { state ->
                    when (state) {
                        is PostsUiState.Loading -> showLoading()
                        is PostsUiState.Success -> showPosts(state.posts)
                        is PostsUiState.Error -> showError(state.message)
                    }
                }
            }
        }
    }

    private fun showLoading() {
        recyclerView.visibility = android.view.View.GONE
        progressBar.visibility = android.view.View.VISIBLE
        textError.visibility = android.view.View.GONE
    }

    private fun showPosts(posts: List<Post>) {
        recyclerView.visibility = android.view.View.VISIBLE
        progressBar.visibility = android.view.View.GONE
        textError.visibility = android.view.View.GONE
        adapter.submitList(posts)
    }

    private fun showError(message: String) {
        recyclerView.visibility = android.view.View.GONE
        progressBar.visibility = android.view.View.GONE
        textError.visibility = android.view.View.VISIBLE
        textError.text = "Ошибка: $message"
    }
}
```

## Индивидуальное задание: Детальный экран поста
Реализован переход на экран деталей поста при клике на элемент списка. В `PostsAdapter` установлен `OnClickListener` на корневой элемент карточки, передающий объект `Post` через `Intent` в `DetailActivity`. Модель `Post` реализует интерфейс `Parcelable` с помощью аннотации `@Parcelize` для безопасной передачи сложных объектов между активностями. В `DetailActivity` данные извлекаются через `getParcelableExtra()` и отображаются в `TextView`. Архитектура сохраняет разделение ответственности: ViewModel управляет загрузкой данных, Adapter — отображением списка, DetailActivity — отображением деталей.


<img width="388" height="834" alt="image" src="https://github.com/user-attachments/assets/557e0fca-a8d1-4c84-a499-3830e24c3f41" />


## Ответы на контрольные вопросы
1. **Retrofit** — типобезопасный HTTP-клиент для Android, упрощающий работу с REST API. Основные аннотации: `@GET`, `@POST`, `@PUT`, `@DELETE`, `@Path`, `@Query`, `@Body`.
2. Сетевые запросы нельзя выполнять в главном потоке, потому что это блокирует отрисовку UI и обработку событий, приводя к ошибке `ANR` (Application Not Responding).
3. `suspend` — функция, которая может приостанавливать выполнение без блокировки потока. Она может вызываться только из другой `suspend`-функции или корутины, обеспечивая асинхронность.
4. `Dispatchers.IO` — планировщик корутин, оптимизированный для операций ввода-вывода (сеть, диск). Он использует пул потоков, подходящий для длительных блокирующих операций.
5. Ошибки обрабатываются через конструкцию `try-catch` внутри корутины. При перехвате исключения эмитится состояние `UiState.Error`, позволяющее UI отобразить сообщение пользователю.
6. **JSONPlaceholder** — бесплатный тестовый REST API, предоставляющий фейковые данные (посты, комментарии, пользователи) для прототипирования и тестирования мобильных приложений без необходимости разворачивать собственный сервер.

## Вывод

В результате выполнения лабораторной работы было создано Android-приложение `PostsApp`, выполняющее сетевые запросы к тестовому API JSONPlaceholder, парсящее JSON-ответы и отображающее данные в виде списка с возможностью перехода к деталям поста.

**Изученные и применённые технологии:**
- **Retrofit:** декларативное описание API через аннотации, автоматическая десериализация JSON в Kotlin-объекты с помощью `GsonConverterFactory`.
- **Корутины и ViewModel:** асинхронные запросы выполняются в `viewModelScope` с использованием `Dispatchers.IO`, что гарантирует безопасную работу с сетью без блокировки UI.
- **StateFlow и sealed class:** реактивное управление состоянием экрана (Loading, Success, Error) через `StateFlow`, типобезопасная обработка через `when`.
- **RecyclerView и ViewBinding:** эффективное отображение списков с переиспользованием элементов, типобезопасная привязка данных к разметке.
- **Parcelable:** безопасная передача сложных объектов между активностями через `Intent`.

**Архитектура и разделение ответственности:**
- **Repository** инкапсулирует источник данных (сеть), предоставляя чистый API для ViewModel.
- **ViewModel** управляет состоянием UI и бизнес-логикой, не зная о деталях реализации сети.
- **Adapter** отвечает только за отображение данных и обработку кликов, делегируя навигацию в `MainActivity`.
- **DetailActivity** отображает детали поста, получая данные через `Intent`.

**Индивидуальное задание («Детальный экран поста»):**
- Реализован переход на `DetailActivity` при клике на элемент списка через `Intent` с передачей `Parcelable`-объекта `Post`.
- Модель `Post` аннотирована `@Parcelize` для автоматической генерации кода сериализации.
- В `DetailActivity` данные извлекаются через `getParcelableExtra()` и отображаются в `TextView`.
- Архитектура сохраняет слабую связанность: Adapter не знает о существовании `DetailActivity`, навигация инкапсулирована в `MainActivity`.

**Интерфейс и запуск:**
- Приложение отображает индикатор загрузки при старте, затем список постов в карточках с закруглёнными углами.
- Клик по карточке открывает экран деталей с заголовком и текстом поста.
- Кнопка "Обновить" инициирует повторную загрузку данных с сервера.
- При отсутствии интернета отображается сообщение об ошибке.

**Итог:** Цель работы достигнута. Получены практические навыки работы с Retrofit, корутинами, реактивным управлением состоянием и навигацией между экранами в Android. Реализованный код соответствует современным рекомендациям по архитектуре и готов к расширению функционала.
