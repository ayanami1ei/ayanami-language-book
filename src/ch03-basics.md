# 第 3 章 基本编程概念

在 Ayanami 中编写程序时，我们首先需要理解一些基本的编程概念。这些概念包括变量、类型、运算符、控制流以及注释等。本章将带你逐步了解这些内容，并通过示例代码展示它们如何在实际中使用。

## 变量与赋值

Ayanami 中的变量声明非常简洁，不需要像 `let` 这样的关键字。你可以直接写变量名并赋值，例如：

```ayanami
x = 42
```

这里，变量 `x` 被赋予了整数类型（`int`）的值 `42`。Ayanami 会自动推断变量的类型。如果后续对变量进行重新赋值，可以直接写赋值语句：

```ayanami
x = x + 1
```

这表示将变量 `x` 的当前值加一后，再赋回给它。这种语法简单直观，避免了复杂的声明与初始化过程。

## 常量与全局变量

`const` 是编译期常量，不占运行时存储，可用于数组大小：

```ayanami
const MAX_SIZE = 16
const MAGIC = 0x12345678
const MASK = (1 << 8) - 1
const PI: float = 3.14159

buf = [int; MAX_SIZE]
```

- 顶层声明，`pub` 与类型标注可选，行尾 `;` 可选；
- 初始化式必须是编译期常量（字面量、一元/二元运算、引用此前声明的 const）；
- 编译期求值并内联到使用点；局部变量可以遮蔽同名 const。

`static` 是可寻址的全局存储（常量初始化）：

```ayanami
static MAX = 10
static mut COUNTER = 0

fn bump() -> int {
    COUNTER = COUNTER + 1
    return COUNTER
}
```

- `static` 不可变，`static mut` 可写（赋值穿透引用写回）；
- 可用 `ref` / `ref mut` 取全局的引用，传给需要借用的函数；
- 全局值不能包含堆所有权（`String` / 动态数组）；需要表时用固定大小数组 `[T; n]`；
- `pub static` 可跨模块导出（`.lcl` 携带类型，定义在被导入包）。

`#[compile_time]` 标注的函数既可以运行期调用，也可以在 const / static 初始化式里被编译期解释执行：

```ayanami
#[compile_time]
fn square(int x) -> int { return x * x }

const S = square(12)      // 编译期求值
```

编译期求值支持标量、局部变量、`if` / `while` / `for`、块尾表达式与 const fn 互调（递归），
不支持堆 / 字符串 / 方法调用。详见主仓 `docs/const-globals.md`。

## 基本类型

Ayanami 的基本类型如下：

| 类型 | 说明 |
| --- | --- |
| `int` | 64 位有符号整数（`int` ≡ `i64` ≡ `isize`，同一类型） |
| `i8` / `i16` / `i32` / `i64` / `i128` | 定宽有符号整数 |
| `u8` / `u16` / `u32` / `u64` / `u128` | 定宽无符号整数（`u64` ≡ `usize`） |
| `float` / `f64` | 64 位浮点（同一类型） |
| `f32` | 32 位浮点 |
| `char` | 单字节字符（当前仅 ASCII） |
| `bool` | `true` / `false` |
| `()` / `void` | 单元类型别名（无值） |

```ayanami
x = 42          // int
hex = 0xFF      // 255
bin = 0b1010    // 10
oct = 0o17      // 15
big = 1_000_000 // 下划线分隔
narrow = 200u8  // 带后缀的定宽整数
f = 3.14        // float
f32v = 1.5f32   // 32 位浮点
c = 'X'
b = true
s = "hello"     // 标准库的 String 结构体
```

- 无后缀整数默认 `int`，无后缀浮点默认 `float`；
- 字面量支持 `0x` / `0b` / `0o` 进制、下划线分隔与指数（`1e3`、`1.5e-3`）；
- 后缀（`1u8`、`100usize`、`1.5f32`、`3f64`）直接指定字面量类型，整数后缀会做范围检查；
- 无后缀字面量会自动适配期望类型（如传给 `f32` 形参的 `1.5`）。

## 运算符

Ayanami 支持常见的运算符，包括算术、比较和逻辑运算。

### 算术运算符

```ayanami
a = 10 + 5   // 加法
b = 10 - 5   // 减法
c = 10 * 5   // 乘法
d = 10 / 5   // 除法
e = 10 % 3   // 取模
```

### 比较运算符

```ayanami
if x == 42 { ... }     // 等于
if x != 42 { ... }     // 不等于
if x < 42 { ... }      // 小于
if x > 42 { ... }      // 大于
if x <= 42 { ... }     // 小于等于
if x >= 42 { ... }     // 大于等于
```

### 逻辑运算符

```ayanami
if a && b { ... }      // 与
if a || b { ... }      // 或
if !a { ... }          // 非
```

### 字符串拼接

Ayanami 支持字符串拼接操作，使用 `+` 运算符即可：

```ayanami
println("value: " + 42)
```

这里的 `42` 会被自动转换为字符串并拼接到 `"value: "` 后面。这种行为依赖于类型实现的 `to_string()` 方法。

## 注释

Ayanami 支持两种注释方式：

- **行注释**：以 `//` 开头，直到行尾的内容都会被忽略。
    ```ayanami
    // 这是一条注释
    x = 42 // 这个变量是 42
    ```

- **块注释**：使用 `/* ... */` 包裹内容，可以跨越多行。
    ```ayanami
    /*
     * 多行注释
     * 可以写很多内容
     */
    ```

## 控制流

Ayanami 提供了常见的控制结构来处理程序逻辑。

### 条件语句：`if / elif / else`

```ayanami
if x > 42 {
    println("bigger")
} elif x == 42 {
    println("equal")
} else {
    println("smaller")
}
```

`if` 还是**表达式**：每个分支的块尾值就是它的值，可以直接赋给变量或 `return`：

```ayanami
level = if score > 90 { 3 } elif score > 60 { 2 } else { 1 }
return if x > 0 { x } else { -x }
```

分支类型不同时会向公共类型提升（如 `int` 分支与 `float` 分支 → `float`）。

### 循环语句：`while` 和 `for`

#### while 循环

```ayanami
n = 0
while n < 5 {
    n = n + 1
    if n == 3 { continue }
    if n == 5 { break }
}
```

#### for 循环（左闭右开区间）

```ayanami
sum = 0
for i in (0, 10) {
    sum = sum + i
}
```

这里 `i` 的取值范围是 `[0, 10)`，即从 0 到 9。

### break 和 continue

在循环中可以使用 `break` 跳出当前循环，或者用 `continue` 跳过本次迭代：

```ayanami
if n == 3 { continue }
if n == 5 { break }
```

## 运算符与类型细节

### 括号与优先级

括号可以用来改变运算优先级：

```ayanami
a = 2 * (3 + 4)   // 14
```

运算符优先级从紧到松大致为：`* / %` > `+ -` > `<< >>` > `&` > `^` > `|` > 比较 > `&&` > `||`。

### 负数

负数可以直接写，一元负号也适用于变量与表达式：

```ayanami
a = -5
b = -5.0
c = 3 - -5     // 8
x = 4
d = -x         // -4
e = x * -2     // -8
```

### 没有隐式数值转换

不同类型的数值之间**不会自动提升**，需要显式 `as` 转换：

```ayanami
x = 4
y = (x as float) / 4.0    // 1.0
g = ('a' as int) + 1      // 98
```

例外：**无后缀字面量**会适配期望类型，所以 `1 + 2.5` 这类写法可以直接用；
变量之间的混合运算（如 `x + 2.5`，`x` 是 `int`）必须写 `x as float`。
转换语义（截断/饱和、按符号扩展等）见主仓 `docs/int-types.md`。

## 字符串基础

Ayanami 中的字符串拼接非常方便。你可以将任意实现了 `ToString` 接口的类型与字符串相加：

```ayanami
println("sum=" + sum)
```

在这个例子中，`sum` 是一个整数，它会被自动转换为字符串后拼接到 `"sum="` 后面。

## 示例程序

下面是一个完整的示例程序，展示了上述所有概念的实际应用：

```ayanami
import "io"

fn main() -> int {
    x = 42
    f = 3.14
    c = 'X'
    b = true
    s = "hello"
    x = x + 1
    if x > 42 {
        println("bigger")
    } elif x == 42 {
        println("equal")
    } else {
        println("smaller")
    }
    sum = 0
    for i in (0, 10) {
        sum = sum + i
    }
    n = 0
    while n < 5 {
        n = n + 1
        if n == 3 { continue }
        if n == 5 { break }
    }
    println("sum=" + sum)
    return 0
}
```

这段代码演示了变量赋值、条件判断、循环结构以及字符串拼接等基本编程概念。

## 预告

在下一章中，我们将深入探讨函数和命名空间的概念。你将学会如何定义和调用函数，并了解如何使用命名空间组织代码结构。
