# 第 19 章 并发与异步

本章介绍 Ayanami 的两条并发路径：**线程 + 锁/通道**（std）与**异步 `#[async]` / `await`**（编译器）。

## 线程（thread）

```ayanami
import "thread"

fn main() -> int {
    t = Thread::spawn(() -> void {
        // 在新线程里工作
    })
    t.join()          // 等待结束
    return 0
}
```

- `Thread::spawn(Fn() -> void)` 把闭包**按值**搬入运行时（捕获值所有权转移），`join()` 等待线程结束并回收；
- `current_id()` 取内核 tid，`yield_now()` 让出 CPU；
- 未 `join` 直接丢弃 `Thread` 只泄漏 pthread 可连接状态，线程本身仍会完成并清理。

## 原子与锁（sync）

```ayanami
import "sync"

a = AtomicInt::new(0)
old = a.fetch_add(1)          // 返回旧值
a.compare_exchange(1, 2)      // 旧值 == 期望值则写入
fence()                       // 全内存栅栏
cpu_relax()                   // 自旋提示
```

`AtomicInt` 提供 `load/store/swap/fetch_add/sub/and/or/xor/compare_exchange`，默认顺序一致（seq_cst），
纯内联汇编实现，无运行时依赖。

`Mutex[T]` 用 futex 实现（非重入）：

```ayanami
m = Mutex::new[int](0)
m.with_lock((ref mut int v) -> void { *v = *v + 1 })
v = m.with_lock((ref mut int v) -> int { return *v })
```

- `with_lock` 加锁 → 回调 → 解锁；`try_with_lock` 不阻塞；
- `lock()` / `unlock()` / `data()` 是显式接口（必须配对）；
- `RwLock`：`with_read` / `with_write`，写者优先；
- `Barrier`（可重复会合）、`Once`（至多执行一次）、`Condvar`（条件变量，
  约定 `while !pred { cv.wait(pm) }` 循环重查）。

## `#[sync]` 同步块

手工配对 `lock`/`unlock` 在 `return` / `break` / `continue` 路径上很容易漏解锁。
`#[sync]` 把临界区变成一个块，编译器在所有出口自动补 `unlock`：

```ayanami
#[sync(m)]
{
    d = m.data()
    if d > 0 { return d }   // 提前返回 → 先 unlock 再返回
}
```

- 进入块调对象的 `lock()`，离开调 `unlock()`；`return` / `break` / `continue` 都会正确解锁；
- `#[sync(m, guard = d)]` 在锁内绑定载荷（`d: ref mut T`，等价于 `d = m.data()`）；
- 只允许用于块语句；非重入（同一对象嵌套会死锁）；`#[async]` 内禁止跨 `await` 持锁。

## 通道（channel）

```ayanami
import "channel"
import "thread"

c = Channel::new[int]()
c.send(42)
x = c.recv()              // 42
```

- `Channel::new[T]()` 无界；`new_bounded[T](cap)` 有界（满时 `send` 阻塞，形成背压）；
- 多生产者单消费者（mpsc），FIFO；`recv` 阻塞，`try_recv` 非阻塞；
- `close()` 后已缓冲数据仍可取；取空后 `recv_opt()` 返回 `None`（`recv` 会 panic）。

## 异步（async）

`#[async]` 把函数降级为**普通枚举状态机**（实现 `Future[T]`），`await` 在挂起点让出：

```ayanami
#[async]
fn twice(int x) -> int {
    y = await inc(x)
    return y + 1
}
```

编译器只认一个最小协议（当前由程序自己定义，std 侧后续提供）：

```ayanami
enum Poll[T] {
    Ready(T),
    Pending,
}

interface Future[T] {
    fn poll(ref mut self, ref Context cx) -> Poll[T];
}

struct Waker { int id }
struct Context { Waker waker }
```

执行器就是一个轮询循环（可完全由用户实现）：

```ayanami
#[state]
fn block_on[F: Future[int]](ref mut F f) -> int {
    w = Waker { id = 1 }
    cx = Context { waker = w }
    while true {
        p = f.poll(ref cx)
        match p {
            Ready(v) => { return v }
            Pending => { }
        }
    }
}
```

- `#[async]` 支持自由函数、方法与泛型；返回类型写业务类型 `T`；
- `await expr` 是前缀表达式，只能在 async 上下文使用；支持连续多个挂起点，载荷可为
  `int` / `float` / `bool` / `String`；
- 状态机是普通值：可移动、装箱、存字段、传参；实现上复用枚举布局（tag + 变体载荷）；
- 跨 `await` 不持有 `ref`（自引用需要固定地址，当前不支持）；
- 取消 = drop 状态机；执行器不绑定编译器，可替换（后续可经 `env::runtime` 注入）。

完整设计见主仓 `docs/async.md` 与 `docs/sync-blocks.md`。

## 小结

- 线程：`Thread::spawn/join`；
- 同步：`AtomicInt`、`Mutex`、`RwLock`、`Barrier`、`Once`、`Condvar`，以及自动解锁的 `#[sync]` 块；
- 通道：`Channel`（无界 / 有界背压）；
- 异步：`#[async]` + `await`，执行器在协议之外。

下一章是全书收官：用递归下降解析器写一个表达式计算器。
