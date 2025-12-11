---
title: "JavaScript 디바운스(Debounce)와 스로틀(Throttle) 완벽 가이드 - 이벤트 최적화의 핵심"
description: "JavaScript 디바운스와 스로틀로 이벤트 성능을 최적화하세요. 검색 자동완성, 스크롤, 리사이즈 최적화 구현 방법과 React 커스텀 훅까지 실전 코드로 배웁니다."
date: 2025-12-11 10:00:00 +0900
categories: [Frontend, JavaScript]
tags: [javascript, debounce, throttle, performance, web-performance, event-optimization, lodash, react-hooks, custom-hooks, search-autocomplete, scroll-optimization, infinite-scroll, form-validation, requestAnimationFrame]
---

## 개요: 왜 디바운스와 스로틀이 필요한가?

웹 애플리케이션에서 사용자 인터랙션을 처리하다 보면, 특정 이벤트가 **너무 자주 발생**하는 상황을 마주하게 됩니다. `scroll`, `resize`, `input`, `mousemove` 같은 이벤트는 초당 수십에서 수백 번까지 발생할 수 있습니다.

### 고빈도 이벤트의 문제점

```javascript
// 스크롤할 때마다 실행 - 심각한 성능 문제
window.addEventListener('scroll', () => {
  console.log('스크롤 이벤트 발생!');
  // 복잡한 계산이나 DOM 조작이 매번 실행됨
});

// 입력할 때마다 API 호출 - 서버 부하 급증
searchInput.addEventListener('input', (e) => {
  fetch(`/api/search?q=${e.target.value}`);
  // 'hello' 입력시 5번의 API 호출: h, he, hel, hell, hello
});
```

이런 상황에서 발생하는 문제들:

1. **성능 저하**: 짧은 시간에 수많은 함수가 실행되어 UI가 버벅거림
2. **불필요한 API 호출**: 서버 부하 증가, 비용 낭비, Rate Limit 초과 위험
3. **리소스 낭비**: CPU, 메모리, 네트워크 대역폭의 비효율적 사용
4. **사용자 경험 저하**: 응답 지연, 프레임 드롭

**디바운스(Debounce)**와 **스로틀(Throttle)**은 이런 문제를 해결하는 핵심 기법입니다. 디바운스와 스로틀이 setTimeout을 사용하는 방식을 이해하려면 [JavaScript Event Loop](/posts/javascript-event-loop/)에 대한 이해가 도움이 됩니다.

### 핵심 개념 한눈에 보기

| 기법 | 핵심 동작 | 비유 |
|-----|---------|-----|
| **디바운스** | 마지막 이벤트 후 일정 시간 대기 후 실행 | 엘리베이터 문 - 사람이 계속 타면 문이 안 닫힘 |
| **스로틀** | 일정 시간당 최대 1회만 실행 | 지하철 배차 - 아무리 사람이 몰려도 정해진 시간에만 출발 |

## 디바운스(Debounce) 완벽 이해

디바운스는 연속된 이벤트 중 **마지막 이벤트만 처리**합니다. 이벤트가 발생할 때마다 타이머를 초기화하고, 설정한 시간 동안 새 이벤트가 없으면 함수를 실행합니다.

### 디바운스 동작 원리

```
이벤트 발생 타임라인 (300ms 디바운스):

시간: 0ms   100ms  200ms  350ms  400ms  700ms  1000ms
이벤트:  A      B      C      -      D      -      -
타이머: [시작]  [리셋]  [리셋]  -    [리셋]  -    [실행!]
                                              (D 실행)

설명:
- A 발생: 300ms 타이머 시작
- B 발생: 타이머 리셋 (다시 300ms)
- C 발생: 타이머 리셋 (다시 300ms)
- D 발생: 타이머 리셋 (다시 300ms)
- 700ms 이후 이벤트 없음 → 1000ms에 D 실행
```

### 기본 디바운스 구현

```javascript
function debounce(func, delay) {
  let timeoutId = null;

  return function(...args) {
    // 이전 타이머 취소
    if (timeoutId) {
      clearTimeout(timeoutId);
    }

    // 새 타이머 설정
    timeoutId = setTimeout(() => {
      func.apply(this, args);
      timeoutId = null;
    }, delay);
  };
}

// 사용 예시
const handleSearch = debounce((query) => {
  console.log('API 호출:', query);
  // fetch(`/api/search?q=${query}`);
}, 300);

// 테스트
handleSearch('h');      // 타이머 시작
handleSearch('he');     // 타이머 리셋
handleSearch('hel');    // 타이머 리셋
handleSearch('hell');   // 타이머 리셋
handleSearch('hello');  // 타이머 리셋 → 300ms 후 'hello'로 1번만 호출
```

위 디바운스 함수는 클로저를 활용하여 `timeoutId`를 외부에서 접근하지 못하게 보호합니다. 클로저의 동작 원리에 대해서는 [JavaScript 클로저 완벽 가이드](/posts/javascript-closure-complete-guide/)를 참고하세요.

### Leading vs Trailing 디바운스

디바운스는 함수 실행 시점에 따라 두 가지 모드가 있습니다:

| 모드 | 실행 시점 | 사용 사례 |
|-----|---------|---------|
| **Trailing (기본)** | 이벤트 종료 후 실행 | 검색 자동완성, 폼 유효성 검사 |
| **Leading** | 첫 이벤트에서 즉시 실행 | 버튼 클릭, 즉각적 피드백 필요 시 |
| **Both** | 처음과 끝 모두 실행 | 실시간 미리보기 + 최종 저장 |

```javascript
function debounce(func, delay, options = {}) {
  let timeoutId = null;
  let lastArgs = null;
  const { leading = false, trailing = true } = options;

  return function(...args) {
    lastArgs = args;
    const isFirstCall = timeoutId === null;

    // Leading: 첫 호출 시 즉시 실행
    if (leading && isFirstCall) {
      func.apply(this, args);
    }

    // 이전 타이머 취소
    if (timeoutId) {
      clearTimeout(timeoutId);
    }

    // 새 타이머 설정
    timeoutId = setTimeout(() => {
      // Trailing: 마지막에 실행 (leading이 이미 실행했으면 스킵)
      if (trailing && !(leading && isFirstCall)) {
        func.apply(this, lastArgs);
      }
      timeoutId = null;
      lastArgs = null;
    }, delay);
  };
}

// Leading 디바운스: 첫 클릭만 처리
const handleClick = debounce(
  () => console.log('버튼 클릭!'),
  1000,
  { leading: true, trailing: false }
);

// Both: 처음과 끝 모두 실행
const handleInput = debounce(
  (value) => console.log('입력:', value),
  500,
  { leading: true, trailing: true }
);
```

### 실전 활용: 검색 자동완성

```javascript
class SearchAutocomplete {
  constructor(inputElement, resultsContainer) {
    this.input = inputElement;
    this.results = resultsContainer;
    this.abortController = null;

    // 디바운스된 검색 함수
    this.debouncedSearch = debounce(
      this.performSearch.bind(this),
      300
    );

    this.init();
  }

  init() {
    this.input.addEventListener('input', (e) => {
      const query = e.target.value.trim();

      if (query.length < 2) {
        this.clearResults();
        return;
      }

      this.showLoading();
      this.debouncedSearch(query);
    });
  }

  async performSearch(query) {
    // 이전 요청 취소
    if (this.abortController) {
      this.abortController.abort();
    }
    this.abortController = new AbortController();

    try {
      const response = await fetch(
        `/api/search?q=${encodeURIComponent(query)}`,
        { signal: this.abortController.signal }
      );

      if (!response.ok) throw new Error('검색 실패');

      const data = await response.json();
      this.renderResults(data.results);
    } catch (error) {
      if (error.name !== 'AbortError') {
        this.showError('검색 중 오류가 발생했습니다.');
      }
    }
  }

  showLoading() {
    this.results.innerHTML = '<div class="loading">검색 중...</div>';
  }

  renderResults(items) {
    if (items.length === 0) {
      this.results.innerHTML = '<div class="no-results">결과 없음</div>';
      return;
    }

    this.results.innerHTML = items
      .map(item => `<div class="result-item">${item.title}</div>`)
      .join('');
  }

  clearResults() {
    this.results.innerHTML = '';
  }

  showError(message) {
    this.results.innerHTML = `<div class="error">${message}</div>`;
  }
}

// 사용
const autocomplete = new SearchAutocomplete(
  document.getElementById('search-input'),
  document.getElementById('search-results')
);
```

### 실전 활용: 폼 유효성 검사

```javascript
class FormValidator {
  constructor(form) {
    this.form = form;
    this.validators = new Map();

    // 입력별 디바운스된 검증 함수 생성
    this.debouncedValidate = debounce(
      this.validateField.bind(this),
      400
    );
  }

  addValidator(fieldName, validatorFn, errorMessage) {
    this.validators.set(fieldName, { validatorFn, errorMessage });

    const field = this.form.querySelector(`[name="${fieldName}"]`);
    if (field) {
      field.addEventListener('input', () => {
        this.debouncedValidate(fieldName, field.value);
      });
    }
  }

  async validateField(fieldName, value) {
    const validator = this.validators.get(fieldName);
    if (!validator) return;

    const field = this.form.querySelector(`[name="${fieldName}"]`);
    const errorElement = this.form.querySelector(`[data-error="${fieldName}"]`);

    try {
      const isValid = await validator.validatorFn(value);

      if (isValid) {
        field.classList.remove('invalid');
        field.classList.add('valid');
        if (errorElement) errorElement.textContent = '';
      } else {
        field.classList.remove('valid');
        field.classList.add('invalid');
        if (errorElement) errorElement.textContent = validator.errorMessage;
      }

      return isValid;
    } catch (error) {
      console.error('검증 오류:', error);
      return false;
    }
  }
}

// 사용 예시
const validator = new FormValidator(document.getElementById('signup-form'));

// 이메일 중복 검사 (서버 확인)
validator.addValidator(
  'email',
  async (value) => {
    if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value)) return false;

    const response = await fetch(`/api/check-email?email=${value}`);
    const { available } = await response.json();
    return available;
  },
  '이미 사용 중인 이메일입니다.'
);

// 사용자명 검사
validator.addValidator(
  'username',
  async (value) => {
    if (value.length < 3 || value.length > 20) return false;
    if (!/^[a-zA-Z0-9_]+$/.test(value)) return false;

    const response = await fetch(`/api/check-username?username=${value}`);
    const { available } = await response.json();
    return available;
  },
  '사용할 수 없는 사용자명입니다.'
);
```

## 스로틀(Throttle) 완벽 이해

스로틀은 **일정 시간 간격으로 최대 1회만** 함수를 실행합니다. 아무리 이벤트가 많이 발생해도 설정한 간격보다 자주 실행되지 않습니다.

### 스로틀 동작 원리

```
이벤트 발생 타임라인 (200ms 스로틀):

시간: 0ms   50ms   100ms  150ms  200ms  250ms  300ms  350ms  400ms
이벤트:  A      B      C      D      E      F      G      H      I
실행:  [A]    -      -      -     [E]    -      -      -     [I]

설명:
- 0ms: A 발생 → 즉시 실행, 200ms 동안 잠금
- 50~150ms: B, C, D 발생 → 잠금 상태라 무시
- 200ms: E 발생 → 잠금 해제됨, 실행, 다시 200ms 잠금
- 250~350ms: F, G, H 발생 → 잠금 상태라 무시
- 400ms: I 발생 → 잠금 해제됨, 실행
```

### 기본 스로틀 구현

```javascript
function throttle(func, limit) {
  let inThrottle = false;

  return function(...args) {
    if (!inThrottle) {
      func.apply(this, args);
      inThrottle = true;

      setTimeout(() => {
        inThrottle = false;
      }, limit);
    }
  };
}

// 사용 예시
const handleScroll = throttle(() => {
  console.log('스크롤 위치:', window.scrollY);
}, 200);

window.addEventListener('scroll', handleScroll);
// 스크롤해도 200ms당 최대 1번만 실행
```

### Leading vs Trailing 스로틀

스로틀도 디바운스처럼 leading/trailing 옵션을 지원할 수 있습니다:

```javascript
function throttle(func, limit, options = {}) {
  let lastTime = 0;
  let timeoutId = null;
  const { leading = true, trailing = true } = options;

  return function(...args) {
    const now = Date.now();
    const remaining = limit - (now - lastTime);

    // 충분한 시간이 지났거나 첫 호출
    if (remaining <= 0 || remaining > limit) {
      if (timeoutId) {
        clearTimeout(timeoutId);
        timeoutId = null;
      }

      if (leading) {
        lastTime = now;
        func.apply(this, args);
      }
    } else if (trailing && !timeoutId) {
      // Trailing: 마지막 호출 예약
      timeoutId = setTimeout(() => {
        lastTime = leading ? Date.now() : 0;
        timeoutId = null;
        func.apply(this, args);
      }, remaining);
    }
  };
}

// Leading만: 첫 이벤트에서 즉시 실행
const handleMouseMove = throttle(
  (e) => console.log(e.clientX, e.clientY),
  100,
  { leading: true, trailing: false }
);

// Trailing만: 간격 끝에서 실행
const handleResize = throttle(
  () => console.log('리사이즈 완료'),
  300,
  { leading: false, trailing: true }
);
```

### 실전 활용: 스크롤 이벤트 최적화

```javascript
class ScrollTracker {
  constructor() {
    this.sections = document.querySelectorAll('section[id]');
    this.navLinks = document.querySelectorAll('.nav-link');
    this.progressBar = document.getElementById('progress-bar');

    // 스로틀된 스크롤 핸들러
    this.throttledScroll = throttle(
      this.handleScroll.bind(this),
      100
    );

    window.addEventListener('scroll', this.throttledScroll);
  }

  handleScroll() {
    this.updateProgress();
    this.updateActiveSection();
    this.checkLazyLoad();
  }

  updateProgress() {
    const scrollTop = window.scrollY;
    const docHeight = document.documentElement.scrollHeight - window.innerHeight;
    const progress = (scrollTop / docHeight) * 100;

    if (this.progressBar) {
      this.progressBar.style.width = `${progress}%`;
    }
  }

  updateActiveSection() {
    const scrollPosition = window.scrollY + 100;

    this.sections.forEach(section => {
      const sectionTop = section.offsetTop;
      const sectionHeight = section.offsetHeight;
      const sectionId = section.getAttribute('id');

      if (scrollPosition >= sectionTop &&
          scrollPosition < sectionTop + sectionHeight) {
        this.navLinks.forEach(link => {
          link.classList.remove('active');
          if (link.getAttribute('href') === `#${sectionId}`) {
            link.classList.add('active');
          }
        });
      }
    });
  }

  checkLazyLoad() {
    const lazyImages = document.querySelectorAll('img[data-src]');

    lazyImages.forEach(img => {
      if (this.isInViewport(img)) {
        img.src = img.dataset.src;
        img.removeAttribute('data-src');
      }
    });
  }

  isInViewport(element) {
    const rect = element.getBoundingClientRect();
    return (
      rect.top <= window.innerHeight &&
      rect.bottom >= 0
    );
  }
}

// 초기화
const scrollTracker = new ScrollTracker();
```

### 실전 활용: 무한 스크롤

```javascript
class InfiniteScroll {
  constructor(options) {
    this.container = options.container;
    this.loadMore = options.loadMore;
    this.threshold = options.threshold || 200;
    this.isLoading = false;
    this.hasMore = true;
    this.page = 1;

    // 스로틀된 스크롤 체크
    this.throttledCheck = throttle(
      this.checkScroll.bind(this),
      200
    );

    window.addEventListener('scroll', this.throttledCheck);
  }

  checkScroll() {
    if (this.isLoading || !this.hasMore) return;

    const scrollPosition = window.innerHeight + window.scrollY;
    const threshold = document.documentElement.scrollHeight - this.threshold;

    if (scrollPosition >= threshold) {
      this.loadMoreItems();
    }
  }

  async loadMoreItems() {
    this.isLoading = true;
    this.showLoader();

    try {
      const items = await this.loadMore(this.page);

      if (items.length === 0) {
        this.hasMore = false;
        this.showEndMessage();
        return;
      }

      this.renderItems(items);
      this.page++;
    } catch (error) {
      console.error('로드 실패:', error);
      this.showError();
    } finally {
      this.isLoading = false;
      this.hideLoader();
    }
  }

  renderItems(items) {
    const fragment = document.createDocumentFragment();

    items.forEach(item => {
      const element = document.createElement('div');
      element.className = 'item';
      element.innerHTML = `
        <h3>${item.title}</h3>
        <p>${item.description}</p>
      `;
      fragment.appendChild(element);
    });

    this.container.appendChild(fragment);
  }

  showLoader() {
    const loader = document.createElement('div');
    loader.id = 'infinite-loader';
    loader.className = 'loader';
    loader.textContent = '로딩 중...';
    this.container.appendChild(loader);
  }

  hideLoader() {
    const loader = document.getElementById('infinite-loader');
    if (loader) loader.remove();
  }

  showEndMessage() {
    const message = document.createElement('div');
    message.className = 'end-message';
    message.textContent = '모든 항목을 불러왔습니다.';
    this.container.appendChild(message);
  }

  showError() {
    const error = document.createElement('div');
    error.className = 'error-message';
    error.textContent = '로드 중 오류가 발생했습니다. 다시 시도해주세요.';
    this.container.appendChild(error);
  }

  destroy() {
    window.removeEventListener('scroll', this.throttledCheck);
  }
}

// 사용
const infiniteScroll = new InfiniteScroll({
  container: document.getElementById('items-container'),
  loadMore: async (page) => {
    const response = await fetch(`/api/items?page=${page}&limit=20`);
    const data = await response.json();
    return data.items;
  },
  threshold: 300
});
```

### 실전 활용: 리사이즈 핸들링

```javascript
class ResponsiveHandler {
  constructor() {
    this.breakpoints = {
      mobile: 480,
      tablet: 768,
      desktop: 1024,
      wide: 1440
    };
    this.currentBreakpoint = this.getBreakpoint();

    // 스로틀된 리사이즈 핸들러
    this.throttledResize = throttle(
      this.handleResize.bind(this),
      250
    );

    window.addEventListener('resize', this.throttledResize);
  }

  getBreakpoint() {
    const width = window.innerWidth;

    if (width < this.breakpoints.mobile) return 'mobile';
    if (width < this.breakpoints.tablet) return 'tablet';
    if (width < this.breakpoints.desktop) return 'desktop';
    if (width < this.breakpoints.wide) return 'wide';
    return 'ultrawide';
  }

  handleResize() {
    const newBreakpoint = this.getBreakpoint();

    // 브레이크포인트가 변경된 경우에만 처리
    if (newBreakpoint !== this.currentBreakpoint) {
      this.currentBreakpoint = newBreakpoint;
      this.onBreakpointChange(newBreakpoint);
    }

    // 항상 실행되는 리사이즈 로직
    this.updateLayout();
  }

  onBreakpointChange(breakpoint) {
    console.log('브레이크포인트 변경:', breakpoint);

    // 브레이크포인트별 로직
    document.body.dataset.breakpoint = breakpoint;

    // 커스텀 이벤트 발생
    window.dispatchEvent(
      new CustomEvent('breakpointchange', { detail: { breakpoint } })
    );
  }

  updateLayout() {
    // 레이아웃 재계산
    const elements = document.querySelectorAll('[data-responsive]');

    elements.forEach(element => {
      // 요소별 리사이즈 로직
      const rect = element.getBoundingClientRect();
      element.style.setProperty('--element-width', `${rect.width}px`);
    });
  }
}

// 초기화
const responsiveHandler = new ResponsiveHandler();

// 브레이크포인트 변경 감지
window.addEventListener('breakpointchange', (e) => {
  console.log('새 브레이크포인트:', e.detail.breakpoint);
});
```

## 디바운스 vs 스로틀 비교

### 동작 차이 시각화

```
1초 동안 연속 입력 (|)이 있을 때:

입력:        ||||||||||||||||||||||||||||||||||
시간:        0                               1000ms

디바운스 (300ms):
실행:        --------------------------------[O]
             (마지막 입력 후 300ms 뒤 1번 실행)

스로틀 (300ms):
실행:        [O]--------[O]--------[O]--------[O]
             (300ms마다 최대 1번씩 실행)
```

### 상세 비교표

| 특성 | 디바운스 | 스로틀 |
|-----|---------|-------|
| **실행 시점** | 이벤트 종료 후 | 일정 간격마다 |
| **실행 횟수** | 연속 이벤트당 1회 | 간격당 최대 1회 |
| **첫 이벤트 반응** | 지연됨 (trailing) | 즉시 (leading) |
| **중간 상태 반영** | 안 됨 | 됨 |
| **예측 가능성** | 낮음 (이벤트 종료 시점 의존) | 높음 (고정 간격) |

### 사용 사례별 선택 가이드

| 사용 사례 | 권장 기법 | 이유 |
|----------|---------|-----|
| 검색 자동완성 | 디바운스 | 타이핑 완료 후 검색해야 함 |
| 폼 유효성 검사 | 디바운스 | 입력 완료 후 검증해야 함 |
| 자동 저장 | 디바운스 | 편집 완료 후 저장해야 함 |
| 스크롤 위치 추적 | 스로틀 | 중간 위치도 알아야 함 |
| 무한 스크롤 | 스로틀 | 스크롤 중에도 로드 체크 필요 |
| 윈도우 리사이즈 | 스로틀 | 중간 크기도 반영해야 함 |
| 마우스 이동 추적 | 스로틀 | 중간 위치도 필요함 |
| 버튼 중복 클릭 방지 | 디바운스 (leading) | 첫 클릭만 처리 |
| 게임 입력 처리 | 스로틀 | 일정 간격 입력 필요 |

### 선택 기준 플로우차트

```
Q: 이벤트가 끝난 후 최종 결과만 필요한가?
├── YES → 디바운스 사용
│         예: 검색, 폼 검증, 자동 저장
│
└── NO → Q: 이벤트 진행 중에도 주기적 업데이트가 필요한가?
          ├── YES → 스로틀 사용
          │         예: 스크롤, 리사이즈, 드래그
          │
          └── NO → Q: 첫 이벤트에만 반응하면 되는가?
                    ├── YES → 디바운스 (leading) 사용
                    │         예: 버튼 클릭, 토글
                    │
                    └── NO → 상황에 맞게 선택
```

## 라이브러리 활용

직접 구현도 가능하지만, 프로덕션 환경에서는 검증된 라이브러리를 사용하는 것이 안전합니다.

### Lodash의 debounce/throttle

```bash
npm install lodash.debounce lodash.throttle
# 또는 전체 lodash
npm install lodash
```

```javascript
import debounce from 'lodash.debounce';
import throttle from 'lodash.throttle';

// Lodash debounce
const debouncedSearch = debounce(
  (query) => {
    console.log('검색:', query);
  },
  300,
  {
    leading: false,   // 첫 호출 시 실행 안 함
    trailing: true,   // 마지막에 실행 (기본값)
    maxWait: 1000     // 최대 대기 시간 - 이 시간이 지나면 강제 실행
  }
);

// maxWait 옵션 설명:
// 디바운스가 계속 리셋되어도 maxWait 시간이 지나면 강제 실행
// 예: 300ms 디바운스 + 1000ms maxWait
// → 사용자가 계속 입력해도 1초마다 최소 1번은 실행됨

// Lodash throttle
const throttledScroll = throttle(
  () => {
    console.log('스크롤:', window.scrollY);
  },
  200,
  {
    leading: true,    // 첫 호출 시 즉시 실행 (기본값)
    trailing: true    // 마지막에도 실행 (기본값)
  }
);

// 취소 기능
debouncedSearch('검색어');
debouncedSearch.cancel();  // 대기 중인 실행 취소

// 즉시 실행
debouncedSearch.flush();   // 대기 중인 함수 즉시 실행
```

### Lodash vs 직접 구현 비교

| 특성 | 직접 구현 | Lodash |
|-----|---------|--------|
| **번들 크기** | 작음 | ~2KB (개별 임포트 시) |
| **기능** | 기본적 | maxWait, cancel, flush 등 |
| **엣지 케이스 처리** | 직접 해야 함 | 검증됨 |
| **TypeScript 지원** | 직접 작성 | @types/lodash |
| **유지보수** | 직접 | 커뮤니티 |

### 가벼운 대안: throttle-debounce

```bash
npm install throttle-debounce
```

```javascript
import { debounce, throttle } from 'throttle-debounce';

// 더 가벼운 구현 (약 600 bytes)
const debouncedFn = debounce(300, (value) => {
  console.log(value);
});

const throttledFn = throttle(200, (value) => {
  console.log(value);
});

// 옵션
const debouncedWithOptions = debounce(
  300,
  (value) => console.log(value),
  { atBegin: true }  // leading 모드
);
```

## React에서의 활용

React 환경에서는 컴포넌트 라이프사이클과 함께 디바운스/스로틀을 관리해야 합니다. 커스텀 훅 패턴에 익숙하지 않다면 [React Custom Hooks 패턴 가이드](/posts/react-custom-hooks-patterns-guide/)를 먼저 읽어보시길 권장합니다.

### useDebounce 커스텀 훅

```tsx
import { useState, useEffect, useRef, useCallback } from 'react';

// 값을 디바운스하는 훅
function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => {
      clearTimeout(timer);
    };
  }, [value, delay]);

  return debouncedValue;
}

// 함수를 디바운스하는 훅
function useDebouncedCallback<T extends (...args: unknown[]) => unknown>(
  callback: T,
  delay: number
): T {
  const callbackRef = useRef(callback);
  const timeoutRef = useRef<NodeJS.Timeout | null>(null);

  // 최신 콜백 유지
  useEffect(() => {
    callbackRef.current = callback;
  }, [callback]);

  // 클린업
  useEffect(() => {
    return () => {
      if (timeoutRef.current) {
        clearTimeout(timeoutRef.current);
      }
    };
  }, []);

  const debouncedCallback = useCallback(
    (...args: Parameters<T>) => {
      if (timeoutRef.current) {
        clearTimeout(timeoutRef.current);
      }

      timeoutRef.current = setTimeout(() => {
        callbackRef.current(...args);
      }, delay);
    },
    [delay]
  ) as T;

  return debouncedCallback;
}

// 사용 예시: 검색 컴포넌트
function SearchComponent() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState<string[]>([]);
  const [isLoading, setIsLoading] = useState(false);

  // 값 디바운스
  const debouncedQuery = useDebounce(query, 300);

  // 디바운스된 쿼리가 변경될 때 검색 실행
  useEffect(() => {
    if (!debouncedQuery.trim()) {
      setResults([]);
      return;
    }

    const searchItems = async () => {
      setIsLoading(true);
      try {
        const response = await fetch(
          `/api/search?q=${encodeURIComponent(debouncedQuery)}`
        );
        const data = await response.json();
        setResults(data.results);
      } catch (error) {
        console.error('검색 오류:', error);
      } finally {
        setIsLoading(false);
      }
    };

    searchItems();
  }, [debouncedQuery]);

  return (
    <div className="search-container">
      <input
        type="text"
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="검색어를 입력하세요"
      />
      {isLoading && <div className="loading">검색 중...</div>}
      <ul className="results">
        {results.map((item, index) => (
          <li key={index}>{item}</li>
        ))}
      </ul>
    </div>
  );
}
```

### useThrottle 커스텀 훅

```tsx
import { useState, useEffect, useRef, useCallback } from 'react';

// 값을 스로틀하는 훅
function useThrottle<T>(value: T, limit: number): T {
  const [throttledValue, setThrottledValue] = useState(value);
  const lastRan = useRef(Date.now());

  useEffect(() => {
    const handler = setTimeout(() => {
      if (Date.now() - lastRan.current >= limit) {
        setThrottledValue(value);
        lastRan.current = Date.now();
      }
    }, limit - (Date.now() - lastRan.current));

    return () => {
      clearTimeout(handler);
    };
  }, [value, limit]);

  return throttledValue;
}

// 함수를 스로틀하는 훅
function useThrottledCallback<T extends (...args: unknown[]) => unknown>(
  callback: T,
  limit: number
): T {
  const callbackRef = useRef(callback);
  const lastRan = useRef(0);
  const timeoutRef = useRef<NodeJS.Timeout | null>(null);

  useEffect(() => {
    callbackRef.current = callback;
  }, [callback]);

  useEffect(() => {
    return () => {
      if (timeoutRef.current) {
        clearTimeout(timeoutRef.current);
      }
    };
  }, []);

  const throttledCallback = useCallback(
    (...args: Parameters<T>) => {
      const now = Date.now();
      const remaining = limit - (now - lastRan.current);

      if (remaining <= 0) {
        lastRan.current = now;
        callbackRef.current(...args);
      } else if (!timeoutRef.current) {
        timeoutRef.current = setTimeout(() => {
          lastRan.current = Date.now();
          timeoutRef.current = null;
          callbackRef.current(...args);
        }, remaining);
      }
    },
    [limit]
  ) as T;

  return throttledCallback;
}

// 사용 예시: 스크롤 위치 추적
function ScrollTracker() {
  const [scrollPosition, setScrollPosition] = useState(0);

  const handleScroll = useThrottledCallback(() => {
    setScrollPosition(window.scrollY);
  }, 100);

  useEffect(() => {
    window.addEventListener('scroll', handleScroll);
    return () => {
      window.removeEventListener('scroll', handleScroll);
    };
  }, [handleScroll]);

  return (
    <div className="scroll-tracker">
      <div className="fixed-indicator">
        스크롤 위치: {scrollPosition}px
      </div>
    </div>
  );
}
```

### useDeferredValue와의 차이점

React 18에서 도입된 `useDeferredValue`는 디바운스/스로틀과 비슷해 보이지만 다른 개념입니다.

```tsx
import { useState, useDeferredValue, useMemo } from 'react';

function ComparisonExample() {
  const [query, setQuery] = useState('');

  // useDeferredValue: React가 자동으로 우선순위 관리
  const deferredQuery = useDeferredValue(query);

  // useDebounce: 고정 시간 지연
  const debouncedQuery = useDebounce(query, 300);

  // useDeferredValue로 무거운 렌더링 최적화
  const expensiveList = useMemo(() => {
    return items.filter(item =>
      item.name.toLowerCase().includes(deferredQuery.toLowerCase())
    );
  }, [deferredQuery]);

  return (
    <div>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
      />
      {/* deferredQuery가 query와 다르면 로딩 표시 */}
      {query !== deferredQuery && <span>검색 중...</span>}
      <ExpensiveList items={expensiveList} />
    </div>
  );
}
```

**주요 차이점:**

| 특성 | useDeferredValue | useDebounce |
|-----|-----------------|-------------|
| **지연 시간** | React가 자동 결정 | 개발자가 직접 설정 |
| **목적** | UI 반응성 유지 | 이벤트 빈도 제어 |
| **사용 사례** | 무거운 렌더링 지연 | API 호출 최적화 |
| **Concurrent Mode** | 활용 | 무관 |
| **네트워크 요청** | 부적합 | 적합 |

**선택 가이드:**

- **API 호출 최적화** → `useDebounce`
- **무거운 목록 필터링** → `useDeferredValue`
- **타이핑 중 렌더링 지연** → `useDeferredValue`
- **검색 자동완성** → `useDebounce`

## 고급 패턴

### requestAnimationFrame 기반 스로틀

`requestAnimationFrame`은 브라우저 렌더링 주기(보통 60fps)에 맞춰 실행되어 애니메이션과 시각적 업데이트에 최적화되어 있습니다. setTimeout, setInterval, requestAnimationFrame의 차이점에 대해서는 [JavaScript 타이머와 애니메이션](/posts/javascript-timers-animation/)에서 자세히 다룹니다.

```javascript
function rafThrottle(callback) {
  let requestId = null;
  let lastArgs = null;

  const later = () => {
    requestId = null;
    callback(...lastArgs);
  };

  return function(...args) {
    lastArgs = args;

    if (requestId === null) {
      requestId = requestAnimationFrame(later);
    }
  };
}

// 사용 예시: 부드러운 스크롤 기반 애니메이션
const parallaxScroll = rafThrottle((scrollY) => {
  const parallaxElements = document.querySelectorAll('.parallax');

  parallaxElements.forEach(element => {
    const speed = element.dataset.speed || 0.5;
    const yPos = -(scrollY * speed);
    element.style.transform = `translate3d(0, ${yPos}px, 0)`;
  });
});

window.addEventListener('scroll', () => {
  parallaxScroll(window.scrollY);
});
```

**rAF 스로틀 vs 일반 스로틀:**

| 특성 | rAF 스로틀 | 일반 스로틀 |
|-----|----------|-----------|
| **간격** | ~16ms (60fps) | 사용자 지정 |
| **렌더링 동기화** | 동기화됨 | 비동기 |
| **백그라운드 탭** | 자동 중지 | 계속 실행 |
| **적합한 용도** | 시각적 업데이트 | 범용 |

### 취소 가능한 디바운스/스로틀

```javascript
function cancellableDebounce(func, delay) {
  let timeoutId = null;
  let cancelled = false;

  const debouncedFn = function(...args) {
    if (cancelled) return;

    if (timeoutId) {
      clearTimeout(timeoutId);
    }

    timeoutId = setTimeout(() => {
      if (!cancelled) {
        func.apply(this, args);
      }
      timeoutId = null;
    }, delay);
  };

  // 대기 중인 실행 취소
  debouncedFn.cancel = function() {
    if (timeoutId) {
      clearTimeout(timeoutId);
      timeoutId = null;
    }
  };

  // 즉시 실행
  debouncedFn.flush = function(...args) {
    if (timeoutId) {
      clearTimeout(timeoutId);
      timeoutId = null;
      func.apply(this, args);
    }
  };

  // 완전히 비활성화
  debouncedFn.dispose = function() {
    cancelled = true;
    debouncedFn.cancel();
  };

  return debouncedFn;
}

// 사용 예시
const debouncedSave = cancellableDebounce(saveDocument, 1000);

// 문서 편집
document.addEventListener('input', () => {
  debouncedSave();
});

// 저장 버튼 클릭 - 즉시 저장
saveButton.addEventListener('click', () => {
  debouncedSave.flush();
});

// 페이지 이탈 - 정리
window.addEventListener('beforeunload', () => {
  debouncedSave.dispose();
});
```

### AbortController와 함께 사용하기

비동기 작업을 디바운스/스로틀할 때는 이전 요청을 취소하는 것이 중요합니다.

```javascript
function debouncedFetch(url, delay) {
  let timeoutId = null;
  let abortController = null;

  return async function(params) {
    // 이전 타이머 취소
    if (timeoutId) {
      clearTimeout(timeoutId);
    }

    // 이전 요청 취소
    if (abortController) {
      abortController.abort();
    }

    return new Promise((resolve, reject) => {
      timeoutId = setTimeout(async () => {
        abortController = new AbortController();

        try {
          const response = await fetch(
            `${url}?${new URLSearchParams(params)}`,
            { signal: abortController.signal }
          );

          if (!response.ok) {
            throw new Error(`HTTP ${response.status}`);
          }

          const data = await response.json();
          resolve(data);
        } catch (error) {
          if (error.name !== 'AbortError') {
            reject(error);
          }
        } finally {
          abortController = null;
        }
      }, delay);
    });
  };
}

// 사용 예시
const searchAPI = debouncedFetch('/api/search', 300);

async function handleSearch(query) {
  try {
    const results = await searchAPI({ q: query });
    console.log('결과:', results);
  } catch (error) {
    if (error.name !== 'AbortError') {
      console.error('검색 오류:', error);
    }
  }
}

// 입력할 때마다 호출
searchInput.addEventListener('input', (e) => {
  handleSearch(e.target.value);
});
```

### React 훅으로 AbortController 통합

```tsx
import { useCallback, useRef, useEffect } from 'react';

function useDebouncedFetch<T>(delay: number) {
  const timeoutRef = useRef<NodeJS.Timeout | null>(null);
  const abortControllerRef = useRef<AbortController | null>(null);

  // 클린업
  useEffect(() => {
    return () => {
      if (timeoutRef.current) {
        clearTimeout(timeoutRef.current);
      }
      if (abortControllerRef.current) {
        abortControllerRef.current.abort();
      }
    };
  }, []);

  const debouncedFetch = useCallback(
    async (url: string, options?: RequestInit): Promise<T | null> => {
      // 이전 타이머 취소
      if (timeoutRef.current) {
        clearTimeout(timeoutRef.current);
      }

      // 이전 요청 취소
      if (abortControllerRef.current) {
        abortControllerRef.current.abort();
      }

      return new Promise((resolve, reject) => {
        timeoutRef.current = setTimeout(async () => {
          abortControllerRef.current = new AbortController();

          try {
            const response = await fetch(url, {
              ...options,
              signal: abortControllerRef.current.signal
            });

            if (!response.ok) {
              throw new Error(`HTTP ${response.status}`);
            }

            const data = await response.json();
            resolve(data);
          } catch (error) {
            if ((error as Error).name === 'AbortError') {
              resolve(null);  // 취소된 경우 null 반환
            } else {
              reject(error);
            }
          }
        }, delay);
      });
    },
    [delay]
  );

  const cancel = useCallback(() => {
    if (timeoutRef.current) {
      clearTimeout(timeoutRef.current);
      timeoutRef.current = null;
    }
    if (abortControllerRef.current) {
      abortControllerRef.current.abort();
      abortControllerRef.current = null;
    }
  }, []);

  return { debouncedFetch, cancel };
}

// 사용 예시
function SearchWithAbort() {
  const [results, setResults] = useState<SearchResult[]>([]);
  const { debouncedFetch, cancel } = useDebouncedFetch<SearchResponse>(300);

  const handleSearch = async (query: string) => {
    if (!query.trim()) {
      setResults([]);
      return;
    }

    const data = await debouncedFetch(
      `/api/search?q=${encodeURIComponent(query)}`
    );

    if (data) {
      setResults(data.results);
    }
  };

  // 컴포넌트 언마운트 시 취소
  useEffect(() => {
    return () => cancel();
  }, [cancel]);

  return (
    <input
      type="text"
      onChange={(e) => handleSearch(e.target.value)}
      placeholder="검색"
    />
  );
}
```

## 성능 측정 및 비교

디바운스와 스로틀은 전체 웹 성능 최적화 전략의 일부입니다. 더 넓은 관점의 성능 최적화 기법은 [웹 성능 최적화 가이드](/posts/web-performance-optimization-guide/)에서 확인할 수 있습니다.

### 디바운스/스로틀 효과 측정

```javascript
class PerformanceMonitor {
  constructor() {
    this.callCounts = {
      raw: 0,
      debounced: 0,
      throttled: 0
    };
    this.startTime = null;
  }

  start() {
    this.startTime = performance.now();
    this.callCounts = { raw: 0, debounced: 0, throttled: 0 };
  }

  recordRaw() {
    this.callCounts.raw++;
  }

  recordDebounced() {
    this.callCounts.debounced++;
  }

  recordThrottled() {
    this.callCounts.throttled++;
  }

  report() {
    const elapsed = performance.now() - this.startTime;

    console.table({
      '원본 호출': {
        횟수: this.callCounts.raw,
        '초당 호출': (this.callCounts.raw / (elapsed / 1000)).toFixed(2)
      },
      '디바운스 적용': {
        횟수: this.callCounts.debounced,
        '감소율': `${(100 - (this.callCounts.debounced / this.callCounts.raw * 100)).toFixed(1)}%`
      },
      '스로틀 적용': {
        횟수: this.callCounts.throttled,
        '감소율': `${(100 - (this.callCounts.throttled / this.callCounts.raw * 100)).toFixed(1)}%`
      }
    });
  }
}

// 테스트 실행
const monitor = new PerformanceMonitor();
monitor.start();

const rawHandler = () => monitor.recordRaw();
const debouncedHandler = debounce(() => monitor.recordDebounced(), 300);
const throttledHandler = throttle(() => monitor.recordThrottled(), 200);

// 입력 이벤트 시뮬레이션
const input = document.getElementById('test-input');
input.addEventListener('input', () => {
  rawHandler();
  debouncedHandler();
  throttledHandler();
});

// 5초 후 결과 출력
setTimeout(() => {
  monitor.report();
}, 5000);

// 예상 결과:
// 5초간 빠른 타이핑 시
// +----------------+--------+------------+
// |                |  횟수  |   감소율   |
// +----------------+--------+------------+
// | 원본 호출      |   150  |     -      |
// | 디바운스 적용  |     1  |   99.3%    |
// | 스로틀 적용    |    25  |   83.3%    |
// +----------------+--------+------------+
```

## 실무 적용 체크리스트

디바운스/스로틀을 적용할 때 확인해야 할 사항들입니다.

### 구현 체크리스트

- [ ] **적절한 기법 선택**: 사용 사례에 맞는 디바운스/스로틀 선택
- [ ] **적절한 지연 시간 설정**: 너무 짧으면 효과 없음, 너무 길면 반응 느림
- [ ] **메모리 누수 방지**: 컴포넌트 언마운트 시 타이머 정리
- [ ] **이전 요청 취소**: 비동기 작업 시 AbortController 사용
- [ ] **에러 처리**: try-catch로 에러 상황 대응
- [ ] **로딩 상태 표시**: 사용자에게 진행 상황 피드백
- [ ] **테스트**: 다양한 입력 패턴에서 동작 확인

### 권장 지연 시간

| 사용 사례 | 권장 지연 시간 | 이유 |
|----------|-------------|-----|
| 검색 자동완성 | 200-400ms | 타이핑 완료 감지 |
| 폼 유효성 검사 | 300-500ms | 입력 완료 대기 |
| 자동 저장 | 1000-2000ms | 편집 흐름 유지 |
| 스크롤 이벤트 | 100-200ms | 부드러운 반응 |
| 리사이즈 | 200-300ms | 레이아웃 재계산 |
| 마우스 이동 | 50-100ms | 실시간 추적 필요 |

## 핵심 정리

디바운스와 스로틀은 웹 애플리케이션 성능 최적화의 필수 기법입니다.

### 핵심 포인트

1. **디바운스**: 연속 이벤트의 마지막만 처리 - 검색, 폼 검증, 자동 저장
2. **스로틀**: 일정 간격으로 실행 제한 - 스크롤, 리사이즈, 마우스 이동
3. **선택 기준**: "최종 결과만 필요" → 디바운스, "중간 상태도 필요" → 스로틀
4. **React**: 커스텀 훅으로 구현하고 cleanup 필수
5. **비동기 작업**: AbortController로 이전 요청 취소
6. **프로덕션**: Lodash 같은 검증된 라이브러리 권장

### 자주 하는 실수

- 타이머 정리 누락으로 인한 메모리 누수
- 너무 긴 지연 시간으로 인한 반응 지연
- 이전 비동기 요청 미취소로 인한 race condition
- this 바인딩 문제 (화살표 함수 또는 bind 사용)

### 참고 자료

- [MDN - Debounce와 Throttle](https://developer.mozilla.org/en-US/docs/Glossary/Debounce)
- [Lodash - debounce](https://lodash.com/docs/4.17.15#debounce)
- [Lodash - throttle](https://lodash.com/docs/4.17.15#throttle)
- [JavaScript.info - Debounce와 Throttle](https://javascript.info/task/debounce)
- [React 공식 문서 - useDeferredValue](https://react.dev/reference/react/useDeferredValue)
