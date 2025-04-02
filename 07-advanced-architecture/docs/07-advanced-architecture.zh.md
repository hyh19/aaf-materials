# 7. 高级架构

在上一章中，你通过实现 ViewModel 和 Flows 并进行一些 UI 改进来完善聊天应用。在本章中，你将学习移动开发中另一个重要的架构模式：仓库模式（repository pattern）。你还将学习如何通过 Ditto SDK 使用革命性的新型 P2P 网格技术来完成 Kodeco Chat 应用，并实现设备之间无需云服务的通信。

## Ditto SDK

现在，你已经能够在应用中添加真实的聊天消息。但如果只有一台设备或一个人可以聊天，聊天应用有什么用呢？那只是在自言自语！

因此，你需要一种让设备或人们相互聊天的方式。这就是 Ditto SDK 的用武之地。Ditto 是一个跨平台的 P2P SDK，它允许两台设备使用任何类型的无线传输方式（蓝牙、Wi-Fi 等）进行通信，无需互联网连接或云后端。

### 在 Ditto 注册你的应用

你需要在 Ditto 创建一个开发者账户，并在其网站上构建一个个人应用。这样可以获取 SDK 所需的认证密钥。在网络浏览器中，导航到 [Ditto.live](https://ditto.live/)，然后点击 **Get Started** 按钮。接着，按照步骤创建一个账户。最简单的方法是使用你现有的 Google 账户注册。勾选同意服务条款的复选框，以启用 **Sign up with Google** 按钮。

![1743589760270](https://cdn.jsdelivr.net/gh/hyh19/images3@master/1743589758039.png)

在 Ditto 网站上完成创建新应用的步骤。完成后，你将能够访问应用的网页，获取聊天应用所需的唯一 SDK 密钥。

创建账户后，你应该会看到在 Ditto 网站上创建应用的提示。

<!-- ![1743589817206](https://cdn.jsdelivr.net/gh/hyh19/images3@master/1743589815291.png) -->
![1743592962209](https://cdn.jsdelivr.net/gh/hyh19/images3@master/1743592960517.png)

输入应用的名称，URL 应该会自动填充。通常，你会想使用与 Android 应用相同或类似的名称，但这并不重要。点击 **Create App** 按钮，网站会将你带到快速入门页面的 **Step 1**。

<!-- ![1743589837872](https://cdn.jsdelivr.net/gh/hyh19/images3@master/1743589835913.png) -->
![1743592994078](https://cdn.jsdelivr.net/gh/hyh19/images3@master/1743592991706.png)

选择 Android 作为框架。网站将带你到 **Step 2**，上面写着 **Install Ditto**。下一节将介绍如何执行此操作。点击 **Next step** 按钮；它会显示一些示例代码。不要复制它，但请注意其中显示的 `appID` 和 `token` 值与页面顶部显示的 **App ID** 和 **Playground Token** 相同。稍后你将需要这些密钥。

### 在应用中设置 Ditto SDK

你需要在应用中添加 Ditto 的 Gradle 依赖项，因此打开本章的起始项目。打开 **libs.versions.toml**，并在 `[versions]` 部分添加：

```kotlin
ditto = "4.5.0"
```

在 `[libraries]` 部分，添加：

```kotlin
ditto = { module = "live.ditto:ditto", version.ref = "ditto" }
```

在模块级 Gradle 文件中，添加以下依赖项：

```kotlin
implementation(libs.ditto)
```

你会希望使用最新版本的 Android Ditto SDK。查看[此页面](https://docs.ditto.live/kotlin-release-s-4xx)以了解最新版本；在撰写本文时，版本为 4.5.0。

使用 Gradle 文件同步项目。构建并运行应用，确保一切仍如之前一样正常工作。

由于 Ditto 使用 BLE 和其他无线协议在设备间同步数据，它需要特定权限。Ditto SDK 已经方便地在清单中添加了所需的权限，因此你不需要在 **AndroidManifest.xml** 文件中添加它们。但在运行时，你还需要提示用户授予权限。Ditto SDK 提供了一个 `DittoSyncPermissions` 辅助工具，使这变得简单。

打开 **MainActivity.kt** 并将以下函数添加到 `MainActivity` 中：

```kotlin
private fun checkPermissions() {
  val missing = DittoSyncPermissions(this).missingPermissions()
  if (missing.isNotEmpty()) {
    this.requestPermissions(missing, 0)
  }
}
```

你需要导入以下内容：

```kotlin
import live.ditto.transports.DittoSyncPermissions
```

然后，通过在函数末尾调用 `checkPermissions()` 来更新 `onCreate()`。构建并运行。

![1743589857245](https://cdn.jsdelivr.net/gh/hyh19/images3@master/1743589855154.png)

当应用启动时，在创建 UI 后，它应该会显示权限弹出屏幕。点击 **Allow** 以启用权限。

你需要做的最后一件事是在应用中实例化 Ditto 的单例实例。要在应用中使用 Ditto SDK，你需要之前在 Ditto 网站上看到的 `appID` 和 playground `token`。这些是你的密钥，你永远不应该公开暴露它们。

如果你在第 4 章"Gradle 基础：揭开幕后"中按步骤操作，你已经创建了 **keys.properties** 文件。如果没有，请参考该章节了解如何创建此文件并更新模块级 Gradle 文件以引用该文件，方法是在 `plugins { }` 和 `android { }` 块之间添加以下内容：

```kotlin
val keysPropertiesFile: File = rootProject.file("keys.properties")
val keysProperties = Properties()
keysProperties.load(FileInputStream(keysPropertiesFile))
```

在 **keys.properties** 文件中添加以下值，将每个变量的值替换为你的 `appID` 和 `token` 的值：

```properties
DITTO_APP_ID = "replace with your app ID"
DITTO_TOKEN = "replace with your token"
```

然后，回到你的模块级 Gradle 文件，在 `buildTypes { }` 块中，添加：

```kotlin
debug {
  buildConfigField("String", "DITTO_APP_ID", keysProperties["DITTO_APP_ID"] as String)
  buildConfigField("String", "DITTO_TOKEN", keysProperties["DITTO_TOKEN"] as String)
}
```

在 `release { }` 构建类型的末尾添加相同的属性。

在 `buildFeatures { }` 块中，添加：

```kotlin
buildConfig = true
```

> **注意**：添加这些值后，你可能需要进行项目清理和构建，以便 **BuildConfig.java** 自动生成这些值。

从最终项目中复制 **DittoHandler.kt** 到你项目中的相同位置。

最后，回到 **MainActivity.kt**，添加此函数：

```kotlin
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

你需要导入以下内容：

```kotlin
import com.kodeco.chat.DittoHandler.Companion.ditto
import live.ditto.Ditto
import live.ditto.DittoIdentity
import live.ditto.DittoLogLevel
import live.ditto.DittoLogger
import live.ditto.android.DefaultAndroidDittoDependencies
```

然后，更新 `onCreate()` 以在 `checkPermissions()` 调用之后的末尾调用 `setupDitto()`。构建并运行。如果你正确添加了应用 ID 和令牌，你的应用应该像以前一样运行。如果没有，它会在启动时崩溃，并且你的 logcat 会显示一个指示找不到许可证的错误。

很好，现在你已经在应用中安装并运行了 Ditto SDK。接下来，你必须在代码中利用它来在设备之间传递聊天消息、用户信息和任何其他你想要的数据。现在，你将学习另一种设计模式，它允许你以干净且可扩展的方式做到这一点：**仓库模式**。

## 仓库模式

有时，你希望在应用中存储和获取本地数据库中的数据。其他时候，你可能希望使用网络 API 从互联网或云服务获取数据。有时，你想从网络获取数据，但如果网络不可用，你希望从上次网络可用时创建的本地缓存中获取相同的数据。换句话说，你可能希望结合网络和本地数据。

无论你从哪里获取数据，ViewModel 都应该请求数据，UI 应该通过 ViewModel 的 MVI 和 Flows 监听数据的变化并更新。但最好不要将获取数据的所有实现细节都放在 ViewModel 类中。

理想情况下，你的 ViewModel 将完全不知道数据来自哪里。相反，它将通过 Flow 说"给我数据！"，你的 Compose UI 将监听这些流的变化并根据需要重组。

关于数据来自何处以及如何获取的细节，还有一层抽象。这一层是**仓库**。仓库模式为应用提供了一种组织多个数据源的方式，作为应用数据的单一真实来源，并将数据源（网络、缓存等）从 ViewModel 中抽象出来。

仓库类负责以下任务：

+ 向应用的其余部分公开数据。
+ 集中处理数据变更。
+ 解决数据源之间的冲突。
+ 从应用的其余部分抽象数据源。
+ 包含与数据转换相关的业务逻辑。

在 Android 应用中实现仓库模式的典型方式是创建一个接口，定义你将从 ViewModel 调用的所有方法，然后创建一个实现类来实现如何实现这些方法的细节。你可能有多个仓库类；例如，你可能有一个用于访问网络 API 的仓库，另一个用于处理 Data Store。

打开本章最终项目中的 **Constants.kt**，并将其中的所有值复制并粘贴到应用中的相同类中。这提供了便于使用的键字符串。这是一种常见的约定，可以避免开发人员在应用的不同位置手动键入相同的键字符串并可能打错的错误。

在 **com.kodeco.chat ▸ data** 下创建一个新包，并将其命名为 **repository**。

从最终项目的 **repository** 包中复制 **Repository.kt** 和 **RepositoryImpl.kt**，并将它们粘贴到项目中的相同位置。**Repository.kt** 是接口，**RepositoryImpl.kt** 是其实现。打开 **Repository.kt** 并查看其中定义的方法。特别注意，有一个 `getAllUsers()` 方法来检索参与此应用 Ditto 网格的所有用户。最终，此流由 **RepositoryImpl.kt** 中的以下函数填充：

```kotlin
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

仓库提供了有关如何获取和发送数据到 Ditto P2P 网格的实现细节。你：

1. 获取所有用户的集合。
2. *订阅*此集合，以便在集合发生变化（例如新用户加入应用的网格）时收到通知。
3. 将此订阅映射到 `allUsers: MutableStateFlow` 属性，`getAllUsers()` 会检索该属性。

`getAllMessagesForRoom()` 同样从 Ditto 填充。

`createMessageForRoom()` 类似，但使用 Ditto 函数 `upsert` 将聊天消息作为 Ditto `document` 插入到房间的消息集合中。

你不再需要 **FakeData.kt**，因此通过右键点击并从上下文菜单中选择 **Refactor ▸ Safe Delete** 来安全删除此类。

在 **MainViewModel.kt** 中，删除 `import com.kodeco.chat.data.initialMessages` 的导入。然后，按如下方式更新类：

```kotlin
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

1. 在这里硬编码 `userId`。理想情况下，你会在首次应用启动时生成此值并存储以供将来使用。参见第 9 章"Data Store"，了解如何在 Data Store 中存储类似这样的值。
2. 获取仓库的单例实例。
3. 这个流是你存储特定聊天室消息的地方。请注意，此流是两个流的*组合*：一个是所有用户的流，另一个是聊天室中所有消息的流。对任一流的更新也会更新组合流，进而触发监听流更新的可组合函数的更新。
4. 为了方便，用户名设置为设备名称，这样当你在设备之间聊天时，可以清楚地知道每条消息来自哪里。你可以稍后添加功能，让用户设置自己的名称。
5. 创建新消息的函数现在使用仓库来处理向 Ditto P2P 网格发送消息，而不是更新仅对一台设备本地的消息列表。

你需要导入以下内容：

```kotlin
import com.kodeco.chat.data.repository.RepositoryImpl
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.combine
import live.ditto.DittoAttachmentToken
```

打开 **MainActivity.kt**。按如下方式更新 `messagesWithUsers` 的定义：

```kotlin
val messagesWithUsers: List<MessageUiModel> by viewModel
  .roomMessagesWithUsersFlow
  .collectAsStateWithLifecycle(initialValue = emptyList())
```

你现在检索在 ViewModel 中创建的新消息列表流，将其传递给 `ConversationContent` 可组合函数。仓库使用 Ditto SDK 更新此流，可组合函数监听流的变化，UI 自动更新。

同时，更新 `currentUiState`：

```kotlin
val currentUiState =
  ConversationUiState(
    channelName = "Android Apprentice",
    initialMessages = messagesWithUsers.asReversed(),
    viewModel = viewModel
)
```

这里唯一的变化是添加 `.asReversed()`，以确保添加到流中的消息以正确的顺序显示，因为列表在 UI 中也是反向显示的。

接下来，你需要更新数据类。打开 **ConversationUiState.kt** 并按如下方式更新：

```kotlin
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

这里的主要变化是添加了 `constructor`，它将 `Message` 映射到 Ditto 文档类型。

导入以下内容：

```kotlin
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

类似地，向 **ChatRoom.kt** 添加以下 `constructor`：

```kotlin
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

导入以下内容：

```kotlin
import com.kodeco.chat.data.collectionIdKey
import com.kodeco.chat.data.createdByKey
import com.kodeco.chat.data.createdOnKey
import com.kodeco.chat.data.dbIdKey
import com.kodeco.chat.data.isPrivateKey
import com.kodeco.chat.data.messagesIdKey
import com.kodeco.chat.data.nameKey
import live.ditto.DittoDocument
```

向 **User.kt** 添加以下 `constructor`：

```kotlin
constructor(document: DittoDocument) : this(
  document[dbIdKey].stringValue,
  document[firstNameKey].stringValue,
  document[lastNameKey].stringValue
)
```

你需要导入以下内容：

```kotlin
import com.kodeco.chat.data.dbIdKey
import com.kodeco.chat.data.firstNameKey
import com.kodeco.chat.data.lastNameKey
import live.ditto.DittoDocument
```

就是这样！

构建并运行。输入一些聊天消息。现在，接下来是令人震惊的部分！在*第二台*设备或模拟器上构建并运行。你在第一台设备上输入的聊天消息会出现在第二台设备上。在第二台设备上输入一些消息...它们会立即出现在你运行应用的*所有*设备上。

![1743589066733](https://cdn.jsdelivr.net/gh/hyh19/images3@master/1743589064592.png)

尝试在任意多台设备上运行。

恭喜！你已经构建了一个完全功能的 P2P 聊天应用！

## 要点

干得好！在本章中，你涵盖了很多内容，并创建了一个真实世界的 P2P Android 聊天应用！回顾一下，你学到了：

+ 如何使用仓库模式进一步抽象你的应用逻辑，使你的应用后端与具体实现无关。
+ 如何使用 Ditto SDK 创建一个 P2P、可选云的聊天应用！

## 接下来去哪里？

如需深入了解应用架构和设计模式，请查阅 [*Advanced Android App Architecture*](https://www.kodeco.com/books/advanced-android-app-architecture) 一书。

有关 Ditto SDK 的更多详细信息，请参阅 Ditto [开发者文档](https://docs.ditto.live/)。

在本章中，有一些用于处理 `DittoAttachment` 的存根代码，但你没有实现将图像附加到聊天消息的功能。如果你想用 Ditto 进一步挑战自己，请尝试在聊天应用中实现图像附件。你在本章中还只实现了一个聊天室；另一个挑战是尝试实现多个聊天室。

最后，你可能想在应用中本地缓存用户 ID 等内容，以便应用可以在启动间"记住"它。在第 9 章"Data Store"中，你将学习如何使用 **Data Store** 在本地存储少量数据。在第 10 章"Room Database"中，你将学习如何使用 **Room** 在应用中本地存储各种数据。
