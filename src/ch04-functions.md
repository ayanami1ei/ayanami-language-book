# 第 4 章 函数与命名空间

在 Ayanami 语言中，函数是程序的基本构建单元。你可以将函数看作是一段可重复使用的代码块，它接收输入参数并可能返回一个结果。定义函数时，必须明确指定参数类型和返回值类型。

## 定义函数

我们从最简单的加法函数开始：

```ayanami
fn add(int a, int b) -> int {
    return a + b;
}
```

这个函数名为 `add`，接受两个整数参数 `a` 和 `b`，并返回它们的和。注意这里的参数顺序是「类型 名称」，这与 C 系语言不同，例如在 C 中你写的是 `int add(int a, int b)`，而在 Ayanami 中则是 `fn add(int a, int b) -> int`。

函数体使用大括号 `{}` 包裹。语句末尾的分号可以省略，但必须显式使用 `return` 来返回值。如果函数没有返回值，则不需要写 `->` 和 `return` 语句。

## 返回值与提前退出

函数的返回类型通过 `->` 显式声明。例如：

```ayanami
fn square(int x) -> int {
    return x * x;
}
```

如果你需要在函数中提前退出并返回某个值，可以使用 `return` 语句。比如下面这个函数会把负数取反，从而返回绝对值：

```ayanami
fn abs(int x) -> int {
    if x < 0 {
        return -x;
    }
    return x;
}
```

`main` 函数比较特殊：`fn main() -> int` 末尾可以省略 `return`，此时缺省返回 0（C 语义，显式 `return` 优先）。其他非 void 函数必须显式返回；void 函数里可以用 `return;` 提前结束。

## 无返回值函数

如果一个函数不返回任何值，你可以省略 `->` 和 `return`。例如：

```ayanami
fn greet() {
    println("Hello, world!");
}
```

这种函数被称为“过程”，它们执行某些操作但不产生结果。

## 函数重载

Ayanami 支持函数重载机制，即允许存在多个同名函数，只要它们的参数类型不同即可。例如：

```ayanami
fn add(int a, int b) -> int {
    return a + b;
}

fn add(float a, float b) -> float {
    return a + b;
}
```

这两个 `add` 函数分别处理整数和浮点数加法，编译器会根据调用时传入的参数类型自动选择正确的版本。

## 函数指针与高阶函数

函数本身也可以被赋值给变量或作为参数传递。这种变量被称为函数指针。例如：

```ayanami
fn apply(int x, int y, fn(int,int)->int f) -> int {
    return f(x, y);
}
```

这里 `f` 是一个函数指针，它接受两个整数并返回一个整数。你可以这样调用它：

```ayanami
f = add;
a = apply(3, 4, f);
```

这表示我们将 `add` 函数赋值给变量 `f`，然后将其传入 `apply` 函数中执行。

## 命名空间

为了更好地组织代码，Ayanami 提供了命名空间（namespace）机制。你可以将相关的函数封装在命名空间内，避免名字冲突。例如：

```ayanami
namespace math {
    fn add(int a, int b) -> int {
        return a + b;
    }
}
```

之后可以通过 `math::add(10, 20)` 的方式调用该函数。

标准库中的模块也属于命名空间的一种形式，比如 `import "io"` 就引入了一个名为 `io` 的命名空间，其中包含如 `println` 等函数。

## 匿名函数

Ayanami 还支持匿名函数（lambda 表达式），其语法为 `(参数列表) { 函数体 }`。你可以将匿名函数作为参数传递给其他函数：

```ayanami
import "io"
import "string"
import "arraylist"

fn main() -> int {
    list = ArrayList::new[int]();
    list.push(1);
    list.push(2);
    list.iter((int x) { println(x.to_string()); });
    return 0;
}
```

在这个例子中，`(int x) { println(x); }` 是一个匿名函数，它被传入 `list.iter` 方法中，用于遍历列表中的每个元素并打印出来。

## 综合示例

让我们看一个完整的示例程序，结合了以上所有概念：

```ayanami
fn add(int a, int b) -> int { return a + b; }

fn apply(int x, int y, fn(int,int)->int f) -> int {
    return f(x, y);
}

namespace math {
    fn add(int a, int b) -> int {
        return a + b;
    }
}

fn main() -> int {
    f = add
    a = apply(3, 4, f)
    b = math::add(10, 20)
    return a + b - 37
}
```

在这个程序中，我们定义了一个基本的 `add` 函数，并通过 `apply` 将其作为函数指针传递。同时，在 `math` 命名空间中也定义了一个同名函数，用于演示命名空间的使用。最后在主函数中分别调用了这两个函数，并计算出最终结果。

## 预告

下一章我们将深入探讨结构体与方法，学习如何定义和使用自定义数据类型，以及如何为这些类型添加行为（即方法）。
