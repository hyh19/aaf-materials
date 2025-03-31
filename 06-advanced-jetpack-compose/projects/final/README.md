# Kodeco Chat 应用

这是一个使用 Jetpack Compose 构建的 Android 聊天应用示例项目。

## 环境要求

- Android Studio Hedgehog（2023.1.1）或更高版本
- Android SDK 34（compileSdk = 34，targetSdk = 34，minSdk = 30）
- Kotlin 1.9.10
- JDK 17
- Gradle 8.2

## 配置项目

1. 克隆仓库到本地：

   ```bash
   git clone <仓库地址>
   ```

2. 打开 Android Studio，选择"Open an existing project"并导航到项目目录。

3. 确保已配置好 Android SDK 路径。如果没有，你可以通过以下两种方式之一进行配置：
   - 在项目根目录创建 `local.properties` 文件，并添加 SDK 路径：

     ```properties
     sdk.dir=/Users/<用户名>/Library/Android/sdk
     ```

   - 或设置 ANDROID_HOME 环境变量：

     ```bash
     export ANDROID_HOME=/Users/<用户名>/Library/Android/sdk
     ```

4. 等待 Gradle 同步完成。

## 构建项目

### 使用 Android Studio 构建

1. 打开项目后，点击工具栏中的"Build"菜单。
2. 选择"Build Project"或"Make Project"选项。

### 使用命令行构建

在项目根目录下，运行以下命令构建调试版本：

```bash
./gradlew assembleDebug
```

构建发布版本：

```bash
./gradlew assembleRelease
```

## 运行项目

### 使用 Android Studio 运行

1. 从设备下拉菜单中选择一个已连接的设备或模拟器。
2. 点击"Run"按钮（绿色三角形图标）。

### 使用命令行运行

1. 启动模拟器（如果尚未运行）：

   ```bash
   ~/Library/Android/sdk/emulator/emulator -avd <模拟器名称>
   ```

   例如：

   ```bash
   ~/Library/Android/sdk/emulator/emulator -avd Pixel_3a_API_34_extension_level_7_arm64-v8a
   ```

2. 构建并安装应用：

   ```bash
   ./gradlew installDebug
   ```

3. 启动应用：

   ```bash
   adb shell am start -n com.kodeco.chat/.MainActivity
   ```

## 使用的主要依赖库

- Jetpack Compose BOM（2023.10.01）
- Compose Material 3（1.1.2）
- Coil Compose（2.4.0）- 图片加载
- KotlinX DateTime（0.4.0）- 日期时间处理

## 常见问题解决

### 找不到 Android SDK 的位置

确保正确设置了 `local.properties` 文件或 `ANDROID_HOME` 环境变量。

### Gradle 构建失败

尝试在项目根目录下运行：

```bash
./gradlew clean
```

然后重新构建项目。

### 模拟器列表为空

使用以下命令检查可用的模拟器：

```bash
~/Library/Android/sdk/emulator/emulator -list-avds
```

如果没有可用的模拟器，请通过 Android Studio 的 AVD Manager 创建一个。

项目已验证在 Pixel_3a_API_34_extension_level_7_arm64-v8a 模拟器上可正常运行。

## 项目结构

该项目使用了 Jetpack Compose 构建用户界面，采用了现代 Android 开发架构。主要组件包括：

- `MainActivity`：应用程序的入口点
- `conversation`：包含聊天会话相关的组件和状态管理
- `components`：包含可重用的 UI 组件
- `data`：包含应用程序的数据模型和假数据
- `theme`：包含应用程序的主题和样式定义
- `ui.theme`：包含 UI 主题相关定义
- `utilities`：包含实用工具和扩展函数

## 许可证

本项目使用 Kodeco 授权许可。详细信息请参阅源代码头部的版权信息。
