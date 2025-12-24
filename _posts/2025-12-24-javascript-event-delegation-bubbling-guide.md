---
title: "JavaScript 이벤트 위임(Event Delegation) 완벽 가이드 - 버블링, 캡처링, 그리고 성능 최적화"
description: "JavaScript 이벤트 버블링과 캡처링의 동작 원리를 시각적으로 이해하고, 이벤트 위임(Event Delegation) 패턴으로 메모리 효율적인 DOM 이벤트 처리를 구현하는 방법을 코드 예제와 함께 알아봅니다. 프론트엔드 면접 필수 개념."
date: 2025-12-24 09:00:00 +0900
categories: [Frontend, JavaScript]
tags: [JavaScript, Event, Event-Delegation, Event-Bubbling, Event-Capturing, DOM, 이벤트위임, 이벤트버블링, 이벤트캡처링, stopPropagation, addEventListener, 프론트엔드면접, React-SyntheticEvent, closest, event-target]
---

## 개요

**이벤트 위임(Event Delegation)**은 JavaScript에서 이벤트를 효율적으로 처리하기 위한 핵심 패턴입니다. 이 패턴을 이해하려면 먼저 DOM 이벤트가 어떻게 전파되는지 - **버블링(Bubbling)**과 **캡처링(Capturing)** - 알아야 합니다.

이 글에서는 DOM 이벤트 흐름의 동작 원리를 시각적으로 설명하고, 이벤트 위임 패턴을 활용하여 메모리 효율적이고 동적 요소에도 대응 가능한 이벤트 처리 방법을 다룹니다. 프론트엔드 면접에서 자주 나오는 주제이므로 정확하게 이해해두시기 바랍니다.

### 학습 목표

- DOM 이벤트 흐름(캡처링 -> 타겟 -> 버블링) 이해
- 이벤트 버블링과 캡처링의 차이점 파악
- `event.target`과 `event.currentTarget`의 차이 이해
- 이벤트 위임 패턴 구현 방법 습득
- React의 합성 이벤트(SyntheticEvent)와 비교
- 실무 적용 Best Practices 학습

### 사전 지식

- JavaScript 함수 기초
- DOM 조작 기본 개념
- HTML 요소 구조

### 관련 포스트

- [JavaScript 실행 컨텍스트 완벽 가이드](/posts/javascript-execution-context-guide/) - 이벤트 핸들러 내 this 바인딩 이해
- [JavaScript this 바인딩 완벽 가이드](/posts/javascript-this-binding-guide/) - 화살표 함수와 일반 함수의 this 차이
- [JavaScript 클로저 완벽 가이드](/posts/javascript-closure-complete-guide/) - 이벤트 핸들러에서 클로저 활용
- [JavaScript 메모리 관리와 가비지 컬렉션](/posts/javascript-memory-management-garbage-collection-guide/) - 이벤트 위임의 메모리 효율성

---

## DOM 이벤트 흐름 개요

브라우저에서 이벤트가 발생하면, 해당 이벤트는 3가지 단계를 거쳐 전파됩니다:

| 단계 | 이름 | 전파 방향 |
|:---:|------|----------|
| **1** | 캡처링 단계 (Capturing Phase) | `window` → `document` → `html` → `body` → ... → 타겟의 부모 |
| **2** | 타겟 단계 (Target Phase) | 이벤트가 실제로 발생한 요소 |
| **3** | 버블링 단계 (Bubbling Phase) | 타겟의 부모 → ... → `body` → `html` → `document` → `window` |

### 시각적 예시

다음과 같은 HTML 구조가 있다고 가정해봅시다:

```html
<div id="outer">
  <div id="inner">
    <button id="button">클릭</button>
  </div>
</div>
```

버튼을 클릭하면 이벤트는 다음 순서로 전파됩니다:

| 순서 | 단계 | 요소 | 설명 |
|:---:|:---:|------|------|
| ① | 캡처링 | `#outer` | 최상위에서 캡처링 시작 |
| ② | 캡처링 | `#inner` | 중간 요소 캡처링 |
| ③ | 타겟 | `#button` | 타겟 요소의 캡처링 핸들러 실행 |
| ④ | 타겟 | `#button` | 타겟 요소의 버블링 핸들러 실행 |
| ⑤ | 버블링 | `#inner` | 상위로 버블링 시작 |
| ⑥ | 버블링 | `#outer` | 최상위까지 버블링 |

> **전파 순서**: ① → ② → ③④ → ⑤ → ⑥

> 대부분의 이벤트 핸들러는 기본적으로 **버블링 단계**에서 실행됩니다. 캡처링 단계에서 이벤트를 잡으려면 명시적으로 설정해야 합니다.
{: .prompt-info }

---

## 이벤트 버블링(Event Bubbling)

### 버블링이란?

**이벤트 버블링**은 특정 요소에서 이벤트가 발생했을 때, 해당 이벤트가 상위 요소들로 전파되는 현상입니다. 마치 물속에서 거품(bubble)이 위로 올라가는 것처럼, 이벤트가 DOM 트리를 따라 위로 올라갑니다.

```javascript
// HTML 구조
// <div id="grandparent">
//   <div id="parent">
//     <button id="child">클릭</button>
//   </div>
// </div>

const grandparent = document.getElementById('grandparent');
const parent = document.getElementById('parent');
const child = document.getElementById('child');

grandparent.addEventListener('click', () => {
  console.log('grandparent 클릭!');
});

parent.addEventListener('click', () => {
  console.log('parent 클릭!');
});

child.addEventListener('click', () => {
  console.log('child 클릭!');
});

// 버튼(child)을 클릭하면:
// "child 클릭!"
// "parent 클릭!"
// "grandparent 클릭!"
// (버블링으로 인해 상위 요소의 핸들러도 실행됨)
```

### event.stopPropagation()

버블링을 중단하고 싶을 때는 `event.stopPropagation()` 메서드를 사용합니다:

```javascript
child.addEventListener('click', (event) => {
  console.log('child 클릭!');
  event.stopPropagation(); // 버블링 중단
});

// 이제 버튼을 클릭하면:
// "child 클릭!" 만 출력됨
// parent와 grandparent의 핸들러는 실행되지 않음
```

> `stopPropagation()`은 신중하게 사용해야 합니다. 무분별하게 사용하면 상위 요소에서 이벤트를 감지할 수 없어 디버깅이 어려워지고, 분석 도구나 로깅 시스템에 영향을 줄 수 있습니다.
{: .prompt-warning }

### event.stopImmediatePropagation()

같은 요소에 여러 이벤트 핸들러가 등록되어 있을 때, 현재 핸들러 이후의 모든 핸들러 실행을 막으려면 `stopImmediatePropagation()`을 사용합니다:

```javascript
child.addEventListener('click', (event) => {
  console.log('첫 번째 핸들러');
  event.stopImmediatePropagation();
});

child.addEventListener('click', () => {
  console.log('두 번째 핸들러'); // 실행되지 않음
});

// 출력: "첫 번째 핸들러" 만 출력됨
```

### 버블링이 발생하지 않는 이벤트

모든 이벤트가 버블링되는 것은 아닙니다. 다음 이벤트들은 버블링되지 않습니다:

| 이벤트 | 설명 |
|--------|------|
| `focus` / `blur` | 포커스 관련 이벤트 |
| `mouseenter` / `mouseleave` | 마우스 진입/이탈 이벤트 |
| `load` / `unload` | 로드 관련 이벤트 |
| `scroll` | 스크롤 이벤트 (일부 상황) |

> `focus`/`blur` 대신 버블링되는 `focusin`/`focusout`을, `mouseenter`/`mouseleave` 대신 `mouseover`/`mouseout`을 사용하면 이벤트 위임이 가능합니다.
{: .prompt-tip }

---

## 이벤트 캡처링(Event Capturing)

### 캡처링이란?

**이벤트 캡처링**은 버블링의 반대 방향입니다. 이벤트가 최상위 요소(window)에서 시작하여 타겟 요소까지 내려가는 단계입니다. 캡처링은 버블링보다 먼저 발생합니다.

### addEventListener의 세 번째 인자

캡처링 단계에서 이벤트를 처리하려면 `addEventListener`의 세 번째 인자를 설정해야 합니다:

```javascript
// 세 번째 인자: capture 옵션
element.addEventListener('click', handler, true);
// 또는 옵션 객체로 전달
element.addEventListener('click', handler, { capture: true });
```

### 캡처링 예제

```javascript
const grandparent = document.getElementById('grandparent');
const parent = document.getElementById('parent');
const child = document.getElementById('child');

// 캡처링 단계에서 실행
grandparent.addEventListener('click', () => {
  console.log('grandparent 캡처링');
}, true);

parent.addEventListener('click', () => {
  console.log('parent 캡처링');
}, true);

child.addEventListener('click', () => {
  console.log('child 캡처링');
}, true);

// 버블링 단계에서 실행 (기본값)
grandparent.addEventListener('click', () => {
  console.log('grandparent 버블링');
});

parent.addEventListener('click', () => {
  console.log('parent 버블링');
});

child.addEventListener('click', () => {
  console.log('child 버블링');
});

// child 버튼 클릭 시 출력 순서:
// "grandparent 캡처링"
// "parent 캡처링"
// "child 캡처링"
// "child 버블링"
// "parent 버블링"
// "grandparent 버블링"
```

### 캡처링 사용 시나리오

캡처링은 버블링보다 덜 사용되지만, 다음과 같은 상황에서 유용합니다:

**1. 이벤트를 하위 요소보다 먼저 처리해야 할 때**

```javascript
// 모달 외부 클릭 감지 (이벤트가 모달에 도달하기 전에 처리)
document.addEventListener('click', (event) => {
  const modal = document.getElementById('modal');
  if (!modal.contains(event.target)) {
    closeModal();
  }
}, true); // 캡처링 단계에서 처리
```

**2. 이벤트를 가로채서 조건부로 전파를 막을 때**

```javascript
// 특정 조건에서 모든 클릭 이벤트 차단
document.addEventListener('click', (event) => {
  if (isEditMode && !event.target.classList.contains('editable')) {
    event.stopPropagation();
    console.log('편집 모드에서는 editable 요소만 클릭 가능합니다.');
  }
}, true);
```

**3. 포커스 이벤트 위임 구현**

```javascript
// focus는 버블링되지 않으므로 캡처링 사용
form.addEventListener('focus', (event) => {
  event.target.classList.add('focused');
}, true);

form.addEventListener('blur', (event) => {
  event.target.classList.remove('focused');
}, true);
```

---

## event.target vs event.currentTarget

이벤트 위임을 이해하는 데 가장 중요한 개념 중 하나입니다.

### 핵심 차이점

| 속성 | 설명 | 변경 여부 |
|------|------|-----------|
| `event.target` | 이벤트가 **실제로 발생한** 요소 | 이벤트 전파 중 변하지 않음 |
| `event.currentTarget` | 이벤트 핸들러가 **등록된** 요소 | 이벤트 전파 단계에 따라 변함 |

### 시각적 설명

```html
<ul id="list">           <!-- event.currentTarget: 핸들러가 등록된 요소 -->
  <li>
    <button>클릭</button> <!-- event.target: 실제 클릭된 요소 -->
  </li>
</ul>
```

| 속성 | 가리키는 요소 | 설명 |
|------|-------------|------|
| `event.currentTarget` | `<ul id="list">` | 핸들러가 등록된 요소 |
| `event.target` | `<button>` | 실제 클릭이 발생한 요소 |

### 실제 예제

```javascript
const list = document.getElementById('list');

list.addEventListener('click', function(event) {
  console.log('event.target:', event.target);          // 클릭된 실제 요소
  console.log('event.currentTarget:', event.currentTarget); // ul#list
  console.log('this:', this);  // event.currentTarget과 동일

  // event.target과 event.currentTarget이 같은지 확인
  console.log('같은 요소?:', event.target === event.currentTarget);
});

// <li> 클릭 시:
// event.target: <li>...</li>
// event.currentTarget: <ul id="list">...</ul>
// 같은 요소?: false

// <ul> 직접 클릭 시 (li 바깥 영역):
// event.target: <ul id="list">...</ul>
// event.currentTarget: <ul id="list">...</ul>
// 같은 요소?: true
```

> 면접에서 자주 나오는 질문입니다: "`event.target`과 `event.currentTarget`의 차이점을 설명해주세요." 핸들러가 등록된 요소와 실제 이벤트가 발생한 요소의 차이를 명확히 이해해야 합니다.
{: .prompt-warning }

### 화살표 함수에서의 this

주의할 점은, 화살표 함수에서 `this`는 `event.currentTarget`을 가리키지 않습니다:

```javascript
// 일반 함수
list.addEventListener('click', function(event) {
  console.log(this === event.currentTarget); // true
});

// 화살표 함수
list.addEventListener('click', (event) => {
  console.log(this === event.currentTarget); // false (this는 상위 스코프의 this)
});
```

> 화살표 함수와 일반 함수의 this 바인딩 차이에 대해 더 자세히 알아보려면 [JavaScript this 바인딩 완벽 가이드](/posts/javascript-this-binding-guide/)를 참고하세요.
{: .prompt-tip }

---

## 이벤트 위임(Event Delegation) 패턴

### 이벤트 위임이란?

**이벤트 위임**은 여러 요소에 각각 이벤트 핸들러를 등록하는 대신, 공통 상위 요소에 하나의 핸들러만 등록하여 이벤트를 처리하는 패턴입니다. 이벤트 버블링을 활용합니다.

#### 이벤트 위임 vs 개별 등록 비교

| 구분 | 개별 등록 방식 | 이벤트 위임 방식 |
|------|--------------|----------------|
| **핸들러 위치** | 각 `<li>`에 핸들러 등록 | `<ul>`에 핸들러 1개만 등록 |
| **핸들러 수** | N개 | 1개 |
| **메모리 사용** | 많음 | 적음 |
| **동적 요소** | 별도 처리 필요 | 자동 처리 |

### 왜 이벤트 위임을 사용하는가?

**1. 메모리 효율성**

```javascript
// 비효율적: 각 항목마다 핸들러 등록
const items = document.querySelectorAll('.item');
items.forEach(item => {
  item.addEventListener('click', handleClick); // N개의 핸들러
});

// 효율적: 부모에 하나의 핸들러만 등록
const container = document.getElementById('container');
container.addEventListener('click', handleClick); // 1개의 핸들러
```

1000개의 리스트 항목이 있다면, 개별 등록 방식은 1000개의 이벤트 핸들러가 메모리에 존재합니다. 이벤트 위임을 사용하면 단 1개의 핸들러만 필요합니다.

> 이벤트 위임의 메모리 효율성과 관련하여 JavaScript의 메모리 관리 원리를 이해하면 더 효과적입니다. [JavaScript 메모리 관리와 가비지 컬렉션](/posts/javascript-memory-management-garbage-collection-guide/)에서 자세히 알아보세요.
{: .prompt-tip }

**2. 동적으로 추가되는 요소 처리**

```javascript
// 문제: 나중에 추가된 요소에는 핸들러가 없음
document.querySelectorAll('.item').forEach(item => {
  item.addEventListener('click', handleClick);
});

// 나중에 추가되는 요소
const newItem = document.createElement('li');
newItem.className = 'item';
container.appendChild(newItem);
// 클릭해도 handleClick이 실행되지 않음!

// 해결: 이벤트 위임 사용
container.addEventListener('click', (event) => {
  if (event.target.classList.contains('item')) {
    handleClick(event);
  }
});
// 동적으로 추가된 요소도 자동으로 처리됨!
```

**3. 코드 간결성**

핸들러 등록/해제 로직이 단순해지고, 유지보수가 쉬워집니다.

### 기본 구현 방법

```javascript
const todoList = document.getElementById('todo-list');

todoList.addEventListener('click', (event) => {
  const target = event.target;

  // 클릭된 요소가 삭제 버튼인지 확인
  if (target.classList.contains('delete-btn')) {
    const todoItem = target.closest('.todo-item');
    todoItem.remove();
    return;
  }

  // 클릭된 요소가 완료 체크박스인지 확인
  if (target.classList.contains('complete-checkbox')) {
    const todoItem = target.closest('.todo-item');
    todoItem.classList.toggle('completed');
    return;
  }

  // 클릭된 요소가 todo-item 자체인지 확인
  if (target.classList.contains('todo-item')) {
    openEditModal(target);
  }
});
```

### closest() 메서드 활용

`closest()` 메서드는 현재 요소에서 시작하여 조건에 맞는 가장 가까운 조상 요소를 찾습니다. 이벤트 위임에서 매우 유용합니다:

```javascript
// HTML 구조
// <ul id="list">
//   <li class="item" data-id="1">
//     <span class="text">항목 1</span>
//     <button class="delete">삭제</button>
//   </li>
// </ul>

list.addEventListener('click', (event) => {
  // 삭제 버튼이 아니라 span이나 li를 클릭해도
  // 가장 가까운 .item을 찾을 수 있음
  const item = event.target.closest('.item');

  if (!item) return; // .item 외부 클릭 시 무시

  const itemId = item.dataset.id;
  console.log(`항목 ${itemId} 클릭됨`);

  // 삭제 버튼 클릭 확인
  if (event.target.closest('.delete')) {
    deleteItem(itemId);
  }
});
```

### 실전 예제: 동적 테이블

```html
<table id="user-table">
  <thead>
    <tr>
      <th>이름</th>
      <th>이메일</th>
      <th>액션</th>
    </tr>
  </thead>
  <tbody>
    <!-- 동적으로 행이 추가됨 -->
  </tbody>
</table>
```

```javascript
const table = document.getElementById('user-table');
const tbody = table.querySelector('tbody');

// 이벤트 위임으로 모든 행의 이벤트 처리
table.addEventListener('click', (event) => {
  const target = event.target;
  const row = target.closest('tr');

  if (!row || row.parentElement.tagName === 'THEAD') return;

  const userId = row.dataset.userId;

  // 편집 버튼
  if (target.closest('.edit-btn')) {
    editUser(userId);
    return;
  }

  // 삭제 버튼
  if (target.closest('.delete-btn')) {
    if (confirm('정말 삭제하시겠습니까?')) {
      row.remove();
      deleteUserFromServer(userId);
    }
    return;
  }

  // 행 클릭 (상세 보기)
  if (target.tagName === 'TD') {
    showUserDetails(userId);
  }
});

// 새 사용자 추가 (이벤트 핸들러 등록 불필요!)
function addUser(user) {
  const row = document.createElement('tr');
  row.dataset.userId = user.id;
  row.innerHTML = `
    <td>${user.name}</td>
    <td>${user.email}</td>
    <td>
      <button class="edit-btn">편집</button>
      <button class="delete-btn">삭제</button>
    </td>
  `;
  tbody.appendChild(row);
}
```

### 실전 예제: 탭 컴포넌트

```html
<div class="tabs" id="tabs">
  <div class="tab-list" role="tablist">
    <button class="tab" data-tab="tab1" aria-selected="true">탭 1</button>
    <button class="tab" data-tab="tab2">탭 2</button>
    <button class="tab" data-tab="tab3">탭 3</button>
  </div>
  <div class="tab-panels">
    <div id="tab1" class="tab-panel active">탭 1 내용</div>
    <div id="tab2" class="tab-panel">탭 2 내용</div>
    <div id="tab3" class="tab-panel">탭 3 내용</div>
  </div>
</div>
```

```javascript
const tabs = document.getElementById('tabs');

tabs.addEventListener('click', (event) => {
  const tab = event.target.closest('.tab');

  if (!tab) return;

  const tabId = tab.dataset.tab;

  // 모든 탭 비활성화
  tabs.querySelectorAll('.tab').forEach(t => {
    t.setAttribute('aria-selected', 'false');
  });

  // 모든 패널 숨김
  tabs.querySelectorAll('.tab-panel').forEach(panel => {
    panel.classList.remove('active');
  });

  // 클릭된 탭 활성화
  tab.setAttribute('aria-selected', 'true');
  document.getElementById(tabId).classList.add('active');
});
```

---

## React의 합성 이벤트(SyntheticEvent)

React는 브라우저의 네이티브 이벤트를 래핑한 **합성 이벤트(SyntheticEvent)** 시스템을 사용합니다.

### React 17 이전 vs 이후

#### React 이벤트 위임 방식 변화

| 구분 | React 16 이하 | React 17 이상 |
|------|--------------|--------------|
| **이벤트 위임 위치** | `document` | `<div id="root">` (루트 컨테이너) |
| **장점** | - | 여러 React 버전 공존 가능, 다른 라이브러리와 호환성 개선 |

```html
<!-- React 16 이하: document에 이벤트 위임 -->
<html> <!-- ← 이벤트 핸들러가 여기에 등록 -->
  <body>
    <div id="root">
      <App />
    </div>
  </body>
</html>

<!-- React 17 이상: root 컨테이너에 이벤트 위임 -->
<html>
  <body>
    <div id="root"> <!-- ← 이벤트 핸들러가 여기에 등록 -->
      <App />
    </div>
  </body>
</html>
```

React 17부터 `document` 대신 React 앱의 루트 컨테이너에 이벤트를 위임합니다. 이로 인해 여러 React 버전의 공존과 다른 라이브러리와의 호환성이 개선되었습니다.

### React에서의 이벤트 처리

```tsx
function TodoList() {
  const [todos, setTodos] = useState([
    { id: 1, text: '할 일 1', completed: false },
    { id: 2, text: '할 일 2', completed: false },
  ]);

  // React에서도 이벤트 위임 패턴 활용 가능
  const handleClick = (event: React.MouseEvent<HTMLUListElement>) => {
    const target = event.target as HTMLElement;
    const todoItem = target.closest('[data-todo-id]') as HTMLElement;

    if (!todoItem) return;

    const todoId = Number(todoItem.dataset.todoId);

    // 삭제 버튼
    if (target.closest('.delete-btn')) {
      setTodos(prev => prev.filter(todo => todo.id !== todoId));
      return;
    }

    // 완료 토글
    if (target.closest('.toggle-btn')) {
      setTodos(prev => prev.map(todo =>
        todo.id === todoId
          ? { ...todo, completed: !todo.completed }
          : todo
      ));
    }
  };

  return (
    <ul onClick={handleClick}>
      {todos.map(todo => (
        <li key={todo.id} data-todo-id={todo.id}>
          <span className={todo.completed ? 'completed' : ''}>
            {todo.text}
          </span>
          <button className="toggle-btn">완료</button>
          <button className="delete-btn">삭제</button>
        </li>
      ))}
    </ul>
  );
}
```

### React에서 네이티브 이벤트 접근

```tsx
function Component() {
  const handleClick = (event: React.MouseEvent) => {
    // React 합성 이벤트
    console.log('합성 이벤트:', event);

    // 네이티브 DOM 이벤트에 접근
    console.log('네이티브 이벤트:', event.nativeEvent);

    // event.target과 event.currentTarget은 동일하게 동작
    console.log('target:', event.target);
    console.log('currentTarget:', event.currentTarget);
  };

  return <button onClick={handleClick}>클릭</button>;
}
```

### React에서 stopPropagation 주의사항

```tsx
function Parent() {
  // document에 네이티브 이벤트 등록
  useEffect(() => {
    const handleDocumentClick = () => {
      console.log('document 클릭');
    };
    document.addEventListener('click', handleDocumentClick);
    return () => document.removeEventListener('click', handleDocumentClick);
  }, []);

  return (
    <div onClick={() => console.log('Parent 클릭')}>
      <Child />
    </div>
  );
}

function Child() {
  const handleClick = (event: React.MouseEvent) => {
    event.stopPropagation();
    // React 17+에서는 document까지 전파되지 않음
    // React 16에서는 document까지 이미 전파된 후였음
    console.log('Child 클릭');
  };

  return <button onClick={handleClick}>클릭</button>;
}
```

### React 권장 패턴

React에서는 일반적으로 개별 핸들러를 사용하는 것이 더 선언적이고 명확합니다:

```tsx
function TodoList() {
  const [todos, setTodos] = useState([...]);

  const handleToggle = (id: number) => {
    setTodos(prev => prev.map(todo =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    ));
  };

  const handleDelete = (id: number) => {
    setTodos(prev => prev.filter(todo => todo.id !== id));
  };

  return (
    <ul>
      {todos.map(todo => (
        <TodoItem
          key={todo.id}
          todo={todo}
          onToggle={() => handleToggle(todo.id)}
          onDelete={() => handleDelete(todo.id)}
        />
      ))}
    </ul>
  );
}

function TodoItem({ todo, onToggle, onDelete }) {
  return (
    <li>
      <span className={todo.completed ? 'completed' : ''}>
        {todo.text}
      </span>
      <button onClick={onToggle}>완료</button>
      <button onClick={onDelete}>삭제</button>
    </li>
  );
}
```

> React에서는 Virtual DOM과 효율적인 렌더링 시스템 덕분에, 개별 핸들러 방식도 성능에 큰 영향을 주지 않습니다. 수백 개 이상의 요소가 있는 경우에만 이벤트 위임을 고려하세요. 더 자세한 React 성능 최적화 기법은 [React Custom Hooks 패턴](/posts/react-custom-hooks-patterns-guide/)과 [React Hooks 성능 최적화](/posts/react-hooks-performance-optimization/)에서 확인할 수 있습니다.
{: .prompt-tip }

---

## 이벤트 위임 주의사항과 한계

### 1. 버블링되지 않는 이벤트

`focus`, `blur`, `mouseenter`, `mouseleave` 등은 버블링되지 않아 이벤트 위임이 직접적으로 작동하지 않습니다:

```javascript
// 작동하지 않음
container.addEventListener('focus', (event) => {
  console.log('포커스됨:', event.target);
});

// 해결 1: 캡처링 단계에서 처리
container.addEventListener('focus', (event) => {
  console.log('포커스됨:', event.target);
}, true); // 캡처링 사용

// 해결 2: 버블링되는 대체 이벤트 사용
container.addEventListener('focusin', (event) => {
  console.log('포커스됨:', event.target);
});
```

### 2. stopPropagation으로 인한 문제

하위 요소에서 `stopPropagation()`을 호출하면 이벤트 위임이 작동하지 않습니다:

```javascript
// 상위 컴포넌트의 이벤트 위임
container.addEventListener('click', (event) => {
  console.log('컨테이너에서 처리'); // 실행되지 않음
});

// 하위 컴포넌트에서 전파 중단
button.addEventListener('click', (event) => {
  event.stopPropagation();
  console.log('버튼에서 처리');
});
```

### 3. 복잡한 타겟 판별

중첩된 요소가 많을 경우 올바른 타겟을 찾기 어려울 수 있습니다:

```javascript
// 복잡한 구조
// <div class="card">
//   <div class="card-header">
//     <img class="avatar" />
//     <span class="name">...</span>
//   </div>
//   <div class="card-body">...</div>
// </div>

container.addEventListener('click', (event) => {
  const target = event.target;

  // 어떤 요소가 클릭되었는지에 따라 다른 처리 필요
  // img, span, div 등 여러 요소가 클릭될 수 있음

  // closest()를 사용하여 해결
  const card = target.closest('.card');
  if (!card) return;

  const header = target.closest('.card-header');
  if (header) {
    // 헤더 클릭 처리
    return;
  }

  // 카드 본문 클릭 처리
});
```

### 4. 이벤트 객체의 속성 변화

이벤트 위임 시 `event.currentTarget`이 예상과 다를 수 있습니다:

```javascript
container.addEventListener('click', function(event) {
  // this와 event.currentTarget은 container
  // event.target은 실제 클릭된 요소

  console.log(this);              // container
  console.log(event.currentTarget); // container
  console.log(event.target);        // 실제 클릭된 하위 요소
});
```

---

## 실무 적용 Best Practices

### 1. 데이터 속성(data-*) 활용

```html
<ul id="product-list">
  <li data-product-id="123" data-action="view">상품 보기</li>
  <li data-product-id="456" data-action="edit">상품 편집</li>
</ul>
```

```javascript
productList.addEventListener('click', (event) => {
  const item = event.target.closest('[data-product-id]');
  if (!item) return;

  const productId = item.dataset.productId;
  const action = item.dataset.action;

  switch (action) {
    case 'view':
      viewProduct(productId);
      break;
    case 'edit':
      editProduct(productId);
      break;
  }
});
```

### 2. 이벤트 위임 헬퍼 함수

```javascript
function delegate(parent, selector, eventType, handler) {
  parent.addEventListener(eventType, (event) => {
    const targetElement = event.target.closest(selector);

    if (targetElement && parent.contains(targetElement)) {
      handler.call(targetElement, event);
    }
  });
}

// 사용 예시
delegate(
  document.getElementById('list'),
  '.item',
  'click',
  function(event) {
    // this는 .item 요소
    console.log('클릭된 항목:', this);
  }
);
```

### 3. 조건부 이벤트 처리

```javascript
const handlers = {
  'delete-btn': (event, element) => {
    const item = element.closest('.item');
    if (confirm('삭제하시겠습니까?')) {
      item.remove();
    }
  },
  'edit-btn': (event, element) => {
    const item = element.closest('.item');
    openEditModal(item.dataset.id);
  },
  'toggle-btn': (event, element) => {
    const item = element.closest('.item');
    item.classList.toggle('completed');
  }
};

container.addEventListener('click', (event) => {
  const target = event.target;

  for (const [className, handler] of Object.entries(handlers)) {
    if (target.closest(`.${className}`)) {
      handler(event, target.closest(`.${className}`));
      return;
    }
  }
});
```

### 4. 성능을 위한 이벤트 위임 전략

```javascript
// 나쁜 예: 너무 높은 레벨에 위임
document.body.addEventListener('click', (event) => {
  // 모든 클릭에 대해 처리해야 함
});

// 좋은 예: 적절한 레벨에 위임
const appRoot = document.getElementById('app');
appRoot.addEventListener('click', (event) => {
  // 앱 내부 클릭만 처리
});

// 더 좋은 예: 기능 단위로 분리
const sidebar = document.getElementById('sidebar');
const content = document.getElementById('content');

sidebar.addEventListener('click', handleSidebarClick);
content.addEventListener('click', handleContentClick);
```

### 5. TypeScript에서의 이벤트 위임

```typescript
interface DelegateOptions<T extends Element = Element> {
  parent: Element;
  selector: string;
  eventType: keyof HTMLElementEventMap;
  handler: (event: Event, element: T) => void;
}

function createDelegate<T extends Element = Element>(
  options: DelegateOptions<T>
): () => void {
  const { parent, selector, eventType, handler } = options;

  const listener = (event: Event) => {
    const target = event.target as Element;
    const delegateTarget = target.closest(selector) as T | null;

    if (delegateTarget && parent.contains(delegateTarget)) {
      handler(event, delegateTarget);
    }
  };

  parent.addEventListener(eventType, listener);

  // cleanup 함수 반환
  return () => parent.removeEventListener(eventType, listener);
}

// 사용 예시
const cleanup = createDelegate<HTMLButtonElement>({
  parent: document.getElementById('toolbar')!,
  selector: 'button',
  eventType: 'click',
  handler: (event, button) => {
    console.log('클릭된 버튼:', button.textContent);
  }
});

// 컴포넌트 언마운트 시
cleanup();
```

---

## 면접 대비 핵심 포인트

### 자주 나오는 질문

**Q1. 이벤트 버블링과 캡처링의 차이점은?**

> 버블링은 이벤트가 타겟 요소에서 시작하여 상위 요소로 전파되는 것이고, 캡처링은 최상위 요소에서 시작하여 타겟 요소로 내려가는 것입니다. 이벤트는 캡처링 -> 타겟 -> 버블링 순서로 전파되며, 기본적으로 이벤트 핸들러는 버블링 단계에서 실행됩니다.

**Q2. event.target과 event.currentTarget의 차이점은?**

> `event.target`은 이벤트가 실제로 발생한 요소이고, `event.currentTarget`은 이벤트 핸들러가 등록된 요소입니다. 이벤트 위임에서 이 차이가 중요합니다.

**Q3. 이벤트 위임이란 무엇이고, 왜 사용하나요?**

> 이벤트 위임은 여러 자식 요소에 개별 핸들러를 등록하는 대신 공통 부모 요소에 하나의 핸들러를 등록하는 패턴입니다. 메모리 효율성, 동적 요소 처리, 코드 간결성 등의 장점이 있습니다.

**Q4. stopPropagation()과 stopImmediatePropagation()의 차이는?**

> `stopPropagation()`은 이벤트의 상위 요소로의 전파를 막고, `stopImmediatePropagation()`은 같은 요소에 등록된 다른 핸들러의 실행까지 막습니다.

**Q5. React에서 이벤트 처리 방식은 Vanilla JS와 어떻게 다른가요?**

> React는 합성 이벤트(SyntheticEvent)를 사용하며, 내부적으로 이벤트 위임을 사용합니다. React 17부터는 document 대신 root 컨테이너에 이벤트를 위임합니다.

---

## 마무리

### 핵심 정리

| 개념 | 설명 |
|------|------|
| **이벤트 흐름** | 캡처링 -> 타겟 -> 버블링 순서로 전파 |
| **버블링** | 타겟에서 상위 요소로 이벤트 전파 |
| **캡처링** | 상위 요소에서 타겟으로 이벤트 전파 |
| **event.target** | 이벤트가 발생한 실제 요소 |
| **event.currentTarget** | 핸들러가 등록된 요소 |
| **이벤트 위임** | 부모 요소에 핸들러를 등록하여 자식 이벤트 처리 |

### 언제 이벤트 위임을 사용해야 하나?

- 동적으로 추가되는 요소가 많을 때
- 비슷한 동작을 하는 여러 요소가 있을 때
- 메모리 사용을 최적화하고 싶을 때
- 이벤트 핸들러 관리를 단순화하고 싶을 때

### 주의사항 요약

- `stopPropagation()` 남용 주의
- 버블링되지 않는 이벤트(focus, blur 등) 확인
- 적절한 레벨의 부모 요소 선택
- `closest()` 메서드를 활용한 정확한 타겟 판별

이벤트 버블링, 캡처링, 그리고 이벤트 위임은 DOM 이벤트 처리의 핵심 개념입니다. 이를 올바르게 이해하고 활용하면 더 효율적이고 유지보수하기 쉬운 코드를 작성할 수 있습니다.

### 함께 읽으면 좋은 글

- [JavaScript Event Loop 이해하기](/posts/javascript-event-loop/) - 비동기 이벤트 처리의 원리
- [JavaScript 실행 컨텍스트 완벽 가이드](/posts/javascript-execution-context-guide/) - 실행 컨텍스트와 스코프 체인
- [JavaScript this 바인딩 완벽 가이드](/posts/javascript-this-binding-guide/) - this의 동작 원리
- [JavaScript 클로저 완벽 가이드](/posts/javascript-closure-complete-guide/) - 클로저와 스코프
- [웹 성능 최적화 가이드](/posts/web-performance-optimization-guide/) - 전반적인 웹 성능 개선
