---
title: CSS box-sizing 완벽 가이드 - border-box로 레이아웃 문제 해결하기
description: CSS box-sizing 속성의 개념과 content-box, border-box의 차이를 이해하고, 실무에서 필수적인 border-box 활용법을 학습합니다.
author: changsu
date: 2020-06-13 17:56:14 +0900
categories: [Web, CSS]
tags: [css, box-sizing, border-box, content-box, box-model, layout]
---

## 1. box-sizing이란?

### 1.1 문제 상황

CSS로 요소의 너비를 지정할 때, **눈으로 보이는 실제 너비**와 **코드에서 지정한 width 값**이 다른 경우가 발생합니다.

```css
.box {
  width: 200px;
  padding: 20px;
  border: 5px solid black;
}
```

위 코드의 실제 너비는?
- **250px** = width(200px) + padding-left(20px) + padding-right(20px) + border-left(5px) + border-right(5px)

이런 계산은 직관적이지 않고 레이아웃을 복잡하게 만듭니다.

### 1.2 box-sizing 속성

**box-sizing**은 요소의 너비와 높이를 계산하는 방법을 지정하는 CSS 속성입니다.

```css
.box {
  box-sizing: border-box;  /* 해결책! */
}
```

## 2. box-sizing 값

### 2.1 content-box (기본값)

**width/height가 콘텐츠 영역만 포함**

```css
.content-box {
  box-sizing: content-box;  /* 기본값 */
  width: 200px;
  padding: 20px;
  border: 5px solid black;
}
```

**실제 너비 계산:**
```
총 너비 = width + padding-left + padding-right + border-left + border-right
       = 200px + 20px + 20px + 5px + 5px
       = 250px
```

#### 시각적 표현

```
┌─────────────────────────────────────┐
│ border (5px)                        │
│  ┌─────────────────────────────┐   │
│  │ padding (20px)              │   │
│  │  ┌─────────────────────┐   │   │
│  │  │ content (200px)     │   │   │ ← width: 200px 적용 영역
│  │  └─────────────────────┘   │   │
│  └─────────────────────────────┘   │
└─────────────────────────────────────┘
      총 너비: 250px
```

### 2.2 border-box (권장)

**width/height가 border까지 포함**

```css
.border-box {
  box-sizing: border-box;
  width: 200px;
  padding: 20px;
  border: 5px solid black;
}
```

**실제 너비:**
```
총 너비 = width
       = 200px (padding과 border 포함)
```

#### 시각적 표현

```
┌─────────────────────────────────────┐
│ border (5px)                        │
│  ┌─────────────────────────────┐   │
│  │ padding (20px)              │   │
│  │  ┌─────────────────────┐   │   │
│  │  │ content (150px)     │   │   │
│  │  └─────────────────────┘   │   │
│  └─────────────────────────────┘   │
└─────────────────────────────────────┘
      총 너비: 200px ← width: 200px이 전체 너비
```

콘텐츠 영역은 자동으로 조정:
```
content width = width - padding-left - padding-right - border-left - border-right
              = 200px - 20px - 20px - 5px - 5px
              = 150px
```

## 3. content-box vs border-box 비교

### 3.1 실전 예제

```html
<div class="container">
  <div class="box content-box">content-box</div>
  <div class="box border-box">border-box</div>
</div>
```

```css
.container {
  width: 300px;
  background-color: #f0f0f0;
  padding: 10px;
}

.box {
  width: 300px;
  padding: 20px;
  border: 5px solid #007bff;
  margin-bottom: 10px;
  background-color: white;
}

.content-box {
  box-sizing: content-box;
  /* 실제 너비: 350px - 컨테이너를 벗어남! */
}

.border-box {
  box-sizing: border-box;
  /* 실제 너비: 300px - 딱 맞음! */
}
```

**결과:**
- `content-box`: 컨테이너를 50px 벗어남 (350px > 300px)
- `border-box`: 컨테이너에 딱 맞음 (300px = 300px)

### 3.2 비교표

| 속성 | content-box | border-box |
|------|-------------|------------|
| **width 적용 대상** | 콘텐츠만 | border까지 전체 |
| **실제 너비** | width + padding + border | width |
| **예측 가능성** | 어려움 | 쉬움 |
| **레이아웃 제어** | 복잡 | 간단 |
| **실무 사용** | 거의 사용 안 함 | 표준 |

## 4. 전역 적용 방법

### 4.1 모든 요소에 적용

대부분의 웹 프로젝트에서 **모든 요소**에 `border-box`를 적용합니다.

```css
* {
  box-sizing: border-box;
}
```

### 4.2 권장 방법 (상속 활용)

```css
html {
  box-sizing: border-box;
}

*, *::before, *::after {
  box-sizing: inherit;
}
```

**장점:**
- 상속을 통해 자식 요소에 자동 적용
- 특정 컴포넌트에서 `content-box`로 변경 가능
- 가상 요소(`::before`, `::after`)도 포함

### 4.3 CSS Reset에 포함

```css
/* Modern CSS Reset */
*, *::before, *::after {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
```

## 5. 실전 활용

### 5.1 그리드 레이아웃

```css
.grid {
  display: flex;
  gap: 20px;
}

.grid-item {
  box-sizing: border-box;
  width: 33.333%;  /* 3등분 */
  padding: 20px;
  border: 2px solid #ddd;
}
```

**border-box 없이:**
```
실제 너비 = 33.333% + 40px(padding) + 4px(border)
→ 100%를 초과하여 레이아웃 깨짐
```

**border-box 사용:**
```
실제 너비 = 33.333% (padding과 border 포함)
→ 정확히 3등분
```

### 5.2 입력 폼

```css
.form-group {
  margin-bottom: 20px;
}

input, textarea, select {
  box-sizing: border-box;
  width: 100%;  /* 부모 요소의 100% */
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
}
```

**장점:**
- `width: 100%`가 padding/border 포함
- 모든 입력 요소의 너비가 일치
- 레이아웃 계산이 간단

### 5.3 카드 컴포넌트

```css
.card {
  box-sizing: border-box;
  width: 300px;
  padding: 20px;
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.card-header {
  margin-bottom: 15px;
  padding-bottom: 15px;
  border-bottom: 1px solid #e0e0e0;
}

.card-content {
  line-height: 1.6;
}
```

카드의 실제 너비가 정확히 300px이므로 그리드 배치가 쉽습니다.

### 5.4 반응형 레이아웃

```css
.container {
  box-sizing: border-box;
  width: 100%;
  max-width: 1200px;
  padding: 0 20px;
  margin: 0 auto;
}

@media (max-width: 768px) {
  .container {
    padding: 0 15px;
  }
}
```

padding이 변경되어도 전체 너비는 유지됩니다.

## 6. border-box를 사용해야 하는 이유

### 6.1 직관적인 코딩

```css
/* ✅ border-box: 예상대로 동작 */
.sidebar {
  width: 250px;  /* 실제 너비: 250px */
  padding: 20px;
}

.main {
  width: calc(100% - 250px);  /* 정확한 계산 */
}
```

```css
/* ❌ content-box: 복잡한 계산 필요 */
.sidebar {
  width: 210px;  /* 250px - 40px(padding) */
  padding: 20px;  /* 실제 너비: 250px */
}

.main {
  width: calc(100% - 250px);
}
```

### 6.2 유지보수 용이

```css
/* padding 변경 시 */
.box {
  box-sizing: border-box;
  width: 300px;
  padding: 20px;  /* 30px로 변경해도 width는 300px 유지 */
}
```

`content-box`라면 padding 변경 시 width도 함께 수정해야 합니다.

### 6.3 퍼센트 기반 레이아웃

```css
.col-4 {
  box-sizing: border-box;
  width: 33.333%;
  padding: 15px;
  float: left;
}
```

padding이 있어도 정확히 3등분됩니다.

### 6.4 미디어 쿼리에서 안정적

```css
.container {
  box-sizing: border-box;
  width: 100%;
  padding: 20px;
}

@media (max-width: 768px) {
  .container {
    padding: 10px;  /* width는 여전히 100% */
  }
}
```

## 7. 주의사항

### 7.1 기존 프로젝트에 적용 시

```css
/* 전체 적용 전 테스트 필요 */
* {
  box-sizing: border-box;
}
```

이미 `content-box` 기반으로 작성된 레이아웃이 깨질 수 있습니다.

### 7.2 외부 라이브러리와의 충돌

```css
/* 특정 컴포넌트만 content-box 사용 */
.external-widget {
  box-sizing: content-box;
}

.external-widget * {
  box-sizing: content-box;
}
```

### 7.3 SVG 요소

```css
/* SVG는 box-sizing이 적용되지 않음 */
svg {
  /* box-sizing 무시됨 */
}
```

## 8. 실전 패턴

### 8.1 균등 분할 레이아웃

```css
.row {
  display: flex;
}

.col {
  box-sizing: border-box;
  flex: 1;
  padding: 15px;
  border: 1px solid #ddd;
}
```

```html
<div class="row">
  <div class="col">1</div>
  <div class="col">2</div>
  <div class="col">3</div>
</div>
```

### 8.2 고정 너비 + 유동 너비

```css
.layout {
  display: flex;
}

.sidebar {
  box-sizing: border-box;
  width: 250px;
  padding: 20px;
  background-color: #f5f5f5;
}

.content {
  box-sizing: border-box;
  flex: 1;
  padding: 20px;
}
```

### 8.3 중첩된 패딩

```css
.outer {
  box-sizing: border-box;
  width: 500px;
  padding: 20px;
  border: 2px solid #333;
}

.inner {
  box-sizing: border-box;
  width: 100%;  /* outer의 content 영역 100% */
  padding: 15px;
  background-color: #f0f0f0;
}
```

## 9. 브라우저 지원

### 9.1 호환성

| 브라우저 | 지원 버전 |
|---------|----------|
| Chrome | 10+ (완전 지원) |
| Firefox | 29+ (완전 지원) |
| Safari | 5.1+ (완전 지원) |
| Edge | 모든 버전 |
| IE | 8+ (부분 지원), 9+ (완전 지원) |

### 9.2 벤더 프리픽스 (레거시)

```css
/* IE 8+ 지원 필요 시 */
* {
  -webkit-box-sizing: border-box;  /* Safari 5-, Chrome 9- */
  -moz-box-sizing: border-box;     /* Firefox 28- */
  box-sizing: border-box;
}
```

현대 브라우저에서는 프리픽스 불필요합니다.

## 10. 디버깅 팁

### 10.1 개발자 도구에서 확인

Chrome DevTools에서:
1. 요소 선택
2. **Computed** 탭 확인
3. **Box Model** 다이어그램에서 실제 크기 확인

### 10.2 시각적 확인

```css
/* 임시 디버깅용 */
* {
  outline: 1px solid red;
}
```

요소의 실제 경계를 시각적으로 확인할 수 있습니다.

## 정리

### 핵심 개념

1. **content-box (기본값)**
   - `width` = 콘텐츠 영역만
   - 실제 너비 = width + padding + border
   - 직관적이지 않음

2. **border-box (권장)**
   - `width` = 전체 너비 (padding + border 포함)
   - 실제 너비 = width
   - 예측 가능하고 직관적

### 필수 설정

```css
/* 모든 프로젝트에 필수! */
*, *::before, *::after {
  box-sizing: border-box;
}
```

### 장점

- ✅ 직관적인 너비 계산
- ✅ 레이아웃 작성이 쉬움
- ✅ 반응형 디자인에 유리
- ✅ 유지보수 용이

> **중요**: `box-sizing: border-box`는 개인 프로젝트와 실무에서 **필수적으로 적용해야 하는 속성**입니다.
{: .prompt-warning }

## 참고 자료

- [MDN - box-sizing](https://developer.mozilla.org/ko/docs/Web/CSS/box-sizing)
- [CSS Tricks - box-sizing](https://css-tricks.com/box-sizing/)
- [W3C CSS Box Model](https://www.w3.org/TR/css-box-3/)
- [Paul Irish - box-sizing: border-box FTW](https://www.paulirish.com/2012/box-sizing-border-box-ftw/)
