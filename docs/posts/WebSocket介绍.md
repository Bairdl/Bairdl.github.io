---
description: WebSocket API 的基础知识。
tags:
  - web
---

# WebSocket API 介绍

WebSocket API 是一种**可在用户浏览器和服务器之间开启双向交互式通信会话**的技术。  
利用该 API，可以向服务器发送信息，并接受事件驱动的响应，而无需轮询服务器以获得回复。  

## 接口

### `WebSocket`

WebSocket 对象提供了用于创建和管理 WebSocket 连接，以及可以通过该连接发送和接收数据的 API。

#### 构造函数

`WebSocket(url[, protocols])`：返回一个 WebSocket 对象。  

#### 常量

| 常量 | 值 |
| ---- | -- |
| `WebSocket.CONNECTING` | 0 |
| `WebSocket.OPEN` | 1 |
| `WebSocket.CLOSING` | 2 |
| `WebSocket.CLOSED` | 3 |

#### 属性

- `WebSocket.binaryType`：使用二进制的数据类型连接。
- `WebSocket.bufferedAmount` **（只读）**：未发送至服务器的字节数。
- `WebSocket.extensions` **（只读）**：服务器选择的扩展。
- `WebSocket.onclose`：用于指定关闭后的回调函数。
- `WebSocket.onerror`：用于指定连接失败后的回调函数。
- `WebSocket.onmessage`：用于指定当从服务器接受到信息时的回调函数。
- `WebSocket.onopen`：用于指定连接成功后的回调函数。
- `WebSocket.protocal` **（只读）**：服务器选择的下属协议。
- `WebSocket.readyState` **（只读）**：当前的连接状态。
- `WebSocket.url` **（只读）**：WebSocket 的绝对路径。

#### 方法

- `WebSocket.close([code[, reason]])` : 关闭当前链接。
- `WebSocket.send(data)` : 对要传输的数据进行排队。

#### 事件

使用 `addEventListener()` 或将一个事件监听器赋值给本接口的 `oneventname` 属性，来监听下面的事件。

- `close` : 当一个 WebSocket 连接被关闭时触发。 也可以通过 `onclose` 属性来设置。
- `error` : 当一个 WebSocket 连接因错误而关闭时触发，例如无法发送数据时。 也可以通过 `onerror` 属性来设置。
- `message` : 当通过 WebSocket 收到数据时触发。 也可以通过 `onmessage` 属性来设置。
- `open` : 当一个 WebSocket 连接成功时触发。 也可以通过 `onopen` 属性来设置。

#### 示例

``` js
// Create WebSocket connection.
const socket = new WebSocket("ws://localhost:8080");

// Connection opened
socket.addEventListener("open", function (event) {
  socket.send("Hello Server!");
});

// Listen for messages
socket.addEventListener("message", function (event) {
  console.log("Message from server ", event.data);
});
```

### `CloseEvent`

`CloseEvent` 会在连接关闭时发送给使用 WebSockets 的客户端。它在 WebSocket 对象的 `onclose` 事件监听器中使用。

#### 构造器

`CloseEvent()` : 创建一个 `CloseEvent`.

#### 属性

该接口也继承了其父类 Event 的属性。

`CloseEvent.code` **(只读)** : 返回一个 `unsigned short` 类型的数字，表示服务端发送的关闭码。

| 状态码 | 名称 | 描述 |
| ------ | ---- | ---- |
| 0–999  | | 保留段，未使用。|
| 1000 | CLOSE_NORMAL | 正常关闭; 无论为何目的而创建，该链接都已成功完成任务。|
| 1001 | LOSE_GOING_AWAY | 终端离开，可能因为服务端错误，也可能因为浏览器正从打开连接的页面跳转离开。|
| 1002 |CLOSE_PROTOCOL_ERROR | 由于协议错误而中断连接。|
| 1003 |CLOSE_UNSUPPORTED | 由于接收到不允许的数据类型而断开连接 (如仅接收文本数据的终端接收到了二进制数据).|
| 1004 |  |保留。 其意义可能会在未来定义。|
| 1005 |CLOSE_NO_STATUS | 保留。 表示没有收到预期的状态码。|
| 1006 |CLOSE_ABNORMAL | 保留。 用于期望收到状态码时连接非正常关闭 (也就是说，没有发送关闭帧).|
| 1007 |Unsupported Data | 由于收到了格式不符的数据而断开连接 (如文本消息中包含了非 UTF-8 数据).|
|1008 |Policy Violation | 由于收到不符合约定的数据而断开连接。这是一个通用状态码，用于不适合使用 1003 和 1009 状态码的场景。|
|1009 |CLOSE_TOO_LARGE | 由于收到过大的数据帧而断开连接。|
|1010 |Missing Extension | 客户端期望服务器商定一个或多个拓展，但服务器没有处理，因此客户端断开连接。|
|1011 |Internal Error | 客户端由于遇到没有预料的情况阻止其完成请求，因此服务端断开连接。|
|1012 |Service Restart | 服务器由于重启而断开连接。|
|1013 |Try Again Later | 服务器由于临时原因断开连接，如服务器过载因此断开一部分客户端连接。|
|1014 |  | 由 WebSocket 标准保留以便未来使用。 |
|1015 | | TLS Handshake 保留。 表示连接由于无法完成 TLS 握手而关闭 (例如无法验证服务器证书). |
|1016–1999 | | 由 WebSocket 标准保留以便未来使用。|
|2000–2999 | | 由 WebSocket 拓展保留使用。|
|3000–3999 | | 可以由库或框架使用.不应由应用使用。可以在 IANA 注册，先到先得。|
|4000–4999 | | 可以由应用使用。 |

`CloseEvent.reason` **(只读)** : 返回一个 DOMString 用以表示服务器关闭连接的原因。这是由服务器和子协议决定的。

`CloseEvent.wasClean` **(只读)** : 返回一个 Boolean 用以表示连接是否完全关闭。

#### 方法

该接口也继承了其父类 Event 的属性。

`CloseEvent.initCloseEvent()` **非标准 已弃用**
初始化创建的 CloseEvent 的值。如果该事件已经被处理，这个方法什么也不做。不要直接使用这个方法，使用 CloseEvent() ?构造器来代替。

