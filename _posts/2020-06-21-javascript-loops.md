---
title: JavaScript 반복문 완벽 가이드 - for, while, for...of, forEach
description: JavaScript의 모든 반복문을 마스터하세요. for, while, for...of, for...in, forEach, map의 차이점과 성능 비교, 실무 패턴까지 완벽 정리합니다.
author: cotes
date: 2020-06-21 18:40:45 +0900
categories: [JavaScript, Basics]
tags: [javascript, for-loop, while-loop, for-of, for-in, foreach, iteration, loop]
---

## 반복문이란?

**반복문(Loop)**은 코드를 원하는 만큼 **반복 실행**하는 제어 구조입니다. 같은 작업을 여러 번 수행해야 할 때 코드 중복 없이 효율적으로 처리할 수 있습니다.

```javascript
// ❌ 반복문 없이 (비효율적)
console.log(0);
console.log(1);
console.log(2);
console.log(3);
console.log(4);

// ✅ 반복문 사용 (효율적)
for (let i = 0; i < 5; i++) {
  console.log(i);
}
```

## for 문 - 기본 반복문

### 기본 문법

```javascript
for (초기화; 조건식; 증감식) {
  // 반복할 코드
}
```

**구성 요소:**
1. **초기화**: 반복문 시작 전 한 번 실행 (`let i = 0`)
2. **조건식**: 매 반복마다 평가, `true`면 계속, `false`면 종료 (`i < 5`)
3. **증감식**: 매 반복 후 실행 (`i++`)

### 기본 예제

```javascript
// 0부터 5까지 출력
for (let i = 0; i <= 5; i++) {
  console.log(i);
}
// 0, 1, 2, 3, 4, 5

// < 와 <= 의 차이
for (let i = 0; i < 5; i++) {
  console.log(i);
}
// 0, 1, 2, 3, 4 (5는 제외)
```

### 실행 순서

```javascript
for (let i = 0; i < 3; i++) {
  console.log(i);
}

// 실행 순서:
// 1. i = 0 (초기화, 한 번만)
// 2. i < 3? true → console.log(0)
// 3. i++ → i = 1
// 4. i < 3? true → console.log(1)
// 5. i++ → i = 2
// 6. i < 3? true → console.log(2)
// 7. i++ → i = 3
// 8. i < 3? false → 종료
```

### 배열 순회

```javascript
const cities = ['서울', '대전', '대구', '부산', '광주'];

// 인덱스로 접근
for (let i = 0; i < cities.length; i++) {
  console.log(cities[i]);
}

// 조건 검사와 함께
const home = '대전';

for (let i = 0; i < cities.length; i++) {
  if (cities[i] === home) {
    console.log(`아, ${cities[i]} 사시는군요!`);
  }
}
```

### 역순 반복

```javascript
// 뒤에서부터 순회
for (let i = 4; i >= 0; i--) {
  console.log(i);
}
// 4, 3, 2, 1, 0

// 배열 역순 순회
const arr = ['a', 'b', 'c', 'd'];
for (let i = arr.length - 1; i >= 0; i--) {
  console.log(arr[i]);
}
// 'd', 'c', 'b', 'a'
```

### 증감량 변경

```javascript
// 2씩 증가
for (let i = 0; i < 10; i += 2) {
  console.log(i);
}
// 0, 2, 4, 6, 8

// 10씩 감소
for (let i = 100; i > 0; i -= 10) {
  console.log(i);
}
// 100, 90, 80, 70, ..., 10
```

## while 문 - 조건 기반 반복

### 기본 문법

```javascript
while (조건식) {
  // 조건이 true인 동안 반복
}
```

### 기본 예제

```javascript
let i = 0;
while (i < 5) {
  console.log(i);
  i++;
}
// 0, 1, 2, 3, 4
```

### for vs while

```javascript
// for 문
for (let i = 0; i < 5; i++) {
  console.log(i);
}

// 동일한 while 문
let i = 0;
while (i < 5) {
  console.log(i);
  i++;
}
```

### 무한 루프와 break

```javascript
let count = 0;
while (true) {
  console.log(count);
  count++;

  if (count >= 5) {
    break;  // 루프 탈출
  }
}
// 0, 1, 2, 3, 4

// 실용 예제: 사용자 입력 대기
while (true) {
  const input = prompt('종료하려면 "exit"를 입력하세요:');

  if (input === 'exit') {
    break;
  }

  console.log(`입력하신 값: ${input}`);
}
```

## do...while 문 - 최소 1회 실행

### 기본 문법

```javascript
do {
  // 최소 1회 실행, 그 후 조건 검사
} while (조건식);
```

### while vs do...while

```javascript
// while: 조건이 처음부터 false면 한 번도 실행 안 됨
let i = 10;
while (i < 5) {
  console.log(i);  // 실행 안 됨
}

// do...while: 조건과 관계없이 최소 1회 실행
let j = 10;
do {
  console.log(j);  // 10 출력
} while (j < 5);
```

### 실용 예제

```javascript
// 사용자 입력 검증 (최소 1회는 물어봐야 함)
let password;
do {
  password = prompt('비밀번호를 입력하세요 (최소 6자):');
} while (password.length < 6);

console.log('비밀번호가 설정되었습니다.');
```

## for...of 문 - 이터러블 순회 (ES6)

### 기본 문법

```javascript
for (const 요소 of 이터러블) {
  // 각 요소에 대해 실행
}
```

### 배열 순회

```javascript
const fruits = ['🍎', '🍌', '🍇', '🍊'];

// for...of (권장)
for (const fruit of fruits) {
  console.log(fruit);
}

// 기존 for 문 (장황함)
for (let i = 0; i < fruits.length; i++) {
  console.log(fruits[i]);
}
```

### 문자열 순회

```javascript
const str = 'Hello';

for (const char of str) {
  console.log(char);
}
// 'H', 'e', 'l', 'l', 'o'
```

### Map, Set 순회

```javascript
// Map
const map = new Map([
  ['name', 'Alice'],
  ['age', 25]
]);

for (const [key, value] of map) {
  console.log(`${key}: ${value}`);
}
// "name: Alice"
// "age: 25"

// Set
const set = new Set([1, 2, 3, 4, 5]);

for (const num of set) {
  console.log(num);
}
// 1, 2, 3, 4, 5
```

### entries(), keys(), values()

```javascript
const arr = ['a', 'b', 'c'];

// entries(): 인덱스와 값
for (const [index, value] of arr.entries()) {
  console.log(`${index}: ${value}`);
}
// "0: a", "1: b", "2: c"

// keys(): 인덱스만
for (const index of arr.keys()) {
  console.log(index);
}
// 0, 1, 2

// values(): 값만 (기본 동작과 동일)
for (const value of arr.values()) {
  console.log(value);
}
// 'a', 'b', 'c'
```

## for...in 문 - 객체 프로퍼티 순회

### 기본 문법

```javascript
for (const 키 in 객체) {
  // 각 프로퍼티에 대해 실행
}
```

### 객체 순회

```javascript
const user = {
  name: 'Alice',
  age: 25,
  job: 'Developer'
};

for (const key in user) {
  console.log(`${key}: ${user[key]}`);
}
// "name: Alice"
// "age: 25"
// "job: Developer"
```

### 배열 순회 (비권장)

```javascript
const arr = ['a', 'b', 'c'];

// ⚠️ for...in (비권장: 배열에는 for...of 사용)
for (const index in arr) {
  console.log(index);  // "0", "1", "2" (문자열!)
  console.log(arr[index]);
}

// ✅ for...of (권장)
for (const value of arr) {
  console.log(value);
}
```

### hasOwnProperty() 사용

프로토타입 체인의 프로퍼티는 제외하고 순회합니다.

```javascript
const obj = {
  a: 1,
  b: 2
};

Object.prototype.c = 3;  // 프로토타입에 추가

// ❌ 프로토타입 프로퍼티도 포함
for (const key in obj) {
  console.log(key);  // "a", "b", "c"
}

// ✅ 자신의 프로퍼티만
for (const key in obj) {
  if (obj.hasOwnProperty(key)) {
    console.log(key);  // "a", "b"
  }
}

// ✅ 더 좋은 방법
for (const key of Object.keys(obj)) {
  console.log(key);  // "a", "b"
}
```

## forEach() - 배열 메서드

### 기본 문법

```javascript
배열.forEach((요소, 인덱스, 배열) => {
  // 각 요소에 대해 실행
});
```

### 기본 사용

```javascript
const numbers = [1, 2, 3, 4, 5];

// forEach
numbers.forEach((num) => {
  console.log(num * 2);
});

// 인덱스 사용
numbers.forEach((num, index) => {
  console.log(`${index}: ${num}`);
});
// "0: 1", "1: 2", "2: 3", ...
```

### for vs forEach

```javascript
const arr = ['a', 'b', 'c'];

// for 문
for (let i = 0; i < arr.length; i++) {
  console.log(arr[i]);
}

// for...of
for (const item of arr) {
  console.log(item);
}

// forEach
arr.forEach(item => {
  console.log(item);
});
```

### forEach의 제약

```javascript
// ❌ break, continue 사용 불가
const numbers = [1, 2, 3, 4, 5];

numbers.forEach(num => {
  if (num === 3) {
    break;  // SyntaxError!
  }
  console.log(num);
});

// ✅ 대신 return으로 현재 반복만 건너뛰기 (continue와 유사)
numbers.forEach(num => {
  if (num === 3) {
    return;  // 현재 반복만 건너뜀
  }
  console.log(num);
});
// 1, 2, 4, 5

// ✅ 루프 중단이 필요하면 for...of 사용
for (const num of numbers) {
  if (num === 3) {
    break;  // 루프 종료
  }
  console.log(num);
}
// 1, 2
```

## break와 continue

### break - 루프 즉시 종료

```javascript
// 특정 조건에서 루프 종료
for (let i = 0; i < 10; i++) {
  if (i === 5) {
    break;  // i가 5일 때 루프 종료
  }
  console.log(i);
}
// 0, 1, 2, 3, 4

// 실용 예제: 원하는 값 찾으면 중단
const users = [
  { id: 1, name: 'Alice' },
  { id: 2, name: 'Bob' },
  { id: 3, name: 'Charlie' }
];

let foundUser = null;
for (const user of users) {
  if (user.id === 2) {
    foundUser = user;
    break;  // 찾았으니 더 이상 검색 안 함
  }
}

console.log(foundUser);  // { id: 2, name: 'Bob' }
```

### continue - 현재 반복 건너뛰기

```javascript
// 홀수만 출력
for (let i = 0; i < 10; i++) {
  if (i % 2 === 0) {
    continue;  // 짝수는 건너뛰기
  }
  console.log(i);
}
// 1, 3, 5, 7, 9

// 실용 예제: 유효한 데이터만 처리
const data = [10, null, 20, undefined, 30, 0, 40];

for (const value of data) {
  if (value == null || value === 0) {
    continue;  // null, undefined, 0은 건너뛰기
  }
  console.log(value);
}
// 10, 20, 30, 40
```

## 중첩 루프

### 2차원 배열 순회

```javascript
const matrix = [
  [1, 2, 3],
  [4, 5, 6],
  [7, 8, 9]
];

for (let i = 0; i < matrix.length; i++) {
  for (let j = 0; j < matrix[i].length; j++) {
    console.log(`matrix[${i}][${j}] = ${matrix[i][j]}`);
  }
}
```

### 구구단 출력

```javascript
for (let i = 2; i <= 9; i++) {
  console.log(`=== ${i}단 ===`);
  for (let j = 1; j <= 9; j++) {
    console.log(`${i} x ${j} = ${i * j}`);
  }
}
```

### 레이블과 break

중첩 루프를 한 번에 빠져나가야 할 때 사용합니다.

```javascript
// ❌ 내부 루프만 종료
for (let i = 0; i < 3; i++) {
  for (let j = 0; j < 3; j++) {
    if (i === 1 && j === 1) {
      break;  // 내부 루프만 종료
    }
    console.log(`${i}, ${j}`);
  }
}

// ✅ 레이블로 외부 루프까지 종료
outer: for (let i = 0; i < 3; i++) {
  for (let j = 0; j < 3; j++) {
    if (i === 1 && j === 1) {
      break outer;  // 외부 루프까지 종료
    }
    console.log(`${i}, ${j}`);
  }
}
// 0,0  0,1  0,2  1,0
```

## 성능 비교

### 속도 테스트

```javascript
const arr = Array.from({ length: 1000000 }, (_, i) => i);

// 1. for 문 (가장 빠름)
console.time('for');
let sum1 = 0;
for (let i = 0; i < arr.length; i++) {
  sum1 += arr[i];
}
console.timeEnd('for');  // 약 5ms

// 2. for...of (약간 느림)
console.time('for...of');
let sum2 = 0;
for (const num of arr) {
  sum2 += num;
}
console.timeEnd('for...of');  // 약 10ms

// 3. forEach (가장 느림)
console.time('forEach');
let sum3 = 0;
arr.forEach(num => {
  sum3 += num;
});
console.timeEnd('forEach');  // 약 20ms
```

### 성능 비교표

| 반복문 | 속도 | 가독성 | break/continue | 용도 |
|--------|------|--------|----------------|------|
| **for** | ⚡⚡⚡ | ⭐⭐ | ✅ | 고성능 필요 시 |
| **while** | ⚡⚡⚡ | ⭐⭐⭐ | ✅ | 조건 기반 반복 |
| **for...of** | ⚡⚡ | ⭐⭐⭐⭐⭐ | ✅ | 일반적 배열 순회 |
| **for...in** | ⚡ | ⭐⭐⭐ | ✅ | 객체 프로퍼티 순회 |
| **forEach** | ⚡ | ⭐⭐⭐⭐ | ❌ | 함수형 스타일 |

## 실무 패턴

### 1. 배열 변환 (map과 비교)

```javascript
const numbers = [1, 2, 3, 4, 5];

// ❌ for 문 (장황함)
const doubled1 = [];
for (let i = 0; i < numbers.length; i++) {
  doubled1.push(numbers[i] * 2);
}

// ✅ map (간결함, 권장)
const doubled2 = numbers.map(n => n * 2);
```

### 2. 배열 필터링 (filter와 비교)

```javascript
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// ❌ for 문
const evens1 = [];
for (const num of numbers) {
  if (num % 2 === 0) {
    evens1.push(num);
  }
}

// ✅ filter (권장)
const evens2 = numbers.filter(n => n % 2 === 0);
```

### 3. 조기 종료가 필요한 검색

```javascript
const users = [/* 대량의 데이터 */];

// ✅ for...of with break (효율적)
function findUser(id) {
  for (const user of users) {
    if (user.id === id) {
      return user;  // 찾으면 즉시 반환
    }
  }
  return null;
}

// ⚠️ find 메서드도 좋음
const user = users.find(u => u.id === 123);

// ❌ forEach (비효율: 끝까지 순회함)
let foundUser = null;
users.forEach(user => {
  if (user.id === 123) {
    foundUser = user;  // 찾아도 계속 순회
  }
});
```

### 4. 역순 순회 (배열 수정 시)

```javascript
// 배열에서 요소 제거할 때
const arr = [1, 2, 3, 4, 5];

// ❌ 앞에서부터 제거 (인덱스 꼬임)
for (let i = 0; i < arr.length; i++) {
  if (arr[i] % 2 === 0) {
    arr.splice(i, 1);  // 인덱스가 꼬임!
  }
}

// ✅ 뒤에서부터 제거 (안전)
for (let i = arr.length - 1; i >= 0; i--) {
  if (arr[i] % 2 === 0) {
    arr.splice(i, 1);
  }
}

// ✅ filter 사용 (더 좋음)
const arr2 = arr.filter(n => n % 2 !== 0);
```

### 5. 비동기 반복

```javascript
const urls = ['/api/user/1', '/api/user/2', '/api/user/3'];

// ❌ forEach (병렬 실행, 순서 보장 안 됨)
urls.forEach(async (url) => {
  const data = await fetch(url).then(res => res.json());
  console.log(data);
});

// ✅ for...of (순차 실행)
for (const url of urls) {
  const data = await fetch(url).then(res => res.json());
  console.log(data);
}

// ✅ Promise.all (병렬 실행, 더 빠름)
const promises = urls.map(url => fetch(url).then(res => res.json()));
const results = await Promise.all(promises);
```

### 6. 청크 단위 처리

```javascript
function processInChunks(array, chunkSize, callback) {
  for (let i = 0; i < array.length; i += chunkSize) {
    const chunk = array.slice(i, i + chunkSize);
    callback(chunk, i);
  }
}

const data = Array.from({ length: 100 }, (_, i) => i);

processInChunks(data, 10, (chunk, startIndex) => {
  console.log(`청크 ${startIndex / 10 + 1}:`, chunk);
});
```

### 7. 객체 배열 그룹화

```javascript
const users = [
  { name: 'Alice', role: 'admin' },
  { name: 'Bob', role: 'user' },
  { name: 'Charlie', role: 'admin' },
  { name: 'Dave', role: 'user' }
];

// role별로 그룹화
const grouped = {};
for (const user of users) {
  if (!grouped[user.role]) {
    grouped[user.role] = [];
  }
  grouped[user.role].push(user);
}

console.log(grouped);
// {
//   admin: [{ name: 'Alice', ... }, { name: 'Charlie', ... }],
//   user: [{ name: 'Bob', ... }, { name: 'Dave', ... }]
// }

// ✅ reduce로 더 간결하게
const grouped2 = users.reduce((acc, user) => {
  acc[user.role] = acc[user.role] || [];
  acc[user.role].push(user);
  return acc;
}, {});
```

## Best Practices

### ✅ 권장 사항

```javascript
// 1. 배열 순회는 for...of 사용
for (const item of array) { }  // ✅

// 2. 객체 순회는 Object.keys/values/entries
for (const key of Object.keys(obj)) { }  // ✅
for (const [key, value] of Object.entries(obj)) { }  // ✅

// 3. 변환/필터링은 map/filter 사용
array.map(x => x * 2);  // ✅
array.filter(x => x > 0);  // ✅

// 4. 조기 종료가 필요하면 for...of + break
for (const item of array) {
  if (condition) break;  // ✅
}

// 5. 비동기 반복은 for...of
for (const item of array) {
  await processAsync(item);  // ✅
}
```

### ❌ 피해야 할 사항

```javascript
// 1. forEach에서 break 사용 (불가능)
array.forEach(item => {
  if (condition) break;  // ❌ SyntaxError
});

// 2. 배열에 for...in 사용
for (const i in array) { }  // ❌ for...of 사용

// 3. 루프 내에서 배열 길이 계산
for (let i = 0; i < array.length; i++) { }  // ⚠️ 괜찮지만
for (let i = 0, len = array.length; i < len; i++) { }  // ✅ 더 효율적

// 4. 무한 루프 조심
while (true) { }  // ❌ break 없으면 위험
```

## 언제 무엇을 사용할까?

### 의사결정 트리

```
배열을 순회해야 하는가?
├─ 변환이 필요한가? → map()
├─ 필터링이 필요한가? → filter()
├─ 집계가 필요한가? → reduce()
├─ 조기 종료가 필요한가? → for...of + break
├─ 비동기 처리가 필요한가? → for...of + await
└─ 단순 반복인가? → forEach() 또는 for...of

객체를 순회해야 하는가?
├─ 키만 필요한가? → Object.keys() + for...of
├─ 값만 필요한가? → Object.values() + for...of
└─ 키와 값 모두 필요한가? → Object.entries() + for...of

특정 횟수만 반복하는가?
└─ for (let i = 0; i < n; i++)

조건이 true인 동안 반복하는가?
└─ while (condition)

최소 1회는 실행해야 하는가?
└─ do...while
```

## 핵심 정리

### 반복문 종류

1. **for**: 전통적, 고성능, 세밀한 제어
2. **while**: 조건 기반 반복
3. **do...while**: 최소 1회 실행
4. **for...of**: 이터러블 순회 (배열, 문자열, Map, Set)
5. **for...in**: 객체 프로퍼티 순회
6. **forEach**: 함수형 스타일 (break 불가)

### 실무 추천

- **일반 배열 순회**: `for...of`
- **객체 순회**: `Object.keys/values/entries` + `for...of`
- **변환**: `map()`
- **필터링**: `filter()`
- **집계**: `reduce()`
- **조기 종료 필요**: `for...of` + `break`
- **비동기**: `for...of` + `await`
- **고성능 필요**: 기본 `for` 문

> **결론**: 대부분의 경우 `for...of`, `map`, `filter`를 사용하고, 조기 종료나 고성능이 필요할 때만 기본 `for` 문을 사용하세요!
{: .prompt-tip }

## 참고 자료

- [MDN - Loops and iteration](https://developer.mozilla.org/ko/docs/Web/JavaScript/Guide/Loops_and_iteration)
- [MDN - for...of](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Statements/for...of)
- [MDN - for...in](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Statements/for...in)
- [MDN - Array.prototype.forEach()](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach)
- [JavaScript.info - Loops](https://javascript.info/while-for)
