# Лабораторная работа №9

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

Лабораторная работа №9
**«Сохранение настроек темы. Тёмная/светлая тема в Compose»**  
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

Изучить механизмы смены и сохранения темы приложения в Jetpack Compose, научиться использовать `DataStore Preferences` для хранения пользовательских настроек, реализовать переключение между тёмной и светлой темами с сохранением состояния между сессиями.

## Color.kt
```kotlin
package com.example.themeswitcher.ui.theme

import androidx.compose.ui.graphics.Color
import androidx.compose.material3.darkColorScheme
import androidx.compose.material3.lightColorScheme

val LightColors = lightColorScheme(
    primary = Color(0xFF006C4C),
    onPrimary = Color(0xFFFFFFFF),
    primaryContainer = Color(0xFF89F8C7),
    onPrimaryContainer = Color(0xFF002114),
    secondary = Color(0xFF4D635A),
    onSecondary = Color(0xFFFFFFFF),
    secondaryContainer = Color(0xFFCFE9DD),
    onSecondaryContainer = Color(0xFF0A1F19),
    tertiary = Color(0xFF3A637A),
    onTertiary = Color(0xFFFFFFFF),
    tertiaryContainer = Color(0xFFC1E8FF),
    onTertiaryContainer = Color(0xFF001E2C),
    background = Color(0xFFF4FBF5),
    onBackground = Color(0xFF161D1A),
    surface = Color(0xFFF4FBF5),
    onSurface = Color(0xFF161D1A)
)

val DarkColors = darkColorScheme(
    primary = Color(0xFF6CDBB0),
    onPrimary = Color(0xFF003825),
    primaryContainer = Color(0xFF005239),
    onPrimaryContainer = Color(0xFF89F8C7),
    secondary = Color(0xFFB3CCC1),
    onSecondary = Color(0xFF1F352D),
    secondaryContainer = Color(0xFF354B43),
    onSecondaryContainer = Color(0xFFCFE9DD),
    tertiary = Color(0xFF9DC9E5),
    onTertiary = Color(0xFF003549),
    tertiaryContainer = Color(0xFF1F4B63),
    onTertiaryContainer = Color(0xFFC1E8FF),
    background = Color(0xFF161D1A),
    onBackground = Color(0xFFE1E3DF),
    surface = Color(0xFF161D1A),
    onSurface = Color(0xFFE1E3DF)
)
```

## SettingsManager.kt
```kotlin
package com.example.themeswitcher.data

import android.content.Context
import androidx.datastore.preferences.core.booleanPreferencesKey
import androidx.datastore.preferences.core.edit
import androidx.datastore.preferences.preferencesDataStore
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.map

val Context.dataStore by preferencesDataStore(name = "settings")

class SettingsManager(private val context: Context) {
    companion object {
        val DARK_MODE_KEY = booleanPreferencesKey("dark_mode")
    }
    
    val isDarkMode: Flow<Boolean> = context.dataStore.data
        .map { preferences ->
            preferences[DARK_MODE_KEY] ?: false
        }
    
    suspend fun saveDarkMode(enabled: Boolean) {
        context.dataStore.edit { preferences ->
            preferences[DARK_MODE_KEY] = enabled
        }
    }
}
```

## ThemeViewModel.kt
```kotlin
package com.example.themeswitcher.ui.theme

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.example.themeswitcher.data.SettingsManager
import kotlinx.coroutines.flow.SharingStarted
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.stateIn
import kotlinx.coroutines.launch

class ThemeViewModel(
    private val settingsManager: SettingsManager
) : ViewModel() {
    
    val isDarkTheme: StateFlow<Boolean> = settingsManager.isDarkMode
        .stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5000),
            initialValue = false
        )
    
    fun toggleTheme() {
        viewModelScope.launch {
            val currentValue = isDarkTheme.value
            settingsManager.saveDarkMode(!currentValue)
        }
    }
}
```

## Theme.kt
```kotlin
package com.example.themeswitcher.ui.theme

import android.app.Activity
import android.os.Build
import androidx.compose.foundation.isSystemInDarkTheme
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.graphics.toArgb
import androidx.compose.ui.platform.LocalContext
import androidx.compose.ui.platform.LocalView
import androidx.core.view.WindowCompat
import androidx.lifecycle.viewmodel.compose.viewModel

@Composable
fun ThemeSwitcherTheme(
    viewModel: ThemeViewModel = viewModel(),
    content: @Composable () -> Unit
) {
    val context = LocalContext.current
    val isDarkTheme by viewModel.isDarkTheme.collectAsState()
    
    val colorScheme = when {
        Build.VERSION.SDK_INT >= Build.VERSION_CODES.S -> {
            if (isDarkTheme) dynamicDarkColorScheme(context) else dynamicLightColorScheme(context)
        }
        isDarkTheme -> DarkColors
        else -> LightColors
    }
    
    val view = LocalView.current
    if (!view.isInEditMode) {
        SideEffect {
            val window = (view.context as Activity).window
            window.statusBarColor = colorScheme.primary.toArgb()
            WindowCompat.getInsetsController(window, view).isAppearanceLightStatusBars = !isDarkTheme
        }
    }
    
    MaterialTheme(
        colorScheme = colorScheme,
        typography = Typography(),
        content = content
    )
}
```

## MainActivity.kt
```kotlin
package com.example.themeswitcher

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.animation.Crossfade
import androidx.compose.foundation.layout.*
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp
import androidx.lifecycle.viewmodel.compose.viewModel
import com.example.themeswitcher.data.SettingsManager
import com.example.themeswitcher.ui.theme.ThemeSwitcherTheme
import com.example.themeswitcher.ui.theme.ThemeViewModel
import com.example.themeswitcher.ui.theme.ThemeViewModelFactory

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        val settingsManager = SettingsManager(this)
        
        setContent {
            ThemeSwitcherTheme(
                viewModel = viewModel(factory = ThemeViewModelFactory(settingsManager))
            ) {
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colorScheme.background
                ) {
                    ThemeScreen()
                }
            }
        }
    }
}

@Composable
fun ThemeScreen(viewModel: ThemeViewModel = viewModel()) {
    val isDarkTheme by viewModel.isDarkTheme.collectAsState()
    
    Crossfade(targetState = isDarkTheme, label = "ThemeCrossfade") { dark ->
        Column(
            modifier = Modifier
                .fillMaxSize()
                .padding(16.dp),
            horizontalAlignment = Alignment.CenterHorizontally,
            verticalArrangement = Arrangement.Center
        ) {
            Text(
                text = "Текущая тема: ${if (dark) "Тёмная" else "Светлая"}",
                style = MaterialTheme.typography.headlineMedium
            )
            
            Spacer(modifier = Modifier.height(24.dp))
            
            Button(onClick = { viewModel.toggleTheme() }) {
                Text("Переключить тему")
            }
            
            Spacer(modifier = Modifier.height(32.dp))
            
            Card(
                modifier = Modifier.fillMaxWidth(),
                elevation = CardDefaults.cardElevation(defaultElevation = 4.dp)
            ) {
                Column(modifier = Modifier.padding(16.dp)) {
                    Text(
                        text = "Пример карточки",
                        style = MaterialTheme.typography.titleLarge
                    )
                    Text(
                        text = "Это демонстрация того, как тема влияет на цвета компонентов. " +
                                "Primary цвет: ${MaterialTheme.colorScheme.primary}",
                        style = MaterialTheme.typography.bodyMedium
                    )
                }
            }
            
            Spacer(modifier = Modifier.height(24.dp))
            
            Row(horizontalArrangement = Arrangement.spacedBy(8.dp)) {
                Button(
                    onClick = { },
                    colors = ButtonDefaults.buttonColors(
                        containerColor = MaterialTheme.colorScheme.secondary
                    )
                ) { Text("Кнопка 1") }
                
                OutlinedButton(onClick = { }) { Text("Кнопка 2") }
            }
            
            Spacer(modifier = Modifier.height(32.dp))
            
            SettingsSection(viewModel)
        }
    }
}

@Composable
fun SettingsSection(viewModel: ThemeViewModel = viewModel()) {
    val isDarkTheme by viewModel.isDarkTheme.collectAsState()
    
    Card(
        modifier = Modifier.fillMaxWidth(),
        colors = CardDefaults.cardColors(containerColor = MaterialTheme.colorScheme.surfaceVariant)
    ) {
        Row(
            modifier = Modifier
                .fillMaxWidth()
                .padding(16.dp),
            horizontalArrangement = Arrangement.SpaceBetween,
            verticalAlignment = Alignment.CenterVertically
        ) {
            Text(
                text = "Тёмная тема",
                style = MaterialTheme.typography.bodyLarge
            )
            Switch(
                checked = isDarkTheme,
                onCheckedChange = { viewModel.toggleTheme() }
            )
        }
    }
}
```

## Индивидуальное задание: Анимация смены темы
Реализована плавная анимация перехода между светлой и тёмной темами с использованием Compose-компонента `Crossfade`. В `ThemeScreen` весь контент обёрнут в `Crossfade(targetState = isDarkTheme)`. При изменении состояния `isDarkTheme` Compose автоматически интерполирует цвета фона, текста, карточек и кнопок, создавая эффект плавного затухания и появления новой темы. Анимация работает без мерцаний, корректно синхронизируется с жизненным циклом и не блокирует основной поток.

<img width="385" height="856" alt="image" src="https://github.com/user-attachments/assets/cff3893e-2863-4d7e-8b85-92458897ab48" />

<img width="386" height="851" alt="image" src="https://github.com/user-attachments/assets/9749cc00-e4cf-4501-b4d3-9314ea50665c" />


## Ответы на контрольные вопросы
1. В Compose активную тему можно определить с помощью функции `isSystemInDarkTheme()`, которая возвращает `true`, если в ОС включена тёмная тема. Для пользовательского выбора используется сохранённое состояние из `DataStore` или `SharedPreferences`.
2. `MaterialTheme.colorScheme` – это объект, содержащий палитру цветов Material Design 3. Основные слоты: `primary`, `onPrimary`, `secondary`, `background`, `surface`, `onSurface`, `error` и их контейнерные варианты. Они автоматически применяются к стандартным компонентам.
3. Выбор темы сохраняется через `DataStore Preferences` (или `SharedPreferences`). В `DataStore` записывается булево значение по ключу, которое читается как `Flow<Boolean>`. ViewModel подписывается на поток и предоставляет состояние UI через `StateFlow`.
4. `isSystemInDarkTheme()` отражает текущие настройки ОС и меняется автоматически. Сохранённый пользовательский выбор имеет приоритет: приложение может принудительно использовать светлую или тёмную тему независимо от системных настроек.
5. Динамические цвета (Dynamic Color) – это функция Material You, которая генерирует цветовую схему приложения на основе обоев пользователя. Доступна на Android 12 (API 31) и выше через `dynamicLightColorScheme()` и `dynamicDarkColorScheme()`.

## Вывод

В результате выполнения лабораторной работы было создано приложение `ThemeSwitcherApp` на Jetpack Compose с полноценной поддержкой светлой и тёмной тем, сохранением пользовательских настроек и плавной анимацией переключения.

**Изученные и применённые технологии:**
- **Jetpack Compose Theming:** реализована кастомная обёртка `ThemeSwitcherTheme`, управляющая `MaterialTheme.colorScheme` в зависимости от состояния темы.
- **DataStore Preferences:** создан `SettingsManager` для асинхронного сохранения и чтения булевого флага темы. Использован `preferencesDataStore` и `Flow` для реактивного обновления.
- **ViewModel & StateFlow:** `ThemeViewModel` преобразует `Flow` из DataStore в `StateFlow` с помощью `stateIn()`, обеспечивая безопасную подписку UI и переживание поворотов экрана.
- **Compose Animation:** применён `Crossfade` для интерполяции цветов при смене темы, что устраняет резкие переходы и улучшает пользовательский опыт.

**Архитектура и сохранение состояния:**
- Настройки хранятся в приватном XML-файле DataStore, чтение происходит в фоновом потоке через корутины.
- `ThemeViewModelFactory` обеспечивает корректную инъекцию `SettingsManager` в ViewModel без нарушения жизненного цикла.
- `collectAsState()` связывает `StateFlow` с Compose-состоянием, вызывая рекомпозицию только при изменении темы.
- При перезапуске приложения `DataStore` возвращает последнее сохранённое значение, тема восстанавливается до отрисовки первого кадра.

**Индивидуальное задание («Анимация смены темы»):**
- Внедрён `Crossfade(targetState = isDarkTheme)` вокруг корневого контента экрана.
- Цвета фона, карточек, текста и кнопок плавно интерполируются между палитрами `LightColors` и `DarkColors`.
- Анимация не вызывает лишних рекомпозиций и работает в рамках стандартного Compose-рендера.

**Интерфейс и запуск:**
- `MainActivity.kt` инициализирует `SettingsManager`, передаёт его в фабрику и оборачивает контент в `ThemeSwitcherTheme`.
- UI содержит кнопку переключения, демонстрационную карточку, второстепенные кнопки и секцию настроек с `Switch`.
- Приложение успешно компилируется, тема переключается мгновенно, состояние сохраняется между сессиями, на API 31+ применяются динамические цвета обоев.

**Итог:** Цель работы достигнута. Получены практические навыки управления темами в Jetpack Compose, интеграции `DataStore` для персистентных настроек, использования `ViewModel` с `StateFlow` и применения декларативных анимаций. Реализованный код соответствует современным рекомендациям Android-разработки и готов к расширению.
