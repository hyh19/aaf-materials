# 9. Data Store

Picture this: You’re browsing recipes and find one you like. You’re in a hurry and want to bookmark it to check it later. Can you build an app that does that? You sure can! Read on to find out how.

In this chapter, your goal is to learn how to use the Data Store library to save important pieces of information to your device.

## Saving Data

There are three primary ways to save data to your device:

2. Write formatted data, like JSON, to a file.
4. Use a library to write simple data to a shared location.
6. Use a SQLite database.

Writing data to a file is simple, but it requires you to handle reading and writing data in the correct format and order. For more complex data, you can save the information to a local database.

### Why Save Small Bits of Data?

There are many reasons to save small bits of data. For example, you could save the user ID when the user has logged in — or if the user has logged in at all. You could also save the onboarding state or data that the user has bookmarked to consult later.

Note, that this simple data saved to a shared location is lost when the user uninstalls the app.

The goal for this chapter is to show a menu of previously entered search strings so the user can reselect a search. This will be shown even if the user restarts the app. The second goal is to restore the currently selected screen. For example, if the user was reviewing the groceries screen, if they come back after a long time and the app restarts, they’ll want to return to the screen they were previously on. Here’s what the main screen will look like:

 ![](./Android Fundamentals by Tutorials, Chapter 9_Data Store_ Kodeco_files/original.png)

#### SharedPreferences

Android has a built-in interface named **SharedPreferences** that is available in every version of Android. You can retrieve any preference from any **Context**. You can also retrieve the full interface from an **Activity**.

> **Note**: The **SharedPreferences** method has been deprecated. We’ll be using the newer library.

#### How Does It Work?

**SharedPreferences** uses the system to store data into a file. These are small bits of information like integers, strings or booleans. It has several sets of function calls:

2. `getXXX()`: Get methods retrieve the data of that specific data type.
4. `contains()`: This method checks if the key exists.
6. `edit()`: Returns an editor class to make changes.

#### Editor

To make changes, you need to call the `edit()` method to get an editor. You then have access to the following methods:

2. `putXXX()`: Set methods save the data of that specific data type.
4. `clear()`: This method deletes all saved data.
6. `remove()`: This method removes a specific value.
8. `apply()` or `commit()`: Save changes made. The `apply()` method is now recommended as it is asynchronous. However, if you need something saved immediately, the use the `commit()` method.

All of these methods except the `clear()` method use a **key** to access an item. By giving the library a unique key, you can store, retrieve and delete specific item. Here is an example:

```
// 1
val sharedPreferences = getSharedPreferences("MyPreferences", Context.MODE_PRIVATE)
// 2
if (sharedPreferences.contains("MyKey")) {
  // 3
  val myValue = sharedPreferences.getString("MyKey", "MyDefault")
  Timber.d("My Value is $myValue")
}
```

2. Retrieve an instance of **SharedPreferences** using the file name of “MyPreferences” and is private (You’ll always want to make it private).
4. First check to make sure the key exists.
6. Retrieve a string value with the key “MyKey” and a default value of “MyDefault” if it doesn’t exist.

Here you are retrieving a string with the given key. Here are the types you can store and retrieve:

+ String
+ Boolean
+ Int
+ Long
+ Float

Note that you can’t store anything more complex than these. That means classes or even data classes cannot be store unless you save each part separately.

## DataStore Library

Google has been transitioning to using external libraries instead of just updating the system. This strategy provides them with the flexibility to frequently roll out modifications to the libraries that are immediately accessible to users. Instead of updating the **SharedPreferences** code in the system, they’ve created a new library called **DataStore**. This is a modern library for storing key/value pairs. The DataStore library uses the **DataStore** interface and is designed a bit differently. If you take a look at the definition, it is pretty simple:

```
interface DataStore<T> {
  val data: Flow<T>
  suspend fun updateData(transform: suspend (t: T) -> T): T
}
```

To create a variable named `dataStore`:

```
val Context.dataStore: DataStore<Preferences> by preferencesDataStore(name = "settings")
```

This is an extension to `Context` and uses the delegated property from `preferencesDataStore`. You can use whatever name you want. Here the preference file will be named “settings”.

To access an entry, you use a string key like the older system, but you first have to create a variable around the key. If you are trying to retrieve a string value you would use:

```
val prefKey = stringPreferencesKey("MyKey")
```

The library also uses Kotlin’s more modern Flow class from the Coroutine library. To retrieve a value you would do:

```
context.dataStore.data.first()[prefKey]
```

The `first()` method is considered a “terminating” operator - it will only retrieve the first value.

> Important: To avoid unintended behaviour, do not create more than one instance of a DataStore pointing to the same preference file.

#### Add DataStore Library

If you’re following along with your app from the previous chapters, open it and keep using it with this chapter. If not, just locate the **projects** folder for this chapter and open **starter** in Android Studio. Open up **gradle/libs.versions.toml**. At the end of the **versions** section add:

```
prefsVersion = "1.0.0"
```

Then at the end of the **\[libraries\]** section add:

```
# Preferences
prefs = {module = "androidx.datastore:datastore-preferences", version.ref = "prefsVersion" }
```

Do a gradle sync. Inside of **app/build.gradle.kts** add after the timber library:

```
implementation(libs.prefs)
```

Do another gradle sync.

#### Saving UI States

You’ll use **DataStore** to save a list of saved searches in this section. Later, you’ll also save the tab that the user has selected so the app always opens to last selected tab.

Start by going to the **app/src/main/java/com/kodeco/recipefinder/data** directory and creating a new Kotlin file named: **Prefs.kt**. Add the following:

```
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

Now replace `// TODO: Add dataStore` with:

```
private val Context.dataStore: DataStore<Preferences> by preferencesDataStore(name = "recipes")
```

This is similar to the example, but you will name the file “recipes”. Next, replace `// TODO: Add saveString` with:

```
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

2. Pass in the key and the value to save.
4. Create a string preference key.
6. Start editing.
8. Set the value for the given key.

Now replace `// TODO: Add getString` with:

```
suspend fun getString(key: String): String? {
  val prefKey = stringPreferencesKey(key)
  return context.dataStore.data.first()[prefKey]
}
```

This starts out the same but uses the `data.first()` method to retrieve the entry using key as index.

Replace the next two TODOs with:

```
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

These method do the same thing as get/save String but for Ints. Finally, replace `// TODO: Add hasKey` with:

```
suspend fun hasKey(key: String): Boolean {
  val prefKey = stringPreferencesKey(key)
  return context.dataStore.data.first().contains(prefKey)
}
```

This just checks to see if the entry exists. Now that we have this class to save and retrieve preferences, let’s change the code to use it.

Begin by opening **RecipeApp.kt**. Locate `// TODO: Add Prefs` and replace it with:

```
lateinit var prefs: Prefs
```

Now, look for the next `// TODO: Add Prefs` and replace it with:

```
prefs = Prefs(this)
```

You’ve now set up your instance of `Prefs`, now it’s time to use it. Open **MainActivity.kt** and replace: `// TODO: Add Pref Provider` with:

```
val LocalPrefsProvider =
    compositionLocalOf<Prefs> { error("No prefs provided") }
```

This uses Compose’s provider ability. Import any required imports. Note that while this starts out with an error, you’ll set the provider later. Next, replace `// TODO: Add Prefs` with:

```
val prefs = remember { Prefs(context) }
```

After `LocalNavigatorProvider provides navController` add:

```
LocalPrefsProvider provides (application as RecipeApp).prefs,
```

This provides your Prefs class to any composable below.

#### Updating ViewModels

Now that you have your Prefs class written, you want to provide it to your view models to be able to save data. Open up **RecipeViewModel.kt** and at `// TODO: Add Prefs`, change the constructor to include the prefs. The constructor should look like:

```
class RecipeViewModel(private val prefs: Prefs) : ViewModel() {
```

Find `// TODO: Save previous searches` in the `savePreviousSearches()` method and replace it with:

```
viewModelScope.launch {
  val searchString = _uiState.value.previousSearches.joinToString(",")
  prefs.saveString(PREVIOUS_SEARCH_KEY, searchString)
}
```

This takes the list of previous searches from the ui state and creates one long string (separated by commas). Then it saves the string using the prefs class using the `PREVIOUS_SEARCH_KEY` as the key. To update the list of previous searches, find the `addPreviousSearch` method and add:

```
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

2. Check to make sure that the search string doesn’t already exist in the list to avoid duplicates.
4. With a new list, add all the previous searches and the new one.
6. Update the UI state with modified copy of the previous searches.
8. Use the method you worked on before to save the list.

If you were to try and run now, you’d see an error in **RecipeDetails**. Open this up and find `// TODO: Add Prefs*`. Add:

```
val prefs = LocalPrefsProvider.current
```

The uses the **LocalPrefsProvider** defined earlier. Now it’s just a matter of using the `current` version of it. Update the factory for the viewmodel to be:

```
RecipeViewModel(prefs)
```

#### Exercise

Convert the remaining methods, just like you just did with **RecipeDetails**.

Do the same thing for the following files:

+ **GroceryList**
+ **ChipRow**
+ **RecipeList**
+ **SearchRow**
+ **ShowBookmarks**
+ **ShowRecipeList**

For the preview code, use the given context to create a new Prefs class. If you have any problems, preview the final project. Now, restart the app. Do a search, like chicken and then click on the three dots menu to see the search has been saved.

 ![](./Android Fundamentals by Tutorials, Chapter 9_Data Store_ Kodeco_files/original(1).png)

Now, stop the app and restart. Verify that the list is empty. Why is that? You haven’t written any code to load the preferences.

Reopen **RecipeViewModel** and replace `// TODO: Retrieve previous searches` with:

```
viewModelScope.launch {
  val previousSearchString = prefs.getString(PREVIOUS_SEARCH_KEY)
  if (!previousSearchString.isNullOrEmpty()) {
      val storedList = previousSearchString.split(",")
      _uiState.value = _uiState.value.copy(previousSearches = storedList.toMutableList())
  }
}
```

This is the opposite of the `savePreviousSearches` method. This method gets the string list, separates it by splitting the string by “,”, creating a new list and setting the UI state’s previous search list. Restart the app. You should now see the items.

#### Saving the Selected Tab

In this section, you’ll use shared preferences to save the current UI tab that the user has navigated to. Open **ui/MainScreen.kt**. Start by getting an instance of your prefs class. Replace `// TODO: Add Prefs` with:

```
val prefs = LocalPrefsProvider.current
```

Next, replace `// TODO: Get screen position from prefs` with:

```
val currentIndex = prefs.getInt(CURRENT_INDEX_KEY)
if (currentIndex != null) {
  selectedIndex.intValue = currentIndex
}
```

This will get the current index of the tabs and if it exists, will set the selectedIndex.value. Next, replace `// TODO: Save screen position to prefs` with:

```
scope.launch {
  prefs.saveInt(CURRENT_INDEX_KEY, 0)
}
```

This saves the index by running it in a coroutine scope (since this needs to be done asynchronously). Finally, replace the next TODO with:

```
scope.launch {
  prefs.saveInt(CURRENT_INDEX_KEY, 1)
}
```

The only difference with this call is it is using index 1 instead of 0. Rerun the app. Click the Groceries icon at the bottom. Kill the app and restart. It should start with the groceries tab selected.

## Key Points

+ There are multiple ways to save data in an app: to **files**, in **shared preferences** and to a **SQLite** database.

+ Shared preferences are best used to store simple, **key-value pairs** of primitive types like `strings`, `numbers` and `booleans`.

+ An example of when to use **shared preferences** is to save the tab a user is viewing, so the next time the user starts the app, they’re brought to the same tab.

## Where to Go From Here?

In this chapter, you learned how to persist simple data.

If you want to learn more about Android **SharedPreferences**, go to [https://developer.android.com/reference/kotlin/android/content/SharedPreferences?hl=en](https://developer.android.com/reference/kotlin/android/content/SharedPreferences?hl=en).

To learn about **DataStore** go to [https://developer.android.com/topic/libraries/architecture/datastore](https://developer.android.com/topic/libraries/architecture/datastore).

In the next chapter, you’ll learn about storing data in databases using Room. See you there!
