---
title: CSS 상속과 그룹 선택자 완벽 가이드
description: CSS의 상속(Inheritance) 개념과 그룹 선택자(Grouping) 활용법을 이해하고, 효율적인 스타일 작성 방법을 학습합니다.
author: changsu
date: 2020-06-13 16:00:15 +0900
categories: [Web, CSS]
tags: [css, inheritance, grouping, selector, cascade, specificity]
---

## 1. CSS 상속(Inheritance)

### 1.1 상속이란?

**상속(Inheritance)**은 부모 요소의 CSS 속성이 자식 요소에게 자동으로 전달되는 CSS의 핵심 특성입니다.

```html
<style>
body {
  color: red;
  font-size: 14px;
}
</style>

<body>
  <p>이 텍스트는 빨간색입니다.</p>
  <div>
    <span>이것도 빨간색입니다.</span>
  </div>
</body>
```

위 예제에서 `<p>`, `<div>`, `<span>` 모두 `body`의 색상과 폰트 크기를 상속받습니다.

### 1.2 상속되는 속성 vs 상속되지 않는 속성

#### 상속되는 속성 (Inherited Properties)

주로 **텍스트 관련 속성**이 상속됩니다.

| 속성 카테고리 | 예시 |
|--------------|------|
| **폰트** | `font-family`, `font-size`, `font-weight`, `font-style` |
| **텍스트** | `color`, `text-align`, `line-height`, `letter-spacing` |
| **리스트** | `list-style`, `list-style-type` |
| **커서** | `cursor` |
| **가시성** | `visibility` |

```css
/* 부모 요소 */
.parent {
  font-family: 'Arial', sans-serif;
  font-size: 16px;
  color: #333;
  line-height: 1.6;
}

/* 자식 요소는 자동으로 상속받음 */
.child {
  /* font-family: Arial (상속) */
  /* font-size: 16px (상속) */
  /* color: #333 (상속) */
  /* line-height: 1.6 (상속) */
}
```

#### 상속되지 않는 속성 (Non-inherited Properties)

주로 **레이아웃과 박스 모델 속성**은 상속되지 않습니다.

| 속성 카테고리 | 예시 |
|--------------|------|
| **박스 모델** | `width`, `height`, `margin`, `padding`, `border` |
| **레이아웃** | `display`, `position`, `float`, `flex`, `grid` |
| **배경** | `background`, `background-color`, `background-image` |
| **변형** | `transform`, `transition`, `animation` |

```css
.parent {
  width: 500px;          /* 상속 안 됨 */
  padding: 20px;         /* 상속 안 됨 */
  background: blue;      /* 상속 안 됨 */
  border: 1px solid;     /* 상속 안 됨 */
}

.child {
  /* 위 속성들은 자동으로 적용되지 않음 */
}
```

### 1.3 상속 제어하기

CSS는 상속을 명시적으로 제어할 수 있는 키워드를 제공합니다.

#### inherit (상속받기)

```css
.parent {
  color: red;
  border: 2px solid black;
}

.child {
  /* 일반적으로 상속되지 않는 border를 강제로 상속 */
  border: inherit;
  /* 결과: border: 2px solid black */
}
```

#### initial (초기값으로 리셋)

```css
.parent {
  color: red;
  font-size: 20px;
}

.child {
  /* 브라우저의 기본 초기값으로 리셋 */
  color: initial;  /* 보통 black */
  font-size: initial;  /* 보통 16px */
}
```

#### unset (상속 가능하면 상속, 아니면 초기값)

```css
.parent {
  color: red;
  border: 2px solid black;
}

.child {
  color: unset;   /* 상속 가능 → red */
  border: unset;  /* 상속 불가 → initial (none) */
}
```

#### revert (브라우저 기본 스타일로)

```css
h1 {
  font-size: revert;  /* 브라우저의 h1 기본 크기로 */
}
```

### 1.4 상속 우선순위

자식 요소에 직접 스타일이 지정되면 **상속된 스타일보다 우선**합니다.

```html
<style>
.parent {
  color: red;
}

.child {
  color: blue;  /* 이게 우선 적용됨 */
}
</style>

<div class="parent">
  <p class="child">이 텍스트는 파란색입니다.</p>
</div>
```

> **중요**: 상속된 스타일은 명시도(Specificity)가 0입니다. 따라서 어떤 직접 선택된 스타일보다도 우선순위가 낮습니다.
{: .prompt-warning }

## 2. 그룹 선택자(Grouping)

### 2.1 그룹 선택자란?

여러 선택자에 동일한 스타일을 적용할 때 **쉼표(,)**로 구분하여 그룹화할 수 있습니다.

```css
/* ❌ 비효율적: 반복적인 코드 */
h1 {
  color: green;
  font-weight: bold;
}

h2 {
  color: green;
  font-weight: bold;
}

h3 {
  color: green;
  font-weight: bold;
}

/* ✅ 효율적: 그룹 선택자 사용 */
h1, h2, h3 {
  color: green;
  font-weight: bold;
}
```

### 2.2 다양한 그룹 선택자 패턴

#### 태그 선택자 그룹

```css
p, span, div, section {
  margin: 0;
  padding: 0;
}
```

#### 클래스 선택자 그룹

```css
.btn-primary, .btn-secondary, .btn-danger {
  padding: 10px 20px;
  border-radius: 4px;
  cursor: pointer;
}
```

#### 혼합 선택자 그룹

```css
h1, .title, #main-heading {
  font-family: 'Georgia', serif;
  line-height: 1.2;
}
```

#### 복합 선택자 그룹

```css
.header .nav-link,
.footer .nav-link,
.sidebar .nav-link {
  text-decoration: none;
  color: #333;
}
```

### 2.3 그룹 선택자의 장점

#### 1. 코드 재사용성 향상

```css
/* Before: 72줄 */
.card-header { font-size: 18px; font-weight: bold; color: #333; }
.card-body { font-size: 18px; font-weight: bold; color: #333; }
.card-footer { font-size: 18px; font-weight: bold; color: #333; }

/* After: 4줄 */
.card-header,
.card-body,
.card-footer {
  font-size: 18px;
  font-weight: bold;
  color: #333;
}
```

#### 2. 유지보수 용이

```css
/* 한 곳만 수정하면 모든 요소에 적용 */
.primary-text,
.highlight-text,
.important-text {
  color: #007bff;  /* 여기만 수정 */
}
```

#### 3. 파일 크기 감소

```css
/* 불필요한 반복 제거 → CSS 파일 크기 감소 */
input, select, textarea, button {
  font-family: inherit;
  font-size: 100%;
}
```

### 2.4 개별 스타일 추가하기

그룹 선택자로 공통 스타일을 지정한 후, 개별적으로 추가 스타일을 지정할 수 있습니다.

```css
/* 공통 스타일 */
.btn-primary,
.btn-secondary,
.btn-danger {
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

/* 개별 스타일 */
.btn-primary {
  background-color: #007bff;
  color: white;
}

.btn-secondary {
  background-color: #6c757d;
  color: white;
}

.btn-danger {
  background-color: #dc3545;
  color: white;
}
```

## 3. 상속과 그룹 선택자 함께 활용하기

### 3.1 전역 스타일 설정

```css
/* 상속을 활용한 전역 폰트 설정 */
body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
  font-size: 16px;
  line-height: 1.6;
  color: #333;
}

/* 그룹 선택자로 제목들의 공통 스타일 */
h1, h2, h3, h4, h5, h6 {
  font-weight: 700;
  margin-top: 0;
  margin-bottom: 0.5em;
}

/* 개별 크기 지정 */
h1 { font-size: 2.5em; }
h2 { font-size: 2em; }
h3 { font-size: 1.75em; }
```

### 3.2 리셋 CSS

```css
/* 그룹 선택자로 브라우저 기본 스타일 리셋 */
html, body, div, span, h1, h2, h3, h4, h5, h6,
p, blockquote, pre, a, code, img, small, strong,
ul, ol, li, form, label, table, tbody, thead, tr, th, td,
article, aside, footer, header, nav, section {
  margin: 0;
  padding: 0;
  border: 0;
  font: inherit;
  vertical-align: baseline;
}

/* 상속을 활용한 박스 모델 설정 */
html {
  box-sizing: border-box;
}

*, *:before, *:after {
  box-sizing: inherit;
}
```

### 3.3 컴포넌트 스타일링

```css
/* 카드 컴포넌트 */
.card {
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 20px;
  font-family: 'Helvetica', sans-serif;  /* 상속됨 */
  color: #333;  /* 상속됨 */
}

/* 카드 내부 요소들은 폰트와 색상 상속 */
.card-title {
  font-size: 1.5em;
  font-weight: bold;
  /* font-family와 color는 .card에서 상속 */
}

.card-content {
  margin-top: 10px;
  /* font-family와 color는 .card에서 상속 */
}

/* 그룹 선택자로 카드 변형 */
.card-primary,
.card-success,
.card-danger {
  padding: 24px;
  font-weight: 500;
}

.card-primary { border-color: #007bff; }
.card-success { border-color: #28a745; }
.card-danger { border-color: #dc3545; }
```

## 4. 실전 예제

### 4.1 네비게이션 메뉴

```css
/* 공통 네비게이션 스타일 (상속 활용) */
nav {
  font-family: 'Arial', sans-serif;
  font-size: 14px;
}

/* 모든 링크에 공통 스타일 (그룹 선택자) */
nav a,
nav button {
  text-decoration: none;
  color: #333;
  padding: 10px 15px;
  display: inline-block;
  transition: all 0.3s;
}

/* 호버 효과 (그룹 선택자) */
nav a:hover,
nav button:hover {
  background-color: #f0f0f0;
  color: #007bff;
}

/* 활성 링크 */
nav a.active {
  color: #007bff;
  font-weight: bold;
}
```

**HTML:**
```html
<nav>
  <a href="/" class="active">홈</a>
  <a href="/about">소개</a>
  <a href="/services">서비스</a>
  <button>문의하기</button>
</nav>
```

### 4.2 폼 요소 스타일링

```css
/* 폼 컨테이너 (상속 설정) */
.form-container {
  font-family: 'Roboto', sans-serif;
  font-size: 14px;
  color: #333;
}

/* 모든 입력 요소에 공통 스타일 (그룹 선택자) */
input[type="text"],
input[type="email"],
input[type="password"],
textarea,
select {
  width: 100%;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-family: inherit;  /* 부모 폰트 상속 */
  font-size: inherit;    /* 부모 크기 상속 */
}

/* 포커스 상태 (그룹 선택자) */
input[type="text"]:focus,
input[type="email"]:focus,
input[type="password"]:focus,
textarea:focus,
select:focus {
  outline: none;
  border-color: #007bff;
  box-shadow: 0 0 0 3px rgba(0, 123, 255, 0.1);
}

/* 레이블 스타일 */
label {
  display: block;
  margin-bottom: 5px;
  font-weight: 500;
  /* color는 .form-container에서 상속 */
}
```

### 4.3 타이포그래피 시스템

```css
/* 기본 타이포그래피 (상속 활용) */
body {
  font-family: 'Noto Sans KR', sans-serif;
  font-size: 16px;
  line-height: 1.6;
  color: #2c3e50;
}

/* 제목 그룹 (그룹 선택자) */
h1, h2, h3, h4, h5, h6 {
  font-weight: 700;
  line-height: 1.2;
  margin-top: 1.5em;
  margin-bottom: 0.5em;
  color: #1a1a1a;
}

/* 개별 크기 */
h1 { font-size: 2.5rem; }
h2 { font-size: 2rem; }
h3 { font-size: 1.75rem; }
h4 { font-size: 1.5rem; }
h5 { font-size: 1.25rem; }
h6 { font-size: 1rem; }

/* 텍스트 강조 (그룹 선택자) */
strong, b, .bold {
  font-weight: 700;
}

em, i, .italic {
  font-style: italic;
}

/* 코드 블록 (그룹 선택자) */
code, pre, kbd {
  font-family: 'Monaco', 'Courier New', monospace;
  background-color: #f4f4f4;
  padding: 2px 4px;
  border-radius: 3px;
}
```

### 4.4 버튼 시스템

```css
/* 모든 버튼의 기본 스타일 (그룹 선택자) */
.btn,
button[type="button"],
button[type="submit"],
input[type="button"],
input[type="submit"] {
  display: inline-block;
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  font-size: 14px;
  font-weight: 500;
  text-align: center;
  cursor: pointer;
  transition: all 0.3s;
}

/* 버튼 크기 변형 (그룹 선택자) */
.btn-sm, .btn.small {
  padding: 6px 12px;
  font-size: 12px;
}

.btn-lg, .btn.large {
  padding: 14px 28px;
  font-size: 16px;
}

/* 버튼 색상 */
.btn-primary {
  background-color: #007bff;
  color: white;
}

.btn-primary:hover {
  background-color: #0056b3;
}

.btn-secondary {
  background-color: #6c757d;
  color: white;
}

.btn-secondary:hover {
  background-color: #545b62;
}
```

## 5. Best Practices

### 5.1 상속 활용 팁

```css
/* ✅ 좋은 예: 상속 활용 */
body {
  font-family: 'Helvetica', sans-serif;
  font-size: 16px;
  line-height: 1.6;
  color: #333;
}

/* 자식 요소들은 자동으로 상속받음 */
/* 필요한 경우에만 오버라이드 */
h1 {
  font-size: 2em;  /* 크기만 변경 */
}

/* ❌ 나쁜 예: 불필요한 반복 */
p {
  font-family: 'Helvetica', sans-serif;
  line-height: 1.6;
  color: #333;
}

div {
  font-family: 'Helvetica', sans-serif;
  line-height: 1.6;
  color: #333;
}
```

### 5.2 그룹 선택자 가독성

```css
/* ✅ 좋은 예: 한 줄에 하나씩 (가독성 좋음) */
h1,
h2,
h3,
h4 {
  font-weight: bold;
}

/* ❌ 나쁜 예: 너무 긴 한 줄 */
h1, h2, h3, h4, h5, h6, .title, .heading, .subtitle, .header-text {
  font-weight: bold;
}
```

### 5.3 논리적 그룹화

```css
/* ✅ 좋은 예: 관련 있는 선택자끼리 그룹화 */
.header-link,
.footer-link,
.nav-link {
  text-decoration: none;
}

/* ❌ 나쁜 예: 관련 없는 선택자 그룹화 */
.button,
.input,
.header-link {
  text-decoration: none;  /* button에는 부적절 */
}
```

### 5.4 특이성(Specificity) 고려

```css
/* 그룹 선택자는 각 선택자의 특이성을 유지 */
#unique,        /* 특이성: 100 */
.common,        /* 특이성: 10 */
div {           /* 특이성: 1 */
  color: blue;
}

/* 각 선택자는 독립적으로 특이성 계산됨 */
```

## 6. 주의사항

### 6.1 상속 관련

```css
/* ⚠️ 주의: 모든 속성이 상속되는 것은 아님 */
.parent {
  border: 2px solid black;  /* 상속 안 됨 */
  padding: 20px;            /* 상속 안 됨 */
  color: red;               /* 상속됨 */
}

.child {
  /* border와 padding은 상속되지 않음 */
  /* 필요하면 명시적으로 inherit 사용 */
  border: inherit;
}
```

### 6.2 그룹 선택자 오류

```css
/* ⚠️ 주의: 잘못된 선택자 하나가 전체를 무효화 */
input::-webkit-input-placeholder,
input::-moz-placeholder,
input::invalid-selector {  /* 이 선택자가 틀림 */
  color: gray;
}
/* Firefox에서는 -webkit- 선택자를 인식 못 함 */
/* 전체 규칙이 무시됨! */

/* ✅ 해결: 개별적으로 작성 */
input::-webkit-input-placeholder {
  color: gray;
}
input::-moz-placeholder {
  color: gray;
}
```

### 6.3 성능 고려

```css
/* ✅ 좋은 예: 적절한 그룹화 */
.btn-primary,
.btn-secondary,
.btn-tertiary {
  padding: 10px;
}

/* ⚠️ 주의: 너무 많은 선택자 (가독성/유지보수 저하) */
.selector1, .selector2, .selector3, ... .selector50 {
  /* 50개 이상의 선택자 */
}
```

## 정리

### 상속(Inheritance)

1. **상속되는 속성**: 주로 텍스트 관련 (color, font-family, line-height 등)
2. **상속 안 되는 속성**: 레이아웃, 박스 모델 (width, margin, border 등)
3. **명시적 제어**: `inherit`, `initial`, `unset`, `revert` 키워드
4. **우선순위**: 직접 지정한 스타일 > 상속된 스타일

### 그룹 선택자(Grouping)

1. **문법**: 쉼표(,)로 선택자 구분
2. **장점**: 코드 재사용, 유지보수 용이, 파일 크기 감소
3. **활용**: 공통 스타일 + 개별 스타일 조합
4. **주의**: 잘못된 선택자 하나가 전체 규칙 무효화

> **핵심**: 상속과 그룹 선택자를 적절히 활용하면 효율적이고 유지보수하기 쉬운 CSS 코드를 작성할 수 있습니다.
{: .prompt-tip }

## 참고 자료

- [MDN - CSS Inheritance](https://developer.mozilla.org/ko/docs/Web/CSS/inheritance)
- [MDN - CSS Selectors](https://developer.mozilla.org/ko/docs/Web/CSS/CSS_Selectors)
- [W3C CSS Cascade](https://www.w3.org/TR/css-cascade-4/)
- [CSS Specificity Calculator](https://specificity.keegan.st/)
