# 第 2 章 猜数字游戏

在本章中，我们将一起编写一个简单的猜数字游戏。程序预先设定一个秘密数字（42），玩家通过输入猜测的数字来尝试猜中它。如果猜对了，程序会输出“Correct!”；如果猜小了或猜大了，程序会提示继续尝试。

这个程序将帮助你掌握 Ayanami 语言中的以下概念：

- 如何导入标准库；
- 函数定义与返回值；
- 循环结构 `while`；
- 条件分支 `if / elif / else`；
- 变量与类型推断；
- 字符串打印。

下面是完整的程序代码，你可以将其保存为 `guess.aya` 文件：

```ayanami
import "io"

fn read_int() -> int {
    n = 0
    c = getchar()
    while c >= 0 && c != 10 {
        if c >= 48 && c <= 57 {
            n = n * 10 + c - 48
        }
        c = getchar()
    }
    return n
}

fn main() -> int {
    secret = 42
    println("Guess the number (0-99):")
    while true {
        guess = read_int()
        if guess == secret {
            println("Correct!")
            return 0
        } elif guess < secret {
            println("Too small, try again")
        } else {
            println("Too big, try again")
        }
    }
    return 0
}
```

现在我们逐段解释这个程序。

首先，程序开头导入了标准库 `io`。这使得我们可以使用如 `println` 和 `getchar` 等函数：

```ayanami
import "io"
```

接下来定义了一个函数 `read_int()`，用于从输入中读取一个整数。这个函数会逐个读取字符，直到遇到换行符或文件结束符为止，并将这些字符转换为整数返回。

```ayanami
fn read_int() -> int {
    n = 0
    c = getchar()
    while c >= 0 && c != 10 {
        if c >= 48 && c <= 57 {
            n = n * 10 + c - 48
        }
        c = getchar()
    }
    return n
}
```

函数内部使用了变量 `n` 来累积结果，初始值为 0。变量 `c` 存储每次读取的字符（ASCII 码）。循环条件 `while c >= 0 && c != 10` 表示只要字符不是文件结束符（负数）且不是换行符（ASCII 码为 10），就继续读取。

在循环中，我们判断当前字符是否是数字（ASCII 码在 48 到 57 之间）。如果是，则将其转换为数字并累加到 `n` 中。计算方式是 `n * 10 + c - 48`，其中 `-48` 是为了将 ASCII 码转为对应的数字值。

主函数 `main` 定义了秘密数字 `secret = 42`，并提示用户输入猜测的数字：

```ayanami
fn main() -> int {
    secret = 42
    println("Guess the number (0-99):")
```

接着是一个无限循环 `while true`，不断读取用户的输入并进行判断。每次循环中调用 `read_int()` 获取用户输入的数字：

```ayanami
    while true {
        guess = read_int()
```

然后使用 `if / elif / else` 判断猜测结果：

```ayanami
        if guess == secret {
            println("Correct!")
            return 0
        } elif guess < secret {
            println("Too small, try again")
        } else {
            println("Too big, try again")
        }
    }
```

如果猜对了，程序输出“Correct!”并返回 0；如果猜小了，提示“Too small, try again”；否则提示“Too big, try again”。

最后，我们使用 `return 0` 确保程序正常退出。

要运行这个程序，请在终端中执行以下命令：

```bash
ayanami run guess.aya
```

程序运行后会提示你输入数字。例如，你可以尝试输入 `42` 来猜对，或者输入其他数字来测试程序的反馈。以下是运行示例：

```
Guess the number (0-99):
40
Too small, try again
50
Too big, try again
42
Correct!
```

## 小结

本章我们通过一个简单的猜数字游戏，学习了以下内容：

- 如何使用 `import` 导入标准库；
- 函数的定义与返回值类型；
- 使用 `while` 循环实现重复输入；
- 使用 `if / elif / else` 进行条件判断；
- 变量的类型推断机制；
- 使用 `println` 打印字符串；
- 利用 `getchar()` 读取字符并转换为整数。

这些是程序设计的基础概念，通过本章的学习，你已经可以编写一些简单的交互式程序了。在下一章中，我们将系统地学习变量、类型、运算符与控制流。
