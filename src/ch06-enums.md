# 第 6 章 枚举与模式匹配

在 Ayanami 中，枚举是一种强大的数据结构，它允许你定义一个类型，该类型可以是多个不同变体中的任意一种。每个变体可以携带数据，也可以不携带。这种结构非常适合表示具有多种可能状态的值。

我们先来看一个常见的枚举定义：

```ayanami
enum Option[T] {
    Some(T),
    None,
}
```

这个 `Option` 枚举有两个变体：`Some(T)` 和 `None`。其中，`Some` 带有一个泛型参数 `T`，表示它可以包含任意类型的值；而 `None` 则没有携带任何数据。

你可以这样构造一个 `Option[int]` 类型的值：

```ayanami
x = Option::Some(42)
y = Option::None
```

这里，`x` 是一个带有整数 `42` 的 `Some` 变体，而 `y` 是空的 `None` 变体。

在内存中，枚举的布局由两部分组成：一个是 `_tag` 字段，用于标识当前是哪一个变体；另一个是 `_data_` 字段，存储实际的数据。例如：

```ayanami
x = Option::Some(42)
tag = x._tag        // tag 为 0（因为 Some 是第一个变体）
v = x._data_Some._0 // v 为 42
```

通过 `_tag` 字段，你可以判断当前枚举的类型；而 `_data_` 字段则保存了具体的值。如果变体是 `None`，那么它不会有任何数据字段。

模式匹配（`match`）是处理枚举的核心机制。它允许你根据不同的变体执行不同的代码逻辑。`match` 是一个表达式，这意味着它可以有返回值，并且必须覆盖所有可能的变体：

```ayanami
fn describe(Option[int] x) -> int {
    match x {
        Some(v) => return v,
        None => return 0,
    }
}
```

在这个例子中，如果传入的是 `Some(v)`，就返回这个值；如果是 `None`，则返回 `0`。注意，在 `match` 中必须对所有可能的变体进行处理，否则编译器会报错。

此外，枚举还可以为特定变体定义方法。比如我们可以给 `Option_Some` 类型添加一个 `get()` 方法：

```ayanami
impl Option_Some[T] {
    fn get(ref self) -> T { return self._0 }
}
```

然后就可以像这样调用：

```ayanami
e = Option::Some(10)
v = e.get() // v 为 10
```

系统会自动根据当前枚举的 `_tag` 来决定调用哪个方法。这就是所谓的“按标签自动派发”。

为了进一步说明枚举的使用方式，我们来看一个完整的示例程序：

```ayanami
enum Option[T] {
    Some(T),
    None,
}

impl Option_Some[T] {
    fn get(ref self) -> T { return self._0 }
}

fn describe(Option[int] x) -> int {
    match x {
        Some(v) => return v,
        None => return 0,
    }
}

fn main() -> int {
    x = Option::Some(42)
    tag = x._tag
    v = x._data_Some._0
    y = describe(Option::None)
    return tag + v + y - 42
}
```

在这个程序中，我们首先定义了一个 `Option` 枚举，并为其 `Some` 变体实现了 `get()` 方法。接着，在 `describe` 函数里使用了 `match` 来处理不同的情况。最后在 `main` 函数中构造了两个值：一个是 `Some(42)`，另一个是 `None`。程序通过访问 `_tag` 和 `_data_` 字段来获取内部信息，并将这些值参与计算后返回。

除了 `Option` 之外，Ayanami 标准库还提供了另一个常用的枚举类型：`Result[T, E]`。它通常用于表示操作可能成功或失败的情况。虽然我们会在错误处理章节详细介绍其用法，但你可以简单理解为：

```ayanami
enum Result[T, E] {
    Ok(T),
    Err(E),
}
```

这代表一个结果要么是成功的值 `T`，要么是一个错误信息 `E`。

通过本章的学习，你应该已经掌握了如何定义和使用枚举、如何利用模式匹配来处理不同变体，并了解了如何为特定变体编写方法。下一章我们将介绍接口与动态派发机制，进一步拓展你的程序设计能力。
