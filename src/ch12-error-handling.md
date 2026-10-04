# 第 12 章 错误处理

在程序运行过程中，错误是不可避免的。Ayanami 提供了一种显式处理错误的方式，通过 `Result` 枚举类型来表达可能失败的操作。这种设计鼓励开发者在编写代码时就考虑错误情况，并明确地处理它们。

## Result 类型

`Result[T, E]` 是 Ayanami 中用于表示操作结果的枚举类型。它有两个变体：

- `Ok(T)`：表示操作成功，其中包含一个值。
- `Err(E)`：表示操作失败，其中包含一个错误信息。

例如，一个函数如果尝试执行某项可能出错的操作，可以返回 `Result[int, int]` 类型，意味着成功时返回一个整数，失败时也返回一个整数作为错误码。

```ayanami
fn inner(int x) -> Result[int, int] {
    if x < 0 { return Result::Err(x) }
    return Result::Ok(x + 1)
}
```

在这个例子中，`inner` 函数接受一个整数 `x`。如果 `x` 小于 0，则返回错误（`Result::Err(x)`），否则返回成功的结果（`Result::Ok(x + 1)`）。

## 错误传播与 `?` 运算符

当调用一个返回 `Result` 的函数时，你可以选择使用 `match` 来显式处理结果，也可以用 `?` 运算符来自动传播错误。使用 `?` 的前提是：**当前函数的返回类型与 `?` 后面表达式的类型一致**（都是同一个 `Result` 类型）。

```ayanami
fn outer(int x) -> Result[int, int] {
    v = inner(x)?
    return Result::Ok(v * 10)
}
```

`inner(x)?` 的语义是：

- 如果 `inner(x)` 是 `Ok(v)`，整个表达式的值就是 `v`，继续执行；
- 如果 `inner(x)` 是 `Err(e)`，则 `outer` 立即 `return Err(e)`，把错误交给上层。

这样错误就会沿着调用链一直向上传播，直到某一层用 `match` 显式处理它。标准库的 `Result` 还提供了一个便捷方法 `try_unwrap()`，它在 `Err` 时返回 `0`（适合错误值无关紧要的场合），与 `?` 的传播语义不同。

## 使用 `match` 处理结果

除了使用 `?` 运算符外，还可以通过 `match` 表达式来显式处理 `Result` 的两个分支：

```ayanami
fn main() -> int {
    a = outer(1)
    b = outer(-5)
    match a {
        Ok(v) => println("ok: " + v),
        Err(e) => println("err: " + e),
    }
    return a._tag + b._tag - 1
}
```

在这个例子中，我们首先调用了 `outer` 函数两次。第一次传入的是正数 1，第二次是 `-5`。然后使用 `match` 来判断每个结果是成功还是失败，并分别输出不同的信息。

## 自定义错误类型

虽然你可以直接使用整数作为错误码，但在实际开发中，通常会定义自己的错误类型以提供更丰富的错误信息。标准库 `std` 里的 `Error` 接口要求实现 `what(ref self) -> String`，返回错误的描述字符串。只要你的类型有同名方法，就自动满足这个接口（结构匹配）：

```ayanami
import "../std/std.aya"

struct CustomError {
    int code
    String message
}

impl CustomError {
    fn what(ref self) -> String {
        return "Custom error: " + self.message;
    }
}
```

这样，你就可以在 `Result[T, CustomError>` 中携带自定义错误，并通过 `what()` 获取详细描述。

## 总结

Ayanami 的错误处理机制基于 `Result` 枚举类型，允许开发者显式地处理可能失败的操作。你可以使用 `match` 处理结果，或用 `?` 运算符把错误传播给上层调用者。此外，还可以通过实现 `Error` 接口来自定义错误类型，使程序更具可读性和维护性。

在下一章中，我们将介绍 Ayanami 的标注系统，包括如何使用各种注解来增强代码的语义和优化性能。
