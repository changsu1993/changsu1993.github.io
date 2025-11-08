---
title: CSS 입문 가이드 - 스타일시트의 기초
description: CSS의 개념과 HTML에 CSS를 적용하는 3가지 방법, 선택자의 종류와 기본 작성법을 학습합니다.
author: changsu
date: 2020-06-12 12:54:29 +0900
categories: [Web, CSS]
tags: [css, stylesheet, selector, inline-style, external-css]
---

## 1. CSS란?

**CSS(Cascading Style Sheets)**는 HTML에 디자인을 입히는 스타일시트 언어입니다.

- **HTML**: 구조와 내용
- **CSS**: 디자인과 레이아웃

```html
<!-- HTML: 구조 -->
<h1>제목</h1>
<p>내용</p>
```

```css
/* CSS: 디자인 */
h1 {
  color: blue;
  font-size: 32px;
}

p {
  color: gray;
  line-height: 1.6;
}
```

## 2. CSS 적용 방법

### 2.1 인라인 스타일 (Inline Style)

```html
<h1 style="color: red; font-size: 24px;">제목</h1>
```

**장점:**
- ✅ 빠르고 간단

**단점:**
- ❌ 가독성 저하
- ❌ 재사용 불가
- ❌ 유지보수 어려움

> **Warning**: 실무에서는 거의 사용하지 않습니다.
{: .prompt-warning }

### 2.2 내부 스타일시트 (Internal CSS)

```html
<!DOCTYPE html>
<html>
<head>
  <style>
    h2 {
      color: #123456;
      font-size: 20px;
    }

    p {
      line-height: 1.6;
    }
  </style>
</head>
<body>
  <h2>제목</h2>
  <p>내용</p>
</body>
</html>
```

**장점:**
- ✅ HTML과 같은 파일에서 관리
- ✅ 단일 페이지에 적합

**단점:**
- ❌ 여러 페이지에서 재사용 불가
- ❌ HTML과 CSS가 혼재

### 2.3 외부 스타일시트 (External CSS) ⭐️

**index.html:**
```html
<!DOCTYPE html>
<html>
<head>
  <link rel="stylesheet" type="text/css" href="style.css">
</head>
<body>
  <h1>제목</h1>
  <p>내용</p>
</body>
</html>
```

**style.css:**
```css
h1 {
  color: blue;
  font-size: 28px;
}

p {
  color: #333;
  line-height: 1.6;
}
```

**장점:**
- ✅ HTML과 CSS 분리 (관심사의 분리)
- ✅ 여러 페이지에서 재사용
- ✅ 유지보수 용이
- ✅ 캐싱으로 성능 향상

> **Tip**: 실무에서는 외부 스타일시트를 사용합니다.
{: .prompt-tip }

### link 태그 속성

```html
<link href="style.css" rel="stylesheet" type="text/css">
```

| 속성 | 설명 | 값 |
|------|------|-----|
| **href** | CSS 파일 경로 | `"style.css"` |
| **rel** | 관계 명시 | `"stylesheet"` (고정) |
| **type** | 파일 타입 | `"text/css"` (생략 가능) |

## 3. CSS 기본 문법

### 3.1 구조

```css
선택자 {
  속성: 값;
  속성: 값;
}
```

```css
p {
  color: red;
  font-size: 16px;
}
```

**구성 요소:**
- **선택자(Selector)**: `p` - 스타일을 적용할 대상
- **속성(Property)**: `color`, `font-size` - 변경할 스타일 종류
- **값(Value)**: `red`, `16px` - 속성의 값
- **선언(Declaration)**: `color: red;` - 속성과 값의 쌍

### 3.2 주석

```css
/* 한 줄 주석 */

/*
여러 줄
주석
*/

p {
  color: red;
}
```

## 4. CSS 선택자

### 4.1 태그 선택자

```css
/* 모든 p 태그에 적용 */
p {
  font-size: 12px;
  color: #333;
}

h1 {
  font-size: 32px;
  font-weight: bold;
}
```

```html
<p>이 단락은 12px입니다.</p>
<p>이 단락도 12px입니다.</p>
```

### 4.2 클래스 선택자

```css
/* .클래스명 */
.highlight {
  background-color: yellow;
}

.profile-detail {
  font-weight: bold;
  color: #007bff;
}
```

```html
<p class="highlight">강조된 텍스트</p>
<span class="profile-detail">프로필 상세</span>
```

### 4.3 아이디 선택자

```css
/* #아이디명 */
#header {
  background-color: #333;
  color: white;
  padding: 20px;
}

#profile {
  border: 1px solid black;
  text-align: center;
}
```

```html
<header id="header">헤더</header>
<div id="profile">프로필</div>
```

### 4.4 선택자 비교

| 선택자 | 문법 | 사용 | 예시 |
|--------|------|------|------|
| **태그** | `태그명` | 모든 해당 태그 | `p { }` |
| **클래스** | `.클래스명` | 여러 요소 | `.btn { }` |
| **아이디** | `#아이디명` | 고유 요소 | `#main { }` |

## 5. 실전 예제

### 5.1 완전한 웹페이지

**index.html:**
```html
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <title>내 웹사이트</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <header id="main-header">
    <h1>웹사이트 제목</h1>
  </header>

  <main class="content">
    <h2 class="section-title">섹션 제목</h2>
    <p>본문 내용입니다.</p>
    <p class="highlight">강조된 내용입니다.</p>
  </main>

  <footer id="main-footer">
    <p>&copy; 2024</p>
  </footer>
</body>
</html>
```

**style.css:**
```css
/* 전체 스타일 */
body {
  font-family: 'Noto Sans KR', sans-serif;
  margin: 0;
  padding: 0;
  line-height: 1.6;
}

/* 헤더 */
#main-header {
  background-color: #333;
  color: white;
  padding: 20px;
  text-align: center;
}

#main-header h1 {
  margin: 0;
}

/* 메인 콘텐츠 */
.content {
  max-width: 800px;
  margin: 40px auto;
  padding: 0 20px;
}

.section-title {
  color: #007bff;
  border-bottom: 2px solid #007bff;
  padding-bottom: 10px;
}

/* 강조 텍스트 */
.highlight {
  background-color: #fff3cd;
  padding: 10px;
  border-left: 4px solid #ffc107;
}

/* 푸터 */
#main-footer {
  background-color: #f8f9fa;
  text-align: center;
  padding: 20px;
  margin-top: 40px;
}
```

## 6. Best Practices

### ✅ 좋은 예

```css
/* 외부 CSS 파일 사용 */
/* 의미 있는 클래스명 */
.button-primary {
  background-color: #007bff;
  color: white;
  padding: 10px 20px;
}

.card-title {
  font-size: 20px;
  font-weight: bold;
}
```

### ❌ 나쁜 예

```html
<!-- 인라인 스타일 남용 -->
<p style="color: red; font-size: 16px; margin: 10px; padding: 5px;">...</p>

<!-- 의미 없는 클래스명 -->
<div class="a1">...</div>
<div class="box2">...</div>
```

## 7. 적용 방법 비교

| 방법 | 장점 | 단점 | 사용 시기 |
|------|------|------|----------|
| **인라인** | 빠름 | 유지보수 어려움 | ❌ 사용 지양 |
| **내부** | 한 파일에서 관리 | 재사용 불가 | 단일 페이지 |
| **외부** | 재사용, 유지보수 용이 | 파일 관리 필요 | ✅ 권장 |

## 정리

### CSS 적용 (권장)

```html
<!-- HTML -->
<link rel="stylesheet" href="style.css">
```

```css
/* style.css */
선택자 {
  속성: 값;
}
```

### 선택자 종류

```css
/* 태그 */
p { }

/* 클래스 */
.class-name { }

/* 아이디 */
#id-name { }
```

**핵심 원칙:**
1. **외부 CSS** 사용
2. **의미 있는 이름** 사용
3. **HTML과 CSS 분리**

## 참고 자료

- [MDN - CSS 기초](https://developer.mozilla.org/ko/docs/Learn/Getting_started_with_the_web/CSS_basics)
- [MDN - CSS 선택자](https://developer.mozilla.org/ko/docs/Web/CSS/CSS_Selectors)
- [W3Schools - CSS Tutorial](https://www.w3schools.com/css/)
