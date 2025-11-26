---
title: 컴파일러(Compiler)와 인터프리터(Interpreter) 완벽 가이드
description: 프로그래밍 언어 실행 방식의 양대 산맥인 컴파일러와 인터프리터의 동작 원리, 컴파일 과정의 단계, JIT 컴파일러와 하이브리드 방식, 그리고 현대 언어들의 최신 접근 방식까지 완벽하게 이해합니다.
author: changsu
date: 2020-09-22 23:26:35 +0900
categories: [Programming, Computer Science]
tags: [compiler, interpreter, programming]
---

프로그래밍 언어를 실행하는 두 가지 주요 방식인 **컴파일러(Compiler)**와 **인터프리터(Interpreter)**의 동작 원리와 특징을 이해하면, 각 언어의 성능 특성과 적합한 사용 사례를 파악할 수 있습니다.

## 컴파일러(Compiler)

컴파일러는 고수준 언어로 작성된 **소스 코드 전체**를 기계어(또는 중간 표현)로 **한 번에 번역**하여 실행 파일을 생성하는 프로그램입니다.

### 컴파일 과정의 단계

<div style="text-align: center; margin: 25px 0; font-size: 16px; color: #555; font-weight: bold;">
  Source Code → Compiler → Executable File
</div>

<div style="background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); padding: 25px; border-radius: 15px; margin: 20px 0; box-shadow: 0 8px 20px rgba(0,0,0,0.2);">
  <div style="display: flex; flex-direction: column; gap: 15px; align-items: center;">
    <!-- Source Code -->
    <div style="background: linear-gradient(135deg, #3498db 0%, #2980b9 100%); color: white; padding: 15px 30px; border-radius: 10px; text-align: center; font-weight: bold; min-width: 250px; box-shadow: 0 4px 10px rgba(52, 152, 219, 0.3);">
      Source Code<br>
      <span style="font-size: 14px; opacity: 0.9;">(예: main.c)</span>
    </div>

    <div style="color: white; font-size: 24px;">↓</div>

    <!-- 1. Lexical Analysis -->
    <div style="background: rgba(255,255,255,0.95); padding: 15px; border-radius: 10px; min-width: 300px; box-shadow: 0 4px 10px rgba(0,0,0,0.1);">
      <div style="color: #e74c3c; font-weight: bold; margin-bottom: 5px;">1. Lexical Analysis (어휘 분석)</div>
      <div style="color: #555; font-size: 14px;">문자열을 토큰으로 분해</div>
    </div>

    <div style="color: white; font-size: 24px;">↓</div>

    <!-- 2. Syntax Analysis -->
    <div style="background: rgba(255,255,255,0.95); padding: 15px; border-radius: 10px; min-width: 300px; box-shadow: 0 4px 10px rgba(0,0,0,0.1);">
      <div style="color: #e67e22; font-weight: bold; margin-bottom: 5px;">2. Syntax Analysis (구문 분석)</div>
      <div style="color: #555; font-size: 14px;">토큰을 구문 트리로 변환</div>
    </div>

    <div style="color: white; font-size: 24px;">↓</div>

    <!-- 3. Semantic Analysis -->
    <div style="background: rgba(255,255,255,0.95); padding: 15px; border-radius: 10px; min-width: 300px; box-shadow: 0 4px 10px rgba(0,0,0,0.1);">
      <div style="color: #f39c12; font-weight: bold; margin-bottom: 5px;">3. Semantic Analysis (의미 분석)</div>
      <div style="color: #555; font-size: 14px;">의미 검사 및 타입 체크</div>
    </div>

    <div style="color: white; font-size: 24px;">↓</div>

    <!-- 4. Optimization -->
    <div style="background: rgba(255,255,255,0.95); padding: 15px; border-radius: 10px; min-width: 300px; box-shadow: 0 4px 10px rgba(0,0,0,0.1);">
      <div style="color: #27ae60; font-weight: bold; margin-bottom: 5px;">4. Optimization (최적화)</div>
      <div style="color: #555; font-size: 14px;">코드 최적화</div>
    </div>

    <div style="color: white; font-size: 24px;">↓</div>

    <!-- 5. Code Generation -->
    <div style="background: rgba(255,255,255,0.95); padding: 15px; border-radius: 10px; min-width: 300px; box-shadow: 0 4px 10px rgba(0,0,0,0.1);">
      <div style="color: #8e44ad; font-weight: bold; margin-bottom: 5px;">5. Code Generation (코드 생성)</div>
      <div style="color: #555; font-size: 14px;">기계어 코드 생성</div>
    </div>

    <div style="color: white; font-size: 24px;">↓</div>

    <!-- 6. Linking -->
    <div style="background: rgba(255,255,255,0.95); padding: 15px; border-radius: 10px; min-width: 300px; box-shadow: 0 4px 10px rgba(0,0,0,0.1);">
      <div style="color: #16a085; font-weight: bold; margin-bottom: 5px;">6. Linking (링킹)</div>
      <div style="color: #555; font-size: 14px;">라이브러리 연결</div>
    </div>

    <div style="color: white; font-size: 24px;">↓</div>

    <!-- Executable File -->
    <div style="background: linear-gradient(135deg, #2ecc71 0%, #27ae60 100%); color: white; padding: 15px 30px; border-radius: 10px; text-align: center; font-weight: bold; min-width: 250px; box-shadow: 0 4px 10px rgba(46, 204, 113, 0.3);">
      Executable File<br>
      <span style="font-size: 14px; opacity: 0.9;">(예: main.exe)</span>
    </div>
  </div>
</div>

### 컴파일러의 특징

**장점:**
- **빠른 실행 속도**: 이미 기계어로 번역되어 있어 실행 시 번역 과정 불필요
- **최적화**: 전체 코드를 분석하여 고도의 최적화 가능
- **소스 코드 보호**: 실행 파일만 배포하므로 원본 코드 노출 방지
- **조기 오류 발견**: 컴파일 시점에 문법 오류 및 타입 오류 감지

**단점:**
- **긴 컴파일 시간**: 전체 코드를 번역해야 하므로 대규모 프로젝트에서 시간 소요
- **플랫폼 종속성**: 특정 플랫폼용으로 컴파일되어 다른 플랫폼에서 재컴파일 필요
- **개발 사이클**: 코드 수정 후 매번 재컴파일 필요

### 대표적인 컴파일 언어

```c
// C 언어 예시
#include <stdio.h>

int main() {
    printf("Hello, World!\n");
    return 0;
}

// 컴파일 과정
// $ gcc main.c -o main     (컴파일)
// $ ./main                 (실행)
```

**주요 컴파일 언어:**
- **C/C++**: 시스템 프로그래밍, 게임 엔진, 임베디드
- **Rust**: 시스템 프로그래밍, 안전성 중시
- **Go**: 백엔드 서버, 클라우드 인프라
- **Swift**: iOS/macOS 애플리케이션

## 인터프리터(Interpreter)

인터프리터는 소스 코드를 **한 줄씩** 또는 **문장 단위로** 읽어서 **즉시 실행**하는 프로그램입니다.

### 인터프리터 실행 과정

<div style="text-align: center; margin: 25px 0; font-size: 16px; color: #555; font-weight: bold;">
  Source Code → Interpreter → Direct Execution
</div>

<div style="background: linear-gradient(135deg, #f39c12 0%, #e67e22 100%); padding: 25px; border-radius: 15px; margin: 20px 0; box-shadow: 0 8px 20px rgba(0,0,0,0.2);">
  <div style="display: flex; flex-direction: column; gap: 15px; align-items: center;">
    <!-- Source Code -->
    <div style="background: linear-gradient(135deg, #3498db 0%, #2980b9 100%); color: white; padding: 15px 30px; border-radius: 10px; text-align: center; font-weight: bold; min-width: 250px; box-shadow: 0 4px 10px rgba(52, 152, 219, 0.3);">
      Source Code<br>
      <span style="font-size: 14px; opacity: 0.9;">(예: script.py)</span>
    </div>

    <div style="color: white; font-size: 24px;">↓</div>

    <!-- Interpreter -->
    <div style="background: rgba(255,255,255,0.95); padding: 20px; border-radius: 10px; min-width: 300px; box-shadow: 0 4px 10px rgba(0,0,0,0.1);">
      <div style="color: #e67e22; font-weight: bold; text-align: center; margin-bottom: 15px; font-size: 16px;">Interpreter</div>
      <div style="display: flex; flex-direction: column; gap: 8px;">
        <div style="background: #ecf0f1; padding: 8px 15px; border-radius: 5px; color: #2c3e50; font-size: 14px;">Line 1 → Execute</div>
        <div style="background: #ecf0f1; padding: 8px 15px; border-radius: 5px; color: #2c3e50; font-size: 14px;">Line 2 → Execute</div>
        <div style="background: #ecf0f1; padding: 8px 15px; border-radius: 5px; color: #2c3e50; font-size: 14px;">Line 3 → Execute</div>
        <div style="text-align: center; color: #7f8c8d; font-size: 14px;">...</div>
      </div>
    </div>

    <div style="color: white; font-size: 24px;">↓</div>

    <!-- Output -->
    <div style="background: linear-gradient(135deg, #2ecc71 0%, #27ae60 100%); color: white; padding: 15px 30px; border-radius: 10px; text-align: center; font-weight: bold; min-width: 250px; box-shadow: 0 4px 10px rgba(46, 204, 113, 0.3);">
      Output
    </div>
  </div>
</div>

### 인터프리터의 특징

**장점:**
- **즉시 실행**: 컴파일 과정 없이 바로 실행 가능
- **플랫폼 독립성**: 인터프리터만 있으면 어디서든 실행 가능
- **빠른 개발 사이클**: 코드 수정 후 즉시 테스트 가능
- **동적 타이핑**: 런타임에 타입 결정으로 유연한 개발
- **REPL 지원**: 대화형 프로그래밍 환경 제공

**단점:**
- **느린 실행 속도**: 매번 코드를 해석하며 실행
- **런타임 오류**: 실행 시점에 오류 발견
- **소스 코드 노출**: 배포 시 원본 코드가 그대로 노출

### 대표적인 인터프리터 언어

```python
# Python 예시
def greet(name):
    print(f"Hello, {name}!")

greet("World")

# 실행 과정
# $ python script.py    (인터프리터가 즉시 실행)
```

**주요 인터프리터 언어:**
- **Python**: 데이터 과학, 웹 개발, 자동화
- **Ruby**: 웹 개발 (Ruby on Rails)
- **JavaScript** (전통적): 웹 브라우저 스크립팅
- **PHP**: 서버 사이드 웹 개발

## 컴파일러 vs 인터프리터 비교

| 구분 | 컴파일러 | 인터프리터 |
|------|---------|-----------|
| **번역 방식** | 전체 코드를 한 번에 번역 | 한 줄씩 번역 및 실행 |
| **실행 속도** | 빠름 (이미 기계어) | 느림 (매번 해석) |
| **컴파일 시간** | 소요됨 (초기에 한 번) | 없음 (즉시 실행) |
| **메모리 사용** | 실행 파일 크기 큼 | 소스 코드만 필요 |
| **오류 발견** | 컴파일 시점 (조기 발견) | 런타임 시점 (늦은 발견) |
| **개발 편의성** | 수정 후 재컴파일 필요 | 수정 후 즉시 실행 |
| **디버깅** | 어려움 (기계어 분석) | 쉬움 (소스 코드 직접 확인) |
| **플랫폼 이식성** | 낮음 (재컴파일 필요) | 높음 (인터프리터만 필요) |
| **보안** | 소스 코드 노출 안 됨 | 소스 코드 노출됨 |
| **배포 방식** | 실행 파일 배포 | 소스 코드 배포 |

## 하이브리드 방식: JIT 컴파일러

현대의 많은 언어들은 **컴파일러와 인터프리터의 장점을 결합한 하이브리드 방식**을 사용합니다.

### 바이트코드 + JIT (Just-In-Time) 컴파일

<div style="text-align: center; margin: 25px 0; font-size: 16px; color: #555; font-weight: bold;">
  Source Code → Bytecode → JIT Compiler → Machine Code
</div>

<div style="background: linear-gradient(135deg, #9b59b6 0%, #8e44ad 100%); padding: 25px; border-radius: 15px; margin: 20px 0; box-shadow: 0 8px 20px rgba(0,0,0,0.2);">
  <div style="display: flex; flex-direction: column; gap: 15px; align-items: center;">
    <!-- Source Code -->
    <div style="background: linear-gradient(135deg, #3498db 0%, #2980b9 100%); color: white; padding: 15px 30px; border-radius: 10px; text-align: center; font-weight: bold; min-width: 250px; box-shadow: 0 4px 10px rgba(52, 152, 219, 0.3);">
      Source Code
    </div>

    <div style="color: white; font-size: 24px;">↓</div>

    <!-- AOT Compiler -->
    <div style="background: rgba(255,255,255,0.95); padding: 15px; border-radius: 10px; min-width: 300px; box-shadow: 0 4px 10px rgba(0,0,0,0.1);">
      <div style="color: #e74c3c; font-weight: bold; margin-bottom: 5px;">AOT Compiler (사전 컴파일)</div>
      <div style="color: #555; font-size: 14px;">Ahead-Of-Time Compilation</div>
    </div>

    <div style="color: white; font-size: 24px;">↓</div>

    <!-- Bytecode -->
    <div style="background: rgba(255,255,255,0.95); padding: 15px; border-radius: 10px; min-width: 300px; box-shadow: 0 4px 10px rgba(0,0,0,0.1);">
      <div style="color: #f39c12; font-weight: bold; margin-bottom: 5px;">Bytecode (중간 코드)</div>
      <div style="color: #555; font-size: 14px;">중간 표현 (플랫폼 독립적)</div>
    </div>

    <div style="color: white; font-size: 24px;">↓</div>

    <!-- Interpreter + JIT Compiler -->
    <div style="background: rgba(255,255,255,0.95); padding: 20px; border-radius: 10px; min-width: 300px; box-shadow: 0 4px 10px rgba(0,0,0,0.1);">
      <div style="color: #8e44ad; font-weight: bold; text-align: center; margin-bottom: 15px; font-size: 16px;">Interpreter + JIT Compiler</div>
      <div style="display: flex; flex-direction: column; gap: 8px;">
        <div style="background: #ecf0f1; padding: 8px 15px; border-radius: 5px; color: #2c3e50; font-size: 14px;">초기 실행: 인터프리터 모드</div>
        <div style="text-align: center; color: #7f8c8d; font-size: 20px;">↓</div>
        <div style="background: #fff3cd; padding: 8px 15px; border-radius: 5px; color: #856404; font-size: 14px; font-weight: bold;">Hot Code 발견 🔥</div>
        <div style="color: #7f8c8d; font-size: 13px; text-align: center;">자주 실행되는 코드 식별</div>
        <div style="text-align: center; color: #7f8c8d; font-size: 20px;">↓</div>
        <div style="background: #d4edda; padding: 8px 15px; border-radius: 5px; color: #155724; font-size: 14px; font-weight: bold;">기계어로 컴파일 ⚡</div>
        <div style="color: #7f8c8d; font-size: 13px; text-align: center;">해당 부분만 최적화 컴파일</div>
      </div>
    </div>

    <div style="color: white; font-size: 24px;">↓</div>

    <!-- Fast Execution -->
    <div style="background: linear-gradient(135deg, #2ecc71 0%, #27ae60 100%); color: white; padding: 15px 30px; border-radius: 10px; text-align: center; font-weight: bold; min-width: 250px; box-shadow: 0 4px 10px rgba(46, 204, 113, 0.3);">
      Fast Execution
    </div>
  </div>
</div>

### Java의 JVM (Java Virtual Machine)

```java
// Java 코드
public class Hello {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}

// 실행 과정:
// 1. javac Hello.java  → Hello.class (바이트코드)
// 2. java Hello        → JVM이 바이트코드 실행
//    - 초기: 인터프리터 모드
//    - Hot Spot 발견: JIT 컴파일러가 기계어로 컴파일
```

**JVM의 실행 전략:**
1. **인터프리터**: 초기 실행은 바이트코드를 인터프리팅
2. **프로파일링**: 자주 실행되는 코드(Hot Spot) 식별
3. **JIT 컴파일**: Hot Spot을 기계어로 컴파일하여 캐싱
4. **최적화**: 런타임 정보를 활용한 고도의 최적화

### JavaScript의 V8 엔진

```javascript
// JavaScript 코드
function fibonacci(n) {
    if (n <= 1) return n;
    return fibonacci(n - 1) + fibonacci(n - 2);
}

// V8 엔진의 실행 과정:
// 1. Parser: 소스 코드 → AST (Abstract Syntax Tree)
// 2. Ignition (Interpreter): AST → 바이트코드 → 실행
// 3. TurboFan (JIT Compiler): Hot 함수를 최적화된 기계어로 컴파일
```

**V8의 최적화 파이프라인:**

<div style="background: linear-gradient(135deg, #16a085 0%, #138d75 100%); padding: 25px; border-radius: 15px; margin: 20px 0; box-shadow: 0 8px 20px rgba(0,0,0,0.2);">
  <div style="display: flex; flex-direction: column; gap: 15px; align-items: center;">
    <!-- Source Code -->
    <div style="background: linear-gradient(135deg, #3498db 0%, #2980b9 100%); color: white; padding: 15px 30px; border-radius: 10px; text-align: center; font-weight: bold; min-width: 220px; box-shadow: 0 4px 10px rgba(52, 152, 219, 0.3);">
      Source Code
    </div>

    <div style="color: white; font-size: 24px;">↓</div>

    <!-- Parser -->
    <div style="background: rgba(255,255,255,0.95); padding: 15px 30px; border-radius: 10px; text-align: center; font-weight: bold; min-width: 220px; color: #e74c3c; box-shadow: 0 4px 10px rgba(0,0,0,0.1);">
      Parser
    </div>

    <div style="color: white; font-size: 24px;">↓</div>

    <!-- AST -->
    <div style="background: rgba(255,255,255,0.95); padding: 15px 30px; border-radius: 10px; text-align: center; font-weight: bold; min-width: 220px; color: #e67e22; box-shadow: 0 4px 10px rgba(0,0,0,0.1);">
      AST
    </div>

    <div style="color: white; font-size: 24px;">↓</div>

    <!-- Ignition -->
    <div style="background: rgba(255,255,255,0.95); padding: 15px; border-radius: 10px; min-width: 280px; box-shadow: 0 4px 10px rgba(0,0,0,0.1);">
      <div style="color: #f39c12; font-weight: bold; text-align: center; margin-bottom: 5px; font-size: 16px;">Ignition (바이트코드)</div>
      <div style="color: #555; font-size: 14px; text-align: center;">Interpreter</div>
    </div>

    <div style="color: white; font-size: 14px; font-style: italic;">(Hot Code 감지 🔥)</div>

    <div style="color: white; font-size: 24px;">↓</div>

    <!-- TurboFan -->
    <div style="background: rgba(255,255,255,0.95); padding: 15px; border-radius: 10px; min-width: 280px; box-shadow: 0 4px 10px rgba(0,0,0,0.1);">
      <div style="color: #9b59b6; font-weight: bold; text-align: center; margin-bottom: 5px; font-size: 16px;">TurboFan (최적화)</div>
      <div style="color: #555; font-size: 14px; text-align: center;">Optimizing Compiler</div>
    </div>

    <div style="color: white; font-size: 24px;">↓</div>

    <!-- Machine Code -->
    <div style="background: linear-gradient(135deg, #2ecc71 0%, #27ae60 100%); color: white; padding: 15px 30px; border-radius: 10px; text-align: center; font-weight: bold; min-width: 220px; box-shadow: 0 4px 10px rgba(46, 204, 113, 0.3);">
      Machine Code
    </div>
  </div>
</div>

### Python의 실행 방식

```python
# Python 코드
def factorial(n):
    return 1 if n <= 1 else n * factorial(n - 1)

# 실행 과정:
# 1. CPython: .py → .pyc (바이트코드)
# 2. PVM (Python Virtual Machine): 바이트코드 인터프리팅

# PyPy: JIT 컴파일러 사용 (CPython보다 빠름)
```

**Python 구현체별 차이:**

| 구현체 | 방식 | 특징 |
|--------|------|------|
| **CPython** | 바이트코드 + 인터프리터 | 표준 구현, C 확장 호환성 |
| **PyPy** | 바이트코드 + JIT | 순수 Python 코드 3-5배 빠름 |
| **Jython** | 바이트코드 (JVM) | Java 라이브러리 사용 가능 |
| **IronPython** | 바이트코드 (.NET) | .NET 라이브러리 사용 가능 |

## 현대 언어들의 접근 방식

### TypeScript

```typescript
// TypeScript 코드
function greet(name: string): string {
    return `Hello, ${name}!`;
}

// 실행 과정:
// 1. tsc (TypeScript Compiler): .ts → .js (트랜스파일)
// 2. JavaScript 엔진 (V8 등): .js 실행
```

**트랜스파일(Transpile)**: 한 고수준 언어를 다른 고수준 언어로 변환

### WebAssembly (Wasm)

```text
C/C++/Rust 코드
    │
    ▼
LLVM 컴파일러
    │
    ▼
WebAssembly (.wasm)
    │
    ▼
브라우저 JIT 컴파일러
    │
    ▼
네이티브 기계어 (거의 네이티브 속도)
```

### Rust의 AOT 컴파일

```rust
// Rust 코드
fn main() {
    println!("Hello, World!");
}

// 컴파일 과정:
// 1. rustc: .rs → LLVM IR
// 2. LLVM: IR → 최적화 → 기계어
// 특징: 제로 코스트 추상화, 메모리 안전성 보장
```

## 선택 기준

### 컴파일 언어를 선택하는 경우

```markdown
✅ 성능이 중요한 시스템 (게임 엔진, OS, 임베디드)
✅ 대규모 연산 (과학 계산, 시뮬레이션)
✅ 제한된 자원 환경 (모바일, IoT)
✅ 소스 코드 보호가 필요한 상용 소프트웨어
```

### 인터프리터 언어를 선택하는 경우

```markdown
✅ 빠른 프로토타이핑 및 개발 속도가 중요
✅ 스크립팅 및 자동화 작업
✅ 플랫폼 독립성이 중요한 경우
✅ 동적 타이핑의 유연성이 필요한 경우
✅ REPL을 활용한 대화형 개발
```

### 하이브리드(JIT) 방식을 선택하는 경우

```markdown
✅ 개발 편의성과 성능 둘 다 중요
✅ 크로스 플랫폼 지원 필요
✅ 웹 애플리케이션 개발
✅ 엔터프라이즈 시스템 (Java, C# 등)
```

## 실전 성능 비교

### 피보나치 수열 계산 (n=40) 실행 시간 비교

```text
언어/구현체          실행 시간    상대 속도
─────────────────────────────────────────
C (gcc -O3)         0.3초       1.0x (기준)
Rust (release)      0.3초       1.0x
Go                  0.5초       1.7x
Java (JIT warmed)   0.6초       2.0x
JavaScript (V8)     0.8초       2.7x
PyPy (JIT)          1.2초       4.0x
Python (CPython)    15.0초      50.0x
Ruby                18.0초      60.0x

* 실제 성능은 코드 최적화, 알고리즘, 하드웨어에 따라 달라질 수 있습니다.
```

## 컴파일러/인터프리터 만들기

### 간단한 인터프리터 예시 (Python)

```python
# 간단한 수식 계산 인터프리터
def tokenize(expression):
    """문자열을 토큰으로 분해"""
    tokens = []
    current = ""

    for char in expression:
        if char.isdigit():
            current += char
        elif char in "+-*/":
            if current:
                tokens.append(int(current))
                current = ""
            tokens.append(char)
        elif char == " ":
            continue

    if current:
        tokens.append(int(current))

    return tokens

def interpret(tokens):
    """토큰을 해석하고 계산"""
    result = tokens[0]
    i = 1

    while i < len(tokens):
        operator = tokens[i]
        operand = tokens[i + 1]

        if operator == '+':
            result += operand
        elif operator == '-':
            result -= operand
        elif operator == '*':
            result *= operand
        elif operator == '/':
            result /= operand

        i += 2

    return result

# 사용 예시
expression = "10 + 5 * 2"
tokens = tokenize(expression)
result = interpret(tokens)
print(f"{expression} = {result}")  # 출력: 10 + 5 * 2 = 20
```

## 마치며

컴파일러와 인터프리터는 각각의 장단점이 있으며, 현대 언어들은 두 방식의 장점을 결합한 하이브리드 접근 방식을 많이 사용합니다. 프로젝트의 요구사항(성능, 개발 속도, 플랫폼 독립성 등)에 따라 적절한 언어와 실행 방식을 선택하는 것이 중요합니다.

**핵심 요약:**
- **컴파일러**: 빠른 실행, 긴 컴파일 시간, 플랫폼 종속적
- **인터프리터**: 즉시 실행, 느린 성능, 플랫폼 독립적
- **JIT 컴파일러**: 두 방식의 장점 결합, 런타임 최적화

## 참고 자료

- [Compiler Design - Tutorialspoint](https://www.tutorialspoint.com/compiler_design/index.htm)
- [How JavaScript Works - V8 Engine](https://blog.sessionstack.com/how-javascript-works-inside-the-v8-engine-5-tips-on-how-to-write-optimized-code-ac089e62b12e)
- [JVM Internals - Oracle](https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html)
- [LLVM Compiler Infrastructure](https://llvm.org/)
- [WebAssembly - MDN](https://developer.mozilla.org/en-US/docs/WebAssembly)
- [Crafting Interpreters](https://craftinginterpreters.com/) - 인터프리터 구현 가이드
- [Compiler Explorer](https://godbolt.org/) - 컴파일러 동작 시각화 도구
