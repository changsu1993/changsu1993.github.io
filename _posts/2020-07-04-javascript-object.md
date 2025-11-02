---
title: JavaScript Object 완벽 가이드 - 객체의 모든 것
description: JavaScript 객체의 기초부터 실무 활용까지 완벽 정리. 객체 생성, 접근, 수정, Object 메서드, 구조 분해 할당, Optional Chaining 등 실전 예제와 함께 알아봅니다.
author: cotes
date: 2020-07-04 18:56:58 +0900
categories: [JavaScript, Basics]
tags: [javascript, object, destructuring, optional-chaining, object-methods]
---

## 객체(Object)란?

**객체(Object)**는 JavaScript에서 가장 중요한 데이터 타입입니다. 객체는 **키(key)와 값(value)의 쌍**으로 이루어진 프로퍼티(Property)들의 집합입니다.

```javascript
let plan1 = {
  name: "Basic",        // 프로퍼티: name (키) : "Basic" (값)
  price: 3.99,          // 프로퍼티: price : 3.99
  space: 100,           // 프로퍼티: space : 100
  transfer: 1000,       // 프로퍼티: transfer : 1000
  pages: 10             // 프로퍼티: pages : 10
};
```

### 객체의 특징

- **중괄호 `{}`**로 감싸서 표현
- **키(key)**와 **값(value)**은 **콜론(`:`)으로 구분**
- 각 프로퍼티는 **쉼표(`,`)로 구분**
- 프로퍼티 이름(키)은 **중복 불가**
- 값은 **모든 타입** 가능 (string, number, array, object, function 등)

## 객체 생성 방법

### 1. 객체 리터럴 (가장 일반적)

```javascript
const user = {
  name: '김개발',
  age: 28,
  job: 'Developer'
};
```

### 2. Object 생성자

```javascript
const user = new Object();
user.name = '김개발';
user.age = 28;
user.job = 'Developer';
```

### 3. Object.create()

```javascript
const personProto = {
  greet() {
    console.log(`안녕하세요, ${this.name}입니다.`);
  }
};

const user = Object.create(personProto);
user.name = '김개발';
user.greet();  // "안녕하세요, 김개발입니다."
```

### 4. 생성자 함수

```javascript
function User(name, age, job) {
  this.name = name;
  this.age = age;
  this.job = job;
}

const user = new User('김개발', 28, 'Developer');
```

### 5. Class (ES6)

```javascript
class User {
  constructor(name, age, job) {
    this.name = name;
    this.age = age;
    this.job = job;
  }
}

const user = new User('김개발', 28, 'Developer');
```

## 프로퍼티 접근 방법

### 1. 점 표기법 (Dot Notation)

```javascript
const plan1 = {
  name: "Basic",
  price: 3.99
};

console.log(plan1.name);   // "Basic"
console.log(plan1.price);  // 3.99
```

### 2. 대괄호 표기법 (Bracket Notation)

```javascript
const plan1 = {
  name: "Basic",
  price: 3.99
};

console.log(plan1["name"]);   // "Basic"
console.log(plan1["price"]);  // 3.99
```

### 점 표기법 vs 대괄호 표기법

| 구분 | 점 표기법 | 대괄호 표기법 |
|------|----------|--------------|
| **문법** | `obj.key` | `obj["key"]` |
| **변수 사용** | 불가능 | 가능 |
| **공백 포함 키** | 불가능 | 가능 |
| **숫자로 시작** | 불가능 | 가능 |
| **동적 키** | 불가능 | 가능 |

### 대괄호 표기법의 강력함

```javascript
const plan1 = {
  name: "Basic",
  price: 3.99
};

// 변수를 키로 사용
let propertyName = "name";
console.log(plan1[propertyName]);  // "Basic"

propertyName = "price";
console.log(plan1[propertyName]);  // 3.99

// 표현식 사용 가능
const myObj = {
  property1: "hello",
  property2: [1, 2, 3, 4, 5],
  property3: {
    childproperty: "haha"
  }
};

const name = "property";
console.log(myObj[name + "1"]);  // "hello"
console.log(myObj[name + "2"]);  // [1, 2, 3, 4, 5]
console.log(myObj[name + "3"]);  // { childproperty: "haha" }

// 중첩된 객체도 접근 가능
console.log(myObj[name + "3"]["child" + name]);  // "haha"
```

## 프로퍼티 추가, 수정, 삭제

### 1. 프로퍼티 추가

```javascript
const user = {
  name: '김개발'
};

// 점 표기법
user.age = 28;

// 대괄호 표기법
user['job'] = 'Developer';

console.log(user);
// { name: '김개발', age: 28, job: 'Developer' }
```

### 2. 프로퍼티 수정

```javascript
let myObj = {
  property1: "hello"
};

// 값 수정
myObj.property1 = "안녕하세요";

// 또는 대괄호 표기법
let name = "property1";
myObj[name] = ["hi", "hello"];

console.log(myObj);
// { property1: ["hi", "hello"] }
```

### 3. 프로퍼티 삭제

```javascript
const user = {
  name: '김개발',
  age: 28,
  job: 'Developer'
};

delete user.age;

console.log(user);
// { name: '김개발', job: 'Developer' }
```

## 중첩 객체 (Nested Objects)

실무에서는 복잡한 데이터 구조를 표현하기 위해 중첩 객체를 자주 사용합니다.

```javascript
let objData = {
  name: "김개발",
  address: {
    email: "gaebal@gmail.com",
    home: "위워크 선릉2호점"
  },
  books: {
    year: [2019, 2018, 2006],
    info: [
      {
        name: "JS Guide",
        price: 9000
      },
      {
        name: "HTML Guide",
        price: 19000,
        author: "Kim, gae bal"
      }
    ]
  }
};

// 중첩 객체 접근
console.log(objData.address.email);
// "gaebal@gmail.com"

console.log(objData.books.year[0]);
// 2019

console.log(objData.books.info[1].name);
// "HTML Guide"

// 대괄호 표기법으로도 가능
let bookName = objData["books"]["info"][1]["name"];
console.log(bookName);
// "HTML Guide"
```

### 배열이 포함된 객체

```javascript
myObj.property1 = ["hi", "hello"];

// 배열 요소 접근
console.log(myObj.property1[0]);  // "hi"
console.log(myObj.property1[1]);  // "hello"

// 중첩 객체에 프로퍼티 추가
myObj.property3.siblingproperty = [3, 6, 9];
console.log(myObj);
```

## Object 메서드

### 1. Object.keys() - 모든 키 가져오기

```javascript
const user = {
  name: '김개발',
  age: 28,
  job: 'Developer'
};

const keys = Object.keys(user);
console.log(keys);
// ['name', 'age', 'job']

// 활용: 프로퍼티 개수 확인
console.log(Object.keys(user).length);  // 3
```

### 2. Object.values() - 모든 값 가져오기

```javascript
const values = Object.values(user);
console.log(values);
// ['김개발', 28, 'Developer']

// 활용: 값의 합 계산
const prices = { apple: 1000, banana: 1500, orange: 2000 };
const total = Object.values(prices).reduce((sum, price) => sum + price, 0);
console.log(total);  // 4500
```

### 3. Object.entries() - 키-값 쌍 배열

```javascript
const entries = Object.entries(user);
console.log(entries);
// [['name', '김개발'], ['age', 28], ['job', 'Developer']]

// 활용: 반복문
Object.entries(user).forEach(([key, value]) => {
  console.log(`${key}: ${value}`);
});
// name: 김개발
// age: 28
// job: Developer
```

### 4. Object.assign() - 객체 병합/복사

```javascript
// 객체 복사
const user1 = { name: '김개발', age: 28 };
const user2 = Object.assign({}, user1);

user2.age = 30;
console.log(user1.age);  // 28 (원본 유지)
console.log(user2.age);  // 30

// 객체 병합
const defaults = { theme: 'light', language: 'ko' };
const userSettings = { language: 'en' };
const settings = Object.assign({}, defaults, userSettings);

console.log(settings);
// { theme: 'light', language: 'en' }
```

### 5. Object.freeze() - 객체 동결

```javascript
const config = Object.freeze({
  API_URL: 'https://api.example.com',
  MAX_RETRY: 3
});

config.API_URL = 'https://hacked.com';  // 무시됨 (strict mode에서는 에러)
console.log(config.API_URL);
// 'https://api.example.com'
```

### 6. Object.seal() - 객체 봉인

```javascript
const user = Object.seal({
  name: '김개발',
  age: 28
});

user.age = 30;        // ✅ 수정 가능
user.job = 'Developer';  // ❌ 추가 불가
delete user.age;      // ❌ 삭제 불가

console.log(user);
// { name: '김개발', age: 30 }
```

### 7. Object.hasOwn() - 프로퍼티 존재 확인 (ES2022)

```javascript
const user = { name: '김개발' };

console.log(Object.hasOwn(user, 'name'));  // true
console.log(Object.hasOwn(user, 'age'));   // false

// 이전 방식 (hasOwnProperty)
console.log(user.hasOwnProperty('name'));  // true
```

## 구조 분해 할당 (Destructuring)

### 기본 구조 분해

```javascript
const user = {
  name: '김개발',
  age: 28,
  job: 'Developer'
};

// ES5 방식
const name = user.name;
const age = user.age;

// ES6 구조 분해 할당
const { name, age, job } = user;

console.log(name);  // '김개발'
console.log(age);   // 28
console.log(job);   // 'Developer'
```

### 변수명 변경

```javascript
const { name: userName, age: userAge } = user;

console.log(userName);  // '김개발'
console.log(userAge);   // 28
```

### 기본값 설정

```javascript
const user = { name: '김개발' };

const { name, age = 25, job = 'Unknown' } = user;

console.log(name);  // '김개발'
console.log(age);   // 25 (기본값)
console.log(job);   // 'Unknown' (기본값)
```

### 중첩 객체 구조 분해

```javascript
const user = {
  name: '김개발',
  address: {
    city: '서울',
    district: '강남구'
  }
};

const { name, address: { city, district } } = user;

console.log(name);      // '김개발'
console.log(city);      // '서울'
console.log(district);  // '강남구'
```

### 함수 매개변수에서 구조 분해

```javascript
function printUser({ name, age, job = 'Unknown' }) {
  console.log(`${name} (${age}세, ${job})`);
}

const user = { name: '김개발', age: 28 };
printUser(user);
// "김개발 (28세, Unknown)"
```

## Optional Chaining (?.)

중첩 객체에서 안전하게 값을 접근할 수 있는 ES2020 문법입니다.

```javascript
const user = {
  name: '김개발',
  address: {
    city: '서울'
  }
};

// ❌ 에러 발생 위험
console.log(user.address.city);        // '서울'
console.log(user.contact.email);       // TypeError!

// ✅ Optional Chaining
console.log(user.address?.city);       // '서울'
console.log(user.contact?.email);      // undefined (에러 없음)

// 중첩된 경우
console.log(user.contact?.email?.primary);  // undefined
```

### 배열과 함께 사용

```javascript
const data = {
  users: [
    { name: '김개발', age: 28 },
    { name: '박코딩', age: 30 }
  ]
};

console.log(data.users?.[0]?.name);  // '김개발'
console.log(data.admins?.[0]?.name); // undefined
```

### 함수 호출과 함께 사용

```javascript
const obj = {
  method: () => console.log('실행됨')
};

obj.method?.();      // '실행됨'
obj.notExist?.();    // 에러 없음 (아무것도 실행 안 됨)
```

## Nullish Coalescing (??)

`null` 또는 `undefined`일 때만 기본값을 사용하는 연산자입니다.

```javascript
const user = {
  name: '김개발',
  age: 0,
  job: null
};

// || 연산자 (falsy 값 모두 체크)
console.log(user.age || 25);   // 25 (0은 falsy)
console.log(user.job || 'Unknown');  // 'Unknown'

// ?? 연산자 (null/undefined만 체크)
console.log(user.age ?? 25);   // 0 (null/undefined가 아님)
console.log(user.job ?? 'Unknown');  // 'Unknown'
```

## 실전 활용 패턴

### 1. API 응답 데이터 처리

```javascript
const apiResponse = {
  status: 'success',
  data: {
    user: {
      id: 123,
      profile: {
        name: '김개발',
        email: 'dev@example.com'
      },
      stats: {
        posts: 42,
        followers: 1520
      }
    }
  }
};

// 안전한 데이터 접근
const userName = apiResponse.data?.user?.profile?.name ?? 'Anonymous';
const followerCount = apiResponse.data?.user?.stats?.followers ?? 0;

console.log(userName);       // '김개발'
console.log(followerCount);  // 1520
```

### 2. 설정 객체 병합

```javascript
function createConfig(userConfig = {}) {
  const defaultConfig = {
    theme: 'light',
    language: 'ko',
    notifications: {
      email: true,
      push: false
    }
  };

  return {
    ...defaultConfig,
    ...userConfig,
    notifications: {
      ...defaultConfig.notifications,
      ...userConfig.notifications
    }
  };
}

const config = createConfig({ theme: 'dark', notifications: { push: true } });
console.log(config);
// {
//   theme: 'dark',
//   language: 'ko',
//   notifications: { email: true, push: true }
// }
```

### 3. 객체 변환

```javascript
const users = [
  { id: 1, name: '김개발', role: 'admin' },
  { id: 2, name: '박코딩', role: 'user' },
  { id: 3, name: '이자바', role: 'user' }
];

// 배열을 객체로 변환 (ID를 키로 사용)
const usersById = users.reduce((acc, user) => {
  acc[user.id] = user;
  return acc;
}, {});

console.log(usersById);
// {
//   1: { id: 1, name: '김개발', role: 'admin' },
//   2: { id: 2, name: '박코딩', role: 'user' },
//   3: { id: 3, name: '이자바', role: 'user' }
// }

// 빠른 조회
console.log(usersById[2].name);  // '박코딩'
```

### 4. 객체 필터링

```javascript
const user = {
  id: 123,
  name: '김개발',
  password: 'secret123',
  email: 'dev@example.com',
  role: 'admin'
};

// 특정 키 제외
function omit(obj, ...keys) {
  return Object.fromEntries(
    Object.entries(obj).filter(([key]) => !keys.includes(key))
  );
}

const publicUser = omit(user, 'password', 'email');
console.log(publicUser);
// { id: 123, name: '김개발', role: 'admin' }

// 특정 키만 선택
function pick(obj, ...keys) {
  return Object.fromEntries(
    keys.map(key => [key, obj[key]])
  );
}

const userInfo = pick(user, 'id', 'name');
console.log(userInfo);
// { id: 123, name: '김개발' }
```

## 비교 정리표

| 접근 방법 | 문법 | 장점 | 단점 |
|----------|------|------|------|
| **점 표기법** | `obj.key` | 간결, 직관적 | 동적 키 불가 |
| **대괄호 표기법** | `obj["key"]` | 동적 키, 특수문자 가능 | 장황함 |
| **구조 분해** | `{ key } = obj` | 여러 값 한번에 추출 | 객체 구조 알아야 함 |
| **Optional Chaining** | `obj?.key` | 안전한 접근 | 최신 문법 |

## Best Practices

### ✅ 권장 사항

```javascript
// 1. 일관된 프로퍼티 접근 방식 사용
const name = user.name;        // ✅ 정적 키는 점 표기법
const key = 'age';
const age = user[key];         // ✅ 동적 키는 대괄호

// 2. 구조 분해로 간결하게
const { name, age } = user;    // ✅

// 3. Optional Chaining으로 안전하게
const email = user.contact?.email;  // ✅

// 4. Spread 연산자로 복사/병합
const newUser = { ...user, age: 30 };  // ✅
```

### ❌ 피해야 할 사항

```javascript
// 1. 존재 여부 확인 없이 접근
const email = user.contact.email;  // ❌ 에러 위험

// 2. 직접 수정 (불변성)
user.age = 30;  // ❌ (React 등에서)
const newUser = { ...user, age: 30 };  // ✅

// 3. 얕은 복사로 중첩 객체 복사
const copy = { ...user };
copy.address.city = 'Busan';  // ❌ 원본도 변경됨!

// 4. 예약어를 키로 사용
const obj = { class: 'A' };  // ❌ 가능하지만 혼란스러움
```

## 핵심 정리

1. **객체는 키-값 쌍의 집합**
2. **접근 방법**: 점 표기법, 대괄호 표기법
3. **동적 키**는 대괄호 표기법 사용
4. **중첩 객체**는 체인으로 접근
5. **구조 분해 할당**으로 간결하게 값 추출
6. **Optional Chaining**으로 안전한 접근
7. **Object 메서드**로 객체 조작

## 참고 자료

- [MDN - Working with Objects](https://developer.mozilla.org/ko/docs/Web/JavaScript/Guide/Working_with_Objects)
- [MDN - Object Methods](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object)
- [MDN - Destructuring Assignment](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)
- [MDN - Optional Chaining](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/Optional_chaining)
