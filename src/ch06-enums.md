# 第 6 章 枚举与模式匹配

在 Ayanami 中，枚举允许你定义一个类型，它的值可以是若干**变体**（variant）中的任意一种。每个变体可以携带数据，也可以不携带。这种结构非常适合表示具有多种可能状态的值。

我们先定义一个描述形状的枚举：

```ayanami
enum Shape {
    Circle(int),
    Square(int),
    Point,
}
```

`Shape` 有三个变体：`Circle(int)` 与 `Square(int)` 各携带一个整数，`Point` 不携带数据。

## 构造与内存布局

构造变体使用 `枚举名::变体名(...)`：

```ayanami
a = Shape::Circle(3)
b = Shape::Square(4)
c = Shape::Point
```

在内存中，枚举布局为 `{ _tag, _data_变体名, ... }`：`_tag` 是判别值（按声明顺序从 0 开始），`_data_变体名` 存放该变体的载荷：

```ayanami
a = Shape::Circle(3)
tag = a._tag          // 0（Circle 是第一个变体）
r = a._data_Circle._0 // 3
```

## match 表达式

`match` 是**表达式**：每个分支的值就是整个 `match` 的值，可以直接 `return`，也可以赋给变量。分支按 `_tag` 匹配，必须覆盖所有变体：

```ayanami
fn describe(Shape s) -> int {
    return match s {
        Circle(r) => r * r,
        Square(a) => a * a,
        Point => 0,
    }
}
```

如果某个变体没有处理，编译器会报错，这保证了不会遗漏状态。用下划线可以忽略不需要的载荷：

```ayanami
fn is_circle(Shape s) -> bool {
    return match s {
        Circle(_) => true,
        Square(_) => false,
        Point => false,
    }
}
```

## 在枚举上定义方法

可以给整个枚举类型写方法，在方法内部用 `match` 分派：

```ayanami
impl Shape {
    fn corners(ref self) -> int {
        return match self {
            Circle(_) => 0,
            Square(_) => 4,
            Point => 1,
        }
    }
}
```

## 更多模式

`match` 的模式不止变体名，还支持字面量、区间、`|`、guard、结构体解构与嵌套：

```ayanami
match n {
    0 => println("zero"),
    1..=9 => println("digit"),
    10 | 20 => println("ten or twenty"),
    v if v < 0 => println("negative"),
    _ => println("other"),
}
```

- 区间：`1..10`（半开）、`10..=20`（闭），限整数与 `char`；
- `|` 组合多个模式；`if` guard 在匹配后追加条件；
- `_` 通配，`v` 绑定整个值（顶层仅 Copy 类型）。

结构体可以解构：

```ayanami
struct Point {
    int x
    int y
}

match p {
    Point { x, y } => println(x + y),          // 简写绑定同名字段
    Point { x = a, y = b } => println(a - b),  // 重命名
}
```

枚举载荷可以嵌套与解构：

```ayanami
match opt {
    Some(Some(v)) => println(v),
    Some(Point { x, y }) => println(x + y),
    Some(None) => println("inner none"),
    None => println("none"),
}
```

臂体可以写成块（尾表达式为值，块内可用 `return` / `break` / `continue`）：

```ayanami
return match x {
    0 => 0,
    n if n > 0 => { n * 2 },
    _ => { 0 - 1 },
}
```

穷尽检查会合并区间与字面量覆盖：例如 `u8` 上写 `0..=255` 或列全所有字面量即视为穷尽。

## 标准库中的泛型枚举

标准库提供了两个常用的泛型枚举（定义在 `std` 模块）：

```ayanami
enum Option[T] {
    Some(T),
    None,
}

enum Result[T, E] {
    Ok(T),
    Err(E),
}
```

`Option[T]` 表示“可能有值”，用 `Option::Some(v)` / `Option::None()` 构造；`Result[T, E]` 表示“成功或失败”，用 `Result::Ok(v)` / `Result::Err(e)` 构造（详细用法见第 12 章）。

`Option` 提供了 `unwrap_or(default)` 与 `is_some()` 方法：

```ayanami
import "std"

fn find(int x) -> Option[int] {
    if x > 0 { return Option::Some(x) }
    return Option::None()
}

fn main() -> int {
    println(find(5).unwrap_or(0))    // 5
    println(find(-1).unwrap_or(42))  // 42
    return 0
}
```

`Option` 也支持 `match`：

```ayanami
fn describe(Option[int] o) -> int {
    return match o {
        Some(v) => v,
        None => 0,
    }
}
```

## 小结

- 用 `enum` 定义变体；`枚举名::变体名(...)` 构造；
- `_tag` 是判别值，`_data_变体名` 是载荷；
- `match` 是表达式，分支必须覆盖所有变体；
- 标准库的 `Option` / `Result` 是泛型枚举，`match` 与普通枚举一样可用。

下一章我们将介绍接口与动态派发机制。
