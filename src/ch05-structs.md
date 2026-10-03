# 第 5 章 结构体与方法

在 Ayanami 中，结构体是一种复合数据类型，用于将多个值组合成一个整体。你可以把结构体看作是自定义的数据模板，它允许你组织和管理相关的数据项。

## 定义结构体

定义一个结构体使用 `struct` 关键字，后跟结构体名称以及字段列表。每个字段由类型和名称组成，字段之间不需要分号分隔。例如：

```ayanami
struct Point {
    int x
    int y
}
```

这段代码定义了一个名为 `Point` 的结构体，它包含两个整型字段：`x` 和 `y`。

## 结构体字面量与字段访问

你可以通过结构体字面量来创建一个结构体实例。语法是使用结构体名称，并在大括号中指定每个字段的值。字段赋值时使用 `字段名 = 值` 的形式：

```ayanami
p = Point { x = 1, y = 2 };
```

这里我们创建了一个 `Point` 类型的变量 `p`，其 `x` 字段为 1，`y` 字段为 2。

访问结构体字段时，使用点号（`.`）操作符。比如：

```ayanami
import "io"
import "string"

println(p.x.to_string()); // 输出 1
```

## 方法定义与接收者

结构体的方法是通过 `impl` 块来定义的。每个方法都必须有一个接收者，它决定了该方法如何处理调用它的值。接收者可以是以下三种形式之一：

- `self`：消费调用者的值（移动语义），适用于非 Copy 类型。
- `ref self`：借用调用者的值（不可变引用），适用于需要读取但不修改的情况。
- `ref mut self`：可变借用调用者的值，适用于需要修改的情况。

例如：

```ayanami
impl Point {
    fn get_x(ref self) -> int {
        return self.x;
    }
}
```

这个方法定义了一个名为 `get_x` 的函数，它接受一个不可变引用的 `Point` 类型作为接收者，并返回该点的 `x` 坐标。

## 结构体参数与返回值

结构体可以作为函数参数传递，也可以作为返回值。当结构体作为参数传入时，根据接收者的类型不同，会发生不同的行为：

- 如果使用 `self` 接收者，则会移动该结构体；
- 如果使用 `ref self` 或 `ref mut self`，则只是借用。

下面是一个简单的例子，展示了如何将结构体作为参数传递并返回一个新结构体：

```ayanami
fn add_points(Point a, Point b) -> Point {
    return Point { x = a.x + b.x, y = a.y + b.y };
}
```

在这个函数中，`a` 和 `b` 都是通过值传入的结构体，因此它们会被移动。函数返回一个新的 `Point` 实例。

## 方法调用

一旦定义了方法，就可以像调用普通函数一样调用它。方法调用语法为：

```ayanami
p.get_x()
```

这会调用 `Point` 类型上的 `get_x` 方法，并返回其结果。注意，由于我们使用的是 `ref self` 接收者，所以原结构体 `p` 不会被消耗。

## 结构体与生命周期

结构体中的字段可以包含引用类型。例如：

```ayanami
struct Holder {
    #[follow_with(owner)]
    ref String item
}
```

这里的 `item` 是一个指向 `String` 的引用，`#[follow_with(owner)]` 标注表明该引用至少要活到 `Holder` 实例失效为止（`owner` 指包含它的实例本身）。详细规则见第 14 章。

## 原语类型上的方法

除了自定义结构体外，你还可以为原语类型（如 `int`, `float`, `char`, `bool`）实现方法。例如：

```ayanami
impl int {
    fn square(self) -> int {
        return self * self;
    }
}
```

这样就可以对整数进行平方运算：

```ayanami
n = 5
x = n.square(); // x 等于 25
```

## 示例程序

让我们看一个完整的示例，结合了上述所有概念：

```ayanami
struct Point {
    int x
    int y
}

impl Point {
    fn get_x(ref self) -> int {
        return self.x;
    }
}

struct Counter {
    int n
}

impl Counter {
    fn get(ref self) -> int {
        return self.n
    }
    #[state]
    fn inc(ref mut self) {
        self.n = self.n + 1
    }
    fn into(self) -> int {
        return self.n
    }
}

fn main() -> int {
    p = Point { x = 42, y = 0 };
    c = Counter { n = 41 };
    c.inc();
    d = c.get();
    e = c.into();
    return p.get_x() + d - 41;
}
```

在这个程序中，我们定义了两个结构体：`Point` 和 `Counter`。其中 `Counter` 包含了三种不同类型的接收者方法：`get` 使用 `ref self`、`inc` 使用 `ref mut self`、`into` 使用 `self`。最后在主函数里，我们创建了这些结构体的实例，并调用了它们的方法。

## 预告

在下一章中，我们将学习 Ayanami 中的枚举类型和模式匹配机制。这将帮助你更好地处理具有多种可能状态的数据，并编写更清晰、更具表达力的代码。
