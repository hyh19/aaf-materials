# 10. Room 数据库

到目前为止，你已经有了一个很棒的应用程序，可以在互联网上搜索食谱。但是还没有书签或杂货功能。当你去商店时，你会希望有一份这些食谱所需的配料清单。难道你想在商店里还要再次搜索才能获取这些信息吗？

## SQLite

持久化数据的最佳方式之一是使用数据库。Android 提供了对 **SQLite** 数据库系统的访问。这让你可以插入、读取、更新和移除持久化在磁盘上的结构化数据。

在本章中，你将学习如何使用 **Room** 库。

在本章结束时，你将了解：

+ 如何插入、获取和移除食谱或配料。
+ 如何使用仓库模式（Repository pattern）提供执行这些操作的通用方法。

### Room

Room 是谷歌在 2018 年创建的库，它在 Android 提供的内置 SQLite 数据库之上添加了一层封装，使其更易于使用。

在深入代码之前，了解 Room 的三个基本组件很重要：

2. **数据库**：这是与底层 SQLite 数据库交互的主要接口。这个组件维护一个或多个**数据访问对象**（DAOs）并使用数据库使用的所有**实体**列表进行注解。数据库类继承自 `RoomDatabase` 并使用 `@Database` 注解。

4. **实体**：这代表存储在数据库中的单个数据类型。Room 为每个实体在数据库中创建一个表，表中的行代表单个实体项。

    实体被定义为使用 `@Entity` 注解的简单类。除非你使用 `@Ignore` 注解，否则所有实体类属性都会自动定义为数据库中的字段。你应该使用 `@PrimaryKey` 注解将至少一个实体属性指定为主键。

6. **DAO**：数据访问对象是 Room 的英雄。这里是你定义访问数据库的接口的地方。DAO 应该是你的应用程序中直接与数据库交谈的唯一部分。数据库类必须包含至少一个返回使用 DAO 注解的接口的抽象方法。

下图说明了这三个组件：

获取/设置属性 读取/写入实体 从数据库获取 DAOs 主应用程序代码 数据库标签 DAOs 标签 实体标签 实体 SQLite 数据库

#### Room 和 Android 架构组件

Room 是一组名为 **Android 架构组件**的更大库的一部分。其他组件包括：

+ **生命周期管理**：提供几个类来帮助构建生命周期感知对象。
+ **LiveData**：保存可以观察变化的数据并尊重生命周期。
+ **ViewModel**：管理与视图相关的数据，而不与配置更改绑定。这是 UI 视图和应用程序其余部分之间的桥梁。

现在不要担心这些组件的细节；你将在构建应用程序时更详细地介绍它们。

#### 应用架构

在创建第一个 Room 类之前，你必须组织应用程序以实现清晰的架构。你将沿着以下几条线将应用程序分为不同的责任区域：

+ 数据访问和持久化（Room）。
+ 数据模型（Model）。
+ 数据抽象（Repository）。
+ 业务/领域逻辑（ViewModel）。
+ 用户界面（Activity/Fragment）。

一个关键目标是确保这些层之间的通信只朝一个方向流动。这导致了松散耦合的架构，易于修改而没有副作用。

架构看起来是这样的：

仓库 数据访问 持久化 数据模型 UI（Compose）ViewModel

箭头代表通信和可见性线。请注意，UI 层完全独立于除 ViewModel 之外的所有其他层。ViewModel 层对 UI 层一无所知。

当你构建应用程序的其余部分时，你不会在坚持严格遵循上图所示的通信流程方面做出妥协。有时需要做更多的工作才能严格遵守此模式，但对于较大的应用程序来说，付出的努力是值得的。即使对于一个小型应用程序，你也可以立即认识到一些好处：

+ 在 Room 中存储数据的方式可以完全替换，影响最小。唯一受影响的层是**持久化**层及其直接父级，即**数据访问**层。

+ 你可以替换 UI 层，而不会让任何其他层知道。

+ 你可以在没有任何活动 UI 运行的情况下轻松测试所有层。

#### 开发方法

将架构想象成一个多层蛋糕。你有没有见过有人一次吃一层蛋糕？这有点奇怪！同样，你不会一次构建一层应用程序。你会一次取一片。每一片可能会穿过所有层，你慢慢构建最终产品。

UI（Compose）ViewModel 仓库 数据访问 持久化（Room）数据模型 架构蛋糕 一次一片

以下是你将使用的目录：

+ **data/database**：数据访问和持久化。你将在这里保存 **Room 数据库**和 **DAO** 对象。
+ **data/models**：模型对象。这包括所有 **Room 实体**类。
+ **ui**：用户界面。所有视图和视图控制逻辑都属于这里。
+ **viewmodels**：业务/领域逻辑。这包含驱动用户界面和应用程序逻辑的 ViewModel 类。

#### 添加 Room 库

如果你正在跟随前几章的应用程序，请打开并继续使用它进行本章的内容。如果没有，请找到本章的 **projects** 文件夹并在 Android Studio 中打开 **starter**。

如果你使用的是 starter 项目，请打开 **SpoonacularService.kt** 文件，并使用你在 [https://www.spoonacular.com](https://www.spoonacular.com/) 创建的账户中的 API 密钥更新 **apiKey**。

打开 **libs.versions.toml** 文件添加 Room 库。在 **versions** 部分的末尾，添加：

```
room="2.5.2"
```

然后，在 **libraries** 部分的末尾，添加：

```
# Room
room = { module= "androidx.room:room-ktx", version.ref="room" }
room-runtime ={ module= "androidx.room:room-runtime", version.ref="room" }
room-compiler = { module = "androidx.room:room-compiler", version.ref="room" }
```

最后，打开应用模块的 **build.gradle.kts** 文件并添加：

```
id("kotlin-parcelize")
```

作为 **plugins** 部分的最后一行。使用 `@Parcelize` 注解时，此插件会为 `Parcelable` 类型自动生成代码。

然后，在 **dependencies** 部分，添加：

```
// Room
implementation(libs.room)
implementation(libs.room.runtime)
ksp (libs.room.compiler)
```

并执行 Gradle 同步。

## Room 类

现在，你已经准备好添加 Room 所需的基本类。这包括实体、DAO 和数据库。在后台，Room 会根据你的类结构创建一个带有表和列定义的 SQLite 数据库。

对于 Room，将数据库命名为：`recipe_database`，模型类命名为：`RecipeDb` 和 `IngredientDb`。下图将帮助你可视化 Room 用于将类转换为底层数据库的过程：

数据库 RecipeDao IngredientDao RecipeDatabase id:Int title: String image: String? summary: String instructions:String? sourceUrl: String preparationMinutes: Int cookingMinutes: Int readyInMinutes: Int servings: Int 模型实体 RecipeDb id:Int recipeId: Int? name: String aisle:String? image: String? original: String amount: Double unit: String IngredientDb recipe\_database id title image summary instructions sourceUrl Recipe 表 id recipeId name aisle image original Ingredient 表 Room Room Room Room 数据库创建过程

### 实体

Recipe Finder 需要两种实体类型来存储食谱：`RecipeDb` 和 `IngredientDb`。

#### RecipeDb

在 **data** 包中创建一个名为 **database** 的包。在这个包中，创建一个名为 **RecipeDb.kt** 的 Kotlin 文件，并将内容替换为以下内容：

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

以下是上面代码的解释：

2. Kotlin 使用 `@Parcelize` 注解来生成类的 `Parcelable` 实现。`Parcelable` 是一个接口，用于序列化和反序列化对象。这对于在 Android 应用中的活动、片段或其他组件之间传输数据很有用。

4. `@Entity` 注解告诉 Room 这是一个数据库实体类。

    > **注意**：虽然在这个例子中没有使用，但你可以对 Entity 注解应用几个属性。
    >
    > `foreignKeys()`：外键约束列表。
    >
    > `indices()`：要包含在表中的索引列表。
    >
    > `primaryKeys()`：主键列名称列表。如果使用 `PrimaryKey` 注解，则不需要。
    >
    > `tableName()`：在数据库中使用的表名。默认为类名。

6. `RecipeDb` 类的主构造函数使用定义了默认值的所有属性的参数来定义。定义默认值允许你使用部分属性列表构造食谱。

    > **注意**：Room 在定义表字段时查找构造函数上的参数和类属性。在这种情况下，你只使用属性来定义表字段。

8. 你使用 `@PrimaryKey` 注解定义了 `id` 属性。每个实体类必须至少有一个这样的注解。`autoGenerate` 属性自动告诉 Room 为这个字段生成递增的数字。

    在数据库术语中，这被认为是一个代理键或合成键，为每个食谱记录提供唯一标识符。

10. 你定义了其余的字段，并带有默认值。

#### IngredientDb

在 **data/database** 包中创建一个名为 **IngredientDb.kt** 的 Kotlin 文件，并将内容替换为以下内容：

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

这与 `RecipeDb` 类类似。

### DAOs

接下来，你将定义数据访问对象，用于从数据库读取和写入数据。

在 **data/database** 包中创建一个名为 **RecipeDao.kt** 的 Kotlin 文件，并将内容替换为以下内容：

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

`RecipeDao` 定义了传统上称为 **CRUD** 数据库操作。CRUD 操作包括：

+ *C*：创建。在数据库中创建新对象。
+ *R*：读取。从数据库读取对象。
+ *U*：更新。更新数据库中的对象。
+ *D*：删除。从数据库中删除对象。

对食谱数据的所有访问都将通过这个类。你可以随意命名这些方法，但真正的力量在于注解。`@Query`、`@Insert`、`@Update` 和 `@Delete` 注解为 Room 提供了有价值的信息。Room 使用这些信息生成代码，自动将数据实体转换为数据库行，反之亦然。

这个类引入了几个新概念：

2. `@Dao` 注解告诉 Room 这是一个**数据访问对象**。DAO 类必须是接口或抽象类。Room 根据你定义的方法定义在运行时创建具体类。

4. 你使用 `@Insert` 注解定义了 `addRecipe()`。这将单个 `RecipeDb` 对象保存到数据库，并返回与新食谱关联的新主键 ID。`@Insert` 注解的 `onConflict` 属性定义了当存在具有相同主键的现有记录时会发生什么。

    > **注意**：要了解有关冲突选项的更多信息，请参阅此页面：[https://developer.android.com/reference/androidx/room/OnConflictStrategy](https://developer.android.com/reference/androidx/room/OnConflictStrategy)。有关每种冲突策略的底层详细信息以及 SQLite 如何定义它们的更多信息，请参阅：[https://sqlite.org/lang\_conflict.html](https://sqlite.org/lang_conflict.html)。

6. 这个方法返回单个 `RecipeDb` 对象。在这里，你使用 `@Query` 注解告诉 Room 如何检索单个食谱。此方法基于 `id` 加载 `RecipeDb` 对象。要执行数据库查询，Room 接受传递到你的方法中的参数，并替换查询中匹配的 `:?` 字符串，其中 `?` 匹配方法上的参数名称。在这种情况下，它用传递给 `findRecipeById()` 的 `id` 参数的值替换 `:id`。

8. `getAllRecipes()` 使用 `@Query` 注解来定义一个 SQL 语句，用于从数据库读取所有食谱并将它们作为 `List` 的 `Recipes` 返回。

    > **注意**：SQL，即结构化查询语言，是一种众所周知的方法，用于处理关系型数据库，如 SQLite。构建应用程序时，你不需要了解太多 SQL。如果你想了解更多关于 SQL 的信息，特别是 SQLite 使用的语法，请阅读 [https://sqlite.org/lang.html](https://sqlite.org/lang.html)。

10. 你使用 `@Update` 注解定义了 `updateRecipeDetails()`。这使用传入的 `recipe` 参数更新数据库中的单个食谱。

12. 最后，你使用 `@Delete` 注解定义了 `deleteRecipe()` 和使用带有自定义 `DELETE` 语句的 `@Query` 注解定义了 `deleteRecipeById()`。这基于传入的 `RecipeDb` 对象或 `recipeId` 删除现有食谱。

在 **data/database** 包中创建一个名为 **IngredientDao.kt** 的 Kotlin 文件，并将内容替换为以下内容：

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

这与 `RecipeDao` 类类似。

### 数据库

完成 Room 类所需的最后一个部分是数据库。

在 **data/database** 包中创建一个名为 **RecipeDatabase.kt** 的 Kotlin 文件，并将内容替换为以下内容：

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
      /*volatile 变量的值永远不会被缓存，所有的写入和读取都将直接在主内存中完成。
      这有助于确保 INSTANCE 的值始终是最新的，并且对所有执行线程都是相同的。
      这意味着一个线程对 INSTANCE 所做的更改对所有其他线程立即可见。*/
      @Volatile
      // 4
      private var INSTANCE: RecipeDatabase? = null

      // 5
      fun getInstance(context: Context): RecipeDatabase {
        // 一次只有一个执行线程可以进入此代码块
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

这段代码的工作原理如下：

2. `@Database` 注解向 Room 标识一个 `Database` 类。`entities` 是 `@Database` 注解中的必需属性，它定义了数据库使用的所有实体的数组。这个数据库将存储两个实体。

    Room 要求你的数据库类是抽象的并继承自 `RoomDatabase`。

4. 抽象方法 `recipeDao` 和 `ingredientDao` 被定义为返回 DAO 接口。请注意，你可以拥有任意多的 DAO。你将其声明为抽象的，因为 Room 根据你之前定义的接口为你实现 `RecipeDao` 和 `IngredientDao` 类。

    这就是 `Database` 类所需的全部内容。其余代码让你将 `Database` 接口对象用作单例。谷歌推荐这样做，因为创建新的 `Database` 对象可能很昂贵。

6. 在 `RecipeDatabase` 上定义一个 `companion object`。

8. 在伴生对象上定义唯一的 `instance` 变量。

10. 定义 `getInstance()` 接受一个 `Context` 并返回单个 `RecipeDatabase` 实例。

12. 如果这是第一次调用 `getInstance`，则创建单个 `RecipeDatabase` 实例。`Room.databaseBuilder()` 基于抽象的 `RecipeDatabase` 类创建一个 Room 数据库。

14. 返回 `RecipeDatabase` 实例。

> **注意**：现在你已经定义了数据库，你可以测试 Room 的一个很棒的功能。它在编译时验证 @Query 注解中的 SQL。
>
> 如果 SQL 语法有错误，例如引用了不存在的表名，它会给你一个错误。如果你的方法的返回类型与 SQL 语句的返回类型不匹配，它也会警告你。
>
> 通过将 **RecipeDao.kt** 中的一个 @Query 字符串中的 `recipes` 更改为 `recipe` 来测试这一点。注意 Android Studio 会将其标记为错误。如果你尝试构建项目，它会产生一个编译错误，内容为："查询存在问题：\[SQLITE\_ERROR\] SQL 错误或缺少数据库（没有这样的表：recipe）"。
>
> 如果你曾经在 Room 可用之前使用过 Android SQLite 数据库，你会意识到这有多么有用。Room 提供了一个安全网，防止 SQL 语句中的常见拼写错误。

#### 创建仓库

你的基本 Room 类已经准备就绪。但是你将在 Room 和应用程序其余代码之间添加另一层抽象。这样做可以轻松更改应用程序数据的存储方式和位置。这个抽象层将使用**仓库**模式提供。仓库是一个通用的数据存储，可以管理多个数据源，但向应用程序的其余部分公开统一的接口。

你将创建一个名为 `RecipeRepository` 的单一仓库类来管理你的食谱书签。这个类将在内部使用 `RecipeDatabase` 中的 `RecipeDao` 和 `IngredientDao` 来访问底层的食谱和配料。它将定义一些用于保存和加载的基本方法。

在 **data** 包中创建一个名为 **RecipeRepository.kt** 的 Kotlin 文件，并将内容替换为以下内容：

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

以下是代码的详细解析：

2. 定义带有一个构造函数的 `RecipeRepository` 类，该构造函数传入 `RecipeDatabase`。

4. `RecipeRepository` 使用这两个属性作为其数据源。第一个是 `RecipeDao`，第二个是来自 `IngredientDao` 的 `DAO` 对象。

6. `findAllRecipes()` 返回所有食谱的列表。

8. 创建 `insertRecipe()` 来添加单个食谱。

10. 添加 `deleteRecipe()` 来删除特定食谱。

在构建 ViewModel 时，你将看到如何详细使用这个类。

### ViewModels

现在你已经构建了仓库，你将在 ViewModel 中使用它，这是一个放置它的完美位置。

首先，你需要取消注释一些方法，用于在仓库和 UI 之间转换食谱。打开 **data/Conversions.kt** 并取消注释代码。

接下来，打开 **RecipeViewModel.kt**。找到 `// TODO: Add Repository` 并用以下内容更新 ViewModel 的构造函数：

```
class RecipeViewModel(
  private val prefs: Prefs,
  private val repository: RecipeRepository,
) : ViewModel() {
```

确保添加 `RecipeRepository` 的导入。

现在，找到 `// TODO: get Bookmarks`。请注意，你将这些称为书签，即使它们是食谱。这是因为你在给食谱添加书签。用以下内容替换该方法：

```
suspend fun getBookmarks() {
  withContext(Dispatchers.IO) {
    val allRecipes = repository.findAllRecipes()
    _bookmarksState.value = recipeDbsToRecipes(allRecipes).toMutableList()
  }
}
```

你必须导入一些类。这将在 IO 协程调度器上运行调用，确保它在后台运行。使用传递给 ViewModel 构造函数的仓库，查找所有已添加书签的食谱并更新书签状态（这通知 UI 变化）。对 `getIngredients()` 方法做同样的事情：

```
suspend fun getIngredients() {
  withContext(Dispatchers.IO) {
    val allIngredients = repository.findAllIngredients()
    _ingredientsState.value = ingredientDbsToIngredients(allIngredients).toMutableList()
  }
}
```

确保添加 `ingredientDbsToIngredients` 的导入。这获取所有配料，将它们转换为 UI 模型并设置配料状态列表。

要获取书签，将 `// TODO: Get Bookmark` 替换为：

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

添加 `recipeDbToRecipeInformation` 和 `ingredientDbsToExtendedIngredients` 的导入。这找到食谱及其配料并创建食谱信息。

要保存单个书签，将 `bookmarkRecipe()` 方法替换为：

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

添加 `recipeInformationToRecipeDb` 和 `extendedIngredientsToIngredientDbs` 的导入。在这里，你需要插入食谱及其配料。

要删除食谱，找到第一个 `// TODO: Delete Bookmark` 并将该方法替换为：

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

确保添加 `recipeToDb` 的导入。以下是你所做的：

2. 将 UI 食谱模型转换为 DB 食谱模型。
4. 从仓库中删除食谱和配料。
6. 将当前书签列表转换为可变列表。
8. 从列表中删除食谱并更新书签状态中的列表。

找到下一个 `// TODO: Delete Bookmark`。这是删除书签的另一种方式。如果你只有食谱 ID 而不是食谱本身，这个方法就能派上用场。将该方法替换为：

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

这与你之前编写的代码类似。主要区别在于你使用的是需要 `recipeId` 的仓库函数。

这完成了 ViewModel。

### 实例化仓库

现在你已经设置了仓库的使用，是时候创建它了。就像 `RecipeApp` 中创建 `Prefs` 实例的方式一样，你将创建一个新的 `RecipeRepository` 实例。首先打开 `RecipeApp` 并找到第一个 `// TODO: Add Repository` 注释。用以下内容替换该注释：

```
lateinit var repository: RecipeRepository
```

添加 `RecipeRepository` 的导入。现在，找到第二个 `// TODO: Add Repository` 注释并用以下内容替换：

```
repository = RecipeRepository(
  Room.databaseBuilder(
    this,
    RecipeDatabase::class.java,
    "Recipes"
  ).build()
)
```

这使用 Room 创建了仓库的新实例。你需要导入 `RecipeDatabase` 和 `Room`。

### 本地仓库提供者

很多类使用仓库。你如何向 UI 中的所有可组合项提供该仓库呢？通过使用本地提供者概念。这是一种向其他可组合项提供类的方法。你将在更高级别的可组合项中创建类，并使用本地提供者来提供该实例。打开 **MainActivity.kt** 并在 `LocalNavigatorProvider` 全局变量之后添加：

```
val LocalRepositoryProvider =
    compositionLocalOf<RecipeRepository> { error("No repository provided") }
```

这创建了一个作为本地提供者的全局变量。

你需要导入：

```
import com.kodeco.recipefinder.data.RecipeRepository
```

找到 `// TODO: Add LocalRepositoryProvider` 并用以下内容替换：

```
LocalRepositoryProvider provides (application as RecipeApp).repository,
```

添加 `LocalRepositoryProvider` 和 `RecipeApp` 的导入。

这使用在 `RecipeApp` 中创建和存储的 `RecipeRepository` 并将其添加到你的本地仓库提供者中。

### 书签

现在是时候更新 **ui/recipes/ShowBookmarks.kt** 文件了。找到 `// TODO: Provide current item` 注释并用以下内容替换其下方的方法调用：

```
viewModel.deleteBookmark(currentItem)
```

### 食谱详情

要更新的最后一个文件是 **ui/RecipeDetails.kt**。找到第一个 `// TODO: Add Repository` 并用以下内容替换：

```
val repository = LocalRepositoryProvider.current
```

并导入 `LocalRepositoryProvider`。然后，用以下内容替换 `RecipeViewModel` 工厂实例化：

```
RecipeViewModel(prefs, repository)
```

找到 `// TODO: Provide recipe ID` 并用以下内容替换其下方的方法调用：

```
viewModel.getBookmark(databaseRecipeId)
```

再往下，找到下一个 `// TODO: Provide recipe ID` 并用以下内容替换：

```
 viewModel.deleteBookmark(recipe.id)
```

以便使用给定的食谱 ID 删除书签。找到下一个 `// TODO: Provide recipe` 并用以下内容替换：

```
viewModel.bookmarkRecipe(recipe)
```

这是相反的操作，添加书签。

### 更新 ViewModel 引用

因为你更新了 `RecipeViewModel` 以接收仓库，所以必须修复 `GroceryList` 和 `RecipeList` 可组合函数中的实例化。要修复这个问题，请打开以下文件

+ **ui/recipes/RecipeList.kt**
+ **ui/groceries/GroceryList.kt**

找到 `// TODO: Add Repository` 并添加：

```
val repository = LocalRepositoryProvider.current
```

并导入 `LocalRepositoryProvider`。

这获取当前仓库。现在，用以下内容更新 `RecipeViewModel` 实例化：

```
RecipeViewModel(prefs, repository)
```

### 更新预览

在构建和运行更改之前的最后一个要求是更新预览可组合项。几个预览使用 `RecipeViewModel` 现在需要一个仓库。在以下每个类中：

+ **ui/recipes/ChipRow.kt**
+ **ui/recipes/SearchRow.kt**
+ **ui/recipes/ShowBookmarks.kt**
+ **ui/recipes/ShowRecipeList.kt**

在每个文件底部的相关预览方法中查找 `// TODO: Add Repository` 注释，并用以下内容替换：

```
val repository = LocalRepositoryProvider.current
```

添加 `LocalRepositoryProvider` 的导入。然后，用以下内容更新 `RecipeViewModel` 实例化：

```
RecipeViewModel(prefs, repository)
```

最后，你已经准备好测试并确保你的应用程序正常工作。运行应用程序并搜索你喜欢的食物。点击图像进入详情，然后点击书签图标：

 ![](./Android Fundamentals by Tutorials, Chapter 10_Room Database_ Kodeco_files/original.png)

当你点击书签图标时，你会返回到列表。点击顶部的书签按钮：

 ![](./Android Fundamentals by Tutorials, Chapter 10_Room Database_ Kodeco_files/original(1).png)

 ![](./Android Fundamentals by Tutorials, Chapter 10_Room Database_ Kodeco_files/original(2).png)

要删除书签，向左或向右滑动。要查看食谱，点击卡片。恭喜！你现在有一个功能齐全的食谱查找器应用程序，可以保存书签食谱。

### 杂货

如果你点击底部的杂货按钮，你会看到没有杂货。要修复这个问题，打开 **ui/groceries/GroceryList.kt**。找到 `// TODO: Get Ingredients` 并用以下内容替换：

```
scope.launch {
  recipeViewModel.getIngredients()
}
```

这检索当前的配料列表。如果你查看上面的代码：

```
scope.launch {
  recipeViewModel.ingredientsState.collect { ingredients ->
    groceryListViewModel.setIngredients(ingredients)
  }
}
```

你可以看到它正在监听配料列表的变化。重新启动应用程序并确保配料出现在杂货页面上。

 ![](./Android Fundamentals by Tutorials, Chapter 10_Room Database_ Kodeco_files/original(3).png)

#### 替代方案

你可以使用 SQLite 和围绕它的类，但创建和维护数据库需要一些工作。以下是一些替代方案：

+ **GreenDAO**（[https://greenrobot.org/greendao/](https://greenrobot.org/greendao/)）：易于使用的开源 ORM。
+ **Realm**（[https://realm.io/](https://realm.io/)）：一个快速的数据库，需要低级 C++ 代码，不使用 SQLite。
+ **Firebase Realtime Database**（[https://firebase.google.com/docs/database/](https://firebase.google.com/docs/database/)）：托管在云中的 NoSQL 数据库。付费。
+ **Apollo/GraphQL**（[https://graphql.org/](https://graphql.org/)）：仅在你有使用 GraphQL 的远程数据库时使用。GraphQL 是由 Facebook 创建的一种更灵活的新格式，比 REST 格式更灵活。许多公司正在转向这种格式。

## 要点

+ Room 是创建数据库和保存数据的好方法。
+ 创建数据库并存储食谱很容易。

## 下一步

在本章中，你学习了如何在数据库中存储数据。

要了解有关 Room 的更多信息，请访问：[https://developer.android.com/training/data-storage/room](https://developer.android.com/training/data-storage/room)。

在下一章中，你将学习高级存储技术。我们再见！
