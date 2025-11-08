---
title: CSS 선택자와 명시도 완벽 가이드
description: CSS 선택자의 모든 종류와 결합 방법, 그리고 명시도(Specificity) 우선순위 계산 방법을 이해하고 효율적인 스타일링을 학습합니다.
author: changsu
date: 2020-06-13 17:10:58 +0900
categories: [Web, CSS]
tags: [css, selector, specificity, cascade, priority, css-selector]
---

## 1. CSS 선택자(Selector) 기본

### 1.1 선택자란?

**선택자(Selector)**는 스타일을 적용할 HTML 요소를 지정하는 패턴입니다.

```css
선택자 {
  속성: 값;
}
```

### 1.2 기본 선택자

#### 태그 선택자 (Type Selector)

```css
p {
  font-size: 20px;
}
```

모든 `<p>` 태그에 스타일 적용

#### 클래스 선택자 (Class Selector)

```css
.p-tag {
  color: gray;
}
```

`class="p-tag"`를 가진 모든 요소에 적용

#### 아이디 선택자 (ID Selector)

```css
#third-line {
  text-decoration: underline;
}
```

`id="third-line"`를 가진 요소에 적용 (페이지 내 유일해야 함)

### 1.3 선택자 비교표

| 선택자 | 문법 | 예시 | 명시도 |
|--------|------|------|--------|
| **태그** | `태그명` | `p`, `div`, `span` | 1점 |
| **클래스** | `.클래스명` | `.container`, `.btn` | 10점 |
| **아이디** | `#아이디명` | `#header`, `#main` | 100점 |
| **전체** | `*` | `* { margin: 0; }` | 0점 |
| **속성** | `[속성]` | `[type="text"]` | 10점 |

## 2. 선택자 결합(Combinator)

### 2.1 결합 선택자

#### 태그 + 클래스 결합

```css
p.p-tag {
  color: gray;
}
```

**조건**: `<p>` 태그이면서 `class="p-tag"`인 요소

```html
<p class="p-tag">적용됨 ✅</p>
<p>적용 안 됨 ❌</p>
<div class="p-tag">적용 안 됨 ❌</div>
```

#### 태그 + 아이디 결합

```css
p#third-line {
  text-decoration: underline;
}
```

**조건**: `<p>` 태그이면서 `id="third-line"`인 요소

#### 다중 결합

```css
p#third-line.p-tag {
  font-weight: bold;
}
```

**조건**: `<p>` 태그이면서 `id="third-line"` 그리고 `class="p-tag"`

> **참고**: 아이디는 페이지 내에서 유일하므로 `p#third-line.p-tag`처럼 과도하게 결합하는 것은 불필요합니다.
{: .prompt-info }

### 2.2 자손 선택자 (Descendant Selector)

#### 기본 문법

```css
.pre span {
  background-color: yellow;
}
```

**의미**: `.pre` 클래스 내부에 있는 모든 `<span>` (직계 자식 + 모든 자손)

#### 예제

```html
<div class="pre">
  <span>노란색 배경 ✅</span>
</div>

<div class="main">
  <span>적용 안 됨 ❌</span>
</div>

<span class="pre">적용 안 됨 ❌ (span이 부모가 아님)</span>

<div>
  <p class="pre">
    <span>노란색 배경 ✅ (pre 내부의 span)</span>
  </p>
</div>
```

> **중요**: 자손 선택자는 **공백(스페이스)**으로 구분합니다. 직계 자식뿐만 아니라 모든 하위 요소에 적용됩니다.
{: .prompt-warning }

### 2.3 복잡한 자손 선택자

```css
.a div .b .pre span {
  background-color: yellow;
}
```

**의미**: `.a` → `div` → `.b` → `.pre` → `span` 순서로 중첩된 요소

```html
<div class="a">
  <div>
    <header class="b">
      <h1 class="pre">
        <span>노란색 배경 ✅</span>
        <span class="title">이것도 노란색 ✅</span>
      </h1>
      <span>적용 안 됨 ❌ (pre 내부가 아님)</span>
    </header>
  </div>
  <span>적용 안 됨 ❌</span>
</div>
```

## 3. 다양한 선택자 조합

### 3.1 자식 선택자 (Child Selector)

```css
.parent > .child {
  color: blue;
}
```

**직계 자식**만 선택 (자손 선택자와 다름)

```html
<div class="parent">
  <p class="child">파란색 ✅ (직계 자식)</p>
  <div>
    <p class="child">적용 안 됨 ❌ (손자)</p>
  </div>
</div>
```

### 3.2 인접 형제 선택자 (Adjacent Sibling)

```css
h1 + p {
  font-weight: bold;
}
```

`<h1>` **바로 다음**에 오는 `<p>`만 선택

```html
<h1>제목</h1>
<p>굵게 표시 ✅</p>
<p>적용 안 됨 ❌</p>
```

### 3.3 일반 형제 선택자 (General Sibling)

```css
h1 ~ p {
  color: gray;
}
```

`<h1>` **이후**에 오는 모든 형제 `<p>` 선택

```html
<h1>제목</h1>
<p>회색 ✅</p>
<div>...</div>
<p>회색 ✅</p>
```

### 3.4 속성 선택자 (Attribute Selector)

```css
/* 속성이 있는 요소 */
input[type] {
  border: 1px solid #ccc;
}

/* 속성 값이 일치 */
input[type="text"] {
  background-color: white;
}

/* 속성 값이 포함 */
a[href*="google"] {
  color: blue;
}

/* 속성 값으로 시작 */
a[href^="https"] {
  color: green;
}

/* 속성 값이 포함 */
a[href*="pdf"] {
  color: red;
}
```

### 3.5 가상 클래스 선택자 (Pseudo-class)

```css
/* 링크 상태 */
a:hover {
  color: red;
}

a:visited {
  color: purple;
}

/* 입력 상태 */
input:focus {
  outline: 2px solid blue;
}

input:disabled {
  background-color: #f0f0f0;
}

/* 구조적 가상 클래스 */
li:first-child {
  font-weight: bold;
}

li:last-child {
  border-bottom: none;
}

/* 짝수 번째 */
li:nth-child(even) {
  background-color: #f9f9f9;
}
```

### 3.6 가상 요소 선택자 (Pseudo-element)

```css
/* 첫 글자 */
p::first-letter {
  font-size: 2em;
  font-weight: bold;
}

/* 첫 줄 */
p::first-line {
  color: blue;
}

/* 요소 앞/뒤에 콘텐츠 삽입 */
.icon::before {
  content: "★ ";
  color: gold;
}

.link::after {
  content: " →";
}
```

## 4. CSS 명시도(Specificity)

### 4.1 명시도란?

**명시도(Specificity)**는 같은 요소에 여러 CSS 규칙이 적용될 때 어떤 규칙을 우선 적용할지 결정하는 우선순위 점수입니다.

### 4.2 명시도 점수 계산

| 선택자 타입 | 점수 | 예시 |
|------------|------|------|
| **인라인 스타일** | 1000점 | `<p style="color: red;">` |
| **아이디** | 100점 | `#header` |
| **클래스/속성/가상클래스** | 10점 | `.btn`, `[type="text"]`, `:hover` |
| **태그/가상요소** | 1점 | `p`, `div`, `::before` |
| **전체 선택자** | 0점 | `*` |

### 4.3 명시도 계산 예제

```css
/* 예제 1: 태그 vs 클래스 */
/* 1점 */
p {
  font-size: 30px;
}

/* 10점 - 적용됨 */
.p-tag {
  font-size: 15px;
}
```

```html
<p class="p-tag">나는 15px입니다</p>
```

**계산**: 클래스(10점) > 태그(1점) → `.p-tag` 스타일 적용

---

```css
/* 예제 2: 결합 선택자 */
/* 1 + 10 = 11점 - 적용됨 */
p.p-tag {
  font-size: 100px;
}

/* 10점 */
.p-tag {
  font-size: 15px;
}
```

**계산**: `p.p-tag`(11점) > `.p-tag`(10점)

---

```css
/* 예제 3: 복잡한 선택자 */
/* 100 + 10 + 1 + 1 = 112점 */
#header .nav ul li {
  color: blue;
}

/* 10 + 1 = 11점 */
.nav li {
  color: red;
}

/* 1점 */
li {
  color: green;
}
```

```html
<header id="header">
  <nav class="nav">
    <ul>
      <li>파란색 ✅ (112점이 가장 높음)</li>
    </ul>
  </nav>
</header>
```

### 4.4 명시도 우선순위 요약

```
인라인 스타일(1000점)
     >>>
아이디(100점)
     >>>
클래스/속성/가상클래스(10점)
     >>>
태그/가상요소(1점)
```

### 4.5 명시도 계산 연습

```css
/* 문제: 다음 중 어떤 스타일이 적용될까? */

/* 1 + 1 = 2점 */
div p {
  color: red;
}

/* 10점 */
.text {
  color: blue;
}

/* 100 + 10 = 110점 */
#main .text {
  color: green;
}

/* 1 + 100 + 10 = 111점 - 최종 적용 */
div#main .text {
  color: purple;
}
```

```html
<div id="main">
  <p class="text">보라색으로 표시됩니다</p>
</div>
```

## 5. !important와 예외 상황

### 5.1 !important

```css
/* 모든 명시도를 무시하고 최우선 적용 */
p {
  color: red !important;
}

/* 100점이지만 !important보다 낮음 */
#unique {
  color: blue;
}
```

```html
<p id="unique">빨간색입니다 (!important 때문)</p>
```

> **Warning**: `!important`는 명시도 시스템을 깨뜨리므로 가급적 사용하지 않는 것이 좋습니다.
{: .prompt-warning }

### 5.2 !important 사용이 허용되는 경우

```css
/* 유틸리티 클래스 */
.hidden {
  display: none !important;
}

.text-center {
  text-align: center !important;
}

/* 외부 라이브러리 스타일 덮어쓰기 */
.external-lib-component {
  margin: 0 !important;
}
```

### 5.3 인라인 스타일 덮어쓰기

```html
<!-- 인라인 스타일 (1000점) -->
<p style="color: blue;">파란색</p>
```

```css
/* 인라인 스타일을 덮어쓰려면 !important 필요 */
/* 이제 빨간색으로 표시됨 */
p {
  color: red !important;
}
```

## 6. 실전 Best Practices

### 6.1 명시도를 낮게 유지하기

```css
/* ❌ 나쁜 예: 과도하게 구체적 */
/* 5점 - 유지보수 어려움 */
body div.container div.content div.article p.text {
  color: red;
}

/* ✅ 좋은 예: 간결하고 명확 */
/* 10점 - 간단하고 재사용 가능 */
.article-text {
  color: red;
}
```

### 6.2 클래스 중심으로 스타일링

```css
/* ✅ 권장: 클래스 사용 */
.btn {
  padding: 10px 20px;
}

.btn-primary {
  background-color: blue;
}

/* ❌ 비권장: 아이디는 재사용 불가 */
#submit-btn {
  padding: 10px 20px;
}
```

### 6.3 BEM 명명 규칙

```css
/* Block__Element--Modifier 패턴 */
.card { }
.card__title { }
.card__content { }
.card--featured { }
```

```html
<div class="card card--featured">
  <h2 class="card__title">제목</h2>
  <p class="card__content">내용</p>
</div>
```

### 6.4 선택자 깊이 제한

```css
/* ❌ 나쁜 예: 너무 깊은 중첩 */
.header .nav .menu .item .link { }

/* ✅ 좋은 예: 최대 2-3단계 */
.nav-link { }
.menu .nav-link { }
```

### 6.5 !important 대신 명시도 활용

```css
/* ❌ 나쁜 예 */
.button {
  color: blue !important;
}

/* ✅ 좋은 예 */
/* 명시도: 20점 */
.button.button-override {
  color: blue;
}
```

## 7. 명시도 디버깅

### 7.1 브라우저 개발자 도구 활용

```css
.text {
  color: red;
}

#main .text {
  color: blue;
}
```

Chrome DevTools에서:
1. 요소 선택
2. **Computed** 탭에서 적용된 스타일 확인
3. 취소선이 그어진 스타일은 명시도가 낮아 무시됨

### 7.2 명시도 계산기

온라인 도구 활용:
- [Specificity Calculator](https://specificity.keegan.st/)
- [CSS Specificity Graph](https://jonassebastianohlsson.com/specificity-graph/)

## 8. 실전 예제

### 8.1 네비게이션 메뉴

```css
/* 기본 스타일 (10점) */
.nav-link {
  color: #333;
  text-decoration: none;
  padding: 10px 15px;
}

/* 호버 상태 (10 + 10 = 20점) */
.nav-link:hover {
  color: #007bff;
  background-color: #f0f0f0;
}

/* 활성 링크 (10 + 10 = 20점) */
.nav-link.active {
  color: #007bff;
  font-weight: bold;
}

/* 특정 위치의 링크 (10 + 10 = 20점) */
.header .nav-link {
  font-size: 14px;
}
```

### 8.2 버튼 컴포넌트

```css
/* 기본 버튼 (10점) */
.btn {
  display: inline-block;
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

/* 버튼 색상 (10 + 10 = 20점) */
.btn.btn-primary {
  background-color: #007bff;
  color: white;
}

.btn.btn-secondary {
  background-color: #6c757d;
  color: white;
}

/* 버튼 크기 (10 + 10 = 20점) */
.btn.btn-lg {
  padding: 14px 28px;
  font-size: 18px;
}

/* 비활성 버튼 (10 + 10 = 20점) */
.btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
```

### 8.3 폼 요소

```css
/* 모든 입력 요소 (10점) */
.form-input {
  width: 100%;
  padding: 8px 12px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

/* 포커스 상태 (10 + 10 = 20점) */
.form-input:focus {
  outline: none;
  border-color: #007bff;
  box-shadow: 0 0 0 3px rgba(0, 123, 255, 0.1);
}

/* 에러 상태 (10 + 10 = 20점) */
.form-input.error {
  border-color: #dc3545;
}

/* 에러 입력의 포커스 (10 + 10 + 10 = 30점) */
.form-input.error:focus {
  border-color: #dc3545;
  box-shadow: 0 0 0 3px rgba(220, 53, 69, 0.1);
}
```

## 정리

### CSS 선택자 핵심

1. **기본 선택자**: 태그, 클래스(`.`), 아이디(`#`)
2. **결합 선택자**: `p.class`, `p#id`, `p.class.other`
3. **자손 선택자**: `.parent .child` (공백으로 구분)
4. **자식 선택자**: `.parent > .child` (직계 자식만)
5. **형제 선택자**: `h1 + p` (인접), `h1 ~ p` (일반)

### 명시도 우선순위

```
!important (절대 우선)
  ↓
인라인 스타일 (1000점)
  ↓
아이디 (100점)
  ↓
클래스/속성/가상클래스 (10점)
  ↓
태그/가상요소 (1점)
  ↓
전체 선택자 (0점)
```

### 실무 권장사항

1. **클래스 중심**으로 스타일링
2. **명시도를 낮게** 유지 (간결한 선택자)
3. **!important 지양** (예외: 유틸리티 클래스)
4. **선택자 깊이 제한** (2-3단계 권장)
5. **BEM 등 명명 규칙** 활용

> **핵심**: 명시도를 이해하면 CSS 충돌 문제를 예방하고 유지보수하기 쉬운 코드를 작성할 수 있습니다.
{: .prompt-tip }

## 참고 자료

- [MDN - CSS Selectors](https://developer.mozilla.org/ko/docs/Web/CSS/CSS_Selectors)
- [MDN - Specificity](https://developer.mozilla.org/ko/docs/Web/CSS/Specificity)
- [W3C CSS Selectors Level 4](https://www.w3.org/TR/selectors-4/)
- [Specificity Calculator](https://specificity.keegan.st/)
- [CSS Guidelines](https://cssguidelin.es/)
