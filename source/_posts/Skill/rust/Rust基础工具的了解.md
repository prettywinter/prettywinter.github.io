---
title: Rust基础工具的了解
categories: rust
abbrlink: a49987e3
---

Rust 编程语言可帮助您编写更快、更可靠的软件。高级人体工程学和低级控制在编程语言设计中经常存在分歧;Rust 挑战了这场冲突。通过平衡强大的技术能力和出色的开发人员体验，Rust 为您提供了控制低级细节（如内存使用情况）的选项，而无需传统上与此类控制相关的所有麻烦。


<!-- more -->

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=true} -->

## 一、Rust

本文简单介绍一下 rust 的基础工具，系统学习该语言可以去下两个网站：官网和 rust 语言圣经，都是很不错的。

[学习Rust](https://www.rust-lang.org/learn)
[rust语言圣经](https://course.rs/about-book.html)

运行 Rust 之前必须先编译，`rustc 源文件名.rs`;`rustc` 只适合简单的程序~~(比如Hello World)~~。对于项目来说，我们都是用 cargo。

### 1.1 rust的一些命令

```bash
# 查看安装版本
rustc --version
# 更新
rustup update
# 卸载
rustup self uninstall
# 查看本地文档
rustup doc
# 编译
rustc source_name.rs
```

## 二、cargo

```bash
cargo -V
# 构建项目
cargo build
# 构建并运行项目，重复执行，源码没有修改的程序，会直接运行。不会再次编译
cargo run
# 检查代码：确保能通过编译，但不会生成任何可执行文件
cargo check
# 发布构建，会在 target/release 生成可执行文件
cargo build --release
```

> 尽量使用 Cargo

如果项目不是使用 `cargo` 创建的，也可以改变目录结构，使用 cargo 进行项目管理。

1. 把源代码都移动到 `src` 下；
2. 创建 `Cargo.toml` 并填写相应的配置。

- `cargo build`：在 `target/debug` 目录创建可执行文件。第一次执行 cargo build 命令会生成 `Cargo.lock` 文件。该文件负责追踪项目依赖的精确版本，类似于 `package-lock.json`。

- `cargo run`: 构建并且运行项目；如果之前该项目编译过且没有修改内容，再次执行该命令直接运行而无须编译。

- `cargo check`: 检查代码，确保能通过编译，但不产生任何可执行文件。速度比 cargo build 快得多。编写代码时可以周期性使用改命令去检查代码，提高效率。

- `cargo build --release`: 为发布构建，项目上线时可以使用。编译时会进行优化，代码会运行的更快，但是编译时间更长，会在 `target/release` 而不是 `target/debug` 生成可执行文件。
