# 8. Networking

Loading data from the network to show it on the UI is a very common task for apps. In this chapter, you’ll learn how to make network calls, convert the data from those calls to model classes and handle asynchronous operations.

Your goal is to learn how to use networking libraries to save important information on your device.

## Getting Started

Open the **starter** project for this chapter in Android Studio and then run the app.

Notice the two tabs at the bottom — each shows a different screen when you tap it. The “Recipes” tab looks like this:

 ![](./Android Fundamentals by Tutorials, Chapter 8_Networking_ Kodeco_files/original.png)

The “Groceries” tab looks like this:

 ![](./Android Fundamentals by Tutorials, Chapter 8_Networking_ Kodeco_files/original(1).png)

Once you finish, you can search for recipes, display them in a grid, bookmark the ones you want to keep and show the list of groceries needed for those meals.

### Coroutines

Asynchronous programming requires tasks to be executed on different threads. Usually, there’s a “main” thread for UI drawing. To do other tasks without slowing the UI thread, you need a way to execute them without slowing what the user is working on. How can you do two or more things at once? When there was just one processor, the system would have to switch between tasks quickly, giving the appearance of multitasking. Different tasks can run on different cores now that multi-core processors exist. You can have as many threads as you want. But typically, you’ll have just a UI, IO and a default thread or “dispatcher”. For example, if you needed to retrieve some data from the internet, you wouldn’t want your UI to freeze while that data is downloaded. You would use a default dispatcher (which is different than the UI dispatcher) to download the data and then switch to the UI dispatcher to display that data.

Kotlin has the **kotlinx.coroutines** library for coroutines. For more details, see: [https://kotlinlang.org/docs/coroutines-guide.html](https://kotlinlang.org/docs/coroutines-guide.html). Coroutines are like mini-threads. You can use thousands without causing any issues (unlike threads). Kotlin uses the `suspend` keyword to mark a function as asynchronous. If you want to call a suspend function, you must either be in another suspend function or start one with a coroutine builder. The most common one is the `launch` method. This method comes from a CoroutineScope. If you’ve seen Android’s `ViewModel`, you’ll notice it has its own built-in `viewModelScope`. If you look at the `viewModelScope` code:

```
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

You see it:

2. Creates a getter function.
4. Checks to see if the scope already exists and returns it.
6. Creates and stores a new scope made of a `SupervisorJob` and a main Dispatcher (UI thread).

Scopes combine a job, a dispatcher and possibly an exception handler. This code uses a `SupervisorJob`. This is important because normal `Job`s will kill the task if it dies. If any child jobs die while using a `SupervisorJob`, the main coroutine still runs.

Look at the `launch` method:

```
public fun CoroutineScope.launch(
    context: CoroutineContext = EmptyCoroutineContext,
    start: CoroutineStart = CoroutineStart.DEFAULT,
    block: suspend CoroutineScope.() -> Unit
): Job {
  ...
}
```

Usually, you’ll use `launch` in a ViewModel like this:

```
viewModelScope.launch {
  ...
}
```

In this instance, the last parameter (the `block`) is your code and uses the default `EmptyCoroutineContext` for the coroutine context.

#### Adding the Coroutine Library

To add the coroutine library, open the **libs.versions.toml** file in the **Gradle** directory. Under the **versions** section, add:

```
kotlinx-coroutines = "1.7.2"
```

Then, at the end of the **libraries** section, add:

```
coroutines-android = { module = "org.jetbrains.kotlinx:kotlinx-coroutines-android", version.ref = "kotlinx-coroutines" }
```

Open the app module’s **build.gradle.kts** and add the following to the **dependencies** section:

```
implementation(libs.coroutines.android)
```

Notice how a “.” is used instead of a “-” in coroutines-android? This provides an object-notation approach when filling out the dependencies in the **build.gradle.kts** file. It also helps group similar libraries together and allows auto-completion of your dependencies as you type them. Now, do a Gradle sync to add the library.

### Flows

Most Android developers have been using the LiveData library for quite some time. It provides a way of notifying the UI of events and handles the Android lifecycle. The new kid on the block is **Flows**. Like with LiveData, you’ll create a mutable flow in the ViewModel but only expose a non-mutable version. Here’s an example:

```
private val _queryState = MutableStateFlow(QueryState())
val queryState = _queryState.asStateFlow()
```

The private `_queryState` is a state that can change. The UI will listen to `queryState`. To listen to events, you would do something like:

```
val uiState by viewModel.uiState.collectAsState()
```

If you wanted to act on the state changing, you could `collect` state changes like:

```
val scope = rememberCoroutineScope()
LaunchedEffect(Unit) {
    scope.launch {
        viewModel.recipeListState.collect { state ->
            recipeListState.value = state
        }
    }
}
```

The `collect` method requires a coroutine scope to listen to events because it’s a suspend function.

`LaunchedEffect` is a composable side-effect that launches a coroutine when the composable is first displayed. It also cancels the coroutine when the composable is removed from the screen. This is important because you don’t want to keep listening to events when the composable is no longer visible.

### Network Requests

To retrieve information from the internet, you need to make a network request. The easiest way to do that is with a library, and one of the best and most used libraries is **Retrofit**.

#### Alternatives

In this chapter, you’ll use Retrofit, but other libraries exist:

+ **Fuel** ([https://github.com/kittinunf/fuel](https://github.com/kittinunf/fuel)) is a newer one and is being updated.
+ **Ktor** ([https://ktor.io/](https://ktor.io/)) is used in the Multiplatform world, but you can also use it in Android.

### Retrofit

Square developed Retrofit. It’s a type-safe HTTP client for Android and Java/Kotlin. Although there are newer libraries, knowing Retrofit will let you work with almost any app.

Retrofit works by creating an interface and using annotations. A builder then uses that information to create code that makes the calls for you.

#### Adding Retrofit

To add the Retrofit library, open the **libs.versions.toml** file in the **Gradle** directory. Under the **versions** section, add:

```
retrofit="2.9.0"
```

Then, in the **libraries** section, add:

```
# Retrofit
retrofit = {module="com.squareup.retrofit2:retrofit", version.ref="retrofit" }
```

This adds the main Retrofit library.

Open the app module’s **build.gradle.kts** and add the following to the **dependencies** section:

```
implementation(libs.retrofit)
```

Now, do a Gradle sync to add the library.

### Response Parsing

To convert network response data (usually returned as a JSON string), you need an easy way to convert the string into models and vice versa. You’ll use the **Moshi** library (also created by Square). Moshi allows you to parse JSON into Kotlin classes. There are other parsing libraries (a common one is Gson), but Moshi is newer and more modern.

#### Adding Moshi

To add the Moshi library, open the **libs.versions.toml** file in the **Gradle** directory.

Under the **versions** section, add:

```
moshi="1.15.0"
```

In the **libraries** section, add:

```
moshi-kotlin = {module="com.squareup.moshi:moshi-kotlin", version.ref="moshi" }
retrofit-moshi-converter = {module="com.squareup.retrofit2:converter-moshi", version.ref="retrofit" }
```

Open the app module’s **build.gradle.kts** and add the following to the **dependencies** section:

```
implementation(libs.moshi.kotlin)
implementation(libs.retrofit.moshi.converter)
```

## Signing Up With the Recipe API

For your remote content, you’ll use the Spoonacular Recipe API. Open this link in your browser: [https://spoonacular.com/food-api](https://spoonacular.com/food-api).

Click the **Start Now** button at the top right to create an account.

 ![](./Android Fundamentals by Tutorials, Chapter 8_Networking_ Kodeco_files/original(2).png)

Fill out the email and passwords, then click the checkbox to accept the terms and conditions. Finally, click the **Sign up** button. Finish the process using the free tier.

 ![](./Android Fundamentals by Tutorials, Chapter 8_Networking_ Kodeco_files/original(3).png)

You’ll see the following:

 ![](./Android Fundamentals by Tutorials, Chapter 8_Networking_ Kodeco_files/original(4).png)

Once you have confirmed your email, click this link and log in.

 ![](./Android Fundamentals by Tutorials, Chapter 8_Networking_ Kodeco_files/original(5).png)

You see the API Console. Once you start making requests, you’ll see the graph fill up.

 ![](./Android Fundamentals by Tutorials, Chapter 8_Networking_ Kodeco_files/original(6).png)

Now, go to the documents:

 ![](./Android Fundamentals by Tutorials, Chapter 8_Networking_ Kodeco_files/original(7).png)

Here, you can see the documents for searching for recipes:

 ![](./Android Fundamentals by Tutorials, Chapter 8_Networking_ Kodeco_files/original(8).png)

If you scroll down, you can see many fields returned. But you aren’t interested in most of these.

You see the **path** and a list of the parameters available for the `GET` request you’ll make.

 ![](./Android Fundamentals by Tutorials, Chapter 8_Networking_ Kodeco_files/original(9).png)

There’s much more API information on this page than you’ll need for your app, so you might want to bookmark it for the future.

Click **My Console** and then the **Profile** section:

 ![](./Android Fundamentals by Tutorials, Chapter 8_Networking_ Kodeco_files/original(10).png)

Click **Show/Hide API Key**. Copy the API Key and save it in a secure place.

### Using Your API Key

For your next step, you need to use your new API Key.

> **Note**: The free developer version of the API is rate-limited. If you use the API often, you’ll probably receive some JSON responses with errors and emails warning you about the limit.

### SpoonacularService

To fetch data from the recipe API, you’ll create a Kotlin interface and an object to manage the connection. This Kotlin class file contains your API Key, ID and URL. You’ll also need a model class for the response.

In the Project sidebar, right-click **main/java/com/kodeco/recipefinder/data/models**, create a new Kotlin file and name it **SearchRecipesResponse.kt**. After the file opens, import the following:

```
import com.squareup.moshi.Json
```

Then, add the data class:

```
data class SearchRecipesResponse(
  val offset: Int,
  val number: Int,
  val totalResults: Int,
  @Json(name = "results")
  val recipes: List<Recipe>
)
```

You use the `@Json` annotation to map the JSON field named “results” to a variable named “recipes”.

Next, open the Kotlin file named **SpoonacularService.kt**. Import the following:

```
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

Then, add:

```
const val apiKey = "<Replace with API Key>"
```

> **Note**: Remember that you should never commit your API key to a public repository. You should always store it securely and reference it from there. For this tutorial, you’ll just add it to the code for simplicity. If you’re interested in learning how to store things like API keys and secrets, a common approach recommended by Google is to use the [secrets-gradle-plugin](https://github.com/google/secrets-gradle-plugin) library.

Replace the string with the key you retrieved when creating your account.

Next, this service makes two network calls:

+ One to search for recipes with a query string ([https://spoonacular.com/food-api/docs#Search-Recipes-Complex](https://spoonacular.com/food-api/docs#Search-Recipes-Complex)).
+ Another to get the details for a specific recipe ([https://spoonacular.com/food-api/docs#Get-Recipe-Information](https://spoonacular.com/food-api/docs#Get-Recipe-Information)).

Add:

```
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

This piece of code:

2. Uses the **@GET** annotation to specify the URL to use for this call. Notice it doesn’t contain the beginning portion of the URL. You’ll add this later.
4. Creates a suspend function that takes three query parameters: one for the query, one for the starting point in the list of queries and a third for the number of results to return. This function returns a `SearchRecipesResponse`.
6. Creates another suspend function. This will query a specific recipe with the given ID. It responds with a `RecipeInformationResponse`.

Notice you put the API Key into the URL, and the second call uses `{id}` to substitute the ID in the URL. To create an instance of Retrofit, you need some methods to use the Retrofit builder. Add the following in **SpoonacularService.kt**:

```
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

With this code, you:

2. Provide the base URL for the calls. This code prepends it to all the services you defined before in the interface.
4. Provide an instance of Moshi that uses the `KotlinJsonAdapterFactory` factory to parse JSON.
6. Use the Retrofit builder to assemble the base URL and Moshi. Then, call the `build` method to create an instance of Retrofit. Using `lazy` means this won’t be created until it’s first used and then will be cached and not re-created.
8. A variable for others to use to make service calls.

You now have all you need to make network calls.

### Implement the ViewModel

Now that you’ve written the service, you must retrieve the list of recipes matching the query string. Open **viewmodels/RecipeViewModel.kt**. Find `// TODO: Add Service` and replace it with:

```
private val spoonacularService = RetrofitInstance.spoonacularService
```

Import:

```
import com.kodeco.recipefinder.network.RetrofitInstance
```

This gives you an instance of the service. Replace `// TODO: query recipes` with:

```
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

This code:

2. Uses the service to query for recipes that have the given query string, offset and number of recipes.
4. Replaces the existing recipe list with the new recipes from the API response.
6. Updates the query state to include the latest information.
8. Sets the list to an empty listIf there were any errors.

Import the following:

```
import androidx.lifecycle.viewModelScope
import kotlinx.coroutines.launch
import timber.log.Timber
```

The UI listens to the recipe list state (`recipeListState` variable). When a new list is available, the recipe list state notifies the UI, which updates accordingly. Check this by opening the **RecipeList.kt** file:

```
scope.launch {
  viewModel.recipeListState.collect { state ->
    recipeListState.value = state
  }
}
```

## Run the App

Run the app on a device or emulator by pressing the **Play** button.

 ![](./Android Fundamentals by Tutorials, Chapter 8_Networking_ Kodeco_files/original(11).png)

You see something like:

 ![](./Android Fundamentals by Tutorials, Chapter 8_Networking_ Kodeco_files/original(12).png)

Type a query in the search bar (like Sushi):

 ![](./Android Fundamentals by Tutorials, Chapter 8_Networking_ Kodeco_files/original(13).png)

Now, press the **Search** icon:

 ![](./Android Fundamentals by Tutorials, Chapter 8_Networking_ Kodeco_files/original(14).png)

After the progress indicator spins for a while, you see some recipes:

 ![](./Android Fundamentals by Tutorials, Chapter 8_Networking_ Kodeco_files/original(15).png)

> If you don’t see any recipes, ensure you have your API Key in the **SpoonacularService.kt** file.

### Use Moshi’s Codegen

Currently, Moshi uses reflection to convert JSON to Kotlin classes. But for better performance, you can use Moshi’s codegen support, which can generate adapters at compile-time.

To enable this, open the **libs.versions.toml** file in the **Gradle** directory and, in the **libraries** section, replace:

```
moshi-kotlin = {module="com.squareup.moshi:moshi-kotlin", version.ref="moshi" }
```

With the following:

```
moshi = {module="com.squareup.moshi:moshi", version.ref="moshi" }
moshiCodeGen = {module="com.squareup.moshi:moshi-kotlin-codegen", version.ref="moshi" }
```

Next, open the app module’s **build.gradle.kts** file and, in the **dependencies** section, replace:

```
implementation(libs.moshi.kotlin)
```

With the following:

```
implementation(libs.moshi)
ksp (libs.moshiCodeGen)
```

The **ksp** command is the Kotlin Symbol Processing API. The Moshi codegen plugin creates converters for your model classes. Now, do a Gradle sync.

You no longer need the `KotlinJsonAdapterFactory` so open **SpoonacularService.kt** and update the `provideMoshi()` to the following:

```
private fun provideMoshi(): Moshi =
  Moshi
    .Builder()
    .build()
```

Also, remove the following import:

```
import com.squareup.moshi.kotlin.reflect.KotlinJsonAdapterFactory
```

Finally, you must annotate all the classes you want to convert to and from JSON. Open **SearchRecipesResponse.kt** and add the `@JsonClass` annotation:

```
@JsonClass(generateAdapter = true)
data class SearchRecipesResponse(
  val offset: Int,
  val number: Int,
  val totalResults: Int,
  @Json(name = "results")
  val recipes: List<Recipe>
)
```

Also, import this:

```
import com.squareup.moshi.JsonClass
```

The `@JsonClass` annotation creates an adapter for you. When you build your project, it creates a file for you named `SearchRecipesResponseJsonAdapter`. This has a lot of boilerplate for converting JSON to your class and back.

Do the same with the rest of the files from **main/java/com/kodeco/recipefinder/data/models**.

Build and run the app to check that everything still works.

### Details Screen

If you were to click a recipe, you would see a blank screen. It’s time to fix that.

Open **RecipeViewModel.kt**. Replace `// TODO: Query a Recipe` with:

```
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

With this code, you:

2. Launch a new coroutine on the default dispatcher.
4. Call the SpoonacularService to get the full recipe details.
6. Set the Recipe state’s value that the Details screen listens to.

You need to import this:

```
import kotlinx.coroutines.Dispatchers
```

Rerun the app by pressing the **Re-play** button. Click a recipe to see the details:

 ![](./Android Fundamentals by Tutorials, Chapter 8_Networking_ Kodeco_files/original(16).png)

If you press the **Bookmark** or the **Back arrow** icon, you return to the recipe list.

 ![](./Android Fundamentals by Tutorials, Chapter 8_Networking_ Kodeco_files/original(17).png)

## Key Points

+ Coroutines are a great way to run code in the background.

+ Flows replace LiveData and provide a way to notify the UI of events.

+ Retrofit is a great library for handling network requests.

+ The Moshi library provides response parsing.

## Where to Go From Here?

In this chapter, you learned how to retrieve network data easily.

To learn more about **Retrofit**, go to [https://square.github.io/retrofit/](https://square.github.io/retrofit/).

To learn more about **Moshi**, check [https://github.com/square/moshi](https://github.com/square/moshi).

In the next chapter, you’ll learn about storing data using Shared Preferences.
