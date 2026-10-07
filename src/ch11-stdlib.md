# 第 11 章 标准库

Ayanami 语言内置了丰富的标准库模块，帮助你快速构建程序。这些模块以 `.lcl` 文件形式预编译并存放于编译器目录下的 `std/` 文件夹中。使用 `import "模块名"` 即可导入所需功能。

## 总览

当前标准库包含以下主要模块：

- `io`：输入输出（`print` / `println` / `getchar` / `read_line` / `read_int`）
- `math`：数学函数（整数与浮点）
- `string`：字符串与字符处理
- `convert`：严格解析与类型转换（`try_parse_int`、`to_int`、`Into[T]`）
- `text`：文本处理（`split` / `join` / `replace` / 填充）
- `rand`：伪随机数（`Rng`）
- `time`：时间（`now_millis` / `now_unix`）
- `env`：命令行参数与环境变量
- `fs`：文件读写
- `option`：`Option[T]`（`std` 也会导入）
- `eq` / `sort`：`Eq` 接口与排序（`Ord` 接口）
- `hashset` / `hashmap`：哈希集合与哈希表（`Hash` 接口）
- `iter`：迭代器与 `for x in ...`（`Iterator[T]`、`into_iter` 与适配器）
- `panic`：运行时错误（`#panic` 宏与越界检查，见第 12 章）
- `test`：测试框架（`#[test]` 与 `#assert` 宏，见第 17 章）
- `std`：主入口，整合常用模块，并提供 `Error` / `Result` / `Option`
- `list` / `arraylist` / `linkedlist`：集合

此外还有一个实验性模块 `mir`，用于插件开发，详情请见附录 A。

## 输入输出（io）

`io` 模块提供了基本的输入输出功能。现在 `print` 和 `println` 支持多种类型重载，因此你可以直接打印整数、浮点数、布尔值和字符：

```ayanami
import "io";

fn main() -> int {
    println("hello")     // 字符串
    println(42)          // 整数
    println(3.5)         // 浮点
    println(true)        // 布尔
    println('x')         // 字符
    return 0
}
```

你仍然可以使用 `putchar` 和 `getchar` 进行字符级别的读写：

```ayanami
c = getchar();
putchar(65);   // 输出 'A'
```

### 行输入

```ayanami
line = read_line()          // 读取一行（不含换行；EOF 返回空串）
n = read_int()              // 读一行并宽松解析（trim + parse_int，失败 0）
opt = try_read_int()        // 严格解析，返回 Option[int]
f = read_float()            // 宽松解析浮点（失败 0.0）
b = read_bool()             // 宽松解析布尔（失败 false）
```

`try_read_float()` / `try_read_bool()` 是对应的严格版本，返回 `Option`。

## 数学函数（math）

`math` 模块提供常用的数学运算函数，分为整数和浮点版本。

对于整数类型，支持 `abs`、`min`、`max`、`clamp` 和 `pow`：

```ayanami
import "math";

fn main() -> int {
    println(min(3, 7))     // 3
    println(max(3, 7))     // 7
    println(clamp(5, 1, 10)) // 5
    println(pow(2, 3))     // 8
    return 0
}
```

对于浮点类型，除了上述函数外还支持 `sqrt`、`floor` 和 `ceil`：

```ayanami
import "math";

fn main() -> int {
    println(sqrt(9.0))   // 3.0
    println(floor(3.7))  // 3.0
    println(ceil(3.2))   // 4.0
    return 0
}
```

此外还提供 `gcd` / `lcm` / `is_prime`（整数），`exp` / `ln` / `log10` / `round` / `trunc`（浮点），
以及方法 `n.is_even()` / `n.is_odd()`、`f.is_nan()`。

## 字符串处理（string）

`string` 模块提供了丰富的字符串操作方法。我们来逐一介绍一些常用功能。

### 基本操作

```ayanami
import "string";

fn main() -> int {
    s = "hello"
    println(s.len())           // 5
    println(s.index(0))        // 'h'
    t = s + " world"
    println(t)                 // "hello world"
    return 0
}
```

### 构造字符串

字符串字面量会直接创建 `String`；标准库还提供了构造函数：

```ayanami
s1 = String::empty()          // 空串
s2 = String::new()            // 等价于 empty
buf = ['h', 'i']
s3 = String::new(buf, 2)      // 从拥有所有权的 [char] 缓冲构造
```

### 字符串分析

```ayanami
import "string";

fn main() -> int {
    s = "  Ayanami  "
    clean = s.trim()
    println(clean.to_upper())     // AYANAMI
    println("len=" + clean.len()) // len=7

    csv = "a,b,c"
    println(csv.contains(","))       // true
    println(csv.index_of("b"))       // 2

    n = "123".parse_int()
    println(n + 1)                   // 124
    return 0
}
```

`index_of` 方法在未找到子串时返回 `-1`。`s.index(i)`（或 `s[i]`）越界会 panic（退出码 101）；`parse_int` 是前缀式解析（跳过前导空白与正负号，遇到非数字停止），要求整个串合法请用 `is_int()`。

字符串与集合的长度、下标参数使用 `usize`（无符号 64 位，`usize ≡ u64`）；索引处写任意整数类型都会被统一转换，所以 `a[1u8]`、`s[i]` 都可以直接使用。

`trim_start` / `trim_end` 只去掉一侧空白；`remove_prefix` / `remove_suffix` 有则去掉、无则返回拷贝；
字符串支持字典序比较（`<` `>` `<=` `>=`）；`split_whitespace` / `count` 见 `text` 模块。

### 格式化

```ayanami
255.to_hex()         // "ff"
255.to_hex_upper()   // "FF"
255.to_bin()         // "11111111"
255.to_oct()         // "377"
3.14159.to_fixed(2)  // "3.14"（定点小数）
1234.5.to_sci(2)     // "1.23e+03"（科学计数法）
```

### 字符工具

每个字符（`char`）也支持分类和转换方法：

```ayanami
import "string";

fn main() -> int {
    c = '7'
    println(c.is_digit())     // true
    println(c.to_upper())     // '7' (不变)
    println(c.to_digit())     // 7

    d = 'a'
    println(d.is_alpha())     // true
    println(d.is_lower())     // true
    println(d.to_upper())     // 'A'
    return 0
}
```

## 标准模块（std）

`std` 模块是所有标准库功能的入口点，它等价于导入 `io`、`string`、`math`、`list`、`linkedlist`、`arraylist`、`option` 和 `panic`，并提供 `Error` 接口与 `Result[T, E]`。

### 错误处理接口

`Error` 接口用于表示错误：

```ayanami
interface Error {
    fn what(ref self) -> String;
}
```

### Result 类型

`Result[T, E]` 是一种封装可能失败操作结果的类型。它有两个变体：成功（`Ok(T)`）或失败（`Err(E)`）。用 `?` 运算符传播错误，或用 `match` 处理两个分支；还有 `try_unwrap()` / `is_ok()` / `map` 等辅助方法（见第 12 章）。

### Option 类型

`Option[T]` 表示一个可能存在也可能不存在的值：

```ayanami
enum Option[T] {
    Some(T),
    None,
}
```

你可以通过 `unwrap_or` 提供默认值，或用 `is_some` 判断是否为 `Some`：

```ayanami
import "std";

fn or_zero(Option[int] o) -> int {
    return o.unwrap_or(0)
}

fn main() -> int {
    println(or_zero(Option::Some(42)))  // 42
    println(or_zero(Option::None()))    // 0
    return 0
}
```

注意：`unwrap_or` 与 `is_some` 会消费 `Option`，同一个值不要连续调用（每次用新的函数调用结果）。
组合子 `or` / `map` / `and_then` / `filter` / `unwrap_or_else` / `map_or` 支持按值捕获的 lambda；
`Result` 也有 `is_ok` / `is_err` / `unwrap_or` / `ok` / `map` / `map_err` / `unwrap_or_else` / `map_or`。

## 集合（list/arraylist/linkedlist）

标准库提供了三种集合实现：`List[T]` 接口、`ArrayList[T]` 和 `LinkedList[T]`。

### ArrayList

`ArrayList[T]` 是基于数组的顺序表，支持动态扩容。标准库提供了构造函数：

```ayanami
import "arraylist";

fn main() -> int {
    a = ArrayList::new[int]()      // 空表
    a.push(3)
    a.push(1)
    a.set(1, 5)              // 设置索引为 1 的元素为 5
    println(a.to_string())   // [3, 5]
    println(a.pop())         // 5
    return a.len()           // 1
}
```

需要预分配容量时用 `ArrayList::with_capacity[T](n)`；泛型实参也可以省略，由后续用法推断：

```ayanami
b = ArrayList::new()
b.push("hi")            // 由 push 推断 T = String
```

此外还支持 `is_empty` 和 `clear` 方法：

```ayanami
fn main() -> int {
    a = ArrayList::new[int]()
    println(a.is_empty())   // true
    a.push(1)
    a.clear()
    println(a.is_empty())   // true
    return 0
}
```

集合对元素类型**没有约束**：任意类型（包括未实现 `ToString` 的结构体、枚举）都可以放入；
只有调用 `to_string()` 时才要求元素实现 `ToString`。

`ArrayList` 还提供 `contains` / `index_of` / `remove`（要求元素实现 `Eq`：`same(ref self, ref Self other) -> bool`）、
`reverse`、`insert_at` / `remove_at`、`first` / `last` 等操作，以及函数式方法
`map` / `filter` / `fold` / `any` / `all` / `find` / `position` / `count` / `retain`：

```ayanami
import "arraylist"

a = ArrayList::new[int]()
a.push(1)
a.push(2)
a.push(3)
b = a.filter((int x) -> bool { return x > 1 })   // [2, 3]
sum = a.fold(0, (int acc, int x) -> int { return acc + x })  // 6
```

集合支持索引语法 `a[i]`（等价于 `a.index(i)`）。越界访问（`a.index(5)` 或 `a[5]`）、空表 `pop()` 都会在运行时 panic（退出码 101），错误位置指向你的调用行。`iter` 的回调支持按值捕获外部变量（见第 4 章）。

### 排序（sort）

排序基于 `Ord` 接口（`cmp(ref self, ref Self other) -> int`，返回 `<0 / 0 / >0`），内置 `int` 与 `String`（字典序）：

```ayanami
import "arraylist"
import "sort"

a = ArrayList::new[int]()
a.push(3)
a.push(1)
a.push(2)
a.sort()                 // [1, 2, 3]（稳定插入排序）
a.min()                  // Option[int]：Some(1)
a.binary_search(2)       // 1（要求已升序；未找到 -1）
```

自由函数形式是 `sort(a)`，另有兼容包装 `sort_int(a)` / `sort_string(a)`。自定义类型只要提供
`cmp` 方法即满足 `Ord` 约束。

### 哈希集合与哈希表

```ayanami
import "hashset"
import "hashmap"

s = HashSet::new[String]()
s.insert("a")             // true
s.contains("a")           // true

m = HashMap::new[String, int]()
m.insert("x", 1)
m.get("x").unwrap_or(0)   // 1
```

键类型需实现 `Hash` 接口（`hash` + `hash_eq`），内置 `int` 与 `String`；
实现为开放寻址，装填因子超过 0.5 自动扩容。

### LinkedList

`LinkedList[T]` 提供了链式结构的集合实现。你可以使用 `LinkedList::new[T]()` 创建新实例：

```ayanami
import "linkedlist";

fn main() -> int {
    l = LinkedList::new[int]()
    l.push(1)
    l.push(2)
    return 0
}
```

## 迭代器与 for-in（iter）

`for x in it { ... }` 基于迭代器协议：任何具备 `next(ref mut self) -> Option[T]` 方法的类型都可迭代：

```ayanami
import "iter"
import "arraylist"

a = ArrayList::new[int]()
a.push(1)
a.push(2)
a.push(3)

s = 0
for x in a.into_iter() {
    s = s + x            // 6
}
```

- `into_iter()` 消费集合，产出拥有型迭代器（`ArrayListIter` / `LinkedListIter` / `HashMapIter` / `HashSetIter`）；
- `for x in (start, end[, step])` 的区间形式保持原有语义；
- 自定义迭代器只需提供 `next`；`Iterator[T]` 接口形参走虚调用。

适配器（可任意嵌套）：

| 构造 | 说明 |
|---|---|
| `MapIter::new(it, f)` | 映射 |
| `FilterIter::new(it, pred)` | 过滤 |
| `TakeIter::new(it, n)` | 最多前 `n` 个 |
| `EnumerateIter::new(it)` | 产出 `Pair[usize, T]`（下标从 0 起） |
| `ZipIter::new(a, b)` | 逐对产出 `Pair[T, U]`，任一耗尽即结束 |

```ayanami
m = MapIter::new(a.into_iter(), (int x) -> int { return x * 10 })
f = FilterIter::new(m, (int x) -> bool { return x > 20 })
t = TakeIter::new(f, 2usize)

total = 0
for x in t { total = total + x }   // 30 + 40 = 70
```

## 转换与解析（convert）

```ayanami
import "convert"

parse_int_or("42", 0)         // 42（严格解析，失败用 fallback）
"2.5".try_parse_float()       // Option[float]
3.9.to_int()                  // 3（截断向零）
65.to_char()                  // 'A'
true.to_int()                 // 1
```

严格解析（`try_parse_int` / `try_parse_float` / `try_parse_bool`）不允许首尾空白，失败返回 `None`；
`try_parse_int_radix(base)` / `parse_int_radix(base)` 支持 2~36 进制，`is_float()` 判断整个串是否为合法浮点。
`parse_int_or` / `parse_float_or` / `parse_bool_or` 是带默认值的便捷包装。
`parse_int_result` / `parse_float_result` / `parse_bool_result` 返回 `Result[T, ParseError]`
（`Empty` / `Invalid`）。`Into[T]` 接口提供自然转换（`int -> float`、`char -> int`、`bool -> int`），
可作泛型约束（见第 8 章）。

## 文本处理（text）

```ayanami
import "text"

parts = split("a,b,c", ",")     // ArrayList[String]
join(parts, "-")                // "a-b-c"
replace("a-b", "-", "+")        // "a+b"
pad_left("42", 5, '0')          // "00042"
lines("a\nb")                   // ["a", "b"]（兼容 \r\n）
split_whitespace("a  b")        // ["a", "b"]
split_once("k=v", "=")          // Split { found, before, after }
count("the fox and the dog", "the")  // 2
```

`s.lines_iter()` / `s.split_whitespace_iter()` 是惰性版本（消费字符串，供 `for` 迭代）。

## 随机数（rand）

```ayanami
import "rand"

rng = Rng::new(42)              // 固定种子（序列可复现）
secret = rng.next_range(0, 100) // [0, 100) 内的整数
n = rng.next_int()              // 步进并返回新状态
rng.next_bool()                 // 随机布尔
rng.next_float01()              // [0.0, 1.0) 内的浮点
```

`Rng::from_entropy()` 用系统熵源创建非确定性的生成器。这是伪随机（LCG），不要用于加密。

## 时间（time）

```ayanami
import "time"

t0 = now_millis()      // 单调毫秒，适合计时/差值
now_unix()             // Unix 秒（墙上时钟）
elapsed_ms(t0)         // 自 t0 起的毫秒差
sleep_ms(100)          // 睡眠 100 毫秒（<= 0 直接返回）
```

## 命令行参数与环境变量（env）

```ayanami
import "env"

arg_count()                    // 参数个数（含程序名，下标 0）
arg(0)                         // 程序路径
args()                         // ArrayList[String]：全部参数
get_env("PATH").unwrap_or("")  // Option[String]
get_env_or("HOME", "(unset)")  // 缺失时用 fallback
```

## 文件读写（fs）

```ayanami
import "fs"

if write_file("/tmp/a.txt", "hello") {
    s = read_file("/tmp/a.txt").unwrap_or("")
}
exists("/tmp/a.txt")
lines = read_lines("/tmp/a.txt")   // Option[ArrayList[String]]
```

`read_file` / `read_lines` 返回 `Option`，`write_file` 返回是否成功；按字节读写（二进制安全）。

## 总结

本章介绍了 Ayanami 标准库的最新功能，包括增强的输入输出、数学函数、字符串处理、错误处理机制以及集合类型。所有 API 的详细列表请参见附录 A。

在下一章中，我们将深入探讨 Ayanami 中的错误处理机制，并介绍如何使用 `Result` 和 `Option` 来编写更健壮的程序。
