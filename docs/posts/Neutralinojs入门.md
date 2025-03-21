---
description: Neutralinojs 跟随官网进行第一个项目。
tags:
    - web
    - 桌面应用
---

---

2025年03月20日 补充
不好用，相关的多个库显示已被弃用。  
此外，示例项目无法正常运行，创建时显示有内部错误。

---

## 介绍

### Neutralinojs 是什么？
Neutralinojs是一个轻量级可移植的桌面应用程序开发框架。它可以让你使用JavaScript, HTML和CSS开发轻量的跨平台桌面应用。 您可以使用任何编程语言（通过扩展IPC）扩展Neutranojs，并将NeutranoJS用作任何代码的一部分（通过子进程IPC）。

### 为什么使用 Neutralinojs？
在Electron和NWjs中，你必须安装Node.js和成百上千的依赖库。内嵌的Chromium和Node使简单的应用也变的很臃肿。   
Neutralizojs提供了一个轻量级和可移植的SDK，它是Electron和NW.js的替代品。 Neutralizojs不捆绑Chromium，而是在操作系统中使用现有的web浏览器库（例如在Linux中使用gtk-webkit2）。   
Neutralizojs为本机操作实现了WebSocket连接，并嵌入了一个静态web服务器来提供web内容。 此外，它还为开发人员提供了一个内置的JavaScript客户端库。

### 适用场景
- 跨平台的桌面应用开发
- 使用本机操作构建web应用程序。
- 用作云消息代理。
- 通过用作IPC代理连接多个进程。

## 第一个应用
将使用 `Neutralinojs` 创建一个简单的应用程序。  
此示例应用程序将显示 Hello和您的操作系统的当前用户名。例如，如果当前用户名为John，则会显示Hello John。

### 第 0 步：安装 neu CLI
您需要安装neu CLI来创建Neutralinojs应用程序。您也可以在没有CLI的情况下开发应用程序。但是，CLI会使一些困难的事情变得容易。在终端上运行以下命令，全局安装neu CLI。

``` shell
npm install -g @neutralinojs/neu
```

如果你不想全局安装，可以通过npx使用neu CLI。

``` shell
npx @neutralinojs/neu <command>
```

### 第 1 步：创建一个新的app
输入以下命令创建一个新的应用程序。

``` shell
neu create myapp
```

上面的命令将在myapp目录中创建一个新的应用程序。确保一切正常后，请运行您的应用程序。

``` shell
cd myapp
neu run
```

### 第 2 步：编写 JS、HTML 和 CSS
在项目源文件中插入下面的代码。

index.html
``` html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>NeutralinoJs</title>
    <link rel="stylesheet" href="styles.css">
  </head>
  <body>
    <div id="neutralinoapp">
      <h1 id="name"></h1>
    </div>
    <script src="js/neutralino.js"></script>
    <script src="js/main.js"></script>
  </body>
</html>
```

CSS
``` css
html, body{
    margin: 0px;
    padding: 0px;
}

#neutralinoapp {
    position: absolute;
    width: 100%;
    height: 100%;
    background: #FFD700;
}

#neutralinoapp h1 {
    position: relative;
    float: left;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    font-family: Arial;
    font-size: 48px;
    padding: 0px;
    margin: 0px;
}
```

main.js

``` js
let getUsername = async () => {
  const key = NL_OS == 'Windows' ? 'USERNAME' : 'USER';
  let value = '';
  try {
    value = await Neutralino.os.getEnv(key);
  }
  catch(err) {
    console.error(err);
  }
  document.getElementById('name').innerText = `Hello ${value}`;
}

Neutralino.init();
getUsername();
```

与Node.js环境不同，Neutralenojs阻止任意的本机API函数执行。 在这里，我们使用了os.getEnv本机函数，因此请检查您的应用程序配置是否允许应用程序前端执行它。 默认配置已经允许整个os命名空间，如下nativeAllowList权限设置：
``` text
"nativeAllowList": [
    "app.*",
    "os.*",
    "debug.log"
],
```

我们不需要更新权限设置中的任何内容，因为它已经允许os.getEnv本机函数调用。

### 第 3 步：运行应用
可以使用run命令启动应用程序。
``` shell
neu run
```

### 第 4 步：构建应用
要为应用程序制作可移植的二进制文件，我们可以使用build命令。在您的终端上输入以下命令，为所有支持的平台制作最终的二进制文件。
``` sh
neu build --release
```
上面的命令将在dist目录中生成所有平台的二进制文件，并且--release标志还将把二进制文件打包到.zip文件中。这个命令将很快完成，因为不需要编译。