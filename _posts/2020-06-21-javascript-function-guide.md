---
title: JavaScript 함수 완벽 가이드 - 기초부터 실전까지
description: JavaScript 함수의 기본 개념부터 다양한 선언 방식, 매개변수와 인자, 반환값, 스코프까지 상세히 알아봅니다. 실무에서 바로 활용할 수 있는 예제와 모범 사례를 제공합니다.
author: changsu
date: 2020-06-21 00:42:13 +0900
categories: [Programming, JavaScript]
tags: [javascript, function, parameter, argument, return, scope, arrow-function, callback]
---

## 함수란?

**함수(Function)**는 특정 작업을 수행하도록 설계된 독립적인 코드 블록입니다. 재사용 가능한 코드 조각을 만들어 프로그램의 구조를 체계화하고 유지보수를 용이하게 합니다.

### 함수의 필요성

```javascript
// ❌ 함수 없이 반복적인 코드
console.log("홍길동님 환영합니다!");
console.log("김철수님 환영합니다!");
console.log("이영희님 환영합니다!");

// ✅ 함수를 사용한 재사용 가능한 코드
function greet(name) {
  console.log(name + "님 환영합니다!");
}

greet("홍길동");
greet("김철수");
greet("이영희");
```

### 함수의 장점

| 장점 | 설명 |
|------|------|
| **재사용성** | 한 번 정의한 함수를 여러 곳에서 반복 사용 |
| **모듈화** | 복잡한 프로그램을 작은 단위로 분리 |
| **유지보수** | 코드 수정이 필요할 때 한 곳만 변경 |
| **가독성** | 함수 이름으로 코드의 의도 명확히 전달 |
| **추상화** | 복잡한 구현을 숨기고 간단한 인터페이스 제공 |

## 함수의 기본 구조

### 함수의 구성 요소

```javascript
function functionName(parameter1, parameter2) {
  // 함수 본문(body) - 실행될 코드
  let result = parameter1 + parameter2;
  return result;  // 반환값
}
```

**구성 요소 설명**:

1. **`function` 키워드**: 함수 선언 시작
2. **함수 이름**: 함수를 식별하는 이름
3. **매개변수(Parameters)**: 함수가 받을 입력값 (선택사항)
4. **함수 본문**: 중괄호 `{}` 안의 실행 코드
5. **`return` 문**: 함수 실행 결과 반환 (선택사항)

### 기본 예제

```javascript
function test() {
  let hi = "안녕하세요";
  return hi;
}

// 함수 호출
let greeting = test();
console.log(greeting);  // "안녕하세요"
```

## 함수 정의와 호출

### 정의(Definition)

함수를 정의하는 것은 함수를 생성하는 과정입니다. 정의만으로는 함수가 실행되지 않습니다.

```javascript
// 함수 정의
function test() {
  let a = 1 + 1;
  console.log("함수가 실행되었습니다!");
  return a;
}

// 이 시점에서는 아무것도 출력되지 않음
```

### 호출(Call/Invocation)

함수 이름 뒤에 `()`를 붙여 함수를 실행합니다.

```javascript
// 함수 호출
test();  // "함수가 실행되었습니다!" 출력

// 반환값을 변수에 저장
let result = test();
console.log(result);  // 2
```

**중요**: 함수를 호출하기 전까지는 함수 내부의 코드가 실행되지 않습니다.

## 함수 선언 방식

JavaScript에서는 여러 방식으로 함수를 선언할 수 있습니다.

### 1. 함수 선언식 (Function Declaration)

```javascript
function add(a, b) {
  return a + b;
}

console.log(add(5, 3));  // 8
```

**특징**:
- 호이스팅(Hoisting) 발생
- 선언 전에 호출 가능

```javascript
// 선언 전 호출 가능 (호이스팅)
console.log(multiply(4, 5));  // 20

function multiply(a, b) {
  return a * b;
}
```

### 2. 함수 표현식 (Function Expression)

```javascript
const subtract = function(a, b) {
  return a - b;
};

console.log(subtract(10, 3));  // 7
```

**특징**:
- 변수에 함수를 할당
- 호이스팅 발생하지 않음
- 선언 전 호출 불가

```javascript
// ❌ 에러 발생!
console.log(divide(10, 2));  // ReferenceError

const divide = function(a, b) {
  return a / b;
};
```

### 3. 화살표 함수 (Arrow Function) - ES6

```javascript
const multiply = (a, b) => {
  return a * b;
};

// 간결한 형태 (한 줄일 때 return 생략 가능)
const square = x => x * x;

console.log(multiply(3, 4));  // 12
console.log(square(5));       // 25
```

**화살표 함수의 특징**:

```javascript
// 매개변수가 없을 때
const greet = () => "Hello!";

// 매개변수가 하나일 때 (괄호 생략 가능)
const double = x => x * 2;

// 매개변수가 여러 개일 때
const add = (a, b) => a + b;

// 여러 줄일 때
const calculateArea = (width, height) => {
  const area = width * height;
  return area;
};

// 객체 반환 시 괄호로 감싸기
const makePerson = (name, age) => ({ name, age });
```

### 함수 선언 방식 비교

| 방식 | 호이스팅 | this 바인딩 | 사용 시기 |
|------|----------|-------------|-----------|
| **함수 선언식** | O | 동적 | 전역/독립 함수 |
| **함수 표현식** | X | 동적 | 변수에 할당 |
| **화살표 함수** | X | 렉시컬 | 콜백, 간결한 함수 |

## 매개변수와 인자

### 매개변수 (Parameter)

함수 정의 시 선언하는 변수입니다.

```javascript
function getName(name) {  // name은 매개변수
  return name + '님';
}
```

### 인자 (Argument)

함수 호출 시 전달하는 실제 값입니다.

```javascript
let result = getName('개발자');  // '개발자'는 인자
console.log(result);  // "개발자님"
```

### 여러 매개변수 사용

```javascript
function introduce(name, age, job) {
  return `안녕하세요, 저는 ${age}세 ${job}인 ${name}입니다.`;
}

console.log(introduce('홍길동', 30, '개발자'));
// "안녕하세요, 저는 30세 개발자인 홍길동입니다."
```

### 기본 매개변수 (Default Parameters) - ES6

```javascript
function greet(name = '손님') {
  return `안녕하세요, ${name}님!`;
}

console.log(greet());          // "안녕하세요, 손님님!"
console.log(greet('김철수'));  // "안녕하세요, 김철수님!"
```

### 나머지 매개변수 (Rest Parameters) - ES6

```javascript
function sum(...numbers) {
  return numbers.reduce((total, num) => total + num, 0);
}

console.log(sum(1, 2, 3));        // 6
console.log(sum(1, 2, 3, 4, 5));  // 15
```

## 반환값 (Return Value)

### 명시적 반환

`return` 키워드를 사용하여 값을 반환합니다.

```javascript
function add(a, b) {
  return a + b;
}

let result = add(5, 3);
console.log(result);  // 8
```

### 암시적 반환 (undefined)

`return` 문이 없으면 `undefined`를 반환합니다.

```javascript
function greet(name) {
  console.log(`Hello, ${name}!`);
  // return 문 없음
}

let result = greet('Alice');
console.log(result);  // undefined
```

### return의 역할

1. **값 반환**: 함수 실행 결과를 호출자에게 전달
2. **함수 종료**: `return` 이후 코드는 실행되지 않음

```javascript
function checkAge(age) {
  if (age < 18) {
    return "미성년자";
    // 아래 코드는 실행되지 않음
  }
  return "성인";
}

console.log(checkAge(15));  // "미성년자"
console.log(checkAge(25));  // "성인"
```

### 여러 값 반환 (배열/객체 사용)

```javascript
// 배열로 반환
function getMinMax(numbers) {
  return [Math.min(...numbers), Math.max(...numbers)];
}

const [min, max] = getMinMax([5, 2, 8, 1, 9]);
console.log(min, max);  // 1 9

// 객체로 반환
function getUserInfo() {
  return {
    name: '홍길동',
    age: 30,
    job: '개발자'
  };
}

const user = getUserInfo();
console.log(user.name);  // "홍길동"
```

## 함수 스코프

### 지역 변수 (Local Variable)

함수 내부에서 선언한 변수는 함수 내부에서만 접근 가능합니다.

```javascript
function test() {
  let localVar = "지역 변수";
  console.log(localVar);  // "지역 변수"
}

test();
console.log(localVar);  // ReferenceError: localVar is not defined
```

### 전역 변수 (Global Variable)

함수 외부에서 선언한 변수는 어디서든 접근 가능합니다.

```javascript
let globalVar = "전역 변수";

function test() {
  console.log(globalVar);  // "전역 변수" (접근 가능)
}

test();
console.log(globalVar);  // "전역 변수"
```

### 스코프 체인

```javascript
let global = "전역";

function outer() {
  let outerVar = "외부 함수";

  function inner() {
    let innerVar = "내부 함수";
    console.log(innerVar);   // "내부 함수"
    console.log(outerVar);   // "외부 함수" (접근 가능)
    console.log(global);     // "전역" (접근 가능)
  }

  inner();
}

outer();
```

## 실전 예제

### 1. 계산기 함수

```javascript
const calculator = {
  add: (a, b) => a + b,
  subtract: (a, b) => a - b,
  multiply: (a, b) => a * b,
  divide: (a, b) => b !== 0 ? a / b : 'Error: Division by zero'
};

console.log(calculator.add(10, 5));       // 15
console.log(calculator.divide(10, 0));    // "Error: Division by zero"
```

### 2. 배열 처리 함수

```javascript
function getEvenNumbers(numbers) {
  return numbers.filter(num => num % 2 === 0);
}

function sumArray(numbers) {
  return numbers.reduce((sum, num) => sum + num, 0);
}

const nums = [1, 2, 3, 4, 5, 6];
console.log(getEvenNumbers(nums));  // [2, 4, 6]
console.log(sumArray(nums));        // 21
```

### 3. 문자열 처리 함수

```javascript
function capitalize(str) {
  return str.charAt(0).toUpperCase() + str.slice(1).toLowerCase();
}

function truncate(str, maxLength) {
  return str.length > maxLength
    ? str.slice(0, maxLength) + '...'
    : str;
}

console.log(capitalize('hello'));           // "Hello"
console.log(truncate('Long text here', 8)); // "Long tex..."
```

### 4. 유효성 검사 함수

```javascript
function isValidEmail(email) {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
}

function isValidPassword(password) {
  return password.length >= 8 && /[A-Z]/.test(password) && /[0-9]/.test(password);
}

console.log(isValidEmail('test@example.com'));  // true
console.log(isValidEmail('invalid-email'));     // false
console.log(isValidPassword('Pass123'));        // true
console.log(isValidPassword('weak'));           // false
```

## 콜백 함수

함수를 다른 함수의 인자로 전달할 수 있습니다.

```javascript
function processUserInput(callback) {
  const name = '홍길동';
  callback(name);
}

processUserInput(function(name) {
  console.log(`안녕하세요, ${name}님!`);
});

// 화살표 함수 사용
processUserInput(name => console.log(`반갑습니다, ${name}님!`));
```

### 배열 메서드와 콜백

```javascript
const numbers = [1, 2, 3, 4, 5];

// forEach
numbers.forEach(num => console.log(num * 2));

// map
const doubled = numbers.map(num => num * 2);
console.log(doubled);  // [2, 4, 6, 8, 10]

// filter
const evens = numbers.filter(num => num % 2 === 0);
console.log(evens);  // [2, 4]

// reduce
const sum = numbers.reduce((total, num) => total + num, 0);
console.log(sum);  // 15
```

## 즉시 실행 함수 (IIFE)

정의와 동시에 실행되는 함수입니다.

```javascript
(function() {
  console.log('즉시 실행됩니다!');
})();

// 화살표 함수 버전
(() => {
  console.log('화살표 함수로 즉시 실행!');
})();

// 매개변수 전달
(function(name) {
  console.log(`Hello, ${name}!`);
})('Alice');
```

## 재귀 함수

함수가 자기 자신을 호출하는 함수입니다.

```javascript
// 팩토리얼 계산
function factorial(n) {
  if (n <= 1) return 1;
  return n * factorial(n - 1);
}

console.log(factorial(5));  // 120 (5 * 4 * 3 * 2 * 1)

// 피보나치 수열
function fibonacci(n) {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
}

console.log(fibonacci(7));  // 13
```

## 모범 사례

### 1. 명확한 함수 이름 사용

```javascript
// ❌ 나쁜 예
function f(x) {
  return x * 2;
}

// ✅ 좋은 예
function doubleNumber(number) {
  return number * 2;
}
```

### 2. 단일 책임 원칙

```javascript
// ❌ 나쁜 예: 여러 작업을 한 함수에서 처리
function processUser(user) {
  validateUser(user);
  saveUser(user);
  sendEmail(user);
  logActivity(user);
}

// ✅ 좋은 예: 각 작업을 별도 함수로 분리
function processUser(user) {
  if (!isValidUser(user)) return false;

  const saved = saveUser(user);
  sendWelcomeEmail(user);
  logUserActivity(user, 'registered');

  return saved;
}
```

### 3. 순수 함수 작성

```javascript
// ❌ 나쁜 예: 외부 상태 변경
let total = 0;
function addToTotal(value) {
  total += value;  // 외부 변수 수정
}

// ✅ 좋은 예: 순수 함수
function add(a, b) {
  return a + b;  // 외부 상태에 영향 없음
}
```

### 4. 기본 매개변수 활용

```javascript
// ❌ 나쁜 예
function greet(name) {
  if (!name) {
    name = '손님';
  }
  return `안녕하세요, ${name}님!`;
}

// ✅ 좋은 예
function greet(name = '손님') {
  return `안녕하세요, ${name}님!`;
}
```

### 5. 조기 반환 (Early Return)

```javascript
// ❌ 나쁜 예: 중첩된 조건문
function processOrder(order) {
  if (order) {
    if (order.items && order.items.length > 0) {
      if (order.total > 0) {
        return '주문 처리 완료';
      }
    }
  }
  return '주문 처리 실패';
}

// ✅ 좋은 예: 조기 반환
function processOrder(order) {
  if (!order) return '주문 처리 실패';
  if (!order.items || order.items.length === 0) return '주문 처리 실패';
  if (order.total <= 0) return '주문 처리 실패';

  return '주문 처리 완료';
}
```

## 마치며

JavaScript 함수는 프로그래밍의 핵심 빌딩 블록입니다:

1. **함수 선언**: 함수 선언식, 함수 표현식, 화살표 함수
2. **매개변수와 인자**: 입력값 전달과 기본값 설정
3. **반환값**: `return`을 통한 결과 반환
4. **스코프**: 지역 변수와 전역 변수의 접근 범위
5. **고급 개념**: 콜백 함수, 재귀 함수, IIFE

함수를 효과적으로 사용하면 코드의 재사용성과 유지보수성이 크게 향상됩니다. 명확한 함수 이름, 단일 책임 원칙, 순수 함수 작성 등의 모범 사례를 따라 깔끔하고 이해하기 쉬운 코드를 작성하세요.


## 참고 자료

- [MDN - Functions](https://developer.mozilla.org/ko/docs/Web/JavaScript/Guide/Functions)
- [MDN - Arrow Functions](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Functions/Arrow_functions)
- [MDN - Default Parameters](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Functions/Default_parameters)
- [MDN - Rest Parameters](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Functions/rest_parameters)
- [JavaScript.info - Functions](https://javascript.info/function-basics)
- [JavaScript.info - Arrow Functions](https://javascript.info/arrow-functions-basics)
