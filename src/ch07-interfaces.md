# 第 7 章 接口与动态派发

在 Ayanami 中，接口（interface）是一种定义行为契约的方式。它允许我们编写通用的函数，这些函数可以接受实现了特定接口的任意类型作为参数。这种机制支持动态派发，即在运行时决定调用哪个具体实现。

## 定义接口

接口通过 `interface` 关键字声明，其内容是一组方法签名。例如：

```ayanami
interface Drawable {
    fn draw(ref self) -> void;
    fn get_id(ref self) -> int;
}
```

这个接口定义了两个方法：`draw` 和 `get_id`。任何类型只要实现了这两个方法，就可以被视为实现了该接口。

## 实现接口

我们可以为任意类型实现接口。例如：

```ayanami
impl int {
    fn draw(self) -> void {}
    fn get_id(self) -> int { return self; }
}

impl float {
    fn draw(self) -> void {}
    fn get_id(self) -> int { return 0; }
}
```

这里我们为 `int` 和 `float` 类型分别实现了 `Drawable` 接口。每个方法都必须有正确的签名，否则编译会报错。

## 结构匹配

Ayanami 的接口采用 **Go 式结构匹配**：只要某个类型拥有与接口一致的方法签名（方法名、形参、返回类型齐全），它就被自动视为实现了该接口。无需（也不支持）`impl Point: Drawable {}` 这类显式声明，且 `self` 关键字不参与匹配。例如：

```ayanami
struct Point {
    int x;
    int y;
}

impl Point {
    fn draw(ref self) -> void {
        println("Drawing a point");
    }
    
    fn get_id(ref self) -> int {
        return self.x + self.y;
    }
}
```

在这个例子中，`Point` 类型没有也不需要写 `impl Point: Drawable {}`，因为它已经实现了 `draw` 和 `get_id` 方法，所以自动满足 `Drawable` 接口的要求——这就是结构匹配。

## 动态派发

当函数参数类型是接口时，调用者会使用胖指针（fat pointer）来传递值。胖指针包含指向数据的指针以及指向虚表（vtable）的指针，从而允许在运行时选择正确的实现。

```ayanami
fn render_drawable(ref Drawable d) -> void {
    d.draw();
}

fn get_any_id(ref Drawable d) -> int {
    return d.get_id();
}
```

这两个函数接受的是 `ref Drawable` 类型的参数。这意味着它们可以接收任何实现了 `Drawable` 接口的类型。在调用时，系统会根据实际传入的对象类型动态选择对应的 `draw` 或 `get_id` 实现。

例如，在主函数中：

```ayanami
fn main() -> int {
    a = 42;
    b = 3.0;
    render_drawable(a);
    render_drawable(b);
    return get_any_id(a) + get_any_id(b) - 42;
}
```

变量 `a` 是一个整数，而 `b` 是浮点数。尽管它们类型不同，但都实现了 `Drawable` 接口，因此可以传入 `render_drawable` 函数中。

## 使用标准库接口

Ayanami 提供了一些常用的接口，比如 `ToString`。只要一个类型实现了 `to_string` 方法，
它就能参与字符串拼接：

```ayanami
import "string";

fn main() -> int {
    s = "value: " + 42
    println(s)          // 输出 "value: 42"
    println("pi = " + 3.14)
    return 0;
}
```

标准库已经为 `int`、`float`、`char`、`bool` 和 `String` 实现了 `ToString`，
所以它们都可以直接和字符串相加。

## 接口与泛型的区别

虽然接口和泛型都可以实现代码复用，但它们的工作方式不同：

- **接口**：在运行时进行派发，适用于需要动态行为的场景。
- **泛型**：编译期单态化，每个具体类型都会生成一份独立的代码副本。

例如，如果我们使用泛型来处理类似的问题：

```ayanami
fn print_generic[T: ToString](ref T s) -> void {
    println(s.to_string());
}
```

这将在编译时为每种类型生成不同的版本。而接口则在运行时决定调用哪个方法。

## 总结

本章介绍了 Ayanami 中的接口机制，包括如何定义、实现以及使用接口进行动态派发。通过接口，我们可以编写更加灵活和通用的代码，同时保持良好的性能和类型安全。下一章我们将探讨泛型与单态化的概念，并比较它们与接口的不同之处。
