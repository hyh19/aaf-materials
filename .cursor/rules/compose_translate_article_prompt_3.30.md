---
description: 翻译 Jetpack Compose 英文文章
globs:
alwaysApply: false
---
# Jetpack Compose 英文文章翻译指令

## 任务描述

你的任务是将我提供的 Jetpack Compose 英文技术文章准确翻译成中文。请保持翻译的专业性和技术准确性，确保使用符合中文 Android/Compose 开发社区习惯的术语和表达方式。翻译结果面向具有 Jetpack Compose 基础知识的中文开发者。直接提供完整的中文翻译内容，无需添加额外说明或解释翻译过程。

## 具体要求

- 将英文文章翻译成地道、流畅、符合中文表达习惯的技术文档
- 严格保持原文的技术准确性、专业性和逻辑关系
- 代码块内容处理：
  - 代码和输出结果保留英文原样，不翻译
  - 代码注释需要翻译成中文，保持技术准确性
  - 翻译注释时保持原注释的缩进和格式
- Jetpack Compose 专业术语处理：
  - 使用中文 Android/Compose 社区或官方认可的术语翻译
  - 无官方中文翻译时保留英文原词（如："Compose"、"Modifier" 等）
  - 组合函数名称保持英文原样（如：`Column`、`Row`、`LazyColumn` 等）
- 完整保留原文的段落结构、标题层级和格式
- 专有名词首次出现时采用"中文（英文）"的格式标注
  - 例如：可组合函数（Composable function）、记忆化（remember）
- 确保整体翻译上下文连贯，避免逐句机械翻译导致的语义不通
- 当遇到无法确定含义的术语时，保留英文并用方括号标注可能的解释
- 输出格式：仅提供完整翻译内容，不包含翻译过程说明或其他额外注释

## 术语参考

### 推荐翻译术语

以下术语应翻译成对应的中文：

| 英文术语 | 中文翻译 |
|---------|---------|
| Composable Function | 可组合函数 |
| Recomposition | 重组 |
| State | 状态 |
| State Hoisting | 状态提升 |
| Side Effect | 副作用 |
| Slot API | 插槽 API |
| Composition | 组合 |
| Content Alpha | 内容透明度 |
| Material Design | Material 设计 |
| Layout | 布局 |
| Theming | 主题化 |
| Scaffolding | 脚手架 |
| Navigation | 导航 |
| Animation | 动画 |
| Preview | 预览 |
| Custom Layout | 自定义布局 |
| Text Style | 文本样式 |
| Shape | 形状 |
| Color | 颜色 |
| Typography | 排版 |
| Padding | 内边距 |
| Margin | 外边距 |
| Constraint | 约束 |
| Intrinsic Measurement | 固有测量 |
| Gesture | 手势 |
| Accessibility | 无障碍 |
| Theme | 主题 |
| Drawable | 可绘制对象 |
| Scrolling | 滚动 |
| Canvas | 画布 |
| Lifecycle | 生命周期 |
| Immutability | 不可变性 |
| Composable Tree | 组合树 |
| Semantics | 语义 |
| Composition Local | 组合本地值 |
| Surface | 表面 |
| LaunchedEffect | 启动效应 |
| DerivedState | 派生状态 |
| DisposableEffect | 可处理效应 |
| RememberCoroutineScope | 记住协程作用域 |
| SnapshotFlow | 快照流 |
| Stateless | 无状态 |
| Stateful | 有状态 |
| Box Constraints | 盒约束 |
| Alignment | 对齐方式 |
| Arrangement | 排列方式 |
| Weight | 权重 |
| Scope | 作用域 |
| Source-based Transformation | 基于源代码的转换 |

### 保留原文术语

以下术语通常保留英文原文，无需翻译：

| 英文术语 | 使用方式 |
|---------|---------|
| Jetpack Compose | 直接使用 Jetpack Compose |
| Modifier | 直接使用 Modifier |
| Compose Compiler | 直接使用 Compose Compiler |
| Column | 直接使用 Column |
| Row | 直接使用 Row |
| Box | 直接使用 Box |
| LazyColumn | 直接使用 LazyColumn |
| LazyRow | 直接使用 LazyRow |
| LazyVerticalGrid | 直接使用 LazyVerticalGrid |
| remember | 直接使用 remember |
| mutableStateOf | 直接使用 mutableStateOf |
| CompositionLocal | 直接使用 CompositionLocal |
| MaterialTheme | 直接使用 MaterialTheme |
| Scaffold | 直接使用 Scaffold |
| TopAppBar | 直接使用 TopAppBar |
| BottomAppBar | 直接使用 BottomAppBar |
| FloatingActionButton | 直接使用 FloatingActionButton |
| ConstraintLayout | 直接使用 ConstraintLayout |
| AnimatedVisibility | 直接使用 AnimatedVisibility |
| Crossfade | 直接使用 Crossfade |
| Animatable | 直接使用 Animatable |
| Preview | 直接使用 @Preview |
| Composable | 直接使用 @Composable |
| Card | 直接使用 Card |
| OutlinedTextField | 直接使用 OutlinedTextField |
| Button | 直接使用 Button |
| Icon | 直接使用 Icon |
| Image | 直接使用 Image |
| Slider | 直接使用 Slider |
| Switch | 直接使用 Switch |
| Checkbox | 直接使用 Checkbox |
| RadioButton | 直接使用 RadioButton |
| DropdownMenu | 直接使用 DropdownMenu |
| Dialog | 直接使用 Dialog |
| AlertDialog | 直接使用 AlertDialog |
| ModalBottomSheet | 直接使用 ModalBottomSheet |
| NavHost | 直接使用 NavHost |
| NavController | 直接使用 NavController |
| Coil | 直接使用 Coil |
| Glide | 直接使用 Glide |
| Accompanist | 直接使用 Accompanist |
| ViewModel | 直接使用 ViewModel |
| Coroutine | 可使用"协程"或直接使用 Coroutine |
| Flow | 可使用"流"或直接使用 Flow |
