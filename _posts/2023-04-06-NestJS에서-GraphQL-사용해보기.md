---
layout: post
title: "NestJS에서 GraphQL 사용해보기"
categories: NestJS
tags: NestJS GraphQL
---

![https://user-images.githubusercontent.com/108377235/232286594-f0d9e031-0824-4782-95a7-9e55f2abddc9.png](https://user-images.githubusercontent.com/108377235/232286594-f0d9e031-0824-4782-95a7-9e55f2abddc9.png)

<br/>

## Intro

이전에 GraphQL을 다루면서 Datetime에 대해 헤맨적이 있어서 그 관련 글을 포스팅했던 적이 있었습니다.

생각해보니 GraphQL이 정확히 무엇이고 어떤 특징을 가지는 지에 대해 막연히 알고만 있어 이를 글로 작성해보고 싶어졌습니다.

그렇기에 이번 포스팅에서는 **GraphQL**을 간단히 소개하고 가지는 특성과 모든 언어의 첫 시작이 되는 `"Hello, World!"`를 구현하는 코드 예시를 작성해보겠습니다.

<br/>

## GraphQL?

`GraphQL`은 **Facebook**에서 개발된 쿼리 언어 및 런타임입니다.

`RESTful API`와 달리 GraphQL은 클라이언트가 데이터를 요청할 때 필요한 데이터만을 포함하여 정확히 원하는 데이터를 가져올 수 있도록합니다.

**GraphQL**은 또한 다양한 데이터 소스에서 데이터를 가져와서 일관된 인터페이스를 제공하므로 개발자가 데이터 가져오기를 단순화하고 관리하기 쉽게 만듭니다.

이를 통해 개발자는 어떠한 제약 없이 데이터를 가져올 수 있으며, 개발 및 유지 보수 비용을 낮출 수 있습니다.

<br/>

GraphQL은 REST API와는 다른 몇 가지 **독특한 특징**을 가지고 있습니다.

그 특징들은 다음과 같습니다.

### 강력한 타입 시스템

GraphQL은 **강력한 타입 시스템**을 제공합니다.

모든 쿼리와 뮤테이션은 사전에 정의된 타입과 인터페이스에 따라 작성됩니다.

이러한 타입 시스템은 API 사용자에게 API의 데이터 구조와 기능에 대한 명확하고 직관적인 이해를 제공합니다.

### 유연성

GraphQL은 REST API보다 **유연**합니다.

REST API에서는 클라이언트가 서버에서 반환하는 데이터 구조를 결정합니다.

하지만 GraphQL에서는 클라이언트가 필요한 데이터만 요청할 수 있습니다.

따라서 클라이언트는 필요한 데이터만 가져와서 불필요한 데이터를 가져오지 않을 수 있습니다.

### 단일 엔드포인트

GraphQL은 **단일 엔드포인트**를 제공합니다.

이는 여러 엔드포인트를 사용하는 REST API와는 대조적입니다.

GraphQL에서는 클라이언트가 필요한 데이터만 요청하면 됩니다.

이는 HTTP 요청 횟수를 줄이고 네트워크 대역폭을 절약하는 데 도움이 됩니다.

### 자동 문서화

GraphQL은 API를 **자동으로 문서화**할 수 있습니다.

GraphQL 스키마는 API의 데이터 구조와 기능을 정의합니다.

이러한 스키마를 사용하면 API를 자동으로 문서화하고 새로운 기능을 추가할 때마다 문서를 업데이트하는 데 필요한 작업을 줄일 수 있습니다.

### 데이터 가져오기 성능 최적화

GraphQL은 데이터 가져오기 성능을 **최적화**할 수 있습니다.

REST API에서는 데이터를 가져오기 위해 여러 번의 요청이 필요할 수 있습니다.

하지만 GraphQL에서는 클라이언트가 필요한 데이터만 요청하면 됩니다.

따라서 클라이언트는 필요한 데이터만 가져와서 불필요한 데이터를 가져오지 않아도 됩니다.

<br/>

## With NestJS?

**NestJS**에서는 `GraphQL` 모듈이 제공되며, 이를 사용하여 **GraphQL API**를 빠르고 쉽게 구축할 수 있습니다.

NestJS의 GraphQL 모듈은 `Apollo Server` 기반으로 작동하며, 다양한 GraphQL 기능을 제공합니다.

NestJS는 또한 `TypeORM` 등의 ORM(Object Relational Mapper)을 지원하므로, 데이터베이스에서 데이터를 가져와서 GraphQL API를 제공하는 애플리케이션을 쉽게 구축할 수 있습니다.

<br/>

### Install GraphQL Module

이제 본격적으로 코드를 작성해보겠습니다.

먼저, **NestJS** 애플리케이션에서 **GraphQL 모듈**을 설치해야합니다.

이를 위해 다음 명령어를 실행해보겠습니다.

```bash
npm install --save @nestjs/graphql apollo-server-express graphql-tools graphql
```

<br/>

### Setting GraphQL

다음으로, NestJS 애플리케이션에서 GraphQL 모듈을 설정해야합니다.

이를 위해 **`app.module.ts`** 파일에서 **`GraphQLModule`**을 import하고, **`GraphQLModule.forRoot()`** 메서드를 사용하여 GraphQL 설정을 구성해야합니다.

```tsx
import { Module } from "@nestjs/common";
import { GraphQLModule } from "@nestjs/graphql";

@Module({
  imports: [
    GraphQLModule.forRoot({
      autoSchemaFile: "schema.gql",
    }),
  ],
})
export class AppModule {}
```

위 예시 코드에서는 **`autoSchemaFile`** 옵션을 사용하여 스키마 파일을 자동으로 생성하도록 설정하였습니다.

<br/>

### **GraphQL Resolver**

이제 GraphQL 서비스를 만들어야합니다.

이를 위해, **`graphql.service.ts`** 파일을 생성하고, **`@Resolver()`** 데코레이터를 사용하여 Resolver 클래스를 정의할 수 있습니다.

```tsx
import { Resolver, Query } from "@nestjs/graphql";

@Resolver()
export class HelloResolver {
  @Query(() => String)
  async hello() {
    return "Hello, World!";
  }
}
```

위 예시 코드에서는 **`@Resolver()`** 데코레이터를 사용하여 Resolver 클래스를 정의하였습니다.

**`@Query()`** 데코레이터를 사용하여 Query 메서드를 정의하고, **`async hello()`** 메서드를 정의하여 'Hello, World!' 문자열을 반환하도록 설정하였습니다.

<br/>

### Setting **Endpoint**

마지막으로, 애플리케이션의 엔드포인트를 설정해야합니다.

이를 위해, **`main.ts`** 파일에서 Express 앱 객체를 가져와 **`app.use()`** 메서드를 사용하여 GraphQL 엔드포인트를 설정할 수 있습니다.

```tsx
import { NestFactory } from "@nestjs/core";
import { AppModule } from "./app.module";
import * as express from "express";

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  const server = express();
  app.use("/graphql", server);
  await app.listen(3000);
}
bootstrap();
```

위 예시 코드에서는 **`app.use('/graphql', server)`** 메서드를 사용하여 GraphQL 엔드포인트를 설정하였습니다.

이제 NestJS 애플리케이션에서 GraphQL을 사용할 수 있습니다.

<br/>

### **GraphQL Playground**

GraphQL Playground는 GraphQL API를 시각적으로 탐색할 수 있는 IDE입니다.

이를 실행하려면 다음과 같이 **`app.module.ts`** 파일에서 **`GraphQLModule.forRoot()`** 메서드의 **`playground`** 옵션을 활성화해야합니다.

```tsx
import { Module } from "@nestjs/common";
import { GraphQLModule } from "@nestjs/graphql";

@Module({
  imports: [
    GraphQLModule.forRoot({
      autoSchemaFile: "schema.gql",
      playground: true,
    }),
  ],
})
export class AppModule {}
```

위 예시 코드에서는 **`playground`** 옵션을 **`true`**로 설정하여 GraphQL Playground를 활성화하였습니다.

이제 애플리케이션을 실행하고 브라우저에서 **`http://localhost:3000/graphql`**에 접속하면 GraphQL Playground를 사용할 수 있습니다.

<br/>

![https://user-images.githubusercontent.com/108377235/232286782-d5839d34-ad7f-4371-913a-712c2ad187f7.png](https://user-images.githubusercontent.com/108377235/232286782-d5839d34-ad7f-4371-913a-712c2ad187f7.png)

위 모습은 GraphQL Playgound에서의 예시입니다.

<br/>

## End

NestJS에서 GraphQL 모듈을 사용하여 GraphQL API를 쉽게 구축할 수 있었습니다..

또한 **`playground`** 옵션을 활성화하면 GraphQL Playground를 사용하여 API를 시각적으로 탐색할 수 있는 것을 확인했습니다.

GraphQL을 직접 사용해보니 매우 강력하고 유연한 API 디자인 도구라는 생각이 들었습니다.

<br/>

하지만 모든 경우에 GraphQL이 적합하지는 않을 수 있습니다.

따라서 사용 시 장단점을 고려하여 적절하게 사용해야 하는 것이 좋다고 생각합니다.
