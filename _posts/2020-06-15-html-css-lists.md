---
title: HTML/CSS 리스트 완벽 가이드 - ul, ol, li 태그와 스타일링
description: HTML 리스트 태그(ul, ol, li)의 사용법과 CSS 스타일링, 가상 클래스 선택자를 활용한 고급 리스트 디자인 기법을 학습합니다.
author: changsu
date: 2020-06-15 10:37:18 +0900
categories: [Web, HTML]
tags: [html, css, list, ul, ol, li, list-style, pseudo-class, selector]
---

## 1. HTML 리스트 기본

### 1.1 리스트 태그의 종류

HTML에서 목록을 표현하는 방법은 두 가지입니다:

| 태그 | 의미 | 특징 |
|------|------|------|
| **`<ul>`** | Unordered List (순서 없는 목록) | 글머리 기호 (•) |
| **`<ol>`** | Ordered List (순서 있는 목록) | 숫자 (1, 2, 3...) |
| **`<li>`** | List Item (목록 항목) | ul/ol 내부에 사용 |

### 1.2 기본 문법

```html
<!-- 순서 있는 목록 (Ordered List) -->
<ol>
  <li>List</li>
  <li>Set</li>
  <li>HashMap</li>
  <li>Queue</li>
  <li>Stack</li>
</ol>

<!-- 순서 없는 목록 (Unordered List) -->
<ul>
  <li>HTML</li>
  <li>CSS</li>
  <li>JavaScript</li>
</ul>
```

**결과:**

```
1. List
2. Set
3. HashMap
4. Queue
5. Stack

• HTML
• CSS
• JavaScript
```

> **중요**: `<li>` 태그는 반드시 `<ul>` 또는 `<ol>` 태그 내부에 있어야 합니다.
{: .prompt-warning }

## 2. Ordered List (ol)

### 2.1 기본 사용법

```html
<ol>
  <li>아침 먹기</li>
  <li>양치하기</li>
  <li>출근하기</li>
</ol>
```

**결과:**
```
1. 아침 먹기
2. 양치하기
3. 출근하기
```

### 2.2 type 속성

```html
<!-- 숫자 (기본값) -->
<ol type="1">
  <li>첫 번째</li>
  <li>두 번째</li>
</ol>

<!-- 알파벳 대문자 -->
<ol type="A">
  <li>첫 번째</li>
  <li>두 번째</li>
</ol>

<!-- 알파벳 소문자 -->
<ol type="a">
  <li>첫 번째</li>
  <li>두 번째</li>
</ol>

<!-- 로마 숫자 대문자 -->
<ol type="I">
  <li>첫 번째</li>
  <li>두 번째</li>
</ol>

<!-- 로마 숫자 소문자 -->
<ol type="i">
  <li>첫 번째</li>
  <li>두 번째</li>
</ol>
```

### 2.3 start 속성

```html
<!-- 5부터 시작 -->
<ol start="5">
  <li>다섯 번째</li>
  <li>여섯 번째</li>
  <li>일곱 번째</li>
</ol>
```

**결과:**
```
5. 다섯 번째
6. 여섯 번째
7. 일곱 번째
```

### 2.4 reversed 속성

```html
<!-- 역순 -->
<ol reversed>
  <li>세 번째</li>
  <li>두 번째</li>
  <li>첫 번째</li>
</ol>
```

**결과:**
```
3. 세 번째
2. 두 번째
1. 첫 번째
```

## 3. Unordered List (ul)

### 3.1 기본 사용법

```html
<ul>
  <li>사과</li>
  <li>바나나</li>
  <li>오렌지</li>
</ul>
```

**결과:**
```
• 사과
• 바나나
• 오렌지
```

### 3.2 중첩 리스트

```html
<ul>
  <li>과일
    <ul>
      <li>사과</li>
      <li>바나나</li>
    </ul>
  </li>
  <li>채소
    <ul>
      <li>당근</li>
      <li>브로콜리</li>
    </ul>
  </li>
</ul>
```

**결과:**
```
• 과일
  ◦ 사과
  ◦ 바나나
• 채소
  ◦ 당근
  ◦ 브로콜리
```

## 4. CSS로 리스트 스타일링

### 4.1 list-style-type

```css
/* 글머리 기호 제거 */
ul {
  list-style-type: none;
}

/* 다양한 글머리 기호 */
ul.disc { list-style-type: disc; }      /* ● (기본값) */
ul.circle { list-style-type: circle; }  /* ○ */
ul.square { list-style-type: square; }  /* ■ */

/* 순서 목록 */
ol.decimal { list-style-type: decimal; }         /* 1, 2, 3... */
ol.lower-alpha { list-style-type: lower-alpha; } /* a, b, c... */
ol.upper-alpha { list-style-type: upper-alpha; } /* A, B, C... */
ol.lower-roman { list-style-type: lower-roman; } /* i, ii, iii... */
ol.upper-roman { list-style-type: upper-roman; } /* I, II, III... */
```

### 4.2 list-style 축약형

```css
ul {
  /* list-style: type position image; */
  list-style: square inside url('icon.png');
}

/* 개별 속성 */
ul {
  list-style-type: square;
  list-style-position: inside;
  list-style-image: url('icon.png');
}
```

### 4.3 브라우저 기본 스타일 제거

```css
ul, ol {
  list-style: none;  /* 글머리 기호 제거 */
  padding: 0;        /* 기본 padding 제거 */
  margin: 0;         /* 기본 margin 제거 */
}
```

## 5. 실전 스타일링 예제

### 5.1 커스텀 스타일 목록

```html
<ul class="custom-list">
  <li>항목 1</li>
  <li>항목 2</li>
  <li>항목 3</li>
</ul>
```

```css
.custom-list {
  list-style: none;
  padding: 0;
  border-left: 3px solid #007bff;
}

.custom-list li {
  padding: 10px 15px;
  border-bottom: 1px solid #e0e0e0;
}

.custom-list li:last-child {
  border-bottom: none;
}
```

### 5.2 체크리스트

```html
<ul class="checklist">
  <li>✅ 완료된 작업</li>
  <li>⬜ 진행 중인 작업</li>
  <li>⬜ 대기 중인 작업</li>
</ul>
```

```css
.checklist {
  list-style: none;
  padding: 0;
}

.checklist li {
  padding: 8px 0;
  font-size: 16px;
}
```

### 5.3 아이콘 리스트

```html
<ul class="icon-list">
  <li>HTML5</li>
  <li>CSS3</li>
  <li>JavaScript</li>
</ul>
```

```css
.icon-list {
  list-style: none;
  padding: 0;
}

.icon-list li {
  padding-left: 25px;
  position: relative;
  margin-bottom: 10px;
}

.icon-list li::before {
  content: "▶";
  position: absolute;
  left: 0;
  color: #007bff;
}
```

### 5.4 번호 스타일 커스텀

```html
<ol class="custom-counter">
  <li>첫 번째 항목</li>
  <li>두 번째 항목</li>
  <li>세 번째 항목</li>
</ol>
```

```css
.custom-counter {
  list-style: none;
  counter-reset: item;
  padding: 0;
}

.custom-counter li {
  counter-increment: item;
  margin-bottom: 10px;
  padding-left: 40px;
  position: relative;
}

.custom-counter li::before {
  content: counter(item);
  position: absolute;
  left: 0;
  top: 0;
  background-color: #007bff;
  color: white;
  width: 25px;
  height: 25px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
}
```

## 6. 가상 클래스 선택자

### 6.1 기본 가상 클래스

```css
/* 첫 번째 항목 */
li:first-child {
  font-weight: bold;
  color: #007bff;
}

/* 마지막 항목 */
li:last-child {
  border-bottom: none;
}
```

### 6.2 홀수/짝수 선택

```css
/* 홀수 번째 (1, 3, 5...) */
li:nth-child(odd) {
  background-color: #f9f9f9;
}

/* 짝수 번째 (2, 4, 6...) */
li:nth-child(even) {
  background-color: white;
}
```

### 6.3 n번째마다 선택

**nth-child() 패턴 예제:**

| 패턴 | 선택되는 요소 | 설명 |
|------|-------------|------|
| `nth-child(odd)` | 1, 3, 5, 7... | 홀수 번째 |
| `nth-child(even)` | 2, 4, 6, 8... | 짝수 번째 |
| `nth-child(3n)` | 3, 6, 9, 12... | 3의 배수 |
| `nth-child(3n+1)` | 1, 4, 7, 10... | 3으로 나눈 나머지가 1 |
| `nth-child(3n+2)` | 2, 5, 8, 11... | 3으로 나눈 나머지가 2 |

```css
/* 3줄마다 구분선 추가 */
li:nth-child(odd) {
  background-color: #f9f9f9;
}
```

### 6.4 범위 선택

**범위 선택 패턴:**

| 패턴 | 선택되는 요소 | 설명 |
|------|-------------|------|
| `nth-child(-n+3)` | 1, 2, 3 | 처음 3개 |
| `nth-child(n+4)` | 4, 5, 6, 7... | 4번째부터 끝까지 |
| `nth-child(n+3):nth-child(-n+7)` | 3, 4, 5, 6, 7 | 3번째부터 7번째까지 |

```css
/* 처음 몇 개 항목 강조 */
li:first-child {
  font-weight: bold;
}
```

### 6.5 타입 기반 선택자

```css
/* li 중에서 첫 번째 */
li:first-of-type {
  border-top: 2px solid #333;
}

/* li 중에서 마지막 */
li:last-of-type {
  border-bottom: 2px solid #333;
}
```

## 7. 실전 네비게이션 메뉴

### 7.1 수평 네비게이션

```html
<nav>
  <ul class="horizontal-nav">
    <li><a href="/">홈</a></li>
    <li><a href="/about">소개</a></li>
    <li><a href="/services">서비스</a></li>
    <li><a href="/contact">문의</a></li>
  </ul>
</nav>
```

```css
.horizontal-nav {
  list-style: none;
  padding: 0;
  margin: 0;
  display: flex;
  background-color: #333;
}

.horizontal-nav li {
  flex: 1;
}

.horizontal-nav a {
  display: block;
  padding: 15px 20px;
  text-decoration: none;
  color: white;
  text-align: center;
  transition: background-color 0.3s;
}

.horizontal-nav a:hover {
  background-color: #555;
}
```

### 7.2 수직 사이드바

```html
<aside>
  <ul class="sidebar-menu">
    <li><a href="/dashboard">대시보드</a></li>
    <li><a href="/profile">프로필</a></li>
    <li><a href="/settings">설정</a></li>
    <li><a href="/logout">로그아웃</a></li>
  </ul>
</aside>
```

```css
.sidebar-menu {
  list-style: none;
  padding: 0;
  margin: 0;
  background-color: #f8f9fa;
  border-radius: 8px;
}

.sidebar-menu li {
  border-bottom: 1px solid #e0e0e0;
}

.sidebar-menu li:last-child {
  border-bottom: none;
}

.sidebar-menu a {
  display: block;
  padding: 15px 20px;
  text-decoration: none;
  color: #333;
  transition: all 0.3s;
}

.sidebar-menu a:hover {
  background-color: #007bff;
  color: white;
  padding-left: 30px;
}
```

### 7.3 드롭다운 메뉴

```html
<ul class="dropdown-menu">
  <li>
    <a href="#">메뉴 1</a>
    <ul class="submenu">
      <li><a href="#">하위 메뉴 1-1</a></li>
      <li><a href="#">하위 메뉴 1-2</a></li>
    </ul>
  </li>
  <li>
    <a href="#">메뉴 2</a>
  </li>
</ul>
```

```css
.dropdown-menu {
  list-style: none;
  padding: 0;
  margin: 0;
  display: flex;
  background-color: #333;
}

.dropdown-menu > li {
  position: relative;
}

.dropdown-menu > li > a {
  display: block;
  padding: 15px 20px;
  color: white;
  text-decoration: none;
}

.submenu {
  display: none;
  position: absolute;
  top: 100%;
  left: 0;
  background-color: #444;
  min-width: 200px;
  list-style: none;
  padding: 0;
  margin: 0;
}

.dropdown-menu > li:hover .submenu {
  display: block;
}

.submenu a {
  display: block;
  padding: 10px 20px;
  color: white;
  text-decoration: none;
}

.submenu a:hover {
  background-color: #555;
}
```

## 8. 고급 리스트 패턴

### 8.1 타임라인

```html
<ul class="timeline">
  <li>
    <span class="date">2024.01</span>
    <span class="event">프로젝트 시작</span>
  </li>
  <li>
    <span class="date">2024.02</span>
    <span class="event">베타 출시</span>
  </li>
  <li>
    <span class="date">2024.03</span>
    <span class="event">정식 출시</span>
  </li>
</ul>
```

```css
.timeline {
  list-style: none;
  padding: 0;
  position: relative;
  padding-left: 30px;
}

.timeline::before {
  content: '';
  position: absolute;
  left: 0;
  top: 0;
  bottom: 0;
  width: 3px;
  background-color: #007bff;
}

.timeline li {
  position: relative;
  margin-bottom: 30px;
}

.timeline li::before {
  content: '';
  position: absolute;
  left: -35px;
  top: 5px;
  width: 12px;
  height: 12px;
  border-radius: 50%;
  background-color: #007bff;
  border: 3px solid white;
  box-shadow: 0 0 0 3px #007bff;
}

.timeline .date {
  display: block;
  font-weight: bold;
  color: #007bff;
  margin-bottom: 5px;
}

.timeline .event {
  display: block;
  color: #333;
}
```

### 8.2 스텝 인디케이터

```html
<ol class="steps">
  <li class="completed">회원가입</li>
  <li class="active">정보 입력</li>
  <li>결제</li>
  <li>완료</li>
</ol>
```

```css
.steps {
  list-style: none;
  padding: 0;
  display: flex;
  justify-content: space-between;
  counter-reset: step;
}

.steps li {
  flex: 1;
  text-align: center;
  position: relative;
  counter-increment: step;
  padding-top: 50px;
}

.steps li::before {
  content: counter(step);
  position: absolute;
  top: 0;
  left: 50%;
  transform: translateX(-50%);
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background-color: #e0e0e0;
  color: #999;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  font-size: 18px;
}

.steps li.completed::before {
  background-color: #28a745;
  color: white;
  content: "✓";
}

.steps li.active::before {
  background-color: #007bff;
  color: white;
}

.steps li:not(:last-child)::after {
  content: '';
  position: absolute;
  top: 20px;
  left: 50%;
  width: 100%;
  height: 2px;
  background-color: #e0e0e0;
  z-index: -1;
}

.steps li.completed::after {
  background-color: #28a745;
}
```

### 8.3 카드 리스트

```html
<ul class="card-list">
  <li class="card">
    <h3>카드 제목 1</h3>
    <p>카드 내용입니다.</p>
  </li>
  <li class="card">
    <h3>카드 제목 2</h3>
    <p>카드 내용입니다.</p>
  </li>
  <li class="card">
    <h3>카드 제목 3</h3>
    <p>카드 내용입니다.</p>
  </li>
</ul>
```

```css
.card-list {
  list-style: none;
  padding: 0;
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 20px;
}

.card {
  padding: 20px;
  background-color: white;
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  transition: transform 0.3s, box-shadow 0.3s;
}

.card:hover {
  transform: translateY(-5px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

.card h3 {
  margin-top: 0;
  color: #333;
}

.card p {
  color: #666;
  line-height: 1.6;
}
```

## 9. 접근성 고려사항

### 9.1 시맨틱 HTML

```html
<!-- ✅ 좋은 예: 의미 있는 순서 -->
<ol>
  <li>단계 1: 재료 준비</li>
  <li>단계 2: 조리</li>
  <li>단계 3: 완성</li>
</ol>

<!-- ✅ 좋은 예: 순서 없는 항목 -->
<ul>
  <li>사과</li>
  <li>바나나</li>
  <li>오렌지</li>
</ul>

<!-- ❌ 나쁜 예: div로 리스트 구현 -->
<div class="fake-list">
  <div>• 항목 1</div>
  <div>• 항목 2</div>
</div>
```

### 9.2 ARIA 레이블

```html
<nav aria-label="메인 네비게이션">
  <ul>
    <li><a href="/">홈</a></li>
    <li><a href="/about">소개</a></li>
  </ul>
</nav>
```

## 10. 주의사항

### 10.1 잘못된 사용

```html
<!-- ❌ 나쁜 예: li를 단독 사용 -->
<li>항목 1</li>
<li>항목 2</li>

<!-- ✅ 좋은 예: ul/ol로 감싸기 -->
<ul>
  <li>항목 1</li>
  <li>항목 2</li>
</ul>
```

### 10.2 중첩 구조

```html
<!-- ✅ 올바른 중첩 -->
<ul>
  <li>항목 1
    <ul>
      <li>하위 항목 1-1</li>
    </ul>
  </li>
</ul>

<!-- ❌ 잘못된 중첩 -->
<ul>
  <ul>
    <li>하위 항목</li>
  </ul>
</ul>
```

## 정리

### HTML 리스트 핵심

```html
<!-- 순서 있는 목록 -->
<ol>
  <li>항목</li>
</ol>

<!-- 순서 없는 목록 -->
<ul>
  <li>항목</li>
</ul>
```

### CSS 스타일링 핵심

```css
/* 기본 스타일 제거 */
ul {
  list-style: none;
  padding: 0;
  margin: 0;
}

/* 가상 클래스 활용 */
li:first-child { }
li:last-child { }
li:nth-child(even) { }
```

### 실무 팁

1. **기본 스타일 제거**: `list-style: none; padding: 0;`
2. **시맨틱 사용**: 순서가 중요하면 `<ol>`, 아니면 `<ul>`
3. **네비게이션**: 리스트로 구현 (SEO 및 접근성)
4. **가상 클래스**: `:first-child`, `:last-child`, `:nth-child()` 활용
5. **중첩 허용**: 하위 목록은 `<li>` 내부에 배치

> **핵심**: 리스트는 단순히 목록을 표시하는 것을 넘어 네비게이션, 메뉴, 카드 등 다양한 UI 패턴의 기반이 됩니다.
{: .prompt-tip }

## 참고 자료

- [MDN - ul 요소](https://developer.mozilla.org/ko/docs/Web/HTML/Element/ul)
- [MDN - ol 요소](https://developer.mozilla.org/ko/docs/Web/HTML/Element/ol)
- [MDN - li 요소](https://developer.mozilla.org/ko/docs/Web/HTML/Element/li)
- [MDN - list-style](https://developer.mozilla.org/ko/docs/Web/CSS/list-style)
- [CSS Tricks - Custom List Styles](https://css-tricks.com/almanac/properties/l/list-style/)
