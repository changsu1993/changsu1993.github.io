---
title: CSS 폰트 스타일 가이드 - font-family, size, weight, color
description: CSS로 텍스트를 꾸미는 방법을 학습합니다. font-family, font-size, font-weight, font-style, color 속성의 사용법을 다룹니다.
author: changsu
date: 2020-06-12 13:47:51 +0900
categories: [Web, CSS]
tags: [css, font, font-family, font-size, font-weight, color, typography]
---

## 1. font-family

### 기본 사용법

```css
#title {
  font-family: Georgia, "Times New Roman", Times, serif;
}
```

**폰트 우선순위:**
1. Georgia (1순위)
2. "Times New Roman" (2순위)
3. Times (3순위)
4. serif (마지막 대체 폰트)

> **Tip**: 폰트명에 공백이 있으면 따옴표로 감싸야 합니다.
{: .prompt-tip }

### 실전 예제

```css
body {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "Noto Sans KR", sans-serif;
}

h1 {
  font-family: Georgia, serif;
}

code {
  font-family: "Monaco", "Courier New", monospace;
}
```

### 폰트 스택 (Font Stack)

| 용도 | 폰트 스택 |
|------|-----------|
| **본문** | `'Noto Sans KR', sans-serif` |
| **제목** | `Georgia, serif` |
| **코드** | `'Monaco', 'Courier New', monospace` |

## 2. font-size

### 단위

```css
.big-size-font {
  font-size: 50px;    /* 픽셀 (권장) */
}

.em-size {
  font-size: 2em;     /* 상대 크기 */
}

.rem-size {
  font-size: 1.5rem;  /* 루트 기준 상대 크기 */
}
```

### 단위 비교

| 단위 | 설명 | 예시 |
|------|------|------|
| **px** | 픽셀 (절대 단위) | `16px` |
| **em** | 부모 요소 기준 | `1.5em` |
| **rem** | 루트(html) 기준 | `1.5rem` |
| **%** | 부모 요소의 % | `120%` |

### 태그별 크기 조정

```css
/* 브라우저 기본값 무시하고 직접 지정 */
h1 {
  font-size: 30px;  /* h1이 작아짐 */
}

h4 {
  font-size: 50px;  /* h4가 커짐 */
}
```

## 3. font-weight

### 기본 사용법

```css
.bold-font {
  font-weight: bold;
}

.normal-font {
  font-weight: normal;
}
```

### 숫자 값

```css
.thin {
  font-weight: 100;    /* 가장 얇음 */
}

.normal {
  font-weight: 400;    /* normal과 동일 */
}

.bold {
  font-weight: 700;    /* bold와 동일 */
}

.heavy {
  font-weight: 900;    /* 가장 두꺼움 */
}
```

**값 비교:**
- `normal` = `400`
- `bold` = `700`

## 4. font-style

```css
a {
  font-style: italic;   /* 이탤릭체 */
}

p {
  font-style: normal;   /* 기본 (직립) */
}

em {
  font-style: oblique;  /* 기울임 */
}
```

## 5. color

### 색상 지정 방법

```css
/* 1. 색상 이름 */
.pink {
  color: pink;
}

/* 2. HEX 코드 (가장 많이 사용) */
h1 {
  color: #eb4639;
}

/* 3. RGB */
h1 {
  color: rgb(235, 70, 57);
}

/* 4. RGBA (투명도 포함) */
p {
  color: rgba(235, 70, 57, 0.8);  /* 80% 불투명 */
}

/* 5. HSL */
h1 {
  color: hsl(4, 82%, 57%);
}
```

### 색상 표현 비교

| 방법 | 예시 | 장점 |
|------|------|------|
| **이름** | `red`, `blue` | 간단 |
| **HEX** | `#eb4639` | 짧고 명확 ⭐️ |
| **RGB** | `rgb(235, 70, 57)` | 직관적 |
| **RGBA** | `rgba(235, 70, 57, 0.8)` | 투명도 가능 |
| **HSL** | `hsl(4, 82%, 57%)` | 색상 조정 쉬움 |

## 6. 실전 예제

### 기본 타이포그래피

```css
body {
  font-family: 'Noto Sans KR', sans-serif;
  font-size: 16px;
  font-weight: 400;
  color: #333;
  line-height: 1.6;
}

h1 {
  font-size: 32px;
  font-weight: 700;
  color: #1a1a1a;
  margin-bottom: 20px;
}

h2 {
  font-size: 24px;
  font-weight: 600;
  color: #2c2c2c;
}

.highlight {
  color: #007bff;
  font-weight: bold;
}

.muted {
  color: #6c757d;
  font-style: italic;
}
```

### 버튼 스타일

```css
.btn {
  font-family: inherit;
  font-size: 16px;
  font-weight: 600;
  color: white;
  background-color: #007bff;
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.btn:hover {
  background-color: #0056b3;
}
```

### 코드 블록

```css
code {
  font-family: 'Monaco', 'Courier New', monospace;
  font-size: 14px;
  color: #e83e8c;
  background-color: #f8f9fa;
  padding: 2px 6px;
  border-radius: 3px;
}

pre {
  font-family: 'Monaco', monospace;
  font-size: 14px;
  color: #333;
  background-color: #f5f5f5;
  padding: 15px;
  border-radius: 4px;
  overflow-x: auto;
}
```

## 7. 웹 폰트 사용하기

### Google Fonts

```html
<!-- HTML head에 추가 -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@400;700&display=swap" rel="stylesheet">
```

```css
body {
  font-family: 'Noto Sans KR', sans-serif;
}
```

### @font-face

```css
@font-face {
  font-family: 'MyFont';
  src: url('/fonts/myfont.woff2') format('woff2'),
       url('/fonts/myfont.woff') format('woff');
  font-weight: 400;
  font-style: normal;
}

body {
  font-family: 'MyFont', sans-serif;
}
```

## 8. Best Practices

```css
/* ✅ 좋은 예 */
body {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
  font-size: 16px;
  font-weight: 400;
  color: #333;
}

h1 {
  font-size: 2rem;
  font-weight: 700;
  color: #1a1a1a;
}

/* ❌ 나쁜 예 */
body {
  font-size: 10px;  /* 너무 작음 */
  color: #f0f0f0;   /* 배경과 대비 부족 */
}
```

## 9. 단축 속성

```css
/* 개별 속성 */
p {
  font-style: italic;
  font-weight: bold;
  font-size: 16px;
  font-family: Arial, sans-serif;
}

/* 단축 속성 */
p {
  font: italic bold 16px Arial, sans-serif;
  /* font: [style] [weight] size family */
}
```

## 정리

### 주요 속성

```css
/* 폰트 패밀리 */
font-family: 'Noto Sans KR', sans-serif;

/* 크기 */
font-size: 16px;

/* 두께 */
font-weight: 700;

/* 스타일 */
font-style: italic;

/* 색상 */
color: #333;
```

### 권장 단위

- **font-size**: `px` 또는 `rem`
- **font-weight**: `400` (normal), `700` (bold)
- **color**: `#hex` 또는 `rgb()`

### 색상 도구

- [Google Color Picker](https://www.google.com/search?q=color+picker)
- [Coolors.co](https://coolors.co/)
- [Adobe Color](https://color.adobe.com/)

## 참고 자료

- [MDN - CSS Fonts](https://developer.mozilla.org/ko/docs/Web/CSS/CSS_Fonts)
- [Google Fonts](https://fonts.google.com/)
- [Web Safe Fonts](https://www.w3schools.com/cssref/css_websafe_fonts.asp)
