---
title: CSS Media Query 완벽 가이드 - 반응형 웹디자인의 핵심
description: CSS Media Query로 반응형 웹사이트를 구현하는 방법을 학습합니다. 문법, Breakpoint 전략, Mobile-first 접근법, 실전 활용까지 다룹니다.
author: changsu
date: 2020-06-15 14:55:50 +0900
categories: [Web, CSS]
tags: [css, responsive, media-query]
---

## 개요

Media Query는 다양한 화면 크기와 디바이스에 대응하는 반응형 웹사이트(Responsive Web Design, RWD)를 구현하는 핵심 CSS 기술입니다. 특정 조건에서 다른 스타일을 적용할 수 있어, 하나의 코드로 모바일부터 데스크톱까지 모든 환경에 최적화된 경험을 제공할 수 있습니다.

## 1. Media Query 기본 개념

### 반응형 웹디자인이란?

```css
/* 기본 스타일 */
.container {
  width: 100%;
  padding: 20px;
}

/* 작은 화면: 모바일 */
@media (max-width: 768px) {
  .container {
    padding: 10px;
  }
}

/* 큰 화면: 데스크톱 */
@media (min-width: 1200px) {
  .container {
    max-width: 1140px;
    margin: 0 auto;
  }
}
```

**결과**: 화면 크기에 따라 자동으로 레이아웃이 변경됩니다.

## 2. Media Query 문법

### 기본 구조

```css
@media [media-type] and ([media-feature]) {
  /* CSS 규칙 */
}
```

**구성 요소:**
1. `@media`: Media query 시작 키워드
2. `media-type`: 미디어 타입 (screen, print 등)
3. `media-feature`: 조건 (width, height 등)
4. `{ }`: 조건이 참일 때 적용할 CSS

### 실제 예제

```css
@media only screen and (max-width: 480px) {
  body {
    font-size: 12px;
  }
}
```

**해석:**
- `@media`: 미디어 쿼리 시작
- `only`: 미디어 쿼리를 지원하는 브라우저만 (생략 가능)
- `screen`: 화면에 표시할 때
- `and`: 조건 연결
- `(max-width: 480px)`: 너비가 480px 이하일 때
- `{ }`: 적용할 스타일

## 3. Media Type

### 주요 타입

```css
/* 모든 디바이스 */
@media all {
  /* 모든 미디어에 적용 */
}

/* 화면 (컴퓨터, 태블릿, 스마트폰) */
@media screen {
  /* 화면에만 적용 */
}

/* 인쇄 */
@media print {
  /* 인쇄할 때만 적용 */
}

/* 화면 읽기 (접근성) */
@media speech {
  /* 스크린 리더에만 적용 */
}
```

### 타입 비교

| 타입 | 설명 | 사용 빈도 |
|------|------|-----------|
| **all** | 모든 디바이스 | ⭐️⭐️⭐️ |
| **screen** | 화면 디바이스 | ⭐️⭐️⭐️⭐️⭐️ |
| **print** | 인쇄용 | ⭐️⭐️⭐️ |
| **speech** | 음성 합성기 | ⭐️ |

### 실전 활용

```css
/* 화면용 스타일 */
@media screen {
  body {
    background-color: #f8f9fa;
  }

  .no-print {
    display: block;
  }
}

/* 인쇄용 스타일 */
@media print {
  body {
    background-color: white;
    color: black;
  }

  .no-print {
    display: none;  /* 인쇄 시 숨김 */
  }

  a::after {
    content: " (" attr(href) ")";  /* 링크 URL 표시 */
  }
}
```

## 4. Media Features

### Width 관련

```css
/* 최대 너비 (이하) */
@media (max-width: 768px) {
  /* 768px 이하에서 적용 */
}

/* 최소 너비 (이상) */
@media (min-width: 769px) {
  /* 769px 이상에서 적용 */
}

/* 정확한 너비 */
@media (width: 768px) {
  /* 정확히 768px일 때 (거의 사용 안 함) */
}

/* 범위 지정 (CSS4) */
@media (768px <= width <= 1024px) {
  /* 768px ~ 1024px 사이 */
}
```

### Height 관련

```css
/* 높이 조건 */
@media (max-height: 600px) {
  .tall-content {
    display: none;
  }
}

@media (min-height: 800px) {
  .hero {
    height: 100vh;
  }
}
```

### Orientation (방향)

```css
/* 가로 모드 */
@media (orientation: landscape) {
  .sidebar {
    width: 300px;
  }
}

/* 세로 모드 */
@media (orientation: portrait) {
  .sidebar {
    width: 100%;
  }
}
```

### Aspect Ratio (화면 비율)

```css
/* 16:9 비율 */
@media (aspect-ratio: 16/9) {
  .video {
    width: 100%;
  }
}

/* 최소 화면 비율 */
@media (min-aspect-ratio: 16/9) {
  /* 와이드 스크린 */
}
```

### Resolution (해상도)

```css
/* 레티나 디스플레이 */
@media (min-resolution: 2dppx) {
  .logo {
    background-image: url('logo@2x.png');
  }
}

/* 저해상도 */
@media (max-resolution: 1dppx) {
  .logo {
    background-image: url('logo.png');
  }
}
```

### Hover 지원

```css
/* 마우스 호버 가능 (데스크톱) */
@media (hover: hover) {
  .button:hover {
    background-color: #0056b3;
  }
}

/* 호버 불가능 (터치 디바이스) */
@media (hover: none) {
  .button:active {
    background-color: #0056b3;
  }
}
```

### Pointer 정밀도

```css
/* 정밀한 포인터 (마우스) */
@media (pointer: fine) {
  .button {
    padding: 5px 10px;
  }
}

/* 부정확한 포인터 (터치) */
@media (pointer: coarse) {
  .button {
    padding: 12px 20px;  /* 더 큰 터치 영역 */
  }
}
```

### Prefers Color Scheme (다크 모드)

```css
/* 라이트 모드 */
@media (prefers-color-scheme: light) {
  body {
    background-color: white;
    color: black;
  }
}

/* 다크 모드 */
@media (prefers-color-scheme: dark) {
  body {
    background-color: #1a1a1a;
    color: white;
  }
}
```

### Prefers Reduced Motion (애니메이션 축소)

```css
/* 일반 사용자 */
@media (prefers-reduced-motion: no-preference) {
  .animated {
    transition: all 0.3s;
  }
}

/* 애니메이션 최소화 선호 */
@media (prefers-reduced-motion: reduce) {
  * {
    animation: none !important;
    transition: none !important;
  }
}
```

## 5. 논리 연산자

### AND 연산자

```css
/* 두 조건 모두 만족 */
@media screen and (min-width: 768px) and (max-width: 1024px) {
  /* 태블릿 크기 */
}

/* 여러 조건 */
@media (min-width: 768px) and (orientation: landscape) {
  /* 가로 모드 태블릿 이상 */
}
```

### OR 연산자 (쉼표)

```css
/* 둘 중 하나만 만족 */
@media (max-width: 768px), (orientation: portrait) {
  /* 모바일 또는 세로 모드 */
}

/* 여러 범위 */
@media (max-width: 480px), (min-width: 1200px) {
  /* 아주 작거나 아주 큰 화면 */
}
```

### NOT 연산자

```css
/* 조건 부정 */
@media not screen and (color) {
  /* 컬러 스크린이 아닌 경우 */
}

/* 특정 크기 제외 */
@media not all and (min-width: 768px) {
  /* 768px 미만 */
}
```

### 복합 조건

```css
/* 복잡한 조합 */
@media screen and (min-width: 768px) and (max-width: 1024px),
       screen and (orientation: portrait) {
  /* 태블릿 크기 또는 세로 모드 */
}
```

## 6. Breakpoint 전략

### 일반적인 Breakpoint

```css
/* 모바일 (기본) */
.container {
  width: 100%;
  padding: 15px;
}

/* 태블릿 (768px 이상) */
@media (min-width: 768px) {
  .container {
    padding: 30px;
  }
}

/* 데스크톱 (1024px 이상) */
@media (min-width: 1024px) {
  .container {
    max-width: 960px;
    margin: 0 auto;
  }
}

/* 큰 데스크톱 (1200px 이상) */
@media (min-width: 1200px) {
  .container {
    max-width: 1140px;
  }
}
```

### Bootstrap 기준

```css
/* Extra small devices (phones, 0~575px) */
/* 기본 스타일 */

/* Small devices (tablets, 576px 이상) */
@media (min-width: 576px) { }

/* Medium devices (desktops, 768px 이상) */
@media (min-width: 768px) { }

/* Large devices (large desktops, 992px 이상) */
@media (min-width: 992px) { }

/* Extra large devices (larger desktops, 1200px 이상) */
@media (min-width: 1200px) { }

/* XXL devices (1400px 이상) */
@media (min-width: 1400px) { }
```

### Tailwind CSS 기준

```css
/* sm: 640px */
@media (min-width: 640px) { }

/* md: 768px */
@media (min-width: 768px) { }

/* lg: 1024px */
@media (min-width: 1024px) { }

/* xl: 1280px */
@media (min-width: 1280px) { }

/* 2xl: 1536px */
@media (min-width: 1536px) { }
```

### 커스텀 Breakpoint

```css
/* 콘텐츠 기반 */
:root {
  --bp-mobile: 480px;
  --bp-tablet: 768px;
  --bp-desktop: 1024px;
  --bp-wide: 1440px;
}

@media (min-width: 480px) { /* 모바일 L */ }
@media (min-width: 768px) { /* 태블릿 */ }
@media (min-width: 1024px) { /* 데스크톱 */ }
@media (min-width: 1440px) { /* 와이드 */ }
```

## 7. Mobile-first vs Desktop-first

### Mobile-first (권장)

작은 화면부터 시작해서 큰 화면으로 확장합니다.

```css
/* 기본: 모바일 */
.container {
  width: 100%;
  padding: 15px;
}

.grid {
  display: block;
}

/* 태블릿 이상 */
@media (min-width: 768px) {
  .container {
    padding: 30px;
  }

  .grid {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
  }
}

/* 데스크톱 이상 */
@media (min-width: 1024px) {
  .container {
    max-width: 960px;
    margin: 0 auto;
  }

  .grid {
    grid-template-columns: repeat(3, 1fr);
  }
}
```

**장점:**
- 모바일 성능 우선
- 점진적 향상(Progressive Enhancement)
- 더 간단한 코드

### Desktop-first

큰 화면부터 시작해서 작은 화면으로 축소합니다.

```css
/* 기본: 데스크톱 */
.container {
  max-width: 1140px;
  margin: 0 auto;
  padding: 40px;
}

.grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 30px;
}

/* 태블릿 이하 */
@media (max-width: 1023px) {
  .container {
    padding: 30px;
  }

  .grid {
    grid-template-columns: repeat(2, 1fr);
    gap: 20px;
  }
}

/* 모바일 이하 */
@media (max-width: 767px) {
  .container {
    padding: 15px;
  }

  .grid {
    display: block;
  }
}
```

**단점:**
- 모바일에서 불필요한 코드 로드
- 코드 복잡도 증가

> **Tip**: 현대 웹 개발에서는 Mobile-first 접근을 권장합니다.
{: .prompt-tip }

## 8. 실전 활용 예제

### 반응형 네비게이션

```html
<nav class="navbar">
  <div class="nav-brand">Logo</div>
  <button class="nav-toggle">☰</button>
  <ul class="nav-menu">
    <li><a href="#">Home</a></li>
    <li><a href="#">About</a></li>
    <li><a href="#">Contact</a></li>
  </ul>
</nav>
```

```css
/* 모바일: 햄버거 메뉴 */
.navbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px 20px;
  background-color: #333;
}

.nav-toggle {
  display: block;
  background: none;
  border: none;
  color: white;
  font-size: 24px;
  cursor: pointer;
}

.nav-menu {
  display: none;
  position: absolute;
  top: 60px;
  left: 0;
  right: 0;
  background-color: #333;
  flex-direction: column;
}

.nav-menu.active {
  display: flex;
}

.nav-menu li {
  list-style: none;
}

.nav-menu a {
  display: block;
  padding: 15px 20px;
  color: white;
  text-decoration: none;
}

/* 태블릿 이상: 가로 메뉴 */
@media (min-width: 768px) {
  .nav-toggle {
    display: none;
  }

  .nav-menu {
    display: flex !important;
    position: static;
    flex-direction: row;
  }

  .nav-menu a {
    padding: 10px 20px;
  }
}
```

### 반응형 그리드

```html
<div class="grid">
  <div class="card">Card 1</div>
  <div class="card">Card 2</div>
  <div class="card">Card 3</div>
  <div class="card">Card 4</div>
</div>
```

```css
/* 모바일: 1열 */
.grid {
  display: grid;
  gap: 15px;
  padding: 15px;
}

.card {
  padding: 20px;
  background-color: white;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

/* 태블릿: 2열 */
@media (min-width: 768px) {
  .grid {
    grid-template-columns: repeat(2, 1fr);
    gap: 20px;
    padding: 30px;
  }
}

/* 데스크톱: 3열 */
@media (min-width: 1024px) {
  .grid {
    grid-template-columns: repeat(3, 1fr);
    gap: 30px;
    max-width: 1200px;
    margin: 0 auto;
  }
}

/* 큰 화면: 4열 */
@media (min-width: 1400px) {
  .grid {
    grid-template-columns: repeat(4, 1fr);
  }
}
```

### 반응형 타이포그래피

```css
/* 모바일 */
body {
  font-size: 14px;
  line-height: 1.5;
}

h1 {
  font-size: 24px;
  margin-bottom: 16px;
}

h2 {
  font-size: 20px;
  margin-bottom: 12px;
}

p {
  margin-bottom: 16px;
}

/* 태블릿 */
@media (min-width: 768px) {
  body {
    font-size: 16px;
    line-height: 1.6;
  }

  h1 {
    font-size: 32px;
    margin-bottom: 20px;
  }

  h2 {
    font-size: 24px;
    margin-bottom: 16px;
  }
}

/* 데스크톱 */
@media (min-width: 1024px) {
  body {
    font-size: 18px;
    line-height: 1.7;
  }

  h1 {
    font-size: 48px;
    margin-bottom: 24px;
  }

  h2 {
    font-size: 32px;
    margin-bottom: 20px;
  }
}
```

### 반응형 이미지

```html
<picture>
  <source media="(min-width: 1024px)" srcset="image-large.jpg">
  <source media="(min-width: 768px)" srcset="image-medium.jpg">
  <img src="image-small.jpg" alt="Responsive image">
</picture>
```

```css
img {
  max-width: 100%;
  height: auto;
  display: block;
}

/* 레티나 대응 */
@media (min-resolution: 2dppx) {
  .hero {
    background-image: url('hero@2x.jpg');
  }
}
```

### 다크 모드

```css
/* 라이트 모드 (기본) */
:root {
  --bg-color: white;
  --text-color: #333;
  --border-color: #ddd;
}

body {
  background-color: var(--bg-color);
  color: var(--text-color);
}

.card {
  background-color: var(--bg-color);
  border: 1px solid var(--border-color);
}

/* 다크 모드 */
@media (prefers-color-scheme: dark) {
  :root {
    --bg-color: #1a1a1a;
    --text-color: #e0e0e0;
    --border-color: #444;
  }
}
```

## 9. SCSS/Sass로 관리하기

### 변수로 관리

```scss
// _breakpoints.scss
$breakpoints: (
  'mobile': 480px,
  'tablet': 768px,
  'desktop': 1024px,
  'wide': 1440px
);

// Mixin 생성
@mixin respond-to($breakpoint) {
  @if map-has-key($breakpoints, $breakpoint) {
    @media (min-width: map-get($breakpoints, $breakpoint)) {
      @content;
    }
  }
}
```

### 사용 예제

```scss
// main.scss
@import 'breakpoints';

.container {
  width: 100%;
  padding: 15px;

  @include respond-to('tablet') {
    padding: 30px;
  }

  @include respond-to('desktop') {
    max-width: 960px;
    margin: 0 auto;
  }
}

.grid {
  display: grid;
  gap: 15px;

  @include respond-to('tablet') {
    grid-template-columns: repeat(2, 1fr);
    gap: 20px;
  }

  @include respond-to('desktop') {
    grid-template-columns: repeat(3, 1fr);
    gap: 30px;
  }
}
```

### 간단한 버전

```scss
// _mediaQueries.scss
$phone: "only screen and (max-width: 768px)";
$tablet: "screen and (min-width: 769px) and (max-width: 1023px)";
$desktop: "screen and (min-width: 1024px)";

// 사용
@import './mediaQueries';

.box {
  width: 100%;

  @media #{$tablet} {
    width: 50%;
  }

  @media #{$desktop} {
    width: 33.333%;
  }
}
```

## 10. Container Queries (미래)

컨테이너 쿼리는 부모 요소의 크기에 따라 스타일을 적용합니다.

```css
/* Container 설정 */
.sidebar {
  container-type: inline-size;
  container-name: sidebar;
}

/* Container Query */
@container sidebar (min-width: 400px) {
  .widget {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
  }
}
```

> **Info**: Container Queries는 최신 브라우저에서 지원됩니다.
{: .prompt-info }

## 11. Best Practices

### ✅ 좋은 예

```css
/* Mobile-first 접근 */
.element {
  /* 모바일 기본 */
}

@media (min-width: 768px) {
  .element {
    /* 태블릿 이상 */
  }
}

/* 의미 있는 Breakpoint */
@media (min-width: 768px) { /* 콘텐츠가 깨지는 지점 */ }

/* CSS 변수 활용 */
:root {
  --container-width: 100%;
}

@media (min-width: 1024px) {
  :root {
    --container-width: 960px;
  }
}

/* 접근성 고려 */
@media (prefers-reduced-motion: reduce) {
  * {
    animation: none !important;
  }
}
```

### ❌ 나쁜 예

```css
/* Desktop-first */
.element {
  width: 1200px;
}

@media (max-width: 768px) {
  .element {
    width: 100%;
  }
}

/* 너무 많은 Breakpoint */
@media (max-width: 320px) { }
@media (max-width: 375px) { }
@media (max-width: 425px) { }
@media (max-width: 768px) { }
/* ... */

/* 고정된 픽셀값 */
.box {
  width: 375px;  /* 특정 디바이스 크기에만 맞음 */
}

/* 인라인 Media Query (관리 어려움) */
<div style="width: 100%; @media (min-width: 768px) { width: 50%; }">
```

## 12. 디버깅 팁

### 개발자 도구 활용

1. **Device Mode**: Chrome DevTools에서 다양한 디바이스 시뮬레이션
2. **Responsive Design Mode**: 커스텀 화면 크기 테스트
3. **Elements > Computed**: 적용된 Media Query 확인

### 시각화

```css
/* 현재 Breakpoint 표시 */
.breakpoint-indicator {
  content: 'Mobile';
  position: fixed;
  top: 0;
  left: 0;
  background-color: red;
  color: white;
  padding: 5px 10px;
  z-index: 9999;
}

@media (min-width: 768px) {
  .breakpoint-indicator {
    content: 'Tablet';
    background-color: orange;
  }
}

@media (min-width: 1024px) {
  .breakpoint-indicator {
    content: 'Desktop';
    background-color: green;
  }
}
```

## 13. 성능 최적화

### Critical CSS

```html
<head>
  <style>
    /* 초기 렌더링에 필요한 CSS만 인라인 */
    body { margin: 0; }
    .header { height: 60px; }
  </style>

  <!-- 나머지 CSS는 비동기 로드 -->
  <link rel="stylesheet" href="styles.css" media="print" onload="this.media='all'">
</head>
```

### 조건부 로드

```html
<!-- 모바일용 CSS -->
<link rel="stylesheet" href="mobile.css" media="(max-width: 768px)">

<!-- 데스크톱용 CSS -->
<link rel="stylesheet" href="desktop.css" media="(min-width: 769px)">
```

## 정리

### Media Query 기본 문법

```css
@media [type] and ([feature]) {
  /* CSS */
}

/* 예제 */
@media screen and (min-width: 768px) {
  .container {
    max-width: 720px;
  }
}
```

### 주요 Breakpoint

| 디바이스 | 크기 | Media Query |
|---------|------|-------------|
| 모바일 | ~767px | 기본 스타일 |
| 태블릿 | 768px~ | `@media (min-width: 768px)` |
| 데스크톱 | 1024px~ | `@media (min-width: 1024px)` |
| 와이드 | 1440px~ | `@media (min-width: 1440px)` |

### Mobile-first 패턴

```css
/* 1. 기본 (모바일) */
.element {
  width: 100%;
}

/* 2. 태블릿 이상 */
@media (min-width: 768px) {
  .element {
    width: 50%;
  }
}

/* 3. 데스크톱 이상 */
@media (min-width: 1024px) {
  .element {
    width: 33.333%;
  }
}
```

### 주요 Media Features

```css
/* 크기 */
@media (min-width: 768px) { }
@media (max-width: 1023px) { }

/* 방향 */
@media (orientation: landscape) { }
@media (orientation: portrait) { }

/* 해상도 */
@media (min-resolution: 2dppx) { }

/* 다크 모드 */
@media (prefers-color-scheme: dark) { }

/* 애니메이션 축소 */
@media (prefers-reduced-motion: reduce) { }

/* 호버 지원 */
@media (hover: hover) { }
```

## 참고 자료

- [MDN - Using media queries](https://developer.mozilla.org/ko/docs/Web/CSS/Media_Queries/Using_media_queries)
- [MDN - @media](https://developer.mozilla.org/ko/docs/Web/CSS/@media)
- [CSS Tricks - A Complete Guide to CSS Media Queries](https://css-tricks.com/a-complete-guide-to-css-media-queries/)
- [Web.dev - Responsive web design basics](https://web.dev/responsive-web-design-basics/)
