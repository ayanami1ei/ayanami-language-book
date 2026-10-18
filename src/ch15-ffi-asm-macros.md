# 第 15 章 FFI、内联汇编与宏

在本章中，我们将介绍 Ayanami 语言中用于与外部代码交互的特性：外部函数接口（FFI）、内联汇编以及实验性的宏系统。这些功能允许你调用 C 函数、编写底层指令，并在编译期生成代码。

## 外部函数接口（FFI）

Ayanami 支持通过 C ABI 标注声明外部函数。新语法用 `#[extern]` 标注，旧语法 `extern "C"` 完全等价：

```ayanami
#[extern]
fn strlen([char] s) -> int;

// 等价写法：extern "C" fn strlen([char] s) -> int;
```

这个声明告诉编译器：有一个名为 `strlen` 的 C 函数，接受一个字符数组（即 `[char]` 类型），返回一个整数。一旦声明完成，你就可以直接调用它：

```ayanami
fn main() -> int {
    s = "hello"
    return strlen(s.data) - 5
}
```

这里我们把字符串 `s` 的 `.data` 字段传入了 `strlen`。因为 Ayanami 中的字符串结构体内部就是 `[char]` 类型，所以可以直接传递。

你还可以声明不返回的函数：

```ayanami
#[extern]
fn abort();
```

这种函数通常用于终止程序执行，例如在错误处理中调用 C 的 `abort()` 函数。

## 导出 C 符号（`#[export]`）

反向的互操作：让 C 调用 Ayanami。`#[export]` 定义函数并保留原始符号名（不加修饰）：

```ayanami
#[export]
fn aya_add(int a, int b) -> int { return a + b }
```

C 侧声明：`int64_t aya_add(int64_t, int64_t);`

类型映射（当前 LLVM 后端）：

| Ayanami | C 侧建议 |
|---|---|
| `int` / `i64` | `int64_t` |
| `i8`…`i32` / `u8`…`u32` | `intN_t` / `uintN_t` |
| `float` | `double` |
| `f32` | `float` |
| `char` | `char`（当前仅 ASCII） |
| `bool` | `_Bool`（跨边界建议用 `int`） |
| `ref T` / `ref mut T` | `const T*` / `T*`（出参用 `ref mut`） |

```ayanami
#[export]
fn aya_fill(ref mut int dst, int v) -> void {
    dst = v
}
```

限制：`#[export]` 不能用于泛型函数、结构体/枚举/接口/impl 块；与 libc 同名会覆盖（有意为之）。
完整说明见主仓 `docs/c-interop.md`。

## 自定义 runtime

项目可以通过 `ayanami.toml` 或环境变量替换内置 `runtime.c`：

```toml
[runtime]
path = "custom_runtime.c"   # .c / .a / .o，相对项目根
```

```bash
AYANAMI_RUNTIME=/path/to/runtime.a ayanami build   # 环境变量优先
```

这意味着你可以用 Ayanami 自己写运行时并打包成 `libruntime.a`。

## 标注与优化

为了帮助编译器更好地优化跨语言调用，你可以为外部函数添加各种标注。这些标注会传递给 LLVM 编译器后端，从而提升性能或保证语义正确性。

例如：

```ayanami
#[pure]
#[nounwind]
extern "C" fn strlen([char] s) -> int;
```

- `#[pure]` 表示该函数没有副作用，只依赖于参数。
- `#[nounwind]` 表示函数不会抛出异常。
- `#[noreturn]` 表示函数从不返回（如 `abort()`）。
- `#[nonnull]` 表示参数不能为 null。
- `#[noalias]` 表示指针不会指向相同内存区域。

这些标注不仅有助于优化，也增强了代码的可读性和安全性。

## 内联汇编

Ayanami 支持内联汇编语法，允许你在函数中嵌入机器指令。汇编属于机器级操作，**必须放在
`unsafe` 上下文**（`unsafe fn` 或 `unsafe {}`）中（见第 18 章）：

```ayanami
unsafe fn asm_demo(int a) -> int {
    result = 0
    asm("mov $1, $0", in(reg) a, out(reg) result)
    return result
}
```

这段代码将变量 `a` 的值移动到 `result` 中。其中：
- `"mov $1, $0"` 是汇编指令；
- `in(reg) a` 表示输入参数 `a` 被放入寄存器中；
- `out(reg) result` 表示输出结果保存在寄存器中。

占位符 `$0` 和 `$1` 分别对应第一个和第二个操作数。你可以使用多个输入输出约束来实现更复杂的汇编逻辑。

## 宏系统（实验特性）

Ayanami 提供了一个实验性的宏系统，允许你在编译期生成源码。宏函数必须定义在单独的库文件中，并标记为 `#[macro]`。

例如，在一个名为 `macro_lib.aya` 的文件中：

```ayanami
// 宏库：编译期执行，返回要替换进去的源码字符串
#[alloc]
#[macro]
pub fn answer() -> String { return "fn answer() -> int { return 42 }" }
```

这个宏函数 `answer()` 返回一个字符串 `"fn answer() -> int { return 42 }"`。在使用方文件中，你可以通过如下方式引入并使用它：

```ayanami
import "macro_lib.aya"

#[macro_lib::answer]
fn placeholder() -> int { return 0 }

fn main() -> int { return answer() - 42 }
```

编译器会在编译期将 `#[macro_lib::answer]` 占位函数替换为宏返回的源码。因此，最终生成的代码等价于：

```ayanami
fn answer() -> int { return 42 }

fn main() -> int { return answer() - 42 }
```

### token 宏（结构化）

除文本宏外，宏库可以用 `#[macro(tokens)]` 定义 **token 宏**：输入/输出是 TokenStream
（含 span 与分组），编译器负责编解码，宏侧不需要拼接源码字符串：

```ayanami
#[macro(tokens)]
pub fn tok_twice(input: TokenStream) -> TokenStream { /* ... */ }
```

配套能力：

- **hygiene 原语**：`__ayanami_macro_hygienic(name)` 给宏内定义名加确定性后缀，避免捕获或泄漏调用点名字；
- **诊断通道**：`__ayanami_macro_diag(level, msg)` 产生带调用点定位的 warn / error；
- **展开调试**：`ayanami dump expanded <file>` 输出展开后的源码，`AYANAMI_MACRO_TRACE=1` 追踪展开过程。

宏系统仍处于演进阶段，接口可能变化；完整设计见主仓 `docs/macros.md`。

## 示例程序

下面是一个结合了 FFI、内联汇编和宏使用的完整示例：

```ayanami
#[pure]
#[nounwind]
#[extern]
fn strlen([char] s) -> int;

#[noreturn]
#[extern]
fn abort();

#[inline]
fn fast() -> int { return 2 }

unsafe fn asm_demo(int a) -> int {
    result = 0
    asm("mov $1, $0", in(reg) a, out(reg) result)
    return result
}

fn main() -> int {
    s = "hello"
    x = 0
    unsafe { x = asm_demo(7) }
    return strlen(s.data) - 5 + fast() - 2 + x - 7
}
```

该程序调用了 C 标准库中的 `strlen`，在 `unsafe fn` 中用内联汇编移动寄存器值，并通过 `#[inline]` 提示优化函数调用。

## 预告

在下一章中，我们将介绍 Ayanami 的工具链与发布流程。你将学习如何配置项目、构建可执行文件以及打包发布程序。
