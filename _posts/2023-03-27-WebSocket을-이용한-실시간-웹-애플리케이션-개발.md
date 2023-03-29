---
layout: post
title: "WebSocket을 이용한 실시간 웹 애플리케이션 개발"
categories: Node.js
tags: Node.js WebSocket
---

![https://user-images.githubusercontent.com/108377235/232225371-67c6eeb9-0e48-4ba2-91f7-3bad415fd887.png](https://user-images.githubusercontent.com/108377235/232225371-67c6eeb9-0e48-4ba2-91f7-3bad415fd887.png)

<br/>

## Intro

`WebSocket`은 HTTP와 달리 서버와 클라이언트 간 양방향 통신을 지원하는 프로토콜입니다.

이를 이용하면 실시간으로 데이터를 주고받는 웹 애플리케이션을 쉽게 개발할 수 있습니다.

`Node.js`는 비동기 I/O 처리를 지원하므로 `WebSocket`과 잘 어울립니다.

이번 게시물에서는 `Node.js`와 `WebSocket`을 이용하여 간단한 실시간 채팅 애플리케이션을 만드는 방법과 예제 코드를 작성해보겠습니다.

<br/>

## WebSocket?

WebSocket은 서버와 클라이언트 간 양방향 통신을 지원하는 프로토콜입니다.

HTTP와는 달리 연결을 유지하며, 서버와 클라이언트가 데이터를 주고받을 수 있습니다.

이를 이용하면 실시간으로 데이터를 주고받는 웹 애플리케이션을 쉽게 개발할 수 있습니다.

<br/>

## Using WebSocket in Node.js

Node.js에서 WebSocket을 사용하기 위해서는 **`ws`** 모듈을 설치해야 합니다.

```bash
npm install ws
```

<br/>

WebSocket 서버를 생성하려면 다음과 같이 **`WebSocket.Server`** 클래스를 이용합니다.

```jsx
const WebSocket = require("ws");

const wss = new WebSocket.Server({ port: 8080 });
```

이제 클라이언트와 연결이 이루어졌을 때의 이벤트를 처리할 수 있습니다.

<br/>

```jsx
wss.on("connection", function connection(ws) {
  console.log("client connected");
});
```

이제 클라이언트와 연결이 이루어졌을 때 **`client connected`** 메시지가 출력됩니다.

<br/>

이제 클라이언트가 보낸 메시지를 수신하고, 다른 클라이언트에게 전송하는 기능을 구현해보겠습니다.

```jsx
wss.on("connection", function connection(ws) {
  console.log("client connected");

  ws.on("message", function incoming(message) {
    console.log("received: %s", message);

    // 다른 클라이언트에게 메시지 전송
    wss.clients.forEach(function each(client) {
      if (client !== ws && client.readyState === WebSocket.OPEN) {
        client.send(message);
      }
    });
  });
});
```

위 코드에서 **`ws.on('message')`**는 클라이언트로부터 메시지를 수신할 때 호출됩니다.

메시지를 수신하면 해당 메시지를 다른 클라이언트에게 전송합니다.

**`wss.clients.forEach()`**를 이용하여 WebSocket 서버에 연결된 모든 클라이언트에게 메시지를 전송합니다.

<br/>

이제 클라이언트 측에서도 WebSocket을 이용하여 서버와 연결할 수 있습니다.

```jsx
const ws = new WebSocket("ws://localhost:8080");

ws.onopen = function () {
  console.log("connected to server");
};

ws.onmessage = function (event) {
  console.log("received: " + event.data);
};

ws.send("hello");
```

**`WebSocket()`** 생성자를 이용하여 서버에 연결합니다.

연결이 성공하면 **`onopen`** 이벤트가 호출됩니다.

**`onmessage`** 이벤트는 서버로부터 메시지를 수신할 때 호출됩니다.

**`send()`** 메서드를 이용하여 서버에 메시지를 전송합니다.

<br/>

이제 Node.js와 WebSocket을 이용하여 간단한 실시간 채팅 애플리케이션을 만들어보겠습니다.

<br/>

## Create a real-time chat application

Express를 이용하여 HTTP 서버를 구현하고, WebSocket을 이용하여 실시간 채팅 애플리케이션을 구현해보겠습니다.

먼저 Express 애플리케이션을 생성합니다.

```bash
npm install express
```

```jsx
const express = require("express");
const app = express();

app.get("/", function (req, res) {
  res.sendFile(__dirname + "/index.html");
});

app.listen(3000, function () {
  console.log("listening on port 3000");
});
```

위 코드에서는 Express 애플리케이션을 생성하고, 루트 URL('/')에 접속했을 때 **`index.html`** 파일을 반환하도록 합니다.

<br/>

이제 **`index.html`** 파일에서 WebSocket을 이용하여 서버와 연결하고, 채팅 기능을 구현합니다.

```html
<!DOCTYPE html>
<html>
  <head>
    <title>WebSocket Chat Example</title>
  </head>
  <body>
    <input type="text" id="input" />
    <button id="send">Send</button>
    <ul id="messages"></ul>

    <script>
      const socket = new WebSocket("ws://localhost:8080");

      socket.onopen = function () {
        console.log("connected to server");
      };

      socket.onmessage = function (event) {
        const messages = document.getElementById("messages");
        const li = document.createElement("li");
        li.appendChild(document.createTextNode(event.data));
        messages.appendChild(li);
      };

      document.getElementById("send").addEventListener("click", function () {
        const input = document.getElementById("input");
        socket.send(input.value);
        input.value = "";
      });
    </script>
  </body>
</html>
```

위 코드에서는 **`WebSocket()`** 생성자를 이용하여 서버와 연결합니다.

**`onopen`** 이벤트에서는 서버에 연결되었다는 메시지를 출력합니다.

**`onmessage`** 이벤트에서는 서버로부터 메시지를 수신하면 **`ul`** 요소에 해당 메시지를 추가합니다.

**`click`** 이벤트에서는 **`input`** 요소에 입력된 값을 서버로 전송하고, **`input`** 요소를 초기화합니다.

<br/>

이제 WebSocket을 이용하여 클라이언트와 서버가 실시간으로 통신할 수 있습니다.

서버에서는 WebSocket을 이용하여 모든 클라이언트에게 메시지를 전송하면 되며, 클라이언트에서는 WebSocket을 이용하여 서버로부터 메시지를 수신하고, 채팅창에 해당 메시지를 출력하면 됩니다.

<br/>

```jsx
const WebSocket = require("ws");
const wss = new WebSocket.Server({ port: 8080 });

wss.on("connection", function (ws) {
  console.log("client connected");

  ws.on("message", function (message) {
    console.log("received: %s", message);

    wss.clients.forEach(function (client) {
      if (client !== ws && client.readyState === WebSocket.OPEN) {
        client.send(message);
      }
    });
  });
});
```

위 코드에서는 **`WebSocket.Server()`** 생성자를 이용하여 WebSocket 서버를 생성합니다.

**`on('connection')`** 이벤트에서는 클라이언트와 연결되었다는 메시지를 출력합니다.

**`on('message')`** 이벤트에서는 클라이언트로부터 메시지를 수신하면 해당 메시지를 모든 클라이언트에게 전송합니다.

<br/>

---

지금까지 `Node.js`와 `WebSocket`을 이용하여 실시간 채팅 애플리케이션을 구현하는 방법을 알아보았습니다.

`WebSocket`을 이용하면 클라이언트와 서버 간에 실시간으로 데이터를 주고받을 수 있으며, 이를 이용하여 다양한 실시간 애플리케이션을 구현할 수 있습니다.

다음 포스팅에서는 `socket.io`에 대한 내용을 다뤄보도록 하겠습니다.
