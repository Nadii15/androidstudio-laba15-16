# Student Planner - Студенческий планер

## Описание приложения
Приложение "Студенческий планер" - это многоэкранное Android-приложение для управления учебными дисциплинами. Оно позволяет просматривать список предметов, изучать подробную информацию о каждой дисциплине, а также получать доступ к профилю студента и настройкам приложения.

## Реализованные экраны

1. **Home Screen (Главный экран)** - список всех учебных дисциплин в виде карточек с краткой информацией
2. **Details Screen (Экран деталей)** - подробная информация о выбранной дисциплине с описанием курса
3. **Profile Screen (Профиль студента)** - информация о студенте, успеваемости и статистике
4. **Settings Screen (Настройки)** - настройки приложения с переключателями и пунктами меню

## Используемые технологии

- **Kotlin** - основной язык программирования
- **Jetpack Compose** - современный toolkit для создания UI
- **Navigation Compose** - библиотека для навигации между экранами
- **Material Design 3** - дизайн-система для создания интерфейса

## Схема навигации
Home Screen (home)
│
├──→ Details Screen (details/{subjectId})
│ └──→ [возврат назад]
│
├──→ Profile Screen (profile)
│ └──→ [возврат назад]
│
└──→ Settings Screen (settings)
└──→ [возврат назад]

## Структура проекта
app/
├── data/
│ └── Subject.kt # Модель данных и тестовые данные
├── navigation/
│ ├── Screen.kt # Sealed class с маршрутами
│ └── NavGraph.kt # Настройка NavHost
├── ui_model/
│ ├── HomeScreen.kt # Главный экран
│ ├── DetailsScreen.kt # Экран деталей
│ ├── ProfileScreen.kt # Экран профиля
│ └── SettingsScreen.kt # Экран настроек
└── MainActivity.kt # Основная активность


## Скриншоты

### Главный экран
![Home Screen](img/step7_navigation_working_FIO_1.png)

### Экран деталей дисциплины
![Details Screen](img/step7_navigation_working_FIO_2.png)

### Экран профиля
![Profile Screen](img/step7_navigation_working_FIO_3.png)

### Экран настроек
![Settings Screen](img/step7_navigation_working_FIO_4.png)

## Контрольные вопросы

### 1. Что такое NavController и для чего он используется?

**NavController** - это основной класс управления навигацией в приложении. Он отвечает за:
- Переходы между экранами (navigate)
- Управление стеком навигации (back stack)
- Передачу данных между экранами
- Обработку кнопки "Назад"

**rememberNavController()** создаёт NavController и сохраняет его при рекомпозиции Composable-функции. Это важно, потому что:
- Без `remember` при каждой рекомпозиции создавался бы новый контроллер
- Это привело бы к потере состояния навигации
- NavController должен существовать на протяжении всего жизненного цикла экрана

### 2. Как передать параметр в маршрут навигации?

**Процесс передачи параметра:**

1. **Определение маршрута с placeholder:**
```kotlin
object Details : Screen("details/{subjectId}") {
    fun createRoute(subjectId: String) = "details/$subjectId"
}
```
2. **Регистрация маршрута с аргументами:**
```kotlin
composable(
    route = Screen.Details.route,
    arguments = listOf(
        navArgument("subjectId") { type = NavType.StringType }
    )
) { backStackEntry ->
    val subjectId = backStackEntry.arguments?.getString("subjectId")
    DetailsScreen(subjectId = subjectId)
}
```
3.**Переход с параметром:**
```kotlin
navController.navigate(Screen.Details.createRoute("123"))
```
#### Разница между параметрами:
Обязательные - должны быть указаны всегда (например, ID предмета)
Опциональные - имеют значения по умолчанию, можно не указывать

### 3. Зачем использовать sealed class для маршрутов?

#### Преимущества sealed class:
1. Type-safety - компилятор проверяет все возможные экраны
2. Централизация - все маршруты в одном месте
3. Автодополнение - IDE предлагает доступные экраны
4. Защита от опечаток - нельзя ввести неверный маршрут
## Пример ошибки без sealed class:
```kotlin
// С обычными строками - возможна опечатка
navController.navigate("hom")  // Ошибка! Должно быть "home"
// Приложение скомпилируется, но экран не откроется

// С sealed class - ошибка на этапе компиляции
navController.navigate(Screen.Home.route)  // Безопасно
```
### 4. Что такое Back Stack и как им управлять?
**Back Stack** - это стек (LIFO - Last In First Out) экранов, которые посетил пользователь.
Схема back stack для Home → Profile → Settings:

| Settings  | ← Текущий экран (верх стека)
|-----------|
| Profile   |
|-----------|
| Home      | ← Начальный экран (низ стека)

При вызове popBackStack() на экране Settings:
- Settings удаляется из стека
- Отображается Profile (теперь верх стека)
- Back stack: Home → Profile

#### Управление back stack:
navController.popBackStack() - возврат назад
navController.navigate() - добавляет экран в стек
popUpTo() - удаление экранов из стека при навигации
### 5. Как работает startDestination в NavHost?
   **startDestination** - это начальный экран, который отображается при запуске приложения. 
   ```kotlin
NavHost(
    navController = navController,
    startDestination = Screen.Home.route,  // Первый экран
    modifier = modifier
)
```
**Какой экран покажется первым:**

- При запуске приложения отображается экран, указанный в startDestination
- В нашем случае - Home Screen (список дисциплин)

**Изменение startDestination:**
- Статически - нельзя изменить в runtime
- Динамически - можно передать как параметр в NavHost или использовать условную логику:

```kotlin
val startScreen = if (isLoggedIn) Screen.Home.route else Screen.Login.route
NavHost(navController, startDestination = startScreen) { ... }
```

### 6. Что произойдёт, если навигировать на несуществующий маршрут?
**Поведение NavController:**
- Приложение упадёт с исключением IllegalArgumentException
- Ошибка: "navigation destination is not found"
**Как обработать:**
1. Использовать sealed class - предотвращает ошибки на этапе компиляции
2. Проверка перед навигацией:

```kotlin
if (navController.currentBackStackEntry?.destination?.route != targetRoute) {
    navController.navigate(targetRoute)
}
```
Обработка исключений:
```kotlin
try {
    navController.navigate(route)
} catch (e: IllegalArgumentException) {
    // Обработка ошибки
}
```
### 7. Зачем нужен параметр launchSingleTop в навигации?
   launchSingleTop предотвращает создание дубликатов экранов в back stack.
  **Пример проблемы без launchSingleTop:**

Пользователь: Home → Profile → Home → Profile → Home
Back stack: Home → Profile → Home → Profile → Home

При нажатии "Назад" пользователь будет многократно возвращаться через одни и те же экраны.
С launchSingleTop:
```kotlin
navController.navigate("profile") {
    launchSingleTop = true
}
```

Влияние на back stack:
Если экран уже на вершине стека - не создаётся новый
Если экран есть в стеке, но не на вершине - можно использовать popUpTo для очистки

Автор
ФИО: Рыхлюк Надежда
Группа: Исп-233
Дата: 30.03.2026