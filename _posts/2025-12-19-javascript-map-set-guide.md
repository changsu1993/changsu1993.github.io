---
title: "JavaScript Map과 Set 완벽 가이드 - ES6 컬렉션 자료구조 마스터하기"
description: "JavaScript Map, Set, WeakMap, WeakSet 완벽 가이드. ES6 컬렉션 자료구조의 기본 사용법부터 Object/Array와의 차이점, O(1) 성능 이점, LRU 캐시 구현, 중복 제거, 빈도 카운팅 등 실전 활용 패턴까지 코드 예제와 함께 상세히 설명합니다."
date: 2025-12-19 14:00:00 +0900
categories: [Frontend, JavaScript]
tags: [javascript, map, set, weakmap, weakset, es6, collection, data-structure, cache, deduplication, hash-table, iterable, performance, memory-management, lru-cache]
---

## 개요

JavaScript에서 데이터를 저장하고 관리할 때 주로 객체(Object)와 배열(Array)을 사용합니다. 하지만 이들만으로는 해결하기 어려운 상황이 있습니다. 키로 객체를 사용해야 하거나, 고유한 값들의 집합이 필요하거나, 메모리 효율적인 캐싱이 필요한 경우가 그렇습니다.

ES6에서 도입된 **Map**과 **Set**은 이러한 한계를 극복하기 위해 설계된 컬렉션 자료구조입니다. Map은 키-값 쌍을 저장하되 어떤 타입이든 키로 사용할 수 있고, Set은 중복 없는 고유한 값들의 집합을 관리합니다. 여기에 **WeakMap**과 **WeakSet**은 약한 참조를 통해 메모리 누수 없는 데이터 관리를 가능하게 합니다.

### 학습 목표

- Map의 기본 개념과 Object와의 차이점 이해
- Set의 기본 개념과 Array와의 차이점 이해
- WeakMap/WeakSet의 약한 참조 개념과 활용법 습득
- 실전 활용 사례: 캐싱, 중복 제거, 빈도 카운팅, DOM 메타데이터 관리
- 성능 특성과 적절한 사용 시나리오 파악

### 사전 지식

- JavaScript 기초 ([JavaScript 함수 완벽 가이드](/posts/javascript-function-guide/) 참고)
- 객체와 배열 ([JavaScript 객체 가이드](/posts/javascript-objects/), [배열 가이드](/posts/javascript-array-guide/) 참고)
- ES6+ 문법 (화살표 함수, 구조 분해 할당, for...of)

---

## Map 완벽 가이드

**Map**은 키-값 쌍을 저장하는 컬렉션으로, 키의 원래 삽입 순서를 기억합니다. Object와 달리 어떤 값이든(객체, 함수, 원시값 등) 키로 사용할 수 있습니다.

### Map 생성하기

```javascript
// 빈 Map 생성
const map = new Map();

// 초기값과 함께 생성 (이터러블 전달)
const map2 = new Map([
  ['name', '김철수'],
  ['age', 30],
  ['city', '서울']
]);

// 다른 Map으로부터 생성
const map3 = new Map(map2);

// Object.entries()로 객체를 Map으로 변환
const obj = { a: 1, b: 2, c: 3 };
const mapFromObj = new Map(Object.entries(obj));
```

### 기본 메서드 (CRUD)

```javascript
const users = new Map();

// CREATE: set(key, value) - 키-값 쌍 추가
users.set('user1', { name: '김철수', age: 30 });
users.set('user2', { name: '이영희', age: 25 });
users.set('user3', { name: '박민수', age: 35 });

// set()은 체이닝 가능
users
  .set('user4', { name: '정수진', age: 28 })
  .set('user5', { name: '최동훈', age: 32 });

// READ: get(key) - 값 조회
console.log(users.get('user1')); // { name: '김철수', age: 30 }
console.log(users.get('user99')); // undefined (존재하지 않는 키)

// READ: has(key) - 키 존재 여부 확인
console.log(users.has('user1')); // true
console.log(users.has('user99')); // false

// READ: size - 요소 개수
console.log(users.size); // 5

// UPDATE: set()으로 기존 키의 값 덮어쓰기
users.set('user1', { name: '김철수', age: 31 }); // 나이 업데이트

// DELETE: delete(key) - 특정 키-값 쌍 삭제
users.delete('user5'); // true 반환 (성공)
users.delete('user99'); // false 반환 (존재하지 않는 키)

// DELETE: clear() - 모든 요소 삭제
users.clear();
console.log(users.size); // 0
```

### Map 이터레이션

Map은 삽입 순서를 유지하며 여러 방법으로 순회할 수 있습니다. 이터레이터와 제너레이터에 대해 더 알고 싶다면 [JavaScript 제너레이터와 이터레이터 가이드](/posts/javascript-generator-iterator-guide/)를 참고하세요.

```javascript
const fruits = new Map([
  ['apple', 3],
  ['banana', 5],
  ['orange', 2]
]);

// 1. for...of로 순회 (기본: entries)
for (const [key, value] of fruits) {
  console.log(`${key}: ${value}개`);
}
// apple: 3개
// banana: 5개
// orange: 2개

// 2. keys() - 키만 순회
for (const key of fruits.keys()) {
  console.log(key); // apple, banana, orange
}

// 3. values() - 값만 순회
for (const value of fruits.values()) {
  console.log(value); // 3, 5, 2
}

// 4. entries() - [키, 값] 쌍으로 순회
for (const entry of fruits.entries()) {
  console.log(entry); // ['apple', 3], ['banana', 5], ['orange', 2]
}

// 5. forEach() 메서드
fruits.forEach((value, key, map) => {
  console.log(`${key} => ${value}`);
});

// Map을 배열로 변환
const keysArray = [...fruits.keys()];      // ['apple', 'banana', 'orange']
const valuesArray = [...fruits.values()];  // [3, 5, 2]
const entriesArray = [...fruits.entries()]; // [['apple', 3], ['banana', 5], ['orange', 2]]

// 또는 Array.from() 사용
const entriesArray2 = Array.from(fruits);
```

### 객체를 키로 사용하기

Map의 가장 큰 장점 중 하나는 객체를 키로 사용할 수 있다는 것입니다.

```javascript
const userPermissions = new Map();

const user1 = { id: 1, name: '김철수' };
const user2 = { id: 2, name: '이영희' };
const user3 = { id: 3, name: '박민수' };

// 객체를 키로 사용
userPermissions.set(user1, ['read', 'write', 'delete']);
userPermissions.set(user2, ['read', 'write']);
userPermissions.set(user3, ['read']);

// 객체로 값 조회
console.log(userPermissions.get(user1)); // ['read', 'write', 'delete']
console.log(userPermissions.get(user2)); // ['read', 'write']

// 주의: 같은 내용이라도 다른 객체는 다른 키로 취급
const sameContent = { id: 1, name: '김철수' };
console.log(userPermissions.get(sameContent)); // undefined (다른 참조)
console.log(user1 === sameContent); // false

// DOM 요소를 키로 사용하는 예제
const elementData = new Map();
const button = document.querySelector('#myButton');
if (button) {
  elementData.set(button, { clicks: 0, lastClicked: null });
}
```

### Map의 키 동등성 비교

Map은 키를 비교할 때 **SameValueZero** 알고리즘을 사용합니다.

```javascript
const map = new Map();

// NaN은 자기 자신과 같다고 판단 (일반 === 비교와 다름)
map.set(NaN, 'Not a Number');
console.log(map.get(NaN)); // 'Not a Number'
console.log(NaN === NaN);  // false (일반 비교)

// +0과 -0은 같은 키로 취급
map.set(0, 'zero');
map.set(-0, 'negative zero');
console.log(map.get(0));  // 'negative zero' (덮어씌워짐)
console.log(map.size);    // 1

// 문자열과 숫자는 다른 키
map.set('1', 'string one');
map.set(1, 'number one');
console.log(map.get('1')); // 'string one'
console.log(map.get(1));   // 'number one'
console.log(map.size);     // 3
```

---

## Object vs Map 비교

Object와 Map은 모두 키-값 쌍을 저장하지만 중요한 차이점이 있습니다.

### 기능 비교

| 특성 | Object | Map |
|------|--------|-----|
| 키 타입 | 문자열, Symbol만 | 모든 값 (객체, 함수, 원시값) |
| 키 순서 | ES2015+ 특정 규칙 | 삽입 순서 보장 |
| 크기 확인 | Object.keys(obj).length | map.size (O(1)) |
| 이터러블 | 직접 불가 | 바로 가능 |
| 성능 | 잦은 추가/삭제에 최적화되지 않음 | 잦은 추가/삭제에 최적화됨 |
| 프로토타입 | 기본 프로토타입 있음 | 없음 |
| JSON 직렬화 | JSON.stringify() 직접 가능 | 변환 필요 |

### 키 타입의 차이

```javascript
// Object: 키가 문자열로 변환됨
const obj = {};
const keyObj = { id: 1 };
obj[keyObj] = 'value';
console.log(Object.keys(obj)); // ['[object Object]']

// Map: 객체 키 그대로 유지
const map = new Map();
const keyObj1 = { id: 1 };
const keyObj2 = { id: 2 };
map.set(keyObj1, 'value1');
map.set(keyObj2, 'value2');
console.log(map.get(keyObj1)); // 'value1'
console.log(map.get(keyObj2)); // 'value2'
```

### 성능 비교

```javascript
// 크기 확인 성능
const largeObj = {};
const largeMap = new Map();

for (let i = 0; i < 1000000; i++) {
  largeObj[`key${i}`] = i;
  largeMap.set(`key${i}`, i);
}

// Object: O(n) - 모든 키를 열거해야 함
console.time('Object size');
const objSize = Object.keys(largeObj).length;
console.timeEnd('Object size'); // 약 50-100ms

// Map: O(1) - size 프로퍼티로 바로 접근
console.time('Map size');
const mapSize = largeMap.size;
console.timeEnd('Map size'); // 0.01ms 이하
```

### 삽입/삭제 성능 비교

```javascript
// 빈번한 추가/삭제 시나리오
function testObjectPerformance() {
  const obj = {};
  console.time('Object add/delete');
  for (let i = 0; i < 100000; i++) {
    obj[`key${i}`] = i;
  }
  for (let i = 0; i < 100000; i++) {
    delete obj[`key${i}`];
  }
  console.timeEnd('Object add/delete');
}

function testMapPerformance() {
  const map = new Map();
  console.time('Map add/delete');
  for (let i = 0; i < 100000; i++) {
    map.set(`key${i}`, i);
  }
  for (let i = 0; i < 100000; i++) {
    map.delete(`key${i}`);
  }
  console.timeEnd('Map add/delete');
}

testObjectPerformance(); // 약 30-50ms
testMapPerformance();    // 약 15-25ms (일반적으로 더 빠름)
```

### 언제 무엇을 사용할까?

**Object를 사용해야 할 때:**
- JSON과의 직렬화/역직렬화가 필요한 경우
- 고정된 키 집합으로 레코드를 표현할 때
- 객체 리터럴로 간단히 선언할 때
- 메서드를 포함한 객체가 필요할 때

```javascript
// Object가 적합한 경우
const user = {
  name: '김철수',
  age: 30,
  greet() {
    return `안녕하세요, ${this.name}입니다.`;
  }
};

// JSON 직렬화가 자연스러움
const json = JSON.stringify(user);
```

**Map을 사용해야 할 때:**
- 키가 문자열이 아닌 경우 (객체, 함수 등)
- 잦은 추가/삭제가 발생하는 경우
- 삽입 순서가 중요한 경우
- 키-값 쌍의 개수를 자주 확인해야 하는 경우
- 순수한 데이터 컬렉션이 필요한 경우

```javascript
// Map이 적합한 경우
const clickCounts = new Map();

document.querySelectorAll('button').forEach(button => {
  clickCounts.set(button, 0);

  button.addEventListener('click', () => {
    clickCounts.set(button, clickCounts.get(button) + 1);
    console.log(`클릭 수: ${clickCounts.get(button)}`);
  });
});
```

---

## Set 완벽 가이드

**Set**은 중복되지 않는 유일한 값들의 집합입니다. 배열과 달리 같은 값을 두 번 저장할 수 없으며, 값의 존재 여부를 빠르게 확인할 수 있습니다.

### Set 생성하기

```javascript
// 빈 Set 생성
const set = new Set();

// 이터러블로부터 생성 (중복 자동 제거)
const set2 = new Set([1, 2, 3, 2, 1]);
console.log(set2); // Set(3) {1, 2, 3}

// 문자열로부터 생성 (각 문자가 요소)
const charSet = new Set('hello');
console.log(charSet); // Set(4) {'h', 'e', 'l', 'o'}

// 다른 Set으로부터 생성
const set3 = new Set(set2);
```

### 기본 메서드

```javascript
const numbers = new Set();

// add(value) - 값 추가
numbers.add(1);
numbers.add(2);
numbers.add(3);
numbers.add(2); // 중복 무시됨
console.log(numbers); // Set(3) {1, 2, 3}

// add()는 체이닝 가능
numbers.add(4).add(5).add(6);

// has(value) - 값 존재 여부 확인 (O(1))
console.log(numbers.has(3)); // true
console.log(numbers.has(99)); // false

// size - 요소 개수
console.log(numbers.size); // 6

// delete(value) - 특정 값 삭제
numbers.delete(6); // true 반환
numbers.delete(99); // false 반환 (존재하지 않음)

// clear() - 모든 요소 삭제
numbers.clear();
console.log(numbers.size); // 0
```

### Set 이터레이션

```javascript
const colors = new Set(['red', 'green', 'blue']);

// 1. for...of로 순회
for (const color of colors) {
  console.log(color);
}
// red, green, blue

// 2. forEach() 메서드
// (value, valueAgain, set) - Map과 일관성을 위해 value가 두 번 전달됨
colors.forEach((value, value2, set) => {
  console.log(value); // value === value2
});

// 3. keys(), values() - 동일한 결과 (Set은 키가 없으므로)
for (const value of colors.keys()) {
  console.log(value);
}

for (const value of colors.values()) {
  console.log(value);
}

// 4. entries() - [값, 값] 쌍 반환
for (const entry of colors.entries()) {
  console.log(entry); // ['red', 'red'], ['green', 'green'], ['blue', 'blue']
}

// Set을 배열로 변환
const colorArray = [...colors]; // ['red', 'green', 'blue']
const colorArray2 = Array.from(colors);
```

### 집합 연산

ES6의 Set에는 집합 연산 메서드가 없었지만, ES2024(ES15)부터 공식 메서드가 추가되었습니다.

```javascript
const setA = new Set([1, 2, 3, 4, 5]);
const setB = new Set([4, 5, 6, 7, 8]);

// ES2024+ 공식 메서드 (최신 브라우저/Node.js 22+)
// 합집합 (Union)
const union = setA.union(setB);
console.log([...union]); // [1, 2, 3, 4, 5, 6, 7, 8]

// 교집합 (Intersection)
const intersection = setA.intersection(setB);
console.log([...intersection]); // [4, 5]

// 차집합 (Difference)
const difference = setA.difference(setB);
console.log([...difference]); // [1, 2, 3]

// 대칭 차집합 (Symmetric Difference)
const symmetricDiff = setA.symmetricDifference(setB);
console.log([...symmetricDiff]); // [1, 2, 3, 6, 7, 8]

// 부분집합 여부
const setC = new Set([1, 2]);
console.log(setC.isSubsetOf(setA)); // true

// 상위집합 여부
console.log(setA.isSupersetOf(setC)); // true

// 서로소 집합 여부
const setD = new Set([10, 11]);
console.log(setA.isDisjointFrom(setD)); // true
```

**ES2024 이전 환경을 위한 폴리필:**

```javascript
// 합집합
function union(setA, setB) {
  return new Set([...setA, ...setB]);
}

// 교집합
function intersection(setA, setB) {
  return new Set([...setA].filter(x => setB.has(x)));
}

// 차집합 (A - B)
function difference(setA, setB) {
  return new Set([...setA].filter(x => !setB.has(x)));
}

// 대칭 차집합
function symmetricDifference(setA, setB) {
  const diff = new Set(setA);
  for (const elem of setB) {
    if (diff.has(elem)) {
      diff.delete(elem);
    } else {
      diff.add(elem);
    }
  }
  return diff;
}

// 사용 예
const a = new Set([1, 2, 3]);
const b = new Set([2, 3, 4]);
console.log([...union(a, b)]);        // [1, 2, 3, 4]
console.log([...intersection(a, b)]); // [2, 3]
console.log([...difference(a, b)]);   // [1]
```

### Set의 값 동등성 비교

Set도 Map과 마찬가지로 **SameValueZero** 알고리즘을 사용합니다.

```javascript
const set = new Set();

// NaN
set.add(NaN);
set.add(NaN); // 중복이므로 추가되지 않음
console.log(set.size); // 1
console.log(set.has(NaN)); // true

// +0과 -0
set.add(0);
set.add(-0); // 같은 값으로 취급
console.log(set.size); // 2 (NaN과 0)

// 객체는 참조로 비교
const obj1 = { id: 1 };
const obj2 = { id: 1 };
set.add(obj1);
set.add(obj2); // 다른 참조이므로 추가됨
console.log(set.size); // 4
```

---

## Array vs Set 비교

### 기능 비교

| 특성 | Array | Set |
|------|-------|-----|
| 중복 허용 | O | X |
| 인덱스 접근 | O (arr[0]) | X |
| 순서 보장 | O | O (삽입 순서) |
| 값 존재 확인 | O(n) includes() | O(1) has() |
| 값 삭제 | O(n) splice() | O(1) delete() |
| 크기 확인 | length | size |

배열 메서드에 대해 더 자세히 알고 싶다면 [JavaScript 배열 메서드 완벽 가이드](/posts/javascript-array-methods/)를 참고하세요.

### 성능 비교

```javascript
// 값 존재 확인 성능 테스트
const size = 100000;
const arr = [];
const set = new Set();

for (let i = 0; i < size; i++) {
  arr.push(i);
  set.add(i);
}

const searchValue = size - 1; // 최악의 경우 (배열 끝에 있는 값)

console.time('Array includes');
for (let i = 0; i < 10000; i++) {
  arr.includes(searchValue);
}
console.timeEnd('Array includes'); // 약 500-1000ms

console.time('Set has');
for (let i = 0; i < 10000; i++) {
  set.has(searchValue);
}
console.timeEnd('Set has'); // 약 1-2ms
```

### 언제 무엇을 사용할까?

**Array를 사용해야 할 때:**
- 순서와 인덱스가 중요한 경우
- 중복 값을 허용해야 하는 경우
- map, filter, reduce 등 배열 메서드가 필요한 경우
- 정렬이 필요한 경우

```javascript
// Array가 적합한 경우
const scores = [85, 90, 78, 92, 88];
const average = scores.reduce((sum, s) => sum + s, 0) / scores.length;
const sorted = scores.sort((a, b) => b - a);
const topThree = sorted.slice(0, 3);
```

**Set을 사용해야 할 때:**
- 고유한 값들의 집합이 필요한 경우
- 값의 존재 여부를 빠르게 확인해야 하는 경우
- 중복 제거가 필요한 경우
- 집합 연산(합집합, 교집합 등)이 필요한 경우

```javascript
// Set이 적합한 경우
const visitedPages = new Set();

function trackPageVisit(page) {
  if (visitedPages.has(page)) {
    console.log('이미 방문한 페이지입니다.');
    return;
  }
  visitedPages.add(page);
  console.log(`새 페이지 방문: ${page}`);
}
```

---

## WeakMap 완벽 가이드

**WeakMap**은 Map과 유사하지만, 키가 반드시 객체여야 하며 **약한 참조(weak reference)**로 저장됩니다. 이는 WeakMap의 키가 다른 곳에서 참조되지 않으면 가비지 컬렉션의 대상이 된다는 의미입니다. 가비지 컬렉션에 대해 더 자세히 알고 싶다면 [JavaScript 메모리 관리와 가비지 컬렉션 가이드](/posts/javascript-memory-management-garbage-collection-guide/)를 참고하세요.

### WeakMap의 특징

```javascript
const weakMap = new WeakMap();

let user = { name: '김철수' };

// 객체를 키로 사용 (원시값은 불가)
weakMap.set(user, { lastLogin: new Date() });
console.log(weakMap.get(user)); // { lastLogin: ... }

// 원시값은 키로 사용 불가
// weakMap.set('key', 'value'); // TypeError!
// weakMap.set(123, 'value');   // TypeError!

// user 변수가 null이 되면, WeakMap의 엔트리도 가비지 컬렉션 대상
user = null;
// 이제 { name: '김철수' } 객체는 WeakMap에서 자동으로 제거될 수 있음
```

### WeakMap vs Map 비교

| 특성 | Map | WeakMap |
|------|-----|---------|
| 키 타입 | 모든 값 | 객체만 |
| 참조 방식 | 강한 참조 | 약한 참조 |
| 이터레이션 | 가능 | 불가능 |
| size 프로퍼티 | 있음 | 없음 |
| clear() 메서드 | 있음 | 없음 |
| 가비지 컬렉션 | 키가 참조되는 한 유지 | 키가 다른 곳에서 참조되지 않으면 수집 |

### WeakMap의 사용 가능한 메서드

```javascript
const weakMap = new WeakMap();
const obj = { id: 1 };

// 사용 가능한 메서드
weakMap.set(obj, 'value');     // 값 설정
weakMap.get(obj);              // 값 조회
weakMap.has(obj);              // 존재 여부
weakMap.delete(obj);           // 삭제

// 사용 불가능 (이터레이션 관련)
// weakMap.keys()    - 불가능
// weakMap.values()  - 불가능
// weakMap.entries() - 불가능
// weakMap.forEach() - 불가능
// weakMap.size      - 불가능
// weakMap.clear()   - 불가능
```

### WeakMap 활용 사례 1: Private 데이터

WeakMap은 클래스의 private 데이터를 저장하는 패턴에 활용됩니다.

```javascript
// WeakMap을 사용한 private 데이터 패턴
const privateData = new WeakMap();

class Person {
  constructor(name, age) {
    // private 데이터를 WeakMap에 저장
    privateData.set(this, {
      name,
      age,
      ssn: this.generateSSN()
    });
  }

  generateSSN() {
    return Math.random().toString(36).substring(2, 15);
  }

  getName() {
    return privateData.get(this).name;
  }

  getAge() {
    return privateData.get(this).age;
  }

  // SSN은 외부에서 직접 접근 불가
  getLastFourSSN() {
    const ssn = privateData.get(this).ssn;
    return '***-**-' + ssn.slice(-4);
  }

  celebrateBirthday() {
    const data = privateData.get(this);
    data.age += 1;
  }
}

const person = new Person('김철수', 30);
console.log(person.getName()); // '김철수'
console.log(person.getAge());  // 30
console.log(person.getLastFourSSN()); // '***-**-xxxx'

// private 데이터에 직접 접근 불가
console.log(person.name); // undefined
console.log(person.ssn);  // undefined

// person이 가비지 컬렉션되면 privateData의 해당 엔트리도 자동 정리
```

**참고**: ES2022부터는 클래스의 `#` prefix로 진정한 private 필드를 사용할 수 있습니다.

```javascript
// ES2022+ private 필드
class Person {
  #name;
  #age;
  #ssn;

  constructor(name, age) {
    this.#name = name;
    this.#age = age;
    this.#ssn = Math.random().toString(36).substring(2, 15);
  }

  getName() {
    return this.#name;
  }
}
```

### WeakMap 활용 사례 2: 객체 메타데이터 캐싱

```javascript
// DOM 요소에 메타데이터 연결 (요소가 제거되면 자동 정리)
const elementMetadata = new WeakMap();

function attachMetadata(element, data) {
  const existing = elementMetadata.get(element) || {};
  elementMetadata.set(element, { ...existing, ...data });
}

function getMetadata(element) {
  return elementMetadata.get(element);
}

// 사용 예
const button = document.createElement('button');
attachMetadata(button, { clickCount: 0, createdAt: Date.now() });

button.addEventListener('click', () => {
  const meta = getMetadata(button);
  meta.clickCount++;
  attachMetadata(button, meta);
});

// button이 DOM에서 제거되고 참조가 없어지면
// elementMetadata의 해당 엔트리도 자동으로 가비지 컬렉션됨
```

### WeakMap 활용 사례 3: 함수 결과 캐싱 (메모이제이션)

```javascript
// 객체를 인자로 받는 함수의 결과 캐싱
const cache = new WeakMap();

function expensiveOperation(obj) {
  // 캐시 확인
  if (cache.has(obj)) {
    console.log('캐시에서 반환');
    return cache.get(obj);
  }

  // 비용이 큰 연산 수행
  console.log('연산 수행');
  const result = Object.keys(obj).reduce((sum, key) => {
    return sum + JSON.stringify(obj[key]).length;
  }, 0);

  // 결과 캐싱
  cache.set(obj, result);
  return result;
}

const data = { name: '김철수', details: { age: 30, city: '서울' } };

expensiveOperation(data); // "연산 수행"
expensiveOperation(data); // "캐시에서 반환"

// data가 더 이상 사용되지 않으면 캐시도 자동 정리
```

---

## WeakSet 완벽 가이드

**WeakSet**은 Set과 유사하지만, 객체만 저장할 수 있으며 약한 참조로 저장됩니다.

### WeakSet의 특징

```javascript
const weakSet = new WeakSet();

let obj1 = { id: 1 };
let obj2 = { id: 2 };

// 객체만 추가 가능
weakSet.add(obj1);
weakSet.add(obj2);

console.log(weakSet.has(obj1)); // true
console.log(weakSet.has(obj2)); // true

// 원시값은 추가 불가
// weakSet.add(1);       // TypeError!
// weakSet.add('hello'); // TypeError!

// obj1에 대한 참조가 없어지면 WeakSet에서 자동 제거
obj1 = null;
// { id: 1 } 객체는 가비지 컬렉션 대상
```

### WeakSet의 사용 가능한 메서드

```javascript
const weakSet = new WeakSet();
const obj = { id: 1 };

// 사용 가능한 메서드
weakSet.add(obj);    // 추가
weakSet.has(obj);    // 존재 여부
weakSet.delete(obj); // 삭제

// 사용 불가능
// weakSet.size       - 불가능
// weakSet.keys()     - 불가능
// weakSet.values()   - 불가능
// weakSet.entries()  - 불가능
// weakSet.forEach()  - 불가능
// weakSet.clear()    - 불가능
```

### WeakSet 활용 사례 1: 객체 방문/처리 추적

```javascript
// 순환 참조가 있는 객체를 처리할 때 방문 추적
const visited = new WeakSet();

function deepClone(obj) {
  // 원시값은 그대로 반환
  if (obj === null || typeof obj !== 'object') {
    return obj;
  }

  // 순환 참조 감지
  if (visited.has(obj)) {
    throw new Error('순환 참조가 감지되었습니다!');
  }

  visited.add(obj);

  // 배열 처리
  if (Array.isArray(obj)) {
    return obj.map(item => deepClone(item));
  }

  // 객체 처리
  const cloned = {};
  for (const key in obj) {
    if (obj.hasOwnProperty(key)) {
      cloned[key] = deepClone(obj[key]);
    }
  }

  return cloned;
}

// 순환 참조가 있는 객체
const circular = { name: '순환' };
circular.self = circular;

try {
  deepClone(circular);
} catch (e) {
  console.log(e.message); // '순환 참조가 감지되었습니다!'
}
```

### WeakSet 활용 사례 2: 객체 검증/마킹

```javascript
// 특정 생성자로 만들어진 객체인지 추적
const validatedUsers = new WeakSet();

class User {
  constructor(name, email) {
    this.name = name;
    this.email = email;

    // 유효성 검사 통과 후 마킹
    if (this.validate()) {
      validatedUsers.add(this);
    }
  }

  validate() {
    return this.name.length > 0 && this.email.includes('@');
  }

  static isValidated(user) {
    return validatedUsers.has(user);
  }
}

const user1 = new User('김철수', 'kim@example.com');
const user2 = new User('', 'invalid');
const fakeUser = { name: '가짜', email: 'fake@test.com' };

console.log(User.isValidated(user1));   // true
console.log(User.isValidated(user2));   // false
console.log(User.isValidated(fakeUser)); // false

// user1이 더 이상 참조되지 않으면 validatedUsers에서도 자동 제거
```

### WeakSet 활용 사례 3: DOM 요소 상태 추적

```javascript
// 이미 처리된 DOM 요소 추적
const processedElements = new WeakSet();

function processElement(element) {
  if (processedElements.has(element)) {
    console.log('이미 처리된 요소입니다.');
    return;
  }

  // 요소 처리 로직
  element.classList.add('processed');
  element.dataset.timestamp = Date.now();

  processedElements.add(element);
  console.log('요소 처리 완료');
}

// 사용 예
document.querySelectorAll('.item').forEach(element => {
  processElement(element);
});

// 같은 요소를 다시 처리하려고 해도 무시됨
document.querySelectorAll('.item').forEach(element => {
  processElement(element); // "이미 처리된 요소입니다."
});

// DOM에서 요소가 제거되면 processedElements에서도 자동 정리
```

---

## 실전 활용 예제

### 1. 효율적인 캐시 시스템

```javascript
// LRU(Least Recently Used) 캐시 구현
class LRUCache {
  constructor(maxSize = 100) {
    this.maxSize = maxSize;
    this.cache = new Map(); // Map은 삽입 순서를 유지
  }

  get(key) {
    if (!this.cache.has(key)) {
      return undefined;
    }

    // 접근한 항목을 가장 최근으로 이동
    const value = this.cache.get(key);
    this.cache.delete(key);
    this.cache.set(key, value);
    return value;
  }

  set(key, value) {
    // 이미 존재하면 삭제 후 다시 추가 (순서 갱신)
    if (this.cache.has(key)) {
      this.cache.delete(key);
    }

    this.cache.set(key, value);

    // 최대 크기 초과 시 가장 오래된 항목 삭제
    if (this.cache.size > this.maxSize) {
      const oldestKey = this.cache.keys().next().value;
      this.cache.delete(oldestKey);
    }
  }

  has(key) {
    return this.cache.has(key);
  }

  get size() {
    return this.cache.size;
  }

  clear() {
    this.cache.clear();
  }
}

// 사용 예
const cache = new LRUCache(3);
cache.set('a', 1);
cache.set('b', 2);
cache.set('c', 3);
console.log(cache.get('a')); // 1 (a가 가장 최근으로 이동)
cache.set('d', 4); // b가 제거됨 (가장 오래됨)
console.log(cache.has('b')); // false
```

### 2. 배열 중복 제거

```javascript
// 원시값 배열 중복 제거
const numbers = [1, 2, 3, 2, 1, 4, 5, 4, 3];
const uniqueNumbers = [...new Set(numbers)];
console.log(uniqueNumbers); // [1, 2, 3, 4, 5]

// 문자열 배열 중복 제거
const names = ['김철수', '이영희', '김철수', '박민수', '이영희'];
const uniqueNames = [...new Set(names)];
console.log(uniqueNames); // ['김철수', '이영희', '박민수']

// 객체 배열에서 특정 속성 기준 중복 제거
const users = [
  { id: 1, name: '김철수' },
  { id: 2, name: '이영희' },
  { id: 1, name: '김철수' },
  { id: 3, name: '박민수' }
];

function uniqueByProperty(array, property) {
  const seen = new Set();
  return array.filter(item => {
    const value = item[property];
    if (seen.has(value)) {
      return false;
    }
    seen.add(value);
    return true;
  });
}

const uniqueUsers = uniqueByProperty(users, 'id');
console.log(uniqueUsers);
// [{ id: 1, name: '김철수' }, { id: 2, name: '이영희' }, { id: 3, name: '박민수' }]
```

### 3. 빈도 카운팅

```javascript
// 단어 빈도 카운팅
function countWordFrequency(text) {
  const words = text.toLowerCase().match(/\b\w+\b/g) || [];
  const frequency = new Map();

  for (const word of words) {
    frequency.set(word, (frequency.get(word) || 0) + 1);
  }

  return frequency;
}

const text = 'JavaScript is great. JavaScript is fun. JavaScript is powerful.';
const freq = countWordFrequency(text);
console.log(freq);
// Map(4) { 'javascript' => 3, 'is' => 3, 'great' => 1, 'fun' => 1, 'powerful' => 1 }

// 가장 빈도 높은 단어 찾기
const sortedByFreq = [...freq.entries()].sort((a, b) => b[1] - a[1]);
console.log(sortedByFreq[0]); // ['javascript', 3]

// 문자 빈도 카운팅
function countCharFrequency(str) {
  const frequency = new Map();

  for (const char of str) {
    if (char !== ' ') {
      frequency.set(char, (frequency.get(char) || 0) + 1);
    }
  }

  return frequency;
}

console.log(countCharFrequency('hello'));
// Map(4) { 'h' => 1, 'e' => 1, 'l' => 2, 'o' => 1 }
```

### 4. 두 배열의 교집합, 합집합, 차집합

```javascript
function arrayOperations(arr1, arr2) {
  const set1 = new Set(arr1);
  const set2 = new Set(arr2);

  return {
    // 합집합
    union: [...new Set([...arr1, ...arr2])],

    // 교집합
    intersection: arr1.filter(x => set2.has(x)),

    // 차집합 (arr1 - arr2)
    difference: arr1.filter(x => !set2.has(x)),

    // 대칭 차집합
    symmetricDifference: [
      ...arr1.filter(x => !set2.has(x)),
      ...arr2.filter(x => !set1.has(x))
    ]
  };
}

const a = [1, 2, 3, 4, 5];
const b = [4, 5, 6, 7, 8];

const result = arrayOperations(a, b);
console.log(result.union);              // [1, 2, 3, 4, 5, 6, 7, 8]
console.log(result.intersection);       // [4, 5]
console.log(result.difference);         // [1, 2, 3]
console.log(result.symmetricDifference); // [1, 2, 3, 6, 7, 8]
```

### 5. 양방향 매핑

```javascript
// 양방향 Map (Bidirectional Map)
class BidirectionalMap {
  constructor(entries = []) {
    this.forward = new Map();
    this.reverse = new Map();

    for (const [key, value] of entries) {
      this.set(key, value);
    }
  }

  set(key, value) {
    // 기존 매핑 제거
    if (this.forward.has(key)) {
      this.reverse.delete(this.forward.get(key));
    }
    if (this.reverse.has(value)) {
      this.forward.delete(this.reverse.get(value));
    }

    this.forward.set(key, value);
    this.reverse.set(value, key);
    return this;
  }

  get(key) {
    return this.forward.get(key);
  }

  getKey(value) {
    return this.reverse.get(value);
  }

  has(key) {
    return this.forward.has(key);
  }

  hasValue(value) {
    return this.reverse.has(value);
  }

  delete(key) {
    if (this.forward.has(key)) {
      this.reverse.delete(this.forward.get(key));
      this.forward.delete(key);
      return true;
    }
    return false;
  }

  get size() {
    return this.forward.size;
  }
}

// 사용 예: 국가 코드 매핑
const countryCodes = new BidirectionalMap([
  ['KR', '대한민국'],
  ['US', '미국'],
  ['JP', '일본'],
  ['CN', '중국']
]);

console.log(countryCodes.get('KR'));       // '대한민국'
console.log(countryCodes.getKey('미국'));   // 'US'
console.log(countryCodes.hasValue('일본')); // true
```

### 6. 그래프 구현

```javascript
// 인접 리스트 방식의 그래프
class Graph {
  constructor() {
    this.adjacencyList = new Map();
  }

  addVertex(vertex) {
    if (!this.adjacencyList.has(vertex)) {
      this.adjacencyList.set(vertex, new Set());
    }
    return this;
  }

  addEdge(vertex1, vertex2) {
    this.addVertex(vertex1);
    this.addVertex(vertex2);
    this.adjacencyList.get(vertex1).add(vertex2);
    this.adjacencyList.get(vertex2).add(vertex1); // 무방향 그래프
    return this;
  }

  removeEdge(vertex1, vertex2) {
    if (this.adjacencyList.has(vertex1)) {
      this.adjacencyList.get(vertex1).delete(vertex2);
    }
    if (this.adjacencyList.has(vertex2)) {
      this.adjacencyList.get(vertex2).delete(vertex1);
    }
    return this;
  }

  removeVertex(vertex) {
    if (this.adjacencyList.has(vertex)) {
      for (const neighbor of this.adjacencyList.get(vertex)) {
        this.adjacencyList.get(neighbor).delete(vertex);
      }
      this.adjacencyList.delete(vertex);
    }
    return this;
  }

  getNeighbors(vertex) {
    return this.adjacencyList.get(vertex) || new Set();
  }

  hasEdge(vertex1, vertex2) {
    return this.adjacencyList.has(vertex1) &&
           this.adjacencyList.get(vertex1).has(vertex2);
  }

  // BFS 탐색
  bfs(startVertex) {
    const visited = new Set();
    const result = [];
    const queue = [startVertex];

    visited.add(startVertex);

    while (queue.length > 0) {
      const vertex = queue.shift();
      result.push(vertex);

      for (const neighbor of this.getNeighbors(vertex)) {
        if (!visited.has(neighbor)) {
          visited.add(neighbor);
          queue.push(neighbor);
        }
      }
    }

    return result;
  }

  // DFS 탐색
  dfs(startVertex) {
    const visited = new Set();
    const result = [];

    const traverse = (vertex) => {
      visited.add(vertex);
      result.push(vertex);

      for (const neighbor of this.getNeighbors(vertex)) {
        if (!visited.has(neighbor)) {
          traverse(neighbor);
        }
      }
    };

    traverse(startVertex);
    return result;
  }
}

// 사용 예
const graph = new Graph();
graph.addEdge('A', 'B');
graph.addEdge('A', 'C');
graph.addEdge('B', 'D');
graph.addEdge('C', 'D');
graph.addEdge('D', 'E');

console.log(graph.bfs('A')); // ['A', 'B', 'C', 'D', 'E']
console.log(graph.dfs('A')); // ['A', 'B', 'D', 'C', 'E']
```

---

## 성능 분석 (Big O)

### Map 성능

| 연산 | 시간 복잡도 |
|------|-------------|
| set(key, value) | O(1) 평균 |
| get(key) | O(1) 평균 |
| has(key) | O(1) 평균 |
| delete(key) | O(1) 평균 |
| size | O(1) |
| clear() | O(n) |
| 이터레이션 | O(n) |

### Set 성능

| 연산 | 시간 복잡도 |
|------|-------------|
| add(value) | O(1) 평균 |
| has(value) | O(1) 평균 |
| delete(value) | O(1) 평균 |
| size | O(1) |
| clear() | O(n) |
| 이터레이션 | O(n) |

### 비교: Object vs Map, Array vs Set

| 시나리오 | Object | Map | Array | Set |
|----------|--------|-----|-------|-----|
| 키/값 조회 | O(1) | O(1) | O(n) | O(1) |
| 삽입 | O(1) | O(1) | O(1)/O(n)* | O(1) |
| 삭제 | O(1) | O(1) | O(n) | O(1) |
| 크기 확인 | O(n) | O(1) | O(1) | O(1) |
| 존재 확인 | O(1) | O(1) | O(n) | O(1) |

*배열 끝 삽입은 O(1), 중간 삽입은 O(n)

---

## Map/Set과 JSON

Map과 Set은 JSON.stringify()로 직접 직렬화할 수 없습니다.

```javascript
const map = new Map([['a', 1], ['b', 2]]);
console.log(JSON.stringify(map)); // '{}'

const set = new Set([1, 2, 3]);
console.log(JSON.stringify(set)); // '{}'
```

### 직렬화/역직렬화 유틸리티

```javascript
// Map을 JSON으로 변환
function mapToJSON(map) {
  return JSON.stringify([...map.entries()]);
}

// JSON을 Map으로 변환
function jsonToMap(jsonStr) {
  return new Map(JSON.parse(jsonStr));
}

// Set을 JSON으로 변환
function setToJSON(set) {
  return JSON.stringify([...set]);
}

// JSON을 Set으로 변환
function jsonToSet(jsonStr) {
  return new Set(JSON.parse(jsonStr));
}

// 사용 예
const map = new Map([['name', '김철수'], ['age', 30]]);
const mapJson = mapToJSON(map);
console.log(mapJson); // '[["name","김철수"],["age",30]]'

const restoredMap = jsonToMap(mapJson);
console.log(restoredMap.get('name')); // '김철수'

const set = new Set([1, 2, 3]);
const setJson = setToJSON(set);
console.log(setJson); // '[1,2,3]'

const restoredSet = jsonToSet(setJson);
console.log(restoredSet.has(2)); // true
```

### 커스텀 replacer와 reviver 사용

```javascript
// JSON.stringify에 커스텀 replacer 사용
function replacer(key, value) {
  if (value instanceof Map) {
    return {
      dataType: 'Map',
      value: [...value.entries()]
    };
  }
  if (value instanceof Set) {
    return {
      dataType: 'Set',
      value: [...value]
    };
  }
  return value;
}

// JSON.parse에 커스텀 reviver 사용
function reviver(key, value) {
  if (typeof value === 'object' && value !== null) {
    if (value.dataType === 'Map') {
      return new Map(value.value);
    }
    if (value.dataType === 'Set') {
      return new Set(value.value);
    }
  }
  return value;
}

// 사용 예
const data = {
  users: new Map([['user1', { name: '김철수' }]]),
  tags: new Set(['js', 'map', 'set'])
};

const json = JSON.stringify(data, replacer, 2);
console.log(json);
/*
{
  "users": {
    "dataType": "Map",
    "value": [["user1", {"name": "김철수"}]]
  },
  "tags": {
    "dataType": "Set",
    "value": ["js", "map", "set"]
  }
}
*/

const restored = JSON.parse(json, reviver);
console.log(restored.users.get('user1')); // { name: '김철수' }
console.log(restored.tags.has('js'));      // true
```

---

## 브라우저 호환성

Map, Set, WeakMap, WeakSet은 ES6(ES2015)에서 도입되었으며, 모든 최신 브라우저에서 지원됩니다.

| 기능 | Chrome | Firefox | Safari | Edge | Node.js |
|------|--------|---------|--------|------|---------|
| Map | 38+ | 13+ | 8+ | 12+ | 0.12+ |
| Set | 38+ | 13+ | 8+ | 12+ | 0.12+ |
| WeakMap | 36+ | 6+ | 8+ | 12+ | 0.12+ |
| WeakSet | 36+ | 34+ | 9+ | 12+ | 0.12+ |
| Set 집합 연산 (ES2024) | 122+ | 127+ | 17+ | 122+ | 22+ |

ES2024의 Set 집합 연산 메서드(union, intersection 등)는 아직 일부 환경에서 지원되지 않을 수 있으므로, 이전 섹션에서 제공한 폴리필을 사용하거나 core-js 같은 폴리필 라이브러리를 사용하세요.

---

## 베스트 프랙티스

### 1. 적절한 자료구조 선택

```javascript
// 나쁜 예: 모든 곳에 객체/배열 사용
const userPermissions = {}; // 객체를 키로 써야 하는데 문자열로 제한됨
const uniqueItems = []; // 중복 체크를 위해 매번 includes() 호출

// 좋은 예: 상황에 맞는 자료구조 사용
const userPermissions = new Map(); // 객체를 키로 사용 가능
const uniqueItems = new Set(); // O(1)로 중복 확인
```

### 2. 메모리 누수 방지

```javascript
// 나쁜 예: 객체 참조가 계속 유지됨
const cache = new Map();
function processUser(user) {
  cache.set(user, expensiveCalculation(user));
  // user 객체가 필요 없어져도 cache가 참조를 유지
}

// 좋은 예: WeakMap으로 자동 정리
const cache = new WeakMap();
function processUser(user) {
  cache.set(user, expensiveCalculation(user));
  // user 객체가 필요 없어지면 cache의 엔트리도 자동 정리
}
```

### 3. 불필요한 변환 피하기

```javascript
// 나쁜 예: 불필요한 배열 변환
const set = new Set([1, 2, 3, 4, 5]);
const hasThree = [...set].includes(3); // O(n) + O(n)

// 좋은 예: Set의 메서드 직접 사용
const hasThree = set.has(3); // O(1)
```

### 4. 초기화 시 데이터 전달

```javascript
// 나쁜 예: 개별 추가
const map = new Map();
map.set('a', 1);
map.set('b', 2);
map.set('c', 3);

// 좋은 예: 생성 시 초기 데이터 전달
const map = new Map([
  ['a', 1],
  ['b', 2],
  ['c', 3]
]);

// Object.entries()로 객체를 Map으로 변환
const obj = { a: 1, b: 2, c: 3 };
const mapFromObj = new Map(Object.entries(obj));
```

### 5. 타입 안전성 고려 (TypeScript)

```typescript
// TypeScript에서 타입 지정
const userScores = new Map<string, number>();
userScores.set('kim', 95);
userScores.set('lee', 87);
// userScores.set(123, 'high'); // 타입 에러

const tags = new Set<string>();
tags.add('javascript');
tags.add('typescript');
// tags.add(123); // 타입 에러

// WeakMap 타입 지정
interface UserData {
  name: string;
  age: number;
}

const userData = new WeakMap<object, UserData>();
```

---

## 마무리

Map과 Set은 ES6에서 도입된 강력한 컬렉션 자료구조입니다.

**핵심 요약:**

- **Map**: 어떤 타입이든 키로 사용 가능, 삽입 순서 보장, 빈번한 추가/삭제에 최적화
- **Set**: 중복 없는 고유 값 집합, O(1) 존재 확인, 집합 연산 지원
- **WeakMap**: 객체만 키로 사용, 약한 참조로 메모리 누수 방지, private 데이터 패턴에 유용
- **WeakSet**: 객체만 저장, 약한 참조, 객체 추적/마킹에 유용

**언제 사용할까:**

| 상황 | 추천 |
|------|------|
| 객체를 키로 사용해야 할 때 | Map |
| 잦은 추가/삭제가 있을 때 | Map |
| 중복 없는 값 집합이 필요할 때 | Set |
| 값 존재를 빠르게 확인할 때 | Set |
| 메모리 누수 없는 캐싱 | WeakMap |
| private 데이터 저장 | WeakMap |
| 객체 방문/처리 추적 | WeakSet |

적절한 자료구조 선택은 코드의 가독성, 성능, 메모리 효율성을 모두 향상시킵니다. Object와 Array만 사용하던 습관에서 벗어나 상황에 맞는 컬렉션을 활용해보세요.

---

## 참고 자료

- [MDN - Map](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Map)
- [MDN - Set](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Set)
- [MDN - WeakMap](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/WeakMap)
- [MDN - WeakSet](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/WeakSet)
- [ECMAScript Specification - Keyed Collections](https://tc39.es/ecma262/#sec-keyed-collections)
- [JavaScript.info - Map and Set](https://javascript.info/map-set)
- [TC39 - Set Methods Proposal (ES2024)](https://github.com/tc39/proposal-set-methods)
