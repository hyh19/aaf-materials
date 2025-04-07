# 11. 高级存储

有时，应用需要将信息存储到设备上的文件中。如果这些文件仅供应用使用，而不是给用户使用，你可以将它们存储在应用的文件和缓存目录中。只有你的应用可以访问这些目录，它们相对安全（黑客可能获取访问权限，但普通用户不能）。顾名思义，缓存目录文件用于缓存项目，并且它们的存储空间有限。Context 对象（Activity 实现了它）可以让你通过以下调用（从任何上下文）访问它们：

## 文件和目录

```kotlin
Context.cacheDir
Context.filesDir
```

这些 File 类允许你列出、创建和删除目录文件。以下是如何将文件写入目录的示例：

```kotlin
val file = File(context.filesDir, "test.txt")
file.bufferedWriter().use { out ->  out.write("This is a test") }
```

这会在 `context.filesDir` 目录中创建一个文件。如果你将其放在 MainActivity 中并运行应用，这个文件将在 files 目录中生成。

### Device Explorer

通过 **View ▸ Tool Windows ▸ Device Explorer** 打开 Device Explorer。你将在 **data/data** 文件夹中找到所有设备的应用文件夹：

![1743987721293](https://cdn.jsdelivr.net/gh/hyh19/images3@master/1743987719861.png)  

滚动到末尾找到 **com.kodeco.recipefinder**。你将在这里找到 **test.txt** 文件：

![1743987748852](https://cdn.jsdelivr.net/gh/hyh19/images3@master/1743987746986.png)  

双击该文件在 Android Studio 中打开它：

![1743987757234](https://cdn.jsdelivr.net/gh/hyh19/images3@master/1743987755797.png)  

如果你想查找缓存目录的大小，请在你的 Activity 中尝试以下代码：

```kotlin
lifecycleScope.launch {

  val storageManager = getSystemService(STORAGE_SERVICE) as StorageManager
  Timber.e(
    "Cache Quota: ${
      storageManager.getCacheQuotaBytes(
        storageManager.getUuidForPath(
          context.cacheDir
        )
      )
    }"
  )
}
```

因为存储管理器是一个系统服务，你必须从主线程检索它。获取服务后，调用 `getCacheQuotaBytes()` 方法。它需要路径的 UUID，并有一个方便的方法来检索它：`getUuidForPath()`。

### 缓存文件

要创建缓存文件，只需在 File 类上调用 `createTempFile()` 方法，如：

```kotlin
val tempFile = File.createTempFile(fileName, null, context.cacheDir)
```

然后，你将使用指向它的 `tempFile` File 实例访问该文件。确保在稍后尝试访问它时它仍然存在，因为系统可能会删除缓存文件。要删除文件，调用 `tempFile.delete()`。

### 外部文件

在 Android 中，有一些明确定义的目录名称，例如：

+ Download
+ Pictures
+ Movies
+ Documents

这些在 **Environment** 类中定义了常量，如：

+ DIRECTORY\_DOWNLOADS
+ DIRECTORY\_PICTURES
+ DIRECTORY\_MOVIES
+ DIRECTORY\_DOCUMENTS

要访问这些文件，使用 `Context.getExternalFilesDir()` 调用或 `Environment.getExternalStoragePublicDirectory()`。每个都返回一个 File 对象。

要将文件写入 Documents 目录，你可以使用类似这样的代码：

```kotlin
val documentFile = Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_DOCUMENTS)
val textFile = File(documentFile, "test.txt")
textFile.bufferedWriter().use { out ->  out.write("This is a test") }
```

这在模拟器上运行良好，但你不能在真实设备上访问任何文件。应用需要权限才能访问用户创建的文件。最简单的方法是使用系统文件选择器为你请求权限。如果你获得权限，你就可以访问该文件。该系统称为 **Storage Access Framework**。

### 存储访问框架

存储访问框架允许你使用系统选择器让用户为你选择要打开、创建或修改的文件。这样，你就不必通过权限系统强制用户决定是否授予你的应用写入请求目录的权限。

### 创建文件

要创建文件，使用 `ACTION_CREATE_DOCUMENT` 意图。这是一个可以运行以创建文本文件的示例函数：

```kotlin
// Request code for creating a Text document.
const val CREATE_FILE = 1

fun createFile(activity: Activity) {
  val intent = Intent(Intent.ACTION_CREATE_DOCUMENT).apply {
    addCategory(Intent.CATEGORY_OPENABLE)
    type = "text/plain"
    putExtra(Intent.EXTRA_TITLE, "test.txt")
  }
  activity.startActivityForResult(intent, CREATE_FILE)
}
```

这会在下载目录中启动系统选择器并保存名为"test.txt"的文件。添加此方法并将其绑定到按钮，例如搜索按钮。然后，使用 Device Explorer 在 Downloads 文件夹中找到该文件。

> **注意**：使用较新的 Activity Result Launchers 从结果中获取 URI。有关使用启动器的详细信息，请参阅[此 Android 开发者指南](https://developer.android.com/training/basics/intents/result)。

除了创建文件外，你还可以使用 `ACTION_OPEN_DOCUMENT` 意图打开它们。要让用户授予你的应用对特定目录的权限，使用 `ACTION_OPEN_DOCUMENT_TREE` 意图。你将无法访问以下目录：

+ Root level
+ Download
+ Android/data
+ Android/obb

一旦你收到用户选择的 URI，你只能使用该 URI 直到用户重启手机。要"保留"该访问权限（除非文件被移动或删除），你可以请求永久访问权限。为此，请使用以下示例：

```kotlin
val contentResolver = applicationContext.contentResolver

val takeFlags: Int = Intent.FLAG_GRANT_READ_URI_PERMISSION or
    Intent.FLAG_GRANT_WRITE_URI_PERMISSION

contentResolver.takePersistableUriPermission(uri, takeFlags)
```

这会导致 URI 在设备重启后被记住。

### 数据库浏览器

在上一章中，你使用 Room 创建了一个 SQLite 数据库。Android Studio 有一个数据库浏览器，可以让你轻松查看数据。要访问浏览器，使用菜单 **View ▸ Tool Windows ▸ App Inspection**。这将显示如下视图：

 ![](./Android Fundamentals by Tutorials, Chapter 11_Advanced Storage_ Kodeco_files/original(3).png)

在这里，你可以看到你的 Recipe Database，其中包含 ingredients、recipes 和一个名为 room\_master\_table 的特殊表。双击 recipes 表可查看当前存储在表中的数据：

 ![](./Android Fundamentals by Tutorials, Chapter 11_Advanced Storage_ Kodeco_files/original(4).png)

现在，打开 **RecipeDao** 文件。你会注意到左侧装订线中有几个数据库图标：

 ![](./Android Fundamentals by Tutorials, Chapter 11_Advanced Storage_ Kodeco_files/original(5).png)

通过单击该图标运行这些查询。这会弹出一个对话框，如：

 ![](./Android Fundamentals by Tutorials, Chapter 11_Advanced Storage_ Kodeco_files/original(6).png)

这里显示了一个食谱的 ID。单击"Run"可以看到类似的内容：

 ![](./Android Fundamentals by Tutorials, Chapter 11_Advanced Storage_ Kodeco_files/original(7).png)

你还可以通过键入 SQL 语句并单击新查询按钮来运行查询：

 ![](./Android Fundamentals by Tutorials, Chapter 11_Advanced Storage_ Kodeco_files/original(8).png)

新标签出现后，你可以通过键入 SQL 命令并单击"Run"来运行查询：

 ![](./Android Fundamentals by Tutorials, Chapter 11_Advanced Storage_ Kodeco_files/original(9).png)

运行足够多的查询后，你可以从历史按钮访问它们：

 ![](./Android Fundamentals by Tutorials, Chapter 11_Advanced Storage_ Kodeco_files/original(10).png)

即使你的应用在离线模式下崩溃，你也可以运行检查器。你不能进行更改，但可以查看当前缓存的数据。如果你想导出数据，单击"Export as File"按钮：

 ![](./Android Fundamentals by Tutorials, Chapter 11_Advanced Storage_ Kodeco_files/original(11).png)

然后，选择文件类型：

+ **DB**：SQLite 数据库
+ **SQL**：SQL 语句
+ **CSV**：逗号分隔值

 ![](./Android Fundamentals by Tutorials, Chapter 11_Advanced Storage_ Kodeco_files/original(12).png)

## 安全性

到目前为止，你已经学会了如何在 SQLite 数据库和 Data Store 中存储数据。请记住，Data Store 是 Android 共享首选项的较新 API。要创建安全的共享首选项文件，你必须使用 Android Security 库的一部分——加密首选项包。该库尚未转换为较新的 Data Store 格式，因此你将学习如何以其共享首选项格式使用它。

### Android Keystore

**Android Keystore** 系统是一个加密密钥容器，使密钥非常难以提取。**可信执行环境（Trusted Execution Environment，TEE）** 是与主操作系统分离的区域。如果手机具有带有自己的 CPU 和存储的安全硬件（**安全元件（Secure Element，SE）**），数据会更安全。要检查此功能，请在 API 级别 28 或更低版本上使用 `KeyInfo.isInsideSecurityHardware()`，或在 API 级别 29 或更高版本上使用 `KeyInfo.getSecurityLevel()`。这些安全硬件组件包含以下内容：

+ 独立 CPU
+ 安全存储
+ 真随机数生成器
+ 抵抗包篡改和未经授权的应用侧载的机制
+ 安全计时器
+ 重启通知引脚

要获取 Android Keystore 的实例，你可以这样写：

```kotlin
val keystore: KeyStore = KeyStore.getInstance("AndroidKeyStore").apply {
  load(null)
}
```

你可以使用 `getKey()` 方法从 Keystore 获取项目。这需要一个别名字符串并返回一个条目（通常是 `SecretKeyEntry`）。你还可以使用 `KeyPairGenerator` 类创建私钥和密钥。

还有用于系统范围凭据的 **KeyChain** API。该类具有以下一些方法：

+ **`createManageCredentialsIntent`**：用于启动应用以请求管理用户的凭据。
+ **`choosePrivateKeyAlias`**：用于启动应用以选择私钥和证书的别名。
+ **`getCertificateChain`**：返回请求别名的 X509Certificate 链。
+ **`getPrivateKey`**：返回请求别名的 PrivateKey。

### 加密首选项

加密首选项类实际上位于 **security-crypto** 库中。该库包含 **EncryptedSharedPreferences** 和 **EncryptedFile**（用于创建加密文件）。还有一个方便的 **MasterKeys** 类，其中包含用于从 Android Keystore 创建和获取主密钥的方法。

### 添加安全库

如果你正在跟随前面章节中的应用，请打开它并继续使用它进行本章。如果没有，请找到本章的 **projects** 文件夹并在 Android Studio 中打开 **starter**。打开 **gradle/libs.versions.toml**。在 **versions** 部分的末尾，添加：

```
security = "1.0.0"
```

然后，在 **\[libraries\]** 部分的末尾，添加：

```
# Security
security = { module = "androidx.security:security-crypto", version.ref = "security" }
```

执行 Gradle 同步。在 **app/build.gradle.kts** 依赖项部分的 timber 库之后，添加：

```kotlin
implementation(libs.security)
```

再次执行 Gradle 同步。

### 安全首选项

你将使用加密首选项类以加密格式保存数据。

首先转到 **app/src/main/java/com/kodeco/recipefinder/data** 目录并创建一个名为 **SecurePrefs.kt** 的 Kotlin 文件。添加以下内容：

```kotlin
import android.content.Context
import android.content.SharedPreferences
import androidx.security.crypto.EncryptedSharedPreferences
import androidx.security.crypto.MasterKeys

class SecurePrefs(context: Context) {
  private val prefs: SharedPreferences
  // TODO: Add init
  // TODO: Add get/save methods
}
```

这使用了 Android 的 `SharedPreferences` 类。不同之处在于你将创建一个加密版本。将 `// TODO: Add init` 替换为：

```kotlin
init {
  // 1
  val masterKeyAlias = MasterKeys.getOrCreate(MasterKeys.AES256_GCM_SPEC)

  // 2
  prefs = EncryptedSharedPreferences.create(
    // 3
    "encrypted_preferences", // fileName
    masterKeyAlias, // masterKeyAlias
    context, // context
    EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV, // prefKeyEncryptionScheme
    EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM // prefvalueEncryptionScheme
  )
}
```

以下是代码分解：

1. 使用 `AES256_GCM_SPEC`（一堆指定具有不同设置的加密/解密密钥的设置）创建或检索 MasterKey。
2. 使用 `EncryptedSharedPreferences` 类创建 SharedPreference 类。
3. 为文件名提供任何你想要的名称。

这些都是不错的设置，但你可以更改它们。安全方案的复杂性超出了本书的范围。有关更多信息，请参见下面的"从这里去哪里？"部分。

像前一章一样，添加用于保存和返回值的方法。将 `// TODO: Add get/save methods` 替换为：

```kotlin
fun saveString(key: String, value: String) {
  prefs.edit().putString(key, value).apply()
}

fun getString(key: String): String? {
  return prefs.getString(key, null)
}

fun saveInt(key: String, value: Int) {
  prefs.edit().putInt(key, value).apply()
}

fun getInt(key: String): Int {
  return prefs.getInt(key, 0)
}

fun hasKey(key: String): Boolean {
  return prefs.contains(key)
}
```

注意 `edit()` 和 `apply()` 方法。这是旧风格（Data Store 之前），你首先必须获取编辑器类，完成后使用 `apply()` 方法。

现在你已经编写了这个类，用它替换旧的 `Prefs` 类。打开 **MainActivity** 并更改：

```kotlin
val LocalPrefsProvider =
  compositionLocalOf<Prefs> { error("No prefs provided") }
```

为：

```kotlin
val LocalPrefsProvider =
  compositionLocalOf<SecurePrefs> { error("No prefs provided") }
```

接下来，将 `RecipeApp.kt` 中的 `prefs` 类型从：

```kotlin
lateinit var prefs: Prefs
```

改为：

```kotlin
lateinit var securePrefs: SecurePrefs
```

确保添加 `SecurePrefs` 导入并将 `prefs` 的所有实例重命名为 `securePrefs`。通过右键单击原始变量名称并在上下文菜单中选择 **Refactor** 然后选择 **Rename…** 来完成此操作。

现在，将 `RecipeApp` 的 `onCreate()` 中的实例化行从：

```kotlin
prefs = Prefs(this)
```

改为：

```kotlin
prefs = SecurePrefs(this)
```

接下来，打开 **RecipeViewModel** 并将 `Prefs` 替换为 `SecurePrefs`。启动应用并确保保存首选项对已保存的搜索和当前选定的屏幕仍然有效。

要验证你的首选项是否已加密，请打开你创建的文件。为此，Android Studio 有 Device Explorer 标签。在屏幕上找到该标签或转到菜单 **View ▸ Tool Windows ▸ Device Explorer**。

你会看到类似这样的内容：

 ![](./Android Fundamentals by Tutorials, Chapter 11_Advanced Storage_ Kodeco_files/original(13).png)

注意，当前突出显示的路径是 **data/data**。这是应用存储所有信息的地方。打开这个文件夹并滚动到底部的应用：

 ![](./Android Fundamentals by Tutorials, Chapter 11_Advanced Storage_ Kodeco_files/original(14).png)

打开它。在 **shared\_prefs** 文件夹中，找到并打开 **encrypted\_preferences.xml** 文件。你应该会看到类似这样的内容：

 ![](./Android Fundamentals by Tutorials, Chapter 11_Advanced Storage_ Kodeco_files/original(15).png)

你理解其中的任何内容吗？不理解？这意味着它已加密。密钥和值都已加密。你现在拥有一组安全的首选项。

### 加密 Room

现在你已经加密了首选项，是时候加密数据库了。Room 没有加密库，但在底层它只是一个 SQLite 数据库。你可以使用 **SQLCipher** 库来加密数据，该库将自身包装在 SQLite 周围以加密数据。

### 添加 SQLCipher 库

打开 **gradle/libs.versions.toml**。在 **versions** 部分的末尾，添加：

```
sqlcipher = "4.4.0"
```

然后，在 **\[libraries\]** 部分的末尾，添加：

```
# Secure Room
sqlcipher = { module = "net.zetetic:android-database-sqlcipher", version.ref = "sqlcipher" }
```

执行 Gradle 同步。在 **app/build.gradle.kts** 依赖项部分，在安全库之后，添加：

```kotlin
implementation(libs.sqlcipher)
```

再次执行 Gradle 同步。

打开 **RecipeDatabase**。添加以下导入：

```kotlin
import net.sqlcipher.database.SQLiteDatabase
import net.sqlcipher.database.SupportFactory
import kotlin.random.Random
```

在 `companion object` 块中，添加：

```kotlin
const val PASSCODE_KEY = "PASSCODE_KEY"
```

这将是用于在数据库中存储密码的首选项键。

接下来，在 `private var INSTANCE: RecipeDatabase? = null` 之后，添加以下方法：

```kotlin
fun getPassCode(stringSize: Int): String {
  val randomString = StringBuilder()
  val random = Random.Default
  for (i in 0 until stringSize) {
    randomString.append('a' + random.nextInt(26))
  }
  return randomString.toString()
}
```

这个简单的方法创建一个用作密码的随机字符串。如果你在应用中硬编码密码会发生什么？黑客可以反编译你的应用并提取密码来解密你的数据库。使用随机字符串意味着字符串永远不会出现在代码中供黑客获取。你将使用安全首选项来存储每次使用的密码。

接下来，将 `getInstance` 方法更改为如下所示：

```kotlin
fun getInstance(context: Context, passCode: CharArray): RecipeDatabase {
```

这会将密码作为参数添加。接下来，在 `if (instance == null) {` 之后，添加：

```kotlin
val supportFactory = SupportFactory(SQLiteDatabase.getBytes(passCode))
```

SupportFactory 是 SQLCipher 的一部分，需要来自 `passCode` 的字节进行加密。

将数据库创建调用替换为以下内容：

```kotlin
instance = Room.databaseBuilder(
  context.applicationContext,
  RecipeDatabase::class.java,
  "recipe_database"
)
  .openHelperFactory(supportFactory)
  .fallbackToDestructiveMigration()
  .build()
```

这添加了 `openHelperFactory(supportFactory)` 调用，该调用使用 SQLCipher。

返回到 **MainActivity** 并替换创建存储库的代码：

```kotlin
repository = RecipeRepository(
  Room.databaseBuilder(
    this,
    RecipeDatabase::class.java,
    "Recipes"
  ).build()
)
```

使用以下内容：

```kotlin
val randomPassCode: String
// 1
if (!securePrefs.hasKey(PASSCODE_KEY)) {
  // 2
  randomPassCode = RecipeDatabase.getPassCode(15)
  // 3
  securePrefs.saveString(PASSCODE_KEY, randomPassCode)
} else {
  // 4
  randomPassCode = securePrefs.getString(PASSCODE_KEY)!!
}
// 5
repository = RecipeRepository(
  RecipeDatabase.getInstance(this, randomPassCode.toCharArray())
)
```

以下是该代码的作用：

1. 首先，它检查你是否已经存储了密码。
2. 如果没有，它会创建一个新的随机密码。
3. 将密码保存到共享首选项。
4. 如果已经存在，则检索它。
5. 使用数据库的实例创建 Repository。

重新构建并运行你的应用。尝试保存书签。退出应用并返回以确保它们仍然存在。你现在拥有一个安全的应用，需要付出很大努力才能破解它。

## 要点

+ 加密是安全存储数据的关键。
+ 你可以加密首选项和数据库。
+ 安全库提供对加密首选项类的访问。
+ SQLCipher 为 SQLite 数据库提供加密。

## 从这里去哪里？

在本章中，你学习了如何在首选项和数据库中存储加密数据。

要了解有关 Keystore 的更多信息，请访问：[https://developer.android.com/training/articles/keystore](https://developer.android.com/training/articles/keystore)。

有关 KeyChain 的更多信息，请访问：[https://developer.android.com/reference/android/security/KeyChain](https://developer.android.com/reference/android/security/KeyChain)。

要了解有关加密文件和首选项的更多信息，请访问：[https://developer.android.com/topic/security/data](https://developer.android.com/topic/security/data)。

要了解有关 SQLCipher 的更多信息，请访问：[https://www.zetetic.net/sqlcipher/sqlcipher-for-android/](https://www.zetetic.net/sqlcipher/sqlcipher-for-android/)。
