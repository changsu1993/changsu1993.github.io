---
title: "JavaScript MutationObserver API 완벽 가이드 - DOM 변화 감지와 동적 컨텐츠 처리"
description: "MutationObserver API로 DOM 변화를 효율적으로 감지하고 반응하는 방법을 알아봅니다. 속성, 자식 노드, 텍스트 변화 관찰부터 React 커스텀 훅 구현, 성능 최적화까지 실전 활용 패턴을 코드 예제와 함께 상세히 설명합니다."
date: 2026-01-03 10:00:00 +0900
categories: [Frontend, JavaScript]
tags: [JavaScript, MutationObserver, DOM, Web API, Observer Pattern, 자바스크립트, React, 동적요소감지, 비동기]
---

## 개요

웹 애플리케이션이 복잡해지면서 DOM이 동적으로 변하는 상황이 빈번해졌습니다. 써드파티 스크립트가 요소를 추가하거나, SPA에서 컴포넌트가 동적으로 렌더링되거나, 사용자 인터랙션에 의해 DOM 구조가 변경되는 경우가 많습니다.

**MutationObserver API**는 DOM 트리의 변화를 비동기적으로 감지하는 브라우저 내장 API입니다. 요소의 추가/삭제, 속성 변경, 텍스트 변경 등 다양한 DOM 변화를 효율적으로 관찰할 수 있습니다.

> **참고**: 요소가 화면에 보이는지(뷰포트 진입/이탈)를 감지하려면 [Intersection Observer API](/posts/javascript-intersection-observer-guide/)를 사용하세요. MutationObserver는 DOM 구조 자체의 변화를 감지하는 용도입니다.
{: .prompt-info }

### 학습 목표

- MutationObserver의 동작 원리와 Mutation Events와의 차이 이해
- observe(), disconnect(), takeRecords() 메서드 활용법 습득
- MutationObserverInit 옵션과 MutationRecord 구조 이해
- 동적 요소 감지, 폼 모니터링 등 실전 패턴 구현
- React에서 MutationObserver 활용하기

### 사전 지식

- JavaScript 기초 문법
- DOM 조작 기초 (querySelector, appendChild 등)
- ES6+ 문법 (화살표 함수, 구조 분해 할당)

---

## Mutation Events에서 MutationObserver로

MutationObserver가 등장하기 전에는 **Mutation Events**를 사용했습니다. DOM Level 2에서 정의된 이벤트들로, `DOMNodeInserted`, `DOMNodeRemoved`, `DOMAttrModified` 등이 있었습니다.

### Mutation Events의 문제점

```javascript
// 과거 방식 - 더 이상 사용하지 마세요!
document.addEventListener('DOMNodeInserted', function(event) {
  console.log('노드가 추가됨:', event.target);
});

document.addEventListener('DOMAttrModified', function(event) {
  console.log('속성이 변경됨:', event.attrName);
});
```

**왜 deprecated 되었을까요?**

| 문제점 | 설명 |
|--------|------|
| **동기적 실행** | DOM 변경마다 즉시 이벤트가 발생하여 성능 저하 |
| **이벤트 버블링** | 모든 변경이 버블링되어 불필요한 핸들러 호출 |
| **과도한 호출** | 단일 작업에서 여러 이벤트가 연속 발생 |
| **브라우저 최적화 방해** | DOM 변경을 배치 처리하기 어려움 |

예를 들어, 1000개의 리스트 아이템을 추가하면 `DOMNodeInserted` 이벤트가 1000번 발생했습니다. 각 이벤트마다 핸들러가 동기적으로 실행되어 심각한 성능 문제를 야기했습니다.

### MutationObserver의 해결책

MutationObserver는 이러한 문제들을 해결하기 위해 설계되었습니다:

| 특징 | 설명 |
|------|------|
| **비동기 처리** | 변경사항을 모아서 [마이크로태스크](/posts/javascript-event-loop/)로 일괄 전달 |
| **배치 처리** | 여러 변경을 하나의 콜백으로 처리 |
| **세밀한 제어** | 관찰할 변경 유형을 선택적으로 지정 |
| **성능 최적화** | 브라우저의 렌더링 최적화와 조화 |

---

## 기본 사용법

### MutationObserver 생성과 관찰 시작

```javascript
// 1. 콜백 함수 정의
function mutationCallback(mutationsList, observer) {
  for (const mutation of mutationsList) {
    console.log('변경 유형:', mutation.type);
    console.log('변경된 타겟:', mutation.target);
  }
}

// 2. MutationObserver 인스턴스 생성
const observer = new MutationObserver(mutationCallback);

// 3. 관찰 대상과 옵션 설정
const targetNode = document.getElementById('container');
const config = {
  childList: true,    // 자식 노드 추가/삭제 감지
  attributes: true,   // 속성 변경 감지
  subtree: true       // 모든 하위 요소도 관찰
};

// 4. 관찰 시작
observer.observe(targetNode, config);
```

### 관찰 중단하기

```javascript
// 특정 시점에 관찰 중단
observer.disconnect();

// 다시 관찰을 시작할 수도 있음
observer.observe(targetNode, config);
```

### takeRecords()로 대기 중인 변경 가져오기

```javascript
// 콜백이 호출되기 전에 대기 중인 변경 목록을 가져옴
const pendingMutations = observer.takeRecords();

// 가져온 후에는 콜백에서 중복 처리되지 않음
pendingMutations.forEach(mutation => {
  console.log('수동으로 처리:', mutation.type);
});

// disconnect 전에 남은 변경사항 처리하기
function cleanupObserver() {
  const remaining = observer.takeRecords();
  if (remaining.length > 0) {
    mutationCallback(remaining, observer);
  }
  observer.disconnect();
}
```

---

## MutationObserverInit 옵션 상세

`observe()` 메서드의 두 번째 인자로 전달하는 옵션 객체입니다. 어떤 유형의 변경을 관찰할지 지정합니다.

### 필수 옵션 (최소 하나는 true여야 함)

| 옵션 | 타입 | 설명 |
|------|------|------|
| `childList` | boolean | 자식 노드의 추가/삭제 관찰 |
| `attributes` | boolean | 속성 변경 관찰 |
| `characterData` | boolean | 텍스트 노드의 내용 변경 관찰 |

### 부가 옵션

| 옵션 | 타입 | 설명 |
|------|------|------|
| `subtree` | boolean | 모든 하위 요소까지 관찰 범위 확장 |
| `attributeOldValue` | boolean | 변경 전 속성값 기록 |
| `characterDataOldValue` | boolean | 변경 전 텍스트 내용 기록 |
| `attributeFilter` | string[] | 관찰할 속성 이름 목록 (화이트리스트) |

### 옵션 조합 예제

```javascript
// 1. 자식 요소 추가/삭제만 관찰
const childOnlyConfig = {
  childList: true
};

// 2. 특정 속성만 관찰 (class와 data-status만)
const specificAttrsConfig = {
  attributes: true,
  attributeFilter: ['class', 'data-status'],
  attributeOldValue: true  // 변경 전 값도 기록
};

// 3. 텍스트 변경 관찰 (예: contenteditable 요소)
const textConfig = {
  characterData: true,
  characterDataOldValue: true,
  subtree: true  // 내부 텍스트 노드도 관찰
};

// 4. 모든 변경 관찰 (주의: 성능 영향)
const allChangesConfig = {
  childList: true,
  attributes: true,
  characterData: true,
  subtree: true,
  attributeOldValue: true,
  characterDataOldValue: true
};
```

### attributeFilter 활용

특정 속성만 관찰하면 불필요한 콜백 호출을 줄일 수 있습니다:

```javascript
// 테마 관련 속성만 관찰
const themeObserver = new MutationObserver(mutations => {
  mutations.forEach(mutation => {
    if (mutation.attributeName === 'data-theme') {
      const newTheme = mutation.target.getAttribute('data-theme');
      console.log('테마 변경됨:', newTheme);
      applyThemeStyles(newTheme);
    }
  });
});

themeObserver.observe(document.documentElement, {
  attributes: true,
  attributeFilter: ['data-theme', 'data-color-scheme']
});
```

---

## MutationRecord 이해하기

콜백 함수로 전달되는 각 변경사항은 `MutationRecord` 객체입니다. 변경의 종류와 상세 정보를 담고 있습니다.

### MutationRecord 프로퍼티

| 프로퍼티 | 설명 |
|----------|------|
| `type` | 변경 유형: 'childList', 'attributes', 'characterData' |
| `target` | 변경이 발생한 노드 |
| `addedNodes` | 추가된 노드들 (NodeList) |
| `removedNodes` | 삭제된 노드들 (NodeList) |
| `previousSibling` | 추가/삭제된 노드의 이전 형제 |
| `nextSibling` | 추가/삭제된 노드의 다음 형제 |
| `attributeName` | 변경된 속성 이름 |
| `attributeNamespace` | 변경된 속성의 네임스페이스 |
| `oldValue` | 변경 전 값 (옵션 활성화 시) |

### 변경 유형별 처리

```javascript
const observer = new MutationObserver(mutations => {
  mutations.forEach(mutation => {
    switch (mutation.type) {
      case 'childList':
        handleChildListChange(mutation);
        break;
      case 'attributes':
        handleAttributeChange(mutation);
        break;
      case 'characterData':
        handleTextChange(mutation);
        break;
    }
  });
});

function handleChildListChange(mutation) {
  // 추가된 노드 처리
  mutation.addedNodes.forEach(node => {
    if (node.nodeType === Node.ELEMENT_NODE) {
      console.log('요소 추가됨:', node.tagName, node.id || '(no id)');

      // 특정 클래스를 가진 요소가 추가되었는지 확인
      if (node.classList?.contains('lazy-image')) {
        initializeLazyImage(node);
      }
    }
  });

  // 삭제된 노드 처리
  mutation.removedNodes.forEach(node => {
    if (node.nodeType === Node.ELEMENT_NODE) {
      console.log('요소 삭제됨:', node.tagName);
      cleanupRemovedElement(node);
    }
  });
}

function handleAttributeChange(mutation) {
  const { target, attributeName, oldValue } = mutation;
  const newValue = target.getAttribute(attributeName);

  console.log(`속성 변경: ${attributeName}`);
  console.log(`  이전 값: ${oldValue}`);
  console.log(`  새 값: ${newValue}`);
}

function handleTextChange(mutation) {
  const { target, oldValue } = mutation;
  const newValue = target.textContent;

  console.log('텍스트 변경:');
  console.log(`  이전: ${oldValue}`);
  console.log(`  현재: ${newValue}`);
}
```

### NodeList 순회 시 주의사항

`addedNodes`와 `removedNodes`는 **NodeList**입니다. 텍스트 노드나 주석 노드도 포함될 수 있으므로 필터링이 필요합니다:

```javascript
mutation.addedNodes.forEach(node => {
  // 요소 노드만 처리 (텍스트, 주석 노드 제외)
  if (node.nodeType !== Node.ELEMENT_NODE) return;

  // 또는 특정 태그만 처리
  if (node.tagName === 'DIV') {
    processNewDiv(node);
  }
});
```

---

## 실전 활용 패턴

### 1. 동적으로 추가되는 요소 감지

써드파티 스크립트나 비동기 로딩으로 추가되는 요소를 감지하고 처리합니다:

```javascript
// 동적으로 추가되는 광고 요소 감지
function observeAdElements() {
  const observer = new MutationObserver(mutations => {
    mutations.forEach(mutation => {
      mutation.addedNodes.forEach(node => {
        if (node.nodeType !== Node.ELEMENT_NODE) return;

        // 광고 컨테이너가 추가되었는지 확인
        if (node.matches?.('.ad-container, [data-ad-slot]')) {
          console.log('광고 요소 감지:', node);
          trackAdImpression(node);
        }

        // 추가된 요소 내부에도 광고가 있는지 확인
        const nestedAds = node.querySelectorAll?.('.ad-container, [data-ad-slot]');
        nestedAds?.forEach(ad => trackAdImpression(ad));
      });
    });
  });

  observer.observe(document.body, {
    childList: true,
    subtree: true
  });

  return observer;
}
```

### 2. 특정 요소가 나타날 때까지 대기

```javascript
// Promise 기반으로 특정 요소가 DOM에 추가될 때까지 대기
function waitForElement(selector, timeout = 10000) {
  return new Promise((resolve, reject) => {
    // 이미 존재하는지 확인
    const existing = document.querySelector(selector);
    if (existing) {
      resolve(existing);
      return;
    }

    const observer = new MutationObserver(mutations => {
      const element = document.querySelector(selector);
      if (element) {
        observer.disconnect();
        resolve(element);
      }
    });

    observer.observe(document.body, {
      childList: true,
      subtree: true
    });

    // 타임아웃 설정
    setTimeout(() => {
      observer.disconnect();
      reject(new Error(`Element ${selector} not found within ${timeout}ms`));
    }, timeout);
  });
}

// 사용 예시
async function initializeWidget() {
  try {
    const widget = await waitForElement('#third-party-widget');
    console.log('위젯이 로드됨:', widget);
    customizeWidget(widget);
  } catch (error) {
    console.error('위젯 로드 실패:', error.message);
  }
}
```

### 3. 폼 필드 변화 감지

```javascript
// 폼 필드의 disabled, required 등 상태 변화 감지
function observeFormState(formElement) {
  const observer = new MutationObserver(mutations => {
    mutations.forEach(mutation => {
      const { target, attributeName, oldValue } = mutation;
      const newValue = target.getAttribute(attributeName);

      if (attributeName === 'disabled') {
        console.log(`${target.name} 필드 ${newValue !== null ? '비활성화' : '활성화'}됨`);
        updateFormValidation();
      }

      if (attributeName === 'required') {
        console.log(`${target.name} 필드 필수 여부 변경`);
        updateRequiredIndicators();
      }
    });
  });

  // 폼 내 모든 입력 요소 관찰
  const inputs = formElement.querySelectorAll('input, select, textarea');
  inputs.forEach(input => {
    observer.observe(input, {
      attributes: true,
      attributeFilter: ['disabled', 'required', 'readonly'],
      attributeOldValue: true
    });
  });

  return observer;
}
```

### 4. contenteditable 요소의 변경 감지

> 디바운스 패턴에 대한 자세한 내용은 [Debounce & Throttle 가이드](/posts/javascript-debounce-throttle-guide/)를 참고하세요.
{: .prompt-tip }

```javascript
// 리치 텍스트 에디터의 내용 변화 감지
function observeEditor(editorElement) {
  let debounceTimer;

  const observer = new MutationObserver(mutations => {
    // 디바운스로 연속적인 변경을 하나로 묶음
    clearTimeout(debounceTimer);
    debounceTimer = setTimeout(() => {
      const content = editorElement.innerHTML;
      console.log('에디터 내용 변경됨');
      autoSave(content);
    }, 500);
  });

  observer.observe(editorElement, {
    childList: true,
    characterData: true,
    subtree: true,
    characterDataOldValue: true
  });

  return observer;
}

// 사용 예시
const editor = document.querySelector('[contenteditable="true"]');
const editorObserver = observeEditor(editor);
```

### 5. A/B 테스트 도구 연동

```javascript
// A/B 테스트 도구가 DOM을 수정할 때 감지
function trackABTestChanges() {
  const originalContent = new Map();

  const observer = new MutationObserver(mutations => {
    mutations.forEach(mutation => {
      if (mutation.type === 'childList' || mutation.type === 'characterData') {
        const target = mutation.target;
        const testId = target.closest?.('[data-ab-test]')?.dataset.abTest;

        if (testId) {
          console.log(`A/B 테스트 ${testId} 변형 적용됨`);

          // 분석 이벤트 전송
          analytics.track('ab_test_variation_applied', {
            testId,
            timestamp: Date.now()
          });
        }
      }
    });
  });

  observer.observe(document.body, {
    childList: true,
    subtree: true,
    characterData: true
  });

  return observer;
}
```

---

## React에서 MutationObserver 사용하기

> React 커스텀 훅 패턴에 대한 자세한 내용은 [React Custom Hooks 패턴 가이드](/posts/react-custom-hooks-patterns-guide/)를 참고하세요.
{: .prompt-tip }

### 주의사항

React는 Virtual DOM을 통해 DOM을 관리합니다. 대부분의 경우 React의 상태 관리와 생명주기를 활용하면 MutationObserver가 필요 없습니다. 하지만 다음 상황에서는 유용합니다:

- 써드파티 라이브러리가 DOM을 직접 조작할 때
- iframe 내부의 변화를 감지할 때
- React 외부에서 주입되는 콘텐츠를 처리할 때

### useMutationObserver 커스텀 훅

```tsx
import { useEffect, useRef, useCallback } from 'react';

interface UseMutationObserverOptions extends MutationObserverInit {
  // 관찰을 일시적으로 비활성화
  enabled?: boolean;
}

function useMutationObserver(
  callback: MutationCallback,
  options: UseMutationObserverOptions = {}
) {
  const { enabled = true, ...observerOptions } = options;
  const targetRef = useRef<HTMLElement>(null);
  const observerRef = useRef<MutationObserver | null>(null);

  // 콜백을 ref로 관리하여 최신 상태 유지
  const callbackRef = useRef(callback);
  callbackRef.current = callback;

  useEffect(() => {
    if (!enabled || !targetRef.current) return;

    // Observer 생성
    observerRef.current = new MutationObserver((mutations, observer) => {
      callbackRef.current(mutations, observer);
    });

    // 관찰 시작
    observerRef.current.observe(targetRef.current, {
      childList: true,
      ...observerOptions
    });

    // 정리 함수
    return () => {
      observerRef.current?.disconnect();
    };
  }, [enabled, JSON.stringify(observerOptions)]);

  // 수동으로 disconnect 하는 함수
  const disconnect = useCallback(() => {
    observerRef.current?.disconnect();
  }, []);

  // 대기 중인 변경사항 가져오기
  const takeRecords = useCallback(() => {
    return observerRef.current?.takeRecords() ?? [];
  }, []);

  return { targetRef, disconnect, takeRecords };
}

export default useMutationObserver;
```

### 사용 예시

```tsx
import React, { useState } from 'react';
import useMutationObserver from './useMutationObserver';

function ThirdPartyWidgetContainer() {
  const [widgetLoaded, setWidgetLoaded] = useState(false);

  const { targetRef } = useMutationObserver(
    (mutations) => {
      mutations.forEach(mutation => {
        mutation.addedNodes.forEach(node => {
          if (node instanceof HTMLElement && node.classList.contains('widget-content')) {
            console.log('써드파티 위젯 로드됨');
            setWidgetLoaded(true);
          }
        });
      });
    },
    {
      childList: true,
      subtree: true
    }
  );

  return (
    <div ref={targetRef} id="widget-container">
      {/* 써드파티 스크립트가 여기에 콘텐츠를 추가함 */}
      {widgetLoaded && <div className="overlay">위젯 준비 완료!</div>}
    </div>
  );
}
```

### cleanup 처리의 중요성

React 컴포넌트에서 MutationObserver를 사용할 때는 반드시 cleanup을 해야 합니다:

```tsx
useEffect(() => {
  const observer = new MutationObserver(callback);
  observer.observe(element, config);

  // 컴포넌트 언마운트 시 반드시 disconnect!
  return () => {
    observer.disconnect();
  };
}, []);
```

cleanup을 하지 않으면:
- 메모리 누수 발생
- 언마운트된 컴포넌트의 상태 업데이트 시도로 경고 발생
- 예상치 못한 동작 발생 가능

---

## 성능 최적화

### 1. 필요한 옵션만 사용하기

```javascript
// 나쁜 예: 모든 것을 관찰
const badConfig = {
  childList: true,
  attributes: true,
  characterData: true,
  subtree: true,
  attributeOldValue: true,
  characterDataOldValue: true
};

// 좋은 예: 필요한 것만 관찰
const goodConfig = {
  attributes: true,
  attributeFilter: ['class']  // class 속성만 관찰
};
```

### 2. subtree 사용 시 주의

`subtree: true`는 모든 하위 요소를 관찰하므로 대규모 DOM에서는 성능에 영향을 줄 수 있습니다:

```javascript
// 주의: body 전체를 subtree로 관찰하면 성능 저하 가능
observer.observe(document.body, { childList: true, subtree: true });

// 개선: 가능하면 더 구체적인 컨테이너를 대상으로 지정
const container = document.getElementById('dynamic-content');
observer.observe(container, { childList: true, subtree: true });
```

### 3. 콜백 내 DOM 조작 최소화

```javascript
// 나쁜 예: 콜백에서 직접 많은 DOM 조작
const badObserver = new MutationObserver(mutations => {
  mutations.forEach(mutation => {
    mutation.addedNodes.forEach(node => {
      // 이 조작이 또 다른 mutation을 발생시킬 수 있음!
      node.classList.add('processed');
      node.innerHTML = '<span>Processed!</span>';
    });
  });
});

// 좋은 예: requestAnimationFrame으로 배치 처리
const goodObserver = new MutationObserver(mutations => {
  const nodesToProcess = [];

  mutations.forEach(mutation => {
    mutation.addedNodes.forEach(node => {
      if (node.nodeType === Node.ELEMENT_NODE) {
        nodesToProcess.push(node);
      }
    });
  });

  if (nodesToProcess.length > 0) {
    requestAnimationFrame(() => {
      nodesToProcess.forEach(node => {
        node.classList.add('processed');
      });
    });
  }
});
```

### 4. 무한 루프 방지

콜백에서 관찰 대상을 수정하면 무한 루프에 빠질 수 있습니다:

```javascript
// 위험: 무한 루프 가능성
const dangerousObserver = new MutationObserver(mutations => {
  mutations.forEach(mutation => {
    // 이 변경이 또 다른 mutation을 발생시킴!
    mutation.target.setAttribute('data-count',
      parseInt(mutation.target.dataset.count || 0) + 1
    );
  });
});

// 안전: 조건부 처리로 무한 루프 방지
const safeObserver = new MutationObserver(mutations => {
  mutations.forEach(mutation => {
    // 이미 처리된 요소는 건너뜀
    if (mutation.target.dataset.processed === 'true') return;

    mutation.target.dataset.processed = 'true';
    // 추가 처리...
  });
});
```

또는 일시적으로 관찰을 중단하는 방법:

```javascript
const observer = new MutationObserver(mutations => {
  // 관찰 일시 중단
  observer.disconnect();

  mutations.forEach(mutation => {
    // DOM 조작 수행
    mutation.target.classList.add('modified');
  });

  // 관찰 재개
  observer.observe(targetNode, config);
});
```

---

## Intersection Observer vs MutationObserver

두 Observer API는 이름이 비슷하지만 완전히 다른 목적을 가집니다:

| 특성 | MutationObserver | Intersection Observer |
|------|------------------|----------------------|
| **감지 대상** | DOM 구조/속성/텍스트 변화 | 요소의 가시성 (뷰포트 교차) |
| **주요 용도** | 동적 요소 감지, 속성 모니터링 | 지연 로딩, 무한 스크롤 |
| **트리거 조건** | DOM 변경 시 | 스크롤, 리사이즈 시 |
| **콜백 데이터** | MutationRecord 배열 | IntersectionObserverEntry 배열 |

### 함께 사용하는 패턴

```javascript
// 동적으로 추가되는 이미지에 지연 로딩 적용
function setupLazyLoadForDynamicImages() {
  // Intersection Observer: 이미지 지연 로딩
  const lazyLoadObserver = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        const img = entry.target;
        img.src = img.dataset.src;
        lazyLoadObserver.unobserve(img);
      }
    });
  });

  // 기존 이미지에 적용
  document.querySelectorAll('img[data-src]').forEach(img => {
    lazyLoadObserver.observe(img);
  });

  // MutationObserver: 새로 추가되는 이미지 감지
  const mutationObserver = new MutationObserver(mutations => {
    mutations.forEach(mutation => {
      mutation.addedNodes.forEach(node => {
        if (node.nodeType !== Node.ELEMENT_NODE) return;

        // 추가된 요소가 lazy 이미지인 경우
        if (node.matches?.('img[data-src]')) {
          lazyLoadObserver.observe(node);
        }

        // 추가된 요소 내부의 lazy 이미지도 처리
        node.querySelectorAll?.('img[data-src]').forEach(img => {
          lazyLoadObserver.observe(img);
        });
      });
    });
  });

  mutationObserver.observe(document.body, {
    childList: true,
    subtree: true
  });

  return { lazyLoadObserver, mutationObserver };
}
```

---

## 주의사항과 안티패턴

### 1. 메모리 누수 방지

> JavaScript 메모리 관리와 가비지 컬렉션에 대한 자세한 내용은 [메모리 관리 가이드](/posts/javascript-memory-management-garbage-collection-guide/)를 참고하세요.
{: .prompt-info }

```javascript
// 나쁜 예: disconnect 호출 없음
function badExample() {
  const observer = new MutationObserver(callback);
  observer.observe(element, config);
  // observer가 계속 동작 중...
}

// 좋은 예: 정리 함수 반환
function goodExample() {
  const observer = new MutationObserver(callback);
  observer.observe(element, config);

  return function cleanup() {
    observer.disconnect();
  };
}

const cleanup = goodExample();
// 나중에...
cleanup();
```

### 2. 동기적 DOM 접근 주의

MutationObserver 콜백은 마이크로태스크로 실행되므로, 콜백 시점에 DOM 상태가 이미 변경되었을 수 있습니다:

```javascript
const observer = new MutationObserver(mutations => {
  mutations.forEach(mutation => {
    // 주의: 이 시점에 removedNodes의 요소들은
    // 이미 DOM에서 제거된 상태
    mutation.removedNodes.forEach(node => {
      // node.parentNode는 null
      console.log(node.parentNode); // null

      // 하지만 노드 자체의 정보는 접근 가능
      console.log(node.id, node.className);
    });
  });
});
```

### 3. Shadow DOM 관찰

Shadow DOM 내부를 관찰하려면 Shadow Root에 직접 observe해야 합니다:

```javascript
// Shadow DOM 외부에서는 내부 변화를 감지할 수 없음
const hostElement = document.querySelector('my-component');

// 이렇게 하면 Shadow DOM 내부는 관찰되지 않음
observer.observe(hostElement, { childList: true, subtree: true });

// Shadow Root에 직접 observe해야 함
if (hostElement.shadowRoot) {
  observer.observe(hostElement.shadowRoot, {
    childList: true,
    subtree: true
  });
}
```

---

## 브라우저 호환성

MutationObserver는 모든 모던 브라우저에서 지원됩니다:

| 브라우저 | 지원 버전 |
|----------|-----------|
| Chrome | 26+ |
| Firefox | 14+ |
| Safari | 7+ |
| Edge | 12+ |
| IE | 11 |

> IE 11을 지원해야 하는 경우에도 MutationObserver를 사용할 수 있습니다. 다만 IE에서는 polyfill 없이도 동작하지만, 약간의 성능 차이가 있을 수 있습니다.
{: .prompt-tip }

---

## 정리

MutationObserver는 DOM 변화를 효율적으로 감지하는 강력한 API입니다:

| 핵심 포인트 | 설명 |
|-------------|------|
| **비동기 처리** | 마이크로태스크로 배치 처리되어 성능 우수 |
| **세밀한 제어** | childList, attributes, characterData 등 선택적 관찰 |
| **실전 활용** | 동적 요소 감지, 써드파티 통합, 폼 모니터링 |
| **주의사항** | disconnect 필수, 무한 루프 방지, 성능 고려 |

### 언제 사용해야 할까?

- 써드파티 스크립트가 DOM을 수정하는 것을 감지해야 할 때
- 동적으로 추가되는 요소에 자동으로 기능을 적용해야 할 때
- DOM 속성 변화에 반응해야 할 때
- contenteditable이나 designMode 요소의 변경을 추적할 때

### 언제 사용하지 말아야 할까?

- React/Vue 등 프레임워크 내에서 상태 변화를 추적할 때 (상태 관리 사용)
- 요소의 가시성을 감지할 때 (Intersection Observer 사용)
- 요소의 크기 변화를 감지할 때 (ResizeObserver 사용)
- 단순한 이벤트 처리 (이벤트 리스너 사용)

---

## 참고 자료

- [MDN - MutationObserver](https://developer.mozilla.org/ko/docs/Web/API/MutationObserver)
- [MDN - MutationRecord](https://developer.mozilla.org/ko/docs/Web/API/MutationRecord)
- [DOM Standard - Mutation observers](https://dom.spec.whatwg.org/#mutation-observers)
- [W3C DOM4 - Mutation Observers](https://www.w3.org/TR/dom/#mutation-observers)
