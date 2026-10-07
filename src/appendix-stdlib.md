# 附录 A：标准库 API 参考

Ayanami 标准库以预编译 `.lcl` 文件形式随编译器一同分发，使用 `import "模块名"` 即可导入。`std` 模块是主入口，等价于同时导入 `io`、`string`、`math`、`list`、`linkedlist`、`arraylist`、`option` 与 `panic`，并提供 `Error` 接口与 `Result[T, E]`。

## io 模块

| 函数 | 说明 |
|------|------|
| `getchar() -> int` | 读取一个字符 |
| `putchar(int c)` | 输出一个字符（ASCII 码） |
| `print(ref String n)` | 输出字符串（不消费） |
| `print(int n)` / `print(usize n)` / `print(float f)` / `print(bool b)` / `print(char c)` | 输出常用类型（重载） |
| `println()` | 输出换行 |
| `println(ref String s)` | 输出字符串并换行 |
| `println(int/float/bool/char)` | 输出常用类型并换行（重载） |
| `read_line() -> String` | 读取一行（不含换行；EOF 返回空串） |
| `read_int() -> int` | 读一行并宽松解析（`trim` + `parse_int`，失败 0） |
| `try_read_int() -> Option[int]` | 读一行并严格解析（失败 `None`） |
| `read_float() -> float` / `try_read_float()` | 浮点行输入（宽松 / `Option`） |
| `read_bool() -> bool` / `try_read_bool()` | 布尔行输入（宽松 / `Option`） |

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
| `gcd(int a, int b)` / `lcm(int a, int b)` | 最大公约数 / 最小公倍数（`gcd(0,0)==0`；`lcm` 含 0 返回 0） |
| `is_prime(int n)` | 是否素数（`n < 2` 为 false） |
| `n.is_even()` / `n.is_odd()` | 奇偶判断（`impl int`） |

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
| `exp(float x)` / `ln(float x)` / `log10(float x)` | e^x / 自然对数 / 常用对数 |
| `round(float x)` / `trunc(float x)` | 四舍五入 / 向零截断 |
| `f.is_nan()` | 是否 NaN（`impl float`） |

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
| `s.len() -> usize` | 长度 |
| `s.index(usize i) -> char` | 按索引访问字符（也可写 `s[i]`；越界 panic 101） |
| `s.add(ref String other) -> String` | 拼接（`+` 运算符） |
| `s.eq(ref String other) -> bool` | 相等比较（`==`） |
| `s.ne(ref String other) -> bool` | 不等比较（`!=`） |
| `s.lt / gt / le / ge(ref String) -> bool` | 字典序比较（`<` `>` `<=` `>=`） |
| `s.copy() -> String` | 深拷贝 |
| `s.to_string() -> String` | 消费 self 并返回（ToString 接口） |
| `42.to_string()` | int/float/char/bool → String |
| `3.14159.to_fixed(2)` / `1234.5.to_sci(2)` | 定点 / 科学计数法格式化 |
| `255.to_hex()` / `to_hex_upper()` / `to_bin()` / `to_oct()` | 整数进制格式化 |
| `s.is_empty() -> bool` | 是否空串 |
| `s.contains(ref String) -> bool` | 是否包含子串 |
| `s.index_of(ref String) -> int` | 子串位置（未找到 -1） |
| `s.starts_with(ref String) -> bool` / `s.ends_with(ref String) -> bool` | 前缀/后缀 |
| `s.substring(usize start, usize end) -> String` | 区间 `[start, end)`，越界自动夹取 |
| `s.trim() -> String` | 去首尾空白 |
| `s.trim_start()` / `s.trim_end()` | 去首部 / 尾部空白 |
| `s.remove_prefix(p)` / `s.remove_suffix(s)` | 有则去掉，无则返回拷贝 |
| `s.to_upper() -> String` / `s.to_lower() -> String` | 大小写转换 |
| `s.repeat(usize n) -> String` | 重复拼接 |
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
| `try_unwrap(self) -> T` | 提取成功值；遇到 `Err` 时返回 0 |
| `is_ok()` / `is_err() -> bool` | 判断 |
| `unwrap_or(default) -> T` | `Ok` 取值，`Err` 返回默认值 |
| `ok() -> Option[T]` | `Ok(v)` → `Some(v)`，`Err` → `None` |
| `map[U](Fn(T) -> U)` / `map_err[F](Fn(E) -> F)` | 映射成功 / 错误值 |
| `unwrap_or_else(Fn(E) -> T)` / `map_or[U](default, Fn(T) -> U)` | 惰性默认值 / 映射或默认值 |
| `unwrap_or_else(Fn(E) -> T)` / `map_or[U](default, Fn(T) -> U)` | 惰性默认值 / 映射或默认值 |

> `?` 错误传播与 `match` 均可正常使用。

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
| `or(self, Option[T] other) -> Option[T]` | `Some` 返回自身，`None` 返回 `other` |
| `map[U](Fn(T) -> U)` | `Some(v)` → `Some(f(v))`，`None` 不变 |
| `and_then[U](Fn(T) -> Option[U])` | `Some(v)` → `f(v)`（链式），`None` 不变 |
| `filter(Fn(T) -> bool)` | 谓词为假时变 `None` |
| `unwrap_or_else(Fn() -> T)` / `map_or[U](default, Fn(T) -> U)` | 惰性默认值 / 映射或默认值 |

方法都会消费 `self`，同一个值不要连续调用。组合子接受**按值捕获**的 lambda，
例如 `limit = 2; o.map((int x) -> int { return x + limit })`。

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

## option 模块

`import "option"` 单独导入时提供 `Option[T]`、`Result[T, E]` 与 `Error` 接口（`std` 已包含）：

| 构造 / 方法 | 说明 |
|---|---|
| `Option::Some(v)` / `Option::None()` | 构造 |
| `o.is_some() -> bool` / `o.unwrap_or(default) -> T` | 消费 `self` |
| `Result::Ok(v)` / `Result::Err(e)` | 构造 |

## convert 模块

`import "convert"`

| 方法 / 函数 | 说明 |
|---|---|
| `s.try_parse_int() -> Option[int]` | 严格整数解析（允许 `+/-`；空白/非法/溢出 → `None`） |
| `s.try_parse_float() -> Option[float]` | `[+/-] digits [. digits]`，无指数 |
| `s.try_parse_bool() -> Option[bool]` | 仅 `"true"` / `"false"` |
| `s.try_parse_int_radix(base) -> Option[int]` / `s.parse_int_radix(base) -> int` | 按进制解析（base 2..36；宽松版失败 0） |
| `s.is_float() -> bool` | 整个串是否为合法浮点 |
| `s.parse_float() -> float` | 宽松，失败 0.0 |
| `parse_int_or(ref String, int fallback) -> int` 等 | 带默认值的便捷包装 |
| `parse_int_result(ref String) -> Result[int, ParseError]` 等 | Result 版（`Empty` / `Invalid`） |
| `3.to_float()` / `3.into()` | int → float |
| `3.9.to_int()` | 截断向零 |
| `'A'.to_int()` / `65.to_char()` | char ↔ int |
| `true.to_int()` | bool → int |

`Into[T]` 接口：`fn into(self) -> T`（自然转换：int→float、char→int、bool→int），可作泛型约束。

## text 模块

`import "text"`

| 函数 | 说明 |
|---|---|
| `split(ref String s, ref String sep) -> ArrayList[String]` | 按分隔符切分（`sep` 为空返回原串列表） |
| `join(ref ArrayList[String] list, ref String sep) -> String` | 连接元素 |
| `replace(ref String s, ref String old, ref String new) -> String` | 替换全部出现 |
| `pad_left(ref String s, usize width, char fill) -> String` | 左侧填充到 `width` |
| `pad_right(ref String s, usize width, char fill) -> String` | 右侧填充到 `width` |
| `lines(ref String s) -> ArrayList[String]` | 按行拆分（兼容 `\r\n`，末尾换行不产生空行） |
| `split_whitespace(ref String s) -> ArrayList[String]` | 按空白拆分（连续空白视为一个） |
| `count(ref String s, ref String needle) -> int` | 非重叠出现次数（needle 为空返回 0） |
| `s.lines_iter()` / `s.split_whitespace_iter()` | 惰性迭代器（消费 `s`，供 `for` 使用） |
| `split_once(ref String s, ref String sep) -> Split` | 首个分隔符处切分（`Split { found, before, after }`） |

## rand 模块

`import "rand"`

| API | 说明 |
|---|---|
| `Rng::new(int seed)` | 固定种子（归一化到 `[1, 2^31)`） |
| `Rng::from_entropy()` | 系统熵源（非确定性） |
| `r.next_int() -> int` | 步进并返回新状态 `[0, 2^31)` |
| `r.next_range(int lo, int hi) -> int` | `[lo, hi)`；`hi <= lo` 返回 `lo` |
| `r.next_bool() -> bool` | 随机布尔 |
| `r.next_float01() -> float` | `[0.0, 1.0)` 内的浮点 |

伪随机（LCG），序列可复现，非加密用途。

## time 模块

`import "time"`

| 函数 | 说明 |
|---|---|
| `now_millis() -> int` | 单调毫秒（适合计时/差值） |
| `now_unix() -> int` | Unix 秒（墙上时钟） |
| `elapsed_ms(int start) -> int` | 自 `start`（来自 `now_millis`）起的毫秒差 |
| `sleep_ms(int ms)` | 睡眠 `ms` 毫秒（`<= 0` 直接返回） |

## env 模块

`import "env"`

| 函数 | 说明 |
|---|---|
| `arg_count() -> int` | 参数个数（含程序名，下标 0） |
| `arg(int i) -> String` | 第 i 个参数（越界返回空串） |
| `get_env(String name) -> Option[String]` | 环境变量（缺失 `None`） |
| `get_env_or(String name, String fallback) -> String` | 缺失用 fallback |
| `args() -> ArrayList[String]` | 全部命令行参数（含程序名） |

## fs 模块

`import "fs"`

| 函数 | 说明 |
|---|---|
| `exists(String path) -> bool` | 文件是否存在 |
| `read_file(String path) -> Option[String]` | 读取整个文件（失败 `None`） |
| `write_file(String path, String data) -> bool` | 覆盖写入 |
| `read_lines(String path) -> Option[ArrayList[String]]` | 按行拆分（`\r\n` 兼容；失败 `None`） |

## test 模块

`import "test"` 提供测试框架：`#[test]` / `#[should_panic]` 标注与
`#assert` / `#assert_eq` / `#assert_ne` / `#assert_lt` / `#assert_contains` /
`#assert_some` / `#assert_none` / `#assert_close` / `#fail` 断言宏，详见第 17 章。

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
    fn index(ref self, usize index)->T;
    fn len(ref self)->usize;
    fn iter(ref self, Fn(T) f);
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
| `index(ref self, usize i) -> T` | 按下标读取（或 `a[i]`；越界 panic 101） |
| `set(ref mut self, usize i, T v)` | 按下标写入（越界 panic 101） |
| `pop(ref mut self) -> T` | 弹出末元素（空表 panic 101） |
| `len(ref self) -> usize` | 元素个数 |
| `is_empty(ref self) -> bool` | 是否为空 |
| `clear(ref mut self)` | 清空（保留底层缓冲） |
| `contains(v)`（`T: Eq`） | 是否包含（线性查找） |
| `index_of(v) -> int`（`T: Eq`） | 首次出现下标；未找到 -1 |
| `remove(v) -> bool`（`T: Eq`） | 删除首个等于 `v` 的元素 |
| `reverse()` | 原地反转 |
| `insert_at(usize i, v)` / `remove_at(usize i)` | 指定位置插入 / 删除；越界 panic 101 |
| `first()` / `last() -> T` | 首 / 末元素；空表 panic 101 |
| `map[U](Fn(T) -> U)` | 映射为新表 |
| `filter(Fn(T) -> bool)` | 过滤为新表 |
| `fold[U](init, Fn(U, T) -> U)` | 折叠 |
| `any(Fn(T) -> bool)` / `all(Fn(T) -> bool)` | 存在 / 全部满足 |
| `find(Fn(T) -> bool) -> Option[T]` | 首个满足的元素 |
| `position(Fn(T) -> bool) -> int` / `count(Fn(T) -> bool) -> usize` | 首个满足下标（无则 -1）/ 个数 |
| `retain(Fn(T) -> bool)` | 原地保留满足谓词的元素 |
| `iter(ref self, Fn(T) f)` | 依次调用 `f`（支持按值捕获） |
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
| `LinkedList::new[T]()` / `LinkedList::with_capacity[T](n)` | 构造空表 / 预分配容量 |
| `push` / `index` / `set` / `pop` / `clear` / `is_empty` | 与 `ArrayList` 对齐 |
| `contains` / `index_of` / `remove` / `reverse` / `insert_at` / `remove_at` / `first` / `last` | 同 `ArrayList`（`T: Eq` 约束同上） |
| `iter` / `to_string` | 遍历 / 字符串形式（`T: ToString`） |

注意：越界与空表 `pop` 会运行时 panic（退出码 101，位置指向调用行）；`clear()` 保留底层缓冲。

### sort 模块

`import "sort"`；基于 `Ord` 接口（`cmp(ref self, ref Self other) -> int`，内置 `int` / `String` 字典序）：

| 入口 | 说明 |
|---|---|
| `a.sort()` / `sort(a)` | 泛型升序（稳定插入排序） |
| `sort_int(a)` / `sort_string(a)` | 兼容包装 |
| `a.min()` / `a.max() -> Option[T]` | 最小 / 最大元素（空表 `None`） |
| `a.is_sorted() -> bool` | 是否已升序 |
| `a.binary_search(v) -> int` | 二分查找（要求已升序）；未找到 -1 |

### hash / hashset / hashmap 模块

`Hash` 接口：`hash(ref self) -> int` + `hash_eq(ref self, ref Self other) -> bool`（内置 `int` / `String`）。

| 方法 | 说明 |
|---|---|
| `HashSet::new[K]()` | 空集合 |
| `s.insert(k) -> bool` / `s.contains(k) -> bool` / `s.remove(k) -> bool` | 插入 / 包含 / 删除 |
| `s.to_list() -> ArrayList[K]` | 所有元素（浅拷贝） |
| `s.for_each(Fn(K))` / `s.fold[U](init, Fn(U, K) -> U)` | 遍历 / 折叠 |
| `s.retain(Fn(K) -> bool)` | 原地保留 |
| `s.union(other)` / `s.intersection(other)` / `s.difference(other)` | 并 / 交 / 差（新集合） |
| `s.into_iter() -> HashSetIter[K]` | 拥有型迭代器 |
| `s.len()` / `s.is_empty()` / `s.to_string()`（`K: ToString`） | 长度 / 空判断 / 字符串形式 |
| `HashMap::new[K, V]()` | 空表 |
| `m.insert(k, v) -> bool` / `m.get(k) -> Option[V]` | 插入（覆盖返回 false）/ 取值（浅拷贝） |
| `m.contains_key(k) -> bool` / `m.remove(k) -> bool` | 包含键 / 删除 |
| `m.keys()` / `m.values() -> ArrayList[K]/ArrayList[V]` | 所有键 / 值（浅拷贝） |
| `m.for_each(Fn(K, V))` / `m.fold[U](init, Fn(U, K, V) -> U)` | 遍历 / 折叠 |
| `m.retain(Fn(K, V) -> bool)` | 原地保留满足谓词的键值对 |
| `m.to_string()`（`K/V: ToString`） | 形如 `{a: 1}` |
| `m.into_iter() -> HashMapIter[K, V]` | 拥有型迭代器，产出 `Pair[K, V]` |
| `m.len()` / `m.is_empty()` | 长度 / 是否为空 |

开放寻址（线性探测），装填因子超过 0.5 自动扩容。

## iter 模块

`import "iter"`；`for x in it` 基于 `next(ref mut self) -> Option[T]`（结构化匹配，无需显式实现接口）。

| API | 说明 |
|---|---|
| `a.into_iter()` / `l.into_iter()` | `ArrayListIter[T]` / `LinkedListIter[T]`（消费集合） |
| `m.into_iter()` / `s.into_iter()` | `HashMapIter[K, V]`（产出 `Pair[K, V]`）/ `HashSetIter[K]` |
| `Iterator[T]` 接口 | `fn next(ref mut self) -> Option[T]`；接口形参走虚调用 |
| `MapIter::new(it, f)` / `FilterIter::new(it, pred)` | 映射 / 过滤 |
| `TakeIter::new(it, n)` / `EnumerateIter::new(it)` / `ZipIter::new(a, b)` | 取前 n 个 / 带下标 / 逐对 |

`Pair[A, B] { first, second }` 用于 zip / enumerate 的元素；适配器可任意嵌套。

## mir 模块（实验性）

`mir` 模块仅供编译器插件 / 宏使用，提供 `MirFunction` 只读视图、`scan_effects`、`warn`/`error` 诊断、`replace_int`/`replace_float`/`replace_bool`/`replace_char`/`replace_with`/`delete_stmt` 编辑辅助。

## 更新源

本附录以标准库子仓 [Ayanami-std](https://github.com/ayanami1ei/Ayanami-std) 的
[`docs/modules/`](https://github.com/ayanami1ei/Ayanami-std/tree/main/docs/modules)（分模块 API）与
[`docs/dev/`](https://github.com/ayanami1ei/Ayanami-std/tree/main/docs/dev)（测试 / 运行时 / 设计）为准，标准库更新时请同步。
