---
title: 웹 접근성 완벽 가이드 - WCAG 2.1 준수하기
description: 웹 접근성의 중요성부터 WCAG 2.1 가이드라인 준수, ARIA 활용, React에서의 구현까지. 시맨틱 HTML, 키보드 네비게이션, 스크린 리더 대응 방법을 실제 코드 예제와 함께 완벽하게 정리합니다. 법적 요구사항과 비즈니스 이점을 이해하고 접근성 테스트 도구로 검증하는 실전 가이드입니다.
author: changsu
date: 2025-11-16 10:00:00 +0900
categories: [Frontend, Accessibility]
tags: [accessibility, a11y, wcag, aria, screen-reader, semantic-html, 접근성, 웹접근성, ARIA]
toc: true
comments: true
---

## 들어가며

"우리 웹사이트는 대부분의 사용자가 마우스로 잘 사용하고 있으니 접근성은 나중에 신경쓰자"라고 생각하시나요?

**전 세계 인구의 15% 이상, 약 10억 명이 장애를 가지고 있습니다.** 웹 접근성은 단순히 장애인을 위한 기능이 아닙니다. 키보드만 사용하는 파워 유저, 임시로 손을 다친 사람, 밝은 햇빛 아래서 화면을 보는 사람 모두에게 중요합니다.

이 글에서는 WCAG 2.1 기준을 준수하는 웹 접근성 구현 방법을 실전 코드와 함께 다룹니다.

> 웹 접근성은 선택이 아닌 필수입니다. 법적 요구사항이자, 모든 사용자를 위한 더 나은 UX입니다.
{: .prompt-tip }

## 왜 웹 접근성이 중요한가?

### 1. 법적 요구사항

**대한민국**
- 장애인차별금지법(2008년 시행)
- 국가 및 공공기관 웹사이트: 의무 준수
- 법인 웹사이트: 권장 사항 (일부 업종 의무)

**미국**
- ADA (Americans with Disabilities Act)
- Section 508
- 위반 시 소송 위험 및 벌금

**EU**
- European Accessibility Act
- 2025년부터 모든 공공 및 민간 웹사이트 의무 준수

### 2. 비즈니스 이점

**시장 확대**
```
전 세계 장애인 시장 규모: 약 13조 달러
접근 가능한 웹사이트 = 더 많은 잠재 고객
```

**SEO 개선**
- 시맨틱 HTML → 검색 엔진이 콘텐츠를 더 잘 이해
- 스크린 리더 최적화 → 구조화된 콘텐츠
- 이미지 대체 텍스트 → 검색 노출 증가

**코드 품질 향상**
- 더 명확한 구조와 시맨틱
- 더 나은 유지보수성
- 전체 사용자 경험 개선

### 3. 실제 사용 사례

```text
// 접근성이 좋은 웹사이트는 다음 상황에서도 사용 가능합니다:

✅ 시각 장애인이 스크린 리더로 콘텐츠를 듣는 경우
✅ 손 부상으로 마우스를 사용할 수 없어 키보드만 사용하는 경우
✅ 색맹인 사용자가 색상 대비를 통해 정보를 인지하는 경우
✅ 청각 장애인이 비디오 자막으로 내용을 이해하는 경우
✅ 인지 장애가 있는 사용자가 명확한 레이블과 안내로 폼을 작성하는 경우
```

## WCAG 2.1 가이드라인 이해하기

### POUR 원칙

WCAG는 **4가지 핵심 원칙**을 기반으로 합니다.

#### 1. Perceivable (인지 가능)

**정보와 UI 컴포넌트를 사용자가 인지할 수 있어야 합니다.**

```html
<!-- ❌ 나쁜 예: 이미지에 대체 텍스트 없음 -->
<img src="product.jpg">

<!-- ✅ 좋은 예: 명확한 대체 텍스트 -->
<img src="product.jpg" alt="MacBook Pro 14인치 M3 실버 색상">

<!-- ❌ 나쁜 예: 색상으로만 정보 전달 -->
<span style="color: red">필수 항목</span>

<!-- ✅ 좋은 예: 색상 + 텍스트 + 아이콘 -->
<span class="required">
  <span class="visually-hidden">필수</span>
  <span aria-hidden="true">*</span>
  필수 항목
</span>
```

#### 2. Operable (조작 가능)

**모든 기능을 키보드로 조작할 수 있어야 합니다.**

```jsx
// ❌ 나쁜 예: 클릭만 가능한 div
<div onClick={handleClick}>클릭하세요</div>

// ✅ 좋은 예: 키보드 접근 가능한 버튼
<button onClick={handleClick}>클릭하세요</button>

// ✅ 좋은 예: 커스텀 컴포넌트에 키보드 지원
<div
  role="button"
  tabIndex={0}
  onClick={handleClick}
  onKeyDown={(e) => {
    if (e.key === 'Enter' || e.key === ' ') {
      e.preventDefault();
      handleClick();
    }
  }}
>
  클릭하세요
</div>
```

#### 3. Understandable (이해 가능)

**정보와 UI 조작 방법을 이해할 수 있어야 합니다.**

```jsx
// ❌ 나쁜 예: 모호한 레이블
<label>입력</label>
<input type="text" />

// ✅ 좋은 예: 명확한 레이블과 도움말
<label htmlFor="email">
  이메일 주소
  <span className="required" aria-label="필수 항목">*</span>
</label>
<input
  id="email"
  type="email"
  aria-describedby="email-help"
  required
/>
<div id="email-help" className="help-text">
  로그인에 사용할 이메일 주소를 입력하세요
</div>
```

#### 4. Robust (견고함)

**다양한 기술과 보조 기술로 접근 가능해야 합니다.**

```html
<!-- ✅ 시맨틱 HTML 사용 -->
<header>
  <nav>
    <ul>
      <li><a href="/">홈</a></li>
      <li><a href="/about">소개</a></li>
    </ul>
  </nav>
</header>

<main>
  <article>
    <h1>메인 제목</h1>
    <p>본문 내용...</p>
  </article>
</main>

<footer>
  <p>&copy; 2025 회사명</p>
</footer>
```

### 준수 레벨 (A, AA, AAA)

| 레벨 | 설명 | 요구사항 | 권장 대상 |
|------|------|----------|-----------|
| **A** | 최소 수준 | 가장 기본적인 접근성 | 모든 웹사이트 필수 |
| **AA** | 권장 수준 | 대부분의 장벽 제거 | 대부분의 웹사이트 목표 |
| **AAA** | 최고 수준 | 모든 가능한 접근성 | 특수 목적 사이트 |

**대부분의 법적 요구사항은 AA 레벨을 기준으로 합니다.**

## 시맨틱 HTML의 중요성

### 시맨틱 요소 사용하기

```html
<!-- ❌ 나쁜 예: 의미 없는 div 남용 -->
<div class="header">
  <div class="nav">
    <div class="nav-item">홈</div>
    <div class="nav-item">소개</div>
  </div>
</div>
<div class="content">
  <div class="article">
    <div class="title">제목</div>
    <div class="text">본문...</div>
  </div>
</div>

<!-- ✅ 좋은 예: 시맨틱 요소 사용 -->
<header>
  <nav>
    <a href="/">홈</a>
    <a href="/about">소개</a>
  </nav>
</header>
<main>
  <article>
    <h1>제목</h1>
    <p>본문...</p>
  </article>
</main>
```

### 제목 구조 (Heading Hierarchy)

```html
<!-- ❌ 나쁜 예: 순서가 뒤죽박죽 -->
<h1>메인 제목</h1>
<h3>섹션 제목</h3> <!-- h2를 건너뜀 -->
<h2>다른 섹션</h2>

<!-- ✅ 좋은 예: 논리적 순서 -->
<h1>메인 제목</h1>
<section>
  <h2>첫 번째 섹션</h2>
  <h3>하위 섹션</h3>
  <h3>또 다른 하위 섹션</h3>
</section>
<section>
  <h2>두 번째 섹션</h2>
</section>
```

### 랜드마크 (Landmarks)

```html
<!-- 스크린 리더가 페이지 구조를 쉽게 파악 -->
<header role="banner">
  <nav role="navigation" aria-label="주 메뉴">
    <!-- 네비게이션 -->
  </nav>
</header>

<main role="main">
  <article>
    <!-- 메인 콘텐츠 -->
  </article>

  <aside role="complementary">
    <!-- 사이드바 -->
  </aside>
</main>

<footer role="contentinfo">
  <!-- 푸터 -->
</footer>
```

## ARIA (Accessible Rich Internet Applications)

### ARIA란?

ARIA는 **HTML이 표현할 수 없는 복잡한 UI의 접근성을 보완**하는 속성입니다.

> **ARIA 황금률**: 네이티브 HTML을 사용할 수 있다면 ARIA를 사용하지 마세요.
{: .prompt-warning }

### ARIA의 3가지 구성 요소

**1. Roles (역할)**

```html
<!-- 요소의 역할 정의 -->
<div role="button">클릭</div>
<div role="alert">경고 메시지</div>
<div role="tablist">
  <div role="tab">탭 1</div>
  <div role="tab">탭 2</div>
</div>
```

**2. Properties (속성)**

```html
<!-- 요소의 특성 정의 -->
<input aria-label="검색" type="search" />
<button aria-describedby="help-text">제출</button>
<div id="help-text">도움말 텍스트</div>
```

**3. States (상태)**

```html
<!-- 요소의 현재 상태 정의 -->
<button aria-pressed="true">토글 버튼</button>
<div aria-expanded="false">접힌 내용</div>
<input aria-invalid="true" aria-errormessage="error-msg" />
<div id="error-msg">유효하지 않은 입력입니다</div>
```

### 주요 ARIA 패턴 구현

#### 1. 모달 (Dialog)

```jsx
import { useEffect, useRef } from 'react';

function Modal({ isOpen, onClose, title, children }) {
  const modalRef = useRef(null);
  const previousFocusRef = useRef(null);

  useEffect(() => {
    if (isOpen) {
      // 모달 열릴 때 현재 포커스 저장
      previousFocusRef.current = document.activeElement;

      // 첫 번째 포커스 가능 요소로 포커스 이동
      const firstFocusable = modalRef.current.querySelector(
        'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
      );
      firstFocusable?.focus();

      // 배경 스크롤 방지
      document.body.style.overflow = 'hidden';
    } else {
      // 모달 닫힐 때 이전 포커스 복원
      previousFocusRef.current?.focus();
      document.body.style.overflow = '';
    }

    return () => {
      document.body.style.overflow = '';
    };
  }, [isOpen]);

  const handleKeyDown = (e) => {
    if (e.key === 'Escape') {
      onClose();
    }

    // 포커스 트랩 (Tab 순환)
    if (e.key === 'Tab') {
      const focusableElements = modalRef.current.querySelectorAll(
        'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
      );
      const firstElement = focusableElements[0];
      const lastElement = focusableElements[focusableElements.length - 1];

      if (e.shiftKey && document.activeElement === firstElement) {
        e.preventDefault();
        lastElement.focus();
      } else if (!e.shiftKey && document.activeElement === lastElement) {
        e.preventDefault();
        firstElement.focus();
      }
    }
  };

  if (!isOpen) return null;

  return (
    <>
      {/* 배경 오버레이 */}
      <div
        className="modal-overlay"
        onClick={onClose}
        aria-hidden="true"
      />

      {/* 모달 */}
      <div
        ref={modalRef}
        role="dialog"
        aria-modal="true"
        aria-labelledby="modal-title"
        onKeyDown={handleKeyDown}
        className="modal"
      >
        <header>
          <h2 id="modal-title">{title}</h2>
          <button
            onClick={onClose}
            aria-label="모달 닫기"
            className="modal-close"
          >
            ✕
          </button>
        </header>

        <div className="modal-content">
          {children}
        </div>

        <footer>
          <button onClick={onClose}>취소</button>
          <button>확인</button>
        </footer>
      </div>
    </>
  );
}

// 사용 예시
function App() {
  const [isModalOpen, setIsModalOpen] = useState(false);

  return (
    <>
      <button onClick={() => setIsModalOpen(true)}>
        모달 열기
      </button>

      <Modal
        isOpen={isModalOpen}
        onClose={() => setIsModalOpen(false)}
        title="알림"
      >
        <p>모달 내용입니다.</p>
      </Modal>
    </>
  );
}
```

#### 2. 탭 (Tabs)

```jsx
import { useState } from 'react';

function Tabs({ tabs }) {
  const [activeTab, setActiveTab] = useState(0);

  const handleKeyDown = (e, index) => {
    let newIndex = index;

    switch (e.key) {
      case 'ArrowLeft':
        newIndex = index > 0 ? index - 1 : tabs.length - 1;
        break;
      case 'ArrowRight':
        newIndex = index < tabs.length - 1 ? index + 1 : 0;
        break;
      case 'Home':
        newIndex = 0;
        break;
      case 'End':
        newIndex = tabs.length - 1;
        break;
      default:
        return;
    }

    e.preventDefault();
    setActiveTab(newIndex);

    // 새 탭으로 포커스 이동
    document.getElementById(`tab-${newIndex}`).focus();
  };

  return (
    <div className="tabs">
      {/* 탭 목록 */}
      <div role="tablist" aria-label="콘텐츠 탭">
        {tabs.map((tab, index) => (
          <button
            key={index}
            id={`tab-${index}`}
            role="tab"
            aria-selected={activeTab === index}
            aria-controls={`panel-${index}`}
            tabIndex={activeTab === index ? 0 : -1}
            onClick={() => setActiveTab(index)}
            onKeyDown={(e) => handleKeyDown(e, index)}
          >
            {tab.label}
          </button>
        ))}
      </div>

      {/* 탭 패널 */}
      {tabs.map((tab, index) => (
        <div
          key={index}
          id={`panel-${index}`}
          role="tabpanel"
          aria-labelledby={`tab-${index}`}
          hidden={activeTab !== index}
          tabIndex={0}
        >
          {tab.content}
        </div>
      ))}
    </div>
  );
}

// 사용 예시
function App() {
  const tabs = [
    { label: '프로필', content: <div>프로필 내용</div> },
    { label: '설정', content: <div>설정 내용</div> },
    { label: '알림', content: <div>알림 내용</div> },
  ];

  return <Tabs tabs={tabs} />;
}
```

#### 3. 드롭다운 (Combobox)

```jsx
import { useState, useRef, useEffect } from 'react';

function Combobox({ options, value, onChange, label }) {
  const [isOpen, setIsOpen] = useState(false);
  const [highlightedIndex, setHighlightedIndex] = useState(-1);
  const listboxRef = useRef(null);

  const handleKeyDown = (e) => {
    switch (e.key) {
      case 'ArrowDown':
        e.preventDefault();
        if (!isOpen) {
          setIsOpen(true);
          setHighlightedIndex(0);
        } else {
          setHighlightedIndex((prev) =>
            prev < options.length - 1 ? prev + 1 : 0
          );
        }
        break;

      case 'ArrowUp':
        e.preventDefault();
        if (isOpen) {
          setHighlightedIndex((prev) =>
            prev > 0 ? prev - 1 : options.length - 1
          );
        }
        break;

      case 'Enter':
        e.preventDefault();
        if (isOpen && highlightedIndex >= 0) {
          onChange(options[highlightedIndex]);
          setIsOpen(false);
        }
        break;

      case 'Escape':
        e.preventDefault();
        setIsOpen(false);
        break;

      default:
        break;
    }
  };

  useEffect(() => {
    if (isOpen && highlightedIndex >= 0) {
      const option = listboxRef.current?.children[highlightedIndex];
      option?.scrollIntoView({ block: 'nearest' });
    }
  }, [highlightedIndex, isOpen]);

  return (
    <div className="combobox">
      <label id="combobox-label">{label}</label>

      <div className="combobox-wrapper">
        <input
          type="text"
          role="combobox"
          aria-expanded={isOpen}
          aria-controls="listbox"
          aria-labelledby="combobox-label"
          aria-activedescendant={
            highlightedIndex >= 0 ? `option-${highlightedIndex}` : undefined
          }
          value={value}
          onKeyDown={handleKeyDown}
          onFocus={() => setIsOpen(true)}
          onBlur={() => setTimeout(() => setIsOpen(false), 200)}
          onChange={(e) => onChange(e.target.value)}
        />

        <button
          type="button"
          aria-label="옵션 열기"
          tabIndex={-1}
          onClick={() => setIsOpen(!isOpen)}
        >
          ▼
        </button>
      </div>

      {isOpen && (
        <ul
          ref={listboxRef}
          id="listbox"
          role="listbox"
          aria-labelledby="combobox-label"
        >
          {options.map((option, index) => (
            <li
              key={index}
              id={`option-${index}`}
              role="option"
              aria-selected={highlightedIndex === index}
              onClick={() => {
                onChange(option);
                setIsOpen(false);
              }}
              className={highlightedIndex === index ? 'highlighted' : ''}
            >
              {option}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}

// 사용 예시
function App() {
  const [selectedFruit, setSelectedFruit] = useState('');
  const fruits = ['사과', '바나나', '오렌지', '포도', '딸기'];

  return (
    <Combobox
      options={fruits}
      value={selectedFruit}
      onChange={setSelectedFruit}
      label="과일 선택"
    />
  );
}
```

### ARIA Live Regions (동적 콘텐츠)

```jsx
function LiveRegionExample() {
  const [status, setStatus] = useState('');
  const [items, setItems] = useState([]);

  const handleAddItem = () => {
    const newItem = `아이템 ${items.length + 1}`;
    setItems([...items, newItem]);
    setStatus(`${newItem}이(가) 추가되었습니다.`);
  };

  return (
    <div>
      <button onClick={handleAddItem}>아이템 추가</button>

      {/* 상태 메시지 - 스크린 리더가 즉시 읽음 */}
      <div
        role="status"
        aria-live="polite"
        aria-atomic="true"
        className="visually-hidden"
      >
        {status}
      </div>

      {/* 긴급 알림 - 현재 읽고 있는 내용을 중단하고 읽음 */}
      <div
        role="alert"
        aria-live="assertive"
        aria-atomic="true"
      >
        {/* 에러 메시지 등 */}
      </div>

      <ul>
        {items.map((item, index) => (
          <li key={index}>{item}</li>
        ))}
      </ul>
    </div>
  );
}
```

**ARIA Live 속성:**

| 속성 | 값 | 설명 |
|------|-----|------|
| `aria-live` | `off` | 업데이트를 알리지 않음 (기본값) |
| | `polite` | 현재 작업 후 알림 |
| | `assertive` | 즉시 알림 (긴급) |
| `aria-atomic` | `true` | 전체 영역을 읽음 |
| | `false` | 변경된 부분만 읽음 |

## 키보드 네비게이션

### 기본 키보드 동작

| 키 | 동작 |
|-----|------|
| `Tab` | 다음 포커스 가능 요소로 이동 |
| `Shift + Tab` | 이전 포커스 가능 요소로 이동 |
| `Enter` | 링크/버튼 활성화 |
| `Space` | 버튼 활성화, 체크박스 토글 |
| `Arrow Keys` | 라디오 버튼, 탭, 메뉴 등에서 이동 |
| `Escape` | 모달/드롭다운 닫기 |

### Skip Navigation 구현

```jsx
function Layout({ children }) {
  return (
    <>
      {/* Skip to main content 링크 */}
      <a href="#main-content" className="skip-link">
        메인 콘텐츠로 건너뛰기
      </a>

      <header>
        <nav>
          {/* 네비게이션 메뉴 많음 */}
          <a href="/">홈</a>
          <a href="/about">소개</a>
          <a href="/products">제품</a>
          {/* ... 더 많은 링크 ... */}
        </nav>
      </header>

      <main id="main-content" tabIndex={-1}>
        {children}
      </main>
    </>
  );
}
```

```css
/* Skip link 스타일 */
.skip-link {
  position: absolute;
  top: -40px;
  left: 0;
  background: #000;
  color: #fff;
  padding: 8px;
  text-decoration: none;
  z-index: 100;
}

.skip-link:focus {
  top: 0;
}
```

### 포커스 관리

```jsx
import { useRef, useEffect } from 'react';

function FocusManagement() {
  const headingRef = useRef(null);

  // 페이지 로드 시 제목으로 포커스 이동
  useEffect(() => {
    headingRef.current?.focus();
  }, []);

  return (
    <div>
      <h1 ref={headingRef} tabIndex={-1}>
        새 페이지
      </h1>
      <p>내용...</p>
    </div>
  );
}

// 커스텀 훅으로 포커스 트랩 구현
function useFocusTrap(isActive) {
  const containerRef = useRef(null);

  useEffect(() => {
    if (!isActive) return;

    const container = containerRef.current;
    const focusableElements = container.querySelectorAll(
      'a[href], button:not([disabled]), textarea, input, select, [tabindex]:not([tabindex="-1"])'
    );
    const firstElement = focusableElements[0];
    const lastElement = focusableElements[focusableElements.length - 1];

    const handleKeyDown = (e) => {
      if (e.key === 'Tab') {
        if (e.shiftKey && document.activeElement === firstElement) {
          e.preventDefault();
          lastElement.focus();
        } else if (!e.shiftKey && document.activeElement === lastElement) {
          e.preventDefault();
          firstElement.focus();
        }
      }
    };

    container.addEventListener('keydown', handleKeyDown);
    firstElement?.focus();

    return () => {
      container.removeEventListener('keydown', handleKeyDown);
    };
  }, [isActive]);

  return containerRef;
}
```

### 포커스 가시성

```css
/* 기본 outline 제거하지 마세요! */
/* ❌ 나쁜 예 */
*:focus {
  outline: none;
}

/* ✅ 좋은 예: 더 나은 포커스 표시 */
*:focus {
  outline: 2px solid #0066cc;
  outline-offset: 2px;
}

/* 더 나은 방법: :focus-visible 사용 */
*:focus-visible {
  outline: 2px solid #0066cc;
  outline-offset: 2px;
}

/* 마우스 클릭시에는 outline 숨김 */
*:focus:not(:focus-visible) {
  outline: none;
}
```

## 색상 대비 및 시각적 접근성

### 색상 대비 비율

**WCAG 기준:**

| 레벨 | 일반 텍스트 | 큰 텍스트 (18pt+) |
|------|-------------|-------------------|
| **AA** | 4.5:1 이상 | 3:1 이상 |
| **AAA** | 7:1 이상 | 4.5:1 이상 |

```css
/* ❌ 나쁜 예: 낮은 대비 (2.1:1) */
.low-contrast {
  color: #999;
  background-color: #fff;
}

/* ✅ 좋은 예: 충분한 대비 (4.6:1) */
.good-contrast {
  color: #666;
  background-color: #fff;
}

/* ✅ 더 나은 예: 높은 대비 (21:1) */
.high-contrast {
  color: #000;
  background-color: #fff;
}
```

### 색상에만 의존하지 않기

```jsx
// ❌ 나쁜 예: 색상으로만 구분
function StatusBadge({ status }) {
  const color = status === 'success' ? 'green' : 'red';
  return <span style={{ color }}>{status}</span>;
}

// ✅ 좋은 예: 색상 + 텍스트 + 아이콘
function AccessibleStatusBadge({ status }) {
  const config = {
    success: { color: 'green', icon: '✓', text: '성공' },
    error: { color: 'red', icon: '✕', text: '실패' },
  };

  const { color, icon, text } = config[status];

  return (
    <span className={`badge badge-${status}`}>
      <span className="icon" aria-hidden="true">{icon}</span>
      <span>{text}</span>
    </span>
  );
}
```

### 텍스트 크기 조정

```css
/* 사용자가 브라우저 설정으로 텍스트 크기 조정 가능하도록 */
/* ✅ rem 단위 사용 */
body {
  font-size: 16px; /* 기본 크기 */
}

h1 {
  font-size: 2rem; /* 32px */
}

p {
  font-size: 1rem; /* 16px */
}

/* ❌ px 고정 크기는 피하기 */
.fixed-size {
  font-size: 14px; /* 사용자가 조정 불가 */
}
```

## 폼 접근성

### 레이블과 입력 필드 연결

```jsx
function AccessibleForm() {
  const [formData, setFormData] = useState({
    email: '',
    password: '',
    subscribe: false,
  });

  const [errors, setErrors] = useState({});

  return (
    <form onSubmit={handleSubmit}>
      {/* 이메일 입력 */}
      <div className="form-group">
        <label htmlFor="email">
          이메일 주소
          <span className="required" aria-label="필수 항목">*</span>
        </label>
        <input
          id="email"
          type="email"
          value={formData.email}
          onChange={(e) => setFormData({ ...formData, email: e.target.value })}
          aria-required="true"
          aria-invalid={errors.email ? 'true' : 'false'}
          aria-describedby={errors.email ? 'email-error email-help' : 'email-help'}
          required
        />
        <div id="email-help" className="help-text">
          로그인에 사용할 이메일을 입력하세요
        </div>
        {errors.email && (
          <div id="email-error" className="error" role="alert">
            {errors.email}
          </div>
        )}
      </div>

      {/* 비밀번호 입력 */}
      <div className="form-group">
        <label htmlFor="password">
          비밀번호
          <span className="required" aria-label="필수 항목">*</span>
        </label>
        <input
          id="password"
          type="password"
          value={formData.password}
          onChange={(e) => setFormData({ ...formData, password: e.target.value })}
          aria-required="true"
          aria-invalid={errors.password ? 'true' : 'false'}
          aria-describedby={errors.password ? 'password-error password-help' : 'password-help'}
          required
        />
        <div id="password-help" className="help-text">
          8자 이상, 영문, 숫자, 특수문자 포함
        </div>
        {errors.password && (
          <div id="password-error" className="error" role="alert">
            {errors.password}
          </div>
        )}
      </div>

      {/* 체크박스 */}
      <div className="form-group">
        <input
          id="subscribe"
          type="checkbox"
          checked={formData.subscribe}
          onChange={(e) => setFormData({ ...formData, subscribe: e.target.checked })}
        />
        <label htmlFor="subscribe">
          뉴스레터 구독하기
        </label>
      </div>

      <button type="submit">가입하기</button>
    </form>
  );
}
```

### 에러 메시지와 유효성 검사

```jsx
import { useState } from 'react';

function FormWithValidation() {
  const [email, setEmail] = useState('');
  const [error, setError] = useState('');
  const [touched, setTouched] = useState(false);

  const validateEmail = (value) => {
    if (!value) {
      return '이메일은 필수 항목입니다.';
    }
    if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value)) {
      return '올바른 이메일 형식이 아닙니다.';
    }
    return '';
  };

  const handleBlur = () => {
    setTouched(true);
    setError(validateEmail(email));
  };

  const handleChange = (e) => {
    const value = e.target.value;
    setEmail(value);

    if (touched) {
      setError(validateEmail(value));
    }
  };

  return (
    <div className="form-group">
      <label htmlFor="email">
        이메일 주소
        <span className="required" aria-label="필수 항목">*</span>
      </label>

      <input
        id="email"
        type="email"
        value={email}
        onChange={handleChange}
        onBlur={handleBlur}
        aria-required="true"
        aria-invalid={error ? 'true' : 'false'}
        aria-describedby={error ? 'email-error' : undefined}
      />

      {error && (
        <div
          id="email-error"
          className="error"
          role="alert"
          aria-live="polite"
        >
          {error}
        </div>
      )}
    </div>
  );
}
```

### 필드셋과 레전드

```jsx
function RadioGroup() {
  const [selectedPayment, setSelectedPayment] = useState('');

  return (
    <fieldset>
      <legend>결제 방법을 선택하세요</legend>

      <div>
        <input
          id="card"
          type="radio"
          name="payment"
          value="card"
          checked={selectedPayment === 'card'}
          onChange={(e) => setSelectedPayment(e.target.value)}
        />
        <label htmlFor="card">신용카드</label>
      </div>

      <div>
        <input
          id="transfer"
          type="radio"
          name="payment"
          value="transfer"
          checked={selectedPayment === 'transfer'}
          onChange={(e) => setSelectedPayment(e.target.value)}
        />
        <label htmlFor="transfer">계좌이체</label>
      </div>

      <div>
        <input
          id="phone"
          type="radio"
          name="payment"
          value="phone"
          checked={selectedPayment === 'phone'}
          onChange={(e) => setSelectedPayment(e.target.value)}
        />
        <label htmlFor="phone">휴대폰 결제</label>
      </div>
    </fieldset>
  );
}
```

## 스크린 리더 대응

### 스크린 리더용 텍스트

```css
/* 시각적으로 숨기지만 스크린 리더는 읽을 수 있게 */
.visually-hidden {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}
```

```jsx
function AccessibleButton() {
  return (
    <>
      {/* 아이콘 버튼에 텍스트 레이블 추가 */}
      <button aria-label="검색">
        <span aria-hidden="true">🔍</span>
      </button>

      {/* 또는 visually-hidden 클래스 사용 */}
      <button>
        <span className="visually-hidden">검색</span>
        <span aria-hidden="true">🔍</span>
      </button>

      {/* 링크에 추가 컨텍스트 제공 */}
      <a href="/products/iphone">
        iPhone
        <span className="visually-hidden"> 제품 상세 페이지로 이동</span>
      </a>
    </>
  );
}
```

### 의미없는 콘텐츠 숨기기

```jsx
function Card({ title, description }) {
  return (
    <div className="card">
      {/* 장식용 아이콘은 스크린 리더에서 숨김 */}
      <span className="icon" aria-hidden="true">★</span>

      <h3>{title}</h3>
      <p>{description}</p>

      {/* 이미지가 순수 장식인 경우 */}
      <img src="decoration.png" alt="" role="presentation" />
    </div>
  );
}
```

### 주요 스크린 리더 테스트

| 스크린 리더 | 플랫폼 | 브라우저 |
|------------|--------|---------|
| **NVDA** | Windows | Chrome, Firefox |
| **JAWS** | Windows | Chrome, Edge, IE |
| **VoiceOver** | macOS, iOS | Safari |
| **TalkBack** | Android | Chrome |

**기본 단축키 (NVDA):**
- `Insert + Down`: 읽기 시작
- `Insert + Space`: 탐색 모드 전환
- `H`: 다음 제목으로 이동
- `K`: 다음 링크로 이동
- `B`: 다음 버튼으로 이동
- `F`: 다음 폼 필드로 이동

## React에서의 접근성 구현

### React 특화 접근성 팁

```jsx
// 1. Fragment 사용으로 불필요한 div 제거
import { Fragment } from 'react';

function List({ items }) {
  return (
    <ul>
      {items.map((item) => (
        <Fragment key={item.id}>
          <li>{item.name}</li>
        </Fragment>
      ))}
    </ul>
  );
}

// 2. htmlFor 사용 (for 대신)
<label htmlFor="username">사용자명</label>
<input id="username" type="text" />

// 3. className 사용 (class 대신)
<div className="container" />

// 4. aria-* 속성은 그대로 사용
<button aria-label="닫기" aria-pressed="false">
```

### 접근 가능한 React 컴포넌트 라이브러리

```tsx
// Headless UI 사용 예시 (Tailwind CSS 팀)
import { Dialog, Transition } from '@headlessui/react';

function MyDialog({ isOpen, onClose }) {
  return (
    <Transition show={isOpen}>
      <Dialog onClose={onClose}>
        <Dialog.Overlay />
        <Dialog.Title>제목</Dialog.Title>
        <Dialog.Description>설명</Dialog.Description>
        <button onClick={onClose}>닫기</button>
      </Dialog>
    </Transition>
  );
}

// Radix UI 사용 예시
import * as Dialog from '@radix-ui/react-dialog';

function MyRadixDialog() {
  return (
    <Dialog.Root>
      <Dialog.Trigger>열기</Dialog.Trigger>
      <Dialog.Portal>
        <Dialog.Overlay />
        <Dialog.Content>
          <Dialog.Title>제목</Dialog.Title>
          <Dialog.Description>설명</Dialog.Description>
          <Dialog.Close>닫기</Dialog.Close>
        </Dialog.Content>
      </Dialog.Portal>
    </Dialog.Root>
  );
}
```

### 커스텀 훅으로 접근성 개선

```tsx
import { useEffect, useRef } from 'react';

// 페이지 제목 관리
export function useDocumentTitle(title: string) {
  useEffect(() => {
    const prevTitle = document.title;
    document.title = title;

    return () => {
      document.title = prevTitle;
    };
  }, [title]);
}

// 공지 (announcement)
export function useAnnouncement() {
  const announcementRef = useRef<HTMLDivElement>(null);

  const announce = (message: string, priority: 'polite' | 'assertive' = 'polite') => {
    if (announcementRef.current) {
      announcementRef.current.setAttribute('aria-live', priority);
      announcementRef.current.textContent = message;
    }
  };

  const AnnouncementRegion = () => (
    <div
      ref={announcementRef}
      role="status"
      aria-live="polite"
      aria-atomic="true"
      className="visually-hidden"
    />
  );

  return { announce, AnnouncementRegion };
}

// 사용 예시
function MyComponent() {
  const { announce, AnnouncementRegion } = useAnnouncement();

  const handleSave = () => {
    // 저장 로직...
    announce('변경사항이 저장되었습니다.');
  };

  return (
    <>
      <AnnouncementRegion />
      <button onClick={handleSave}>저장</button>
    </>
  );
}

// 외부 클릭 감지
export function useClickOutside<T extends HTMLElement>(
  callback: () => void
) {
  const ref = useRef<T>(null);

  useEffect(() => {
    const handleClick = (e: MouseEvent) => {
      if (ref.current && !ref.current.contains(e.target as Node)) {
        callback();
      }
    };

    document.addEventListener('mousedown', handleClick);
    return () => document.removeEventListener('mousedown', handleClick);
  }, [callback]);

  return ref;
}
```

## 접근성 테스트 도구 및 방법

### 1. 자동화 테스트 도구

#### axe DevTools (브라우저 확장)

```bash
# 크롬 웹스토어에서 설치
# https://chrome.google.com/webstore/detail/axe-devtools-web-accessibility/lhdoppojpmngadmnindnejefpokejbdd
```

**사용법:**
1. 개발자 도구 열기 (F12)
2. "axe DevTools" 탭 선택
3. "Scan ALL of my page" 클릭
4. 발견된 문제 확인 및 수정

#### Lighthouse (Chrome DevTools 내장)

```bash
# Chrome DevTools > Lighthouse 탭
# 1. Accessibility 체크
# 2. Generate report 클릭
# 3. 점수 및 개선사항 확인
```

#### WAVE (Web Accessibility Evaluation Tool)

```bash
# 브라우저 확장 설치
# https://wave.webaim.org/extension/
```

### 2. 자동화 테스트 (코드)

```bash
npm install --save-dev @axe-core/react jest-axe
```

```typescript
// setupTests.ts
import { toHaveNoViolations } from 'jest-axe';

expect.extend(toHaveNoViolations);
```

```tsx
// Button.test.tsx
import { render } from '@testing-library/react';
import { axe } from 'jest-axe';
import { Button } from './Button';

describe('Button 접근성', () => {
  test('접근성 위반이 없어야 함', async () => {
    const { container } = render(<Button>클릭</Button>);
    const results = await axe(container);
    expect(results).toHaveNoViolations();
  });

  test('aria-label이 있어야 함 (아이콘 버튼)', async () => {
    const { container } = render(
      <Button aria-label="검색">
        <SearchIcon />
      </Button>
    );
    const results = await axe(container);
    expect(results).toHaveNoViolations();
  });
});
```

```tsx
// Form.test.tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { axe } from 'jest-axe';
import { LoginForm } from './LoginForm';

describe('LoginForm 접근성', () => {
  test('접근성 위반이 없어야 함', async () => {
    const { container } = render(<LoginForm />);
    const results = await axe(container);
    expect(results).toHaveNoViolations();
  });

  test('키보드로 폼을 작성할 수 있어야 함', async () => {
    const user = userEvent.setup();
    render(<LoginForm />);

    // Tab으로 이동
    await user.tab();
    expect(screen.getByLabelText('이메일')).toHaveFocus();

    // 입력
    await user.keyboard('test@example.com');

    // 다음 필드로 이동
    await user.tab();
    expect(screen.getByLabelText('비밀번호')).toHaveFocus();

    await user.keyboard('password123');

    // 제출 버튼으로 이동
    await user.tab();
    expect(screen.getByRole('button', { name: '로그인' })).toHaveFocus();

    // Enter로 제출
    await user.keyboard('{Enter}');
  });

  test('에러 메시지가 스크린 리더에 알려져야 함', async () => {
    const user = userEvent.setup();
    render(<LoginForm />);

    const submitButton = screen.getByRole('button', { name: '로그인' });
    await user.click(submitButton);

    // role="alert"인 에러 메시지 확인
    const errorMessage = screen.getByRole('alert');
    expect(errorMessage).toHaveTextContent('이메일을 입력하세요');
  });
});
```

### 3. 수동 테스트 체크리스트

```tsx
// 테스트 체크리스트 컴포넌트
function A11yTestChecklist() {
  return (
    <div>
      <h2>접근성 수동 테스트 체크리스트</h2>

      <section>
        <h3>1. 키보드 네비게이션</h3>
        <ul>
          <li>
            <input type="checkbox" id="tab-order" />
            <label htmlFor="tab-order">
              Tab 키로 모든 인터랙티브 요소에 접근 가능한가?
            </label>
          </li>
          <li>
            <input type="checkbox" id="focus-visible" />
            <label htmlFor="focus-visible">
              포커스된 요소가 시각적으로 명확한가?
            </label>
          </li>
          <li>
            <input type="checkbox" id="keyboard-trap" />
            <label htmlFor="keyboard-trap">
              키보드 트랩이 없는가? (모달에서 빠져나올 수 있는가?)
            </label>
          </li>
          <li>
            <input type="checkbox" id="skip-link" />
            <label htmlFor="skip-link">
              Skip to main content 링크가 있는가?
            </label>
          </li>
        </ul>
      </section>

      <section>
        <h3>2. 스크린 리더</h3>
        <ul>
          <li>
            <input type="checkbox" id="screen-reader-test" />
            <label htmlFor="screen-reader-test">
              스크린 리더로 모든 콘텐츠를 읽을 수 있는가?
            </label>
          </li>
          <li>
            <input type="checkbox" id="alt-text" />
            <label htmlFor="alt-text">
              모든 이미지에 적절한 alt 텍스트가 있는가?
            </label>
          </li>
          <li>
            <input type="checkbox" id="headings" />
            <label htmlFor="headings">
              제목 구조가 논리적인가? (h1, h2, h3...)
            </label>
          </li>
          <li>
            <input type="checkbox" id="labels" />
            <label htmlFor="labels">
              모든 폼 필드에 레이블이 있는가?
            </label>
          </li>
        </ul>
      </section>

      <section>
        <h3>3. 시각적 접근성</h3>
        <ul>
          <li>
            <input type="checkbox" id="color-contrast" />
            <label htmlFor="color-contrast">
              색상 대비가 4.5:1 이상인가?
            </label>
          </li>
          <li>
            <input type="checkbox" id="text-resize" />
            <label htmlFor="text-resize">
              텍스트를 200% 확대해도 사용 가능한가?
            </label>
          </li>
          <li>
            <input type="checkbox" id="color-only" />
            <label htmlFor="color-only">
              색상에만 의존하지 않는가?
            </label>
          </li>
        </ul>
      </section>

      <section>
        <h3>4. 콘텐츠</h3>
        <ul>
          <li>
            <input type="checkbox" id="link-text" />
            <label htmlFor="link-text">
              링크 텍스트가 명확한가? ("여기를 클릭" 금지)
            </label>
          </li>
          <li>
            <input type="checkbox" id="error-messages" />
            <label htmlFor="error-messages">
              에러 메시지가 명확하고 구체적인가?
            </label>
          </li>
          <li>
            <input type="checkbox" id="time-limits" />
            <label htmlFor="time-limits">
              시간 제한이 있다면 연장 가능한가?
            </label>
          </li>
        </ul>
      </section>
    </div>
  );
}
```

### 4. CI/CD 통합

```yaml
# .github/workflows/accessibility.yml
name: Accessibility Tests

on:
  pull_request:
    branches: [main]

jobs:
  a11y-tests:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run accessibility tests
        run: npm run test:a11y

      - name: Build
        run: npm run build

      - name: Run Lighthouse CI
        uses: treosh/lighthouse-ci-action@v9
        with:
          urls: |
            http://localhost:3000
          uploadArtifacts: true
          temporaryPublicStorage: true
```

## 실전 체크리스트

### 개발 전 (설계 단계)

- [ ] 색상 팔레트의 대비 비율 확인 (4.5:1 이상)
- [ ] 키보드 네비게이션 플로우 설계
- [ ] 시맨틱 HTML 구조 계획
- [ ] 스크린 리더 경험 고려

### 개발 중

- [ ] 시맨틱 HTML 요소 사용 (`<button>`, `<nav>`, `<main>` 등)
- [ ] 모든 이미지에 alt 텍스트 추가
- [ ] 폼 필드에 `<label>` 연결
- [ ] ARIA 속성 적절히 사용 (과용하지 않기)
- [ ] 키보드로 모든 기능 테스트
- [ ] 포커스 관리 구현
- [ ] 에러 메시지를 `role="alert"`로 표시
- [ ] 동적 콘텐츠에 Live Regions 사용

### 개발 후 (QA 단계)

- [ ] axe DevTools로 자동 테스트
- [ ] Lighthouse 접근성 점수 90 이상
- [ ] 실제 스크린 리더로 테스트 (NVDA, VoiceOver)
- [ ] 키보드만으로 전체 플로우 테스트
- [ ] 텍스트 200% 확대 테스트
- [ ] 색맹 시뮬레이터로 확인
- [ ] 다양한 브라우저에서 테스트

### 지속적 개선

- [ ] 접근성 테스트를 CI/CD에 통합
- [ ] 정기적인 접근성 감사
- [ ] 사용자 피드백 수집 및 반영
- [ ] 팀 접근성 교육
- [ ] 접근성 가이드라인 문서화

## 마치며

웹 접근성은 **한 번 구현하고 끝나는 것이 아닙니다**. 지속적으로 관리하고 개선해야 하는 영역입니다.

**핵심 정리:**

1. **시맨틱 HTML이 최우선** - ARIA는 보완 수단
2. **키보드로 모든 것을 할 수 있어야 함** - 포커스 관리 중요
3. **스크린 리더 사용자 고려** - 명확한 레이블과 구조
4. **색상 대비 준수** - WCAG AA 기준 (4.5:1)
5. **자동화 테스트 + 수동 테스트** - 둘 다 필요

> 접근성은 일부 사용자를 위한 특별한 기능이 아닙니다. 모든 사람이 더 나은 경험을 할 수 있도록 만드는 것입니다.
{: .prompt-info }

**시작이 막막하다면:**

1. 새 기능 개발할 때마다 시맨틱 HTML 사용하기
2. axe DevTools 설치하고 주기적으로 검사하기
3. 한 번씩 키보드만으로 사이트 사용해보기
4. 팀원들과 접근성 중요성 공유하기

작은 것부터 시작해서 점진적으로 개선해 나가세요. **완벽한 접근성보다 지속적인 개선이 중요합니다!**

## FAQ

### Q1. 모든 div를 시맨틱 태그로 바꿔야 하나요?

**아니요.** 순수하게 레이아웃 목적의 컨테이너는 `<div>`를 사용해도 됩니다. 하지만 의미있는 콘텐츠 영역은 시맨틱 태그(`<header>`, `<nav>`, `<main>`, `<article>`, `<section>`, `<footer>`)를 사용하세요.

### Q2. ARIA를 많이 사용하면 접근성이 좋아지나요?

**아니요.** ARIA는 HTML로 표현할 수 없을 때만 사용하세요. 잘못 사용된 ARIA는 오히려 접근성을 해칩니다. "No ARIA is better than Bad ARIA"라는 원칙을 기억하세요.

### Q3. 접근성 준수가 SEO에 도움이 되나요?

**예.** 시맨틱 HTML, 명확한 구조, alt 텍스트는 검색 엔진이 콘텐츠를 이해하는 데 도움이 됩니다. 접근성과 SEO는 많은 부분에서 겹칩니다.

### Q4. 모바일 앱도 웹 접근성 기준을 따라야 하나요?

모바일 앱은 **WCAG 기반이지만 플랫폼별 가이드라인**을 따릅니다:
- iOS: [Apple Accessibility Guidelines](https://developer.apple.com/accessibility/)
- Android: [Android Accessibility Guidelines](https://developer.android.com/guide/topics/ui/accessibility)

### Q5. 스크린 리더 테스트는 어떻게 하나요?

**무료 옵션:**
- **Windows**: NVDA (무료 오픈소스)
- **macOS**: VoiceOver (내장)
- **iOS**: VoiceOver (내장)
- **Android**: TalkBack (내장)

기본적인 사용법을 익히고 주요 플로우를 테스트하세요.

## 참고 자료

### 공식 문서 및 가이드라인
- [WCAG 2.1 가이드라인](https://www.w3.org/WAI/WCAG21/quickref/)
- [WAI-ARIA Authoring Practices Guide (APG)](https://www.w3.org/WAI/ARIA/apg/)
- [MDN Web Accessibility](https://developer.mozilla.org/en-US/docs/Web/Accessibility)
- [React 접근성 문서](https://react.dev/learn/accessibility)
- [한국형 웹 콘텐츠 접근성 지침 (KWCAG)](https://www.wa.or.kr)

### 테스트 도구
- [axe DevTools](https://www.deque.com/axe/devtools/)
- [Lighthouse](https://developers.google.com/web/tools/lighthouse)
- [WAVE](https://wave.webaim.org/)
- [Color Contrast Checker](https://webaim.org/resources/contrastchecker/)
- [NVDA 스크린 리더](https://www.nvaccess.org/)

### React 접근성 라이브러리
- [Headless UI](https://headlessui.com/) - Tailwind CSS 팀
- [Radix UI](https://www.radix-ui.com/) - 접근성 우선 컴포넌트
- [React Aria](https://react-spectrum.adobe.com/react-aria/) - Adobe
- [Reach UI](https://reach.tech/) - React Router 팀

### 학습 자료
- [WebAIM](https://webaim.org/) - 접근성 학습 리소스
- [A11ycasts by Google](https://www.youtube.com/playlist?list=PLNYkxOF6rcICWx0C9LVWWVqvHlYJyqw7g)
- [The A11Y Project](https://www.a11yproject.com/)
- [Inclusive Components](https://inclusive-components.design/)

### 법률 및 정책
- [장애인차별금지법](https://www.law.go.kr/법령/장애인차별금지및권리구제등에관한법률)
- [ADA (Americans with Disabilities Act)](https://www.ada.gov/)
- [European Accessibility Act](https://ec.europa.eu/social/main.jsp?catId=1202)
