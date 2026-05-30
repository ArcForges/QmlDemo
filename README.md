# QmlDemo — Qt / QML 的 VS Code 配置 Demo

[![Windows CI](https://github.com/ArcForges/QmlDemo/actions/workflows/windows.yml/badge.svg?branch=main)](https://github.com/ArcForges/QmlDemo/actions)
[![Linux CI](https://github.com/ArcForges/QmlDemo/actions/workflows/linux.yml/badge.svg?branch=main)](https://github.com/ArcForges/QmlDemo/actions)

本仓库是一个 **Qt / QML 项目在 VS Code 下的开箱即用配置示例（Demo）**。

它演示了如何把 VS Code 打造成一个完整的 Qt6 + QML 开发环境：用 **clangd** 做 C++ 智能感知、用 **qmlls（QML Language Server）** 做 QML 智能感知与跳转、用 **CMake Presets** 驱动构建、用 **LLDB** 调试。重点在 `.vscode/` 目录下的配置文件，尤其是 [`settings.json`](.vscode/settings.json) ——下文会逐项讲解。

> 仓库里的应用源码（`src/`）来自一个 Qt/QML 笔记应用，仅作为「真实可编译的样例工程」存在。如果你只关心 VS Code 配置，直接看 [`.vscode/`](.vscode/) 即可。

---

## 目录结构

```
QmlDemo/
├── .vscode/
│   ├── settings.json     # 编辑器 / clangd / qmlls 核心配置（本 Demo 的主角）
│   ├── extensions.json   # 推荐安装的扩展
│   ├── launch.json       # LLDB 调试配置
│   └── tasks.json        # CMake 构建任务
├── cmake/
│   └── sync_qmlls_ini.cmake   # 构建后生成 .qmlls.ini，供 qmlls 读取
├── CMakePresets.json     # 构建预设（windows-debug / linux-debug ...）
├── CMakeLists.txt
└── src/                  # 样例 Qt/QML 应用源码
```

---

## 前置条件

| 工具 | 说明 |
| ---- | ---- |
| **Qt 6.11.x** | 需包含 `qmlls`（位于 `<Qt>/<ver>/<kit>/bin/qmlls.exe`）。Demo 默认指向 `C:/Qt/6.11.1/msvc2022_64`。 |
| **Qt Docs** | 可选，用于 qmlls 文档悬浮提示，默认 `C:/Qt/Docs/Qt-6.11.1`。 |
| **CMake ≥ 4.2** | `CMakePresets.json` 要求的最低版本。 |
| **clangd** | C++ 语言服务器（由扩展 `llvm-vs-code-extensions.vscode-clangd` 提供）。 |
| **clang / clang-cl** | Demo 的构建预设使用 Clang 工具链。 |

安装下表中的推荐扩展后即可开箱即用（VS Code 会根据 `extensions.json` 提示安装）。

---

## 快速开始

1. 用 VS Code 打开仓库根目录，按提示安装推荐扩展。
2. 把 `settings.json` 里的 Qt 路径改成你本机的安装位置（见下文「需要按本机修改的项」）。
3. 用 CMake 配置并构建一次 Debug：
   ```powershell
   cmake --preset windows-debug
   cmake --build --preset debug
   ```
   构建会在 `build/debug/` 生成产物，并由 `sync_qmlls_ini.cmake` 写出根目录的 `.qmlls.ini`。
4. 打开任意 `.qml` 文件，qmlls 即可提供补全、跳转、诊断；打开 `.cpp/.h`，clangd 接管 C++ 智能感知。

---

## `settings.json` 详解

这是本 Demo 的核心。完整文件见 [`.vscode/settings.json`](.vscode/settings.json)。下面按功能分组逐项说明每个键的作用。

### 1. C++ 智能感知：交给 clangd

```jsonc
"C_Cpp.intelliSenseEngine": "disabled",
"editor.defaultFormatter": "llvm-vs-code-extensions.vscode-clangd",
"[cpp]": {
  "editor.defaultFormatter": "llvm-vs-code-extensions.vscode-clangd"
}
```

| 键 | 作用 |
| --- | --- |
| `C_Cpp.intelliSenseEngine: "disabled"` | **关闭微软 C/C++ 扩展自带的 IntelliSense 引擎**，避免它与 clangd 抢补全 / 双重报红。微软扩展仍可保留用于调试等功能，但代码分析交给 clangd。 |
| `editor.defaultFormatter` | 默认格式化器设为 clangd（依据项目 `.clang-format`）。 |
| `[cpp]` 区块 | 针对 C++ 文件再次显式指定 clangd 为格式化器，确保不被其它扩展覆盖。 |

> clangd 依赖编译数据库 `compile_commands.json`。`CMakeLists.txt` 中开启了 `CMAKE_EXPORT_COMPILE_COMMANDS`，并通过 `sync-compile-commands` 目标把它同步到 `build/`，clangd 默认即可找到。

### 2. CMake：始终使用 Presets

```jsonc
"cmake.useCMakePresets": "always"
```

强制 CMake Tools 扩展使用 [`CMakePresets.json`](CMakePresets.json)（而非旧的 kit/variant 机制）。这样「配置 / 构建 / 调试」都走统一的预设，跨平台行为一致。

### 3. 编辑器外观与排版

```jsonc
"editor.fontFamily": "'Maple Mono NF CN', Menlo, Consolas, 'Maple UI', PingFang, 'Microsoft YaHei', monospace",
"editor.fontSize": 14,
"editor.fontLigatures": "'calt', 'cv01', 'ss01', 'zero'",
"editor.tabSize": 2,
"editor.insertSpaces": true,
"editor.detectIndentation": false,
"editor.renderWhitespace": "boundary",
"editor.rulers": [120],
"editor.formatOnSave": true
```

| 键 | 作用 |
| --- | --- |
| `editor.fontFamily` | 等宽字体回退链，首选支持连字与中文的 Maple Mono。 |
| `editor.fontLigatures` | 启用指定 OpenType 特性：`calt`（上下文连字）、`cv01`/`ss01`（字形变体）、`zero`（带斜杠的 0）。 |
| `editor.tabSize: 2` + `insertSpaces: true` | 缩进为 **2 个空格**。 |
| `editor.detectIndentation: false` | 关闭「按文件自动探测缩进」，强制使用上面的设置，避免不同文件缩进不一致。 |
| `editor.renderWhitespace: "boundary"` | 只在单词之间、行尾等边界处显示空白字符。 |
| `editor.rulers: [120]` | 在第 120 列画参考线（与 CMake/CI 的行宽约定一致）。 |
| `editor.formatOnSave: true` | 保存时自动格式化。 |

### 4. 文件行为

```jsonc
"files.associations": {
  "*.h":   "cpp",
  "*.hpp": "cpp",
  "*.inl": "cpp"
},
"files.trimTrailingWhitespace": true,
"files.insertFinalNewline": true
```

| 键 | 作用 |
| --- | --- |
| `files.associations` | 把 `.h` / `.hpp` / `.inl` 一律识别为 **C++**（默认 `.h` 可能被当成 C），保证 clangd 用 C++ 模式解析。 |
| `files.trimTrailingWhitespace` | 保存时删除行尾空白。 |
| `files.insertFinalNewline` | 保存时确保文件末尾有一个换行符。 |

### 5. 搜索排除

```jsonc
"search.exclude": {
  "**/build": true,
  "**/.cache": true,
  "**/node_modules": true,
  "**/*_autogen": true
}
```

把构建产物（`build/`）、缓存（`.cache/`，clangd 索引）、依赖（`node_modules/`）和 Qt 自动生成目录（`*_autogen/`）排除出全局搜索，让 <kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>F</kbd> 只命中真实源码。

### 6. 集成终端字体

```jsonc
"terminal.integrated.fontFamily": "'Maple Mono NF CN', Consolas, monospace",
"terminal.integrated.fontSize": 13
```

让内置终端使用 Nerd Font，正确显示图标 / 连字。

### 7. ⭐ QML Language Server（qmlls）

这是本 Demo 最关键的部分——让 VS Code 对 **QML** 拥有补全、跳转定义、内联诊断的能力。由 Qt 官方扩展 `theqtcompany.qt-qml` 提供。

```jsonc
"qt-qml.qmlls.enabled": true,
"qt-qml.qmlls.customExePath": "C:/Qt/6.11.1/msvc2022_64/bin/qmlls.exe",
"qt-qml.qmlls.additionalImportPaths": ["${workspaceFolder}/build/debug/src"],
"qt-qml.qmlls.customArgs": [
  "-b${workspaceFolder}/build/debug",
  "-dC:/Qt/Docs/Qt-6.11.1"
],
"qt-qml.qmlls.useNoCMakeCalls": true,
"qt-qml.qmlls.traceLsp": "messages",
"qt-qml.qmlls.verboseOutput": true,
"qt-qml.preview.additionalBuildDirs": ["${workspaceFolder}/build/debug"]
```

| 键 | 作用 |
| --- | --- |
| `qt-qml.qmlls.enabled` | 启用 qmlls 语言服务器（关闭则回退到扩展自带的简单解析）。 |
| `qt-qml.qmlls.customExePath` | 指定 `qmlls` 可执行文件路径。**务必指向与你工程相同 Qt 版本的 qmlls**，否则类型信息会对不上。 |
| `qt-qml.qmlls.additionalImportPaths` | 追加 QML import 搜索路径。这里指向构建目录下生成的 QML 模块（`build/debug/src`），使 qmlls 能解析本项目自定义的 QML 类型 / 单例。 |
| `qt-qml.qmlls.customArgs` | 传给 qmlls 的命令行参数：<br>• `-b<dir>` = `--build-dir`，指向构建目录 `build/debug`，让 qmlls 读取该工程已编译出的类型信息（`.qmltypes` 等）。<br>• `-d<dir>` = `--docs`，指向 Qt 文档目录，启用悬浮文档提示。 |
| `qt-qml.qmlls.useNoCMakeCalls` | 传入 `--no-cmake-calls`：**禁止 qmlls 自己去调用 CMake** 重新配置工程。改为信任既有构建目录，避免后台意外触发构建、提升稳定性。 |
| `qt-qml.qmlls.traceLsp: "messages"` | 记录 LSP 收发消息（用于排查问题，可在「输出」面板查看）。 |
| `qt-qml.qmlls.verboseOutput` | qmlls 输出更详细的日志。 |
| `qt-qml.preview.additionalBuildDirs` | QML 预览功能额外查找的构建目录，让「QML Preview」能找到本工程的模块。 |

#### qmlls 与 `.qmlls.ini` 的关系

qmlls 既可以从命令行参数读取配置，也可以从工程根目录的 `.qmlls.ini` 读取。本仓库在 **构建后** 由 [`cmake/sync_qmlls_ini.cmake`](cmake/sync_qmlls_ini.cmake) 自动生成 `.qmlls.ini`：

```ini
[General]
buildDir=<构建目录>
no-cmake-calls=true
docDir=<Qt 文档目录>
importPaths=<构建目录>;<可执行目录>;<可执行目录>/qml
```

也就是说，`settings.json` 里的 `customArgs` / `additionalImportPaths` / `useNoCMakeCalls` 与构建生成的 `.qmlls.ini` 表达的是同一组意图。两者保持一致时，无论从命令行还是配置文件读取，qmlls 行为都一样。**首次使用前请至少完整构建一次**，以生成 `.qmlls.ini` 并产出类型信息。

#### 需要按本机修改的项

以下几处是与本机 Qt 安装强相关的**绝对路径**，请改成你自己的：

- `qt-qml.qmlls.customExePath` → 你的 `qmlls(.exe)` 路径
- `qt-qml.qmlls.customArgs` 里的 `-d...` → 你的 Qt 文档目录
- 若不在 Windows，构建目录（`build/debug`）通常无需改，但 qmlls 路径需换成对应平台的可执行文件。

---

## 其它 `.vscode/` 配置

### `extensions.json` — 推荐扩展

打开工程时 VS Code 会提示安装：

| 扩展 | 用途 |
| --- | --- |
| `llvm-vs-code-extensions.vscode-clangd` | C++ 语言服务器（补全 / 跳转 / 格式化）。 |
| `ms-vscode.cmake-tools` | CMake 配置 / 构建 / 调试集成。 |
| `ms-vscode.cpptools` | 微软 C/C++ 扩展（此处主要用于调试，IntelliSense 已禁用）。 |
| `twxs.cmake` | `CMakeLists.txt` 语法高亮。 |
| `vadimcn.vscode-lldb` | LLDB 调试器（CodeLLDB）。 |
| `xaver.clang-format` | clang-format 支持。 |

> QML 智能感知所需的 Qt 官方扩展 `theqtcompany.qt-qml` 请单独安装。

### `launch.json` — 调试

```jsonc
{
  "name": "Debug (LLDB)",
  "type": "lldb",
  "request": "launch",
  "program": "${command:cmake.launchTargetPath}",
  "preLaunchTask": "CMake: Build Linux Debug"
}
```

用 CodeLLDB 启动 CMake 当前选中的目标，调试前自动执行构建任务。

### `tasks.json` — 构建任务

封装了 CMake Presets 的配置 / 构建命令（Linux Debug/Release），默认构建任务为 **CMake: Build Linux Debug**，可用 <kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>B</kbd> 触发。Windows 用户可直接用上文的 `--preset windows-debug` 命令，或在 CMake Tools 中选择 `windows-debug` 预设。

---

## 常见问题

- **QML 文件全是红波浪线 / 无补全**：确认已完整构建一次（生成 `build/debug` 与 `.qmlls.ini`），且 `customExePath` 指向的 qmlls 与工程 Qt 版本一致。
- **C++ 出现重复报错**：确认 `C_Cpp.intelliSenseEngine` 为 `disabled`，避免微软引擎与 clangd 同时分析。
- **clangd 找不到系统头文件**：先构建一次以生成 `compile_commands.json`（会同步到 `build/`）。
- **改了 Qt 路径不生效**：修改 `settings.json` 后重启 qmlls / clangd（命令面板里有 “Restart” 命令）或重载窗口。

---

## License

样例应用源码沿用其上游许可证，详见仓库内 `LICENSE` 与各资源目录下的授权文件。本 Demo 的 `.vscode/` 配置可自由复制到你自己的 Qt/QML 工程中使用。
