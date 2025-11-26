---
title: JavaScript 함수 매개변수 완벽 가이드 - Parameters & Arguments
description: JavaScript 함수 매개변수의 모든 것. 기본값, Rest Parameters, Destructuring, Spread 연산자, Arguments 객체 등 ES6+ 기능과 실무 패턴을 완벽 정리합니다.
author: cotes
date: 2020-06-21 18:00:52 +0900
categories: [JavaScript, Functions]
tags: [javascript, function, es6, destructuring]
---

## 매개변수와 인자의 차이

### 매개변수 (Parameter)

**함수 정의 시** 괄호 안에 선언하는 변수입니다. 함수 내부에서 **변수처럼 사용**됩니다.

```javascript
function greet(name) {  // ← name은 매개변수 (Parameter)
  console.log(`Hello, ${name}!`);
}
```

### 인자 (Argument)

**함수 호출 시** 괄호 안에 전달하는 **실제 값**입니다.

```javascript
greet('Alice');  // ← 'Alice'는 인자 (Argument)
```

### 용어 비교

| 구분 | 영어 | 한국어 | 시점 | 위치 |
|------|------|--------|------|------|
| **Parameter** | Parameter | 매개변수 | 함수 정의 | `function fn(param)` |
| **Argument** | Argument | 인자 | 함수 호출 | `fn(arg)` |

```javascript
// 함수 정의
function doubleNumber(myNumber) {  // myNumber: 매개변수 (Parameter)
  return myNumber * 2;
}

// 함수 호출
doubleNumber(5);     // 5: 인자 (Argument)

const num = 10;
doubleNumber(num);   // num의 값 10이 인자로 전달
```

## 기본 매개변수 사용

### 단일 매개변수

```javascript
function greet(name) {
  console.log(`Hello, ${name}!`);
}

greet('Bob');  // "Hello, Bob!"
```

### 복수 매개변수

```javascript
function introduce(name, age, job) {
  console.log(`제 이름은 ${name}이고, ${age}살이며, ${job}입니다.`);
}

introduce('김개발', 28, '개발자');
// "제 이름은 김개발이고, 28살이며, 개발자입니다."
```

### 매개변수와 인자 개수 불일치

```javascript
function greet(name, message) {
  console.log(`${name}: ${message}`);
}

// 인자가 부족한 경우
greet('Alice');
// "Alice: undefined"

// 인자가 많은 경우
greet('Bob', 'Hello', 'Extra');
// "Bob: Hello" (Extra는 무시됨)
```

## 기본값 매개변수 (Default Parameters) - ES6

### 기본 문법

```javascript
function greet(name = 'Guest') {
  console.log(`Hello, ${name}!`);
}

greet();         // "Hello, Guest!"
greet('Alice');  // "Hello, Alice!"
```

### ES5 방식 vs ES6 방식

```javascript
// ❌ ES5 방식 (구식)
function greet(name) {
  name = name || 'Guest';
  console.log(`Hello, ${name}!`);
}

// ⚠️ 문제: 0, false, '' 등 falsy 값도 기본값으로 대체됨
greet(0);  // "Hello, Guest!" (0이 무시됨)

// ✅ ES6 방식 (권장)
function greet(name = 'Guest') {
  console.log(`Hello, ${name}!`);
}

greet(0);  // "Hello, 0!" (0이 유지됨)
greet(undefined);  // "Hello, Guest!" (undefined만 기본값 사용)
```

### 표현식을 기본값으로 사용

```javascript
// 함수 호출
function getDefaultName() {
  return 'Guest_' + Date.now();
}

function greet(name = getDefaultName()) {
  console.log(`Hello, ${name}!`);
}

greet();  // "Hello, Guest_1234567890!"

// 다른 매개변수 참조
function createUser(name, role = 'user', status = role === 'admin' ? 'verified' : 'pending') {
  return { name, role, status };
}

console.log(createUser('Alice', 'admin'));
// { name: 'Alice', role: 'admin', status: 'verified' }

console.log(createUser('Bob'));
// { name: 'Bob', role: 'user', status: 'pending' }
```

## Rest 파라미터 (...args) - ES6

### 기본 사용법

나머지 매개변수를 **배열로** 수집합니다.

```javascript
function sum(...numbers) {
  console.log(numbers);  // 배열
  return numbers.reduce((total, num) => total + num, 0);
}

console.log(sum(1, 2, 3));        // 6
console.log(sum(1, 2, 3, 4, 5));  // 15
```

### 일반 매개변수와 함께 사용

```javascript
function greet(greeting, ...names) {
  return `${greeting} ${names.join(', ')}!`;
}

console.log(greet('Hello', 'Alice', 'Bob', 'Charlie'));
// "Hello Alice, Bob, Charlie!"
```

> **주의**: Rest 파라미터는 **반드시 마지막**에 위치해야 합니다!
{: .prompt-warning }

```javascript
// ❌ 에러: Rest 파라미터는 마지막이어야 함
function wrong(...args, last) { }

// ✅ 올바른 사용
function correct(first, ...rest) { }
```

## Arguments 객체 (ES5)

### 기본 사용법

`arguments`는 **유사 배열 객체**로, 모든 인자를 담고 있습니다.

```javascript
function sum() {
  console.log(arguments);  // [Arguments] { '0': 1, '1': 2, '2': 3 }
  console.log(Array.isArray(arguments));  // false (배열 아님)

  // 배열 메서드 사용 불가
  // arguments.reduce(...) // ❌ 에러

  // 배열로 변환 필요
  const args = Array.from(arguments);
  return args.reduce((total, num) => total + num, 0);
}

console.log(sum(1, 2, 3));  // 6
```

### Arguments vs Rest 파라미터

| 구분 | Arguments 객체 | Rest 파라미터 |
|------|---------------|--------------|
| **타입** | 유사 배열 객체 | 진짜 배열 |
| **배열 메서드** | 사용 불가 (변환 필요) | 바로 사용 가능 |
| **화살표 함수** | 사용 불가 | 사용 가능 |
| **ES 버전** | ES5 | ES6 |
| **권장도** | ⚠️ 레거시 | ✅ 권장 |

```javascript
// ❌ Arguments 객체 (구식)
function sum() {
  return Array.from(arguments).reduce((a, b) => a + b, 0);
}

// ✅ Rest 파라미터 (권장)
function sum(...numbers) {
  return numbers.reduce((a, b) => a + b, 0);
}

// ❌ 화살표 함수는 arguments 없음
const sum = () => {
  console.log(arguments);  // ReferenceError!
};

// ✅ Rest 파라미터 사용
const sum = (...numbers) => {
  return numbers.reduce((a, b) => a + b, 0);
};
```

## Spread 연산자 (...) - ES6

### 배열을 인자로 펼치기

```javascript
function sum(a, b, c) {
  return a + b + c;
}

const numbers = [1, 2, 3];

// ES5 방식
console.log(sum.apply(null, numbers));  // 6

// ES6 방식
console.log(sum(...numbers));  // 6
```

### Math 함수와 함께 사용

```javascript
const numbers = [5, 2, 9, 1, 7];

console.log(Math.max(...numbers));  // 9
console.log(Math.min(...numbers));  // 1

// 여러 배열 결합
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
console.log(Math.max(...arr1, ...arr2));  // 6
```

## 구조 분해 할당 (Destructuring)

### 객체 매개변수

```javascript
// ❌ 일반 방식 (장황함)
function createUser(config) {
  const name = config.name;
  const age = config.age;
  const job = config.job || 'Unknown';
  // ...
}

// ✅ 구조 분해 할당 (간결함)
function createUser({ name, age, job = 'Unknown' }) {
  console.log(name, age, job);
}

createUser({ name: 'Alice', age: 25 });
// "Alice 25 Unknown"

createUser({ name: 'Bob', age: 30, job: 'Developer' });
// "Bob 30 Developer"
```

### 배열 매개변수

```javascript
function getFirstTwo([first, second]) {
  return { first, second };
}

console.log(getFirstTwo([1, 2, 3, 4]));
// { first: 1, second: 2 }
```

### 중첩 구조 분해

```javascript
function displayUser({ name, address: { city, country } }) {
  console.log(`${name} lives in ${city}, ${country}`);
}

const user = {
  name: 'Alice',
  address: {
    city: 'Seoul',
    country: 'Korea'
  }
};

displayUser(user);
// "Alice lives in Seoul, Korea"
```

### 기본값과 함께 사용

```javascript
function createConfig({
  host = 'localhost',
  port = 3000,
  protocol = 'http'
} = {}) {
  return `${protocol}://${host}:${port}`;
}

console.log(createConfig());
// "http://localhost:3000"

console.log(createConfig({ port: 8080 }));
// "http://localhost:8080"

console.log(createConfig({ host: 'example.com', protocol: 'https' }));
// "https://example.com:3000"
```

## 명명된 매개변수 패턴 (Named Parameters)

### 장점

1. **순서 무관**: 매개변수 순서를 기억할 필요 없음
2. **가독성**: 무엇을 전달하는지 명확함
3. **선택적 인자**: 필요한 것만 전달
4. **확장성**: 새 매개변수 추가 시 기존 코드 영향 최소

```javascript
// ❌ 위치 기반 매개변수 (순서 중요)
function createServer(host, port, protocol, timeout, retries) {
  // ...
}

createServer('localhost', 3000, 'http', 5000, 3);
// 순서 기억해야 하고, 중간 값을 건너뛸 수 없음

// ✅ 명명된 매개변수 (순서 무관)
function createServer({ host, port, protocol, timeout, retries }) {
  // ...
}

createServer({
  port: 3000,
  host: 'localhost',  // 순서 바뀌어도 OK
  timeout: 5000,
  protocol: 'http',
  retries: 3
});

// 일부만 전달 가능
createServer({
  port: 8080,
  host: 'example.com'
});
```

## 실무 패턴

### 1. 옵션 객체 패턴

```javascript
function fetchData(url, {
  method = 'GET',
  headers = {},
  timeout = 5000,
  retries = 3,
  cache = true
} = {}) {
  console.log(`Fetching ${url} with`, { method, headers, timeout, retries, cache });
  // 실제 fetch 로직...
}

// 기본값만 사용
fetchData('/api/users');

// 일부 옵션만 재정의
fetchData('/api/users', {
  method: 'POST',
  timeout: 10000
});

// 모든 옵션 지정
fetchData('/api/users', {
  method: 'PUT',
  headers: { 'Content-Type': 'application/json' },
  timeout: 15000,
  retries: 5,
  cache: false
});
```

### 2. 빌더 패턴 함수

```javascript
function createQuery({
  table,
  fields = ['*'],
  where = {},
  orderBy = null,
  limit = null
}) {
  let query = `SELECT ${fields.join(', ')} FROM ${table}`;

  const conditions = Object.entries(where)
    .map(([key, value]) => `${key} = '${value}'`)
    .join(' AND ');

  if (conditions) {
    query += ` WHERE ${conditions}`;
  }

  if (orderBy) {
    query += ` ORDER BY ${orderBy}`;
  }

  if (limit) {
    query += ` LIMIT ${limit}`;
  }

  return query;
}

console.log(createQuery({
  table: 'users',
  fields: ['id', 'name', 'email'],
  where: { status: 'active', role: 'admin' },
  orderBy: 'created_at DESC',
  limit: 10
}));
// "SELECT id, name, email FROM users WHERE status = 'active' AND role = 'admin' ORDER BY created_at DESC LIMIT 10"
```

### 3. 콜백과 함께 사용

```javascript
function processData(data, {
  onSuccess = () => {},
  onError = () => {},
  onProgress = () => {},
  transform = x => x
} = {}) {
  try {
    onProgress(0);

    const processed = data.map(transform);
    onProgress(50);

    // 추가 처리...
    onProgress(100);

    onSuccess(processed);
  } catch (error) {
    onError(error);
  }
}

processData([1, 2, 3], {
  transform: x => x * 2,
  onSuccess: result => console.log('성공:', result),
  onError: error => console.error('에러:', error),
  onProgress: percent => console.log(`진행: ${percent}%`)
});
```

### 4. Partial Application (부분 적용)

```javascript
function createMultiplier(factor) {
  return function(number) {
    return number * factor;
  };
}

const double = createMultiplier(2);
const triple = createMultiplier(3);

console.log(double(5));  // 10
console.log(triple(5));  // 15

// Rest 파라미터와 함께
function createFormatter(prefix, ...suffixes) {
  return function(value) {
    return prefix + value + suffixes.join('');
  };
}

const formatPrice = createFormatter('$', ' USD');
console.log(formatPrice(100));  // "$100 USD"
```

### 5. 파이프라인 함수

```javascript
function pipe(...functions) {
  return function(initialValue) {
    return functions.reduce((value, fn) => fn(value), initialValue);
  };
}

const addOne = x => x + 1;
const double = x => x * 2;
const square = x => x * x;

const process = pipe(addOne, double, square);

console.log(process(3));  // ((3 + 1) * 2)^2 = 64
```

### 6. 유효성 검사

```javascript
function createUser({
  name,
  email,
  age,
  role = 'user'
} = {}) {
  // 필수 매개변수 검사
  if (!name) throw new Error('name is required');
  if (!email) throw new Error('email is required');

  // 타입 검사
  if (typeof age !== 'number') {
    throw new Error('age must be a number');
  }

  // 값 검사
  if (age < 0 || age > 150) {
    throw new Error('age must be between 0 and 150');
  }

  return { name, email, age, role };
}

try {
  createUser({ name: 'Alice', email: 'alice@example.com', age: 25 });
} catch (error) {
  console.error(error.message);
}
```

### 7. 커링 (Currying)

```javascript
// 일반 함수
function add(a, b, c) {
  return a + b + c;
}

// 커링된 함수
function curriedAdd(a) {
  return function(b) {
    return function(c) {
      return a + b + c;
    };
  };
}

console.log(add(1, 2, 3));              // 6
console.log(curriedAdd(1)(2)(3));       // 6

// 화살표 함수로 간결하게
const curriedAdd = a => b => c => a + b + c;

console.log(curriedAdd(1)(2)(3));       // 6

// 부분 적용
const add1 = curriedAdd(1);
const add1and2 = add1(2);
console.log(add1and2(3));               // 6
```

## 고급 패턴

### 1. 함수 오버로딩 시뮬레이션

JavaScript는 함수 오버로딩을 지원하지 않지만 시뮬레이션할 수 있습니다.

```javascript
function createElement(tagOrOptions, content, attributes) {
  let tag, text, attrs;

  // 첫 번째 인자가 객체인 경우
  if (typeof tagOrOptions === 'object') {
    ({ tag, content: text, attributes: attrs } = tagOrOptions);
  } else {
    // 위치 기반 인자
    tag = tagOrOptions;
    text = content;
    attrs = attributes;
  }

  const element = document.createElement(tag);
  if (text) element.textContent = text;
  if (attrs) {
    Object.entries(attrs).forEach(([key, value]) => {
      element.setAttribute(key, value);
    });
  }

  return element;
}

// 사용법 1: 위치 기반
createElement('div', 'Hello', { class: 'greeting' });

// 사용법 2: 객체 기반
createElement({
  tag: 'div',
  content: 'Hello',
  attributes: { class: 'greeting' }
});
```

### 2. 메모이제이션 with Rest 파라미터

```javascript
function memoize(fn) {
  const cache = new Map();

  return function(...args) {
    const key = JSON.stringify(args);

    if (cache.has(key)) {
      console.log('캐시에서 반환');
      return cache.get(key);
    }

    console.log('계산 실행');
    const result = fn(...args);
    cache.set(key, result);
    return result;
  };
}

// 피보나치 함수
const fibonacci = memoize(function fib(n) {
  if (n <= 1) return n;
  return fib(n - 1) + fib(n - 2);
});

console.log(fibonacci(10));  // 계산 실행
console.log(fibonacci(10));  // 캐시에서 반환
```

## 타입 안전성 (TypeScript 스타일)

JavaScript에서도 런타임 타입 체크를 할 수 있습니다.

```javascript
function createUser({
  name,
  age,
  email,
  role = 'user',
  active = true
}) {
  // 타입 체크
  const validators = {
    name: v => typeof v === 'string' && v.length > 0,
    age: v => typeof v === 'number' && v >= 0,
    email: v => typeof v === 'string' && v.includes('@'),
    role: v => ['user', 'admin', 'moderator'].includes(v),
    active: v => typeof v === 'boolean'
  };

  // 검증 실행
  Object.entries({ name, age, email, role, active }).forEach(([key, value]) => {
    if (!validators[key](value)) {
      throw new TypeError(`Invalid ${key}: ${value}`);
    }
  });

  return { name, age, email, role, active };
}

try {
  createUser({
    name: 'Alice',
    age: 25,
    email: 'alice@example.com'
  });
  console.log('유효한 사용자');
} catch (error) {
  console.error(error.message);
}
```

## Best Practices

### ✅ 권장 사항

```javascript
// 1. 기본값 매개변수 사용
function greet(name = 'Guest') { }  // ✅
function greet(name) {
  name = name || 'Guest';  // ❌ 구식
}

// 2. Rest 파라미터 사용
function sum(...numbers) { }  // ✅
function sum() {
  const numbers = Array.from(arguments);  // ❌ 구식
}

// 3. 명명된 매개변수 (객체 구조 분해)
function createUser({ name, age, email }) { }  // ✅
function createUser(name, age, email) { }  // ⚠️ 순서 기억 필요

// 4. 기본값과 함께 구조 분해
function fetchData({ url, method = 'GET' } = {}) { }  // ✅

// 5. Spread 연산자로 배열 전달
Math.max(...numbers);  // ✅
Math.max.apply(null, numbers);  // ❌ 구식
```

### ❌ 피해야 할 사항

```javascript
// 1. arguments 객체 (화살표 함수에서 불가)
function sum() {
  return Array.from(arguments).reduce((a, b) => a + b);  // ❌
}
const sum = (...args) => args.reduce((a, b) => a + b);  // ✅

// 2. 너무 많은 위치 기반 매개변수
function createUser(name, age, email, phone, address, city, country) { }  // ❌
function createUser({ name, age, email, phone, address, city, country }) { }  // ✅

// 3. 필수 매개변수 검증 누락
function process(data) {
  return data.map(x => x * 2);  // ❌ data가 없으면 에러
}
function process(data = []) {  // ✅
  return data.map(x => x * 2);
}

// 4. 부작용 있는 기본값
let counter = 0;
function test(value = counter++) { }  // ❌ 호출마다 counter 증가
```

## 비교 정리표

| 기능 | ES5 | ES6+ | 권장 |
|------|-----|------|------|
| **가변 인자** | `arguments` | `...rest` | ES6+ |
| **기본값** | `param \|\| default` | `param = default` | ES6+ |
| **배열 전개** | `apply()` | `...spread` | ES6+ |
| **명명된 매개변수** | 직접 구현 | 객체 구조 분해 | ES6+ |

## 핵심 정리

### 용어

- **Parameter**: 함수 정의 시 선언하는 변수
- **Argument**: 함수 호출 시 전달하는 값

### ES6+ 필수 기능

1. **기본값 매개변수**: `function fn(a = 1) {}`
2. **Rest 파라미터**: `function fn(...args) {}`
3. **Spread 연산자**: `fn(...array)`
4. **구조 분해 할당**: `function fn({ a, b }) {}`

### 실무 팁

- 매개변수가 3개 이상이면 → 객체로 받기
- 옵션이 많으면 → 명명된 매개변수 사용
- 가변 인자가 필요하면 → Rest 파라미터
- 순서가 중요하지 않으면 → 구조 분해 할당

> **결론**: ES6+의 기본값, Rest, Spread, 구조 분해를 활용하면 더 깔끔하고 유연한 함수를 작성할 수 있습니다!
{: .prompt-tip }

## 참고 자료

- [MDN - Function Parameters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Functions#function_parameters)
- [MDN - Rest Parameters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters)
- [MDN - Default Parameters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Default_parameters)
- [MDN - Spread Syntax](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax)
- [JavaScript.info - Function Parameters](https://javascript.info/function-basics#parameters)
