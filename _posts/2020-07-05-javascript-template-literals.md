---
title: JavaScript Template Literals 완벽 가이드 - 문자열 처리의 혁명
description: ES6 템플릿 리터럴의 강력한 기능과 실무 활용법을 다룹니다. Tagged Templates, String 메서드, 그리고 다양한 활용 예제를 통해 현대적인 JavaScript 문자열 처리 방법을 알아봅니다.
author: cotes
date: 2020-07-05 18:47:12 +0900
categories: [JavaScript, ES6]
tags: [javascript, es6, template-literals, string, tagged-templates]
---

## Template Literals란?

ES6에서 도입된 **Template Literals(템플릿 리터럴)**는 백틱(`` ` ``)을 사용하여 문자열을 표현하는 새로운 문법입니다. 단순히 따옴표를 대체하는 것이 아니라, **문자열 보간, 여러 줄 문자열, Tagged Templates** 등 강력한 기능을 제공합니다.

## 기본 문법

### 1. 백틱으로 문자열 생성

```javascript
// ES5 - 따옴표
const name1 = '김개발';
const name2 = "박코딩";

// ES6 - 백틱
const name3 = `이자바`;
```

백틱만 사용한다면 일반 문자열과 동일합니다. 하지만 템플릿 리터럴의 진짜 힘은 다음 기능들에서 발휘됩니다.

### 2. 문자열 보간 (String Interpolation)

변수나 표현식을 `${}` 안에 넣어 문자열에 삽입할 수 있습니다.

```javascript
const name = '김개발';
const age = 28;

// ES5 - 문자열 연결 (가독성 나쁨)
const greeting1 = '안녕하세요. 저는 ' + name + '이고, ' + age + '살입니다.';

// ES6 - 템플릿 리터럴 (가독성 좋음)
const greeting2 = `안녕하세요. 저는 ${name}이고, ${age}살입니다.`;
```

### 3. 표현식 실행

`${}` 안에는 변수뿐만 아니라 **모든 JavaScript 표현식**을 넣을 수 있습니다.

```javascript
const a = 10;
const b = 20;

console.log(`${a} + ${b} = ${a + b}`);
// "10 + 20 = 30"

console.log(`올해는 ${new Date().getFullYear()}년입니다.`);
// "올해는 2024년입니다."

console.log(`랜덤 숫자: ${Math.floor(Math.random() * 100)}`);
// "랜덤 숫자: 42"

// 삼항 연산자도 가능
const score = 85;
console.log(`성적: ${score >= 60 ? '합격' : '불합격'}`);
// "성적: 합격"
```

### 4. 여러 줄 문자열 (Multiline Strings)

템플릿 리터럴은 **줄바꿈을 그대로 유지**합니다.

```javascript
// ES5 - 줄바꿈 처리 (번거로움)
const poem1 = '자세히\n' +
              '보아야\n' +
              '이쁘다\n' +
              '\n' +
              '내 코드..';

// ES6 - 템플릿 리터럴 (직관적)
const poem2 = `자세히
보아야
이쁘다

내 코드..`;

console.log(poem2);
// 자세히
// 보아야
// 이쁘다
//
// 내 코드..
```

## String 메서드

ES6는 템플릿 리터럴과 함께 유용한 문자열 메서드도 추가했습니다.

### 1. startsWith() - 시작 문자열 확인

```javascript
const email = 'yealee.kim87@gmail.com';

console.log(email.startsWith('yealee'));  // true
console.log(email.startsWith('kim'));     // false

// 시작 위치 지정 가능
console.log(email.startsWith('kim', 7));  // true (7번째부터 시작)
```

### 2. endsWith() - 끝 문자열 확인

```javascript
const email = 'yealee.kim87@gmail.com';

console.log(email.endsWith('.com'));      // true
console.log(email.endsWith('gmail'));     // false

// 특정 길이까지만 확인
console.log(email.endsWith('@gmail', 20)); // true (앞 20자만 확인)
```

### 3. includes() - 포함 여부 확인

```javascript
const email = 'yealee.kim87@gmail.com';

console.log(email.includes('@gmail'));    // true
console.log(email.includes('naver'));     // false

// 시작 위치 지정 가능
console.log(email.includes('kim', 10));   // false (10번째부터 검색)
```

### 4. repeat() - 문자열 반복

```javascript
const star = '⭐';
console.log(star.repeat(5));  // "⭐⭐⭐⭐⭐"

// 실용 예제: 패딩
const rating = 4;
const stars = '★'.repeat(rating) + '☆'.repeat(5 - rating);
console.log(stars);  // "★★★★☆"

// 로딩 애니메이션
console.log('Loading' + '.'.repeat(3));  // "Loading..."
```

## 실무 활용 예제

### 1. HTML 템플릿 생성

```javascript
function createUserCard(user) {
  return `
    <div class="user-card">
      <h2>${user.name}</h2>
      <p class="email">${user.email}</p>
      <p class="role">${user.role || 'Member'}</p>
      <span class="status ${user.active ? 'active' : 'inactive'}">
        ${user.active ? '활성' : '비활성'}
      </span>
    </div>
  `;
}

const user = {
  name: '김개발',
  email: 'dev@example.com',
  role: 'Developer',
  active: true
};

document.body.innerHTML = createUserCard(user);
```

### 2. API URL 생성

```javascript
function buildApiUrl(endpoint, params = {}) {
  const queryString = Object.entries(params)
    .map(([key, value]) => `${key}=${encodeURIComponent(value)}`)
    .join('&');

  return `https://api.example.com/${endpoint}${queryString ? '?' + queryString : ''}`;
}

const url = buildApiUrl('users', { page: 2, limit: 10, sort: 'name' });
console.log(url);
// "https://api.example.com/users?page=2&limit=10&sort=name"
```

### 3. 동적 클래스명 생성

```javascript
function getButtonClass(type, size, disabled) {
  return `btn btn-${type} btn-${size}${disabled ? ' btn-disabled' : ''}`;
}

console.log(getButtonClass('primary', 'lg', false));
// "btn btn-primary btn-lg"

console.log(getButtonClass('danger', 'sm', true));
// "btn btn-danger btn-sm btn-disabled"
```

### 4. SQL 쿼리 생성 (주의: 실제로는 Prepared Statement 사용)

```javascript
function buildSelectQuery(table, conditions) {
  const where = Object.entries(conditions)
    .map(([key, value]) => `${key} = '${value}'`)
    .join(' AND ');

  return `SELECT * FROM ${table}${where ? ` WHERE ${where}` : ''}`;
}

console.log(buildSelectQuery('users', { status: 'active', role: 'admin' }));
// "SELECT * FROM users WHERE status = 'active' AND role = 'admin'"
```

> **보안 경고**: 실제 SQL 쿼리는 반드시 Prepared Statement를 사용하여 SQL Injection을 방지해야 합니다!
{: .prompt-warning }

### 5. 로그 메시지 포맷팅

```javascript
function log(level, message, data = {}) {
  const timestamp = new Date().toISOString();
  const dataStr = Object.keys(data).length > 0
    ? `\n${JSON.stringify(data, null, 2)}`
    : '';

  return `[${timestamp}] ${level.toUpperCase()}: ${message}${dataStr}`;
}

console.log(log('info', 'User logged in', { userId: 123, ip: '192.168.1.1' }));
// [2024-01-15T10:30:00.000Z] INFO: User logged in
// {
//   "userId": 123,
//   "ip": "192.168.1.1"
// }
```

## Tagged Templates (고급)

Tagged Templates는 템플릿 리터럴을 **함수로 파싱**할 수 있게 해주는 강력한 기능입니다.

### 기본 개념

```javascript
function myTag(strings, ...values) {
  console.log('문자열 조각들:', strings);
  console.log('값들:', values);

  return strings.reduce((result, str, i) => {
    return result + str + (values[i] || '');
  }, '');
}

const name = '김개발';
const age = 28;

const result = myTag`제 이름은 ${name}이고, ${age}살입니다.`;
console.log(result);

// 출력:
// 문자열 조각들: ['제 이름은 ', '이고, ', '살입니다.']
// 값들: ['김개발', 28]
// 제 이름은 김개발이고, 28살입니다.
```

### 실용 예제 1: HTML 이스케이프

```javascript
function escapeHTML(strings, ...values) {
  const escape = (str) => String(str)
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#039;');

  return strings.reduce((result, str, i) => {
    const value = values[i] !== undefined ? escape(values[i]) : '';
    return result + str + value;
  }, '');
}

const userInput = '<script>alert("XSS")</script>';
const safeHTML = escapeHTML`<div>${userInput}</div>`;

console.log(safeHTML);
// <div>&lt;script&gt;alert(&quot;XSS&quot;)&lt;/script&gt;</div>
```

### 실용 예제 2: 다국어 처리 (i18n)

```javascript
const translations = {
  ko: {
    greeting: '안녕하세요, {0}님!',
    welcome: '{0}님, {1}에 오신 것을 환영합니다!'
  },
  en: {
    greeting: 'Hello, {0}!',
    welcome: 'Welcome to {1}, {0}!'
  }
};

function i18n(lang) {
  return function(strings, ...values) {
    const key = strings.join('{0}').trim();
    let translation = translations[lang][key] || key;

    values.forEach((value, i) => {
      translation = translation.replace(`{${i}}`, value);
    });

    return translation;
  };
}

const t = i18n('ko');
console.log(t`greeting ${'김개발'}`);
// "안녕하세요, 김개발님!"
```

### 실용 예제 3: 스타일링 (styled-components 방식)

```javascript
function css(strings, ...values) {
  return strings.reduce((result, str, i) => {
    const value = typeof values[i] === 'function'
      ? values[i]()
      : (values[i] || '');
    return result + str + value;
  }, '');
}

const primaryColor = '#007bff';
const getSize = () => '16px';

const buttonStyle = css`
  background-color: ${primaryColor};
  font-size: ${getSize};
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
`;

console.log(buttonStyle);
// background-color: #007bff;
// font-size: 16px;
// padding: 10px 20px;
// border: none;
// border-radius: 4px;
```

## String 메서드 실무 활용

### 1. 이메일 유효성 검사

```javascript
function isValidEmail(email) {
  return email.includes('@') &&
         email.includes('.') &&
         email.indexOf('@') > 0 &&
         email.lastIndexOf('.') > email.indexOf('@');
}

console.log(isValidEmail('user@example.com'));  // true
console.log(isValidEmail('@example.com'));      // false
console.log(isValidEmail('user@example'));      // false
```

### 2. 파일 확장자 검사

```javascript
function isImageFile(filename) {
  const imageExtensions = ['.jpg', '.jpeg', '.png', '.gif', '.webp'];
  return imageExtensions.some(ext => filename.toLowerCase().endsWith(ext));
}

console.log(isImageFile('photo.jpg'));      // true
console.log(isImageFile('document.pdf'));   // false
console.log(isImageFile('IMAGE.PNG'));      // true
```

### 3. URL 프로토콜 확인

```javascript
function isSecureUrl(url) {
  return url.startsWith('https://') || url.startsWith('wss://');
}

console.log(isSecureUrl('https://example.com'));  // true
console.log(isSecureUrl('http://example.com'));   // false
console.log(isSecureUrl('wss://socket.io'));      // true
```

### 4. 금지어 필터링

```javascript
function containsBadWords(text) {
  const badWords = ['스팸', '광고', '욕설'];
  return badWords.some(word => text.includes(word));
}

console.log(containsBadWords('이건 스팸입니다'));   // true
console.log(containsBadWords('정상 메시지'));       // false
```

### 5. 진행 표시줄 생성

```javascript
function createProgressBar(progress, length = 20) {
  const filled = Math.round((progress / 100) * length);
  const empty = length - filled;

  return `[${'█'.repeat(filled)}${'░'.repeat(empty)}] ${progress}%`;
}

console.log(createProgressBar(0));
// [░░░░░░░░░░░░░░░░░░░░] 0%

console.log(createProgressBar(50));
// [██████████░░░░░░░░░░] 50%

console.log(createProgressBar(100));
// [████████████████████] 100%
```

## 비교: Template Literals vs 문자열 연결

| 기능 | 문자열 연결 | Template Literals |
|------|-----------|------------------|
| **가독성** | ⭐⭐ | ⭐⭐⭐⭐⭐ |
| **여러 줄** | `\n` 필요 | 자동 지원 |
| **변수 삽입** | `+` 연산자 | `${}` 간편 |
| **표현식** | 복잡함 | 간단함 |
| **성능** | 약간 빠름 | 약간 느림 (무시할 수준) |
| **Tagged Templates** | 불가능 | 가능 |

## Best Practices

### ✅ 좋은 사용 예

```javascript
// 1. 여러 변수를 포함한 문자열
const message = `안녕하세요, ${name}님. ${unreadCount}개의 안 읽은 메시지가 있습니다.`;

// 2. 조건부 표현
const status = `주문 상태: ${isPaid ? '결제 완료' : '결제 대기'}`;

// 3. HTML 생성
const html = `<article>${title}</article>`;

// 4. 여러 줄 문자열
const query = `
  SELECT *
  FROM users
  WHERE active = true
  ORDER BY created_at DESC
`;
```

### ❌ 피해야 할 사용 예

```javascript
// 1. 단순 문자열은 일반 따옴표 사용
const name = `김개발`;  // ❌
const name = '김개발';   // ✅

// 2. 불필요한 표현식
const greeting = `안녕하세요${'!'}`;  // ❌
const greeting = '안녕하세요!';       // ✅

// 3. XSS 취약점 주의 (사용자 입력을 그대로 삽입)
const html = `<div>${userInput}</div>`;  // ❌ XSS 위험
const html = `<div>${escapeHTML(userInput)}</div>`;  // ✅
```

## 성능 고려사항

```javascript
// 벤치마크 예제
const iterations = 1000000;
const name = 'Test';
const value = 123;

// 1. 문자열 연결
console.time('String Concatenation');
for (let i = 0; i < iterations; i++) {
  const str = 'Name: ' + name + ', Value: ' + value;
}
console.timeEnd('String Concatenation'); // ~15ms

// 2. Template Literals
console.time('Template Literals');
for (let i = 0; i < iterations; i++) {
  const str = `Name: ${name}, Value: ${value}`;
}
console.timeEnd('Template Literals'); // ~20ms
```

> **결론**: Template Literals가 약간 느리지만, 실무에서는 무시할 수준입니다. **가독성과 유지보수성**이 더 중요합니다!
{: .prompt-tip }

## 핵심 정리

### Template Literals의 장점

1. **가독성**: 변수와 표현식을 직관적으로 삽입
2. **여러 줄 지원**: 줄바꿈을 자연스럽게 표현
3. **표현식 실행**: `${}` 안에서 모든 JavaScript 표현식 사용 가능
4. **Tagged Templates**: 고급 문자열 처리 가능

### String 메서드 정리

| 메서드 | 기능 | 반환값 |
|--------|------|--------|
| `startsWith(str)` | 시작 문자열 확인 | Boolean |
| `endsWith(str)` | 끝 문자열 확인 | Boolean |
| `includes(str)` | 포함 여부 확인 | Boolean |
| `repeat(n)` | n번 반복 | String |

## 참고 자료

- [MDN - Template Literals](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals)
- [MDN - String Methods](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)
- [ECMAScript 2015 Specification](https://www.ecma-international.org/ecma-262/6.0/)
- [Tagged Templates Examples](https://wesbos.com/tagged-template-literals)
