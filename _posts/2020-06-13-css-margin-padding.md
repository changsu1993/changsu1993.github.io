---
title: CSS Margin과 Padding 완벽 가이드 - 박스 모델의 이해
description: CSS Box Model의 핵심인 margin과 padding의 차이점을 이해하고, 여백을 효과적으로 조정하는 방법을 학습합니다.
author: changsu
date: 2020-06-13 15:26:11 +0900
categories: [Web, CSS]
tags: [css, margin, padding, box-model, spacing, layout]
---

## 1. CSS Box Model

모든 HTML 요소는 박스 모델(Box Model)을 따릅니다.

```
┌─────────────────────────────────────┐
│           Margin (외부 여백)          │
│  ┌─────────────────────────────────┐ │
│  │         Border (테두리)          │ │
│  │  ┌─────────────────────────────┐│ │
│  │  │     Padding (내부 여백)      ││ │
│  │  │  ┌─────────────────────────┐││ │
│  │  │  │    Content (내용)       │││ │
│  │  │  └─────────────────────────┘││ │
│  │  └─────────────────────────────┘│ │
│  └─────────────────────────────────┘ │
└─────────────────────────────────────┘
```

### Box Model 구성 요소

| 요소 | 설명 | 위치 |
|------|------|------|
| **Content** | 실제 내용 (텍스트, 이미지 등) | 중심 |
| **Padding** | 내용과 테두리 사이의 여백 | 테두리 안쪽 |
| **Border** | 요소의 테두리 | Padding 외부 |
| **Margin** | 다른 요소와의 간격 | 테두리 바깥 |

## 2. Margin (외부 여백)

### 기본 개념

**Margin**은 요소의 **테두리 바깥**에 생기는 여백입니다.

```css
.box {
  margin: 20px;  /* 모든 방향 20px */
}
```

### 사용 방법

```css
/* 1. 한 값: 모든 방향 */
.box {
  margin: 50px;
  /* top, right, bottom, left 모두 50px */
}

/* 2. 네 값: 각 방향 지정 */
.box {
  margin: 50px 50px 50px 50px;
  /* 순서: 위, 오른쪽, 아래, 왼쪽 (시계방향) */
}

/* 3. 개별 속성 */
.box {
  margin-top: 50px;
  margin-right: 50px;
  margin-bottom: 50px;
  margin-left: 50px;
}
```

### Margin 단축 표기법

```css
/* 한 값 */
margin: 20px;
/* top=20px, right=20px, bottom=20px, left=20px */

/* 두 값 */
margin: 20px 40px;
/* top=20px, right=40px, bottom=20px, left=40px */
/* (상하) (좌우) */

/* 세 값 */
margin: 20px 40px 30px;
/* top=20px, right=40px, bottom=30px, left=40px */
/* (상) (좌우) (하) */

/* 네 값 */
margin: 20px 40px 30px 10px;
/* top=20px, right=40px, bottom=30px, left=10px */
/* (상) (우) (하) (좌) - 시계방향 */
```

> **Tip**: 시계방향(12시부터)으로 위(top) → 오른쪽(right) → 아래(bottom) → 왼쪽(left)
{: .prompt-tip }

### 실전 예제

```css
/* 카드 간격 */
.card {
  margin: 20px;
  border: 1px solid #ddd;
  padding: 15px;
}

/* 중앙 정렬 */
.container {
  width: 800px;
  margin: 0 auto;  /* 좌우 여백 자동 = 중앙 정렬 */
}

/* 상하만 여백 */
.section {
  margin: 40px 0;  /* 위아래 40px, 좌우 0 */
}
```

## 3. Padding (내부 여백)

### 기본 개념

**Padding**은 요소의 **테두리 안쪽**, 내용과 테두리 사이의 여백입니다.

```css
.box {
  padding: 20px;  /* 내용 주변에 20px 여백 */
}
```

### 사용 방법

```css
/* 1. 한 값: 모든 방향 */
.box {
  padding: 50px;
  /* top, right, bottom, left 모두 50px */
}

/* 2. 네 값: 각 방향 지정 */
.box {
  padding: 50px 50px 50px 50px;
  /* 순서: 위, 오른쪽, 아래, 왼쪽 */
}

/* 3. 개별 속성 */
.box {
  padding-top: 50px;
  padding-right: 50px;
  padding-bottom: 50px;
  padding-left: 50px;
}
```

### Padding 단축 표기법

```css
/* Margin과 동일한 규칙 */

/* 한 값 */
padding: 20px;

/* 두 값 */
padding: 20px 40px;  /* (상하) (좌우) */

/* 세 값 */
padding: 20px 40px 30px;  /* (상) (좌우) (하) */

/* 네 값 */
padding: 20px 40px 30px 10px;  /* (상) (우) (하) (좌) */
```

### 실전 예제

```css
/* 버튼 */
.btn {
  padding: 10px 20px;  /* 위아래 10px, 좌우 20px */
  background-color: #007bff;
  color: white;
  border: none;
}

/* 카드 콘텐츠 */
.card-body {
  padding: 20px;
  background-color: white;
}

/* 텍스트 영역 */
.text-container {
  padding: 15px 20px;
  line-height: 1.6;
}
```

## 4. Margin vs Padding 비교

### 차이점

| 구분 | Margin | Padding |
|------|--------|---------|
| **위치** | 테두리 **바깥** | 테두리 **안쪽** |
| **배경색** | 적용 안 됨 | 적용됨 |
| **음수 값** | ✅ 가능 | ❌ 불가능 |
| **auto** | ✅ 가능 (중앙 정렬) | ❌ 불가능 |
| **용도** | 요소 간 간격 | 내용과 테두리 간격 |

### 시각적 비교

```css
/* Margin: 배경색 적용 안 됨 */
.box-margin {
  width: 200px;
  height: 100px;
  margin: 50px;
  background-color: lightblue;
  /* 배경색은 content + padding 영역만 */
}

/* Padding: 배경색 적용됨 */
.box-padding {
  width: 200px;
  height: 100px;
  padding: 50px;
  background-color: lightblue;
  /* 배경색이 padding 영역까지 확장 */
}
```

## 5. 박스 크기 계산

### 기본 계산법

```css
.example {
  width: 273px;
  height: 90px;
  margin: 50px;
  border: 5px solid black;
  padding: 50px;
}
```

**계산:**
- **순수 내용 영역**: 163px × (계산 필요)
  - 가로: 273 - 50(좌 padding) - 50(우 padding) - 5(좌 border) - 5(우 border) = 163px
- **테두리 포함**: 273px (width 값)
- **전체 영역** (margin 포함): 373px
  - 273 + 50(좌 margin) + 50(우 margin) = 373px

### box-sizing 속성

```css
/* 기본값: content-box */
.box-content {
  box-sizing: content-box;
  width: 200px;
  padding: 20px;
  border: 5px solid black;
  /* 실제 너비: 200 + 20*2 + 5*2 = 250px */
}

/* border-box: padding과 border 포함 */
.box-border {
  box-sizing: border-box;
  width: 200px;
  padding: 20px;
  border: 5px solid black;
  /* 실제 너비: 200px (padding, border 포함) */
}
```

> **Tip**: `box-sizing: border-box`를 사용하면 계산이 훨씬 쉬워집니다.
{: .prompt-tip }

## 6. 특수한 사용법

### margin: auto (중앙 정렬)

```css
.container {
  width: 800px;
  margin: 0 auto;  /* 좌우 중앙 정렬 */
}

/* 완전한 중앙 정렬 */
.center {
  width: 600px;
  margin-left: auto;
  margin-right: auto;
}
```

### 음수 margin

```css
/* 요소를 위로 올리기 */
.overlap {
  margin-top: -20px;
}

/* 요소를 왼쪽으로 이동 */
.pull-left {
  margin-left: -30px;
}
```

> **Warning**: 음수 margin은 레이아웃을 깨뜨릴 수 있으니 신중하게 사용하세요.
{: .prompt-warning }

### Margin Collapse (마진 병합)

```css
/* 위 요소 */
.box1 {
  margin-bottom: 30px;
}

/* 아래 요소 */
.box2 {
  margin-top: 20px;
}

/* 실제 간격: 50px이 아닌 30px (큰 값 적용) */
```

## 7. 실전 활용

### 카드 레이아웃

```css
.card {
  width: 300px;
  margin: 20px;          /* 카드 간 간격 */
  padding: 20px;         /* 내부 여백 */
  border: 1px solid #ddd;
  border-radius: 8px;
  background-color: white;
}

.card-header {
  padding: 15px 20px;
  border-bottom: 1px solid #eee;
}

.card-body {
  padding: 20px;
}

.card-footer {
  padding: 15px 20px;
  border-top: 1px solid #eee;
}
```

### 네비게이션

```css
nav {
  padding: 0 20px;       /* 좌우 여백 */
  background-color: #333;
}

nav ul {
  margin: 0;
  padding: 0;
  list-style: none;
}

nav li {
  display: inline-block;
  margin: 0 15px;        /* 메뉴 항목 간격 */
}

nav a {
  display: block;
  padding: 15px 10px;    /* 클릭 영역 확보 */
  color: white;
}
```

### 컨테이너

```css
.container {
  max-width: 1200px;
  margin: 0 auto;        /* 중앙 정렬 */
  padding: 0 20px;       /* 모바일 대응 */
}

.section {
  margin: 60px 0;        /* 섹션 간 간격 */
  padding: 40px 0;       /* 섹션 내부 여백 */
}
```

## 8. Best Practices

### ✅ 좋은 예

```css
/* 명확한 의도 */
.content {
  margin: 40px auto;     /* 상하 40px, 좌우 중앙 */
  padding: 20px;
  max-width: 800px;
}

/* box-sizing 사용 */
* {
  box-sizing: border-box;
}

/* 일관된 간격 */
.section {
  margin-bottom: 60px;
}
```

### ❌ 나쁜 예

```css
/* 불필요한 복잡성 */
.box {
  margin-top: 10px;
  margin-right: 15px;
  margin-bottom: 10px;
  margin-left: 15px;
  /* margin: 10px 15px; 로 간단히 */
}

/* 음수 margin 남용 */
.overlap {
  margin-top: -100px;    /* 레이아웃 깨짐 위험 */
}
```

## 9. 반응형 여백

```css
/* 모바일 */
.container {
  padding: 15px;
  margin: 20px 0;
}

/* 태블릿 */
@media (min-width: 768px) {
  .container {
    padding: 30px;
    margin: 40px auto;
  }
}

/* 데스크톱 */
@media (min-width: 1024px) {
  .container {
    padding: 40px;
    margin: 60px auto;
    max-width: 1200px;
  }
}
```

## 정리

### 핵심 개념

```css
/* Margin: 외부 여백 (요소 간 간격) */
.box {
  margin: 20px;
  margin: 20px 40px;           /* 상하 좌우 */
  margin: 20px 40px 30px 10px; /* 상 우 하 좌 */
}

/* Padding: 내부 여백 (내용과 테두리 간격) */
.box {
  padding: 20px;
  padding: 20px 40px;          /* 상하 좌우 */
  padding: 20px 40px 30px 10px;/* 상 우 하 좌 */
}
```

### 선택 가이드

| 상황 | 사용 |
|------|------|
| 요소 사이 간격 | `margin` |
| 클릭 영역 확장 | `padding` |
| 중앙 정렬 | `margin: 0 auto` |
| 배경색 영역 확장 | `padding` |

### 단축 표기 순서

```
┌────────────┐
│    top     │  1번
├────────────┤
│ left │right│  4번  2번
├────────────┤
│   bottom   │  3번
└────────────┘

margin: top right bottom left;  (시계방향)
```

## 참고 자료

- [MDN - Box Model](https://developer.mozilla.org/ko/docs/Web/CSS/CSS_Box_Model)
- [MDN - margin](https://developer.mozilla.org/ko/docs/Web/CSS/margin)
- [MDN - padding](https://developer.mozilla.org/ko/docs/Web/CSS/padding)
- [MDN - box-sizing](https://developer.mozilla.org/ko/docs/Web/CSS/box-sizing)
