# Ayanami 语言教程

这是 [Ayanami 编程语言](https://github.com/ayanami1ei/Ayanami-language)的官方教程，
结构参照《Rust 程序设计语言》（Rust 圣经）：从安装和第一个程序开始，逐步讲到
类型系统、所有权、接口、泛型、标准库、标注系统与综合项目。

## 在线阅读

教程使用 [mdBook](https://github.com/rust-lang/mdBook) 构建：

```bash
# 安装 mdBook（任选其一）
cargo install mdbook
# 或从 GitHub Releases 下载预编译二进制

# 本地构建
mdbook build          # 输出到 book/
mdbook serve --open   # 本地预览
```

## 目录

见 [`src/SUMMARY.md`](src/SUMMARY.md)。

## 贡献

勘误与改进欢迎提 Issue 或 PR。示例代码以主仓库 `example/` 下的用例为准。

## 许可

MIT，与主仓库一致。
