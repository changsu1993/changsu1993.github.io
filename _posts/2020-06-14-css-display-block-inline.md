---
title: CSS Display 속성 완벽 가이드 - Block, Inline, Inline-block
description: CSS display 속성의 block, inline, inline-block, none 값의 차이와 활용법을 이해하고 레이아웃 제어 방법을 학습합니다.
author: changsu
date: 2020-06-14 19:32:13 +0900
categories: [Web, CSS]
tags: [css, display, block, inline, inline-block, layout, float]
---

## 1. Display 속성이란?

**display**는 요소가 화면에 어떻게 표시될지 결정하는 CSS 속성입니다.

```css
선택자 {
  display: block | inline | inline-block | none;
}
```

## 2. Block 요소

### 2.1 특징

**Block 요소**는 항상 새로운 줄에서 시작하며, 가로폭을 최대한 차지합니다.

```html
<div>첫 번째 블록</div>
<p>두 번째 블록</p>
<h1>세 번째 블록</h1>
```

**특징:**
- ✅ 새로운 줄에서 시작
- ✅ 가로폭 100% (부모 요소의 전체 너비)
- ✅ width, height 지정 가능
- ✅ margin, padding 모두 적용 가능
- ✅ 좌우에 다른 요소 배치 불가

### 2.2 대표적인 Block 요소

| 요소 | 설명 |
|------|------|
| `<div>` | 범용 컨테이너 |
| `<p>` | 단락 |
| `<h1>` ~ `<h6>` | 제목 |
| `<header>` | 헤더 |
| `<footer>` | 푸터 |
| `<section>` | 섹션 |
| `<article>` | 독립적인 콘텐츠 |
| `<ul>`, `<ol>`, `<li>` | 목록 |
| `<table>` | 표 |
| `<form>` | 폼 |

### 2.3 Block 요소 예제

```html
<div style="background: yellow;">
  DIV 블록 요소
</div>
<p style="background: lightblue;">
  P 블록 요소
</p>
<h1 style="background: lightgreen;">
  H1 블록 요소
</h1>
```

**결과:**
```
┌─────────────────────────────────┐
│ DIV 블록 요소                    │  ← 전체 너비 차지
└─────────────────────────────────┘
┌─────────────────────────────────┐
│ P 블록 요소                      │  ← 새로운 줄에서 시작
└─────────────────────────────────┘
┌─────────────────────────────────┐
│ H1 블록 요소                     │  ← 역시 새로운 줄
└─────────────────────────────────┘
```

### 2.4 Block 요소 CSS 제어

```css
.block-element {
  display: block;
  width: 500px;     /* 너비 지정 가능 */
  height: 100px;    /* 높이 지정 가능 */
  margin: 20px;     /* 모든 방향 margin */
  padding: 15px;    /* 모든 방향 padding */
  background-color: #f0f0f0;
}
```

## 3. Inline 요소

### 3.1 특징

**Inline 요소**는 콘텐츠 크기만큼만 공간을 차지하며, 한 줄에 여러 요소가 나란히 배치됩니다.

```html
<span>첫 번째</span> <span>두 번째</span> <span>세 번째</span>
```

**특징:**
- ✅ 콘텐츠 크기만큼만 차지
- ✅ 같은 줄에 나란히 배치
- ❌ width, height 지정 불가 (무시됨)
- ⚠️ margin: 좌우만 적용, 상하는 무시
- ⚠️ padding: 모두 적용되지만 상하는 줄 높이에 영향 없음

### 3.2 대표적인 Inline 요소

| 요소 | 설명 |
|------|------|
| `<span>` | 범용 인라인 컨테이너 |
| `<a>` | 링크 |
| `<img>` | 이미지 |
| `<strong>` | 강조 (굵게) |
| `<em>` | 강조 (기울임) |
| `<b>` | 굵게 |
| `<i>` | 기울임 |
| `<code>` | 코드 |
| `<label>` | 레이블 |
| `<button>` | 버튼 |

### 3.3 Inline 요소 예제

```html
<p>
  이것은 <span style="background: yellow;">span</span>과
  <strong style="background: lightblue;">strong</strong>과
  <a href="#" style="background: lightgreen;">링크</a>입니다.
</p>
```

**결과:**
```
이것은 span과 strong과 링크입니다.  ← 한 줄에 나란히
```

### 3.4 Inline 요소의 제약

```css
/* ❌ width, height 무시됨 */
span {
  display: inline;
  width: 200px;     /* 적용 안 됨 */
  height: 100px;    /* 적용 안 됨 */
}

/* ⚠️ margin 상하 무시됨 */
span {
  margin: 20px 10px;  /* 좌우만 적용 */
  margin-top: 20px;   /* 무시됨 */
  margin-bottom: 20px; /* 무시됨 */
}

/* ✅ padding은 적용되지만 레이아웃에 영향 없음 */
span {
  padding: 10px;  /* 시각적으로는 보이지만 다른 요소를 밀어내지 않음 */
}
```

## 4. Inline-block 요소

### 4.1 특징

**Inline-block**은 inline과 block의 장점을 결합한 속성입니다.

```css
.inline-block-element {
  display: inline-block;
}
```

**특징:**
- ✅ 같은 줄에 나란히 배치 (inline처럼)
- ✅ width, height 지정 가능 (block처럼)
- ✅ margin, padding 모두 적용 가능 (block처럼)
- ✅ 콘텐츠 크기 또는 지정한 크기만큼만 차지

### 4.2 Inline vs Inline-block 비교

```html
<!-- Inline -->
<span style="background: yellow; width: 200px; height: 100px;">
  Inline (크기 지정 안 됨)
</span>

<!-- Inline-block -->
<span style="display: inline-block; background: lightblue; width: 200px; height: 100px;">
  Inline-block (크기 지정 됨)
</span>
```

### 4.3 비교표

| 속성 | block | inline | inline-block |
|------|-------|--------|--------------|
| **새 줄 시작** | ✅ | ❌ | ❌ |
| **한 줄에 여러 요소** | ❌ | ✅ | ✅ |
| **width, height** | ✅ | ❌ | ✅ |
| **margin (모든 방향)** | ✅ | 좌우만 | ✅ |
| **padding (모든 방향)** | ✅ | 시각적으로만 | ✅ |
| **기본 너비** | 100% | 콘텐츠 | 콘텐츠 |

### 4.4 Inline-block 활용 예제

#### 수평 네비게이션

```html
<nav>
  <a href="/" class="nav-link">홈</a>
  <a href="/about" class="nav-link">소개</a>
  <a href="/services" class="nav-link">서비스</a>
  <a href="/contact" class="nav-link">문의</a>
</nav>
```

```css
.nav-link {
  display: inline-block;
  padding: 10px 20px;
  background-color: #007bff;
  color: white;
  text-decoration: none;
  margin-right: 5px;
}

.nav-link:hover {
  background-color: #0056b3;
}
```

#### 그리드 레이아웃 (Flexbox 이전 방식)

```html
<div class="grid">
  <div class="grid-item">1</div>
  <div class="grid-item">2</div>
  <div class="grid-item">3</div>
</div>
```

```css
.grid-item {
  display: inline-block;
  width: 30%;
  height: 100px;
  background-color: #f0f0f0;
  margin: 1%;
  vertical-align: top;  /* 상단 정렬 */
}
```

> **참고**: inline-block을 사용할 때 요소 사이에 공백이 생기는 문제가 있습니다. 이는 HTML의 줄바꿈이 공백으로 인식되기 때문입니다.
{: .prompt-info }

## 5. Display 값 변경하기

### 5.1 Block → Inline

```css
/* p 태그를 inline으로 */
p.inline-paragraph {
  display: inline;
}
```

```html
<p class="inline-paragraph">첫 번째</p>
<p class="inline-paragraph">두 번째</p>
<!-- 한 줄에 나란히 표시됨 -->
```

### 5.2 Inline → Block

```css
/* span 태그를 block으로 */
span.block-span {
  display: block;
}
```

```html
<span class="block-span">첫 번째</span>
<span class="block-span">두 번째</span>
<!-- 각각 새로운 줄에 표시됨 -->
```

### 5.3 실전 예제

```html
<ul class="horizontal-list">
  <li><a href="#">메뉴1</a></li>
  <li><a href="#">메뉴2</a></li>
  <li><a href="#">메뉴3</a></li>
</ul>
```

```css
/* li는 원래 block 요소 */
.horizontal-list li {
  display: inline-block;  /* inline-block으로 변경 */
  margin-right: 20px;
}

.horizontal-list a {
  text-decoration: none;
  padding: 10px 15px;
  background-color: #333;
  color: white;
}
```

## 6. display: none

### 6.1 요소 숨기기

```css
.hide {
  display: none;
}
```

**효과:**
- 요소가 화면에서 완전히 사라짐
- 공간도 차지하지 않음
- DOM에는 존재하지만 렌더링되지 않음

### 6.2 visibility: hidden과의 차이

```css
/* display: none - 공간도 차지 안 함 */
.hidden-display {
  display: none;
}

/* visibility: hidden - 공간은 차지함 */
.hidden-visibility {
  visibility: hidden;
}
```

**비교:**

```html
<div>요소 1</div>
<div class="hidden-display">display: none</div>
<div>요소 3</div>
```
**결과:**
```
요소 1
요소 3  ← display:none은 공간도 없음
```

```html
<div>요소 1</div>
<div class="hidden-visibility">visibility: hidden</div>
<div>요소 3</div>
```
**결과:**
```
요소 1
        ← 빈 공간 (공간은 차지함)
요소 3
```

### 6.3 실전 활용: 토글 메뉴

```html
<button onclick="toggleMenu()">메뉴 열기/닫기</button>
<nav id="menu" class="hidden">
  <a href="/">홈</a>
  <a href="/about">소개</a>
  <a href="/contact">문의</a>
</nav>
```

```css
.hidden {
  display: none;
}

.visible {
  display: block;
}

#menu a {
  display: block;
  padding: 10px;
  background-color: #f0f0f0;
  margin-bottom: 5px;
  text-decoration: none;
  color: #333;
}
```

```javascript
function toggleMenu() {
  const menu = document.getElementById('menu');
  menu.classList.toggle('hidden');
}
```

### 6.4 반응형 디자인에서 활용

```css
/* 모바일: 햄버거 메뉴 보임 */
.hamburger-menu {
  display: block;
}

.full-menu {
  display: none;
}

/* 데스크톱: 전체 메뉴 보임 */
@media (min-width: 768px) {
  .hamburger-menu {
    display: none;
  }

  .full-menu {
    display: block;
  }
}
```

## 7. Float를 이용한 레이아웃 (레거시)

### 7.1 Float 기본

```css
.float-left {
  float: left;
}

.float-right {
  float: right;
}
```

**효과:**
- 요소가 inline처럼 동작
- 다른 요소들이 주위에 배치됨

### 7.2 Float 예제

```html
<div class="container">
  <div class="sidebar">사이드바</div>
  <div class="content">메인 콘텐츠</div>
</div>
```

```css
.sidebar {
  float: left;
  width: 250px;
  background-color: #f0f0f0;
}

.content {
  margin-left: 270px;  /* sidebar width + gap */
}

/* clearfix */
.container::after {
  content: "";
  display: block;
  clear: both;
}
```

> **참고**: Float는 레거시 레이아웃 방식입니다. 현대적인 레이아웃은 Flexbox나 Grid를 사용하세요.
{: .prompt-warning }

## 8. 실전 패턴

### 8.1 카드 그리드

```html
<div class="card-grid">
  <div class="card">카드 1</div>
  <div class="card">카드 2</div>
  <div class="card">카드 3</div>
</div>
```

```css
.card {
  display: inline-block;
  width: 300px;
  height: 200px;
  margin: 10px;
  padding: 20px;
  background-color: white;
  border: 1px solid #ddd;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  vertical-align: top;
}
```

### 8.2 버튼 그룹

```html
<div class="button-group">
  <button>저장</button>
  <button>취소</button>
  <button>삭제</button>
</div>
```

```css
.button-group button {
  display: inline-block;
  padding: 10px 20px;
  margin-right: 5px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.button-group button:hover {
  background-color: #0056b3;
}
```

### 8.3 이미지 + 텍스트

```html
<div class="profile">
  <img src="avatar.jpg" alt="프로필" class="avatar">
  <div class="info">
    <h3>홍길동</h3>
    <p>개발자</p>
  </div>
</div>
```

```css
.avatar {
  display: inline-block;
  width: 60px;
  height: 60px;
  border-radius: 50%;
  vertical-align: middle;
}

.info {
  display: inline-block;
  margin-left: 15px;
  vertical-align: middle;
}
```

### 8.4 조건부 표시

```html
<div class="notification success">
  ✅ 저장되었습니다!
</div>

<div class="notification error" style="display: none;">
  ❌ 오류가 발생했습니다.
</div>
```

```css
.notification {
  padding: 15px;
  margin-bottom: 10px;
  border-radius: 4px;
}

.notification.success {
  background-color: #d4edda;
  color: #155724;
}

.notification.error {
  background-color: #f8d7da;
  color: #721c24;
}
```

## 9. 모던 레이아웃 대안

### 9.1 Flexbox (권장)

```css
/* inline-block 대신 Flexbox */
.container {
  display: flex;
  gap: 20px;
}

.item {
  flex: 1;
}
```

### 9.2 Grid (권장)

```css
/* float 대신 Grid */
.container {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 20px;
}
```

> **중요**: 새로운 프로젝트에서는 Flexbox나 Grid를 사용하세요. Display: block/inline/inline-block은 기본 개념 이해를 위해 필수적이지만, 복잡한 레이아웃은 Flexbox/Grid가 더 효율적입니다.
{: .prompt-tip }

## 10. 디버깅 팁

### 10.1 요소 경계 시각화

```css
/* 디버깅용 */
* {
  outline: 1px solid red;
}

/* 또는 */
.debug {
  background-color: rgba(255, 0, 0, 0.1);
}
```

### 10.2 개발자 도구 활용

Chrome DevTools에서:
1. 요소 선택
2. **Computed** 탭 확인
3. **Display** 값 확인
4. **Box Model** 다이어그램 확인

## 정리

### Display 값 요약

| 값 | 새 줄 | 한 줄 배치 | width/height | margin/padding |
|----|-------|----------|-------------|----------------|
| **block** | ✅ | ❌ | ✅ | ✅ 전체 |
| **inline** | ❌ | ✅ | ❌ | 좌우만 |
| **inline-block** | ❌ | ✅ | ✅ | ✅ 전체 |
| **none** | 숨김 | - | - | - |

### 언제 무엇을 사용할까?

```css
/* Block: 섹션, 컨테이너 */
display: block;

/* Inline: 텍스트 내 강조 */
display: inline;

/* Inline-block: 버튼, 네비게이션 항목 */
display: inline-block;

/* None: 조건부 표시/숨김 */
display: none;

/* 모던: Flexbox (권장) */
display: flex;

/* 모던: Grid (권장) */
display: grid;
```

### 핵심 원칙

1. **Block**: 전체 너비 차지, 새 줄 시작
2. **Inline**: 콘텐츠 크기만큼, 한 줄에 배치
3. **Inline-block**: 둘의 장점 결합
4. **Display 변경**: CSS로 요소의 성질 변경 가능
5. **모던 레이아웃**: Flexbox/Grid 사용 권장

> **참고**: Display 속성은 CSS 레이아웃의 기초입니다. Block, Inline, Inline-block의 차이를 이해하면 더 복잡한 Flexbox와 Grid를 쉽게 배울 수 있습니다.
{: .prompt-info }

## 참고 자료

- [MDN - display](https://developer.mozilla.org/ko/docs/Web/CSS/display)
- [MDN - Block-level elements](https://developer.mozilla.org/ko/docs/Web/HTML/Block-level_elements)
- [MDN - Inline elements](https://developer.mozilla.org/ko/docs/Web/HTML/Inline_elements)
- [CSS Tricks - display](https://css-tricks.com/almanac/properties/d/display/)
