# 第 9 章 所有权与借用

在 Ayanami 语言中，所有权系统是其核心特性之一。它通过编译时检查来保证内存安全，避免了运行时的垃圾回收机制（GC），也未提供共享指针或弱引用等机制。理解所有权和借用对于编写高效、安全的代码至关重要。

## 默认所有权与移动语义

默认情况下，非 Copy 类型的值在赋值或传参时会发生“移动”（move）。这意味着原始变量不再拥有该值的所有权，而新的变量获得了所有权。例如：

```ayanami
struct Point {
    int x
    int y
}

fn main() -> int {
    p = Point { x = 1, y = 2 };
    q = p; // 此处将 p 的所有权转移给 q
    return p.x; // 错误！p 已经被移动，不能再使用
}
```

如果尝试访问 `p.x`，编译器会报错：use-after-move。这是因为 `Point` 是一个结构体，属于非 Copy 类型。

移动分析对**路径敏感**：`if c { return x }` 之后再使用 `x` 是合法的（该分支不会到达后续代码）；
但如果分支会落空并移动了 `x`（例如 `if c { y = x }`），之后使用 `x` 仍会报 use-after-move。

而像整数、浮点数、布尔值这样的类型是 Copy 类型，在赋值或传参时会进行复制操作，而不是移动。
非捕获的 `Fn` 值（命名函数 / 不捕获的 lambda）也是 Copy：

```ayanami
fn main() -> int {
    a = 42;
    b = a; // 正确：a 被复制给 b，a 仍可使用
    return a + b;
}
```

## 借用（Borrowing）

为了能够在不获取所有权的情况下访问数据，Ayanami 提供了借用机制。使用 `ref T` 表示不可变借用，`ref mut T` 表示可变借用。

```ayanami
struct S {
    int v
}

fn f(ref S s) -> int { return s.v }

fn g(ref mut S s) -> int {
    s.v = s.v + 1
    return s.v
}
```

在上面的例子中，`f` 函数接受一个不可变借用，而 `g` 接受一个可变借用。调用这些函数时，不需要显式地取地址：

```ayanami
fn main() -> int {
    x = S { v = 1 };
    a = f(x); // 自动将 x 借用为 ref S
    b = g(x); // 同样自动借用
    return a + b;
}
```

### 引用的自动解引用

在值上下文（运算、比较、传值形参、返回）中，`ref T` 会自动读出 `T`；对 `ref mut T` 赋值会穿透引用写回被借用的变量；写入不可变的 `ref T` 会报错：

```ayanami
fn bump(ref mut int i) {
    i = i + 1            // 穿透写回
}

fn add_one(int v) -> int { return v + 1 }

fn via_value(ref mut int i) -> int {
    return add_one(i)    // 值上下文自动读出
}

fn main() -> int {
    x = 0
    bump(x)              // x 变为 1
    y = via_value(x)     // y = 2
    return x + y - 3     // 0
}
```

## 接收者与借用

在结构体方法中，接收者的类型决定了是否可以修改数据。`self` 消费所有权，`ref self` 借用不可变地访问数据，`ref mut self` 则允许修改：

```ayanami
impl S {
    fn get(ref self) -> int { return self.v }
    fn set(ref mut self, int val) { self.v = val }
}
```

## NLL（Non-Lexical Lifetimes）

Ayanami 的借用检查器采用非词法生命周期（NLL）策略，即借用在最后一次使用后失效。这使得代码更灵活：

```ayanami
fn main() -> int {
    x = S { v = 1 };
    r = ref x;
    a = r.v;          // r 的最后一次使用
    x.v = 5;          // 合法：r 的借用已失效
    return a + x.v;
}
```

在这个例子中，`a = r.v` 是对 `r` 的最后一次使用，因此 `r` 的生命周期在此处结束。之后再修改 `x.v` 是合法的。

## 引用存储与生命周期省略

引用可以存储在局部变量中，并且当函数返回一个引用时，如果该引用恰好来自某个参数，则编译器会自动推断其生命周期：

```ayanami
fn pick(ref S s) -> ref S {
    return s
}

fn main() -> int {
    x = S { v = 1 };
    p = pick(x); // 自动推断返回值的生命周期
    return p.v;
}
```

## 没有 GC / shared / weak

Ayanami 不提供垃圾回收、共享指针或弱引用。如果需要共享数据，应使用借用；若需要一份独立的副本，`String` 可以调用 `.copy()`，结构体则可以手动构造一个新值：

```ayanami
fn main() -> int {
    x = S { v = 1 };
    y = S { v = x.v }; // 手动复制字段
    s = "hi";
    t = s.copy();      // String 的深拷贝
    return x.v + y.v + t.len() - 2;
}
```

## 示例程序

下面是一个综合示例，展示了上述所有概念的使用方式：

```ayanami
struct S {
    int v
}

fn f(ref S s) -> int { return s.v }

fn g(ref mut S s) -> int {
    s.v = s.v + 1
    return s.v
}

fn pick(ref S s) -> ref S {
    return s
}

fn main() -> int {
    x = S { v = 1 };
    r = ref x;
    a = r.v;          // r 的最后一次使用
    x.v = 5;          // 合法：r 的借用已失效
    b = f(x) + g(x);  // 5 + 6
    p = pick(x);
    c = p.v;
    return b + a + c - 17;
}
```

这段代码中，`x` 被多次使用，但通过借用和生命周期管理确保了安全性。

## 预告

在下一章中，我们将介绍数组与内存管理的相关内容。你将学习如何处理堆分配的数组、动态数组 `ArrayList` 以及它们的内存布局和操作方式。
