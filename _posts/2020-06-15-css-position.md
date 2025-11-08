---
title: CSS Position 완벽 가이드 - 요소 배치의 모든 것
description: CSS position 속성으로 요소를 자유롭게 배치하는 방법을 학습합니다. static, relative, absolute, fixed, sticky의 차이점과 실전 활용법을 다룹니다.
author: changsu
date: 2020-06-15 12:30:23 +0900
categories: [Web, CSS]
tags: [css, position, layout, absolute, relative, fixed, sticky, z-index]
---

## 개요

CSS `position` 속성은 요소를 원하는 위치에 배치할 수 있게 해주는 강력한 기능입니다. HTML 작성 순서와 상관없이 요소를 자유롭게 배치할 수 있으며, 모달, 드롭다운, 고정 헤더 등 다양한 UI 패턴을 구현할 수 있습니다.

## 1. Position 기본 개념

### Position 값의 종류

```css
.element {
  /* 기본값 */
  position: static;
  /* 상대 위치 */
  position: relative;
  /* 절대 위치 */
  position: absolute;
  /* 고정 위치 */
  position: fixed;
  /* 스티키 위치 */
  position: sticky;
}
```

| 값 | 설명 | 사용 빈도 |
|---|------|-----------|
| **static** | 기본값, 일반적인 문서 흐름 | ⭐️ |
| **relative** | 원래 위치 기준 이동 | ⭐️⭐️⭐️⭐️ |
| **absolute** | 부모 기준 절대 위치 | ⭐️⭐️⭐️⭐️⭐️ |
| **fixed** | 뷰포트 기준 고정 | ⭐️⭐️⭐️⭐️ |
| **sticky** | 스크롤에 따라 고정 | ⭐️⭐️⭐️ |

## 2. position: static

### 기본값

모든 요소의 기본 position 값입니다.

```css
/* 기본값, 명시할 필요 없음 */
.static {
  position: static;
}
```

**특징:**
- 일반적인 문서 흐름을 따름
- `top`, `right`, `bottom`, `left` 속성 무시됨
- `z-index` 속성 무시됨
- 거의 명시적으로 사용하지 않음

```html
<div class="box">일반적인 박스</div>
<div class="box">문서 흐름대로 배치</div>
```

```css
.box {
  /* position: static; 이 기본값 */
  background-color: lightblue;
  padding: 20px;
  margin: 10px;
}
```

## 3. position: relative

### 상대적 위치

원래 위치를 기준으로 상대적으로 이동합니다.

```css
.relative {
  position: relative;
  /* 위에서 20px 아래로 */
  top: 20px;
  /* 왼쪽에서 30px 오른쪽으로 */
  left: 30px;
}
```

### 핵심 특징

1. **원래 공간 유지**: 이동해도 원래 공간은 그대로 유지됨
2. **다른 요소에 영향 없음**: 이동이 다른 요소 배치에 영향을 주지 않음
3. **absolute의 기준점**: 자식의 absolute 위치 지정 기준이 됨

### 실습 예제

```html
<div class="container">
  <div class="box">Box 1</div>
  <div class="box relative">Box 2 (Relative)</div>
  <div class="box">Box 3</div>
</div>
```

```css
.box {
  width: 100px;
  height: 100px;
  background-color: lightblue;
  border: 2px solid #333;
  margin: 10px;
}

.relative {
  position: relative;
  /* 음수: 위로 이동 */
  top: -20px;
  /* 양수: 오른쪽으로 이동 */
  left: 30px;
  background-color: lightcoral;
}
```

**결과:**
- Box 2가 위로 20px, 오른쪽으로 30px 이동
- Box 2의 원래 공간은 유지됨
- Box 3는 Box 2의 원래 위치 아래에 배치됨

### 좌표 시스템

```css
/* 음수와 양수의 방향 */
.element {
  position: relative;
}

/* 아래로 이동 */
.down {
  top: 10px;
}

/* 위로 이동 */
.up {
  top: -10px;
}

/* 오른쪽으로 이동 */
.right {
  left: 10px;
}

/* 왼쪽으로 이동 */
.left {
  left: -10px;
}
```

> **Tip**: `top`과 `bottom`, `left`와 `right`를 동시에 사용하면 `top`과 `left`가 우선됩니다.
{: .prompt-tip }

## 4. position: absolute

### 절대적 위치

가장 가까운 **positioned 부모**(position이 static이 아닌 부모)를 기준으로 절대 위치에 배치됩니다.

```css
/* 자식의 기준점 */
.parent {
  position: relative;
}

.child {
  position: absolute;
  top: 0;
  right: 0;
}
```

### 핵심 특징

1. **문서 흐름에서 제거**: 다른 요소가 absolute 요소를 무시함
2. **부모 기준 배치**: positioned 부모를 기준으로 위치 지정
3. **width/height 자동 축소**: block 요소도 내용 크기만큼만 차지

### Positioned 부모 찾기

```html
<div>
  <!-- position: static (기본값) -->
  <div class="parent">
    <!-- position: relative -->
    <div class="child">
      <!-- position: absolute -->
      부모(.parent)를 기준으로 배치됨
    </div>
  </div>
</div>
```

**우선순위:**
1. 부모 중 `position: relative` 찾기
2. 없으면 `position: absolute` 찾기
3. 없으면 `position: fixed` 찾기
4. 아무것도 없으면 `<html>` (viewport) 기준

### 실습 예제

```html
<div class="container">
  <div class="box absolute top-right">Top Right</div>
  <div class="box absolute bottom-left">Bottom Left</div>
  <div class="box absolute center">Center</div>
</div>
```

```css
.container {
  position: relative;  /* 자식들의 기준점 */
  width: 400px;
  height: 300px;
  border: 2px solid #333;
  margin: 50px auto;
}

.box {
  padding: 10px 15px;
  background-color: lightblue;
  border-radius: 4px;
}

.absolute {
  position: absolute;
}

.top-right {
  top: 10px;
  right: 10px;
}

.bottom-left {
  bottom: 10px;
  left: 10px;
}

.center {
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
```

### Width 동작 방식

```html
<div class="parent">
  <p class="normal">일반 블록 요소</p>
  <p class="absolute">Absolute 요소</p>
</div>
```

```css
.parent {
  position: relative;
  width: 400px;
  border: 2px solid #333;
}

.normal {
  background-color: yellow;
  /* width: 100% (부모 전체) */
}

.absolute {
  position: absolute;
  right: 0;
  bottom: 0;
  background-color: lightcoral;
  /* width: auto (내용 크기만큼) */
}
```

### 전체 너비 만들기

```css
/* 방법 1: width 지정 */
.absolute {
  position: absolute;
  width: 100%;
}

/* 방법 2: left와 right 동시 사용 */
.absolute {
  position: absolute;
  left: 0;
  right: 0;
  /* width가 자동으로 100%가 됨 */
}
```

## 5. position: fixed

### 뷰포트 기준 고정

스크롤과 상관없이 **뷰포트(브라우저 창)**를 기준으로 고정됩니다.

```css
.fixed {
  position: fixed;
  top: 0;
  left: 0;
}
```

### 핵심 특징

1. **뷰포트 기준**: 항상 화면의 같은 위치에 표시
2. **스크롤 무시**: 스크롤해도 위치 변하지 않음
3. **문서 흐름 제거**: absolute처럼 다른 요소에 영향 없음
4. **부모 무시**: 모든 부모의 position 무시

### 실습 예제

```html
<header class="fixed-header">
  <h1>고정 헤더</h1>
</header>

<main class="content">
  <p>긴 콘텐츠...</p>
  <!-- 스크롤 가능한 긴 내용 -->
</main>

<button class="scroll-to-top">
  ↑ Top
</button>
```

```css
/* 고정 헤더 */
.fixed-header {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  background-color: white;
  box-shadow: 0 2px 5px rgba(0,0,0,0.1);
  padding: 20px;
  z-index: 100;
}

/* 본문 (헤더만큼 위 여백 필요) */
.content {
  padding-top: 80px;  /* 헤더 높이만큼 */
}

/* 상단 이동 버튼 */
.scroll-to-top {
  position: fixed;
  bottom: 20px;
  right: 20px;
  width: 50px;
  height: 50px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 50%;
  cursor: pointer;
  box-shadow: 0 2px 10px rgba(0,0,0,0.2);
}
```

## 6. position: sticky

### 스크롤 위치에 따라 변하는 위치

일반적으로는 relative처럼 동작하다가, 스크롤 시 지정한 위치에 도달하면 fixed처럼 고정됩니다.

```css
.sticky {
  position: sticky;
  top: 0;  /* 상단에서 0px 위치에 고정 */
}
```

### 핵심 특징

1. **하이브리드**: relative + fixed의 조합
2. **부모 내에서만 유효**: 부모를 벗어나면 사라짐
3. **threshold 필요**: `top`, `bottom` 등 최소 하나 필요
4. **IE 미지원**: 브라우저 호환성 확인 필요

### 실습 예제

```html
<div class="container">
  <h2 class="sticky-header">섹션 1</h2>
  <p>내용...</p>
  <p>내용...</p>

  <h2 class="sticky-header">섹션 2</h2>
  <p>내용...</p>
  <p>내용...</p>
</div>
```

```css
.sticky-header {
  position: sticky;
  top: 0;
  background-color: white;
  padding: 15px;
  border-bottom: 2px solid #007bff;
  z-index: 10;
}

/* 각 섹션마다 스티키 헤더가 독립적으로 동작 */
```

### 테이블 헤더 고정

```html
<table>
  <thead>
    <tr>
      <th class="sticky-th">이름</th>
      <th class="sticky-th">이메일</th>
      <th class="sticky-th">역할</th>
    </tr>
  </thead>
  <tbody>
    <!-- 많은 데이터 행 -->
  </tbody>
</table>
```

```css
.sticky-th {
  position: sticky;
  top: 0;
  background-color: #f8f9fa;
  z-index: 10;
}
```

## 7. 좌표 속성 (top, right, bottom, left)

### 기본 사용법

```css
.positioned {
  position: absolute;

  /* 상단에서 거리 */
  top: 20px;

  /* 오른쪽에서 거리 */
  right: 30px;

  /* 하단에서 거리 */
  bottom: 40px;

  /* 왼쪽에서 거리 */
  left: 50px;
}
```

### 단위

```css
.element {
  /* 픽셀 */
  top: 20px;

  /* 퍼센트 (부모 기준) */
  top: 50%;

  /* rem, em */
  top: 2rem;

  /* 뷰포트 단위 */
  top: 10vh;

  /* calc() 함수 */
  top: calc(50% - 50px);
}
```

### 중앙 정렬

```css
/* 방법 1: transform 사용 (권장) */
.center {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

/* 방법 2: margin auto */
.center-alt {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  margin: auto;
  width: 200px;   /* 크기 지정 필요 */
  height: 100px;
}

/* 방법 3: calc() 사용 */
.center-calc {
  position: absolute;
  top: calc(50% - 50px);   /* 높이의 절반 */
  left: calc(50% - 100px); /* 너비의 절반 */
  width: 200px;
  height: 100px;
}
```

### 전체 영역 채우기

```css
/* 부모 전체를 덮는 오버레이 */
.overlay {
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  /* 또는 inset: 0; */
}
```

## 8. z-index와 Stacking Context

### z-index 기본

`z-index`는 요소의 쌓임 순서를 제어합니다.

```css
.element {
  position: relative;  /* position 필수 */
  z-index: 10;         /* 높을수록 위에 */
}
```

**특징:**
- **position 필수**: static이 아닌 position이 있어야 동작
- **정수 값**: 음수, 0, 양수 모두 가능
- **기본값**: auto (0과 동일)

### Stacking Order (쌓임 순서)

같은 stacking context 내에서의 순서:

1. **배경/테두리** (부모)
2. **음수 z-index**
3. **block 요소** (position: static)
4. **float 요소**
5. **inline 요소**
6. **z-index: 0 또는 auto**
7. **양수 z-index**

### 실습 예제

```html
<div class="container">
  <div class="box box-1">z-index: 1</div>
  <div class="box box-2">z-index: 2</div>
  <div class="box box-3">z-index: 3</div>
</div>
```

```css
.box {
  position: absolute;
  width: 100px;
  height: 100px;
  padding: 10px;
}

.box-1 {
  top: 50px;
  left: 50px;
  background-color: red;
  z-index: 1;
}

.box-2 {
  top: 80px;
  left: 80px;
  background-color: green;
  z-index: 2;
}

.box-3 {
  top: 110px;
  left: 110px;
  background-color: blue;
  z-index: 3;  /* 가장 위에 */
}
```

### Stacking Context 생성

다음 조건에서 새로운 stacking context가 생성됩니다:

```css
/* 1. position + z-index */
.context-1 {
  position: relative;
  z-index: 1;
}

/* 2. opacity < 1 */
.context-2 {
  opacity: 0.99;
}

/* 3. transform */
.context-3 {
  transform: translateX(0);
}

/* 4. filter */
.context-4 {
  filter: blur(1px);
}

/* 5. flex/grid item with z-index */
.parent {
  display: flex;
}
.context-5 {
  z-index: 1;
}
```

> **Warning**: 부모가 낮은 z-index를 가지면, 자식의 z-index를 아무리 높여도 다른 stacking context의 요소보다 위로 올릴 수 없습니다.
{: .prompt-warning }

## 9. 실전 활용 예제

### 모달 (Modal)

```html
<div class="modal-overlay">
  <div class="modal">
    <h2>모달 제목</h2>
    <p>모달 내용</p>
    <button class="close-btn">닫기</button>
  </div>
</div>
```

```css
/* 오버레이 (배경 어둡게) */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

/* 모달 */
.modal {
  position: relative;
  background-color: white;
  padding: 30px;
  border-radius: 8px;
  max-width: 500px;
  width: 90%;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.3);
  z-index: 1001;
}

/* 닫기 버튼 */
.close-btn {
  position: absolute;
  top: 10px;
  right: 10px;
  background: none;
  border: none;
  font-size: 24px;
  cursor: pointer;
}
```

### 툴팁 (Tooltip)

```html
<button class="tooltip-trigger">
  마우스를 올려보세요
  <span class="tooltip">툴팁 내용입니다!</span>
</button>
```

```css
.tooltip-trigger {
  position: relative;
  padding: 10px 20px;
}

.tooltip {
  position: absolute;
  bottom: 100%;  /* 버튼 위에 표시 */
  left: 50%;
  transform: translateX(-50%);
  background-color: #333;
  color: white;
  padding: 8px 12px;
  border-radius: 4px;
  white-space: nowrap;
  opacity: 0;
  visibility: hidden;
  transition: opacity 0.3s, visibility 0.3s;
  z-index: 10;
}

/* 화살표 */
.tooltip::after {
  content: '';
  position: absolute;
  top: 100%;
  left: 50%;
  transform: translateX(-50%);
  border: 5px solid transparent;
  border-top-color: #333;
}

.tooltip-trigger:hover .tooltip {
  opacity: 1;
  visibility: visible;
}
```

### 드롭다운 메뉴

```html
<nav class="navbar">
  <div class="dropdown">
    <button>메뉴</button>
    <div class="dropdown-menu">
      <a href="#">항목 1</a>
      <a href="#">항목 2</a>
      <a href="#">항목 3</a>
    </div>
  </div>
</nav>
```

```css
.navbar {
  background-color: #333;
  padding: 10px;
}

.dropdown {
  position: relative;
  display: inline-block;
}

.dropdown button {
  background-color: #007bff;
  color: white;
  padding: 10px 20px;
  border: none;
  cursor: pointer;
}

.dropdown-menu {
  position: absolute;
  top: 100%;
  left: 0;
  background-color: white;
  min-width: 200px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  border-radius: 4px;
  opacity: 0;
  visibility: hidden;
  transform: translateY(-10px);
  transition: all 0.3s;
  z-index: 100;
}

.dropdown:hover .dropdown-menu {
  opacity: 1;
  visibility: visible;
  transform: translateY(0);
}

.dropdown-menu a {
  display: block;
  padding: 12px 20px;
  color: #333;
  text-decoration: none;
  transition: background-color 0.2s;
}

.dropdown-menu a:hover {
  background-color: #f8f9fa;
}
```

### 고정 헤더 (Fixed Header)

```html
<header class="header">
  <nav>
    <h1>로고</h1>
    <ul>
      <li><a href="#">홈</a></li>
      <li><a href="#">소개</a></li>
      <li><a href="#">문의</a></li>
    </ul>
  </nav>
</header>

<main class="main-content">
  <!-- 본문 내용 -->
</main>
```

```css
.header {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  background-color: white;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
  z-index: 1000;
}

.header nav {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px 40px;
  max-width: 1200px;
  margin: 0 auto;
}

/* 헤더 높이만큼 본문에 여백 추가 */
.main-content {
  padding-top: 80px;
}
```

### 배지 (Badge)

```html
<button class="icon-button">
  <svg><!-- 아이콘 --></svg>
  <span class="badge">5</span>
</button>
```

```css
.icon-button {
  position: relative;
  padding: 10px;
  background: none;
  border: none;
  cursor: pointer;
}

.badge {
  position: absolute;
  top: 0;
  right: 0;
  background-color: red;
  color: white;
  font-size: 12px;
  padding: 2px 6px;
  border-radius: 10px;
  min-width: 20px;
  text-align: center;
}
```

### 이미지 캡션 오버레이

```html
<div class="image-container">
  <img src="image.jpg" alt="설명">
  <div class="caption">
    <h3>제목</h3>
    <p>설명</p>
  </div>
</div>
```

```css
.image-container {
  position: relative;
  overflow: hidden;
}

.image-container img {
  width: 100%;
  display: block;
}

.caption {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  background: linear-gradient(
    to top,
    rgba(0, 0, 0, 0.8),
    transparent
  );
  color: white;
  padding: 20px;
  transform: translateY(100%);
  transition: transform 0.3s;
}

.image-container:hover .caption {
  transform: translateY(0);
}
```

## 10. Best Practices

### ✅ 좋은 예

```css
/* absolute 사용 시 부모에 relative */
.parent {
  position: relative;
}

.child {
  position: absolute;
  top: 0;
  right: 0;
}

/* 중앙 정렬은 transform 사용 */
.center {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

/* z-index는 필요한 만큼만 */
.modal {
  z-index: 100;
}

.tooltip {
  z-index: 101;
}
```

### ❌ 나쁜 예

```css
/* position 없이 z-index 사용 */
.element {
  z-index: 999;  /* position이 없어서 동작 안 함 */
}

/* 과도한 z-index 값 */
.modal {
  z-index: 999999;  /* 필요 이상으로 큰 값 */
}

/* absolute 없이 top/left 사용 */
.element {
  top: 20px;  /* position이 없어서 무시됨 */
}

/* fixed 요소에 부모의 position 의존 */
.parent {
  position: relative;
}

.child {
  position: fixed;
  /* 부모의 position을 무시하므로 의미 없음 */
}
```

## 11. 성능 고려사항

### GPU 가속 활용

```css
/* transform을 함께 사용하면 GPU 가속 */
.smooth {
  position: absolute;
  transform: translateZ(0);  /* GPU 가속 활성화 */
}
```

### will-change 사용

```css
/* 애니메이션 예정인 요소 */
.animated {
  position: absolute;
  will-change: transform;
}

/* 애니메이션 완료 후 제거 */
.animated.done {
  will-change: auto;
}
```

> **Warning**: `will-change`를 너무 많은 요소에 사용하면 오히려 성능이 저하될 수 있습니다.
{: .prompt-warning }

### Reflow 최소화

```css
/* ❌ Reflow 유발 */
.element {
  position: absolute;
  top: 100px;
  /* JavaScript로 top 값을 자주 변경 */
}

/* ✅ Transform 사용 (Reflow 없음) */
.element {
  position: absolute;
  transform: translateY(100px);
  /* JavaScript로 transform 값 변경 */
}
```

## 12. 브라우저 호환성

### Position 지원

| 속성 | IE | Edge | Chrome | Firefox | Safari |
|------|-----|------|--------|---------|--------|
| static, relative, absolute, fixed | ✅ 전체 | ✅ | ✅ | ✅ | ✅ |
| sticky | ❌ | ✅ 16+ | ✅ 56+ | ✅ 32+ | ✅ 13+ |

### Sticky Fallback

```css
.sticky-header {
  position: sticky;
  top: 0;
}

/* IE 대응 */
@supports not (position: sticky) {
  .sticky-header {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
  }
}
```

## 13. 디버깅 팁

### 시각화

```css
/* 개발 중 position 요소 시각화 */
[style*="position: absolute"] {
  outline: 2px solid red !important;
}

[style*="position: fixed"] {
  outline: 2px solid blue !important;
}

[style*="position: relative"] {
  outline: 2px solid green !important;
}
```

### 개발자 도구 활용

1. **Elements 패널**: Computed 탭에서 실제 position 값 확인
2. **Layers 패널**: Stacking context 확인
3. **Layout 패널**: 요소 위치와 크기 확인

## 정리

### Position 선택 가이드

| 사용 목적 | 추천 Position |
|----------|--------------|
| 일반 배치 | `static` (기본) |
| 약간 이동 | `relative` |
| absolute의 기준점 | `relative` |
| 자유로운 배치 | `absolute` |
| 화면에 고정 | `fixed` |
| 스크롤 시 고정 | `sticky` |

### 주요 특징 비교

```css
/* static: 기본값 */
position: static;
/* - 일반적인 흐름
   - top/left 무시
   - z-index 무시 */

/* relative: 상대 위치 */
position: relative;
/* - 원래 공간 유지
   - 자신 기준 이동
   - absolute의 부모 */

/* absolute: 절대 위치 */
position: absolute;
/* - 문서 흐름 제거
   - 부모 기준 배치
   - width 자동 축소 */

/* fixed: 고정 위치 */
position: fixed;
/* - 뷰포트 기준
   - 스크롤 무시
   - 항상 같은 위치 */

/* sticky: 스티키 */
position: sticky;
/* - relative + fixed
   - 부모 내에서만
   - IE 미지원 */
```

### 좌표 속성

```css
/* 양수: 해당 방향에서 멀어짐 */
.positive {
  top: 20px;
  left: 30px;
}

/* 음수: 해당 방향으로 이동 */
.negative {
  top: -20px;
  left: -30px;
}
```

### Z-index 규칙

1. Position이 static이 아니어야 함
2. 같은 stacking context 내에서만 비교
3. 높을수록 위에 표시
4. 부모의 z-index가 낮으면 자식도 낮아짐

## 참고 자료

- [MDN - position](https://developer.mozilla.org/ko/docs/Web/CSS/position)
- [MDN - z-index](https://developer.mozilla.org/ko/docs/Web/CSS/z-index)
- [MDN - Stacking Context](https://developer.mozilla.org/ko/docs/Web/CSS/CSS_Positioning/Understanding_z_index/The_stacking_context)
- [CSS Tricks - Position](https://css-tricks.com/almanac/properties/p/position/)
