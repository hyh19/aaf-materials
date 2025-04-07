# 11. Advanced Storage

Sometimes, an app needs to store information to the files on a device. If these files are needed just by the app and are not to be used by the user, you can store them in the app’s files and cache directories. Only your app can access these directories, which are relatively secure (a hacker could get access, but not the average user). As the name implies, cache directory files are for cached items, and there’s limited storage for them. The Context object (which an Activity implements) can let you access them through the following calls (from any context):

## Files and Directories

```
Context.cacheDir
Context.filesDir
```

These File classes let you list, create and delete the directory files. Here’s an example of how to write a file to the directory:

```
val file = File(context.filesDir, "test.txt")
file.bufferedWriter().use { out ->  out.write("This is a test") }
```

This creates a file in the `context.filesDir` directory. If you were to put this in MainActivity and run your app, this file would be generated in the files directory.

### Device Explorer

Open the Device Explorer by going to **View ▸ Tool Windows ▸ Device Explorer**. You’ll find all the device’s app folders in the **data/data** folder:

 ![](./Android Fundamentals by Tutorials, Chapter 11_Advanced Storage_ Kodeco_files/original.png)

Scroll to the end to find **com.kodeco.recipefinder**. You’ll find the **test.txt** file here:

 ![](./Android Fundamentals by Tutorials, Chapter 11_Advanced Storage_ Kodeco_files/original(1).png)

Double-click the file to open it in Android Studio:

 ![](./Android Fundamentals by Tutorials, Chapter 11_Advanced Storage_ Kodeco_files/original(2).png)

If you want to find the size of your cache directory, try this code in your Activity:

```
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

Because the storage manager is a system service, you must retrieve this from the main thread. Once you get the service, call the `getCacheQuotaBytes()` method. It requires a UUID for the path and has a handy method to retrieve it: `getUuidForPath()`.

### Cache Files

To create a cache file, you simply call the `createTempFile()` method on the File class like:

```
val tempFile = File.createTempFile(fileName, null, context.cacheDir)
```

You would then access the file using the `tempFile` File instance pointing to it. Ensure that it still exists when trying to access it later because the system can delete cache files. To delete the file, call `tempFile.delete()`.

### External Files

In Android, there are a few well-defined directory names, such as:

+ Download
+ Pictures
+ Movies
+ Documents

These have constants defined for them in the **Environment** class, like:

+ DIRECTORY\_DOWNLOADS
+ DIRECTORY\_PICTURES
+ DIRECTORY\_MOVIES
+ DIRECTORY\_DOCUMENTS

To access these files, use either the `Context.getExternalFilesDir()` call or `Environment.getExternalStoragePublicDirectory()`. Each returns a File object.

To write a file to the Documents directory, you would use something like:

```
val documentFile = Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_DOCUMENTS)
val textFile = File(documentFile, "test.txt")
textFile.bufferedWriter().use { out ->  out.write("This is a test") }
```

This works well on an emulator, but you can’t just access any file on a real device. The app needs permission to access files the user has created. The easiest way to do that is to use a system file picker that requests permissions for you. If you receive permission, you can access the file. The system is called the **Storage Access Framework**.

### Storage Access Framework

The Storage Access Framework lets you use a system picker to have your user pick files for you to open, create or modify. This way, you don’t have to go through the permission system that forces the user to decide whether to grant your app permission to write to the requested directories.

### Creating Files

To create a file, use the `ACTION_CREATE_DOCUMENT` intent. This is an example function you can run to create a text file:

```
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

This starts a system picker in the download directory and saves a file named “test.txt”. Add this method and tie it to a button like the search button. Then, use Device Explorer and find the file in the Downloads folder.

> **Note**: Use the newer Activity Result Launchers to get the URI from the result. For details on using the launchers, see [this Android Developers guide](https://developer.android.com/training/basics/intents/result).

In addition to creating files, you can also open them with the `ACTION_OPEN_DOCUMENT` intent. To have the user give your app permission to a specific directory, use the `ACTION_OPEN_DOCUMENT_TREE` intent. You won’t be able to access the following directories:

+ Root level
+ Download
+ Android/data
+ Android/obb

Once you receive the URI the user selected, you can only use that URI until the user restarts their phone. To “keep” that access, you can request permanent access (unless the file gets moved or deleted). To do so, use the following example:

```
val contentResolver = applicationContext.contentResolver

val takeFlags: Int = Intent.FLAG_GRANT_READ_URI_PERMISSION or
    Intent.FLAG_GRANT_WRITE_URI_PERMISSION

contentResolver.takePersistableUriPermission(uri, takeFlags)
```

This causes the URI to be remembered across device reboots.

### Database Explorer

In the previous chapter, you created a SQLite database using Room. Android Studio has a Database Explorer that allows you to view your data easily. To reach the explorer, use the menu **View ▸ Tool Windows ▸ App Inspection**. This will show a view like:

 ![](./Android Fundamentals by Tutorials, Chapter 11_Advanced Storage_ Kodeco_files/original(3).png)

Here, you see your Recipe Database with the ingredients, recipes and a special table called room\_master\_table. Double-click the recipes table to see the data currently stored in the table:

 ![](./Android Fundamentals by Tutorials, Chapter 11_Advanced Storage_ Kodeco_files/original(4).png)

Now, open the **RecipeDao** file. You’ll notice several database icons in the left gutter:

 ![](./Android Fundamentals by Tutorials, Chapter 11_Advanced Storage_ Kodeco_files/original(5).png)

Run these queries by clicking that icon. This brings up a dialog like:

 ![](./Android Fundamentals by Tutorials, Chapter 11_Advanced Storage_ Kodeco_files/original(6).png)

Here, the ID of a recipe appears. Click “Run” to see something like:

 ![](./Android Fundamentals by Tutorials, Chapter 11_Advanced Storage_ Kodeco_files/original(7).png)

You can also run a SQL query by typing it in and clicking the new query button:

 ![](./Android Fundamentals by Tutorials, Chapter 11_Advanced Storage_ Kodeco_files/original(8).png)

After the new tab appears, you can run a query by typing in the SQL command and clicking “Run”:

 ![](./Android Fundamentals by Tutorials, Chapter 11_Advanced Storage_ Kodeco_files/original(9).png)

Once you’ve run enough queries, you can access them from the history button:

 ![](./Android Fundamentals by Tutorials, Chapter 11_Advanced Storage_ Kodeco_files/original(10).png)

You can run the inspector even if your app crashes in offline mode. You can’t make changes but can view the currently cached data. If you want to export your data, click on the “Export as File” button:

 ![](./Android Fundamentals by Tutorials, Chapter 11_Advanced Storage_ Kodeco_files/original(11).png)

Then, select the file type:

+ **DB**: SQLite Database
+ **SQL**: SQL statements
+ **CSV**: Comma-separated values

 ![](./Android Fundamentals by Tutorials, Chapter 11_Advanced Storage_ Kodeco_files/original(12).png)

## Security

So far, you’ve learned how to store data in a SQLite database and a Data Store. Remember, Data Store is a newer API for Android’s Shared Preferences. To create a secure Shared Preference file, you must use the Encrypted Preferences package that’s part of the Android Security library. This library hasn’t been converted to the newer Data Store format yet, so you’ll learn how to use it in its Shared Preferences format.

### Android Keystore

The **Android Keystore** system is a container for cryptographic keys and makes it very difficult to extract. A **Trusted Execution Environment (TEE)** is an area separate from the main operating system. The data is even safer if the phone has secure hardware (**Secure Element (SE)**) with its own CPU and storage. To check for this feature, use `KeyInfo.isInsideSecurityHardware()` on API level 28 or lower, or `KeyInfo.getSecurityLevel()` on API level 29 or higher. These secure hardware components contain the following:

+ Separate CPU
+ Secure Storage
+ True Random Number generator
+ Mechanisms to resist package tampering and unauthorized sideloading of apps
+ Secure timer
+ Reboot notification pin

To get an instance of the Android Keystore, you would write this:

```
val keystore: KeyStore = KeyStore.getInstance("AndroidKeyStore").apply {
  load(null)
}
```

You get items from the Keystore with the `getKey()` method. This takes an alias string and returns an entry (usually a `SecretKeyEntry`). You also can create private and secret keys using the `KeyPairGenerator` class.

There’s also the **KeyChain** API for system-wide credentials. This class has some of the following methods:

+ **`createManageCredentialsIntent`**: Used to start an app to request to manage the user’s credentials.
+ **`choosePrivateKeyAlias`**: Used to start an app to select the alias for a private key and certificate.
+ **`getCertificateChain`**: Returns the X509Certificate chain for the requested alias.
+ **`getPrivateKey`**: Returns the PrivateKey for the requested alias.

### Encrypted Preferences

The Encrypted Preferences class is actually in the **security-crypto** library. This library contains both **EncryptedSharedPreferences** and **EncryptedFile** (used to create encrypted files). There’s also a convenient **MasterKeys** class with methods for creating and obtaining master keys from the Android Keystore.

### Adding the Security Library

If you’re following along with your app from the previous chapters, open it and keep using it with this chapter. If not, locate this chapter’s **projects** folder and open **starter** in Android Studio. Open **gradle/libs.versions.toml**. At the end of the **versions** section, add:

```
security = "1.0.0"
```

Then, at the end of the **\[libraries\]** section, add:

```
# Security
security = { module = "androidx.security:security-crypto", version.ref = "security" }
```

Do a Gradle sync. In the **app/build.gradle.kts** dependencies section, add:

```
implementation(libs.security)
```

After the timber library. Do another Gradle sync.

### Secure Preferences

You’ll use the Encrypted Preferences class to save data in an encrypted format.

Start by going to the **app/src/main/java/com/kodeco/recipefinder/data** directory and creating a Kotlin file named **SecurePrefs.kt**. Add the following:

```
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

This uses Android’s `SharedPreferences` class. The difference is that you’ll be creating an encrypted version. Replace `// TODO: Add init` with:

```
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

Here’s a code breakdown:

2. Create or retrieve a MasterKey using the `AES256_GCM_SPEC` (a bunch of settings specifying an encryption/decryption key with different settings).
4. Use the `EncryptedSharedPreferences` class to create a SharedPreference class.
6. Give any name you want for the file name.

These are all good settings, but you can change them. The complexity of the security schemes is beyond the scope of this book. See the “Where to Go From Here?” section below for more information.

Like in the previous chapter, add the methods for saving and returning values. Replace `// TODO: Add get/save methods` with:

```
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

Notice the `edit()` and `apply()` methods. This is the old style (pre-Data Store), where you first would have to get an editor class and, when finished, use the `apply()` method.

Now that you’ve written this class, replace the older `Prefs` class with it. Open **MainActivity** and change:

```
val LocalPrefsProvider =
  compositionLocalOf<Prefs> { error("No prefs provided") }
```

To:

```
val LocalPrefsProvider =
  compositionLocalOf<SecurePrefs> { error("No prefs provided") }
```

Next, change the `prefs` type in `RecipeApp.kt` from:

```
lateinit var prefs: Prefs
```

To:

```
lateinit var securePrefs: SecurePrefs
```

Make sure to add the `SecurePrefs` import and rename all instances of `prefs` to `securePrefs`. Do this by right-clicking the original variable name and choosing **Refactor** in the context menu followed by **Rename…**.

Now, change the instantiation line inside the `RecipeApp`’s `onCreate()` from:

```
prefs = Prefs(this)
```

To:

```
prefs = SecurePrefs(this)
```

Next, open **RecipeViewModel** and replace `Prefs` with `SecurePrefs`. Start the app and ensure that saving preferences still work for the saved searches and the currently selected screen.

To verify whether your preferences are encrypted, open the file you created. To do that, Android Studio has the Device Explorer tab. Find the tab on the screen or go to the menu **View ▸ Tool Windows ▸ Device Explorer**.

You’ll see something like:

 ![](./Android Fundamentals by Tutorials, Chapter 11_Advanced Storage_ Kodeco_files/original(13).png)

Notice that the current path highlighted is **data/data**. This is where the app stores all its information. Open this folder and scroll to your app at the bottom:

 ![](./Android Fundamentals by Tutorials, Chapter 11_Advanced Storage_ Kodeco_files/original(14).png)

Open this. In the **shared\_prefs** folder, find and open the **encrypted\_preferences.xml** file. You should see something like:

 ![](./Android Fundamentals by Tutorials, Chapter 11_Advanced Storage_ Kodeco_files/original(15).png)

Do you understand any of it? No? That means it’s encrypted. Both the key and the value are encrypted. You now have a secure set of preferences.

### Encrypted Room

Now that you’ve encrypted your preferences, it’s time to encrypt your database. Room doesn’t have an encryption library, but underneath it is just a SQLite database. You can encrypt this using the **SQLCipher** library, which wraps itself around SQLite to encrypt the data.

### Adding the SQLCipher Library

Open **gradle/libs.versions.toml**. At the end of the **versions** section, add:

```
sqlcipher = "4.4.0"
```

Then, at the end of the **\[libraries\]** section, add:

```
# Secure Room
sqlcipher = { module = "net.zetetic:android-database-sqlcipher", version.ref = "sqlcipher" }
```

Do a Gradle sync. In the **app/build.gradle.kts** dependencies section, after the security library, add:

```
implementation(libs.sqlcipher)
```

Do another Gradle sync.

Open **RecipeDatabase**. Add the following imports:

```
import net.sqlcipher.database.SQLiteDatabase
import net.sqlcipher.database.SupportFactory
import kotlin.random.Random
```

In the `companion object` block, add:

```
const val PASSCODE_KEY = "PASSCODE_KEY"
```

This will be a preference key for storing a passcode in the database.

Next, after `private var INSTANCE: RecipeDatabase? = null`, add the following method:

```
fun getPassCode(stringSize: Int): String {
  val randomString = StringBuilder()
  val random = Random.Default
  for (i in 0 until stringSize) {
    randomString.append('a' + random.nextInt(26))
  }
  return randomString.toString()
}
```

This simple method creates a random string used for a passcode. What would happen if you were to hard-code a passcode into the app? A hacker could decompile your app and pull the passcode to decrypt your database. Using a random string means the string never appears in the code for hackers to get. You’ll use secure preferences to store that passcode to use each time.

Next, change the `getInstance` method to look like:

```
fun getInstance(context: Context, passCode: CharArray): RecipeDatabase {
```

This would add the passcode as a parameter. Next, after `if (instance == null) {`, add:

```
val supportFactory = SupportFactory(SQLiteDatabase.getBytes(passCode))
```

SupportFactory is part of SQLCipher and needs the bytes from `passCode` for encryption.

Replace the database creation call with the following:

```
instance = Room.databaseBuilder(
  context.applicationContext,
  RecipeDatabase::class.java,
  "recipe_database"
)
  .openHelperFactory(supportFactory)
  .fallbackToDestructiveMigration()
  .build()
```

This adds the `openHelperFactory(supportFactory)` call, which uses SQLCipher.

Return to **MainActivity** and replace the code that creates the repository:

```
repository = RecipeRepository(
  Room.databaseBuilder(
    this,
    RecipeDatabase::class.java,
    "Recipes"
  ).build()
)
```

With the following:

```
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

Here’s what that code does:

2. First, it checks to see if you have the passcode already stored.
4. If not, it creates a new random passcode.
6. Saves the passcode to shared preferences.
8. It already exists, so it retrieves it.
10. Creates the Repository with an instance of the database.

Rebuild and run your app. Try to save bookmarks. Exit the app and come back to ensure they still exist. You now have a secure app that takes a lot of effort to break.

## Key Points

+ Encryption is key to securely storing your data.
+ You can encrypt both preferences and databases.
+ The security library provides access to the Encrypted Preferences class.
+ SQLCipher provides encryption for SQLite databases.

## Where to Go From Here?

In this chapter, you learned how to store encrypted data in preferences and a database.

To learn more about Keystore, go to: [https://developer.android.com/training/articles/keystore](https://developer.android.com/training/articles/keystore).

For more information on KeyChain, go to: [https://developer.android.com/reference/android/security/KeyChain](https://developer.android.com/reference/android/security/KeyChain).

To learn more about Encrypted Files and Preferences, go to: [https://developer.android.com/topic/security/data](https://developer.android.com/topic/security/data).

To learn more about SQLCipher, go to: [https://www.zetetic.net/sqlcipher/sqlcipher-for-android/](https://www.zetetic.net/sqlcipher/sqlcipher-for-android/).
