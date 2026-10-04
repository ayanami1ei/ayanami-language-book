# 第 11 章 标准库

Ayanami 语言内置了丰富的标准库模块，帮助你快速构建程序。这些模块以 `.lcl` 文件形式预编译并存放于编译器目录下的 `std/` 文件夹中。使用 `import "模块名"` 即可导入所需功能。

## 总览

当前标准库包含以下主要模块：

- `io`：输入输出操作，如 `print`、`println`、`putchar` 和 `getchar`
- `math`：数学函数，包括整数与浮点运算
- `string`：字符串处理功能
- `std`：主入口模块，整合了 `io`、`string`、`math` 等，并提供 `Error` 接口、`Result[T, E]` 和 `Option[T]`
- `list`：集合接口定义
- `arraylist`：顺序表实现
- `linkedlist`：链表实现

此外，还有一个实验性模块 `mir`，用于插件开发，详情请见附录 A。

## 输入输出（io）

`io` 模块提供了基本的输入输出功能。现在 `print` 和 `println` 支持多种类型重载，因此你可以直接打印整数、浮点数、布尔值和字符：

```ayanami
import "io";

fn main() -> int {
    println("hello")     // 字符串
    println(42)          // 整数
    println(3.5)         // 浮点
    println(true)        // 布尔
    println('x')         // 字符
    return 0
}
```

你仍然可以使用 `putchar` 和 `getchar` 进行字符级别的读写：

```ayanami
c = getchar();
putchar(65);   // 输出 'A'
```

## 数学函数（math）

`math` 模块提供常用的数学运算函数，分为整数和浮点版本。

对于整数类型，支持 `abs`、`min`、`max`、`clamp` 和 `pow`：

```ayanami
import "math";

fn main() -> int {
    println(min(3, 7))     // 3
    println(max(3, 7))     // 7
    println(clamp(5, 1, 10)) // 5
    println(pow(2, 3))     // 8
    return 0
}
```

对于浮点类型，除了上述函数外还支持 `sqrt`、`floor` 和 `ceil`：

```ayanami
import "math";

fn main() -> int {
    println(sqrt(9.0))   // 3.0
    println(floor(3.7))  // 3.0
    println(ceil(3.2))   // 4.0
    return 0
}
```

## 字符串处理（string）

`string` 模块提供了丰富的字符串操作方法。我们来逐一介绍一些常用功能。

### 基本操作

```ayanami
import "string";

fn main() -> int {
    s = "hello"
    println(s.len())           // 5
    println(s.index(0))        // 'h'
    t = s + " world"
    println(t)                 // "hello world"
    return 0
}
```

### 字符串分析

```ayanami
import "string";

fn main() -> int {
    s = "  Ayanami  "
    clean = s.trim()
    println(clean.to_upper())     // AYANAMI
    println("len=" + clean.len()) // len=7

    csv = "a,b,c"
    println(csv.contains(","))       // true
    println(csv.index_of("b"))       // 2

    n = "123".parse_int()
    println(n + 1)                   // 124
    return 0
}
```

`index_of` 方法在未找到子串时返回 `-1`，这与 C 风格的字符串查找一致。

### 字符工具

每个字符（`char`）也支持分类和转换方法：

```ayanami
import "string";

fn main() -> int {
    c = '7'
    println(c.is_digit())     // true
    println(c.to_upper())     // '7' (不变)
    println(c.to_digit())     // 7

    d = 'a'
    println(d.is_alpha())     // true
    println(d.is_lower())     // true
    println(d.to_upper())     // 'A'
    return 0
}
```

## 标准模块（std）

`std` 模块是所有标准库功能的入口点，它等价于导入 `io`、`string`、`math`、`list`、`linkedlist` 和 `arraylist`，并定义了错误处理相关的接口和类型。

### 错误处理接口

`Error` 接口用于表示错误：

```ayanami
interface Error {
    fn what(ref self) -> String;
}
```

### Result 类型

`Result[T, E]` 是一种封装可能失败操作结果的类型。它有两个变体：成功（`Ok(T)`）或失败（`Err(E)`）。你可以使用 `try_unwrap` 来提取值。

### Option 类型

`Option[T]` 表示一个可能存在也可能不存在的值：

```ayanami
enum Option[T] {
    Some(T),
    None,
}
```

你可以通过 `unwrap_or` 提供默认值，或用 `is_some` 判断是否为 `Some`：

```ayanami
import "std";

fn or_zero(Option[int] o) -> int {
    return o.unwrap_or(0)
}

fn main() -> int {
    println(or_zero(Option::Some(42)))  // 42
    println(or_zero(Option::None))      // 0
    return 0
}
```

## 集合（list/arraylist/linkedlist）

标准库提供了三种集合实现：`List[T]` 接口、`ArrayList[T]` 和 `LinkedList[T]`。

### ArrayList

`ArrayList[T]` 是基于数组的顺序表，支持动态扩容。它新增了以下方法：

```ayanami
import "arraylist";

fn main() -> int {
    a = ArrayList[int] { data = null, len = 0, capability = 0 }
    a.push(3)
    a.push(1)
    a.set(1, 5)              // 设置索引为1的元素为5
    println(a.to_string())   // [3, 5]
    println(a.pop())         // 5
    return a.len()           // 1
}
```

此外，还支持 `is_empty` 和 `clear` 方法：

```ayanami
fn main() -> int {
    a = ArrayList[int] { data = null, len = 0, capability = 0 }
    println(a.is_empty())   // true
    a.push(1)
    a.clear()
    println(a.is_empty())   // true
    return 0
}
```

元素类型不限于 `int`：只要实现了 `ToString`，`ArrayList[String]` 也可以正常使用。

### LinkedList

`LinkedList[T]` 提供了链式结构的集合实现。你可以使用 `LinkedList::new[T]()` 创建新实例：

```ayanami
import "linkedlist";

fn main() -> int {
    l = LinkedList::new[int]()
    l.push(1)
    l.push(2)
    return 0
}
```

## 总结

本章介绍了 Ayanami 标准库的最新功能，包括增强的输入输出、数学函数、字符串处理、错误处理机制以及集合类型。所有 API 的详细列表请参见附录 A。

在下一章中，我们将深入探讨 Ayanami 中的错误处理机制，并介绍如何使用 `Result` 和 `Option` 来编写更健壮的程序。
