# 9. 数据存储

想象一下：你正在浏览食谱，发现了一个你喜欢的。你很匆忙，想把它收藏起来以便稍后查看。你能构建一个实现这个功能的应用吗？当然可以！继续阅读，了解如何实现。

在本章中，你的目标是学习如何使用 Data Store 库将重要信息保存到设备中。

## 保存数据

将数据保存到设备上有三种主要方式：

2. 将格式化数据（如 JSON）写入文件。
4. 使用库将简单数据写入共享位置。
6. 使用 SQLite 数据库。

将数据写入文件很简单，但需要你处理正确格式和顺序的数据读写。对于更复杂的数据，你可以将信息保存到本地数据库。

### 为什么要保存小块数据？

保存小块数据有很多原因。例如，当用户登录时，你可以保存用户 ID，或者记录用户是否已登录。你还可以保存引导状态或用户收藏的数据，以便之后查看。

注意，保存到共享位置的这些简单数据在用户卸载应用时会丢失。

本章的目标是展示一个之前输入的搜索字符串菜单，让用户可以重新选择搜索。即使用户重启应用，这些内容也会显示。第二个目标是恢复当前选择的屏幕。例如，如果用户正在查看杂货屏幕，如果他们在长时间后回来并且应用重启，他们希望返回到之前所在的屏幕。以下是主屏幕的样子：

 ![](./Android Fundamentals by Tutorials, Chapter 9_Data Store_ Kodeco_files/original.png)

#### SharedPreferences

Android 有一个内置接口，名为 **SharedPreferences**，它在每个版本的 Android 中都可用。你可以从任何 **Context** 中检索任何偏好设置。你也可以从 **Activity** 中获取完整的接口。

> **注意**：**SharedPreferences** 方法已被弃用。我们将使用更新的库。

#### 它是如何工作的？

**SharedPreferences** 使用系统将数据存储到文件中。这些是一些小块信息，如整数、字符串或布尔值。它有几组函数调用：

2. `getXXX()`：Get 方法检索特定数据类型的数据。
4. `contains()`：此方法检查键是否存在。
6. `edit()`：返回一个编辑器类来进行更改。

#### 编辑器

要进行更改，你需要调用 `edit()` 方法获取编辑器。然后你可以访问以下方法：

2. `putXXX()`：Set 方法保存特定数据类型的数据。
4. `clear()`：此方法删除所有保存的数据。
6. `remove()`：此方法删除特定值。
8. `apply()` 或 `commit()`：保存所做的更改。现在推荐使用 `apply()` 方法，因为它是异步的。但是，如果你需要立即保存数据，请使用 `commit()` 方法。

除了 `clear()` 方法外，所有这些方法都使用 **key** 来访问项目。通过给库提供一个唯一的键，你可以存储、检索和删除特定项目。这里是一个例子：

```kotlin
// 1
val sharedPreferences = getSharedPreferences("MyPreferences", Context.MODE_PRIVATE)
// 2
if (sharedPreferences.contains("MyKey")) {
  // 3
  val myValue = sharedPreferences.getString("MyKey", "MyDefault")
  Timber.d("My Value is $myValue")
}
```

2. 使用文件名"MyPreferences"并设为私有（你总是会希望将其设为私有）来检索 **SharedPreferences** 实例。
4. 首先检查确保键存在。
6. 使用键"MyKey"检索字符串值，如果不存在则使用默认值"MyDefault"。

这里你使用给定键检索字符串。以下是你可以存储和检索的类型：

+ String
+ Boolean
+ Int
+ Long
+ Float

注意，你不能存储比这些更复杂的内容。这意味着类或数据类不能被存储，除非你单独保存每个部分。

## DataStore 库

Google 一直在转向使用外部库而不是仅仅更新系统。这一策略为他们提供了灵活性，可以频繁推出对库的修改，用户可以立即访问这些修改。他们没有更新系统中的 **SharedPreferences** 代码，而是创建了一个名为 **DataStore** 的新库。这是一个用于存储键/值对的现代库。DataStore 库使用 **DataStore** 接口，设计略有不同。如果你查看定义，它相当简单：

```kotlin
interface DataStore<T> {
  val data: Flow<T>
  suspend fun updateData(transform: suspend (t: T) -> T): T
}
```

要创建一个名为 `dataStore` 的变量：

```kotlin
val Context.dataStore: DataStore<Preferences> by preferencesDataStore(name = "settings")
```

这是 `Context` 的扩展，使用 `preferencesDataStore` 的委托属性。你可以使用任何你想要的名称。这里偏好文件将被命名为"settings"。

要访问条目，你使用字符串键，就像旧系统一样，但首先必须围绕键创建一个变量。如果你尝试检索字符串值，你可以使用：

```kotlin
val prefKey = stringPreferencesKey("MyKey")
```

该库还使用 Kotlin 的更现代的 Flow 类，来自协程库。要检索值，你可以执行：

```kotlin
context.dataStore.data.first()[prefKey]
```

`first()` 方法被视为"终止"操作符 - 它只会检索第一个值。

> 重要提示：为避免意外行为，不要创建多个指向同一偏好文件的 DataStore 实例。

#### 添加 DataStore 库

如果你正在跟随前几章的应用程序，请打开它并继续使用。如果没有，只需找到本章的 **projects** 文件夹，在 Android Studio 中打开 **starter**。打开 **gradle/libs.versions.toml**。在 **versions** 部分末尾添加：

```toml
prefsVersion = "1.0.0"
```

然后在 **\[libraries\]** 部分末尾添加：

```toml
# Preferences
prefs = {module = "androidx.datastore:datastore-preferences", version.ref = "prefsVersion" }
```

执行 gradle 同步。在 **app/build.gradle.kts** 中的 timber 库后添加：

```kotlin
implementation(libs.prefs)
```

再次执行 gradle 同步。

#### 保存 UI 状态

在本节中，你将使用 **DataStore** 保存已保存搜索列表。之后，你还将保存用户选择的标签，使应用始终打开最后选择的标签。

首先，转到 **app/src/main/java/com/kodeco/recipefinder/data** 目录，创建一个名为 **Prefs.kt** 的新 Kotlin 文件。添加以下内容：

```kotlin
import android.content.Context
import androidx.datastore.core.DataStore
import androidx.datastore.preferences.core.Preferences
import androidx.datastore.preferences.core.edit
import androidx.datastore.preferences.core.intPreferencesKey
import androidx.datastore.preferences.core.stringPreferencesKey
import androidx.datastore.preferences.preferencesDataStore
import kotlinx.coroutines.flow.first

class Prefs(val context: Context) {
  // TODO: Add dataStore
  // TODO: Add saveString
  // TODO: Add getString
  // TODO: Add saveInt
  // TODO: Add getInit
  // TODO: Add hasKey
}
```

现在将 `// TODO: Add dataStore` 替换为：

```kotlin
private val Context.dataStore: DataStore<Preferences> by preferencesDataStore(name = "recipes")
```

这与示例类似，但你将文件命名为"recipes"。接下来，将 `// TODO: Add saveString` 替换为：

```kotlin
// 1
suspend fun saveString(key: String, value: String) {
  // 2
  val prefKey = stringPreferencesKey(key)
  // 3
  context.dataStore.edit { prefs ->
      // 4
      prefs[prefKey] = value
  }
}
```

2. 传入要保存的键和值。
4. 创建字符串偏好键。
6. 开始编辑。
8. 为给定键设置值。

现在将 `// TODO: Add getString` 替换为：

```kotlin
suspend fun getString(key: String): String? {
  val prefKey = stringPreferencesKey(key)
  return context.dataStore.data.first()[prefKey]
}
```

这与之前开始方式相同，但使用 `data.first()` 方法通过键作为索引检索条目。

替换接下来的两个 TODO：

```kotlin
suspend fun saveInt(key: String, value: Int) {
  val prefKey = intPreferencesKey(key)
  context.dataStore.edit { prefs ->
      prefs[prefKey] = value
  }
}
suspend fun getInt(key: String): Int? {
  val prefKey = intPreferencesKey(key)
  return context.dataStore.data.first()[prefKey]
}
```

这些方法与获取/保存字符串的方法相同，但针对 Int 类型。最后，替换 `// TODO: Add hasKey` 为：

```kotlin
suspend fun hasKey(key: String): Boolean {
  val prefKey = stringPreferencesKey(key)
  return context.dataStore.data.first().contains(prefKey)
}
```

这只是检查条目是否存在。现在我们有了这个用于保存和检索偏好的类，让我们修改代码来使用它。

首先打开 **RecipeApp.kt**。找到 `// TODO: Add Prefs` 并替换为：

```kotlin
lateinit var prefs: Prefs
```

现在，找到下一个 `// TODO: Add Prefs` 并替换为：

```kotlin
prefs = Prefs(this)
```

你现在已经设置了 `Prefs` 的实例，是时候使用它了。打开 **MainActivity.kt** 并替换 `// TODO: Add Pref Provider` 为：

```kotlin
val LocalPrefsProvider =
    compositionLocalOf<Prefs> { error("No prefs provided") }
```

这使用了 Compose 的提供者能力。导入所需的导入。注意，虽然这一开始会有错误，但你稍后会设置提供者。接下来，替换 `// TODO: Add Prefs` 为：

```kotlin
val prefs = remember { Prefs(context) }
```

在 `LocalNavigatorProvider provides navController` 之后添加：

```kotlin
LocalPrefsProvider provides (application as RecipeApp).prefs,
```

这将你的 Prefs 类提供给下面的任何可组合项。

#### 更新 ViewModels

现在你已经编写了 Prefs 类，你想要将其提供给你的视图模型以便能够保存数据。打开 **RecipeViewModel.kt**，在 `// TODO: Add Prefs` 处，修改构造函数以包含 prefs。构造函数应该看起来像：

```kotlin
class RecipeViewModel(private val prefs: Prefs) : ViewModel() {
```

在 `savePreviousSearches()` 方法中找到 `// TODO: Save previous searches` 并替换为：

```kotlin
viewModelScope.launch {
  val searchString = _uiState.value.previousSearches.joinToString(",")
  prefs.saveString(PREVIOUS_SEARCH_KEY, searchString)
}
```

这从 ui 状态中获取之前搜索的列表，并创建一个长字符串（用逗号分隔）。然后使用 prefs 类和 `PREVIOUS_SEARCH_KEY` 作为键保存字符串。要更新之前搜索的列表，找到 `addPreviousSearch` 方法并添加：

```kotlin
// 1
if (!_uiState.value.previousSearches.contains(searchString)) {
  val updatedSearches = mutableListOf<String>()
  // 2
  updatedSearches.addAll(uiState.value.previousSearches)
  updatedSearches.add(searchString)
  // 3
  _uiState.value = _uiState.value.copy(previousSearches = updatedSearches)
  // 4
  savePreviousSearches()
}
```

2. 检查确保搜索字符串在列表中不存在，以避免重复。
4. 使用新列表，添加所有之前的搜索和新的搜索。
6. 使用之前搜索的修改副本更新 UI 状态。
8. 使用你之前处理过的方法保存列表。

如果你现在尝试运行，你会在 **RecipeDetails** 中看到一个错误。打开它并找到 `// TODO: Add Prefs*`。添加：

```kotlin
val prefs = LocalPrefsProvider.current
```

这使用了之前定义的 **LocalPrefsProvider**。现在只需使用它的 `current` 版本。将视图模型的工厂更新为：

```kotlin
RecipeViewModel(prefs)
```

#### 练习

像刚才处理 **RecipeDetails** 一样转换其余方法。

对以下文件执行相同操作：

+ **GroceryList**
+ **ChipRow**
+ **RecipeList**
+ **SearchRow**
+ **ShowBookmarks**
+ **ShowRecipeList**

对于预览代码，使用给定的上下文创建一个新的 Prefs 类。如果你有任何问题，可以预览最终项目。现在，重启应用。进行搜索，比如"chicken"，然后点击三点菜单，查看搜索是否已保存。

 ![](./Android Fundamentals by Tutorials, Chapter 9_Data Store_ Kodeco_files/original(1).png)

现在，停止应用并重启。验证列表是否为空。为什么会这样？你尚未编写加载偏好设置的代码。

重新打开 **RecipeViewModel** 并替换 `// TODO: Retrieve previous searches` 为：

```kotlin
viewModelScope.launch {
  val previousSearchString = prefs.getString(PREVIOUS_SEARCH_KEY)
  if (!previousSearchString.isNullOrEmpty()) {
      val storedList = previousSearchString.split(",")
      _uiState.value = _uiState.value.copy(previousSearches = storedList.toMutableList())
  }
}
```

这与 `savePreviousSearches` 方法相反。该方法获取字符串列表，通过按","分割字符串来分离它，创建一个新列表并设置 UI 状态的前一个搜索列表。重启应用。你现在应该可以看到这些项目。

#### 保存选定的标签

在本节中，你将使用共享偏好设置来保存用户已导航到的当前 UI 标签。打开 **ui/MainScreen.kt**。首先获取 prefs 类的实例。将 `// TODO: Add Prefs` 替换为：

```kotlin
val prefs = LocalPrefsProvider.current
```

接下来，替换 `// TODO: Get screen position from prefs` 为：

```kotlin
val currentIndex = prefs.getInt(CURRENT_INDEX_KEY)
if (currentIndex != null) {
  selectedIndex.intValue = currentIndex
}
```

这将获取标签的当前索引，如果存在，将设置 selectedIndex.value。接下来，替换 `// TODO: Save screen position to prefs` 为：

```kotlin
scope.launch {
  prefs.saveInt(CURRENT_INDEX_KEY, 0)
}
```

这通过在协程作用域中运行（因为这需要异步完成）来保存索引。最后，替换下一个 TODO 为：

```kotlin
scope.launch {
  prefs.saveInt(CURRENT_INDEX_KEY, 1)
}
```

这个调用的唯一区别是它使用索引 1 而不是 0。重新运行应用。点击底部的 Groceries 图标。关闭应用并重启。它应该以选定的杂货标签开始。

## 要点

+ 在应用程序中保存数据有多种方式：保存到**文件**，保存在**共享偏好设置**中，以及保存到 **SQLite** 数据库中。

+ 共享偏好设置最适合存储简单的**键值对**，如`字符串`、`数字`和`布尔值`等原始类型。

+ 使用**共享偏好设置**的一个例子是保存用户正在查看的标签，这样下次用户启动应用时，他们会被带到同一个标签。

## 接下来去哪里？

在本章中，你学习了如何持久化简单数据。

如果你想了解更多关于 Android **SharedPreferences** 的信息，请访问 [https://developer.android.com/reference/kotlin/android/content/SharedPreferences?hl=en](https://developer.android.com/reference/kotlin/android/content/SharedPreferences?hl=en)。

要了解 **DataStore**，请访问 [https://developer.android.com/topic/libraries/architecture/datastore](https://developer.android.com/topic/libraries/architecture/datastore)。

在下一章中，你将学习使用 Room 在数据库中存储数据。下次见！
