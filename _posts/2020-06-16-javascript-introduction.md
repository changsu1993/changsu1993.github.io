---
title: JavaScript 입문 가이드 - 웹을 동적으로 만드는 언어
description: JavaScript의 기본 개념, 역할, 실행 환경을 학습합니다. alert, console.log부터 시작하는 JavaScript 첫걸음입니다.
author: changsu
date: 2020-06-16 09:40:30 +0900
categories: [Programming, JavaScript]
tags: [javascript, js, web, frontend, programming, console, alert]
---

## 개요

JavaScript는 웹 페이지를 동적이고 상호작용 가능하게 만드는 프로그래밍 언어입니다. HTML이 웹의 구조를, CSS가 디자인을 담당한다면, JavaScript는 웹의 동작과 상호작용을 담당합니다. 현대 웹 개발에서 필수적인 언어로, 프론트엔드 개발자의 핵심 역할은 JavaScript를 사용해 사용자 경험을 향상시키는 것입니다.

## 1. JavaScript란?

### 웹의 3요소

<div style="max-width: 600px; margin: 30px auto; padding: 20px; background: #f8f9fa; border-radius: 10px;">
  <!-- HTML -->
  <div style="margin-bottom: 20px; padding: 20px; background: linear-gradient(135deg, #e74c3c 0%, #c0392b 100%); border-radius: 8px; box-shadow: 0 4px 6px rgba(231, 76, 60, 0.3);">
    <div style="color: white; font-size: 22px; font-weight: bold; margin-bottom: 8px; text-align: center;">
      HTML
    </div>
    <div style="color: rgba(255, 255, 255, 0.9); font-size: 14px; text-align: center; margin-bottom: 5px;">
      구조 (Structure)
    </div>
    <div style="color: rgba(255, 255, 255, 0.85); font-size: 13px; text-align: center;">
      웹 페이지의 뼈대
    </div>
  </div>

  <!-- CSS -->
  <div style="margin-bottom: 20px; padding: 20px; background: linear-gradient(135deg, #3498db 0%, #2980b9 100%); border-radius: 8px; box-shadow: 0 4px 6px rgba(52, 152, 219, 0.3);">
    <div style="color: white; font-size: 22px; font-weight: bold; margin-bottom: 8px; text-align: center;">
      CSS
    </div>
    <div style="color: rgba(255, 255, 255, 0.9); font-size: 14px; text-align: center; margin-bottom: 5px;">
      스타일 (Presentation)
    </div>
    <div style="color: rgba(255, 255, 255, 0.85); font-size: 13px; text-align: center;">
      웹 페이지의 디자인
    </div>
  </div>

  <!-- JavaScript -->
  <div style="padding: 20px; background: linear-gradient(135deg, #f39c12 0%, #e67e22 100%); border-radius: 8px; box-shadow: 0 4px 6px rgba(243, 156, 18, 0.3);">
    <div style="color: white; font-size: 22px; font-weight: bold; margin-bottom: 8px; text-align: center;">
      JavaScript
    </div>
    <div style="color: rgba(255, 255, 255, 0.9); font-size: 14px; text-align: center; margin-bottom: 5px;">
      동작 (Behavior)
    </div>
    <div style="color: rgba(255, 255, 255, 0.85); font-size: 13px; text-align: center;">
      웹 페이지의 상호작용
    </div>
  </div>
</div>

### JavaScript의 역할

**1. 사용자 상호작용 처리**
- 버튼 클릭, 마우스 이동, 키보드 입력 등

**2. 데이터 처리**
- 폼 유효성 검증
- 데이터 저장 및 불러오기
- 계산 및 변환

**3. 네트워크 통신**
- 서버와 데이터 주고받기 (AJAX, Fetch)
- API 호출

**4. DOM 조작**
- HTML 요소 동적 생성, 수정, 삭제
- CSS 스타일 동적 변경

**5. 애니메이션**
- 시각적 효과
- 부드러운 화면 전환

### 예제: 정적 vs 동적

```html
<!-- HTML만 사용 (정적) -->
<button>클릭하세요</button>
<p>안녕하세요</p>

<!-- JavaScript 추가 (동적) -->
<button onclick="showMessage()">클릭하세요</button>
<p id="message">안녕하세요</p>

<script>
function showMessage() {
  document.getElementById('message').textContent = '버튼이 클릭되었습니다!';
}
</script>
```

## 2. JavaScript 실행 환경

### 브라우저에서 실행

JavaScript는 브라우저에 내장된 **JavaScript 엔진**에서 실행됩니다.

| 브라우저 | JavaScript 엔진 |
|----------|----------------|
| Chrome, Edge | V8 |
| Firefox | SpiderMonkey |
| Safari | JavaScriptCore (Nitro) |

### HTML에 JavaScript 연결하기

#### 방법 1: 인라인 (권장하지 않음)

```html
<button onclick="alert('클릭!')">버튼</button>
```

**단점:**
- HTML과 JavaScript가 섞여 가독성 저하
- 유지보수 어려움
- 재사용 불가능

#### 방법 2: 내부 스크립트

```html
<!DOCTYPE html>
<html>
<head>
  <title>JavaScript 예제</title>
</head>
<body>
  <h1>Hello JavaScript</h1>

  <script>
    console.log('페이지가 로드되었습니다!');
  </script>
</body>
</html>
```

**위치:**
- `<head>` 안: HTML 파싱 전 실행 (권장하지 않음)
- `</body>` 직전: HTML 파싱 후 실행 (권장)

#### 방법 3: 외부 파일 (권장)

```html
<!DOCTYPE html>
<html>
<head>
  <title>JavaScript 예제</title>
</head>
<body>
  <h1>Hello JavaScript</h1>

  <!-- 외부 JavaScript 파일 -->
  <script src="script.js"></script>
</body>
</html>
```

```javascript
// script.js
console.log('외부 파일에서 실행됩니다!');
```

**장점:**
- 코드 분리로 가독성 향상
- 재사용 가능
- 캐싱으로 성능 개선

### defer와 async 속성

```html
<!-- defer: HTML 파싱 완료 후 순서대로 실행 (권장) -->
<script src="script.js" defer></script>

<!-- async: 다운로드 완료 즉시 실행 (순서 보장 안 됨) -->
<script src="script.js" async></script>
```

**비교:**

<div style="max-width: 700px; margin: 30px auto; padding: 25px; background: #f8f9fa; border-radius: 10px;">
  <!-- 일반 script -->
  <div style="margin-bottom: 30px;">
    <div style="font-weight: bold; margin-bottom: 10px; color: #333; font-size: 16px;">
      &lt;script src="..."&gt;
    </div>
    <div style="position: relative; height: 60px; background: white; border-radius: 6px; padding: 5px; box-shadow: 0 2px 4px rgba(0,0,0,0.1);">
      <div style="position: absolute; top: 8px; left: 5px; width: 35%; height: 20px; background: #3498db; border-radius: 4px; display: flex; align-items: center; justify-content: center; color: white; font-size: 12px; font-weight: bold;">HTML 파싱</div>
      <div style="position: absolute; top: 8px; left: 36%; width: 25%; height: 20px; background: #e74c3c; border-radius: 4px; display: flex; align-items: center; justify-content: center; color: white; font-size: 12px; font-weight: bold;">⏸ 중단</div>
      <div style="position: absolute; top: 8px; left: 62%; width: 18%; height: 20px; background: #f39c12; border-radius: 4px; display: flex; align-items: center; justify-content: center; color: white; font-size: 11px; font-weight: bold;">다운로드</div>
      <div style="position: absolute; top: 8px; left: 81%; width: 18%; height: 20px; background: #9b59b6; border-radius: 4px; display: flex; align-items: center; justify-content: center; color: white; font-size: 12px; font-weight: bold;">실행</div>
      <div style="position: absolute; top: 33px; left: 5px; right: 5px; font-size: 13px; color: #666; text-align: center;">
        HTML 파싱이 중단되고 스크립트 다운로드 및 실행
      </div>
    </div>
  </div>

  <!-- defer -->
  <div style="margin-bottom: 30px;">
    <div style="font-weight: bold; margin-bottom: 10px; color: #333; font-size: 16px;">
      &lt;script src="..." defer&gt;
    </div>
    <div style="position: relative; height: 60px; background: white; border-radius: 6px; padding: 5px; box-shadow: 0 2px 4px rgba(0,0,0,0.1);">
      <div style="position: absolute; top: 8px; left: 5px; width: 75%; height: 20px; background: #3498db; border-radius: 4px; display: flex; align-items: center; justify-content: center; color: white; font-size: 12px; font-weight: bold;">HTML 파싱 계속</div>
      <div style="position: absolute; top: 8px; left: 5px; width: 25%; height: 20px; background: #f39c12; border-radius: 4px; display: flex; align-items: center; justify-content: center; color: white; font-size: 11px; font-weight: bold; opacity: 0.9;">다운로드</div>
      <div style="position: absolute; top: 8px; left: 76%; width: 23%; height: 20px; background: #2ecc71; border-radius: 4px; display: flex; align-items: center; justify-content: center; color: white; font-size: 12px; font-weight: bold;">✓ 실행</div>
      <div style="position: absolute; top: 33px; left: 5px; right: 5px; font-size: 13px; color: #666; text-align: center;">
        백그라운드 다운로드, HTML 파싱 완료 후 실행
      </div>
    </div>
  </div>

  <!-- async -->
  <div>
    <div style="font-weight: bold; margin-bottom: 10px; color: #333; font-size: 16px;">
      &lt;script src="..." async&gt;
    </div>
    <div style="position: relative; height: 60px; background: white; border-radius: 6px; padding: 5px; box-shadow: 0 2px 4px rgba(0,0,0,0.1);">
      <div style="position: absolute; top: 8px; left: 5px; width: 98%; height: 20px; background: #3498db; border-radius: 4px; display: flex; align-items: center; justify-content: center; color: white; font-size: 12px; font-weight: bold;">HTML 파싱 계속</div>
      <div style="position: absolute; top: 8px; left: 5px; width: 20%; height: 20px; background: #f39c12; border-radius: 4px; display: flex; align-items: center; justify-content: center; color: white; font-size: 11px; font-weight: bold; opacity: 0.9;">다운로드</div>
      <div style="position: absolute; top: 8px; left: 20.5%; width: 15%; height: 20px; background: #9b59b6; border-radius: 4px; display: flex; align-items: center; justify-content: center; color: white; font-size: 12px; font-weight: bold;">⚡실행</div>
      <div style="position: absolute; top: 33px; left: 5px; right: 5px; font-size: 13px; color: #666; text-align: center;">
        다운로드 완료 즉시 실행 (순서 보장 안 됨)
      </div>
    </div>
  </div>
</div>

## 3. 첫 번째 JavaScript 코드

### alert() - 알림창

```javascript
alert("안녕하세요!");
```

**특징:**
- 브라우저 알림창 표시
- 사용자가 확인할 때까지 코드 실행 중단
- 문자열을 인자로 받음

**활용 예시:**

```javascript
// 로그인 실패 알림
alert("아이디 또는 비밀번호가 올바르지 않습니다.");

// 회원가입 완료
alert("회원가입이 완료되었습니다!");

// 폼 유효성 검증
if (password.length < 8) {
  alert("비밀번호는 8자 이상이어야 합니다.");
}
```

> **Warning**: `alert()`는 사용자 경험을 방해할 수 있어 현대 웹에서는 모달이나 토스트 메시지를 더 선호합니다.
{: .prompt-warning }

### console.log() - 콘솔 출력

```javascript
console.log("아래에 나와요");
```

**특징:**
- 개발자 도구 콘솔에 출력
- 일반 사용자에게는 보이지 않음
- 디버깅 목적으로 사용

**개발자 도구 열기:**
- Windows/Linux: `F12` 또는 `Ctrl + Shift + I`
- Mac: `Cmd + Option + I`
- 우클릭 → 검사

### console 메서드들

```javascript
// 일반 로그
console.log("일반 메시지");

// 정보 로그
console.info("정보 메시지");

// 경고 로그
console.warn("경고 메시지");

// 에러 로그
console.error("에러 메시지");

// 표 형태로 출력
console.table([
  { name: "Alice", age: 25 },
  { name: "Bob", age: 30 }
]);

// 그룹화
console.group("사용자 정보");
console.log("이름: Alice");
console.log("나이: 25");
console.groupEnd();

// 시간 측정
console.time("작업");
// 어떤 작업...
console.timeEnd("작업");

// 객체 상세 출력
console.dir(document.body);
```

### 디버깅 활용

```javascript
// 변수 값 확인
let username = "Alice";
console.log("사용자명:", username);

// 여러 값 동시 출력
let age = 25;
let city = "Seoul";
console.log("이름:", username, "나이:", age, "도시:", city);

// 템플릿 리터럴 사용
console.log(`${username}님은 ${age}살이고 ${city}에 살고 있습니다.`);

// 조건부 로깅
if (age < 18) {
  console.warn("미성년자입니다");
}

// 에러 추적
try {
  // 에러가 발생할 수 있는 코드
  throw new Error("의도적인 에러");
} catch (error) {
  console.error("에러 발생:", error.message);
}
```

## 4. JavaScript 기본 문법

### 문(Statement)과 세미콜론

```javascript
// 문(Statement): 실행 가능한 최소 단위
console.log("Hello");

// 세미콜론은 문의 끝을 나타냄 (생략 가능하지만 권장)
alert("World");
```

### 주석(Comment)

```javascript
// 한 줄 주석
console.log("코드 실행"); // 줄 끝 주석도 가능

/*
  여러 줄 주석
  이렇게 작성할 수 있습니다
*/
console.log("주석은 실행되지 않습니다");
```

### 대소문자 구분

```javascript
// JavaScript는 대소문자를 구분합니다
let name = "Alice";
let Name = "Bob";     // 다른 변수
let NAME = "Charlie"; // 또 다른 변수

console.log(name); // "Alice"
console.log(Name); // "Bob"

// 함수도 마찬가지
console.log("소문자");
Console.log("에러 발생!"); // ❌ Console is not defined
```

## 5. 변수 선언

### var, let, const

```javascript
// var (옛날 방식, 사용 지양)
var oldWay = "구식";

// let (재할당 가능)
let count = 0;
count = 1;
count = 2; // ✅ 가능

// const (재할당 불가, 상수)
const PI = 3.14159;
PI = 3.14; // ❌ 에러 발생
```

### 변수 명명 규칙

```javascript
// ✅ 올바른 변수명
let userName = "Alice";
let user_name = "Alice";
let userName2 = "Alice";
let $price = 100;
let _private = "secret";

// ❌ 잘못된 변수명
let 2users = "Alice";     // 숫자로 시작 불가
let user-name = "Alice";  // 하이픈 사용 불가
let user name = "Alice";  // 공백 사용 불가
let let = "Alice";        // 예약어 사용 불가
```

### 네이밍 컨벤션

```javascript
// camelCase (JavaScript 권장)
let firstName = "Alice";
let totalPrice = 100;

// snake_case
let first_name = "Alice";

// UPPER_CASE (상수)
const MAX_SIZE = 100;
const API_KEY = "abc123";
```

## 6. 데이터 타입

### 원시 타입(Primitive)

```javascript
// 문자열 (String)
let name = "Alice";
let greeting = 'Hello';
let message = `안녕하세요, ${name}님`; // 템플릿 리터럴

// 숫자 (Number)
let age = 25;
let price = 99.99;
let negative = -10;

// 불리언 (Boolean)
let isStudent = true;
let isAdult = false;

// undefined (값이 할당되지 않음)
let notDefined;
console.log(notDefined); // undefined

// null (의도적으로 빈 값)
let emptyValue = null;

// Symbol (고유한 식별자, ES6)
let id = Symbol('id');
```

### 객체 타입(Object)

```javascript
// 객체 (Object)
let person = {
  name: "Alice",
  age: 25,
  city: "Seoul"
};

// 배열 (Array)
let colors = ["red", "green", "blue"];
let numbers = [1, 2, 3, 4, 5];

// 함수 (Function)
function sayHello() {
  console.log("Hello!");
}
```

### typeof 연산자

```javascript
console.log(typeof "Hello");     // "string"
console.log(typeof 42);          // "number"
console.log(typeof true);        // "boolean"
console.log(typeof undefined);   // "undefined"
console.log(typeof null);        // "object" (역사적 버그)
console.log(typeof {});          // "object"
console.log(typeof []);          // "object"
console.log(typeof function(){}); // "function"
```

## 7. 예약어(Reserved Words)

JavaScript에서 이미 정의된 특별한 의미를 가진 단어들입니다.

```javascript
// 예약어 목록 (일부)
break, case, catch, class, const, continue,
debugger, default, delete, do, else, export,
extends, false, finally, for, function, if,
import, in, instanceof, let, new, null, return,
super, switch, this, throw, true, try, typeof,
var, void, while, with, yield
```

**사용 불가:**

```javascript
// ❌ 예약어를 변수명으로 사용할 수 없습니다

// let if = 5;           (예약어 사용 불가)
// let class = "...";    (예약어 사용 불가)
// let return = true;    (예약어 사용 불가)

// ✅ 대신 이렇게 사용하세요
let condition = 5;
let className = "JavaScript";
let returnValue = true;
```

## 8. 실전 예제

### 간단한 계산기

```html
<!DOCTYPE html>
<html>
<head>
  <title>계산기</title>
</head>
<body>
  <h1>간단한 계산기</h1>
  <input type="number" id="num1" placeholder="첫 번째 숫자">
  <input type="number" id="num2" placeholder="두 번째 숫자">
  <button onclick="calculate()">계산</button>
  <p id="result"></p>

  <script>
    function calculate() {
      // 입력값 가져오기
      let num1 = document.getElementById('num1').value;
      let num2 = document.getElementById('num2').value;

      // 문자열을 숫자로 변환
      num1 = Number(num1);
      num2 = Number(num2);

      // 계산
      let sum = num1 + num2;

      // 결과 표시
      document.getElementById('result').textContent =
        `결과: ${sum}`;

      // 콘솔에도 출력
      console.log(`${num1} + ${num2} = ${sum}`);
    }
  </script>
</body>
</html>
```

### 로그인 유효성 검증

```html
<!DOCTYPE html>
<html>
<head>
  <title>로그인</title>
</head>
<body>
  <h1>로그인</h1>
  <input type="text" id="username" placeholder="아이디">
  <input type="password" id="password" placeholder="비밀번호">
  <button onclick="login()">로그인</button>

  <script>
    function login() {
      let username = document.getElementById('username').value;
      let password = document.getElementById('password').value;

      // 유효성 검증
      if (username === "") {
        alert("아이디를 입력해주세요.");
        return;
      }

      if (password === "") {
        alert("비밀번호를 입력해주세요.");
        return;
      }

      if (password.length < 8) {
        alert("비밀번호는 8자 이상이어야 합니다.");
        return;
      }

      // 로그인 성공 (실제로는 서버 통신 필요)
      console.log("로그인 시도:");
      console.log("아이디:", username);
      console.log("비밀번호:", "*".repeat(password.length));

      alert("로그인 성공!");
    }
  </script>
</body>
</html>
```

### 버튼 클릭 카운터

```html
<!DOCTYPE html>
<html>
<head>
  <title>클릭 카운터</title>
</head>
<body>
  <h1>버튼을 클릭하세요</h1>
  <button onclick="incrementCount()">클릭!</button>
  <p>클릭 횟수: <span id="count">0</span></p>

  <script>
    let clickCount = 0;

    function incrementCount() {
      clickCount = clickCount + 1;
      // 또는: clickCount++;

      // 화면 업데이트
      document.getElementById('count').textContent = clickCount;

      // 콘솔 로그
      console.log(`버튼이 ${clickCount}번 클릭되었습니다.`);

      // 10번 클릭 시 축하 메시지
      if (clickCount === 10) {
        alert("축하합니다! 10번 클릭하셨습니다!");
      }
    }
  </script>
</body>
</html>
```

## 9. 디버깅 팁

### 브레이크포인트

```javascript
// debugger 키워드로 실행 일시 중지
function calculate(a, b) {
  debugger; // 여기서 멈춤
  let result = a + b;
  return result;
}
```

### 에러 메시지 읽기

```javascript
// 에러 예시
console.log(undefinedVariable);
// ❌ ReferenceError: undefinedVariable is not defined

// 해결: 변수를 먼저 선언
let undefinedVariable = "now defined";
console.log(undefinedVariable); // ✅ "now defined"
```

### 단계별 디버깅

```javascript
function processData(data) {
  console.log("1. 함수 시작, data:", data);

  let processed = data * 2;
  console.log("2. 처리 완료, processed:", processed);

  let final = processed + 10;
  console.log("3. 최종 결과, final:", final);

  return final;
}

processData(5);
// 콘솔 출력:
// 1. 함수 시작, data: 5
// 2. 처리 완료, processed: 10
// 3. 최종 결과, final: 20
```

## 10. Best Practices

### ✅ 좋은 예

```javascript
// 명확한 변수명
let userName = "Alice";
let totalPrice = 100;

// const 우선 사용 (변하지 않는 값)
const TAX_RATE = 0.1;

// 의미 있는 함수명
function calculateTotalPrice(price, quantity) {
  return price * quantity;
}

// 주석으로 의도 설명
// 사용자가 로그인했는지 확인
if (isLoggedIn) {
  showDashboard();
}

// 콘솔 로그로 디버깅
console.log("현재 사용자:", currentUser);
```

### ❌ 나쁜 예

```javascript
// 의미 없는 변수명
let a = "Alice";
let x = 100;

// var 사용 (let/const 사용 권장)
var oldStyle = "bad";

// 마법의 숫자 (상수로 정의해야 함)
let price = amount * 1.1; // 1.1이 무엇인지 불명확

// 불필요한 alert 남발
alert("함수 시작");
alert("처리 중...");
alert("완료");

// 주석 없는 복잡한 로직
if (a && b || c && !d) {
  // 무슨 조건인지 알기 어려움
}
```

## 정리

### JavaScript 핵심 개념

1. **역할**: 웹 페이지를 동적이고 상호작용 가능하게 만듦
2. **실행 환경**: 브라우저의 JavaScript 엔진
3. **연결 방법**:
   - 인라인 (비권장)
   - 내부 스크립트
   - 외부 파일 (권장)

### 기본 출력 방법

```javascript
// 사용자에게 보이는 알림
alert("메시지");

// 개발자 도구 콘솔에 출력
console.log("디버깅 정보");
console.warn("경고");
console.error("에러");
```

### 변수 선언

```javascript
// 재할당 가능
let variable = "value";

// 상수 (재할당 불가)
const CONSTANT = "value";
```

### 데이터 타입

| 타입 | 예시 |
|------|------|
| String | `"Hello"`, `'World'` |
| Number | `42`, `3.14` |
| Boolean | `true`, `false` |
| undefined | 값 없음 |
| null | 의도적 빈 값 |
| Object | `{}`, `[]`, `function(){}` |

### 기본 문법

```javascript
// 문(Statement)
console.log("Hello");

// 주석
// 한 줄 주석
/* 여러 줄 주석 */

// 대소문자 구분
let name = "Alice";
let Name = "Bob"; // 다른 변수
```

## 다음 단계

JavaScript 기초를 익혔다면 다음 주제를 학습하세요:

1. **연산자**: 산술, 비교, 논리 연산
2. **제어문**: if, switch, 반복문
3. **함수**: 함수 선언, 표현식, 화살표 함수
4. **배열**: 배열 메서드, 반복
5. **객체**: 객체 생성, 메서드, this
6. **DOM 조작**: 요소 선택, 이벤트 처리
7. **비동기**: Promise, async/await

## 참고 자료

- [MDN - JavaScript](https://developer.mozilla.org/ko/docs/Web/JavaScript)
- [MDN - JavaScript 첫걸음](https://developer.mozilla.org/ko/docs/Learn/JavaScript/First_steps)
- [JavaScript.info](https://ko.javascript.info/)
- [Eloquent JavaScript](https://eloquentjavascript.net/)
