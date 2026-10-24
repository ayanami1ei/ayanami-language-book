# 第 18 章 unsafe 与底层编程

Ayanami 默认是安全语言：数组越界、所有权、借用都由编译器或运行时检查。但有些操作——内联汇编、
裸指针、union 字段读取、跳过边界检查——无法被安全规则完全覆盖，语言用 `unsafe` 把它们纳入显式边界。

`unsafe` 不引入新类型或运行时，只是把机器级操作标记出来；unsafe 块内的违规不影响块外的检查。

## unsafe 块与 unsafe fn

```ayanami
unsafe fn read_reg(int a) -> int {
    r = 0
    asm("mov $1, $0", in(reg) a, out(reg) r)
    return r
}

fn main() -> int {
    x = 0
    unsafe { x = read_reg(7) }     // 调用 unsafe fn 需要 unsafe 上下文
    if x != 7 { return 1 }
    return 0
}
```

- `unsafe { ... }` 是普通作用域（drop 语义不变），可以嵌套；
- `unsafe fn` 的函数体整体处于 unsafe 上下文，但**调用**它需要 unsafe；
- 以下操作在 safe 上下文会报错：`asm(...)`、调用 `unsafe fn`、读写 `static mut`、
  裸指针解引用与转换、`get_unchecked`、`bits::reinterpret`、`offset` / `read_volatile` /
  `write_volatile`、union 字段读取。

## 裸指针 `*T`

`*T` 是 Copy 标量（LLVM `ptr`），无所有权、无 drop，可空：

```ayanami
struct Box2 {
    int v
}

fn main() -> int {
    p = Ptr::new(Box2 { v = 41 })
    unsafe {
        raw = p.as_ptr()             // Ptr[T] -> *T
        if (*raw).v != 41 { return 1 }
        (*raw).v = 42                // 解引用写
        addr = raw as usize          // 指针 -> 整数
        q = addr as *Box2            // 整数 -> 指针
        if (*q).v != 42 { return 2 }
    }
    x = 7
    unsafe {
        r = ref mut x as *int        // 引用 -> 裸指针
        *r = 9
        if x != 9 { return 3 }
    }
    return 0
}
```

- 解引用 `*p` 读、`*p = v` 写；字段访问写 `(*p).field`；
- 转换（`int` / `ref` / `Ptr[T]` ↔ `*T`）都需 unsafe；`null` 可以 `as *T` 构造空指针；
- 指针比较 `==` / `!=` 是地址比较，无需 unsafe；
- `*T` 可用于签名、局部变量、结构体字段，例如
  `extern "C" fn memcpy(*void dst, *void src, usize n) -> *void`。

## `Ptr[T]`：拥有指针

`Ptr[T]` 是安全代码里的堆拥有指针（内部是拥有值），适合树、链表等递归结构：

```ayanami
import "option"

struct Tree {
    int v
    Option[Ptr[Tree]] left
    Option[Ptr[Tree]] right
}

t = Tree::leaf(2)                 // 内部 Ptr::new(...)
t.left = Option::Some(Tree::leaf(1))
t.right = Option::Some(Tree::leaf(3))
if t.contains(1) { /* ... */ }
```

- `Ptr::new(v)` 堆分配；`p.get()` / `p.get_mut()` 借用 pointee；
- 字段与方法自动解引用（`p.x`、`p.method()` 等价于 `p.get().x` / `p.get().method()`）；
- 移动语义、作用域结束递归释放；`clone` 深拷贝 pointee；
- 没有裸 `null`：空值用 `Option[Ptr[T]]`。

## 引用协议与 Arena

编译器按**方法形态**识别“引用协议”：任何类型只要有

```ayanami
fn get(ref self) -> ref T
fn get_mut(ref mut self) -> ref mut T   // 可选
```

就支持字段/方法自动解引用（`p.x` ≡ `p.get().x`），`Ptr[T]`、`ref T`、`*T` 都是它的实现；
协议调用在 release 下会内联为裸访问（零开销，objdump 中 getter 调用为 0）。

基于协议可以在安全代码里实现 **arena + 句柄**（稳定地址、批量释放，适合图 / 双向链表）：

```ayanami
#[copy]
struct Idx {
    usize v
}

struct Arena[T] {
    [Ptr[T]] data
    usize len
    usize cap
}

impl[T] Arena[T] {
    fn alloc(ref mut self, T v) -> Idx { /* Ptr::new(v) 存入 data，返回新下标 */ }
    fn get(ref self, Idx i) -> ref T { /* 越界句柄 panic 101 */ }
}
```

- arena 拥有全部节点，句柄（`#[copy]` 结构体）不拥有节点，可自由复制；
- 越界句柄在运行时 panic（退出码 101，与数组越界一致）；
- 自引用结构、图、双向链表都用「arena + 句柄」表达，不引入 RC/GC；
- 自定义解析器（ID → 句柄 → 节点）遵循同一协议（`resolve` 方法形态）。

完整设计见主仓 `docs/reference-model.md`。

## 指针算术、volatile 与 reinterpret

```ayanami
a = [1, 2, 3, 4]
unsafe {
    p = ref mut a as *int
    q = p.offset(2)              // 按元素步长前进
    if *q != 3 { return 1 }
    v = p.read_volatile()        // volatile 读（MMIO 语义，不被优化掉）
    p.write_volatile(20)         // volatile 写
}
x = 3.5
unsafe {
    b = bits::reinterpret[int](x)     // f64 位模式 -> i64
    y = bits::reinterpret[float](b)   // 再解释回来
}
```

`bits::reinterpret[T]` 要求两侧同尺寸且为 Copy 类型（基元、指针等），否则报错。

## union

```ayanami
union Value {
    int i
    float f
    *char p
}

v = Value { i = 42 }      // 字面量恰好初始化一个字段（其余别名清零）
v.f = 1.5                 // 字段写入无需 unsafe
unsafe {
    x = v.i               // 字段读取需 unsafe（共享存储）
}
```

字段共享同一存储（布局取最大字段的 size/align）；字段限**无 drop 类型**
（基元、指针、引用、无堆结构体），泛型 union 暂不支持。

## 数组越界检查与 get_unchecked

固定长度数组 `[T; n]` 的读写默认做越界检查（越界 panic 101，位置指向调用行）：

```ayanami
a = [10, 20, 30]
i = 2
x = a[i]                        // 有检查
unsafe {
    y = a.get_unchecked(i)      // 跳过检查，调用方保证不越界
}
```

动态数组 `[T]`（无长度信息，如 `extern [char]` 缓冲）保持未检查原语；
`ArrayList` 等带容量/长度字段的类型按容量字段检查。

## 与 FFI 的关系

`extern "C"` 声明与 `#[export]` 的用法见第 15 章；`*void` / `*T` 是跨语言指针的自然类型。
当前 `extern "C"` 调用本身尚未纳入 unsafe 门控（迁移中），但裸指针的构造与解引用已经需要 unsafe。

## 小结

- `unsafe {}` / `unsafe fn` 圈出机器级操作；asm、static mut、裸指针操作等都必须放在其中；
- `*T` 裸指针用于 FFI 与极致控制，解引用/转换需 unsafe；
- `Ptr[T]` 是安全拥有指针，表达树/递归结构；
- union、reinterpret、volatile、`get_unchecked` 都通过 unsafe 显式标注风险。

下一章我们学习并发与异步，然后进入全书收官项目。
