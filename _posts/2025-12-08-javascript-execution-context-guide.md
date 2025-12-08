---
title: "JavaScript 실행 컨텍스트(Execution Context) 완벽 가이드 - 코드 실행의 원리"
description: "JavaScript 엔진이 코드를 실행하는 방식을 실행 컨텍스트를 통해 완벽히 이해합니다. Variable Environment, Lexical Environment, 호이스팅, 스코프 체인의 원리를 시각적 다이어그램과 단계별 예제로 명확하게 설명합니다."
date: 2025-12-08 14:00:00 +0900
categories: [Frontend, JavaScript]
tags: [JavaScript, 실행 컨텍스트, Execution Context, 호이스팅, 스코프 체인, Lexical Environment, Variable Environment, TDZ, 클로저, Call Stack]
---

## 개요

JavaScript 코드가 실행될 때, 엔진 내부에서는 어떤 일이 일어날까요? `this`가 왜 그렇게 바인딩되는지, 호이스팅이 왜 발생하는지, 클로저가 어떻게 외부 변수를 기억하는지 - 이 모든 현상의 근본 원리가 바로 **실행 컨텍스트(Execution Context)**입니다.

이 글에서는 JavaScript 엔진이 코드를 실행할 때 내부적으로 생성하는 실행 컨텍스트의 구조와 동작 원리를 상세히 설명합니다. 실행 컨텍스트를 이해하면 JavaScript의 핵심 개념들이 왜 그렇게 동작하는지 명확하게 알 수 있습니다.

### 학습 목표

- 실행 컨텍스트의 정의와 종류 이해
- Variable Environment와 Lexical Environment의 차이점 파악
- 호이스팅이 발생하는 원리를 실행 컨텍스트 관점에서 이해
- 스코프 체인이 형성되는 메커니즘 학습
- 실행 컨텍스트의 생성 단계와 실행 단계 구분

### 사전 지식

- JavaScript 기본 문법
- 스코프 개념 ([JavaScript Scope 완벽 가이드](/posts/javascript-scope/) 참고)
- Call Stack 개념 ([JavaScript Call Stack 완벽 가이드](/posts/javascript-call-stack/) 참고)

---

## 실행 컨텍스트란?

### 정의

**실행 컨텍스트(Execution Context)**는 JavaScript 코드가 실행되는 환경을 추상화한 개념입니다. 코드가 실행되기 위해 필요한 모든 정보를 담고 있는 객체라고 생각할 수 있습니다.

> ECMAScript 명세: "An execution context is a specification device that is used to track the runtime evaluation of code by an ECMAScript implementation."

쉽게 말해, 실행 컨텍스트는 **"코드가 실행되는 데 필요한 환경 정보들을 모아놓은 객체"**입니다.

### 실행 컨텍스트가 담고 있는 정보

실행 컨텍스트는 다음과 같은 정보를 포함합니다:

| 구성 요소 | 설명 |
|-----------|------|
| **Variable Environment** | 변수, 함수 선언, 매개변수 정보 저장 |
| **Lexical Environment** | 현재 컨텍스트 내의 식별자 정보와 외부 환경 참조 |
| **ThisBinding** | `this` 키워드가 참조하는 객체 |

### 실행 컨텍스트와 Call Stack의 관계

실행 컨텍스트는 Call Stack에서 관리됩니다. 함수가 호출되면 해당 함수의 실행 컨텍스트가 생성되어 Call Stack에 push되고, 함수가 종료되면 pop됩니다.

```javascript
function outer() {
  console.log('outer 시작');
  inner();
  console.log('outer 끝');
}

function inner() {
  console.log('inner 실행');
}

outer();
```

**Call Stack 변화:**

```
// 1. 전역 실행 컨텍스트
[Global Execution Context]

// 2. outer() 호출
[outer Execution Context]
[Global Execution Context]

// 3. inner() 호출
[inner Execution Context]
[outer Execution Context]
[Global Execution Context]

// 4. inner() 종료
[outer Execution Context]
[Global Execution Context]

// 5. outer() 종료
[Global Execution Context]
```

---

## 실행 컨텍스트의 종류

JavaScript에서는 세 가지 유형의 실행 컨텍스트가 있습니다.

### 1. 전역 실행 컨텍스트 (Global Execution Context)

JavaScript 코드가 처음 실행될 때 생성됩니다. 프로그램에서 **단 하나만** 존재합니다.

**특징:**
- 전역 객체(브라우저: `window`, Node.js: `global`) 생성
- `this`를 전역 객체에 바인딩
- 전역 변수와 함수를 저장

```javascript
// 전역 컨텍스트에서 실행
var globalVar = '전역 변수';

function globalFunc() {
  console.log('전역 함수');
}

console.log(this === window); // true (브라우저 환경)
```

### 2. 함수 실행 컨텍스트 (Function Execution Context)

함수가 **호출될 때마다** 새로운 실행 컨텍스트가 생성됩니다. 같은 함수를 여러 번 호출하면 그만큼 실행 컨텍스트가 생성됩니다.

```javascript
function greet(name) {
  const message = `안녕하세요, ${name}님!`;
  console.log(message);
}

greet('철수'); // 새로운 실행 컨텍스트 생성
greet('영희'); // 또 다른 새로운 실행 컨텍스트 생성
```

### 3. Eval 실행 컨텍스트 (Eval Execution Context)

`eval()` 함수 내부에서 실행되는 코드를 위한 실행 컨텍스트입니다.

> `eval()` 함수는 보안과 성능 문제로 사용을 권장하지 않습니다. 현대 JavaScript에서는 거의 사용되지 않습니다.
{: .prompt-warning }

---

## 실행 컨텍스트의 구성 요소

ES5 이후의 실행 컨텍스트는 다음 세 가지 컴포넌트로 구성됩니다.

### 1. Variable Environment

**Variable Environment**는 실행 컨텍스트 생성 시점의 변수와 함수 선언 정보를 저장합니다.

```javascript
function example() {
  var x = 10;
  let y = 20;
  const z = 30;

  function inner() {}
}
```

Variable Environment에 저장되는 정보:
- `var`로 선언된 변수: `x` (초기값: `undefined`)
- 함수 선언문: `inner` (함수 객체 참조)

### 2. Lexical Environment

**Lexical Environment**는 식별자와 해당 값의 매핑 정보, 그리고 **외부 환경에 대한 참조**를 포함합니다.

Lexical Environment는 두 가지 컴포넌트로 구성됩니다:

#### Environment Record (환경 레코드)

현재 컨텍스트 내에서 선언된 식별자들의 정보를 저장합니다.

**Environment Record의 종류:**

| 종류 | 설명 | 저장 내용 |
|------|------|----------|
| **Declarative Environment Record** | 함수, 변수, catch 절 등 | `let`, `const`, `class`, 함수 선언 |
| **Object Environment Record** | `with`문, 전역 컨텍스트 | 전역 객체의 속성들 |
| **Global Environment Record** | 전역 컨텍스트 전용 | 전역 변수, 전역 함수 |

#### Outer Lexical Environment Reference (외부 렉시컬 환경 참조)

상위 스코프의 Lexical Environment를 참조합니다. 이 참조를 통해 **스코프 체인**이 형성됩니다.

```javascript
const globalVar = '전역';

function outer() {
  const outerVar = '외부';

  function inner() {
    const innerVar = '내부';
    console.log(innerVar);  // 현재 Environment Record에서 찾음
    console.log(outerVar);  // outer의 Lexical Environment에서 찾음
    console.log(globalVar); // 전역 Lexical Environment에서 찾음
  }

  inner();
}

outer();
```

**Lexical Environment 구조:**

```
inner의 Lexical Environment
├── Environment Record: { innerVar: '내부' }
└── outer: outer의 Lexical Environment 참조

outer의 Lexical Environment
├── Environment Record: { outerVar: '외부', inner: function }
└── outer: 전역 Lexical Environment 참조

전역 Lexical Environment
├── Environment Record: { globalVar: '전역', outer: function }
└── outer: null
```

### 3. ThisBinding

현재 실행 컨텍스트에서 `this` 키워드가 참조하는 객체를 결정합니다.

> `this` 바인딩의 상세한 규칙은 [JavaScript this 바인딩 완벽 가이드](/posts/javascript-this-binding-guide/)를 참고하세요.
{: .prompt-tip }

---

## 실행 컨텍스트의 생성 과정

실행 컨텍스트는 **두 단계**를 거쳐 생성됩니다.

### 1. 생성 단계 (Creation Phase)

코드가 실행되기 전, 실행 컨텍스트가 생성되는 단계입니다.

**생성 단계에서 일어나는 일:**

1. **Lexical Environment 생성**
2. **Variable Environment 생성**
3. **ThisBinding 결정**

#### 변수와 함수의 초기화

생성 단계에서 변수와 함수는 다르게 처리됩니다:

| 선언 방식 | 생성 단계 초기화 | 접근 가능 시점 |
|-----------|-----------------|---------------|
| `var` | `undefined`로 초기화 | 선언 전에도 접근 가능 (호이스팅) |
| `let`, `const` | 초기화되지 않음 (TDZ) | 선언문 이후에만 접근 가능 |
| 함수 선언문 | 함수 객체로 완전히 초기화 | 선언 전에도 호출 가능 |
| 함수 표현식 | 변수 선언 방식에 따름 | 선언 방식에 따름 |

```javascript
console.log(a); // undefined (var 호이스팅)
console.log(b); // ReferenceError: Cannot access 'b' before initialization
console.log(c); // ReferenceError: Cannot access 'c' before initialization
greet();        // "Hello!" (함수 선언문 호이스팅)

var a = 10;
let b = 20;
const c = 30;

function greet() {
  console.log('Hello!');
}
```

**생성 단계 후 실행 컨텍스트 상태:**

```javascript
// 의사 코드로 표현
ExecutionContext = {
  LexicalEnvironment: {
    EnvironmentRecord: {
      b: <uninitialized>,  // TDZ
      c: <uninitialized>,  // TDZ
      greet: function object
    },
    outer: <Global Environment>
  },
  VariableEnvironment: {
    EnvironmentRecord: {
      a: undefined  // var는 undefined로 초기화
    },
    outer: <Global Environment>
  },
  ThisBinding: <Global Object>
}
```

### 2. 실행 단계 (Execution Phase)

코드가 한 줄씩 실행되면서 변수에 실제 값이 할당됩니다.

```javascript
var a = 10;  // a에 10 할당
let b = 20;  // b에 20 할당 (TDZ 종료)
const c = 30; // c에 30 할당 (TDZ 종료)
```

**실행 단계 후 실행 컨텍스트 상태:**

```javascript
// 의사 코드로 표현
ExecutionContext = {
  LexicalEnvironment: {
    EnvironmentRecord: {
      b: 20,
      c: 30,
      greet: function object
    },
    outer: <Global Environment>
  },
  VariableEnvironment: {
    EnvironmentRecord: {
      a: 10
    },
    outer: <Global Environment>
  },
  ThisBinding: <Global Object>
}
```

---

## 호이스팅의 원리

### 호이스팅이란?

**호이스팅(Hoisting)**은 변수와 함수 선언이 코드의 최상단으로 끌어올려지는 것처럼 동작하는 현상입니다. 이 현상은 실행 컨텍스트의 **생성 단계**에서 발생합니다.

> 실제로 코드가 이동하는 것이 아니라, 실행 컨텍스트 생성 시 변수와 함수 선언 정보가 먼저 Environment Record에 기록되기 때문에 발생하는 현상입니다.
{: .prompt-info }

### var 호이스팅

`var`로 선언된 변수는 생성 단계에서 `undefined`로 초기화됩니다.

```javascript
console.log(name); // undefined
var name = '홍길동';
console.log(name); // '홍길동'
```

**실행 컨텍스트 관점:**

```javascript
// 1. 생성 단계
VariableEnvironment = {
  EnvironmentRecord: {
    name: undefined  // var 선언 발견, undefined로 초기화
  }
}

// 2. 실행 단계
// console.log(name) 실행 -> undefined 출력
// name = '홍길동' 할당
// console.log(name) 실행 -> '홍길동' 출력
```

### let과 const의 TDZ (Temporal Dead Zone)

`let`과 `const`도 호이스팅되지만, 생성 단계에서 **초기화되지 않습니다**. 선언문에 도달하기 전까지 **일시적 사각지대(TDZ)**에 있습니다.

```javascript
// TDZ 시작
console.log(age); // ReferenceError: Cannot access 'age' before initialization
let age = 25;     // TDZ 종료
console.log(age); // 25
```

**TDZ가 존재하는 이유:**

1. **버그 조기 발견**: 선언 전 사용을 명확한 에러로 알려줌
2. **코드 가독성**: 변수 선언과 사용 순서가 명확해짐
3. **const 의미 보장**: `const`는 선언과 동시에 초기화되어야 함

### 함수 선언문 호이스팅

함수 선언문은 생성 단계에서 **완전히 초기화**됩니다.

```javascript
greet(); // "Hello, World!"

function greet() {
  console.log('Hello, World!');
}
```

**실행 컨텍스트 관점:**

```javascript
// 1. 생성 단계
LexicalEnvironment = {
  EnvironmentRecord: {
    greet: <함수 객체 참조>  // 함수 선언문은 완전히 초기화
  }
}

// 2. 실행 단계
// greet() 호출 -> 함수 실행
```

### 함수 표현식은 호이스팅되지 않는다

함수 표현식은 변수 선언 방식에 따라 호이스팅이 결정됩니다.

```javascript
// var로 선언한 함수 표현식
console.log(func1); // undefined
// func1(); // TypeError: func1 is not a function

var func1 = function() {
  console.log('func1');
};

// let으로 선언한 함수 표현식
// console.log(func2); // ReferenceError
// func2(); // ReferenceError

let func2 = function() {
  console.log('func2');
};
```

---

## 스코프 체인의 형성

### 스코프 체인이란?

**스코프 체인(Scope Chain)**은 현재 실행 컨텍스트에서 변수를 찾을 때, 현재 Lexical Environment에서 시작하여 **outer 참조**를 따라 상위 Environment를 탐색하는 체인입니다.

### 스코프 체인 형성 원리

각 Lexical Environment는 생성될 때 자신의 **외부 렉시컬 환경(outer)**을 기록합니다. 이 outer 참조가 연결되어 스코프 체인을 형성합니다.

```javascript
const global = '전역';

function outer() {
  const outerVar = '외부';

  function middle() {
    const middleVar = '중간';

    function inner() {
      const innerVar = '내부';
      console.log(innerVar);  // '내부'
      console.log(middleVar); // '중간'
      console.log(outerVar);  // '외부'
      console.log(global);    // '전역'
    }

    inner();
  }

  middle();
}

outer();
```

**스코프 체인 구조:**

```
inner Lexical Environment
├── Environment Record: { innerVar: '내부' }
└── outer ──────────────────────────────────────┐
                                                 ▼
middle Lexical Environment
├── Environment Record: { middleVar: '중간', inner: function }
└── outer ──────────────────────────────────────┐
                                                 ▼
outer Lexical Environment
├── Environment Record: { outerVar: '외부', middle: function }
└── outer ──────────────────────────────────────┐
                                                 ▼
Global Lexical Environment
├── Environment Record: { global: '전역', outer: function }
└── outer: null (체인의 끝)
```

### 변수 검색 과정

`inner` 함수에서 `outerVar`를 참조할 때:

1. **inner의 Environment Record** 검색 -> 없음
2. **outer 참조**를 따라 middle의 Environment Record 검색 -> 없음
3. **outer 참조**를 따라 outer의 Environment Record 검색 -> `outerVar` 발견!
4. 값 반환

```javascript
function searchVariable(name, lexicalEnvironment) {
  // 현재 Environment Record에서 검색
  if (lexicalEnvironment.environmentRecord.has(name)) {
    return lexicalEnvironment.environmentRecord.get(name);
  }

  // outer가 null이면 변수를 찾지 못함
  if (lexicalEnvironment.outer === null) {
    throw new ReferenceError(`${name} is not defined`);
  }

  // outer를 따라 상위 환경에서 검색 (재귀)
  return searchVariable(name, lexicalEnvironment.outer);
}
```

### 클로저와 스코프 체인

클로저가 외부 변수를 기억할 수 있는 이유도 스코프 체인 때문입니다.

```javascript
function createCounter() {
  let count = 0; // createCounter의 Lexical Environment에 저장

  return function increment() {
    count++; // outer 참조를 통해 count에 접근
    return count;
  };
}

const counter = createCounter();
console.log(counter()); // 1
console.log(counter()); // 2
```

**클로저 형성 과정:**

1. `createCounter()` 호출 -> 새로운 실행 컨텍스트 생성
2. `count` 변수가 `createCounter`의 Lexical Environment에 저장
3. `increment` 함수 생성 시, outer 참조로 `createCounter`의 Lexical Environment 연결
4. `createCounter()` 종료 후에도 `increment`의 outer 참조가 유지됨
5. `counter()`호출 시 outer 참조를 통해 `count`에 접근 가능

> 클로저에 대한 자세한 설명은 [JavaScript 클로저(Closure) 완벽 가이드](/posts/javascript-closure-complete-guide/)를 참고하세요.
{: .prompt-tip }

---

## 실전 예제: 코드 실행 과정 시각화

### 예제 1: 전역 코드와 함수 호출

```javascript
var globalVar = '전역';
let globalLet = '전역 let';

function outer(param) {
  var outerVar = '외부';
  let outerLet = '외부 let';

  function inner() {
    var innerVar = '내부';
    console.log(innerVar);    // '내부'
    console.log(outerVar);    // '외부'
    console.log(param);       // '매개변수'
    console.log(globalVar);   // '전역'
    console.log(globalLet);   // '전역 let'
  }

  inner();
}

outer('매개변수');
```

**단계별 실행 컨텍스트 변화:**

#### 1단계: 전역 실행 컨텍스트 생성 (생성 단계)

```javascript
GlobalExecutionContext = {
  LexicalEnvironment: {
    EnvironmentRecord: {
      globalLet: <uninitialized>,
      outer: <function object>
    },
    outer: null
  },
  VariableEnvironment: {
    EnvironmentRecord: {
      globalVar: undefined
    },
    outer: null
  },
  ThisBinding: window // 브라우저 환경
}
```

#### 2단계: 전역 코드 실행 (실행 단계)

```javascript
GlobalExecutionContext = {
  LexicalEnvironment: {
    EnvironmentRecord: {
      globalLet: '전역 let',
      outer: <function object>
    },
    outer: null
  },
  VariableEnvironment: {
    EnvironmentRecord: {
      globalVar: '전역'
    },
    outer: null
  },
  ThisBinding: window
}
```

#### 3단계: outer() 호출 -> outer 실행 컨텍스트 생성 (생성 단계)

```javascript
outerExecutionContext = {
  LexicalEnvironment: {
    EnvironmentRecord: {
      outerLet: <uninitialized>,
      inner: <function object>,
      param: '매개변수'  // 매개변수도 여기에 저장
    },
    outer: GlobalExecutionContext.LexicalEnvironment
  },
  VariableEnvironment: {
    EnvironmentRecord: {
      outerVar: undefined
    },
    outer: GlobalExecutionContext.VariableEnvironment
  },
  ThisBinding: window // 일반 함수 호출이므로
}
```

#### 4단계: outer 코드 실행 (실행 단계)

```javascript
outerExecutionContext = {
  LexicalEnvironment: {
    EnvironmentRecord: {
      outerLet: '외부 let',
      inner: <function object>,
      param: '매개변수'
    },
    outer: GlobalExecutionContext.LexicalEnvironment
  },
  VariableEnvironment: {
    EnvironmentRecord: {
      outerVar: '외부'
    },
    outer: GlobalExecutionContext.VariableEnvironment
  },
  ThisBinding: window
}
```

#### 5단계: inner() 호출 -> inner 실행 컨텍스트 생성

```javascript
innerExecutionContext = {
  LexicalEnvironment: {
    EnvironmentRecord: {},
    outer: outerExecutionContext.LexicalEnvironment
  },
  VariableEnvironment: {
    EnvironmentRecord: {
      innerVar: '내부'
    },
    outer: outerExecutionContext.VariableEnvironment
  },
  ThisBinding: window
}
```

### 예제 2: 호이스팅과 TDZ

```javascript
function hoistingExample() {
  console.log(a); // undefined
  console.log(b); // ReferenceError
  console.log(c); // ReferenceError

  var a = 1;
  let b = 2;
  const c = 3;

  console.log(a); // 1
  console.log(b); // 2
  console.log(c); // 3
}
```

**실행 컨텍스트 생성 단계:**

```javascript
hoistingExampleExecutionContext = {
  LexicalEnvironment: {
    EnvironmentRecord: {
      b: <uninitialized>,  // TDZ - 접근 시 ReferenceError
      c: <uninitialized>   // TDZ - 접근 시 ReferenceError
    },
    outer: GlobalLexicalEnvironment
  },
  VariableEnvironment: {
    EnvironmentRecord: {
      a: undefined  // var는 undefined로 초기화
    },
    outer: GlobalVariableEnvironment
  }
}
```

### 예제 3: 블록 스코프

```javascript
function blockScopeExample() {
  var x = 1;
  let y = 2;

  if (true) {
    var x = 10;  // 같은 x (함수 스코프)
    let y = 20;  // 새로운 y (블록 스코프)
    let z = 30;  // 블록 내부에서만 유효

    console.log(x); // 10
    console.log(y); // 20
    console.log(z); // 30
  }

  console.log(x); // 10 (var는 블록 스코프 무시)
  console.log(y); // 2 (let은 블록 스코프)
  // console.log(z); // ReferenceError
}
```

**블록 스코프의 Lexical Environment:**

블록문(`if`, `for`, `while` 등)도 자체적인 Lexical Environment를 가질 수 있습니다. 단, `let`과 `const`로 선언된 변수만 블록 스코프에 저장됩니다.

```javascript
// if 블록의 Lexical Environment
ifBlockLexicalEnvironment = {
  EnvironmentRecord: {
    y: 20,  // let은 블록 스코프에 새로 선언
    z: 30
  },
  outer: functionLexicalEnvironment  // 함수의 Lexical Environment 참조
}

// var x = 10은 함수의 Variable Environment에 저장 (블록 무시)
```

---

## Variable Environment vs Lexical Environment

### 차이점

ES6 이후로 `let`, `const`가 도입되면서 Variable Environment와 Lexical Environment가 구분되었습니다.

| 구분 | Variable Environment | Lexical Environment |
|------|---------------------|---------------------|
| **저장 대상** | `var` 선언, `function` 선언 | `let`, `const`, `class` 선언, 블록 스코프 바인딩 |
| **스코프** | 함수 스코프 | 블록 스코프 |
| **초기화** | `undefined`로 초기화 | 초기화 안 됨 (TDZ) |
| **변경** | 실행 중 변경되지 않음 | 블록 진입/종료 시 변경될 수 있음 |

### 동작 예시

```javascript
function example() {
  // Variable Environment: { a: undefined, func: function }
  // Lexical Environment: { b: <uninitialized> }

  var a = 1;
  let b = 2;

  function func() {}

  if (true) {
    // 새로운 Lexical Environment 생성
    // { c: <uninitialized>, d: <uninitialized> }
    // outer -> 함수의 Lexical Environment

    let c = 3;
    const d = 4;
    var e = 5; // Variable Environment에 저장 (블록 무시)
  }

  // if 블록 종료 -> 블록의 Lexical Environment 해제
  // c, d는 더 이상 접근 불가
  console.log(e); // 5 (var는 함수 스코프)
}
```

---

## 정리 및 요약

### 실행 컨텍스트의 핵심 개념

| 개념 | 설명 |
|------|------|
| **실행 컨텍스트** | 코드 실행에 필요한 환경 정보를 담은 객체 |
| **생성 단계** | 변수/함수 선언 정보를 Environment Record에 기록 |
| **실행 단계** | 코드를 순차적으로 실행하며 값 할당 |
| **Lexical Environment** | 식별자 바인딩과 외부 환경 참조를 포함 |
| **Variable Environment** | `var` 선언과 함수 선언 저장 |
| **스코프 체인** | outer 참조를 통해 형성되는 환경 연결 |

### 호이스팅 정리

| 선언 방식 | 호이스팅 | 초기값 | TDZ |
|-----------|---------|--------|-----|
| `var` | O | `undefined` | X |
| `let` | O | 없음 | O |
| `const` | O | 없음 | O |
| 함수 선언문 | O | 함수 객체 | X |
| 함수 표현식 | 변수 선언 방식에 따름 | 변수 선언 방식에 따름 | 변수 선언 방식에 따름 |

### 기억해야 할 핵심 원칙

1. **실행 컨텍스트는 Call Stack에서 관리된다**
2. **생성 단계에서 호이스팅이 발생한다**
3. **let/const는 TDZ로 인해 선언 전 접근이 불가능하다**
4. **스코프 체인은 outer 참조를 통해 형성된다**
5. **클로저는 함수가 생성될 때의 Lexical Environment를 기억한다**

---

## 참고 자료

- [ECMAScript Specification - Execution Contexts](https://tc39.es/ecma262/#sec-execution-contexts)
- [MDN - JavaScript 실행 컨텍스트](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/this)
- [JavaScript.info - Variable scope, closure](https://ko.javascript.info/closure)
- [You Don't Know JS - Scope & Closures](https://github.com/getify/You-Dont-Know-JS/blob/2nd-ed/scope-closures/README.md)
- [V8 Blog - Understanding V8's Bytecode](https://v8.dev/blog/understanding-ecmascript-part-1)
