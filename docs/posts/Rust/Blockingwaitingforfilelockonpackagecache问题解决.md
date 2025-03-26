---
description: Blocking waiting for file lock on package cache 问题的解决方案
tags:
    - rust
---

# cargo build 报错的解决方案

如果你同时打开了 VSCODE 和命令行，然后修改了 `Cargo.toml`，此时 VSCODE 的 `rust-analyzer` 插件会自动检测到依赖的变更，去下载新的依赖。

在 VSCODE 下载的过程中（特别是更新索引，可能会耗时很久），假如你又在命令行中运行类似 `cargo run` 或者 `cargo build` 的命令，就会出现：
``` text
$ cargo build
Blocking waiting for file lock on package cache
Blocking waiting for file lock on package cache
```

其实这个报错就是因为 VSCODE 的下载太慢了，而且该下载构建还锁住了当前的项目，导致你无法在另一个地方再次进行构建。

解决办法也很简单：
- 耐心等待持有锁的用户构建完成
- 强行停止正在构建的进程，例如杀掉 IDE 使用的 `rust-analyzer` 插件进程，然后删除 ` $HOME/.cargo/.package_cache ` 目录

