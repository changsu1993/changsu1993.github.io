---
title: 팰린드롬(Palindrome) 완벽 가이드 - 다양한 알고리즘 접근법
description: 거꾸로 읽어도 똑같은 문자열 팰린드롬을 확인하는 다양한 알고리즘을 학습합니다. Two Pointer, Reverse, Recursion 방식과 시간/공간 복잡도 분석, 실전 코딩 테스트 문제까지 완벽하게 정리합니다.
author: changsu
date: 2020-10-13 01:46:46 +0900
categories: [Programming, Algorithms]
tags: [palindrome, algorithm, string, two-pointer, recursion, coding-test, javascript, complexity, 팰린드롬, 알고리즘, 문자열, 코딩테스트]
---

"토마토", "기러기"처럼 **거꾸로 읽어도 똑같은 문자열**을 **팰린드롬(Palindrome)**이라고 합니다. 팰린드롬 확인은 코딩 테스트에서 자주 출제되는 기본적이면서도 중요한 알고리즘 문제입니다.

## 팰린드롬(Palindrome)이란?

### 정의

**팰린드롬(Palindrome)**은 앞에서부터 읽으나 뒤에서부터 읽으나 동일한 문자열 또는 숫자를 의미합니다.

```text
팰린드롬 예시:

한글:
- 토마토
- 기러기
- 스위스
- 역삼역

영어:
- level
- radar
- noon
- civic
- kayak
- racecar
- madam

숫자:
- 121
- 12321
- 1221

문장 (공백 무시):
- "A man a plan a canal Panama"
- "Was it a car or a cat I saw"
```

## Two Pointer 접근법 (가장 효율적)

가장 효율적이고 직관적인 방법입니다.

### 알고리즘 설명

```text
원리:
1. 문자열의 양 끝에서 시작
2. 왼쪽 포인터는 앞에서 뒤로 이동
3. 오른쪽 포인터는 뒤에서 앞으로 이동
4. 각 단계마다 두 문자가 같은지 비교
5. 다르면 false, 끝까지 같으면 true

시각화:
"기러기"
 ↓   ↓
 기 = 기  ✓  (비교 1)
  ↓ ↓
  러=러  ✓  (비교 2)
   ↓
   기      (중앙 도달, 팰린드롬!)
```

### 기본 구현

```javascript
function palindrome(word) {
  // 왼쪽 포인터: 0부터 시작
  // 오른쪽 포인터: 마지막 인덱스부터 시작
  for (let i = 0; i < Math.floor(word.length / 2); i++) {
    let left = word[i];                    // 왼쪽에서 i번째
    let right = word[word.length - 1 - i]; // 오른쪽에서 i번째

    if (left !== right) {
      return false;  // 다르면 팰린드롬 아님
    }
  }
  return true;  // 모두 같으면 팰린드롬
}

// 테스트
console.log(palindrome("기러기"));    // true
console.log(palindrome("토마토"));    // true
console.log(palindrome("hello"));    // false
console.log(palindrome("abccba"));   // true
console.log(palindrome("123321"));   // true
console.log(palindrome("123456"));   // false
```

**왜 `Math.floor(word.length / 2)`까지만 반복하는가?**

```text
예시: "기러기" (길이 3)
- floor(3 / 2) = 1
- i = 0일 때: word[0] vs word[2] 비교 ("기" vs "기")
- 중간 문자는 비교할 필요 없음 (자기 자신과 항상 같음)

예시: "abccba" (길이 6)
- floor(6 / 2) = 3
- i = 0: word[0] vs word[5] ("a" vs "a")
- i = 1: word[1] vs word[4] ("b" vs "b")
- i = 2: word[2] vs word[3] ("c" vs "c")
- 3번만 비교하면 충분!
```

### While 문을 사용한 구현

```javascript
function isPalindrome(str) {
  let left = 0;
  let right = str.length - 1;

  while (left < right) {
    if (str[left] !== str[right]) {
      return false;
    }
    left++;   // 왼쪽 포인터 오른쪽으로 이동
    right--;  // 오른쪽 포인터 왼쪽으로 이동
  }

  return true;
}

console.log(isPalindrome("racecar"));  // true
console.log(isPalindrome("hello"));    // false
```

### 복잡도 분석

```text
시간 복잡도: O(n)
- n은 문자열의 길이
- 문자열의 절반만 비교 (n/2 번 반복)
- 하지만 Big-O 표기법에서는 O(n)

공간 복잡도: O(1)
- 추가 배열이나 데이터 구조 사용 안 함
- 변수 몇 개만 사용 (left, right, i 등)
```

## 문자열 뒤집기 방법

문자열을 뒤집어서 원본과 비교하는 방법입니다.

### 배열 메서드 사용

```javascript
function isPalindrome(str) {
  // 1. 문자열을 배열로 변환
  // 2. 배열 순서 뒤집기
  // 3. 다시 문자열로 변환
  const reversed = str.split('').reverse().join('');

  // 원본과 뒤집은 문자열이 같으면 팰린드롬
  return str === reversed;
}

console.log(isPalindrome("level"));   // true
console.log(isPalindrome("world"));   // false
```

**단계별 설명:**

```javascript
const str = "기러기";

// 1. split(''): 문자열을 문자 배열로 변환
str.split('')  // ["기", "러", "기"]

// 2. reverse(): 배열 순서 뒤집기
["기", "러", "기"].reverse()  // ["기", "러", "기"]

// 3. join(''): 배열을 문자열로 결합
["기", "러", "기"].join('')  // "기러기"

// 4. 비교
"기러기" === "기러기"  // true
```

### 복잡도 분석

```text
시간 복잡도: O(n)
- reverse() 메서드: O(n)
- 문자열 비교: O(n)
- 총 O(n) + O(n) = O(n)

공간 복잡도: O(n)
- 뒤집은 문자열을 저장하는 새로운 문자열 필요
- 원본의 길이만큼 추가 공간 사용
- Two Pointer보다 비효율적
```

## 재귀 방법

재귀를 사용한 우아한 해법입니다.

### 기본 재귀

```javascript
function isPalindrome(str) {
  // Base Case 1: 빈 문자열 또는 한 글자
  if (str.length <= 1) {
    return true;
  }

  // Base Case 2: 첫 글자와 마지막 글자가 다름
  if (str[0] !== str[str.length - 1]) {
    return false;
  }

  // Recursive Case: 양 끝 제거하고 재귀 호출
  return isPalindrome(str.slice(1, -1));
}

console.log(isPalindrome("radar"));   // true
console.log(isPalindrome("hello"));   // false
```

**재귀 과정 시각화:**

```text
isPalindrome("racecar")
├─ "r" === "r" ✓
├─ isPalindrome("aceca")
│  ├─ "a" === "a" ✓
│  ├─ isPalindrome("cec")
│  │  ├─ "c" === "c" ✓
│  │  ├─ isPalindrome("e")
│  │  │  └─ length = 1, return true
│  │  └─ return true
│  └─ return true
└─ return true
```

### 최적화된 재귀 (인덱스 사용)

```javascript
function isPalindrome(str, left = 0, right = str.length - 1) {
  // Base Case: 포인터가 교차하면 팰린드롬
  if (left >= right) {
    return true;
  }

  // 현재 문자가 다르면 팰린드롬 아님
  if (str[left] !== str[right]) {
    return false;
  }

  // 양쪽 포인터를 안쪽으로 이동하여 재귀 호출
  return isPalindrome(str, left + 1, right - 1);
}

console.log(isPalindrome("kayak"));   // true
console.log(isPalindrome("world"));   // false
```

### 복잡도 분석

```text
기본 재귀 (slice 사용):
- 시간: O(n²) - slice()가 매번 O(n)
- 공간: O(n) - 재귀 스택 + 새 문자열

최적화된 재귀 (인덱스 사용):
- 시간: O(n)
- 공간: O(n) - 재귀 스택만 사용
```

## 실전 문제: 문장의 팰린드롬

공백, 구두점, 대소문자를 무시하고 팰린드롬을 확인합니다.

### 문제

```text
입력: "A man, a plan, a canal: Panama"
출력: true

설명:
- 알파벳과 숫자만 추출: "AmanaplanacanalPanama"
- 소문자로 변환: "amanaplanacanalpanama"
- 팰린드롬 확인: true
```

### 구현

```javascript
function isPalindromeAdvanced(str) {
  // 1. 알파벳과 숫자만 추출하고 소문자로 변환
  const cleaned = str
    .toLowerCase()
    .replace(/[^a-z0-9]/g, '');

  // 2. Two Pointer로 팰린드롬 확인
  let left = 0;
  let right = cleaned.length - 1;

  while (left < right) {
    if (cleaned[left] !== cleaned[right]) {
      return false;
    }
    left++;
    right--;
  }

  return true;
}

// 테스트
console.log(isPalindromeAdvanced("A man, a plan, a canal: Panama"));
// true

console.log(isPalindromeAdvanced("race a car"));
// false

console.log(isPalindromeAdvanced("Was it a car or a cat I saw?"));
// true
```

### 정규 표현식 설명

```javascript
// /[^a-z0-9]/g 설명:

// [^...]: NOT (부정)
// a-z: 소문자 알파벳
// 0-9: 숫자
// g: global (모든 매칭)

// 의미: "소문자 알파벳과 숫자가 아닌 모든 문자"를 빈 문자열로 대체

const str = "A man, a plan!";
str.toLowerCase().replace(/[^a-z0-9]/g, '');
// "amanaplan"
```

## 실전 문제: 가장 긴 팰린드롬 부분 문자열

문자열에서 가장 긴 팰린드롬 부분 문자열을 찾습니다.

### 문제 (LeetCode #5)

```text
입력: "babad"
출력: "bab" 또는 "aba"

입력: "cbbd"
출력: "bb"
```

### 구현 (중앙 확장법)

```javascript
function longestPalindrome(s) {
  if (s.length < 2) return s;

  let start = 0;
  let maxLen = 1;

  // 중앙에서 양쪽으로 확장하는 헬퍼 함수
  function expandAroundCenter(left, right) {
    while (left >= 0 && right < s.length && s[left] === s[right]) {
      const currentLen = right - left + 1;
      if (currentLen > maxLen) {
        start = left;
        maxLen = currentLen;
      }
      left--;
      right++;
    }
  }

  for (let i = 0; i < s.length; i++) {
    // 홀수 길이 팰린드롬 (중심이 한 문자)
    expandAroundCenter(i, i);

    // 짝수 길이 팰린드롬 (중심이 두 문자 사이)
    expandAroundCenter(i, i + 1);
  }

  return s.substring(start, start + maxLen);
}

// 테스트
console.log(longestPalindrome("babad"));   // "bab" 또는 "aba"
console.log(longestPalindrome("cbbd"));    // "bb"
console.log(longestPalindrome("racecar")); // "racecar"
```

**알고리즘 설명:**

```text
중앙 확장법 (Expand Around Center):

1. 각 문자를 중심으로 팰린드롬 탐색
2. 홀수 길이: "a b a" (중심 = b)
3. 짝수 길이: "a b b a" (중심 = bb)

시각화:
"babad"
 ↓
 b (홀수: "b", 짝수: X)
  ↓
  a (홀수: "bab", 짝수: X)
   ↓
   b (홀수: "aba", 짝수: X)
    ↓
    a (홀수: "a", 짝수: "ab" X)
     ↓
     d (홀수: "d", 짝수: X)

결과: "bab" 또는 "aba" (길이 3)
```

### 복잡도 분석

```text
시간 복잡도: O(n²)
- n개의 중심점
- 각 중심에서 최대 n번 확장
- 최악의 경우: O(n²)

공간 복잡도: O(1)
- 추가 배열 사용 안 함
```

## 성능 비교

각 방법의 특징을 비교해봅시다.

```javascript
// 방법 1: Two Pointer (가장 효율적)
function method1(str) {
  for (let i = 0; i < Math.floor(str.length / 2); i++) {
    if (str[i] !== str[str.length - 1 - i]) {
      return false;
    }
  }
  return true;
}

// 방법 2: Reverse (간결하지만 메모리 사용)
function method2(str) {
  return str === str.split('').reverse().join('');
}

// 방법 3: 재귀 (우아하지만 스택 사용)
function method3(str, left = 0, right = str.length - 1) {
  if (left >= right) return true;
  if (str[left] !== str[right]) return false;
  return method3(str, left + 1, right - 1);
}
```

**성능 요약:**

```text
방법별 특징:

1. Two Pointer
   ├─ 시간: O(n)
   ├─ 공간: O(1)
   └─ 추천: ⭐⭐⭐⭐⭐ (최적)

2. Reverse
   ├─ 시간: O(n)
   ├─ 공간: O(n)
   └─ 추천: ⭐⭐⭐ (간결하지만 메모리 사용)

3. 재귀 (인덱스)
   ├─ 시간: O(n)
   ├─ 공간: O(n)
   └─ 추천: ⭐⭐ (우아하지만 스택 오버플로우 위험)

실무 권장: Two Pointer 방식
```

## 유니코드와 이모지 처리

JavaScript의 문자열 인덱싱은 UTF-16 기반이므로 이모지에 주의해야 합니다.

### 문제점

```javascript
const str = "👨‍👩‍👧‍👦";  // 가족 이모지

console.log(str.length);  // 25 (예상과 다름!)
console.log([...str].length);  // 7 (스프레드 연산자 사용)

// 일반적인 reverse는 이모지를 망가뜨림
console.log(str.split('').reverse().join(''));  // 깨진 이모지

// 올바른 방법
console.log([...str].reverse().join(''));  // 정상 동작
```

### 유니코드 안전한 팰린드롬 확인

```javascript
function isPalindromeUnicode(str) {
  const chars = [...str];  // 스프레드로 올바른 문자 배열 생성
  let left = 0;
  let right = chars.length - 1;

  while (left < right) {
    if (chars[left] !== chars[right]) {
      return false;
    }
    left++;
    right--;
  }

  return true;
}

// 테스트
console.log(isPalindromeUnicode("기러기"));  // true
console.log(isPalindromeUnicode("🚀🌙🚀"));  // true
```

## 코딩 테스트 팁

### 자주 나오는 변형 문제

```markdown
1. **팰린드롬 부분 문자열 개수**
   - 문자열에서 팰린드롬인 부분 문자열의 개수 세기

2. **최소 편집으로 팰린드롬 만들기**
   - 최소한의 문자 삽입/삭제로 팰린드롬 만들기

3. **팰린드롬 분할**
   - 문자열을 팰린드롬들로 분할하는 모든 방법

4. **가장 긴 팰린드롬 부분 수열**
   - 연속되지 않아도 되는 부분 수열 중 가장 긴 팰린드롬

5. **팰린드롬 숫자**
   - 주어진 범위에서 팰린드롬 숫자 찾기
```

### 체크리스트

```markdown
코딩 테스트 체크리스트:
- [ ] 대소문자 구분하는지 확인
- [ ] 공백/특수문자 처리 방법 확인
- [ ] 유니코드/이모지 고려 필요한지 확인
- [ ] 시간/공간 복잡도 요구사항 확인
- [ ] 엣지 케이스 테스트 (빈 문자열, 한 글자)
```

## 마치며

팰린드롬 확인은 간단해 보이지만 다양한 알고리즘 기법을 연습할 수 있는 좋은 문제입니다.

**핵심 요약:**
- **Two Pointer**: 가장 효율적 (시간 O(n), 공간 O(1))
- **Reverse**: 간결하지만 메모리 사용 (공간 O(n))
- **재귀**: 우아하지만 스택 오버플로우 주의
- **실무 권장**: Two Pointer 방식

## 참고 자료

- [LeetCode - Palindrome Problems](https://leetcode.com/tag/palindrome/)
- [Two Pointers Technique](https://www.geeksforgeeks.org/two-pointers-technique/)
- [LeetCode #125 - Valid Palindrome](https://leetcode.com/problems/valid-palindrome/)
- [LeetCode #5 - Longest Palindromic Substring](https://leetcode.com/problems/longest-palindromic-substring/)
- [프로그래머스 - 문자열 문제](https://programmers.co.kr/learn/challenges)
