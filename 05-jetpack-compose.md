# 5. Jetpack Compose

Written by Fuad Kamal

Traditionally, Android applications relied on XML-based layouts. Android development has now shifted to Compose as the standard framework. Compose describes a much larger set of frameworks and application architecture in Kotlin, which isn’t specific to Android development. For example, the development teams at Slack have developed the [Circuit](https://slackhq.github.io/circuit/) framework, built on top of Compose. Compose UI is just one of the seven Compose frameworks specific to the UI layer of application development. Using Compose UI brings many improvements over the old View implementation, including significantly reduced build times, APK size and runtime performance. It also makes building the UI easier, more intuitive, and easier to maintain and debug. For a more in-depth comparison of Compose and the old Android View implementation, see [this article](https://developer.android.com/jetpack/compose/migrate/compare-performance).

Modern Android development has shifted from MVVM (Model-View-Viewmodel) to **MVI** (Model-View-Intent) architecture, with Compose UI now used for the UI layer rather than XML layouts. One of the key concepts behind MVI is unidirectional data flow. Often, you can have multiple data sources, including local storage and network sources. These are typically abstracted and accessed using a **Repository** pattern. A ViewModel accesses methods on the Repository and provides unidirectional data to the UI using **Flow**. A Flow typically consists of a data emitter and subscribers to the emitter. It provides an efficient memory usage model because the Flow won’t consume memory until and unless an active subscriber is using the data from the Flow. For Android, there are specific implementations of Flow that are Android lifecycle aware. Hence, when a view is no longer visible to the user, data is no longer consumed, freeing up memory.

In this chapter, you’ll learn the basics of Compose UI and build a simple Android app UI with it. In the next chapter, you’ll dive deeper and learn about the other essential pieces to wire a Compose app together, including the ViewModel, MVI, the Repository pattern and Navigation.

## Compose Fundamentals

If you’ve been following along with the previous chapters, open your Kodeco Chat app in Android Studio. Otherwise, open this chapter’s **starter** project using Android Studio and select **Open an existing project**. Next, navigate to **05-jetpack-compose/projects** and select the **starter** folder as the project root. Once the project opens, let it build and sync, and you’ll be ready to go!

Recall that in Chapter 2 you created a new Android project from scratch. The default Activity created for you contained the following function:

```kotlin
@Composable
fun Greeting(name: String, modifier: Modifier = Modifier) {
  Text(
    text = "Hello $name!",
    modifier = modifier
  )
}
```

This code has two notable things about it. First, it’s a function; second, it’s annotated with `@Composable`. This is all you need to create a UI component in Compose, or, in Compose speak, a **composable**.

In your project, open **MainActivity.kt**. Look through the code — where is the `@Composable` annotation?

```kotlin
class MainActivity : ComponentActivity() {
  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContent {
      Column { ...
```

It doesn’t seem to appear anywhere…

But take a closer look: MainActivity is a subclass of **ComponentActivity**. In the `onCreate()` function you call `setContent`.

Command-click (if using a Mac; Control-click if using a PC) `setContent`. Android Studio opens this function’s definition, which is defined in **ComponentActivity.kt**:

```kotlin
public fun ComponentActivity.setContent(
  parent: CompositionContext? = null,
  content: @Composable () -> Unit
) {...}
```

Aha! So, first of all, `setContent` is an extension function of `ComponentActivity`. Extension functions add additional functionality to a class without changing its source code. Calling `setContent()` sets the given composable function named `content` as the root view, to which you can add any number of elements. You call the rest of your composable functions from within this container.

Second, note that `content` is also annotated with `@Composable`. You don’t need to add it again before putting in composables like the `Column` above because the annotation is here.

Right-click **com.kodeco.chat** in the Project Navigator and select **New -> Package** from the context-menu:

 ![](https://assets.alexandria.kodeco.com/books/9f59dbccde22edd2ab5693b6e6b2b51c5d4fe4261fa1a6477dd65d3d2a9028de/images/ef0eab801d90527c96c8da354263c6b7/original.png)

Name the new package “conversation”. This is where you’ll create the components that compose the pieces of the chat UI.

Next, right-click the **conversation** package and select **New -> Kotlin Class/File**:

 ![](https://assets.alexandria.kodeco.com/books/9f59dbccde22edd2ab5693b6e6b2b51c5d4fe4261fa1a6477dd65d3d2a9028de/images/ebbab1fb765cfa5809f22881486b05d5/original.png)

Ensure that “file” is selected and name the new file “Conversation”:

 ![](https://assets.alexandria.kodeco.com/books/9f59dbccde22edd2ab5693b6e6b2b51c5d4fe4261fa1a6477dd65d3d2a9028de/images/de9835ae6b08d1e31353ecfea39af674/original.png)

Android Studio creates an empty Kotlin file in the **conversation** package named **Conversation.kt**.

In **Conversation.kt**, type the following:

```kotlin
@Composable
fun ConversationContent() {
  // TODO: create conversation UI here
}
```

Congratulations on writing your first Compose function! It doesn’t do anything yet, but you’ll soon change that.

Go back to **MainActivity.kt** and copy everything from inside the braces `setContent{}` and paste it into the body of `ConversationContent`:

```kotlin
@Composable
fun ConversationContent() {
  Column {
    val context = LocalContext.current
    var chatInputText by remember { mutableStateOf(context.getString(R.string.chat_entry_default)) }
    var chatOutputText by remember { mutableStateOf(context.getString(R.string.chat_display_default)) }
    Text(text = chatOutputText)

    OutlinedTextField(
      value = chatInputText,
      onValueChange = {
        chatInputText = it
      },
      label = { Text(text = stringResource(id = R.string.chat_entry_label)) }
    )

    Button(onClick = {
      chatOutputText = chatInputText
      chatInputText = ""
    }) {
      Text(text = stringResource(id = R.string.send_button))
    }
  }
}
```

Then, go back to **MainActivity.kt** and replace everything in `setContent{}` with `ConversationContent()`. Your Activity class should look much simpler and cleaner:

```kotlin
class MainActivity : ComponentActivity() {
  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContent {
      ConversationContent()
    }
  }
}
```

Build and run your app. It should work exactly as before.

## Breaking Down Composables

Although the simple copy/paste action you performed might seem trivial, it highlights a couple key concepts. First, note that **Conversation.kt** is a separate file, not a class. It simply houses a Compose function. This function could have been placed in **MainActivity.kt**. But as your codebase expands, maintaining a single file for all your code can become cumbersome. Second, you’ve created a **composable function** named `ConversationContent`, which you can reuse throughout your app. This reusability is a cornerstone of Compose UI, akin to constructing a large sculpture from individual Lego blocks.

Some things to note about composable functions:

+ Composable functions can be called only from other composable functions.
+ Composable functions can receive parameters and use them to build the UI.
+ Composable functions can be invoked only from a compose scope, like how coroutines work.

The UI you’ve created so far showcases a variety of Compose UI elements, each one a composable function capable of accepting various parameters:

+ **Text**: A basic text element that displays text and provides accessibility information.
+ **OutlinedTextField**: Unlike Text, which only displays text, a text field allows the user to type text into the UI. This type of text field has a de-emphasized visual style.
+ **Button**: This is just what it sounds like; users click buttons to initiate an action.
+ **Column**: This differs from the other composables you’ve used. Although Text, TextField and Button all are UI elements, a Column is a type of *layout* composable. Layouts let you arrange UI elements in various ways. In the case of a Column, all of its *children* — the elements it contains — are laid out in a vertical or horizontal column. You’ll learn about several other layout composables and can create custom layouts in Compose.

To jump into its class definition, Command-click (if using a Mac; Control-click if using a PC) each of these controls in your code. Each control is accompanied by a detailed comment above its code, the customizable parameters and links to sample code.

Compose uses a declarative UI approach: You declare everything about how your UI should look using composable functions.

Look again at the code for the Button in `ConversationContent`:

```kotlin
Button(onClick = {
 chatOutputText = chatInputText
 chatInputText = ""
}) {
 Text(text = stringResource(id = R.string.send_button))
}
```

The `Button()` composable has all the features you’ve learned about for Composable functions. It takes a parameter, `onClick()`, which is itself a function. In Kotlin, functions can take other functions, or *lambdas*, as parameters. The `onClick()` function, defined inline here, defines what action will occur when the button is clicked. The body of the `Button()` function contains a `Text()`, which is the button’s label.

The `Button()` composable is a base button class, which is highly customizable. But there are five subclasses of `Button()` types you can use when you don’t need as much customization. For more information on the different button types and where and when to use them, see the [official documentation](https://developer.android.com/jetpack/compose/components/button) for `Button()`.

You might have noticed that composable functions use **Pascal case**, unlike the camel case that Kotlin code commonly uses. Consequently, the top-level composable you defined is `ConversationContent` instead of `conversationContent`. This distinction stems from the fact that composable functions return UI objects, thus adopting the same naming convention as classes.

## Improving the UI & UX

You might have noticed the inconvenience of having to manually delete the placeholder text when you start typing in the text field. This unnecessary step disrupts the user’s flow and introduces an additional mental hurdle, hindering their interaction with the app. Anything that detracts from the seamlessness and intuitiveness of the UI can lead to a poor user experience.

In **Conversation.kt**, Command-click (if using a Mac; Control-click if using a PC) `OutlinedTextField` to jump into its class definition. Throughout your App development career, you’ll develop a much deeper understanding of how things work by studying the source code of your frameworks. This leads to practical knowledge that will really help you level up your expertise.

Note the many parameters that can be passed into `OutlinedTextField`:

```kotlin
// 1
@OptIn(ExperimentalMaterial3Api::class)
// 2
@Composable
fun OutlinedTextField(
  value: String,
  onValueChange: (String) -> Unit,
  modifier: Modifier = Modifier,
  enabled: Boolean = true,
  readOnly: Boolean = false,
  textStyle: TextStyle = LocalTextStyle.current,
  // 3  
  label: @Composable (() -> Unit)? = null,
  // 4  
  placeholder: @Composable (() -> Unit)? = null,
  leadingIcon: @Composable (() -> Unit)? = null,
  trailingIcon: @Composable (() -> Unit)? = null,
  prefix: @Composable (() -> Unit)? = null,
  suffix: @Composable (() -> Unit)? = null,
  supportingText: @Composable (() -> Unit)? = null,
  isError: Boolean = false,
  visualTransformation: VisualTransformation = VisualTransformation.None,
  keyboardOptions: KeyboardOptions = KeyboardOptions.Default,
  keyboardActions: KeyboardActions = KeyboardActions.Default,
  singleLine: Boolean = false,
  maxLines: Int = if (singleLine) 1 else Int.MAX_VALUE,
  minLines: Int = 1,
  interactionSource: MutableInteractionSource = remember { MutableInteractionSource() },
  shape: Shape = OutlinedTextFieldDefaults.shape,
  colors: TextFieldColors = OutlinedTextFieldDefaults.colors()
) {...
```

Also, note the explanations provided in the comment block above the definition of `OutlinedTextField`.

2. `@OptIn(ExperimentalMaterial3Api::class)`: You’ll often see this `OptIn` followed by “Experimental” in Compose and other frameworks. Although these frameworks are now mature, they’re continually evolving. So, to access some of the new features, sometimes you’ll need to “opt in” to use experimental features. Don’t worry — Android Studio typically prompts you to add this annotation when it’s needed and warns you when the feature is no longer experimental and the annotation should be removed.
4. Again, as required, you see the `@Composable` annotation marking this as a composable.
6. The `label` parameter, currently set to “Enter Chat Text,” is intended to display a label in the text field container, according to the code comments. But the label appears inline with the text field’s outline in your UI, deviating from the expected placement.

 ![](https://assets.alexandria.kodeco.com/books/9f59dbccde22edd2ab5693b6e6b2b51c5d4fe4261fa1a6477dd65d3d2a9028de/images/fae714698ef2ce7437d224c93d65a325/original.png)

The “Enter Chat Text” label isn’t visible in the text field because placeholder text — “Type your text here” — currently occupies that space. Removing the placeholder text would allow the label to appear in the text field as intended.

5. Aha! What’s this `placeholder`field? The documentation says, “the optional placeholder to be displayed when the text field is in focus and the input text is empty”. Update your code to use this parameter instead.

Go back to **Conversation.kt** and update `ConversationConent` as follows:

```kotlin
@Composable
fun ConversationContent() {
  Column {
    val context = LocalContext.current
    var chatInputText by remember { mutableStateOf("") }
    var chatOutputText by remember { mutableStateOf(context.getString(R.string.chat_display_default)) }
    Text(text = chatOutputText)

    OutlinedTextField(
      value = chatInputText,
      placeholder = { Text(text = stringResource(id = R.string.chat_entry_default)) },
      onValueChange = {
        chatInputText = it
      },
    )

    Button(onClick = {
      chatOutputText = chatInputText
      chatInputText = ""
    }) {
      Text(text = stringResource(id = R.string.send_button))
    }

  }
}
```

You removed the default value of `chatInputText` and replaced it with an empty string, because you won’t need the default text there anymore. Next, you pass that default text to the `placeholder` parameter instead. Finally, you delete the `label` because it’s now redundant.

Build and run, and try typing some text into the text field. Voilà! Now, once you start typing, the default text automatically disappears.

 ![](https://assets.alexandria.kodeco.com/books/9f59dbccde22edd2ab5693b6e6b2b51c5d4fe4261fa1a6477dd65d3d2a9028de/images/4b49608c9f1cde7ca8b0695bb4de5243/original.png)

By looking at the source code for `OutlinedTextField`, you’ve better understood how to use it.

## Layout Groups

Layout Groups in Compose allow you to arrange your UI elements on the device screen in various ways. You can define layouts directly using the Compose `Layout()` class or use predefined layout types. Like you can combine composables and use them within one another, you can also nest layout groups to make more complex layouts. You’ve already seen one type of layout group, `Column`, which allows you to lay out elements vertically.

To arrange elements horizontally instead, you can use a `Row`.

Another layout composable is the `Box`. It’s used to display children (elements it contains) relative to their parent’s edges. It also allows you to stack or overlap children.

Finally, a `Surface` is a special layout that’s typically the top-level, or root, layout in a series of nested composables. A `Surface` can only have one child at a time, but it provides many style treatments for its children. It’s used as the central metaphor for **Material Design**, Google’s standard design library used in Android to provide a uniform user experience across devices.

Replace the body of `ConverstationContent()` with the following:

```kotlin
Surface {
  Box {
    Column {
      Messages()
      SimpleUserInput()
    }
  // Channel name bar floats above the messages
  ChannelNameBar(channelName = "Android Apprentice")
  }
}
```

Now, you’re using several layout composables you just learned about: Surface, Box and Column. But you’ve also added references to composables that don’t exist yet, causing Android Studio to show you some errors.

Click one of the composables that appears in red, such as `Messages`. Android Studio shows the error message and offers a solution.

 ![](https://assets.alexandria.kodeco.com/books/9f59dbccde22edd2ab5693b6e6b2b51c5d4fe4261fa1a6477dd65d3d2a9028de/images/0884119ae0b281d79279bf1f0c27c51d/original.png)

Or you can click the red light bulb icon to the left of the composable to see the same options.

 ![](https://assets.alexandria.kodeco.com/books/9f59dbccde22edd2ab5693b6e6b2b51c5d4fe4261fa1a6477dd65d3d2a9028de/images/36bd683eb454d7403e6913752560bd47/original.png)

### Dealing With Compile Issues

Select the “Create @Composable function…” for each of the undefined composables. Android Studio creates a stub function for that composable at the bottom of the file with a call to `TODO()` in the body of each. This special inline function will cause a compile error if you try to run the app, forcing you to implement the function before you can compile the app. You should now see the following functions at the end of **Conversation.kt**:

```kotlin
@Composable
fun Messages() {
  TODO("Not yet implemented")
}

@Composable
fun SimpleUserInput() {
  TODO("Not yet implemented")
}

@Composable
fun ChannelNameBar(channelName: String) {

}
```

Note that there isn’t a `TODO()` for `ChannelNameBar`. This is because you already partially implemented it by defining a parameter for the function.

Create another package, **components**, under **com.kodeco.chat**, then copy and paste **KodecochatAppBar.kt** and **KodecochatIcon.kt** from the final project for this chapter into your project. Then copy and paste **kodeco\_logo.xml** and **kodeco\_logo\_back.xml** from **res->drawable** of the starter into the same location in your project. These last two files contain vector assets, which scale without pixelation and are rendered at runtime by Android. Also, copy the values from **Strings.xml** under **res -> values** so you can access all the localized strings used in this chapter.

Next, replace the body of `ChannelNameBar()` with the following:

```kotlin
KodecochatAppBar(
  title = {
    Column(horizontalAlignment = Alignment.CenterHorizontally) {
      // Channel name
      Text(
        text = channelName,
        style = MaterialTheme.typography.titleMedium
      )
    }
  },
  actions = {
    // Info icon
    Icon(
      imageVector = Icons.Outlined.Info,
      tint = MaterialTheme.colorScheme.onSurfaceVariant,
      modifier = Modifier
        .clickable(onClick = { })
        .padding(horizontal = 12.dp, vertical = 16.dp)
        .height(24.dp),
      contentDescription = stringResource(id = R.string.info)
    )
  }
)
```

`KodecochatAppBar` might appear with a red underline in Android Studio. If so, move the mouse over it to see this dialogue in Android Studio:

 ![](https://assets.alexandria.kodeco.com/books/9f59dbccde22edd2ab5693b6e6b2b51c5d4fe4261fa1a6477dd65d3d2a9028de/images/9c925dc14580586aae3646c80eb109e0/original.png)

Click the “Opt in…” and Android Studio adds the `@OptIn(ExperimentalMaterial3Api::class)` annotation above the function definition, as discussed earlier.

Replace any calls to `TODO("Not yet implemented")` with a regular comment like `// TODO - Implement`. This allows you to build and run the app with compile issues, but the TODO comment gets highlighted with a blue tick (as opposed to red for errors and yellow for warnings) in the right gutter of the Android Studio code pane:

 ![](https://assets.alexandria.kodeco.com/books/9f59dbccde22edd2ab5693b6e6b2b51c5d4fe4261fa1a6477dd65d3d2a9028de/images/8f55158dd2067eb7c75252edae41bef8/original.png)

Build and run. You should now see a new app bar at the top of the device screen:

 ![](https://assets.alexandria.kodeco.com/books/9f59dbccde22edd2ab5693b6e6b2b51c5d4fe4261fa1a6477dd65d3d2a9028de/images/b628102653eba5bc9efd3de122d42320/original.png)

Command-click (if using a Mac; Control-click if using a PC) `KodecochatAppBar` to view the source:

```kotlin
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun KodecochatAppBar(
  modifier: Modifier = Modifier,
  scrollBehavior: TopAppBarScrollBehavior? = null,
  onNavIconPressed: () -> Unit = { },
  title: @Composable () -> Unit,
  actions: @Composable RowScope.() -> Unit = {}
) {
  // 1  
  CenterAlignedTopAppBar(
    modifier = modifier,
    // 2
    actions = actions,
    title = title,
    scrollBehavior = scrollBehavior,
    // 3    
    navigationIcon = {
      KodecoChatIcon(
        contentDescription = stringResource(id = R.string.navigation_drawer_open),
        modifier = Modifier
          .size(64.dp)
          .clickable(onClick = onNavIconPressed)
          .padding(16.dp)
      )
    }
  )
}
```

2. `KodecochatAppBar` is basically just a wrapper around the built-in class `CenterAlignedTopAppBar`. This is typically used to display information and actions at the top of a screen. The `channelName` parameter you passed into `ChannelNameBar` gets used for the `title` property in the `CenterAlignedTopAppBar`, which displays the title you now see at the top of the app screen. Further, in **Conversation.kt**, the value passed to `title` isn’t just a string; it’s an entire composable function. This composable consists of a column with its alignment property set to center horizontally and its content set to a `Text` field capturing the title string. In this way, you can see how Compose allows you to nest composables within one another to create more complex layouts and functionality.
4. `CenterAlignedTopAppBar`, and therefore `KodecochatAppBar`, takes a parameter called `actions`. This is typically supposed to be a list of `IconButtons`. These Material Design compact buttons help the user take some supplementary action. In this case, you’re passing an information button, which looks like an “i” with a circle around it. The premise is that you could add code later to provide the user with information about the chat channel when they tap it. Also, note that the actions get rendered in a `Row`.
6. `CenterAlignedTopAppBar` has another parameter, `navigationIcon`, that you’ll use to enable opening a side menu in your app later. For the icon, you use the Kodeco logo, which is provided as a vector asset. This, too, is built up as a composable. Notice again that you’re passing an actual composable function to this parameter.

## Previews

Open **KodecochatAppBar.kt**. At the top of Android Studio, beneath the run and debug icons, click the icon for **split** view, which is a combination of the code and design views in Android Studio:

 ![](https://assets.alexandria.kodeco.com/books/9f59dbccde22edd2ab5693b6e6b2b51c5d4fe4261fa1a6477dd65d3d2a9028de/images/d370047e10889bd0c975d728b304294a/original.png)

You should see Android Studio split into a few different panes:

 ![](https://assets.alexandria.kodeco.com/books/9f59dbccde22edd2ab5693b6e6b2b51c5d4fe4261fa1a6477dd65d3d2a9028de/images/87154da148dd5c25b8d1257720d9bea9/original.png)

2. In the middle is a preview of what the `KodecochatAppBar` will look like in both light and dark modes on the device.
4. You, the developer, write the code to generate these previews.

```kotlin
@OptIn(ExperimentalMaterial3Api::class)
@Preview
@Composable
fun KodecochatAppBarPreview() {
  KodecochatTheme {
    KodecochatAppBar(title = { Text("Preview!") })
  }
}
```

To create a preview in composable, you just write a compose function and add the `@Preview` annotation above it in addition to the `@Composable` annotation. Although doing so isn’t required, the naming convention is to put “Preview” at the end of the function name for readability. You can then define the parameters your composable function requires, and Android Studio will render the UI elements in the design view automatically. Another cool feature of Compose previews is that Android Studio updates them live as you edit your Compose source code.

Try this now: Back in `KodecochatAppBar()`, change the `.padding()` parameter passed in for `KodecoChatIcon` from 16 dp to 3 dp. As soon as you make the change, you should see the Kodeco logo on the left become much bigger. Change it back to the original value and it shrinks back. You didn’t need to build or run the app to see these changes! This can make developing your app much faster because you can often see design changes without running or rebuilding the app. You can also use previews to see how a composable will look when rendered on different devices and in different conditions, all at once. In this example, by defining two previews, you can see how the top bar looks in light and dark mode simultaneously. For more information on using composable previews, see the [Android documentation on Composable previews](https://developer.android.com/jetpack/compose/tooling/previews).

## Modifiers

**Modifiers** tell a UI element how to **lay out**, **display** or **behave** within its parent layout. You can also say they **decorate** or **add behavior** to UI elements.

In the code in **KodecochatAppBar.kt**, you see `modifier` used repeatedly. Most often, it’s an attribute on a composable function that’s then passed to a nested composable within it. This is a common practice you’ll see a lot in Compose, and not just with modifiers. Sometimes, you’ll add an attribute to a composable not because you need to use it in that composable directly but because you want to pass it on to another composable further down the line in the nesting hierarchy. This isn’t always the best practice, though — especially if you’re trying to pass data to the UI. In the next chapter, you’ll learn about **ViewModels** and how to use them to properly pass data unidirectionally to your UI.

Replace the body of `SimpleUserInput()` with this code, which is based on code you wrote earlier but now is in a separate composable:

```kotlin
@Composable
fun SimpleUserInput() {
  val context = LocalContext.current
  var chatInputText by remember { mutableStateOf("") }
  var chatOutputText by remember { mutableStateOf(context.getString(R.string.chat_display_default)) }
  Text(text = chatOutputText)
  Row {
    OutlinedTextField(
      value = chatInputText,
      placeholder = { Text(text = stringResource(id = R.string.chat_entry_default)) },
      onValueChange = {
        chatInputText = it
      },
    )
    Button(onClick = {
      chatOutputText = chatInputText
      chatInputText = ""
    }) {
      Text(text = stringResource(id = R.string.send_button))
    }
  }
}
```

Build and run the app. Hmm, this doesn’t look quite right…

 ![](https://assets.alexandria.kodeco.com/books/9f59dbccde22edd2ab5693b6e6b2b51c5d4fe4261fa1a6477dd65d3d2a9028de/images/0c4399fbaee09fa3664c703cdeb5c972/original.png)

It looks like the top app bar overlaps the text field and button! Use some modifiers to fix the layout. In `ConversationContent()` add a modifier to the `Box` :

```kotlin
Surface {
   Box(modifier = Modifier.fillMaxSize()) {...
```

Run the app and observe the changes. You might miss the subtle color change indicating the box’s expanded area. To enhance visibility, add another modifier to the `Box` as follows:

```kotlin
Box(
  modifier = Modifier
    .fillMaxSize()
    .background(color = Color.DarkGray)
) {...
```

Build and run. Now, the difference is clear:

 ![](https://assets.alexandria.kodeco.com/books/9f59dbccde22edd2ab5693b6e6b2b51c5d4fe4261fa1a6477dd65d3d2a9028de/images/d874e78a2314e0034399a42debd2b565/original.png)

Great, the `Box` has definitely expanded to fill the entire screen, but the text entry portion of the UI is still covered. Update the contents of the `Surface()` as follows:

```kotlin
Surface {
  Box(modifier = Modifier.fillMaxSize()) {
    Column(
      Modifier
        .fillMaxSize()
     ) {
       Messages(
          modifier = Modifier.weight(1f),
        )
        SimpleUserInput()
      }
      // Channel name bar floats above the messages
      ChannelNameBar(channelName = "Android Apprentice")
    }
  }
```

Then, update the definition of `Message()` as follows:

```kotlin
@Composable
fun Messages(modifier: Modifier = Modifier){
  Box(modifier = modifier) {
    // TODO: implement this part in the next section!
  }
}
```

Build and run. Now, the layout looks much better!

 ![](https://assets.alexandria.kodeco.com/books/9f59dbccde22edd2ab5693b6e6b2b51c5d4fe4261fa1a6477dd65d3d2a9028de/images/225b1144dbf72a254506b7f3c234b471/original.png)

You’ve already seen what `fillMaxSize` does, but what about `weight`? When Compose lays out a composable’s children, it measures them and then distributes and sizes them according to those measurements in the order the children are listed. The measurements are affected by what they contain and by modifiers. The `weight` modifier takes a float value and sizes the element’s height according to the weight values of the other children in the `Column`, the parent container in this case. The parent divides the vertical space remaining after measuring unweighted child elements and distributes it according to this weight. In this case, it gives most of the space to the `Box` of `Messages` because everything else in the column is unweighted.

## Lists

What happens when you must display more elements than you can fit on the screen? In that case, although the elements are all composed, the limited screen size prevents you from seeing them all. There are even situations when you want to dynamically add new elements on the screen and still be able to see them all, like in a chat app!

The solution to this problem is allowing your content to scroll, either vertically or horizontally. Jetpack Compose gives a way to build one of the most common UI components mobile apps use — using scrollable and lazily composed containers, a.k.a. the `List`.

Loading data only when it’s needed is called **lazy loading**, and Jetpack Compose uses this method to handle lists. The main two components you use for lazy lists in Compose are the `LazyColumn` and `LazyRow`.

Update `Messages()` as follows:

```kotlin
@Composable
fun Messages(
  messages: List<String>,
//  scrollState: LazyListState,
  modifier: Modifier = Modifier
) {
  Box(modifier = modifier) {
    LazyColumn(
      // Add content padding so that the content can be scrolled (y-axis)
      // below the status bar + app bar
      contentPadding =
      WindowInsets.statusBars.add(WindowInsets(top = 90.dp)).asPaddingValues(),
      modifier = Modifier
        .fillMaxSize()
    ) {
      item {
        Text(text = "First message")
      }
      item {
        Text(text = "Second message")
      }
      item {
        Text(text = "Third message")
      }
    }
  }
}
```

You’ve added a `LazyColumn` and hard-coded a few dummy chat messages.

Build and run:

 ![](https://assets.alexandria.kodeco.com/books/9f59dbccde22edd2ab5693b6e6b2b51c5d4fe4261fa1a6477dd65d3d2a9028de/images/1042ee3168c3844173f676f806ed191f/original.png)

Although the current approach of hard-coding messages provides a basic foundation, you need a more dynamic solution to enhance the chat functionality. Ideally, messages entered into the text box should seamlessly be added to the existing list. Additionally, you need to know who sent the message and when it was sent. Further, distinguishing your messages from those of others through visual styling would significantly improve readability and user experience.

## Building the Message UI

First, you should define what a `Message` actually is. Create a new package inside **com.kodeco.chat** , the **data.model**. Then, copy **DateExtensions.kt**, **MessageUiModel.kt** and **User.kt** from the Final project for this chapter from the same location to this one in your project.

Add the following dependency to your module-level Gradle file, then sync the project with Gradle:

```kotlin
// Date Time Library - the latest way to handle dates in Kotlin
  implementation("org.jetbrains.kotlinx:kotlinx-datetime:0.4.0")
```

Create another package named **utilities** and copy over **Extensions.kt** from the final project.

Copy **MessageFormatter.kt** from the **conversation** package in the final project to the same location in your project. This is a utility class to handle the formatting of the text in the messages.

You won’t enable adding new messages to the message list until the next chapter, when you learn about `ViewModel`. So in the meantime, you need some placeholder data to see how your Message UI will look. From the **data** package in the final project, copy **FakeData.kt** to the same location in your project.

Finally, you’ll need some graphic assets. From **res -> drawable** in the Final project for this chapter, copy **profile\_photo\_android\_developer.png** and **someone\_else.png** to the same location in your project.

In the **conversation** package, create a new Kotlin class and name it **ConversationUiState.kt**. Replace the contents with the following:

```kotlin
class ConversationUiState(
  val channelName: String,
  initialMessages: List<MessageUiModel>,
) {
  private val _messages: MutableList<MessageUiModel> = initialMessages.toMutableStateList()

  val messages: List<MessageUiModel> = _messages

  fun addMessage(msg: String, photoUri: Uri?) {
    // TODO: implement in Chapter 6 😀
  }
}

@Immutable
data class Message(
  val _id: String = UUID.randomUUID().toString(),
  val createdOn: Instant? = Clock.System.now(),
  val roomId: String = "public", // "public" is the roomID for the default public chat room
  val text: String = "test",
  val userId: String = UUID.randomUUID().toString(),
  val photoUri: Uri? = null,
  val authorImage: Int = if (userId == "me") R.drawable.profile_photo_android_developer else R.drawable.someone_else
)
```

You’ll learn more about state in Compose in the next chapter. For now, focus on the second half of the code in this class, which defines a Kotlin data class, `Message()`. It has all the properties a chat message might have, along with default values. Some properties, such as `photoUri`, are optional and therefore defined as *nullable*; a chat message might not always contain an image attachment.

Change the signature of `ConversationContent` to accept a parameter: `fun ConversationContent(uiState: ConversationUiState) {...`

Then, still in `ConversationContent`, update the `Messages` block to pass in the dummy messages:

```kotlin
Messages(
  messages = uiState.messages,
  modifier = Modifier.weight(1f)
)
```

Next, update the definition of `Messages()` to accept a list of `MessageUiModel` instead of a list of `String`:

```kotlin
@Composable
fun Messages(
  messages: List<MessageUiModel>,
  modifier: Modifier = Modifier
) {
  Box(modifier = modifier) {
    LazyColumn(
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
          authorId = "me",
          userId = userId ?: "",
          isFirstMessageByAuthor = isFirstMessageByAuthor,
          isLastMessageByAuthor = isLastMessageByAuthor,
        )
      }
    }
  }
}
```

Instead of a hard-coded list, this composable now renders messages based on `MessageUiModel`, a complex object comprising a `Message`, a `User` and a unique `id` for each message.

There’s also some logic so the profile image, date and user’s name are rendered only once if the same user sends multiple text messages in a row. Last, add the following composables:

```kotlin
@Composable
fun MessageUi(
  onAuthorClick: (String) -> Unit,
  msg: MessageUiModel,
  authorId: String,
  userId: String,
  isFirstMessageByAuthor: Boolean,
  isLastMessageByAuthor: Boolean,
) {
  val isUserMe = userId == "me" // hard coded for now, next chapter will be = authorId == userId
  val borderColor = if (isUserMe) {
    MaterialTheme.colorScheme.primary
  } else {
    MaterialTheme.colorScheme.tertiary
  }

  val authorImageId: Int = if (isUserMe) R.drawable.profile_photo_android_developer else R.drawable.someone_else
  val spaceBetweenAuthors = if (isLastMessageByAuthor) Modifier.padding(top = 8.dp) else Modifier
  Row(modifier = spaceBetweenAuthors) {
    if (isLastMessageByAuthor) {
      // Avatar
      Image(
        modifier = Modifier
          .clickable(onClick = { onAuthorClick(msg.message.userId) })
          .padding(horizontal = 16.dp)
          .size(42.dp)
          .border(1.5.dp, borderColor, CircleShape)
          .border(3.dp, MaterialTheme.colorScheme.surface, CircleShape)
          .clip(CircleShape)
          .align(Alignment.Top),
        painter = painterResource(id = authorImageId),
        contentScale = ContentScale.Crop,
        contentDescription = null
      )
    } else {
      // Space under avatar
      Spacer(modifier = Modifier.width(74.dp))
    }
    AuthorAndTextMessage(
      msg = msg,
      isUserMe = isUserMe,
      isFirstMessageByAuthor = isFirstMessageByAuthor,
      isLastMessageByAuthor = isLastMessageByAuthor,
      authorClicked = onAuthorClick,
      modifier = Modifier
        .padding(end = 16.dp)
        .weight(1f)
    )
  }
}

@Composable
fun AuthorAndTextMessage(
  msg: MessageUiModel,
  isUserMe: Boolean,
  isFirstMessageByAuthor: Boolean,
  isLastMessageByAuthor: Boolean,
  authorClicked: (String) -> Unit,
  modifier: Modifier = Modifier
) {
  Column(modifier = modifier) {
    if (isLastMessageByAuthor) {
      AuthorNameTimestamp(msg, isUserMe)
    }
    ChatItemBubble(
      msg.message,
      isUserMe,
      authorClicked = authorClicked)
    if (isFirstMessageByAuthor) {
      // Last bubble before next author
      Spacer(modifier = Modifier.height(8.dp))
    } else {
      // Between bubbles
      Spacer(modifier = Modifier.height(4.dp))
    }
  }
}

@Composable
private fun AuthorNameTimestamp(msg: MessageUiModel, isUserMe: Boolean = false) {
  var userFullName: String = msg.user.fullName
  if (isUserMe) {
    userFullName = "me"
  }

  // Combine author and timestamp for author.
  Row(modifier = Modifier.semantics(mergeDescendants = true) {}) {
    Text(
      text = userFullName,
      style = MaterialTheme.typography.titleMedium,
      modifier = Modifier
        .alignBy(LastBaseline)
        .paddingFrom(LastBaseline, after = 8.dp) // Space to 1st bubble
    )
    Spacer(modifier = Modifier.width(8.dp))
    Text(
      text = msg.message.createdOn.toString().isoToTimeAgo(),
      style = MaterialTheme.typography.bodySmall,
      modifier = Modifier.alignBy(LastBaseline),
      color = MaterialTheme.colorScheme.onSurfaceVariant
    )
  }
}

@Composable
fun ChatItemBubble(
  message: Message,
  isUserMe: Boolean,
  authorClicked: (String) -> Unit
) {
  val ChatBubbleShape = RoundedCornerShape(4.dp, 20.dp, 20.dp, 20.dp)
  val pressedState = remember { mutableStateOf(false) }
  val backgroundBubbleColor = if (isUserMe) {
    MaterialTheme.colorScheme.primary
  } else {
    MaterialTheme.colorScheme.surfaceVariant
  }
  Column {
    Surface(
      color = backgroundBubbleColor,
      shape = ChatBubbleShape
    ) {
      if (message.text.isNotEmpty()) {
        ClickableMessage(
          message = message,
          isUserMe = isUserMe,
          authorClicked = authorClicked
        )
      }
    }
  }
}

@Composable
fun ClickableMessage(
  message: Message,
  isUserMe: Boolean,
  authorClicked: (String) -> Unit
) {
  val uriHandler = LocalUriHandler.current
  val styledMessage = messageFormatter(
    text = message.text,
    primary = isUserMe
  )

  ClickableText(
    text = styledMessage,
    style = MaterialTheme.typography.bodyLarge.copy(color = LocalContentColor.current),
    modifier = Modifier.padding(16.dp),
    onClick = {
      styledMessage
        .getStringAnnotations(start = it, end = it)
        .firstOrNull()
        ?.let { annotation ->
          when (annotation.tag) {
            SymbolAnnotationType.LINK.name -> uriHandler.openUri(annotation.item)
            SymbolAnnotationType.PERSON.name -> authorClicked(annotation.item)
            else -> Unit
          }
        }
    }
  )
}
```

It might seem like a lot of code. But it’s just composables that define each part of the message UI and handle interactions with the messages. By breaking it down into many small composables, you can individually address minute details of the UI.

Last, in **MainActivity.kt**, update the call to `ConversationContent` to include the new parameter you added and supply the dummy chat data:

```kotlin
setContent {
  ConversationContent(
    uiState = exampleUiState
  )
}
```

Build and run the app.

 ![](https://assets.alexandria.kodeco.com/books/9f59dbccde22edd2ab5693b6e6b2b51c5d4fe4261fa1a6477dd65d3d2a9028de/images/870052abc1d0ba30390007a854d990ab/original.png)

Wow, this is starting to look like a real chat app now! Tap the hyperlink in the first chat message. The composable code correctly handles it by opening the link in a browser window. Cool beans!

## Themes & Working With Fonts

In Compose, you can use the downloadable fonts API in your Compose app to download [Google fonts](https://fonts.google.com/) asynchronously and use them in your app. For step-by-step details on using Google fonts, see the [official documentation](https://developer.android.com/jetpack/compose/text/fonts#downloadable-fonts). You can define a theme in your app and then use your custom fonts, including Google fonts. Once you’ve done that, wrapping your UI in the theme in the `setContent` is a simple step. To demonstrate, add the dependency for Google fonts to your Module level gradle file:

`implementation("androidx.compose.ui:ui-text-google-fonts:1.5.4")`

Then, from the final project for this chapter, copy to your project:

+ **res -> font** and all the font files therein
+ **com.kodeco.chat -> theme** - **Color.kt**, **Themes.kt**, **Typography.kt**

Finally, in **MainActivty.kt**, update the call to `setContent`:

```kotlin
setContent {
  KodecochatTheme {
     ConversationContent(
       uiState = exampleUiState
     )
  }
}
```

All you do is wrap everything inside `KodecochatTheme{}`.

Build and run.

 ![](https://assets.alexandria.kodeco.com/books/9f59dbccde22edd2ab5693b6e6b2b51c5d4fe4261fa1a6477dd65d3d2a9028de/images/28115af486e524de590118d9090a6d44/original.png)

Voilà! It has a completely different look, with fonts and colors dictated by the new theme.

## Updating the User Input Field

One last touch and you’re done for now! In **Conversation.kt**, delete the call to `SimpleUserInput` in `ConversationContent` and then delete that composable’s definition. Replace it with this:

`UserInput(onMessageSent = {})`

Build and run.

 ![](https://assets.alexandria.kodeco.com/books/9f59dbccde22edd2ab5693b6e6b2b51c5d4fe4261fa1a6477dd65d3d2a9028de/images/87b44d9fff70222c37ac8869576e9933/original.png)

This just provides a cleaner input interface. Play around with typing in the text input area and look at the source code to see how it’s put together. Last, here it is in dark mode, thanks to the theme you implemented:

 ![](https://assets.alexandria.kodeco.com/books/9f59dbccde22edd2ab5693b6e6b2b51c5d4fe4261fa1a6477dd65d3d2a9028de/images/0b2dd83118cfe85e517c0407ab900705/original.png)

Pretty slick!

## Key Points

Well done! Congratulations on getting this far! This was a packed chapter, and you had a whirlwind adventure with Compose UI! In this chapter, you learned:

+ The fundamentals of Compose functions, composables, and how to nest composables and build complex layouts from smaller composable building blocks.
+ More about Android Studio and some of the powerful and convenient features it offers to make your development life easier.
+ How to lay out your UI using Layout Groups.
+ How to render previews of your UI without the need to build or run anything.
+ How to affect the way the UI is rendered using Modifiers.
+ How to implement custom Themes and work with downloadable fonts.

## Where to Go From Here?

To learn more about Jetpack Compose fundamentals, see the official Android documentation at [https://developer.android.com/jetpack/compose/documentation](https://developer.android.com/jetpack/compose/documentation). In the next chapter, you’ll dive even deeper into Compose UI, leveraging some very powerful design patterns that will take your code to the next level. You’ll also continue to develop the Kodeco Chat app!
