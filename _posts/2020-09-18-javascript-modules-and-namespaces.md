---
title: JavaScript ES6 Modules와 Namespace 패턴 완벽 가이드
description: ES6 모듈 시스템의 동작 원리부터 export/import, 모듈의 핵심 기능, 브라우저 환경에서의 특징까지. 전역 변수 오염을 방지하는 Namespace 패턴도 함께 알아봅니다.
author: changsu
date: 2020-09-18 15:58:55 +0900
categories: [Programming, JavaScript]
tags: [javascript, es6, modules, export, import, namespace, iife, webpack, bundler, 모듈, 네임스페이스]
---

현대 JavaScript 개발에서 빼놓을 수 없는 **ES6 Modules**와 전역 변수 오염을 방지하는 **Namespace 패턴**을 알아봅니다.

## Modules이란?

애플리케이션의 크기가 커지면 파일을 여러 개로 분리해야 합니다. 이때 분리된 파일 각각을 **모듈(Module)**이라 합니다.

### 모듈의 정의

> 모듈은 보통 클래스 하나 혹은 특정한 목적을 가진 복수의 함수로 구성된 라이브러리입니다.

**모듈 = 파일 (스크립트) 하나**

## export와 import

모듈에 특수한 지시자 `export`와 `import`를 적용하면 다른 모듈을 불러와 기능을 공유할 수 있습니다.

### export (모듈 내보내기)

변수나 함수 앞에 `export`를 붙이면 외부 모듈에서 접근할 수 있습니다.

```javascript
// greet.js
export function greet(user) {
  return `Hello, ${user}!`;
}

export const VERSION = '1.0.0';

export class User {
  constructor(name) {
    this.name = name;
  }
}
```

### import (모듈 가져오기)

`import` 지시자를 사용하면 외부 모듈의 기능을 가져올 수 있습니다.

```javascript
// main.js
import { greet, VERSION, User } from './greet.js';

console.log(greet('Alice'));  // "Hello, Alice!"
console.log(VERSION);          // "1.0.0"

const user = new User('Bob');
```

## 브라우저에서 모듈 사용하기

### HTML에서 모듈 스크립트 선언

```html
<!DOCTYPE html>
<html>
<head>
  <title>Module Example</title>
</head>
<body>
  <!-- type="module" 속성 필수 -->
  <script type="module">
    import { greet } from './greet.js';

    document.body.innerHTML = greet('Tim');
  </script>
</body>
</html>
```

브라우저가 자동으로 모듈을 가져오고 평가한 다음 실행합니다.

## 모듈의 핵심 기능

### 1. 엄격 모드(Strict Mode)로 실행

모듈은 항상 `'use strict'`로 실행됩니다.

```html
<script type="module">
  a = 5;  // ReferenceError: a is not defined
</script>
```

선언되지 않은 변수에 값을 할당하는 등의 코드는 에러를 발생시킵니다.

### 2. 모듈 레벨 스코프

모듈은 자신만의 **독립적인 스코프**를 가집니다.

```html
<script type="module">
  let user = "Tim";  // 이 모듈 안에서만 접근 가능
</script>

<script type="module">
  console.log(user);  // ReferenceError: user is not defined
</script>
```

**해결 방법**: `export`/`import` 사용

```javascript
// userModule.js
export let user = "Tim";

// main.js
import { user } from './userModule.js';

console.log(user);  // "Tim"
```

**브라우저 전역 변수가 필요한 경우**:

```html
<script type="module">
  window.user = "Tim";  // 명시적으로 window에 할당
</script>

<script type="module">
  console.log(window.user);  // "Tim"
</script>
```

> ⚠️ **주의**: 전역 변수는 최소한으로 사용하세요!

### 3. 단 한 번만 실행

동일한 모듈이 여러 곳에서 사용되어도 **최초 호출 시 단 한 번만** 실행됩니다.

```javascript
// alert.js
console.log("모듈 실행!");

// 1.js
import './alert.js';  // "모듈 실행!"

// 2.js
import './alert.js';  // (아무 출력 없음)
```

#### 싱글톤 패턴 구현

```javascript
// config.js
export let config = {
  apiUrl: '',
  theme: 'light'
};

// init.js
import { config } from './config.js';
config.apiUrl = 'https://api.example.com';
config.theme = 'dark';

// app.js
import { config } from './config.js';

console.log(config.apiUrl);  // "https://api.example.com"
console.log(config.theme);   // "dark"

// 같은 객체를 참조하므로 init.js에서의 변경사항이 반영됨
```

**실전 활용 - 모듈 설정**:

```javascript
// admin.js
export let admin = {};

export function sayHi() {
  console.log(`${admin.name}, Hi!`);
}

// init.js (초기화 모듈)
import { admin } from './admin.js';
admin.name = "Peter";

// other.js
import { admin, sayHi } from './admin.js';

console.log(admin.name);  // "Peter"
sayHi();                  // "Peter, Hi!"
```

### 4. import.meta

`import.meta` 객체는 현재 모듈에 대한 정보를 제공합니다.

```html
<script type="module">
  console.log(import.meta.url);
  // 현재 실행 중인 모듈의 URL (또는 HTML 페이지의 URL)
</script>
```

### 5. this는 undefined

모듈 최상위 레벨의 `this`는 `undefined`입니다.

```html
<!-- 일반 스크립트 -->
<script>
  console.log(this);  // window
</script>

<!-- 모듈 스크립트 -->
<script type="module">
  console.log(this);  // undefined
</script>
```

## 브라우저 환경의 특정 기능

### 1. 지연 실행 (Deferred Execution)

모듈 스크립트는 **항상 지연 실행**됩니다 (`defer` 속성 자동 적용).

```html
<!DOCTYPE html>
<html>
<body>
  <h1>Title</h1>

  <script type="module">
    // DOM이 완전히 로드된 후 실행됨
    console.log(document.querySelector('h1'));  // <h1>Title</h1>
  </script>

  <h2>Subtitle</h2>
</body>
</html>
```

**특징**:
- 외부 모듈 다운로드 시 HTML 파싱이 멈추지 않음
- HTML 문서가 완전히 준비된 후 실행
- 스크립트의 상대적 순서 유지

**일반 스크립트 vs 모듈 스크립트**:

```html
<script type="module">
  console.log("모듈");  // 2번째 출력 (지연 실행)
</script>

<script>
  console.log("일반");  // 1번째 출력 (즉시 실행)
</script>
```

> 💡 **팁**: 로딩 인디케이터나 투명 오버레이를 사용하여 모듈 로딩 중 사용자 경험을 개선하세요.

### 2. async 속성

모듈 스크립트에서는 인라인 스크립트에도 `async` 속성을 적용할 수 있습니다.

```html
<!-- 모듈 로드가 완료되면 HTML이나 다른 스크립트를 기다리지 않고 즉시 실행 -->
<script async type="module">
  import { counter } from './analytics.js';

  counter.count();
</script>
```

**활용 사례**:
- 광고
- 문서 레벨 이벤트 리스너
- 카운터
- 어디에도 종속되지 않는 독립적 기능

### 3. 외부 스크립트

#### 동일 URL은 한 번만 실행

```html
<!-- my.js는 한 번만 로드 및 실행 -->
<script type="module" src="my.js"></script>
<script type="module" src="my.js"></script>
```

#### CORS 헤더 필요

다른 도메인에서 모듈을 가져오려면 **CORS 헤더**가 필요합니다.

```html
<!-- another-site.com이 Access-Control-Allow-Origin 헤더를 제공해야 함 -->
<script type="module" src="https://another-site.com/their.js"></script>
```

원격 서버 설정:
```
Access-Control-Allow-Origin: *
또는
Access-Control-Allow-Origin: https://your-domain.com
```

### 4. 경로 명시 필수

브라우저에서는 상대 경로나 절대 경로를 반드시 명시해야 합니다.

```javascript
// ❌ 에러
import { greet } from 'greet';

// ✅ 정상
import { greet } from './greet.js';
import { greet } from '/modules/greet.js';
import { greet } from 'https://example.com/greet.js';
```

> 📌 **참고**: Node.js나 번들링 툴(Webpack, Vite 등)은 경로 없이도 모듈을 찾을 수 있지만, 브라우저는 불가능합니다.

### 5. 구식 브라우저 대응 - nomodule

```html
<script type="module">
  console.log("모던 브라우저");
</script>

<script nomodule>
  console.log("구식 브라우저 (IE 등)");
</script>
```

- 모던 브라우저: `type="module"` 실행, `nomodule` 무시
- 구식 브라우저: `type="module"` 무시, `nomodule` 실행

## 빌드 툴 (Build Tools)

### 왜 빌드 툴이 필요한가?

실무에서는 모듈을 직접 사용하기보다 **Webpack**, **Vite**, **Rollup** 같은 번들러를 사용합니다.

**장점**:
- 여러 모듈을 하나의 파일로 번들링
- 경로 없는 모듈 사용 가능
- CSS, HTML 포맷 모듈 사용 가능
- 코드 최적화 및 압축

### 빌드 툴의 역할

```
1. 진입점 모듈 선택
   └─> main.js를 진입점으로 지정

2. 의존성 분석
   └─> main.js가 import하는 모듈들 파악
       └─> 그 모듈들이 import하는 모듈들 재귀적으로 파악

3. 번들링
   └─> 모든 모듈을 하나의 파일로 통합
       import문 → 번들러 내부 함수로 대체

4. 최적화
   ├─> Tree-shaking (사용하지 않는 코드 제거)
   ├─> Minification (공백 제거, 변수명 축약)
   ├─> Babel 변환 (최신 문법 → 구식 문법)
   └─> Dead code elimination (도달 불가능한 코드 삭제)
```

### 번들링 후 사용

```html
<!-- type="module" 불필요 -->
<script src="bundle.js"></script>
```

## Namespace 패턴

ES6 모듈이 나오기 전, 전역 변수 오염을 방지하기 위해 사용했던 패턴입니다.

### 문제: 전역 변수 오염

```javascript
// script1.js
var user = "Alice";

function greet() {
  console.log("Hello, " + user);
}

// script2.js
var user = "Bob";  // script1.js의 user를 덮어씀!

function greet() {  // script1.js의 greet을 덮어씀!
  console.log("Hi, " + user);
}
```

### 해결책 1: 객체 리터럴 네임스페이스

```javascript
// 전역 객체 하나만 생성
let MyApp = {};

// 모든 기능을 이 객체에 추가
MyApp.user = "Alice";

MyApp.greet = function() {
  console.log("Hello, " + MyApp.user);
};

MyApp.utils = {
  formatDate: function(date) {
    // ...
  },
  validateEmail: function(email) {
    // ...
  }
};

// 사용
MyApp.greet();
MyApp.utils.formatDate(new Date());
```

**장점**: 전역 변수 충돌 방지

**단점**:
- 모든 접근에 `MyApp.` 접두사 필요
- 체인이 길어짐 (`MyApp.module.submodule.function()`)

### 해결책 2: 범용 네임스페이스 함수

```javascript
// 기본 패턴
if (typeof MYAPP === "undefined") {
  var MYAPP = {};
}

// 권장 패턴 (짧고 간결)
var MYAPP = MYAPP || {};
```

**네임스페이스 생성 함수**:

```javascript
var MYAPP = MYAPP || {};

MYAPP.namespace = function(ns_string) {
  const parts = ns_string.split('.');
  let parent = MYAPP;

  // "MYAPP"을 제외한 나머지만 처리
  if (parts[0] === "MYAPP") {
    parts = parts.slice(1);
  }

  for (let i = 0; i < parts.length; i++) {
    // 해당 속성이 없으면 생성
    if (typeof parent[parts[i]] === "undefined") {
      parent[parts[i]] = {};
    }
    parent = parent[parts[i]];
  }

  return parent;
};

// 사용
MYAPP.namespace("MYAPP.modules.module2");

// 위 코드는 다음과 동일:
var MYAPP = {
  modules: {
    module2: {}
  }
};

// 직접 참조 가능
const module2 = MYAPP.namespace("MYAPP.modules.module2");
module2 === MYAPP.modules.module2;  // true

// 이제 안전하게 추가 가능
module2.myFunction = function() {
  console.log("Safe!");
};
```

### IIFE와 결합

```javascript
var MYAPP = MYAPP || {};

MYAPP.utils = (function() {
  // private 변수
  let privateVar = "secret";

  // private 함수
  function privateFunc() {
    return privateVar;
  }

  // public API
  return {
    publicMethod: function() {
      return privateFunc();
    },
    anotherMethod: function() {
      return "public";
    }
  };
})();

// 사용
MYAPP.utils.publicMethod();  // "secret"
MYAPP.utils.privateFunc();   // undefined (접근 불가)
```

## ES6 Modules vs Namespace 패턴

| 구분 | ES6 Modules | Namespace 패턴 |
|------|-------------|----------------|
| **문법** | `export`/`import` | 객체 리터럴 |
| **스코프** | 모듈 레벨 | 전역 객체 내부 |
| **로딩** | 정적 (컴파일 타임) | 동적 (런타임) |
| **최적화** | Tree-shaking 가능 | 불가능 |
| **브라우저 지원** | 모던 브라우저 | 모든 브라우저 |
| **사용 시기** | 현대 프로젝트 | 레거시 코드 |

## 실전 활용 예제

### Named Export vs Default Export

```javascript
// utils.js - Named Export (여러 개 내보내기)
export function add(a, b) {
  return a + b;
}

export function subtract(a, b) {
  return a - b;
}

export const PI = 3.14159;

// main.js
import { add, subtract, PI } from './utils.js';
```

```javascript
// Calculator.js - Default Export (하나만 내보내기)
export default class Calculator {
  add(a, b) {
    return a + b;
  }
}

// main.js
import Calculator from './Calculator.js';  // 중괄호 없음
```

### 별칭 (Alias)

```javascript
// 긴 이름을 짧게
import { veryLongFunctionName as short } from './module.js';

// 충돌 방지
import { user as user1 } from './module1.js';
import { user as user2 } from './module2.js';
```

### 전체 가져오기

```javascript
// math.js
export function add(a, b) { return a + b; }
export function subtract(a, b) { return a - b; }
export const PI = 3.14159;

// main.js
import * as math from './math.js';

console.log(math.add(1, 2));     // 3
console.log(math.PI);             // 3.14159
```

### Re-export

```javascript
// api/index.js (진입점 파일)
export { getUsers, createUser } from './users.js';
export { getPosts, createPost } from './posts.js';
export { login, logout } from './auth.js';

// app.js
import { getUsers, getPosts, login } from './api/index.js';
```

## 마치며

ES6 Modules는 현대 JavaScript 개발의 표준이 되었습니다.

### 핵심 요약

```markdown
✅ ES6 Modules
- export/import로 모듈 시스템 구축
- 파일 = 모듈, 독립적인 스코프
- 한 번만 실행, 결과 공유
- 엄격 모드 자동 적용
- 지연 실행 (defer)

✅ 브라우저 환경
- type="module" 필수
- 경로 명시 필수
- CORS 헤더 필요 (외부 도메인)
- nomodule로 구식 브라우저 대응

✅ 빌드 툴
- Webpack, Vite, Rollup 등
- 번들링, 최적화, Tree-shaking
- 프로덕션 필수

✅ Namespace 패턴
- 레거시 코드에서 사용
- 전역 변수 오염 방지
- 현대 프로젝트는 ES6 Modules 사용
```

모듈 시스템을 제대로 이해하고 활용하면, 유지보수가 쉽고 확장 가능한 JavaScript 애플리케이션을 만들 수 있습니다.

## 참고 자료

- [MDN - JavaScript Modules](https://developer.mozilla.org/ko/docs/Web/JavaScript/Guide/Modules)
- [모던 JavaScript 튜토리얼 - 모듈](https://ko.javascript.info/modules-intro)
- [MDN - import](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Statements/import)
- [MDN - export](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Statements/export)
- [Webpack 공식 문서](https://webpack.js.org/)
- [Vite 공식 문서](https://vitejs.dev/)
- [JavaScript Pattern - 객체 리터럴 네임스페이스](https://joshua1988.github.io/web-development/javascript/javascript-pattern-object/)
