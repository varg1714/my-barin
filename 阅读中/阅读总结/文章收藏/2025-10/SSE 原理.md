---
source: "[[希音面试：SSE 底层原理是什么？快 20 年了， 为何 突然 爆火？]]"
create: 2025-10-09
---

## 1. SSE (Server-Sent Events) 学习笔记

### 1.1. 一、核心概念：什么是 SSE？

**Server-Sent Events (SSE)** 是一种允许服务器向客户端单向推送事件（数据）的 Web 技术。它本质上是建立在单个持久化的 
**HTTP 连接**之上，服务器通过这个连接可以持续地将数据流式传输到客户端。

**核心特征**：

* **单向通信**：数据流只能从 **服务器 -> 客户端**。
* **基于 HTTP**：它不是一个新协议，而是对 HTTP 协议的一种应用和扩展，因此兼容性好，易于实现。
* **文本协议**：传输的数据流遵循 `text/event-stream` 格式的文本规范。
* **自动重连**：浏览器原生的 `EventSource` API 内置了标准的断线自动重连机制。

SSE 非常适合那些只需要从服务器接收实时更新的场景，例如：新闻源更新、股票价格推送、状态监控、日志显示等。

### 1.2. 二、工作原理与协议细节

整个 SSE 的生命周期遵循标准的 HTTP 请求-响应模型，但响应是持续的。

1. **客户端发起连接**：
    * 客户端通过 JavaScript 创建一个 `EventSource` 对象，指向服务器的一个特定 URL。
    * 浏览器会向该 URL 发起一个标准的 HTTP **GET** 请求。
    * 请求头中会自动包含 `Accept: text/event-stream`，表明客户端期望接收事件流。

2. **服务端响应与连接保持**：
    * 服务器收到请求后，必须发送特定的响应头来建立 SSE 连接：
        * `Content-Type: text/event-stream`：告知浏览器这是一个 SSE 流。
        * `Connection: keep-alive`：指示保持长连接。
        * `Cache-Control: no-cache`：防止任何中间代理缓存响应。
    * 发送完头部后，服务器**不关闭**这个连接，而是将其保持，以便后续随时推送数据。

3. **事件流格式 (Event Stream Format)**：
    服务器推送的每一条消息都由一个或多个 `字段: 值` 格式的文本行组成，并以**两个换行符 (`\n\n`)** 作为消息的结束标记。

    * `data: <message>`
        * **必需字段**。定义了消息的数据内容。如果数据是 JSON 格式，需要先将其序列化为字符串。一条消息可以包含多个 `data` 行，客户端会自动将它们拼接起来。
    * `id: <message-id>`
        * **可选字段**。为事件设置一个唯一的 ID。这个 ID 会在客户端断线重连时，通过 `Last-Event-ID` HTTP 头自动发送回服务器，是实现消息补发的关键。
    * `event: <event-name>`
        * **可选字段**。为事件指定一个自定义名称。如果未指定，事件默认为 `message` 类型。客户端需要使用 `addEventListener('<event-name>', ...)` 来监听这种自定义事件。
    * `retry: <milliseconds>`
        * **可选字段**。告知客户端在连接断开后，应等待多少毫秒再尝试重连。这会覆盖浏览器的默认重连间隔。

    **示例消息：**

    ```
    id: 1
    event: user-update
    data: {"username": "Alice", "status": "online"}
    
    id: 2
    data: 这是一条没有事件名的默认消息
    data: 这是第二行数据
    
    ```

### 1.3. 三、客户端实现 (`EventSource` API)

浏览器提供了非常简洁的原生 API 来处理 SSE。

```javascript
// 1. 创建 EventSource 实例，指向服务器端点
const sse = new EventSource('/events');

// 2. 监听连接成功事件
sse.onopen = () => {
  console.log('SSE connection established.');
};

// 3. 监听默认的 "message" 事件
sse.onmessage = (event) => {
  // event.data 是服务器发送的原始字符串数据
  console.log('Received default message:', event.data);
  
  // 如果是 JSON 字符串，需要解析
  const parsedData = JSON.parse(event.data);
  console.log('Parsed data:', parsedData);
};

// 4. 监听自定义名称的事件 ("user-update")
sse.addEventListener('user-update', (event) => {
  console.log('Received custom event [user-update]:', event.data);
});

// 5. 监听错误事件（如连接中断）
sse.onerror = (error) => {
  console.error('EventSource failed:', error);
  // 注意：浏览器会自动尝试重连，除非服务器返回 204 或发生致命错误
};

// 6. 关闭连接
// sse.close();
```

### 1.4. 四、服务端实现 (Node.js/Express 示例)

服务端的关键是**持有客户端的响应对象 (`res`)**，并持续向其写入数据。

```javascript
const express = require('express');
const app = express();

// 用于存储所有连接的客户端
let clients = [];

app.get('/events', (req, res) => {
  // 1. 设置 SSE 必需的响应头
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Connection', 'keep-alive');
  res.setHeader('Cache-Control', 'no-cache');
  res.flushHeaders(); // 立即发送头部

  // 2. 将客户端响应对象存储起来
  const clientId = Date.now();
  const newClient = {
    id: clientId,
    res, // res 对象就是通往客户端的数据通道
  };
  clients.push(newClient);
  console.log(`Client ${clientId} connected.`);

  // 3. 监听客户端断开连接事件，进行清理
  req.on('close', () => {
    clients = clients.filter(client => client.id !== clientId);
    console.log(`Client ${clientId} disconnected.`);
  });
});

// 示例：每隔2秒向所有连接的客户端发送一条消息
setInterval(() => {
  const data = { message: 'Server time is ' + new Date().toLocaleTimeString() };
  const message = `id: ${Date.now()}\ndata: ${JSON.stringify(data)}\n\n`;
  
  clients.forEach(client => client.res.write(message));
}, 2000);

app.listen(3000, () => {
  console.log('SSE server listening on port 3000');
});
```

### 1.5. 五、深入探讨与工程实践

#### 1.5.1. 数据类型：SSE 只能传输文本吗？

协议本身 (`text/event-stream`) 是基于文本的。但可以通过**序列化**来传输复杂数据。

* **结构化数据**：最常见的方式是使用 `JSON.stringify()` 将对象转换为 JSON 字符串发送，客户端再通过 `JSON.parse()` 解析。
* **二进制数据**：理论上可以通过 **Base64** 编码将二进制数据（如小图片）转为文本发送，但这会增加数据体积和编解码开销，通常不推荐。

#### 1.5.2. 单向通信：为什么不利用 TCP 实现双向？

> **类比：SSE 像广播电台，WebSocket 像对讲机。**

虽然底层 TCP 是双工的，但 SSE 构建在 **HTTP 应用层**之上。它遵循 HTTP 请求-响应模型，只是将“响应”变成了一个无限流。在这个模型中，没有为客户端定义一个在响应流中“插话”的标准。

**WebSocket** 则不同，它通过一个 HTTP 请求进行**协议升级**，成功后就完全抛弃 HTTP，切换到一个全新的、为双向通信设计的 `ws://` 协议。

因此，SSE 的单向性是其**设计哲学**的体现：为仅需服务器推送的场景提供一个比 WebSocket 更轻量、更简单的方案。

#### 1.5.3. 可靠的重连机制 (`Last-Event-ID`)

这是 SSE 的一个强大特性，但需要**服务端与客户端协同**完成。

* **客户端 (自动)**：当连接意外中断，`EventSource` 会自动尝试重连。在发起重连请求时，它会自动将最后收到的事件 ID 放入 HTTP 请求头 `Last-Event-ID` 中。
* **服务端 (必须实现)**：
    1. 您的后端代码**必须**读取这个 `Last-Event-ID` 请求头。
    2. **必须**有一个持久化或缓存机制（如 Redis、数据库）来存储历史消息。**如果中断期间产生的数据只在内存中，它们将会丢失。**
    3. 根据 `Last-Event-ID`，从存储中查询所有“错过的”消息。
    4. 将这些错过的消息**补发**给客户端，然后再恢复实时推送。

#### 1.5.4. 重连时的数据交叉问题

在服务端补发旧数据期间，如果又有新数据产生，直接推送会导致客户端数据流顺序错乱。必须通过程序逻辑避免。

**推荐流程：**
1. **标记状态**：为重连的客户端连接设置一个 `recovering` (恢复中) 状态。
2. **补发旧数据**：从持久化存储中查询并发送历史消息。
3. **缓存新数据**：在此期间，若有新消息产生，检查客户端状态。如果是 `recovering`，则将新消息存入该连接的**临时队列**，而不是立即发送。
4. **切换状态**：旧数据补发完毕后，先发送临时队列中的所有消息，然后将客户端状态改回 `live` (实时)。
5. **恢复正常**：此后，新消息可以直接实时发送。

#### 1.5.5. 多用户与集群环境下的挑战

示例代码中的全局 `clients` 数组在真实项目中是不可行的。

* **区分用户**：
    * 通过 **JWT (Authorization Header)** 或 **Cookie** 在 SSE 连接请求中进行身份验证。
    * 使用 `Map` 或字典结构，以 `userId` 为键，存储每个用户的 `res` 响应对象，实现定向推送。
    `const userConnections = new Map<userId, response>();`

* **集群部署**：
    如果服务有多个实例，用户 A 可能连在服务器 1，但业务逻辑在服务器 2 触发。
    1. **粘性会话 (Sticky Sessions)**：通过负载均衡器将同一用户的所有请求固定路由到同一台服务器。实现简单，但存在单点故障和负载不均的风险。
    2. **消息总线 (Message Bus) - 推荐方案**：
        * 使用 **Redis Pub/Sub** 等中间件。
        * 服务器实例订阅其连接用户的专属频道（如 `user-channel:<userId>`）。
        * 任何服务器实例想给某用户发消息时，只需向该用户的 Redis 频道**发布 (Publish)** 消息。
        * 持有该用户连接的服务器实例会从 Redis 收到消息，再通过 SSE 连接推送给最终的客户端。
        * 此方案完美解耦，扩展性与可用性都非常高。

### 1.6. 六、SSE vs. WebSocket

| 特性       | Server-Sent Events (SSE) | WebSocket                 |
| :------- | :----------------------- | :------------------------ |
| **通信方向** | 单向（服务端 -> 客户端）           | 双向                        |
| **协议**   | 标准 HTTP/1.1              | 独立的 WebSocket 协议 (ws/wss) |
| **自动重连** | **浏览器原生支持**              | 需手动实现                     |
| **数据格式** | 纯文本（常用 JSON 字符串）         | 文本或二进制                    |
| **适用场景** | 状态更新、通知、新闻源              | 实时聊天、在线游戏、协同编辑            |
| **复杂度**  | 低                        | 较高                        |
| **兼容性**  | 好，可复用现有 HTTP 基础设施        | 可能需要网络设备额外配置              |
