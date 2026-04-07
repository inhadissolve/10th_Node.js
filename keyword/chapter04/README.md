## 1. ESM, CommonJS 비교

Node.js에는 대표적으로 두 가지 모듈 방식이 있다.

- CommonJS
- ESM(ES Module)

### CommonJS

기존 Node.js에서 기본적으로 많이 사용되던 방식이다.

예시

```jsx
const express = require('express');
module.exports = app;
```

### ESM

ES6에서 도입된 표준 모듈 방식이다.

예시

```
importexpressfrom'express';
exportdefaultapp;
```

### 차이점

### 1) 문법 차이

- CommonJS는 `require`, `module.exports`
- ESM은 `import`, `export`

### 2) 분석 시점 차이

ESM은 코드 실행 전에 import/export를 분석할 수 있다.

그래서 번들러, 린터, 컴파일러가 의존성 구조를 더 정확하게 파악할 수 있다.

### 3) 로딩 방식 차이

CommonJS는 기본적으로 동기적으로 모듈을 불러오고,

ESM은 더 현대적인 모듈 시스템으로 비동기적 처리와 정적 분석에 더 유리하다.

### 왜 ESM을 쓰는가

이번 워크북에서도 Express 예제를 `import express from 'express'` 형태로 사용했고, 이를 위해 `package.json`에 `"type": "module"`을 설정했다.

즉, ESM은 최신 JavaScript/TypeScript 프로젝트에서 더 자연스럽고, 앞으로 TS 환경으로 넘어갈 때도 더 잘 맞는 방식이라고 느꼈다.

### 내가 이해한 정리

CommonJS는 기존 Node 스타일,

ESM은 최신 JavaScript 표준 방식이라고 이해했다.

앞으로는 ESM을 기본으로 쓰는 흐름이 더 자연스럽다고 느꼈다.

## 2. ES6의 주요 문법

### 2-1. let, const, var 차이

### const

재할당이 불가능한 변수 선언 방식이다.

```
constport=3000;
```

가급적 기본은 `const`를 쓰고, 값이 변할 때만 `let`을 쓰는 것이 좋다.

### let

재할당이 가능한 변수 선언 방식이다.

```
letcount=1;
count=2;
```

### var

예전 방식이고, 호이스팅과 스코프 문제 때문에 지금은 사용을 지양하는 편이다.

### 내가 느낀 점

무조건 `let`을 쓰는 것보다,

“안 바뀌는 값은 const, 바뀌는 값만 let”

이 기준으로 코드를 쓰는 게 더 명확하다고 느꼈다.

---

### 2-2. 구조분해할당

객체나 배열의 값을 더 쉽게 꺼내는 문법이다.

```
constuser= { name:"JaeSeo", age:25 };
const { name, age }=user;
```

배열도 가능하다.

```
constnumbers= [10,20,30];
const [first,second]=numbers;
```

Express에서는 특히 `req.body`, `req.params`에서 자주 사용된다.

```
const { title, content }=req.body;
```

### 내가 느낀 점

구조분해할당은 단순히 문법을 줄이는 기능이 아니라,

어떤 값을 꺼내서 쓸지 한눈에 보여준다는 점에서 코드가 더 읽기 쉬워진다고 느꼈다.

---

### 2-3. async / await

자바스크립트는 단일 스레드로 작동하기 때문에, 오래 걸리는 작업은 비동기로 처리하는 것이 중요하다.

`async / await`은 비동기 코드를 동기 코드처럼 읽기 쉽게 만들어준다.

```
asyncfunctionfetchData() {
return"데이터";
}
```

`await`은 Promise가 끝날 때까지 기다린다.

```
asyncfunctiongetUser() {
constresponse=awaitfetch("https://api.example.com/user");
constdata=awaitresponse.json();
console.log(data);
}
```

에러 처리는 `try-catch`로 감싸는 것이 중요하다.

```
asyncfunctionfetchData() {
try {
constres=awaitfetch("https://api.example.com/user");
constdata=awaitres.json();
console.log(data);
  }catch (error) {
console.error("오류 발생:",error.message);
  }
}
```

### 내가 느낀 점

Node.js 서버에서는 DB 조회나 외부 API 요청처럼 기다려야 하는 작업이 많기 때문에,

`async / await`은 사실상 필수 문법처럼 느껴졌다

### 2-4. 옵셔널 체이닝과 nullish 문법

### 옵셔널 체이닝 `?.`

값이 `null` 또는 `undefined`일 수 있을 때 안전하게 접근하는 문법이다.

```
constcity=user?.address?.city;
```

중간 값이 없으면 에러 대신 `undefined`를 반환한다.

### nullish 병합 연산자 `??`

왼쪽 값이 `null` 또는 `undefined`일 때만 기본값을 사용한다.

```
constnickname=user.nickname??"익명";
```

## 3. TypeScript 문법

### 3-1. TypeScript를 사용하는 이유

워크북은 TypeScript를 JavaScript의 슈퍼셋이라고 설명한다.

즉, JavaScript 문법을 그대로 포함하면서 타입 시스템 같은 추가 기능을 제공하는 언어다.

### 왜 TypeScript를 쓰는가

JavaScript는 동적 타입 언어라서 유연하지만, 프로젝트 규모가 커질수록 예상치 못한 타입 문제를 발견하기 어려워질 수 있다.

TypeScript는 컴파일 단계에서 타입 검사를 통해 이런 문제를 미리 잡아준다.

예를 들어 JavaScript에서는 아래 코드가 동작할 수 있다.

```
consta=1;
constb='1';
console.log(a-b);// 0
```

하지만 TypeScript에서는 이런 코드에 경고를 줄 수 있다.

즉, “지금은 돌아가도 나중에 문제가 될 수 있는 코드”를 미리 알려주는 역할을 한다.

### 내가 느낀 점

TypeScript는 문법이 복잡해서 쓰는 게 아니라,

프로젝트가 커질수록 코드의 안정성을 높이고 유지보수를 쉽게 하려고 쓰는 언어라고 이해했다.

- Typescript를 사용하는 이유를 JS와 비교해서 설명하고, 기본타입(string, number, any, unknown, never)에 대해 정리해주세요

## 3-2. TypeScript 기본 타입 정리

### string

문자열 타입

```
leta:string="UMC";
```

### number

숫자 타입

```
letb:number=10;
```

### any

아무 타입이나 허용하는 타입

타입 검사의 장점을 거의 포기하는 느낌이라 남용하면 안 된다고 느꼈다.

```
letc:any="hello";
c=123;
```

### unknown

값의 타입을 아직 모를 때 사용하는 타입

`any`와 비슷해 보이지만, 실제 사용 전에 타입 확인이 필요해서 더 안전하다.

```
letd:unknown="test";
```

### never

절대 값이 발생하지 않는 타입

예외를 던지거나, 절대 끝나지 않는 함수 같은 곳에서 사용된다.

```
functionthrowError(message:string):never {
thrownewError(message);
}
```

### 추가로 같이 알아둔 타입

- boolean
- null
- undefined

## 4. 프로젝트 아키텍쳐

### 4-1. 무엇이고 왜 중요한지?
프로젝트 아키텍처는 프로젝트의 전체 구조를 어떻게 나눌지에 대한 설계라고 이해했다.

즉, 코드를 어떤 기준으로 폴더와 역할을 나눌지 정하는 큰 틀이다.

### 왜 중요한가

워크북도 프로젝트 구조에는 정답이 없지만, 구조를 잘 잡아야

- 코드 찾기가 쉬워지고
- 팀 개발 충돌이 줄고
- 변경 영향 범위를 줄일 수 있다고 설명하고 있다.

### 내가 느낀 점

처음에는 폴더 구조가 그냥 취향 차이처럼 느껴졌는데,

실제로는 프로젝트가 커질수록 유지보수성과 협업 효율에 큰 영향을 주는 부분이라고 느꼈다.

## 4-2. 레이어드 아키텍처와 각 계층의 역할

이번 워크북은 **모듈형 모놀리스 + 레이어드 아키텍처(Service Layer Pattern)** 방향을 제안하고 있다.

### 레이어드 아키텍처란

요청 처리 과정을 역할별로 나누는 구조다.

대표적으로

- Controller
- Service
- Repository

이 세 계층으로 나눈다.

### Controller

- 클라이언트의 요청을 받는다
- 필요한 값을 꺼내 서비스에 전달한다
- 최종 응답을 반환한다

### Service

- 비즈니스 로직을 처리한다
- 어떤 요청을 어떤 순서로 처리할지 결정한다
- 필요하면 Repository를 호출한다

### Repository

- DB와 직접 소통한다
- 쿼리 실행, 데이터 저장, 조회를 담당한다

### 흐름

클라이언트 요청 → Controller → Service → Repository → DB → 다시 Service → Controller → 클라이언트 응답

### 내가 느낀 점

이 구조를 쓰면 각 파일의 책임이 분명해져서 코드가 덜 섞인다는 장점이 있다.

특히 서비스 로직과 DB 접근을 나눠두면 나중에 수정할 때 훨씬 편할 것 같다고 느꼈다.

## 5. DTO가 무엇인지, DTO 없이 req.body 만 사용하면 어떤 문제가 있을지 조사

DTO는 Data Transfer Object의 약자다.

말 그대로 계층 사이에서 데이터를 주고받기 위한 객체라고 이해했다.

### 왜 필요한가

워크북에서는 DTO 없이 그냥 `req.body`를 바로 사용하면

- 값 누락
- 타입 문제
- 유효성 검사 코드 분산
- 서비스 함수 재사용성 저하
같은 문제가 생길 수 있다고 설명한다.

### DTO의 장점

- 데이터 형식을 명확하게 맞출 수 있다
- 유효성 검사와 타입 제한을 붙이기 쉽다
- 민감한 정보를 걸러내는 데도 유리하다
- Controller와 Service 사이 데이터 전달이 더 명확해진다

### DTO 없이 req.body만 쓰면 생길 수 있는 문제

- 어떤 값이 들어와야 하는지 한눈에 안 보임
- 잘못된 값이 들어와도 놓치기 쉬움
- 검증 코드가 여기저기 흩어짐
- 서비스 함수가 특정 요청 형식에 너무 종속될 수 있음

### 내가 느낀 점

처음에는 DTO가 조금 번거롭게 느껴질 수도 있지만,

입력값이 많아지고 검증이 필요해질수록 DTO가 없으면 오히려 코드가 더 지저분해질 것 같다고 느꼈다.