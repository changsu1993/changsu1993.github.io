---
title: JavaScript Call Stack 완벽 가이드 - 실행 컨텍스트와 비동기 처리의 핵심
date: 2020-09-03 22:20:00 +0900
categories: [개발, JavaScript]
tags: [javascript, call-stack, stack-frame, heap, web-api, queue, 비동기, 호출스택]
author: changsu
toc: true
comments: true
description: JavaScript Call Stack의 동작 원리부터 Stack Overflow, Web API, Message Queue까지. 비동기 처리의 핵심 메커니즘을 실전 예제와 함께 알아봅니다.
---

## 들어가며

**Call Stack(호출 스택)**은 JavaScript 엔진이 함수 호출을 관리하는 핵심 메커니즘입니다.

JavaScript는 **싱글 스레드(Single-threaded)** 프로그래밍 언어이기 때문에 Call Stack이 하나만 존재하며, 따라서 **한 번에 하나의 작업만** 처리할 수 있습니다.

> Call Stack은 함수의 호출을 기록하는 자료구조로, 프로그램 실행 중 현재 위치를 추적합니다.
{: .prompt-info }

## Call Stack이란?

**Call Stack**은 함수 호출들을 기록하는 자료구조입니다.

### Stack의 동작 방식

- **함수 실행**: 스택 위에 함수를 올림 (**Push**)
- **함수 반환**: 스택의 맨 위에서 함수를 제거 (**Pop**)

### 실행 예제

```javascript
function d() {
  console.log('d');
}

function c() {
  console.log('c');
  d();
}

function b() {
  console.log('b');
  c();
}

function a() {
  console.log('a');
  b();
}

a();
```

### Call Stack 실행 과정

| 순서 | Call Stack 상태 | 콘솔 출력 | 설명 |
|------|----------------|----------|------|
| 1 | `a()` | - | a 함수 호출 |
| 2 | `console.log('a')`<br>`a()` | - | console.log 실행 준비 |
| 3 | `a()` | `'a'` | console.log 완료, 스택에서 제거 |
| 4 | `b()`<br>`a()` | - | b 함수 호출 |
| 5 | `console.log('b')`<br>`b()`<br>`a()` | - | console.log 실행 준비 |
| 6 | `b()`<br>`a()` | `'b'` | console.log 완료, 스택에서 제거 |
| 7 | `c()`<br>`b()`<br>`a()` | - | c 함수 호출 |
| 8 | `console.log('c')`<br>`c()`<br>`b()`<br>`a()` | - | console.log 실행 준비 |
| 9 | `c()`<br>`b()`<br>`a()` | `'c'` | console.log 완료, 스택에서 제거 |
| 10 | `d()`<br>`c()`<br>`b()`<br>`a()` | - | d 함수 호출 |
| 11 | `console.log('d')`<br>`d()`<br>`c()`<br>`b()`<br>`a()` | - | console.log 실행 준비 |
| 12 | `d()`<br>`c()`<br>`b()`<br>`a()` | `'d'` | console.log 완료, 스택에서 제거 |
| 13 | `c()`<br>`b()`<br>`a()` | - | d 함수 완료, 스택에서 제거 |
| 14 | `b()`<br>`a()` | - | c 함수 완료, 스택에서 제거 |
| 15 | `a()` | - | b 함수 완료, 스택에서 제거 |
| 16 | (empty) | - | a 함수 완료, 스택 비어있음 |

**최종 콘솔 출력 순서:** `'a'` → `'b'` → `'c'` → `'d'`

> **중요**: 함수를 선언한 것이 아니라 **함수를 호출한 것**이 Call Stack에 기록됩니다!
{: .prompt-tip }

## Stack Frame

Call Stack의 각 항목을 **Stack Frame**이라고 합니다.

Stack Frame은 함수 호출과 관련된 정보를 담고 있으며, 후입선출(LIFO, Last In First Out) 방식으로 관리됩니다.

```javascript
function multiply(a, b) {
  return a * b;
}

function square(n) {
  return multiply(n, n);
}

function printSquare(n) {
  const result = square(n);
  console.log(result);
}

printSquare(5);
```

**Call Stack 상태:**
```
console.log(result)    ← Top
printSquare(5)
(empty)                ← Bottom
```

### 에러 스택 트레이스

브라우저 콘솔에서 에러가 발생하면 Stack Trace(스택 트레이스)를 보여줍니다.

```javascript
function foo() {
  throw new Error('Something went wrong!');
}

function bar() {
  foo();
}

function baz() {
  bar();
}

baz();
```

**에러 메시지:**
```
Error: Something went wrong!
    at foo (script.js:2)
    at bar (script.js:6)
    at baz (script.js:10)
    at script.js:13
```

> Stack Trace는 예외가 발생했을 때 Call Stack의 현재 상태를 보여줍니다. Top부터 Bottom까지 순서대로 표시됩니다.
{: .prompt-info }

## Stack Overflow

**Stack Overflow**는 Call Stack의 크기 제한을 초과했을 때 발생하는 에러입니다.

### 발생 원인

재귀 함수를 잘못 작성하여 무한 루프에 빠지는 경우 발생합니다.

```javascript
function recursiveFunction() {
  recursiveFunction();  // ❌ 종료 조건이 없는 무한 재귀
}

recursiveFunction();
// RangeError: Maximum call stack size exceeded
```

### 브라우저의 Stack 제한

| 브라우저 | 최대 Stack Frame 수 |
|----------|-------------------|
| Chrome | 약 16,000 프레임 |
| Firefox | 약 50,000 프레임 |
| Safari | 약 64,000 프레임 |

> Stack 제한을 넘어서면 **"Maximum call stack size exceeded"** 에러가 발생하고 실행이 중단됩니다. 이를 **"Blowing the stack"**(스택 날림)이라고 합니다.
{: .prompt-warning }

### 올바른 재귀 함수 작성

```javascript
// ✅ 종료 조건이 있는 재귀 함수
function countdown(n) {
  if (n <= 0) {  // 종료 조건
    console.log('Done!');
    return;
  }
  console.log(n);
  countdown(n - 1);
}

countdown(5);  // 5, 4, 3, 2, 1, Done!
```

## Heap (힙)

**Heap**은 메모리 할당이 이루어지는 영역입니다.

### Heap의 특징

| 특징 | 설명 |
|------|------|
| **용도** | 객체(Object)와 변수의 메모리 할당 |
| **구조** | 거의 구조화되지 않은(Unstructured) 메모리 영역 |
| **생명주기** | 가비지 컬렉션(GC)에 의해 관리 |

```javascript
// 객체는 Heap에 할당됩니다
const person = {
  name: 'John',
  age: 30
};

// 배열도 Heap에 할당됩니다
const numbers = [1, 2, 3, 4, 5];
```

> Call Stack은 함수 호출을 관리하고, Heap은 데이터를 저장하는 메모리 공간입니다.
{: .prompt-info }

## Web API

**Web API**는 브라우저에서 제공하는 API로, Call Stack과는 별도로 동작합니다.

### 주요 Web API

| API | 설명 | 예시 |
|-----|------|------|
| **DOM API** | HTML 요소 조작 | `document.querySelector()` |
| **Timer API** | 시간 지연 실행 | `setTimeout()`, `setInterval()` |
| **AJAX** | 비동기 HTTP 요청 | `fetch()`, `XMLHttpRequest` |
| **Event Listener** | 이벤트 처리 | `addEventListener()` |

### Web API의 역할

1. Call Stack에서 비동기 함수 실행
2. Web API가 해당 작업을 처리
3. 작업 완료 후 콜백 함수를 **Callback Queue**에 전달

> Web API는 브라우저가 관리하므로, JavaScript 엔진의 Call Stack과 독립적으로 동작합니다.
{: .prompt-info }

## Message Queue (메시지 큐)

**Message Queue**(또는 Callback Queue)는 실행 대기 중인 콜백 함수들을 FIFO(First In First Out) 방식으로 관리합니다.

### Queue의 특징

| 특징 | 설명 |
|------|------|
| **자료구조** | 선형(Linear) 자료구조 |
| **순서** | FIFO (먼저 들어간 것이 먼저 나옴) |
| **작업** | `enqueue` (추가), `dequeue` (제거) |

### Event Loop의 동작 원리

**Event Loop**는 Call Stack과 Message Queue를 감시하며, Call Stack이 비어있을 때만 Queue의 작업을 Stack으로 이동시킵니다.

**동작 규칙:**
1. Call Stack이 비어있는지 확인
2. 비어있으면 Message Queue의 첫 번째 콜백을 Call Stack으로 이동
3. 해당 콜백 함수 실행

> 콜백 함수가 등록되지 않은 이벤트는 Message Queue에 추가되지 않습니다.
{: .prompt-tip }

## 비동기 처리 예제

### setTimeout 동작 방식

```javascript
console.log('Start');

setTimeout(() => {
  console.log('Timeout');
}, 1000);

console.log('End');
```

**실행 순서:**

| 단계 | Call Stack | Web API | Message Queue | 콘솔 출력 |
|------|-----------|---------|--------------|----------|
| 1 | `console.log('Start')` | - | - | - |
| 2 | - | - | - | `'Start'` |
| 3 | `setTimeout(...)` | - | - | - |
| 4 | - | `setTimeout (1초 대기)` | - | - |
| 5 | `console.log('End')` | `setTimeout (1초 대기)` | - | - |
| 6 | - | `setTimeout (1초 대기)` | - | `'End'` |
| 7 | - | - | `callback` | - |
| 8 | `callback` | - | - | - |
| 9 | `console.log('Timeout')` | - | - | - |
| 10 | - | - | - | `'Timeout'` |

**최종 출력:**
```
Start
End
Timeout
```

> `setTimeout`은 지정된 시간 후에 콜백을 **Message Queue**에 추가하지만, 실제 실행은 Call Stack이 비어있을 때만 가능합니다.
{: .prompt-warning }

### fetch 동작 방식

```javascript
console.log('Start');

fetch('https://api.example.com/data')
  .then(response => response.json())
  .then(data => console.log('Data:', data));

console.log('End');
```

**실행 과정:**

1. **Call Stack**: `fetch()` 실행
2. **Web API**: HTTP 요청 시작 (브라우저가 처리)
3. **Call Stack**: `console.log('End')` 실행 → `'End'` 출력
4. **Web API**: HTTP 응답 수신 완료
5. **Message Queue**: `.then()` 콜백 추가
6. **Event Loop**: Call Stack이 비어있으므로 콜백 이동
7. **Call Stack**: 콜백 실행 → `'Data: ...'` 출력

**출력 순서:**
```
Start
End
Data: { ... }
```

## 실전 예제: 비동기 처리 순서 이해하기

```javascript
console.log('1');

setTimeout(() => {
  console.log('2');
}, 0);

Promise.resolve().then(() => {
  console.log('3');
});

console.log('4');
```

**출력 결과:**
```
1
4
3
2
```

**설명:**

| 순서 | 이유 |
|------|------|
| `'1'` | 동기 코드 먼저 실행 |
| `'4'` | 동기 코드 먼저 실행 |
| `'3'` | Promise는 Microtask Queue 사용 (우선순위 높음) |
| `'2'` | setTimeout은 Macrotask Queue 사용 (우선순위 낮음) |

> **Microtask Queue**는 **Macrotask Queue**보다 우선순위가 높습니다. Promise는 Microtask, setTimeout은 Macrotask입니다.
{: .prompt-tip }

## 핵심 정리

### JavaScript 실행 환경 구성 요소

| 구성 요소 | 역할 |
|----------|------|
| **Call Stack** | 함수 호출 관리 (LIFO) |
| **Heap** | 메모리 할당 (객체, 변수 저장) |
| **Web API** | 비동기 작업 처리 (브라우저 제공) |
| **Message Queue** | 콜백 함수 대기 (FIFO) |
| **Event Loop** | Stack과 Queue 간 작업 조정 |

### 비동기 처리 흐름

1. **동기 코드**가 Call Stack에서 먼저 실행
2. **비동기 함수**는 Web API로 이동
3. 작업 완료 후 콜백이 **Message Queue**에 추가
4. **Event Loop**가 Call Stack이 비어있을 때 콜백 실행

### Stack Overflow 방지

- ✅ 재귀 함수에 **종료 조건** 명확히 작성
- ✅ 반복 작업은 **재귀 대신 반복문** 사용 고려
- ✅ 꼬리 재귀 최적화 활용 (일부 환경에서 지원)

> Call Stack, Web API, Message Queue, Event Loop의 관계를 이해하면 JavaScript의 비동기 처리 메커니즘을 완벽하게 이해할 수 있습니다.
{: .prompt-info }

## 참고 자료

- [MDN - Concurrency model and Event Loop](https://developer.mozilla.org/ko/docs/Web/JavaScript/Event_loop)
- [JavaScript Visualizer - Call Stack](https://www.jsv9000.app/)
- [Philip Roberts - What the heck is the event loop anyway?](https://www.youtube.com/watch?v=8aGhZQkoFbQ)
