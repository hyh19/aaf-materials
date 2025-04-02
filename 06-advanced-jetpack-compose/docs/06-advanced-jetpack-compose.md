# 6. Advanced Jetpack Compose

In the previous chapter, you learned about some building blocks in Compose UI to start developing a basic UI for an Android app. Using Compose, you built up the interface for the chat app using mocked data. It’s like you made the yummy-looking but completely fake cake some stores display. But you want to have your cake and eat it, too! In this chapter, you’ll learn how to make your app more functional using ViewModel for managing app data, adopt MVI (Model-View-Intent) to structure your app behavior and navigate through app screens using the Navigation library. Get ready to *chat it up*!

## State

To make any app functional, you must know how to **manage state**. At its core, every app works with specific values that can **change**. For example, in Kodeco chat, a user can:

+ Add a new chat message
+ Delete a chat message
+ Upload an image attachment to a chat message

**State** is any **value** that **can change over time**. Those values can include anything from a database entry to a class property. As the state changes, it is crucial that the UI accurately reflects that state, so you’ll need to **update UI** when the state changes.

Compose is declarative, so the only way to update it is by calling the same composable with new arguments. These arguments are representations of the UI state. Any time a state is updated, a *recomposition* occurs. You might not have realized it, but you’ve already been updating the state of composables. Recall in `UserInputText`, the composable you used in the last chapter for the user to type in a chat message:

```
@ExperimentalFoundationApi
@Composable
private fun UserInputText(
  keyboardType: KeyboardType = KeyboardType.Text,
  onTextChanged: (TextFieldValue) -> Unit,
  textFieldValue: TextFieldValue,
  photoUri: Uri?,
  keyboardShown: Boolean,
  onTextFieldFocused: (Boolean) -> Unit,
  focusState: Boolean,
  onMessageSent: KeyboardActionScope.() -> Unit
) {
  // ...
  BasicTextField(
    value = textFieldValue,
    onValueChange = { onTextChanged(it) }
  )
  // ...
```

When you input text into a text field, it’s important for the displayed value to reflect your input in real-time. This is where you’ll use `onValueChange`. Every time you type a character into the text field, this composable gets *recomposed*. In other words, the *state* of the text field updates.

Also, recall you previously used `remember` to store the state of a composable:

```
var chatInputText by remember { mutableStateOf("") }
```

A composable that uses `remember` to store an object creates internal state, making the composable *stateful*. This can be useful when you have simple composables that you want to manage their own state. But these are also less reusable and harder to test.

### State Hoisting

A *stateless* composable is a composable that doesn’t hold any state. An easy way to achieve stateless-ness is by using *state hoisting*. You’ve also been using this already. Again:

```
BasicTextField(
  value = textFieldValue,
  onValueChange = { onTextChanged(it) }
)
```

State hoisting is a programming pattern in which you **move state to the caller of a composable** by replacing internal state in a composable with **a parameter** and **events**.

For composables, this often means introducing two parameters to the composable:

+ **value: T**: The current value to display.
+ **onValueChange: (T) -> Unit**: An event that requests a change to a value, where `(T)` represents providing a new value.

The token `T` represents a generic type that depends on the data and the UI you’re showing. If you look at the parameters of `UserInputText` again, you see that you follow the same approach for your state and events. In that case, your `T` is a `TextFieldValue`.

By applying **state hoisting** to a composable, you make it stateless — which means it can’t change any state. Stateless composables are easier to test, have fewer bugs and offer more reuse opportunities.

### Unidirectional Data Flow

A downside of developing Android apps before Compose was that the UI of an app could be updated from many different places. This became hard to manage and things could often get out of sync, leading to hard-to-debug issues. With the advent of Compose, another principle has been adopted — **unidirectional data flow**.

In unidirectional data flow, both the state changes and UI updates have only one direction. This means that the state change events can come from only one source, usually user interactions, and UI updates can come only from the state manager. Compose was based on the concept of **decoupling** components that **display state** in the UI from the app parts that **store and change state**.

Event State State UI

Another key concept is that the UI **observes the state**. Every time there’s a new state, the UI *recomposes* to display it. Android provides some very handy **Android Architecture Components** to help with this. For the state manager, there’s the **ViewModel**. And for observing data in a unidirectional manner, there’s **Flow**.

## ViewModel

In Chapter 3, “Android Fundamentals”, you learned about the lifecycle of Activities. In Android, Activity lifecycle events get triggered whenever a configuration change occurs, such as the device being rotated. Essentially, something benign like rotating the device causes activity to re-create from scratch. If you have data or state information tightly coupled to the UI, you might lose it when a configuration change occurs. As your app grows in complexity and scale, it becomes increasingly important to decouple your application data and logic outside the **composables** and the UI layer. Fortunately, Android provides a built-in architecture component to help you do that: the ViewModel.

Create a new package under **com.kodeco.chat** and name it **viewmodel**. Right-click that package and create a new **MainViewModel** Kotlin class. You can have more than one ViewModel; you could have a ViewModel for each activity in your app and share a single ViewModel between activities. For now, you’ll just have one.

Open **MainViewModel** and update it to extend the ViewModel class:

```
class MainViewModel : ViewModel() {
}
```

Android Studio should automatically import the `androidx.lifecycle.ViewModel` package.

You need to create the logic in your ViewModel for adding a new message to the list of messages. But to do that, you first must keep a list of messages somewhere!

You might recall that in the last chapter you created a `MessageUiModel` — an object comprising a chat `Message` and a `User`. The list of messages will be a list of `MessageUiModel`. Add the following properties to `MainViewModel`:

```
// 1
private val userId = UUID.randomUUID().toString()
// 2
private val _messages: MutableList<MessageUiModel> = initialMessages.toMutableStateList()
// 3
private val _messagesFlow: MutableStateFlow<List<MessageUiModel>> by lazy {
  MutableStateFlow(emptyList())
}
val messages = _messagesFlow.asStateFlow()
```

2. This is the ID of the person using the app on this device. It’s a random UUID that ensures each chat user has a unique identity. Ideally, you would generate this on the first app launch and store it for future use. You’ll learn how to store a value like this between app launches in 9, “Data Store”.
4. This is the list of messages in the chat. Later in this chapter, you’ll retrieve this from a service, but store it in the ViewModel for now. It gets populated initially with the list of fake messages. If you didn’t want to see the dummy message data, you could initialize it with an empty list.
6. You’ll learn all about MVI and Flow in the next section. Basically, `_messagesFlow` is a private flow that can be modified only by methods inside the ViewModel. `messages` is the variable that your Compose UI listens to changes for and updates when the variable is updated.

Next, add these functions to your ViewModel for handling the action when the user clicks “send” in the message entry UI:

```
// 1
fun onCreateNewMessageClick(messageText: String, photoUri: Uri?) {
  // 2
  val currentMoment: Instant = Clock.System.now()
  // 3
  val message = Message(
    UUID.randomUUID().toString(),
     currentMoment,
     currentRoom.value.id,
     messageText,
     userId,
     photoUri
  )
  // 4
  if (message.photoUri == null) {
    viewModelScope.launch(Dispatchers.Default) {
      createMessageForRoom(message, currentRoom.value)
    }
  }
}
// 5
suspend fun createMessageForRoom(message: Message, chatRoom: ChatRoom) {
  // 6
  val user = User(userId)
  val messageUIModel = MessageUiModel(message, user)
  // 7
  _messages.add(messageUIModel)
  // 8
  _messagesFlow.emit(_messages)
}
```

2. The function to handle the message click takes two parameters. The actual message text is required, and a photo attachment is optional. The message entry UI already has a button for adding a photo from the device’s photo library. Tapping it brings up images you can attach from the library, but they don’t actually get attached because you haven’t added that functionality yet.
4. The current time is calculated when the message is sent. This timestamp is part of the UI. But its phrasing depends on how long ago the moment was, as you already saw with the dummy message data.
6. A new `Message` is created.
8. For now, you’re only handling the case where there’s no image attachment in the message. A `ViewModelScope` is defined for each [`ViewModel`](https://developer.android.com/topic/libraries/architecture/viewmodel) in your app. Any coroutine launched in this scope is automatically canceled if the `ViewModel` is cleared. Hence, out of the box, the `ViewModel` gives you a very important way to scope coroutines dealing with your UI so they’re canceled when a given screen is no longer active. This avoids an age-old problem of legacy Android development: memory leaks caused by something being put into memory by a view and then orphaned when the view goes out of scope.
10. Because you’re using a coroutine, creating the message for a chat room must be a `suspend` function. Right now, you only have one chat room in the app called “Android Apprentice”. But once you have multiple devices and users, you could have many chat rooms: direct messages between two or more users. For now, the `chatRoom` parameter remains unused.
12. Create a `MessageUiModel` instance using the userID generated earlier and the message text.
14. Add the `MessageUiModel` to the list you’re storing on the ViewModel.
16. Here is where the real “magic” happens. The flow that the UI listens to gets updated with a new value, which is the updated message list, using `emit()`.

### MVI, Flows and StateFlow

Traditionally, if your app needed data, it might create a request for this data via a network API or database service, etc. For example, when the view starts, you request data from the ViewModel, and then the ViewModel requests that data from a data layer. The received data returns in the other direction, from the data layer to the ViewModel, and then the UI gets updated. You might do all this asynchronously using suspend functions (coroutines).

Data Layer creates request View Requests Data View Starts Data Layer receives data ViewModel Receives Data View Receives Data

But a more efficient architecture is to *observe* for data changes instead of continuously requesting them. Then, any updates in the data source automatically *Flow* down to the view.

Data Layer observes source ViewModel Observes Data Layer View Observes ViewModel

This type of system is called *reactive* because observers react automatically to changes in the things being observed. Another important design pattern of note here is that the data flows in only one direction. This type of unidirectional data flow is a design pattern called **MVI**. MVI, or “Model-View-Intent”, focuses on unidirectional data flow and immutability.

The **Model** represents the application state. The **View** represents the UI, which is rendered based on the state received from the model. The **Intents** represent user actions or interactions with the view, triggering events that get dispatched to the Model.

A *Flow* is a type that can emit values sequentially, as opposed to coroutine *suspend* functions that return only a single value. For example, you can use a flow to receive live updates from a database or a cloud API.

This lets you fetch data from different sources or update the UI without blocking the main thread.

In Compose, UI components subscribe to a Flow. When the Flow updates, these subscribing *composables* are recomposed to display the updated values.

**StateFlow** extends Flow, providing built-in lifecycle awareness. Typically, in an Android app, you want to use a StateFlow, which is specific to Android and provides the lifecycle-aware benefits, rather than a generic Flow that isn’t specific to Android and requires additional management.

The combination of ViewModels and Compose UI offers an efficient paradigm for updating the UI when the view is being viewed and then freeing memory when the UI in question is no longer being used.

To learn more about Flows and StateFlow in Android, see [this guide](https://developer.android.com/kotlin/flow) from the Android Developer documentation.

Open **FakeData.kt**. To get the fake data to work with your new ViewModel, delete the `private` modifier for `initialMessages` so it can be accessed from the ViewModel. Then, delete `exampleUiState` because you no longer need it.

Open **ConversationUiState.kt** and add this property:

```
val viewModel: MainViewModel
```

Then, replace the TODO comment in `addMessage()` with a call to the function you just added to the ViewModel:

```
viewModel.onCreateNewMessageClick(msg, photoUri)
```

Next, open **Conversation.kt**. Update the call to `UserInput` from

`UserInput(onMessageSent = {})` to:

```
UserInput(onMessageSent = { content ->
  uiState.addMessage(content, null)
},
resetScroll = {
  scope.launch {
    scrollState.scrollToItem(0)
  }
},
// Use navigationBarsPadding() imePadding() and , to move the input panel above both the
modifier = Modifier
  .navigationBarsPadding()
  .imePadding(),
)
```

This wires up the send functionality with the function call to the ViewModel through `ConversationUiState`. It also adds a modifier for padding as well as scrolling behavior.

Also, update the call to `Messages()` with `scrollState`:

```
Messages(
  messages = uiState.messages,
  scrollState = scrollState,
  modifier = Modifier.weight(1f),
)
```

And update the actual `Messages()` composable accordingly:

```
@Composable
fun Messages(
  messages: List<MessageUiModel>,
  scrollState: LazyListState,
  modifier: Modifier = Modifier,
) {
  Box(modifier = modifier) {
    LazyColumn(
      state = scrollState,
      // ...everything else is the same as before
```

Finally, update **MainActivity.kt**:

```
class MainActivity : ComponentActivity() {
  // 1
  private val viewModel: MainViewModel by viewModels()

  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContent {
      // 2
      val messagesWithUsers by viewModel.messages.collectAsStateWithLifecycle()
      // 3
      val currentUiState =
        ConversationUiState(
          channelName = "Android Apprentice",
          initialMessages = messagesWithUsers,
          viewModel = viewModel
        )

      KodecochatTheme {
        ConversationContent(
          currentUiState,
        )
      }
    }
  }
}
```

2. Add a property for the ViewModel. `by viewModels()` is how you access a ViewModel from an Activity. It’s a Kotlin property delegate provided by the `activity-ktx` library. To access it from a Fragment, you would instead use `by activityViewModels()`.
4. Inside the `setContent{...}`, you’re now using composables. This is how you access publicly exposed properties on your ViewModel from within a composable, i.e., `by viewModel.` followed by the property name. The next part of this expression is key: This is where you *collect* the data *emitted* from the flow. Remember, you’re using the `StateFlow` variety of flow here to benefit from being lifecycle aware.
6. You create an instance of `ConversationUiState` to pass to your composable. The chat room’s name is hard-coded here because there’s only one chat room at this point. The initial messages are passed from the ViewModel. Note that this list is initialized as an `emptyList()` in the ViewModel. The dummy message list is added only after the user sends their first chat message. If you wanted the dummy chat messages to appear in the UI from the beginning, you could have an init function do this. And, as mentioned before, if you don’t want any dummy data, you don’t need to pass this data in. For now, you can leave the dummy data in to test the scrolling behaviors.

To use `collectAsStateWithLifecycle`, you must ensure your Gradle includes the dependency for `androidx.lifecycle:lifecycle-runtime-compose`. If you followed along in Chapter 4, “Gradle Basics: A Look Behind the Curtain”, and are using a Version Catalog, ensure you have the following in your **libs.versions.toml** file:

```
[versions]
lifecycle-runtime-compose = "2.6.2"

[libraries]
androidx-lifecycle-runtime-compose = { module = "androidx.lifecycle:lifecycle-runtime-compose", version.ref = "lifecycle-runtime-compose" }
```

In your module-level Gradle, add the following dependency:

```
// compose lifecycle
    implementation(libs.androidx.lifecycle.runtime.compose)
```

Sync the project with the Gradle files. Now, `collectAsStateWithLifecycle` should be available for you to use.

Build and run. Type something into the text entry area and tap the send button or the send icon on the keyboard (it’s an icon in the lower-right corner that looks like a paper airplane). Your text messages should now update the UI, adding the dummy data followed by any messages you type!

 ![](./Android Fundamentals by Tutorials, Chapter 6_Advanced Jetpack Compose_ Kodeco_files/original.png)

This is great, but there are some things you can improve, and this is still a one-way conversation. First, notice that when you send a chat, the UI shows the chat message coming from the non-“me” persona instead of from “me”.

Next, if the chat contains a lot of text and you’re not already at the end, it would be great to quickly jump to the latest message with a tap.

In **Conversation.kt**, look at the `Messages()` composable. Notice the call to `MessageUi()`:

```
MessageUi(
  onAuthorClick = {  },
  msg = content,
  authorId = "me",
  userId = userId ?: "",
  isFirstMessageByAuthor = isFirstMessageByAuthor,
  isLastMessageByAuthor = isLastMessageByAuthor,
)
```

`authorId` is hard-coded as “me”. But, in the ViewModel, you now track the user’s identity by a private `userId` UUID. This is necessary; otherwise, every device user would be “me”, and there would be no way to distinguish users on different devices.

Add the following to your ViewModel:

```
var currentUserId = MutableStateFlow(userId)
```

This creates a `StateFlow` and initializes it with the userId on that device.

In **ConversationUiState.kt**, add the following property:

```
val authorId: MutableStateFlow<String> = viewModel.currentUserId
```

This will be used to tell if the message is sent from this user (self) when rendering the UI.

In **Conversation.kt**, add the following property to `ConversationContent()`:

```
val authorId = uiState.authorId.collectAsStateWithLifecycle()
```

Update the call to `Messages()` to pass in this value:

```
Messages(
  messages = uiState.messages,
  authorId = authorId.value,
  scrollState = scrollState,
  modifier = Modifier.weight(1f),
)
```

Update the signature of `Messages()` accordingly:

```
@Composable
fun Messages(
  messages: List<MessageUiModel>,
  authorId: String,
  scrollState: LazyListState,
  modifier: Modifier = Modifier,
) {
```

In the call to `MessageUi()`, change the assignment of `authorId` to:

```
authorId = content.user.id,
```

Then, in the definition of `MessageUi()`, change `val isUserMe = userId == "me"` to:

```
val isUserMe = authorId == userId
```

Build and run. Any messages you type appear to be sent from “me” instead of the “other” persona.

Next, to add a “jump to bottom” button, copy **JumpToBottom.kt** from the **conversation** package in the final project for this chapter to the same location in your project.

Update your **strings.xml** file **res** to provide a label for the button:

```
<string name="jumpBottom">Jump to bottom</string>
```

In **Conversation.kt**, update the `Messages()` composable as follows:

```
@Composable
fun Messages(
  messages: List<MessageUiModel>,
  authorId: String,
  scrollState: LazyListState,
  modifier: Modifier = Modifier,
) {
  // 1
  val scope = rememberCoroutineScope()
  Box(modifier = modifier) {
    LazyColumn(
      // 2
      reverseLayout = true,
      state = scrollState,
      // Add content padding so that the content can be scrolled (y-axis)
      // below the status bar + app bar
      contentPadding =
      WindowInsets.statusBars.add(WindowInsets(top = 90.dp)).asPaddingValues(),
      modifier = Modifier
        .fillMaxSize()
    ) {
      itemsIndexed(
        items = messages,
        key= { _, message -> message.id }
      ) { index, content ->
        val prevAuthor = messages.getOrNull(index - 1)?.message?.userId
        val nextAuthor = messages.getOrNull(index + 1)?.message?.userId
        val userId = messages.getOrNull(index)?.message?.userId
        val isFirstMessageByAuthor = prevAuthor != content.message.userId
        val isLastMessageByAuthor = nextAuthor != content.message.userId
        MessageUi(
          onAuthorClick = {  },
          msg = content,
          authorId = authorId,
          userId = userId ?: "",
          isFirstMessageByAuthor = isFirstMessageByAuthor,
          isLastMessageByAuthor = isLastMessageByAuthor,
        )
      }
    }
    // 3
    //
    val jumpThreshold = with(LocalDensity.current) {
      JumpToBottomThreshold.toPx()
    }
    // 4
    val jumpToBottomButtonEnabled by remember {
      derivedStateOf {
        scrollState.firstVisibleItemIndex != 0 ||
            scrollState.firstVisibleItemScrollOffset > jumpThreshold
      }
    }
    JumpToBottom(
      // 5
      enabled = jumpToBottomButtonEnabled,
      onClicked = {
        scope.launch {
          scrollState.animateScrollToItem(0)
        }
      },
      modifier = Modifier.align(Alignment.BottomCenter)
    )
  }
}
```

Here are the main changes:

2. Define a coroutine scope.
4. Reverse the layout so the messages appear in the opposite order. This is important because of how the jump to bottom is implemented: It jumps to the beginning of the list, which now appears at the bottom.
6. The jump-to-bottom button appears when the user scrolls past a threshold. You calculate the value in pixels.
8. Show the button if the first visible item isn’t the first one or if the offset is greater than the threshold.
10. Only show the button if the scroller isn’t at the bottom.

At the very end of **Conversation.kt**, add:

```
private val JumpToBottomThreshold = 56.dp
```

Finally, in **MainViewModel.kt**, update `createMessageForRoom()` so messages get added to the beginning of the message list instead of the end:

```
_messages.add(0, messageUIModel)
```

Build and run. Add new messages to the chat until the message list grows past what shows in the UI. Scroll up, and the “Jump to bottom” button appears.

 ![](./Android Fundamentals by Tutorials, Chapter 6_Advanced Jetpack Compose_ Kodeco_files/original(1).png)

Tap the button, and the messages list scrolls to the bottom!

## Key Points

Well done! You’ve covered some important concepts, architectures and design patterns in Android and Compose. To recap, you learned:

+ All about state in Compose and how to make stateful and stateless composables.
+ How to separate concerns between UI, data and application logic using ViewModel.
+ How to take advantage of unidirectional data flow in Compose using Flow and the MVI architecture pattern.

## Where to Go From Here?

In this chapter, you’ve gotten a taste of some of the new architecture used with Jetpack Compose, namely MVI, and how to use Compose with other Jetpack libraries, such as ViewModel. To learn more about Google’s thoughts on architecture, their recommendations and learning paths, see the article [*Rebuilding our Guide to app Architecture*](https://android-developers.googleblog.com/2021/12/rebuilding-our-guide-to-app-architecture.html?m=1) and the actual [guide to app architecture](https://developer.android.com/topic/architecture), which remains a work in progress but goes into much greater depth and broader scope than this chapter does.

ViewModel and Compose work well with another concept, **Dependency Injection** (DI). DI makes your application code cleaner and much easier to test. For an in-depth guide to using Android’s DI library, **Hilt** — which is built on top of and simplifies the use of another DI library, **Dagger** — be sure to dive into yet another amazing resource from Kodeco, [*Dagger by Tutorials*](https://www.kodeco.com/books/dagger-by-tutorials).

In the next chapter, you’ll learn about one more important design pattern in mobile development: the **repository pattern**. You’ll also complete the Kodeco chat app!
