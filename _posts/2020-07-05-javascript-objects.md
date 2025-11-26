---
title: JavaScript 객체(Object) 완벽 가이드 - 핵심 개념부터 실전까지
description: JavaScript 객체의 개념부터 프로퍼티 접근, 메서드, 중첩 객체, 참조 타입까지 상세히 알아봅니다. 객체 리터럴, 구조 분해 할당, Object 메서드, 불변성 관리까지 실전에서 바로 활용할 수 있는 가이드를 제공합니다.
author: changsu
date: 2020-07-05 12:34:16 +0900
categories: [Programming, JavaScript]
tags: [javascript, object, destructuring]
---

## JavaScript 객체란?

**객체(Object)**는 키(key)-값(value) 쌍으로 이루어진 데이터 집합입니다. JavaScript에서 가장 중요한 데이터 타입 중 하나입니다.

### 배열 vs 객체

```javascript
// 배열: 순서가 있는 데이터 (인덱스로 접근)
let arr = ['고양이', '귀여워', '멍멍이', '좋아'];
console.log(arr[0]);  // "고양이"
console.log(arr[1]);  // "귀여워"

// 객체: 순서가 없는 데이터 (키로 접근)
let obj = {
  animal1: '고양이',
  feeling1: '귀여워',
  animal2: '멍멍이',
  feeling2: '좋아'
};
console.log(obj.animal1);  // "고양이"
console.log(obj.feeling1); // "귀여워"
```

### 객체가 필요한 이유

```javascript
// ❌ 변수만 사용 (관리 어려움)
let userName = "홍길동";
let userAge = 30;
let userCity = "서울";

// ✅ 객체 사용 (관련 데이터 그룹화)
let user = {
  name: "홍길동",
  age: 30,
  city: "서울"
};
```

## 객체 생성

### 객체 리터럴 (권장)

```javascript
// 빈 객체
let empty = {};

// 데이터가 있는 객체
let person = {
  name: "홍길동",
  age: 30,
  city: "서울"
};

console.log(typeof person);  // "object"
```

### Object 생성자

```javascript
// 빈 객체
let obj1 = new Object();

// 프로퍼티 추가
obj1.name = "김철수";
obj1.age = 25;

console.log(obj1);  // { name: "김철수", age: 25 }
```

### Object.create()

```javascript
// 프로토타입 지정하여 생성
let prototype = {
  greet: function() {
    console.log("Hello!");
  }
};

let obj = Object.create(prototype);
obj.greet();  // "Hello!"
```

## 프로퍼티 (Property)

프로퍼티는 **키(key)**와 **값(value)**으로 구성됩니다.

### 프로퍼티 키 규칙

```javascript
let obj = {
  // 일반 키 (따옴표 생략 가능)
  name: "홍길동",
  age: 30,

  // 공백 포함 키 (따옴표 필수)
  'my name': 'boong',

  // 한글 키 (따옴표 생략 가능)
  키: '한글인 키는 따옴표가 없어도 되는군!!',

  // 특수문자 키 (따옴표 필수)
  '!키': '느낌표 있는 키는 따옴표가 필요하군',

  // $ 기호 (따옴표 생략 가능)
  $special: '$는 없어도 되는군',

  // 숫자 키
  33: '숫자 형식도 되네'
};
```

**키 명명 규칙**:
- 공백, 특수문자 없으면 따옴표 생략 가능
- `$`, `_`, 한글은 따옴표 없이 사용 가능
- 공백이나 특수문자가 있으면 따옴표 필수
- 숫자는 자동으로 문자열로 변환됨

### 프로퍼티 값 타입

```javascript
let obj = {
  // 문자열
  name: "홍길동",

  // 숫자
  age: 30,

  // 불리언
  isActive: true,

  // 배열
  hobbies: ['독서', '영화', '운동'],

  // 객체
  address: {
    city: "서울",
    district: "강남구"
  },

  // 함수 (메서드)
  greet: function() {
    console.log("안녕하세요!");
  },

  // null
  profile: null,

  // undefined
  nickname: undefined
};
```

## 프로퍼티 접근

### 점 표기법 (Dot Notation)

```javascript
let person = {
  name: "홍길동",
  age: 30,
  city: "서울"
};

// 점 표기법 (권장)
console.log(person.name);  // "홍길동"
console.log(person.age);   // 30
console.log(person.city);  // "서울"
```

### 대괄호 표기법 (Bracket Notation)

```javascript
let obj = {
  'my name': 'boong',
  color: 'silver',
  33: '숫자 키'
};

// 대괄호 표기법 (키를 문자열로)
console.log(obj['my name']);  // "boong"
console.log(obj['color']);    // "silver"
console.log(obj['33']);       // "숫자 키"
console.log(obj[33]);         // "숫자 키" (숫자도 가능)

// 변수로 키 접근
let key = 'color';
console.log(obj[key]);  // "silver"
```

### 점 vs 대괄호

| 경우 | 점 표기법 | 대괄호 표기법 |
|------|-----------|---------------|
| 일반 키 | ✅ `obj.name` | ✅ `obj['name']` |
| 공백 포함 | ❌ | ✅ `obj['my name']` |
| 숫자 키 | ❌ | ✅ `obj[33]` |
| 변수 사용 | ❌ | ✅ `obj[key]` |
| 동적 키 | ❌ | ✅ `obj[computedKey]` |

### 존재하지 않는 프로퍼티

```javascript
let person = {
  name: "홍길동"
};

console.log(person.age);  // undefined
console.log(person.city); // undefined

// 프로퍼티 존재 여부 확인
console.log('name' in person);  // true
console.log('age' in person);   // false
```

## 프로퍼티 추가/수정/삭제

### 추가

```javascript
let person = {
  name: "홍길동"
};

// 새 프로퍼티 추가
person.age = 30;
person.city = "서울";

console.log(person);
// { name: "홍길동", age: 30, city: "서울" }
```

### 수정

```javascript
let person = {
  name: "홍길동",
  age: 30
};

// 프로퍼티 수정
person.age = 31;
person.name = "김철수";

console.log(person);
// { name: "김철수", age: 31 }
```

### 삭제

```javascript
let person = {
  name: "홍길동",
  age: 30,
  city: "서울"
};

// delete 연산자로 삭제
delete person.city;

console.log(person);
// { name: "홍길동", age: 30 }

console.log(person.city);  // undefined
```

## const와 객체

### const 객체의 특징

```javascript
const person = {
  name: "홍길동"
};

// ❌ 객체 자체 재할당 불가
// person = { name: "김철수" };  // TypeError

// ✅ 프로퍼티 수정/추가 가능
person.name = "김철수";  // 가능
person.age = 30;         // 가능

console.log(person);
// { name: "김철수", age: 30 }
```

**이유**: const는 **변수가 가리키는 참조(reference)**를 변경할 수 없지만, 참조가 가리키는 **객체 내부**는 변경 가능합니다.

### 객체 불변성 보장

```javascript
// Object.freeze() - 완전 동결
const frozen = Object.freeze({
  name: "홍길동",
  age: 30
});

frozen.age = 31;           // 무시됨 (strict mode에서 에러)
frozen.city = "서울";      // 무시됨
delete frozen.name;        // 무시됨

console.log(frozen);  // { name: "홍길동", age: 30 }

// Object.seal() - 프로퍼티 추가/삭제 금지, 수정은 가능
const sealed = Object.seal({
  name: "홍길동",
  age: 30
});

sealed.age = 31;           // 가능
sealed.city = "서울";      // 무시됨
delete sealed.name;        // 무시됨

console.log(sealed);  // { name: "홍길동", age: 31 }
```

## 메서드 (Method)

객체에 저장된 함수를 **메서드**라고 합니다.

### 메서드 정의

```javascript
let person = {
  name: "홍길동",
  age: 30,

  // 메서드 정의 (기존 방식)
  greet: function() {
    console.log("안녕하세요!");
  },

  // 메서드 축약 표기 (ES6)
  introduce() {
    console.log(`저는 ${this.name}입니다.`);
  }
};

// 메서드 호출
person.greet();       // "안녕하세요!"
person.introduce();   // "저는 홍길동입니다."
```

### this 키워드

```javascript
let person = {
  name: "홍길동",
  age: 30,

  introduce() {
    console.log(`제 이름은 ${this.name}이고, ${this.age}세입니다.`);
  },

  getBirthYear() {
    return new Date().getFullYear() - this.age;
  }
};

person.introduce();
// "제 이름은 홍길동이고, 30세입니다."

console.log(person.getBirthYear());  // 1990 (2020년 기준)
```

### 화살표 함수 주의

```javascript
let person = {
  name: "홍길동",

  // ❌ 화살표 함수 (this가 객체를 가리키지 않음)
  greet1: () => {
    console.log(`안녕하세요, ${this.name}님!`);  // undefined
  },

  // ✅ 일반 함수
  greet2() {
    console.log(`안녕하세요, ${this.name}님!`);  // "홍길동"
  }
};

person.greet1();  // "안녕하세요, undefined님!"
person.greet2();  // "안녕하세요, 홍길동님!"
```

## 중첩 객체 (Nested Object)

객체 안에 객체가 있는 구조입니다.

### 중첩 객체 접근

```javascript
let user = {
  name: "홍길동",
  age: 30,
  address: {
    city: "서울",
    district: "강남구",
    street: "테헤란로"
  },
  hobbies: ["독서", "영화"]
};

// 중첩 객체 접근
console.log(user.address.city);       // "서울"
console.log(user.address.district);   // "강남구"
console.log(user.hobbies[0]);         // "독서"
```

### 복잡한 중첩 구조

```javascript
let nestedObj = {
  type: {
    year: '2019',
    'comment-type': [{
      name: 'simple',
      id: 1
    }, {
      name: 'complex',
      id: 2
    }]
  }
};

// 깊은 중첩 접근
console.log(nestedObj.type.year);
// "2019"

console.log(nestedObj.type['comment-type'][0].name);
// "simple"

console.log(nestedObj.type['comment-type'][1].id);
// 2
```

**접근 순서 이해**:
1. `nestedObj.type` → `{ year: '2019', 'comment-type': [...] }`
2. `nestedObj.type['comment-type']` → `[{name: 'simple', id: 1}, ...]`
3. `nestedObj.type['comment-type'][0]` → `{name: 'simple', id: 1}`
4. `nestedObj.type['comment-type'][0].name` → `'simple'`

## 참조 타입 (Reference Type)

객체는 **참조(reference)**로 저장됩니다.

### 원시 타입 vs 참조 타입

```javascript
// 원시 타입: 값 자체를 저장
const a = '안녕';
const b = '안녕';
console.log(a === b);  // true (값이 같음)

// 참조 타입: 메모리 주소(참조)를 저장
const obj1 = { name: '안녕' };
const obj2 = { name: '안녕' };
console.log(obj1 === obj2);  // false (참조가 다름)

// 프로퍼티 값 비교
console.log(obj1.name === obj2.name);  // true (값은 같음)
```

### 참조 복사

```javascript
let original = {
  name: "홍길동",
  age: 30
};

// 참조 복사 (같은 객체를 가리킴)
let copy = original;

// copy를 수정하면 original도 변경됨
copy.age = 31;

console.log(original.age);  // 31 (변경됨!)
console.log(copy.age);      // 31

console.log(original === copy);  // true (같은 참조)
```

### 얕은 복사 (Shallow Copy)

```javascript
let original = {
  name: "홍길동",
  age: 30
};

// 방법 1: Spread 연산자
let copy1 = { ...original };

// 방법 2: Object.assign()
let copy2 = Object.assign({}, original);

// 독립적인 객체
copy1.age = 31;
console.log(original.age);  // 30 (변경 안됨)
console.log(copy1.age);     // 31

console.log(original === copy1);  // false (다른 참조)
```

### 깊은 복사 (Deep Copy)

```javascript
let original = {
  name: "홍길동",
  address: {
    city: "서울",
    district: "강남구"
  }
};

// 얕은 복사 (중첩 객체는 참조 복사됨)
let shallowCopy = { ...original };
shallowCopy.address.city = "부산";
console.log(original.address.city);  // "부산" (변경됨!)

// 깊은 복사 (JSON 사용)
let deepCopy = JSON.parse(JSON.stringify(original));
deepCopy.address.city = "대구";
console.log(original.address.city);  // "서울" (변경 안됨)

// 깊은 복사 (재귀 함수)
function deepClone(obj) {
  if (obj === null || typeof obj !== 'object') return obj;

  const clone = Array.isArray(obj) ? [] : {};

  for (let key in obj) {
    if (obj.hasOwnProperty(key)) {
      clone[key] = deepClone(obj[key]);
    }
  }

  return clone;
}

let deepCopy2 = deepClone(original);
```

## 객체 메서드

### Object.keys()

```javascript
let person = {
  name: "홍길동",
  age: 30,
  city: "서울"
};

let keys = Object.keys(person);
console.log(keys);  // ["name", "age", "city"]
```

### Object.values()

```javascript
let values = Object.values(person);
console.log(values);  // ["홍길동", 30, "서울"]
```

### Object.entries()

```javascript
let entries = Object.entries(person);
console.log(entries);
// [["name", "홍길동"], ["age", 30], ["city", "서울"]]

// 순회
for (let [key, value] of Object.entries(person)) {
  console.log(`${key}: ${value}`);
}
// name: 홍길동
// age: 30
// city: 서울
```

### Object.assign()

```javascript
let target = { a: 1 };
let source1 = { b: 2 };
let source2 = { c: 3 };

Object.assign(target, source1, source2);
console.log(target);  // { a: 1, b: 2, c: 3 }

// 객체 병합
let obj1 = { name: "홍길동" };
let obj2 = { age: 30 };
let merged = Object.assign({}, obj1, obj2);
console.log(merged);  // { name: "홍길동", age: 30 }
```

## 구조 분해 할당 (Destructuring)

### 기본 구조 분해

```javascript
let person = {
  name: "홍길동",
  age: 30,
  city: "서울"
};

// 구조 분해 할당
let { name, age, city } = person;

console.log(name);  // "홍길동"
console.log(age);   // 30
console.log(city);  // "서울"
```

### 변수명 변경

```javascript
let { name: userName, age: userAge } = person;

console.log(userName);  // "홍길동"
console.log(userAge);   // 30
```

### 기본값 설정

```javascript
let user = {
  name: "홍길동"
};

let { name, age = 25, city = "서울" } = user;

console.log(name);  // "홍길동"
console.log(age);   // 25 (기본값)
console.log(city);  // "서울" (기본값)
```

### 중첩 구조 분해

```javascript
let user = {
  name: "홍길동",
  address: {
    city: "서울",
    district: "강남구"
  }
};

let { name, address: { city, district } } = user;

console.log(name);      // "홍길동"
console.log(city);      // "서울"
console.log(district);  // "강남구"
```

## 실전 예제

### 1. 사용자 정보 관리

```javascript
function createUser(name, age, email) {
  return {
    name,
    age,
    email,
    createdAt: new Date(),
    isActive: true,

    getInfo() {
      return `${this.name} (${this.age}세)`;
    },

    deactivate() {
      this.isActive = false;
    }
  };
}

let user = createUser("홍길동", 30, "hong@example.com");
console.log(user.getInfo());  // "홍길동 (30세)"
```

### 2. 설정 객체 병합

```javascript
function initApp(userConfig) {
  const defaultConfig = {
    theme: 'light',
    language: 'ko',
    showNotifications: true,
    maxItems: 10
  };

  return { ...defaultConfig, ...userConfig };
}

let config = initApp({ theme: 'dark', maxItems: 20 });
console.log(config);
// { theme: 'dark', language: 'ko', showNotifications: true, maxItems: 20 }
```

### 3. 객체 필터링

```javascript
function filterObject(obj, predicate) {
  return Object.keys(obj)
    .filter(key => predicate(obj[key], key))
    .reduce((result, key) => {
      result[key] = obj[key];
      return result;
    }, {});
}

let numbers = { a: 1, b: 2, c: 3, d: 4, e: 5 };
let evens = filterObject(numbers, value => value % 2 === 0);
console.log(evens);  // { b: 2, d: 4 }
```

## 마치며

JavaScript 객체는 데이터를 구조화하는 핵심 도구입니다:
1. **객체 생성**: 리터럴 `{}` 사용 (권장)
2. **프로퍼티 접근**: 점 표기법, 대괄호 표기법
3. **메서드**: 객체에 저장된 함수
4. **참조 타입**: 얕은 복사와 깊은 복사 이해
5. **Object 메서드**: keys, values, entries, assign 활용
6. **구조 분해 할당**: 편리한 프로퍼티 추출

const로 선언한 객체도 내부는 변경 가능하다는 점, 객체는 참조로 저장된다는 점을 항상 기억하세요.

## 참고 자료

- [MDN - Object](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object)
- [MDN - Working with objects](https://developer.mozilla.org/ko/docs/Web/JavaScript/Guide/Working_with_Objects)
- [MDN - Object initializer](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/Object_initializer)
- [MDN - Destructuring assignment](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)
- [JavaScript.info - Objects](https://javascript.info/object)
- [JavaScript.info - Object references and copying](https://javascript.info/object-copy)
