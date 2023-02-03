---
layout: post
categories: TypeScript
tags: TypeScript
---

![2560px-TypeScript_Logo_(Blue) svg](https://user-images.githubusercontent.com/108377235/216560590-a3b01972-1296-4ac5-9440-d9b0f641260e.png)

<br/>

본 게시물은 TypeScript-Handbook 한글 문서의 내용을 정리한 내용입니다.

추가적인 정보를 원하신다면 아래의 링크를 참고하시길 바랍니다.

[https://typescript-kr.github.io](https://typescript-kr.github.io)

---

<br/>

이번에 학습 중인 부트캠프에서 진행하는

2차 프로젝트가 종료되었습니다.

이번 프로젝트에서도 백엔드 파트를 맡았고

Django를 사용해 구현하는 작업을 하였습니다.

이번 프로젝트를 통해 Django를 통해 돌아가는 Python 서버의 특징에 대해 알 수 있었고

node.js와의 차이점에 대해 확실히 배울 수 있었습니다.

곧 진행될 3차 프로젝트를 대비해서 이번에는

TypeScript를 공부해보기로 했습니다.

추후에 Django를 학습하게 될 일이 생긴다면

Django REST Framework 공식 문서의 내용을 정리하도록 하겠습니다.

이번에 공부할 TypeScript는 JavaScript의 단점을 보완하기 위해 만들어진 프로그래밍 언어입니다.

TypeScript를 설명하는 위키백과의 내용을 인용하면 다음과 같습니다.

> 타입스크립트(TypeScript)는 자바스크립트의 슈퍼셋인 오픈소스 프로그래밍 언어이다. 마이크로소프트에서 개발, 유지하고 있으며 엄격한 문법을 지원한다. ... 클라이언트 사이드와 서버 사이드를 위한 개발에 사용할 수 있다.
> 타입스크립트는 자바스크립트 엔진을 사용하면서 커다란 애플리케이션을 개발할 수 있게 설계된 언어이다.자바스크립트의 슈퍼셋이기 때문에 자바스크립트로 작성된 프로그램이 타입스크립트 프로그램으로도 동작한다.
> 타입스크립트에서 자신이 원하는 타입을 정의하고 프로그래밍을 하면 자바스크립트로 컴파일되어 실행할 수 있다.
> 타입스크립트는 모든 운영 체제, 모든 브라우저, 모든 호스트에서 사용 가능한 오픈 소스이다.

TypeScript가 가지는 특징에 대해서는 함께 TypeScript에 대해 학습하면서 알려드릴 수 있도록 하겠습니다.

우선 TypeScript를 통해 간단한 웹 애플리케이션을 만들어보며 TypeScript에 대해 입문해보도록 하겠습니다.

<br/>

## Installing TypeScript

TypeScript를 설치하는 방법에는 크게 두 가지가 있습니다.

- npm을 이용한 설치 (Node.js 패키지 매니저)
- TypeScript의 Visual Studio 플러그인 설치

Visuak Studio 2017rhk 2015 Update 3는 기본적으로 TypeScript를 포함하고 있다고 합니다.

저는 VSC를 사용 중이기 때문에 npm을 이용해 설치하도록 하겠습니다.

```shell
$ npm install -g typescript
```

<br/>

## Building your first TypeScript file

현재 사용중인 에디터에서 `greeter.ts` 라는 파일을 생성 후 다음의 JavaScript 코드를 입력해봅니다.

```typescript
function greeter(person) {
  return "Hello, " + person;
}

let user = "Jane User";

document.body.textContent = greeter(user);
```

<br/>

## Compiling your code

`.ts` 확장자를 사용했지만, 이 코드는 아직 일반적인 JavaScript 코드입니다.

기존의 JavaScript 앱에서 이 코드를 바로 복사하여 붙여 넣을 수 있습니다.

커맨드 라인에서 TypeScript 컴파일러를 실행해봅니다.

```shell
$ tsc greeter.ts
```

결과는 동일한 JavaScript 코드를 포함하고 있는 `greeter.ts` 파일이 될 것입니다.

JavaScript 앱에서 TypeScript를 사용하여 시작했습니다.

이제 TypeScript가 제공하는 몇 가지의 새로운 기능을 이용할 수 있습니다.

다음과 같이 `: string` 타입 표기를 'person' 함수의 인수에 추가해봅니다.

```typescript
function greeter(person: string) {
  return "Hello, " + person;
}

let user = "Jane User";

document.body.textContent = greeter(user);
```

<br/>

## Type annotations

TypeScript의 타입 표기는 함수나 변수의 의도된 계약을 기록하는 간단한 방법입니다.

아래의 경우, greeter 함수를 단일 문자열 매개변수와 함께 호출하려고 합니다.

우리는 greeter 함수 호출 시 매개변수로 배열을 전달하도록 변경해 볼 수 있습니다.

```typescript
function greeter(person: string) {
  return "Hello, " + person;
}

let user = [0, 1, 2];

document.body.textContent = greeter(user);
```

다시 컴파일해보면, 오류가 발생한 것을 볼 수 있습니다.

```
error TS2345: Argument of type 'number[]' is not assignable to parameter of type 'string'.
```

마찬가지로, greeter 함수 호출에서 모든 인수를 제거해본다면

TypeScript는 당신이 예상치 못한 개수의 매개변수로 이 함수를 호출했다는 것을 알려줄 것입니다.

두 경우 모두, TypeScript는 코드의 구조와 타입 표기에 기반해서 정적 분석을 제공합니다.

오류가 존재하기는 하지만, greeter.js 파일은 생성되었습니다.

코드에 오류가 존재하더라도 TypeScript를 사용할 수 있습니다.

그러나 TypeScript는 코드가 예상대로 작동하지 않을 것이라는 경고를 하게 됩니다.

<br/>

## Interfaces

예제를 더 발전시켜 보도록 하겠습니다.

여기서는 firstName 및 lastName 필드를 갖고 있는 객체를 나타내는 인터페이스를 사용합니다.

TypeScript에서, 내부 구조가 호환되는 두 타입은 서로 호환 됩니다.

그래서 명시적인 `implements` 절 없이, 인터페이스가 요구하는 형태를 사용하는 것만으로도 인터페이스를 구현할 수 있습니다.

```typescript
interface Person {
  firstName: string;
  lastName: string;
}

function greeter(person: Person) {
  return "Hello, " + person.firstName + " " + person.lastName;
}

let user = { firstName: "Jane", lastName: "User" };

document.body.textContent = greeter(user);
```

<br/>

## Classes

마지막으로, 클래스를 사용하여 예제를 확장해 보겠습니다.

TypeScript는 클래스 기반 객체 지향 프로그래밍 지원과 같은 JavaScript의 새로운 기능을 지원합니다.

생성자와 public 필드를 사용해 Student 클래스를 만들겠습니다.

클래스와 인터페이스가 잘 작동하여, 프로그래머가 올바른 추상화 수준을 결정할 수 있게 해준다는 점을 주목하시면 됩니다.

또한, 생성자의 인수에 public을 사용하는 것은 그 인수의 이름으로 프로퍼티를 자동으로 생성하는 축약형입니다.

```typescript
class Student {
  fullName: string;
  constructor(
    public firstName: string,
    public middleInitial: string,
    public lastName: string
  ) {
    this.fullName = firstName + " " + middleInitial + " " + lastName;
  }
}

interface Person {
  firstName: string;
  lastName: string;
}

function greeter(person: Person) {
  return "Hello, " + person.firstName + " " + person.lastName;
}

let user = new Student("Jane", "M.", "User");

document.body.textContent = greeter(user);
```

`tsc greeter.ts`를 재실행하면 생성된 JavaScript 코드가 이전의 코드와 동일하다는 것을 알 수 있습니다.

TypeScript의 클래스는 단지 JavaScript에서 자주 사용되는 프로토타입-기반 OO의 축약형일 뿐입니다.

<br/>

## Running your TypeScript web app

이제 아래의 코드를 `greeter.html`에 작성합니다.

```html
<!DOCTYPE html>
<html>
  <head>
    <title>TypeScript Greeter</title>
  </head>
  <body>
    <script src="greeter.js"></script>
  </body>
</html>
```

브라우저에서 greeter.html을 열어 간단한 첫 번째 TypeScript 웹 앱을 실행해봅니다.

<br/>
<br/>
<br/>

---

본 게시물은 TypeScript-Handbook 한글 문서의 내용을 정리한 내용입니다.

추가적인 정보를 원하신다면 아래의 링크를 참고하시길 바랍니다.

[https://typescript-kr.github.io](https://typescript-kr.github.io)
