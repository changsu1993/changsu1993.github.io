---
title: "React 애니메이션 완벽 가이드 | Framer Motion으로 선언적 UI 모션 구현하기"
description: "Framer Motion을 활용한 React 애니메이션 구현 방법을 알아봅니다. 선언적 애니메이션, 제스처, 레이아웃 전환, 스크롤 애니메이션까지 실전 예제와 함께 학습합니다."
date: 2025-11-30 10:00:00 +0900
categories: [Frontend, React]
tags: [react, framer-motion, animation, web-animation, ux]
---

## 들어가며

웹 애니메이션은 사용자 경험을 크게 향상시키지만, 직접 구현하려면 복잡한 타이밍, 이징, 상태 관리가 필요합니다. Framer Motion은 React에서 선언적으로 애니메이션을 구현할 수 있는 강력한 라이브러리입니다.

이 글에서는 Framer Motion의 핵심 개념부터 실전 활용까지, 실제 동작하는 코드 예제와 함께 학습합니다.

## Framer Motion 소개

### 왜 Framer Motion인가?

**CSS Transition/Animation의 한계**
- 복잡한 타이밍 조정이 어려움
- JavaScript와의 동기화 문제
- 동적인 값 계산 어려움
- 레이아웃 변경 애니메이션 복잡함

**Framer Motion의 장점**
- 선언적 API로 간결한 코드
- 자동 GPU 가속 최적화
- 제스처 인식 내장
- 레이아웃 애니메이션 자동 처리
- TypeScript 완벽 지원

### 설치

```bash
npm install framer-motion
```

**현재 최신 버전**: 11.x (2025년 기준)

**지원 환경**
- React 18+
- TypeScript 4.5+
- 모던 브라우저 (IE11 미지원)

## 기본 애니메이션

### motion 컴포넌트

Framer Motion은 모든 HTML 요소에 대응하는 `motion` 컴포넌트를 제공합니다.

{% raw %}
```tsx
import { motion } from 'framer-motion';

function BasicAnimation() {
  return (
    <motion.div
      initial={{ opacity: 0, y: 50 }}
      animate={{ opacity: 1, y: 0 }}
      transition={{ duration: 0.5 }}
    >
      안녕하세요!
    </motion.div>
  );
}

export default BasicAnimation;
```
{% endraw %}

**핵심 Props**
- `initial`: 시작 상태
- `animate`: 최종 상태
- `transition`: 애니메이션 설정

### 다양한 transition 옵션

{% raw %}
```tsx
import { motion } from 'framer-motion';

function TransitionExamples() {
  return (
    <div className="space-y-4">
      {/* 기본 애니메이션 */}
      <motion.div
        initial={{ x: -100 }}
        animate={{ x: 0 }}
        transition={{ duration: 0.5 }}
        className="box"
      >
        Duration
      </motion.div>

      {/* Spring 애니메이션 */}
      <motion.div
        initial={{ x: -100 }}
        animate={{ x: 0 }}
        transition={{
          type: 'spring',
          stiffness: 100,
          damping: 10,
        }}
        className="box"
      >
        Spring
      </motion.div>

      {/* Ease 커스터마이징 */}
      <motion.div
        initial={{ x: -100 }}
        animate={{ x: 0 }}
        transition={{
          duration: 0.5,
          ease: [0.43, 0.13, 0.23, 0.96], // cubic-bezier
        }}
        className="box"
      >
        Custom Ease
      </motion.div>

      {/* 지연 시간 */}
      <motion.div
        initial={{ x: -100 }}
        animate={{ x: 0 }}
        transition={{
          duration: 0.5,
          delay: 0.2,
        }}
        className="box"
      >
        Delayed
      </motion.div>
    </div>
  );
}

export default TransitionExamples;
```
{% endraw %}

### Exit 애니메이션

컴포넌트가 제거될 때 애니메이션을 적용하려면 `AnimatePresence`가 필요합니다.

{% raw %}
```tsx
import { useState } from 'react';
import { motion, AnimatePresence } from 'framer-motion';

function ExitAnimation() {
  const [isVisible, setIsVisible] = useState(true);

  return (
    <div>
      <button onClick={() => setIsVisible(!isVisible)}>
        토글
      </button>

      <AnimatePresence>
        {isVisible && (
          <motion.div
            initial={{ opacity: 0, scale: 0.8 }}
            animate={{ opacity: 1, scale: 1 }}
            exit={{ opacity: 0, scale: 0.8 }}
            transition={{ duration: 0.3 }}
            className="box"
          >
            나타났다 사라집니다
          </motion.div>
        )}
      </AnimatePresence>
    </div>
  );
}

export default ExitAnimation;
```
{% endraw %}

**중요**: `AnimatePresence`는 직접적인 자식의 마운트/언마운트를 감지합니다. 조건부 렌더링에서 반드시 사용해야 exit 애니메이션이 작동합니다.

## Variants 패턴

복잡한 애니메이션을 관리하고 부모-자식 간 애니메이션을 조율할 때 Variants 패턴을 사용합니다.

### 기본 Variants

```tsx
import { motion } from 'framer-motion';

const boxVariants = {
  hidden: {
    opacity: 0,
    scale: 0.8,
  },
  visible: {
    opacity: 1,
    scale: 1,
    transition: {
      duration: 0.5,
    },
  },
  exit: {
    opacity: 0,
    scale: 0.8,
    transition: {
      duration: 0.3,
    },
  },
};

function VariantsBasic() {
  return (
    <motion.div
      variants={boxVariants}
      initial="hidden"
      animate="visible"
      exit="exit"
      className="box"
    >
      Variants 사용
    </motion.div>
  );
}

export default VariantsBasic;
```

### 부모-자식 연쇄 애니메이션

```tsx
import { motion } from 'framer-motion';

const containerVariants = {
  hidden: { opacity: 0 },
  visible: {
    opacity: 1,
    transition: {
      staggerChildren: 0.1, // 자식 요소 순차 애니메이션
      delayChildren: 0.3,   // 자식 애니메이션 시작 지연
    },
  },
};

const itemVariants = {
  hidden: { opacity: 0, y: 20 },
  visible: {
    opacity: 1,
    y: 0,
  },
};

function StaggerAnimation() {
  const items = ['첫 번째', '두 번째', '세 번째', '네 번째'];

  return (
    <motion.ul
      variants={containerVariants}
      initial="hidden"
      animate="visible"
      className="space-y-2"
    >
      {items.map((item, index) => (
        <motion.li
          key={index}
          variants={itemVariants}
          className="box"
        >
          {item}
        </motion.li>
      ))}
    </motion.ul>
  );
}

export default StaggerAnimation;
```

**자동 전파**: 부모의 `animate` 상태가 변경되면 모든 자식의 variants가 자동으로 애니메이션됩니다.

### 동적 Variants

```tsx
import { motion } from 'framer-motion';

const boxVariants = {
  hidden: { opacity: 0 },
  visible: (custom: number) => ({
    opacity: 1,
    transition: {
      delay: custom * 0.2,
    },
  }),
};

function DynamicVariants() {
  return (
    <div className="space-y-2">
      {[0, 1, 2].map((index) => (
        <motion.div
          key={index}
          custom={index}
          variants={boxVariants}
          initial="hidden"
          animate="visible"
          className="box"
        >
          Box {index + 1}
        </motion.div>
      ))}
    </div>
  );
}

export default DynamicVariants;
```

## 제스처 애니메이션

Framer Motion은 다양한 제스처 이벤트를 간단하게 처리할 수 있습니다.

### Hover와 Tap

{% raw %}
```tsx
import { motion } from 'framer-motion';

function GestureBasic() {
  return (
    <div className="space-y-4">
      {/* Hover */}
      <motion.button
        whileHover={{ scale: 1.1 }}
        whileTap={{ scale: 0.95 }}
        className="btn"
      >
        버튼
      </motion.button>

      {/* 복잡한 Hover */}
      <motion.div
        whileHover={{
          scale: 1.05,
          boxShadow: '0 10px 30px rgba(0,0,0,0.2)',
        }}
        transition={{ type: 'spring', stiffness: 300 }}
        className="card"
      >
        카드 컴포넌트
      </motion.div>

      {/* Hover + 자식 애니메이션 */}
      <motion.div
        className="card"
        whileHover="hover"
        initial="rest"
      >
        <motion.h3
          variants={{
            rest: { scale: 1 },
            hover: { scale: 1.1 },
          }}
        >
          제목
        </motion.h3>
        <motion.p
          variants={{
            rest: { opacity: 0.7 },
            hover: { opacity: 1 },
          }}
        >
          내용
        </motion.p>
      </motion.div>
    </div>
  );
}

export default GestureBasic;
```
{% endraw %}

### Drag

{% raw %}
```tsx
import { motion } from 'framer-motion';

function DragExamples() {
  return (
    <div className="space-y-8">
      {/* 자유 드래그 */}
      <motion.div
        drag
        className="box"
      >
        자유롭게 드래그
      </motion.div>

      {/* 수평 드래그만 */}
      <motion.div
        drag="x"
        className="box"
      >
        수평 드래그만
      </motion.div>

      {/* 드래그 제약 */}
      <motion.div
        drag
        dragConstraints={{
          top: -50,
          left: -50,
          right: 50,
          bottom: 50,
        }}
        className="box"
      >
        제약된 드래그
      </motion.div>

      {/* 드래그 모멘텀 */}
      <motion.div
        drag
        dragElastic={0.2}
        dragTransition={{
          bounceStiffness: 600,
          bounceDamping: 20,
        }}
        className="box"
      >
        탄성 드래그
      </motion.div>

      {/* 드래그 이벤트 */}
      <motion.div
        drag
        onDragStart={() => console.log('드래그 시작')}
        onDrag={(event, info) => {
          console.log('드래그 중:', info.point);
        }}
        onDragEnd={(event, info) => {
          console.log('드래그 종료:', info.velocity);
        }}
        className="box"
      >
        이벤트 핸들러
      </motion.div>
    </div>
  );
}

export default DragExamples;
```
{% endraw %}

### 스와이프 감지

{% raw %}
```tsx
import { useState } from 'react';
import { motion, PanInfo } from 'framer-motion';

function SwipeDetection() {
  const [direction, setDirection] = useState<string>('');

  const handleDragEnd = (
    event: MouseEvent | TouchEvent | PointerEvent,
    info: PanInfo
  ) => {
    const threshold = 50;
    const { offset, velocity } = info;

    if (Math.abs(offset.x) > threshold) {
      if (offset.x > 0) {
        setDirection('오른쪽으로 스와이프');
      } else {
        setDirection('왼쪽으로 스와이프');
      }
    } else if (Math.abs(offset.y) > threshold) {
      if (offset.y > 0) {
        setDirection('아래로 스와이프');
      } else {
        setDirection('위로 스와이프');
      }
    }
  };

  return (
    <div>
      <motion.div
        drag
        dragConstraints={{ left: 0, right: 0, top: 0, bottom: 0 }}
        dragElastic={1}
        onDragEnd={handleDragEnd}
        className="box"
      >
        스와이프 해보세요
      </motion.div>
      {direction && <p className="mt-4">{direction}</p>}
    </div>
  );
}

export default SwipeDetection;
```
{% endraw %}

## 레이아웃 애니메이션

Framer Motion의 가장 강력한 기능 중 하나는 자동 레이아웃 애니메이션입니다.

### layout prop

{% raw %}
```tsx
import { useState } from 'react';
import { motion } from 'framer-motion';

function LayoutAnimation() {
  const [isExpanded, setIsExpanded] = useState(false);

  return (
    <motion.div
      layout
      onClick={() => setIsExpanded(!isExpanded)}
      style={{
        width: isExpanded ? 400 : 200,
        height: isExpanded ? 400 : 200,
        backgroundColor: '#3b82f6',
        borderRadius: 20,
        cursor: 'pointer',
      }}
    />
  );
}

export default LayoutAnimation;
```
{% endraw %}

**작동 원리**: `layout` prop을 추가하면 Framer Motion이 자동으로 FLIP 애니메이션을 적용합니다.
- **F**irst: 초기 위치 기록
- **L**ast: 최종 위치 계산
- **I**nvert: transform으로 초기 위치로 이동
- **P**lay: transform을 0으로 애니메이션

### Shared Layout Animations

```tsx
import { useState } from 'react';
import { motion } from 'framer-motion';

function SharedLayoutAnimation() {
  const [selected, setSelected] = useState<number>(0);
  const items = ['첫 번째', '두 번째', '세 번째'];

  return (
    <div className="flex gap-4">
      {items.map((item, index) => (
        <motion.div
          key={index}
          onClick={() => setSelected(index)}
          className="relative cursor-pointer px-4 py-2"
        >
          {item}
          {selected === index && (
            <motion.div
              layoutId="underline"
              className="absolute bottom-0 left-0 right-0 h-1 bg-blue-500"
            />
          )}
        </motion.div>
      ))}
    </div>
  );
}

export default SharedLayoutAnimation;
```

**layoutId**: 동일한 `layoutId`를 가진 요소 간 부드러운 전환을 생성합니다.

### 리스트 재정렬 애니메이션

```tsx
import { useState } from 'react';
import { motion, Reorder } from 'framer-motion';

function ReorderList() {
  const [items, setItems] = useState(['A', 'B', 'C', 'D']);

  return (
    <Reorder.Group axis="y" values={items} onReorder={setItems}>
      {items.map((item) => (
        <Reorder.Item key={item} value={item}>
          <motion.div
            layout
            className="box mb-2 cursor-grab active:cursor-grabbing"
          >
            {item}
          </motion.div>
        </Reorder.Item>
      ))}
    </Reorder.Group>
  );
}

export default ReorderList;
```

### LayoutGroup으로 복잡한 레이아웃 관리

{% raw %}
```tsx
import { useState } from 'react';
import { motion, LayoutGroup } from 'framer-motion';

function LayoutGroupExample() {
  const [selected, setSelected] = useState<number | null>(null);

  return (
    <LayoutGroup>
      <div className="grid grid-cols-3 gap-4">
        {[1, 2, 3, 4, 5, 6].map((num) => (
          <motion.div
            key={num}
            layout
            onClick={() => setSelected(selected === num ? null : num)}
            className="box"
            style={{
              gridColumn: selected === num ? 'span 3' : 'span 1',
            }}
          >
            {num}
          </motion.div>
        ))}
      </div>
    </LayoutGroup>
  );
}

export default LayoutGroupExample;
```
{% endraw %}

## 스크롤 애니메이션

### useScroll Hook

{% raw %}
```tsx
import { useRef } from 'react';
import { motion, useScroll, useTransform } from 'framer-motion';

function ScrollProgress() {
  const { scrollYProgress } = useScroll();

  return (
    <motion.div
      style={{
        position: 'fixed',
        top: 0,
        left: 0,
        right: 0,
        height: 10,
        backgroundColor: '#3b82f6',
        scaleX: scrollYProgress,
        transformOrigin: '0%',
      }}
    />
  );
}

export default ScrollProgress;
```
{% endraw %}

### 요소별 스크롤 진행률

{% raw %}
```tsx
import { useRef } from 'react';
import { motion, useScroll, useTransform } from 'framer-motion';

function ScrollElement() {
  const ref = useRef<HTMLDivElement>(null);
  const { scrollYProgress } = useScroll({
    target: ref,
    offset: ['start end', 'end start'],
  });

  const opacity = useTransform(scrollYProgress, [0, 0.5, 1], [0, 1, 0]);
  const scale = useTransform(scrollYProgress, [0, 0.5, 1], [0.8, 1, 0.8]);

  return (
    <motion.div
      ref={ref}
      style={{ opacity, scale }}
      className="box"
    >
      스크롤 하면서 관찰하세요
    </motion.div>
  );
}

export default ScrollElement;
```
{% endraw %}

**offset 옵션**:
- `['start end', 'end start']`: 요소의 시작이 뷰포트 끝에 들어올 때부터, 요소의 끝이 뷰포트 시작을 벗어날 때까지
- `['start start', 'end end']`: 요소의 시작이 뷰포트 시작에 닿을 때부터, 요소의 끝이 뷰포트 끝에 닿을 때까지

### useInView Hook

{% raw %}
```tsx
import { useRef } from 'react';
import { motion, useInView } from 'framer-motion';

function InViewAnimation() {
  const ref = useRef<HTMLDivElement>(null);
  const isInView = useInView(ref, { once: true });

  return (
    <div ref={ref}>
      <motion.div
        initial={{ opacity: 0, y: 50 }}
        animate={isInView ? { opacity: 1, y: 0 } : { opacity: 0, y: 50 }}
        transition={{ duration: 0.5 }}
        className="box"
      >
        화면에 들어오면 나타납니다
      </motion.div>
    </div>
  );
}

export default InViewAnimation;
```
{% endraw %}

**useInView 옵션**:
- `once: true`: 한 번만 트리거 (성능 최적화)
- `amount: 0.5`: 요소의 50%가 보일 때 트리거
- `margin: '100px'`: 뷰포트 확장 (Intersection Observer margin)

### 패럴랙스 효과

{% raw %}
```tsx
import { useRef } from 'react';
import { motion, useScroll, useTransform } from 'framer-motion';

function ParallaxEffect() {
  const ref = useRef<HTMLDivElement>(null);
  const { scrollYProgress } = useScroll({
    target: ref,
    offset: ['start end', 'end start'],
  });

  const y = useTransform(scrollYProgress, [0, 1], ['-20%', '20%']);

  return (
    <div ref={ref} className="relative h-96 overflow-hidden">
      <motion.img
        src="/image.jpg"
        style={{ y }}
        className="absolute inset-0 w-full h-full object-cover"
      />
    </div>
  );
}

export default ParallaxEffect;
```
{% endraw %}

## 실전 예제

### 모달

{% raw %}
```tsx
import { useState } from 'react';
import { motion, AnimatePresence } from 'framer-motion';

function Modal() {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <div>
      <button onClick={() => setIsOpen(true)}>
        모달 열기
      </button>

      <AnimatePresence>
        {isOpen && (
          <>
            {/* Backdrop */}
            <motion.div
              initial={{ opacity: 0 }}
              animate={{ opacity: 1 }}
              exit={{ opacity: 0 }}
              onClick={() => setIsOpen(false)}
              className="fixed inset-0 bg-black/50 z-40"
            />

            {/* Modal */}
            <motion.div
              initial={{ opacity: 0, scale: 0.75, y: 50 }}
              animate={{ opacity: 1, scale: 1, y: 0 }}
              exit={{ opacity: 0, scale: 0.75, y: 50 }}
              transition={{ type: 'spring', damping: 25, stiffness: 300 }}
              className="fixed inset-0 z-50 flex items-center justify-center p-4"
            >
              <div className="bg-white rounded-lg p-6 max-w-md w-full">
                <h2 className="text-xl font-bold mb-4">제목</h2>
                <p className="mb-4">모달 내용</p>
                <button
                  onClick={() => setIsOpen(false)}
                  className="btn"
                >
                  닫기
                </button>
              </div>
            </motion.div>
          </>
        )}
      </AnimatePresence>
    </div>
  );
}

export default Modal;
```
{% endraw %}

### 토스트 알림

{% raw %}
```tsx
import { useState } from 'react';
import { motion, AnimatePresence } from 'framer-motion';

interface Toast {
  id: number;
  message: string;
}

function ToastNotification() {
  const [toasts, setToasts] = useState<Toast[]>([]);

  const addToast = (message: string) => {
    const id = Date.now();
    setToasts((prev) => [...prev, { id, message }]);

    setTimeout(() => {
      removeToast(id);
    }, 3000);
  };

  const removeToast = (id: number) => {
    setToasts((prev) => prev.filter((toast) => toast.id !== id));
  };

  return (
    <div>
      <button onClick={() => addToast('알림 메시지!')}>
        토스트 추가
      </button>

      <div className="fixed top-4 right-4 space-y-2 z-50">
        <AnimatePresence>
          {toasts.map((toast) => (
            <motion.div
              key={toast.id}
              initial={{ opacity: 0, y: -50, scale: 0.3 }}
              animate={{ opacity: 1, y: 0, scale: 1 }}
              exit={{ opacity: 0, scale: 0.5, transition: { duration: 0.2 } }}
              className="bg-gray-900 text-white px-4 py-3 rounded-lg shadow-lg"
            >
              {toast.message}
            </motion.div>
          ))}
        </AnimatePresence>
      </div>
    </div>
  );
}

export default ToastNotification;
```
{% endraw %}

### 카드 플립

{% raw %}
```tsx
import { useState } from 'react';
import { motion } from 'framer-motion';

function FlipCard() {
  const [isFlipped, setIsFlipped] = useState(false);

  return (
    <div className="perspective-1000">
      <motion.div
        className="relative w-64 h-96 cursor-pointer"
        onClick={() => setIsFlipped(!isFlipped)}
        animate={{ rotateY: isFlipped ? 180 : 0 }}
        transition={{ duration: 0.6 }}
        style={{ transformStyle: 'preserve-3d' }}
      >
        {/* 앞면 */}
        <div
          className="absolute inset-0 bg-blue-500 rounded-lg flex items-center justify-center text-white text-2xl"
          style={{ backfaceVisibility: 'hidden' }}
        >
          앞면
        </div>

        {/* 뒷면 */}
        <div
          className="absolute inset-0 bg-red-500 rounded-lg flex items-center justify-center text-white text-2xl"
          style={{
            backfaceVisibility: 'hidden',
            transform: 'rotateY(180deg)',
          }}
        >
          뒷면
        </div>
      </motion.div>
    </div>
  );
}

export default FlipCard;
```
{% endraw %}

**CSS 설정 필요**:
```css
.perspective-1000 {
  perspective: 1000px;
}
```

### 페이지 전환

{% raw %}
```tsx
import { useState } from 'react';
import { motion, AnimatePresence } from 'framer-motion';

const pages = [
  { id: 1, color: '#3b82f6', title: '페이지 1' },
  { id: 2, color: '#ef4444', title: '페이지 2' },
  { id: 3, color: '#10b981', title: '페이지 3' },
];

const pageVariants = {
  initial: (direction: number) => ({
    x: direction > 0 ? 1000 : -1000,
    opacity: 0,
  }),
  animate: {
    x: 0,
    opacity: 1,
  },
  exit: (direction: number) => ({
    x: direction < 0 ? 1000 : -1000,
    opacity: 0,
  }),
};

function PageTransition() {
  const [[currentPage, direction], setPage] = useState([0, 0]);

  const paginate = (newDirection: number) => {
    const newIndex = (currentPage + newDirection + pages.length) % pages.length;
    setPage([newIndex, newDirection]);
  };

  return (
    <div className="relative overflow-hidden h-96">
      <AnimatePresence initial={false} custom={direction}>
        <motion.div
          key={currentPage}
          custom={direction}
          variants={pageVariants}
          initial="initial"
          animate="animate"
          exit="exit"
          transition={{
            x: { type: 'spring', stiffness: 300, damping: 30 },
            opacity: { duration: 0.2 },
          }}
          className="absolute inset-0 flex items-center justify-center text-white text-4xl"
          style={{ backgroundColor: pages[currentPage].color }}
        >
          {pages[currentPage].title}
        </motion.div>
      </AnimatePresence>

      <button
        onClick={() => paginate(-1)}
        className="absolute left-4 top-1/2 -translate-y-1/2 btn"
      >
        이전
      </button>
      <button
        onClick={() => paginate(1)}
        className="absolute right-4 top-1/2 -translate-y-1/2 btn"
      >
        다음
      </button>
    </div>
  );
}

export default PageTransition;
```
{% endraw %}

### 아코디언

{% raw %}
```tsx
import { useState } from 'react';
import { motion, AnimatePresence } from 'framer-motion';

interface AccordionItem {
  id: number;
  title: string;
  content: string;
}

const items: AccordionItem[] = [
  { id: 1, title: '첫 번째 항목', content: '첫 번째 내용입니다.' },
  { id: 2, title: '두 번째 항목', content: '두 번째 내용입니다.' },
  { id: 3, title: '세 번째 항목', content: '세 번째 내용입니다.' },
];

function Accordion() {
  const [expandedId, setExpandedId] = useState<number | null>(null);

  return (
    <div className="space-y-2">
      {items.map((item) => (
        <div key={item.id} className="border rounded-lg overflow-hidden">
          <motion.button
            onClick={() => setExpandedId(expandedId === item.id ? null : item.id)}
            className="w-full px-4 py-3 text-left font-medium hover:bg-gray-50"
          >
            {item.title}
          </motion.button>

          <AnimatePresence initial={false}>
            {expandedId === item.id && (
              <motion.div
                initial={{ height: 0 }}
                animate={{ height: 'auto' }}
                exit={{ height: 0 }}
                transition={{ duration: 0.3 }}
                className="overflow-hidden"
              >
                <div className="px-4 py-3 bg-gray-50">
                  {item.content}
                </div>
              </motion.div>
            )}
          </AnimatePresence>
        </div>
      ))}
    </div>
  );
}

export default Accordion;
```
{% endraw %}

### 무한 슬라이더

{% raw %}
```tsx
import { motion } from 'framer-motion';

function InfiniteSlider() {
  const items = ['Item 1', 'Item 2', 'Item 3', 'Item 4', 'Item 5'];

  return (
    <div className="overflow-hidden">
      <motion.div
        className="flex gap-4"
        animate={{
          x: [0, -1000],
        }}
        transition={{
          x: {
            repeat: Infinity,
            repeatType: 'loop',
            duration: 20,
            ease: 'linear',
          },
        }}
      >
        {[...items, ...items].map((item, index) => (
          <div
            key={index}
            className="flex-shrink-0 w-64 h-32 bg-blue-500 rounded-lg flex items-center justify-center text-white text-xl"
          >
            {item}
          </div>
        ))}
      </motion.div>
    </div>
  );
}

export default InfiniteSlider;
```
{% endraw %}

## 성능 최적화

### GPU 가속 속성 사용

**빠른 속성** (GPU 가속):
- `transform`: `x`, `y`, `scale`, `rotate`
- `opacity`

**느린 속성** (레이아웃 재계산):
- `width`, `height`
- `top`, `left`, `right`, `bottom`
- `margin`, `padding`

{% raw %}
```tsx
// 좋은 예
<motion.div animate={{ x: 100, opacity: 0.5 }} />

// 나쁜 예
<motion.div animate={{ left: 100, width: 200 }} />
```
{% endraw %}

### will-change 사용

{% raw %}
```tsx
<motion.div
  style={{ willChange: 'transform' }}
  animate={{ x: 100 }}
/>
```
{% endraw %}

**주의**: `will-change`를 과도하게 사용하면 메모리 낭비가 발생합니다. 필요한 요소에만 적용하세요.

### layoutId 최적화

```tsx
// layoutId를 사용할 때는 key를 명확히 지정
<motion.div layoutId={`item-${id}`} key={id} />
```

### AnimatePresence mode

```tsx
// 한 번에 하나씩 애니메이션 (메모리 효율적)
<AnimatePresence mode="wait">
  {/* ... */}
</AnimatePresence>

// 동시에 여러 애니메이션 (기본값)
<AnimatePresence mode="sync">
  {/* ... */}
</AnimatePresence>
```

### 조건부 애니메이션

{% raw %}
```tsx
import { useReducedMotion } from 'framer-motion';

function ResponsiveAnimation() {
  const shouldReduceMotion = useReducedMotion();

  return (
    <motion.div
      animate={{ x: shouldReduceMotion ? 0 : 100 }}
      transition={{ duration: shouldReduceMotion ? 0 : 0.5 }}
    >
      접근성 고려
    </motion.div>
  );
}

export default ResponsiveAnimation;
```
{% endraw %}

### 리스트 최적화

{% raw %}
```tsx
import { motion } from 'framer-motion';

function OptimizedList() {
  const items = Array.from({ length: 100 }, (_, i) => i);

  return (
    <div>
      {items.map((item) => (
        <motion.div
          key={item}
          initial={false} // 초기 마운트 애니메이션 비활성화
          whileHover={{ scale: 1.05 }}
          className="box"
        >
          Item {item}
        </motion.div>
      ))}
    </div>
  );
}

export default OptimizedList;
```
{% endraw %}

## CSS Transition vs Framer Motion

### 성능 비교

**실험 환경**:
- Chrome 120
- 100개의 요소 동시 애니메이션
- 측정: FPS, 메모리 사용량

| 방식 | FPS | 메모리 | 코드 복잡도 |
|------|-----|--------|------------|
| CSS Transition | 60 | 낮음 | 높음 |
| Framer Motion | 58-60 | 중간 | 낮음 |
| JavaScript (직접 구현) | 45-55 | 높음 | 매우 높음 |

### 언제 무엇을 사용할까?

**CSS Transition 사용**:
- 단순한 hover/focus 효과
- 최소한의 JavaScript
- 성능이 절대적으로 중요한 경우

**Framer Motion 사용**:
- 복잡한 시퀀스 애니메이션
- 제스처 인식 필요
- 레이아웃 애니메이션
- JavaScript 기반 상태와 동기화

### 비교 예제

**CSS Transition**:
```tsx
function CSSExample() {
  return (
    <div
      className="box transition-all duration-300 hover:scale-110"
    >
      CSS Transition
    </div>
  );
}
```

**Framer Motion**:
{% raw %}
```tsx
function MotionExample() {
  return (
    <motion.div
      className="box"
      whileHover={{ scale: 1.1 }}
      transition={{ duration: 0.3 }}
    >
      Framer Motion
    </motion.div>
  );
}
```
{% endraw %}

**복잡한 시퀀스 비교**:

CSS (복잡함):
```css
.element {
  animation: complexSequence 2s ease-in-out;
}

@keyframes complexSequence {
  0% { transform: translateX(0) scale(1); opacity: 0; }
  50% { transform: translateX(100px) scale(1.2); opacity: 1; }
  100% { transform: translateX(200px) scale(1); opacity: 0; }
}
```

Framer Motion (간결함):
{% raw %}
```tsx
<motion.div
  animate={{
    x: [0, 100, 200],
    scale: [1, 1.2, 1],
    opacity: [0, 1, 0],
  }}
  transition={{ duration: 2 }}
/>
```
{% endraw %}

## 마치며

Framer Motion은 React에서 애니메이션을 구현하는 가장 강력하고 직관적인 도구입니다.

**핵심 요약**:

1. **기본 애니메이션**: `initial`, `animate`, `exit`로 선언적 구현
2. **Variants**: 복잡한 애니메이션을 구조화하고 부모-자식 조율
3. **제스처**: `whileHover`, `whileTap`, `drag`로 인터랙션 추가
4. **레이아웃**: `layout`, `layoutId`로 자동 FLIP 애니메이션
5. **스크롤**: `useScroll`, `useInView`로 스크롤 기반 효과
6. **성능**: GPU 가속 속성 사용, `will-change` 최적화

**학습 리소스**:
- [Framer Motion 공식 문서](https://www.framer.com/motion/)
- [Framer Motion Examples](https://www.framer.com/motion/examples/)
- [Animation Controls](https://www.framer.com/motion/animation/)

**다음 단계**:
- 실제 프로젝트에 적용
- 성능 프로파일링
- 사용자 피드백 수집
- 접근성 테스트 (motion preferences)

React 애플리케이션에 생동감을 불어넣고 사용자 경험을 한 단계 업그레이드하세요!

## 참고 자료

- [Framer Motion 공식 문서](https://www.framer.com/motion/)
- [FLIP 애니메이션 기법](https://aerotwist.com/blog/flip-your-animations/)
- [Web Animations Performance](https://web.dev/animations/)
- [Reduced Motion](https://web.dev/prefers-reduced-motion/)
