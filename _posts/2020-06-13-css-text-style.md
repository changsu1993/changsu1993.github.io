---
title: CSS 텍스트 스타일 가이드 - text-align, text-indent
description: CSS text-align으로 텍스트 정렬하는 방법과 text-indent로 들여쓰기하는 방법을 학습합니다.
author: changsu
date: 2020-06-13 01:06:36 +0900
categories: [Web, CSS]
tags: [css, typography]
---

## 1. text-align (텍스트 정렬)

### 기본 사용법

```css
.left {
  text-align: left;    /* 왼쪽 정렬 (기본값) */
}

.center {
  text-align: center;  /* 가운데 정렬 */
}

.right {
  text-align: right;   /* 오른쪽 정렬 */
}

.justify {
  text-align: justify; /* 양쪽 정렬 */
}
```

### 실전 예제

```html
<h1 class="center">제목 (가운데 정렬)</h1>
<p class="left">본문 (왼쪽 정렬)</p>
<p class="right">날짜 (오른쪽 정렬)</p>
```

```css
.center {
  text-align: center;
}

.left {
  text-align: left;
}

.right {
  text-align: right;
}
```

### Block vs Inline 요소

```css
/* ✅ Block 요소: 정렬 가능 */
div {
  text-align: center;  /* 작동함 */
}

p {
  text-align: right;   /* 작동함 */
}

/* ❌ Inline 요소: 정렬 안 됨 */
span {
  text-align: center;  /* 작동 안 함 */
}

/* ✅ 해결: 부모 요소에 적용 */
div {
  text-align: center;
}
```

> **Warning**: `text-align`은 Block 요소에만 적용됩니다. Inline 요소는 부모에게 적용하세요.
{: .prompt-warning }

## 2. text-indent (들여쓰기)

### 기본 사용법

```css
.js-description {
  text-indent: 50px;  /* 들여쓰기 */
}

.negative-indent {
  text-indent: -20px; /* 내어쓰기 */
}
```

### 단위

```css
/* 픽셀 */
p {
  text-indent: 30px;
}

/* em (상대 단위) */
p {
  text-indent: 2em;
}

/* 퍼센트 (부모 너비 기준) */
p {
  text-indent: 10%;
}
```

### 실전 예제

```html
<p class="paragraph">
  첫 번째 줄은 들여쓰기됩니다.
  두 번째 줄부터는 정상적으로 표시됩니다.
</p>
```

```css
.paragraph {
  text-indent: 2em;
  line-height: 1.8;
}
```

**결과:**
```
    첫 번째 줄은 들여쓰기됩니다.
두 번째 줄부터는 정상적으로 표시됩니다.
```

## 3. 추가 텍스트 스타일

### text-decoration

```css
a {
  text-decoration: none;           /* 밑줄 제거 */
}

.underline {
  text-decoration: underline;      /* 밑줄 */
}

.line-through {
  text-decoration: line-through;   /* 취소선 */
}

.overline {
  text-decoration: overline;       /* 윗줄 */
}
```

### text-transform

```css
.uppercase {
  text-transform: uppercase;  /* 대문자 */
}

.lowercase {
  text-transform: lowercase;  /* 소문자 */
}

.capitalize {
  text-transform: capitalize; /* 첫 글자만 대문자 */
}
```

### letter-spacing

```css
h1 {
  letter-spacing: 2px;   /* 글자 간격 */
}

.tight {
  letter-spacing: -1px;  /* 좁은 간격 */
}
```

### line-height

```css
p {
  line-height: 1.6;    /* 줄 간격 (권장) */
}

.double-space {
  line-height: 2;      /* 2배 간격 */
}
```

## 4. 실전 활용

### 본문 텍스트

```css
.article {
  text-align: left;
  text-indent: 2em;
  line-height: 1.8;
  color: #333;
}
```

### 제목 스타일

```css
h1 {
  text-align: center;
  letter-spacing: 2px;
  text-transform: uppercase;
}
```

### 인용문

```html
<blockquote>
  "인용문 내용"
</blockquote>
```

```css
blockquote {
  text-indent: 0;
  padding-left: 20px;
  border-left: 4px solid #007bff;
  font-style: italic;
  color: #666;
}
```

## 5. HTML 특수 문자

### 공백 추가

```html
<!-- ❌ HTML에서 공백은 하나만 인식 -->
<p>스페이스     여러개</p>
<!-- 결과: "스페이스 여러개" -->

<!-- ✅ &nbsp; 사용 -->
<p>스페이스&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;넣기</p>
<!-- 결과: "스페이스      넣기" -->
```

### 주요 특수 문자

| 문자 | HTML 코드 | 설명 |
|------|-----------|------|
| 공백 | `&nbsp;` | Non-breaking space |
| < | `&lt;` | Less than |
| > | `&gt;` | Greater than |
| & | `&amp;` | Ampersand |
| " | `&quot;` | Quotation mark |
| © | `&copy;` | Copyright |

> **Tip**: CSS로 공백을 조정하는 것이 더 좋습니다. `&nbsp;`는 최소한으로 사용하세요.
{: .prompt-tip }

## 6. 완전한 예제

```html
<!DOCTYPE html>
<html lang="ko">
<head>
  <style>
    body {
      font-family: 'Noto Sans KR', sans-serif;
      line-height: 1.6;
      max-width: 800px;
      margin: 0 auto;
      padding: 20px;
    }

    h1 {
      text-align: center;
      letter-spacing: 2px;
      color: #333;
    }

    .date {
      text-align: right;
      color: #999;
      font-size: 14px;
    }

    .article p {
      text-indent: 2em;
      text-align: justify;
      line-height: 1.8;
    }

    blockquote {
      padding-left: 20px;
      border-left: 4px solid #007bff;
      font-style: italic;
      color: #666;
      margin: 20px 0;
    }

    .author {
      text-align: right;
      font-style: italic;
      color: #666;
    }
  </style>
</head>
<body>
  <article>
    <h1>기사 제목</h1>
    <p class="date">2024.01.15</p>

    <div class="article">
      <p>첫 번째 단락입니다. 들여쓰기가 적용되어 있습니다.</p>
      <p>두 번째 단락입니다. 양쪽 정렬이 적용되어 있습니다.</p>

      <blockquote>
        "인용문은 이렇게 표시됩니다."
      </blockquote>

      <p>마지막 단락입니다.</p>
    </div>

    <p class="author">작성자: 홍길동</p>
  </article>
</body>
</html>
```

## 정리

### 주요 속성

```css
.text-style {
  /* 정렬 */
  text-align: center;

  /* 들여쓰기 (양수: 들여쓰기, 음수: 내어쓰기) */
  text-indent: 2em;

  /* 줄 간격 */
  line-height: 1.6;

  /* 글자 간격 */
  letter-spacing: 1px;

  /* 텍스트 꾸미기 */
  text-decoration: none;

  /* 대소문자 변환 */
  text-transform: uppercase;
}
```

### Best Practices

1. **본문**: `text-align: left`, `text-indent: 2em`
2. **제목**: `text-align: center`
3. **날짜/메타정보**: `text-align: right`
4. **줄 간격**: `line-height: 1.6` (가독성 향상)

## 참고 자료

- [MDN - text-align](https://developer.mozilla.org/ko/docs/Web/CSS/text-align)
- [MDN - text-indent](https://developer.mozilla.org/ko/docs/Web/CSS/text-indent)
- [MDN - HTML 특수문자](https://developer.mozilla.org/ko/docs/Glossary/Entity)
