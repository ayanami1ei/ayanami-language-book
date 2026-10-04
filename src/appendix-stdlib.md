# 附录 A：标准库 API 参考

Ayanami 标准库以预编译 `.lcl` 文件形式随编译器一同分发，使用 `import "模块名"` 即可导入。`std` 模块是主入口，等价于同时导入 `io`、`string`、`math`、`list`、`linkedlist`、`arraylist` 与 `panic`，并包含 `Error` 接口、`Result[T, E]` 与 `Option[T]` 枚举。

## io 模块

| 函数 | 说明 |
|------|------|
| `getchar() -> int` | 读取一个字符 |
| `putchar(int c)` | 输出一个字符（ASCII 码） |
| `print(ref String n)` | 输出字符串（不消费） |
| `print(int n)` / `print(float f)` / `print(bool b)` / `print(char c)` | 输出常用类型（重载） |
| `println()` | 输出换行 |
| `println(ref String s)` | 输出字符串并换行 |
| `println(int/float/bool/char)` | 输出常用类型并换行（重载） |

```ayanami
import "io";

fn main() -> int {
    println("hello")
    println(42.to_string())
    putchar(65);
    c = getchar();
    return 0
}
```

## math 模块

### 整数函数

| 函数 | 说明 |
|------|------|
| `abs(int x)` | 绝对值 |
| `min(int a, int b)` | 最小值 |
| `max(int a, int b)` | 最大值 |
| `clamp(int x, int lo, int hi)` | 限制范围 |
| `pow(int base, int exp)` | 整数幂 |

### 浮点函数

| 函数 | 说明 |
|------|------|
| `abs(float x)` | 绝对值 |
| `min(float a, float b)` | 最小值 |
| `max(float a, float b)` | 最大值 |
| `clamp(float x, float lo, float hi)` | 限制范围 |
| `pow(float base, int exp)` | 浮点幂 |
| `sqrt(float x)` | 平方根 |
| `floor(float x)` / `ceil(float x)` | 向下/向上取整 |

## string 模块

### String 结构体定义

```ayanami
struct String {
    [char] data
    int len
}
```

### String 方法

| 方法 | 说明 |
|------|------|
| `s.len() -> int` | 长度 |
| `s.index(int i) -> char` | 按索引访问字符（也可写 `s[i]`；越界 panic 101） |
| `s.add(ref String other) -> String` | 拼接（`+` 运算符） |
| `s.eq(ref String other) -> bool` | 相等比较（`==`） |
| `s.ne(ref String other) -> bool` | 不等比较（`!=`） |
| `s.copy() -> String` | 深拷贝 |
| `s.to_string() -> String` | 消费 self 并返回（ToString 接口） |
| `42.to_string()` | int/float/char/bool → String |
| `s.is_empty() -> bool` | 是否空串 |
| `s.contains(ref String) -> bool` | 是否包含子串 |
| `s.index_of(ref String) -> int` | 子串位置（未找到 -1） |
| `s.starts_with(ref String) -> bool` / `s.ends_with(ref String) -> bool` | 前缀/后缀 |
| `s.substring(int start, int end) -> String` | 区间 `[start, end)`，越界自动夹取 |
| `s.trim() -> String` | 去首尾空白 |
| `s.to_upper() -> String` / `s.to_lower() -> String` | 大小写转换 |
| `s.repeat(int n) -> String` | 重复拼接 |
| `s.parse_int() -> int` | 前缀式解析：跳过前导空白/正负号，遇非数字停止 |
| `s.is_int() -> bool` | 整个串是否为合法十进制整数 |

### char 工具方法

| 方法 | 说明 |
|------|------|
| `c.is_digit()` / `c.is_alpha()` / `c.is_alnum()` / `c.is_space()` | 分类判断 |
| `c.is_upper()` / `c.is_lower()` | 大小写判断 |
| `c.to_digit() -> int` | 数字字符 → 数值 |
| `c.to_upper() -> char` / `c.to_lower() -> char` | 大小写转换 |
| `char_code(char c) -> int` | 字符转 ASCII 码 |

```ayanami
import "string";

fn main() -> int {
    s = "Hello World"
    println(s.len())                    // 11
    println(s.index(0))                 // 'H'
    println(s.substring(6, 11))         // "World"
    println(s.to_upper())               // "HELLO WORLD"
    println(s.parse_int())              // 0（非数字）
    return 0
}
```

## std 模块

### Error 接口

```ayanami
interface Error {
    fn what(ref self) -> String;
}
```

用于表示错误类型，提供错误信息描述。

### Result[T, E]

```ayanami
enum Result[T, E] {
    Ok(T),
    Err(E),
}
```

构造：`Result::Ok(v)` / `Result::Err(e)`（泛型实参由返回类型推导）。

| 方法 | 说明 |
|------|------|
| `try_unwrap(self) -> T` | 提取成功值；遇到 `Err` 时返回 0（当前受编译器 bug 影响，见下） |

> **当前限制**：泛型枚举载荷的 `match` 与 `try_unwrap()` 因编译器 bug 暂不可用
> （主仓 issue #68 / #69）；修复前用 `_tag` / `_data_Ok._0` / `_data_Err._0` 字段访问。
> `?` 错误传播可以正常使用。

### Option[T]

```ayanami
enum Option[T] {
    Some(T),
    None,
}
```

构造：`Option::Some(v)` / `Option::None()`。

| 方法 | 说明 |
|------|------|
| `unwrap_or(self, T default) -> T` | `Some` 时返回内部值，否则返回 `default` |
| `is_some(self) -> bool` | 是否为 `Some` |

两个方法都会消费 `self`，同一个值不要连续调用。

## panic 模块

`import "panic"` 提供运行时错误支持：

| 形式 | 说明 |
|------|------|
| `#panic("消息")` | 函数宏：在调用点展开，自动带上行号/列号/文件名 |
| `panic_at(line, col, file, msg)` | 底层函数（宏与库内部使用） |
| `panic_bounds_at(line, col, file, index, len)` | 越界专用（集合/字符串边界检查使用） |

运行时输出到 stderr，退出码 **101**：

```ayanami
import "panic"

fn main() -> int {
    #panic("boom")     // thread 'main' panicked at main.aya:4:5: boom
    return 0
}
```

标准库在越界/空表时自动 panic，位置指向调用行（`arr[i]`、`a.index(i)`、`s[i]`、空表 `a.pop()`）。语言没有 `try/catch`，panic 不可捕获；可恢复的错误请使用 `Result`。

## 构造函数（显式泛型调用）

命名空间构造函数使用 `类型::new[T](...)` 形式；泛型实参可省略，由上下文或后续用法推断：

| 构造 | 说明 |
|------|------|
| `ArrayList::new[T]()` | 空表 |
| `ArrayList::with_capacity[T](n)` | 预分配容量（n <= 0 时等价于 `new`） |
| `LinkedList::new[T]()` | 空链表 |
| `String::empty()` / `String::new()` | 空字符串 |
| `String::new([char] data, int len)` | 从拥有所有权的字符缓冲构造 |

```ayanami
a = ArrayList::new[int]()
b = ArrayList::with_capacity[String](8)
c = LinkedList::new[int]()
s = String::new(['h', 'i'], 2)
```

## 集合模块

### list 模块

`List[T]` 接口（`list.lcl`，元素无约束；`to_string` 由具体集合在 `impl[T:ToString]` 中提供）：

```ayanami
interface List[T] {
    fn push(ref mut self, T val);
    fn index(ref self, int index)->T;
    fn len(ref self)->int;
    fn iter(ref self, fn(T) f);
}
```

### arraylist 模块

`ArrayList[T]` 顺序表（`arraylist.lcl`；元素无约束，`to_string` 要求 `T: ToString`）：

```ayanami
struct ArrayList[T] {
    [T] data
    int len
    int capability
}
```

| 方法 | 说明 |
|------|------|
| `ArrayList::new[T]()` / `ArrayList::with_capacity[T](n)` | 构造空表 / 预分配容量 |
| `push(ref mut self, T val)` | 追加元素，自动扩容 |
| `index(ref self, int i) -> T` | 按下标读取（或 `a[i]`；越界 panic 101） |
| `set(ref mut self, int i, T v)` | 按下标写入（越界 panic 101） |
| `pop(ref mut self) -> T` | 弹出末元素（空表 panic 101） |
| `len(ref self) -> int` | 元素个数 |
| `is_empty(ref self) -> bool` | 是否为空 |
| `clear(ref mut self)` | 清空（保留底层缓冲） |
| `iter(ref self, fn(T) f)` | 依次调用 `f`（回调不能捕获外部变量） |
| `to_string(ref self) -> String` | 形如 `[1, 2, 3]`（要求 `T: ToString`） |

### linkedlist 模块

`LinkedList[T]`（`linkedlist.lcl`，当前为数组缓冲实现，接口与 `List` 一致；元素无约束）：

```ayanami
struct LinkedList[T] {
    [T] data
    int len
    int capability
}
```

| 方法 | 说明 |
|------|------|
| `LinkedList::new[T]() -> LinkedList[T]` | 创建空表（命名空间函数） |
| `push(ref mut self, T val)` | 追加元素 |
| `index(ref self, int i) -> T` | 按下标读取 |
| `len(ref self) -> int` | 元素个数 |
| `iter(ref self, fn(T) f)` | 依次调用 `f` |
| `to_string(ref self) -> String` | 字符串形式（要求 `T: ToString`） |

注意：越界与空表 `pop` 会运行时 panic（退出码 101，位置指向调用行）；`clear()` 保留底层缓冲。

## mir 模块（实验性）

`mir` 模块仅供编译器插件 / 宏使用，提供 `MirFunction` 只读视图、`scan_effects`、`warn`/`error` 诊断、`replace_int`/`replace_float`/`replace_bool`/`replace_char`/`replace_with`/`delete_stmt` 编辑辅助。

## 更新源

本附录以标准库子仓 [Ayanami-std](https://github.com/ayanami1ei/Ayanami-std) 的 `README.md` 与 [`docs/`](https://github.com/ayanami1ei/Ayanami-std/tree/main/docs)（分模块教程式文档）为准，标准库更新时请同步。
