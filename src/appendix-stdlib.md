# 附录 A：标准库 API 参考

Ayanami 标准库以预编译 `.lcl` 文件形式随编译器一同分发，使用 `import "模块名"` 即可导入。`std` 模块是主入口，等价于同时导入 `io`、`string`、`math`、`list`、`linkedlist` 与 `arraylist`，并包含 `Error` 接口、`Result[T, E]` 与 `Option[T]` 枚举。

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
| `s.index(int i) -> char` | 按索引访问字符（也可写 `s.data[i]`） |
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
| `s.substring(int start, int end) -> String` | 区间子串（越界自动夹取） |
| `s.trim() -> String` | 去首尾空白 |
| `s.to_upper() -> String` / `s.to_lower() -> String` | 大小写转换 |
| `s.repeat(int n) -> String` | 重复拼接 |
| `s.parse_int() -> int` | 十进制解析（非法输入返回 0） |
| `s.is_int() -> bool` | 是否为合法十进制整数 |

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

| 方法 | 说明 |
|------|------|
| `try_unwrap(self) -> T` | 提取成功值；遇到 `Err` 时返回 0 |

### Option[T]

```ayanami
enum Option[T] {
    Some(T),
    None,
}
```

| 方法 | 说明 |
|------|------|
| `unwrap_or(self, T default) -> T` | `Some` 时返回内部值，否则返回 `default` |
| `is_some(self) -> bool` | 是否为 `Some` |

## 集合模块

### list 模块

`List[T]` 接口（`list.lcl`）：

```ayanami
interface List[T:ToString] {
    fn push(ref mut self, T val);
    fn index(ref self, int index)->T;
    fn to_string(ref self)->String;
    fn len(ref self)->int;
    fn iter(ref self, fn(T) f);
}
```

### arraylist 模块

`ArrayList[T]` 顺序表（`arraylist.lcl`）：

```ayanami
struct ArrayList[T:ToString] {
    [T] data
    int len
    int capability
}
```

| 方法 | 说明 |
|------|------|
| `push(ref mut self, T val)` | 追加元素，自动扩容 |
| `index(ref self, int i) -> T` | 按下标读取 |
| `set(ref mut self, int i, T v)` | 按下标写入 |
| `pop(ref mut self) -> T` | 弹出末元素（调用方保证非空） |
| `len(ref self) -> int` | 元素个数 |
| `is_empty(ref self) -> bool` | 是否为空 |
| `clear(ref mut self)` | 清空（保留底层缓冲） |
| `iter(ref self, fn(T) f)` | 依次调用 `f` |
| `to_string(ref self) -> String` | 形如 `[1, 2, 3]` |

### linkedlist 模块

`LinkedList[T]`（`linkedlist.lcl`，当前为数组缓冲实现，接口与 `List` 一致）：

```ayanami
struct LinkedList[T:ToString] {
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
| `to_string(ref self) -> String` | 字符串形式 |

注意：`pop()` 调用方需自行保证非空；`clear()` 保留底层缓冲。

## mir 模块（实验性）

`mir` 模块仅供编译器插件 / 宏使用，提供 `MirFunction` 只读视图、`scan_effects`、`warn`/`error` 诊断、`replace_int`/`replace_float`/`replace_bool`/`replace_char`/`replace_with`/`delete_stmt` 编辑辅助。

## 更新源

本附录以主仓库 [`std/README.md`](https://github.com/ayanami1ei/Ayanami-language/blob/rust/std/README.md) 为准，标准库更新时请同步该文件。
