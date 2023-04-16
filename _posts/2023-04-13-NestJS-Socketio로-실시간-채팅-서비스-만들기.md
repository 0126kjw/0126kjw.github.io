---
layout: post
title: "[NestJS] Socket.io로 실시간 채팅 서비스 만들기"
categories: NestJS
tags: NestJS Socket.io
---

![https://user-images.githubusercontent.com/108377235/232318304-12b0d128-602b-4563-b337-de26c1f23aa4.png](https://user-images.githubusercontent.com/108377235/232318304-12b0d128-602b-4563-b337-de26c1f23aa4.png)

<br/>

## Start

실시간 채팅 서비스는 요즘 매우 인기 있는 기능 중 하나입니다.

이번에는 `NestJS`와 `Socket.io`를 사용하여 아주 간단한 실시간 채팅 서비스를 만드는 방법을 알아보겠습니다.

<br/>

### NestJS 애플리케이션 생성

먼저, `NestJS` 애플리케이션을 생성합니다. 터미널에서 다음과 같이 입력합니다.

```bash
$ nest new chat-app
```

이렇게 하면 새로운 NestJS 프로젝트가 생성됩니다.

<br/>

### Socket.io 설치

다음으로, `Socket.io`를 설치합니다. 터미널에서 다음과 같이 입력합니다.

```bash
$ npm install --save @nestjs/platform-socket.io socket.io
```

이렇게 하면 Socket.io와 NestJS용 Socket.io 모듈이 설치됩니다.

<br/>

### Socket.io 모듈 설정

Socket.io 모듈을 NestJS 애플리케이션에 설정해야 합니다.

`app.module.ts` 파일에서 다음과 같이 추가합니다.

```tsx
import { Module } from "@nestjs/common";
import { ChatModule } from "./chat/chat.module";

@Module({
  imports: [ChatModule],
  controllers: [],
  providers: [],
})
export class AppModule {}
```

`AppController`와 `AppService`는 사용하지 않기 때문에 파일을 삭제하고 코드에서도 지워주시면 됩니다.

<br/>

### Socket.io 모듈 생성

다음으로, `Socket.io` 모듈을 생성합니다. 터미널에서 다음과 같이 입력합니다.

```bash
$ nest g module chat
```

이렇게 하면 `chat`이라는 이름의 새로운 모듈이 생성됩니다.

<br/>

그리고 다음과 같이 **채팅 서비스**를 생성합니다.

```bash
$ nest generate service chat
```

그리고 `app.module.ts` 파일에서 `ChatModule`과 `ChatService`를 가져와서 추가합니다.

또한 `@nestjs/platform-socket.io` 에서 `IoAdapter`를 가져와 추가합니다.

```tsx
import { Module } from "@nestjs/common";
import { ChatGateway } from "./chat.gateway";
import { ChatService } from "./chat.service";
import { IoAdapter } from "@nestjs/platform-socket.io";

@Module({
  providers: [ChatGateway, ChatService, IoAdapter],
})
export class SocketModule {}
```

<br/>

### Socket.io 게이트웨이 생성

다음으로, Socket.io 게이트웨이를 생성합니다. 터미널에서 다음과 같이 입력합니다.

```bash
$ nest g gateway chat
```

이렇게 하면 `socket`이라는 이름의 새로운 게이트웨이가 생성됩니다.

`ChatGateway`는 Socket.io 이벤트 처리기를 포함합니다.

<br/>

### Socket.io 이벤트 처리기 생성

`ChatGateway`에서 Socket.io 이벤트 처리기를 만듭니다. 다음과 같이 작성합니다.

```tsx
import {
  SubscribeMessage,
  WebSocketGateway,
  WebSocketServer,
  OnGatewayConnection,
  OnGatewayDisconnect,
} from "@nestjs/websockets";
import { Socket, Server } from "socket.io";
import { ChatService } from "./chat.service";

@WebSocketGateway()
export class ChatGateway {
  constructor(private readonly chatService: ChatService) {}

  @WebSocketServer() server: Server;

  @SubscribeMessage("chat")
  handleMessage(client: Socket, payload: any): void {
    this.chatService.addMessage(payload);
    this.server.emit("chat", payload);
  }

  @SubscribeMessage("getMessages")
  handleGetMessages(client: Socket): void {
    const messages = this.chatService.getMessages();
    client.emit("messages", messages);
  }

  handleConnection(client: Socket, ...args: any[]): any {
    console.log(`Client connected: ${client.id}`);
  }

  handleDisconnect(client: Socket): any {
    console.log(`Client disconnected: ${client.id}`);
  }
}
```

위 코드에서 `@WebSocketGateway()` 데코레이터를 사용하여 ChatGateway 클래스를 WebSocket 게이트웨이로 정의합니다.

`@WebSocketServer()` 데코레이터를 사용하여 Socket.io 서버 인스턴스를 가져올 수 있습니다.

그런 다음, `@SubscribeMessage()` 데코레이터를 사용하여 클라이언트로부터 수신한 'chat' 및 'getMessages' 이벤트를 처리합니다.

`'chat'` 이벤트를 수신하면 ChatService에서 메시지를 추가하고 모든 클라이언트에게 'chat' 이벤트를 발행합니다.

`'getMessages'` 이벤트를 수신하면 ChatService에서 모든 메시지를 가져와 클라이언트에게 'messages' 이벤트를 발행합니다.

<br/>

### 채팅 서비스 작성

마지막으로, `ChatService` 클래스를 작성합니다.

`ChatService`는 채팅 메시지를 저장하고 반환합니다. 다음과 같이 작성합니다.

```tsx
import { Injectable } from "@nestjs/common";

@Injectable()
export class ChatService {
  private messages: string[] = [];

  addMessage(message: string): void {
    this.messages.push(message);
  }

  getMessages(): string[] {
    return this.messages;
  }
}
```

ChatService 클래스에서는 `addMessage()` 메서드를 사용하여 채팅 메시지를 저장하고 `getMessages()` 메서드를 사용하여 모든 채팅 메시지를 반환합니다.

<br/>

### 클라이언트 측 Socket.io 설정

마지막으로, 클라이언트 측 Socket.io 설정을 추가합니다. `static` 폴더 생성 후 다음과 같이 `index.html` 파일을 만듭니다.

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Chat App</title>
  </head>
  <body>
    <ul id="messages"></ul>
    <form id="chat-form">
      <input type="text" id="chat-input" />
      <button type="submit">Send</button>
    </form>
    <script src="/socket.io/socket.io.js"></script>
    <script>
      const socket = io();

      const messages = document.getElementById("messages");
      const chatForm = document.getElementById("chat-form");
      const chatInput = document.getElementById("chat-input");

      chatForm.addEventListener("submit", (event) => {
        event.preventDefault();
        const message = chatInput.value;
        socket.emit("chat", message);
        chatInput.value = "";
      });

      socket.emit("getMessages");

      socket.on("chat", (message) => {
        const li = document.createElement("li");
        li.textContent = message;
        messages.appendChild(li);
      });

      socket.on("messages", (messages) => {
        messages.forEach((message) => {
          const li = document.createElement("li");
          li.textContent = message;
          messages.appendChild(li);
        });
      });
    </script>
  </body>
</html>
```

위 코드에서는 [Socket.io](http://socket.io/) 클라이언트를 초기화하고 HTML 요소를 가져와서 이벤트 처리기를 등록합니다.

'chat-form' 폼을 제출하면 클라이언트는 'chat' 이벤트를 발행하고, 'getMessages' 이벤트를 발행하여 ChatService에서 모든 메시지를 가져옵니다.

'chat' 이벤트 및 'messages' 이벤트를 수신하면 채팅 메시지를 DOM에 추가합니다.

<br/>

### 서버 실행

마지막으로, `NestJS` 서버를 실행합니다. 터미널에서 다음과 같이 입력합니다.

```bash
$ npm run start
```

이렇게 하면 NestJS 서버가 실행됩니다.

<br/>

### 결과 확인

웹 브라우저에서 다음 URL에 접속합니다.

```
http://localhost:3000/
```

이제 브라우저에서 메시지를 입력하고 전송 버튼을 클릭하면 서버로 메시지가 전송되며, 모든 클라이언트에게 메시지가 전송됩니다.

또한 브라우저를 새로 고침하여 이전 메시지 목록을 확인할 수 있습니다.

<br/>

## End

이렇게 NestJS와 Socket.io를 사용하여 간단한 실시간 채팅 서비스를 만들어보았습니다.

이 코드를 기반으로 더 복잡하고 더 가독성 좋은 실시간 애플리케이션을 만들 수 있을 것 같습니다.

<br/>

코드를 참고하시려면 아래 깃허브 레포지토리에서 확인하실 수 있습니다.

[GitHub - 0126kjw/chat-app: [NestJS] Socket.io로 간단한 실시간 채팅 서비스 만들기](https://github.com/0126kjw/chat-app)
