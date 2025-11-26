---
title: HTML 문서 구조 가이드 - DOCTYPE, head, body
description: HTML 문서의 기본 구조와 DOCTYPE, head, body 태그의 역할을 이해하고 올바른 HTML 작성법을 학습합니다.
author: changsu
date: 2020-06-12 01:30:33 +0900
categories: [Web, HTML]
tags: [html, structure]
---

## HTML 기본 구조

```html
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>페이지 제목</title>
</head>
<body>
  <!-- 콘텐츠 -->
</body>
</html>
```

## 1. DOCTYPE

```html
<!DOCTYPE html>
```

- HTML5 문서임을 선언
- 문서 첫 줄에 위치
- 브라우저에게 HTML 버전 알림

## 2. html 태그

```html
<html lang="ko">
  <!-- 모든 HTML 요소 -->
</html>
```

- 루트 요소
- `lang` 속성: 문서 언어 지정

## 3. head 태그

```html
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>페이지 제목</title>
  <link rel="stylesheet" href="style.css">
</head>
```

### 주요 요소

```html
<!-- 문자 인코딩 -->
<meta charset="UTF-8">

<!-- 반응형 -->
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<!-- 제목 -->
<title>페이지 제목</title>

<!-- CSS -->
<link rel="stylesheet" href="style.css">

<!-- JavaScript -->
<script src="script.js"></script>
```

## 4. body 태그

```html
<body>
  <h1>제목</h1>
  <p>단락</p>
  <span>텍스트</span>
</body>
```

### 주요 태그

| 태그 | 설명 | 특징 |
|------|------|------|
| `<h1>`~`<h6>` | 제목 | Block 요소 |
| `<p>` | 단락 | Block 요소 |
| `<span>` | 텍스트 | Inline 요소 |
| `<div>` | 컨테이너 | Block 요소 |

## 완전한 예제

```html
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta name="description" content="페이지 설명">
  <title>내 웹사이트</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <header>
    <h1>웹사이트 제목</h1>
  </header>
  <main>
    <h2>부제목</h2>
    <p>본문 내용입니다.</p>
  </main>
  <footer>
    <p>&copy; 2024</p>
  </footer>
</body>
</html>
```

## 정리

**필수 구조:**
```html
<!DOCTYPE html>
<html>
  <head><!-- 메타 정보 --></head>
  <body><!-- 콘텐츠 --></body>
</html>
```

## 참고 자료

- [MDN - HTML 문서 구조](https://developer.mozilla.org/ko/docs/Learn/HTML/Introduction_to_HTML/Document_and_website_structure)
