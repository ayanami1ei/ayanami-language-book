# 第 16 章 工具链与发布

本章把散落在前面的工具串起来：命令行、项目配置、格式化、打包，以及编辑器支持。

## 16.1 命令行总览

| 命令 | 作用 |
| --- | --- |
| `ayanami new <name>` | 创建新项目 |
| `ayanami check [--watch] <file/proj>` | 前端检查；`--watch` 监听文件变更自动重检 |
| `ayanami fmt [file]` | 格式化代码 |
| `ayanami package <file/proj>` | 打包为 `.lcl`（不生成可执行文件） |
| `ayanami build [file/proj]` | 构建可执行文件 + `.lcl` 包 |
| `ayanami run [file/proj]` | 构建并运行 |
| `ayanami install <lcl>` | 从 `.lcl` 构建目标产物 |
| `ayanami defs <file>` | 输出符号定义列表（JSON） |
| `ayanami types <file>` | 输出 HIR 推断的变量类型（JSON，供编辑器类型提示） |
| `ayanami clean` | 清除 `build/` 目录 |

构建产物默认放在项目根目录的 `build/` 下。

## 16.2 项目配置 `ayanami.toml`

单文件可以直接编译，但正式项目建议用 `ayanami.toml` 描述构建目标：

```toml
[package]
name = "my_project"
version = "0.6.0"

[build]
target = "executable"

[build.targets]
"src/lib.aya" = "dynamic-lib"
"src/utils.aya" = "static-lib"
```

- `[package]`：项目名与版本号；
- `[build] target`：默认产物类型，可选 `executable`、`static-lib`、`dynamic-lib`；
- `[build.targets]`：为单个源文件指定不同的产物类型。

`ayanami run`、`ayanami build`、`ayanami package` 在项目模式下都会读取这份配置。

## 16.3 格式化与检查

```bash
ayanami fmt src/main.aya      # 就地格式化
ayanami fmt                   # 项目模式：格式化全部源文件
ayanami check --watch src/    # 保存即检查
```

`ayanami check` 只走前端（词法、语法、HIR），速度很快，适合放进编辑器保存钩子。
`ayanami defs` 输出 JSON 形式的符号表，可供 IDE 或文档工具消费：

```bash
ayanami defs src/main.aya
```

## 16.4 VSCode 插件

发布包中带有 `ayanami-0.6.0.vsix`：

```bash
code --install-extension ayanami-0.6.0.vsix
```

插件提供语法高亮、结构体字段补全、方法补全、变量类型推断、
保存/打开时错误检查、hover 文档和 Run CodeLens。

## 16.5 打包与发布

- `ayanami package file.aya`：把源码编译为 `.lcl` 包（含预编译 IR 与符号信息）；
- `ayanami install pkg.lcl`：在目标机器上从 `.lcl` 生成可执行文件或库；
- `./scripts/package_release.sh`：维护者用的发布脚本，产出 `tar.gz` 与 `.vsix`。

维护者在提交前运行仓库不变量检查：

```bash
./scripts/check_all.sh      # 版本 + 行数 + 符号地图 + 零告警 + 语言回归 + IR 快照
./scripts/regression.sh     # 单独跑语言正/负回归（check_all 已包含）
./scripts/ir_snapshot.sh    # 校验各阶段 IR 快照（--update 生成）
```

- `regression.sh`：正例按 `tests/positive_exit.txt` 校验退出码，负例在 `tests/compile_fail/` 校验报错子串；
- `ir_snapshot.sh`：对比 `tests/ir_snapshots/` 下 ast/hir/mir/lir 各阶段输出，防止优化与降级悄悄漂移；
- 设置 `AYANAMI_SKIP_REGRESSION=1` 可跳过 `check_all.sh` 中的回归与快照。

`.lcl` 是 Ayanami 的分发格式，标准库就是以预编译 `.lcl` 形式随编译器分发的
（见 `std/*.lcl`）。导入方式：

```ayanami
import "math_lib.lcl";
```

## 16.6 编译管线回顾

```text
源码 → Lexer → Parser(AST) → HIR → MIR → LIR → LLVM IR → .o → 可执行文件
```

- **HIR**：高层 IR，保留类型与结构化控制流；
- **MIR**：中层 IR，插入内存操作与借用检查（`mir/borrow/`）；
- **LIR**：低层 IR，接近 LLVM，负责名字修饰与字符串表；
- **emit**：发射 LLVM IR，交给 `llc` 编译成目标文件，再由 `gcc` 链接。

理解这条管线有助于读懂错误信息：例如「hir error」是类型/名称解析阶段，
「borrow error」来自 MIR 借用检查，而「llc failed」说明问题出在 LLVM IR 生成或优化阶段。

## 16.7 小结

- 单文件用 `ayanami run/check`，工程用 `ayanami.toml` + 项目模式；
- `fmt` 与 `check --watch` 改善日常开发体验；
- `.lcl` 是分发单元，标准库与第三方库都走这套机制。

下一章是全书收官：用递归下降解析器写一个表达式计算器。
