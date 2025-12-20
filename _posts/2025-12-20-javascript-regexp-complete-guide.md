---
title: "JavaScript 정규표현식(RegExp) 완벽 가이드 - 패턴 매칭과 문자열 처리"
description: "JavaScript 정규표현식의 기초부터 고급 패턴까지. 메타문자, 그룹, 전방탐색, 유니코드 지원, 실전 유효성 검사 패턴을 마스터합니다."
date: 2025-12-20 14:00:00 +0900
categories: [Frontend, JavaScript]
tags: [javascript, regexp, regex, pattern-matching, validation]
---

## 개요

**정규표현식(Regular Expression, RegExp)**은 문자열에서 특정 패턴을 찾고, 추출하고, 변환하는 강력한 도구입니다. 이메일 유효성 검사, URL 파싱, 텍스트 검색/치환 등 문자열 처리가 필요한 거의 모든 곳에서 정규표현식이 활용됩니다.

JavaScript는 정규표현식을 1급 객체로 지원하며, 리터럴 문법과 다양한 문자열/정규표현식 메서드를 제공합니다. ES2018 이후로는 명명된 캡처 그룹, 후방탐색, 유니코드 프로퍼티 이스케이프 등 강력한 기능이 추가되어 더욱 정교한 패턴 매칭이 가능해졌습니다.

### 학습 목표

- 정규표현식의 기본 문법과 생성 방법 이해
- 메타문자, 문자 클래스, 수량자 완벽 마스터
- 그룹, 캡처, 전방탐색/후방탐색 활용법 습득
- 실전 유효성 검사 패턴 구현
- 정규표현식 성능 최적화 기법 학습

### 사전 지식

- JavaScript 문자열 기초 ([JavaScript 문자열 연결과 타입 변환 완벽 가이드](/posts/javascript-string-concatenation/) 참고)
- JavaScript 함수 기초

---

## 정규표현식 생성 방법

JavaScript에서 정규표현식을 생성하는 방법은 두 가지입니다.

### 리터럴 방식

슬래시(`/`)로 패턴을 감싸는 방식으로, 가장 일반적으로 사용됩니다.

```javascript
// 기본 형태: /패턴/플래그
const regex1 = /hello/;        // 'hello' 문자열 매칭
const regex2 = /hello/i;       // 대소문자 무시
const regex3 = /hello/gi;      // 전역 검색 + 대소문자 무시

// 리터럴은 스크립트 로드 시 컴파일됨
console.log(regex1.test('hello world')); // true
console.log(regex2.test('Hello World')); // true
```

### RegExp 생성자 방식

동적으로 패턴을 생성해야 할 때 사용합니다.

```javascript
// 문자열로 패턴 전달
const pattern = 'hello';
const regex1 = new RegExp(pattern);
const regex2 = new RegExp(pattern, 'gi');

// 동적 패턴 생성
function createSearchRegex(searchTerm) {
  // 특수문자 이스케이프 필요
  const escaped = searchTerm.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');
  return new RegExp(escaped, 'gi');
}

const userInput = 'hello (world)';
const dynamicRegex = createSearchRegex(userInput);
console.log(dynamicRegex); // /hello \(world\)/gi
```

### 리터럴 vs 생성자 비교

```javascript
// 리터럴 방식의 이스케이프
const regex1 = /\d+/;          // 숫자 매칭

// 생성자 방식의 이스케이프 (역슬래시 두 번)
const regex2 = new RegExp('\\d+');

console.log(regex1.source === regex2.source); // true

// 언제 무엇을 사용할까?
// - 리터럴: 패턴이 고정된 경우 (권장)
// - 생성자: 사용자 입력, 변수 기반 동적 패턴
```

---

## 메타문자와 특수문자

메타문자는 정규표현식에서 특별한 의미를 가지는 문자들입니다.

### 기본 메타문자

```javascript
// . (점) - 줄바꿈 제외 모든 문자 1개 매칭
console.log(/a.c/.test('abc'));  // true
console.log(/a.c/.test('aXc'));  // true
console.log(/a.c/.test('ac'));   // false (문자 없음)

// ^ - 문자열/줄의 시작
console.log(/^hello/.test('hello world')); // true
console.log(/^hello/.test('say hello'));   // false

// $ - 문자열/줄의 끝
console.log(/world$/.test('hello world')); // true
console.log(/world$/.test('world peace')); // false

// | - OR 연산자
console.log(/cat|dog/.test('I have a cat')); // true
console.log(/cat|dog/.test('I have a dog')); // true
console.log(/cat|dog/.test('I have a bird')); // false
```

### 문자 클래스 []

대괄호 안의 문자 중 하나와 매칭됩니다.

```javascript
// 문자 집합
console.log(/[abc]/.test('apple'));  // true (a 매칭)
console.log(/[abc]/.test('banana')); // true (a, b 매칭)
console.log(/[abc]/.test('xyz'));    // false

// 범위 지정
console.log(/[a-z]/.test('hello'));  // true (소문자)
console.log(/[A-Z]/.test('Hello'));  // true (대문자)
console.log(/[0-9]/.test('abc123')); // true (숫자)
console.log(/[a-zA-Z0-9]/.test('Test1')); // true (영숫자)

// 부정 문자 클래스 [^...]
console.log(/[^0-9]/.test('123'));   // false (숫자만 있음)
console.log(/[^0-9]/.test('12a3'));  // true (숫자 아닌 문자 있음)

// 특수문자는 [] 안에서 대부분 리터럴로 취급
console.log(/[.+*?]/.test('3+4'));   // true (+ 매칭)
console.log(/[$^]/.test('$100'));    // true ($ 매칭)
```

### 이스케이프 문자

특수문자를 리터럴로 매칭하려면 역슬래시(`\`)로 이스케이프합니다.

```javascript
// 메타문자 이스케이프
console.log(/3\.14/.test('3.14'));       // true
console.log(/3\.14/.test('3X14'));       // false
console.log(/\$100/.test('$100'));       // true
console.log(/\(hello\)/.test('(hello)')); // true

// 자주 이스케이프하는 문자: . * + ? ^ $ { } [ ] ( ) | \ /
const priceRegex = /\$\d+\.\d{2}/;
console.log(priceRegex.test('$19.99'));  // true
console.log(priceRegex.test('$5.00'));   // true
```

---

## 문자 클래스 단축형

자주 사용되는 문자 집합에 대한 단축 표기법입니다.

### 기본 문자 클래스

```javascript
// \d - 숫자 [0-9]
console.log(/\d/.test('abc123'));  // true
console.log(/\d/.test('abcdef'));  // false

// \D - 숫자가 아닌 문자 [^0-9]
console.log(/\D/.test('123'));     // false
console.log(/\D/.test('12a3'));    // true

// \w - 단어 문자 [a-zA-Z0-9_]
console.log(/\w/.test('hello'));   // true
console.log(/\w/.test('_var'));    // true
console.log(/\w/.test('$$$'));     // false

// \W - 단어 문자가 아닌 것 [^a-zA-Z0-9_]
console.log(/\W/.test('hello'));   // false
console.log(/\W/.test('hello!'));  // true

// \s - 공백 문자 (스페이스, 탭, 줄바꿈 등)
console.log(/\s/.test('hello world')); // true
console.log(/\s/.test('helloworld'));  // false

// \S - 공백이 아닌 문자
console.log(/\S/.test('   '));     // false
console.log(/\S/.test('  a  '));   // true
```

### 경계 매칭

```javascript
// \b - 단어 경계
console.log(/\bcat\b/.test('cat'));        // true
console.log(/\bcat\b/.test('the cat sat')); // true
console.log(/\bcat\b/.test('category'));    // false (cat이 단어 경계에 없음)
console.log(/\bcat\b/.test('tomcat'));      // false

// \B - 단어 경계가 아닌 곳
console.log(/\Bcat/.test('tomcat'));  // true (cat 앞이 경계가 아님)
console.log(/cat\B/.test('category')); // true (cat 뒤가 경계가 아님)

// 실용적 예제: 독립된 단어만 찾기
const text = 'I love JavaScript, not Java or JavaBeans';
console.log(text.match(/\bJava\b/g)); // ['Java']
```

---

## 수량자 (Quantifiers)

패턴의 반복 횟수를 지정합니다.

### 기본 수량자

```javascript
// * - 0회 이상
console.log(/ab*c/.test('ac'));     // true (b 0개)
console.log(/ab*c/.test('abc'));    // true (b 1개)
console.log(/ab*c/.test('abbbc'));  // true (b 3개)

// + - 1회 이상
console.log(/ab+c/.test('ac'));     // false (b 0개는 불일치)
console.log(/ab+c/.test('abc'));    // true
console.log(/ab+c/.test('abbbc'));  // true

// ? - 0회 또는 1회
console.log(/colou?r/.test('color'));  // true
console.log(/colou?r/.test('colour')); // true
console.log(/colou?r/.test('colouur')); // false

// 선택적 그룹
console.log(/https?:\/\//.test('http://'));  // true
console.log(/https?:\/\//.test('https://')); // true
```

### 정확한 횟수 지정

```javascript
// {n} - 정확히 n회
console.log(/\d{4}/.test('2024'));  // true
console.log(/\d{4}/.test('123'));   // false

// {n,} - n회 이상
console.log(/\d{2,}/.test('1'));    // false
console.log(/\d{2,}/.test('12'));   // true
console.log(/\d{2,}/.test('12345')); // true

// {n,m} - n회 이상 m회 이하
console.log(/\d{2,4}/.test('1'));     // false
console.log(/\d{2,4}/.test('12'));    // true
console.log(/\d{2,4}/.test('1234'));  // true
console.log(/\d{2,4}/.test('12345')); // true (1234 부분 매칭)

// 실용적 예제: 전화번호 형식
const phoneRegex = /\d{3}-\d{3,4}-\d{4}/;
console.log(phoneRegex.test('010-1234-5678')); // true
console.log(phoneRegex.test('02-123-4567'));   // true
```

### 탐욕적 vs 게으른 수량자

기본적으로 수량자는 **탐욕적(greedy)**으로 동작하여 가능한 많이 매칭합니다.

```javascript
// 탐욕적 수량자 (기본)
const html = '<div>Hello</div><div>World</div>';
console.log(html.match(/<div>.*<\/div>/));
// ['<div>Hello</div><div>World</div>'] - 전체 매칭

// 게으른 수량자 (? 추가)
console.log(html.match(/<div>.*?<\/div>/));
// ['<div>Hello</div>'] - 최소 매칭

// 게으른 수량자 종류
// *? - 0회 이상 (최소)
// +? - 1회 이상 (최소)
// ?? - 0회 또는 1회 (최소)
// {n,m}? - n~m회 (최소)

// 모든 div 태그 찾기
console.log(html.match(/<div>.*?<\/div>/g));
// ['<div>Hello</div>', '<div>World</div>']
```

---

## 그룹과 캡처

괄호를 사용하여 패턴을 그룹화하고 매칭된 부분을 캡처할 수 있습니다.

### 캡처 그룹

```javascript
// 기본 캡처 그룹
const dateRegex = /(\d{4})-(\d{2})-(\d{2})/;
const match = '2024-12-20'.match(dateRegex);

console.log(match[0]); // '2024-12-20' (전체 매칭)
console.log(match[1]); // '2024' (첫 번째 그룹)
console.log(match[2]); // '12' (두 번째 그룹)
console.log(match[3]); // '20' (세 번째 그룹)

// 역참조 (backreference)
const duplicateRegex = /(\w+)\s+\1/;
console.log(duplicateRegex.test('hello hello')); // true
console.log(duplicateRegex.test('hello world')); // false

// HTML 태그 매칭
const tagRegex = /<(\w+)>.*?<\/\1>/;
console.log(tagRegex.test('<div>content</div>')); // true
console.log(tagRegex.test('<div>content</span>')); // false
```

### 명명된 캡처 그룹 (ES2018)

```javascript
// (?<name>pattern) 문법
const dateRegex = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/;
const match = '2024-12-20'.match(dateRegex);

console.log(match.groups.year);  // '2024'
console.log(match.groups.month); // '12'
console.log(match.groups.day);   // '20'

// 구조 분해 할당과 함께
const { groups: { year, month, day } } = '2024-12-20'.match(dateRegex);
console.log(`${year}년 ${month}월 ${day}일`); // '2024년 12월 20일'

// 명명된 역참조
const duplicateRegex = /(?<word>\w+)\s+\k<word>/;
console.log(duplicateRegex.test('hello hello')); // true
```

### 비캡처 그룹

그룹화는 필요하지만 캡처는 필요 없을 때 사용합니다.

```javascript
// (?:pattern) - 비캡처 그룹
const regex = /(?:http|https):\/\/(\w+)/;
const match = 'https://example'.match(regex);

console.log(match[0]); // 'https://example'
console.log(match[1]); // 'example' (첫 번째 캡처 그룹)
// http|https는 캡처되지 않음

// 성능상 이점: 캡처하지 않으므로 메모리 절약
const emailDomain = /\w+@(?:\w+\.)+\w+/;
```

---

## 전방탐색과 후방탐색

특정 패턴 앞이나 뒤에 있는 텍스트를 매칭하되, 탐색 자체는 결과에 포함시키지 않습니다.

### 전방탐색 (Lookahead)

```javascript
// 긍정 전방탐색 (?=pattern) - 뒤에 패턴이 있어야 매칭
const priceRegex = /\d+(?=원)/g;
console.log('사과 1000원, 바나나 500원'.match(priceRegex));
// ['1000', '500'] - '원'은 결과에 포함되지 않음

// 부정 전방탐색 (?!pattern) - 뒤에 패턴이 없어야 매칭
const notDollar = /\d+(?!\$)/g;
console.log('100$ 200 300$'.match(notDollar));
// ['10', '200', '30'] - $ 앞 숫자 제외

// 비밀번호 강도 검사 (전방탐색 활용)
// 최소 8자, 대문자, 소문자, 숫자 포함
const strongPassword = /^(?=.*[A-Z])(?=.*[a-z])(?=.*\d).{8,}$/;
console.log(strongPassword.test('Password1'));  // true
console.log(strongPassword.test('password1'));  // false (대문자 없음)
console.log(strongPassword.test('PASSWORD1'));  // false (소문자 없음)
console.log(strongPassword.test('Password'));   // false (숫자 없음)
```

### 후방탐색 (Lookbehind, ES2018)

```javascript
// 긍정 후방탐색 (?<=pattern) - 앞에 패턴이 있어야 매칭
const dollarAmount = /(?<=\$)\d+/g;
console.log('$100 and $200'.match(dollarAmount));
// ['100', '200'] - '$'는 결과에 포함되지 않음

// 부정 후방탐색 (?<!pattern) - 앞에 패턴이 없어야 매칭
const notWon = /(?<!원)\d+/g;
console.log('1000원 2000 3000원'.match(notWon));
// ['1000', '2000', '3000'] - 복잡한 결과

// 실용적 예제: 통화 기호에 따른 금액 추출
const text = '$100 USD, 50000원, EUR200';
console.log(text.match(/(?<=\$)\d+/g));  // ['100']
console.log(text.match(/\d+(?=원)/g));   // ['50000']
console.log(text.match(/(?<=EUR)\d+/g)); // ['200']
```

---

## 플래그 (Flags)

정규표현식의 동작을 제어하는 옵션입니다.

### 기본 플래그

```javascript
// g (global) - 전역 검색
console.log('banana'.match(/a/));  // ['a'] (첫 번째만)
console.log('banana'.match(/a/g)); // ['a', 'a', 'a'] (모두)

// i (ignoreCase) - 대소문자 무시
console.log(/hello/i.test('HELLO')); // true

// m (multiline) - 다중 행 모드
const text = `first line
second line
third line`;

console.log(text.match(/^.+/g));  // ['first line'] (첫 줄만)
console.log(text.match(/^.+/gm)); // ['first line', 'second line', 'third line']

// s (dotAll, ES2018) - .이 줄바꿈도 매칭
const html = `<div>
  content
</div>`;
console.log(/<div>.*<\/div>/s.test(html)); // true (줄바꿈 포함)
console.log(/<div>.*<\/div>/.test(html));  // false (s 없으면)
```

### 고급 플래그

```javascript
// u (unicode) - 유니코드 모드
// 이모지, 특수 문자 올바르게 처리
console.log(/^.$/u.test('A'));  // true - 단일 문자로 인식

// 유니코드 프로퍼티 이스케이프 (u 플래그 필요)
console.log(/\p{Emoji}/u.test('B'));      // false - 이모지로 처리

// y (sticky) - 고정 위치 검색
const regex = /\d+/y;
const str = '123abc456';
regex.lastIndex = 0;
console.log(regex.exec(str)); // ['123']
regex.lastIndex = 3;
console.log(regex.exec(str)); // null (위치 3에서 숫자 시작 안 함)
regex.lastIndex = 6;
console.log(regex.exec(str)); // ['456']

// d (indices, ES2022) - 캡처 그룹 인덱스 정보
const dateRegex = /(?<year>\d{4})-(?<month>\d{2})/d;
const match = '2024-12'.match(dateRegex);
console.log(match.indices.groups.year);  // [0, 4]
console.log(match.indices.groups.month); // [5, 7]
```

---

## String 메서드와 정규표현식

문자열 객체에서 정규표현식을 사용하는 메서드들입니다.

### match와 matchAll

```javascript
// match - 매칭 결과 반환
const str = 'Hello World';
console.log(str.match(/o/));   // ['o', index: 4, input: 'Hello World']
console.log(str.match(/o/g));  // ['o', 'o'] (g 플래그시 배열만)
console.log(str.match(/x/));   // null

// matchAll (ES2020) - 이터레이터 반환, 캡처 그룹 포함
const text = '2024-12-20, 2025-01-15';
const dateRegex = /(\d{4})-(\d{2})-(\d{2})/g;

for (const match of text.matchAll(dateRegex)) {
  console.log(match[0], '-> Year:', match[1], 'Month:', match[2]);
}
// 2024-12-20 -> Year: 2024 Month: 12
// 2025-01-15 -> Year: 2025 Month: 01

// matchAll을 배열로 변환
const matches = [...text.matchAll(dateRegex)];
console.log(matches.length); // 2
```

### replace와 replaceAll

```javascript
// replace - 첫 번째 매칭만 치환 (g 플래그 없을 때)
console.log('hello world'.replace(/o/, '0'));  // 'hell0 world'
console.log('hello world'.replace(/o/g, '0')); // 'hell0 w0rld'

// 캡처 그룹 참조
const dateStr = '2024-12-20';
console.log(dateStr.replace(/(\d{4})-(\d{2})-(\d{2})/, '$2/$3/$1'));
// '12/20/2024'

// 명명된 그룹 참조
console.log(dateStr.replace(
  /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/,
  '$<month>/$<day>/$<year>'
));
// '12/20/2024'

// 함수를 사용한 치환
const prices = '사과 1000원, 바나나 500원';
const result = prices.replace(/(\d+)원/g, (match, price) => {
  return `${(Number(price) * 1.1).toFixed(0)}원 (VAT 포함)`;
});
console.log(result);
// '사과 1100원 (VAT 포함), 바나나 550원 (VAT 포함)'

// replaceAll (ES2021) - 모든 매칭 치환 (문자열도 가능)
console.log('a-b-c'.replaceAll('-', '_')); // 'a_b_c'
console.log('a-b-c'.replaceAll(/-/g, '_')); // 'a_b_c' (정규식도 가능, g 필수)
```

### split과 search

```javascript
// split - 패턴으로 문자열 분할
console.log('a, b,  c,   d'.split(/,\s*/));
// ['a', 'b', 'c', 'd'] - 쉼표와 공백 제거

// 캡처 그룹 포함시 구분자도 결과에 포함
console.log('a1b2c3d'.split(/(\d)/));
// ['a', '1', 'b', '2', 'c', '3', 'd']

// search - 매칭 위치 반환
console.log('hello world'.search(/o/));     // 4
console.log('hello world'.search(/world/)); // 6
console.log('hello world'.search(/x/));     // -1 (없음)

// search는 g 플래그 무시
console.log('hello'.search(/l/g)); // 2 (첫 번째 위치만)
```

---

## RegExp 메서드

RegExp 객체의 인스턴스 메서드입니다.

### test 메서드

```javascript
// test - 매칭 여부 반환 (가장 빠름)
const emailRegex = /\S+@\S+\.\S+/;
console.log(emailRegex.test('user@example.com')); // true
console.log(emailRegex.test('invalid-email'));    // false

// g 플래그와 lastIndex 주의사항
const regex = /a/g;
const str = 'ababa';

console.log(regex.test(str)); // true, lastIndex: 1
console.log(regex.test(str)); // true, lastIndex: 3
console.log(regex.test(str)); // true, lastIndex: 5
console.log(regex.test(str)); // false, lastIndex: 0 (리셋)
console.log(regex.test(str)); // true, lastIndex: 1

// 매번 새로운 검사가 필요하면 리터럴 사용 권장
const check = str => /a/.test(str);
```

### exec 메서드

```javascript
// exec - 상세한 매칭 정보 반환
const regex = /(\w+)@(\w+)\.(\w+)/;
const result = regex.exec('user@example.com');

console.log(result[0]);     // 'user@example.com' (전체)
console.log(result[1]);     // 'user'
console.log(result[2]);     // 'example'
console.log(result[3]);     // 'com'
console.log(result.index);  // 0 (매칭 시작 위치)
console.log(result.input);  // 'user@example.com' (원본 문자열)

// g 플래그로 반복 실행
const regex2 = /\d+/g;
const str = '12 apples and 34 oranges';
let match;

while ((match = regex2.exec(str)) !== null) {
  console.log(`Found ${match[0]} at index ${match.index}`);
}
// Found 12 at index 0
// Found 34 at index 14
```

---

## 유니코드와 정규표현식

### u 플래그의 중요성

```javascript
// 서로게이트 페어 문제
const emoji = 'A';  // 이모지 예시 문자

// u 플래그 없이는 이모지가 2개 문자로 인식될 수 있음
console.log('Hello'.length);  // 5

// 유니코드 코드포인트 이스케이프 (u 플래그 필요)
console.log(/\u{1F600}/u.test('Hello'));  // 이모지 테스트
console.log(/\u{AC00}/u.test('가'));       // true (한글 '가')

// 유니코드 범위 매칭
const koreanRegex = /[\u{AC00}-\u{D7AF}]+/u;
console.log(koreanRegex.test('안녕하세요')); // true
console.log(koreanRegex.test('Hello'));      // false
```

### 유니코드 프로퍼티 이스케이프 (ES2018)

```javascript
// \p{Property} - 유니코드 프로퍼티 매칭 (u 플래그 필요)
// \P{Property} - 부정 매칭

// 스크립트 기반 매칭
const koreanRegex = /\p{Script=Hangul}+/u;
console.log(koreanRegex.test('한글'));     // true
console.log(koreanRegex.test('English'));  // false

const japaneseRegex = /\p{Script=Hiragana}+/u;
console.log(japaneseRegex.test('ひらがな')); // true

// 카테고리 기반 매칭
const letterRegex = /\p{Letter}+/u;  // 모든 문자(letter)
console.log(letterRegex.test('Hello'));  // true
console.log(letterRegex.test('한글'));    // true
console.log(letterRegex.test('123'));    // false

const numberRegex = /\p{Number}+/u;  // 모든 숫자
console.log(numberRegex.test('123'));    // true
console.log(numberRegex.test('abc'));    // false

// 실용적 예제: 다국어 텍스트 검증
const multilingualName = /^[\p{Letter}\s]+$/u;
console.log(multilingualName.test('John Smith'));    // true
console.log(multilingualName.test('김철수'));         // true
console.log(multilingualName.test('田中太郎'));       // true
console.log(multilingualName.test('Name123'));       // false
```

---

## 실전 패턴 예제

### 이메일 유효성 검사

```javascript
// 기본 이메일 검증
const basicEmail = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

// 더 엄격한 이메일 검증
const strictEmail = /^[a-zA-Z0-9.!#$%&'*+/=?^_`{|}~-]+@[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?(?:\.[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?)*$/;

function validateEmail(email) {
  return strictEmail.test(email);
}

console.log(validateEmail('user@example.com'));     // true
console.log(validateEmail('user.name@domain.co.kr')); // true
console.log(validateEmail('invalid@'));              // false
console.log(validateEmail('@domain.com'));           // false
```

### 전화번호 형식 검증

```javascript
// 한국 전화번호 (다양한 형식 지원)
const koreanPhone = /^(01[016789]|02|0[3-9]{1}[0-9]{1})-?(\d{3,4})-?(\d{4})$/;

function formatPhoneNumber(phone) {
  const cleaned = phone.replace(/\D/g, '');
  const match = cleaned.match(/^(01[016789]|02|0[3-9]{1}[0-9]{1})(\d{3,4})(\d{4})$/);

  if (match) {
    return `${match[1]}-${match[2]}-${match[3]}`;
  }
  return null;
}

console.log(formatPhoneNumber('01012345678'));   // '010-1234-5678'
console.log(formatPhoneNumber('021234567'));     // '02-123-4567'
console.log(formatPhoneNumber('031-123-4567'));  // '031-123-4567'
```

### URL 파싱

```javascript
// URL 구성 요소 추출
const urlRegex = /^(?<protocol>https?):\/\/(?<host>[^/:]+)(?::(?<port>\d+))?(?<path>\/[^?#]*)?(?:\?(?<query>[^#]*))?(?:#(?<hash>.*))?$/;

function parseUrl(url) {
  const match = url.match(urlRegex);
  if (!match) return null;

  return {
    protocol: match.groups.protocol,
    host: match.groups.host,
    port: match.groups.port || (match.groups.protocol === 'https' ? '443' : '80'),
    path: match.groups.path || '/',
    query: match.groups.query || '',
    hash: match.groups.hash || ''
  };
}

console.log(parseUrl('https://example.com:8080/path/to/page?id=123#section'));
// {
//   protocol: 'https',
//   host: 'example.com',
//   port: '8080',
//   path: '/path/to/page',
//   query: 'id=123',
//   hash: 'section'
// }
```

### 비밀번호 강도 검사

```javascript
// 비밀번호 규칙 검사기
const passwordRules = {
  minLength: /.{8,}/,
  hasUpperCase: /[A-Z]/,
  hasLowerCase: /[a-z]/,
  hasNumber: /\d/,
  hasSpecialChar: /[!@#$%^&*(),.?":{}|<>]/,
  noSpaces: /^\S+$/
};

function checkPasswordStrength(password) {
  const results = {
    minLength: passwordRules.minLength.test(password),
    hasUpperCase: passwordRules.hasUpperCase.test(password),
    hasLowerCase: passwordRules.hasLowerCase.test(password),
    hasNumber: passwordRules.hasNumber.test(password),
    hasSpecialChar: passwordRules.hasSpecialChar.test(password),
    noSpaces: passwordRules.noSpaces.test(password)
  };

  const passedRules = Object.values(results).filter(Boolean).length;
  const strength = passedRules <= 2 ? 'weak' : passedRules <= 4 ? 'medium' : 'strong';

  return { results, strength, score: passedRules };
}

console.log(checkPasswordStrength('Password1!'));
// { results: {...}, strength: 'strong', score: 6 }
```

### HTML 태그 처리

```javascript
// HTML 태그 제거
function stripHtmlTags(html) {
  return html.replace(/<[^>]*>/g, '');
}

console.log(stripHtmlTags('<p>Hello <strong>World</strong></p>'));
// 'Hello World'

// 특정 태그만 추출
function extractTags(html, tagName) {
  const regex = new RegExp(`<${tagName}[^>]*>([\\s\\S]*?)<\\/${tagName}>`, 'gi');
  const matches = [];
  let match;

  while ((match = regex.exec(html)) !== null) {
    matches.push({
      full: match[0],
      content: match[1]
    });
  }

  return matches;
}

const html = '<div>First</div><span>Skip</span><div>Second</div>';
console.log(extractTags(html, 'div'));
// [{ full: '<div>First</div>', content: 'First' }, ...]

// XSS 방지를 위한 스크립트 태그 제거
function sanitizeHtml(html) {
  return html
    .replace(/<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/gi, '')
    .replace(/on\w+="[^"]*"/gi, '')
    .replace(/on\w+='[^']*'/gi, '');
}
```

---

## 성능 고려사항

### 백트래킹 이해하기

정규표현식 엔진은 매칭 실패 시 **백트래킹(backtracking)**을 수행합니다.

```javascript
// 백트래킹 예시
const regex = /a+b/;
const str = 'aaaaaac';

// 엔진 동작:
// 1. a+ 가 'aaaaaa'를 매칭
// 2. b를 찾지만 'c'가 있음 -> 실패
// 3. 백트래킹: a+가 'aaaaa'만 매칭
// 4. 다음 문자 'a'에서 b 찾기 -> 실패
// 5. 반복... 결국 매칭 실패
```

### 치명적 백트래킹 방지

```javascript
// 나쁜 예: 치명적 백트래킹 발생 가능
const badRegex = /^(a+)+$/;

// 이 패턴은 'aaaaaaaaaaaaaaaaaaaaaaX' 같은 입력에서
// 기하급수적인 백트래킹 발생 (ReDoS 취약점)

// 좋은 예: 중첩 수량자 제거
const goodRegex = /^a+$/;

// 또는 소유 수량자 사용 (ES2024 제안 중, 현재 일부 환경만 지원)
// const atomicRegex = /^(?>a+)+$/;  // 백트래킹 방지

// 실용적 조언
function safeRegexTest(regex, str, timeout = 100) {
  const start = Date.now();

  // 입력 길이 제한
  if (str.length > 10000) {
    throw new Error('Input too long');
  }

  return regex.test(str);
}
```

### 성능 최적화 팁

```javascript
// 1. 정규표현식 재사용 (컴파일 비용 절약)
// 나쁜 예
function checkEmailBad(email) {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email); // 매번 컴파일
}

// 좋은 예
const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
function checkEmailGood(email) {
  return emailRegex.test(email); // 재사용
}

// 2. 구체적인 패턴 사용
// 나쁜 예
const slowRegex = /.*foo.*/;

// 좋은 예
const fastRegex = /foo/;

// 3. 앵커(^, $) 활용
// 앵커가 있으면 엔진이 불필요한 위치 검사 생략

// 4. 비캡처 그룹 사용
// 캡처가 필요 없으면 (?:...) 사용

// 5. 문자 클래스 최적화
// [0-9]보다 \d가 약간 빠를 수 있음 (엔진 의존)
```

---

## 정규표현식 디버깅 팁

### 단계적 테스트

```javascript
// 복잡한 패턴을 작은 단위로 분해하여 테스트
const fullPattern = /^(?<protocol>https?):\/\/(?<domain>[\w.-]+)(?<path>\/\S*)?$/;

// 단계별 테스트
const protocolTest = /^https?/;
const domainTest = /[\w.-]+/;
const pathTest = /\/\S*/;

const url = 'https://example.com/path';
console.log('Protocol:', protocolTest.test(url));
console.log('Domain:', domainTest.test('example.com'));
console.log('Path:', pathTest.test('/path'));
```

### 매칭 시각화

```javascript
// 매칭 위치 확인
function visualizeMatch(str, regex) {
  const result = str.match(regex);

  if (!result) {
    console.log('No match found');
    return;
  }

  const start = result.index;
  const end = start + result[0].length;

  console.log(str);
  console.log(' '.repeat(start) + '^'.repeat(result[0].length));
  console.log(`Matched: "${result[0]}" at index ${start}-${end - 1}`);
}

visualizeMatch('Hello World', /World/);
// Hello World
//       ^^^^^
// Matched: "World" at index 6-10
```

### 온라인 도구 활용

정규표현식 테스트와 디버깅에 유용한 도구들:

- **regex101.com**: 상세한 설명과 디버거 제공
- **regexr.com**: 시각적 매칭 확인
- **debuggex.com**: 정규표현식 시각화

```javascript
// 개발 중 패턴 테스트 함수
function testPattern(pattern, testCases) {
  console.log(`Testing: ${pattern}`);
  console.log('-'.repeat(40));

  testCases.forEach(({ input, expected }) => {
    const result = pattern.test(input);
    const status = result === expected ? 'PASS' : 'FAIL';
    console.log(`${status}: "${input}" -> ${result} (expected: ${expected})`);
  });
}

testPattern(/^[a-z]+@[a-z]+\.[a-z]+$/i, [
  { input: 'user@example.com', expected: true },
  { input: 'USER@EXAMPLE.COM', expected: true },
  { input: 'invalid', expected: false },
  { input: 'user@.com', expected: false }
]);
```

---

## 마치며

정규표현식은 처음에는 복잡해 보이지만, 기본 개념을 이해하고 나면 문자열 처리의 강력한 도구가 됩니다. 이 글에서 다룬 내용을 정리하면:

### 핵심 포인트

1. **생성 방법**: 리터럴(`/pattern/`)은 고정 패턴에, 생성자(`new RegExp()`)는 동적 패턴에 사용
2. **메타문자**: `.`, `*`, `+`, `?`, `^`, `$`, `[]`, `{}`, `()`, `|` 등 특별한 의미
3. **문자 클래스**: `\d`, `\w`, `\s`, `\b` 등 자주 쓰는 패턴의 단축형
4. **수량자**: 탐욕적(`*`, `+`) vs 게으른(`*?`, `+?`) 동작 이해
5. **그룹**: 캡처 그룹, 명명된 그룹, 비캡처 그룹 활용
6. **전방/후방탐색**: 패턴 주변 컨텍스트 확인
7. **유니코드**: `u` 플래그와 `\p{}` 프로퍼티로 다국어 지원
8. **성능**: 백트래킹 이해와 치명적 백트래킹 방지

### 추가 학습 자료

- Symbol을 활용한 고급 JavaScript 패턴: [JavaScript Symbol 완벽 가이드](/posts/javascript-symbol-complete-guide/)
- 문자열 기초: [JavaScript 문자열 연결과 타입 변환 완벽 가이드](/posts/javascript-string-concatenation/)

정규표현식은 연습이 중요합니다. 실제 프로젝트에서 유효성 검사, 텍스트 파싱, 검색/치환 등에 적극 활용해 보세요. 복잡한 패턴을 작성할 때는 반드시 다양한 테스트 케이스로 검증하고, 성능에 민감한 경우 입력 길이 제한과 타임아웃을 고려하세요.

---

## 참고 자료

- [MDN - Regular Expressions](https://developer.mozilla.org/ko/docs/Web/JavaScript/Guide/Regular_Expressions)
- [ECMAScript RegExp Specification](https://tc39.es/ecma262/#sec-regexp-regular-expression-objects)
- [Regular-Expressions.info](https://www.regular-expressions.info/)
- [regex101 - Online Regex Tester](https://regex101.com/)
