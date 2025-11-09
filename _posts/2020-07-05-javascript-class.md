---
title: JavaScript Class 완벽 가이드 - 객체지향 프로그래밍의 핵심
date: 2020-07-05 01:07:00 +0900
categories: [개발, JavaScript]
tags: [javascript, class, oop, es6, constructor, 객체지향]
author: changsu
toc: true
comments: true
description: JavaScript ES6 Class의 개념부터 활용까지. Constructor, Instance, Method를 실전 예제와 함께 알아봅니다. 객체지향 프로그래밍(OOP)의 핵심 개념을 쉽고 명확하게 설명하며, 클래스를 활용한 실무 코드 작성법을 단계별로 학습합니다. 자동차 판매 시스템 예제로 클래스의 활용을 익힙니다.
---

## 들어가며

클래스는 객체지향 프로그래밍(OOP)의 핵심 개념입니다.

JavaScript는 프로토타입 기반 언어지만, ES6부터 `class` 문법을 지원하면서 더욱 직관적으로 객체지향 프로그래밍을 할 수 있게 되었습니다.

## 객체지향 프로그래밍이란?

**객체지향 프로그래밍**(Object-Oriented Programming)은 프로그램을 객체들로 구성하고, 객체들 간에 서로 상호작용하도록 작성하는 방법입니다.

> 여기서 "객체"는 단순한 데이터 타입 `{ num: 1 }`이 아닌, **특정 로직을 갖고 있는 행동(method)과 변경 가능한 상태(멤버 변수)를 가진 사물(object)**을 의미합니다.
{: .prompt-info }

## Class가 필요한 이유

비슷한 구조의 객체를 여러 개 만들어야 한다면 어떻게 할까요?

### ❌ 반복적인 객체 생성 (비효율적)

```javascript
let ray = {
  name: 'Ray',
  price: 2000000,
  getName: function() {
    return this.name;
  },
  getPrice: function() {
    return this.price;
  },
  applyDiscount: function(discount) {
    return this.price * discount;
  }
}

let morning = {
  name: 'Morning',
  price: 1500000,
  getName: function() {
    return this.name;
  },
  getPrice: function() {
    return this.price;
  },
  applyDiscount: function(discount) {
    return this.price * discount;
  }
}

// 새로운 차가 나올 때마다 똑같은 코드 반복... 😰
```

### ✅ Class 사용 (효율적)

```javascript
class Car {
  constructor(name, price) {
    this.name = name;
    this.price = price;
  }

  getName() {
    return this.name;
  }

  getPrice() {
    return this.price;
  }

  applyDiscount(discount) {
    return this.price * discount;
  }
}

// 간단하게 새로운 차 생성!
const ray = new Car('Ray', 2000000);
const morning = new Car('Morning', 1500000);
const spaceship = new Car('우주선', 25000000);
```

> Class는 원하는 구조의 객체 틀을 짜놓고, 비슷한 모양의 객체를 공장처럼 찍어낼 수 있게 해줍니다.
{: .prompt-tip }

## Constructor (생성자)

**Constructor**는 클래스로 객체를 생성할 때 자동으로 호출되는 특별한 메서드입니다.

```javascript
class Car {
  constructor(name, price) {
    this.name = name;
    this.price = price;
  }
}

const morning = new Car('Morning', 2000000);
// new 키워드를 사용하면 constructor가 자동 호출됨
```

### Constructor의 특징

| 특징 | 설명 |
|------|------|
| **자동 호출** | `new` 키워드로 인스턴스 생성 시 자동 실행 |
| **초기화 담당** | 객체의 초기 상태 설정 |
| **한 번만 정의** | 클래스당 하나의 constructor만 가능 |
| **매개변수 받음** | 인스턴스 생성 시 필요한 값을 인자로 받음 |

### this 키워드

`this`는 클래스 내에서 **현재 인스턴스 자신**을 가리킵니다.

```javascript
class Car {
  constructor(name, price) {
    this.name = name;        // 인스턴스의 name 프로퍼티
    this.price = price;      // 인스턴스의 price 프로퍼티
    this.department = "선릉지점";  // 기본값 설정도 가능
  }
}
```

## Instance (인스턴스)

**Instance**는 클래스를 통해 생성된 실제 객체입니다.

```javascript
class Car {
  constructor(name, price) {
    this.name = name;
    this.price = price;
  }
}

// Car 클래스의 인스턴스 생성
const morning = new Car('Morning', 20000000);
const spaceship = new Car('우주선', 25000000);

console.log(morning.name);   // 'Morning'
console.log(spaceship.name); // '우주선'
```

### 인스턴스 생성 과정

1. `new` 키워드로 새로운 객체 생성
2. `constructor()` 메서드 자동 호출
3. `constructor()` 내부에서 `this`로 프로퍼티 초기화
4. 초기화된 인스턴스 반환

> 같은 클래스로 만들어진 인스턴스들은 같은 구조를 가지지만, 각각 다른 값을 가질 수 있습니다.
{: .prompt-info }

## Methods (메서드)

**Method**는 클래스 내부에 정의된 함수로, 인스턴스의 동작을 정의합니다.

```javascript
class Car {
  constructor(name, price) {
    this.name = name;
    this.price = price;
    this.department = "선릉지점";
  }

  // 할인가 계산 메서드
  applyDiscount(discount) {
    return this.price * discount;
  }

  // 지점 변경 메서드
  changeDepartment(departmentName) {
    this.department = departmentName;
  }

  // 정보 출력 메서드
  getInfo() {
    return `${this.name} - ${this.price.toLocaleString()}원`;
  }
}

const morning = new Car('Morning', 20000000);

// 메서드 호출
console.log(morning.getInfo());           // 'Morning - 20,000,000원'
console.log(morning.applyDiscount(0.5));  // 10000000

morning.changeDepartment("강남지점");
console.log(morning.department);          // '강남지점'
```

### 클래스와 객체 리터럴의 차이

```javascript
// 객체 리터럴 - 쉼표(,) 필요
const obj = {
  name: 'test',
  getName: function() {
    return this.name;
  },    // ← 쉼표 필요
  setName: function(name) {
    this.name = name;
  }     // ← 쉼표 필요
};

// 클래스 - 쉼표 불필요
class MyClass {
  constructor(name) {
    this.name = name;
  }    // ← 쉼표 없음

  getName() {
    return this.name;
  }    // ← 쉼표 없음

  setName(name) {
    this.name = name;
  }    // ← 쉼표 없음
}
```

## 실전 예제

### 자동차 판매 시스템

```javascript
class Car {
  constructor(name, price) {
    this.name = name;
    this.price = price;
    this.sold = false;
  }

  // 할인가 계산
  getDiscountedPrice(discountRate) {
    if (discountRate < 0 || discountRate > 1) {
      throw new Error('할인율은 0~1 사이여야 합니다');
    }
    return this.price * (1 - discountRate);
  }

  // 판매 처리
  sell() {
    if (this.sold) {
      console.log(`${this.name}은(는) 이미 판매되었습니다.`);
      return false;
    }
    this.sold = true;
    console.log(`${this.name}이(가) 판매되었습니다!`);
    return true;
  }

  // 차량 정보 출력
  displayInfo() {
    const status = this.sold ? '판매완료' : '판매중';
    return `[${status}] ${this.name} - ${this.price.toLocaleString()}원`;
  }
}

// 차량 등록
const morning = new Car('Morning', 20000000);
const ray = new Car('Ray', 25000000);

console.log(morning.displayInfo());        // '[판매중] Morning - 20,000,000원'
console.log(morning.getDiscountedPrice(0.1)); // 18000000

morning.sell();                            // 'Morning이(가) 판매되었습니다!'
console.log(morning.displayInfo());        // '[판매완료] Morning - 20,000,000원'
morning.sell();                            // 'Morning은(는) 이미 판매되었습니다.'
```

## 핵심 정리

### Class 문법 규칙

| 규칙 | 설명 | 예시 |
|------|------|------|
| **클래스명** | 대문자로 시작, CamelCase | `Car`, `UserProfile` |
| **constructor** | 클래스당 하나만 | `constructor(name) {}` |
| **메서드** | 쉼표 없이 작성 | `getName() {}` |
| **인스턴스 생성** | `new` 키워드 사용 | `new Car()` |

### 멤버 변수와 메서드

- **멤버 변수**: 클래스 내에서 `this`로 접근하는 변수
- **메서드**: 클래스 내에 정의된 함수
- **this**: 현재 인스턴스를 가리키는 키워드

## 주의사항

> **CSS의 class와는 완전히 다른 개념입니다!**
> - CSS class: 스타일을 적용하기 위한 선택자
> - JavaScript class: 객체를 생성하기 위한 설계도
{: .prompt-warning }

## 다음 단계

Class의 기본 개념을 이해했다면, 다음 주제들을 학습해보세요:

- **상속(Inheritance)**: `extends` 키워드로 클래스 확장
- **Getter/Setter**: 프로퍼티 접근 제어
- **Static 메서드**: 인스턴스 없이 호출 가능한 메서드
- **Private 필드**: `#`로 시작하는 비공개 프로퍼티

## 참고 자료

- [MDN - Classes](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Classes)
- [JavaScript.info - Classes](https://ko.javascript.info/classes)
