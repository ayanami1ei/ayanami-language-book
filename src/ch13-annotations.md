# 第 13 章 标注系统

在 Ayanami 语言中，标注（annotation）是一种特殊的语法结构，使用 `#[...]` 表示。它和注释不同，不是简单的代码注解，而是编译器会解析并用于优化、验证或生成代码的重要信息。

标注可以出现在函数、结构体字段、参数甚至语句级别上。它们为编译器提供了额外的语义信息，帮助进行静态分析、性能优化、错误检查以及控制代码行为。例如，你可以通过标注告诉编译器某个函数应该被内联展开，或者某个条件在程序执行时总是成立。

## 优化标注

优化标注用于向编译器提供关于函数或参数的行为提示，以辅助编译器进行更高效的代码生成。

### `#[inline]`

`#[inline]` 提示编译器应尽可能将函数调用替换为函数体本身，避免函数调用开销。这通常用于频繁调用的小函数。

```ayanami
#[inline]
fn add(int a, int b) -> int { return a + b }
```

### `#[cold]`

`#[cold]` 用于标记那些很少执行的函数。编译器会将这些代码放在单独的段中，并优化其空间占用，而不是优先考虑速度。

```ayanami
#[cold]
fn rarely() -> int { return 1 }
```

### `#[willreturn]`、`#[pure]`、`#[nounwind]`、`#[noreturn]`

这些标注分别表示函数不会抛出异常（`#[nounwind]`）、不会返回（`#[noreturn]`）、总是返回一个值（`#[willreturn]`）或纯函数（无副作用，仅依赖输入参数）。

```ayanami
#[pure]
fn square(int x) -> int { return x * x }

#[noreturn]
extern "C" fn abort();
```

### `#[nonnull]` 与 `#[noalias]`

这两个标注通常用于 `extern` 声明或函数参数上，表示该指针不为空（`#[nonnull]`）或不与其他指针共享内存区域（`#[noalias]`）。这有助于编译器进行更激进的优化。

```ayanami
extern "C" fn memcmp(
    #[nonnull]
    #[noalias]
    [char] a,
    #[nonnull]
    #[noalias]
    [char] b,
    int n
) -> int;
```

## 条件编译

条件编译允许你根据目标平台或构建配置来决定是否包含某段代码。这在跨平台开发中非常有用。

### `#[cfg(target = "...")]`、`#[cfg(unix)]`、`#[cfg(!windows)]`

这些标注用于控制代码的编译行为，只有满足条件时才会编译该部分代码。

```ayanami
#[cfg(target = "linux")]
fn on_linux() -> int { return 0 }

#[cfg(unix)]
fn on_unix() -> int { return 1 }

#[cfg(!windows)]
fn not_windows() -> int { return 2 }
```

### 语句级条件编译

你也可以在语句级别使用 `#[cfg(...)]`，以控制某条语句是否参与编译。

```ayanami
fn example() {
    #[cfg(target = "windows")]
    println("Windows only")
}
```

## 契约标注

契约标注用于声明函数的前置条件、后置条件或循环不变式。这些标注可以被编译器验证，也可以在运行时进行检查。

### `#[requires(cond)]`：前置条件

前置条件表示调用函数前必须满足的条件。如果条件不成立，程序会报错（默认启用）。

```ayanami
#[requires(n > 0)]
fn dec(int n) -> int { return n - 1 }
```

### `#[ensures(result ...)]`：后置条件

后置条件表示函数返回时必须满足的条件。`result` 是绑定的返回值变量。

```ayanami
#[ensures(result >= 0)]
fn abs2(int x) -> int {
    if x < 0 { return -x }
    return x
}
```

### `#[invariant(cond)]`：循环不变式

用于标记循环体中始终成立的条件。编译器会验证该条件在每次迭代中是否保持。

```ayanami
n = 0
#[invariant(n >= 0)]
while n < 3 {
    n = n + 1
}
```

### `#[assume(cond)]`：假设

用于向优化器声明某个条件总是成立，编译器不会检查它。这在某些复杂逻辑中可以提升性能。

```ayanami
#[assume(x < 1000)]
fn id(int x) -> int { return x }
```

### 默认运行时检查与 `AYANAMI_CHECKS=0`

默认情况下，所有契约标注都会在运行时被验证。你可以通过设置环境变量 `AYANAMI_CHECKS=0` 来关闭这些检查，此时编译器会将 `requires` 和 `ensures` 转换为 LLVM 的 `assume` 指令。

## 效应标注

效应标注用于描述函数的行为特征，例如是否涉及 I/O、内存分配或抛出异常等。它们有助于编译器进行更精确的优化和错误检测。

### `#[io]`、`#[state]`、`#[alloc]`

这些标注分别表示函数会执行 I/O 操作、修改状态或进行内存分配。

```ayanami
#[io]
fn read_file() -> String { return "file content" }

#[alloc]
fn create_array(int size) -> [int] {
    a = [int; size]
    return a
}
```

### `#[throws(E)]`：可能抛出异常

用于声明函数可能抛出某种类型的错误。

```ayanami
#[throws(ParseError)]
fn parse(int x) -> int { return x }
```

### `#[pure]` 与 `#[no_error]`

`#[pure]` 表示函数是纯函数，无副作用；`#[no_error]` 表示函数不会抛出错误。

```ayanami
#[pure]
fn add(int a, int b) -> int { return a + b }

#[no_error]
fn safe_function() -> int { return 42 }
```

## 完整示例

下面是一个综合使用多种标注的示例程序：

```ayanami
#[cfg(target = "linux")]
fn on_linux() -> int { return 0 }

#[requires(n > 0)]
fn dec(int n) -> int { return n - 1 }

#[ensures(result >= 0)]
fn abs2(int x) -> int {
    if x < 0 { return -x }
    return x
}

#[assume(x < 1000)]
fn id(int x) -> int { return x }

#[inline]
fn add(int a, int b) -> int { return a + b }

fn main() -> int {
    n = 0
    #[invariant(n >= 0)]
    while n < 3 {
        n = n + 1
    }
    return on_linux() + dec(5) - 4 + abs2(-3) - 3 + id(1) - 1 + add(1, 2) - 3 + n - 3
}
```

## 预告

在下一章中，我们将深入讲解 Ayanami 的生命周期系统与 `follow_with` 标注。这将帮助你更好地理解引用和所有权的细节，以及如何在函数参数中声明引用的生命周期关系。
