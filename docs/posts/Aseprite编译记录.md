---
description: Aseprite编译记录。
tags: 
    - 教程
---

# Aseprite编译记录

可直接使用开发者提供的`build.sh`进行编译。
需要在 `build.sh` 的 cmake 部分添加以下内容：

``` bash
  -DCMAKE_CXX_FLAGS:STRING=-stdlib=libc++ \
  -DCMAKE_EXE_LINKER_FLAGS:STRING=-stdlib=libc++ \
```

## 目录

- [Aseprite编译记录](#aseprite编译记录)
  - [目录](#目录)
  - [支持平台](#支持平台)
  - [获取源代码](#获取源代码)
  - [依赖项](#依赖项)
    - [Windows依赖项](#windows依赖项)
    - [macOS依赖项](#macos依赖项)
    - [Linux依赖项](#linux依赖项)
  - [编译指南](#编译指南)
    - [Windows详细说明](#windows详细说明)
      - [MinGW注意事项](#mingw注意事项)
    - [macOS详细说明](#macos详细说明)
      - [Retina显示屏问题](#retina显示屏问题)
    - [Linux详细说明](#linux详细说明)
  - [使用共享第三方库](#使用共享第三方库)

## 支持平台

您可以在以下平台成功编译Aseprite：

• Windows 11 + [Visual Studio Community 2022 + Windows 10.0 SDK（最新版本）](https://imgur.com/a/7zs51IT)（不支持[MinGW](#mingw注意事项)）
• macOS 13.0.1 Ventura + Xcode 14.1 + macOS 11.3 SDK（旧版本可能兼容）
• Linux Ubuntu Bionic 18.04 + clang 10.0

## 获取源代码

您可以通过以下方式获取源代码：

1. 从最新Aseprite发布的 `Aseprite-v1.x-Source.zip` 文件下载（请遵循压缩包内的编译说明）：

    [Github下载](https://github.com/aseprite/aseprite/releases)

2. 或者使用git克隆仓库及其子模块：

    ``` bash
    git clone --recursive https://github.com/aseprite/aseprite.git
    ```

更新已有代码库：

``` bash
cd aseprite
git pull
git submodule update --init --recursive
```

Windows用户可使用[Git for Windows](https://git-for-windows.github.io/)进行克隆。

## 依赖项

编译需要以下工具：
• [CMake](https://cmake.org) 3.16或更高版本
• [Ninja](https://ninja-build.org) 构建系统
• 已编译的[Skia库](https://github.com/aseprite/skia#readme)（`aseprite-m102`分支），可获取[预编译包](https://github.com/aseprite/skia/releases)，更多信息参见[laf依赖项说明](https://github.com/aseprite/laf#dependencies)

### Windows依赖项

• Windows 10/11系统
• [Visual Studio Community 2022](https://visualstudio.microsoft.com/downloads/)
• Visual Studio安装器中勾选["使用C++的桌面开发" + "Windows 10.0.18362.0 SDK"](https://imgur.com/a/7zs51IT)

### macOS依赖项

• macOS 11.3 SDK
• Xcode 13.1或更高版本

### Linux依赖项

Ubuntu/Debian安装命令：

```bash
sudo apt-get install -y g++ clang libc++-dev libc++abi-dev cmake ninja-build libx11-dev libxcursor-dev libxi-dev libgl1-mesa-dev libfontconfig1-dev
```

Fedora安装命令：

```bash
sudo dnf install -y gcc-c++ clang libcxx-devel cmake ninja-build libX11-devel libXcursor-devel libXi-devel mesa-libGL-devel fontconfig-devel
```

Arch Linux安装命令：

```bash
sudo pacman -S gcc clang libc++ cmake ninja libx11 libxcursor mesa-libgl fontconfig libwebp
```

openSUSE安装命令：

```bash
sudo zypper install gcc-c++ clang libc++-devel libc++abi-devel cmake ninja libX11-devel libXcursor-devel libXi-devel Mesa-libGL-devel fontconfig-devel
```

## 编译指南

1. **创建构建目录**

    ```bash
    cd aseprite
    mkdir build
    ```

2. **配置CMake**

    ```bash
    cd build
    cmake -G Ninja -DLAF_BACKEND=skia ..
    ```

   *具体参数根据平台调整，详见下方各系统说明*

3. **执行编译**

    ```bash
    ninja aseprite
    ```

4. **获取可执行文件**
    ◦ 生成文件位于`build/bin/aseprite`（Windows为`.exe`）

### Windows详细说明

1. 使用**x64 Native Tools Command Prompt for VS 2022**（开始菜单搜索）
2. 编译命令示例：

    ```bash
    cmake -DCMAKE_BUILD_TYPE=RelWithDebInfo -DLAF_BACKEND=skia -DSKIA_DIR=C:\deps\skia -DSKIA_LIBRARY_DIR=C:\deps\skia\out\Release-x64 -DSKIA_LIBRARY=C:\deps\skia\out\Release-x64\skia.lib -G Ninja ..
    ninja aseprite
    ```

#### MinGW注意事项

• 不支持MinGW编译器
• 若检测到MinGW路径，需从PATH环境变量中移除或使用：
    ```bash
    cmake -DCMAKE_IGNORE_PATH=C:\MinGW\bin ...
    ```

### macOS详细说明

Intel芯片编译命令：

```bash
cmake \
  -DCMAKE_BUILD_TYPE=RelWithDebInfo \
  -DCMAKE_OSX_ARCHITECTURES=x86_64 \
  -DCMAKE_OSX_DEPLOYMENT_TARGET=10.9 \
  -DCMAKE_OSX_SYSROOT=/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk \
  -DLAF_BACKEND=skia \
  -DSKIA_DIR=$HOME/deps/skia \
  -DSKIA_LIBRARY_DIR=$HOME/deps/skia/out/Release-x64 \
  -DSKIA_LIBRARY=$HOME/deps/skia/out/Release-x64/libskia.a \
  -G Ninja \
  ..
```

Apple Silicon（M1芯片）编译命令：

```bash
cmake \
  -DCMAKE_BUILD_TYPE=RelWithDebInfo \
  -DCMAKE_OSX_ARCHITECTURES=arm64 \
  -DCMAKE_OSX_DEPLOYMENT_TARGET=11.0 \
  -DCMAKE_OSX_SYSROOT=/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk \
  -DLAF_BACKEND=skia \
  -DSKIA_DIR=$HOME/deps/skia \
  -DSKIA_LIBRARY_DIR=$HOME/deps/skia/out/Release-arm64 \
  -DSKIA_LIBRARY=$HOME/deps/skia/out/Release-arm64/libskia.a \
  -DPNG_ARM_NEON:STRING=on \
  -G Ninja \
  ..
```

#### Retina显示屏问题

• 参考issue [#589](https://github.com/aseprite/aseprite/issues/589)

### Linux详细说明

使用clang和libc++编译：

```bash
export CC=clang
export CXX=clang++
cmake \
  -DCMAKE_BUILD_TYPE=RelWithDebInfo \
  -DCMAKE_CXX_FLAGS:STRING=-stdlib=libc++ \
  -DCMAKE_EXE_LINKER_FLAGS:STRING=-stdlib=libc++ \
  -DLAF_BACKEND=skia \
  -DSKIA_DIR=$HOME/deps/skia \
  -DSKIA_LIBRARY_DIR=$HOME/deps/skia/out/Release-x64 \
  -DSKIA_LIBRARY=$HOME/deps/skia/out/Release-x64/libskia.a \
  -G Ninja \
  ..
```

## 使用共享第三方库

若要使用系统安装的共享库（而非内置版本）：

1. 编辑`build/CMakeCache.txt`
2. 将对应的`USE_SHARED_`选项设为`ON`

例如启用Freetype共享库：

```cmake
USE_SHARED_FREETYPE:BOOL=ON
```
