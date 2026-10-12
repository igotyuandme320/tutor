在 Windows 10/11 上，推荐使用 **winget + MSYS2 (UCRT64) + pacman**，通过 CLI 安装 MinGW-w64、CMake、Ninja 和 Qt 6。

相比单独安装各个工具，这种方式可以统一管理依赖，避免 Qt 与 MinGW 编译器版本不兼容。

## 1. 安装 MSYS2

打开 Windows PowerShell：

```powershell
winget install --id MSYS2.MSYS2 -e
```

MSYS2 是 Windows 上的类 Unix 开发环境，提供 `pacman` 包管理器。

安装完成后，打开 **MSYS2 UCRT64** 终端。

也可以从 PowerShell 启动：

```powershell
C:\msys64\msys2_shell.cmd -ucrt64
```

## 2. 更新 MSYS2

在 MSYS2 UCRT64 终端执行：

```bash
pacman -Syu
```

如果提示关闭终端，重新打开 UCRT64，再执行：

```bash
pacman -Syu
```

直到更新完成。

## 3. 一次性安装 MinGW、CMake、Qt

在 MSYS2 UCRT64 终端运行：

```bash
pacman -S --needed \
  mingw-w64-ucrt-x86_64-toolchain \
  mingw-w64-ucrt-x86_64-cmake \
  mingw-w64-ucrt-x86_64-ninja \
  mingw-w64-ucrt-x86_64-qt6-base \
  mingw-w64-ucrt-x86_64-qt6-tools \
  mingw-w64-ucrt-x86_64-qt-creator
```

安装内容：

| 工具 | 用途 |
|---|---|
| MinGW-w64 | GCC/G++ 编译器 |
| CMake | C++ 项目构建配置 |
| Ninja | 高速构建工具 |
| Qt 6 Base | Qt Core、GUI、Widgets 等 |
| Qt 6 Tools | Qt Designer 等开发工具 |
| Qt Creator | Qt 官方 IDE |

这些软件均通过 MSYS2 的 UCRT64 软件仓库安装，保持工具链一致。[mingw-w64](https://www.mingw-w64.org/getting-started/msys2/?utm_source=chatgpt.com)

如果只需要 Qt 库，不需要 Qt Creator，可以去掉最后一个包。

## 4. 配置 Windows PATH

如果希望在 Windows PowerShell、CMD 和 VS Code 终端直接使用这些工具，需要配置 PATH。

使用 PowerShell：

```powershell
$mingw = "C:\msys64\ucrt64\bin"

$userPath = [Environment]::GetEnvironmentVariable("Path", "User")

if (($userPath -split ';') -notcontains $mingw) {
    [Environment]::SetEnvironmentVariable(
        "Path",
        "$userPath;$mingw",
        "User"
    )
}
```

重新启动终端。

这会将以下目录添加到用户 PATH：

```text
C:\msys64\ucrt64\bin
```

不要同时添加多个不同版本的 MinGW 到 PATH，否则容易产生编译器冲突。

## 5. 验证安装

重新打开 PowerShell：

```powershell
gcc --version
g++ --version
cmake --version
ninja --version
qmake6 --version
```

还可以检查编译器位置：

```powershell
where.exe gcc
where.exe g++
where.exe cmake
```

预期路径类似：

```text
C:\msys64\ucrt64\bin\gcc.exe
C:\msys64\ucrt64\bin\g++.exe
C:\msys64\ucrt64\bin\cmake.exe
```

## 6. 测试 Qt + CMake

创建项目目录：

```powershell
mkdir qt-test
cd qt-test
```

创建 `main.cpp`：

```cpp
#include <QApplication>
#include <QPushButton>

int main(int argc, char *argv[]) {
    QApplication app(argc, argv);

    QPushButton button("Hello Qt!");
    button.resize(300, 100);
    button.show();

    return app.exec();
}
```

创建 `CMakeLists.txt`：

```cmake
cmake_minimum_required(VERSION 3.20)

project(QtTest LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

find_package(Qt6 REQUIRED COMPONENTS Widgets)

add_executable(QtTest main.cpp)

target_link_libraries(QtTest PRIVATE Qt6::Widgets)
```

使用 PowerShell 构建：

```powershell
cmake -S . -B build -G Ninja `
  -DCMAKE_BUILD_TYPE=Release `
  -DCMAKE_PREFIX_PATH=C:/msys64/ucrt64

cmake --build build
```

运行：

```powershell
.\build\QtTest.exe
```

正常情况下，会弹出一个写着 `Hello Qt!` 的窗口。

---

### 最终工具链

```text
Windows 11
│
├── MSYS2
│   └── UCRT64
│       ├── GCC / G++
│       ├── GDB
│       ├── CMake
│       ├── Ninja
│       ├── Qt 6
│       └── Qt Creator
│
└── VS Code / PowerShell
    └── CMake + Ninja + Qt
```

**我的建议：** 如果你主要使用 VS Code 或 CLion 开发 C++/Qt 项目，选择 **MSYS2 UCRT64 + GCC + CMake + Ninja + Qt 6** 即可。

这套环境既适合普通 C++ 开发，也适合 Qt GUI 项目，而且全部可以通过命令行管理。
