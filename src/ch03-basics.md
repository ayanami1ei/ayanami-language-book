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

## 基本类型

Ayanami 提供了几种基本数据类型，它们在程序中广泛使用：

- **整数（int）**：64 位有符号整数。
    ```ayanami
    x = 42
    ```

- **浮点数（float）**：64 位双精度浮点数。
    ```ayanami
    f = 3.14
    ```

- **字符（char）**：单字节字符。
    ```ayanami
    c = 'X'
    ```

- **布尔值（bool）**：`true` 或 `false`。
    ```ayanami
    b = true
    ```

- **字符串（String）**：虽然不是基本类型，但它是标准库中的重要结构体。
    ```ayanami
    s = "hello"
    ```

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

## 语法陷阱

在学习 Ayanami 时需要注意两个常见的语法陷阱。

### 不支持括号表达式

Ayanami 不允许使用括号来改变运算优先级，例如：

```ayanami
// ❌ 错误写法
a = b * (c + d)
```

必须改写为中间变量的形式：

```ayanami
temp = c + d
a = b * temp
```

### 负数直接量不能解析

不能直接写 `-1` 这样的负数，而应使用减法表达式：

```ayanami
// ❌ 错误写法
x = -1
// ✅ 正确写法
x = 0 - 1
```

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
