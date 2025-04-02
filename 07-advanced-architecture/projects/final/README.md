# Kodeco Chat 应用

这是一个使用 Jetpack Compose 构建的 Android 聊天应用示例项目，使用 Ditto 作为实时数据同步解决方案。

## 环境要求

- Android Studio Hedgehog（2023.1.1）或更高版本
- Android SDK 34（compileSdk = 34，targetSdk = 34，minSdk = 30）
- Kotlin 1.9.10
- JDK 17（必需，其他版本可能导致构建失败）
- Gradle 8.2
- Ditto SDK

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

4. 在项目根目录创建 `keys.properties` 文件，并添加 Ditto 配置：

   ```properties
   DITTO_APP_ID="your_ditto_app_id"
   DITTO_TOKEN="your_ditto_token"
   ```

5. 等待 Gradle 同步完成。

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

项目提供两种命令行运行方式：快速安装和完整安装流程。

#### 方式一：快速安装到正在运行的模拟器

如果模拟器已经在运行，可以使用以下步骤快速安装：

1. 设置 JDK 17 环境：

   ```bash
   # 设置 JAVA_HOME 环境变量指向 JDK 17
   export JAVA_HOME=$(/usr/libexec/java_home -v 17)
   ```

2. 验证模拟器运行状态：

   ```bash
   # 检查连接的设备列表，确认模拟器在运行
   adb devices
   ```

3. 构建并安装应用：

   ```bash
   # 构建并安装调试版本
   ./gradlew installDebug
   ```

4. 启动应用：

   ```bash
   # 启动主活动
   adb shell am start -n com.kodeco.chat/.MainActivity
   ```

#### 方式二：完整安装流程

如果需要从头开始设置和运行项目，请按以下步骤操作：

1. 检查并关闭已有的模拟器实例（避免多实例冲突）：

   ```bash
   # 此命令会列出所有连接的设备并关闭正在运行的模拟器
   adb devices && adb emu kill
   ```

2. 设置 JDK 17 环境：

   ```bash
   # 设置 JAVA_HOME 环境变量指向 JDK 17
   export JAVA_HOME=$(/usr/libexec/java_home -v 17)
   ```

3. 启动模拟器（使用只读模式避免多实例问题）：

   ```bash
   # 使用 -read-only 标志启动模拟器
   ~/Library/Android/sdk/emulator/emulator -avd Pixel_3a_API_34_extension_level_7_arm64-v8a -read-only
   ```

   启动过程中你会看到一些日志输出，等待直到看到 "Boot completed" 消息。

4. 清理并重新构建项目（确保干净的构建）：

   ```bash
   # 等待模拟器完全启动后再构建（这里等待 10 秒）
   sleep 10 && ./gradlew clean installDebug
   ```

   注意：构建过程中可能会看到一些警告，如未使用的变量，这些不影响应用运行。

5. 启动应用：

   ```bash
   # 启动主活动
   adb shell am start -n com.kodeco.chat/.MainActivity
   ```

完成任一方式的步骤后，应用都会在模拟器中启动并运行。如果遇到问题，请参考"常见问题解决"章节。

## 使用的主要依赖库

- Jetpack Compose BOM（2023.10.01）
- Compose Material 3（1.1.2）
- Coil Compose（2.4.0）- 图片加载
- KotlinX DateTime（0.4.0）- 日期时间处理
- Ditto SDK - 实时数据同步

## 常见问题解决

### JDK 版本不兼容

确保使用 JDK 17：

```bash
# 在 macOS 上安装 JDK 17
brew install openjdk@17

# 创建系统级符号链接
sudo ln -sfn /opt/homebrew/opt/openjdk@17/libexec/openjdk.jdk /Library/Java/JavaVirtualMachines/openjdk-17.jdk

# 设置 JAVA_HOME（可添加到 ~/.bash_profile 或 ~/.zshrc）
export JAVA_HOME=$(/usr/libexec/java_home -v 17)
```

### 模拟器多实例问题

如果遇到以下错误：

```
ERROR   | Running multiple emulators with the same AVD 
ERROR   | is an experimental feature.
ERROR   | Please use -read-only flag to enable this feature.
```

解决方案：

1. 先关闭所有正在运行的模拟器实例：

   ```bash
   adb devices && adb emu kill
   ```

2. 使用 `-read-only` 标志启动模拟器：

   ```bash
   ~/Library/Android/sdk/emulator/emulator -avd <模拟器名称> -read-only
   ```

### Ditto 配置问题

如果遇到 Ditto 相关的构建错误，请确保：

1. `keys.properties` 文件存在且包含有效的 Ditto 凭证
2. 凭证格式正确（使用双引号包裹值）
3. 构建配置正确读取了凭证

### Gradle 构建失败

尝试清理项目后重新构建：

```bash
./gradlew clean
./gradlew installDebug
```

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
- `data`：包含应用程序的数据模型和 Ditto 数据同步逻辑
- `theme`：包含应用程序的主题和样式定义
- `ui.theme`：包含 UI 主题相关定义
- `utilities`：包含实用工具和扩展函数

## 许可证

本项目使用 Kodeco 授权许可。详细信息请参阅源代码头部的版权信息。
