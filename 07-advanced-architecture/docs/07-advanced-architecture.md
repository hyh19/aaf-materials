# 7. Advanced Architecture

In the last chapter, you improved the chat app by implementing ViewModel and Flows and making some UI improvements. In this chapter, you’ll learn about another important architecture pattern in mobile development: the repository pattern. You’ll also learn how to use a revolutionary new P2P mesh technology via the Ditto SDK to finish the Kodeco Chat app and enable communication between devices without cloud service.

## Ditto SDK

Now, you’re able to add real chat messages in your app. But what’s the use of a chat app if only one device or person can chat? That’s only a conversation if you enjoy talking to yourself!

So you need a way for devices or people to chat with each other. This is where the Ditto SDK is handy. Ditto is a cross-platform P2P SDK that lets two devices communicate using any type of wireless transport — Bluetooth, Wi-Fi, etc. — without an internet connection or cloud back end.

### Registering Your App With Ditto

You’ll need to create a developer account with Ditto and build a personal app on its website. This gives you access to the authentication keys needed for the SDK. In a web browser, navigate to [Ditto.live](https://ditto.live/) and click the **Get Started** button. Then, follow the steps to create an account. The simplest way is to sign up with your existing Google account. Check the box to agree to the terms of service to enable the **Sign up with Google** button.

 ![](./Android Fundamentals by Tutorials, Chapter 7_Advanced Architecture_ Kodeco_files/original.png)

Complete the steps to create a new app on the Ditto website. Once you do that, you’ll be able to go to the web page for your app to get the unique SDK keys you’ll need for your chat app.

Once you’ve created an account, you should see a prompt to create an app on Ditto’s website.

 ![](./Android Fundamentals by Tutorials, Chapter 7_Advanced Architecture_ Kodeco_files/original(1).png)

Enter a name for the app, and the URL should be automatically filled out. Typically, you’ll want to use the same or a similar name to your Android app, but it doesn’t matter. Click the **Create App** button, and the site takes you to **Step 1** of the Quickstart page.

 ![](./Android Fundamentals by Tutorials, Chapter 7_Advanced Architecture_ Kodeco_files/original(2).png)

Select Android for the framework. The site will take you to **Step 2**, which states **Install Ditto**. The next section covers how to do that. Click the **Next step** button; it shows some sample code. Don’t copy it, but note that it shows an `appID` and a `token` with the same values as the **App ID** and **Playground Token** displayed at the top of the page. You’ll need these keys later.

### Setting Up Ditto SDK in Your App

You need to add the Gradle dependency for Ditto in your app, so open the starter project of this chapter. Open **libs.versions.toml** and, in the `[versions]` section, add:

```
ditto = "4.5.0"
```

In the `[libraries]` section, add:

```
ditto = { module = "live.ditto:ditto", version.ref = "ditto" }
```

In your module-level Gradle file, add the following dependency:

```
implementation(libs.ditto)
```

You’ll want to use the latest version of the Ditto SDK for Android. Check [this page](https://docs.ditto.live/kotlin-release-s-4xx) to see what the latest version is; at the time of this writing, it’s 4.5.0.

Sync your project with the Gradle files. Build and run your app to make sure everything still works as before.

Because it uses BLE and other wireless protocols to sync data across devices, Ditto requires certain permissions. The Ditto SDK conveniently already adds the permissions it needs in the manifest, so you don’t need to add them in your **AndroidManifest.xml** file. But at runtime, you also need to prompt the user to grant permissions. The Ditto SDK provides a `DittoSyncPermissions` helper, which makes this easy.

Open **MainActivity.kt** and add the following function to `MainActivity`:

```
private fun checkPermissions() {
  val missing = DittoSyncPermissions(this).missingPermissions()
  if (missing.isNotEmpty()) {
    this.requestPermissions(missing, 0)
  }
}
```

You’ll need to import this:

```
import live.ditto.transports.DittoSyncPermissions
```

Then, update `onCreate()` by calling `checkPermissions()` at the end of the function. Build and run.

 ![](./Android Fundamentals by Tutorials, Chapter 7_Advanced Architecture_ Kodeco_files/original(3).png)

When the app launches, it should prompt you with a permissions popup screen after it creates the UI. Tap **Allow** to enable the permissions.

The last thing you need to do is instantiate a singleton instance of Ditto in your app. To use the Ditto SDK in your app, you need the `appID` and the playground `token` you saw before on the Ditto website. These are your secret keys, and you should never expose them publicly.

If you followed along in Chapter 4, “Gradle Basics: A Look Behind the Curtain”, you already created a **keys.properties** file. If not, refer back to that chapter for instructions on how to create it and update your module-level Gradle file to reference that file by adding the following between the `plugins { }` and `android { }` blocks:

```
val keysPropertiesFile: File = rootProject.file("keys.properties")
val keysProperties = Properties()
keysProperties.load(FileInputStream(keysPropertiesFile))
```

Add the following values in your **keys.properties** file, replacing the values for each variable with the values for your `appID` and `token`:

```
DITTO_APP_ID = "replace with your app ID"
DITTO_TOKEN = "replace with your token"
```

Then, back in your module-level Gradle file, in the `buildTypes { }` block, add:

```
debug {
  buildConfigField("String", "DITTO_APP_ID", keysProperties["DITTO_APP_ID"] as String)
  buildConfigField("String", "DITTO_TOKEN", keysProperties["DITTO_TOKEN"] as String)
}
```

Add the same properties at the end of your `release { }` build type.

In the `buildFeatures { }` block, add:

```
buildConfig = true
```

> **Note**: After adding these values, you might need to do a project clean and build so **BuildConfig.java** gets autogenerated with them.

Copy **DittoHandler.kt** from the final project into the same location in your project.

Finally, back in **MainActivity.kt**, add this function:

```
private fun setupDitto() {
  val androidDependencies = DefaultAndroidDittoDependencies(applicationContext)
  DittoLogger.minimumLogLevel = DittoLogLevel.DEBUG
  ditto = Ditto(
    androidDependencies,
      DittoIdentity.OnlinePlayground(
        androidDependencies,
        appId = BuildConfig.DITTO_APP_ID,
        token = BuildConfig.DITTO_TOKEN
      )
  )
  ditto.startSync()
}
```

You’ll need to import these:

```
import com.kodeco.chat.DittoHandler.Companion.ditto
import live.ditto.Ditto
import live.ditto.DittoIdentity
import live.ditto.DittoLogLevel
import live.ditto.DittoLogger
import live.ditto.android.DefaultAndroidDittoDependencies
```

Then, update `onCreate()` to call `setupDitto()` at the end, after the call to `checkPermissions()`. Build and run. If you’ve correctly added the app ID and token, your app should run as before. If not, it’ll crash on launch, and your logcat will show an error indicating the license wasn’t found.

Great, now you have the Ditto SDK installed and running in your app. Next, you must leverage it in your code to pass the chat messages, user information and any other data you want between devices. Now, you’ll learn about another design pattern, which allows you to do that in a clean and scalable way: the **repository pattern**.

## Repository Pattern

Sometimes, you want to store and fetch data from a local database in your app. Other times, you might want to fetch data from the internet or a cloud service using a networking API. And sometimes, you want to fetch data from the network. But if the network isn’t available, you’ll want to fetch the same data from a local cache you created the last time the network was available. In other words, you might want to combine both network and local data.

Regardless of where you get data from, the ViewModel should request the data and the UI should listen for changes in the data and update through MVI and Flows via the ViewModel. But it’s better not to inundate your ViewModel class with all the implementation details of *how* to get the data.

Ideally, your ViewModel will be completely ignorant of *where* the data is coming from. Rather, it’ll say, “Give me data!” via a Flow, and your Compose UI will listen for changes in those flows and recompose as needed.

There’s one more layer of abstraction regarding the details of *where* and *how* the data is coming. That layer is the **repository**. The repository pattern provides a way for your app to organize multiple data sources, serve as a single source of truth for the app’s data and abstract the data source (network, cache, etc.) out of the ViewModel.

Repository classes are responsible for the following tasks:

+ Exposing data to the rest of the app.
+ Centralizing changes to the data.
+ Resolving conflicts between data sources.
+ Abstracting sources of data from the rest of the app.
+ Containing business logic related to data transformation.

A typical way to implement the repository pattern in Android apps is to create an interface that defines all the methods you’ll call from your ViewModel, followed by an implementation class that implements the details of how to fulfill those methods. You might have multiple repository classes; for example, you might have one for accessing a network API and another for dealing with Data Store.

Open **Constants.kt** in the final project for this chapter and copy and paste all the values from there into the same class in your app. This provides key strings as a convenience. It’s a common convention to avoid developer errors from manually typing the same key strings in different places in an app and potentially mistyping something.

Create a new package under **com.kodeco.chat ▸ data** and name it **repository**.

Copy **Repository.kt** and **RepositoryImpl.kt** from the **repository** package in the final project and paste them into the same location in your project. **Repository.kt** is the interface and **RepositoryImpl.kt** is its implementation. Open **Repository.kt** and look at the methods defined there. In particular, note there’s `getAllUsers()` to retrieve all the users participating in this app’s Ditto mesh. Ultimately, this flow gets populated by this function from **RepositoryImpl.kt**:

```
private fun getAllUsersFromDitto() {
  ditto.let { ditto: Ditto ->
    // 1
    usersCollection = ditto.store.collection(usersKey)
    // 2
    usersSubscription = usersCollection.findAll().subscribe()
    usersLiveQuery = usersCollection.findAll().observeLocal { docs, _ ->
      this.usersDocs = docs
      // 3
      allUsers.value = docs.map { User(it) }
    }
  }
}
```

The repository provides the implementation details for how to fetch and send data to the Ditto P2P mesh. You:

2. Fetch a collection of all the users.
4. *Subscribe* to this collection to get notified of changes to the collection, such as a new user joining your app’s mesh.
6. Map this subscription to the `allUsers: MutableStateFlow` property, which `getAllUsers()` retrieves.

`getAllMessagesForRoom()` is similarly populated from Ditto.

`createMessageForRoom()` is similar but uses the Ditto function `upsert` to insert a chat message as a Ditto `document` into the message collection for a room.

You no longer need **FakeData.kt**, so safe-delete this class by right-clicking and choosing **Refactor ▸ Safe Delete** from the context menu.

In **MainViewModel.kt**, delete the import for `import com.kodeco.chat.data.initialMessages`. Then, update the class as follows:

```
class MainViewModel : ViewModel() {
  // 1
  private val userId = UUID.randomUUID().toString()
  var currentUserId = MutableStateFlow(userId)
  private var firstName: String = ""
  private var lastName: String = ""
  // 2
  private val repository = RepositoryImpl.getInstance()
  private val emptyChatRoom = ChatRoom(
    id = "public",
    name = "Android Apprentice",
    createdOn = Clock.System.now(),
    messagesCollectionId = DEFAULT_PUBLIC_ROOM_MESSAGES_COLLECTION_ID,
    isPrivate = false,
    collectionID = "public",
    createdBy = "Kodeco User"
  )

  private val _currentChatRoom = MutableStateFlow(emptyChatRoom)
  val currentRoom = _currentChatRoom.asStateFlow()

  // 3
  val roomMessagesWithUsersFlow: Flow<List<MessageUiModel>> = combine(
    repository.getAllUsers(),
    repository.getAllMessagesForRoom(currentRoom.value)
  ) { users: List<User>, messages:List<Message> ->

    messages.map {
      MessageUiModel.invoke(
        message = it,
        users = users
      )
    }
  }
  // 4
  init {
      // user initialization - we use the device name for the user's name
      val firstName = "My"
      val lastName = android.os.Build.MODEL
      updateUserInfo(firstName, lastName)
  }

  fun updateUserInfo(firstName: String = this.firstName, lastName: String = this.lastName) {
    viewModelScope.launch {
      repository.saveCurrentUser(userId, firstName, lastName)
    }
  }
  // 5
  fun onCreateNewMessageClick(messageText: String, photoUri: Uri?, attachmentToken: DittoAttachmentToken?) {
    val currentMoment: Instant = Clock.System.now()
    val message = Message(
      UUID.randomUUID().toString(),
      currentMoment,
      currentRoom.value.id,
      messageText,
      userId,
      attachmentToken,
      photoUri
    )

    if (message.photoUri == null) {
      viewModelScope.launch(Dispatchers.Default) {
        repository.createMessageForRoom(userId, message, currentRoom.value, null)
      }
    }
  }
}
```

2. You hard-code the `userId` here. Ideally, you’d generate this on the first app launch and store it for future use. See Chapter 9, “Data Store”, to see how you’d store a value like this in Data Store.
4. Obtain a singleton instance of the repository.
6. This flow is where you store messages for a particular chat room. Note that this flow is a *combination* of two flows: one flow of all the users and a second of all the messages in a chat room. Any update to either flow also updates the combined flow, which in turn triggers updates to any composables listening for updates in the flow.
8. As a convenience, the user name is set to the device name, so it’s clear where each message comes from when you chat between devices. You can add functionality later to let the user set their name.
10. The function to create a new message now uses the repository to handle posting messages to the Ditto P2P mesh instead of updating a message list that’s only local to one device.

You need to import the following:

```
import com.kodeco.chat.data.repository.RepositoryImpl
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.combine
import live.ditto.DittoAttachmentToken
```

Open **MainActivity.kt**. Update the definition of `messagesWithUsers` as follows:

```
val messagesWithUsers: List<MessageUiModel> by viewModel
  .roomMessagesWithUsersFlow
  .collectAsStateWithLifecycle(initialValue = emptyList())
```

You now retrieve the new message list flow you created in the ViewModel, passing that into the `ConversationContent` composable. The repository updates this flow using the Ditto SDK, the composable listens for changes to the flow and the UI automatically updates.

Also, update `currentUiState`:

```
val currentUiState =
  ConversationUiState(
    channelName = "Android Apprentice",
    initialMessages = messagesWithUsers.asReversed(),
    viewModel = viewModel
)
```

The only change here is adding `.asReversed()` to ensure messages added to the flow display in the right order, because the list displays in reverse in the UI as well.

Next, you need to update your data classes. Open **ConversationUiState.kt** and update it as follows:

```
class ConversationUiState(
  val channelName: String,
  initialMessages: List<MessageUiModel>,
  val viewModel: MainViewModel
) {
  private val _messages: MutableList<MessageUiModel> = initialMessages.toMutableStateList()

  val messages: List<MessageUiModel> = _messages

  //author ID is set to the user ID - it's used to tell if the message is sent from this user (self) when rendering the UI
  val authorId: MutableStateFlow<String> = viewModel.currentUserId

  fun addMessage(msg: String, photoUri: Uri?) {
    viewModel.onCreateNewMessageClick(msg, photoUri, null)
  }
}

@Immutable
data class Message(
  val _id: String = UUID.randomUUID().toString(),
  val createdOn: Instant? = Clock.System.now(),
  val roomId: String = "public", // "public" is the roomID for the default public chat room
  val text: String = "test",
  val userId: String = UUID.randomUUID().toString(),
  val attachmentToken: DittoAttachmentToken?,
  val photoUri: Uri? = null,
  val authorImage: Int = if (userId == "me") R.drawable.profile_photo_android_developer else R.drawable.someone_else
){
  constructor(document: DittoDocument) : this(
    document[dbIdKey].stringValue,
    document[createdOnKey].stringValue.toInstant(),
    document[roomIdKey].stringValue,
    document[textKey].stringValue,
    document[userIdKey].stringValue,
    document[thumbnailKey].attachmentToken
  )
}
```

The main change here is the addition of the `constructor`, which maps a `Message` to a Ditto document type.

Import these:

```
import com.kodeco.chat.data.createdOnKey
import com.kodeco.chat.data.dbIdKey
import com.kodeco.chat.data.model.toInstant
import com.kodeco.chat.data.roomIdKey
import com.kodeco.chat.data.textKey
import com.kodeco.chat.data.thumbnailKey
import com.kodeco.chat.data.userIdKey
import live.ditto.DittoAttachmentToken
import live.ditto.DittoDocument
```

Similarly, add the following `constructor` to **ChatRoom.kt**:

```
constructor(document: DittoDocument) : this(
  document[dbIdKey].stringValue,
  document[nameKey].stringValue,
  document[createdOnKey].stringValue.toInstant(),
  document[messagesIdKey].stringValue,
  document[isPrivateKey].booleanValue,
  document[collectionIdKey].stringValue,
  document[createdByKey].stringValue,
)
```

Import the following:

```
import com.kodeco.chat.data.collectionIdKey
import com.kodeco.chat.data.createdByKey
import com.kodeco.chat.data.createdOnKey
import com.kodeco.chat.data.dbIdKey
import com.kodeco.chat.data.isPrivateKey
import com.kodeco.chat.data.messagesIdKey
import com.kodeco.chat.data.nameKey
import live.ditto.DittoDocument
```

Add the following `constructor` to **User.kt**:

```
constructor(document: DittoDocument) : this(
  document[dbIdKey].stringValue,
  document[firstNameKey].stringValue,
  document[lastNameKey].stringValue
)
```

You’ll need to import these:

```
import com.kodeco.chat.data.dbIdKey
import com.kodeco.chat.data.firstNameKey
import com.kodeco.chat.data.lastNameKey
import live.ditto.DittoDocument
```

That’s it!

Build and run. Type in some chat messages. Now, for the mind-blowing part! Build and run on a *second* device or emulator. The chat messages you typed on the first device appear on the second. Type some messages on the second device…and they instantly appear across *all* the devices you’re running the app on.

 ![](./Android Fundamentals by Tutorials, Chapter 7_Advanced Architecture_ Kodeco_files/original(4).png)

Try it with as many devices as you like.

Congratulations! You’ve built a completely functional P2P chat app!

## Key Points

Well done! In this chapter, you’ve covered so much ground and created a real-world, P2P Android chat app! To recap, you learned:

+ How to further abstract your app logic, making your app back-end agnostic using the repository pattern.
+ How to create a P2P, cloud-optional chat app using the Ditto SDK!

## Where to Go From Here?

For a more in-depth look at app architecture and design patterns, check out the book [*Advanced Android App Architecture*](https://www.kodeco.com/books/advanced-android-app-architecture).

For more details on the Ditto SDK, see the Ditto [developer documentation](https://docs.ditto.live/).

In this chapter, there was some stub code for handling `DittoAttachment`, but you didn’t implement attaching an image to a chat message. If you’d like to challenge yourself a bit more with Ditto, try implementing image attachments in your chat app. You also only implemented a single chat room in this chapter; for yet another challenge, try implementing multiple chat rooms.

Finally, you might want to cache things like the user’s ID locally in your app so the app can “remember” it between launches. In Chapter 9, “Data Store”, you’ll learn how to store small amounts of data locally using **Data Store**. In Chapter 10, “Room Database”, you’ll learn how to store all kinds of data locally in your app, using **Room**.
