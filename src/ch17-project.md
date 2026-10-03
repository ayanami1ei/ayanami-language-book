# 第 17 章 综合项目：表达式计算器

在本章中，我们将综合运用前面所学的所有知识，构建一个简单的四则运算表达式计算器。这个程序将从标准输入读取一行表达式（如 `12+3*4`），并按照数学中的运算优先级计算出结果。

我们将会使用递归下降解析的方法来实现这一功能，并结合结构体、函数、循环以及字符处理等语言特性。通过本章的学习，你将对 Ayanami 语言的综合应用能力有一个全面的认识。

## 完整程序

```ayanami
import "io"
import "string"

struct Parser {
    int cur
}

fn next_char() -> int {
    return getchar()
}

// 跳过空格
fn skip_spaces(ref mut Parser p) {
    while p.cur == 32 {
        p.cur = next_char()
    }
}

// 解析一个整数；解析完后 p.cur 停在数字后面的字符上
fn parse_number(ref mut Parser p) -> int {
    n = 0
    while p.cur >= 48 && p.cur <= 57 {
        n = n * 10 + p.cur - 48
        p.cur = next_char()
    }
    return n
}

// factor: 数字
fn parse_factor(ref mut Parser p) -> int {
    skip_spaces(p)
    return parse_number(p)
}

// term: factor (('*' | '/') factor)*
fn parse_term(ref mut Parser p) -> int {
    v = parse_factor(p)
    while p.cur == 42 || p.cur == 47 {
        op = p.cur
        p.cur = next_char()
        r = parse_factor(p)
        if op == 42 {
            v = v * r
        } else {
            v = v / r
        }
    }
    return v
}

// expr: term (('+' | '-') term)*
fn parse_expr(ref mut Parser p) -> int {
    v = parse_term(p)
    while p.cur == 43 || p.cur == 45 {
        op = p.cur
        p.cur = next_char()
        r = parse_term(p)
        if op == 43 {
            v = v + r
        } else {
            v = v - r
        }
    }
    return v
}

fn main() -> int {
    println("Enter an expression, e.g. 12+3*4:")
    p = Parser { cur = next_char() }
    result = parse_expr(p)
    println("= " + result)
    return 0
}
```

## 模块讲解

### 结构体与状态管理

我们首先定义了一个 `Parser` 结构体，其中只包含一个字段 `cur`，用于保存当前读取到的字符 ASCII 码。这代表了我们的解析器的状态。

```ayanami
struct Parser {
    int cur
}
```

在主函数中初始化这个结构体：

```ayanami
p = Parser { cur = next_char() }
```

这里的 `next_char()` 是一个简单的包装函数，它调用标准库提供的 `getchar()` 函数获取下一个字符的 ASCII 码。

### 字符处理与空格跳过

为了正确解析表达式，我们需要忽略输入中的空白字符。为此我们实现了 `skip_spaces` 函数：

```ayanami
fn skip_spaces(ref mut Parser p) {
    while p.cur == 32 {
        p.cur = next_char()
    }
}
```

该函数会持续读取字符直到遇到非空格字符为止。

### 数字解析

接下来是 `parse_number` 函数，它负责从当前字符开始解析一个整数：

```ayanami
fn parse_number(ref mut Parser p) -> int {
    n = 0
    while p.cur >= 48 && p.cur <= 57 {
        n = n * 10 + p.cur - 48
        p.cur = next_char()
    }
    return n
}
```

此函数使用了字符的 ASCII 值进行判断（数字 `'0'` 到 `'9'` 对应 ASCII 码 48 到 57）。每次读取一个数字后，将其加入到当前数值中，并继续读取下一个字符。

### 因子解析

在语法分析中，“因子”通常指的是最基础的表达单元。在这个计算器中，我们把“因子”定义为一个整数：

```ayanami
fn parse_factor(ref mut Parser p) -> int {
    skip_spaces(p)
    return parse_number(p)
}
```

这里先跳过空格再解析数字。

### 项解析

“项”由多个因子组成，并且支持乘法和除法运算。我们用 `parse_term` 来处理这一层：

```ayanami
fn parse_term(ref mut Parser p) -> int {
    v = parse_factor(p)
    while p.cur == 42 || p.cur == 47 {
        op = p.cur
        p.cur = next_char()
        r = parse_factor(p)
        if op == 42 {
            v = v * r
        } else {
            v = v / r
        }
    }
    return v
}
```

该函数首先解析第一个因子，然后循环处理后续的乘除操作。由于乘法和除法具有相同的优先级，我们从左到右依次计算。

### 表达式解析

最后是表达式的解析，它由多个项组成，并支持加法和减法运算：

```ayanami
fn parse_expr(ref mut Parser p) -> int {
    v = parse_term(p)
    while p.cur == 43 || p.cur == 45 {
        op = p.cur
        p.cur = next_char()
        r = parse_term(p)
        if op == 43 {
            v = v + r
        } else {
            v = v - r
        }
    }
    return v
}
```

这里同样是从左到右处理加减运算，因为它们的优先级低于乘除。

### 主函数

主函数负责启动整个解析流程：

```ayanami
fn main() -> int {
    println("Enter an expression, e.g. 12+3*4:")
    p = Parser { cur = next_char() }
    result = parse_expr(p)
    println("= " + result)
    return 0
}
```

程序首先提示用户输入表达式，然后初始化解析器并调用 `parse_expr` 开始解析。最终输出计算结果。

## 运行示例

将上述代码保存为 `calc.aya` 文件后，可以通过以下命令运行：

```bash
ayanami run calc.aya
```

程序运行时会提示输入表达式：

```
Enter an expression, e.g. 12+3*4:
12+3*4
= 24
```

这说明程序正确地按照运算优先级计算出 `12 + (3 * 4) = 24`。

## 扩展练习

虽然当前版本的计算器只支持整数和基本四则运算，但你可以尝试扩展其功能：

1. **支持括号**：引入栈或递归机制来处理括号表达式。
2. **支持小数**：修改数字解析逻辑以支持浮点数输入。
3. **添加模运算符 `%`**：在 `parse_term` 中增加对 `%` 的处理。

这些扩展可以让你更深入理解语法分析和语言设计的细节。

## 全书小结

至此，我们已经完成了《Ayanami 语言教程》的学习。从基础的数据类型、变量赋值，到复杂的函数定义与调用、结构体与枚举、泛型与接口等高级特性，再到所有权系统和错误处理机制，你已经掌握了 Ayanami 语言的核心语法和编程思想。

通过本章的综合项目，我们不仅复习了所有已学内容，还实践了如何将这些知识应用于实际问题中。希望你能继续探索 Ayanami 的更多可能性，并在未来的开发工作中灵活运用所学技能。
