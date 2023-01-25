---
layout: post
title: "[GraphQL] DateTime 사용하기"
categories: GraphQL
tags: GraphQL DateTime
---

![og-image](https://user-images.githubusercontent.com/108377235/215277480-3e5b5d6c-7296-497d-ba70-1a32a04402be.png)

<br/>

## Problem

`GraphQL`을 사용해 댓글 기능을 구현하는 중에 문제가 생겼었습니다.

댓글을 작성할 때 자동으로 생성되게 설정해둔 `CreatedAt`과 `UpdatedAt`이

데이터베이스 상에는 Date Type으로 정상적으로 입력되는데

`GraphQL Playground`에서 확인을 하려니 잘 확인되지 않았습니다.

<br/>

`Type`을 `String`으로 설정 후 GraphQL Playground로 요청해보니

```json
{
  "data": {
    "addComment": {
      ...
      "createdAt": "1674922478352",
      "updatedAt": "1674922478352"
    }
  }
}
```

다음과 같은 값을 응답하는 것을 확인하였습니다.

이 값은 Date Type으로 변환되기 이전의 값이므로 Type 설정에 문제가 있다는 것을 알게되었습니다.

<br/>

그 후 GraphQL 문서에서 Type 관련 부분을 확인해보니 `Scalar Type`을 찾을 수 있었습니다.

이것은 간단하게 **GraphQL에 내장된 기본타입 이외의 타입을 선언할 수 있도록** 해주는 것입니다.

구글링을 해보니 Custom해서 사용하는 것도 가능하였습니다.

나중에 학습을 한 후에 Custom Scalar Type에 대해 다루어보겠습니다.

<br/>

## Solve

해결법은 `*.GraphQL` 에

```graphql
scalar DateTime
```

을 추가한 다음

<br/>

Date type으로 표시해야 할 부분에 다음과 같이 입력합니다.

```graphql
type Comment {
	...
  createdAt: DateTime!
  updatedAt: DateTime!
}
```

<br/>

그리고 GraphQL Playground에서 확인해보니 `Date Type`으로 변환되는 것을 확인할 수 있었습니다.

![Untitled](https://user-images.githubusercontent.com/108377235/215277536-0fdc9e2a-7ee8-4230-ad26-1d7f32ddf1fc.png)

<br/>

생각보다 오래 시간을 잡아먹었던 문제였는데 간단한 방법으로 문제가 해결되었습니다.

이 글이 저처럼 GraphQL을 처음 학습하는 분들에게 큰 도움이 됬으면 좋겠습니다.

<br/>

이후 학습 중에 공유할만한 내용이 있으면 포스팅하도록 하겠습니다.
