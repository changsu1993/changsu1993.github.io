---
title: "JavaScript 메모리 관리와 가비지 컬렉션 완벽 가이드 - 메모리 누수 방지와 성능 최적화"
description: "JavaScript 메모리 관리의 핵심인 Stack과 Heap, Mark-and-Sweep 가비지 컬렉션 알고리즘, 흔한 메모리 누수 패턴 5가지와 Chrome DevTools로 디버깅하는 방법을 실전 예제와 함께 완벽하게 이해합니다."
date: 2025-12-10 15:00:00 +0900
categories: [Frontend, JavaScript]
tags: [javascript, memory-management, garbage-collection, memory-leak, chrome-devtools, heap, stack, mark-and-sweep, reference-counting, weakmap, weakset, performance, debugging, v8-engine, react-memory-leak]
---

## 개요

JavaScript는 **자동 메모리 관리** 언어입니다. C/C++처럼 개발자가 직접 메모리를 할당하고 해제할 필요 없이, **가비지 컬렉터(Garbage Collector, GC)**가 더 이상 사용되지 않는 메모리를 자동으로 정리합니다.

하지만 "자동"이라고 해서 메모리 관리를 완전히 무시해도 되는 것은 아닙니다. 메모리 누수(Memory Leak)는 JavaScript 애플리케이션에서 흔히 발생하며, 심각한 성능 저하의 원인이 됩니다.

이 가이드에서는 다음을 학습합니다:

- **Part 1**: JavaScript 메모리 관리의 기초 - Stack과 Heap의 차이
- **Part 2**: 가비지 컬렉션 알고리즘의 동작 원리
- **Part 3**: 흔한 메모리 누수 패턴과 해결 방법
- **Part 4**: Chrome DevTools로 메모리 문제 디버깅하기

> 이 글은 [JavaScript Call Stack](/posts/javascript-call-stack/)과 [원시 타입과 참조 타입](/posts/javascript-primitive-reference-types/)을 먼저 읽으시면 더 깊이 이해할 수 있습니다.
{: .prompt-info }

---

## Part 1: JavaScript 메모리 관리 기초

### 1.1 메모리 생명주기

모든 프로그래밍 언어에서 메모리 관리는 동일한 생명주기를 따릅니다:

```
할당(Allocate) → 사용(Use) → 해제(Release)
```

| 단계 | 설명 | JavaScript에서의 처리 |
|------|------|----------------------|
| **할당** | 필요한 메모리 공간 확보 | 변수 선언, 객체 생성 시 자동 할당 |
| **사용** | 읽기/쓰기 작업 수행 | 변수 접근, 객체 속성 조작 |
| **해제** | 더 이상 필요 없는 메모리 반환 | 가비지 컬렉터가 자동 수행 |

```javascript
// 1. 할당: 메모리 공간 확보
const name = '홍길동';           // 문자열을 위한 메모리 할당
const user = { id: 1, age: 25 }; // 객체를 위한 메모리 할당
const numbers = [1, 2, 3, 4, 5]; // 배열을 위한 메모리 할당

// 2. 사용: 할당된 메모리 읽기/쓰기
console.log(name);              // 읽기
user.age = 26;                  // 쓰기

// 3. 해제: 가비지 컬렉터가 자동으로 처리
// (개발자가 명시적으로 해제할 필요 없음)
```

### 1.2 Stack vs Heap 메모리

JavaScript 엔진은 두 가지 메모리 영역을 사용합니다:

#### Stack 메모리

**Stack**은 **정적 메모리 할당**에 사용됩니다. 원시 타입(Primitive Types)과 함수 호출 정보가 저장됩니다.

```javascript
function calculate() {
  const a = 10;        // Stack에 저장
  const b = 20;        // Stack에 저장
  const result = a + b; // Stack에 저장
  return result;
}

calculate(); // 함수 종료 시 Stack에서 자동 제거
```

**Stack의 특징:**

- **LIFO(Last In, First Out)** 구조
- 고정 크기의 데이터 저장
- 함수 실행 컨텍스트 관리
- 매우 빠른 접근 속도
- 함수 종료 시 자동으로 메모리 해제

#### Heap 메모리

**Heap**은 **동적 메모리 할당**에 사용됩니다. 객체, 배열, 함수 등 참조 타입(Reference Types)이 저장됩니다.

```javascript
function createUser() {
  // 객체는 Heap에 저장, 참조(주소)는 Stack에 저장
  const user = {
    name: '홍길동',
    hobbies: ['독서', '코딩', '운동']
  };
  return user;
}

const user1 = createUser(); // Heap의 객체는 여전히 존재
```

**Heap의 특징:**

- 동적으로 크기 변경 가능
- 참조를 통해 접근
- 가비지 컬렉터가 메모리 해제 담당
- Stack보다 접근 속도가 느림

#### Stack과 Heap의 관계 시각화

**Stack 메모리**

| 변수 선언 | 저장된 값 |
|-----------|-----------|
| `const num = 42` | `42` |
| `const str = "hello"` | `"hello"` |
| `const user = { name: "Kim" }` | `0x001` (Heap 주소 참조) |
| `const arr = [1, 2, 3]` | `0x002` (Heap 주소 참조) |

⬇️ 참조

**Heap 메모리**

| 주소 | 저장된 데이터 |
|------|---------------|
| `0x001` | `{ name: "Kim" }` |
| `0x002` | `[1, 2, 3]` |
| `0x003` | (해제된 메모리 - GC 대상) |

### 1.3 JavaScript의 자동 메모리 할당

JavaScript는 다양한 상황에서 자동으로 메모리를 할당합니다:

```javascript
// 원시 값 할당
const number = 123;
const string = 'JavaScript';
const boolean = true;

// 객체 할당
const object = { key: 'value' };
const array = [1, 2, 3];
const date = new Date();
const regex = /pattern/g;

// 함수 할당
const func = function() { return 'hello'; };
const arrow = () => 'world';

// 함수 호출로 인한 할당
const joined = ['a', 'b'].join('-'); // 새 문자열 생성
const sliced = [1, 2, 3].slice(1);   // 새 배열 생성
const cloned = { ...object };         // 새 객체 생성
```

---

## Part 2: 가비지 컬렉션 알고리즘

### 2.1 "도달 가능성(Reachability)" 개념

가비지 컬렉션의 핵심 개념은 **도달 가능성(Reachability)**입니다. 어떤 값이 **루트(Root)**에서 참조 체인을 통해 도달할 수 있으면 메모리에 유지되고, 도달할 수 없으면 가비지로 간주됩니다.

#### 루트(Root)란?

- 전역 객체 (브라우저: `window`, Node.js: `global`)
- 현재 실행 중인 함수의 지역 변수와 매개변수
- 현재 활성화된 중첩 함수 체인의 변수들

```javascript
// 도달 가능한 값
let user = { name: '홍길동' }; // 전역 변수 user가 참조 → 도달 가능

// 도달 불가능해지는 경우
user = null; // 이제 { name: '홍길동' } 객체는 도달 불가능 → GC 대상
```

### 2.2 Reference Counting (참조 카운팅)

**참조 카운팅**은 초기 가비지 컬렉션 알고리즘입니다. 각 객체가 몇 번 참조되는지 카운트하고, 참조 횟수가 0이 되면 메모리를 해제합니다.

```javascript
let obj1 = { data: 'hello' };  // { data: 'hello' } 참조 횟수: 1
let obj2 = obj1;                // { data: 'hello' } 참조 횟수: 2

obj1 = null;                    // { data: 'hello' } 참조 횟수: 1
obj2 = null;                    // { data: 'hello' } 참조 횟수: 0 → GC 대상
```

#### 참조 카운팅의 치명적 한계: 순환 참조

```javascript
function createCircularReference() {
  const objA = {};
  const objB = {};

  // 순환 참조 생성
  objA.ref = objB; // objA가 objB를 참조
  objB.ref = objA; // objB가 objA를 참조

  return 'done';
}

createCircularReference();
// 함수 종료 후에도 objA와 objB는 서로를 참조
// 참조 카운팅 방식에서는 절대 GC되지 않음 (메모리 누수)
```

> 순환 참조 문제로 인해 현대 JavaScript 엔진은 참조 카운팅 대신 **Mark-and-Sweep** 알고리즘을 사용합니다.
{: .prompt-warning }

### 2.3 Mark-and-Sweep 알고리즘

현대 JavaScript 엔진(V8, SpiderMonkey, JavaScriptCore)은 **Mark-and-Sweep** 알고리즘을 사용합니다.

#### 동작 과정

1. **Mark 단계**: 루트에서 시작하여 도달 가능한 모든 객체를 "표시(mark)"
2. **Sweep 단계**: 표시되지 않은 객체의 메모리 해제

```
Mark 단계:
┌────────┐
│  Root  │
└────┬───┘
     │
     ▼
┌────────┐     ┌────────┐     ┌────────┐
│ Obj A  │────▶│ Obj B  │────▶│ Obj C  │
│ (mark) │     │ (mark) │     │ (mark) │
└────────┘     └────────┘     └────────┘

┌────────┐     ┌────────┐
│ Obj D  │────▶│ Obj E  │     ← 루트에서 도달 불가능
│        │     │        │        (표시되지 않음)
└────────┘     └────────┘

Sweep 단계:
- Obj A, B, C: 유지
- Obj D, E: 메모리 해제
```

```javascript
// Mark-and-Sweep은 순환 참조도 해결
function circularRef() {
  const a = {};
  const b = {};
  a.ref = b;
  b.ref = a;
  // 함수 종료 후 a, b 모두 루트에서 도달 불가능
  // → 서로 참조하더라도 GC 대상이 됨
}

circularRef();
// a와 b는 서로 참조하지만, 루트에서 도달할 수 없으므로 GC됨
```

### 2.4 V8 엔진의 세대별 가비지 컬렉션

Chrome과 Node.js에서 사용하는 **V8 엔진**은 **세대별(Generational) GC**를 구현합니다. V8 엔진이 코드를 실행하는 방식은 [실행 컨텍스트 완벽 가이드](/posts/javascript-execution-context-guide/)에서 확인할 수 있습니다.

#### 세대별 가설 (Generational Hypothesis)

> "대부분의 객체는 생성 후 금방 사용되지 않게 된다"

이 가설을 바탕으로 V8은 Heap을 두 세대로 나눕니다:

| 세대 | 이름 | 특징 | GC 방식 |
|------|------|------|---------|
| Young Generation | New Space | 새로 생성된 객체 | Minor GC (Scavenge) |
| Old Generation | Old Space | 오래 살아남은 객체 | Major GC (Mark-Sweep-Compact) |

#### Young Generation (New Space)

> **Young Generation (1-8MB)**
>
> | From Space (현재 할당 영역) | To Space (GC 시 복사 대상) |
> |:---------------------------:|:--------------------------:|
> | `[Obj1]` `[Obj2]` `[Obj3]` | (비어있음) |

**Scavenge (Minor GC) 과정:**

1. 새 객체는 From Space에 할당
2. From Space가 가득 차면 Minor GC 시작
3. 살아있는 객체만 To Space로 복사
4. From Space와 To Space 역할 교환
5. 여러 번 살아남은 객체는 Old Generation으로 승격(Promotion)

#### Old Generation (Old Space)

> **Old Generation (수백 MB ~ GB)**
>
> | 저장 대상 | GC 알고리즘 |
> |:---------:|:-----------:|
> | `[장수 객체들]` `[전역 상태]` `[캐시 데이터]` ... | Mark-Sweep-Compact |

**Major GC 과정:**

1. **Mark**: 루트에서 도달 가능한 객체 표시
2. **Sweep**: 표시되지 않은 객체 메모리 해제
3. **Compact**: 메모리 단편화 방지를 위해 살아있는 객체들을 한쪽으로 모음

### 2.5 Incremental과 Concurrent GC

GC는 "Stop-the-World" 이벤트를 발생시킵니다. 즉, GC 실행 중에는 JavaScript 코드 실행이 멈춥니다. JavaScript가 싱글 스레드로 동작하는 원리는 [Event Loop 완벽 가이드](/posts/javascript-event-loop/)에서 자세히 다룹니다. V8은 이 멈춤 시간을 최소화하기 위해 여러 기법을 사용합니다:

| 기법 | 설명 |
|------|------|
| **Incremental Marking** | Mark 작업을 작은 단위로 나눠 JavaScript 실행과 번갈아 수행 |
| **Concurrent Marking** | 별도 스레드에서 Mark 작업 수행 (메인 스레드 블로킹 최소화) |
| **Lazy Sweeping** | Sweep 작업을 필요할 때만 수행 |
| **Parallel Compaction** | 여러 스레드에서 동시에 Compact 수행 |

```
기존 GC (Stop-the-World):
JS 실행 ━━━━━━━━━┃ GC (전체 멈춤) ┃━━━━━━━━━ JS 실행

Incremental GC:
JS ━━┃GC┃━━ JS ━━┃GC┃━━ JS ━━┃GC┃━━ JS (짧은 멈춤 여러 번)

Concurrent GC:
메인 스레드: JS ━━━━━━━━━━━━━━ JS ━━━━━━━━━━━━━━
GC 스레드:   ━━━ GC(Mark) ━━━━━━ GC(Sweep) ━━━
```

---

## Part 3: 메모리 누수 패턴과 해결 방법

메모리 누수(Memory Leak)는 더 이상 필요 없는 메모리가 해제되지 않고 계속 유지되는 현상입니다.

### 3.1 전역 변수 남용

```javascript
// 나쁜 예: 의도치 않은 전역 변수
function processData() {
  // 'use strict'가 없으면 실수로 전역 변수 생성
  leakedData = new Array(1000000).fill('data'); // window.leakedData 생성!
}

processData();
// leakedData는 전역 객체에 붙어서 영원히 GC되지 않음
```

**해결 방법:**

```javascript
'use strict'; // strict mode 사용

function processData() {
  // strict mode에서는 ReferenceError 발생
  // leakedData = []; // Error!

  const localData = new Array(1000000).fill('data');
  // 함수 종료 후 자동으로 GC 대상
  return localData.length;
}
```

### 3.2 해제되지 않은 타이머

```javascript
// 나쁜 예: 클리어하지 않은 setInterval
function startPolling() {
  const data = new Array(10000).fill('polling data');

  setInterval(() => {
    console.log(data.length); // data를 참조하므로 GC 안됨
  }, 1000);
}

startPolling();
// data 배열은 setInterval이 계속 참조하므로 영원히 메모리에 유지
```

**해결 방법:**

```javascript
function startPolling() {
  const data = new Array(10000).fill('polling data');

  const intervalId = setInterval(() => {
    console.log(data.length);
  }, 1000);

  // 명시적으로 정리하는 함수 제공
  return function cleanup() {
    clearInterval(intervalId);
    // intervalId 클리어 후 data는 GC 대상이 됨
  };
}

const stopPolling = startPolling();
// 나중에 필요 없을 때
stopPolling();
```

### 3.3 해제되지 않은 이벤트 리스너

```javascript
// 나쁜 예: 제거하지 않은 이벤트 리스너
class HeavyComponent {
  constructor() {
    this.data = new Array(100000).fill('component data');

    // 전역 이벤트에 리스너 등록
    window.addEventListener('resize', this.handleResize);
  }

  handleResize = () => {
    console.log('리사이즈:', this.data.length);
  }
}

let component = new HeavyComponent();
component = null; // 컴포넌트를 null로 해도 리스너가 여전히 참조
// this.data는 절대 GC되지 않음
```

**해결 방법:**

```javascript
class HeavyComponent {
  constructor() {
    this.data = new Array(100000).fill('component data');
    window.addEventListener('resize', this.handleResize);
  }

  handleResize = () => {
    console.log('리사이즈:', this.data.length);
  }

  // 반드시 정리 메서드 구현
  destroy() {
    window.removeEventListener('resize', this.handleResize);
    this.data = null;
  }
}

let component = new HeavyComponent();
// 사용 후 반드시 정리
component.destroy();
component = null;
```

### 3.4 DOM 참조 유지

```javascript
// 나쁜 예: 분리된 DOM 요소 참조 유지
const elements = {
  button: document.getElementById('myButton'),
  container: document.getElementById('container')
};

function removeButton() {
  document.body.removeChild(elements.container);
  // DOM에서는 제거되었지만 elements.button이 여전히 참조
  // 전체 container와 그 자식들이 메모리에 유지됨
}
```

**해결 방법:**

```javascript
const elements = {
  button: document.getElementById('myButton'),
  container: document.getElementById('container')
};

function removeButton() {
  document.body.removeChild(elements.container);

  // 참조도 함께 정리
  elements.button = null;
  elements.container = null;
}
```

### 3.5 클로저로 인한 메모리 누수

[클로저](/posts/javascript-closure-complete-guide/)는 강력하지만, 부주의하게 사용하면 메모리 누수를 유발할 수 있습니다.

```javascript
// 나쁜 예: 불필요한 변수를 클로저가 캡처
function createHandler() {
  const hugeData = new Array(1000000).fill('huge');
  const smallId = 42;

  // hugeData를 사용하지 않지만, 같은 스코프에 있어서 캡처됨
  return function handler() {
    console.log(smallId); // smallId만 사용
  };
}

const handler = createHandler();
// handler가 hugeData까지 참조하여 메모리 낭비
```

**해결 방법:**

```javascript
function createHandler() {
  const hugeData = processAndGetResult(); // 필요한 처리 후
  const smallId = 42;

  // 클로저가 필요한 값만 캡처하도록 분리
  return createSmallHandler(smallId);
}

function createSmallHandler(id) {
  return function handler() {
    console.log(id); // id만 캡처
  };
}

// 또는 즉시 실행 함수로 스코프 분리
function createHandler() {
  const smallId = 42;

  {
    const hugeData = new Array(1000000).fill('huge');
    // hugeData 처리...
  } // 블록 종료 시 hugeData는 GC 대상

  return function handler() {
    console.log(smallId);
  };
}
```

### 3.6 React 컴포넌트에서의 메모리 누수

React에서 가장 흔한 메모리 누수 패턴들입니다:

```tsx
import { useState, useEffect } from 'react';

// 나쁜 예: cleanup 없는 useEffect
function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    // 비동기 작업 시작
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => {
        // 컴포넌트가 언마운트되어도 setUser 호출 시도
        setUser(data); // Warning: Can't perform state update on unmounted component
      });
  }, [userId]);

  return <div>{user?.name}</div>;
}
```

**해결 방법 1: AbortController 사용**

```tsx
import { useState, useEffect } from 'react';

function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    const abortController = new AbortController();

    fetch(`/api/users/${userId}`, { signal: abortController.signal })
      .then(res => res.json())
      .then(data => {
        setUser(data);
      })
      .catch(error => {
        if (error.name !== 'AbortError') {
          console.error(error);
        }
      });

    // Cleanup: 컴포넌트 언마운트 시 요청 취소
    return () => {
      abortController.abort();
    };
  }, [userId]);

  return <div>{user?.name}</div>;
}
```

**해결 방법 2: 플래그 사용**

```tsx
import { useState, useEffect } from 'react';

function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    let isMounted = true;

    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => {
        // 마운트 상태 확인 후 상태 업데이트
        if (isMounted) {
          setUser(data);
        }
      });

    return () => {
      isMounted = false;
    };
  }, [userId]);

  return <div>{user?.name}</div>;
}
```

**해결 방법 3: 타이머와 이벤트 리스너 정리**

```tsx
import { useState, useEffect, useCallback } from 'react';

function RealTimeData() {
  const [data, setData] = useState([]);

  useEffect(() => {
    // 타이머 설정
    const intervalId = setInterval(() => {
      fetchData().then(setData);
    }, 5000);

    // 이벤트 리스너 등록
    const handleVisibility = () => {
      if (document.hidden) {
        clearInterval(intervalId);
      }
    };
    document.addEventListener('visibilitychange', handleVisibility);

    // Cleanup: 모든 리소스 정리
    return () => {
      clearInterval(intervalId);
      document.removeEventListener('visibilitychange', handleVisibility);
    };
  }, []);

  return <DataList items={data} />;
}
```

---

## Part 4: WeakMap과 WeakSet 활용

### 4.1 약한 참조(Weak Reference) 개념

일반 참조(강한 참조)는 객체를 가비지 컬렉션에서 보호합니다. 반면, **약한 참조(Weak Reference)**는 가비지 컬렉션을 방해하지 않습니다.

```javascript
// 강한 참조 (Map)
const strongMap = new Map();
let obj = { data: 'important' };
strongMap.set(obj, 'metadata');

obj = null;
// strongMap이 여전히 { data: 'important' } 객체를 참조
// → GC되지 않음

// 약한 참조 (WeakMap)
const weakMap = new WeakMap();
let obj2 = { data: 'important' };
weakMap.set(obj2, 'metadata');

obj2 = null;
// weakMap의 참조는 약한 참조
// → 다른 참조가 없으면 GC 대상
```

### 4.2 WeakMap 활용 패턴

#### 패턴 1: 프라이빗 데이터 저장

```javascript
// WeakMap을 활용한 프라이빗 데이터
const privateData = new WeakMap();

class User {
  constructor(name, password) {
    this.name = name;
    // 비밀번호는 WeakMap에 저장 (외부에서 접근 불가)
    privateData.set(this, { password });
  }

  checkPassword(input) {
    return privateData.get(this).password === input;
  }
}

const user = new User('홍길동', 'secret123');
console.log(user.name);                    // '홍길동'
console.log(user.password);                // undefined (접근 불가)
console.log(user.checkPassword('secret123')); // true

// user 객체가 GC되면 WeakMap의 엔트리도 자동 정리
```

#### 패턴 2: DOM 요소에 메타데이터 연결

```javascript
const elementMetadata = new WeakMap();

function trackElement(element, metadata) {
  elementMetadata.set(element, {
    ...metadata,
    trackedAt: Date.now()
  });
}

function getElementInfo(element) {
  return elementMetadata.get(element);
}

// 사용 예시
const button = document.getElementById('myButton');
trackElement(button, { clicks: 0, category: 'action' });

button.addEventListener('click', () => {
  const info = getElementInfo(button);
  info.clicks++;
  console.log(`클릭 횟수: ${info.clicks}`);
});

// button이 DOM에서 제거되면 WeakMap 엔트리도 자동 정리
// 메모리 누수 방지!
```

#### 패턴 3: 캐싱

```javascript
const cache = new WeakMap();

function expensiveOperation(obj) {
  // 캐시 확인
  if (cache.has(obj)) {
    console.log('캐시 히트!');
    return cache.get(obj);
  }

  // 비용이 큰 연산 수행
  console.log('연산 수행...');
  const result = heavyComputation(obj);

  // 결과 캐싱
  cache.set(obj, result);
  return result;
}

function heavyComputation(obj) {
  // 무거운 연산 시뮬레이션
  return Object.keys(obj).reduce((sum, key) => sum + obj[key], 0);
}

const data = { a: 1, b: 2, c: 3 };
expensiveOperation(data); // '연산 수행...' → 6
expensiveOperation(data); // '캐시 히트!' → 6

// data가 GC되면 캐시도 자동 정리
```

### 4.3 WeakSet 활용 패턴

```javascript
// 이미 처리된 객체 추적
const processedObjects = new WeakSet();

function processOnce(obj) {
  if (processedObjects.has(obj)) {
    console.log('이미 처리됨, 스킵');
    return;
  }

  // 처리 로직
  console.log('처리 중:', obj);

  // 처리 완료 표시
  processedObjects.add(obj);
}

const item1 = { id: 1 };
const item2 = { id: 2 };

processOnce(item1); // '처리 중:' { id: 1 }
processOnce(item1); // '이미 처리됨, 스킵'
processOnce(item2); // '처리 중:' { id: 2 }

// item1, item2가 GC되면 WeakSet에서도 자동 제거
```

### 4.4 WeakMap/WeakSet 제한사항

| 특성 | Map/Set | WeakMap/WeakSet |
|------|---------|-----------------|
| 키 타입 | 모든 값 | 객체만 가능 |
| 열거 가능 | `keys()`, `values()`, `forEach()` | 불가능 |
| `size` 속성 | 있음 | 없음 |
| GC 대상 | 키가 참조되는 한 유지 | 다른 참조 없으면 GC |

```javascript
const weakMap = new WeakMap();

// 원시 값은 키로 사용 불가
// weakMap.set('string', value); // TypeError!
// weakMap.set(123, value);      // TypeError!

// 객체만 키로 사용 가능
weakMap.set({}, 'value');         // OK
weakMap.set(function() {}, 'fn'); // OK
weakMap.set([], 'array');         // OK
```

---

## Part 5: Chrome DevTools로 메모리 디버깅

### 5.1 Memory 탭 개요

Chrome DevTools의 **Memory** 탭은 메모리 문제를 진단하는 강력한 도구입니다.

**Memory 탭 열기:**
1. Chrome에서 F12 또는 Cmd+Option+I(Mac) / Ctrl+Shift+I(Windows)
2. **Memory** 탭 선택

**세 가지 프로파일링 유형:**

| 유형 | 용도 | 언제 사용? |
|------|------|-----------|
| **Heap Snapshot** | 특정 시점의 메모리 상태 캡처 | 메모리 누수 객체 식별 |
| **Allocation instrumentation** | 시간에 따른 메모리 할당 기록 | 누수가 발생하는 코드 위치 추적 |
| **Allocation sampling** | 저비용 샘플링 방식 기록 | 장시간 모니터링 |

### 5.2 Heap Snapshot 분석

#### 단계 1: Heap Snapshot 캡처

```javascript
// 테스트용 메모리 누수 코드
const leaks = [];

function createLeak() {
  const bigArray = new Array(100000).fill('leak');
  leaks.push(bigArray); // 전역 배열에 계속 추가
}

// 누수 발생
setInterval(createLeak, 1000);
```

1. Memory 탭에서 **Heap snapshot** 선택
2. **Take snapshot** 클릭
3. 몇 초 후 다시 스냅샷 캡처
4. 두 스냅샷 비교

#### 단계 2: 스냅샷 비교 (Comparison View)

1. 두 번째 스냅샷 선택
2. 드롭다운에서 **Comparison** 선택
3. **# Delta** 열 확인 (새로 생성된 객체 수)
4. **Size Delta** 열 확인 (증가한 메모리 크기)

#### 단계 3: Retainers 분석

누수 의심 객체를 선택하고 **Retainers** 패널에서 해당 객체를 참조하는 체인을 확인합니다:

```
Array @123456
└── leaks in Window
    └── (array) in createLeak
        └── createLeak in (closure)
```

### 5.3 Allocation Instrumentation

시간에 따른 메모리 할당을 추적합니다:

1. **Allocation instrumentation on timeline** 선택
2. **Start** 클릭
3. 의심되는 작업 수행
4. **Stop** 클릭
5. 타임라인에서 파란색 막대(할당됨) / 회색 막대(해제됨) 확인

```javascript
// 테스트: 정상적인 할당과 해제
function normalOperation() {
  const temp = new Array(10000).fill('temp');
  // 함수 종료 후 GC됨
  return temp.length;
}

// 테스트: 메모리 누수
const permanentStorage = [];
function leakyOperation() {
  const data = new Array(10000).fill('leak');
  permanentStorage.push(data); // 해제되지 않음
}
```

### 5.4 Performance 탭에서 메모리 모니터링

1. **Performance** 탭 열기
2. **Memory** 체크박스 활성화
3. 레코딩 시작
4. 작업 수행
5. 레코딩 중지

**확인 사항:**
- **JS Heap**: JavaScript 힙 사용량 (계속 증가하면 누수 의심)
- **Nodes**: DOM 노드 수 (분리된 DOM 노드 누수 확인)
- **Listeners**: 이벤트 리스너 수

### 5.5 실전 메모리 누수 디버깅 예제

```javascript
// 메모리 누수가 있는 코드
class DataManager {
  constructor() {
    this.cache = new Map();
    this.listeners = [];

    // 문제 1: 이벤트 리스너 누수
    window.addEventListener('scroll', this.handleScroll);
  }

  handleScroll = () => {
    console.log('스크롤 중...');
  }

  // 문제 2: 캐시 무한 증가
  fetchData(id) {
    const data = { id, timestamp: Date.now(), large: new Array(10000).fill(id) };
    this.cache.set(id, data);
    return data;
  }

  // 문제 3: 리스너 배열 정리 안됨
  addListener(callback) {
    this.listeners.push(callback);
  }
}

// 사용
const manager = new DataManager();
for (let i = 0; i < 1000; i++) {
  manager.fetchData(i); // 캐시 계속 증가
}
```

**수정된 코드:**

```javascript
class DataManager {
  constructor() {
    // 해결 2: LRU 캐시 또는 크기 제한
    this.cache = new Map();
    this.maxCacheSize = 100;
    this.listeners = new Set(); // Set으로 중복 방지

    // 바인딩된 핸들러 저장 (제거 시 필요)
    this.boundHandleScroll = this.handleScroll.bind(this);
    window.addEventListener('scroll', this.boundHandleScroll);
  }

  handleScroll() {
    console.log('스크롤 중...');
  }

  fetchData(id) {
    // 캐시 크기 제한
    if (this.cache.size >= this.maxCacheSize) {
      const firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey);
    }

    const data = { id, timestamp: Date.now(), large: new Array(10000).fill(id) };
    this.cache.set(id, data);
    return data;
  }

  addListener(callback) {
    this.listeners.add(callback);
    // 리스너 제거 함수 반환
    return () => this.listeners.delete(callback);
  }

  // 해결 1: 정리 메서드 구현
  destroy() {
    window.removeEventListener('scroll', this.boundHandleScroll);
    this.cache.clear();
    this.listeners.clear();
  }
}
```

---

## Part 6: 실전 최적화 팁

### 6.1 대용량 데이터 처리

```javascript
// 나쁜 예: 전체 데이터를 메모리에 유지
function processAllData(hugeArray) {
  const results = hugeArray.map(item => transform(item));
  const filtered = results.filter(item => isValid(item));
  const sorted = filtered.sort((a, b) => a.value - b.value);
  return sorted;
  // 중간 결과(results, filtered)가 모두 메모리에 존재
}

// 좋은 예: 제너레이터로 지연 처리
function* processDataLazily(hugeArray) {
  for (const item of hugeArray) {
    const transformed = transform(item);
    if (isValid(transformed)) {
      yield transformed;
    }
  }
}

// 필요한 만큼만 처리
const processor = processDataLazily(hugeArray);
const first10 = [];
for (let i = 0; i < 10; i++) {
  const result = processor.next();
  if (result.done) break;
  first10.push(result.value);
}
```

### 6.2 객체 풀링(Object Pooling)

객체를 반복적으로 생성/삭제하는 경우, 객체 풀을 사용하면 GC 부하를 줄일 수 있습니다.

```javascript
class ObjectPool {
  constructor(createFn, resetFn, initialSize = 10) {
    this.createFn = createFn;
    this.resetFn = resetFn;
    this.pool = [];

    // 초기 객체 생성
    for (let i = 0; i < initialSize; i++) {
      this.pool.push(this.createFn());
    }
  }

  acquire() {
    // 풀에서 객체 꺼내기 (없으면 새로 생성)
    return this.pool.length > 0
      ? this.pool.pop()
      : this.createFn();
  }

  release(obj) {
    // 객체 초기화 후 풀에 반환
    this.resetFn(obj);
    this.pool.push(obj);
  }
}

// 사용 예: 파티클 시스템
const particlePool = new ObjectPool(
  // 생성 함수
  () => ({ x: 0, y: 0, vx: 0, vy: 0, life: 0 }),
  // 리셋 함수
  (p) => { p.x = 0; p.y = 0; p.vx = 0; p.vy = 0; p.life = 0; },
  100
);

function createParticle(x, y) {
  const p = particlePool.acquire();
  p.x = x;
  p.y = y;
  p.vx = Math.random() * 2 - 1;
  p.vy = Math.random() * 2 - 1;
  p.life = 100;
  return p;
}

function destroyParticle(p) {
  particlePool.release(p); // GC 없이 재사용
}
```

### 6.3 메모리 효율적인 데이터 구조

```javascript
// 나쁜 예: 각 객체가 문자열 키를 가짐
const users = [
  { id: 1, firstName: 'John', lastName: 'Doe', email: 'john@example.com' },
  { id: 2, firstName: 'Jane', lastName: 'Doe', email: 'jane@example.com' },
  // ... 수천 개
];

// 좋은 예: 컬럼 기반 저장 (더 나은 메모리 압축)
const usersColumnar = {
  ids: [1, 2, /* ... */],
  firstNames: ['John', 'Jane', /* ... */],
  lastNames: ['Doe', 'Doe', /* ... */],
  emails: ['john@example.com', 'jane@example.com', /* ... */]
};

// TypedArray 활용 (숫자 데이터에 최적)
const ids = new Uint32Array([1, 2, 3, 4, 5]);
const scores = new Float64Array([95.5, 88.3, 92.1, 85.7, 91.2]);
```

### 6.4 청크 단위 처리

```javascript
// 대용량 작업을 청크로 나눠 UI 블로킹 방지
async function processInChunks(items, chunkSize = 100, processFn) {
  const results = [];

  for (let i = 0; i < items.length; i += chunkSize) {
    const chunk = items.slice(i, i + chunkSize);

    // 청크 처리
    const chunkResults = chunk.map(processFn);
    results.push(...chunkResults);

    // 다음 프레임까지 대기 (UI 업데이트 허용)
    await new Promise(resolve => setTimeout(resolve, 0));

    // 진행 상황 표시
    console.log(`처리 중: ${Math.min(i + chunkSize, items.length)} / ${items.length}`);
  }

  return results;
}

// 사용
const hugeArray = new Array(100000).fill(null).map((_, i) => i);
processInChunks(hugeArray, 1000, x => x * 2)
  .then(results => console.log('완료!', results.length));
```

---

## 메모리 관리 체크리스트

### 개발 시 확인 사항

- [ ] `setInterval`, `setTimeout` 사용 시 cleanup 구현
- [ ] 이벤트 리스너 등록 시 제거 로직 구현
- [ ] 클로저가 불필요한 큰 객체를 캡처하지 않는지 확인
- [ ] React `useEffect`에서 cleanup 함수 반환
- [ ] 전역 변수 사용 최소화

### 디버깅 시 확인 사항

- [ ] Performance 탭에서 메모리 그래프가 계속 증가하는지 확인
- [ ] Heap Snapshot으로 분리된 DOM 노드 확인
- [ ] Allocation Timeline으로 누수 발생 위치 추적
- [ ] Retainers에서 예상치 못한 참조 체인 확인

### 최적화 고려 사항

- [ ] 대용량 데이터는 청크 단위로 처리
- [ ] 자주 생성/삭제되는 객체는 풀링 고려
- [ ] 캐시 크기 제한 설정
- [ ] WeakMap/WeakSet 활용 가능 여부 검토

---

## 마무리

JavaScript의 자동 메모리 관리는 개발자의 부담을 줄여주지만, 메모리 누수는 여전히 발생할 수 있습니다. 핵심 내용을 정리하면:

1. **Stack vs Heap**: 원시 값은 Stack, 객체는 Heap에 저장됩니다.
2. **Mark-and-Sweep**: 현대 JavaScript 엔진의 핵심 GC 알고리즘입니다.
3. **메모리 누수 패턴**: 타이머, 이벤트 리스너, 클로저, DOM 참조를 주의하세요.
4. **WeakMap/WeakSet**: 메타데이터 연결과 캐싱에 활용하면 누수를 방지할 수 있습니다.
5. **DevTools 활용**: Heap Snapshot과 Allocation Timeline으로 문제를 진단하세요.

메모리 관리에 대한 이해는 대규모 애플리케이션의 성능 최적화에 필수적입니다. 이 가이드가 메모리 문제를 디버깅하고 더 효율적인 코드를 작성하는 데 도움이 되길 바랍니다.

---

## 참고 자료

- [MDN - Memory Management](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Memory_management)
- [V8 Blog - Trash Talk: the Orinoco Garbage Collector](https://v8.dev/blog/trash-talk)
- [Chrome DevTools - Memory Panel](https://developer.chrome.com/docs/devtools/memory-problems/)
- [JavaScript.info - Garbage Collection](https://javascript.info/garbage-collection)
- [V8 Blog - Concurrent Marking in V8](https://v8.dev/blog/concurrent-marking)
