# 10. Room Database

So far, you have a great app to search the internet for recipes. But there are no bookmarks or groceries. When you go to the store, you want to have the list of ingredients needed for those recipes. Do you want to have to search again at the store to get that information?

## SQLite

One of the best ways to persist data is with a database. Android provides access to the **SQLite** database system. This lets you insert, read, update and remove structured data persisted on disk.

In this chapter, you’ll learn about using the **Room** library.

By the end of the chapter, you’ll know:

+ How to insert, fetch and remove recipes or ingredients.
+ How to use the Repository pattern for a generic way to do these actions.

### Room

Room, which Google created in 2018, adds a layer over the built-in SQLite database that Android provides, making it easier to use.

Before diving into the code, it’s important to understand Room’s three basic components:

2. **Database**: This is the main interface to the underlying SQLite database. This component maintains one or more **Data Access Objects** (DAOs) and is annotated with the list of all **Entities** the database uses. A database class inherits from `RoomDatabase` and uses the `@Database` annotation.

4. **Entity**: This represents a single data type stored in the database. Room creates a table in the database for each entity, and the rows of the table represent individual entity items.

    Entities are defined as simple classes using the `@Entity` annotation. All entity class properties are automatically defined as fields in the database unless you use the `@Ignore` annotation. You should designate at least one entity property as the primary key using the `@PrimaryKey` annotation.

6. **DAO**: Data Access Objects are the heroes of Room. This is where you define the interface for accessing the database. DAOs should be the only part of your app that talks directly to the database. The database class must contain at least one abstract method that returns a DAO-annotated interface.

The following diagram illustrates these three components:

Get/Set Properties Read/Write Entities Get DAOs from Database Main Application Code Database Label DAOs Label Entities Label Entities SQLite Database

#### Room and Android Architecture Components

Room is part of a larger set of libraries known as the **Android Architecture Components**. The other components are:

+ **Lifecycle management**: Provides several classes to help build lifecycle-aware objects.
+ **LiveData**: Holds data that can be observed for changes and respects lifecycles.
+ **ViewModel**: Manages View-related data without being tied to configuration changes. This is the bridge between UI Views and the rest of the app.

Don’t worry about the details of these components right now; you’ll cover them in more detail as you build the app.

#### App Architecture

Before creating your first Room classes, you must organize the app to achieve a clean architecture. You’ll separate the app into distinct areas of responsibility along these lines:

+ Data access and persistence (Room).
+ Data model (Model).
+ Data abstraction (Repository).
+ Business/Domain logic (ViewModel).
+ User interface (Activity/Fragment).

One key goal is ensuring communication flows only in one direction between these layers. This results in a loosely coupled architecture that is easy to modify without side effects.

The architecture looks like this:

Repository Data Access Persistence Data Model UI (Compose) ViewModel

The arrows represent lines of communication and visibility. Notice the UI layer is completely independent of all other layers except for the ViewModel. The ViewModel layer knows nothing about the UI layer.

As you build the rest of the app, you won’t compromise when it comes to sticking to the communication flow shown in the diagram above. It’ll sometimes take a little more work to adhere strictly to this pattern, but the payoff for larger apps is worth the effort. Even for a small app, you can immediately recognize some benefits:

+ How you store data in Room can be completely replaced with minimal impact. The only layers affected are the **Persistence** layer and its immediate parent, the **Data Access** layer.

+ You can replace the UI layer without any other layer knowing.

+ You can easily test all the layers without any active UI running.

#### Development Approach

Think about the architecture as a multi-layered cake. Have you ever seen somebody eat a cake one layer at a time? That would be a little odd! Likewise, you won’t build the app one layer at a time. You’ll take one slice at a time. Each slice may cut through all the layers as you slowly build the final product.

UI (Compose) ViewModel Repository Data Access Persistence (Room) Data Model The Architecture Cake One slice at a time

Here are the directories you’ll use:

+ **data/database**: Data access and persistence. You’ll keep the **Room Database** and **DAO** objects here.
+ **data/models**: Model objects. This includes all **Room Entities** classes.
+ **ui**: User interface. All Views and View control logic belong here.
+ **viewmodels**: Business/Domain logic. This contains ViewModel classes that drive the user interface and app logic.

#### Adding Room Library

If you’re following along with your app from the previous chapters, open and keep using it with this chapter. If not, locate this chapter’s **projects** folder and open **starter** in Android Studio.

If you’re using the starter project, open the **SpoonacularService.kt** file and update **apiKey** with your API key from the account you created at [https://www.spoonacular.com](https://www.spoonacular.com/).

Open the **libs.versions.toml** file to add the Room library. At the end of the **versions** section, add:

```
room="2.5.2"
```

Then, at the end of the **libraries** section, add:

```
# Room
room = { module= "androidx.room:room-ktx", version.ref="room" }
room-runtime ={ module= "androidx.room:room-runtime", version.ref="room" }
room-compiler = { module = "androidx.room:room-compiler", version.ref="room" }
```

Finally, open the app module’s **build.gradle.kts** file and add:

```
id("kotlin-parcelize")
```

as the last line in the **plugins** section. When using the `@Parcelize` annotation, this plugin auto-generates code for `Parcelable` types.

Then, in the **dependencies** section, add:

```
// Room
implementation(libs.room)
implementation(libs.room.runtime)
ksp (libs.room.compiler)
```

And do a Gradle sync.

## Room Classes

Now, you’re ready to add the basic classes required by Room. This includes the Entities, DAOs and the Database. Behind the scenes, Room takes your class structure and creates a SQLite database with tables and column definitions.

For Room, name the database: `recipe_database`, and the model classes: `RecipeDb` and `IngredientDb`. The following diagram will help you visualize the process that Room uses to convert your classes into the underlying database:

Database RecipeDao IngredientDao RecipeDatabase id:Int title: String image: String? summary: String instructions:String? sourceUrl: String preparationMinutes: Int cookingMinutes: Int readyInMinutes: Int servings: Int Model Entities RecipeDb id:Int recipeId: Int? name: String aisle:String? image: String? original: String amount: Double unit: String IngredientDb recipe\_database id title image summary instructions sourceUrl Recipe Table id recipeId name aisle image original Ingredient Table Room Room Room Room Database Creation Process

### Entities

Recipe Finder requires two entity types to store recipes: `RecipeDb` and `IngredientDb`.

#### RecipeDb

Create a package called **database** in the **data** package. Inside this package, create a Kotlin file named **RecipeDb.kt** and replace the contents with the following:

```
import android.os.Parcelable
import androidx.room.ColumnInfo
import androidx.room.Entity
import androidx.room.PrimaryKey
import kotlinx.parcelize.Parcelize

// 1
@Parcelize
// 2
@Entity(tableName = "recipes")
// 3
data class RecipeDb(
  // 4
  @PrimaryKey(autoGenerate = false)
  // 5
  @ColumnInfo(name = "id")
  var id: Int,
  @ColumnInfo(name = "title")
  var title: String,
  @ColumnInfo(name = "image")
  var image: String?,
  @ColumnInfo(name = "summary")
  var summary: String = "",
  @ColumnInfo(name = "instructions")
  var instructions: String? = "",
  @ColumnInfo(name = "sourceUrl")
  var sourceUrl: String = "",
  @ColumnInfo(name = "preparationMinutes")
  var preparationMinutes: Int = 0,
  @ColumnInfo(name = "cookingMinutes")
  var cookingMinutes: Int = 0,
  @ColumnInfo(name = "readyInMinutes")
  var readyInMinutes: Int = 0,
  @ColumnInfo(name = "servings")
  var servings: Int = 0,
) : Parcelable
```

Here’s what’s going on in the code above:

2. Kotlin uses the `@Parcelize` annotation to generate a class’s `Parcelable` implementation. A `Parcelable` is an interface that serializes and deserializes an object. This is useful for transferring data between activities, fragments or other components in an Android app.

4. The `@Entity` annotation tells Room this is a database entity class.

    > **Note**: Although not used in this example, you can apply several attributes to the Entity annotation.
    >
    > `foreignKeys()`: List of ForeignKey constraints.
    >
    > `indices()`: List of indices to include on the table.
    >
    > `primaryKeys()`: List of primary key column names. It’s not required if using the `PrimaryKey` annotation.
    >
    > `tableName()`: Table name to use in the database. Defaults to class name.

6. The `RecipeDb` class’s primary constructor is defined using arguments for all properties with default values defined. Defining default values lets you construct a recipe with a partial list of properties.

    > **Note**: Room looks for arguments on the constructor and class properties when defining the table fields. In this case, you only use properties to define the table fields.

8. You defined the `id` property with the `@PrimaryKey` annotation. There must be at least one of these per Entity class. The `autoGenerate` attribute automatically tells Room to generate incrementing numbers for this field.

    In database terminology, this would be considered a surrogate or synthetic key, providing a unique identifier for each recipe record.

10. You defined the rest of the fields with default values.

#### IngredientDb

Create a Kotlin file named **IngredientDb.kt** in the **data/database** package and replace the contents with the following:

```
import android.os.Parcelable
import androidx.room.ColumnInfo
import androidx.room.Entity
import androidx.room.PrimaryKey
import kotlinx.parcelize.Parcelize

@Parcelize
@Entity(tableName = "ingredients")
data class IngredientDb(
  @PrimaryKey(autoGenerate = false)
  @ColumnInfo(name = "id")
  var id: Int,
  @ColumnInfo(name = "recipeId")
  var recipeId: Int?,
  @ColumnInfo(name = "name")
  var name: String,
  @ColumnInfo(name = "aisle")
  var aisle: String? = "",
  @ColumnInfo(name = "image")
  var image: String? = "",
  @ColumnInfo(name = "original")
  var original: String = "",
  @ColumnInfo(name = "amount")
  var amount: Double = 0.0,
  @ColumnInfo(name = "unit")
  var unit: String = "",
) : Parcelable
```

This is like the `RecipeDb` class.

### DAOs

Next, you’ll define the data access object that reads and writes from the database.

Create a Kotlin file named **RecipeDao.kt** in the **data/database** package and replace the contents with the following:

```
import androidx.room.Dao
import androidx.room.Delete
import androidx.room.Insert
import androidx.room.OnConflictStrategy
import androidx.room.Query
import androidx.room.Update

// 1
@Dao
interface RecipeDao {
  // 2
  @Insert(onConflict = OnConflictStrategy.IGNORE)
  suspend fun addRecipe(recipe: RecipeDb)

  // 3
  @Query("SELECT * FROM recipes WHERE id = :id")
  suspend fun findRecipeById(id: Int): RecipeDb

  // 4
  @Query("SELECT * FROM recipes")
  suspend fun getAllRecipes(): List<RecipeDb>

  // 5
  @Update
  suspend fun updateRecipeDetails(recipe: RecipeDb)

  // 6
  @Delete
  suspend fun deleteRecipe(recipe: RecipeDb)

  @Query("DELETE FROM recipes WHERE id = :recipeId")
  suspend fun deleteRecipeById(recipeId: Int)
}
```

`RecipeDao` defines what would traditionally be known as **CRUD** database operations. The CRUD operations consist of:

+ *C*: Create. Create new objects in the database.
+ *R*: Read. Read objects from the database.
+ *U*: Update. Update objects in the database.
+ *D*: Delete. Delete objects in the database.

All access to the recipe data will be through this class. You can name the methods anything you like, but the real power is in the annotations. The `@Query`, `@Insert`, `@Update` and `@Delete` annotations give Room valuable information. Room uses this to generate the code that automatically converts your data entities to database rows and vice versa.

This class introduced several new concepts:

2. The `@Dao` annotation tells Room this is a **Data Access Object**. DAO classes must be either interfaces or abstract classes. Room creates the concrete class at runtime based on the method definitions you define.

4. You defined `addRecipe()` with the `@Insert` annotation. This saves a single `RecipeDb` object to the database and returns the new primary key ID associated with the new recipe. The `onConflict` attribute of the `@Insert` annotation defines what happens if there’s an existing record with the same primary key.

    > **Note**: To learn more about conflict options, please see this page: [https://developer.android.com/reference/androidx/room/OnConflictStrategy](https://developer.android.com/reference/androidx/room/OnConflictStrategy). For more information on the underlying details around each conflict strategy and how SQLite defines them, see: [https://sqlite.org/lang\_conflict.html](https://sqlite.org/lang_conflict.html).

6. This method returns a single `RecipeDb` object. Here, you used the `@Query` annotation to tell Room how to retrieve a single recipe. This method loads a `RecipeDb` object based on the `id`. To do the database query, Room takes the arguments passed into your method and replaces the matching `:?` strings in the query, where `?` matches an argument name on the method. In this case, it replaces `:id` with the value of the `id` argument passed into `findRecipeById()`.

8. `getAllRecipes()` uses the `@Query` annotation to define a SQL statement to read all the recipes from the database and return them as a `List` of `Recipes`.

    > **Note**: SQL, which stands for Structured Query Language, is a well-known method for working with relational databases such as SQLite. You won’t need to know a lot of SQL to build the app. If you want to learn more about SQL, and specifically the syntax used for SQLite, read [https://sqlite.org/lang.html](https://sqlite.org/lang.html).

10. You defined `updateRecipeDetails()` with the `@Update` annotation. This updates a single recipe in the database using the passed-in `recipe` argument.

12. Finally, you defined `deleteRecipe()` using an `@Delete` annotation and `deleteRecipeById()` using the `@Query` annotation with a custom `DELETE` statement. This deletes an existing recipe based on the passed-in `RecipeDb` object or `recipeId`.

Create a Kotlin file named **IngredientDao.kt** in the **data/database** package and replace the contents with the following:

```
import androidx.room.Dao
import androidx.room.Delete
import androidx.room.Insert
import androidx.room.OnConflictStrategy
import androidx.room.Query
import androidx.room.Update

@Dao
interface IngredientDao {
  @Insert(onConflict = OnConflictStrategy.IGNORE)
  suspend fun addIngredient(ingredientDb: IngredientDb)

  @Query("SELECT * FROM ingredients WHERE id = :id")
  suspend fun findIngredientById(id: Int): IngredientDb

  @Query("SELECT * FROM ingredients WHERE recipeId = :id")
  suspend fun findIngredientsByRecipe(id: Int): List<IngredientDb>

  @Query("SELECT * FROM ingredients")
  suspend fun getAllIngredients(): List<IngredientDb>

  @Update
  suspend fun updateIngredientDetails(ingredientDb: IngredientDb)

  @Delete
  suspend fun deleteIngredient(ingredientDb: IngredientDb)
}
```

This is like the `RecipeDao` class.

### Database

The last piece needed to complete the Room classes is the Database.

Create a Kotlin file named **RecipeDatabase.kt** in the **data/database** package and replace the contents with the following:

```
import android.content.Context
import androidx.room.Database
import androidx.room.Room
import androidx.room.RoomDatabase

// 1
@Database(entities = [RecipeDb::class, IngredientDb::class], version = 1, exportSchema = false)
abstract class RecipeDatabase : RoomDatabase() {
  // 2
  abstract fun recipeDao(): RecipeDao
  abstract fun ingredientDao(): IngredientDao

  // 3
  companion object {
      /*The value of a volatile variable will never be cached, and all writes and reads will be done to and from the main memory.
      This helps make sure the value of INSTANCE is always up-to-date and the same for all execution threads.
      It means that changes made by one thread to INSTANCE are visible to all other threads immediately.*/
      @Volatile
      // 4
      private var INSTANCE: RecipeDatabase? = null

      // 5
      fun getInstance(context: Context): RecipeDatabase {
        // only one thread of execution at a time can enter this block of code
        synchronized(this) {
          var instance = INSTANCE

         // 6
          if (instance == null) {
            instance = Room.databaseBuilder(
                context.applicationContext,
                RecipeDatabase::class.java,
                "recipe_database"
            ).fallbackToDestructiveMigration()
                .build()

            INSTANCE = instance
          }
          // 7
          return instance
        }
      }
  }
}
```

Here’s how this code works:

2. The `@Database` annotation identifies a `Database` class to Room. `entities` is a required attribute in the `@Database` annotation and defines an array of all entities the database uses. This database will store the two entities.

    Room requires your database class to be abstract and inherit from `RoomDatabase`.

4. The abstract methods `recipeDao` and `ingredientDao` are defined to return a DAO interface. Note that you can have as many DAOs as you like. You declare this as abstract because Room implements `RecipeDao` and `IngredientDao` classes for you based on the interfaces you defined earlier.

    This is all the `Database` class requires. The rest of the code lets you use the `Database` interface object as a singleton. Google recommends this because spinning new `Database` objects can be expensive.

6. Define a `companion object` on `RecipeDatabase`.

8. Define the only `instance` variable on the companion object.

10. Define `getInstance()` to take in a `Context` and return the single `RecipeDatabase` instance.

12. If this is the first time `getInstance` is being called, create the single `RecipeDatabase` instance. `Room.databaseBuilder()` creates a Room Database based on the abstract `RecipeDatabase` class.

14. Return the `RecipeDatabase` instance.

> **Note**: Now that you’ve defined the database, you can test a great feature of Room. It verifies the SQL in your @Query annotations at compile time.
>
> If you have an error in the SQL syntax, such as referring to a non-existent table name, it gives you an error. It also warns you if your method’s return type doesn’t match your SQL statement’s return type.
>
> Test this by changing `recipes` to `recipe` in one of the @Query strings in **RecipeDao.kt**. Notice that Android Studio marks this with an error. If you try to build the project, it yields a compile error that reads, “There is a problem with the query: \[SQLITE\_ERROR\] SQL error or missing database (no such table: recipe)”.
>
> If you ever worked with Android SQLite databases before Room was available, you realize how helpful this is. Room provides a safety net to prevent common typos in your SQL statements.

#### Creating the Repository

Your basic Room classes are ready to go. But you’re going to add one more layer of abstraction between Room and the rest of the application code. Doing this makes changing how and where you store the app data easy. This abstraction layer will be provided using a **Repository** pattern. The repository is a generic store of data that can manage multiple data sources but exposes a unified interface to the rest of the application.

You’ll create a single repository class named `RecipeRepository` to manage your recipe bookmarks. This class will internally use `RecipeDao` and `IngredientDao` from `RecipeDatabase` to access the underlying recipes and ingredients in the database. It’ll define some basic methods for saving and loading.

Create a Kotlin file named **RecipeRepository.kt** in the **data** package and replace the contents with the following:

```
import com.kodeco.recipefinder.data.database.IngredientDao
import com.kodeco.recipefinder.data.database.IngredientDb
import com.kodeco.recipefinder.data.database.RecipeDao
import com.kodeco.recipefinder.data.database.RecipeDatabase
import com.kodeco.recipefinder.data.database.RecipeDb

// 1
class RecipeRepository(recipeDatabase: RecipeDatabase) {
  // 2
  private val recipeDao: RecipeDao = recipeDatabase.recipeDao()
  private val ingredientDao: IngredientDao = recipeDatabase.ingredientDao()

  // 3
  suspend fun findAllRecipes(): List<RecipeDb> {
    return recipeDao.getAllRecipes()
  }

  suspend fun findBookmarkById(id: Int): RecipeDb {
    return recipeDao.findRecipeById(id)
  }

  suspend fun findRecipeById(id: Int): RecipeDb {
    return recipeDao.findRecipeById(id)
  }

  suspend fun findAllIngredients(): List<IngredientDb> {
    return ingredientDao.getAllIngredients()
  }

  suspend fun findRecipeIngredients(recipeId: Int): List<IngredientDb> {
    return ingredientDao.findIngredientsByRecipe(recipeId)
  }

 // 4
 suspend fun insertRecipe(recipe: RecipeDb) {
    recipeDao.addRecipe(recipe)
  }

  suspend fun insertIngredients(ingredients: List<IngredientDb>) {
    ingredients.forEach {
        ingredientDao.addIngredient(it)
    }
  }

  // 5
  suspend fun deleteRecipe(recipe: RecipeDb) {
    recipeDao.deleteRecipe(recipe)
  }

  suspend fun deleteRecipeById(recipeId: Int) {
    recipeDao.deleteRecipeById(recipeId)
  }

  suspend fun deleteIngredient(ingredient: IngredientDb) {
    ingredientDao.deleteIngredient(ingredient)
  }

  suspend fun deleteIngredients(ingredients: List<IngredientDb>) {
    ingredients.forEach {
        ingredientDao.deleteIngredient(it)
    }
  }

  suspend fun deleteRecipeIngredients(recipeId: Int) {
    val ingredients = findRecipeIngredients(recipeId)
    ingredients.forEach {
        ingredientDao.deleteIngredient(it)
    }
  }
}
```

Here’s the code breakdown:

2. Define the `RecipeRepository` class with a constructor that passes in the `RecipeDatabase`.

4. `RecipeRepository` uses these two properties for its data source. The first is the `RecipeDao`, and the second is the `DAO` object from `IngredientDao`.

6. `findAllRecipes()` returns a list of all recipes.

8. Create `insertRecipe()` to add a single recipe.

10. Add `deleteRecipe()` to delete a specific recipe.

You’ll see how to use this class in detail as you build the ViewModel.

### ViewModels

Now that you’ve built the repository, you’ll use it in a ViewModel, a perfect place for it.

First, you need to uncomment some methods to convert recipes between the repository and the UI. Open **data/Conversions.kt** and uncomment the code.

Next, open **RecipeViewModel.kt**. Find `// TODO: Add Repository` and update the ViewModel’s constructor with:

```
class RecipeViewModel(
  private val prefs: Prefs,
  private val repository: RecipeRepository,
) : ViewModel() {
```

Make sure to add the import for `RecipeRepository`.

Now, find `// TODO: get Bookmarks`. Notice you’re calling these bookmarks even though they’re recipes. This is because you’re bookmarking the recipe. Replace the method with:

```
suspend fun getBookmarks() {
  withContext(Dispatchers.IO) {
    val allRecipes = repository.findAllRecipes()
    _bookmarksState.value = recipeDbsToRecipes(allRecipes).toMutableList()
  }
}
```

You have to import some classes. This runs the call on the IO coroutine dispatcher, ensuring it runs in the background. Using the provided repository passed in to the ViewModel constructor, find all recipes that have been bookmarked and update the bookmark state (this notifies the UI of the change). Do the same for the `getIngredients()` method:

```
suspend fun getIngredients() {
  withContext(Dispatchers.IO) {
    val allIngredients = repository.findAllIngredients()
    _ingredientsState.value = ingredientDbsToIngredients(allIngredients).toMutableList()
  }
}
```

Make sure to add the import for `ingredientDbsToIngredients`. This gets all the ingredients, converts them to UI models and sets the ingredient state list.

To get a bookmark, replace `// TODO: Get Bookmark` with:

```
suspend fun getBookmark(bookmarkId: Int) {
  withContext(Dispatchers.IO) {
    val recipe = repository.findRecipeById(bookmarkId)
    val ingredients = repository.findRecipeIngredients(bookmarkId)
    _recipeState.value =
        recipeDbToRecipeInformation(recipe, ingredientDbsToExtendedIngredients(ingredients))
  }
}
```

Add the imports for `recipeDbToRecipeInformation` and `ingredientDbsToExtendedIngredients`. This finds the recipe and its ingredients and creates recipe information.

To save an individual bookmark, replace the `bookmarkRecipe()` method with:

```
suspend fun bookmarkRecipe(recipe: RecipeInformationResponse) {
  withContext(Dispatchers.IO) {
    repository.insertRecipe(recipeInformationToRecipeDb(recipe))
    repository.insertIngredients(
      extendedIngredientsToIngredientDbs(
        recipe.id,
        recipe.extendedIngredients
      )
    )
  }
}
```

Add imports for `recipeInformationToRecipeDb` and `extendedIngredientsToIngredientDbs`. Here, you need to insert the recipe and its ingredients.

To delete a recipe, find the first `// TODO: Delete Bookmark` and replace the method with:

```
suspend fun deleteBookmark(recipe: Recipe) {
  withContext(Dispatchers.IO) {
    // 1
    val recipeDb = recipeToDb(recipe)
    // 2
    repository.deleteRecipe(recipeDb)
    repository.deleteRecipeIngredients(recipe.id)
    // 3
    val localList = _bookmarksState.value.toMutableList()
    // 4
    localList.remove(recipe)
    _bookmarksState.value = localList
  }
}
```

Make sure to add the import for `recipeToDb`. Here’s what you did:

2. Convert the UI recipe model to a DB recipe model.
4. Delete the recipe and ingredients from the repository.
6. Convert the current list of bookmarks to a mutable list.
8. Remove the recipe from the list and update the list in the bookmark state.

Find the next `// TODO: Delete Bookmark`. This is an alternative way to delete the bookmark. This method does the trick if you have just the recipe ID and not the recipe itself. Replace the method with:

```
suspend fun deleteBookmark(recipeId: Int) {
  withContext(Dispatchers.IO) {
    repository.deleteRecipeById(recipeId)
    repository.deleteRecipeIngredients(recipeId)
    val localList = _bookmarksState.value.toMutableList()
    localList.removeIf {
      it.id == recipeId
    }
    _bookmarksState.value = localList
  }
}
```

This is like the code you wrote before. The main difference is that you’re using functions from the repository that require a `recipeId`.

This finishes the ViewModel.

### Instantiating the Repository

Now that you’ve set up repository usage, it’s time to create it. Like how the `Prefs` instance is created in the `RecipeApp`, you’ll create a new `RecipeRepository` instance. Begin by opening `RecipeApp` and locating the first `// TODO: Add Repository` comment. Replace the comment with:

```
lateinit var repository: RecipeRepository
```

Add the import for `RecipeRepository`. Now, locate the second `// TODO: Add Repository` comment and replace it with:

```
repository = RecipeRepository(
  Room.databaseBuilder(
    this,
    RecipeDatabase::class.java,
    "Recipes"
  ).build()
)
```

This creates a new instance of the repository using Room. You’ll need to import `RecipeDatabase` and `Room`.

### Local Repository Provider

A lot of classes use the repository. How can you provide that repository to all composables in the UI? By using the Local Provider concept. This is a way to provide classes to other composables. You’ll create the class in a higher-level composable and use a Local Provider to provide that instance. Open **MainActivity.kt** and, after the `LocalNavigatorProvider` global variable, add:

```
val LocalRepositoryProvider =
    compositionLocalOf<RecipeRepository> { error("No repository provided") }
```

This creates a global variable that’s a local provider.

You’ll need to import:

```
import com.kodeco.recipefinder.data.RecipeRepository
```

Find the `// TODO: Add LocalRepositoryProvider` and replace it with:

```
LocalRepositoryProvider provides (application as RecipeApp).repository,
```

Add the imports for `LocalRepositoryProvider` and `RecipeApp`.

This uses the `RecipeRepository` created and stored in the `RecipeApp` and adds it to your local repository provider.

### Bookmarks

It’s time to update the **ui/recipes/ShowBookmarks.kt** file. Find the `// TODO: Provide current item` comment and replace the method call below it with:

```
viewModel.deleteBookmark(currentItem)
```

### Recipe Details

The last file to update is **ui/RecipeDetails.kt**. Find the first `// TODO: Add Repository` and replace it with:

```
val repository = LocalRepositoryProvider.current
```

And import `LocalRepositoryProvider`. Then, replace the `RecipeViewModel` factory instantiation with:

```
RecipeViewModel(prefs, repository)
```

Find the `// TODO: Provide recipe ID` and replace the method call below it with:

```
viewModel.getBookmark(databaseRecipeId)
```

Further down, find the next `// TODO: Provide recipe ID` and replace it with:

```
 viewModel.deleteBookmark(recipe.id)
```

To delete the bookmark with the given recipe ID. Find the next `// TODO: Provide recipe` and replace it with:

```
viewModel.bookmarkRecipe(recipe)
```

To do the opposite and add the bookmark.

### Updating the ViewModel References

Because you updated the `RecipeViewModel` to take in a repository, you must fix the instantiation in the `GroceryList` and `RecipeList` composable functions. To fix that, open the following files

+ **ui/recipes/RecipeList.kt**
+ **ui/groceries/GroceryList.kt**

Find `// TODO: Add Repository` and add:

```
val repository = LocalRepositoryProvider.current
```

And import `LocalRepositoryProvider`.

This gets the current repository. Now, update the `RecipeViewModel` instantiation with:

```
RecipeViewModel(prefs, repository)
```

### Updating Previews

The last requirement before building and running the changes is to update the preview composables. Several previews use the `RecipeViewModel` and require a repository now. In each of the following classes:

+ **ui/recipes/ChipRow.kt**
+ **ui/recipes/SearchRow.kt**
+ **ui/recipes/ShowBookmarks.kt**
+ **ui/recipes/ShowRecipeList.kt**

Look for the `// TODO: Add Repository` comment in the related preview methods at the bottom of each of the files and replace it with:

```
val repository = LocalRepositoryProvider.current
```

Add the import for `LocalRepositoryProvider`. Then, update the `RecipeViewModel` instantiation with:

```
RecipeViewModel(prefs, repository)
```

Finally, you’re ready to test and make sure your app works. Run the app and search for a food you like. Tap the image to go to the details, and then tap the bookmark icon:

 ![](./Android Fundamentals by Tutorials, Chapter 10_Room Database_ Kodeco_files/original.png)

When you tap the bookmark icon, you’re returned to the list. Tap the Bookmark button at the top:

 ![](./Android Fundamentals by Tutorials, Chapter 10_Room Database_ Kodeco_files/original(1).png)

 ![](./Android Fundamentals by Tutorials, Chapter 10_Room Database_ Kodeco_files/original(2).png)

To delete the bookmark, swipe left or right. To view the recipe, tap the card. Congratulations! You have a fully functioning recipe finder app that can save bookmarked recipes.

### Groceries

If you tap the groceries bottom button, you see there are no groceries. To fix that, open **ui/groceries/GroceryList.kt**. Find `// TODO: Get Ingredients` and replace it with:

```
scope.launch {
  recipeViewModel.getIngredients()
}
```

This retrieves the current list of ingredients. If you look at the code above:

```
scope.launch {
  recipeViewModel.ingredientsState.collect { ingredients ->
    groceryListViewModel.setIngredients(ingredients)
  }
}
```

You can see it’s listening for changes to the list of ingredients. Relaunch the app and ensure that ingredients appear on the Groceries page.

 ![](./Android Fundamentals by Tutorials, Chapter 10_Room Database_ Kodeco_files/original(3).png)

#### Alternatives

You could use SQLite and the classes that surround it, but it takes a bit of work to create and maintain the database. Here are a few alternatives:

+ **GreenDAO** ([https://greenrobot.org/greendao/](https://greenrobot.org/greendao/)): Open-source ORM that’s easy to use.
+ **Realm** ([https://realm.io/](https://realm.io/)): A fast database that requires low-level C++ code and doesn’t use SQLite.
+ **Firebase Realtime Database** ([https://firebase.google.com/docs/database/](https://firebase.google.com/docs/database/)): NoSQL database hosted in the cloud. Paid.
+ **Apollo/GraphQL** ([https://graphql.org/](https://graphql.org/)): Only use this if you have remote databases that use GraphQL. GraphQL is a newer format created by Facebook that is more flexible than the REST format. Many companies are moving to this format.

## Key Points

+ Room is a great way to create databases and save data.
+ It’s easy to create a database and store recipes.

## Where to Go From Here?

In this chapter, you learned how to store data in a database.

To learn more about Room, go to: [https://developer.android.com/training/data-storage/room](https://developer.android.com/training/data-storage/room).

In the next chapter, you’ll learn about Advanced Storage techniques. See you there!
