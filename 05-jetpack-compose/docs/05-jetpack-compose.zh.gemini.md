# 5. Jetpack Compose

传统上，Android 应用程序依赖于基于 XML 的布局。如今，Android 开发已转向使用 Compose 作为标准框架。Compose 描述了一套更庞大的基于 Kotlin 的框架和应用架构，它并非 Android 开发所特有。例如，Slack 的开发团队基于 Compose 构建了 [Circuit](https://slackhq.github.io/circuit/) 框架。Compose UI 只是七个 Compose 框架中专门针对应用程序开发 UI 层的框架之一。使用 Compose UI 相较于旧的 View 实现带来了许多改进，包括显著减少构建时间、APK 大小和运行时性能。它还使得构建 UI 更简单、更直观，并且更易于维护和调试。有关 Compose 和旧 Android View 实现的更深入比较，请参阅[这篇文章](https://developer.android.com/jetpack/compose/migrate/compare-performance)。

现代 Android 开发已从 MVVM (Model-View-Viewmodel) 转向 **MVI** (Model-View-Intent) 架构，现在使用 Compose UI 而不是 XML 布局来构建 UI 层。MVI 背后的关键概念之一是单向数据流（unidirectional data flow）。通常，你可能有多个数据源，包括本地存储和网络源。这些通常使用 **Repository** 模式进行抽象和访问。ViewModel 访问 Repository 上的方法，并使用 **Flow** 向 UI 提供单向数据。Flow 通常由一个数据发射器（emitter）和该发射器的订阅者（subscribers）组成。它提供了一种高效的内存使用模型，因为只有当存在活跃的订阅者使用来自 Flow 的数据时，Flow 才会消耗内存。对于 Android，有特定的 Flow 实现能够感知 Android 生命周期。因此，当视图对用户不再可见时，数据将不再被消耗，从而释放内存。

在本章中，你将学习 Compose UI 的基础知识，并用它构建一个简单的 Android 应用 UI。在下一章中，你将深入学习将 Compose 应用连接起来的其他重要部分，包括 ViewModel、MVI、Repository 模式和导航（Navigation）。

## Compose 基础

如果你一直跟着前几章学习，请在 Android Studio 中打开你的 Kodeco Chat 应用。否则，请使用 Android Studio 打开本章的 **starter** 项目，并选择 **Open an existing project**。接下来，导航到 **05-jetpack-compose/projects** 并选择 **starter** 文件夹作为项目根目录。项目打开后，让它完成构建和同步，然后你就可以开始了！

回想一下，在第 2 章中，你从头创建了一个新的 Android 项目。为你创建的默认 Activity 包含以下函数：

```kotlin
@Composable
fun Greeting(name: String, modifier: Modifier = Modifier) {
  Text(
    text = "Hello $name!",
    modifier = modifier
  )
}
```

这段代码有两点值得注意。首先，它是一个函数；其次，它使用了 `@Composable` 注解。这就是在 Compose 中创建 UI 组件所需的全部内容，或者用 Compose 的术语来说，就是一个**可组合函数（composable function）**。

在你的项目中，打开 **MainActivity.kt**。浏览代码——`@Composable` 注解在哪里？

```kotlin
class MainActivity : ComponentActivity() {
  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContent {
      Column { ...
```

它似乎没有出现在任何地方……

但再仔细看看：MainActivity 是 **ComponentActivity** 的子类。在 `onCreate()` 函数中，你调用了 `setContent`。

按住 Command 键（Mac）或 Control 键（PC）并单击 `setContent`。Android Studio 会打开这个函数的定义，它定义在 **ComponentActivity.kt** 中：

```kotlin
public fun ComponentActivity.setContent(
  parent: CompositionContext? = null,
  content: @Composable () -> Unit
) {...}
```

啊哈！所以，首先，`setContent` 是 `ComponentActivity` 的一个扩展函数（extension function）。扩展函数可以在不改变类源代码的情况下向类添加额外的功能。调用 `setContent()` 会将名为 `content` 的给定可组合函数设置为根视图，你可以在其中添加任意数量的元素。你从这个容器内部调用其余的可组合函数。

其次，注意 `content` 也被 `@Composable` 注解了。你不需要在放入像上面的 `Column` 这样的可组合项之前再次添加它，因为注解已经在这里了。

在项目导航器中右键单击 **com.kodeco.chat**，并从上下文菜单中选择 **New -> Package**：

 ![](https://assets.alexandria.kodeco.com/books/9f59dbccde22edd2ab5693b6e6b2b51c5d4fe4261fa1a6477dd65d3d2a9028de/images/ef0eab801d90527c96c8da354263c6b7/original.png)

将新包命名为 “conversation”。你将在这里创建构成聊天 UI 各个部分（pieces）的组件。

接下来，右键单击 **conversation** 包并选择 **New -> Kotlin Class/File**：

 ![](https://assets.alexandria.kodeco.com/books/9f59dbccde22edd2ab5693b6e6b2b51c5d4fe4261fa1a6477dd65d3d2a9028de/images/ebbab1fb765cfa5809f22881486b05d5/original.png)

确保选择了 “file” 并将新文件命名为 “Conversation”：

 ![](https://assets.alexandria.kodeco.com/books/9f59dbccde22edd2ab5693b6e6b2b51c5d4fe4261fa1a6477dd65d3d2a9028de/images/de9835ae6b08d1e31353ecfea39af674/original.png)

Android Studio 会在 **conversation** 包中创建一个名为 **Conversation.kt** 的空 Kotlin 文件。

在 **Conversation.kt** 中，输入以下内容：

```kotlin
@Composable
fun ConversationContent() {
  // TODO: 在这里创建对话 UI
}
```

恭喜你编写了第一个 Compose 函数！它目前还不做任何事情，但你很快就会改变这一点。

回到 **MainActivity.kt**，复制 `setContent{}` 大括号内的所有内容，并将其粘贴到 `ConversationContent` 的函数体中：

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

然后，回到 **MainActivity.kt** 并将 `setContent{}` 中的所有内容替换为 `ConversationContent()`。你的 Activity 类应该看起来更简单、更清晰：

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

构建并运行你的应用。它应该和以前完全一样工作。

## 分解可组合项

尽管你执行的简单复制/粘贴操作可能看起来微不足道，但它突出了几个关键概念。首先，请注意 **Conversation.kt** 是一个单独的文件，而不是一个类。它只是包含了一个 Compose 函数。这个函数本可以放在 **MainActivity.kt** 中。但是随着代码库的扩展，将所有代码维护在单个文件中会变得很麻烦。其次，你创建了一个名为 `ConversationContent` 的**可组合函数**，你可以在整个应用中重用它。这种可重用性是 Compose UI 的基石，类似于用单个乐高积木构建大型雕塑。

关于可组合函数的一些注意事项：

+ 可组合函数只能从其他可组合函数中调用。
+ 可组合函数可以接收参数并使用它们来构建 UI。
+ 可组合函数只能在 compose 作用域（scope）内调用，就像协程（coroutines）的工作方式一样。

到目前为止，你创建的 UI 展示了各种 Compose UI 元素，每一个都是能够接受各种参数的可组合函数：

+ **Text**: 一个基本的文本元素，用于显示文本并提供无障碍信息。
+ **OutlinedTextField**: 与只显示文本的 Text 不同，文本字段允许用户在 UI 中输入文本。这种类型的文本字段具有弱化的视觉样式。
+ **Button**: 这正如其名；用户点击按钮来启动一个操作。
+ **Column**: 这与你使用过的其他可组合项不同。虽然 Text、TextField 和 Button 都是 UI 元素，但 Column 是一种*布局*可组合项。布局允许你以各种方式排列 UI 元素。对于 Column，它的所有*子项*（children）——它包含的元素——都垂直或水平地排列成一列。你将学习其他几种布局可组合项，并且可以在 Compose 中创建自定义布局（custom layouts）。

要跳转到其类定义，请在你的代码中按住 Command 键（Mac）或 Control 键（PC）并单击这些控件中的每一个。每个控件的代码上方都有详细的注释、可自定义的参数以及示例代码的链接。

Compose 使用声明式 UI 方法：你使用可组合函数声明关于 UI 外观的所有内容。

再次查看 `ConversationContent` 中 Button 的代码：

```kotlin
Button(onClick = {
 chatOutputText = chatInputText
 chatInputText = ""
}) {
 Text(text = stringResource(id = R.string.send_button))
}
```

`Button()` 可组合项具有你所学到的可组合函数的所有特性。它接受一个参数 `onClick()`，该参数本身就是一个函数。在 Kotlin 中，函数可以接受其他函数或 *lambda 表达式*作为参数。这里的 `onClick()` 函数是内联定义的，它定义了按钮被点击时将发生的操作。`Button()` 函数的主体包含一个 `Text()`，这是按钮的标签。

`Button()` 可组合项是一个基础按钮类，具有高度可定制性。但是，当你不需要太多定制时，可以使用 `Button()` 的五个子类类型。有关不同按钮类型以及在何时何地使用它们的更多信息，请参阅 `Button()` 的[官方文档](https://developer.android.com/jetpack/compose/components/button)。

你可能已经注意到可组合函数使用**帕斯卡命名法（Pascal case）**，这与 Kotlin 代码通常使用的驼峰命名法（camel case）不同。因此，你定义的顶级可组合项是 `ConversationContent` 而不是 `conversationContent`。这种区别源于可组合函数返回 UI 对象，因此采用了与类相同的命名约定。

## 改进 UI 和 UX

你可能已经注意到，在文本字段中开始输入时必须手动删除占位符文本的不便之处。这个不必要的步骤打断了用户的流程，并引入了额外的思维障碍，阻碍了他们与应用的交互。任何有损 UI 无缝性和直观性的东西都可能导致糟糕的用户体验（user experience）。

在 **Conversation.kt** 中，按住 Command 键（Mac）或 Control 键（PC）并单击 `OutlinedTextField` 以跳转到其类定义。在你的应用开发生涯中，通过研究框架的源代码，你将对事物的工作原理有更深入的理解。这将带来实用的知识，真正帮助你提升专业水平。

注意可以传递给 `OutlinedTextField` 的众多参数：

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

另外，请注意 `OutlinedTextField` 定义上方的注释块中提供的解释。

1. `@OptIn(ExperimentalMaterial3Api::class)`: 你经常会在 Compose 和其他框架中看到这个 `OptIn` 后面跟着 “Experimental”。虽然这些框架现在已经成熟，但它们仍在不断发展。因此，要访问一些新功能，有时你需要“选择加入（opt in）”以使用实验性功能。别担心——Android Studio 通常会在需要时提示你添加此注解，并在该功能不再是实验性且应删除该注解时向你发出警告。
2. 再次，按照要求，你看到 `@Composable` 注解将此标记为可组合项。
3. `label` 参数，当前设置为“Enter Chat Text”，根据代码注释，旨在文本字段容器中显示一个标签。但在你的 UI 中，标签出现在文本字段轮廓的内联位置，偏离了预期的位置。

 ![](https://assets.alexandria.kodeco.com/books/9f59dbccde22edd2ab5693b6e6b2b51c5d4fe4261fa1a6477dd65d3d2a9028de/images/fae714698ef2ce7437d224c93d65a325/original.png)

 “Enter Chat Text” 标签在文本字段中不可见，因为占位符（placeholder）文本——“Type your text here”——目前占据了该空间。移除占位符文本将允许标签按预期出现在文本字段中。

4. 啊哈！这个 `placeholder` 字段是什么？文档说，“当文本字段获得焦点且输入文本为空时显示的可选占位符”。更新你的代码以使用此参数。

回到 **Conversation.kt** 并按如下方式更新 `ConversationContent`：

```kotlin
@Composable
fun ConversationContent() {
  Column {
    val context = LocalContext.current
    var chatInputText by remember { mutableStateOf("") } // 将默认值改为空字符串
    var chatOutputText by remember { mutableStateOf(context.getString(R.string.chat_display_default)) }
    Text(text = chatOutputText)

    OutlinedTextField(
      value = chatInputText,
      placeholder = { Text(text = stringResource(id = R.string.chat_entry_default)) }, // 使用 placeholder
      onValueChange = {
        chatInputText = it
      },
      // label = { Text(text = stringResource(id = R.string.chat_entry_label)) } // 删除了 label
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

你删除了 `chatInputText` 的默认值并将其替换为空字符串，因为你不再需要在那里使用默认文本了。接下来，你将该默认文本传递给 `placeholder` 参数。最后，你删除了 `label`，因为它现在是多余的。

构建并运行，然后尝试在文本字段中输入一些文本。瞧！现在，一旦你开始输入，默认文本就会自动消失。

 ![](https://assets.alexandria.kodeco.com/books/9f59dbccde22edd2ab5693b6e6b2b51c5d4fe4261fa1a6477dd65d3d2a9028de/images/4b49608c9f1cde7ca8b0695bb4de5243/original.png)

通过查看 `OutlinedTextField` 的源代码，你更好地理解了如何使用它。

## 布局组 (Layout Groups)

Compose 中的布局组允许你以各种方式在设备屏幕上排列 UI 元素。你可以直接使用 Compose `Layout()` 类定义布局，也可以使用预定义的布局类型。就像你可以组合可组合项并在彼此内部使用它们一样，你也可以嵌套布局组来制作更复杂的布局。你已经见过一种布局组 `Column`，它允许你垂直布局元素。

要改为水平排列元素，你可以使用 `Row`。

另一个布局可组合项是 `Box`。它用于相对于其父级边缘显示子项（它包含的元素）。它还允许你堆叠或重叠子项。

最后，`Surface` 是一种特殊的布局，通常是一系列嵌套可组合项中的顶级（或根）布局。`Surface` 一次只能有一个子项，但它为其子项提供了许多样式处理。它被用作 **Material Design** 的核心隐喻，Material Design 是 Google 在 Android 中使用的标准设计库，用于在不同设备上提供统一的用户体验。

将 `ConversationContent()` 的主体替换为以下内容：

```kotlin
Surface {
  Box {
    Column {
      Messages()
      SimpleUserInput()
    }
  // 频道名称栏浮动在消息上方
  ChannelNameBar(channelName = "Android Apprentice")
  }
}
```

现在，你正在使用刚刚学到的几个布局可组合项：Surface、Box 和 Column。但是你也添加了对尚不存在的可组合项的引用，导致 Android Studio 向你显示一些错误。

单击以红色显示的可组合项之一，例如 `Messages`。Android Studio 会显示错误消息并提供解决方案。

 ![](https://assets.alexandria.kodeco.com/books/9f59dbccde22edd2ab5693b6e6b2b51c5d4fe4261fa1a6477dd65d3d2a9028de/images/0884119ae0b281d79279bf1f0c27c51d/original.png)

或者你可以单击可组合项左侧的红色灯泡图标以查看相同的选项。

 ![](https://assets.alexandria.kodeco.com/books/9f59dbccde22edd2ab5693b6e6b2b51c5d4fe4261fa1a6477dd65d3d2a9028de/images/36bd683eb454d7403e6913752560bd47/original.png)

### 处理编译问题

为每个未定义的可组合项选择 “Create @Composable function…”。Android Studio 会在文件底部为该可组合项创建一个存根函数（stub function），并在每个函数体中调用 `TODO()`。这个特殊的内联函数如果你尝试运行应用会导致编译错误，强制你在编译应用之前实现该函数。你现在应该在 **Conversation.kt** 的末尾看到以下函数：

```kotlin
@Composable
fun Messages() {
  TODO("Not yet implemented") // 尚未实现
}

@Composable
fun SimpleUserInput() {
  TODO("Not yet implemented") // 尚未实现
}

@Composable
fun ChannelNameBar(channelName: String) {
  // 这里没有 TODO()，因为你已经部分实现了它
}
```

注意 `ChannelNameBar` 没有 `TODO()`。这是因为你已经通过为函数定义参数来部分实现了它。

在 **com.kodeco.chat** 下创建另一个包 **components**，然后将本章最终项目中的 **KodecochatAppBar.kt** 和 **KodecochatIcon.kt** 复制粘贴到你的项目中。然后将 starter 项目的 **res->drawable** 中的 **kodeco_logo.xml** 和 **kodeco_logo_back.xml** 复制粘贴到你项目中的相同位置。最后两个文件包含矢量资源（vector assets），它们可以无像素化地缩放，并在运行时由 Android 渲染。此外，复制 **res -> values** 下的 **Strings.xml** 中的值，以便你可以访问本章中使用的所有本地化字符串。

接下来，将 `ChannelNameBar()` 的主体替换为以下内容：

```kotlin
@Composable
fun ChannelNameBar(channelName: String) {
  KodecochatAppBar(
    title = {
      Column(horizontalAlignment = Alignment.CenterHorizontally) {
        // 频道名称
        Text(
          text = channelName,
          style = MaterialTheme.typography.titleMedium
        )
      }
    },
    actions = {
      // 信息图标
      Icon(
        imageVector = Icons.Outlined.Info,
        tint = MaterialTheme.colorScheme.onSurfaceVariant,
        modifier = Modifier
          .clickable(onClick = { })
          .padding(horizontal = 12.dp, vertical = 16.dp)
          .height(24.dp),
        contentDescription = stringResource(id = R.string.info) // 内容描述：信息
      )
    }
  )
}
```

`KodecochatAppBar` 在 Android Studio 中可能会出现红色下划线。如果是这样，将鼠标悬停在它上面以查看 Android Studio 中的此对话框：

 ![](https://assets.alexandria.kodeco.com/books/9f59dbccde22edd2ab5693b6e6b2b51c5d4fe4261fa1a6477dd65d3d2a9028de/images/9c925dc14580586aae3646c80eb109e0/original.png)

单击 “Opt in…” ，Android Studio 会在函数定义上方添加 `@OptIn(ExperimentalMaterial3Api::class)` 注解，如前所述。

将任何对 `TODO("Not yet implemented")` 的调用替换为常规注释，例如 `// TODO - 实现`。这允许你在有编译问题的情况下构建和运行应用，但 TODO 注释会在 Android Studio 代码窗格的右侧边栏中以蓝色勾号（而不是错误的红色和警告的黄色）突出显示：

 ![](https://assets.alexandria.kodeco.com/books/9f59dbccde22edd2ab5693b6e6b2b51c5d4fe4261fa1a6477dd65d3d2a9028de/images/8f55158dd2067eb7c75252edae41bef8/original.png)

构建并运行。你现在应该在设备屏幕顶部看到一个新的应用栏：

 ![](https://assets.alexandria.kodeco.com/books/9f59dbccde22edd2ab5693b6e6b2b51c5d4fe4261fa1a6477dd65d3d2a9028de/images/b628102653eba5bc9efd3de122d42320/original.png)

按住 Command 键（Mac）或 Control 键（PC）并单击 `KodecochatAppBar` 查看源代码：

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
        contentDescription = stringResource(id = R.string.navigation_drawer_open), // 内容描述：打开导航抽屉
        modifier = Modifier
          .size(64.dp)
          .clickable(onClick = onNavIconPressed)
          .padding(16.dp)
      )
    }
  )
}
```

1. `KodecochatAppBar` 基本上只是内置类 `CenterAlignedTopAppBar` 的一个包装器。这通常用于在屏幕顶部显示信息和操作。你传递给 `ChannelNameBar` 的 `channelName` 参数被用于 `CenterAlignedTopAppBar` 中的 `title` 属性，该属性显示你现在在应用屏幕顶部看到的标题。此外，在 **Conversation.kt** 中，传递给 `title` 的值不仅仅是一个字符串；它是一个完整的可组合函数。这个可组合项由一个 Column 组成，其对齐属性设置为水平居中，其内容设置为捕获标题字符串的 `Text` 字段。通过这种方式，你可以看到 Compose 如何允许你将可组合项嵌套在彼此内部以创建更复杂的布局和功能。
2. `CenterAlignedTopAppBar`（因此 `KodecochatAppBar` 也是）接受一个名为 `actions` 的参数。这通常应该是一个 `IconButton` 列表。这些 Material Design 紧凑型按钮帮助用户执行一些补充操作。在这种情况下，你传递的是一个信息按钮，它看起来像一个带圆圈的“i”。其前提是，你以后可以添加代码，以便在用户点击它时向用户提供有关聊天频道的信息。另请注意，操作在 `Row` 中渲染。
3. `CenterAlignedTopAppBar` 还有另一个参数 `navigationIcon`，你稍后将使用它在你的应用中启用打开侧边菜单。对于图标，你使用 Kodeco 徽标，它以矢量资源的形式提供。这也构建为一个可组合项。再次注意，你正在向此参数传递一个实际的可组合函数。

## 预览 (Previews)

打开 **KodecochatAppBar.kt**。在 Android Studio 顶部，运行和调试图标下方，单击 **split** 视图的图标，这是 Android Studio 中代码和设计视图的组合：

 ![](https://assets.alexandria.kodeco.com/books/9f59dbccde22edd2ab5693b6e6b2b51c5d4fe4261fa1a6477dd65d3d2a9028de/images/d370047e10889bd0c975d728b304294a/original.png)

你应该看到 Android Studio 分成了几个不同的窗格：

 ![](https://assets.alexandria.kodeco.com/books/9f59dbccde22edd2ab5693b6e6b2b51c5d4fe4261fa1a6477dd65d3d2a9028de/images/87154da148dd5c25b8d1257720d9bea9/original.png)

1. 中间是 `KodecochatAppBar` 在设备上浅色和深色模式下的预览。
2. 你，开发者，编写代码来生成这些预览。

```kotlin
@OptIn(ExperimentalMaterial3Api::class)
@Preview
@Composable
fun KodecochatAppBarPreview() {
  KodecochatTheme {
    KodecochatAppBar(title = { Text("Preview!") }) // 预览！
  }
}
```

要在可组合项中创建预览，你只需编写一个 compose 函数，并在 `@Composable` 注解之上添加 `@Preview` 注解。虽然这样做不是必需的，但命名约定是在函数名末尾加上“Preview”以提高可读性。然后你可以定义你的可组合函数所需的参数，Android Studio 将在设计视图中自动渲染 UI 元素。Compose 预览的另一个很酷的功能是，当你编辑 Compose 源代码时，Android Studio 会实时更新它们。

现在试试这个：回到 `KodecochatAppBar()`，将传递给 `KodecoChatIcon` 的 `.padding()` 参数从 16 dp 更改为 3 dp。更改后，你应该会看到左侧的 Kodeco 徽标变得更大。将其改回原始值，它会缩小回去。你无需构建或运行应用即可看到这些更改！这可以使你的应用开发速度快得多，因为你通常可以在不运行或重新构建应用的情况下看到设计更改。你还可以使用预览来查看可组合项在不同设备和不同条件下同时渲染时的外观。在此示例中，通过定义两个预览，你可以同时看到顶部栏在浅色和深色模式下的外观。有关使用可组合预览的更多信息，请参阅 [Android 关于可组合预览的文档](https://developer.android.com/jetpack/compose/tooling/previews)。

## Modifiers

**Modifiers** 告诉 UI 元素如何在父布局内**布局（lay out）**、**显示（display）**或**表现（behave）**。你也可以说它们**装饰（decorate）**或**向 UI 元素添加行为（add behavior）**。

在 **KodecochatAppBar.kt** 的代码中，你看到 `modifier` 被反复使用。通常，它是可组合函数上的一个属性，然后传递给其中嵌套的可组合项。这是你在 Compose 中会经常看到的常见做法，不仅仅是对于 modifiers。有时，你向可组合项添加属性并不是因为你需要直接在那个可组合项中使用它，而是因为你想将它传递给嵌套层次结构中更深层的另一个可组合项。但这并不总是最佳实践——尤其是在你试图将数据传递给 UI 时。在下一章中，你将学习 **ViewModels** 以及如何使用它们正确地将数据单向传递给你的 UI。

将 `SimpleUserInput()` 的主体替换为此代码，该代码基于你之前编写的代码，但现在位于一个单独的可组合项中：

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

构建并运行应用。嗯，这看起来不太对……

 ![](https://assets.alexandria.kodeco.com/books/9f59dbccde22edd2ab5693b6e6b2b51c5d4fe4261fa1a6477dd65d3d2a9028de/images/0c4399fbaee09fa3664c703cdeb5c972/original.png)

看起来顶部应用栏覆盖了文本字段和按钮！使用一些 modifiers 来修复布局。在 `ConversationContent()` 中为 `Box` 添加一个 modifier：

```kotlin
Surface {
   Box(modifier = Modifier.fillMaxSize()) { // 填充最大尺寸
     // ...
   }
}
```

运行应用并观察变化。你可能会错过指示 Box 扩展区域的细微颜色变化。为了增强可见性，按如下方式向 `Box` 添加另一个 modifier：

```kotlin
Box(
  modifier = Modifier
    .fillMaxSize() // 填充最大尺寸
    .background(color = Color.DarkGray) // 背景色为深灰色
) {
  // ...
}
```

构建并运行。现在，区别很明显：

 ![](https://assets.alexandria.kodeco.com/books/9f59dbccde22edd2ab5693b6e6b2b51c5d4fe4261fa1a6477dd65d3d2a9028de/images/d874e78a2314e0034399a42debd2b565/original.png)

很好，`Box` 确实已经扩展到填满整个屏幕，但是文本输入部分的 UI 仍然被覆盖。按如下方式更新 `Surface()` 的内容：

```kotlin
Surface {
  Box(modifier = Modifier.fillMaxSize()) {
    Column(
      Modifier
        .fillMaxSize() // 填充最大尺寸
     ) {
       Messages(
          modifier = Modifier.weight(1f), // 设置权重为 1
        )
        SimpleUserInput()
      }
      // 频道名称栏浮动在消息上方
      ChannelNameBar(channelName = "Android Apprentice")
    }
  }
```

然后，按如下方式更新 `Messages()` 的定义：

```kotlin
@Composable
fun Messages(modifier: Modifier = Modifier){
  Box(modifier = modifier) {
    // TODO: 在下一节实现这部分！
  }
}
```

构建并运行。现在，布局看起来好多了！

 ![](https://assets.alexandria.kodeco.com/books/9f59dbccde22edd2ab5693b6e6b2b51c5d4fe4261fa1a6477dd65d3d2a9028de/images/225b1144dbf72a254506b7f3c234b471/original.png)

你已经看到了 `fillMaxSize` 的作用，但是 `weight` 呢？当 Compose 布局可组合项的子项时，它会测量它们，然后按照子项列出的顺序根据这些测量值来分布和调整它们的大小。测量值受它们包含的内容和 modifiers 的影响。`weight` modifier 接受一个浮点值，并根据 `Column`（本例中的父容器）中其他子项的权重值来调整元素的高度。父级在测量未加权子元素后划分剩余的垂直空间，并根据此权重进行分配。在这种情况下，它将大部分空间分配给了 `Messages` 的 `Box`，因为列中的其他所有内容都是未加权的。

## 列表 (Lists)

当你必须显示比屏幕能容纳的更多元素时会发生什么？在这种情况下，尽管所有元素都已组合（composed），但有限的屏幕尺寸使你无法看到所有元素。甚至在某些情况下，你想动态地在屏幕上添加新元素并且仍然能够看到所有元素，就像在聊天应用中一样！

解决这个问题的方法是允许你的内容滚动，无论是垂直还是水平。Jetpack Compose 提供了一种构建移动应用最常用 UI 组件之一的方法——使用可滚动和懒加载（lazily composed）的容器，也就是 `List`。

仅在需要时加载数据称为**懒加载（lazy loading）**，Jetpack Compose 使用此方法处理列表。在 Compose 中用于懒加载列表的两个主要组件是 `LazyColumn` 和 `LazyRow`。

按如下方式更新 `Messages()`：

```kotlin
@Composable
fun Messages(
  messages: List<String>,
//  scrollState: LazyListState, // 滚动状态，暂时注释掉
  modifier: Modifier = Modifier
) {
  Box(modifier = modifier) {
    LazyColumn(
      // 添加内容内边距，以便内容可以在状态栏 + 应用栏下方滚动（y 轴）
      contentPadding =
      WindowInsets.statusBars.add(WindowInsets(top = 90.dp)).asPaddingValues(),
      modifier = Modifier
        .fillMaxSize() // 填充最大尺寸
    ) {
      item {
        Text(text = "First message") // 第一条消息
      }
      item {
        Text(text = "Second message") // 第二条消息
      }
      item {
        Text(text = "Third message") // 第三条消息
      }
    }
  }
}
```

你添加了一个 `LazyColumn` 并硬编码了一些虚拟聊天消息。

构建并运行：

 ![](https://assets.alexandria.kodeco.com/books/9f59dbccde22edd2ab5693b6e6b2b51c5d4fe4261fa1a6477dd65d3d2a9028de/images/1042ee3168c3844173f676f806ed191f/original.png)

虽然当前硬编码消息的方法提供了一个基本的基础，但你需要一个更动态的解决方案来增强聊天功能。理想情况下，输入到文本框中的消息应该无缝地添加到现有列表中。此外，你需要知道是谁发送了消息以及何时发送的。此外，通过视觉样式区分你的消息和他人的消息将显著提高可读性和用户体验。

## 构建消息 UI

首先，你应该定义 `Message` 到底是什么。在 **com.kodeco.chat** 内创建一个新包 **data.model**。然后，将本章最终项目相同位置的 **DateExtensions.kt**、**MessageUiModel.kt** 和 **User.kt** 复制到你项目中的这个位置。

将以下依赖项添加到你的模块级 Gradle 文件中，然后将项目与 Gradle 同步：

```kotlin
// 日期时间库 - Kotlin 中处理日期的最新方式
implementation("org.jetbrains.kotlinx:kotlinx-datetime:0.4.0")
```

创建另一个名为 **utilities** 的包，并从最终项目中复制 **Extensions.kt**。

从最终项目的 **conversation** 包中将 **MessageFormatter.kt** 复制到你项目中的相同位置。这是一个用于处理消息中文本格式化的实用工具类。

在下一章学习 `ViewModel` 之前，你不会启用向消息列表添加新消息的功能。因此，在此期间，你需要一些占位符数据来看看你的消息 UI 会是什么样子。从最终项目的 **data** 包中，将 **FakeData.kt** 复制到你项目中的相同位置。

最后，你需要一些图形资源。从本章最终项目的 **res -> drawable** 中，将 **profile_photo_android_developer.png** 和 **someone_else.png** 复制到你项目中的相同位置。

在 **conversation** 包中，创建一个新的 Kotlin 类并将其命名为 **ConversationUiState.kt**。将其内容替换为以下内容：

```kotlin
class ConversationUiState(
  val channelName: String,
  initialMessages: List<MessageUiModel>,
) {
  private val _messages: MutableList<MessageUiModel> = initialMessages.toMutableStateList()

  val messages: List<MessageUiModel> = _messages

  fun addMessage(msg: String, photoUri: Uri?) {
    // TODO: 在第 6 章实现 😀
  }
}

@Immutable
data class Message(
  val _id: String = UUID.randomUUID().toString(),
  val createdOn: Instant? = Clock.System.now(),
  val roomId: String = "public", // "public" 是默认公共聊天室的 roomID
  val text: String = "test",
  val userId: String = UUID.randomUUID().toString(),
  val photoUri: Uri? = null,
  val authorImage: Int = if (userId == "me") R.drawable.profile_photo_android_developer else R.drawable.someone_else
)
```

你将在下一章学习更多关于 Compose 中的状态（State）。现在，请关注此类代码的后半部分，它定义了一个 Kotlin 数据类 `Message()`。它具有聊天消息可能拥有的所有属性以及默认值。某些属性，例如 `photoUri`，是可选的，因此定义为*可空（nullable）*；聊天消息可能并不总是包含图像附件。

更改 `ConversationContent` 的签名以接受一个参数：`fun ConversationContent(uiState: ConversationUiState) {...`

然后，仍然在 `ConversationContent` 中，更新 `Messages` 块以传入虚拟消息：

```kotlin
Messages(
  messages = uiState.messages,
  modifier = Modifier.weight(1f)
)
```

接下来，更新 `Messages()` 的定义以接受 `MessageUiModel` 列表而不是 `String` 列表：

```kotlin
@Composable
fun Messages(
  messages: List<MessageUiModel>,
  modifier: Modifier = Modifier
) {
  Box(modifier = modifier) {
    LazyColumn(
      // 添加内容内边距，以便内容可以在状态栏 + 应用栏下方滚动（y 轴）
      contentPadding =
      WindowInsets.statusBars.add(WindowInsets(top = 90.dp)).asPaddingValues(),
      modifier = Modifier
        .fillMaxSize() // 填充最大尺寸
    ) {
      itemsIndexed(
        items = messages,
        key= { _, message -> message.id } // 使用 message ID 作为 key
      ) { index, content ->
        // 获取上一个和下一个消息的作者 ID，以及当前消息的作者 ID
        val prevAuthor = messages.getOrNull(index - 1)?.message?.userId
        val nextAuthor = messages.getOrNull(index + 1)?.message?.userId
        val userId = messages.getOrNull(index)?.message?.userId
        // 判断是否是同一作者的第一条或最后一条连续消息
        val isFirstMessageByAuthor = prevAuthor != content.message.userId
        val isLastMessageByAuthor = nextAuthor != content.message.userId
        MessageUi(
          onAuthorClick = { /* 点击作者头像的回调 */ },
          msg = content,
          authorId = "me", // 暂时硬编码为 "me"，下一章会修改
          userId = userId ?: "", // 当前消息的作者 ID
          isFirstMessageByAuthor = isFirstMessageByAuthor,
          isLastMessageByAuthor = isLastMessageByAuthor,
        )
      }
    }
  }
}
```

这个可组合项现在不再基于硬编码列表，而是根据 `MessageUiModel` 渲染消息，`MessageUiModel` 是一个复杂的对象，包含一个 `Message`、一个 `User` 以及每条消息的唯一 `id`。

还有一些逻辑，以便如果同一用户连续发送多条文本消息，则个人资料图片、日期和用户名只渲染一次。最后，添加以下可组合项：

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
  val isUserMe = userId == "me" // 暂时硬编码，下一章将是 = authorId == userId
  val borderColor = if (isUserMe) {
    MaterialTheme.colorScheme.primary // 如果是当前用户，使用主色调
  } else {
    MaterialTheme.colorScheme.tertiary // 否则使用第三色调
  }

  // 作者头像资源 ID
  val authorImageId: Int = if (isUserMe) R.drawable.profile_photo_android_developer else R.drawable.someone_else
  // 如果是作者的最后一条消息，添加顶部内边距
  val spaceBetweenAuthors = if (isLastMessageByAuthor) Modifier.padding(top = 8.dp) else Modifier
  Row(modifier = spaceBetweenAuthors) {
    if (isLastMessageByAuthor) {
      // 头像
      Image(
        modifier = Modifier
          .clickable(onClick = { onAuthorClick(msg.message.userId) }) // 点击头像
          .padding(horizontal = 16.dp) // 水平内边距
          .size(42.dp) // 大小
          .border(1.5.dp, borderColor, CircleShape) // 头像边框
          .border(3.dp, MaterialTheme.colorScheme.surface, CircleShape) // 表面上的边框（模拟间距）
          .clip(CircleShape) // 裁剪为圆形
          .align(Alignment.Top), // 顶部对齐
        painter = painterResource(id = authorImageId), // 头像图片
        contentScale = ContentScale.Crop, // 图片裁剪方式
        contentDescription = null // 内容描述
      )
    } else {
      // 头像下方的空白占位符
      Spacer(modifier = Modifier.width(74.dp))
    }
    AuthorAndTextMessage(
      msg = msg,
      isUserMe = isUserMe,
      isFirstMessageByAuthor = isFirstMessageByAuthor,
      isLastMessageByAuthor = isLastMessageByAuthor,
      authorClicked = onAuthorClick,
      modifier = Modifier
        .padding(end = 16.dp) // 尾部内边距
        .weight(1f) // 占据剩余空间
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
      // 显示作者姓名和时间戳
      AuthorNameTimestamp(msg, isUserMe)
    }
    // 显示聊天气泡
    ChatItemBubble(
      msg.message,
      isUserMe,
      authorClicked = authorClicked)
    if (isFirstMessageByAuthor) {
      // 下一个作者之前的最后一个气泡，添加底部间距
      Spacer(modifier = Modifier.height(8.dp))
    } else {
      // 同一作者的气泡之间，添加较小的间距
      Spacer(modifier = Modifier.height(4.dp))
    }
  }
}

@Composable
private fun AuthorNameTimestamp(msg: MessageUiModel, isUserMe: Boolean = false) {
  var userFullName: String = msg.user.fullName
  if (isUserMe) {
    userFullName = "me" // 如果是当前用户，显示 "me"
  }

  // 将作者和时间戳组合在一起显示。
  Row(modifier = Modifier.semantics(mergeDescendants = true) {}) {
    Text(
      text = userFullName,
      style = MaterialTheme.typography.titleMedium,
      modifier = Modifier
        .alignBy(LastBaseline) // 按基线对齐
        .paddingFrom(LastBaseline, after = 8.dp) // 基线后的内边距（到第一个气泡的间距）
    )
    Spacer(modifier = Modifier.width(8.dp)) // 作者和时间戳之间的间距
    Text(
      text = msg.message.createdOn.toString().isoToTimeAgo(), // 将 ISO 时间转换为“多久以前”的格式
      style = MaterialTheme.typography.bodySmall,
      modifier = Modifier.alignBy(LastBaseline), // 按基线对齐
      color = MaterialTheme.colorScheme.onSurfaceVariant // 时间戳颜色
    )
  }
}

@Composable
fun ChatItemBubble(
  message: Message,
  isUserMe: Boolean,
  authorClicked: (String) -> Unit
) {
  // 定义聊天气泡的形状
  val ChatBubbleShape = RoundedCornerShape(4.dp, 20.dp, 20.dp, 20.dp)
  val pressedState = remember { mutableStateOf(false) } // 记住按下状态（未使用）
  // 根据是否为当前用户设置气泡背景色
  val backgroundBubbleColor = if (isUserMe) {
    MaterialTheme.colorScheme.primary
  } else {
    MaterialTheme.colorScheme.surfaceVariant
  }
  Column {
    Surface(
      color = backgroundBubbleColor,
      shape = ChatBubbleShape // 应用气泡形状
    ) {
      if (message.text.isNotEmpty()) {
        // 显示可点击的消息内容
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
  val uriHandler = LocalUriHandler.current // 获取用于处理 URI 的 handler
  // 格式化消息文本（例如，高亮链接或 @提及）
  val styledMessage = messageFormatter(
    text = message.text,
    primary = isUserMe // 指示是否为主要样式（当前用户）
  )

  ClickableText(
    text = styledMessage, // 显示格式化后的 AnnotatedString
    style = MaterialTheme.typography.bodyLarge.copy(color = LocalContentColor.current), // 文本样式
    modifier = Modifier.padding(16.dp), // 内边距
    onClick = { offset -> // 点击回调，offset 是点击位置的字符索引
      styledMessage
        .getStringAnnotations(start = offset, end = offset) // 获取点击位置的注解
        .firstOrNull() // 取第一个注解
        ?.let { annotation ->
          // 根据注解的 tag 处理点击事件
          when (annotation.tag) {
            SymbolAnnotationType.LINK.name -> uriHandler.openUri(annotation.item) // 如果是链接，打开 URI
            SymbolAnnotationType.PERSON.name -> authorClicked(annotation.item) // 如果是 @提及，调用作者点击回调
            else -> Unit // 其他情况不做处理
          }
        }
    }
  )
}
```

这看起来可能有很多代码。但它们只是定义消息 UI 各个部分并处理与消息交互的可组合项。通过将其分解为许多小的可组合项，你可以单独处理 UI 的微小细节。

最后，在 **MainActivity.kt** 中，更新对 `ConversationContent` 的调用以包含你添加的新参数并提供虚拟聊天数据：

```kotlin
setContent {
  ConversationContent(
    uiState = exampleUiState // 传入包含虚拟数据的 UI 状态
  )
}
```

构建并运行应用。

 ![](https://assets.alexandria.kodeco.com/books/9f59dbccde22edd2ab5693b6e6b2b51c5d4fe4261fa1a6477dd65d3d2a9028de/images/870052abc1d0ba30390007a854d990ab/original.png)

哇，这开始看起来像一个真正的聊天应用了！点击第一条聊天消息中的超链接。可组合代码通过在浏览器窗口中打开链接正确地处理了它。酷毙了！

## 主题 & 使用字体

在 Compose 中，你可以在你的 Compose 应用中使用可下载字体 API 来异步下载 [Google 字体](https://fonts.google.com/) 并在你的应用中使用它们。有关使用 Google 字体的分步详细信息，请参阅[官方文档](https://developer.android.com/jetpack/compose/text/fonts#downloadable-fonts)。你可以在你的应用中定义一个主题（theme），然后使用你的自定义字体，包括 Google 字体。完成此操作后，在 `setContent` 中用主题包装你的 UI 是一个简单的步骤。为了演示，将 Google 字体的依赖项添加到你的模块级 gradle 文件中：

`implementation("androidx.compose.ui:ui-text-google-fonts:1.5.4")`

然后，从本章的最终项目中，复制到你的项目：

+ **res -> font** 及其中的所有字体文件
+ **com.kodeco.chat -> theme** - **Color.kt**, **Themes.kt**, **Typography.kt**

最后，在 **MainActivity.kt** 中，更新对 `setContent` 的调用：

```kotlin
setContent {
  KodecochatTheme { // 用自定义主题包装内容
     ConversationContent(
       uiState = exampleUiState
     )
  }
}
```

你所做的只是将所有内容包装在 `KodecochatTheme{}` 内部。

构建并运行。

 ![](https://assets.alexandria.kodeco.com/books/9f59dbccde22edd2ab5693b6e6b2b51c5d4fe4261fa1a6477dd65d3d2a9028de/images/28115af486e524de590118d9090a6d44/original.png)

瞧！它有了完全不同的外观，字体和颜色由新主题决定。

## 更新用户输入字段

最后一点改动，你现在就完成了！在 **Conversation.kt** 中，删除 `ConversationContent` 中对 `SimpleUserInput` 的调用，然后删除该可组合项的定义。将其替换为：

`UserInput(onMessageSent = {})`

构建并运行。

 ![](https://assets.alexandria.kodeco.com/books/9f59dbccde22edd2ab5693b6e6b2b51c5d4fe4261fa1a6477dd65d3d2a9028de/images/87b44d9fff70222c37ac8869576e9933/original.png)

这只是提供了一个更简洁的输入界面。尝试在文本输入区域输入，并查看源代码以了解它是如何组合在一起的。最后，这是它在深色模式下的样子，这要归功于你实现的主题：

 ![](https://assets.alexandria.kodeco.com/books/9f59dbccde22edd2ab5693b6e6b2b51c5d4fe4261fa1a6477dd65d3d2a9028de/images/0b2dd83118cfe85e517c0407ab900705/original.png)

相当漂亮！

## 关键点

做得好！恭喜你走到这一步！这是一个内容丰富的章节，你与 Compose UI 进行了一次旋风般的冒险！在本章中，你学到了：

+ Compose 函数、可组合项的基础知识，以及如何嵌套可组合项并从较小的可组合构建块构建复杂的布局。
+ 更多关于 Android Studio 及其提供的一些强大而便捷的功能，使你的开发生活更轻松。
+ 如何使用布局组来布局你的 UI。
+ 如何在无需构建或运行任何东西的情况下渲染 UI 的预览。
+ 如何使用 Modifiers 影响 UI 的渲染方式。
+ 如何实现自定义主题并使用可下载字体。

## 何去何从？

要了解有关 Jetpack Compose 基础知识的更多信息，请参阅官方 Android 文档：[https://developer.android.com/jetpack/compose/documentation](https://developer.android.com/jetpack/compose/documentation)。在下一章中，你将更深入地研究 Compose UI，利用一些非常强大的设计模式，将你的代码提升到一个新的水平。你还将继续开发 Kodeco Chat 应用！
