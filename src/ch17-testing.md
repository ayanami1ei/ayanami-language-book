# 第 17 章 测试

标准库自带一个轻量测试框架：用标注声明用例、用宏做断言，`scripts/test.sh` 统一构建、安装并运行。
这套约定同样可以照搬到你自己的项目。

## 写一个用例

`import "test"` 提供 `#[test]`、`#[should_panic]` 标注与一组 `#assert*` 断言宏：

```ayanami
import "test"

#[test]
fn test_add() {
    #assert(1 + 1 == 2)
    #assert_eq(2 + 2, 4)
    #assert_ne(1, 2)
}

#[should_panic]
fn test_oob() {
    a = ArrayList::new[int]()
    x = a.index(5)      // 期望 panic（退出码 101）
}
```

- 一个函数只写一个标注：`#[test]`（应通过）或 `#[should_panic]`（应 panic）；
- 用例函数写成 void 形式；断言失败即 panic，并打印 `文件:行:列` 与消息；
- `should_panic` 用例只要以退出码 101 结束即通过。

可用断言（失败消息包含源码文本与调用点）：

| 宏 | 说明 |
|---|---|
| `#assert(cond)` | `cond` 为假即失败 |
| `#assert_eq(a, b)` / `#assert_ne(a, b)` | 相等 / 不等 |
| `#assert_lt(a, b)` / `#assert_le` / `#assert_gt` / `#assert_ge` | 比较 |
| `#assert_contains(s, sub)` | 字符串包含 |
| `#assert_some(opt)` / `#assert_none(opt)` | Option 期望（消费该值一次） |
| `#assert_close(a, b, eps)` | 浮点近似：`\|a-b\| <= eps` |
| `#fail(msg)` | 无条件失败 |

复杂条件可以先赋值到局部变量，再 `#assert`。

## 运行

标准库的 `scripts/test.sh` 会先重建并安装 `.lcl`，再运行全部测试：

```bash
AYANAMI_BIN=<主仓>/target/debug/ayanami ./scripts/test.sh
./scripts/test.sh --no-install        # 跳过重建
./scripts/test.sh --filter string     # 只跑名字含 string 的用例
```

输出示例：

```text
running 17 unit tests
  ok   string_test::test_basic
  ...
running 1 compile-fail tests
  ok   compile_fail/missing_import.aya
test result: ok. 18/18 passed
```

运行器为每个用例生成最小 driver 并**独立进程**执行（panic 隔离）：退出码 `0` 通过，
`should_panic` 期望 `101`。

## 三类测试

| 路径 | 说明 |
|---|---|
| `tests/unit/*_test.aya` | 单元用例（`#[test]` / `#[should_panic]`） |
| `tests/unit/<模块>.stdin` | 为该模块用例提供标准输入（每个用例独立进程，从头读取） |
| `tests/compile_fail/*.aya` + `.expected` | 编译期负例：`.expected` 每行一个候选子串，命中任意一行即通过 |
| `tests/golden/*.aya` + `.out` | stdout 黄金输出（精确对比） |

编译期负例示例：

```ayanami
// tests/compile_fail/missing_import.aya
import "no_such_module_xyz"

fn main() -> int {
    return 0
}
```

```text
# tests/compile_fail/missing_import.expected
cannot resolve import 'no_such_module_xyz'
```

## 语言级回归

编译器主仓另有 `./scripts/regression.sh`（`example/*.aya` 正例 + `tests/compile_fail/` 负例）
与 `./scripts/ir_snapshot.sh`（ast/hir/mir/lir 快照），见第 16 章。

## 已知限制

- 带标注用例中，断言报出的行列可能偏移（宏卫生性问题）；以文件与消息为准；
- 用例文件只 `import "test"` 加直接使用的模块，避免重复导入触发重复符号链接错误；
- 语句级 `#[should_panic]` 目前没有运行时语义（panic 即退出 101）。

## 小结

- `#[test]` / `#[should_panic]` 加 `#assert*` 宏写用例；
- `scripts/test.sh` 跑单元、编译期负例与黄金输出；
- 细节见
  [Ayanami-std `docs/dev/testing.md`](https://github.com/ayanami1ei/Ayanami-std/blob/main/docs/dev/testing.md)。

下一章是全书收官：用递归下降解析器写一个表达式计算器。
