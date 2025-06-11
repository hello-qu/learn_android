# Android 开发环境搭建

## 1. 系统要求
- Windows 7/8/10/11 (64位)
- macOS 10.14 (Mojave) 或更高版本
- Linux (Ubuntu 18.04 或更高版本)
- 至少 8GB RAM，推荐 16GB 或更多
- 至少 8GB 可用磁盘空间，推荐 16GB 或更多

## 2. 安装 JDK (Java Development Kit)
1. 访问 [Oracle JDK 下载页面](https://www.oracle.com/java/technologies/downloads/) 或使用 OpenJDK
2. 下载适合您操作系统的 JDK 版本（推荐 JDK 17 或更高版本）
3. 安装 JDK 并设置环境变量

### Windows 环境变量设置
```bash
JAVA_HOME = C:\Program Files\Java\jdk-17
Path = %JAVA_HOME%\bin
```

### macOS/Linux 环境变量设置
```bash
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk-17.jdk/Contents/Home
export PATH=$JAVA_HOME/bin:$PATH
```

## 3. 安装 Android Studio
1. 访问 [Android Studio 下载页面](https://developer.android.com/studio)
2. 下载适合您操作系统的版本
3. 运行安装程序并按照向导完成安装

## 4. 首次启动配置
1. 启动 Android Studio
2. 选择 "Standard" 安装类型
3. 选择 UI 主题（深色/浅色）
4. 等待下载和安装必要的组件

## 5. 安装 Android SDK
1. 打开 Android Studio
2. 点击 Tools > SDK Manager
3. 在 SDK Platforms 标签页中，选择最新的 Android 版本
4. 在 SDK Tools 标签页中，确保选中以下组件：
   - Android SDK Build-Tools
   - Android SDK Command-line Tools
   - Android Emulator
   - Android SDK Platform-Tools

## 6. 配置模拟器
1. 点击 Tools > Device Manager
2. 点击 "Create Device"
3. 选择设备类型（如 Pixel 4）
4. 选择系统镜像（推荐选择最新的稳定版本）
5. 完成模拟器配置

## 7. 验证安装
1. 创建新项目
2. 选择 "Empty Activity"
3. 配置项目信息：
   - Name: HelloWorld
   - Package name: com.example.helloworld
   - Language: Kotlin
   - Minimum SDK: API 24 (Android 7.0)
4. 点击 "Finish" 创建项目
5. 等待项目构建完成
6. 运行项目到模拟器

## 8. 常见问题解决
1. SDK 下载失败
   - 检查网络连接
   - 使用代理或 VPN
   - 手动下载 SDK 组件

2. 模拟器启动失败
   - 检查是否启用了虚拟化技术（BIOS 设置）
   - 更新显卡驱动
   - 增加模拟器内存分配

3. Gradle 同步失败
   - 检查网络连接
   - 更新 Gradle 版本
   - 清理项目缓存

## 9. 推荐插件
- Kotlin
- Android WiFi ADB
- Material Design Icon Generator
- JSON To Kotlin Class
- Git Integration

## 10. 下一步
完成环境搭建后，您可以：
1. 开始学习 [Kotlin 基础语法](./02_kotlin_basics.md)
2. 熟悉 [Android Studio 使用指南](./03_android_studio_guide.md)
3. 创建您的第一个 Android 应用 