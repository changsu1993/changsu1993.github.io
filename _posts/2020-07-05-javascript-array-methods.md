---
title: JavaScript Array Methods 완벽 가이드 - 실무 필수 배열 메서드 정복하기
date: 2020-07-05 21:49:00 +0900
categories: [개발, JavaScript]
tags: [javascript, array, array-methods]
author: changsu
toc: true
comments: true
description: JavaScript 배열 메서드 완벽 정리. map, filter, reduce, forEach, find 등 실무에서 필수적인 배열 조작 메서드를 실전 예제와 함께 마스터합니다.
---

## 들어가며

JavaScript 배열(Array)은 가장 자주 사용하는 자료구조 중 하나입니다.

배열을 효율적으로 다루는 방법을 알면 **코드의 가독성과 생산성**이 크게 향상됩니다. 특히 **함수형 프로그래밍 스타일**의 배열 메서드를 활용하면 더 간결하고 읽기 쉬운 코드를 작성할 수 있습니다.

> 배열 메서드는 대부분 **Arrow Function**과 함께 **콜백 함수**로 사용됩니다!
{: .prompt-info }

## 콜백 함수(Callback Function)란?

배열 메서드를 이해하기 전에 먼저 **콜백 함수**의 개념을 알아야 합니다.

```javascript
// 콜백 함수: 다른 함수의 인자로 전달되는 함수
function greeting(name) {
  console.log(`Hello, ${name}!`);
}

function processUserInput(callback) {
  const name = 'John';
  callback(name);  // greeting 함수를 콜백으로 실행
}

processUserInput(greeting);  // 'Hello, John!'
```

> **콜백 함수**는 인자로 전달되어 특정 시점에 호출되는 함수입니다.
{: .prompt-tip }

## 배열 순회 메서드

### forEach() - 기본 반복문

**`forEach`**는 `for`문을 대체하는 배열 반복 메서드입니다.

#### 기본 문법

```javascript
array.forEach((element, index, array) => {
  // 각 요소에 대해 실행할 코드
});
```

**매개변수:**
- `element`: 현재 처리 중인 요소
- `index`: 현재 요소의 인덱스 (선택)
- `array`: 원본 배열 (선택)

#### 기본 예제

```javascript
const numbers = [1, 2, 3, 4, 5];

// 각 요소 출력
numbers.forEach(num => {
  console.log(num);
});
// 1, 2, 3, 4, 5

// 인덱스와 함께 출력
numbers.forEach((num, index) => {
  console.log(`Index ${index}: ${num}`);
});
// Index 0: 1
// Index 1: 2
// ...
```

#### 실전 예제

```javascript
const names = ['Alice', 'Bob', 'Charlie', 'David'];
const startsWithA = [];

// 'A'로 시작하는 이름 필터링
names.forEach(name => {
  if (name.startsWith('A')) {
    startsWithA.push(name);
  }
});

console.log(startsWithA);  // ['Alice']
```

#### forEach의 특징

| 특징 | 설명 |
|------|------|
| **반환값** | `undefined` (아무것도 반환하지 않음) |
| **중단** | `return`으로 현재 반복만 건너뜀 (break 불가) |
| **용도** | 각 요소에 대해 **부수 효과(Side Effect)** 수행 |

```javascript
// 조기 종료 예제
const arr = ['a', 'b', 'c', 'd'];
let foundC = false;

arr.forEach(el => {
  if (el === 'c') {
    foundC = true;
    return;  // 현재 반복만 종료, 전체 루프는 계속
  }
  console.log(el);
});
// 출력: 'a', 'b', 'd'
```

> ⚠️ `forEach`는 `break`나 `continue`를 사용할 수 없습니다. 조기 종료가 필요하면 `for...of`를 사용하세요!
{: .prompt-warning }

### map() - 배열 변환

**`map`**은 배열의 각 요소를 변환하여 **새로운 배열**을 반환합니다.

#### 기본 문법

```javascript
const newArray = array.map((element, index, array) => {
  return transformedElement;
});
```

#### 기본 예제

```javascript
const numbers = [1, 2, 3, 4, 5];

// 각 요소를 제곱
const squares = numbers.map(num => num * num);
console.log(squares);  // [1, 4, 9, 16, 25]

// 원본 배열은 변경되지 않음
console.log(numbers);  // [1, 2, 3, 4, 5]
```

#### 실전 예제

```javascript
// 사용자 데이터에서 이름만 추출
const users = [
  { id: 1, name: 'Alice', age: 25 },
  { id: 2, name: 'Bob', age: 30 },
  { id: 3, name: 'Charlie', age: 35 }
];

const names = users.map(user => user.name);
console.log(names);  // ['Alice', 'Bob', 'Charlie']

// 객체 변환
const userSummaries = users.map(user => ({
  name: user.name,
  isAdult: user.age >= 18
}));
console.log(userSummaries);
// [
//   { name: 'Alice', isAdult: true },
//   { name: 'Bob', isAdult: true },
//   { name: 'Charlie', isAdult: true }
// ]
```

#### map vs forEach 비교

```javascript
const arr = [1, 2, 3];

// forEach: 반환값 없음
const result1 = arr.forEach(x => x * 2);
console.log(result1);  // undefined

// map: 새 배열 반환
const result2 = arr.map(x => x * 2);
console.log(result2);  // [2, 4, 6]
```

| 구분 | forEach | map |
|------|---------|-----|
| **반환값** | `undefined` | 새 배열 |
| **용도** | 부수 효과 수행 | 배열 변환 |
| **체이닝** | 불가능 | 가능 |

> 💡 **선택 기준**: 새로운 배열이 필요하면 `map`, 단순 반복 작업이면 `forEach`!
{: .prompt-tip }

## 배열 필터링 메서드

### filter() - 조건에 맞는 요소만 추출

**`filter`**는 조건을 만족하는 요소들로만 구성된 **새 배열**을 반환합니다.

#### 기본 문법

```javascript
const newArray = array.filter((element, index, array) => {
  return condition;  // true면 포함, false면 제외
});
```

#### 기본 예제

```javascript
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// 짝수만 필터링
const evenNumbers = numbers.filter(num => num % 2 === 0);
console.log(evenNumbers);  // [2, 4, 6, 8, 10]

// 5보다 큰 수
const greaterThanFive = numbers.filter(num => num > 5);
console.log(greaterThanFive);  // [6, 7, 8, 9, 10]
```

#### 실전 예제

```javascript
const products = [
  { name: 'Laptop', price: 1200, inStock: true },
  { name: 'Phone', price: 800, inStock: false },
  { name: 'Tablet', price: 500, inStock: true },
  { name: 'Monitor', price: 300, inStock: true }
];

// 재고가 있는 상품만
const availableProducts = products.filter(product => product.inStock);
console.log(availableProducts);
// [
//   { name: 'Laptop', price: 1200, inStock: true },
//   { name: 'Tablet', price: 500, inStock: true },
//   { name: 'Monitor', price: 300, inStock: true }
// ]

// 가격이 600 이하인 재고 상품
const affordableProducts = products.filter(product =>
  product.price <= 600 && product.inStock
);
console.log(affordableProducts);
// [
//   { name: 'Tablet', price: 500, inStock: true },
//   { name: 'Monitor', price: 300, inStock: true }
// ]
```

#### map + filter 체이닝

```javascript
const users = [
  { name: 'Alice', age: 17 },
  { name: 'Bob', age: 25 },
  { name: 'Charlie', age: 16 },
  { name: 'David', age: 30 }
];

// 성인 사용자의 이름만 추출
const adultNames = users
  .filter(user => user.age >= 18)
  .map(user => user.name);

console.log(adultNames);  // ['Bob', 'David']
```

> 💡 **메서드 체이닝**: `filter`, `map`, `reduce` 등은 체이닝하여 강력한 데이터 처리 파이프라인을 만들 수 있습니다!
{: .prompt-tip }

## 배열 검색 메서드

### find() - 조건에 맞는 첫 번째 요소

**`find`**는 조건을 만족하는 **첫 번째 요소**를 반환합니다.

#### 기본 문법

```javascript
const element = array.find((element, index, array) => {
  return condition;
});
```

#### 기본 예제

```javascript
const numbers = [5, 12, 8, 130, 44];

// 10보다 큰 첫 번째 수
const found = numbers.find(num => num > 10);
console.log(found);  // 12

// 조건을 만족하는 요소가 없으면 undefined
const notFound = numbers.find(num => num > 200);
console.log(notFound);  // undefined
```

#### 실전 예제

```javascript
const users = [
  { id: 1, name: 'Alice', email: 'alice@example.com' },
  { id: 2, name: 'Bob', email: 'bob@example.com' },
  { id: 3, name: 'Charlie', email: 'charlie@example.com' }
];

// ID로 사용자 찾기
const user = users.find(u => u.id === 2);
console.log(user);  // { id: 2, name: 'Bob', email: 'bob@example.com' }

// 이메일로 사용자 찾기
const userByEmail = users.find(u => u.email === 'alice@example.com');
console.log(userByEmail);  // { id: 1, name: 'Alice', ... }
```

### findIndex() - 조건에 맞는 첫 번째 요소의 인덱스

```javascript
const numbers = [5, 12, 8, 130, 44];

const index = numbers.findIndex(num => num > 10);
console.log(index);  // 1 (12의 인덱스)

// 못 찾으면 -1 반환
const notFoundIndex = numbers.findIndex(num => num > 200);
console.log(notFoundIndex);  // -1
```

### find vs filter 비교

| 구분 | find | filter |
|------|------|--------|
| **반환값** | 첫 번째 요소 (또는 undefined) | 모든 일치 요소의 배열 |
| **성능** | 빠름 (찾으면 즉시 중단) | 느림 (전체 탐색) |
| **용도** | 하나의 요소 찾기 | 여러 요소 필터링 |

## 배열 판별 메서드

### some() - 하나라도 조건 만족?

**`some`**은 **하나 이상**의 요소가 조건을 만족하면 `true`를 반환합니다.

```javascript
const numbers = [1, 2, 3, 4, 5];

// 짝수가 하나라도 있는가?
const hasEven = numbers.some(num => num % 2 === 0);
console.log(hasEven);  // true

// 10보다 큰 수가 있는가?
const hasGreaterThan10 = numbers.some(num => num > 10);
console.log(hasGreaterThan10);  // false
```

### every() - 모두 조건 만족?

**`every`**는 **모든** 요소가 조건을 만족해야 `true`를 반환합니다.

```javascript
const numbers = [2, 4, 6, 8, 10];

// 모두 짝수인가?
const allEven = numbers.every(num => num % 2 === 0);
console.log(allEven);  // true

// 모두 5보다 큰가?
const allGreaterThan5 = numbers.every(num => num > 5);
console.log(allGreaterThan5);  // false
```

#### 실전 예제

```javascript
const users = [
  { name: 'Alice', age: 25, verified: true },
  { name: 'Bob', age: 30, verified: true },
  { name: 'Charlie', age: 17, verified: false }
];

// 한 명이라도 미성년자인가?
const hasMinor = users.some(user => user.age < 18);
console.log(hasMinor);  // true

// 모두 인증된 사용자인가?
const allVerified = users.every(user => user.verified);
console.log(allVerified);  // false
```

## reduce() - 배열 축약하기

**`reduce`**는 배열의 모든 요소를 하나의 값으로 축약(reduce)합니다.

### 기본 문법

```javascript
const result = array.reduce((accumulator, currentValue, index, array) => {
  return newAccumulator;
}, initialValue);
```

**매개변수:**
- `accumulator`: 누적값
- `currentValue`: 현재 처리 중인 요소
- `initialValue`: 초기값 (선택, 권장)

### 기본 예제: 합계 구하기

```javascript
const numbers = [1, 2, 3, 4, 5];

// 배열의 합
const sum = numbers.reduce((acc, num) => acc + num, 0);
console.log(sum);  // 15

// 동작 과정:
// acc = 0, num = 1 → return 1
// acc = 1, num = 2 → return 3
// acc = 3, num = 3 → return 6
// acc = 6, num = 4 → return 10
// acc = 10, num = 5 → return 15
```

### 실전 예제

#### 1. 장바구니 총액 계산

```javascript
const cart = [
  { name: 'Laptop', price: 1200, quantity: 1 },
  { name: 'Phone', price: 800, quantity: 2 },
  { name: 'Tablet', price: 500, quantity: 1 }
];

const total = cart.reduce((sum, item) => {
  return sum + (item.price * item.quantity);
}, 0);

console.log(total);  // 3300
```

#### 2. 배열을 객체로 변환

```javascript
const fruits = ['apple', 'banana', 'apple', 'orange', 'banana', 'apple'];

// 과일 개수 세기
const fruitCount = fruits.reduce((counts, fruit) => {
  counts[fruit] = (counts[fruit] || 0) + 1;
  return counts;
}, {});

console.log(fruitCount);
// { apple: 3, banana: 2, orange: 1 }
```

#### 3. 중첩 배열 평탄화

```javascript
const nestedArray = [[1, 2], [3, 4], [5, 6]];

const flattened = nestedArray.reduce((flat, arr) => {
  return flat.concat(arr);
}, []);

console.log(flattened);  // [1, 2, 3, 4, 5, 6]
```

#### 4. 함수 합성 (고급)

```javascript
const double = x => x * 2;
const square = x => x * x;
const addTen = x => x + 10;

const compose = (...fns) => x =>
  fns.reduceRight((acc, fn) => fn(acc), x);

const calculate = compose(addTen, square, double);
console.log(calculate(5));  // ((5 * 2) ^ 2) + 10 = 110
```

> 💡 **reduce의 힘**: 거의 모든 배열 연산을 reduce로 구현할 수 있습니다!
{: .prompt-tip }

## 기타 유용한 배열 메서드

### includes() - 요소 포함 여부

```javascript
const fruits = ['apple', 'banana', 'orange'];

console.log(fruits.includes('banana'));  // true
console.log(fruits.includes('grape'));   // false
```

### sort() - 배열 정렬

```javascript
const numbers = [3, 1, 4, 1, 5, 9, 2, 6];

// 오름차순
numbers.sort((a, b) => a - b);
console.log(numbers);  // [1, 1, 2, 3, 4, 5, 6, 9]

// 내림차순
numbers.sort((a, b) => b - a);
console.log(numbers);  // [9, 6, 5, 4, 3, 2, 1, 1]

// 객체 배열 정렬
const users = [
  { name: 'Charlie', age: 35 },
  { name: 'Alice', age: 25 },
  { name: 'Bob', age: 30 }
];

users.sort((a, b) => a.age - b.age);
// 나이순 정렬: Alice(25), Bob(30), Charlie(35)
```

> ⚠️ **주의**: `sort()`는 **원본 배열을 변경**합니다!
{: .prompt-warning }

### slice() - 배열 일부 추출

```javascript
const arr = [1, 2, 3, 4, 5];

const sliced = arr.slice(1, 4);  // 인덱스 1부터 3까지 (4 제외)
console.log(sliced);  // [2, 3, 4]
console.log(arr);     // [1, 2, 3, 4, 5] (원본 유지)
```

### concat() - 배열 합치기

```javascript
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];

const combined = arr1.concat(arr2);
console.log(combined);  // [1, 2, 3, 4, 5, 6]

// Spread 연산자 사용 (모던한 방법)
const combined2 = [...arr1, ...arr2];
console.log(combined2);  // [1, 2, 3, 4, 5, 6]
```

### join() - 배열을 문자열로

```javascript
const words = ['Hello', 'World', '!'];

const sentence = words.join(' ');
console.log(sentence);  // 'Hello World !'

const csv = ['apple', 'banana', 'orange'].join(',');
console.log(csv);  // 'apple,banana,orange'
```

## 실전 종합 예제

### 데이터 처리 파이프라인

```javascript
const transactions = [
  { id: 1, type: 'income', amount: 5000, date: '2024-01-15' },
  { id: 2, type: 'expense', amount: 2000, date: '2024-01-16' },
  { id: 3, type: 'income', amount: 3000, date: '2024-01-17' },
  { id: 4, type: 'expense', amount: 1500, date: '2024-01-18' },
  { id: 5, type: 'income', amount: 4000, date: '2024-01-19' }
];

// 1. 수입만 필터링
// 2. 금액에 10% 세금 적용
// 3. 총 세후 수입 계산
const totalNetIncome = transactions
  .filter(t => t.type === 'income')
  .map(t => t.amount * 0.9)
  .reduce((sum, amount) => sum + amount, 0);

console.log(totalNetIncome);  // 10800

// 한 줄로 작성 가능!
const result = transactions
  .filter(t => t.type === 'income')
  .reduce((sum, t) => sum + t.amount * 0.9, 0);
```

### E-commerce 상품 필터링

```javascript
const products = [
  { id: 1, name: 'Laptop', price: 1200, category: 'Electronics', rating: 4.5 },
  { id: 2, name: 'Phone', price: 800, category: 'Electronics', rating: 4.7 },
  { id: 3, name: 'Shirt', price: 50, category: 'Clothing', rating: 4.2 },
  { id: 4, name: 'Tablet', price: 500, category: 'Electronics', rating: 4.6 },
  { id: 5, name: 'Jeans', price: 80, category: 'Clothing', rating: 4.0 }
];

// 전자제품 중 평점 4.5 이상인 제품의 이름
const topElectronics = products
  .filter(p => p.category === 'Electronics' && p.rating >= 4.5)
  .map(p => p.name);

console.log(topElectronics);  // ['Laptop', 'Phone', 'Tablet']

// 카테고리별 평균 가격
const avgPriceByCategory = products.reduce((acc, product) => {
  const cat = product.category;
  if (!acc[cat]) {
    acc[cat] = { sum: 0, count: 0 };
  }
  acc[cat].sum += product.price;
  acc[cat].count++;
  return acc;
}, {});

Object.keys(avgPriceByCategory).forEach(category => {
  const { sum, count } = avgPriceByCategory[category];
  console.log(`${category}: ${sum / count}`);
});
// Electronics: 833.33
// Clothing: 65
```

## 성능 고려사항

### 메서드 체이닝 vs 단일 루프

```javascript
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// ❌ 비효율적: 배열을 3번 순회
const result1 = numbers
  .filter(n => n > 5)      // 1번째 순회
  .map(n => n * 2)         // 2번째 순회
  .reduce((sum, n) => sum + n, 0);  // 3번째 순회

// ✅ 효율적: 배열을 1번만 순회
const result2 = numbers.reduce((sum, n) => {
  if (n > 5) {
    sum += n * 2;
  }
  return sum;
}, 0);
```

> 💡 **팁**: 작은 배열(<1000개)은 가독성을 위해 체이닝, 큰 배열은 성능을 위해 단일 루프를 고려하세요!
{: .prompt-tip }

## 핵심 정리

### 자주 사용하는 메서드 비교

| 메서드 | 반환값 | 원본 변경 | 용도 |
|--------|--------|----------|------|
| **forEach** | `undefined` | ❌ | 각 요소 순회 |
| **map** | 새 배열 | ❌ | 배열 변환 |
| **filter** | 새 배열 | ❌ | 조건에 맞는 요소만 |
| **reduce** | 단일 값 | ❌ | 배열 축약 |
| **find** | 요소 또는 `undefined` | ❌ | 첫 번째 요소 찾기 |
| **some** | `boolean` | ❌ | 하나라도 만족? |
| **every** | `boolean` | ❌ | 모두 만족? |
| **sort** | 정렬된 배열 | ✅ | 배열 정렬 |
| **slice** | 새 배열 | ❌ | 배열 일부 추출 |

### 메서드 선택 가이드

```
무엇을 하고 싶은가?
├─ 각 요소 처리만? → forEach
├─ 새 배열로 변환? → map
├─ 조건에 맞는 요소만? → filter
├─ 하나의 값으로 축약? → reduce
├─ 특정 요소 찾기? → find/findIndex
├─ 조건 판별? → some/every
└─ 배열 일부만? → slice
```

### Best Practices

1. **불변성 유지**: 원본 배열을 변경하지 않는 메서드 선호
2. **체이닝 활용**: 여러 메서드를 연결하여 파이프라인 구성
3. **가독성 우선**: 성능이 문제되지 않으면 가독성 있는 코드 작성
4. **적절한 메서드 선택**: 목적에 맞는 메서드 사용

> 배열 메서드를 마스터하면 더 간결하고 표현력 있는 코드를 작성할 수 있습니다!
{: .prompt-info }

## 참고 자료

- [MDN - Array](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array)
- [JavaScript.info - Array methods](https://ko.javascript.info/array-methods)
- [You Don't Know JS - Array Methods](https://github.com/getify/You-Dont-Know-JS)
