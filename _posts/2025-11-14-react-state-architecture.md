---
title: "대규모 React 앱의 상태 관리 아키텍처 - 설계 패턴과 성능 최적화"
date: 2025-11-14 20:00:00 +0900
categories: [React, Architecture]
tags: [react, state-management, architecture, performance]
author: changsu
toc: true
comments: true
description: 실무 대규모 React 앱의 상태 관리 아키텍처를 마스터하세요. 상태 계층 설계, useState/Zustand/React Query 조합, E-Commerce 앱 실전 예제, 성능 최적화, 테스트 전략까지 완벽 가이드. 시리즈 완결편.
---

## 들어가며

이 글은 **React 상태 관리 시리즈의 최종 완결편**입니다. 앞서 다룬 내용들을 통합하여 실무 대규모 애플리케이션에서 어떻게 상태를 설계하고 관리하는지 알아봅니다.

### 시리즈 전체 구성

1. [React 상태 관리 완벽 가이드 - useState부터 Context API까지](/posts/react-state-management-guide/)
2. [Zustand vs Redux Toolkit - 전역 상태 관리 비교](/posts/zustand-vs-redux-toolkit/)
3. [React Query 완벽 가이드 - 서버 상태 관리의 정석](/posts/react-query-guide/)
4. **대규모 React 앱의 상태 관리 아키텍처** (이번 글)

간단한 Todo 앱을 만들 때는 useState 하나면 충분합니다. 하지만 실무에서 수십 개의 화면, 수백 개의 컴포넌트, 복잡한 비즈니스 로직을 가진 애플리케이션을 개발할 때는 어떻게 해야 할까요?

이 글에서는 실제 E-Commerce 애플리케이션 수준의 복잡도를 다루며, **언제 어떤 도구를 사용해야 하는지**, **어떻게 조합해야 하는지**, **성능과 유지보수성을 어떻게 확보하는지**를 알아봅니다.

## 대규모 앱의 도전 과제

### 1. 상태의 폭발적 증가

작은 앱과 대규모 앱의 차이를 비교해봅시다:

```jsx
// 작은 Todo 앱
function SmallApp() {
  const [todos, setTodos] = useState([]);
  const [filter, setFilter] = useState('all');

  return <TodoList todos={todos} filter={filter} />;
}
```

```jsx
// 대규모 E-Commerce 앱
function LargeApp() {
  // UI 상태
  const [isSidebarOpen, setIsSidebarOpen] = useState(false);
  const [activeModal, setActiveModal] = useState(null);
  const [selectedTab, setSelectedTab] = useState('products');

  // 폼 상태
  const [searchQuery, setSearchQuery] = useState('');
  const [filters, setFilters] = useState({});
  const [sortBy, setSortBy] = useState('popular');

  // 전역 상태
  const [user, setUser] = useState(null);
  const [cart, setCart] = useState([]);
  const [wishlist, setWishlist] = useState([]);

  // 서버 상태
  const [products, setProducts] = useState([]);
  const [categories, setCategories] = useState([]);
  const [orders, setOrders] = useState([]);

  // 이것만 해도 15개 이상의 상태!
  // 실제로는 50-100개 이상...
}
```

**문제점**:
- 상태가 어디에 있는지 찾기 어려움
- Props Drilling이 심각함
- 리렌더링 최적화가 복잡함
- 테스트가 어려움
- 팀 협업이 어려움

### 2. 상태 간 의존성

상태들이 서로 영향을 주는 경우:

```jsx
// 사용자가 로그아웃하면?
// → 장바구니, 위시리스트, 주문 내역 모두 초기화
// → 인증이 필요한 페이지는 리다이렉트
// → 캐시된 서버 데이터 무효화

// 언어를 변경하면?
// → 모든 레이블 번역
// → 서버 데이터 다시 가져오기 (다른 언어)
// → URL 파라미터 변경
```

이런 복잡한 의존성을 어떻게 관리할까요?

### 3. 성능 문제

```jsx
// Context를 사용했더니...
function AppProvider({ children }) {
  const [state, setState] = useState({
    user: null,
    theme: 'light',
    language: 'ko',
    cart: [],
    notifications: []
  });

  return (
    <AppContext.Provider value={state}>
      {children}
    </AppContext.Provider>
  );
}

// 문제: theme만 변경해도 전체 앱이 리렌더링!
```

**해결이 필요한 성능 문제**:
- 불필요한 리렌더링
- 큰 상태 객체의 직렬화
- 메모리 누수
- 느린 UI 업데이트

## 상태 계층 설계 (4단계 피라미드)

대규모 앱의 상태를 효과적으로 관리하려면 **계층적 접근**이 필요합니다.

| Level | 상태 유형 | 도구 | 역할 |
|:-----:|----------|------|------|
| 4 | Server State | React Query, SWR | API 데이터, 서버 동기화 |
| 3 | Global State | Zustand, Redux Toolkit | 앱 전체 공유 클라이언트 상태 |
| 2 | Context State | Context API | 도메인/기능별 공유 |
| 1 | Local State | useState, useReducer | 컴포넌트 내부 |

### Level 1: Local State (컴포넌트 레벨)

**사용 도구**: `useState`, `useReducer`

**사용 시기**:
- 해당 컴포넌트에서만 사용하는 상태
- UI 인터랙션 (열림/닫힘, 호버, 포커스 등)
- 임시 폼 입력 값

```jsx
function ProductCard({ product }) {
  // ✅ 로컬 상태: 이 카드에서만 사용
  const [isExpanded, setIsExpanded] = useState(false);
  const [quantity, setQuantity] = useState(1);

  return (
    <div>
      <h3>{product.name}</h3>
      <button onClick={() => setIsExpanded(!isExpanded)}>
        {isExpanded ? '접기' : '펼치기'}
      </button>
      {isExpanded && (
        <div>
          <p>{product.description}</p>
          <input
            type="number"
            value={quantity}
            onChange={e => setQuantity(Number(e.target.value))}
          />
        </div>
      )}
    </div>
  );
}
```

### Level 2: Context State (도메인 레벨)

**사용 도구**: `Context API` + `useState/useReducer`

**사용 시기**:
- 특정 도메인(기능)에서만 공유하는 상태
- 인증 상태 (사용자 정보)
- 테마, 언어 설정
- 중간 깊이의 컴포넌트 트리

```jsx
// ✅ Context: 인증 관련 컴포넌트들이 공유
const AuthContext = createContext(null);

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [isLoading, setIsLoading] = useState(true);

  const login = async (credentials) => {
    const user = await authAPI.login(credentials);
    setUser(user);
  };

  const logout = () => {
    setUser(null);
    authAPI.logout();
  };

  const value = useMemo(() => ({
    user,
    isLoading,
    login,
    logout,
    isAuthenticated: !!user
  }), [user, isLoading]);

  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  );
}
```

### Level 3: Global State (앱 레벨)

**사용 도구**: `Zustand`, `Redux Toolkit`

**사용 시기**:
- 앱 전체에서 공유하는 클라이언트 상태
- 장바구니, 위시리스트
- 알림, 토스트 메시지
- UI 설정 (사이드바 상태 등)

```jsx
// ✅ Zustand: 장바구니는 앱 전체에서 접근
import create from 'zustand';
import { devtools, persist } from 'zustand/middleware';

export const useCartStore = create(
  devtools(
    persist(
      (set, get) => ({
        items: [],

        addItem: (product) => set((state) => ({
          items: [...state.items, { ...product, quantity: 1 }]
        })),

        removeItem: (productId) => set((state) => ({
          items: state.items.filter(item => item.id !== productId)
        })),

        updateQuantity: (productId, quantity) => set((state) => ({
          items: state.items.map(item =>
            item.id === productId ? { ...item, quantity } : item
          )
        })),

        clearCart: () => set({ items: [] }),

        // Selector
        getTotalPrice: () => {
          const { items } = get();
          return items.reduce((sum, item) => sum + item.price * item.quantity, 0);
        },

        getItemCount: () => {
          const { items } = get();
          return items.reduce((sum, item) => sum + item.quantity, 0);
        }
      }),
      { name: 'cart-storage' }
    )
  )
);
```

### Level 4: Server State (서버 레벨)

**사용 도구**: `React Query`, `SWR`

**사용 시기**:
- 서버에서 가져오는 모든 데이터
- 상품 목록, 카테고리, 주문 내역
- 사용자 프로필 (서버 동기화 필요)
- 실시간 데이터

```jsx
// ✅ React Query: 서버 데이터
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

export function useProducts(filters) {
  return useQuery({
    queryKey: ['products', filters],
    queryFn: () => fetchProducts(filters),
    staleTime: 5 * 60 * 1000, // 5분
    cacheTime: 10 * 60 * 1000, // 10분
  });
}

export function useProduct(id) {
  return useQuery({
    queryKey: ['product', id],
    queryFn: () => fetchProduct(id),
    enabled: !!id, // id가 있을 때만 실행
  });
}

export function useCreateOrder() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (orderData) => createOrder(orderData),
    onSuccess: () => {
      // 주문 목록 무효화하여 다시 가져오기
      queryClient.invalidateQueries(['orders']);
      // 장바구니 초기화
      useCartStore.getState().clearCart();
    },
  });
}
```

### 계층 선택 가이드

```
상태가 필요한가?
  │
  ├─ 서버에서 오는 데이터인가?
  │    └─ Yes → React Query (Level 4) ✅
  │
  ├─ 앱 전체에서 사용하는가?
  │    └─ Yes → Zustand/Redux (Level 3) ✅
  │
  ├─ 특정 기능(도메인)에서만 사용하는가?
  │    └─ Yes → Context API (Level 2) ✅
  │
  └─ 컴포넌트 내부에서만 사용하는가?
       └─ Yes → useState/useReducer (Level 1) ✅
```

## 상태 분리 전략

상태를 **성격에 따라 분리**하면 관리가 훨씬 쉬워집니다.

### 1. UI 상태 (UI State)

**특징**:
- 클라이언트에서만 필요
- 서버와 동기화 불필요
- 일시적 (페이지 새로고침 시 사라져도 됨)

**예시**:
- 모달 열림/닫힘
- 사이드바 확장/축소
- 탭 선택
- 툴팁 표시
- 로딩 스피너

```jsx
// UI 상태 예제
function ProductList() {
  // ✅ UI 상태: 로컬 관리
  const [viewMode, setViewMode] = useState('grid'); // 'grid' | 'list'
  const [selectedFilters, setSelectedFilters] = useState(new Set());
  const [sortMenuOpen, setSortMenuOpen] = useState(false);

  return (
    <div>
      <div className="toolbar">
        <button onClick={() => setViewMode('grid')}>
          Grid View
        </button>
        <button onClick={() => setViewMode('list')}>
          List View
        </button>
        <button onClick={() => setSortMenuOpen(!sortMenuOpen)}>
          Sort
        </button>
      </div>

      {sortMenuOpen && <SortMenu />}

      <div className={viewMode === 'grid' ? 'grid' : 'list'}>
        {/* Products */}
      </div>
    </div>
  );
}
```

### 2. 도메인 상태 (Domain/Client State)

**특징**:
- 비즈니스 로직과 관련
- 클라이언트에서 생성/관리
- 지속성 필요 (localStorage 등)
- 앱 전체 또는 여러 화면에서 사용

**예시**:
- 장바구니
- 위시리스트
- 임시 저장된 폼 데이터
- 사용자 설정 (테마, 언어)

```jsx
// 도메인 상태 예제 (Zustand)
import create from 'zustand';
import { persist } from 'zustand/middleware';

export const useWishlistStore = create(
  persist(
    (set) => ({
      items: [],

      addToWishlist: (product) => set((state) => {
        if (state.items.some(item => item.id === product.id)) {
          return state; // 이미 있으면 추가 안 함
        }
        return { items: [...state.items, product] };
      }),

      removeFromWishlist: (productId) => set((state) => ({
        items: state.items.filter(item => item.id !== productId)
      })),

      isInWishlist: (productId) => {
        return useWishlistStore.getState().items.some(
          item => item.id === productId
        );
      },

      clearWishlist: () => set({ items: [] })
    }),
    {
      name: 'wishlist-storage',
      // localStorage에 자동 저장
    }
  )
);

// 사용
function ProductCard({ product }) {
  const { addToWishlist, removeFromWishlist, isInWishlist } = useWishlistStore();
  const inWishlist = isInWishlist(product.id);

  return (
    <div>
      <h3>{product.name}</h3>
      <button onClick={() =>
        inWishlist
          ? removeFromWishlist(product.id)
          : addToWishlist(product)
      }>
        {inWishlist ? '❤️' : '🤍'}
      </button>
    </div>
  );
}
```

### 3. 서버 상태 (Server State)

**특징**:
- 서버에서 가져오는 데이터
- 서버와 동기화 필요
- 캐싱, 재검증, 백그라운드 업데이트
- 로딩/에러 상태 관리

**예시**:
- 상품 목록
- 사용자 프로필
- 주문 내역
- 카테고리 데이터

```jsx
// 서버 상태 예제 (React Query)
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

// 상품 목록 조회
export function useProducts(category, filters) {
  return useQuery({
    queryKey: ['products', category, filters],
    queryFn: () => productAPI.getProducts(category, filters),
    staleTime: 5 * 60 * 1000, // 5분간 fresh
    cacheTime: 10 * 60 * 1000, // 10분간 캐시 유지
    retry: 3,
  });
}

// 상품 상세 조회
export function useProduct(productId) {
  return useQuery({
    queryKey: ['product', productId],
    queryFn: () => productAPI.getProduct(productId),
    enabled: !!productId,
    staleTime: 10 * 60 * 1000, // 10분
  });
}

// 리뷰 작성 (Mutation)
export function useCreateReview(productId) {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (reviewData) => reviewAPI.createReview(productId, reviewData),
    onSuccess: () => {
      // 상품 상세 정보 무효화 (리뷰 개수 업데이트)
      queryClient.invalidateQueries(['product', productId]);
      // 리뷰 목록 무효화
      queryClient.invalidateQueries(['reviews', productId]);
    },
    onError: (error) => {
      console.error('리뷰 작성 실패:', error);
    },
  });
}

// 사용
function ProductPage({ productId }) {
  const { data: product, isLoading, error } = useProduct(productId);
  const createReview = useCreateReview(productId);

  if (isLoading) return <div>로딩 중...</div>;
  if (error) return <div>에러: {error.message}</div>;

  const handleReviewSubmit = async (reviewData) => {
    await createReview.mutateAsync(reviewData);
  };

  return (
    <div>
      <h1>{product.name}</h1>
      <ReviewForm onSubmit={handleReviewSubmit} />
    </div>
  );
}
```

### 4. URL 상태 (URL State)

**특징**:
- URL 파라미터와 쿼리스트링으로 관리
- 공유 가능 (URL 복사해서 공유)
- 뒤로가기/앞으로가기 지원
- 북마크 가능

**예시**:
- 검색 쿼리
- 필터 옵션
- 페이지네이션
- 정렬 순서

```jsx
// URL 상태 예제 (react-router-dom v6)
import { useSearchParams } from 'react-router-dom';

function ProductListPage() {
  const [searchParams, setSearchParams] = useSearchParams();

  // URL에서 상태 읽기
  const category = searchParams.get('category') || 'all';
  const sort = searchParams.get('sort') || 'popular';
  const page = parseInt(searchParams.get('page') || '1', 10);
  const minPrice = searchParams.get('minPrice');
  const maxPrice = searchParams.get('maxPrice');

  // URL 상태 업데이트
  const handleCategoryChange = (newCategory) => {
    setSearchParams(prev => {
      prev.set('category', newCategory);
      prev.set('page', '1'); // 카테고리 변경 시 페이지 1로
      return prev;
    });
  };

  const handleSortChange = (newSort) => {
    setSearchParams(prev => {
      prev.set('sort', newSort);
      return prev;
    });
  };

  const handlePageChange = (newPage) => {
    setSearchParams(prev => {
      prev.set('page', String(newPage));
      return prev;
    });
  };

  const handlePriceFilter = (min, max) => {
    setSearchParams(prev => {
      if (min) prev.set('minPrice', String(min));
      else prev.delete('minPrice');

      if (max) prev.set('maxPrice', String(max));
      else prev.delete('maxPrice');

      prev.set('page', '1');
      return prev;
    });
  };

  // React Query와 결합
  const filters = { minPrice, maxPrice };
  const { data: products } = useProducts(category, { sort, page, ...filters });

  return (
    <div>
      <CategoryFilter value={category} onChange={handleCategoryChange} />
      <SortDropdown value={sort} onChange={handleSortChange} />
      <PriceFilter onApply={handlePriceFilter} />

      <ProductGrid products={products?.items} />

      <Pagination
        current={page}
        total={products?.totalPages}
        onChange={handlePageChange}
      />
    </div>
  );
}
```

**장점**:
- 링크를 공유하면 같은 상태를 볼 수 있음
- 뒤로가기 버튼이 자연스럽게 작동
- SEO 친화적
- 새로고침해도 상태 유지

## 도구 선택 가이드

### 언제 무엇을 사용할까?

```typescript
// 의사결정 플로우
function selectStateTool(stateType: string) {
  if (isFromServer(stateType)) {
    return 'React Query'; // 또는 SWR
  }

  if (isURLSharable(stateType)) {
    return 'URL State (useSearchParams)';
  }

  if (needsGlobalAccess(stateType)) {
    if (isComplex(stateType)) {
      return 'Zustand' // 또는 Redux Toolkit';
    }
    return 'Context API';
  }

  if (needsDomainScope(stateType)) {
    return 'Context API';
  }

  if (isComponentLocal(stateType)) {
    if (isComplex(stateType)) {
      return 'useReducer';
    }
    return 'useState';
  }
}
```

### 실전 선택 가이드

| 상태 종류 | 추천 도구 | 이유 |
|---------|---------|------|
| **모달 열림/닫힘** | `useState` | 컴포넌트 로컬 UI 상태 |
| **폼 입력값 (1-3개)** | `useState` | 간단한 상태 |
| **폼 입력값 (4개 이상)** | `useReducer` | 복잡한 상태 로직 |
| **검색 필터** | `URL State` | 공유 가능, 북마크 가능 |
| **테마, 언어 설정** | `Context API` | 앱 전체 설정 |
| **사용자 인증** | `Context API` | 도메인별 상태 |
| **장바구니** | `Zustand + persist` | 전역 + 지속성 |
| **알림 메시지** | `Zustand` | 전역, 동적 추가/제거 |
| **상품 목록** | `React Query` | 서버 데이터 |
| **사용자 프로필** | `React Query` | 서버 데이터 |
| **실시간 채팅** | `React Query (polling)` | 서버 데이터 + 실시간 |

## 도구 조합 패턴

실무에서는 **여러 도구를 조합**해서 사용합니다. 각 도구의 강점을 살리는 것이 핵심입니다.

### 패턴 1: useState + React Query

**사용 시기**: 서버 데이터 + 로컬 UI 상태

```jsx
function ProductPage({ productId }) {
  // 서버 상태: React Query
  const { data: product, isLoading } = useQuery({
    queryKey: ['product', productId],
    queryFn: () => fetchProduct(productId),
  });

  // UI 상태: useState
  const [selectedImage, setSelectedImage] = useState(0);
  const [quantity, setQuantity] = useState(1);
  const [isDescriptionExpanded, setIsDescriptionExpanded] = useState(false);

  if (isLoading) return <Skeleton />;

  return (
    <div>
      <ImageGallery
        images={product.images}
        selected={selectedImage}
        onSelect={setSelectedImage}
      />

      <div>
        <h1>{product.name}</h1>
        <p>{product.price}</p>

        <QuantitySelector value={quantity} onChange={setQuantity} />

        <button onClick={() => setIsDescriptionExpanded(!isDescriptionExpanded)}>
          {isDescriptionExpanded ? '접기' : '자세히 보기'}
        </button>

        {isDescriptionExpanded && <p>{product.description}</p>}
      </div>
    </div>
  );
}
```

### 패턴 2: Zustand + React Query

**사용 시기**: 클라이언트 전역 상태 + 서버 데이터

```jsx
// Zustand: 장바구니 (클라이언트 상태)
const useCartStore = create((set) => ({
  items: [],
  addItem: (item) => set((state) => ({
    items: [...state.items, item]
  })),
  removeItem: (id) => set((state) => ({
    items: state.items.filter(item => item.id !== id)
  })),
}));

// React Query: 상품 정보 (서버 상태)
function ProductCard({ productId }) {
  const { data: product } = useQuery({
    queryKey: ['product', productId],
    queryFn: () => fetchProduct(productId),
  });

  const addItem = useCartStore(state => state.addItem);

  const handleAddToCart = () => {
    // 서버 데이터를 클라이언트 상태에 추가
    addItem({
      id: product.id,
      name: product.name,
      price: product.price,
      image: product.image,
      quantity: 1,
    });
  };

  return (
    <div>
      <h3>{product.name}</h3>
      <button onClick={handleAddToCart}>장바구니 추가</button>
    </div>
  );
}

// 장바구니 페이지
function CartPage() {
  const items = useCartStore(state => state.items);
  const removeItem = useCartStore(state => state.removeItem);

  // React Query Mutation: 주문 생성
  const createOrder = useMutation({
    mutationFn: (orderData) => createOrderAPI(orderData),
    onSuccess: () => {
      // 주문 성공 시 장바구니 비우기
      useCartStore.getState().clearCart();
    },
  });

  const handleCheckout = () => {
    createOrder.mutate({
      items: items.map(item => ({
        productId: item.id,
        quantity: item.quantity,
      })),
    });
  };

  return (
    <div>
      <h1>장바구니</h1>
      {items.map(item => (
        <div key={item.id}>
          {item.name} - {item.quantity}개
          <button onClick={() => removeItem(item.id)}>삭제</button>
        </div>
      ))}
      <button onClick={handleCheckout}>주문하기</button>
    </div>
  );
}
```

### 패턴 3: Context + Zustand + React Query

**사용 시기**: 인증 + 전역 상태 + 서버 데이터 모두 필요

{% raw %}
```jsx
// 1. Context: 인증 상태
const AuthContext = createContext(null);

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);

  const login = async (credentials) => {
    const user = await authAPI.login(credentials);
    setUser(user);
  };

  const logout = () => {
    setUser(null);
    // Zustand 스토어 초기화
    useCartStore.getState().clearCart();
    useWishlistStore.getState().clearWishlist();
    // React Query 캐시 초기화
    queryClient.clear();
  };

  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

// 2. Zustand: 장바구니
const useCartStore = create(persist(
  (set) => ({
    items: [],
    addItem: (item) => set((state) => ({
      items: [...state.items, item]
    })),
    clearCart: () => set({ items: [] }),
  }),
  { name: 'cart' }
));

// 3. React Query: 서버 데이터
function useUserOrders() {
  const { user } = useAuth();

  return useQuery({
    queryKey: ['orders', user?.id],
    queryFn: () => fetchOrders(user.id),
    enabled: !!user, // 로그인한 경우에만 실행
  });
}

// 사용
function DashboardPage() {
  const { user, logout } = useAuth(); // Context
  const cartItems = useCartStore(state => state.items); // Zustand
  const { data: orders } = useUserOrders(); // React Query

  return (
    <div>
      <h1>안녕하세요, {user.name}님!</h1>
      <p>장바구니: {cartItems.length}개</p>
      <p>주문 내역: {orders?.length}개</p>
      <button onClick={logout}>로그아웃</button>
    </div>
  );
}
```
{% endraw %}

### 패턴 4: URL State + React Query

**사용 시기**: 검색/필터 + 서버 데이터

```jsx
import { useSearchParams } from 'react-router-dom';

function ProductListPage() {
  // 1. URL State: 필터 조건
  const [searchParams, setSearchParams] = useSearchParams();

  const filters = {
    category: searchParams.get('category') || 'all',
    minPrice: searchParams.get('minPrice'),
    maxPrice: searchParams.get('maxPrice'),
    sort: searchParams.get('sort') || 'popular',
    page: parseInt(searchParams.get('page') || '1', 10),
  };

  // 2. React Query: URL 기반으로 서버 데이터 조회
  const { data, isLoading } = useQuery({
    queryKey: ['products', filters],
    queryFn: () => fetchProducts(filters),
    keepPreviousData: true, // 페이지 전환 시 이전 데이터 유지
  });

  // 3. 필터 변경 → URL 업데이트 → React Query 자동 재실행
  const updateFilter = (key, value) => {
    setSearchParams(prev => {
      prev.set(key, value);
      prev.set('page', '1'); // 필터 변경 시 첫 페이지로
      return prev;
    });
  };

  return (
    <div>
      <FilterBar filters={filters} onChange={updateFilter} />

      {isLoading ? (
        <Skeleton count={12} />
      ) : (
        <ProductGrid products={data.items} />
      )}

      <Pagination
        current={filters.page}
        total={data.totalPages}
        onChange={(page) => updateFilter('page', page)}
      />
    </div>
  );
}
```

**장점**:
- URL이 단일 진실 공급원 (Single Source of Truth)
- 공유 가능한 URL
- 뒤로가기/앞으로가기 자동 지원
- React Query가 URL 변경 감지하여 자동 refetch

## 대규모 앱 폴더 구조

### 권장 폴더 구조 (Feature-First)

```
src/
├── features/                    # 기능별 모듈
│   ├── auth/                   # 인증 기능
│   │   ├── components/
│   │   │   ├── LoginForm.jsx
│   │   │   ├── SignupForm.jsx
│   │   │   └── ProtectedRoute.jsx
│   │   ├── hooks/
│   │   │   ├── useAuth.js
│   │   │   └── useAuthQuery.js
│   │   ├── context/
│   │   │   └── AuthContext.jsx
│   │   ├── api/
│   │   │   └── authAPI.js
│   │   └── index.js            # Public exports
│   │
│   ├── products/               # 상품 기능
│   │   ├── components/
│   │   │   ├── ProductCard.jsx
│   │   │   ├── ProductList.jsx
│   │   │   ├── ProductDetail.jsx
│   │   │   └── ProductFilters.jsx
│   │   ├── hooks/
│   │   │   ├── useProducts.js
│   │   │   ├── useProduct.js
│   │   │   └── useProductFilters.js
│   │   ├── api/
│   │   │   └── productsAPI.js
│   │   └── index.js
│   │
│   ├── cart/                   # 장바구니 기능
│   │   ├── components/
│   │   │   ├── CartDrawer.jsx
│   │   │   ├── CartItem.jsx
│   │   │   └── CartSummary.jsx
│   │   ├── store/
│   │   │   └── cartStore.js    # Zustand store
│   │   └── index.js
│   │
│   └── orders/                 # 주문 기능
│       ├── components/
│       ├── hooks/
│       ├── api/
│       └── index.js
│
├── shared/                     # 공유 리소스
│   ├── components/             # 공통 컴포넌트
│   │   ├── Button.jsx
│   │   ├── Modal.jsx
│   │   ├── Input.jsx
│   │   └── Skeleton.jsx
│   ├── hooks/                  # 공통 훅
│   │   ├── useDebounce.js
│   │   ├── useLocalStorage.js
│   │   └── useMediaQuery.js
│   ├── utils/                  # 유틸리티
│   │   ├── format.js
│   │   ├── validation.js
│   │   └── constants.js
│   └── types/                  # TypeScript 타입
│       └── common.ts
│
├── store/                      # 전역 상태 관리
│   ├── index.js               # Root store (Zustand)
│   ├── cartStore.js
│   ├── notificationStore.js
│   └── uiStore.js
│
├── api/                        # API 클라이언트
│   ├── client.js              # Axios 인스턴스
│   ├── interceptors.js
│   └── endpoints.js
│
├── providers/                  # Context Providers
│   ├── AuthProvider.jsx
│   ├── ThemeProvider.jsx
│   ├── QueryProvider.jsx      # React Query Provider
│   └── AppProviders.jsx       # 모든 Provider 통합
│
├── routes/                     # 라우팅
│   ├── index.jsx
│   ├── ProtectedRoute.jsx
│   └── routes.js
│
├── App.jsx
└── main.jsx
```

### Feature 모듈 예제

```jsx
// features/products/index.js
// 외부로 노출할 것만 export

export { ProductList } from './components/ProductList';
export { ProductDetail } from './components/ProductDetail';
export { ProductCard } from './components/ProductCard';

export { useProducts, useProduct } from './hooks/useProducts';
```

```jsx
// features/products/hooks/useProducts.js
import { useQuery } from '@tanstack/react-query';
import { productsAPI } from '../api/productsAPI';

export function useProducts(filters) {
  return useQuery({
    queryKey: ['products', filters],
    queryFn: () => productsAPI.getProducts(filters),
    staleTime: 5 * 60 * 1000,
  });
}

export function useProduct(id) {
  return useQuery({
    queryKey: ['product', id],
    queryFn: () => productsAPI.getProduct(id),
    enabled: !!id,
  });
}
```

```jsx
// features/products/api/productsAPI.js
import { apiClient } from '@/api/client';

export const productsAPI = {
  getProducts: async (filters) => {
    const { data } = await apiClient.get('/products', { params: filters });
    return data;
  },

  getProduct: async (id) => {
    const { data } = await apiClient.get(`/products/${id}`);
    return data;
  },

  createProduct: async (productData) => {
    const { data } = await apiClient.post('/products', productData);
    return data;
  },
};
```

### Store 구조 (Zustand)

```jsx
// store/cartStore.js
import create from 'zustand';
import { devtools, persist } from 'zustand/middleware';

export const useCartStore = create(
  devtools(
    persist(
      (set, get) => ({
        items: [],

        // Actions
        addItem: (product) => set((state) => {
          const existingItem = state.items.find(item => item.id === product.id);

          if (existingItem) {
            return {
              items: state.items.map(item =>
                item.id === product.id
                  ? { ...item, quantity: item.quantity + 1 }
                  : item
              ),
            };
          }

          return {
            items: [...state.items, { ...product, quantity: 1 }],
          };
        }),

        removeItem: (productId) => set((state) => ({
          items: state.items.filter(item => item.id !== productId),
        })),

        updateQuantity: (productId, quantity) => set((state) => ({
          items: state.items.map(item =>
            item.id === productId ? { ...item, quantity } : item
          ),
        })),

        clearCart: () => set({ items: [] }),

        // Selectors (메모이제이션됨)
        getTotalPrice: () => {
          const { items } = get();
          return items.reduce((sum, item) => sum + item.price * item.quantity, 0);
        },

        getTotalItems: () => {
          const { items } = get();
          return items.reduce((sum, item) => sum + item.quantity, 0);
        },
      }),
      {
        name: 'cart-storage',
        partialize: (state) => ({ items: state.items }), // items만 persist
      }
    ),
    { name: 'CartStore' } // Redux DevTools에서 보이는 이름
  )
);

// Selector Hook (리렌더링 최적화)
export const useCartItems = () => useCartStore(state => state.items);
export const useCartActions = () => useCartStore(state => ({
  addItem: state.addItem,
  removeItem: state.removeItem,
  updateQuantity: state.updateQuantity,
  clearCart: state.clearCart,
}));
```

### Providers 통합

```jsx
// providers/AppProviders.jsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';
import { BrowserRouter } from 'react-router-dom';
import { AuthProvider } from '@/features/auth';
import { ThemeProvider } from './ThemeProvider';

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 60 * 1000, // 1분
      cacheTime: 5 * 60 * 1000, // 5분
      retry: 1,
      refetchOnWindowFocus: false,
    },
  },
});

export function AppProviders({ children }) {
  return (
    <BrowserRouter>
      <QueryClientProvider client={queryClient}>
        <AuthProvider>
          <ThemeProvider>
            {children}
          </ThemeProvider>
        </AuthProvider>
        <ReactQueryDevtools initialIsOpen={false} />
      </QueryClientProvider>
    </BrowserRouter>
  );
}
```

```jsx
// main.jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import { AppProviders } from './providers/AppProviders';
import App from './App';
import './index.css';

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <AppProviders>
      <App />
    </AppProviders>
  </React.StrictMode>
);
```

## 실전: E-Commerce 앱 아키텍처

실제 E-Commerce 앱 수준의 완전한 예제를 구현해봅시다.

### 1. 인증 모듈 (Context API)

```jsx
// features/auth/context/AuthContext.jsx
import { createContext, useContext, useState, useEffect, useMemo } from 'react';
import { authAPI } from '../api/authAPI';

const AuthContext = createContext(null);

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [isLoading, setIsLoading] = useState(true);

  // 초기 로드: 토큰으로 사용자 정보 복원
  useEffect(() => {
    const initAuth = async () => {
      const token = localStorage.getItem('token');
      if (token) {
        try {
          const user = await authAPI.me();
          setUser(user);
        } catch (error) {
          localStorage.removeItem('token');
        }
      }
      setIsLoading(false);
    };

    initAuth();
  }, []);

  const login = async (credentials) => {
    try {
      const { user, token } = await authAPI.login(credentials);
      localStorage.setItem('token', token);
      setUser(user);
      return { success: true };
    } catch (error) {
      return { success: false, error: error.message };
    }
  };

  const signup = async (userData) => {
    try {
      const { user, token } = await authAPI.signup(userData);
      localStorage.setItem('token', token);
      setUser(user);
      return { success: true };
    } catch (error) {
      return { success: false, error: error.message };
    }
  };

  const logout = () => {
    localStorage.removeItem('token');
    setUser(null);

    // 다른 스토어 초기화
    const { useCartStore } = await import('@/store/cartStore');
    useCartStore.getState().clearCart();

    // React Query 캐시 초기화
    const { queryClient } = await import('@/api/queryClient');
    queryClient.clear();
  };

  const updateUser = (updates) => {
    setUser(prev => ({ ...prev, ...updates }));
  };

  const value = useMemo(() => ({
    user,
    isLoading,
    isAuthenticated: !!user,
    login,
    signup,
    logout,
    updateUser,
  }), [user, isLoading]);

  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
}
```

### 2. 장바구니 모듈 (Zustand)

```jsx
// features/cart/store/cartStore.js
import create from 'zustand';
import { devtools, persist } from 'zustand/middleware';
import { immer } from 'zustand/middleware/immer';

export const useCartStore = create(
  devtools(
    persist(
      immer((set, get) => ({
        items: [],

        addItem: (product) => set((state) => {
          const existingItem = state.items.find(item => item.id === product.id);

          if (existingItem) {
            existingItem.quantity += 1;
          } else {
            state.items.push({ ...product, quantity: 1 });
          }
        }),

        removeItem: (productId) => set((state) => {
          state.items = state.items.filter(item => item.id !== productId);
        }),

        updateQuantity: (productId, quantity) => set((state) => {
          const item = state.items.find(item => item.id === productId);
          if (item) {
            if (quantity <= 0) {
              state.items = state.items.filter(item => item.id !== productId);
            } else {
              item.quantity = quantity;
            }
          }
        }),

        clearCart: () => set({ items: [] }),

        // Computed values
        get totalPrice() {
          return get().items.reduce((sum, item) =>
            sum + item.price * item.quantity, 0
          );
        },

        get totalItems() {
          return get().items.reduce((sum, item) =>
            sum + item.quantity, 0
          );
        },

        get itemCount() {
          return get().items.length;
        },
      })),
      {
        name: 'cart-storage',
        partialize: (state) => ({ items: state.items }),
      }
    ),
    { name: 'CartStore' }
  )
);

// Selector hooks (리렌더링 최적화)
export const useCartItems = () => useCartStore(state => state.items);
export const useCartTotalPrice = () => useCartStore(state => state.totalPrice);
export const useCartTotalItems = () => useCartStore(state => state.totalItems);

export const useCartActions = () => useCartStore(state => ({
  addItem: state.addItem,
  removeItem: state.removeItem,
  updateQuantity: state.updateQuantity,
  clearCart: state.clearCart,
}));
```

```jsx
// features/cart/components/CartDrawer.jsx
import { useCartItems, useCartTotalPrice, useCartActions } from '../store/cartStore';
import { useAuth } from '@/features/auth';
import { useCreateOrder } from '@/features/orders';

export function CartDrawer({ isOpen, onClose }) {
  const items = useCartItems();
  const totalPrice = useCartTotalPrice();
  const { removeItem, updateQuantity } = useCartActions();
  const { isAuthenticated } = useAuth();
  const createOrder = useCreateOrder();

  const handleCheckout = async () => {
    if (!isAuthenticated) {
      // 로그인 페이지로 리다이렉트
      window.location.href = '/login?redirect=/checkout';
      return;
    }

    try {
      await createOrder.mutateAsync({
        items: items.map(item => ({
          productId: item.id,
          quantity: item.quantity,
          price: item.price,
        })),
      });

      // 성공 시 체크아웃 페이지로
      window.location.href = '/checkout/success';
    } catch (error) {
      alert('주문 생성 실패: ' + error.message);
    }
  };

  return (
    <div className={`drawer ${isOpen ? 'open' : ''}`}>
      <div className="drawer-header">
        <h2>장바구니 ({items.length})</h2>
        <button onClick={onClose}>×</button>
      </div>

      <div className="drawer-body">
        {items.length === 0 ? (
          <div className="empty-cart">
            <p>장바구니가 비어있습니다</p>
          </div>
        ) : (
          <>
            {items.map(item => (
              <div key={item.id} className="cart-item">
                <img src={item.image} alt={item.name} />
                <div className="item-info">
                  <h4>{item.name}</h4>
                  <p>{item.price.toLocaleString()}원</p>
                </div>
                <div className="quantity-controls">
                  <button onClick={() => updateQuantity(item.id, item.quantity - 1)}>
                    -
                  </button>
                  <span>{item.quantity}</span>
                  <button onClick={() => updateQuantity(item.id, item.quantity + 1)}>
                    +
                  </button>
                </div>
                <button onClick={() => removeItem(item.id)}>삭제</button>
              </div>
            ))}
          </>
        )}
      </div>

      <div className="drawer-footer">
        <div className="total">
          <span>총 금액</span>
          <strong>{totalPrice.toLocaleString()}원</strong>
        </div>
        <button
          onClick={handleCheckout}
          disabled={items.length === 0 || createOrder.isLoading}
        >
          {createOrder.isLoading ? '처리 중...' : '주문하기'}
        </button>
      </div>
    </div>
  );
}
```

### 3. 상품 모듈 (React Query)

```jsx
// features/products/hooks/useProducts.js
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { productsAPI } from '../api/productsAPI';

export function useProducts(filters) {
  return useQuery({
    queryKey: ['products', filters],
    queryFn: () => productsAPI.getProducts(filters),
    staleTime: 5 * 60 * 1000, // 5분
    keepPreviousData: true, // 페이지네이션 시 유용
  });
}

export function useProduct(id) {
  return useQuery({
    queryKey: ['product', id],
    queryFn: () => productsAPI.getProduct(id),
    enabled: !!id,
    staleTime: 10 * 60 * 1000, // 10분
  });
}

export function useProductReviews(productId) {
  return useQuery({
    queryKey: ['reviews', productId],
    queryFn: () => productsAPI.getReviews(productId),
    enabled: !!productId,
  });
}

export function useCreateReview(productId) {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (reviewData) => productsAPI.createReview(productId, reviewData),
    onSuccess: () => {
      // 리뷰 목록 무효화
      queryClient.invalidateQueries(['reviews', productId]);
      // 상품 상세 정보 무효화 (평점 업데이트)
      queryClient.invalidateQueries(['product', productId]);
    },
  });
}
```

```jsx
// features/products/components/ProductList.jsx
import { useSearchParams } from 'react-router-dom';
import { useProducts } from '../hooks/useProducts';
import { ProductCard } from './ProductCard';
import { ProductFilters } from './ProductFilters';
import { Pagination } from '@/shared/components/Pagination';

export function ProductList() {
  const [searchParams, setSearchParams] = useSearchParams();

  const filters = {
    category: searchParams.get('category') || 'all',
    minPrice: searchParams.get('minPrice'),
    maxPrice: searchParams.get('maxPrice'),
    sort: searchParams.get('sort') || 'popular',
    page: parseInt(searchParams.get('page') || '1', 10),
    search: searchParams.get('search') || '',
  };

  const { data, isLoading, error } = useProducts(filters);

  const updateFilter = (key, value) => {
    setSearchParams(prev => {
      if (value) {
        prev.set(key, value);
      } else {
        prev.delete(key);
      }

      // 필터 변경 시 첫 페이지로
      if (key !== 'page') {
        prev.set('page', '1');
      }

      return prev;
    });
  };

  if (error) {
    return (
      <div className="error">
        <p>상품을 불러오는데 실패했습니다.</p>
        <button onClick={() => window.location.reload()}>다시 시도</button>
      </div>
    );
  }

  return (
    <div className="product-list-page">
      <ProductFilters filters={filters} onChange={updateFilter} />

      <div className="results-info">
        <p>총 {data?.total || 0}개의 상품</p>
        <select
          value={filters.sort}
          onChange={e => updateFilter('sort', e.target.value)}
        >
          <option value="popular">인기순</option>
          <option value="latest">최신순</option>
          <option value="price-low">낮은 가격순</option>
          <option value="price-high">높은 가격순</option>
        </select>
      </div>

      {isLoading ? (
        <div className="product-grid">
          {[...Array(12)].map((_, i) => (
            <ProductCardSkeleton key={i} />
          ))}
        </div>
      ) : (
        <>
          <div className="product-grid">
            {data.items.map(product => (
              <ProductCard key={product.id} product={product} />
            ))}
          </div>

          <Pagination
            current={filters.page}
            total={data.totalPages}
            onChange={(page) => updateFilter('page', page)}
          />
        </>
      )}
    </div>
  );
}
```

```jsx
// features/products/components/ProductCard.jsx
import { Link } from 'react-router-dom';
import { useCartActions } from '@/features/cart';
import { useWishlistStore } from '@/store/wishlistStore';

export function ProductCard({ product }) {
  const { addItem } = useCartActions();
  const { isInWishlist, toggleWishlist } = useWishlistStore();

  const inWishlist = isInWishlist(product.id);

  const handleAddToCart = (e) => {
    e.preventDefault(); // Link 클릭 방지
    addItem(product);
    // 토스트 알림 표시
    useNotificationStore.getState().addNotification({
      type: 'success',
      message: '장바구니에 추가되었습니다',
    });
  };

  const handleToggleWishlist = (e) => {
    e.preventDefault();
    toggleWishlist(product);
  };

  return (
    <Link to={`/products/${product.id}`} className="product-card">
      <div className="image-wrapper">
        <img src={product.image} alt={product.name} />
        <button
          className="wishlist-btn"
          onClick={handleToggleWishlist}
          aria-label={inWishlist ? '위시리스트에서 제거' : '위시리스트에 추가'}
        >
          {inWishlist ? '❤️' : '🤍'}
        </button>
      </div>

      <div className="product-info">
        <h3>{product.name}</h3>
        <p className="price">{product.price.toLocaleString()}원</p>
        <div className="rating">
          ⭐ {product.rating} ({product.reviewCount})
        </div>
      </div>

      <button className="add-to-cart-btn" onClick={handleAddToCart}>
        장바구니 담기
      </button>
    </Link>
  );
}
```

### 4. 주문 모듈 (Zustand + React Query)

```jsx
// features/orders/hooks/useOrders.js
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { ordersAPI } from '../api/ordersAPI';
import { useCartStore } from '@/features/cart';

export function useOrders() {
  return useQuery({
    queryKey: ['orders'],
    queryFn: () => ordersAPI.getOrders(),
    staleTime: 60 * 1000, // 1분
  });
}

export function useOrder(orderId) {
  return useQuery({
    queryKey: ['order', orderId],
    queryFn: () => ordersAPI.getOrder(orderId),
    enabled: !!orderId,
  });
}

export function useCreateOrder() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (orderData) => ordersAPI.createOrder(orderData),
    onSuccess: (newOrder) => {
      // 주문 목록에 새 주문 추가
      queryClient.setQueryData(['orders'], (old) => {
        return old ? [newOrder, ...old] : [newOrder];
      });

      // 장바구니 초기화
      useCartStore.getState().clearCart();
    },
  });
}

export function useCancelOrder() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (orderId) => ordersAPI.cancelOrder(orderId),
    onSuccess: (_, orderId) => {
      // 주문 목록 무효화
      queryClient.invalidateQueries(['orders']);
      // 특정 주문 무효화
      queryClient.invalidateQueries(['order', orderId]);
    },
  });
}
```

### 5. 모듈 간 통신 패턴

```jsx
// 예제: 로그아웃 시 모든 상태 초기화
export function useLogout() {
  const { logout: authLogout } = useAuth();
  const queryClient = useQueryClient();

  return async () => {
    // 1. 인증 상태 초기화
    authLogout();

    // 2. Zustand 스토어 초기화
    useCartStore.getState().clearCart();
    useWishlistStore.getState().clearWishlist();
    useNotificationStore.getState().clearAll();

    // 3. React Query 캐시 초기화
    queryClient.clear();

    // 4. localStorage 초기화
    localStorage.removeItem('token');
    localStorage.removeItem('cart-storage');
    localStorage.removeItem('wishlist-storage');

    // 5. 홈으로 리다이렉트
    window.location.href = '/';
  };
}
```

```jsx
// 예제: 주문 성공 시 여러 상태 업데이트
export function useCheckout() {
  const createOrder = useCreateOrder();
  const { user } = useAuth();
  const cartItems = useCartItems();

  const handleCheckout = async (shippingInfo) => {
    try {
      // 1. 주문 생성
      const order = await createOrder.mutateAsync({
        userId: user.id,
        items: cartItems,
        shippingInfo,
        totalPrice: useCartStore.getState().totalPrice,
      });

      // 2. 알림 추가
      useNotificationStore.getState().addNotification({
        type: 'success',
        message: '주문이 완료되었습니다!',
      });

      // 3. 장바구니 자동 초기화 (createOrder onSuccess에서)

      // 4. 주문 완료 페이지로 이동
      return { success: true, orderId: order.id };
    } catch (error) {
      useNotificationStore.getState().addNotification({
        type: 'error',
        message: '주문 실패: ' + error.message,
      });
      return { success: false, error };
    }
  };

  return { handleCheckout, isLoading: createOrder.isLoading };
}
```

## 성능 최적화 전략

### 1. 리렌더링 최적화

#### React.memo로 컴포넌트 메모이제이션

```jsx
// ❌ 최적화 안 된 코드
function ProductCard({ product, onAddToCart }) {
  console.log('ProductCard 렌더링:', product.id);
  return (
    <div>
      <h3>{product.name}</h3>
      <button onClick={() => onAddToCart(product)}>담기</button>
    </div>
  );
}

function ProductList({ products }) {
  const [sortBy, setSortBy] = useState('popular');

  const handleAddToCart = (product) => {
    // 장바구니에 추가
  };

  return (
    <div>
      <button onClick={() => setSortBy('price')}>정렬 변경</button>
      {/* sortBy 변경 시 모든 ProductCard 리렌더링! */}
      {products.map(product => (
        <ProductCard
          key={product.id}
          product={product}
          onAddToCart={handleAddToCart}
        />
      ))}
    </div>
  );
}
```

```jsx
// ✅ 최적화된 코드
import { memo, useCallback } from 'react';

const ProductCard = memo(function ProductCard({ product, onAddToCart }) {
  console.log('ProductCard 렌더링:', product.id);
  return (
    <div>
      <h3>{product.name}</h3>
      <button onClick={() => onAddToCart(product)}>담기</button>
    </div>
  );
});

function ProductList({ products }) {
  const [sortBy, setSortBy] = useState('popular');

  // useCallback으로 함수 메모이제이션
  const handleAddToCart = useCallback((product) => {
    // 장바구니에 추가
  }, []);

  return (
    <div>
      <button onClick={() => setSortBy('price')}>정렬 변경</button>
      {/* sortBy 변경해도 ProductCard는 리렌더링 안 됨! */}
      {products.map(product => (
        <ProductCard
          key={product.id}
          product={product}
          onAddToCart={handleAddToCart}
        />
      ))}
    </div>
  );
}
```

#### Zustand Selector로 최적화

```jsx
// ❌ 비효율적: 전체 상태 구독
function CartBadge() {
  const cart = useCartStore(); // 전체 상태 구독
  const itemCount = cart.items.length;

  // cart의 어떤 값이 변경되어도 리렌더링됨
  return <span>{itemCount}</span>;
}

// ✅ 효율적: Selector로 필요한 값만 구독
function CartBadge() {
  const itemCount = useCartStore(state => state.items.length);

  // items.length가 변경될 때만 리렌더링
  return <span>{itemCount}</span>;
}

// ✅ 더 나은 방법: Computed value 사용
function CartBadge() {
  const itemCount = useCartStore(state => state.itemCount);

  return <span>{itemCount}</span>;
}
```

#### Context 분리로 최적화

```jsx
// ❌ 비효율적: 하나의 Context
const AppContext = createContext();

function AppProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');

  const value = useMemo(() => ({
    user, setUser,
    theme, setTheme,
  }), [user, theme]);

  return <AppContext.Provider value={value}>{children}</AppContext.Provider>;
}

// theme만 사용하는 컴포넌트도 user 변경 시 리렌더링됨!

// ✅ 효율적: Context 분리
const UserContext = createContext();
const ThemeContext = createContext();

function UserProvider({ children }) {
  const [user, setUser] = useState(null);
  const value = useMemo(() => ({ user, setUser }), [user]);
  return <UserContext.Provider value={value}>{children}</UserContext.Provider>;
}

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  const value = useMemo(() => ({ theme, setTheme }), [theme]);
  return <ThemeContext.Provider value={value}>{children}</ThemeContext.Provider>;
}

// 이제 theme만 사용하는 컴포넌트는 user 변경 시 리렌더링 안 됨!
```

### 2. React Query 성능 최적화

```jsx
// ✅ Prefetching으로 UX 개선
import { useQueryClient } from '@tanstack/react-query';

function ProductCard({ product }) {
  const queryClient = useQueryClient();

  // 마우스 호버 시 상세 페이지 데이터 미리 가져오기
  const handleMouseEnter = () => {
    queryClient.prefetchQuery({
      queryKey: ['product', product.id],
      queryFn: () => fetchProduct(product.id),
      staleTime: 10 * 60 * 1000,
    });
  };

  return (
    <Link
      to={`/products/${product.id}`}
      onMouseEnter={handleMouseEnter}
    >
      {product.name}
    </Link>
  );
}
```

```jsx
// ✅ Optimistic Update로 즉각적인 피드백
export function useToggleWishlist() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ({ productId, isInWishlist }) =>
      isInWishlist
        ? removeFromWishlist(productId)
        : addToWishlist(productId),

    // 낙관적 업데이트
    onMutate: async ({ productId, isInWishlist }) => {
      // 진행 중인 쿼리 취소
      await queryClient.cancelQueries(['wishlist']);

      // 이전 상태 백업
      const previousWishlist = queryClient.getQueryData(['wishlist']);

      // 즉시 UI 업데이트
      queryClient.setQueryData(['wishlist'], (old) => {
        if (isInWishlist) {
          return old.filter(id => id !== productId);
        } else {
          return [...old, productId];
        }
      });

      return { previousWishlist };
    },

    // 에러 시 롤백
    onError: (err, variables, context) => {
      queryClient.setQueryData(['wishlist'], context.previousWishlist);
    },

    // 완료 후 서버 데이터와 동기화
    onSettled: () => {
      queryClient.invalidateQueries(['wishlist']);
    },
  });
}
```

```jsx
// ✅ keepPreviousData로 페이지네이션 개선
export function useProducts(page, filters) {
  return useQuery({
    queryKey: ['products', page, filters],
    queryFn: () => fetchProducts(page, filters),
    keepPreviousData: true, // 이전 페이지 데이터 유지
    staleTime: 5 * 60 * 1000,
  });
}

// 사용
function ProductList() {
  const [page, setPage] = useState(1);
  const { data, isPreviousData } = useProducts(page);

  return (
    <div>
      {/* isPreviousData가 true일 때 로딩 표시 */}
      {isPreviousData && <div>업데이트 중...</div>}

      <ProductGrid products={data?.items} />

      <button
        onClick={() => setPage(p => p + 1)}
        disabled={isPreviousData} // 로딩 중에는 비활성화
      >
        다음 페이지
      </button>
    </div>
  );
}
```

### 3. 가상화 (Virtualization)

긴 목록을 렌더링할 때 성능 개선:

```jsx
import { FixedSizeList } from 'react-window';

function VirtualizedProductList({ products }) {
  const Row = ({ index, style }) => {
    const product = products[index];
    return (
      <div style={style}>
        <ProductCard product={product} />
      </div>
    );
  };

  return (
    <FixedSizeList
      height={800}
      itemCount={products.length}
      itemSize={200}
      width="100%"
    >
      {Row}
    </FixedSizeList>
  );
}
```

### 4. 코드 스플리팅

```jsx
import { lazy, Suspense } from 'react';

// ✅ 라우트별 코드 스플리팅
const ProductListPage = lazy(() => import('@/features/products/pages/ProductListPage'));
const ProductDetailPage = lazy(() => import('@/features/products/pages/ProductDetailPage'));
const CheckoutPage = lazy(() => import('@/features/checkout/pages/CheckoutPage'));

function App() {
  return (
    <Suspense fallback={<PageLoader />}>
      <Routes>
        <Route path="/products" element={<ProductListPage />} />
        <Route path="/products/:id" element={<ProductDetailPage />} />
        <Route path="/checkout" element={<CheckoutPage />} />
      </Routes>
    </Suspense>
  );
}
```

### 5. 측정 도구

성능 최적화는 **측정 후 개선**해야 합니다.

```jsx
// React DevTools Profiler 사용
import { Profiler } from 'react';

function onRenderCallback(
  id, // 프로파일러 ID
  phase, // "mount" | "update"
  actualDuration, // 렌더링 시간
  baseDuration, // 메모이제이션 없이 걸리는 시간
  startTime,
  commitTime,
  interactions
) {
  console.log(`${id} ${phase} - ${actualDuration}ms`);
}

function App() {
  return (
    <Profiler id="App" onRender={onRenderCallback}>
      <ProductList />
    </Profiler>
  );
}
```

**측정 기준**:
- **First Contentful Paint (FCP)**: < 1.8s
- **Largest Contentful Paint (LCP)**: < 2.5s
- **Cumulative Layout Shift (CLS)**: < 0.1
- **Time to Interactive (TTI)**: < 3.8s

## 테스트 전략

### 1. 단위 테스트 (Unit Tests)

#### Zustand 스토어 테스트

```jsx
// cartStore.test.js
import { renderHook, act } from '@testing-library/react';
import { useCartStore } from './cartStore';

describe('useCartStore', () => {
  beforeEach(() => {
    // 각 테스트 전에 스토어 초기화
    useCartStore.setState({ items: [] });
  });

  it('should add item to cart', () => {
    const { result } = renderHook(() => useCartStore());

    act(() => {
      result.current.addItem({
        id: 1,
        name: 'Product 1',
        price: 1000,
      });
    });

    expect(result.current.items).toHaveLength(1);
    expect(result.current.items[0]).toMatchObject({
      id: 1,
      name: 'Product 1',
      quantity: 1,
    });
  });

  it('should increase quantity if item already exists', () => {
    const { result } = renderHook(() => useCartStore());

    const product = { id: 1, name: 'Product 1', price: 1000 };

    act(() => {
      result.current.addItem(product);
      result.current.addItem(product);
    });

    expect(result.current.items).toHaveLength(1);
    expect(result.current.items[0].quantity).toBe(2);
  });

  it('should calculate total price correctly', () => {
    const { result } = renderHook(() => useCartStore());

    act(() => {
      result.current.addItem({ id: 1, name: 'A', price: 1000 });
      result.current.addItem({ id: 2, name: 'B', price: 2000 });
    });

    expect(result.current.totalPrice).toBe(3000);
  });

  it('should remove item from cart', () => {
    const { result } = renderHook(() => useCartStore());

    act(() => {
      result.current.addItem({ id: 1, name: 'A', price: 1000 });
      result.current.addItem({ id: 2, name: 'B', price: 2000 });
    });

    act(() => {
      result.current.removeItem(1);
    });

    expect(result.current.items).toHaveLength(1);
    expect(result.current.items[0].id).toBe(2);
  });
});
```

#### 커스텀 훅 테스트

```jsx
// useAuth.test.js
import { renderHook, act, waitFor } from '@testing-library/react';
import { AuthProvider, useAuth } from './AuthContext';

const wrapper = ({ children }) => <AuthProvider>{children}</AuthProvider>;

describe('useAuth', () => {
  it('should login successfully', async () => {
    const { result } = renderHook(() => useAuth(), { wrapper });

    await act(async () => {
      const response = await result.current.login({
        email: 'test@example.com',
        password: 'password',
      });

      expect(response.success).toBe(true);
    });

    await waitFor(() => {
      expect(result.current.user).not.toBeNull();
      expect(result.current.isAuthenticated).toBe(true);
    });
  });

  it('should logout and clear user', async () => {
    const { result } = renderHook(() => useAuth(), { wrapper });

    // 먼저 로그인
    await act(async () => {
      await result.current.login({
        email: 'test@example.com',
        password: 'password',
      });
    });

    // 로그아웃
    act(() => {
      result.current.logout();
    });

    expect(result.current.user).toBeNull();
    expect(result.current.isAuthenticated).toBe(false);
  });
});
```

### 2. 통합 테스트 (Integration Tests)

```jsx
// ProductCard.test.jsx
import { render, screen, fireEvent } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { BrowserRouter } from 'react-router-dom';
import { ProductCard } from './ProductCard';

const queryClient = new QueryClient({
  defaultOptions: {
    queries: { retry: false },
    mutations: { retry: false },
  },
});

const wrapper = ({ children }) => (
  <QueryClientProvider client={queryClient}>
    <BrowserRouter>
      {children}
    </BrowserRouter>
  </QueryClientProvider>
);

describe('ProductCard', () => {
  const mockProduct = {
    id: 1,
    name: 'Test Product',
    price: 10000,
    image: 'test.jpg',
    rating: 4.5,
    reviewCount: 10,
  };

  it('should render product information', () => {
    render(<ProductCard product={mockProduct} />, { wrapper });

    expect(screen.getByText('Test Product')).toBeInTheDocument();
    expect(screen.getByText('10,000원')).toBeInTheDocument();
    expect(screen.getByText('⭐ 4.5 (10)')).toBeInTheDocument();
  });

  it('should add product to cart when button clicked', () => {
    render(<ProductCard product={mockProduct} />, { wrapper });

    const addButton = screen.getByText('장바구니 담기');
    fireEvent.click(addButton);

    // 장바구니 스토어 확인
    const cartItems = useCartStore.getState().items;
    expect(cartItems).toHaveLength(1);
    expect(cartItems[0].id).toBe(mockProduct.id);
  });

  it('should toggle wishlist when heart button clicked', () => {
    render(<ProductCard product={mockProduct} />, { wrapper });

    const wishlistButton = screen.getByLabelText('위시리스트에 추가');

    // 첫 번째 클릭: 추가
    fireEvent.click(wishlistButton);
    expect(useWishlistStore.getState().isInWishlist(mockProduct.id)).toBe(true);

    // 두 번째 클릭: 제거
    fireEvent.click(wishlistButton);
    expect(useWishlistStore.getState().isInWishlist(mockProduct.id)).toBe(false);
  });
});
```

### 3. E2E 테스트 (End-to-End Tests)

```javascript
// e2e/checkout.spec.js (Playwright)
import { test, expect } from '@playwright/test';

test.describe('Checkout flow', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('http://localhost:3000');
  });

  test('should complete checkout successfully', async ({ page }) => {
    // 1. 상품 페이지로 이동
    await page.click('text=상품 보기');

    // 2. 첫 번째 상품 클릭
    await page.click('.product-card:first-child');

    // 3. 장바구니에 추가
    await page.click('text=장바구니 담기');

    // 4. 장바구니 드로어 확인
    await expect(page.locator('.cart-drawer')).toBeVisible();
    await expect(page.locator('.cart-item')).toHaveCount(1);

    // 5. 주문하기 버튼 클릭
    await page.click('text=주문하기');

    // 6. 로그인 (테스트 계정)
    await page.fill('input[name="email"]', 'test@example.com');
    await page.fill('input[name="password"]', 'password');
    await page.click('button[type="submit"]');

    // 7. 배송 정보 입력
    await page.fill('input[name="address"]', '서울시 강남구');
    await page.fill('input[name="phone"]', '010-1234-5678');

    // 8. 결제하기
    await page.click('text=결제하기');

    // 9. 주문 완료 확인
    await expect(page.locator('text=주문이 완료되었습니다')).toBeVisible();

    // 10. 장바구니 비어있는지 확인
    await page.click('[data-testid="cart-badge"]');
    await expect(page.locator('.cart-item')).toHaveCount(0);
  });
});
```

### 4. MSW로 API 모킹

```jsx
// mocks/handlers.js
import { rest } from 'msw';

export const handlers = [
  // 상품 목록 조회
  rest.get('/api/products', (req, res, ctx) => {
    const category = req.url.searchParams.get('category');
    const page = parseInt(req.url.searchParams.get('page') || '1', 10);

    return res(
      ctx.json({
        items: [
          { id: 1, name: 'Product 1', price: 10000 },
          { id: 2, name: 'Product 2', price: 20000 },
        ],
        totalPages: 5,
        currentPage: page,
      })
    );
  }),

  // 상품 상세 조회
  rest.get('/api/products/:id', (req, res, ctx) => {
    const { id } = req.params;

    return res(
      ctx.json({
        id: parseInt(id, 10),
        name: `Product ${id}`,
        price: 10000,
        description: 'Test product',
        images: ['test.jpg'],
      })
    );
  }),

  // 주문 생성
  rest.post('/api/orders', async (req, res, ctx) => {
    const orderData = await req.json();

    return res(
      ctx.json({
        id: Date.now(),
        ...orderData,
        status: 'pending',
        createdAt: new Date().toISOString(),
      })
    );
  }),
];
```

```jsx
// setupTests.js
import { setupServer } from 'msw/node';
import { handlers } from './mocks/handlers';

export const server = setupServer(...handlers);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

### 5. 테스트 커버리지 전략

**목표 커버리지**:
- **비즈니스 로직**: 90% 이상
- **UI 컴포넌트**: 70% 이상
- **유틸리티 함수**: 95% 이상

**우선순위**:
1. **Critical Path**: 로그인, 결제, 주문 생성
2. **비즈니스 로직**: Zustand 스토어, 계산 로직
3. **공통 컴포넌트**: Button, Input, Modal 등
4. **엣지 케이스**: 에러 처리, 빈 상태, 로딩 상태

## 모니터링 및 디버깅

### 1. Redux DevTools (Zustand)

```jsx
// Zustand와 Redux DevTools 연동
import create from 'zustand';
import { devtools } from 'zustand/middleware';

export const useCartStore = create(
  devtools(
    (set) => ({
      items: [],
      addItem: (item) => set((state) => ({
        items: [...state.items, item]
      })),
    }),
    { name: 'CartStore' } // DevTools에 표시될 이름
  )
);
```

### 2. React Query Devtools

```jsx
// App.jsx
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';

function App() {
  return (
    <>
      <YourApp />
      <ReactQueryDevtools initialIsOpen={false} />
    </>
  );
}
```

### 3. 에러 바운더리

```jsx
import { Component } from 'react';

export class ErrorBoundary extends Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    console.error('Error caught by boundary:', error, errorInfo);

    // 에러 로깅 서비스로 전송
    logErrorToService(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return (
        <div>
          <h1>문제가 발생했습니다</h1>
          <p>{this.state.error?.message}</p>
          <button onClick={() => window.location.reload()}>
            새로고침
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}

// 사용
function App() {
  return (
    <ErrorBoundary>
      <YourApp />
    </ErrorBoundary>
  );
}
```

### 4. 성능 모니터링

```jsx
// 성능 측정
import { useEffect } from 'react';

export function usePerformanceMonitor(componentName) {
  useEffect(() => {
    const startTime = performance.now();

    return () => {
      const endTime = performance.now();
      const renderTime = endTime - startTime;

      if (renderTime > 16) { // 60fps 기준
        console.warn(`${componentName} slow render: ${renderTime}ms`);
      }
    };
  });
}

// 사용
function ProductList() {
  usePerformanceMonitor('ProductList');

  return <div>...</div>;
}
```

## Best Practices

### 상태 관리 원칙 10가지

#### 1. Single Source of Truth

```jsx
// ❌ 나쁜 예: 중복된 진실 공급원
function BadExample() {
  const [user, setUser] = useState(null); // 로컬 상태
  const { user: globalUser } = useAuth(); // 전역 상태

  // 어떤 user를 믿어야 할까?
}

// ✅ 좋은 예: 단일 진실 공급원
function GoodExample() {
  const { user } = useAuth(); // 전역 상태만 사용

  // 명확함!
}
```

#### 2. 상태 콜로케이션 (Colocation)

```jsx
// ✅ 상태를 사용하는 곳 가까이에 배치
function ProductCard({ product }) {
  // 이 컴포넌트에서만 사용하는 상태는 여기에
  const [isExpanded, setIsExpanded] = useState(false);

  return (
    <div>
      <h3>{product.name}</h3>
      {isExpanded && <p>{product.description}</p>}
      <button onClick={() => setIsExpanded(!isExpanded)}>
        {isExpanded ? '접기' : '더보기'}
      </button>
    </div>
  );
}
```

#### 3. 파생 상태 최소화

```jsx
// ❌ 나쁜 예: 파생 상태를 별도로 관리
function BadExample({ items }) {
  const [total, setTotal] = useState(0);

  useEffect(() => {
    setTotal(items.reduce((sum, item) => sum + item.price, 0));
  }, [items]);

  return <div>Total: {total}</div>;
}

// ✅ 좋은 예: 렌더링 중 계산
function GoodExample({ items }) {
  const total = items.reduce((sum, item) => sum + item.price, 0);

  return <div>Total: {total}</div>;
}

// ✅ 계산이 복잡하면 useMemo
function BetterExample({ items }) {
  const total = useMemo(() =>
    items.reduce((sum, item) => sum + item.price, 0),
    [items]
  );

  return <div>Total: {total}</div>;
}
```

#### 4. 상태 정규화 (Normalization)

```jsx
// ❌ 나쁜 예: 중첩된 배열
const state = {
  categories: [
    {
      id: 1,
      name: 'Electronics',
      products: [
        { id: 101, name: 'Laptop' },
        { id: 102, name: 'Phone' },
      ],
    },
  ],
};

// 상품 업데이트가 어려움
// categories → products를 탐색해야 함

// ✅ 좋은 예: 정규화된 구조
const state = {
  categories: {
    1: { id: 1, name: 'Electronics', productIds: [101, 102] },
  },
  products: {
    101: { id: 101, name: 'Laptop', categoryId: 1 },
    102: { id: 102, name: 'Phone', categoryId: 1 },
  },
};

// 상품 업데이트가 쉬움
state.products[101] = { ...state.products[101], name: 'Gaming Laptop' };
```

#### 5. 불변성 유지

```jsx
// ❌ 나쁜 예: 직접 수정
const updateUser = (updates) => {
  user.name = updates.name; // 직접 수정
  setUser(user); // 리렌더링 안 됨!
};

// ✅ 좋은 예: 새 객체 생성
const updateUser = (updates) => {
  setUser(prev => ({ ...prev, ...updates }));
};

// ✅ Immer 사용 (복잡한 중첩 구조)
import produce from 'immer';

const updateNestedUser = (updates) => {
  setUser(produce(draft => {
    draft.profile.address.city = updates.city;
    draft.profile.address.zipCode = updates.zipCode;
  }));
};
```

#### 6. 비동기 상태 패턴

```jsx
// ✅ 로딩/에러 상태 관리 패턴
function useAsyncState(asyncFn) {
  const [state, setState] = useState({
    data: null,
    loading: false,
    error: null,
  });

  const execute = async (...args) => {
    setState({ data: null, loading: true, error: null });
    try {
      const data = await asyncFn(...args);
      setState({ data, loading: false, error: null });
      return { success: true, data };
    } catch (error) {
      setState({ data: null, loading: false, error });
      return { success: false, error };
    }
  };

  return { ...state, execute };
}

// 사용
function MyComponent() {
  const { data, loading, error, execute } = useAsyncState(fetchUser);

  useEffect(() => {
    execute(userId);
  }, [userId]);

  if (loading) return <Spinner />;
  if (error) return <Error message={error.message} />;
  return <UserProfile user={data} />;
}
```

#### 7. Context 과다 사용 방지

```jsx
// ❌ Context를 모든 곳에 사용
<UserContext>
  <ThemeContext>
    <LanguageContext>
      <SettingsContext>
        <NotificationContext>
          {/* 너무 많은 Provider */}
        </NotificationContext>
      </SettingsContext>
    </LanguageContext>
  </ThemeContext>
</UserContext>

// ✅ 필요한 곳에만 사용
// - 자주 변경되는 상태: Zustand
// - 서버 데이터: React Query
// - 전역 설정: Context (2-3개만)
```

#### 8. 액션 네이밍 컨벤션

```jsx
// ✅ 명확한 액션 이름
const ACTIONS = {
  // 명령형 동사
  ADD_ITEM: 'ADD_ITEM',
  REMOVE_ITEM: 'REMOVE_ITEM',
  UPDATE_QUANTITY: 'UPDATE_QUANTITY',

  // 과거형 (완료된 이벤트)
  ITEM_ADDED: 'ITEM_ADDED',
  ORDER_CREATED: 'ORDER_CREATED',

  // Request/Success/Failure 패턴
  FETCH_PRODUCTS_REQUEST: 'FETCH_PRODUCTS_REQUEST',
  FETCH_PRODUCTS_SUCCESS: 'FETCH_PRODUCTS_SUCCESS',
  FETCH_PRODUCTS_FAILURE: 'FETCH_PRODUCTS_FAILURE',
};
```

#### 9. 상태 초기화 전략

```jsx
// ✅ 상태 초기화 패턴
const initialState = {
  user: null,
  cart: [],
  wishlist: [],
};

// 리셋 함수 제공
export const useAppStore = create((set) => ({
  ...initialState,

  reset: () => set(initialState),

  resetCart: () => set({ cart: [] }),

  // 부분 리셋
  resetUserData: () => set({
    user: null,
    cart: [],
    wishlist: [],
  }),
}));
```

#### 10. 타입 안전성 (TypeScript)

```typescript
// ✅ TypeScript로 타입 안전성 확보
interface User {
  id: number;
  name: string;
  email: string;
}

interface CartItem {
  id: number;
  name: string;
  price: number;
  quantity: number;
}

interface CartStore {
  items: CartItem[];
  addItem: (product: Omit<CartItem, 'quantity'>) => void;
  removeItem: (productId: number) => void;
  updateQuantity: (productId: number, quantity: number) => void;
  clearCart: () => void;
  totalPrice: number;
  totalItems: number;
}

export const useCartStore = create<CartStore>((set, get) => ({
  items: [],

  addItem: (product) => set((state) => ({
    items: [...state.items, { ...product, quantity: 1 }]
  })),

  // ... 나머지 구현

  get totalPrice() {
    return get().items.reduce((sum, item) => sum + item.price * item.quantity, 0);
  },

  get totalItems() {
    return get().items.reduce((sum, item) => sum + item.quantity, 0);
  },
}));
```

## 마이그레이션 전략

기존 프로젝트를 새로운 아키텍처로 이전하는 방법:

### 1단계: 평가 및 계획

```
현재 상태 파악:
✓ 어떤 상태 관리 도구를 사용 중인가?
✓ 성능 병목 지점은 어디인가?
✓ 가장 큰 문제점은 무엇인가?

목표 설정:
✓ 해결하려는 주요 문제
✓ 성능 개선 목표
✓ 개발자 경험 개선 목표
```

### 2단계: 점진적 마이그레이션

```jsx
// 기존 코드 유지하면서 새 기능부터 적용
// 예: 새 기능은 Zustand + React Query 사용

// Old: Redux
function OldProductList() {
  const products = useSelector(state => state.products);
  const dispatch = useDispatch();

  // ...
}

// New: React Query (새 기능)
function NewProductList() {
  const { data: products } = useProducts();

  // ...
}

// 공존 가능!
```

### 3단계: 모듈별 전환

```
우선순위:
1. 서버 상태 → React Query로 전환
2. 전역 클라이언트 상태 → Zustand로 전환
3. Context 최적화
4. 로컬 상태 정리
```

### 4단계: 테스트 및 검증

```
각 모듈 전환 후:
✓ 기능 테스트
✓ 성능 측정
✓ 에러 모니터링
✓ 사용자 피드백
```

## FAQ

### Q1: 상태 관리 도구가 너무 많은데 어떻게 선택하나요?

**A**: 상태의 **성격**에 따라 선택하세요:

1. **서버에서 오는 데이터** → React Query / SWR
2. **앱 전체 클라이언트 상태** → Zustand / Redux Toolkit
3. **도메인별 상태** → Context API
4. **컴포넌트 로컬 상태** → useState / useReducer

**대부분의 앱은 이 조합이면 충분합니다**:
- React Query (서버 상태)
- Zustand (전역 클라이언트 상태)
- Context API (인증, 테마)
- useState (로컬 UI 상태)

### Q2: Redux가 필요한 경우는 언제인가요?

**A**: 다음 경우에 Redux Toolkit을 고려하세요:

- 팀이 Redux에 익숙함
- 매우 복잡한 상태 로직 (타임 트래블 디버깅 필요)
- 엄격한 상태 변경 추적 필요
- 대규모 팀 프로젝트 (표준화된 패턴)

**하지만**: 대부분의 경우 Zustand + React Query면 충분합니다.

### Q3: Context를 사용하면 성능이 나쁜가요?

**A**: Context 자체는 문제가 아닙니다. **잘못된 사용**이 문제입니다:

**문제가 되는 경우**:
- 자주 변경되는 값을 Context에 넣음
- 모든 상태를 하나의 Context에 넣음
- useMemo 없이 객체를 value로 전달

**해결책**:
- Context 분리 (User, Theme, Language 등)
- 자주 변경되는 상태는 Zustand 사용
- useMemo로 value 메모이제이션

### Q4: URL 상태는 언제 사용해야 하나요?

**A**: 다음 경우에 URL 상태를 사용하세요:

- 검색 쿼리
- 필터 옵션
- 페이지네이션
- 탭 선택
- 정렬 순서

**장점**:
- 공유 가능한 링크
- 북마크 가능
- 뒤로가기/앞으로가기 자동 지원
- SEO 친화적

### Q5: 장바구니는 어떻게 관리해야 하나요?

**A**: Zustand + persist 미들웨어 추천:

```jsx
import create from 'zustand';
import { persist } from 'zustand/middleware';

export const useCartStore = create(
  persist(
    (set) => ({
      items: [],
      addItem: (item) => set((state) => ({
        items: [...state.items, item]
      })),
    }),
    {
      name: 'cart-storage', // localStorage 키
    }
  )
);
```

**이유**:
- 전역 접근 가능
- 새로고침해도 유지
- 간단한 API
- 성능 우수

### Q6: 인증 상태는 어디에 두어야 하나요?

**A**: Context API 추천:

{% raw %}
```jsx
const AuthContext = createContext(null);

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);

  const login = async (credentials) => {
    const user = await authAPI.login(credentials);
    setUser(user);
  };

  const logout = () => {
    setUser(null);
    // 다른 상태들도 초기화
  };

  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}
```
{% endraw %}

**이유**:
- 앱 전체에서 필요
- 자주 변경되지 않음
- 로그아웃 시 다른 상태들 초기화 용이

### Q7: 상태가 너무 많아서 관리가 어렵습니다

**A**: 상태를 **카테고리별로 분류**하세요:

```
UI 상태 (로컬)
├── 모달 열림/닫힘
├── 사이드바 확장/축소
└── 툴팁, 드롭다운

도메인 상태 (전역)
├── 장바구니
├── 위시리스트
└── 사용자 설정

서버 상태 (React Query)
├── 상품 목록
├── 사용자 프로필
└── 주문 내역

URL 상태
├── 검색 쿼리
├── 필터
└── 페이지네이션
```

### Q8: React Query의 캐시 전략은 어떻게 설정하나요?

**A**: 데이터 성격에 따라 다릅니다:

```jsx
// 자주 변경되는 데이터 (실시간성 중요)
useQuery({
  queryKey: ['notifications'],
  queryFn: fetchNotifications,
  staleTime: 0, // 즉시 stale
  cacheTime: 5 * 60 * 1000, // 5분 캐시
  refetchInterval: 30 * 1000, // 30초마다 refetch
});

// 거의 변경되지 않는 데이터
useQuery({
  queryKey: ['categories'],
  queryFn: fetchCategories,
  staleTime: Infinity, // 항상 fresh
  cacheTime: Infinity, // 영구 캐시
});

// 일반적인 데이터
useQuery({
  queryKey: ['products'],
  queryFn: fetchProducts,
  staleTime: 5 * 60 * 1000, // 5분 fresh
  cacheTime: 10 * 60 * 1000, // 10분 캐시
});
```

## 결론

이 글에서는 **대규모 React 애플리케이션의 상태 관리 아키텍처**를 다뤘습니다. 4편의 시리즈를 통해 React 상태 관리의 모든 것을 살펴봤습니다.

### 시리즈 전체 요약

#### 1편: React 기본 상태 관리
- useState: 로컬 상태의 기본
- useReducer: 복잡한 상태 로직
- Context API: Props Drilling 해결

#### 2편: 전역 상태 관리 도구
- Zustand: 간단하고 강력한 전역 상태
- Redux Toolkit: 복잡한 앱을 위한 표준
- 선택 가이드: 언제 무엇을 사용할까

#### 3편: 서버 상태 관리
- React Query: 서버 상태의 정석
- 캐싱, 동기화, 최적화
- 실전 패턴과 Best Practices

#### 4편: 아키텍처 (이번 글)
- 상태 계층 설계 (4단계 피라미드)
- 도구 조합 패턴
- E-Commerce 앱 실전 예제
- 성능 최적화와 테스트 전략

### 핵심 원칙

**1. 올바른 도구 선택**
```
서버 데이터 → React Query
전역 클라이언트 상태 → Zustand
도메인별 상태 → Context API
로컬 UI 상태 → useState
공유 가능한 상태 → URL State
```

**2. 상태 계층 설계**
```
Level 4: Server State (React Query)
Level 3: Global State (Zustand)
Level 2: Context State (Context API)
Level 1: Local State (useState/useReducer)
```

**3. 성능 최적화**
- React.memo로 컴포넌트 메모이제이션
- Zustand Selector로 리렌더링 최적화
- React Query 캐싱 전략
- 코드 스플리팅과 가상화

**4. 테스트 전략**
- 단위 테스트: 스토어, 훅
- 통합 테스트: 컴포넌트
- E2E 테스트: 주요 플로우
- MSW로 API 모킹

### 실전 체크리스트

새 프로젝트 시작 시:
- [ ] 상태를 4가지 타입으로 분류 (UI/도메인/서버/URL)
- [ ] React Query 설정 (QueryClient)
- [ ] Zustand 스토어 구조 설계
- [ ] Context 최소화 (2-3개)
- [ ] 폴더 구조 정의 (Feature-First)
- [ ] 에러 바운더리 설정
- [ ] DevTools 설정
- [ ] 테스트 환경 구축

기존 프로젝트 개선 시:
- [ ] 성능 병목 지점 파악
- [ ] 서버 상태를 React Query로 전환
- [ ] Props Drilling 제거 (Context 또는 Zustand)
- [ ] 불필요한 리렌더링 최적화
- [ ] 테스트 커버리지 확보
- [ ] 모니터링 도구 추가

### 마지막 조언

**간단하게 시작하세요**
- 처음부터 모든 것을 완벽하게 만들려고 하지 마세요
- useState로 시작해서 필요할 때 리팩토링하세요
- 문제가 생기면 그때 해결하세요

**측정 후 최적화하세요**
- 성능 문제가 있을 때만 최적화하세요
- 추측하지 말고 측정하세요
- 과도한 최적화는 복잡성만 증가시킵니다

**팀과 소통하세요**
- 상태 관리 전략을 문서화하세요
- 컨벤션을 정하고 따르세요
- 코드 리뷰로 품질을 유지하세요

**계속 배우세요**
- React는 계속 진화합니다
- 새로운 패턴과 도구를 학습하세요
- 커뮤니티와 경험을 공유하세요

### 다음 단계

이제 여러분은 대규모 React 앱의 상태 관리를 마스터했습니다! 다음으로 학습할 주제들:

- **서버 컴포넌트 (React Server Components)**: Next.js 13+ App Router
- **Concurrent Features**: useTransition, useDeferredValue
- **Form 관리**: React Hook Form, Formik
- **실시간 데이터**: WebSocket, Server-Sent Events
- **오프라인 지원**: PWA, Service Workers

상태 관리 시리즈를 읽어주셔서 감사합니다! 궁금한 점이나 피드백이 있다면 댓글로 남겨주세요.

**Happy Coding!** 🚀

## 참고 자료

- [React 공식 문서 - Managing State](https://react.dev/learn/managing-state)
- [React 공식 문서 - Scaling Up with Reducer and Context](https://react.dev/learn/scaling-up-with-reducer-and-context)
- [Zustand 공식 문서](https://zustand-demo.pmnd.rs/)
- [TanStack Query (React Query) 공식 문서](https://tanstack.com/query/latest)
- [Redux Toolkit 공식 문서](https://redux-toolkit.js.org/)
- [React Router 공식 문서](https://reactrouter.com/)
- [Testing Library 공식 문서](https://testing-library.com/react)
- [MSW (Mock Service Worker) 공식 문서](https://mswjs.io/)
- [Web.dev - State Management Best Practices](https://web.dev/)
- [Kent C. Dodds - Application State Management](https://kentcdodds.com/blog/application-state-management-with-react)
- [Tanner Linsley - React Query 설계 철학](https://tkdodo.eu/blog/practical-react-query)
- [Daishi Kato - Zustand 내부 동작](https://blog.axlight.com/posts/zustand-internals/)
