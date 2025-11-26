---
title: JavaScript 문자열 연결과 타입 변환 완벽 가이드
description: JavaScript에서 문자열을 연결하는 다양한 방법과 문자열-숫자 간 타입 변환을 상세히 알아봅니다. 템플릿 리터럴, 타입 강제 변환, 문자열 메서드 활용법을 마스터합니다.
author: changsu
date: 2020-06-21 13:48:57 +0900
categories: [Programming, JavaScript]
tags: [javascript, string, template-literal]
---

## 문자열 연결

JavaScript에서 문자열을 결합하는 것을 **문자열 연결(String Concatenation)**이라고 합니다.

### + 연산자를 사용한 연결

가장 기본적인 방법은 `+` 연산자를 사용하는 것입니다.

```javascript
console.log("안녕" + "하세요");          // "안녕하세요"
console.log("안녕" + "하" + "세요");      // "안녕하세요"
console.log("안녕" + "하세" + "" + "요"); // "안녕하세요"
```

### 변수와 문자열 연결

```javascript
let hi = "안녕";
console.log(hi + "하세요");  // "안녕하세요"

let ha = "하세요";
console.log(hi + ha);        // "안녕하세요"
```

### 여러 줄 연결

```javascript
let message = "첫 번째 줄" +
              "두 번째 줄" +
              "세 번째 줄";
console.log(message);  // "첫 번째 줄두 번째 줄세 번째 줄"

// 공백 추가
let greeting = "안녕하세요, " +
               "반갑습니다. " +
               "좋은 하루 되세요!";
console.log(greeting);
```

## 문자열과 숫자의 연결

### 타입 강제 변환 (Type Coercion)

JavaScript에서 문자열과 숫자를 `+` 연산자로 결합하면 **숫자가 문자열로 자동 변환**됩니다.

```javascript
console.log("2" + "2");  // "22" (문자열)
console.log("2" + 2);    // "22" (문자열)
console.log(2 + "2");    // "22" (문자열)
```

**주의**: `""` (따옴표)로 감싸면 숫자가 아닌 문자열로 인식됩니다.

### 타입 확인

```javascript
console.log(typeof "2");     // "string"
console.log(typeof 2);       // "number"
console.log(typeof ("2" + 2)); // "string"
```

### 문자열 + 숫자 규칙

```javascript
// String + Number = String
let result1 = "점수: " + 95;
console.log(result1);        // "점수: 95"
console.log(typeof result1); // "string"

// Number + String = String
let result2 = 100 + "점";
console.log(result2);        // "100점"
console.log(typeof result2); // "string"

// 복합 예제
let name = "홍길동";
let age = 30;
console.log(name + "님의 나이는 " + age + "세입니다.");
// "홍길동님의 나이는 30세입니다."
```

### 연산 순서에 따른 차이

```javascript
console.log(1 + 2 + "3");    // "33" (숫자 계산 후 문자열 연결)
// 1. 1 + 2 = 3
// 2. 3 + "3" = "33"

console.log("1" + 2 + 3);    // "123" (모두 문자열로 변환)
// 1. "1" + 2 = "12"
// 2. "12" + 3 = "123"

console.log(1 + "2" + 3);    // "123"
// 1. 1 + "2" = "12"
// 2. "12" + 3 = "123"
```

## 템플릿 리터럴 (ES6)

**템플릿 리터럴(Template Literal)**은 백틱(`` ` ``)을 사용하여 문자열을 더 편리하게 작성할 수 있습니다.

### 기본 사용법

```javascript
// + 연산자 사용 (기존 방식)
let name = "홍길동";
let age = 30;
let oldWay = "안녕하세요, 제 이름은 " + name + "이고, 나이는 " + age + "세입니다.";

// 템플릿 리터럴 사용 (권장)
let newWay = `안녕하세요, 제 이름은 ${name}이고, 나이는 ${age}세입니다.`;

console.log(oldWay);
console.log(newWay);
// 둘 다: "안녕하세요, 제 이름은 홍길동이고, 나이는 30세입니다."
```

### 표현식 삽입

중괄호 `${}` 안에 JavaScript 표현식을 넣을 수 있습니다.

```javascript
let a = 10;
let b = 20;

console.log(`${a} + ${b} = ${a + b}`);  // "10 + 20 = 30"
console.log(`5 * 3 = ${5 * 3}`);        // "5 * 3 = 15"

// 함수 호출
function getGreeting() {
  return "안녕하세요";
}
console.log(`인사말: ${getGreeting()}`);  // "인사말: 안녕하세요"

// 삼항 연산자
let score = 85;
console.log(`성적: ${score >= 90 ? '우수' : '보통'}`);  // "성적: 보통"
```

### 여러 줄 문자열

```javascript
// + 연산자 사용 (복잡함)
let oldMultiline = "첫 번째 줄\n" +
                   "두 번째 줄\n" +
                   "세 번째 줄";

// 템플릿 리터럴 사용 (간편함)
let newMultiline = `첫 번째 줄
두 번째 줄
세 번째 줄`;

console.log(newMultiline);
```

### HTML 템플릿 생성

```javascript
let product = {
  name: "노트북",
  price: 1500000,
  stock: 5
};

let html = `
  <div class="product">
    <h2>${product.name}</h2>
    <p>가격: ${product.price.toLocaleString()}원</p>
    <p>재고: ${product.stock}개</p>
    <p>상태: ${product.stock > 0 ? '구매 가능' : '품절'}</p>
  </div>
`;

console.log(html);
```

## 명시적 타입 변환

### 문자열로 변환

```javascript
// String() 함수
let num = 123;
let str1 = String(num);
console.log(str1);        // "123"
console.log(typeof str1); // "string"

// toString() 메서드
let str2 = num.toString();
console.log(str2);        // "123"
console.log(typeof str2); // "string"

// 빈 문자열 연결
let str3 = num + "";
console.log(str3);        // "123"
console.log(typeof str3); // "string"
```

### 숫자로 변환

```javascript
// Number() 함수
let str = "123";
let num1 = Number(str);
console.log(num1);        // 123
console.log(typeof num1); // "number"

// parseInt() - 정수 변환
let num2 = parseInt("123");
console.log(num2);        // 123

let num3 = parseInt("123.45");
console.log(num3);        // 123 (소수점 버림)

// parseFloat() - 실수 변환
let num4 = parseFloat("123.45");
console.log(num4);        // 123.45

// 단항 + 연산자
let num5 = +"123";
console.log(num5);        // 123
console.log(typeof num5); // "number"
```

### 변환 실패 처리

```javascript
console.log(Number("abc"));      // NaN (Not a Number)
console.log(parseInt("abc"));    // NaN
console.log(Number("123abc"));   // NaN
console.log(parseInt("123abc")); // 123 (숫자 부분만 파싱)

// NaN 체크
let result = Number("abc");
if (isNaN(result)) {
  console.log("숫자로 변환할 수 없습니다.");
}
```

## 문자열 메서드

### 기본 메서드

```javascript
let str = "Hello World";

// 길이
console.log(str.length);  // 11

// 대소문자 변환
console.log(str.toUpperCase());  // "HELLO WORLD"
console.log(str.toLowerCase());  // "hello world"

// 특정 문자 접근
console.log(str.charAt(0));      // "H"
console.log(str[0]);             // "H" (ES5+)

// 부분 문자열 추출
console.log(str.substring(0, 5));  // "Hello"
console.log(str.slice(0, 5));      // "Hello"
console.log(str.slice(-5));        // "World" (뒤에서 5글자)
```

### 검색 메서드

```javascript
let text = "JavaScript는 재미있습니다. JavaScript를 배워봅시다.";

// 포함 여부
console.log(text.includes("재미"));     // true
console.log(text.includes("어려움"));   // false

// 시작/끝 확인
console.log(text.startsWith("Java"));   // true
console.log(text.endsWith("봅시다."));  // true

// 인덱스 찾기
console.log(text.indexOf("JavaScript"));      // 0 (첫 번째 위치)
console.log(text.lastIndexOf("JavaScript"));  // 21 (마지막 위치)
console.log(text.indexOf("Python"));          // -1 (없음)
```

### 변환 메서드

```javascript
let message = "  Hello World  ";

// 공백 제거
console.log(message.trim());       // "Hello World"
console.log(message.trimStart());  // "Hello World  "
console.log(message.trimEnd());    // "  Hello World"

// 문자 치환
let str = "Hello World";
console.log(str.replace("World", "JavaScript"));  // "Hello JavaScript"
console.log(str.replaceAll("o", "0"));            // "Hell0 W0rld" (ES2021)

// 분할
let csv = "사과,바나나,오렌지";
console.log(csv.split(","));  // ["사과", "바나나", "오렌지"]

// 반복
console.log("Ha".repeat(3));  // "HaHaHa"
```

### 패딩 메서드 (ES2017)

```javascript
// 앞쪽 패딩
console.log("5".padStart(3, "0"));     // "005"
console.log("123".padStart(5, "0"));   // "00123"

// 뒤쪽 패딩
console.log("5".padEnd(3, "0"));       // "500"
console.log("12".padEnd(5, "0"));      // "12000"

// 시간 표시
function formatTime(hours, minutes) {
  return `${String(hours).padStart(2, '0')}:${String(minutes).padStart(2, '0')}`;
}

console.log(formatTime(9, 5));   // "09:05"
console.log(formatTime(14, 30)); // "14:30"
```

## 실전 예제

### 1. 이름 포맷팅

```javascript
function formatName(firstName, lastName) {
  return `${lastName} ${firstName}`;
}

console.log(formatName("길동", "홍"));  // "홍 길동"

function formatFullName(obj) {
  return `${obj.lastName}${obj.firstName} (${obj.age}세)`;
}

console.log(formatFullName({ firstName: "길동", lastName: "홍", age: 30 }));
// "홍길동 (30세)"
```

### 2. URL 생성

```javascript
function createUrl(baseUrl, params) {
  const queryString = Object.entries(params)
    .map(([key, value]) => `${key}=${value}`)
    .join('&');

  return `${baseUrl}?${queryString}`;
}

const url = createUrl('https://example.com/search', {
  q: 'javascript',
  page: 1,
  limit: 10
});

console.log(url);
// "https://example.com/search?q=javascript&page=1&limit=10"
```

### 3. 메시지 템플릿

```javascript
function createNotification(user, action, item) {
  return `${user}님이 ${item}을(를) ${action}했습니다.`;
}

console.log(createNotification("홍길동", "구매", "노트북"));
// "홍길동님이 노트북을(를) 구매했습니다."

// 템플릿 리터럴 활용
const notificationTemplate = (user, action, item) =>
  `🔔 알림: ${user}님이 ${item}을(를) ${action}했습니다.`;

console.log(notificationTemplate("김철수", "좋아요", "게시글"));
```

### 4. 금액 포맷팅

```javascript
function formatCurrency(amount) {
  return `${amount.toLocaleString('ko-KR')}원`;
}

console.log(formatCurrency(1234567));  // "1,234,567원"

function formatPrice(price, currency = '₩') {
  return `${currency}${price.toLocaleString()}`;
}

console.log(formatPrice(50000));      // "₩50,000"
console.log(formatPrice(1500, '$'));  // "$1,500"
```

### 5. 파일 경로 생성

```javascript
function createPath(...segments) {
  return segments.join('/');
}

console.log(createPath('home', 'user', 'documents', 'file.txt'));
// "home/user/documents/file.txt"

// 템플릿 리터럴 활용
const getFilePath = (dir, filename) => `${dir}/${filename}`;

console.log(getFilePath('/uploads', 'image.jpg'));
// "/uploads/image.jpg"
```

### 6. 카드 번호 마스킹

```javascript
function maskCardNumber(cardNumber) {
  const cleaned = cardNumber.replace(/\s/g, '');
  const last4 = cleaned.slice(-4);
  const masked = '*'.repeat(cleaned.length - 4);
  return masked + last4;
}

console.log(maskCardNumber("1234 5678 9012 3456"));
// "************3456"

// 정규식 활용
function formatCardNumber(cardNumber) {
  return cardNumber.replace(/\d(?=\d{4})/g, '*');
}

console.log(formatCardNumber("1234567890123456"));
// "************3456"
```

## 주의사항과 모범 사례

### 1. 문자열 vs 숫자 연산

```javascript
// ❌ 예상치 못한 결과
console.log("10" + 5);     // "105" (문자열)
console.log("10" - 5);     // 5 (숫자)
console.log("10" * 2);     // 20 (숫자)
console.log("10" / 2);     // 5 (숫자)

// ✅ 명확한 타입 변환
console.log(Number("10") + 5);  // 15
console.log(parseInt("10") + 5); // 15
```

### 2. 템플릿 리터럴 사용 권장

```javascript
// ❌ + 연산자 (복잡함)
let name = "홍길동";
let age = 30;
let job = "개발자";
let msg1 = "안녕하세요. 제 이름은 " + name + "이고, " + age + "세 " + job + "입니다.";

// ✅ 템플릿 리터럴 (간결함)
let msg2 = `안녕하세요. 제 이름은 ${name}이고, ${age}세 ${job}입니다.`;
```

### 3. 빈 문자열 체크

```javascript
function greet(name) {
  // ❌ 나쁜 예
  if (name == "") {
    return "손님";
  }

  // ✅ 좋은 예
  if (!name || name.trim() === "") {
    return "손님";
  }

  return name;
}

console.log(greet(""));        // "손님"
console.log(greet("   "));     // "손님"
console.log(greet("홍길동"));  // "홍길동"
```

### 4. 불변성(Immutability)

```javascript
// 문자열은 불변(immutable)
let str = "Hello";
str[0] = "h";  // 작동하지 않음
console.log(str);  // "Hello" (변경되지 않음)

// 새로운 문자열 생성
let newStr = "h" + str.slice(1);
console.log(newStr);  // "hello"
```

### 5. 성능 고려

```javascript
// ❌ 나쁜 예: 반복문에서 + 연산자
let result = "";
for (let i = 0; i < 1000; i++) {
  result += i;  // 매번 새 문자열 생성
}

// ✅ 좋은 예: 배열과 join 사용
let arr = [];
for (let i = 0; i < 1000; i++) {
  arr.push(i);
}
let result2 = arr.join("");  // 한 번만 생성
```

## 다른 언어와의 비교

### JavaScript의 특징

JavaScript는 타입이 유연하여 문자열과 숫자를 자동으로 변환합니다.

```javascript
// JavaScript: 자동 변환
console.log("5" + 3);  // "53" (문자열)

// Python에서는
// "5" + 3  → TypeError 발생

// Java에서는
// "5" + 3  → "53" (문자열)
```

### + 연산자의 동작

```javascript
// JavaScript는 왼쪽에서 오른쪽으로 평가
console.log(1 + 2 + "3");    // "33"
console.log("1" + 2 + 3);    // "123"
console.log(1 + "2" + 3);    // "123"

// 명시적 변환 권장
console.log(String(1 + 2) + "3");  // "33"
```

## 마치며

JavaScript에서 문자열을 다루는 방법을 이해하면 다양한 텍스트 처리 작업을 수행할 수 있습니다:

1. **문자열 연결**: `+` 연산자와 템플릿 리터럴
2. **타입 변환**: 문자열-숫자 간 자동/명시적 변환
3. **템플릿 리터럴**: 백틱과 `${}` 사용 (ES6+ 권장)
4. **문자열 메서드**: 검색, 변환, 포맷팅
5. **모범 사례**: 타입 안정성, 성능, 가독성

현대 JavaScript에서는 템플릿 리터럴 사용을 권장하며, 타입 변환 시 명시적 방법을 사용하여 예상치 못한 버그를 방지하세요.

## 참고 자료

- [MDN - String](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/String)
- [MDN - Template literals](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Template_literals)
- [MDN - Type coercion](https://developer.mozilla.org/ko/docs/Glossary/Type_coercion)
- [MDN - String methods](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/String#%EC%9D%B8%EC%8A%A4%ED%84%B4%EC%8A%A4_%EB%A9%94%EC%84%9C%EB%93%9C)
- [JavaScript.info - Strings](https://javascript.info/string)
