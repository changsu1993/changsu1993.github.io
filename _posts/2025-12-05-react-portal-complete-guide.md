---
title: "React Portal 완벽 가이드: 모달, 툴팁, 드롭다운 구현의 핵심"
description: "React Portal의 동작 원리부터 모달, 툴팁, 드롭다운 등 실전 활용법까지 완벽 정리. DOM 계층을 벗어나 UI를 렌더링하는 방법과 SSR 환경, 접근성, 애니메이션 적용법을 코드 예제와 함께 알아봅니다."
date: 2025-12-05 09:00:00 +0900
categories: [Frontend, React]
tags: [react, portal, modal, tooltip, dropdown, createPortal, overlay, accessibility]
---

## 개요

React 애플리케이션에서 모달, 툴팁, 드롭다운 같은 UI를 구현할 때 **z-index**나 **overflow** 문제로 고생한 경험이 있으신가요? **React Portal**은 컴포넌트를 DOM 트리의 다른 위치에 렌더링할 수 있게 해주는 강력한 기능으로, 이러한 문제를 우아하게 해결합니다.

이 글에서는 Portal의 기본 개념부터 모달, 툴팁, 드롭다운 구현, 애니메이션 적용, SSR 환경에서의 사용법까지 실전 예제와 함께 상세히 알아봅니다. Portal과 함께 [React Error Boundary](/posts/react-error-boundary-complete-guide/)를 활용하면 더욱 견고한 오버레이 UI를 구축할 수 있습니다.

---

## React Portal이란?

### DOM 트리 외부에 렌더링

일반적으로 React 컴포넌트는 부모 컴포넌트의 DOM 노드 내부에 렌더링됩니다. 하지만 **Portal**을 사용하면 **DOM 계층 구조와 상관없이** 원하는 DOM 노드에 자식 컴포넌트를 렌더링할 수 있습니다.

```tsx
import { createPortal } from 'react-dom';

function Modal({ children }) {
  // document.body에 직접 렌더링
  return createPortal(
    <div className="modal">{children}</div>,
    document.body
  );
}

function App() {
  return (
    <div className="app">
      <Header />
      <Modal>
        {/* 이 내용은 .app이 아닌 body에 렌더링됨 */}
        <h2>모달 제목</h2>
        <p>모달 내용</p>
      </Modal>
      <Footer />
    </div>
  );
}
```

### 렌더링 결과

위 코드의 실제 DOM 구조는 다음과 같습니다:

```html
<body>
  <div id="root">
    <div class="app">
      <header>...</header>
      <!-- Modal은 여기가 아닌 body에 렌더링 -->
      <footer>...</footer>
    </div>
  </div>
  <!-- Portal로 렌더링된 Modal -->
  <div class="modal">
    <h2>모달 제목</h2>
    <p>모달 내용</p>
  </div>
</body>
```

### React 컴포넌트 트리 vs DOM 트리

중요한 점은 **Portal로 렌더링된 컴포넌트도 여전히 React 컴포넌트 트리의 일부**라는 것입니다.

| 특성 | DOM 트리 | React 컴포넌트 트리 |
|------|----------|---------------------|
| 위치 | Portal 대상 노드 | 원래 부모 컴포넌트 하위 |
| Context | - | 부모로부터 정상 상속 |
| 이벤트 버블링 | DOM 계층 따름 | React 컴포넌트 트리 따름 |
| Props 전달 | - | 정상 동작 |

---

## Portal이 필요한 상황

### 1. z-index 스택 컨텍스트 문제

부모 요소에 `z-index`와 `position`이 설정되어 있으면 새로운 스택 컨텍스트가 생성됩니다. 이 경우 자식 요소의 `z-index`를 아무리 높여도 부모의 스택 컨텍스트를 벗어날 수 없습니다.

```tsx
function Card() {
  const [showTooltip, setShowTooltip] = useState(false);

  return (
    // 이 div가 스택 컨텍스트를 생성
    <div style={{ position: 'relative', zIndex: 1 }}>
      <button onMouseEnter={() => setShowTooltip(true)}>
        호버하세요
      </button>
      {showTooltip && (
        // z-index: 9999여도 부모의 z-index: 1을 벗어날 수 없음
        <div style={{ position: 'absolute', zIndex: 9999 }}>
          툴팁 내용
        </div>
      )}
    </div>
  );
}
```

**Portal 해결책:**

```tsx
function Tooltip({ children, targetRef }) {
  return createPortal(
    <div className="tooltip">
      {children}
    </div>,
    document.body
  );
}
```

### 2. overflow: hidden 문제

부모 요소에 `overflow: hidden`이 설정되어 있으면 자식 요소가 부모 영역 밖으로 나갈 수 없습니다.

```tsx
function Sidebar() {
  const [showDropdown, setShowDropdown] = useState(false);

  return (
    // overflow: hidden으로 자식 요소가 잘림
    <nav style={{ overflow: 'hidden', height: '100vh' }}>
      <button onClick={() => setShowDropdown(true)}>메뉴</button>
      {showDropdown && (
        // 사이드바 영역을 벗어나면 잘림
        <ul className="dropdown">
          <li>옵션 1</li>
          <li>옵션 2</li>
        </ul>
      )}
    </nav>
  );
}
```

### 3. transform 컨텍스트 문제

부모 요소에 `transform`이 적용되어 있으면 `position: fixed`가 제대로 동작하지 않습니다.

```tsx
function AnimatedContainer() {
  return (
    // transform이 있으면 fixed가 이 요소 기준이 됨
    <div style={{ transform: 'translateX(0)' }}>
      <Modal>
        {/* position: fixed여도 viewport가 아닌 부모 기준 */}
        <div style={{ position: 'fixed', top: 0, left: 0 }}>
          모달
        </div>
      </Modal>
    </div>
  );
}
```

### Portal이 필요한 UI 컴포넌트

| 컴포넌트 | 필요 이유 |
|----------|-----------|
| 모달/다이얼로그 | 화면 전체를 덮어야 함 |
| 툴팁 | 부모 overflow를 벗어나야 함 |
| 드롭다운 | 다른 요소 위에 표시되어야 함 |
| 토스트/알림 | 화면 특정 위치에 고정 |
| 컨텍스트 메뉴 | 클릭 위치에 표시 |
| 풀스크린 오버레이 | viewport 전체 커버 |

---

## createPortal API 사용법

### 기본 문법

```tsx
import { createPortal } from 'react-dom';

createPortal(children, domNode, key?)
```

| 매개변수 | 타입 | 설명 |
|----------|------|------|
| children | ReactNode | 렌더링할 React 요소 |
| domNode | Element | 렌더링 대상 DOM 노드 |
| key | string (선택) | Portal의 고유 키 |

### 기본 사용 예제

```tsx
import { createPortal } from 'react-dom';
import { useState } from 'react';

function PortalExample() {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <div>
      <button onClick={() => setIsOpen(true)}>
        포털 열기
      </button>

      {isOpen && createPortal(
        <div className="portal-content">
          <h2>Portal로 렌더링된 컨텐츠</h2>
          <button onClick={() => setIsOpen(false)}>
            닫기
          </button>
        </div>,
        document.body
      )}
    </div>
  );
}
```

### 커스텀 컨테이너 사용

Portal 전용 컨테이너를 만들어 사용하면 관리가 편리합니다.

```html
<!-- index.html -->
<body>
  <div id="root"></div>
  <div id="modal-root"></div>
  <div id="tooltip-root"></div>
  <div id="toast-root"></div>
</body>
```

```tsx
function Modal({ children }) {
  const modalRoot = document.getElementById('modal-root');
  if (!modalRoot) return null;

  return createPortal(children, modalRoot);
}

function Tooltip({ children }) {
  const tooltipRoot = document.getElementById('tooltip-root');
  if (!tooltipRoot) return null;

  return createPortal(children, tooltipRoot);
}
```

### Portal 컨테이너 동적 생성

```tsx
import { useEffect, useState } from 'react';
import { createPortal } from 'react-dom';

function Portal({ children, containerId = 'portal-root' }) {
  const [container, setContainer] = useState<HTMLElement | null>(null);

  useEffect(() => {
    let element = document.getElementById(containerId);

    if (!element) {
      element = document.createElement('div');
      element.id = containerId;
      document.body.appendChild(element);
    }

    setContainer(element);

    return () => {
      // 컴포넌트 언마운트 시 빈 컨테이너 정리 (선택적)
      if (element && element.childNodes.length === 0) {
        element.remove();
      }
    };
  }, [containerId]);

  if (!container) return null;

  return createPortal(children, container);
}
```

---

## 모달 컴포넌트 구현

### 기본 모달

```tsx
import { createPortal } from 'react-dom';
import { ReactNode } from 'react';

interface ModalProps {
  isOpen: boolean;
  onClose: () => void;
  children: ReactNode;
  title?: string;
}

function Modal({ isOpen, onClose, children, title }: ModalProps) {
  if (!isOpen) return null;

  return createPortal(
    <div className="modal-overlay" onClick={onClose}>
      <div
        className="modal-content"
        onClick={(e) => e.stopPropagation()}
      >
        {title && (
          <div className="modal-header">
            <h2>{title}</h2>
            <button
              className="modal-close"
              onClick={onClose}
              aria-label="닫기"
            >
              &times;
            </button>
          </div>
        )}
        <div className="modal-body">{children}</div>
      </div>
    </div>,
    document.body
  );
}
```

### 접근성을 고려한 모달

키보드 내비게이션, 포커스 트랩, 스크린 리더 지원을 추가합니다. 웹 접근성에 대한 더 자세한 내용은 [웹 접근성 완벽 가이드](/posts/web-accessibility-guide/)를 참고하세요.

```tsx
import { createPortal } from 'react-dom';
import {
  ReactNode,
  useEffect,
  useRef,
  useCallback,
  KeyboardEvent,
} from 'react';

interface AccessibleModalProps {
  isOpen: boolean;
  onClose: () => void;
  children: ReactNode;
  title: string;
  description?: string;
}

function AccessibleModal({
  isOpen,
  onClose,
  children,
  title,
  description,
}: AccessibleModalProps) {
  const modalRef = useRef<HTMLDivElement>(null);
  const previousActiveElement = useRef<HTMLElement | null>(null);

  // 포커스 트랩: 모달 내부에서만 Tab 이동
  const handleKeyDown = useCallback(
    (e: KeyboardEvent) => {
      if (e.key === 'Escape') {
        onClose();
        return;
      }

      if (e.key !== 'Tab') return;

      const modal = modalRef.current;
      if (!modal) return;

      const focusableElements = modal.querySelectorAll<HTMLElement>(
        'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
      );
      const firstElement = focusableElements[0];
      const lastElement = focusableElements[focusableElements.length - 1];

      if (e.shiftKey) {
        if (document.activeElement === firstElement) {
          e.preventDefault();
          lastElement.focus();
        }
      } else {
        if (document.activeElement === lastElement) {
          e.preventDefault();
          firstElement.focus();
        }
      }
    },
    [onClose]
  );

  useEffect(() => {
    if (isOpen) {
      // 현재 포커스된 요소 저장
      previousActiveElement.current = document.activeElement as HTMLElement;

      // 모달에 포커스
      modalRef.current?.focus();

      // 배경 스크롤 방지
      document.body.style.overflow = 'hidden';
    }

    return () => {
      // 이전 포커스 복원
      previousActiveElement.current?.focus();

      // 스크롤 복원
      document.body.style.overflow = '';
    };
  }, [isOpen]);

  if (!isOpen) return null;

  const modalId = `modal-${title.replace(/\s+/g, '-').toLowerCase()}`;

  return createPortal(
    <div
      className="modal-overlay"
      onClick={onClose}
      role="presentation"
    >
      <div
        ref={modalRef}
        className="modal-content"
        role="dialog"
        aria-modal="true"
        aria-labelledby={`${modalId}-title`}
        aria-describedby={description ? `${modalId}-description` : undefined}
        tabIndex={-1}
        onClick={(e) => e.stopPropagation()}
        onKeyDown={handleKeyDown}
      >
        <div className="modal-header">
          <h2 id={`${modalId}-title`}>{title}</h2>
          <button
            className="modal-close"
            onClick={onClose}
            aria-label="모달 닫기"
          >
            <svg
              width="24"
              height="24"
              viewBox="0 0 24 24"
              fill="none"
              stroke="currentColor"
              strokeWidth="2"
            >
              <path d="M18 6L6 18M6 6l12 12" />
            </svg>
          </button>
        </div>

        {description && (
          <p id={`${modalId}-description`} className="sr-only">
            {description}
          </p>
        )}

        <div className="modal-body">{children}</div>
      </div>
    </div>,
    document.body
  );
}
```

### 모달 스타일

```css
.modal-overlay {
  position: fixed;
  inset: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.modal-content {
  background: white;
  border-radius: 8px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15);
  max-width: 500px;
  width: 90%;
  max-height: 90vh;
  overflow-y: auto;
  outline: none;
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 16px 20px;
  border-bottom: 1px solid #e5e5e5;
}

.modal-header h2 {
  margin: 0;
  font-size: 18px;
  font-weight: 600;
}

.modal-close {
  background: none;
  border: none;
  cursor: pointer;
  padding: 4px;
  color: #666;
  transition: color 0.2s;
}

.modal-close:hover {
  color: #000;
}

.modal-body {
  padding: 20px;
}

/* 스크린 리더 전용 */
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  border: 0;
}
```

### 모달 사용 예시

```tsx
function App() {
  const [isModalOpen, setIsModalOpen] = useState(false);

  return (
    <div>
      <button onClick={() => setIsModalOpen(true)}>
        모달 열기
      </button>

      <AccessibleModal
        isOpen={isModalOpen}
        onClose={() => setIsModalOpen(false)}
        title="회원가입"
        description="회원가입 양식을 작성해주세요"
      >
        <form>
          <div>
            <label htmlFor="email">이메일</label>
            <input id="email" type="email" />
          </div>
          <div>
            <label htmlFor="password">비밀번호</label>
            <input id="password" type="password" />
          </div>
          <button type="submit">가입하기</button>
        </form>
      </AccessibleModal>
    </div>
  );
}
```

---

## 툴팁 컴포넌트 구현

### 위치 계산 로직

툴팁은 트리거 요소의 위치를 기준으로 배치해야 합니다.

```tsx
import { createPortal } from 'react-dom';
import {
  useState,
  useRef,
  useEffect,
  useCallback,
  ReactNode,
} from 'react';

type TooltipPosition = 'top' | 'bottom' | 'left' | 'right';

interface TooltipProps {
  content: ReactNode;
  position?: TooltipPosition;
  delay?: number;
  children: ReactNode;
}

function Tooltip({
  content,
  position = 'top',
  delay = 200,
  children,
}: TooltipProps) {
  const [isVisible, setIsVisible] = useState(false);
  const [coords, setCoords] = useState({ x: 0, y: 0 });
  const triggerRef = useRef<HTMLDivElement>(null);
  const tooltipRef = useRef<HTMLDivElement>(null);
  const timeoutRef = useRef<number>();

  const calculatePosition = useCallback(() => {
    if (!triggerRef.current || !tooltipRef.current) return;

    const triggerRect = triggerRef.current.getBoundingClientRect();
    const tooltipRect = tooltipRef.current.getBoundingClientRect();
    const gap = 8;

    let x = 0;
    let y = 0;

    switch (position) {
      case 'top':
        x = triggerRect.left + (triggerRect.width - tooltipRect.width) / 2;
        y = triggerRect.top - tooltipRect.height - gap;
        break;
      case 'bottom':
        x = triggerRect.left + (triggerRect.width - tooltipRect.width) / 2;
        y = triggerRect.bottom + gap;
        break;
      case 'left':
        x = triggerRect.left - tooltipRect.width - gap;
        y = triggerRect.top + (triggerRect.height - tooltipRect.height) / 2;
        break;
      case 'right':
        x = triggerRect.right + gap;
        y = triggerRect.top + (triggerRect.height - tooltipRect.height) / 2;
        break;
    }

    // 뷰포트 경계 체크
    const viewportWidth = window.innerWidth;
    const viewportHeight = window.innerHeight;

    // 좌우 경계
    if (x < 0) x = gap;
    if (x + tooltipRect.width > viewportWidth) {
      x = viewportWidth - tooltipRect.width - gap;
    }

    // 상하 경계
    if (y < 0) y = gap;
    if (y + tooltipRect.height > viewportHeight) {
      y = viewportHeight - tooltipRect.height - gap;
    }

    setCoords({ x, y });
  }, [position]);

  const showTooltip = () => {
    timeoutRef.current = window.setTimeout(() => {
      setIsVisible(true);
    }, delay);
  };

  const hideTooltip = () => {
    if (timeoutRef.current) {
      clearTimeout(timeoutRef.current);
    }
    setIsVisible(false);
  };

  useEffect(() => {
    if (isVisible) {
      calculatePosition();

      // 스크롤/리사이즈 시 위치 재계산
      window.addEventListener('scroll', calculatePosition, true);
      window.addEventListener('resize', calculatePosition);

      return () => {
        window.removeEventListener('scroll', calculatePosition, true);
        window.removeEventListener('resize', calculatePosition);
      };
    }
  }, [isVisible, calculatePosition]);

  useEffect(() => {
    return () => {
      if (timeoutRef.current) {
        clearTimeout(timeoutRef.current);
      }
    };
  }, []);

  return (
    <>
      <div
        ref={triggerRef}
        onMouseEnter={showTooltip}
        onMouseLeave={hideTooltip}
        onFocus={showTooltip}
        onBlur={hideTooltip}
        style={{ display: 'inline-block' }}
      >
        {children}
      </div>

      {isVisible &&
        createPortal(
          <div
            ref={tooltipRef}
            className={`tooltip tooltip-${position}`}
            role="tooltip"
            style={{
              position: 'fixed',
              left: `${coords.x}px`,
              top: `${coords.y}px`,
            }}
          >
            {content}
          </div>,
          document.body
        )}
    </>
  );
}
```

### 툴팁 스타일

```css
.tooltip {
  background-color: #333;
  color: white;
  padding: 8px 12px;
  border-radius: 4px;
  font-size: 14px;
  max-width: 300px;
  z-index: 1100;
  pointer-events: none;
  animation: tooltip-fade-in 0.15s ease-out;
}

@keyframes tooltip-fade-in {
  from {
    opacity: 0;
    transform: translateY(4px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* 화살표 */
.tooltip::before {
  content: '';
  position: absolute;
  border: 6px solid transparent;
}

.tooltip-top::before {
  bottom: -12px;
  left: 50%;
  transform: translateX(-50%);
  border-top-color: #333;
}

.tooltip-bottom::before {
  top: -12px;
  left: 50%;
  transform: translateX(-50%);
  border-bottom-color: #333;
}

.tooltip-left::before {
  right: -12px;
  top: 50%;
  transform: translateY(-50%);
  border-left-color: #333;
}

.tooltip-right::before {
  left: -12px;
  top: 50%;
  transform: translateY(-50%);
  border-right-color: #333;
}
```

### 툴팁 사용 예시

```tsx
function App() {
  return (
    <div className="space-x-4 p-8">
      <Tooltip content="저장합니다">
        <button>저장</button>
      </Tooltip>

      <Tooltip content="삭제하면 복구할 수 없습니다" position="bottom">
        <button>삭제</button>
      </Tooltip>

      <Tooltip
        content={
          <div>
            <strong>사용자 정보</strong>
            <p>이름: 홍길동</p>
            <p>이메일: hong@example.com</p>
          </div>
        }
        position="right"
      >
        <span className="cursor-pointer underline">프로필 보기</span>
      </Tooltip>
    </div>
  );
}
```

---

## 드롭다운 메뉴 구현

```tsx
import { createPortal } from 'react-dom';
import {
  useState,
  useRef,
  useEffect,
  useCallback,
  ReactNode,
  KeyboardEvent,
} from 'react';

interface DropdownItem {
  label: string;
  value: string;
  icon?: ReactNode;
  disabled?: boolean;
  onClick?: () => void;
}

interface DropdownProps {
  trigger: ReactNode;
  items: DropdownItem[];
  onSelect?: (value: string) => void;
}

function Dropdown({ trigger, items, onSelect }: DropdownProps) {
  const [isOpen, setIsOpen] = useState(false);
  const [coords, setCoords] = useState({ x: 0, y: 0, width: 0 });
  const [activeIndex, setActiveIndex] = useState(-1);

  const triggerRef = useRef<HTMLButtonElement>(null);
  const menuRef = useRef<HTMLUListElement>(null);

  const calculatePosition = useCallback(() => {
    if (!triggerRef.current) return;

    const rect = triggerRef.current.getBoundingClientRect();
    const menuHeight = 200; // 예상 메뉴 높이
    const viewportHeight = window.innerHeight;

    // 아래에 공간이 부족하면 위에 표시
    const showAbove = rect.bottom + menuHeight > viewportHeight;

    setCoords({
      x: rect.left,
      y: showAbove ? rect.top : rect.bottom,
      width: rect.width,
    });
  }, []);

  const openDropdown = () => {
    calculatePosition();
    setIsOpen(true);
    setActiveIndex(-1);
  };

  const closeDropdown = () => {
    setIsOpen(false);
    setActiveIndex(-1);
    triggerRef.current?.focus();
  };

  const handleSelect = (item: DropdownItem) => {
    if (item.disabled) return;

    item.onClick?.();
    onSelect?.(item.value);
    closeDropdown();
  };

  const handleKeyDown = (e: KeyboardEvent) => {
    if (!isOpen) {
      if (e.key === 'Enter' || e.key === ' ' || e.key === 'ArrowDown') {
        e.preventDefault();
        openDropdown();
      }
      return;
    }

    switch (e.key) {
      case 'Escape':
        e.preventDefault();
        closeDropdown();
        break;

      case 'ArrowDown':
        e.preventDefault();
        setActiveIndex((prev) => {
          const next = prev + 1;
          return next >= items.length ? 0 : next;
        });
        break;

      case 'ArrowUp':
        e.preventDefault();
        setActiveIndex((prev) => {
          const next = prev - 1;
          return next < 0 ? items.length - 1 : next;
        });
        break;

      case 'Enter':
      case ' ':
        e.preventDefault();
        if (activeIndex >= 0) {
          handleSelect(items[activeIndex]);
        }
        break;

      case 'Tab':
        closeDropdown();
        break;
    }
  };

  // 외부 클릭 감지
  useEffect(() => {
    if (!isOpen) return;

    const handleClickOutside = (e: MouseEvent) => {
      const target = e.target as Node;
      if (
        !triggerRef.current?.contains(target) &&
        !menuRef.current?.contains(target)
      ) {
        closeDropdown();
      }
    };

    document.addEventListener('mousedown', handleClickOutside);
    return () => document.removeEventListener('mousedown', handleClickOutside);
  }, [isOpen]);

  // 스크롤/리사이즈 시 위치 재계산
  useEffect(() => {
    if (!isOpen) return;

    window.addEventListener('scroll', calculatePosition, true);
    window.addEventListener('resize', calculatePosition);

    return () => {
      window.removeEventListener('scroll', calculatePosition, true);
      window.removeEventListener('resize', calculatePosition);
    };
  }, [isOpen, calculatePosition]);

  // 활성 아이템으로 스크롤
  useEffect(() => {
    if (activeIndex >= 0 && menuRef.current) {
      const activeItem = menuRef.current.children[activeIndex] as HTMLElement;
      activeItem?.scrollIntoView({ block: 'nearest' });
    }
  }, [activeIndex]);

  return (
    <>
      <button
        ref={triggerRef}
        onClick={() => (isOpen ? closeDropdown() : openDropdown())}
        onKeyDown={handleKeyDown}
        aria-haspopup="listbox"
        aria-expanded={isOpen}
        className="dropdown-trigger"
      >
        {trigger}
      </button>

      {isOpen &&
        createPortal(
          <ul
            ref={menuRef}
            role="listbox"
            className="dropdown-menu"
            style={{
              position: 'fixed',
              left: `${coords.x}px`,
              top: `${coords.y}px`,
              minWidth: `${coords.width}px`,
            }}
          >
            {items.map((item, index) => (
              <li
                key={item.value}
                role="option"
                aria-selected={index === activeIndex}
                aria-disabled={item.disabled}
                className={`dropdown-item ${
                  index === activeIndex ? 'active' : ''
                } ${item.disabled ? 'disabled' : ''}`}
                onClick={() => handleSelect(item)}
                onMouseEnter={() => setActiveIndex(index)}
              >
                {item.icon && <span className="dropdown-icon">{item.icon}</span>}
                {item.label}
              </li>
            ))}
          </ul>,
          document.body
        )}
    </>
  );
}
```

### 드롭다운 스타일

```css
.dropdown-trigger {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 8px 16px;
  background: white;
  border: 1px solid #ddd;
  border-radius: 4px;
  cursor: pointer;
}

.dropdown-menu {
  background: white;
  border: 1px solid #ddd;
  border-radius: 4px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  max-height: 200px;
  overflow-y: auto;
  z-index: 1100;
  list-style: none;
  margin: 4px 0;
  padding: 4px 0;
}

.dropdown-item {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 12px;
  cursor: pointer;
  transition: background-color 0.1s;
}

.dropdown-item:hover,
.dropdown-item.active {
  background-color: #f5f5f5;
}

.dropdown-item.disabled {
  color: #999;
  cursor: not-allowed;
}

.dropdown-icon {
  display: flex;
  width: 20px;
  height: 20px;
}
```

### 드롭다운 사용 예시

```tsx
function App() {
  const [selected, setSelected] = useState('');

  const items = [
    { label: '프로필', value: 'profile' },
    { label: '설정', value: 'settings' },
    { label: '알림', value: 'notifications' },
    { label: '로그아웃', value: 'logout' },
  ];

  return (
    <Dropdown
      trigger={<span>메뉴 {selected && `(${selected})`}</span>}
      items={items}
      onSelect={setSelected}
    />
  );
}
```

---

## Portal에서의 이벤트 버블링

Portal의 중요한 특징 중 하나는 **이벤트가 React 컴포넌트 트리를 따라 버블링**된다는 것입니다.

### 이벤트 버블링 예제

```tsx
function Parent() {
  const handleClick = () => {
    console.log('부모 클릭!');
  };

  return (
    <div onClick={handleClick}>
      <p>부모 컴포넌트</p>
      <PortalChild />
    </div>
  );
}

function PortalChild() {
  return createPortal(
    <button onClick={() => console.log('자식 클릭!')}>
      Portal 버튼
    </button>,
    document.body
  );
}

// "Portal 버튼" 클릭 시:
// 1. "자식 클릭!" 출력
// 2. "부모 클릭!" 출력 (이벤트 버블링)
```

### 이벤트 버블링 활용

이 특성을 활용하면 모달 내부의 이벤트를 부모에서 처리할 수 있습니다.

```tsx
function FormWithModal() {
  const handleSubmit = (e: FormEvent) => {
    e.preventDefault();
    console.log('폼 제출됨');
  };

  return (
    <form onSubmit={handleSubmit}>
      <input name="email" placeholder="이메일" />

      <Modal isOpen={true}>
        {/* Portal 내부지만 form의 onSubmit이 동작 */}
        <button type="submit">제출</button>
      </Modal>
    </form>
  );
}
```

### 이벤트 전파 막기

필요한 경우 `stopPropagation()`으로 이벤트 전파를 막을 수 있습니다.

```tsx
function Modal({ children, onClose }) {
  return createPortal(
    <div className="overlay" onClick={onClose}>
      <div
        className="content"
        onClick={(e) => e.stopPropagation()} // 내부 클릭은 전파 안됨
      >
        {children}
      </div>
    </div>,
    document.body
  );
}
```

---

## Portal과 Context 연동

Portal로 렌더링된 컴포넌트도 React Context를 정상적으로 사용할 수 있습니다.

### Context 상속 예제

```tsx
import { createContext, useContext, useState, ReactNode } from 'react';
import { createPortal } from 'react-dom';

// 테마 Context
const ThemeContext = createContext<'light' | 'dark'>('light');

function ThemeProvider({ children }: { children: ReactNode }) {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');

  return (
    <ThemeContext.Provider value={theme}>
      <button onClick={() => setTheme((t) => (t === 'light' ? 'dark' : 'light'))}>
        테마 전환
      </button>
      {children}
    </ThemeContext.Provider>
  );
}

function ThemedModal({ isOpen, onClose }) {
  // Portal 내부에서도 Context 값 사용 가능
  const theme = useContext(ThemeContext);

  if (!isOpen) return null;

  return createPortal(
    <div className={`modal-overlay theme-${theme}`}>
      <div className="modal-content">
        <p>현재 테마: {theme}</p>
        <button onClick={onClose}>닫기</button>
      </div>
    </div>,
    document.body
  );
}

function App() {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <ThemeProvider>
      <button onClick={() => setIsOpen(true)}>모달 열기</button>
      <ThemedModal isOpen={isOpen} onClose={() => setIsOpen(false)} />
    </ThemeProvider>
  );
}
```

### 모달 매니저 구현

Context를 활용한 전역 모달 관리 시스템을 구현할 수 있습니다.

```tsx
import {
  createContext,
  useContext,
  useState,
  useCallback,
  ReactNode,
} from 'react';
import { createPortal } from 'react-dom';

interface ModalConfig {
  id: string;
  component: ReactNode;
}

interface ModalContextType {
  openModal: (id: string, component: ReactNode) => void;
  closeModal: (id: string) => void;
  closeAllModals: () => void;
}

const ModalContext = createContext<ModalContextType | null>(null);

function ModalProvider({ children }: { children: ReactNode }) {
  const [modals, setModals] = useState<ModalConfig[]>([]);

  const openModal = useCallback((id: string, component: ReactNode) => {
    setModals((prev) => {
      // 이미 열린 모달이면 무시
      if (prev.find((m) => m.id === id)) return prev;
      return [...prev, { id, component }];
    });
  }, []);

  const closeModal = useCallback((id: string) => {
    setModals((prev) => prev.filter((m) => m.id !== id));
  }, []);

  const closeAllModals = useCallback(() => {
    setModals([]);
  }, []);

  return (
    <ModalContext.Provider value={{ openModal, closeModal, closeAllModals }}>
      {children}
      {modals.map((modal) =>
        createPortal(
          <div key={modal.id} className="modal-wrapper">
            {modal.component}
          </div>,
          document.body
        )
      )}
    </ModalContext.Provider>
  );
}

function useModal() {
  const context = useContext(ModalContext);
  if (!context) {
    throw new Error('useModal must be used within ModalProvider');
  }
  return context;
}

// 사용 예시
function Dashboard() {
  const { openModal, closeModal } = useModal();

  const handleOpenConfirm = () => {
    openModal(
      'confirm-delete',
      <ConfirmModal
        title="삭제 확인"
        message="정말 삭제하시겠습니까?"
        onConfirm={() => {
          console.log('삭제됨');
          closeModal('confirm-delete');
        }}
        onCancel={() => closeModal('confirm-delete')}
      />
    );
  };

  return <button onClick={handleOpenConfirm}>항목 삭제</button>;
}
```

---

## 애니메이션 적용

### CSS 트랜지션

```tsx
import { useState, useEffect } from 'react';
import { createPortal } from 'react-dom';

function AnimatedModal({ isOpen, onClose, children }) {
  const [shouldRender, setShouldRender] = useState(false);
  const [isAnimating, setIsAnimating] = useState(false);

  useEffect(() => {
    if (isOpen) {
      setShouldRender(true);
      // 다음 프레임에서 애니메이션 시작
      requestAnimationFrame(() => {
        requestAnimationFrame(() => {
          setIsAnimating(true);
        });
      });
    } else {
      setIsAnimating(false);
    }
  }, [isOpen]);

  const handleTransitionEnd = () => {
    if (!isOpen) {
      setShouldRender(false);
    }
  };

  if (!shouldRender) return null;

  return createPortal(
    <div
      className={`modal-overlay ${isAnimating ? 'visible' : ''}`}
      onClick={onClose}
      onTransitionEnd={handleTransitionEnd}
    >
      <div
        className={`modal-content ${isAnimating ? 'visible' : ''}`}
        onClick={(e) => e.stopPropagation()}
      >
        {children}
      </div>
    </div>,
    document.body
  );
}
```

```css
.modal-overlay {
  position: fixed;
  inset: 0;
  background-color: rgba(0, 0, 0, 0);
  display: flex;
  align-items: center;
  justify-content: center;
  transition: background-color 0.2s ease;
}

.modal-overlay.visible {
  background-color: rgba(0, 0, 0, 0.5);
}

.modal-content {
  transform: scale(0.9);
  opacity: 0;
  transition: transform 0.2s ease, opacity 0.2s ease;
}

.modal-content.visible {
  transform: scale(1);
  opacity: 1;
}
```

### Framer Motion 연동

Framer Motion을 사용하면 더 풍부한 애니메이션을 쉽게 구현할 수 있습니다. 더 자세한 Framer Motion 사용법은 [Framer Motion 애니메이션 가이드](/posts/framer-motion-animation-guide/)를 참고하세요.

```tsx
import { createPortal } from 'react-dom';
import { AnimatePresence, motion } from 'framer-motion';

interface AnimatedModalProps {
  isOpen: boolean;
  onClose: () => void;
  children: ReactNode;
}

function AnimatedModal({ isOpen, onClose, children }: AnimatedModalProps) {
  return createPortal(
    <AnimatePresence>
      {isOpen && (
        <motion.div
          className="modal-overlay"
          initial={{ opacity: 0 }}
          animate={{ opacity: 1 }}
          exit={{ opacity: 0 }}
          transition={{ duration: 0.2 }}
          onClick={onClose}
        >
          <motion.div
            className="modal-content"
            initial={{ scale: 0.9, opacity: 0, y: 20 }}
            animate={{ scale: 1, opacity: 1, y: 0 }}
            exit={{ scale: 0.9, opacity: 0, y: 20 }}
            transition={{ type: 'spring', damping: 25, stiffness: 300 }}
            onClick={(e) => e.stopPropagation()}
          >
            {children}
          </motion.div>
        </motion.div>
      )}
    </AnimatePresence>,
    document.body
  );
}
```

### 토스트 애니메이션

```tsx
import { createPortal } from 'react-dom';
import { AnimatePresence, motion } from 'framer-motion';

interface Toast {
  id: string;
  message: string;
  type: 'success' | 'error' | 'info';
}

function ToastContainer({ toasts }: { toasts: Toast[] }) {
  return createPortal(
    <div className="toast-container">
      <AnimatePresence>
        {toasts.map((toast) => (
          <motion.div
            key={toast.id}
            className={`toast toast-${toast.type}`}
            initial={{ opacity: 0, x: 100 }}
            animate={{ opacity: 1, x: 0 }}
            exit={{ opacity: 0, x: 100, height: 0 }}
            transition={{ type: 'spring', damping: 20 }}
          >
            {toast.message}
          </motion.div>
        ))}
      </AnimatePresence>
    </div>,
    document.body
  );
}
```

```css
.toast-container {
  position: fixed;
  top: 20px;
  right: 20px;
  display: flex;
  flex-direction: column;
  gap: 8px;
  z-index: 1200;
}

.toast {
  padding: 12px 20px;
  border-radius: 4px;
  color: white;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
}

.toast-success {
  background-color: #10b981;
}

.toast-error {
  background-color: #ef4444;
}

.toast-info {
  background-color: #3b82f6;
}
```

---

## SSR 환경에서의 Portal

### 문제점

서버 사이드 렌더링 환경에서는 `document` 객체가 존재하지 않아 Portal이 에러를 발생시킵니다.

```tsx
// SSR에서 에러 발생!
function Modal({ children }) {
  return createPortal(
    children,
    document.body // ReferenceError: document is not defined
  );
}
```

### 해결책 1: 클라이언트 전용 렌더링

```tsx
import { useState, useEffect, ReactNode } from 'react';
import { createPortal } from 'react-dom';

function ClientOnlyPortal({
  children,
  selector = '#portal-root',
}: {
  children: ReactNode;
  selector?: string;
}) {
  const [mounted, setMounted] = useState(false);
  const [container, setContainer] = useState<Element | null>(null);

  useEffect(() => {
    setMounted(true);
    setContainer(document.querySelector(selector));
  }, [selector]);

  if (!mounted || !container) return null;

  return createPortal(children, container);
}
```

### 해결책 2: Next.js에서의 Portal

```tsx
'use client';

import { useState, useEffect, ReactNode } from 'react';
import { createPortal } from 'react-dom';

function Portal({ children }: { children: ReactNode }) {
  const [portalElement, setPortalElement] = useState<Element | null>(null);

  useEffect(() => {
    // 클라이언트에서만 실행
    let element = document.getElementById('portal-root');

    if (!element) {
      element = document.createElement('div');
      element.id = 'portal-root';
      document.body.appendChild(element);
    }

    setPortalElement(element);
  }, []);

  if (!portalElement) return null;

  return createPortal(children, portalElement);
}

export default Portal;
```

### Next.js App Router에서의 모달

```tsx
// components/Modal.tsx
'use client';

import { useEffect, useState, ReactNode } from 'react';
import { createPortal } from 'react-dom';

interface ModalProps {
  isOpen: boolean;
  onClose: () => void;
  children: ReactNode;
}

export function Modal({ isOpen, onClose, children }: ModalProps) {
  const [mounted, setMounted] = useState(false);

  useEffect(() => {
    setMounted(true);
  }, []);

  useEffect(() => {
    if (isOpen) {
      document.body.style.overflow = 'hidden';
    }
    return () => {
      document.body.style.overflow = '';
    };
  }, [isOpen]);

  if (!mounted || !isOpen) return null;

  return createPortal(
    <div className="modal-overlay" onClick={onClose}>
      <div
        className="modal-content"
        onClick={(e) => e.stopPropagation()}
      >
        {children}
      </div>
    </div>,
    document.body
  );
}
```

```tsx
// app/page.tsx
'use client';

import { useState } from 'react';
import { Modal } from '@/components/Modal';

export default function Page() {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <div>
      <button onClick={() => setIsOpen(true)}>모달 열기</button>
      <Modal isOpen={isOpen} onClose={() => setIsOpen(false)}>
        <h2>Next.js 모달</h2>
        <p>SSR 환경에서도 정상 동작합니다.</p>
      </Modal>
    </div>
  );
}
```

Next.js App Router의 더 자세한 사용법은 [Next.js 15 App Router 완벽 가이드](/posts/nextjs-15-app-router-guide/)를 참고하세요. SSR 환경에서 데이터 로딩과 함께 Portal을 사용할 때는 [React Suspense 완벽 가이드](/posts/react-suspense-complete-guide/)도 함께 참고하면 좋습니다.

---

## 테스트 작성 방법

### React Testing Library로 Portal 테스트

```tsx
import { render, screen, fireEvent } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { Modal } from './Modal';

describe('Modal', () => {
  beforeEach(() => {
    // Portal 컨테이너 생성
    const portalRoot = document.createElement('div');
    portalRoot.id = 'portal-root';
    document.body.appendChild(portalRoot);
  });

  afterEach(() => {
    // 테스트 후 정리
    const portalRoot = document.getElementById('portal-root');
    if (portalRoot) {
      portalRoot.remove();
    }
    document.body.style.overflow = '';
  });

  it('isOpen이 true일 때 모달이 렌더링된다', () => {
    render(
      <Modal isOpen={true} onClose={() => {}}>
        <p>모달 내용</p>
      </Modal>
    );

    expect(screen.getByText('모달 내용')).toBeInTheDocument();
  });

  it('isOpen이 false일 때 모달이 렌더링되지 않는다', () => {
    render(
      <Modal isOpen={false} onClose={() => {}}>
        <p>모달 내용</p>
      </Modal>
    );

    expect(screen.queryByText('모달 내용')).not.toBeInTheDocument();
  });

  it('배경 클릭 시 onClose가 호출된다', async () => {
    const handleClose = jest.fn();
    const user = userEvent.setup();

    render(
      <Modal isOpen={true} onClose={handleClose}>
        <p>모달 내용</p>
      </Modal>
    );

    // 오버레이 클릭
    await user.click(screen.getByRole('presentation'));

    expect(handleClose).toHaveBeenCalledTimes(1);
  });

  it('모달 내용 클릭 시 onClose가 호출되지 않는다', async () => {
    const handleClose = jest.fn();
    const user = userEvent.setup();

    render(
      <Modal isOpen={true} onClose={handleClose}>
        <p>모달 내용</p>
      </Modal>
    );

    await user.click(screen.getByText('모달 내용'));

    expect(handleClose).not.toHaveBeenCalled();
  });

  it('Escape 키 입력 시 onClose가 호출된다', async () => {
    const handleClose = jest.fn();
    const user = userEvent.setup();

    render(
      <Modal isOpen={true} onClose={handleClose}>
        <p>모달 내용</p>
      </Modal>
    );

    await user.keyboard('{Escape}');

    expect(handleClose).toHaveBeenCalledTimes(1);
  });

  it('모달 열림 시 body 스크롤이 비활성화된다', () => {
    render(
      <Modal isOpen={true} onClose={() => {}}>
        <p>모달 내용</p>
      </Modal>
    );

    expect(document.body.style.overflow).toBe('hidden');
  });

  it('모달이 document.body에 렌더링된다', () => {
    render(
      <Modal isOpen={true} onClose={() => {}}>
        <p>모달 내용</p>
      </Modal>
    );

    // Portal로 body에 직접 렌더링됨
    const modalOverlay = document.body.querySelector('.modal-overlay');
    expect(modalOverlay).toBeInTheDocument();
  });
});
```

### 툴팁 테스트

```tsx
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { Tooltip } from './Tooltip';

describe('Tooltip', () => {
  it('hover 시 툴팁이 표시된다', async () => {
    const user = userEvent.setup();

    render(
      <Tooltip content="도움말 텍스트">
        <button>호버하세요</button>
      </Tooltip>
    );

    expect(screen.queryByRole('tooltip')).not.toBeInTheDocument();

    await user.hover(screen.getByRole('button'));

    await waitFor(() => {
      expect(screen.getByRole('tooltip')).toBeInTheDocument();
    });

    expect(screen.getByText('도움말 텍스트')).toBeInTheDocument();
  });

  it('hover 해제 시 툴팁이 사라진다', async () => {
    const user = userEvent.setup();

    render(
      <Tooltip content="도움말 텍스트" delay={0}>
        <button>호버하세요</button>
      </Tooltip>
    );

    await user.hover(screen.getByRole('button'));
    await waitFor(() => {
      expect(screen.getByRole('tooltip')).toBeInTheDocument();
    });

    await user.unhover(screen.getByRole('button'));
    await waitFor(() => {
      expect(screen.queryByRole('tooltip')).not.toBeInTheDocument();
    });
  });
});
```

프론트엔드 테스트에 대한 더 자세한 내용은 [프론트엔드 테스트 완벽 가이드](/posts/frontend-testing-guide/)를 참고하세요.

---

## 흔한 실수와 해결책

### 1. SSR에서 document 접근 오류

**문제:**
```tsx
// 서버에서 실행되면 에러
const container = document.getElementById('modal-root');
```

**해결책:**
```tsx
const [container, setContainer] = useState<Element | null>(null);

useEffect(() => {
  setContainer(document.getElementById('modal-root'));
}, []);

if (!container) return null;
```

### 2. 스크롤 잠금 해제 누락

**문제:**
```tsx
useEffect(() => {
  if (isOpen) {
    document.body.style.overflow = 'hidden';
  }
  // cleanup이 없어서 모달 닫아도 스크롤 잠김
}, [isOpen]);
```

**해결책:**
```tsx
useEffect(() => {
  if (isOpen) {
    document.body.style.overflow = 'hidden';
  }

  return () => {
    document.body.style.overflow = '';
  };
}, [isOpen]);
```

### 3. 포커스 관리 누락

**문제:** 모달 열릴 때 포커스가 모달로 이동하지 않음

**해결책:**
```tsx
const modalRef = useRef<HTMLDivElement>(null);
const previousFocusRef = useRef<HTMLElement | null>(null);

useEffect(() => {
  if (isOpen) {
    previousFocusRef.current = document.activeElement as HTMLElement;
    modalRef.current?.focus();
  }

  return () => {
    previousFocusRef.current?.focus();
  };
}, [isOpen]);
```

### 4. 이벤트 전파 문제

**문제:** 모달 내용 클릭해도 모달이 닫힘

**해결책:**
```tsx
<div className="overlay" onClick={onClose}>
  <div
    className="content"
    onClick={(e) => e.stopPropagation()}
  >
    {children}
  </div>
</div>
```

### 5. Portal 컨테이너 없음

**문제:** Portal 대상 DOM 노드가 없어서 에러

**해결책:**
```tsx
useEffect(() => {
  let container = document.getElementById('portal-root');

  if (!container) {
    container = document.createElement('div');
    container.id = 'portal-root';
    document.body.appendChild(container);
  }

  setPortalContainer(container);
}, []);
```

### 6. 애니메이션 중 언마운트

**문제:** exit 애니메이션 없이 바로 사라짐

**해결책:** `AnimatePresence` 사용 또는 상태로 관리
```tsx
const [shouldRender, setShouldRender] = useState(false);
const [animationState, setAnimationState] = useState<'entering' | 'entered' | 'exiting'>('entering');

useEffect(() => {
  if (isOpen) {
    setShouldRender(true);
    setAnimationState('entering');
    requestAnimationFrame(() => setAnimationState('entered'));
  } else if (shouldRender) {
    setAnimationState('exiting');
  }
}, [isOpen]);

const handleTransitionEnd = () => {
  if (animationState === 'exiting') {
    setShouldRender(false);
  }
};
```

---

## 베스트 프랙티스

### 1. 재사용 가능한 Portal 컴포넌트

```tsx
import { useState, useEffect, ReactNode } from 'react';
import { createPortal } from 'react-dom';

interface PortalProps {
  children: ReactNode;
  containerId?: string;
}

function Portal({ children, containerId = 'portal-root' }: PortalProps) {
  const [mounted, setMounted] = useState(false);

  useEffect(() => {
    setMounted(true);

    let container = document.getElementById(containerId);
    if (!container) {
      container = document.createElement('div');
      container.id = containerId;
      document.body.appendChild(container);
    }

    return () => {
      // 선택적: 빈 컨테이너 정리
    };
  }, [containerId]);

  if (!mounted) return null;

  const container = document.getElementById(containerId);
  if (!container) return null;

  return createPortal(children, container);
}
```

### 2. 접근성 우선 설계

```tsx
interface AccessibleOverlayProps {
  isOpen: boolean;
  onClose: () => void;
  children: ReactNode;
  ariaLabel: string;
}

function AccessibleOverlay({
  isOpen,
  onClose,
  children,
  ariaLabel,
}: AccessibleOverlayProps) {
  // 1. 포커스 트랩
  // 2. ESC 키로 닫기
  // 3. aria-* 속성
  // 4. role 속성
  // 5. 배경 스크롤 방지
}
```

### 3. 계층적 z-index 관리

```css
:root {
  --z-dropdown: 1000;
  --z-sticky: 1100;
  --z-modal-backdrop: 1200;
  --z-modal: 1300;
  --z-popover: 1400;
  --z-tooltip: 1500;
  --z-toast: 1600;
}

.dropdown-menu {
  z-index: var(--z-dropdown);
}

.modal-overlay {
  z-index: var(--z-modal-backdrop);
}

.modal-content {
  z-index: var(--z-modal);
}

.tooltip {
  z-index: var(--z-tooltip);
}
```

### 4. 성능 최적화

```tsx
// 불필요한 리렌더링 방지
const MemoizedModalContent = memo(function ModalContent({ children }) {
  return <div className="modal-content">{children}</div>;
});

// Portal 컨테이너 캐싱
const portalContainerRef = useRef<HTMLElement | null>(null);

useEffect(() => {
  if (!portalContainerRef.current) {
    portalContainerRef.current = document.getElementById('portal-root');
  }
}, []);
```

---

## 마무리

React Portal은 DOM 계층 구조의 제약을 벗어나 UI를 렌더링해야 할 때 필수적인 도구입니다.

### 핵심 포인트 정리

| 개념 | 설명 |
|------|------|
| createPortal | DOM 트리 외부에 컴포넌트 렌더링 |
| 이벤트 버블링 | React 컴포넌트 트리 따라 전파 |
| Context | Portal에서도 정상 상속 |
| SSR | 클라이언트에서만 렌더링 필요 |
| 접근성 | 포커스 트랩, 키보드 지원 필수 |

### Portal 사용 시 체크리스트

1. **SSR 호환성**: `useEffect`로 클라이언트 렌더링 보장
2. **접근성**: 포커스 관리, 키보드 내비게이션, ARIA 속성
3. **스크롤 잠금**: 모달 열릴 때 배경 스크롤 방지
4. **이벤트 처리**: 필요시 `stopPropagation()` 사용
5. **애니메이션**: exit 애니메이션 고려
6. **정리**: 언마운트 시 부작용 정리

Portal을 활용하면 z-index, overflow 문제 없이 깔끔한 오버레이 UI를 구현할 수 있습니다. 접근성과 사용자 경험을 고려한 모달, 툴팁, 드롭다운으로 더 나은 애플리케이션을 만들어보세요.

---

## 참고 자료

- [React 공식 문서 - createPortal](https://react.dev/reference/react-dom/createPortal)
- [MDN - Stacking context](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_positioned_layout/Understanding_z-index/Stacking_context)
- [WAI-ARIA Authoring Practices - Dialog](https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/)
- [Framer Motion 애니메이션 가이드](/posts/framer-motion-animation-guide/)
- [React Suspense 완벽 가이드](/posts/react-suspense-complete-guide/)
- [React Error Boundary 완벽 가이드](/posts/react-error-boundary-complete-guide/)
- [웹 접근성 완벽 가이드](/posts/web-accessibility-guide/)
