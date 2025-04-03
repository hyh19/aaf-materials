# 8. 网络

从网络加载数据并在 UI 上显示是应用程序的一项常见任务。在本章中，你将学习如何进行网络调用，将这些调用中的数据转换为模型类以及处理异步操作。

你的目标是学习如何使用网络库在设备上保存重要信息。

## 入门

在 Android Studio 中打开本章的**起始**项目，然后运行应用程序。

注意底部的两个选项卡——当你点击它们时，每个选项卡显示不同的屏幕。"Recipes"（菜谱）选项卡如下所示：

![1743666000773](https://cdn.jsdelivr.net/gh/hyh19/images3@master/1743665998332.png)

"Groceries"（食材）选项卡如下所示：

![1743666027499](https://cdn.jsdelivr.net/gh/hyh19/images3@master/1743666024804.png)

完成后，你可以搜索菜谱，将它们显示在网格中，收藏你想保留的菜谱，并显示这些餐点所需的食材清单。

### 协程

异步编程需要在不同的线程上执行任务。通常，有一个用于 UI 绘制的"主"线程。要在不减慢 UI 线程的情况下执行其他任务，你需要一种方法来执行它们，而不会降低用户正在操作的速度。如何同时做两件或更多的事情？当只有一个处理器时，系统必须快速在任务之间切换，给人多任务处理的感觉。现在多核处理器存在，不同的任务可以在不同的核心上运行。你可以拥有任意数量的线程。但通常，你只会有 UI、IO 和默认线程或"调度器"。例如，如果你需要从互联网获取一些数据，你不会希望在下载数据时 UI 冻结。你会使用默认调度器（与 UI 调度器不同）来下载数据，然后切换到 UI 调度器来显示这些数据。

Kotlin 有用于协程的 **kotlinx.coroutines** 库。欲了解更多详情，请参阅：[https://kotlinlang.org/docs/coroutines-guide.html](https://kotlinlang.org/docs/coroutines-guide.html)。协程就像迷你线程。你可以使用数千个而不会引起任何问题（与线程不同）。Kotlin 使用 `suspend` 关键字将函数标记为异步。如果你想调用一个挂起函数，你必须要么在另一个挂起函数中，要么使用协程构建器启动一个。最常见的是 `launch` 方法。这个方法来自 CoroutineScope。如果你见过 Android 的 `ViewModel`，你会注意到它有自己内置的 `viewModelScope`。如果你查看 `viewModelScope` 代码：

```kotlin
public val ViewModel.viewModelScope: CoroutineScope
    // 1
    get() {
        // 2
        val scope: CoroutineScope? = this.getTag(JOB_KEY)
        if (scope != null) {
            return scope
        }
        // 3
        return setTagIfAbsent(
            JOB_KEY,
            CloseableCoroutineScope(SupervisorJob() + Dispatchers.Main.immediate)
        )
    }
```

你会看到它：

1. 创建一个 getter 函数。
2. 检查作用域是否已经存在并返回它。
3. 创建并存储一个由 `SupervisorJob` 和主调度器（UI 线程）组成的新作用域。

作用域结合了一个任务、一个调度器和可能的异常处理程序。这段代码使用了 `SupervisorJob`。这很重要，因为普通的 `Job` 在任务死亡时会终止任务。如果在使用 `SupervisorJob` 时任何子任务死亡，主协程仍会继续运行。

看一下 `launch` 方法：

```kotlin
public fun CoroutineScope.launch(
    context: CoroutineContext = EmptyCoroutineContext,
    start: CoroutineStart = CoroutineStart.DEFAULT,
    block: suspend CoroutineScope.() -> Unit
): Job {
  ...
}
```

通常，你会在 ViewModel 中这样使用 `launch`：

```kotlin
viewModelScope.launch {
  ...
}
```

在这个例子中，最后一个参数（`block`）是你的代码，并使用默认的 `EmptyCoroutineContext` 作为协程上下文。

#### 添加协程库

要添加协程库，请在 **Gradle** 目录中打开 **libs.versions.toml** 文件。在 **versions** 部分下，添加：

```toml
kotlinx-coroutines = "1.7.2"
```

然后，在 **libraries** 部分末尾，添加：

```toml
coroutines-android = { module = "org.jetbrains.kotlinx:kotlinx-coroutines-android", version.ref = "kotlinx-coroutines" }
```

打开应用模块的 **build.gradle.kts** 并在 **dependencies** 部分添加以下内容：

```kotlin
implementation(libs.coroutines.android)
```

注意在 coroutines-android 中如何使用 "." 而不是 "-"？这在 **build.gradle.kts** 文件中填写依赖项时提供了对象表示法方法。它还有助于将类似的库分组在一起，并允许在输入依赖项时自动完成。现在，进行 Gradle 同步以添加库。

### 流

大多数 Android 开发者已经使用 LiveData 库一段时间了。它提供了一种通知 UI 事件的方式，并处理 Android 生命周期。新秀是 **Flows**。与 LiveData 一样，你将在 ViewModel 中创建一个可变流，但仅暴露一个不可变版本。这里是一个例子：

```kotlin
private val _queryState = MutableStateFlow(QueryState())
val queryState = _queryState.asStateFlow()
```

私有的 `_queryState` 是一个可以改变的状态。UI 将监听 `queryState`。要监听事件，你可以这样做：

```kotlin
val uiState by viewModel.uiState.collectAsState()
```

如果你想对状态变化采取行动，你可以像这样 `collect` 状态变化：

```kotlin
val scope = rememberCoroutineScope()
LaunchedEffect(Unit) {
    scope.launch {
        viewModel.recipeListState.collect { state ->
            recipeListState.value = state
        }
    }
}
```

`collect` 方法需要一个协程作用域来监听事件，因为它是一个挂起函数。

`LaunchedEffect` 是一个可组合的副作用，当可组合项首次显示时启动一个协程。当可组合项从屏幕上移除时，它也会取消协程。这很重要，因为当可组合项不再可见时，你不希望继续监听事件。

### 网络请求

要从互联网检索信息，你需要发出网络请求。最简单的方法是使用库，而最好且最常用的库之一是 **Retrofit**。

#### 替代方案

在本章中，你将使用 Retrofit，但还有其他库存在：

+ **Fuel** ([https://github.com/kittinunf/fuel](https://github.com/kittinunf/fuel)) 是一个较新的库，正在更新中。
+ **Ktor** ([https://ktor.io/](https://ktor.io/)) 在多平台世界中使用，但你也可以在 Android 中使用它。

### Retrofit

Square 开发了 Retrofit。它是 Android 和 Java/Kotlin 的类型安全 HTTP 客户端。虽然有更新的库，但了解 Retrofit 将让你能够使用几乎任何应用程序。

Retrofit 通过创建接口并使用注解工作。然后，一个构建器使用该信息创建代码，为你进行调用。

#### 添加 Retrofit

要添加 Retrofit 库，请在 **Gradle** 目录中打开 **libs.versions.toml** 文件。在 **versions** 部分下，添加：

```toml
retrofit="2.9.0"
```

然后，在 **libraries** 部分，添加：

```toml
# Retrofit
retrofit = {module="com.squareup.retrofit2:retrofit", version.ref="retrofit" }
```

这会添加主要的 Retrofit 库。

打开应用模块的 **build.gradle.kts** 并在 **dependencies** 部分添加以下内容：

```kotlin
implementation(libs.retrofit)
```

现在，进行 Gradle 同步以添加库。

### 响应解析

要转换网络响应数据（通常作为 JSON 字符串返回），你需要一种简单的方法将字符串转换为模型类，反之亦然。你将使用 **Moshi** 库（也是由 Square 创建的）。Moshi 允许你将 JSON 解析为 Kotlin 类。还有其他解析库（常见的是 Gson），但 Moshi 更新且更现代。

#### 添加 Moshi

要添加 Moshi 库，请在 **Gradle** 目录中打开 **libs.versions.toml** 文件。

在 **versions** 部分下，添加：

```toml
moshi="1.15.0"
```

在 **libraries** 部分，添加：

```toml
moshi-kotlin = {module="com.squareup.moshi:moshi-kotlin", version.ref="moshi" }
retrofit-moshi-converter = {module="com.squareup.retrofit2:converter-moshi", version.ref="retrofit" }
```

打开应用模块的 **build.gradle.kts** 并在 **dependencies** 部分添加以下内容：

```kotlin
implementation(libs.moshi.kotlin)
implementation(libs.retrofit.moshi.converter)
```

## 注册菜谱 API

对于你的远程内容，你将使用 Spoonacular Recipe API。在浏览器中打开此链接：[https://spoonacular.com/food-api](https://spoonacular.com/food-api)。

点击右上角的 **Start Now** 按钮创建一个帐户。

![1743666490688](https://cdn.jsdelivr.net/gh/hyh19/images3@master/1743666488906.png)  

填写电子邮件和密码，然后点击复选框接受条款和条件。最后，点击 **Sign up** 按钮。使用免费套餐完成该过程。

![1743666514295](https://cdn.jsdelivr.net/gh/hyh19/images3@master/1743666512533.png)  

你将看到以下内容：

![1743666532537](https://cdn.jsdelivr.net/gh/hyh19/images3@master/1743666531071.png)  

确认电子邮件后，点击此链接并登录。

![1743666554656](https://cdn.jsdelivr.net/gh/hyh19/images3@master/1743666553019.png)  

你会看到 API 控制台。一旦你开始发出请求，你会看到图表填满。

![1743666572883](https://cdn.jsdelivr.net/gh/hyh19/images3@master/1743666571378.png)  

现在，转到文档：

![1743666588755](https://cdn.jsdelivr.net/gh/hyh19/images3@master/1743666587094.png)  

在这里，你可以看到搜索菜谱的文档：

![1743666608620](https://cdn.jsdelivr.net/gh/hyh19/images3@master/1743666606081.png)  

如果你向下滚动，你可以看到许多返回的字段。但你对大多数这些并不感兴趣。

你看到 **path** 和你将要进行的 `GET` 请求可用参数列表。

![1743666627454](https://cdn.jsdelivr.net/gh/hyh19/images3@master/1743666625625.png)  

这个页面上的 API 信息比你的应用需要的要多得多，所以你可能想把它收藏起来以备将来使用。

点击 **My Console**，然后点击 **Profile** 部分：

![1743666652244](https://cdn.jsdelivr.net/gh/hyh19/images3@master/1743666650728.png)  

点击 **Show/Hide API Key**。复制 API Key 并将其保存在安全的地方。

### 使用你的 API Key

对于下一步，你需要使用你的新 API Key。

> **注意**：API 的免费开发者版本是有速率限制的。如果你经常使用 API，你可能会收到一些带有错误的 JSON 响应和警告你关于限制的电子邮件。

### SpoonacularService

要从菜谱 API 获取数据，你将创建一个 Kotlin 接口和一个对象来管理连接。这个 Kotlin 类文件包含你的 API Key、ID 和 URL。你还需要一个响应的模型类。

在项目侧边栏中，右键点击 **main/java/com/kodeco/recipefinder/data/models**，创建一个新的 Kotlin 文件并命名为 **SearchRecipesResponse.kt**。文件打开后，导入以下内容：

```kotlin
import com.squareup.moshi.Json
```

然后，添加数据类：

```kotlin
data class SearchRecipesResponse(
  val offset: Int,
  val number: Int,
  val totalResults: Int,
  @Json(name = "results")
  val recipes: List<Recipe>
)
```

你使用 `@Json` 注解将名为 "results" 的 JSON 字段映射到名为 "recipes" 的变量。

接下来，打开名为 **SpoonacularService.kt** 的 Kotlin 文件。导入以下内容：

```kotlin
import com.kodeco.recipefinder.data.models.RecipeInformationResponse
import com.kodeco.recipefinder.data.models.SearchRecipesResponse
import com.kodeco.recipefinder.viewmodels.PAGE_SIZE
import com.squareup.moshi.Moshi
import com.squareup.moshi.kotlin.reflect.KotlinJsonAdapterFactory
import retrofit2.Retrofit
import retrofit2.converter.moshi.MoshiConverterFactory
import retrofit2.http.GET
import retrofit2.http.Path
import retrofit2.http.Query
```

然后，添加：

```kotlin
const val apiKey = "<Replace with API Key>"
```

> **注意**：请记住，你永远不应该将 API key 提交到公共存储库。你应该始终安全地存储它并从那里引用它。对于本教程，为了简单起见，你只需将其添加到代码中。如果你有兴趣了解如何存储 API keys 和秘密等内容，Google 推荐的一种常见方法是使用 [secrets-gradle-plugin](https://github.com/google/secrets-gradle-plugin) 库。

用你创建账户时获取的密钥替换这个字符串。

接下来，这个服务进行两个网络调用：

+ 一个是用查询字符串搜索菜谱 ([https://spoonacular.com/food-api/docs#Search-Recipes-Complex](https://spoonacular.com/food-api/docs#Search-Recipes-Complex))。
+ 另一个是获取特定菜谱的详细信息 ([https://spoonacular.com/food-api/docs#Get-Recipe-Information](https://spoonacular.com/food-api/docs#Get-Recipe-Information))。

添加：

```kotlin
interface SpoonacularService {
   // 1
  @GET("recipes/complexSearch?&apiKey=$apiKey")
  // 2
  suspend fun queryRecipes(
    @Query("query") query: String,
    @Query("offset") offset: Int,
    @Query("number") number: Int = PAGE_SIZE
  ): SearchRecipesResponse

  // 3
  @GET("recipes/{id}/information?includeNutrition=false&apiKey=$apiKey")
  suspend fun queryRecipe(@Path("id") id: Int): RecipeInformationResponse
}
```

这段代码：

1. 使用 **@GET** 注解指定用于此调用的 URL。注意它不包含 URL 的开始部分。你将在后面添加这个。
2. 创建一个接受三个查询参数的挂起函数：一个用于查询，一个用于查询列表中的起始点，第三个用于返回结果的数量。该函数返回一个 `SearchRecipesResponse`。
3. 创建另一个挂起函数。这将查询具有给定 ID 的特定菜谱。它响应一个 `RecipeInformationResponse`。

注意你将 API Key 放入 URL 中，第二个调用使用 `{id}` 在 URL 中替换 ID。要创建 Retrofit 的实例，你需要一些方法来使用 Retrofit 构建器。在 **SpoonacularService.kt** 中添加以下内容：

```kotlin
object RetrofitInstance {
  // 1
  private const val BASE_URL = "https://api.spoonacular.com/"

  // 2
  private fun provideMoshi(): Moshi =
    Moshi
      .Builder()
      .addLast(KotlinJsonAdapterFactory())
      .build()

  // 3
  private val retrofit: Retrofit by lazy {
    Retrofit.Builder()
      .baseUrl(BASE_URL)
      .addConverterFactory(MoshiConverterFactory.create(provideMoshi()))
      .build()
  }

  // 4
  val spoonacularService: SpoonacularService by lazy {
      retrofit.create(SpoonacularService::class.java)
  }
}
```

通过这段代码，你：

1. 提供调用的基本 URL。这段代码将其前置到你之前在接口中定义的所有服务。
2. 提供一个使用 `KotlinJsonAdapterFactory` 工厂来解析 JSON 的 Moshi 实例。
3. 使用 Retrofit 构建器组装基本 URL 和 Moshi。然后，调用 `build` 方法创建 Retrofit 实例。使用 `lazy` 意味着这不会被创建，直到它首次被使用，然后会被缓存而不会重新创建。
4. 一个供其他人用来进行服务调用的变量。

现在你已经拥有进行网络调用所需的一切。

### 实现 ViewModel

现在你已经编写了服务，你必须检索与查询字符串匹配的菜谱列表。打开 **viewmodels/RecipeViewModel.kt**。找到 `// TODO: Add Service` 并替换为：

```kotlin
private val spoonacularService = RetrofitInstance.spoonacularService
```

导入：

```kotlin
import com.kodeco.recipefinder.network.RetrofitInstance
```

这会给你一个服务实例。替换 `// TODO: query recipes` 为：

```kotlin
viewModelScope.launch {
  try {
    // 1
    val response = spoonacularService.queryRecipes(query, offset, number)
    // 2
    _recipeListState.value = response.recipes
    // 3
    _queryState.value =
        QueryState(query, offset, number, response.totalResults)
  } catch (e: Exception) {
    Timber.e(e, "Problems getting Recipes")
    // 4
    _recipeListState.value = listOf()
  }
}
```

这段代码：

1. 使用服务查询具有给定查询字符串、偏移量和菜谱数量的菜谱。
2. 用 API 响应中的新菜谱替换现有的菜谱列表。
3. 更新查询状态以包含最新信息。
4. 如果有任何错误，将列表设置为空列表。

导入以下内容：

```kotlin
import androidx.lifecycle.viewModelScope
import kotlinx.coroutines.launch
import timber.log.Timber
```

UI 监听菜谱列表状态（`recipeListState` 变量）。当有新列表可用时，菜谱列表状态通知 UI，UI 相应地更新。通过打开 **RecipeList.kt** 文件来检查这一点：

```kotlin
scope.launch {
  viewModel.recipeListState.collect { state ->
    recipeListState.value = state
  }
}
```

## 运行应用

通过按下**播放**按钮在设备或模拟器上运行应用。

![1743667092370](https://cdn.jsdelivr.net/gh/hyh19/images3@master/1743667090735.png)  

你会看到类似的内容：

![1743667109367](https://cdn.jsdelivr.net/gh/hyh19/images3@master/1743667104760.png)  

在搜索栏中输入查询（如 Sushi）：

![1743667119196](https://cdn.jsdelivr.net/gh/hyh19/images3@master/1743667117280.png)  

现在，按下**搜索**图标：

![1743667128311](https://cdn.jsdelivr.net/gh/hyh19/images3@master/1743667126860.png)  

进度指示器旋转一段时间后，你会看到一些菜谱：

![1743667143508](https://cdn.jsdelivr.net/gh/hyh19/images3@master/1743667141487.png)  

> 如果你没有看到任何菜谱，请确保你在 **SpoonacularService.kt** 文件中有 API Key。

### 使用 Moshi 的代码生成

目前，Moshi 使用反射将 JSON 转换为 Kotlin 类。但为了获得更好的性能，你可以使用 Moshi 的代码生成支持，它可以在编译时生成适配器。

要启用这个功能，在 **Gradle** 目录中打开 **libs.versions.toml** 文件，并在 **libraries** 部分替换：

```toml
moshi-kotlin = {module="com.squareup.moshi:moshi-kotlin", version.ref="moshi" }
```

替换为以下内容：

```toml
moshi = {module="com.squareup.moshi:moshi", version.ref="moshi" }
moshiCodeGen = {module="com.squareup.moshi:moshi-kotlin-codegen", version.ref="moshi" }
```

接下来，打开应用模块的 **build.gradle.kts** 文件，并在 **dependencies** 部分替换：

```kotlin
implementation(libs.moshi.kotlin)
```

替换为以下内容：

```kotlin
implementation(libs.moshi)
ksp (libs.moshiCodeGen)
```

**ksp** 命令是 Kotlin 符号处理 API。Moshi 代码生成插件为你的模型类创建转换器。现在，进行 Gradle 同步。

你不再需要 `KotlinJsonAdapterFactory`，所以打开 **SpoonacularService.kt** 并将 `provideMoshi()` 更新为以下内容：

```kotlin
private fun provideMoshi(): Moshi =
  Moshi
    .Builder()
    .build()
```

同时，删除以下导入：

```kotlin
import com.squareup.moshi.kotlin.reflect.KotlinJsonAdapterFactory
```

最后，你必须为所有你想要转换为 JSON 的类添加注解。打开 **SearchRecipesResponse.kt** 并添加 `@JsonClass` 注解：

```kotlin
@JsonClass(generateAdapter = true)
data class SearchRecipesResponse(
  val offset: Int,
  val number: Int,
  val totalResults: Int,
  @Json(name = "results")
  val recipes: List<Recipe>
)
```

同时，导入这个：

```kotlin
import com.squareup.moshi.JsonClass
```

`@JsonClass` 注解为你创建了一个适配器。当你构建项目时，它会为你创建一个名为 `SearchRecipesResponseJsonAdapter` 的文件。这包含了许多将 JSON 转换为你的类并返回的样板代码。

对 **main/java/com/kodeco/recipefinder/data/models** 中的其余文件执行相同的操作。

构建并运行应用以检查一切是否仍然正常工作。

### 详情屏幕

如果你点击一个菜谱，你会看到一个空白屏幕。是时候解决这个问题了。

打开 **RecipeViewModel.kt**。替换 `// TODO: Query a Recipe` 为：

```kotlin
// 1
viewModelScope.launch(Dispatchers.Default) {
  try {
    // 2
    val spoonacularRecipe = spoonacularService.queryRecipe(id)
    // 3
    _recipeState.value = spoonacularRecipe
  } catch (e: Exception) {
    Timber.e(e, "Problems getting Recipe for id $id")
    _recipeState.value = null
  }
}
```

通过这段代码，你：

1. 在默认调度器上启动一个新的协程。
2. 调用 SpoonacularService 获取完整的菜谱详情。
3. 设置详情屏幕监听的 Recipe 状态的值。

你需要导入这个：

```kotlin
import kotlinx.coroutines.Dispatchers
```

通过按下**重新播放**按钮重新运行应用。点击一个菜谱查看详情：

![1743667290514](https://cdn.jsdelivr.net/gh/hyh19/images3@master/1743667288316.png)  

如果你按下**书签**或**返回箭头**图标，你将返回到菜谱列表。

![1743667308742](https://cdn.jsdelivr.net/gh/hyh19/images3@master/1743667307272.png)  

## 要点

+ 协程是在后台运行代码的好方法。

+ 流替代了 LiveData，并提供了一种通知 UI 事件的方式。

+ Retrofit 是处理网络请求的优秀库。

+ Moshi 库提供了响应解析。

## 从这里去哪里？

在本章中，你学习了如何轻松检索网络数据。

要了解更多关于 **Retrofit** 的信息，请访问 [https://square.github.io/retrofit/](https://square.github.io/retrofit/)。

要了解更多关于 **Moshi** 的信息，请查看 [https://github.com/square/moshi](https://github.com/square/moshi)。

在下一章中，你将学习使用 Shared Preferences 存储数据。
