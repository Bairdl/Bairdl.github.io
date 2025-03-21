---
description: 更换 Rust 下载镜像的方法，以解决下载依赖慢的问题。
tags:
    - rust
---

# Rust 更换下载镜像

## 一、覆盖默认的镜像地址（推荐使用）
直接使用新注册的服务来替代默认的 `crates.io` 。  
在 `$HOME/.cargo/config.toml` 添加以下内容：  
``` toml
[source.crates-io]
replace-with = 'ustc'

[source.ustc]
registry = "git://mirrors.ustc.edu.cn/crates.io-index"
```

首先，创建一个新的镜像源 [source.ustc]，然后将默认的 crates-io 替换成新的镜像源: `replace-with = 'ustc'` 。  

### 字节跳动镜像
``` toml
[source.crates-io]
replace-with = 'rsproxy'

[source.rsproxy]
registry = "https://rsproxy.cn/crates.io-index"

# 稀疏索引，要求 cargo >= 1.68
[source.rsproxy-sparse]
registry = "sparse+https://rsproxy.cn/index/"

[registries.rsproxy]
index = "https://rsproxy.cn/crates.io-index"

[net]
git-fetch-with-cli = true
```

## 二、新增镜像地址
首先是在 `crates.io` 之外添加新的注册服务，在 `$HOME/.cargo/config.toml` （如果文件不存在则手动创建一个）中添加以下内容：  
``` toml
[registries]
ustc = { index = "https://mirrors.ustc.edu.cn/crates.io-index/" }
```

这种方式只会新增一个新的镜像地址，因此在引入依赖的时候，需要指定该地址，例如在项目中引入 `time` 包，你需要在 `Cargo.toml` 中使用以下方式引入:  
``` toml
[dependencies]
time = {  registry = "ustc" }
```

**在重新配置后，初次构建可能要较久的时间**，因为要下载更新 ustc 注册服务的索引文件，由于文件比较大，需要等待较长的时间。

此处有两点需要注意：  
1. `cargo 1.68` 版本开始支持稀疏索引，不再需要完整克隆 crates.io-index 仓库，可以加快获取包的速度，如：
``` toml
[source.ustc]
registry = "sparse+https://mirrors.ustc.edu.cn/crates.io-index/"
```

2. `cargo search` 无法使用镜像
