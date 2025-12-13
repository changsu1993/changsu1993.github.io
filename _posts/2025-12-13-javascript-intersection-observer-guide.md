---
title: "JavaScript Intersection Observer API 완벽 가이드 - 스크롤 기반 기능 구현의 최적 솔루션"
description: "Intersection Observer API로 무한 스크롤, 이미지 지연 로딩, 스크롤 애니메이션을 구현하세요. scroll 이벤트보다 성능이 뛰어난 이유와 React 커스텀 훅까지 완벽 정리."
date: 2025-12-13 10:00:00 +0900
categories: [Frontend, JavaScript]
tags: [javascript, lazy-loading, infinite-scroll, intersection-observer, performance, react-hooks, scroll-animation, viewport, browser-api]
---

## 개요

웹 애플리케이션에서 요소가 화면에 보이는지 감지하는 것은 매우 흔한 요구사항입니다. 이미지 지연 로딩, 무한 스크롤, 광고 노출 측정, 스크롤 애니메이션 등 다양한 기능이 이 감지에 의존합니다.

전통적으로 이런 기능은 `scroll` 이벤트 리스너와 `getBoundingClientRect()`를 조합하여 구현했습니다. 하지만 이 방식은 성능 문제를 야기합니다. **Intersection Observer API**는 이 문제를 해결하기 위해 만들어진 브라우저 네이티브 API입니다.

> **참고**: 이벤트 호출 빈도를 제어하는 디바운스와 스로틀 기법은 [JavaScript 디바운스와 스로틀 완벽 가이드](/posts/javascript-debounce-throttle-guide/)를 참고하세요. Intersection Observer는 이런 기법이 필요 없는 근본적으로 다른 접근 방식입니다.
{: .prompt-info }

### 학습 목표

- Intersection Observer API의 동작 원리와 기본 사용법 이해
- root, rootMargin, threshold 옵션 활용법 습득
- 이미지 지연 로딩, 무한 스크롤, 스크롤 애니메이션 구현
- React 환경에서의 활용법과 커스텀 훅 작성
- scroll 이벤트 대비 성능 이점 이해

### 사전 지식

- JavaScript 기초 ([JavaScript 함수 완벽 가이드](/posts/javascript-function-guide/) 참고)
- DOM 조작 기초
- ES6+ 문법 (화살표 함수, 구조 분해 할당)

---

## 기존 scroll 이벤트의 문제점

Intersection Observer가 왜 필요한지 이해하려면, 먼저 기존 방식의 문제점을 알아야 합니다.

### 전통적인 방식: scroll 이벤트 + getBoundingClientRect()

```javascript
// 이미지 지연 로딩을 scroll 이벤트로 구현하는 전통적인 방식
function lazyLoadWithScroll() {
  const images = document.querySelectorAll('img[data-src]');

  function checkImages() {
    const viewportHeight = window.innerHeight;

    images.forEach(img => {
      // getBoundingClientRect()는 리플로우를 발생시킴
      const rect = img.getBoundingClientRect();

      if (rect.top < viewportHeight && rect.bottom > 0) {
        img.src = img.dataset.src;
        img.removeAttribute('data-src');
      }
    });
  }

  // scroll 이벤트는 초당 수십~수백 번 발생
  window.addEventListener('scroll', checkImages);
  checkImages(); // 초기 체크
}
```

### 이 방식의 문제점

**1. 메인 스레드 블로킹**

`scroll` 이벤트 핸들러는 메인 스레드에서 동기적으로 실행됩니다. 핸들러 내에서 복잡한 계산이나 DOM 조작이 이루어지면 스크롤이 버벅거립니다.

**2. 강제 리플로우(Forced Reflow)**

`getBoundingClientRect()` 호출은 브라우저에게 현재 레이아웃을 계산하도록 강제합니다. 스크롤할 때마다 이 계산이 반복되면 심각한 성능 저하가 발생합니다.

**3. 빈번한 호출**

스크롤 이벤트는 사용자가 스크롤할 때 프레임당 여러 번 발생할 수 있습니다. 60fps 기준으로 초당 최대 60회 이상 핸들러가 호출됩니다.

**4. 디바운스/스로틀 필요**

성능 문제를 완화하기 위해 디바운스나 스로틀을 적용해야 하지만, 이는 응답성을 저하시킵니다.

```javascript
// 스로틀을 적용해도 근본적인 문제는 해결되지 않음
const throttledCheck = throttle(checkImages, 100);
window.addEventListener('scroll', throttledCheck);
```

---

## Intersection Observer란?

**Intersection Observer API**는 타겟 요소와 상위 요소(또는 뷰포트) 간의 교차 상태를 비동기적으로 관찰하는 브라우저 내장 API입니다.

### 핵심 특징

| 특징 | 설명 |
|------|------|
| **비동기 처리** | 메인 스레드를 블로킹하지 않음 |
| **네이티브 최적화** | 브라우저가 내부적으로 최적화된 방식으로 교차 감지 |
| **디바운스/스로틀 불필요** | API가 자체적으로 효율적인 호출 관리 |
| **정확한 교차 비율** | 요소가 얼마나 보이는지 퍼센트로 제공 |
| **다중 요소 관찰** | 하나의 옵저버로 여러 요소 관찰 가능 |

### 기본 개념

Intersection Observer에서 알아야 할 핵심 용어:

- **Target**: 관찰 대상이 되는 요소
- **Root**: 교차를 판단하는 기준 요소 (기본값: 뷰포트)
- **Root Margin**: Root의 경계를 확장하거나 축소
- **Threshold**: 콜백을 트리거할 교차 비율

```
┌─────────────────────────────────────────┐
│                                         │
│            Viewport (Root)              │
│   ┌─────────────────────────────┐       │
│   │                             │       │
│   │       rootMargin 영역        │       │
│   │   ┌─────────────────────┐   │       │
│   │   │                     │   │       │
│   │   │   Target Element    │   │       │
│   │   │                     │   │       │
│   │   └─────────────────────┘   │       │
│   │                             │       │
│   └─────────────────────────────┘       │
│                                         │
└─────────────────────────────────────────┘
```

---

## 기본 사용법

### IntersectionObserver 생성

```javascript
// 콜백 함수: 교차 상태가 변경될 때 호출됨
const callback = (entries, observer) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      console.log('요소가 화면에 보입니다:', entry.target);
    } else {
      console.log('요소가 화면에서 사라졌습니다:', entry.target);
    }
  });
};

// 옵션 설정
const options = {
  root: null,        // null = 뷰포트 기준
  rootMargin: '0px', // root 경계 조정
  threshold: 0       // 교차 비율 (0 = 1px이라도 보이면)
};

// 옵저버 생성
const observer = new IntersectionObserver(callback, options);

// 요소 관찰 시작
const target = document.querySelector('.target-element');
observer.observe(target);
```

### 주요 메서드

```javascript
const observer = new IntersectionObserver(callback, options);

// 관찰 시작
observer.observe(element);

// 특정 요소 관찰 중단
observer.unobserve(element);

// 모든 관찰 중단 및 옵저버 해제
observer.disconnect();

// 현재 관찰 중인 모든 엔트리 즉시 반환
const entries = observer.takeRecords();
```

### IntersectionObserverEntry 객체

콜백 함수가 받는 `entries` 배열의 각 항목은 다음 속성을 가집니다:

```javascript
const callback = (entries) => {
  entries.forEach(entry => {
    // 교차 여부 (boolean)
    console.log('isIntersecting:', entry.isIntersecting);

    // 교차 비율 (0 ~ 1)
    console.log('intersectionRatio:', entry.intersectionRatio);

    // 타겟 요소의 경계 정보
    console.log('boundingClientRect:', entry.boundingClientRect);

    // 교차 영역의 경계 정보
    console.log('intersectionRect:', entry.intersectionRect);

    // Root의 경계 정보
    console.log('rootBounds:', entry.rootBounds);

    // 관찰 대상 요소
    console.log('target:', entry.target);

    // 교차 상태가 변경된 시간 (DOMHighResTimeStamp)
    console.log('time:', entry.time);
  });
};
```

---

## 옵션 상세 설명

### root: 관찰 기준 요소

`root` 옵션은 교차를 판단할 기준 요소를 지정합니다.

```javascript
// 뷰포트 기준 (기본값)
const observer1 = new IntersectionObserver(callback, {
  root: null
});

// 특정 컨테이너 기준
const scrollContainer = document.querySelector('.scroll-container');
const observer2 = new IntersectionObserver(callback, {
  root: scrollContainer
});
```

**컨테이너 기준 관찰 예시:**

```javascript
// 수평 스크롤 갤러리에서 현재 보이는 이미지 감지
const gallery = document.querySelector('.horizontal-gallery');

const observer = new IntersectionObserver(
  (entries) => {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        entry.target.classList.add('visible');
      } else {
        entry.target.classList.remove('visible');
      }
    });
  },
  { root: gallery }
);

gallery.querySelectorAll('.gallery-item').forEach(item => {
  observer.observe(item);
});
```

### rootMargin: 감지 영역 확장/축소

`rootMargin`은 CSS margin처럼 root의 경계를 조정합니다. 양수 값은 확장, 음수 값은 축소입니다.

```javascript
// 뷰포트 아래 200px 전에 미리 감지 (이미지 프리로드에 유용)
const observer1 = new IntersectionObserver(callback, {
  rootMargin: '0px 0px 200px 0px' // top right bottom left
});

// 뷰포트보다 100px 안쪽에서 감지
const observer2 = new IntersectionObserver(callback, {
  rootMargin: '-100px'
});

// 상하로만 확장
const observer3 = new IntersectionObserver(callback, {
  rootMargin: '100px 0px' // 상하 100px, 좌우 0px
});
```

**실용 예시 - 이미지 프리로드:**

```javascript
// 이미지가 뷰포트에 들어오기 300px 전에 미리 로드 시작
const lazyImageObserver = new IntersectionObserver(
  (entries) => {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        const img = entry.target;
        img.src = img.dataset.src;
        lazyImageObserver.unobserve(img);
      }
    });
  },
  {
    rootMargin: '0px 0px 300px 0px' // 아래쪽으로 300px 확장
  }
);
```

### threshold: 교차 비율 트리거

`threshold`는 콜백을 트리거할 교차 비율을 지정합니다.

```javascript
// 1px이라도 보이면 (기본값)
const observer1 = new IntersectionObserver(callback, {
  threshold: 0
});

// 50% 이상 보일 때
const observer2 = new IntersectionObserver(callback, {
  threshold: 0.5
});

// 완전히 보일 때
const observer3 = new IntersectionObserver(callback, {
  threshold: 1.0
});

// 여러 단계에서 콜백 (10% 단위)
const observer4 = new IntersectionObserver(callback, {
  threshold: [0, 0.1, 0.2, 0.3, 0.4, 0.5, 0.6, 0.7, 0.8, 0.9, 1.0]
});

// 간단하게 배열 생성
const thresholds = Array.from({ length: 11 }, (_, i) => i / 10);
// [0, 0.1, 0.2, 0.3, 0.4, 0.5, 0.6, 0.7, 0.8, 0.9, 1]
```

**진행률 표시 예시:**

```javascript
// 스크롤에 따른 읽기 진행률 표시
const progressObserver = new IntersectionObserver(
  (entries) => {
    entries.forEach(entry => {
      // intersectionRatio를 사용해 진행률 계산
      const progress = Math.round(entry.intersectionRatio * 100);
      updateProgressBar(progress);
    });
  },
  {
    threshold: Array.from({ length: 101 }, (_, i) => i / 100) // 1% 단위
  }
);

progressObserver.observe(document.querySelector('.article-content'));
```

---

## 실전 활용 사례

### 1. 이미지 지연 로딩 (Lazy Loading)

가장 일반적인 Intersection Observer 활용 사례입니다.

```javascript
// HTML
// <img class="lazy" data-src="image.jpg" alt="설명">

function setupLazyLoading() {
  const lazyImages = document.querySelectorAll('img.lazy');

  const imageObserver = new IntersectionObserver(
    (entries, observer) => {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          const img = entry.target;

          // data-src의 값을 src로 이동
          img.src = img.dataset.src;

          // srcset 지원
          if (img.dataset.srcset) {
            img.srcset = img.dataset.srcset;
          }

          // 로드 완료 후 클래스 변경
          img.onload = () => {
            img.classList.remove('lazy');
            img.classList.add('loaded');
          };

          // 관찰 중단 (한 번만 로드하면 됨)
          observer.unobserve(img);
        }
      });
    },
    {
      rootMargin: '0px 0px 200px 0px', // 200px 미리 로드
      threshold: 0
    }
  );

  lazyImages.forEach(img => imageObserver.observe(img));
}

// CSS
/*
.lazy {
  opacity: 0;
  transition: opacity 0.3s;
}

.loaded {
  opacity: 1;
}
*/
```

**향상된 버전 - 플레이스홀더와 에러 처리:**

```javascript
function advancedLazyLoading() {
  const images = document.querySelectorAll('img[data-src]');

  const observer = new IntersectionObserver(
    (entries, obs) => {
      entries.forEach(entry => {
        if (!entry.isIntersecting) return;

        const img = entry.target;
        const src = img.dataset.src;

        // 이미지 프리로드
        const tempImage = new Image();

        tempImage.onload = () => {
          img.src = src;
          img.classList.add('fade-in');
          img.removeAttribute('data-src');
        };

        tempImage.onerror = () => {
          img.src = '/images/placeholder-error.png';
          img.alt = '이미지를 불러올 수 없습니다';
          console.error(`이미지 로드 실패: ${src}`);
        };

        tempImage.src = src;
        obs.unobserve(img);
      });
    },
    { rootMargin: '100px' }
  );

  images.forEach(img => observer.observe(img));

  // 정리 함수 반환
  return () => observer.disconnect();
}
```

### 2. 무한 스크롤 (Infinite Scroll)

센티널(Sentinel) 요소를 관찰하여 무한 스크롤을 구현합니다.

```javascript
// HTML 구조
// <div class="post-list">
//   <article>...</article>
//   <article>...</article>
//   <div class="sentinel"></div> <!-- 센티널 요소 -->
// </div>

function setupInfiniteScroll() {
  const sentinel = document.querySelector('.sentinel');
  const postList = document.querySelector('.post-list');

  let page = 1;
  let isLoading = false;
  let hasMore = true;

  const observer = new IntersectionObserver(
    async (entries) => {
      const entry = entries[0];

      if (entry.isIntersecting && !isLoading && hasMore) {
        isLoading = true;
        showLoadingSpinner();

        try {
          const newPosts = await fetchPosts(++page);

          if (newPosts.length === 0) {
            hasMore = false;
            showEndMessage();
            observer.disconnect();
            return;
          }

          // 새 포스트 추가 (센티널 앞에)
          newPosts.forEach(post => {
            const article = createPostElement(post);
            postList.insertBefore(article, sentinel);
          });

        } catch (error) {
          console.error('포스트 로드 실패:', error);
          page--; // 재시도 가능하도록
        } finally {
          isLoading = false;
          hideLoadingSpinner();
        }
      }
    },
    {
      rootMargin: '0px 0px 100px 0px' // 100px 미리 로드
    }
  );

  observer.observe(sentinel);

  return () => observer.disconnect();
}

async function fetchPosts(page) {
  const response = await fetch(`/api/posts?page=${page}&limit=10`);
  return response.json();
}

function createPostElement(post) {
  const article = document.createElement('article');
  article.className = 'post-item';
  article.innerHTML = `
    <h2>${post.title}</h2>
    <p>${post.excerpt}</p>
  `;
  return article;
}
```

### 3. 스크롤 애니메이션

요소가 화면에 들어올 때 애니메이션을 트리거합니다.

```javascript
function setupScrollAnimations() {
  const animatedElements = document.querySelectorAll('[data-animate]');

  const observer = new IntersectionObserver(
    (entries) => {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          const element = entry.target;
          const animation = element.dataset.animate;
          const delay = element.dataset.animateDelay || 0;

          setTimeout(() => {
            element.classList.add('animated', animation);
          }, delay);

          // 한 번만 애니메이션
          observer.unobserve(element);
        }
      });
    },
    {
      threshold: 0.2, // 20% 보일 때 시작
      rootMargin: '0px 0px -50px 0px' // 약간 위로 올라와야 시작
    }
  );

  animatedElements.forEach(el => observer.observe(el));
}

// HTML 사용 예시
// <div data-animate="fade-up" data-animate-delay="100">콘텐츠 1</div>
// <div data-animate="fade-left" data-animate-delay="200">콘텐츠 2</div>
```

**CSS 애니메이션:**

```css
/* 초기 상태 */
[data-animate] {
  opacity: 0;
  transition: opacity 0.6s ease, transform 0.6s ease;
}

[data-animate="fade-up"] {
  transform: translateY(30px);
}

[data-animate="fade-left"] {
  transform: translateX(-30px);
}

[data-animate="fade-right"] {
  transform: translateX(30px);
}

[data-animate="scale-up"] {
  transform: scale(0.9);
}

/* 애니메이션 적용 후 */
.animated {
  opacity: 1;
  transform: translateY(0) translateX(0) scale(1);
}
```

### 4. Sticky Header 감지

스크롤 위치에 따라 헤더 스타일을 변경합니다.

```javascript
function setupStickyHeader() {
  // 헤더 바로 아래에 센티널 요소 배치
  const sentinel = document.createElement('div');
  sentinel.className = 'header-sentinel';
  sentinel.style.height = '1px';
  document.body.insertBefore(sentinel, document.body.firstChild);

  const header = document.querySelector('.site-header');

  const observer = new IntersectionObserver(
    ([entry]) => {
      // 센티널이 보이지 않으면 = 스크롤이 내려간 상태
      if (!entry.isIntersecting) {
        header.classList.add('scrolled');
      } else {
        header.classList.remove('scrolled');
      }
    },
    {
      threshold: 0,
      rootMargin: `-${header.offsetHeight}px 0px 0px 0px`
    }
  );

  observer.observe(sentinel);
}

// CSS
/*
.site-header {
  position: sticky;
  top: 0;
  background: white;
  transition: background 0.3s, box-shadow 0.3s;
}

.site-header.scrolled {
  background: rgba(255, 255, 255, 0.95);
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  backdrop-filter: blur(10px);
}
*/
```

### 5. 광고 뷰어빌리티 측정

광고가 화면에 노출된 시간을 측정합니다. (IAB 기준: 50% 이상 1초 이상 노출)

```javascript
function trackAdViewability(adElement) {
  let visibleTime = 0;
  let lastVisibleTimestamp = null;
  let viewabilityTimeout = null;
  const VIEWABILITY_THRESHOLD = 1000; // 1초

  const observer = new IntersectionObserver(
    ([entry]) => {
      if (entry.intersectionRatio >= 0.5) {
        // 50% 이상 보이기 시작
        lastVisibleTimestamp = performance.now();

        viewabilityTimeout = setTimeout(() => {
          // 1초 이상 유지됨 = 유효 노출
          logViewableImpression(adElement.dataset.adId);
        }, VIEWABILITY_THRESHOLD);

      } else if (lastVisibleTimestamp) {
        // 50% 미만으로 줄어듦
        clearTimeout(viewabilityTimeout);
        visibleTime += performance.now() - lastVisibleTimestamp;
        lastVisibleTimestamp = null;
      }
    },
    {
      threshold: [0, 0.5, 1.0]
    }
  );

  observer.observe(adElement);

  return {
    getVisibleTime: () => {
      let total = visibleTime;
      if (lastVisibleTimestamp) {
        total += performance.now() - lastVisibleTimestamp;
      }
      return total;
    },
    disconnect: () => {
      clearTimeout(viewabilityTimeout);
      observer.disconnect();
    }
  };
}

function logViewableImpression(adId) {
  console.log(`광고 ${adId}: 유효 노출 기록`);
  // 실제로는 서버에 노출 데이터 전송
  // fetch('/api/ad-impression', { method: 'POST', body: JSON.stringify({ adId }) });
}
```

### 6. 섹션 기반 네비게이션 하이라이트

현재 보고 있는 섹션에 해당하는 네비게이션 항목을 하이라이트합니다.

```javascript
function setupSectionNavigation() {
  const sections = document.querySelectorAll('section[id]');
  const navLinks = document.querySelectorAll('.nav-link');

  const observer = new IntersectionObserver(
    (entries) => {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          const sectionId = entry.target.id;

          // 모든 링크에서 active 제거
          navLinks.forEach(link => link.classList.remove('active'));

          // 현재 섹션에 해당하는 링크에 active 추가
          const activeLink = document.querySelector(
            `.nav-link[href="#${sectionId}"]`
          );
          if (activeLink) {
            activeLink.classList.add('active');
          }
        }
      });
    },
    {
      rootMargin: '-20% 0px -80% 0px', // 화면 상단 20% ~ 하단 20% 사이
      threshold: 0
    }
  );

  sections.forEach(section => observer.observe(section));
}
```

---

## React에서 활용하기

### 기본 useInView 훅 구현

```tsx
import { useEffect, useRef, useState, RefObject } from 'react';

interface UseInViewOptions {
  root?: Element | null;
  rootMargin?: string;
  threshold?: number | number[];
  triggerOnce?: boolean;
}

interface UseInViewReturn {
  ref: RefObject<HTMLElement>;
  inView: boolean;
  entry: IntersectionObserverEntry | null;
}

function useInView(options: UseInViewOptions = {}): UseInViewReturn {
  const { root = null, rootMargin = '0px', threshold = 0, triggerOnce = false } = options;

  const ref = useRef<HTMLElement>(null);
  const [inView, setInView] = useState(false);
  const [entry, setEntry] = useState<IntersectionObserverEntry | null>(null);

  useEffect(() => {
    const element = ref.current;
    if (!element) return;

    const observer = new IntersectionObserver(
      ([entry]) => {
        setInView(entry.isIntersecting);
        setEntry(entry);

        // triggerOnce가 true이고 요소가 보이면 관찰 중단
        if (triggerOnce && entry.isIntersecting) {
          observer.unobserve(element);
        }
      },
      { root, rootMargin, threshold }
    );

    observer.observe(element);

    return () => observer.disconnect();
  }, [root, rootMargin, threshold, triggerOnce]);

  return { ref, inView, entry };
}

export default useInView;
```

> **참고**: React 커스텀 훅의 설계 원칙과 고급 패턴에 대한 자세한 내용은 [React Custom Hooks 패턴 가이드](/posts/react-custom-hooks-patterns-guide/)를 참고하세요.
{: .prompt-info }

**사용 예시:**

```tsx
function AnimatedCard() {
  const { ref, inView } = useInView({
    threshold: 0.2,
    triggerOnce: true
  });

  return (
    <div
      ref={ref as React.RefObject<HTMLDivElement>}
      className={`card ${inView ? 'fade-in' : ''}`}
    >
      <h2>애니메이션 카드</h2>
      <p>화면에 보이면 나타납니다.</p>
    </div>
  );
}
```

### 이미지 지연 로딩 훅

```tsx
import { useEffect, useRef, useState } from 'react';

interface UseLazyImageOptions {
  src: string;
  placeholder?: string;
  rootMargin?: string;
}

function useLazyImage({ src, placeholder = '', rootMargin = '200px' }: UseLazyImageOptions) {
  const imgRef = useRef<HTMLImageElement>(null);
  const [imageSrc, setImageSrc] = useState(placeholder);
  const [isLoaded, setIsLoaded] = useState(false);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    const img = imgRef.current;
    if (!img) return;

    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          // 이미지 프리로드
          const tempImg = new Image();

          tempImg.onload = () => {
            setImageSrc(src);
            setIsLoaded(true);
          };

          tempImg.onerror = () => {
            setError('이미지 로드 실패');
          };

          tempImg.src = src;
          observer.disconnect();
        }
      },
      { rootMargin }
    );

    observer.observe(img);

    return () => observer.disconnect();
  }, [src, rootMargin]);

  return { imgRef, imageSrc, isLoaded, error };
}

// 사용 예시
function LazyImage({ src, alt, placeholder }: { src: string; alt: string; placeholder?: string }) {
  const { imgRef, imageSrc, isLoaded, error } = useLazyImage({
    src,
    placeholder
  });

  if (error) {
    return <div className="image-error">{error}</div>;
  }

  return (
    <img
      ref={imgRef}
      src={imageSrc}
      alt={alt}
      className={`lazy-image ${isLoaded ? 'loaded' : 'loading'}`}
    />
  );
}
```

### 무한 스크롤 훅

```tsx
import { useEffect, useRef, useCallback, useState } from 'react';

interface UseInfiniteScrollOptions {
  hasMore: boolean;
  isLoading: boolean;
  onLoadMore: () => void;
  rootMargin?: string;
}

function useInfiniteScroll({
  hasMore,
  isLoading,
  onLoadMore,
  rootMargin = '100px'
}: UseInfiniteScrollOptions) {
  const sentinelRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    const sentinel = sentinelRef.current;
    if (!sentinel || !hasMore || isLoading) return;

    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          onLoadMore();
        }
      },
      { rootMargin }
    );

    observer.observe(sentinel);

    return () => observer.disconnect();
  }, [hasMore, isLoading, onLoadMore, rootMargin]);

  return sentinelRef;
}

// 사용 예시
function PostList() {
  const [posts, setPosts] = useState<Post[]>([]);
  const [page, setPage] = useState(1);
  const [isLoading, setIsLoading] = useState(false);
  const [hasMore, setHasMore] = useState(true);

  const loadMore = useCallback(async () => {
    setIsLoading(true);
    try {
      const newPosts = await fetchPosts(page + 1);
      setPosts(prev => [...prev, ...newPosts]);
      setPage(prev => prev + 1);
      setHasMore(newPosts.length > 0);
    } finally {
      setIsLoading(false);
    }
  }, [page]);

  const sentinelRef = useInfiniteScroll({
    hasMore,
    isLoading,
    onLoadMore: loadMore
  });

  return (
    <div className="post-list">
      {posts.map(post => (
        <PostItem key={post.id} post={post} />
      ))}

      <div ref={sentinelRef} className="sentinel">
        {isLoading && <LoadingSpinner />}
        {!hasMore && <p>더 이상 포스트가 없습니다.</p>}
      </div>
    </div>
  );
}
```

### react-intersection-observer 라이브러리

직접 구현 대신 검증된 라이브러리를 사용할 수도 있습니다.

```bash
npm install react-intersection-observer
```

```tsx
import { useInView } from 'react-intersection-observer';

function Component() {
  const { ref, inView, entry } = useInView({
    threshold: 0.5,
    triggerOnce: true,
    rootMargin: '100px'
  });

  return (
    <div ref={ref}>
      {inView ? '화면에 보입니다!' : '아직 안 보입니다'}
    </div>
  );
}

// 콜백 방식
function ComponentWithCallback() {
  const { ref } = useInView({
    onChange: (inView, entry) => {
      console.log('가시성 변경:', inView);
    }
  });

  return <div ref={ref}>콘텐츠</div>;
}
```

---

## 성능 비교: scroll 이벤트 vs Intersection Observer

### 측정 코드

```javascript
// scroll 이벤트 방식
function measureScrollPerformance() {
  const elements = document.querySelectorAll('.target');
  let callCount = 0;

  console.time('scroll-method');

  window.addEventListener('scroll', () => {
    callCount++;
    elements.forEach(el => {
      const rect = el.getBoundingClientRect();
      const isVisible = rect.top < window.innerHeight && rect.bottom > 0;
      el.classList.toggle('visible', isVisible);
    });
  });

  // 5초 후 측정 종료
  setTimeout(() => {
    console.timeEnd('scroll-method');
    console.log('scroll 이벤트 호출 횟수:', callCount);
  }, 5000);
}

// Intersection Observer 방식
function measureObserverPerformance() {
  const elements = document.querySelectorAll('.target');
  let callCount = 0;

  console.time('observer-method');

  const observer = new IntersectionObserver((entries) => {
    callCount++;
    entries.forEach(entry => {
      entry.target.classList.toggle('visible', entry.isIntersecting);
    });
  });

  elements.forEach(el => observer.observe(el));

  // 5초 후 측정 종료
  setTimeout(() => {
    console.timeEnd('observer-method');
    console.log('observer 콜백 호출 횟수:', callCount);
  }, 5000);
}
```

### 성능 비교 결과

| 항목 | scroll 이벤트 | Intersection Observer |
|------|--------------|----------------------|
| **콜백 호출 빈도** | 수백 회/초 | 상태 변경 시에만 |
| **메인 스레드 점유** | 높음 | 낮음 (비동기) |
| **리플로우 발생** | 매 호출마다 | 없음 |
| **CPU 사용량** | 높음 | 낮음 |
| **배터리 소모** | 높음 | 낮음 |
| **추가 최적화 필요** | 디바운스/스로틀 필수 | 불필요 |

### Chrome DevTools로 성능 확인

1. DevTools 열기 (F12)
2. Performance 탭 이동
3. Record 시작
4. 스크롤 동작 수행
5. Record 중지 후 분석

Intersection Observer 사용 시 `Scripting` 시간이 현저히 감소하고, `Idle` 시간이 증가하는 것을 확인할 수 있습니다.

---

## 브라우저 지원과 폴리필

### 브라우저 지원 현황

Intersection Observer API는 모든 모던 브라우저에서 지원됩니다.

| 브라우저 | 지원 버전 |
|---------|---------|
| Chrome | 51+ |
| Firefox | 55+ |
| Safari | 12.1+ |
| Edge | 15+ |
| Opera | 38+ |
| iOS Safari | 12.2+ |
| Android Chrome | 51+ |

> **IE 11**은 지원하지 않습니다. IE 지원이 필요한 경우 폴리필을 사용하세요.
{: .prompt-warning }

### 폴리필 적용

```bash
npm install intersection-observer
```

```javascript
// 엔트리 포인트에서 import
import 'intersection-observer';

// 또는 조건부 로드
if (!('IntersectionObserver' in window)) {
  import('intersection-observer');
}
```

**CDN 사용:**

```html
<script src="https://polyfill.io/v3/polyfill.min.js?features=IntersectionObserver"></script>
```

### 기능 감지

```javascript
function supportsIntersectionObserver() {
  return (
    'IntersectionObserver' in window &&
    'IntersectionObserverEntry' in window &&
    'intersectionRatio' in window.IntersectionObserverEntry.prototype
  );
}

// 사용
if (supportsIntersectionObserver()) {
  setupLazyLoading();
} else {
  // 폴백 또는 폴리필 로드
  loadPolyfillAndRetry();
}
```

---

## 주의사항과 베스트 프랙티스

### 1. 옵저버 정리 (메모리 누수 방지)

> **심화 학습**: 메모리 누수의 원인과 JavaScript 가비지 컬렉션 동작 원리에 대해 더 알고 싶다면 [JavaScript 메모리 관리와 가비지 컬렉션 가이드](/posts/javascript-memory-management-garbage-collection-guide/)를 참고하세요.
{: .prompt-info }

```javascript
// 잘못된 예: 옵저버를 정리하지 않음
function BadComponent() {
  useEffect(() => {
    const observer = new IntersectionObserver(callback);
    observer.observe(element);
    // cleanup 없음 - 메모리 누수!
  }, []);
}

// 올바른 예: cleanup 함수에서 disconnect
function GoodComponent() {
  useEffect(() => {
    const observer = new IntersectionObserver(callback);
    observer.observe(element);

    return () => observer.disconnect(); // 정리
  }, []);
}
```

### 2. 단일 옵저버로 여러 요소 관찰

```javascript
// 비효율적: 요소마다 새 옵저버 생성
document.querySelectorAll('.lazy').forEach(el => {
  const observer = new IntersectionObserver(callback); // 매번 새 옵저버
  observer.observe(el);
});

// 효율적: 하나의 옵저버로 모든 요소 관찰
const observer = new IntersectionObserver(callback);
document.querySelectorAll('.lazy').forEach(el => {
  observer.observe(el);
});
```

### 3. 조건부 관찰 해제

```javascript
const observer = new IntersectionObserver((entries, obs) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      // 작업 수행
      loadImage(entry.target);

      // 더 이상 관찰 필요 없으면 해제
      obs.unobserve(entry.target);
    }
  });
});
```

### 4. rootMargin 활용 팁

```javascript
// 이미지 프리로드: 뷰포트 아래 미리 로드
{ rootMargin: '0px 0px 200px 0px' }

// 스크롤 애니메이션: 요소가 더 올라왔을 때 시작
{ rootMargin: '0px 0px -100px 0px' }

// 섹션 네비게이션: 화면 중앙 기준
{ rootMargin: '-40% 0px -40% 0px' }
```

### 5. threshold 적절히 설정

```javascript
// 단순 가시성 체크: 0
{ threshold: 0 }

// 의미 있게 보일 때: 0.2 ~ 0.5
{ threshold: 0.3 }

// 완전히 보일 때만: 1.0
{ threshold: 1.0 }

// 진행률 추적: 배열
{ threshold: [0, 0.25, 0.5, 0.75, 1.0] }
```

### 6. 동적으로 추가되는 요소 처리

```javascript
// MutationObserver와 함께 사용
const intersectionObserver = new IntersectionObserver(handleIntersection);

const mutationObserver = new MutationObserver((mutations) => {
  mutations.forEach(mutation => {
    mutation.addedNodes.forEach(node => {
      if (node.nodeType === 1 && node.matches('.lazy')) {
        intersectionObserver.observe(node);
      }
    });
  });
});

mutationObserver.observe(document.body, {
  childList: true,
  subtree: true
});
```

---

> **더 알아보기**: 이미지 지연 로딩 외에도 다양한 웹 성능 최적화 기법이 있습니다. [웹 성능 최적화 완벽 가이드](/posts/web-performance-optimization-guide/)에서 Core Web Vitals 개선과 렌더링 최적화 전략을 확인하세요.
{: .prompt-tip }

## 핵심 정리

### Intersection Observer의 장점

| 장점 | 설명 |
|------|------|
| **비동기 처리** | 메인 스레드를 블로킹하지 않아 부드러운 스크롤 |
| **자동 최적화** | 브라우저가 내부적으로 효율적으로 처리 |
| **정확한 감지** | 교차 비율을 퍼센트로 정확하게 제공 |
| **간단한 API** | scroll 이벤트 + getBoundingClientRect 대비 간결 |
| **배터리 효율** | 불필요한 계산 최소화로 모바일 친화적 |

### 주요 활용 사례

| 사례 | 설명 |
|------|------|
| **이미지 지연 로딩** | 뷰포트에 가까워지면 이미지 로드 |
| **무한 스크롤** | 센티널 요소 감지로 추가 콘텐츠 로드 |
| **스크롤 애니메이션** | 요소 등장 시 애니메이션 트리거 |
| **광고 노출 측정** | 가시성 및 노출 시간 추적 |
| **섹션 네비게이션** | 현재 섹션에 맞는 메뉴 하이라이트 |
| **Sticky 헤더** | 스크롤 상태에 따른 헤더 스타일 변경 |

### 베스트 프랙티스

1. **하나의 옵저버로 여러 요소 관찰** - 리소스 효율성
2. **반드시 cleanup** - `disconnect()` 또는 `unobserve()` 호출
3. **rootMargin으로 미리 감지** - 사용자 경험 향상
4. **적절한 threshold 설정** - 필요한 만큼만
5. **triggerOnce 패턴** - 한 번만 필요한 경우 관찰 해제

> Intersection Observer API는 스크롤 기반 기능 구현의 표준 솔루션입니다. scroll 이벤트 대신 사용하면 성능과 코드 품질 모두 개선됩니다.
{: .prompt-info }

---

## 참고 자료

- [MDN - Intersection Observer API](https://developer.mozilla.org/ko/docs/Web/API/Intersection_Observer_API)
- [W3C - Intersection Observer](https://www.w3.org/TR/intersection-observer/)
- [Google Developers - IntersectionObserver's Coming into View](https://developers.google.com/web/updates/2016/04/intersectionobserver)
- [web.dev - Lazy loading images](https://web.dev/lazy-loading-images/)
- [react-intersection-observer](https://github.com/thebuilder/react-intersection-observer)
- [Can I Use - Intersection Observer](https://caniuse.com/intersectionobserver)
