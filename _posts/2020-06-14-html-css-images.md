---
title: HTML/CSS 이미지 완벽 가이드 - img 태그 vs background-image
description: HTML img 태그와 CSS background-image의 차이점, 사용 시나리오, 이미지 최적화 방법을 학습하여 올바른 이미지 활용법을 익힙니다.
author: changsu
date: 2020-06-14 19:16:19 +0900
categories: [Web, HTML]
tags: [html, css, image, responsive]
---

## 1. 이미지 삽입 방법

웹에서 이미지를 표시하는 방법은 크게 두 가지입니다:

1. **HTML `<img>` 태그** - 콘텐츠 이미지
2. **CSS `background-image`** - 장식 이미지

## 2. img 태그로 이미지 넣기

### 2.1 기본 문법

```html
<img src="image.jpg" alt="이미지 설명">
```

#### 필수 속성

| 속성 | 설명 | 예시 |
|------|------|------|
| **src** | 이미지 파일 경로 또는 URL | `src="./images/logo.png"` |
| **alt** | 대체 텍스트 (이미지 로드 실패 시 표시) | `alt="회사 로고"` |

### 2.2 이미지 경로

#### 상대 경로

```html
<!-- 같은 폴더 -->
<img src="photo.jpg" alt="사진">

<!-- 하위 폴더 -->
<img src="images/photo.jpg" alt="사진">

<!-- 상위 폴더 -->
<img src="../photo.jpg" alt="사진">
```

#### 절대 경로 (URL)

```html
<img src="https://example.com/images/photo.jpg" alt="사진">
```

### 2.3 이미지 크기 조절

#### HTML 속성 (비권장)

```html
<img src="photo.jpg" alt="사진" width="300" height="200">
```

> **Warning**: HTML에서 크기를 지정하면 유지보수가 어렵습니다. CSS를 사용하세요.
{: .prompt-warning }

#### CSS로 크기 조절 (권장)

```css
img {
  width: 150px;
}
```

**장점:**
- 가로만 지정하면 세로는 비율에 맞게 자동 조정
- 모든 스타일을 CSS 파일에서 중앙 관리
- 반응형 디자인에 유리

```css
/* 비율 유지하며 크기 조절 */
img {
  width: 100%;  /* 부모 요소에 맞춤 */
  height: auto; /* 비율 유지 */
}

/* 최대 크기 제한 */
img {
  max-width: 500px;
  height: auto;
}

/* 고정 크기 (비율 무시) */
img {
  width: 200px;
  height: 200px;
  object-fit: cover;  /* 이미지 잘림 */
}
```

### 2.4 alt 속성의 중요성

```html
<!-- ✅ 좋은 예: 구체적인 설명 -->
<img src="sunset.jpg" alt="바다 위로 지는 석양">

<!-- ❌ 나쁜 예: 설명 부족 -->
<img src="sunset.jpg" alt="이미지">

<!-- ❌ 나쁜 예: alt 속성 누락 -->
<img src="sunset.jpg">
```

**alt 속성이 중요한 이유:**
1. **접근성**: 스크린 리더 사용자가 이미지를 이해
2. **SEO**: 검색 엔진이 이미지 내용을 파악
3. **사용자 경험**: 이미지 로드 실패 시 대체 텍스트 표시

### 2.5 img 태그 완전한 예제

```html
<img
  src="./images/product.jpg"
  alt="빨간색 운동화"
  width="400"
  height="300"
  loading="lazy"
  decoding="async"
>
```

```css
.product-image {
  width: 100%;
  max-width: 400px;
  height: auto;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}
```

## 3. background-image로 이미지 넣기

### 3.1 기본 문법

CSS를 사용하여 요소의 배경으로 이미지를 설정합니다.

```html
<div class="bg-img">배경 이미지 영역</div>
```

```css
.bg-img {
  background-image: url("./images/background.jpg");
  background-color: #f0f0f0;  /* 이미지 로드 전 배경색 */
  height: 300px;
}
```

### 3.2 background 관련 속성

#### background-size

```css
.bg-img {
  background-image: url("bg.jpg");

  /* 1. cover - 영역을 꽉 채움 (이미지 잘릴 수 있음) */
  background-size: cover;

  /* 2. contain - 이미지 전체 표시 (여백 생길 수 있음) */
  background-size: contain;

  /* 3. 퍼센트 */
  background-size: 100% 100%;  /* 가로 100%, 세로 100% */

  /* 4. 픽셀 */
  background-size: 400px 300px;
}
```

**시각적 비교:**

```css
/* cover - 영역을 완전히 채움 */
.hero {
  background-size: cover;
  /* 이미지가 잘릴 수 있지만 여백 없음 */
}

/* contain - 이미지 전체를 보여줌 */
.gallery {
  background-size: contain;
  /* 이미지가 안 잘리지만 여백 생김 */
}
```

#### background-repeat

```css
.bg-img {
  background-image: url("pattern.png");

  /* 반복 안 함 */
  background-repeat: no-repeat;

  /* 가로로만 반복 */
  background-repeat: repeat-x;

  /* 세로로만 반복 */
  background-repeat: repeat-y;

  /* 양방향 반복 (기본값) */
  background-repeat: repeat;
}
```

#### background-position

```css
.bg-img {
  background-image: url("photo.jpg");
  background-repeat: no-repeat;

  /* 키워드 */
  background-position: center;
  background-position: top right;
  background-position: bottom left;

  /* 퍼센트 */
  background-position: 50% 50%;  /* 중앙 */

  /* 픽셀 */
  background-position: 20px 30px;
}
```

#### background 축약형

```css
/* 개별 속성 */
.bg-img {
  background-color: #333;
  background-image: url("bg.jpg");
  background-repeat: no-repeat;
  background-position: center;
  background-size: cover;
}

/* 축약형 (단, background-size는 따로 작성) */
.bg-img {
  background: #333 url("bg.jpg") no-repeat center;
  background-size: cover;
}
```

### 3.3 실전 예제

#### 히어로 섹션

```html
<section class="hero">
  <div class="hero-content">
    <h1>환영합니다</h1>
    <p>최고의 서비스를 제공합니다</p>
  </div>
</section>
```

```css
.hero {
  background-image: url("./images/hero-bg.jpg");
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
  height: 500px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
  text-align: center;
}

.hero-content {
  background-color: rgba(0, 0, 0, 0.5);  /* 반투명 오버레이 */
  padding: 40px;
  border-radius: 10px;
}
```

#### 카드 배경

```html
<div class="card">
  <div class="card-image"></div>
  <div class="card-content">
    <h3>제목</h3>
    <p>내용</p>
  </div>
</div>
```

```css
.card {
  border: 1px solid #ddd;
  border-radius: 8px;
  overflow: hidden;
}

.card-image {
  background-image: url("./images/card-bg.jpg");
  background-size: cover;
  background-position: center;
  height: 200px;
}

.card-content {
  padding: 20px;
}
```

## 4. img vs background-image 비교

### 4.1 언제 무엇을 사용할까?

#### img 태그 사용 (콘텐츠 이미지)

```html
<!-- ✅ 콘텐츠의 일부인 이미지 -->
<img src="product.jpg" alt="상품 사진">
<img src="profile.jpg" alt="프로필 사진">
<img src="article-photo.jpg" alt="기사 사진">
```

**사용 시나리오:**
- 상품 이미지
- 프로필 사진
- 기사/블로그 포스트 이미지
- 로고 (콘텐츠로 간주되는 경우)
- 갤러리/포트폴리오 이미지

**장점:**
- ✅ SEO에 유리 (검색 엔진이 인덱싱)
- ✅ 접근성 좋음 (alt 텍스트)
- ✅ 인쇄 시 표시됨
- ✅ 이미지 저장/드래그 가능

#### background-image 사용 (장식 이미지)

```css
/* ✅ 디자인/레이아웃을 위한 이미지 */
.hero {
  background-image: url("hero-bg.jpg");
}

.section {
  background-image: url("pattern.png");
}
```

**사용 시나리오:**
- 히어로 섹션 배경
- 패턴/텍스처
- 장식용 배경
- 오버레이가 필요한 경우
- 여러 배경 레이어

**장점:**
- ✅ 요소 크기에 맞게 조절 쉬움
- ✅ 여러 배경 이미지 가능
- ✅ CSS로 완전한 제어
- ✅ 텍스트 오버레이 쉬움

### 4.2 비교표

| 항목 | `<img>` | `background-image` |
|------|---------|-------------------|
| **용도** | 콘텐츠 | 장식 |
| **SEO** | 유리 | 불리 |
| **접근성** | `alt` 제공 | 없음 |
| **인쇄** | 포함됨 | 선택적 |
| **반응형** | `srcset` 사용 가능 | 미디어 쿼리 필요 |
| **이미지 저장** | 가능 | 불가능 |
| **크기 조절** | CSS 필요 | CSS로 쉬움 |
| **여러 이미지** | 불가능 | 가능 |

## 5. 반응형 이미지

### 5.1 img 태그 반응형

```css
/* 기본 반응형 */
img {
  max-width: 100%;
  height: auto;
}

/* 미디어 쿼리 */
@media (max-width: 768px) {
  img {
    max-width: 100%;
  }
}
```

#### srcset 사용 (고급)

```html
<img
  src="photo-800.jpg"
  srcset="
    photo-400.jpg 400w,
    photo-800.jpg 800w,
    photo-1200.jpg 1200w
  "
  sizes="(max-width: 600px) 400px, (max-width: 900px) 800px, 1200px"
  alt="반응형 이미지"
>
```

### 5.2 background-image 반응형

```css
.hero {
  background-image: url("bg-large.jpg");
  background-size: cover;
  background-position: center;
  height: 500px;
}

@media (max-width: 768px) {
  .hero {
    background-image: url("bg-small.jpg");
    height: 300px;
  }
}
```

## 6. 이미지 최적화

### 6.1 파일 형식 선택

| 형식 | 용도 | 장점 | 단점 |
|------|------|------|------|
| **JPEG** | 사진 | 좋은 압축률 | 투명도 불가 |
| **PNG** | 로고, 아이콘 | 투명도 지원 | 파일 크기 큼 |
| **WebP** | 모든 용도 | 작은 파일 크기 | 구형 브라우저 미지원 |
| **SVG** | 벡터 이미지 | 확대해도 선명 | 복잡한 이미지 부적합 |

### 6.2 이미지 최적화 기법

```html
<!-- lazy loading (지연 로딩) -->
<img src="photo.jpg" alt="사진" loading="lazy">

<!-- 비동기 디코딩 -->
<img src="photo.jpg" alt="사진" decoding="async">
```

```css
/* CSS로 이미지 최적화 */
img {
  max-width: 100%;
  height: auto;
  image-rendering: auto;  /* 브라우저 최적화 */
}
```

### 6.3 WebP와 폴백

```html
<picture>
  <source srcset="image.webp" type="image/webp">
  <source srcset="image.jpg" type="image/jpeg">
  <img src="image.jpg" alt="이미지">
</picture>
```

## 7. 실전 패턴

### 7.1 이미지 + 텍스트 오버레이

```html
<div class="image-overlay">
  <img src="photo.jpg" alt="사진">
  <div class="overlay-text">
    <h2>제목</h2>
    <p>설명</p>
  </div>
</div>
```

```css
.image-overlay {
  position: relative;
}

.image-overlay img {
  width: 100%;
  display: block;
}

.overlay-text {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  color: white;
  text-align: center;
  background-color: rgba(0, 0, 0, 0.5);
  padding: 20px;
  border-radius: 8px;
}
```

### 7.2 배경 이미지 + 그라데이션

```css
.hero {
  background:
    linear-gradient(rgba(0, 0, 0, 0.5), rgba(0, 0, 0, 0.5)),
    url("hero-bg.jpg");
  background-size: cover;
  background-position: center;
  color: white;
  padding: 100px 20px;
}
```

### 7.3 이미지 갤러리

```html
<div class="gallery">
  <img src="photo1.jpg" alt="사진 1">
  <img src="photo2.jpg" alt="사진 2">
  <img src="photo3.jpg" alt="사진 3">
  <img src="photo4.jpg" alt="사진 4">
</div>
```

```css
.gallery {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 20px;
}

.gallery img {
  width: 100%;
  height: 250px;
  object-fit: cover;
  border-radius: 8px;
  transition: transform 0.3s;
}

.gallery img:hover {
  transform: scale(1.05);
}
```

### 7.4 프로필 이미지 (원형)

```html
<div class="profile">
  <img src="profile.jpg" alt="프로필">
  <h3>홍길동</h3>
  <p>개발자</p>
</div>
```

```css
.profile {
  text-align: center;
}

.profile img {
  width: 150px;
  height: 150px;
  border-radius: 50%;  /* 원형 */
  object-fit: cover;
  border: 4px solid #007bff;
}
```

## 8. 접근성과 SEO

### 8.1 alt 텍스트 작성 가이드

```html
<!-- ✅ 좋은 예 -->
<img src="sunset.jpg" alt="바다 위로 지는 주황색 석양">

<!-- ✅ 장식 이미지는 빈 alt -->
<img src="decoration.png" alt="">

<!-- ❌ 나쁜 예 -->
<img src="sunset.jpg" alt="이미지">
<img src="sunset.jpg" alt="IMG_1234.jpg">
```

### 8.2 figure와 figcaption

```html
<figure>
  <img src="graph.jpg" alt="2024년 매출 그래프">
  <figcaption>그림 1: 연간 매출 증가 추이</figcaption>
</figure>
```

### 8.3 SEO 최적화

```html
<!-- 파일명을 의미있게 -->
<img src="red-sneakers-nike.jpg" alt="나이키 빨간색 운동화">

<!-- title 속성 추가 (선택) -->
<img
  src="product.jpg"
  alt="프리미엄 노트북"
  title="15인치 고성능 노트북"
>
```

## 9. 성능 최적화

### 9.1 이미지 크기 최적화

```html
<!-- ❌ 나쁜 예: 큰 이미지를 CSS로 축소 -->
<img src="huge-image-5000x3000.jpg" style="width: 300px;">

<!-- ✅ 좋은 예: 적절한 크기로 미리 리사이즈 -->
<img src="optimized-image-600x400.jpg" alt="이미지">
```

### 9.2 지연 로딩

```html
<!-- 뷰포트에 들어올 때 로드 -->
<img src="photo.jpg" alt="사진" loading="lazy">
```

### 9.3 CDN 사용

```html
<img src="https://cdn.example.com/optimized/photo.jpg" alt="사진">
```

## 10. 주의사항

### 10.1 배경 이미지 높이 설정

```css
/* ❌ 높이를 지정하지 않으면 보이지 않음 */
.bg-img {
  background-image: url("bg.jpg");
  /* height 없음 - 이미지 안 보임 */
}

/* ✅ 높이 지정 필수 */
.bg-img {
  background-image: url("bg.jpg");
  height: 300px;  /* 또는 padding으로 높이 확보 */
}
```

### 10.2 object-fit 활용

```css
/* 이미지 왜곡 방지 */
img {
  width: 200px;
  height: 200px;
  object-fit: cover;  /* 비율 유지하며 채우기 */
}
```

## 정리

### img 태그

```html
<img src="photo.jpg" alt="사진 설명">
```

- **용도**: 콘텐츠 이미지 (상품, 프로필, 기사 등)
- **장점**: SEO, 접근성, 인쇄
- **CSS**: `max-width: 100%; height: auto;`

### background-image

```css
.hero {
  background-image: url("bg.jpg");
  background-size: cover;
  background-position: center;
  height: 500px;
}
```

- **용도**: 장식 이미지 (배경, 패턴)
- **장점**: CSS 제어, 여러 배경, 오버레이
- **주의**: 높이 지정 필수

### 선택 가이드

1. **콘텐츠의 일부** → `<img>` 사용
2. **디자인 요소** → `background-image` 사용
3. **SEO 중요** → `<img>` 사용
4. **여러 레이어 필요** → `background-image` 사용

> **핵심**: 이미지의 **목적**에 따라 적절한 방법을 선택하세요.
{: .prompt-tip }

## 참고 자료

- [MDN - img 요소](https://developer.mozilla.org/ko/docs/Web/HTML/Element/img)
- [MDN - background-image](https://developer.mozilla.org/ko/docs/Web/CSS/background-image)
- [Web.dev - Image Optimization](https://web.dev/fast/#optimize-your-images)
- [CSS Tricks - A Guide To The Responsive Images Syntax](https://css-tricks.com/a-guide-to-the-responsive-images-syntax-in-html/)
