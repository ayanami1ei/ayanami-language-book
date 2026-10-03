# 第 11 章 标准库

Ayanami 语言提供了丰富的标准库支持，帮助开发者快速构建功能完备的应用程序。这些库分布在不同的模块中，通过 `import` 语句导入使用。本章将介绍常用的几个标准库模块：`string`、`io`、`math`、`std`、`arraylist` 和 `list`。

## 导入标准库

在 Ayanami 中，使用 `import "模块名"` 来引入标准库中的功能。例如：

```ayanami
import "string"
import "io"
import "math"
import "arraylist"
```

这些语句会自动从标准库目录中查找并加载对应的模块。

## string 模块

`string` 模块提供了字符串相关的结构体和方法。其核心类型是 `String`，它是一个包含字符数组和长度的结构体：

```ayanami
struct String {
    [char] data
    int len
}
```

你可以使用 `len()` 方法获取字符串长度，用 `index(i)` 获取指定位置的字符。此外，支持字符串拼接操作符 `+` 和比较运算符 `eq` 与 `ne`。

```ayanami
s = "Hello"
t = s + " World"
println(t) // 输出 Hello World
println("len=" + t.len()) // 输出 len=11
```

所有基本类型如 `int`、`float`、`char` 和 `bool` 都实现了 `to_string()` 方法，可以方便地转换为字符串：

```ayanami
println("val: " + 42) // 输出 val: 42
```

## io 模块

`io` 模块提供了输入输出功能。最常用的函数是 `println()` 和 `print()`，分别用于打印换行和不换行输出。

```ayanami
println("Hello, world!")
print("Enter your name: ")
```

此外，还有 `putchar(int)` 用于输出单个字符，`getchar() -> int` 用于读取一个字符：

```ayanami
c = getchar()
putchar(c)
```

## math 模块

`math` 模块提供了一些常用的数学函数。这些函数接受整数参数，并返回整数结果。

- `abs(x)`：计算绝对值。
- `min(a, b)`：返回较小值。
- `max(a, b)`：返回较大值。
- `clamp(x, min, max)`：将 x 限制在 [min, max] 范围内。
- `pow(base, exp)`：计算幂次。

```ayanami
println("max=" + max(3, 7)) // 输出 max=7
```

## std 模块

`std` 模块定义了错误处理机制的基础接口和类型，例如 `Error` 接口和 `Result[T, E]` 类型。这部分将在下一章详细讲解。

## 集合类型：ArrayList 与 List

Ayanami 提供了动态数组结构 `ArrayList[T]`，它支持添加元素、访问元素以及遍历操作。要使用 `ArrayList`，需要先导入 `arraylist` 模块。

```ayanami
import "arraylist"
```

创建一个 `ArrayList[int]` 实例并初始化：

```ayanami
a = ArrayList[int] { data = null, len = 0, capability = 0 }
```

然后可以使用 `push()` 添加元素，通过 `index(i)` 访问元素，用 `len()` 获取长度，并调用 `iter(fn)` 对每个元素执行操作。

```ayanami
a.push(10)
a.push(20)
a.push(30)
a.iter((int x) { println("item: " + x) })
```

上面这段代码会依次输出：

```
item: 10
item: 20
item: 30
```

注意：当前编译器对 `ArrayList[String]` 存在已知问题，因此示例中使用了 `ArrayList[int]`。

## 完整示例

下面是一个完整的程序，展示了如何使用上述标准库功能：

```ayanami
import "string"
import "io"
import "math"
import "arraylist"

fn main() -> int {
    s = "Hello"
    t = s + " World"
    println(t)
    println("len=" + t.len())
    println("max=" + max(3, 7))
    println("val: " + 42)

    a = ArrayList[int] { data = null, len = 0, capability = 0 }
    a.push(10)
    a.push(20)
    a.push(30)
    a.iter((int x) { println("item: " + x) })
    return a.index(1) - 20
}
```

这段程序首先构造了一个字符串并打印出来，接着计算两个数的最大值，并将整数转换为字符串进行拼接。随后创建了一个整型数组列表，添加了三个元素，并遍历输出所有元素。最后返回第二个元素减去 20 的结果。

## 小结

本章介绍了 Ayanami 标准库中常用的模块和功能，包括字符串处理、输入输出、数学运算以及集合类型。这些工具为编写实用程序提供了坚实的基础。下一章我们将深入探讨错误处理机制，了解如何使用 `Result[T, E]` 类型来优雅地处理可能失败的操作。
