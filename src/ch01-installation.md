# 第 1 章 安装与第一个程序

欢迎来到 Ayanami！本章我们将安装编译器、运行第一个程序，并认识 `ayanami` 命令行工具。

## 1.1 安装

Ayanami 目前支持 **Linux x86_64**。编译器自带 LLVM 后端，**不需要预装 LLVM**，
只需要系统有 `gcc`（用于链接）和 glibc。

从 [GitHub Releases](https://github.com/ayanami1ei/Ayanami-language/releases) 下载发布包：

```bash
tar xzf ayanami-0.6.0-linux-x86_64.tar.gz
cd install
```

`install/` 目录的结构如下：

```text
install/
├── ayanami           # 编译器本体
├── llc               # LLVM 静态编译器（bundled）
├── libLLVM.so.21.1   # LLVM 共享库
├── libedit.so.2      # libLLVM 依赖（打包兼容副本）
├── runtime.c         # 运行时（libc 包装、RC 分配器）
└── std/              # 标准库（预编译 .lcl）
    ├── string.lcl
    ├── io.lcl
    └── math.lcl
```

编译器会自动在同目录查找 `llc`、`std/` 和 `runtime.c`，所以请保持目录完整。
验证安装：

```bash
./ayanami run ../example/test_struct.aya
```

## 1.2 Hello, World!

新建 `hello.aya`：

```ayanami
import "io"

fn main() -> int {
    println("Hello, World!")
    println("value: " + 42)
    return 0
}
```

运行：

```bash
./ayanami run hello.aya
```

输出：

```text
Hello, World!
value: 42
```

这里有几个值得注意的点：

- `import "io"` 导入标准库的输入输出模块，`println` 来自它；
- `fn main() -> int` 是程序入口，返回值是退出码；
- `println("value: " + 42)` 展示了 Ayanami 的 `ToString` 机制：
  `String + int` 会自动把整数转成字符串。

只想做前端检查、不生成可执行文件时，用：

```bash
./ayanami check hello.aya
# check passed: hello.aya
```

## 1.3 创建新项目

`ayanami new` 会生成一个带配置文件的工程目录：

```bash
./ayanami new my_project
cd my_project
```

项目里有一个 `ayanami.toml`：

```toml
[package]
name = "my_project"
version = "0.6.0"

[build]
target = "executable"
```

以及 `src/main.aya`。项目模式下（在含 `ayanami.toml` 的目录里执行命令），
编译器会自动找到源码和配置：

```bash
./ayanami run       # 构建并运行
./ayanami build     # 构建可执行文件 + .lcl 包
./ayanami check     # 前端检查
```

## 1.4 从源码构建编译器

如果你想参与编译器开发，或者发布包不适用于你的环境：

```bash
git clone git@github.com:ayanami1ei/Ayanami-language.git
cd Ayanami-language
cargo build --release
```

开发版编译器使用系统 `llc`（需要 LLVM 21 工具链），并通过 `src/runtime.c` 链接。
仓库里还提供了常用脚本：

```bash
./scripts/check_all.sh         # 版本号 + 文件行数 + 符号地图 + 零告警
./scripts/package_release.sh   # 构建 tar.gz 与 VSCode 插件
```

## 1.5 小结

- 解压发布包后保持 `install/` 目录完整，编译器会自动查找 `llc` 与 `std/`；
- `ayanami run file.aya` 是构建并运行，`ayanami check` 只做前端检查；
- `ayanami new` 生成带 `ayanami.toml` 的项目骨架。

下一章我们用一个完整的猜数字游戏，快速过一遍 Ayanami 的常用语法。
