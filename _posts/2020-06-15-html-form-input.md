---
title: HTML Form 입력 요소 완벽 가이드 - Input, Textarea, Button
description: HTML input, textarea, button 요소의 다양한 타입과 속성을 학습하고, CSS로 폼 요소를 스타일링하는 방법을 알아봅니다.
author: changsu
date: 2020-06-15 12:13:39 +0900
categories: [Web, HTML]
tags: [html, form, input, textarea, button, css, form-validation, accessibility]
---

## 개요

사용자로부터 데이터를 입력받는 것은 웹 개발의 핵심입니다. 이 글에서는 HTML의 `<input>`, `<textarea>`, `<button>` 요소를 다루며, 다양한 입력 타입과 스타일링 기법을 학습합니다.

## 1. `<input>` 기본 사용법

### input 요소 특징

`<input>` 요소는 **빈 요소(void element)**로, 닫는 태그가 없습니다.

```html
<!-- ✅ 올바른 사용 -->
<input type="text">

<!-- ❌ 잘못된 사용 -->
<input type="text"></input>
```

### 주요 input type

#### type="text"

가장 기본적인 텍스트 입력 필드입니다.

```html
<input type="text" placeholder="이름을 입력하세요">
```

**사용 예시:**
- 이름, 닉네임
- 주소, 도시명
- 검색어

#### type="password"

입력한 내용을 숨겨서 표시합니다.

```html
<input type="password" placeholder="비밀번호">
```

**특징:**
- 입력 내용이 `●`나 `*`로 표시됨
- 화면에는 숨겨지지만 JavaScript로 값 접근 가능
- 보안을 위해 실제 전송은 HTTPS 사용 필수

> **Warning**: 비밀번호는 반드시 HTTPS로 전송해야 합니다.
{: .prompt-warning }

#### type="number"

숫자만 입력할 수 있는 필드입니다.

```html
<input type="number" placeholder="나이" min="0" max="150">
```

**특징:**
- 숫자만 입력 가능
- 증감 버튼(spinner) 자동 표시
- 특수문자(`-`, `.` 제외) 입력 불가

**주의사항:**
```html
<!-- ❌ 전화번호에는 부적합 (하이픈 입력 불가) -->
<input type="number" placeholder="전화번호">

<!-- ✅ 전화번호는 type="tel" 사용 -->
<input type="tel" placeholder="010-1234-5678">
```

### placeholder 속성

입력 필드에 도움말을 표시합니다.

```html
<input type="text" placeholder="아이디를 입력하세요">
```

**특징:**
- 실제 값이 아닌 힌트 텍스트
- 사용자가 입력을 시작하면 자동으로 사라짐
- 접근성을 위해 `label`과 함께 사용 권장

## 2. 다양한 Input Type

### type="email"

이메일 주소 입력 전용 필드입니다.

```html
<input type="email" placeholder="example@domain.com">
```

**특징:**
- 이메일 형식 자동 검증
- 모바일에서 `@` 키보드 자동 표시
- 제출 시 형식 검증

### type="tel"

전화번호 입력 필드입니다.

```html
<input type="tel" placeholder="010-1234-5678">
```

**특징:**
- 모바일에서 숫자 키패드 자동 표시
- 하이픈, 괄호 등 입력 가능

### type="url"

URL 입력 필드입니다.

```html
<input type="url" placeholder="https://example.com">
```

**특징:**
- URL 형식 자동 검증
- 프로토콜(`http://`, `https://`) 확인

### type="date"

날짜 선택기를 제공합니다.

```html
<input type="date" min="2020-01-01" max="2025-12-31">
```

**관련 타입:**
- `type="time"`: 시간 선택
- `type="datetime-local"`: 날짜와 시간
- `type="month"`: 월 선택
- `type="week"`: 주 선택

### type="checkbox"

체크박스를 생성합니다.

```html
<input type="checkbox" id="agree">
<label for="agree">이용약관에 동의합니다</label>
```

**특징:**
- 다중 선택 가능
- `checked` 속성으로 기본 선택 설정

### type="radio"

라디오 버튼을 생성합니다.

```html
<input type="radio" name="gender" value="male" id="male">
<label for="male">남성</label>

<input type="radio" name="gender" value="female" id="female">
<label for="female">여성</label>
```

**특징:**
- 같은 `name`을 가진 그룹 중 하나만 선택 가능
- `value` 속성으로 선택된 값 전달

### type="file"

파일 업로드 버튼을 생성합니다.

```html
<input type="file" accept="image/*">
```

**속성:**
- `accept`: 허용할 파일 타입 지정
- `multiple`: 여러 파일 선택 가능

### type="range"

슬라이더를 생성합니다.

```html
<input type="range" min="0" max="100" value="50">
```

### type="color"

색상 선택기를 제공합니다.

```html
<input type="color" value="#ff0000">
```

## 3. `<textarea>` 요소

여러 줄의 텍스트를 입력받을 때 사용합니다.

```html
<textarea placeholder="소개를 입력하세요"></textarea>
```

### input vs textarea

| 특징 | `<input>` | `<textarea>` |
|------|-----------|--------------|
| **줄 수** | 한 줄 | 여러 줄 |
| **크기 조절** | 불가능 | 가능 (기본) |
| **닫는 태그** | 없음 (빈 요소) | 있음 |
| **기본값** | `value` 속성 | 태그 사이 내용 |

### textarea 기본값 설정

```html
<!-- ❌ 잘못된 방법 (placeholder 아님!) -->
<textarea>소개:</textarea>

<!-- ✅ placeholder 사용 -->
<textarea placeholder="소개:"></textarea>

<!-- ✅ 기본값 설정 -->
<textarea>여기에 기본 텍스트가 들어갑니다.</textarea>
```

### textarea 속성

```html
<textarea
  rows="5"
  cols="30"
  maxlength="500"
  placeholder="최대 500자까지 입력 가능합니다"
></textarea>
```

**주요 속성:**
- `rows`: 표시할 줄 수
- `cols`: 표시할 열 수
- `maxlength`: 최대 글자 수

## 4. Form Validation (유효성 검증)

### required 속성

필수 입력 필드로 지정합니다.

```html
<input type="text" required placeholder="필수 입력 항목">
```

### pattern 속성

정규표현식으로 입력 형식을 제한합니다.

```html
<!-- 전화번호 형식 (010-####-####) -->
<input
  type="tel"
  pattern="[0-9]{3}-[0-9]{4}-[0-9]{4}"
  placeholder="010-1234-5678"
>
```

### min, max 속성

숫자나 날짜의 범위를 제한합니다.

```html
<input type="number" min="18" max="100" placeholder="나이 (18-100)">
<input type="date" min="2020-01-01" max="2025-12-31">
```

### minlength, maxlength 속성

텍스트 길이를 제한합니다.

```html
<input
  type="password"
  minlength="8"
  maxlength="20"
  placeholder="비밀번호 (8-20자)"
>
```

## 5. Label과 Input 연결

### 접근성을 위한 label

`<label>` 요소는 입력 필드를 설명하고 클릭 영역을 확장합니다.

```html
<!-- 방법 1: for 속성 사용 -->
<label for="username">아이디:</label>
<input type="text" id="username">

<!-- 방법 2: label로 감싸기 -->
<label>
  아이디:
  <input type="text">
</label>
```

**장점:**
- label을 클릭해도 input이 포커스됨
- 스크린 리더가 내용을 읽어줌
- 클릭 영역 확대로 UX 개선

### 실전 예제

```html
<form>
  <div class="form-group">
    <label for="email">이메일</label>
    <input
      type="email"
      id="email"
      name="email"
      required
      placeholder="example@domain.com"
    >
  </div>

  <div class="form-group">
    <label for="password">비밀번호</label>
    <input
      type="password"
      id="password"
      name="password"
      required
      minlength="8"
      placeholder="8자 이상 입력"
    >
  </div>
</form>
```

## 6. Input 요소 레이아웃

### Inline Element 특성

`input`, `textarea`, `button`은 모두 **inline 요소**처럼 동작합니다.

```html
<!-- 한 줄에 나란히 표시됨 -->
<input type="text">
<input type="password">
<button>제출</button>
```

### Block 레이아웃 만들기

#### 방법 1: div로 감싸기

```html
<div class="input-wrap">
  <input type="text" placeholder="아이디">
</div>

<div class="input-wrap">
  <input type="password" placeholder="비밀번호">
</div>
```

#### 방법 2: display 속성 변경

```css
input {
  display: block;
  margin-bottom: 10px;
}
```

### Width 통일하기

#### ❌ 비효율적인 방법

```css
input {
  width: 300px;
}

textarea {
  width: 300px;
}

/* 나중에 디자인 변경 시 모두 수정해야 함 */
```

#### ✅ 효율적인 방법

```html
<div class="form-container">
  <div class="input-wrap">
    <input type="text">
  </div>
  <div class="input-wrap">
    <textarea></textarea>
  </div>
</div>
```

```css
.form-container {
  width: 400px;
  margin: 0 auto;  /* 중앙 정렬 */
}

input,
textarea {
  width: 100%;  /* 부모 너비에 맞춤 */
}
```

## 7. Input 스타일링

### 기본 스타일 초기화

브라우저는 input에 기본 스타일을 적용합니다.

```css
input,
textarea {
  /* 브라우저 기본 스타일 제거 */
  border: 1px solid #ddd;
  border-radius: 4px;
  padding: 10px 12px;
  font-size: 14px;
  font-family: inherit;  /* 부모 폰트 상속 */
}

/* 포커스 시 아웃라인 커스터마이징 */
input:focus,
textarea:focus {
  outline: none;
  border-color: #007bff;
  box-shadow: 0 0 0 3px rgba(0, 123, 255, 0.1);
}
```

### Textarea 크기 조절 제한

```css
textarea {
  resize: none;           /* 크기 조절 불가 */
  /* resize: vertical;    /* 세로만 조절 가능 */
  /* resize: horizontal;  /* 가로만 조절 가능 */
}
```

### 입력 요소 간격

```css
input,
textarea {
  margin-bottom: 15px;
}
```

### 완성된 폼 스타일

```css
.form-container {
  max-width: 400px;
  margin: 50px auto;
  padding: 30px;
  background-color: white;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

input,
textarea {
  width: 100%;
  padding: 12px 15px;
  margin-bottom: 15px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
  transition: border-color 0.3s;
}

input:focus,
textarea:focus {
  outline: none;
  border-color: #007bff;
}

textarea {
  min-height: 100px;
  resize: vertical;
}
```

## 8. CSS 선택자 심화

### Placeholder 스타일링

```css
/* 모든 input의 placeholder */
input::placeholder {
  color: #999;
  font-style: italic;
}

/* 특정 type의 placeholder */
input[type="text"]::placeholder {
  color: #bbb;
}
```

> **Tip**: `::placeholder`는 가상 요소(pseudo-element)로, 콜론 2개(`::`)를 사용합니다.
{: .prompt-tip }

### 속성 선택자 (Attribute Selector)

```css
/* type="text"인 input만 선택 */
input[type="text"] {
  border-color: blue;
}

/* type="password"인 input만 선택 */
input[type="password"] {
  border-color: red;
}

/* required 속성이 있는 요소 */
input[required] {
  border-left: 3px solid #f00;
}

/* disabled 속성이 있는 요소 */
input[disabled] {
  background-color: #f5f5f5;
  cursor: not-allowed;
}
```

### 가상 클래스 (Pseudo-class)

```css
/* 포커스 상태 */
input:focus {
  border-color: #007bff;
}

/* 유효한 입력 */
input:valid {
  border-color: #28a745;
}

/* 유효하지 않은 입력 */
input:invalid {
  border-color: #dc3545;
}

/* 비활성화 상태 */
input:disabled {
  opacity: 0.5;
}

/* 체크된 체크박스/라디오 */
input[type="checkbox"]:checked {
  background-color: #007bff;
}
```

### 조합 선택자

```css
/* text 타입의 placeholder */
input[type="text"]::placeholder {
  color: red;
}

/* 포커스된 email 입력 */
input[type="email"]:focus {
  border-color: green;
}

/* 필수 입력 필드의 포커스 */
input[required]:focus {
  box-shadow: 0 0 5px rgba(255, 0, 0, 0.5);
}
```

## 9. Button 스타일링

### 기본 버튼 스타일

```css
button {
  padding: 10px 20px;
  font-size: 15px;
  color: white;
  background-color: #4CAF50;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.3s;
}
```

### 호버 효과

```css
button:hover {
  background-color: #45a049;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

/* 또는 opacity 사용 */
button:hover {
  opacity: 0.8;
}
```

### 버튼 상태

```css
/* 활성 상태 (클릭 시) */
button:active {
  transform: translateY(0);
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}

/* 비활성 상태 */
button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
  opacity: 0.6;
}
```

### 버튼 변형

```css
/* 기본 버튼 */
.btn-primary {
  background-color: #007bff;
}

.btn-primary:hover {
  background-color: #0056b3;
}

/* 성공 버튼 */
.btn-success {
  background-color: #28a745;
}

/* 경고 버튼 */
.btn-warning {
  background-color: #ffc107;
  color: #212529;
}

/* 위험 버튼 */
.btn-danger {
  background-color: #dc3545;
}

/* 외곽선 버튼 */
.btn-outline {
  background-color: transparent;
  border: 2px solid #007bff;
  color: #007bff;
}

.btn-outline:hover {
  background-color: #007bff;
  color: white;
}
```

## 10. Opacity 속성

`opacity`는 요소의 투명도를 조절합니다.

```css
.element {
  opacity: 1;    /* 완전 불투명 (기본값) */
  opacity: 0.8;  /* 80% 불투명 */
  opacity: 0.5;  /* 50% 불투명 */
  opacity: 0;    /* 완전 투명 */
}
```

**값 범위:**
- `0`: 완전 투명 (보이지 않음)
- `0.1` ~ `0.9`: 반투명
- `1`: 완전 불투명

**활용 예시:**
```css
/* 호버 시 흐리게 */
button:hover {
  opacity: 0.8;
}

/* 비활성 요소 */
input:disabled {
  opacity: 0.5;
}

/* 로딩 중 */
.loading {
  opacity: 0.6;
  pointer-events: none;
}
```

## 11. 실전 예제

### 로그인 폼

```html
<form class="login-form">
  <h2>로그인</h2>

  <div class="form-group">
    <label for="login-email">이메일</label>
    <input
      type="email"
      id="login-email"
      name="email"
      placeholder="example@domain.com"
      required
    >
  </div>

  <div class="form-group">
    <label for="login-password">비밀번호</label>
    <input
      type="password"
      id="login-password"
      name="password"
      placeholder="비밀번호를 입력하세요"
      required
      minlength="8"
    >
  </div>

  <div class="form-group">
    <label>
      <input type="checkbox" name="remember">
      로그인 상태 유지
    </label>
  </div>

  <button type="submit">로그인</button>
</form>
```

```css
.login-form {
  max-width: 400px;
  margin: 50px auto;
  padding: 40px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.login-form h2 {
  margin-bottom: 30px;
  text-align: center;
  color: #333;
}

.form-group {
  margin-bottom: 20px;
}

.form-group label {
  display: block;
  margin-bottom: 8px;
  font-weight: 500;
  color: #555;
}

.form-group input[type="email"],
.form-group input[type="password"] {
  width: 100%;
  padding: 12px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
}

.form-group input:focus {
  outline: none;
  border-color: #007bff;
  box-shadow: 0 0 0 3px rgba(0, 123, 255, 0.1);
}

button[type="submit"] {
  width: 100%;
  padding: 12px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  font-size: 16px;
  font-weight: 500;
  cursor: pointer;
  transition: background-color 0.3s;
}

button[type="submit"]:hover {
  background-color: #0056b3;
}
```

### 회원가입 폼

```html
<form class="signup-form">
  <h2>회원가입</h2>

  <div class="form-group">
    <label for="signup-name">이름 *</label>
    <input
      type="text"
      id="signup-name"
      required
      placeholder="홍길동"
    >
  </div>

  <div class="form-group">
    <label for="signup-email">이메일 *</label>
    <input
      type="email"
      id="signup-email"
      required
      placeholder="example@domain.com"
    >
  </div>

  <div class="form-group">
    <label for="signup-password">비밀번호 *</label>
    <input
      type="password"
      id="signup-password"
      required
      minlength="8"
      placeholder="8자 이상"
    >
  </div>

  <div class="form-group">
    <label for="signup-tel">전화번호</label>
    <input
      type="tel"
      id="signup-tel"
      placeholder="010-1234-5678"
      pattern="[0-9]{3}-[0-9]{4}-[0-9]{4}"
    >
  </div>

  <div class="form-group">
    <label for="signup-bio">자기소개</label>
    <textarea
      id="signup-bio"
      rows="5"
      placeholder="자기소개를 작성해주세요 (선택)"
    ></textarea>
  </div>

  <div class="form-group">
    <label>
      <input type="checkbox" required>
      이용약관에 동의합니다 *
    </label>
  </div>

  <button type="submit">가입하기</button>
</form>
```

### 검색 폼

```html
<form class="search-form">
  <input
    type="search"
    placeholder="검색어를 입력하세요"
    aria-label="검색"
  >
  <button type="submit">
    <svg><!-- 검색 아이콘 --></svg>
    검색
  </button>
</form>
```

```css
.search-form {
  display: flex;
  max-width: 600px;
  margin: 20px auto;
}

.search-form input {
  flex: 1;
  padding: 12px 20px;
  border: 2px solid #ddd;
  border-right: none;
  border-radius: 25px 0 0 25px;
  font-size: 15px;
}

.search-form input:focus {
  outline: none;
  border-color: #007bff;
}

.search-form button {
  padding: 12px 30px;
  background-color: #007bff;
  color: white;
  border: 2px solid #007bff;
  border-radius: 0 25px 25px 0;
  cursor: pointer;
}

.search-form button:hover {
  background-color: #0056b3;
  border-color: #0056b3;
}
```

## 12. 접근성 (Accessibility) 고려사항

### Label 필수 사용

```html
<!-- ❌ 접근성 나쁨 -->
<input type="text" placeholder="이름">

<!-- ✅ 접근성 좋음 -->
<label for="name">이름</label>
<input type="text" id="name" placeholder="이름">
```

### ARIA 속성

```html
<input
  type="text"
  aria-label="검색어"
  aria-required="true"
  aria-invalid="false"
>
```

### 키보드 내비게이션

```css
/* 키보드 포커스 시 명확한 표시 */
input:focus,
button:focus {
  outline: 2px solid #007bff;
  outline-offset: 2px;
}

/* 마우스 클릭 시에는 outline 제거 (선택사항) */
input:focus:not(:focus-visible) {
  outline: none;
}
```

### 에러 메시지

```html
<div class="form-group">
  <label for="email">이메일</label>
  <input
    type="email"
    id="email"
    aria-describedby="email-error"
    required
  >
  <span id="email-error" class="error-message">
    올바른 이메일 형식을 입력해주세요
  </span>
</div>
```

## 13. Best Practices

### ✅ 좋은 예

```html
<!-- 명확한 label 사용 -->
<label for="username">사용자명</label>
<input type="text" id="username" name="username">

<!-- 적절한 input type 사용 -->
<input type="email" name="email">  <!-- text 아님 -->
<input type="tel" name="phone">    <!-- number 아님 -->

<!-- 유효성 검증 속성 활용 -->
<input type="password" required minlength="8">

<!-- 접근성 고려 -->
<label>
  <input type="checkbox" required>
  <span>약관에 동의합니다</span>
</label>
```

### ❌ 나쁜 예

```html
<!-- label 없음 -->
<input type="text" placeholder="이름">

<!-- 잘못된 type 사용 -->
<input type="number" placeholder="전화번호">

<!-- placeholder를 label 대용으로 사용 -->
<input type="text" placeholder="이메일">

<!-- 닫는 태그 사용 (잘못됨) -->
<input type="text"></input>
```

## 정리

### Input Type 선택 가이드

| 데이터 | 추천 Type |
|--------|-----------|
| 이름, 주소 | `text` |
| 이메일 | `email` |
| 비밀번호 | `password` |
| 전화번호 | `tel` |
| 웹사이트 | `url` |
| 숫자 | `number` |
| 날짜 | `date` |
| 선택 (하나) | `radio` |
| 선택 (여러개) | `checkbox` |
| 긴 텍스트 | `textarea` |

### 주요 속성

```html
<input
  type="text"           <!-- 입력 타입 -->
  name="username"       <!-- 서버 전송 시 키 -->
  id="username"         <!-- label 연결용 -->
  placeholder="힌트"    <!-- 도움말 -->
  value="기본값"        <!-- 초기값 -->
  required              <!-- 필수 입력 -->
  disabled              <!-- 비활성화 -->
  readonly              <!-- 읽기 전용 -->
  minlength="5"         <!-- 최소 길이 -->
  maxlength="20"        <!-- 최대 길이 -->
  pattern="[A-Za-z]+"   <!-- 정규식 검증 -->
>
```

### CSS 선택자 요약

```css
/* 요소 선택 */
input { }

/* 타입 선택 */
input[type="text"] { }

/* 상태 선택 */
input:focus { }
input:valid { }
input:invalid { }
input:disabled { }

/* 가상 요소 */
input::placeholder { }
```

## 참고 자료

- [MDN - input](https://developer.mozilla.org/ko/docs/Web/HTML/Element/Input)
- [MDN - textarea](https://developer.mozilla.org/ko/docs/Web/HTML/Element/textarea)
- [MDN - button](https://developer.mozilla.org/ko/docs/Web/HTML/Element/button)
- [MDN - Form Validation](https://developer.mozilla.org/ko/docs/Learn/Forms/Form_validation)
- [Web.dev - Forms](https://web.dev/learn/forms/)
