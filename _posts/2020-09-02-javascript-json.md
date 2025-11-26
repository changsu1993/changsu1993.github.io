---
title: JavaScript JSON 완벽 가이드 - API 통신과 데이터 직렬화의 핵심
date: 2020-09-02 01:38:00 +0900
categories: [개발, JavaScript]
tags: [javascript, json, api, fetch]
author: changsu
toc: true
comments: true
description: JavaScript JSON의 모든 것. stringify와 parse 메서드부터 실전 API 통신, 에러 처리까지 완벽 정리합니다.
---

## 들어가며

**JSON**(JavaScript Object Notation)은 현대 웹 개발에서 **가장 보편적인 데이터 교환 형식**입니다.

서버와 클라이언트 간 데이터 통신, 로컬 스토리지 저장, 설정 파일 등 거의 모든 곳에서 JSON을 사용합니다.

> JSON은 1999년 ECMAScript 3의 JavaScript Object에서 영감을 받아 만들어졌습니다!
{: .prompt-info }

## JSON이란?

**JSON**(JavaScript Object Notation)은 데이터를 표현하기 위한 경량 텍스트 기반 형식입니다.

### JSON의 특징

| 특징 | 설명 |
|------|------|
| **경량** | 텍스트 기반으로 용량이 작음 |
| **가독성** | 사람이 읽고 이해하기 쉬움 |
| **언어 독립적** | 모든 프로그래밍 언어에서 사용 가능 |
| **구조적** | key-value 쌍으로 데이터 구조화 |
| **표준화** | ECMA-404, RFC 8259 표준 |

### JSON vs JavaScript Object

```javascript
// JavaScript Object (객체 리터럴)
const jsObject = {
  name: 'John',
  age: 30,
  active: true
};

// JSON (문자열)
const jsonString = '{"name":"John","age":30,"active":true}';
```

**주요 차이점:**

| 구분 | JavaScript Object | JSON |
|------|-------------------|------|
| **타입** | 객체 (Object) | 문자열 (String) |
| **키(Key)** | 따옴표 선택 | 반드시 큰따옴표 |
| **값(Value)** | 함수, undefined 가능 | 불가능 |
| **주석** | 가능 | 불가능 |
| **trailing comma** | 가능 | 불가능 |

> ⚠️ **중요**: JSON은 문자열입니다! JavaScript 객체와 혼동하지 마세요.
{: .prompt-warning }

## JSON 데이터 타입

JSON에서 사용 가능한 데이터 타입은 제한적입니다.

### 지원하는 타입

```json
{
  "string": "문자열",
  "number": 123,
  "boolean": true,
  "null": null,
  "array": [1, 2, 3],
  "object": {"nested": "value"}
}
```

### 지원하지 않는 타입

```javascript
// ❌ JSON에서 불가능한 것들
{
  func: function() {},      // 함수
  undef: undefined,         // undefined
  symbol: Symbol('id'),     // Symbol
  date: new Date(),         // Date 객체 (문자열로 변환됨)
  infinity: Infinity,       // Infinity, NaN
  circular: obj            // 순환 참조
}
```

## JSON.stringify() - Object를 JSON으로

**`JSON.stringify()`**는 JavaScript 값을 JSON 문자열로 변환합니다.

### 기본 문법

```javascript
JSON.stringify(value, replacer, space)
```

**매개변수:**
- `value`: JSON으로 변환할 값
- `replacer`: 변환할 속성을 선택하는 함수 또는 배열 (선택)
- `space`: 들여쓰기 공백 수 (선택)

### 기본 예제

```javascript
// 배열 변환
const arr = [1, 2, 3, 4, 5];
const jsonArr = JSON.stringify(arr);
console.log(jsonArr);  // '[1,2,3,4,5]'
console.log(typeof jsonArr);  // 'string'

// 객체 변환
const user = {
  name: 'Alice',
  age: 25,
  email: 'alice@example.com'
};

const jsonUser = JSON.stringify(user);
console.log(jsonUser);
// '{"name":"Alice","age":25,"email":"alice@example.com"}'
```

### 제외되는 값들

```javascript
const rabbit = {
  name: 'Tori',
  color: 'white',
  size: 'small',
  birthDate: new Date(),
  jump: function() {    // ❌ 함수는 제외
    console.log('Jump!');
  },
  symbol: Symbol('id')  // ❌ Symbol은 제외
};

const json = JSON.stringify(rabbit);
console.log(json);
// '{"name":"Tori","color":"white","size":"small","birthDate":"2024-01-15T12:00:00.000Z"}'
// jump 함수와 symbol은 제외됨
```

> 💡 **주의**: 함수, Symbol, undefined는 JSON 변환 시 자동으로 제외됩니다!
{: .prompt-tip }

### replacer로 속성 선택

```javascript
const user = {
  name: 'Bob',
  age: 30,
  password: 'secret123',
  email: 'bob@example.com'
};

// 배열로 속성 선택
const publicData = JSON.stringify(user, ['name', 'email']);
console.log(publicData);
// '{"name":"Bob","email":"bob@example.com"}'

// 함수로 세밀한 제어
const filtered = JSON.stringify(user, (key, value) => {
  if (key === 'password') return undefined;  // password 제외
  return value;
});
console.log(filtered);
// '{"name":"Bob","age":30,"email":"bob@example.com"}'
```

### space로 가독성 향상

```javascript
const data = {
  name: 'John',
  age: 30,
  address: {
    city: 'Seoul',
    country: 'Korea'
  }
};

// 들여쓰기 2칸
console.log(JSON.stringify(data, null, 2));
/*
{
  "name": "John",
  "age": 30,
  "address": {
    "city": "Seoul",
    "country": "Korea"
  }
}
*/

// 들여쓰기 Tab
console.log(JSON.stringify(data, null, '\t'));
```

> 💡 **팁**: 디버깅이나 로깅 시에는 `space` 매개변수를 사용하여 가독성을 높이세요!
{: .prompt-tip }

## JSON.parse() - JSON을 Object로

**`JSON.parse()`**는 JSON 문자열을 JavaScript 값으로 변환합니다.

### 기본 문법

```javascript
JSON.parse(text, reviver)
```

**매개변수:**
- `text`: 파싱할 JSON 문자열
- `reviver`: 변환된 값을 변형하는 함수 (선택)

### 기본 예제

```javascript
const jsonString = '{"name":"Alice","age":25,"active":true}';

const user = JSON.parse(jsonString);
console.log(user);
// { name: 'Alice', age: 25, active: true }

console.log(user.name);   // 'Alice'
console.log(user.age);    // 25
console.log(typeof user); // 'object'
```

### Date 객체 복원

```javascript
const rabbit = {
  name: 'Tori',
  birthDate: new Date()
};

// JSON으로 변환
const json = JSON.stringify(rabbit);
console.log(json);
// '{"name":"Tori","birthDate":"2024-01-15T12:00:00.000Z"}'

// 기본 parse: birthDate는 문자열
const parsed = JSON.parse(json);
console.log(parsed.birthDate);          // '2024-01-15T12:00:00.000Z' (문자열)
console.log(typeof parsed.birthDate);   // 'string'

// ❌ Date 메서드 사용 불가
// parsed.birthDate.getDate();  // Error!

// ✅ reviver로 Date 복원
const restored = JSON.parse(json, (key, value) => {
  if (key === 'birthDate') {
    return new Date(value);
  }
  return value;
});

console.log(restored.birthDate.getDate());  // 15 (정상 작동)
console.log(typeof restored.birthDate);     // 'object' (Date 객체)
```

### 중첩 객체 처리

```javascript
const jsonData = `{
  "user": {
    "name": "John",
    "registered": "2024-01-15T10:30:00.000Z"
  },
  "posts": [
    {
      "title": "First Post",
      "createdAt": "2024-01-16T09:00:00.000Z"
    }
  ]
}`;

const data = JSON.parse(jsonData, (key, value) => {
  // 모든 날짜 문자열을 Date 객체로 변환
  if (typeof value === 'string' && /^\d{4}-\d{2}-\d{2}T/.test(value)) {
    return new Date(value);
  }
  return value;
});

console.log(data.user.registered.getFullYear());    // 2024
console.log(data.posts[0].createdAt.getMonth());    // 0 (1월)
```

## 실전 활용

### 1. API 통신 (fetch)

```javascript
// GET 요청
async function getUsers() {
  try {
    const response = await fetch('https://api.example.com/users');

    // response.json()은 자동으로 JSON.parse() 수행
    const users = await response.json();
    console.log(users);
  } catch (error) {
    console.error('Error fetching users:', error);
  }
}

// POST 요청
async function createUser(userData) {
  try {
    const response = await fetch('https://api.example.com/users', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(userData)  // ✅ 객체를 JSON 문자열로 변환
    });

    const result = await response.json();
    console.log('User created:', result);
  } catch (error) {
    console.error('Error creating user:', error);
  }
}

// 사용 예
createUser({
  name: 'Alice',
  email: 'alice@example.com',
  age: 25
});
```

### 2. LocalStorage 저장

```javascript
// 데이터 저장
function saveSettings(settings) {
  localStorage.setItem('userSettings', JSON.stringify(settings));
}

// 데이터 불러오기
function loadSettings() {
  const stored = localStorage.getItem('userSettings');
  return stored ? JSON.parse(stored) : getDefaultSettings();
}

// 사용 예
const settings = {
  theme: 'dark',
  language: 'ko',
  notifications: true
};

saveSettings(settings);

const loaded = loadSettings();
console.log(loaded.theme);  // 'dark'
```

### 3. Deep Copy (깊은 복사)

```javascript
const original = {
  name: 'John',
  age: 30,
  address: {
    city: 'Seoul',
    country: 'Korea'
  },
  hobbies: ['reading', 'gaming']
};

// ✅ 간단한 깊은 복사
const deepCopy = JSON.parse(JSON.stringify(original));

deepCopy.address.city = 'Busan';
console.log(original.address.city);  // 'Seoul' (원본 유지)
console.log(deepCopy.address.city);  // 'Busan'

// ⚠️ 주의: 함수, Date, undefined는 복사 안 됨
const complex = {
  name: 'Alice',
  func: function() {},
  date: new Date(),
  undef: undefined
};

const copied = JSON.parse(JSON.stringify(complex));
console.log(copied);
// { name: 'Alice', date: '2024-01-15T12:00:00.000Z' }
// func와 undef는 사라짐, date는 문자열로 변환
```

> ⚠️ **제한사항**: 함수, Date 객체, undefined, 순환 참조가 있으면 정확한 복사가 안 됩니다!
{: .prompt-warning }

### 4. 디버깅 및 로깅

```javascript
const complexObject = {
  users: [
    { id: 1, name: 'Alice', roles: ['admin', 'user'] },
    { id: 2, name: 'Bob', roles: ['user'] }
  ],
  config: {
    theme: 'dark',
    language: 'ko'
  }
};

// ❌ 가독성 나쁨
console.log(complexObject);
// { users: [...], config: {...} }

// ✅ 가독성 좋음
console.log(JSON.stringify(complexObject, null, 2));
/*
{
  "users": [
    {
      "id": 1,
      "name": "Alice",
      "roles": ["admin", "user"]
    },
    ...
  ],
  "config": {
    "theme": "dark",
    "language": "ko"
  }
}
*/
```

## 에러 처리

### JSON 파싱 에러

```javascript
const invalidJson = '{"name": "John", age: 30}';  // age에 따옴표 없음

try {
  const data = JSON.parse(invalidJson);
} catch (error) {
  console.error('JSON 파싱 실패:', error.message);
  // JSON 파싱 실패: Unexpected token a in JSON at position 16
}
```

### 안전한 JSON 파싱

```javascript
function safeJsonParse(jsonString, defaultValue = null) {
  try {
    return JSON.parse(jsonString);
  } catch (error) {
    console.error('JSON parsing error:', error);
    return defaultValue;
  }
}

// 사용 예
const data = safeJsonParse('invalid json', { error: true });
console.log(data);  // { error: true }
```

### 순환 참조 처리

```javascript
const obj = { name: 'John' };
obj.self = obj;  // 순환 참조

try {
  JSON.stringify(obj);
} catch (error) {
  console.error('순환 참조 에러:', error.message);
  // Converting circular structure to JSON
}

// ✅ 해결 방법: replacer 사용
function getCircularReplacer() {
  const seen = new WeakSet();
  return (key, value) => {
    if (typeof value === 'object' && value !== null) {
      if (seen.has(value)) {
        return;  // 순환 참조 제거
      }
      seen.add(value);
    }
    return value;
  };
}

const json = JSON.stringify(obj, getCircularReplacer());
console.log(json);  // '{"name":"John"}'
```

## 성능 최적화

### 큰 데이터 처리

```javascript
// ❌ 비효율적: 매번 stringify
function logMultipleTimes(data) {
  for (let i = 0; i < 1000; i++) {
    console.log(JSON.stringify(data));  // 1000번 stringify
  }
}

// ✅ 효율적: 한 번만 stringify
function logMultipleTimes(data) {
  const jsonString = JSON.stringify(data);  // 1번만 stringify
  for (let i = 0; i < 1000; i++) {
    console.log(jsonString);
  }
}
```

### Streaming Parser 사용 (대용량 JSON)

```javascript
// 대용량 JSON 파일 처리 (Node.js)
const fs = require('fs');
const JSONStream = require('JSONStream');

fs.createReadStream('large-file.json')
  .pipe(JSONStream.parse('users.*'))
  .on('data', (user) => {
    console.log('User:', user);
  });
```

## 보안 고려사항

### XSS 방지

```javascript
// ❌ 위험: 사용자 입력을 직접 파싱
const userInput = '{"name":"<script>alert(\'XSS\')</script>"}';
const data = JSON.parse(userInput);

// ✅ 안전: HTML 이스케이프
function sanitizeJson(jsonString) {
  const data = JSON.parse(jsonString);
  // 모든 문자열 값에 대해 HTML 이스케이프
  return JSON.parse(JSON.stringify(data, (key, value) => {
    if (typeof value === 'string') {
      return value
        .replace(/&/g, '&amp;')
        .replace(/</g, '&lt;')
        .replace(/>/g, '&gt;');
    }
    return value;
  }));
}
```

### eval() 사용 금지

```javascript
const jsonString = '{"name":"John"}';

// ❌ 절대 사용 금지!
const data = eval('(' + jsonString + ')');  // 보안 위험!

// ✅ 항상 JSON.parse() 사용
const safeData = JSON.parse(jsonString);
```

## 유용한 헬퍼 함수

### JSON 유효성 검사

```javascript
function isValidJson(str) {
  try {
    JSON.parse(str);
    return true;
  } catch {
    return false;
  }
}

console.log(isValidJson('{"name":"John"}'));   // true
console.log(isValidJson('{name: "John"}'));    // false
```

### Pretty Print

```javascript
function prettyPrint(obj) {
  return JSON.stringify(obj, null, 2);
}

const data = { name: 'John', age: 30 };
console.log(prettyPrint(data));
```

### JSON 비교

```javascript
function jsonEqual(obj1, obj2) {
  return JSON.stringify(obj1) === JSON.stringify(obj2);
}

const a = { name: 'John', age: 30 };
const b = { age: 30, name: 'John' };

console.log(jsonEqual(a, b));  // false (속성 순서가 다름)

// ✅ 개선: 정렬 후 비교
function jsonEqualSorted(obj1, obj2) {
  const sort = (obj) => {
    if (typeof obj !== 'object' || obj === null) return obj;
    if (Array.isArray(obj)) return obj.map(sort);
    return Object.keys(obj).sort().reduce((result, key) => {
      result[key] = sort(obj[key]);
      return result;
    }, {});
  };

  return JSON.stringify(sort(obj1)) === JSON.stringify(sort(obj2));
}

console.log(jsonEqualSorted(a, b));  // true
```

## 핵심 정리

### JSON 특징

| 특징 | 설명 |
|------|------|
| **타입** | 텍스트 기반 데이터 형식 |
| **구조** | key-value 쌍 |
| **지원 타입** | string, number, boolean, null, array, object |
| **미지원** | function, undefined, Symbol, Date (객체로) |
| **용도** | 데이터 교환, 저장, 설정 파일 |

### 주요 메서드

```javascript
// Object → JSON
JSON.stringify(value, replacer, space)

// JSON → Object
JSON.parse(text, reviver)
```

### Best Practices

1. **에러 처리 필수**
   ```javascript
   try {
     const data = JSON.parse(jsonString);
   } catch (error) {
     // 에러 핸들링
   }
   ```

2. **타입 검증**
   ```javascript
   const data = JSON.parse(jsonString);
   if (typeof data === 'object' && data !== null) {
     // 안전하게 사용
   }
   ```

3. **Date 객체 명시적 처리**
   ```javascript
   // stringify 시
   const json = JSON.stringify({
     date: new Date().toISOString()
   });

   // parse 시
   const data = JSON.parse(json, (key, value) => {
     if (key === 'date') return new Date(value);
     return value;
   });
   ```

4. **보안 고려**
   - `eval()` 절대 사용 금지
   - 사용자 입력 검증
   - XSS 방지를 위한 이스케이프

> JSON은 현대 웹 개발의 핵심입니다. stringify와 parse를 완벽하게 이해하면 API 통신과 데이터 처리가 훨씬 수월해집니다!
{: .prompt-info }

## 참고 자료

- [MDN - JSON](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/JSON)
- [JSON.org](https://www.json.org/json-ko.html)
- [RFC 8259 - JSON Specification](https://datatracker.ietf.org/doc/html/rfc8259)
- [JavaScript.info - JSON methods](https://ko.javascript.info/json)
