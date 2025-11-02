---
title: JavaScript Arrow Function 완벽 가이드
description: ES6 화살표 함수의 문법, this 바인딩, 사용 사례와 일반 함수와의 차이점을 실무 예제와 함께 살펴봅니다.
author: cotes
date: 2020-07-05 15:28:44 +0900
categories: [JavaScript, ES6]
tags: [javascript, es6, arrow-function, this-binding, lexical-scope]
---

## ES6 Arrow Function이란?

ES6(ECMAScript 2015)에서 도입된 화살표 함수(Arrow Function)는 함수를 더 간결하게 작성할 수 있는 문법입니다. 단순히 문법만 간결해진 것이 아니라, `this` 바인딩 방식이 일반 함수와 근본적으로 다르기 때문에 **언제 사용해야 하는지 정확히 이해하는 것이 중요**합니다.

## 기본 문법

### 1. 기본 형태

```javascript
// ES5 - 일반 함수
function() {}

// ES6 - 화살표 함수
() => {}
```

`function` 키워드가 사라지고 `=>` (화살표)가 추가된 간결한 형태입니다.

### 2. 이름이 있는 함수

```javascript
// ES5 - 함수 선언문
function getName() {
  return 'John';
}

// ES5 - 함수 표현식
const getName = function() {
  return 'John';
};

// ES6 - 화살표 함수
const getName = () => {
  return 'John';
};
```

화살표 함수는 항상 **함수 표현식**으로만 사용할 수 있으며, 변수에 할당하거나 콜백으로 전달됩니다.

### 3. 매개변수 처리

```javascript
// 매개변수가 없을 때
const greet = () => console.log('Hello');

// 매개변수가 1개일 때 - 소괄호 생략 가능
const greet = name => console.log(`Hello, ${name}`);
const greet = (name) => console.log(`Hello, ${name}`); // 이것도 가능

// 매개변수가 2개 이상일 때 - 소괄호 필수
const greet = (firstName, lastName) => {
  console.log(`Hello, ${firstName} ${lastName}`);
};
```

> **Best Practice**: 일관성을 위해 매개변수가 1개일 때도 소괄호를 항상 사용하는 것을 권장합니다.
{: .prompt-tip }

### 4. 함수 본문과 반환값

```javascript
// 실행 코드가 여러 줄인 경우 - 중괄호와 return 필요
const add = (a, b) => {
  const result = a + b;
  return result;
};

// 표현식만 반환하는 경우 - 중괄호와 return 생략 가능
const add = (a, b) => a + b;

// 객체를 반환하는 경우 - 소괄호로 감싸기
const createUser = (name, age) => ({ name, age });

// 이렇게 하면 에러! (객체 리터럴을 함수 본문으로 인식)
const createUser = (name, age) => { name, age };
```

## 일반 함수와의 핵심 차이점

### 1. this 바인딩 (가장 중요!)

화살표 함수와 일반 함수의 **가장 큰 차이점**은 `this` 처리 방식입니다.

```javascript
// 일반 함수 - this는 호출 방식에 따라 동적으로 결정
const obj = {
  name: 'Alice',
  greet: function() {
    console.log(this.name); // 'Alice'

    setTimeout(function() {
      console.log(this.name); // undefined (this가 전역 객체를 가리킴)
    }, 1000);
  }
};

// 화살표 함수 - this는 상위 스코프의 this를 그대로 사용 (Lexical this)
const obj2 = {
  name: 'Bob',
  greet: function() {
    console.log(this.name); // 'Bob'

    setTimeout(() => {
      console.log(this.name); // 'Bob' (상위 스코프의 this를 유지)
    }, 1000);
  }
};
```

**화살표 함수는 자신만의 `this`를 생성하지 않고, 선언된 위치의 상위 스코프 `this`를 그대로 사용합니다.**

### 2. arguments 객체

```javascript
// 일반 함수 - arguments 객체 사용 가능
function sum() {
  console.log(arguments); // [1, 2, 3]
  return Array.from(arguments).reduce((a, b) => a + b, 0);
}
sum(1, 2, 3); // 6

// 화살표 함수 - arguments 객체 없음
const sum2 = () => {
  console.log(arguments); // ReferenceError!
};

// 대신 rest 파라미터 사용
const sum3 = (...args) => {
  console.log(args); // [1, 2, 3]
  return args.reduce((a, b) => a + b, 0);
};
sum3(1, 2, 3); // 6
```

### 3. 생성자 함수로 사용 불가

```javascript
// 일반 함수 - 생성자로 사용 가능
function Person(name) {
  this.name = name;
}
const person = new Person('Alice'); // OK

// 화살표 함수 - 생성자로 사용 불가
const Person2 = (name) => {
  this.name = name;
};
const person2 = new Person2('Bob'); // TypeError: Person2 is not a constructor
```

## 실무 사용 사례

### 1. 배열 메서드 콜백

화살표 함수는 배열 메서드의 콜백으로 사용할 때 매우 깔끔합니다.

```javascript
// ES5 방식
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map(function(num) {
  return num * 2;
});

// ES6 화살표 함수
const doubled = numbers.map(num => num * 2);

// 복잡한 변환도 간결하게
const users = [
  { name: 'Alice', age: 25 },
  { name: 'Bob', age: 30 },
  { name: 'Charlie', age: 35 }
];

const names = users
  .filter(user => user.age >= 30)
  .map(user => user.name);
// ['Bob', 'Charlie']
```

### 2. 이벤트 핸들러에서 this 사용

```javascript
class Button {
  constructor() {
    this.count = 0;
  }

  // 일반 함수 - this 바인딩 문제 발생
  setupWrong() {
    document.querySelector('#btn').addEventListener('click', function() {
      this.count++; // Error! this는 button 요소를 가리킴
      console.log(this.count);
    });
  }

  // 화살표 함수 - this가 클래스 인스턴스를 유지
  setupCorrect() {
    document.querySelector('#btn').addEventListener('click', () => {
      this.count++; // OK! this는 Button 인스턴스를 가리킨다
      console.log(this.count);
    });
  }
}
```

### 3. Promise 체이닝

```javascript
class UserService {
  constructor() {
    this.users = [];
  }

  fetchUsers() {
    return fetch('/api/users')
      .then(response => response.json())
      .then(data => {
        this.users = data; // 화살표 함수로 this 유지
        return this.users;
      })
      .catch(error => {
        console.error('Error:', error);
        throw error;
      });
  }
}
```

## 화살표 함수를 사용하면 안 되는 경우

### 1. 객체 메서드

```javascript
// 잘못된 사용
const person = {
  name: 'Alice',
  greet: () => {
    console.log(`Hello, ${this.name}`); // undefined! this가 전역 객체
  }
};

// 올바른 사용 - 일반 함수 또는 메서드 축약형
const person = {
  name: 'Alice',
  greet() {
    console.log(`Hello, ${this.name}`); // 'Hello, Alice'
  }
};
```

### 2. 프로토타입 메서드

```javascript
// 잘못된 사용
Person.prototype.greet = () => {
  console.log(`Hello, ${this.name}`); // this가 인스턴스를 가리키지 않음
};

// 올바른 사용
Person.prototype.greet = function() {
  console.log(`Hello, ${this.name}`);
};
```

### 3. 동적 this가 필요한 경우

```javascript
// jQuery 이벤트 핸들러 등에서 this가 이벤트 대상을 가리켜야 할 때
$('.button').on('click', function() {
  $(this).addClass('active'); // this는 클릭된 버튼
});

// 화살표 함수를 사용하면 안 됨
$('.button').on('click', () => {
  $(this).addClass('active'); // this는 상위 스코프의 this
});
```

## 성능 고려사항

화살표 함수는 일반 함수보다 약간 더 빠른 생성 속도를 가집니다:

```javascript
// 벤치마크 예제
console.time('Regular Function');
for (let i = 0; i < 1000000; i++) {
  const fn = function() { return i; };
}
console.timeEnd('Regular Function'); // ~25ms

console.time('Arrow Function');
for (let i = 0; i < 1000000; i++) {
  const fn = () => i;
}
console.timeEnd('Arrow Function'); // ~20ms
```

하지만 실무에서는 이 차이가 거의 무의미하므로, **가독성과 this 바인딩 동작**을 우선 고려해야 합니다.

## 비교 정리표

| 특성 | 일반 함수 | 화살표 함수 |
|------|----------|------------|
| `this` 바인딩 | 동적 (호출 방식에 따라 결정) | 렉시컬 (상위 스코프의 this) |
| `arguments` 객체 | 사용 가능 | 사용 불가 (rest 파라미터 사용) |
| 생성자 함수 | 사용 가능 (`new` 키워드) | 사용 불가 |
| `prototype` 속성 | 있음 | 없음 |
| 메서드 축약형 | 불가능 | 불가능 (화살표 함수 자체가 표현식) |
| 호이스팅 | 함수 선언문은 호이스팅됨 | 변수 호이스팅 규칙 따름 |

## Best Practices

1. **배열 메서드, 콜백에는 화살표 함수 사용**
   ```javascript
   [1, 2, 3].map(x => x * 2);
   setTimeout(() => console.log('Done'), 1000);
   ```

2. **객체 메서드에는 일반 함수 또는 메서드 축약형 사용**
   ```javascript
   const obj = {
     method() { /* ... */ }
   };
   ```

3. **클래스 메서드는 상황에 따라 선택**
   ```javascript
   class MyClass {
     // 일반 메서드
     normalMethod() { }

     // 화살표 함수 필드 (this 바인딩 유지)
     arrowMethod = () => { }
   }
   ```

4. **간결성과 명확성의 균형 유지**
   ```javascript
   // 너무 복잡하면 명확성을 위해 중괄호 사용
   const process = (data) => {
     const cleaned = data.trim();
     const parsed = JSON.parse(cleaned);
     return parsed;
   };
   ```

## 결론

화살표 함수는 단순한 문법적 설탕(Syntactic Sugar)이 아닙니다. `this` 바인딩의 동작 방식이 근본적으로 다르기 때문에:

- **콜백 함수, 배열 메서드, Promise 체이닝 등**에서는 화살표 함수가 더 직관적
- **객체 메서드, 프로토타입 메서드, 생성자 함수**에서는 일반 함수가 적합

각 상황에 맞는 함수 타입을 선택하여 버그를 예방하고 가독성 높은 코드를 작성하시기 바랍니다.

## 참고 자료

- [MDN - Arrow Functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)
- [ECMAScript 2015 Specification](https://www.ecma-international.org/ecma-262/6.0/)
- [JavaScript.info - Arrow Functions](https://javascript.info/arrow-functions)
