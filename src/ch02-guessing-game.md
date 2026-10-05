# 第 2 章 猜数字游戏

本章我们用一个小游戏快速过一遍 Ayanami 的常用语法：程序随机选一个 0~99 的数字，玩家反复输入猜测，直到猜中。

完整程序如下：

```ayanami
import "io"
import "rand"

fn main() -> int {
    rng = Rng::new(42)
    secret = rng.next_range(0, 100)
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

保存为 `guess.aya`，运行：

```bash
ayanami run guess.aya
```

运行示例：

```text
Guess the number (0-99):
40
Too small, try again
50
Too big, try again
42
Correct!
```

下面逐段解释。

## 导入模块

```ayanami
import "io"
import "rand"
```

`import "模块名"` 从标准库加载模块。`io` 提供输入输出（`println`、`read_int`），`rand` 提供伪随机数（`Rng`）。

## 生成秘密数字

```ayanami
rng = Rng::new(42)
secret = rng.next_range(0, 100)
```

- `Rng::new(42)` 用种子 42 创建一个伪随机数生成器。相同种子产生相同序列，方便调试与测试；
  需要每次都不同，可以用 `Rng::from_entropy()`。
- `rng.next_range(0, 100)` 返回 `[0, 100)` 区间内的整数，即 0~99。

## 读取输入

```ayanami
guess = read_int()
```

`read_int()` 从标准输入读取一行，去掉首尾空白后按十进制解析成 `int`（失败返回 0）。
标准库还提供：

- `read_line() -> String`：读取一行（不含换行）；
- `try_read_int() -> Option[int]`：严格解析，失败返回 `None`。

## 循环与分支

```ayanami
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
```

- `while true` 是无限循环，靠内部的 `return 0` 退出；
- `if / elif / else` 依次判断相等、偏小、偏大；
- 猜中时 `return 0` 结束 `main`，进程退出码为 0。

`main` 的返回值就是进程退出码，0 表示成功。这里显式写了 `return 0`；其实 `main` 末尾
也可以省略 `return`，缺省返回 0（见第 4 章）。

## 小结

- `import` 加载标准库模块；
- `Rng::new(seed)` / `next_range(lo, hi)` 生成随机数；
- `read_int()` 读取并解析一行输入；
- `while` + `if / elif / else` 控制流程，`return` 提前结束。

下一章系统学习变量、类型、运算符与控制流。
