---
title: JavaScript 배열(Array) 완벽 가이드 - 기초부터 고급까지
description: JavaScript 배열의 기본 개념부터 배열 메서드, 고차 함수, 배열 조작 기법까지 상세히 알아봅니다. map, filter, reduce를 활용한 함수형 프로그래밍과 실전 예제를 마스터합니다.
author: changsu
date: 2020-06-21 18:18:01 +0900
categories: [Programming, JavaScript]
tags: [javascript, array, map, filter, reduce, foreach, array-methods, 배열]
---

## 배열이란?

**배열(Array)**은 여러 개의 값을 순서대로 저장하는 자료구조입니다. 하나의 변수에 여러 데이터를 담을 수 있어 효율적인 데이터 관리가 가능합니다.

### 배열의 필요성

```javascript
// ❌ 배열 없이 (비효율적)
let city1 = "서울";
let city2 = "대전";
let city3 = "대구";
let city4 = "부산";
let city5 = "광주";
// ... 수백 개의 변수?

// ✅ 배열 사용 (효율적)
let cities = ["서울", "대전", "대구", "부산", "광주"];
```

### 배열의 장점

| 장점 | 설명 |
|------|------|
| **효율성** | 하나의 변수로 여러 값 관리 |
| **순서 보장** | 데이터의 순서가 유지됨 |
| **반복 처리** | 루프로 쉽게 순회 가능 |
| **동적 크기** | 크기를 자유롭게 조절 |

## 배열 생성

### 배열 리터럴 (권장)

```javascript
// 빈 배열
let emptyArray = [];

// 데이터가 있는 배열
let cities = ["서울", "대전", "대구", "부산", "광주", "제주도"];
let kospi = [2062.82, 2053.2, 2045.92, 2058.82, 2053.12, 2055.7];
let mixed = ["텍스트", 123, true, null];
```

### Array 생성자

```javascript
// 빈 배열
let arr1 = new Array();

// 길이가 5인 빈 배열
let arr2 = new Array(5);
console.log(arr2);        // [empty × 5]
console.log(arr2.length); // 5

// 요소를 지정한 배열
let arr3 = new Array(1, 2, 3);
console.log(arr3);  // [1, 2, 3]
```

### Array.of() (ES6)

```javascript
// 생성자와 달리 항상 요소로 처리
let arr1 = Array.of(5);      // [5]
let arr2 = Array.of(1, 2, 3); // [1, 2, 3]
```

## 배열 요소와 인덱스

### 요소(Element)

배열의 각 값을 **요소(Element)**라고 합니다.

```javascript
let cities = ["서울", "대전", "대구", "부산", "광주"];
// "서울", "대전", "대구", "부산", "광주" 각각이 요소
```

### 인덱스(Index)

배열의 각 요소는 **인덱스(Index)**라는 위치 번호를 가집니다. **인덱스는 0부터 시작**합니다.

```javascript
let cities = ["서울", "대전", "대구", "부산", "광주"];
//  인덱스:    0      1      2      3      4

console.log(cities[0]);  // "서울"
console.log(cities[1]);  // "대전"
console.log(cities[4]);  // "광주"
```

### 요소 접근

```javascript
let numbers = [10, 20, 30, 40, 50];

// 첫 번째 요소
console.log(numbers[0]);  // 10

// 마지막 요소
console.log(numbers[numbers.length - 1]);  // 50

// 존재하지 않는 인덱스
console.log(numbers[10]);  // undefined
```

### 요소 수정

```javascript
let fruits = ["사과", "바나나", "오렌지"];

fruits[1] = "포도";
console.log(fruits);  // ["사과", "포도", "오렌지"]

// 존재하지 않는 인덱스에 할당
fruits[5] = "딸기";
console.log(fruits);  // ["사과", "포도", "오렌지", empty × 2, "딸기"]
console.log(fruits.length);  // 6
```

## 배열 길이

```javascript
let cities = ["서울", "대전", "대구", "부산", "광주"];

console.log(cities.length);  // 5

// 길이 변경 (권장하지 않음)
cities.length = 3;
console.log(cities);  // ["서울", "대전", "대구"]

cities.length = 5;
console.log(cities);  // ["서울", "대전", "대구", empty × 2]
```

## 다양한 타입의 요소

배열은 어떤 타입의 값도 저장할 수 있습니다.

```javascript
let mixed = [
  "문자열",           // String
  123,               // Number
  true,              // Boolean
  null,              // null
  undefined,         // undefined
  { name: "홍길동" }, // Object
  [1, 2, 3],         // Array (중첩 배열)
  function() {}      // Function
];

console.log(mixed[0]);  // "문자열"
console.log(mixed[5]);  // { name: "홍길동" }
console.log(mixed[6]);  // [1, 2, 3]
```

### 중첩 배열 (다차원 배열)

```javascript
let anything = ["대전", 1987, ["하나", "둘", 3]];

console.log(anything[0]);     // "대전"
console.log(anything[1]);     // 1987
console.log(anything[2]);     // ["하나", "둘", 3]
console.log(anything[2][0]);  // "하나" (중첩 배열의 첫 번째 요소)
console.log(anything[2][2]);  // 3

// 2차원 배열
let matrix = [
  [1, 2, 3],
  [4, 5, 6],
  [7, 8, 9]
];

console.log(matrix[0][0]);  // 1
console.log(matrix[1][2]);  // 6
console.log(matrix[2][1]);  // 8
```

## 배열 메서드

### 요소 추가/제거

#### push() - 끝에 추가

```javascript
let fruits = ["사과", "바나나"];

fruits.push("오렌지");
console.log(fruits);  // ["사과", "바나나", "오렌지"]

// 여러 개 추가
fruits.push("포도", "딸기");
console.log(fruits);  // ["사과", "바나나", "오렌지", "포도", "딸기"]

// 반환값: 새로운 배열 길이
let newLength = fruits.push("수박");
console.log(newLength);  // 6
```

#### pop() - 끝에서 제거

```javascript
let fruits = ["사과", "바나나", "오렌지"];

let removed = fruits.pop();
console.log(removed);  // "오렌지"
console.log(fruits);   // ["사과", "바나나"]
```

#### unshift() - 앞에 추가

```javascript
let fruits = ["바나나", "오렌지"];

fruits.unshift("사과");
console.log(fruits);  // ["사과", "바나나", "오렌지"]

// 여러 개 추가
fruits.unshift("포도", "딸기");
console.log(fruits);  // ["포도", "딸기", "사과", "바나나", "오렌지"]
```

#### shift() - 앞에서 제거

```javascript
let fruits = ["사과", "바나나", "오렌지"];

let removed = fruits.shift();
console.log(removed);  // "사과"
console.log(fruits);   // ["바나나", "오렌지"]
```

### 배열 조작

#### slice() - 부분 배열 추출 (원본 유지)

```javascript
let fruits = ["사과", "바나나", "오렌지", "포도", "딸기"];

let sliced1 = fruits.slice(1, 3);
console.log(sliced1);  // ["바나나", "오렌지"]

let sliced2 = fruits.slice(2);
console.log(sliced2);  // ["오렌지", "포도", "딸기"]

let sliced3 = fruits.slice(-2);
console.log(sliced3);  // ["포도", "딸기"]

console.log(fruits);   // 원본 유지: ["사과", "바나나", "오렌지", "포도", "딸기"]
```

#### splice() - 요소 추가/제거 (원본 변경)

```javascript
let fruits = ["사과", "바나나", "오렌지", "포도"];

// 요소 제거
let removed = fruits.splice(1, 2);
console.log(removed);  // ["바나나", "오렌지"]
console.log(fruits);   // ["사과", "포도"]

// 요소 추가
fruits.splice(1, 0, "딸기", "수박");
console.log(fruits);   // ["사과", "딸기", "수박", "포도"]

// 요소 교체
fruits.splice(1, 2, "블루베리");
console.log(fruits);   // ["사과", "블루베리", "포도"]
```

#### concat() - 배열 결합

```javascript
let arr1 = [1, 2, 3];
let arr2 = [4, 5, 6];
let arr3 = [7, 8, 9];

let combined = arr1.concat(arr2, arr3);
console.log(combined);  // [1, 2, 3, 4, 5, 6, 7, 8, 9]

// 스프레드 연산자 사용 (ES6)
let combined2 = [...arr1, ...arr2, ...arr3];
console.log(combined2);  // [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

#### join() - 문자열로 변환

```javascript
let fruits = ["사과", "바나나", "오렌지"];

console.log(fruits.join());      // "사과,바나나,오렌지"
console.log(fruits.join(" "));   // "사과 바나나 오렌지"
console.log(fruits.join(" - ")); // "사과 - 바나나 - 오렌지"
console.log(fruits.join(""));    // "사과바나나오렌지"
```

#### reverse() - 순서 뒤집기 (원본 변경)

```javascript
let numbers = [1, 2, 3, 4, 5];

numbers.reverse();
console.log(numbers);  // [5, 4, 3, 2, 1]
```

#### sort() - 정렬 (원본 변경)

```javascript
// 문자열 정렬
let fruits = ["바나나", "사과", "오렌지"];
fruits.sort();
console.log(fruits);  // ["바나나", "사과", "오렌지"]

// 숫자 정렬 (주의!)
let numbers = [10, 5, 40, 25, 100];
numbers.sort();
console.log(numbers);  // [10, 100, 25, 40, 5] (문자열로 정렬됨)

// 숫자 올바르게 정렬
numbers.sort((a, b) => a - b);
console.log(numbers);  // [5, 10, 25, 40, 100]

// 내림차순
numbers.sort((a, b) => b - a);
console.log(numbers);  // [100, 40, 25, 10, 5]
```

### 검색 메서드

#### indexOf() / lastIndexOf()

```javascript
let fruits = ["사과", "바나나", "오렌지", "바나나", "포도"];

console.log(fruits.indexOf("바나나"));      // 1 (첫 번째 위치)
console.log(fruits.lastIndexOf("바나나"));  // 3 (마지막 위치)
console.log(fruits.indexOf("딸기"));        // -1 (없음)
```

#### includes() - ES2016

```javascript
let fruits = ["사과", "바나나", "오렌지"];

console.log(fruits.includes("바나나"));  // true
console.log(fruits.includes("딸기"));    // false
```

#### find() / findIndex() - ES6

```javascript
let users = [
  { id: 1, name: "홍길동" },
  { id: 2, name: "김철수" },
  { id: 3, name: "이영희" }
];

// find: 조건에 맞는 첫 번째 요소
let user = users.find(user => user.id === 2);
console.log(user);  // { id: 2, name: "김철수" }

// findIndex: 조건에 맞는 첫 번째 요소의 인덱스
let index = users.findIndex(user => user.name === "이영희");
console.log(index);  // 2
```

## 배열 순회

### for 문

```javascript
let fruits = ["사과", "바나나", "오렌지"];

for (let i = 0; i < fruits.length; i++) {
  console.log(fruits[i]);
}
```

### for...of (ES6)

```javascript
let fruits = ["사과", "바나나", "오렌지"];

for (let fruit of fruits) {
  console.log(fruit);
}
```

### forEach()

```javascript
let fruits = ["사과", "바나나", "오렌지"];

fruits.forEach(function(fruit, index) {
  console.log(index, fruit);
});

// 화살표 함수
fruits.forEach((fruit, index) => {
  console.log(`${index}: ${fruit}`);
});
```

## 고차 함수 (Higher-Order Functions)

### map() - 변환

```javascript
let numbers = [1, 2, 3, 4, 5];

// 각 요소를 2배로
let doubled = numbers.map(num => num * 2);
console.log(doubled);  // [2, 4, 6, 8, 10]

// 객체 배열 변환
let users = [
  { name: "홍길동", age: 30 },
  { name: "김철수", age: 25 }
];

let names = users.map(user => user.name);
console.log(names);  // ["홍길동", "김철수"]
```

### filter() - 필터링

```javascript
let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// 짝수만
let evens = numbers.filter(num => num % 2 === 0);
console.log(evens);  // [2, 4, 6, 8, 10]

// 5보다 큰 수
let greaterThan5 = numbers.filter(num => num > 5);
console.log(greaterThan5);  // [6, 7, 8, 9, 10]

// 객체 필터링
let users = [
  { name: "홍길동", age: 30, active: true },
  { name: "김철수", age: 25, active: false },
  { name: "이영희", age: 35, active: true }
];

let activeUsers = users.filter(user => user.active);
console.log(activeUsers);
// [{ name: "홍길동", age: 30, active: true },
//  { name: "이영희", age: 35, active: true }]
```

### reduce() - 집계

```javascript
let numbers = [1, 2, 3, 4, 5];

// 합계
let sum = numbers.reduce((acc, num) => acc + num, 0);
console.log(sum);  // 15

// 곱셈
let product = numbers.reduce((acc, num) => acc * num, 1);
console.log(product);  // 120

// 최대값
let max = numbers.reduce((acc, num) => Math.max(acc, num));
console.log(max);  // 5

// 객체로 변환
let fruits = ["사과", "바나나", "오렌지"];
let fruitObj = fruits.reduce((acc, fruit, index) => {
  acc[index] = fruit;
  return acc;
}, {});
console.log(fruitObj);  // { 0: "사과", 1: "바나나", 2: "오렌지" }
```

### 메서드 체이닝

```javascript
let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// 짝수만 필터링 → 2배로 → 합계
let result = numbers
  .filter(num => num % 2 === 0)
  .map(num => num * 2)
  .reduce((acc, num) => acc + num, 0);

console.log(result);  // 60 (2+4+6+8+10) * 2 = 60
```

## 실전 예제

### 1. 장바구니 합계

```javascript
let cart = [
  { name: "노트북", price: 1500000, quantity: 1 },
  { name: "마우스", price: 30000, quantity: 2 },
  { name: "키보드", price: 80000, quantity: 1 }
];

let total = cart.reduce((sum, item) => {
  return sum + (item.price * item.quantity);
}, 0);

console.log(`총 금액: ${total.toLocaleString()}원`);
// "총 금액: 1,640,000원"
```

### 2. 데이터 그룹핑

```javascript
let users = [
  { name: "홍길동", age: 30, city: "서울" },
  { name: "김철수", age: 25, city: "부산" },
  { name: "이영희", age: 35, city: "서울" },
  { name: "박민수", age: 28, city: "대전" }
];

let byCity = users.reduce((acc, user) => {
  if (!acc[user.city]) {
    acc[user.city] = [];
  }
  acc[user.city].push(user);
  return acc;
}, {});

console.log(byCity);
// {
//   서울: [{ name: "홍길동", ... }, { name: "이영희", ... }],
//   부산: [{ name: "김철수", ... }],
//   대전: [{ name: "박민수", ... }]
// }
```

### 3. 중복 제거

```javascript
let numbers = [1, 2, 3, 2, 4, 3, 5, 1];

// Set 사용 (ES6)
let unique1 = [...new Set(numbers)];
console.log(unique1);  // [1, 2, 3, 4, 5]

// filter 사용
let unique2 = numbers.filter((num, index) => numbers.indexOf(num) === index);
console.log(unique2);  // [1, 2, 3, 4, 5]
```

### 4. 배열 평탄화 (Flatten)

```javascript
let nested = [1, [2, 3], [4, [5, 6]], 7];

// flat() - ES2019
let flat1 = nested.flat(2);
console.log(flat1);  // [1, 2, 3, 4, 5, 6, 7]

// reduce 사용
function flatten(arr) {
  return arr.reduce((acc, val) => {
    return acc.concat(Array.isArray(val) ? flatten(val) : val);
  }, []);
}

let flat2 = flatten(nested);
console.log(flat2);  // [1, 2, 3, 4, 5, 6, 7]
```

## 참고 자료

- [MDN - Array](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array)
- [MDN - Array methods](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array#%EC%9D%B8%EC%8A%A4%ED%84%B4%EC%8A%A4_%EB%A9%94%EC%84%9C%EB%93%9C)
- [MDN - map()](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/map)
- [MDN - filter()](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/filter)
- [MDN - reduce()](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce)
- [JavaScript.info - Arrays](https://javascript.info/array)
- [JavaScript.info - Array methods](https://javascript.info/array-methods)

## 마치며

JavaScript 배열은 데이터를 효율적으로 관리하는 핵심 자료구조입니다:

1. **배열 생성**: 리터럴 `[]` 사용 (권장)
2. **인덱스**: 0부터 시작하는 위치 번호
3. **배열 메서드**: push, pop, slice, splice 등
4. **고차 함수**: map, filter, reduce로 함수형 프로그래밍
5. **배열 순회**: for, forEach, for...of

배열 메서드를 능숙하게 사용하고, map/filter/reduce 같은 고차 함수를 활용하여 선언적이고 읽기 쉬운 코드를 작성하세요.
