---
title: CSS Border 완벽 가이드 - 테두리 스타일링
description: CSS border 속성으로 요소의 테두리를 꾸미는 방법을 학습합니다. border-width, border-style, border-color와 방향별 설정법을 다룹니다.
author: changsu
date: 2020-06-13 15:51:36 +0900
categories: [Web, CSS]
tags: [css, box-model]
---

## 1. Border 기본 개념

**Border**는 요소의 테두리를 꾸미는 CSS 속성입니다.

```css
.box {
  border: 5px solid red;
}
```

### 기본 문법

```css
border: [두께] [스타일] [색상];
```

**구성 요소:**
- **두께(width)**: `1px`, `2px`, `thin`, `medium`, `thick`
- **스타일(style)**: `solid`, `dashed`, `dotted` 등
- **색상(color)**: `red`, `#333`, `rgb(0,0,0)` 등

> **Tip**: Border 순서는 자유롭지만, 관례적으로 두께 → 스타일 → 색상 순서를 사용합니다.
{: .prompt-tip }

## 2. Border 단축 속성

### 한 줄로 작성

```css
p {
  border: 5px solid red;
}

span {
  border: 1px dotted #0000ff;
}

.card {
  border: 2px solid #ddd;
}
```

### 개별 속성으로 작성

```css
.box {
  border-width: 5px;
  border-style: solid;
  border-color: red;
}
```

## 3. border-style 종류

### 주요 스타일

```css
/* 실선 (가장 많이 사용) */
.solid {
  border: 3px solid black;
}

/* 점선 */
.dotted {
  border: 3px dotted black;
}

/* 파선 (대시) */
.dashed {
  border: 3px dashed black;
}

/* 이중선 */
.double {
  border: 5px double black;
}

/* 입체 효과 */
.groove {
  border: 5px groove gray;
}

.ridge {
  border: 5px ridge gray;
}

.inset {
  border: 5px inset gray;
}

.outset {
  border: 5px outset gray;
}

/* 숨김 */
.none {
  border: none;
}

.hidden {
  border: hidden;
}
```

### 스타일 비교표

| 스타일 | 설명 | 사용 빈도 |
|--------|------|-----------|
| **solid** | 실선 | ⭐️⭐️⭐️⭐️⭐️ |
| **dashed** | 파선 | ⭐️⭐️⭐️ |
| **dotted** | 점선 | ⭐️⭐️⭐️ |
| **double** | 이중선 | ⭐️⭐️ |
| **groove** | 3D 홈 | ⭐️ |
| **ridge** | 3D 융기 | ⭐️ |
| **inset** | 3D 안쪽 | ⭐️ |
| **outset** | 3D 바깥쪽 | ⭐️ |
| **none** | 테두리 없음 | ⭐️⭐️⭐️⭐️ |

> **Tip**: 실무에서는 대부분 `solid`를 사용합니다.
{: .prompt-tip }

## 4. 방향별 Border

### 개별 방향 지정

```css
.box {
  border-top: 4px solid red;
  border-right: 2px solid #666;
  border-bottom: 6px dashed darkviolet;
  border-left: 1px dotted #0ee;
}
```

### 실전 활용

```css
/* 하단 테두리만 */
.underline {
  border-bottom: 2px solid #007bff;
}

/* 좌측 강조선 */
.quote {
  border-left: 4px solid #28a745;
  padding-left: 20px;
}

/* 상단 테두리만 */
.section {
  border-top: 1px solid #ddd;
  padding-top: 20px;
}
```

## 5. Border 속성 분리

### border-width (두께)

```css
/* 한 값: 모든 방향 */
.box {
  border-width: 5px;
}

/* 두 값: 상하 / 좌우 */
.box {
  border-width: 5px 10px;
}

/* 네 값: 상 우 하 좌 */
.box {
  border-width: 1px 2px 3px 4px;
}

/* 개별 지정 */
.box {
  border-top-width: 5px;
  border-right-width: 3px;
  border-bottom-width: 5px;
  border-left-width: 3px;
}
```

### border-style

```css
/* 모든 방향 */
.box {
  border-style: solid;
}

/* 방향별 다르게 */
.box {
  border-style: solid dashed dotted double;
  /* 상: solid, 우: dashed, 하: dotted, 좌: double */
}

/* 개별 지정 */
.box {
  border-top-style: solid;
  border-bottom-style: dashed;
}
```

### border-color (색상)

```css
/* 모든 방향 */
.box {
  border-color: red;
}

/* 방향별 다르게 */
.box {
  border-color: red blue green yellow;
  /* 상: red, 우: blue, 하: green, 좌: yellow */
}

/* 개별 지정 */
.box {
  border-top-color: red;
  border-bottom-color: blue;
}
```

## 6. border를 이용한 밑줄

### text-decoration vs border-bottom

```css
/* ❌ text-decoration: 커스터마이징 어려움 */
.underline {
  text-decoration: underline;
  /* 두께, 색상, 간격 조정 제한적 */
}

/* ✅ border-bottom: 완벽한 커스터마이징 */
.custom-underline {
  border-bottom: 2px solid #007bff;
  padding-bottom: 5px;  /* 텍스트와 밑줄 간격 */
}
```

### 실전 예제

```css
/* 링크 호버 효과 */
a {
  text-decoration: none;
  color: #333;
  border-bottom: 2px solid transparent;
  transition: border-color 0.3s;
}

a:hover {
  border-bottom-color: #007bff;
}

/* 제목 밑줄 */
h2 {
  display: inline-block;
  border-bottom: 3px solid #28a745;
  padding-bottom: 10px;
}

/* 활성 탭 */
.tab.active {
  border-bottom: 3px solid #007bff;
  color: #007bff;
}
```

## 7. border-radius (둥근 모서리)

### 기본 사용법

```css
/* 모든 모서리 */
.rounded {
  border: 2px solid #ddd;
  border-radius: 10px;
}

/* 원 만들기 */
.circle {
  width: 100px;
  height: 100px;
  border: 2px solid #007bff;
  border-radius: 50%;
}

/* 캡슐 모양 */
.pill {
  border: 2px solid #28a745;
  border-radius: 25px;
  padding: 10px 20px;
}
```

### 개별 모서리 지정

```css
/* 네 값: 좌상 우상 우하 좌하 */
.box {
  border-radius: 10px 20px 30px 40px;
}

/* 개별 속성 */
.box {
  border-top-left-radius: 10px;
  border-top-right-radius: 20px;
  border-bottom-right-radius: 30px;
  border-bottom-left-radius: 40px;
}
```

### 실전 활용

```css
/* 카드 */
.card {
  border: 1px solid #ddd;
  border-radius: 8px;
  overflow: hidden;
}

/* 버튼 */
.btn {
  border: 2px solid #007bff;
  border-radius: 4px;
  padding: 10px 20px;
}

/* 프로필 이미지 */
.avatar {
  width: 80px;
  height: 80px;
  border: 3px solid white;
  border-radius: 50%;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}
```

## 8. 실전 활용 예제

### 카드 컴포넌트

```css
.card {
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  padding: 20px;
  background-color: white;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.card-header {
  border-bottom: 1px solid #e0e0e0;
  padding-bottom: 15px;
  margin-bottom: 15px;
}

.card-footer {
  border-top: 1px solid #e0e0e0;
  padding-top: 15px;
  margin-top: 15px;
}
```

### 인용문

```css
blockquote {
  border-left: 4px solid #007bff;
  padding-left: 20px;
  margin: 20px 0;
  color: #666;
  font-style: italic;
}

/* 강조 인용문 */
blockquote.highlight {
  border-left-color: #ffc107;
  background-color: #fff9e6;
  padding: 15px 20px;
}
```

### 알림창

```css
/* 성공 */
.alert-success {
  border: 1px solid #c3e6cb;
  border-left: 4px solid #28a745;
  background-color: #d4edda;
  padding: 15px;
  border-radius: 4px;
}

/* 경고 */
.alert-warning {
  border: 1px solid #ffeaa7;
  border-left: 4px solid #ffc107;
  background-color: #fff3cd;
  padding: 15px;
  border-radius: 4px;
}

/* 오류 */
.alert-danger {
  border: 1px solid #f5c6cb;
  border-left: 4px solid #dc3545;
  background-color: #f8d7da;
  padding: 15px;
  border-radius: 4px;
}
```

### 입력 필드

```css
input[type="text"],
textarea {
  border: 1px solid #ccc;
  border-radius: 4px;
  padding: 10px;
  transition: border-color 0.3s;
}

input:focus,
textarea:focus {
  border-color: #007bff;
  outline: none;
}

/* 에러 상태 */
input.error {
  border-color: #dc3545;
}

/* 성공 상태 */
input.success {
  border-color: #28a745;
}
```

### 네비게이션

```css
nav {
  border-bottom: 2px solid #e0e0e0;
}

nav a {
  display: inline-block;
  padding: 15px 20px;
  border-bottom: 3px solid transparent;
  transition: border-color 0.3s;
}

nav a:hover {
  border-bottom-color: #007bff;
}

nav a.active {
  border-bottom-color: #007bff;
  color: #007bff;
}
```

## 9. Border 조합 기법

### 그라데이션 테두리

```css
.gradient-border {
  border: 3px solid;
  border-image: linear-gradient(45deg, #007bff, #28a745) 1;
}
```

### 애니메이션 효과

```css
.animated-border {
  border: 2px solid #007bff;
  transition: all 0.3s;
}

.animated-border:hover {
  border-color: #28a745;
  border-width: 4px;
  box-shadow: 0 0 10px rgba(40, 167, 69, 0.5);
}
```

### 다중 테두리

```css
.multi-border {
  border: 2px solid #007bff;
  box-shadow:
    0 0 0 4px white,
    0 0 0 6px #28a745;
}
```

## 10. Best Practices

### ✅ 좋은 예

```css
/* 명확한 의도 */
.card {
  border: 1px solid #ddd;
  border-radius: 8px;
}

/* 방향별 테두리 활용 */
.section {
  border-bottom: 1px solid #e0e0e0;
}

/* 일관된 스타일 */
button {
  border: 2px solid #007bff;
  border-radius: 4px;
}
```

### ❌ 나쁜 예

```css
/* 불필요한 복잡성 */
.box {
  border-top: 4px double red;
  border-right: 2px solid #666;
  border-bottom: 6px dashed darkviolet;
  border-left: 1px dotted #0ee;
  /* 일관성 없는 디자인 */
}

/* 중복 작성 */
.box {
  border-width: 1px;
  border-style: solid;
  border-color: black;
  /* border: 1px solid black; 로 간단히 */
}
```

## 11. 반응형 Border

```css
/* 모바일 */
.card {
  border: 1px solid #ddd;
  border-radius: 4px;
}

/* 태블릿 */
@media (min-width: 768px) {
  .card {
    border-width: 2px;
    border-radius: 8px;
  }
}

/* 데스크톱 */
@media (min-width: 1024px) {
  .card {
    border-radius: 12px;
  }
}
```

## 정리

### 기본 문법

```css
.box {
  /* 단축 속성 */
  border: 2px solid #333;
}

.box {
  /* 개별 속성 */
  border-width: 2px;
  border-style: solid;
  border-color: #333;
}

.section {
  /* 방향별 */
  border-top: 1px solid #ddd;
  border-bottom: 1px solid #ddd;
}
```

### 주요 스타일

| 스타일 | 용도 |
|--------|------|
| `solid` | 일반 테두리 (가장 많이 사용) |
| `dashed` | 구분선, 점선 효과 |
| `dotted` | 점선 효과 |
| `none` | 테두리 제거 |

### border-radius

```css
.rounded {
  /* 둥근 모서리 */
  border-radius: 8px;
}

.circle {
  /* 원 */
  border-radius: 50%;
}

.pill {
  /* 캡슐 */
  border-radius: 25px;
}
```

### 방향 순서

<div style="max-width: 300px; margin: 20px auto; position: relative; border: 4px solid #333; padding: 60px 80px; background: #f8f9fa;">
  <div style="position: absolute; top: 10px; left: 50%; transform: translateX(-50%); font-weight: bold; color: #e74c3c; font-size: 18px;">
    ① top
  </div>
  <div style="position: absolute; right: 10px; top: 50%; transform: translateY(-50%); font-weight: bold; color: #3498db; font-size: 18px;">
    ② right
  </div>
  <div style="position: absolute; bottom: 10px; left: 50%; transform: translateX(-50%); font-weight: bold; color: #2ecc71; font-size: 18px;">
    ③ bottom
  </div>
  <div style="position: absolute; left: 10px; top: 50%; transform: translateY(-50%); font-weight: bold; color: #f39c12; font-size: 18px;">
    ④ left
  </div>
  <div style="text-align: center; color: #666; font-size: 14px;">
    시계방향 ↻
  </div>
</div>

```css
.box {
  /* border: top right bottom left; (시계방향) */
  border: 1px solid red;
}
```

## 참고 자료

- [MDN - border](https://developer.mozilla.org/ko/docs/Web/CSS/border)
- [MDN - border-style](https://developer.mozilla.org/ko/docs/Web/CSS/border-style)
- [MDN - border-radius](https://developer.mozilla.org/ko/docs/Web/CSS/border-radius)
- [CSS Tricks - Border](https://css-tricks.com/almanac/properties/b/border/)
