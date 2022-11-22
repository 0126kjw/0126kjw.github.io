---
layout: post
title: "[TypeScript] 인터페이스에 대해"
categories: TypeScript
tags: TypeScript
---

<br/>

## Intro

TypeScipt의 핵심 원칙 중 하나는 타입 검사가 값의 형태에 초점을 맞추고 있다는 것 입니다.

이를 `덕 타이핑(duck typing)` 혹은 `구조적 서브타이핑(structural subtyping)`이라고도 합니다.

TypeScipt에서, `인터페이스(Interface)`는 이런 타입들의 이름을 짓는 역할을 하고 코드 안의 계약을 정의하는 것 뿐만 아니라

프로젝트 외부에서 사용하는 코드의 계약을 정의하는 강력한 방법입니다.

<br/>

### Our First Interface

인터페이스가 어떻게 동작하는지 확인하는 가장 쉬운 방법은 간단한 예제로 시작하는 것입니다.

```tsx
function printLabel(labeledObj: { label: string }) {
  console.log(labeledObj.label);
}

let myObj = { size: 10, label: "Size 10 Object" };
printLabel(myObj);
```

<br/>

타입 검사는 `printLabel` 호출을 확인합니다.

`printLabel` 함수는 `string` 타입인 `label`을 갖는 객체를 하나의 매개변수로 가집니다.

이 객체가 실제로는 더 많은 프로퍼티를 갖고 있지만, 컴파일러는 최소한 필요한 프로퍼티가 있는지와 타입이 잘 맞는지만 검사합니다.

이번엔 같은 예제를, 문자열 타입의 프로퍼티 label을 가진 인터페이스로 다시 작성해 보겠습니다.

```tsx
interface LabeledValue {
  label: string;
}

function printLabel(labeledObj: LabeledValue) {
  console.log(labeledObj.label);
}

let myObj = { size: 10, label: "Size 10 Object" };
printLabel(myObj);
```

<br/>

LabeledValue 인터페이스는 이전 예제의 요구사항을 똑같이 기술하는 이름으로 사용할 수 있습니다.

이 인터페이스는 이전 예제와 같이 string 타입의 label 프로퍼티 하나는 가진다는 것을 의미합니다.

다른 언어처럼 printLabel에 전달한 객체가 이 인터페이스를 구현해야 한다고 명시적으로 얘기할 필요는 없습니다.

여기세어 중요한 것은 형태뿐이며, 함수에 전달된 객체가 나열된 요구조건을 충족하면 허용됩니다.

<br/>

### Optional Properties

인터페이스의 모든 프로퍼티가 필요한 것은 아닙니다.

어떤 조건에서만 존재하거나 아예 없을 수도 있습니다.

선택적 프로퍼티(Optional Properties)들은 객체 안의 몇 개의 프로퍼티만 채워 함수에 전달하는 “option bags”같은 패턴을 만들 때 유용합니다.

아래 예제를 한번 확인해보겠습니다.

```tsx
interface SquareConfig {
  color?: string;
  width?: number;
}

function createSquare(config: SquareConfig): { color: string; area: number } {
  let newSquare = { color: "white", area: 100 };
  if (config.color) {
    newSquare.color = config.color;
  }
  if (config.width) {
    newSquare.area = config.width * config.width;
  }
  return newSqaure;
}

let mySquare = createSquare({ color: "black" });
```

<br/>

선택적 프로퍼티를 가지는 인터페이스는 다른 인터페이스와 비슷하게 작성되고, 선택적 프로퍼티는 선언에서 프로퍼티 이름 끝에 `?` 를 붙여 표시합니다.

선택적 프로퍼티의 이점은 **인터페이스에 속하지 않는 프로퍼티의 사용을 방지**하면서, **사용 가능한 속성을 기술**하는 것입니다.

예를 들어, `createSquare` 안의 `color` 프로퍼티 이름을 잘못 입력하면, 오류 메세지로 알려줍니다.

```tsx
interface SquareConfig {
  color?: string;
  width?: number;
}

function createSquare(config: SquareConfig): { color: string; area: number } {
  let newSquare = { color: "white", area: 100 };
  if (config.colr) {
    // Error: Property 'clor' does not exist on type 'SquareConfig'
    newSquare.color = config.color;
  }
  if (config.width) {
    newSquare.area = config.width * config.width;
  }
  return newSqaure;
}

let mySquare = createSquare({ color: "black" });
```

<br/>

### Readonly Properties

일부 프로퍼티들은 객체가 처음 생성될 때만 수정 가능해야 합니다.

이 때, 프로퍼티 이름 앞에 readonly를 넣어서 이를 지정할 수 있습니다.

```tsx
interface Point {
  readonly x: number;
  readonly y: number;
}
```

<br/>

객체 리터럴을 할당하여 `Point`를 생성합니다.

할당 후에는 `x`, `y`를 수정할 수 없습니다.

```tsx
let p1: Point = { x: 10, y: 20 };
p1.x = 5; // 오류 발생
```

<br/>

TypeScript에서는 모든 변경 메서드(Mutating Methods)가 제거된 `Array<T>`와 동일한 `ReadonlyArray<T>` 타입을 제공합니다.

```tsx
let a: number[] = [1, 2, 3, 4];
let ro: ReadonlyArray<number> = a;
ro[0] = 12; // 오류
ro.push(5); // 오류
ro.length = 100; // 오류
a = ro; // 오류
```

<br/>

위 예제의 마지막 줄에서 ReadonlyArray를 일반 배열에 재할당이 불가능한 것을 확인할 수 있습니다.

타입 단언(type assertion)으로 오버라이드하는 것은 가능합니다.

```tsx
a = ro as number[];
```

<br/>

`readonly`와 `const` 중 어떤 것을 사용할 지 기억하기 가장 쉬운 방법은 변수와 프로퍼티 중 어디에서 사용할 지 질문해보는 것입니다.

변수는 `const`를 사용하고 프로퍼티는 `readonly`를 사용합니다.

<br/>

### Excess Property Checks

위의 예제에서 TypeScript가 `{ label: string; }` 을 기대해도 `{ size: number; label: string; }` 를 허용해주었습니다.

또한, 선택적 프로퍼티를 배우고 소위 “option bags”을 기술할 때 유용하다는 것을 배웠습니다.

하지만, 그냥 그 둘을 결합하면 에러가 발생할 수 있습니다.

```tsx
interface SquareConfig {
  color?: string;
  width?: number;
}

function createSquare(config: SquareConfig): { color: string; area: number } {
  // ...
}

let mySquare = createSquare({ colour: "red", width: 100 });
```

<br/>

이 예제의 경우에는 매개변수가 colour로 전달되어 JavaScript에서는 조용히 오류가 발생합니다.

하지만, TypeScript는 이 코드에 버그가 있을 수 있다고 생각합니다.

객체 리터럴은 다른 변수에 할당할 때나 인수로 전달할 때, 특별한 처리를 받고, 초과 프로퍼티 검사(excess property checking)를 받습니다.

만약, 객체 리터럴이 “대상 타입(target type)”이 갖고 있지 않은 프로퍼티를 가지고 있으면 에러가 발생합니다.

```tsx
// error: Object literal may only specify known properties, but 'colour' does not exist in type 'SquareConfig'. Did you mean to write 'color'?
let mySquare = createSquare({ colour: "red", width: 100 });
```

<br/>

이 검사를 피하는 가장 간단한 방법은 타입 단언을 사용하는 것입니다.

```tsx
let mySquare = createSquare({ width: 100, opacity: 0.5 } as SquareConfig);
```

<br/>

하지만 특별한 경우에 추가 프로퍼티가 있음을 확신한다면, 문자열 인덱스 서명(string index signature)을 추가하는 것이 더 나은 방법입니다.

만약, SquareConfig의 `color`와 `width` 프로퍼티를 위와 같은 타입으로 갖고 있고, 또한 다른 프로퍼티를 가질 수 있다면, 다음과 같이 정의할 수 있습니다.

```tsx
interface SquareConfig {
  color?: string;
  width?: number;
  [propName: string]: any;
}
```

<br/>

이 검사를 피하는 마지막 방법은 놀랍게도 객체를 다른 변수에 할당하는 것입니다.

`squareOptions`가 추가 프로퍼티 검사를 받지 않기 때문에 컴파일러는 에러를 주지 않습니다.

```tsx
let squareOptions = { colour: "red", width: 100 };
let mySquare = createSquare(squareOptions);
```

<br/>

`squareOptions` 와 `SquareConfig` 사이에 공통 프로퍼티가 있는 경우에만 위와 같은 방법을 사용할 수 있습니다.

하지만 만약에 변수가 공통 객체 프로퍼티가 없으면 에러가 납니다.

```tsx
let squareOptions = { colour: "red" };
let mySquare = createSquare(squareOptions);
```

<br/>

위처럼 간단한 코드의 경우, 이 검사를 피하는 방법은 시도하지 않는 것이 좋습니다.

하지만 초과 프로퍼티 에러의 대부분은 실제 버그입니다.

그 말은, 만약 옵션 백 같은 곳에서 초과 프로퍼티 검사 문제가 발생하면, 타입 정의를 수정해야 할 필요가 있습니다.
