---
title: HTML Attributes 가이드 - id와 class 속성
description: HTML 속성(Attributes)의 개념과 id, class 속성의 차이점 및 올바른 사용법을 학습합니다.
author: changsu
date: 2020-06-12 12:16:57 +0900
categories: [Web, HTML]
tags: [html, attributes]
---

## HTML Attributes란?

**속성(Attributes)**은 HTML 태그에 추가 정보를 제공합니다.

```html
<태그 속성="값">내용</태그>
```

## 1. id 속성

### 특징
- **고유한 식별자** (페이지 내 단 하나만 존재)
- 주민등록번호처럼 유일한 값
- CSS나 JavaScript에서 특정 요소 선택 시 사용

### 사용법

```html
<div id="header">헤더</div>
<p id="intro">소개 문단</p>

<!-- ❌ 잘못된 예: id 중복 -->
<div id="main">첫 번째</div>
<div id="main">두 번째</div>  <!-- 중복 불가! -->
```

### CSS에서 사용

```css
#header {
  background-color: #333;
  color: white;
}

#intro {
  font-size: 18px;
}
```

## 2. class 속성

### 특징
- **재사용 가능한 이름**
- 여러 요소에 같은 class 사용 가능
- 하나의 요소에 여러 class 적용 가능

### 사용법

```html
<!-- ✅ 같은 class를 여러 요소에 사용 -->
<div class="card">카드 1</div>
<div class="card">카드 2</div>
<p class="card">카드 3</p>

<!-- ✅ 여러 class 사용 (공백으로 구분) -->
<div class="card highlight">강조된 카드</div>
```

### CSS에서 사용

```css
.card {
  padding: 20px;
  border: 1px solid #ddd;
}

.highlight {
  background-color: yellow;
}
```

## 3. id vs class 비교

| 구분 | id | class |
|------|----|----|
| **중복** | ❌ 불가 (유일) | ✅ 가능 (재사용) |
| **선택자** | `#id` | `.class` |
| **명시도** | 100점 | 10점 |
| **용도** | 고유 요소 | 그룹 스타일링 |
| **JavaScript** | `getElementById()` | `getElementsByClassName()` |

## 4. 여러 속성 사용하기

```html
<!-- 공백으로 구분하여 여러 속성 추가 -->
<div id="main" class="container">
  메인 컨테이너
</div>

<img
  src="./photo.jpg"
  alt="사진"
  class="thumbnail"
  id="hero-image"
>

<!-- 속성 순서는 상관없음 -->
<a href="/" class="nav-link" id="home">홈</a>
<a id="about" class="nav-link" href="/about">소개</a>
```

## 5. 실전 예제

```html
<!DOCTYPE html>
<html lang="ko">
<head>
  <style>
    /* id 선택자 */
    #header {
      background-color: #333;
      color: white;
      padding: 20px;
    }

    /* class 선택자 */
    .btn {
      padding: 10px 20px;
      border-radius: 4px;
    }

    .btn-primary {
      background-color: #007bff;
      color: white;
    }

    .btn-secondary {
      background-color: #6c757d;
      color: white;
    }
  </style>
</head>
<body>
  <!-- id: 고유한 요소 -->
  <header id="header">
    <h1>웹사이트</h1>
  </header>

  <!-- class: 재사용 -->
  <button class="btn btn-primary">저장</button>
  <button class="btn btn-secondary">취소</button>
  <button class="btn btn-primary">확인</button>
</body>
</html>
```

## 6. Best Practices

### ✅ 좋은 예

```html
<!-- id: 고유한 레이아웃 요소 -->
<header id="main-header">...</header>
<nav id="primary-nav">...</nav>
<main id="content">...</main>

<!-- class: 재사용 가능한 컴포넌트 -->
<div class="card">...</div>
<div class="card featured">...</div>
<div class="card">...</div>
```

### ❌ 나쁜 예

```html
<!-- id 중복 사용 -->
<div id="box">첫 번째</div>
<div id="box">두 번째</div>

<!-- 불필요한 id 남용 -->
<p id="text1">...</p>
<p id="text2">...</p>
<p id="text3">...</p>
<!-- class를 사용하는 것이 적절 -->
```

## 7. 명명 규칙

```html
<!-- ✅ 좋은 예: kebab-case (HTML/CSS 관례) -->
<div id="main-content" class="product-card">

<!-- ✅ 좋은 예: camelCase (JavaScript 관례) -->
<div id="mainContent" class="productCard">

<!-- ❌ 나쁜 예: 공백 사용 -->
<div id="main content" class="product card">

<!-- ❌ 나쁜 예: 숫자로 시작 -->
<div id="1content" class="2card">
```

## 정리

### 핵심 개념

```html
<!-- id: 고유 식별자 -->
<div id="unique-element">

<!-- class: 재사용 가능 -->
<div class="reusable-style">

<!-- 여러 속성 -->
<div id="main" class="container active">
```

### 선택 가이드

- **id 사용**: 페이지에서 단 하나만 존재하는 요소
- **class 사용**: 같은 스타일을 여러 요소에 적용
- **실무 권장**: class를 주로 사용, id는 최소화

## 참고 자료

- [MDN - HTML Attributes](https://developer.mozilla.org/ko/docs/Web/HTML/Attributes)
- [MDN - id 속성](https://developer.mozilla.org/ko/docs/Web/HTML/Global_attributes/id)
- [MDN - class 속성](https://developer.mozilla.org/ko/docs/Web/HTML/Global_attributes/class)
