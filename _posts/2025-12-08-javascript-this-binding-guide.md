---
title: "JavaScript this 바인딩 완벽 가이드 - 실행 컨텍스트와 바인딩 규칙의 모든 것"
description: "JavaScript this 키워드의 4가지 바인딩 규칙, 화살표 함수의 렉시컬 this, call/apply/bind 메서드 활용법까지 실무 예제와 함께 완벽하게 이해합니다."
date: 2025-12-08 10:00:00 +0900
categories: [Frontend, JavaScript]
tags: [javascript, this, binding, call, apply, bind, arrow-function, execution-context]
---

## 개요

JavaScript에서 `this` 키워드는 가장 혼란스러운 개념 중 하나입니다. 다른 프로그래밍 언어에서 `this`는 항상 인스턴스 자신을 가리키지만, JavaScript의 `this`는 **함수가 호출되는 방식**에 따라 동적으로 결정됩니다.

이 글에서는 `this` 바인딩의 4가지 규칙을 체계적으로 정리하고, 실무에서 자주 마주치는 문제 상황과 해결 방법을 코드 예제와 함께 설명합니다.

### 학습 목표

- `this`가 결정되는 시점과 원리 이해
- 4가지 바인딩 규칙과 우선순위 파악
- `call`, `apply`, `bind` 메서드 활용법 습득
- 화살표 함수의 렉시컬 `this` 이해
- 실무에서 발생하는 `this` 문제 해결 능력 향상

### 사전 지식

- JavaScript 함수 기초 ([JavaScript 함수 완벽 가이드](/posts/javascript-function-guide/) 참고)
- 화살표 함수 기초 ([JavaScript Arrow Function 완벽 가이드](/posts/javascript-arrow-function/) 참고)
- 객체와 메서드 개념 ([JavaScript 객체 완벽 가이드](/posts/javascript-objects/) 참고)

---

## this란 무엇인가?

### 정의

`this`는 현재 실행 중인 코드에서 **자신이 속한 객체 또는 자신이 생성할 인스턴스**를 가리키는 자기 참조 변수(self-referencing variable)입니다.

> MDN 정의: "In most cases, the value of `this` is determined by how a function is called (runtime binding)."

### 다른 언어와의 차이점

Java나 C++에서 `this`는 항상 클래스의 인스턴스를 가리킵니다. 하지만 JavaScript에서는 **함수 호출 방식에 따라 동적으로 결정**됩니다.

```javascript
// Java (정적 this)
public class Person {
    private String name;
    public void greet() {
        System.out.println(this.name); // this는 항상 Person 인스턴스
    }
}

// JavaScript (동적 this)
const person = {
  name: '홍길동',
  greet() {
    console.log(this.name); // this는 호출 방식에 따라 달라짐
  }
};

const greetFn = person.greet;
greetFn(); // undefined - this가 전역 객체를 가리킴
```

### 정적 스코프 vs 동적 바인딩

JavaScript의 **변수 스코프**는 **렉시컬(정적) 스코프**를 따릅니다. 함수가 **정의된 위치**에서 스코프가 결정됩니다.

하지만 `this`는 **동적 바인딩**입니다. 함수가 **호출되는 방식**에 따라 결정됩니다.

```javascript
const outer = {
  value: 10,
  inner: {
    value: 20,
    getValue() {
      return this.value; // this는 호출 시점에 결정
    }
  }
};

console.log(outer.inner.getValue()); // 20 - this는 inner 객체

const fn = outer.inner.getValue;
console.log(fn()); // undefined - this는 전역 객체
```

---

## 4가지 this 바인딩 규칙

JavaScript에서 `this`가 결정되는 4가지 규칙을 알아보겠습니다.

### 1. 기본 바인딩 (Default Binding)

함수를 **단독 호출**할 때 적용되는 기본 규칙입니다.

```javascript
function showThis() {
  console.log(this);
}

showThis(); // 브라우저: window, Node.js: global (비엄격 모드)
```

#### strict mode에서의 기본 바인딩

엄격 모드에서는 기본 바인딩 시 `this`가 `undefined`가 됩니다.

```javascript
'use strict';

function showThis() {
  console.log(this);
}

showThis(); // undefined
```

#### 실행 환경별 전역 객체

| 환경 | 전역 객체 | 접근 방법 |
|------|-----------|-----------|
| 브라우저 | `window` | `window`, `self`, `globalThis` |
| Node.js | `global` | `global`, `globalThis` |
| Web Worker | `self` | `self`, `globalThis` |

> ES2020부터 `globalThis`를 사용하면 모든 환경에서 전역 객체에 접근할 수 있습니다.
{: .prompt-tip }

### 2. 암시적 바인딩 (Implicit Binding)

함수가 **객체의 메서드로 호출**될 때 적용됩니다. `this`는 **메서드를 호출한 객체**에 바인딩됩니다.

```javascript
const user = {
  name: '김철수',
  age: 30,
  introduce() {
    console.log(`안녕하세요, ${this.name}입니다. ${this.age}살입니다.`);
  }
};

user.introduce(); // "안녕하세요, 김철수입니다. 30살입니다."
// this는 user 객체를 가리킴
```

#### 중첩 객체에서의 암시적 바인딩

중첩된 객체에서 메서드를 호출하면 **직접 호출한 객체**에 `this`가 바인딩됩니다.

```javascript
const company = {
  name: '테크회사',
  department: {
    name: '개발팀',
    showName() {
      console.log(this.name);
    }
  }
};

company.department.showName(); // "개발팀" - department 객체에 바인딩
```

#### 암시적 바인딩의 소실 (Implicit Binding Loss)

가장 흔한 실수 중 하나입니다. 메서드를 변수에 할당하거나 콜백으로 전달하면 `this` 바인딩이 소실됩니다.

```javascript
const user = {
  name: '홍길동',
  greet() {
    console.log(`안녕하세요, ${this.name}입니다.`);
  }
};

// 메서드 참조를 변수에 할당
const greetFn = user.greet;
greetFn(); // "안녕하세요, undefined입니다." - this가 전역 객체

// 콜백으로 전달
setTimeout(user.greet, 1000); // "안녕하세요, undefined입니다."
```

### 3. 명시적 바인딩 (Explicit Binding)

`call`, `apply`, `bind` 메서드를 사용해 `this`를 **명시적으로 지정**합니다.

#### call 메서드

함수를 즉시 호출하면서 `this`를 지정합니다. 인수를 **개별적**으로 전달합니다.

```javascript
function greet(greeting, punctuation) {
  console.log(`${greeting}, ${this.name}${punctuation}`);
}

const person = { name: '이영희' };

greet.call(person, '안녕하세요', '!');
// "안녕하세요, 이영희!"
```

#### apply 메서드

`call`과 동일하지만 인수를 **배열**로 전달합니다.

```javascript
function greet(greeting, punctuation) {
  console.log(`${greeting}, ${this.name}${punctuation}`);
}

const person = { name: '박민수' };

greet.apply(person, ['반갑습니다', '~']);
// "반갑습니다, 박민수~"
```

#### bind 메서드

`this`가 바인딩된 **새로운 함수**를 반환합니다. 나중에 호출할 수 있습니다.

```javascript
function greet(greeting) {
  console.log(`${greeting}, ${this.name}입니다.`);
}

const person = { name: '최지우' };

const boundGreet = greet.bind(person);
boundGreet('안녕하세요'); // "안녕하세요, 최지우입니다."

// 부분 적용도 가능
const sayHello = greet.bind(person, '안녕');
sayHello(); // "안녕, 최지우입니다."
```

#### call vs apply vs bind 비교

| 메서드 | 즉시 실행 | 인수 전달 방식 | 반환값 |
|--------|-----------|----------------|--------|
| `call` | O | 개별 인수 | 함수 실행 결과 |
| `apply` | O | 배열 | 함수 실행 결과 |
| `bind` | X | 개별 인수 | 바인딩된 새 함수 |

### 4. new 바인딩

`new` 연산자와 함께 **생성자 함수**를 호출하면, `this`는 새로 생성되는 **인스턴스 객체**에 바인딩됩니다.

```javascript
function Person(name, age) {
  // this = {};  (암묵적으로 빈 객체 생성)
  this.name = name;
  this.age = age;
  // return this; (암묵적으로 this 반환)
}

const person1 = new Person('김철수', 25);
console.log(person1.name); // "김철수"
console.log(person1.age);  // 25

const person2 = new Person('이영희', 30);
console.log(person2.name); // "이영희"
```

#### new 연산자의 동작 과정

1. 빈 객체가 생성되고 `this`에 바인딩
2. 생성된 객체의 `[[Prototype]]`이 생성자 함수의 `prototype` 속성에 연결
3. 생성자 함수 내부 코드 실행 (`this`를 통해 프로퍼티 추가)
4. 생성자가 객체를 반환하지 않으면 `this`가 암묵적으로 반환

```javascript
// new 연산자의 동작을 수동으로 구현
function myNew(Constructor, ...args) {
  // 1. 빈 객체 생성 및 프로토타입 연결
  const obj = Object.create(Constructor.prototype);
  // 2. 생성자 함수 실행 (this 바인딩)
  const result = Constructor.apply(obj, args);
  // 3. 생성자가 객체를 반환하면 그 객체를, 아니면 생성한 객체 반환
  return result instanceof Object ? result : obj;
}

function User(name) {
  this.name = name;
}

const user = myNew(User, '테스트');
console.log(user.name); // "테스트"
console.log(user instanceof User); // true
```

---

## 바인딩 우선순위

4가지 바인딩 규칙이 충돌할 때의 **우선순위**입니다.

### 우선순위 순서

1. **new 바인딩** (가장 높음)
2. **명시적 바인딩** (`call`, `apply`, `bind`)
3. **암시적 바인딩** (객체 메서드)
4. **기본 바인딩** (가장 낮음)

### new 바인딩 vs 암시적 바인딩

```javascript
function Person(name) {
  this.name = name;
}

const obj = {
  Person: Person
};

// 암시적 바인딩
const person1 = obj.Person('암시적');
console.log(obj.name); // "암시적"

// new 바인딩이 우선
const person2 = new obj.Person('new 바인딩');
console.log(person2.name); // "new 바인딩"
console.log(obj.name);     // "암시적" (변경되지 않음)
```

### new 바인딩 vs 명시적 바인딩 (bind)

```javascript
function Person(name) {
  this.name = name;
}

const existingObj = { name: '기존 객체' };
const BoundPerson = Person.bind(existingObj);

// bind로 바인딩된 함수여도 new가 우선
const newPerson = new BoundPerson('new가 우선');
console.log(newPerson.name);    // "new가 우선"
console.log(existingObj.name);  // "기존 객체" (변경되지 않음)
```

> `bind`로 고정된 `this`도 `new` 연산자 앞에서는 무시됩니다.
{: .prompt-info }

### 명시적 바인딩 vs 암시적 바인딩

```javascript
function showName() {
  console.log(this.name);
}

const obj1 = { name: 'obj1', showName };
const obj2 = { name: 'obj2' };

// 암시적 바인딩
obj1.showName(); // "obj1"

// 명시적 바인딩이 우선
obj1.showName.call(obj2); // "obj2"
```

### 바인딩 우선순위 결정 흐름도

```
함수 호출 시 this 결정 흐름:

1. new로 호출되었는가?
   → YES: this = 새로 생성된 객체
   → NO: 다음 단계

2. call/apply로 호출되었는가? 또는 bind로 생성된 함수인가?
   → YES: this = 명시적으로 지정된 객체
   → NO: 다음 단계

3. 객체의 메서드로 호출되었는가? (obj.method())
   → YES: this = 해당 객체
   → NO: 다음 단계

4. 기본 바인딩
   → strict mode: this = undefined
   → non-strict mode: this = 전역 객체
```

---

## 화살표 함수(Arrow Function)의 this

화살표 함수는 일반 함수와 완전히 다른 `this` 바인딩 메커니즘을 가집니다.

### 렉시컬 this 바인딩

화살표 함수는 **자신만의 `this`를 가지지 않습니다.** 대신 화살표 함수가 **정의된 위치**의 상위 스코프 `this`를 그대로 사용합니다.

```javascript
const obj = {
  name: '객체',
  regularFunc() {
    console.log('일반 함수:', this.name);

    const arrowFunc = () => {
      console.log('화살표 함수:', this.name);
    };

    arrowFunc();
  }
};

obj.regularFunc();
// "일반 함수: 객체"
// "화살표 함수: 객체"  - 상위 스코프(regularFunc)의 this를 사용
```

### 일반 함수 vs 화살표 함수 this 비교

```javascript
const obj = {
  name: 'obj',

  // 일반 함수: 호출 방식에 따라 this 결정
  regularMethod: function() {
    console.log('regular:', this.name);
  },

  // 화살표 함수: 정의 시점의 상위 스코프 this 사용
  arrowMethod: () => {
    console.log('arrow:', this.name);
  }
};

obj.regularMethod(); // "regular: obj"
obj.arrowMethod();   // "arrow: undefined" - 상위 스코프(전역)의 this

const regular = obj.regularMethod;
const arrow = obj.arrowMethod;

regular(); // "regular: undefined" - 기본 바인딩
arrow();   // "arrow: undefined"  - 여전히 전역 this
```

### 화살표 함수에서 call/apply/bind가 무시되는 이유

화살표 함수는 자신만의 `this`가 없으므로 `call`, `apply`, `bind`로 `this`를 변경할 수 없습니다.

```javascript
const arrow = () => {
  console.log(this.name);
};

const obj = { name: '객체' };

// 모두 무시됨 - 여전히 상위 스코프의 this 사용
arrow.call(obj);       // undefined
arrow.apply(obj);      // undefined

const bound = arrow.bind(obj);
bound();               // undefined
```

### 화살표 함수를 사용해야 하는 경우

#### 1. 콜백 함수에서 외부 this 유지

```javascript
class Timer {
  constructor() {
    this.seconds = 0;
  }

  start() {
    // 화살표 함수로 this 유지
    setInterval(() => {
      this.seconds++;
      console.log(`${this.seconds}초 경과`);
    }, 1000);
  }
}

const timer = new Timer();
timer.start();
// "1초 경과"
// "2초 경과"
// ...
```

#### 2. 배열 메서드 콜백

```javascript
const team = {
  name: '개발팀',
  members: ['철수', '영희', '민수'],

  showMembers() {
    this.members.forEach(member => {
      console.log(`${this.name}: ${member}`);
    });
  }
};

team.showMembers();
// "개발팀: 철수"
// "개발팀: 영희"
// "개발팀: 민수"
```

### 화살표 함수를 사용하면 안 되는 경우

#### 1. 객체 메서드

```javascript
// 잘못된 사용
const user = {
  name: '홍길동',
  greet: () => {
    console.log(`안녕, ${this.name}`); // this는 전역 객체
  }
};

user.greet(); // "안녕, undefined"

// 올바른 사용
const user2 = {
  name: '홍길동',
  greet() {
    console.log(`안녕, ${this.name}`);
  }
};

user2.greet(); // "안녕, 홍길동"
```

#### 2. 프로토타입 메서드

```javascript
// 잘못된 사용
function Person(name) {
  this.name = name;
}

Person.prototype.greet = () => {
  console.log(`안녕, ${this.name}`); // this가 인스턴스가 아님
};

const person = new Person('김철수');
person.greet(); // "안녕, undefined"

// 올바른 사용
Person.prototype.greet = function() {
  console.log(`안녕, ${this.name}`);
};
```

#### 3. 생성자 함수

```javascript
// 화살표 함수는 생성자로 사용 불가
const Person = (name) => {
  this.name = name;
};

const person = new Person('홍길동');
// TypeError: Person is not a constructor
```

#### 4. 이벤트 핸들러에서 동적 this가 필요한 경우

```javascript
// 클릭된 요소에 접근해야 할 때
document.querySelector('.btn').addEventListener('click', function() {
  console.log(this); // 클릭된 버튼 요소
  this.classList.add('active');
});

// 화살표 함수를 사용하면 버튼 요소에 접근 불가
document.querySelector('.btn').addEventListener('click', () => {
  console.log(this); // window (또는 undefined)
});

// 단, 외부 상태를 참조해야 할 때는 화살표 함수가 유용
class ButtonHandler {
  constructor() {
    this.count = 0;
  }

  init() {
    document.querySelector('.btn').addEventListener('click', () => {
      this.count++; // 클래스 인스턴스의 count에 접근
      console.log(`클릭 횟수: ${this.count}`);
    });
  }
}
```

---

## call, apply, bind 심화

### 실무 활용 패턴

#### 1. 유사 배열 객체를 배열로 변환

```javascript
function convertArgs() {
  // arguments는 유사 배열 객체
  const args = Array.prototype.slice.call(arguments);
  // 또는 ES6: const args = Array.from(arguments);
  // 또는 ES6: const args = [...arguments];

  return args.map(x => x * 2);
}

console.log(convertArgs(1, 2, 3)); // [2, 4, 6]
```

#### 2. 함수 빌려쓰기 (Method Borrowing)

```javascript
const calculator = {
  numbers: [1, 2, 3, 4, 5],
  sum() {
    return this.numbers.reduce((a, b) => a + b, 0);
  }
};

const anotherData = {
  numbers: [10, 20, 30]
};

// calculator의 sum 메서드를 빌려서 사용
const result = calculator.sum.call(anotherData);
console.log(result); // 60
```

#### 3. 최댓값/최솟값 찾기

```javascript
const numbers = [5, 6, 2, 3, 7];

// apply를 사용해 배열을 개별 인수로 전달
const max = Math.max.apply(null, numbers);
const min = Math.min.apply(null, numbers);

console.log(max); // 7
console.log(min); // 2

// ES6 spread 연산자로 더 간단하게
const max2 = Math.max(...numbers);
const min2 = Math.min(...numbers);
```

#### 4. 부분 적용 함수 (Partial Application)

```javascript
function multiply(a, b, c) {
  return a * b * c;
}

// 첫 번째 인수를 2로 고정
const double = multiply.bind(null, 2);
console.log(double(3, 4)); // 24 (2 * 3 * 4)

// 두 개의 인수를 고정
const multiplyBy6 = multiply.bind(null, 2, 3);
console.log(multiplyBy6(4)); // 24 (2 * 3 * 4)
```

#### 5. 이벤트 핸들러 바인딩

```javascript
class Counter {
  constructor() {
    this.count = 0;
    this.button = document.querySelector('#counter-btn');

    // 방법 1: bind 사용
    this.button.addEventListener('click', this.increment.bind(this));

    // 방법 2: 화살표 함수로 래핑
    this.button.addEventListener('click', () => this.increment());

    // 방법 3: 클래스 필드로 화살표 함수 정의
    // increment = () => { this.count++; }
  }

  increment() {
    this.count++;
    console.log(`Count: ${this.count}`);
  }
}
```

### bind의 고급 활용

#### 함수 컨텍스트 고정

```javascript
const logger = {
  prefix: '[LOG]',
  log(message) {
    console.log(`${this.prefix} ${message}`);
  }
};

// 다른 곳에서 사용할 때 this 유지
const boundLog = logger.log.bind(logger);

// setTimeout에서도 this가 유지됨
setTimeout(boundLog, 1000, '1초 후 메시지');

// 콜백으로 전달해도 this 유지
[1, 2, 3].forEach(boundLog);
// "[LOG] 1"
// "[LOG] 2"
// "[LOG] 3"
```

---

## 클래스와 this

ES6 클래스에서의 `this` 바인딩을 살펴봅니다.

### 클래스 메서드에서의 this

```javascript
class Person {
  constructor(name) {
    this.name = name;
  }

  greet() {
    console.log(`안녕하세요, ${this.name}입니다.`);
  }
}

const person = new Person('홍길동');
person.greet(); // "안녕하세요, 홍길동입니다."
```

### 이벤트 핸들러에서 this 잃어버리기 문제

```javascript
class Button {
  constructor(label) {
    this.label = label;
    this.element = document.createElement('button');
    this.element.textContent = label;

    // 문제: this가 버튼 요소를 가리킴
    this.element.addEventListener('click', this.handleClick);
  }

  handleClick() {
    console.log(`${this.label} 클릭됨`); // this.label이 undefined
  }
}
```

### 해결 방법들

#### 방법 1: 생성자에서 bind

```javascript
class Button {
  constructor(label) {
    this.label = label;
    this.handleClick = this.handleClick.bind(this);

    this.element = document.createElement('button');
    this.element.addEventListener('click', this.handleClick);
  }

  handleClick() {
    console.log(`${this.label} 클릭됨`);
  }
}
```

#### 방법 2: 클래스 필드로 화살표 함수 정의

```javascript
class Button {
  constructor(label) {
    this.label = label;
    this.element = document.createElement('button');
    this.element.addEventListener('click', this.handleClick);
  }

  // 클래스 필드 문법 - 인스턴스마다 새 함수 생성
  handleClick = () => {
    console.log(`${this.label} 클릭됨`);
  };
}
```

#### 방법 3: 이벤트 등록 시 화살표 함수로 래핑

```javascript
class Button {
  constructor(label) {
    this.label = label;
    this.element = document.createElement('button');
    this.element.addEventListener('click', () => this.handleClick());
  }

  handleClick() {
    console.log(`${this.label} 클릭됨`);
  }
}
```

### 각 방법의 장단점

| 방법 | 장점 | 단점 |
|------|------|------|
| bind | 프로토타입 메서드 유지, 테스트 용이 | 생성자 코드 증가 |
| 클래스 필드 화살표 함수 | 간결한 문법 | 인스턴스마다 함수 생성, 상속 시 주의 |
| 래핑 화살표 함수 | 유연성 높음 | 이벤트 제거 시 참조 보관 필요 |

---

## React에서의 this

### 클래스 컴포넌트에서의 this 문제

```tsx
import React, { Component } from 'react';

class Counter extends Component {
  state = { count: 0 };

  // 문제: 이벤트 핸들러에서 this가 undefined
  handleClickWrong() {
    this.setState({ count: this.state.count + 1 }); // Error!
  }

  render() {
    return (
      <button onClick={this.handleClickWrong}>
        {this.state.count}
      </button>
    );
  }
}
```

### 클래스 컴포넌트 해결 방법

```tsx
import React, { Component } from 'react';

class Counter extends Component {
  state = { count: 0 };

  constructor(props) {
    super(props);
    // 방법 1: 생성자에서 bind
    this.handleClick1 = this.handleClick1.bind(this);
  }

  handleClick1() {
    this.setState({ count: this.state.count + 1 });
  }

  // 방법 2: 클래스 필드로 화살표 함수
  handleClick2 = () => {
    this.setState({ count: this.state.count + 1 });
  };

  // 방법 3: render에서 화살표 함수 (비권장 - 매 렌더마다 새 함수 생성)
  render() {
    return (
      <div>
        <button onClick={this.handleClick1}>방법 1</button>
        <button onClick={this.handleClick2}>방법 2</button>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          방법 3 (비권장)
        </button>
      </div>
    );
  }
}
```

### 함수형 컴포넌트로의 전환

함수형 컴포넌트에서는 `this`를 사용하지 않으므로 바인딩 문제가 없습니다.

```tsx
import React, { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  const handleClick = () => {
    setCount(count + 1);
  };

  return (
    <button onClick={handleClick}>
      {count}
    </button>
  );
}
```

> React 16.8 이후 Hooks 도입으로 함수형 컴포넌트가 권장됩니다. 더 자세한 내용은 [React Hooks 성능 최적화 가이드](/posts/react-hooks-performance-optimization/)를 참고하세요.
{: .prompt-tip }

---

## 흔한 실수와 해결법

### 1. 콜백 함수에서 this 잃어버리기

```javascript
const user = {
  name: '홍길동',
  friends: ['철수', '영희'],

  // 문제: forEach 콜백에서 this 소실
  showFriendsWrong() {
    this.friends.forEach(function(friend) {
      console.log(`${this.name}의 친구: ${friend}`); // this.name이 undefined
    });
  },

  // 해결 1: 화살표 함수 사용
  showFriends1() {
    this.friends.forEach(friend => {
      console.log(`${this.name}의 친구: ${friend}`);
    });
  },

  // 해결 2: thisArg 매개변수 사용
  showFriends2() {
    this.friends.forEach(function(friend) {
      console.log(`${this.name}의 친구: ${friend}`);
    }, this);
  },

  // 해결 3: 변수에 this 저장
  showFriends3() {
    const self = this;
    this.friends.forEach(function(friend) {
      console.log(`${self.name}의 친구: ${friend}`);
    });
  }
};
```

### 2. 메서드 추출 시 this 문제

```javascript
const calculator = {
  value: 0,
  add(num) {
    this.value += num;
    return this;
  },
  getValue() {
    return this.value;
  }
};

// 문제: 메서드 추출
const { getValue } = calculator;
console.log(getValue()); // undefined (this.value 접근 불가)

// 해결: bind 사용
const boundGetValue = calculator.getValue.bind(calculator);
console.log(boundGetValue()); // 0
```

### 3. 중첩 함수에서의 this

```javascript
const obj = {
  name: '객체',

  // 문제: 중첩 함수에서 this 소실
  methodWrong() {
    console.log('외부:', this.name); // "객체"

    function inner() {
      console.log('내부:', this.name); // undefined
    }
    inner();
  },

  // 해결 1: 화살표 함수
  method1() {
    console.log('외부:', this.name);

    const inner = () => {
      console.log('내부:', this.name);
    };
    inner();
  },

  // 해결 2: call/apply 사용
  method2() {
    console.log('외부:', this.name);

    function inner() {
      console.log('내부:', this.name);
    }
    inner.call(this);
  }
};
```

### 4. setTimeout/setInterval에서의 this

```javascript
class Countdown {
  constructor(seconds) {
    this.seconds = seconds;
  }

  // 문제: setTimeout 콜백에서 this 소실
  startWrong() {
    setTimeout(function() {
      console.log(this.seconds); // undefined
    }, 1000);
  }

  // 해결: 화살표 함수 사용
  start() {
    setTimeout(() => {
      console.log(this.seconds);
    }, 1000);
  }
}
```

---

## 실무 베스트 프랙티스

### this 바인딩 관련 코딩 컨벤션

#### 1. 객체 메서드는 단축 구문 사용

```javascript
// 권장
const obj = {
  method() {
    console.log(this);
  }
};

// 피해야 함
const obj2 = {
  method: () => {
    console.log(this); // 의도한 this가 아닐 수 있음
  }
};
```

#### 2. 콜백에서는 화살표 함수 우선

```javascript
// 권장
array.map(item => item.value);
setTimeout(() => this.update(), 1000);

// 상황에 따라 선택
element.addEventListener('click', (e) => {
  // this.handleClick(e) - 외부 this 필요 시
});

element.addEventListener('click', function(e) {
  // this - 이벤트 대상 요소 필요 시
});
```

#### 3. 클래스에서는 클래스 필드 화살표 함수 활용

```javascript
class Component {
  state = {};

  // 이벤트 핸들러는 화살표 함수로
  handleClick = () => {
    this.setState({ clicked: true });
  };

  // 일반 메서드는 단축 구문으로
  render() {
    return this.state;
  }
}
```

### TypeScript에서의 this 타입 지정

#### 1. this 매개변수

```typescript
interface User {
  name: string;
  greet(this: User): void;
}

const user: User = {
  name: '홍길동',
  greet() {
    console.log(`안녕, ${this.name}`);
  }
};

user.greet(); // OK

const greet = user.greet;
greet(); // TypeScript 에러: this가 void 타입
```

#### 2. ThisParameterType과 OmitThisParameter

```typescript
function toHex(this: Number) {
  return this.toString(16);
}

// this 타입 추출
type ThisType = ThisParameterType<typeof toHex>; // Number

// this 매개변수 제거한 함수 타입
type PureToHex = OmitThisParameter<typeof toHex>; // () => string
```

#### 3. ThisType 유틸리티

```typescript
interface Data {
  x: number;
  y: number;
}

interface Methods {
  moveBy(dx: number, dy: number): void;
}

// ThisType을 사용해 this 타입 지정
const obj: Data & Methods & ThisType<Data & Methods> = {
  x: 0,
  y: 0,
  moveBy(dx, dy) {
    this.x += dx; // this가 Data & Methods 타입으로 추론
    this.y += dy;
  }
};
```

---

## 핵심 정리

### this 바인딩 규칙 요약표

| 호출 방식 | this 바인딩 | 예시 |
|-----------|-------------|------|
| 일반 함수 호출 | 전역 객체 (strict: undefined) | `fn()` |
| 메서드 호출 | 호출한 객체 | `obj.method()` |
| 생성자 호출 | 새로 생성된 인스턴스 | `new Fn()` |
| call/apply | 첫 번째 인수 | `fn.call(obj)` |
| bind | 바인딩된 객체 | `fn.bind(obj)()` |
| 화살표 함수 | 상위 스코프의 this | `() => this` |

### 화살표 함수 사용 가이드

| 상황 | 화살표 함수 | 일반 함수 |
|------|-------------|-----------|
| 배열 메서드 콜백 | O | - |
| setTimeout/setInterval | O | - |
| Promise 체이닝 | O | - |
| 객체 메서드 | - | O |
| 프로토타입 메서드 | - | O |
| 생성자 함수 | - | O |
| 동적 this 필요 시 | - | O |

### 기억해야 할 핵심 원칙

1. **this는 호출 시점에 결정된다** - 함수를 어떻게 호출하느냐가 중요
2. **화살표 함수는 예외** - 정의 시점의 상위 스코프 this를 사용
3. **new가 가장 강력하다** - 모든 바인딩을 이기고 새 객체에 바인딩
4. **bind는 한 번만 적용** - 이미 바인딩된 함수를 다시 bind해도 변경 안 됨
5. **strict mode를 사용하라** - 기본 바인딩에서 undefined로 명확한 에러 발생

---

## 참고 자료

- [MDN - this](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/this)
- [MDN - Function.prototype.bind()](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_objects/Function/bind)
- [MDN - Function.prototype.call()](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_objects/Function/call)
- [MDN - Function.prototype.apply()](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_objects/Function/apply)
- [MDN - Arrow functions](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Functions/Arrow_functions)
- [JavaScript.info - Object methods, "this"](https://ko.javascript.info/object-methods)
- [You Don't Know JS - this & Object Prototypes](https://github.com/getify/You-Dont-Know-JS/blob/1st-ed/this%20%26%20object%20prototypes/README.md)
