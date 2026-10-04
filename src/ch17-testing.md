# 第 17 章 测试

Ayanami 没有内建测试框架，也没有 `assert` 宏。标准库与主仓库使用的是同一套简单的回归模式：
**用真实编译器运行用例，用退出码与输出子串判定成败**。这套模式同样适用于你自己的项目。

## 三类回归

| 类别 | 判定 | 用例与清单 |
|---|---|---|
| 正例 | 运行退出码与期望一致 | `tests/*.aya` + `positive_exit.txt` |
| 运行时 panic | 退出码 101，且输出包含指定子串 | `tests/panic_exit.txt` |
| 编译期负例 | `check` 输出命中 `.expected` 中任一子串 | `tests/compile_fail/` |

## 写一个正例

约定：每个检查失败返回**不同的非 0 码**，全部通过返回 0；用例不依赖交互输入，尽量短。

```ayanami
import "math"

fn main() -> int {
    if abs(-5) != 5 { return 1 }
    if pow(2, 10) != 1024 { return 2 }
    return 0
}
```

`main` 的返回值就是进程退出码，所以一个用例文件就能自包含地表达“哪一条检查失败了”。

## 测试 panic 与编译错误

越界等运行时错误会以退出码 101 终止并输出消息，可以用“退出码 + 输出子串”来断言：

```ayanami
import "arraylist"

fn main() -> int {
    a = ArrayList::new[int]()
    a.push(1)
    x = a.index(5)      // panic: index out of bounds...
    return 0
}
```

清单文件记录期望（panic 类需要额外一列输出子串）：

```text
# positive_exit.txt
test_arraylist.aya 0

# panic_exit.txt
test_panic_bounds.aya 101 index out of bounds
```

编译期负例写在 `tests/compile_fail/<名>.aya`，配同名 `.expected`：每行一个候选子串，
`check` 输出命中任意一行即通过。例如：

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

## 运行与自动化

标准库子仓的 `scripts/test.sh` 会先用 `build.sh --install` 重建 `.lcl`，再跑完三类用例：

```bash
AYANAMI_BIN=<主仓>/target/debug/ayanami ./scripts/test.sh
./scripts/test.sh --no-install    # 不重建 .lcl
```

输出 `std tests: positive=N panic=N negative=N failures=N`，有失败时退出码非 0，可直接接进 CI。

主仓库另有语言级回归，由第 16 章的 `./scripts/regression.sh`（`example/*.aya` 正例 +
`tests/compile_fail/` 负例）与 `./scripts/ir_snapshot.sh`（各阶段 IR 快照）覆盖。

## 给自己的项目照搬

你不需要任何库：建一个 `tests/` 目录，用上面三类清单记录期望，再写几十行脚本遍历
用例、比对退出码与输出即可。要点：

- 失败码从 1 递增，便于定位是哪条检查失败；
- 需要验证输出内容时，用 panic 类别的子串匹配（正例只看退出码）；
- 用例尽量 ≤ 60 行，不依赖交互输入。

## 小结

- 三类回归：正例退出码、panic 退出码 + 子串、编译期负例子串；
- 标准库完整说明见
  [Ayanami-std 的 `docs/testing.md`](https://github.com/ayanami1ei/Ayanami-std/blob/main/docs/testing.md)。

下一章是全书收官：用递归下降解析器写一个表达式计算器。
