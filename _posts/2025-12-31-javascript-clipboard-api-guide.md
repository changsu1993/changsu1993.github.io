---
title: "JavaScript Clipboard API 완벽 가이드 - 복사/붙여넣기 기능 구현과 보안"
description: "JavaScript Clipboard API 완벽 가이드. writeText, readText로 텍스트 복사/붙여넣기 구현, ClipboardItem으로 이미지 처리, 보안 요구사항과 권한 관리, React useClipboard 커스텀 훅까지 실무 예제로 배웁니다."
date: 2025-12-31 09:00:00 +0900
categories: [Frontend, JavaScript]
tags: [JavaScript, Clipboard-API, Browser-API, Web-API, writeText, readText, ClipboardItem, 복사, 붙여넣기, React, TypeScript, Custom-Hooks, HTTPS, 보안, navigator-clipboard]
---

## 개요

웹 애플리케이션에서 복사/붙여넣기 기능은 사용자 경험을 크게 향상시키는 핵심 기능입니다. 코드 스니펫 복사 버튼, 공유 링크 복사, 이미지 붙여넣기 업로드 등 다양한 곳에서 활용됩니다. **Clipboard API**는 이러한 클립보드 작업을 비동기적이고 안전하게 처리할 수 있게 해주는 모던 브라우저 API입니다.

이 글에서는 Clipboard API의 기본 개념부터 텍스트/이미지 복사, 보안 요구사항, 이벤트 기반 처리, React에서의 활용법까지 실무에서 필요한 모든 내용을 다룹니다.

### 학습 목표

- Clipboard API의 구조와 동작 원리 이해
- 텍스트 및 이미지 복사/붙여넣기 구현 방법 습득
- 보안 요구사항과 권한 관리 방법 학습
- 클립보드 이벤트를 활용한 고급 기능 구현
- React에서 Clipboard API 활용 패턴 적용

### 사전 지식

- JavaScript 기초 문법
- [JavaScript Promise/async-await](/posts/javascript-promise-async-await-guide/) 이해
- 브라우저 보안 모델 기초 - [프론트엔드 보안 가이드](/posts/frontend-security-guide/) 참고

---

## Clipboard API 소개

### Clipboard API란?

Clipboard API는 시스템 클립보드에 대한 읽기/쓰기 접근을 제공하는 비동기 API입니다. 기존의 `document.execCommand()` 방식을 대체하며, Promise 기반으로 동작하여 더 안전하고 예측 가능한 클립보드 조작을 가능하게 합니다.

```javascript
// 기본 사용 예시
async function copyText() {
  try {
    await navigator.clipboard.writeText('복사할 텍스트');
    console.log('텍스트가 클립보드에 복사되었습니다.');
  } catch (err) {
    console.error('복사 실패:', err);
  }
}
```

### 왜 Clipboard API를 사용해야 하는가?

| 특징 | Clipboard API | document.execCommand() |
|------|---------------|------------------------|
| 동작 방식 | 비동기 (Promise) | 동기 |
| 이미지 지원 | 지원 | 미지원 |
| 보안 | 권한 기반, HTTPS 필수 | 제한적 |
| 에러 처리 | try-catch 가능 | 불가능 |
| 표준화 | 현대 표준 | Deprecated |
| 포커스 요구 | 필요 | 필요 |

> `document.execCommand()`는 더 이상 사용이 권장되지 않습니다(Deprecated). 새로운 프로젝트에서는 반드시 Clipboard API를 사용하세요.
{: .prompt-warning }

### navigator.clipboard 객체

Clipboard API는 `navigator.clipboard` 객체를 통해 접근합니다. 이 객체는 네 가지 주요 메서드를 제공합니다.

```javascript
// Clipboard API의 주요 메서드
navigator.clipboard.writeText(text);   // 텍스트 쓰기
navigator.clipboard.readText();        // 텍스트 읽기
navigator.clipboard.write(data);       // 다양한 형식 쓰기
navigator.clipboard.read();            // 다양한 형식 읽기
```

---

## 텍스트 복사하기 (writeText)

### 기본 사용법

`writeText()` 메서드는 문자열을 클립보드에 복사합니다. Promise를 반환하며, 복사 완료 시 resolve됩니다.

```javascript
async function copyToClipboard(text) {
  try {
    await navigator.clipboard.writeText(text);
    console.log('복사 완료!');
    return true;
  } catch (error) {
    console.error('복사 실패:', error.message);
    return false;
  }
}

// 사용 예시
copyToClipboard('Hello, World!');
```

### 실용적인 복사 함수

사용자 피드백과 에러 처리를 포함한 실용적인 복사 함수입니다.

```typescript
interface CopyResult {
  success: boolean;
  message: string;
}

async function copyWithFeedback(text: string): Promise<CopyResult> {
  // Clipboard API 지원 여부 확인
  if (!navigator.clipboard) {
    return {
      success: false,
      message: '이 브라우저는 클립보드 API를 지원하지 않습니다.'
    };
  }

  try {
    await navigator.clipboard.writeText(text);
    return {
      success: true,
      message: '클립보드에 복사되었습니다!'
    };
  } catch (error) {
    // 권한 거부 또는 기타 오류
    if (error instanceof Error) {
      if (error.name === 'NotAllowedError') {
        return {
          success: false,
          message: '클립보드 접근 권한이 거부되었습니다.'
        };
      }
      return {
        success: false,
        message: `복사 실패: ${error.message}`
      };
    }
    return {
      success: false,
      message: '알 수 없는 오류가 발생했습니다.'
    };
  }
}

// 사용 예시
const result = await copyWithFeedback('복사할 내용');
if (result.success) {
  showToast(result.message, 'success');
} else {
  showToast(result.message, 'error');
}
```

### 코드 복사 버튼 구현

기술 블로그나 문서 사이트에서 자주 사용되는 코드 복사 버튼을 구현해보겠습니다.

```typescript
class CodeCopyButton {
  private button: HTMLButtonElement;
  private codeBlock: HTMLElement;
  private originalText: string;
  private timeoutId: number | null = null;

  constructor(codeBlock: HTMLElement) {
    this.codeBlock = codeBlock;
    this.button = this.createButton();
    this.originalText = '복사';
    this.attachToCodeBlock();
  }

  private createButton(): HTMLButtonElement {
    const button = document.createElement('button');
    button.className = 'code-copy-btn';
    button.textContent = this.originalText;
    button.setAttribute('aria-label', '코드 복사');
    button.addEventListener('click', () => this.handleCopy());
    return button;
  }

  private attachToCodeBlock(): void {
    const wrapper = document.createElement('div');
    wrapper.className = 'code-block-wrapper';
    wrapper.style.position = 'relative';

    this.codeBlock.parentNode?.insertBefore(wrapper, this.codeBlock);
    wrapper.appendChild(this.codeBlock);
    wrapper.appendChild(this.button);
  }

  private async handleCopy(): Promise<void> {
    const code = this.codeBlock.textContent || '';

    try {
      await navigator.clipboard.writeText(code);
      this.showSuccess();
    } catch (error) {
      this.showError();
    }
  }

  private showSuccess(): void {
    this.button.textContent = '복사됨!';
    this.button.classList.add('copied');
    this.resetButtonAfterDelay();
  }

  private showError(): void {
    this.button.textContent = '실패';
    this.button.classList.add('error');
    this.resetButtonAfterDelay();
  }

  private resetButtonAfterDelay(): void {
    if (this.timeoutId) {
      clearTimeout(this.timeoutId);
    }

    this.timeoutId = window.setTimeout(() => {
      this.button.textContent = this.originalText;
      this.button.classList.remove('copied', 'error');
    }, 2000);
  }
}

// 모든 코드 블록에 복사 버튼 추가
document.querySelectorAll('pre code').forEach((codeBlock) => {
  new CodeCopyButton(codeBlock as HTMLElement);
});
```

```css
.code-block-wrapper {
  position: relative;
}

.code-copy-btn {
  position: absolute;
  top: 8px;
  right: 8px;
  padding: 4px 12px;
  font-size: 12px;
  background: rgba(255, 255, 255, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 4px;
  color: #fff;
  cursor: pointer;
  transition: all 0.2s ease;
}

.code-copy-btn:hover {
  background: rgba(255, 255, 255, 0.2);
}

.code-copy-btn.copied {
  background: #10b981;
  border-color: #10b981;
}

.code-copy-btn.error {
  background: #ef4444;
  border-color: #ef4444;
}
```

---

## 텍스트 읽기 (readText)

### 기본 사용법

`readText()` 메서드는 클립보드의 텍스트 내용을 읽어옵니다. 이 작업은 사용자 권한이 필요하며, 브라우저에 따라 권한 요청 팝업이 표시될 수 있습니다.

```javascript
async function pasteFromClipboard() {
  try {
    const text = await navigator.clipboard.readText();
    console.log('클립보드 내용:', text);
    return text;
  } catch (error) {
    console.error('붙여넣기 실패:', error);
    return null;
  }
}
```

### 클립보드 읽기 예제

입력 필드에 클립보드 내용을 붙여넣는 버튼을 구현합니다.

```typescript
interface PasteInputOptions {
  input: HTMLInputElement | HTMLTextAreaElement;
  pasteButton: HTMLButtonElement;
  onPaste?: (text: string) => void;
  onError?: (error: Error) => void;
}

function setupPasteInput(options: PasteInputOptions): void {
  const { input, pasteButton, onPaste, onError } = options;

  pasteButton.addEventListener('click', async () => {
    try {
      const text = await navigator.clipboard.readText();

      // 입력 필드에 값 설정
      input.value = text;

      // 입력 이벤트 트리거 (React 등에서 상태 업데이트를 위해)
      input.dispatchEvent(new Event('input', { bubbles: true }));

      onPaste?.(text);
    } catch (error) {
      if (error instanceof Error) {
        onError?.(error);
      }
    }
  });
}

// 사용 예시
const input = document.getElementById('url-input') as HTMLInputElement;
const pasteBtn = document.getElementById('paste-btn') as HTMLButtonElement;

setupPasteInput({
  input,
  pasteButton: pasteBtn,
  onPaste: (text) => {
    console.log('붙여넣기 완료:', text);
  },
  onError: (error) => {
    if (error.name === 'NotAllowedError') {
      alert('클립보드 읽기 권한이 필요합니다.');
    }
  }
});
```

> 클립보드 읽기는 보안상 민감한 작업입니다. 브라우저마다 권한 요구사항이 다르며, 일부 브라우저에서는 사용자 제스처(클릭 등) 없이는 작동하지 않습니다.
{: .prompt-info }

---

## 이미지 및 파일 복사하기 (write)

### ClipboardItem 이해하기

`write()` 메서드는 텍스트뿐만 아니라 이미지, HTML 등 다양한 형식의 데이터를 클립보드에 쓸 수 있습니다. 이를 위해 `ClipboardItem` 객체를 사용합니다.

```javascript
// ClipboardItem 기본 구조
const clipboardItem = new ClipboardItem({
  'text/plain': new Blob(['텍스트 내용'], { type: 'text/plain' }),
  'text/html': new Blob(['<b>HTML 내용</b>'], { type: 'text/html' })
});

await navigator.clipboard.write([clipboardItem]);
```

### 지원되는 MIME 타입

| MIME 타입 | 설명 | 지원 여부 |
|-----------|------|-----------|
| `text/plain` | 일반 텍스트 | 항상 지원 |
| `text/html` | HTML 콘텐츠 | 항상 지원 |
| `image/png` | PNG 이미지 | 항상 지원 |
| `image/svg+xml` | SVG 이미지 | 브라우저별 상이 |
| `web ` prefix | 커스텀 타입 | 브라우저별 상이 |

### 이미지 복사하기

캔버스나 이미지 요소의 내용을 클립보드에 복사하는 방법입니다.

```typescript
// 이미지 URL에서 클립보드로 복사
async function copyImageFromUrl(imageUrl: string): Promise<boolean> {
  try {
    // 이미지 데이터 가져오기
    const response = await fetch(imageUrl);
    const blob = await response.blob();

    // PNG 타입 지원 확인
    if (!ClipboardItem.supports('image/png')) {
      console.error('이 브라우저는 이미지 복사를 지원하지 않습니다.');
      return false;
    }

    // 클립보드에 쓰기
    const clipboardItem = new ClipboardItem({
      [blob.type]: blob
    });

    await navigator.clipboard.write([clipboardItem]);
    console.log('이미지가 클립보드에 복사되었습니다.');
    return true;
  } catch (error) {
    console.error('이미지 복사 실패:', error);
    return false;
  }
}

// 사용 예시
await copyImageFromUrl('/images/logo.png');
```

### Canvas를 클립보드에 복사

```typescript
async function copyCanvasToClipboard(canvas: HTMLCanvasElement): Promise<boolean> {
  return new Promise((resolve) => {
    canvas.toBlob(async (blob) => {
      if (!blob) {
        console.error('Canvas를 Blob으로 변환할 수 없습니다.');
        resolve(false);
        return;
      }

      try {
        const clipboardItem = new ClipboardItem({
          'image/png': blob
        });

        await navigator.clipboard.write([clipboardItem]);
        console.log('Canvas가 클립보드에 복사되었습니다.');
        resolve(true);
      } catch (error) {
        console.error('복사 실패:', error);
        resolve(false);
      }
    }, 'image/png');
  });
}

// 사용 예시
const canvas = document.getElementById('my-canvas') as HTMLCanvasElement;
const ctx = canvas.getContext('2d');

if (ctx) {
  ctx.fillStyle = 'cornflowerblue';
  ctx.fillRect(0, 0, 200, 200);
  ctx.fillStyle = 'white';
  ctx.font = '20px sans-serif';
  ctx.fillText('Hello Canvas!', 30, 110);
}

document.getElementById('copy-canvas-btn')?.addEventListener('click', () => {
  copyCanvasToClipboard(canvas);
});
```

### 다중 형식으로 복사하기

하나의 클립보드 항목에 여러 형식을 포함할 수 있습니다. 붙여넣는 애플리케이션이 지원하는 형식을 선택하여 사용합니다.

```typescript
async function copyRichContent(text: string, html: string): Promise<void> {
  const clipboardItem = new ClipboardItem({
    'text/plain': new Blob([text], { type: 'text/plain' }),
    'text/html': new Blob([html], { type: 'text/html' })
  });

  try {
    await navigator.clipboard.write([clipboardItem]);
    console.log('리치 콘텐츠가 복사되었습니다.');
  } catch (error) {
    console.error('복사 실패:', error);
  }
}

// 사용 예시
await copyRichContent(
  '굵은 텍스트와 이탤릭 텍스트',
  '<p><strong>굵은 텍스트</strong>와 <em>이탤릭 텍스트</em></p>'
);
```

---

## 클립보드 읽기 (read)

### 다양한 형식 읽기

`read()` 메서드는 클립보드의 모든 데이터 형식을 읽을 수 있습니다. `ClipboardItem` 배열을 반환합니다.

```typescript
async function readClipboardContents(): Promise<void> {
  try {
    const clipboardItems = await navigator.clipboard.read();

    for (const item of clipboardItems) {
      console.log('사용 가능한 타입:', item.types);

      for (const type of item.types) {
        const blob = await item.getType(type);

        if (type.startsWith('text/')) {
          const text = await blob.text();
          console.log(`${type}:`, text);
        } else if (type.startsWith('image/')) {
          const url = URL.createObjectURL(blob);
          console.log(`${type}: 이미지 URL -`, url);
        }
      }
    }
  } catch (error) {
    console.error('클립보드 읽기 실패:', error);
  }
}
```

### 이미지 붙여넣기 구현

이미지 업로더에서 클립보드의 이미지를 직접 붙여넣는 기능을 구현합니다.

```typescript
interface ImagePasteResult {
  success: boolean;
  imageUrl?: string;
  blob?: Blob;
  error?: string;
}

async function pasteImageFromClipboard(): Promise<ImagePasteResult> {
  try {
    const clipboardItems = await navigator.clipboard.read();

    for (const item of clipboardItems) {
      // 이미지 타입 찾기
      const imageType = item.types.find((type) => type.startsWith('image/'));

      if (imageType) {
        const blob = await item.getType(imageType);
        const imageUrl = URL.createObjectURL(blob);

        return {
          success: true,
          imageUrl,
          blob
        };
      }
    }

    return {
      success: false,
      error: '클립보드에 이미지가 없습니다.'
    };
  } catch (error) {
    return {
      success: false,
      error: error instanceof Error ? error.message : '알 수 없는 오류'
    };
  }
}

// 이미지 미리보기 표시
async function handlePasteImage(): Promise<void> {
  const result = await pasteImageFromClipboard();

  if (result.success && result.imageUrl) {
    const preview = document.getElementById('image-preview') as HTMLImageElement;
    preview.src = result.imageUrl;
    preview.style.display = 'block';

    // 서버 업로드가 필요한 경우 blob 사용
    if (result.blob) {
      // uploadImage(result.blob);
    }
  } else {
    alert(result.error);
  }
}
```

### HTML 콘텐츠 읽기

```typescript
async function readHtmlFromClipboard(): Promise<string | null> {
  try {
    const clipboardItems = await navigator.clipboard.read();

    for (const item of clipboardItems) {
      if (item.types.includes('text/html')) {
        const blob = await item.getType('text/html');
        return await blob.text();
      }
    }

    return null;
  } catch (error) {
    console.error('HTML 읽기 실패:', error);
    return null;
  }
}

// 사용 예시
const html = await readHtmlFromClipboard();
if (html) {
  document.getElementById('editor')!.innerHTML = html;
}
```

---

## 보안 고려사항

### HTTPS 요구사항

Clipboard API는 **보안 컨텍스트(Secure Context)**에서만 사용할 수 있습니다. 이는 다음을 의미합니다.

- HTTPS 프로토콜 필수 (개발 시 localhost 제외)
- HTTP에서는 `navigator.clipboard`가 `undefined`

```javascript
// 보안 컨텍스트 확인
if (window.isSecureContext) {
  console.log('보안 컨텍스트입니다. Clipboard API 사용 가능.');
} else {
  console.log('보안 컨텍스트가 아닙니다. HTTPS가 필요합니다.');
}

// Clipboard API 지원 확인
if (navigator.clipboard) {
  console.log('Clipboard API를 사용할 수 있습니다.');
} else {
  console.log('Clipboard API를 사용할 수 없습니다.');
}
```

### 권한 요구사항

브라우저마다 클립보드 권한 처리 방식이 다릅니다.

| 브라우저 | 쓰기 (writeText) | 읽기 (readText) |
|----------|------------------|-----------------|
| Chrome | 사용자 제스처 또는 권한 | clipboard-read 권한 필요 |
| Firefox | 사용자 제스처 필요 | 사용자 제스처 필요 |
| Safari | 사용자 제스처 필요 | 사용자 제스처 필요 |

### Permissions API로 권한 확인

```typescript
interface ClipboardPermissionStatus {
  read: PermissionState | 'unsupported';
  write: PermissionState | 'unsupported';
}

async function checkClipboardPermissions(): Promise<ClipboardPermissionStatus> {
  const result: ClipboardPermissionStatus = {
    read: 'unsupported',
    write: 'unsupported'
  };

  try {
    // 읽기 권한 확인
    const readPermission = await navigator.permissions.query({
      name: 'clipboard-read' as PermissionName
    });
    result.read = readPermission.state;

    // 쓰기 권한 확인
    const writePermission = await navigator.permissions.query({
      name: 'clipboard-write' as PermissionName
    });
    result.write = writePermission.state;
  } catch (error) {
    // Firefox, Safari는 이 권한 쿼리를 지원하지 않음
    console.log('권한 쿼리가 지원되지 않습니다.');
  }

  return result;
}

// 사용 예시
const permissions = await checkClipboardPermissions();
console.log('읽기 권한:', permissions.read);
console.log('쓰기 권한:', permissions.write);

// 권한 상태: 'granted', 'denied', 'prompt', 'unsupported'
```

### 권한 변경 감지

```typescript
async function watchClipboardPermission(
  onChange: (state: PermissionState) => void
): Promise<void> {
  try {
    const permission = await navigator.permissions.query({
      name: 'clipboard-read' as PermissionName
    });

    // 현재 상태 전달
    onChange(permission.state);

    // 상태 변경 감지
    permission.addEventListener('change', () => {
      onChange(permission.state);
    });
  } catch (error) {
    console.log('권한 감시가 지원되지 않습니다.');
  }
}

// 사용 예시
watchClipboardPermission((state) => {
  if (state === 'granted') {
    enablePasteButton();
  } else if (state === 'denied') {
    disablePasteButton();
    showPermissionDeniedMessage();
  }
});
```

### 문서 포커스 요구사항

Clipboard API의 메서드들은 문서가 포커스된 상태에서만 작동합니다.

```typescript
function isDocumentFocused(): boolean {
  return document.hasFocus();
}

async function safeCopy(text: string): Promise<boolean> {
  if (!document.hasFocus()) {
    console.warn('문서가 포커스되어 있지 않습니다.');
    return false;
  }

  try {
    await navigator.clipboard.writeText(text);
    return true;
  } catch (error) {
    return false;
  }
}
```

> 백그라운드 탭이나 iframe에서 클립보드 작업을 시도하면 실패할 수 있습니다. 항상 사용자 인터랙션(클릭 등)에 응답하여 클립보드 작업을 수행하세요.
{: .prompt-warning }

---

## 클립보드 이벤트

### copy, cut, paste 이벤트

브라우저는 사용자가 복사, 잘라내기, 붙여넣기 동작을 수행할 때 이벤트를 발생시킵니다. 이 이벤트를 가로채서 커스텀 동작을 구현할 수 있습니다.

```typescript
// 복사 이벤트 가로채기
document.addEventListener('copy', (event: ClipboardEvent) => {
  event.preventDefault();

  const selection = window.getSelection()?.toString() || '';

  // 커스텀 데이터 설정
  event.clipboardData?.setData('text/plain', selection);
  event.clipboardData?.setData(
    'text/html',
    `<div style="color: blue;">${selection}</div>`
  );

  console.log('복사됨:', selection);
});

// 잘라내기 이벤트 가로채기
document.addEventListener('cut', (event: ClipboardEvent) => {
  event.preventDefault();

  const selection = window.getSelection();
  const text = selection?.toString() || '';

  event.clipboardData?.setData('text/plain', text);

  // 선택된 텍스트 삭제
  selection?.deleteFromDocument();

  console.log('잘라내기:', text);
});

// 붙여넣기 이벤트 가로채기
document.addEventListener('paste', (event: ClipboardEvent) => {
  event.preventDefault();

  const text = event.clipboardData?.getData('text/plain') || '';
  const html = event.clipboardData?.getData('text/html') || '';

  console.log('붙여넣기 텍스트:', text);
  console.log('붙여넣기 HTML:', html);

  // 커스텀 붙여넣기 처리
  document.execCommand('insertText', false, text);
});
```

### ClipboardEvent 인터페이스

```typescript
interface ClipboardEvent extends Event {
  readonly clipboardData: DataTransfer | null;
}

// DataTransfer 주요 메서드
interface DataTransfer {
  setData(format: string, data: string): void;  // 데이터 설정
  getData(format: string): string;               // 데이터 가져오기
  clearData(format?: string): void;              // 데이터 삭제
  readonly types: ReadonlyArray<string>;         // 사용 가능한 형식
  readonly files: FileList;                      // 파일 목록
  readonly items: DataTransferItemList;          // 데이터 항목 목록
}
```

### 이미지 붙여넣기 이벤트 처리

```typescript
function setupImagePasteHandler(
  targetElement: HTMLElement,
  onImagePaste: (file: File) => void
): void {
  targetElement.addEventListener('paste', (event: ClipboardEvent) => {
    const items = event.clipboardData?.items;
    if (!items) return;

    for (const item of items) {
      if (item.type.startsWith('image/')) {
        event.preventDefault();

        const file = item.getAsFile();
        if (file) {
          onImagePaste(file);
        }
        return;
      }
    }
  });
}

// 사용 예시
const dropzone = document.getElementById('dropzone')!;

setupImagePasteHandler(dropzone, (file) => {
  // 이미지 미리보기 생성
  const reader = new FileReader();
  reader.onload = (e) => {
    const img = document.createElement('img');
    img.src = e.target?.result as string;
    img.style.maxWidth = '300px';
    dropzone.appendChild(img);
  };
  reader.readAsDataURL(file);

  // 서버 업로드
  // uploadFile(file);
});
```

### 특정 요소에서만 이벤트 처리

```typescript
class RichTextEditor {
  private element: HTMLElement;

  constructor(element: HTMLElement) {
    this.element = element;
    this.setupEventListeners();
  }

  private setupEventListeners(): void {
    // 에디터 내에서만 이벤트 처리
    this.element.addEventListener('copy', this.handleCopy.bind(this));
    this.element.addEventListener('paste', this.handlePaste.bind(this));
  }

  private handleCopy(event: ClipboardEvent): void {
    const selection = window.getSelection();
    if (!selection || selection.rangeCount === 0) return;

    // 선택 영역이 에디터 내부인지 확인
    if (!this.element.contains(selection.anchorNode)) return;

    event.preventDefault();

    const range = selection.getRangeAt(0);
    const fragment = range.cloneContents();

    // 텍스트 버전
    const text = selection.toString();

    // HTML 버전
    const div = document.createElement('div');
    div.appendChild(fragment);
    const html = div.innerHTML;

    event.clipboardData?.setData('text/plain', text);
    event.clipboardData?.setData('text/html', html);
  }

  private handlePaste(event: ClipboardEvent): void {
    event.preventDefault();

    // HTML이 있으면 HTML 사용, 없으면 텍스트 사용
    const html = event.clipboardData?.getData('text/html');
    const text = event.clipboardData?.getData('text/plain');

    if (html) {
      // HTML 정화 후 삽입 (XSS 방지)
      const sanitized = this.sanitizeHtml(html);
      document.execCommand('insertHTML', false, sanitized);
    } else if (text) {
      document.execCommand('insertText', false, text);
    }
  }

  private sanitizeHtml(html: string): string {
    // 허용된 태그만 남기고 나머지 제거
    const allowedTags = ['p', 'br', 'strong', 'em', 'u', 'a', 'ul', 'ol', 'li'];
    const doc = new DOMParser().parseFromString(html, 'text/html');

    // 간단한 정화 로직 (실제로는 DOMPurify 같은 라이브러리 사용 권장)
    const walker = document.createTreeWalker(
      doc.body,
      NodeFilter.SHOW_ELEMENT,
      null
    );

    const nodesToRemove: Node[] = [];
    while (walker.nextNode()) {
      const node = walker.currentNode as Element;
      if (!allowedTags.includes(node.tagName.toLowerCase())) {
        nodesToRemove.push(node);
      }
    }

    nodesToRemove.forEach((node) => {
      const parent = node.parentNode;
      while (node.firstChild) {
        parent?.insertBefore(node.firstChild, node);
      }
      parent?.removeChild(node);
    });

    return doc.body.innerHTML;
  }
}

// 사용 예시
const editor = document.getElementById('editor')!;
editor.setAttribute('contenteditable', 'true');
new RichTextEditor(editor);
```

---

## 레거시 방식과 비교

### document.execCommand (Deprecated)

`document.execCommand()`는 오래된 클립보드 접근 방식입니다. 여전히 일부 브라우저에서 작동하지만 더 이상 권장되지 않습니다.

```javascript
// 레거시 방식 - 더 이상 권장되지 않음
function legacyCopy(text) {
  const textarea = document.createElement('textarea');
  textarea.value = text;
  textarea.style.position = 'fixed';
  textarea.style.opacity = '0';
  document.body.appendChild(textarea);
  textarea.select();

  try {
    const success = document.execCommand('copy');
    if (success) {
      console.log('복사 성공 (레거시)');
    } else {
      console.log('복사 실패');
    }
  } catch (err) {
    console.error('복사 오류:', err);
  }

  document.body.removeChild(textarea);
}
```

### 폴백 패턴

구형 브라우저를 지원해야 할 경우 폴백 패턴을 사용할 수 있습니다.

```typescript
async function copyWithFallback(text: string): Promise<boolean> {
  // 모던 API 먼저 시도
  if (navigator.clipboard && window.isSecureContext) {
    try {
      await navigator.clipboard.writeText(text);
      return true;
    } catch (error) {
      console.warn('Clipboard API 실패, 폴백 시도:', error);
    }
  }

  // 레거시 폴백
  return legacyCopyFallback(text);
}

function legacyCopyFallback(text: string): boolean {
  const textarea = document.createElement('textarea');
  textarea.value = text;

  // 화면 밖으로 위치 (스크롤 방지)
  textarea.style.cssText = `
    position: fixed;
    top: 0;
    left: 0;
    width: 2em;
    height: 2em;
    padding: 0;
    border: none;
    outline: none;
    box-shadow: none;
    background: transparent;
  `;

  document.body.appendChild(textarea);
  textarea.focus();
  textarea.select();

  let success = false;
  try {
    success = document.execCommand('copy');
  } catch (err) {
    console.error('execCommand 실패:', err);
  }

  document.body.removeChild(textarea);
  return success;
}
```

### 비교 요약

| 기능 | Clipboard API | execCommand |
|------|---------------|-------------|
| 비동기 | Promise 기반 | 동기 |
| 이미지 지원 | 지원 | 미지원 |
| 에러 처리 | try-catch | 불가 |
| 보안 | HTTPS, 권한 기반 | 제한적 |
| 포커스 요구 | 문서 포커스 필요 | 요소 선택 필요 |
| 상태 | 현대 표준 | Deprecated |
| IE 지원 | 미지원 | 지원 |

---

## React에서 Clipboard API 활용

React에서 Clipboard API를 효과적으로 사용하기 위한 패턴을 알아봅니다. 커스텀 훅에 대한 더 다양한 패턴은 [React Custom Hooks 패턴 가이드](/posts/react-custom-hooks-patterns-guide/)를 참고하세요.

### 기본 복사 컴포넌트

```tsx
import { useState, useCallback } from 'react';

interface CopyButtonProps {
  text: string;
  onCopy?: () => void;
  onError?: (error: Error) => void;
}

function CopyButton({ text, onCopy, onError }: CopyButtonProps) {
  const [copied, setCopied] = useState(false);

  const handleCopy = useCallback(async () => {
    try {
      await navigator.clipboard.writeText(text);
      setCopied(true);
      onCopy?.();

      // 2초 후 상태 리셋
      setTimeout(() => setCopied(false), 2000);
    } catch (error) {
      if (error instanceof Error) {
        onError?.(error);
      }
    }
  }, [text, onCopy, onError]);

  return (
    <button
      onClick={handleCopy}
      aria-label={copied ? '복사됨' : '클립보드에 복사'}
    >
      {copied ? '복사됨!' : '복사'}
    </button>
  );
}

// 사용 예시
function ShareLink() {
  const shareUrl = 'https://example.com/shared-content';

  return (
    <div>
      <input type="text" value={shareUrl} readOnly />
      <CopyButton
        text={shareUrl}
        onCopy={() => console.log('링크가 복사되었습니다.')}
        onError={(error) => console.error('복사 실패:', error)}
      />
    </div>
  );
}
```

### useClipboard 커스텀 훅

재사용 가능한 클립보드 훅을 만들어봅니다.

```tsx
import { useState, useCallback, useEffect, useRef } from 'react';

interface UseClipboardOptions {
  timeout?: number;
  onSuccess?: () => void;
  onError?: (error: Error) => void;
}

interface UseClipboardReturn {
  copy: (text: string) => Promise<boolean>;
  paste: () => Promise<string | null>;
  copied: boolean;
  error: Error | null;
  isSupported: boolean;
}

function useClipboard(options: UseClipboardOptions = {}): UseClipboardReturn {
  const { timeout = 2000, onSuccess, onError } = options;

  const [copied, setCopied] = useState(false);
  const [error, setError] = useState<Error | null>(null);
  const timeoutRef = useRef<NodeJS.Timeout>();

  const isSupported = Boolean(
    navigator.clipboard && window.isSecureContext
  );

  const copy = useCallback(async (text: string): Promise<boolean> => {
    if (!isSupported) {
      const err = new Error('Clipboard API가 지원되지 않습니다.');
      setError(err);
      onError?.(err);
      return false;
    }

    try {
      await navigator.clipboard.writeText(text);
      setCopied(true);
      setError(null);
      onSuccess?.();

      // 타임아웃 초기화
      if (timeoutRef.current) {
        clearTimeout(timeoutRef.current);
      }
      timeoutRef.current = setTimeout(() => {
        setCopied(false);
      }, timeout);

      return true;
    } catch (err) {
      const error = err instanceof Error ? err : new Error('복사 실패');
      setError(error);
      setCopied(false);
      onError?.(error);
      return false;
    }
  }, [isSupported, timeout, onSuccess, onError]);

  const paste = useCallback(async (): Promise<string | null> => {
    if (!isSupported) {
      const err = new Error('Clipboard API가 지원되지 않습니다.');
      setError(err);
      onError?.(err);
      return null;
    }

    try {
      const text = await navigator.clipboard.readText();
      setError(null);
      return text;
    } catch (err) {
      const error = err instanceof Error ? err : new Error('붙여넣기 실패');
      setError(error);
      onError?.(error);
      return null;
    }
  }, [isSupported, onError]);

  // 클린업
  useEffect(() => {
    return () => {
      if (timeoutRef.current) {
        clearTimeout(timeoutRef.current);
      }
    };
  }, []);

  return { copy, paste, copied, error, isSupported };
}

// 사용 예시
function CodeSnippet({ code }: { code: string }) {
  const { copy, copied, error, isSupported } = useClipboard({
    timeout: 3000,
    onSuccess: () => console.log('코드가 복사되었습니다.'),
    onError: (err) => console.error('복사 오류:', err)
  });

  if (!isSupported) {
    return <pre>{code}</pre>;
  }

  return (
    <div className="code-snippet">
      <pre>{code}</pre>
      <button onClick={() => copy(code)} disabled={copied}>
        {copied ? '복사됨!' : '코드 복사'}
      </button>
      {error && <span className="error">{error.message}</span>}
    </div>
  );
}
```

### 이미지 붙여넣기 컴포넌트

```tsx
import { useState, useCallback, DragEvent, ClipboardEvent } from 'react';

interface ImageFile {
  file: File;
  preview: string;
}

interface ImageDropzoneProps {
  onImageSelect: (image: ImageFile) => void;
  accept?: string[];
  maxSize?: number;
}

function ImageDropzone({
  onImageSelect,
  accept = ['image/png', 'image/jpeg', 'image/gif'],
  maxSize = 5 * 1024 * 1024 // 5MB
}: ImageDropzoneProps) {
  const [isDragging, setIsDragging] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const validateFile = useCallback((file: File): string | null => {
    if (!accept.includes(file.type)) {
      return `지원되지 않는 파일 형식입니다. (${accept.join(', ')})`;
    }
    if (file.size > maxSize) {
      return `파일 크기가 ${maxSize / 1024 / 1024}MB를 초과합니다.`;
    }
    return null;
  }, [accept, maxSize]);

  const handleFile = useCallback((file: File) => {
    const validationError = validateFile(file);
    if (validationError) {
      setError(validationError);
      return;
    }

    setError(null);
    const preview = URL.createObjectURL(file);
    onImageSelect({ file, preview });
  }, [validateFile, onImageSelect]);

  const handlePaste = useCallback((event: ClipboardEvent) => {
    const items = event.clipboardData?.items;
    if (!items) return;

    for (const item of items) {
      if (item.type.startsWith('image/')) {
        event.preventDefault();
        const file = item.getAsFile();
        if (file) {
          handleFile(file);
        }
        return;
      }
    }
  }, [handleFile]);

  const handleDrop = useCallback((event: DragEvent) => {
    event.preventDefault();
    setIsDragging(false);

    const file = event.dataTransfer?.files[0];
    if (file && file.type.startsWith('image/')) {
      handleFile(file);
    }
  }, [handleFile]);

  const handleDragOver = useCallback((event: DragEvent) => {
    event.preventDefault();
    setIsDragging(true);
  }, []);

  const handleDragLeave = useCallback(() => {
    setIsDragging(false);
  }, []);

  // Clipboard API로 붙여넣기 버튼
  const handlePasteButton = useCallback(async () => {
    try {
      const clipboardItems = await navigator.clipboard.read();

      for (const item of clipboardItems) {
        const imageType = item.types.find((type) =>
          type.startsWith('image/')
        );

        if (imageType) {
          const blob = await item.getType(imageType);
          const file = new File([blob], 'pasted-image.png', {
            type: imageType
          });
          handleFile(file);
          return;
        }
      }

      setError('클립보드에 이미지가 없습니다.');
    } catch (err) {
      if (err instanceof Error && err.name === 'NotAllowedError') {
        setError('클립보드 접근 권한이 필요합니다.');
      } else {
        setError('이미지를 가져올 수 없습니다.');
      }
    }
  }, [handleFile]);

  return (
    <div
      className={`dropzone ${isDragging ? 'dragging' : ''}`}
      onPaste={handlePaste}
      onDrop={handleDrop}
      onDragOver={handleDragOver}
      onDragLeave={handleDragLeave}
      tabIndex={0}
    >
      <p>이미지를 드래그하거나 Ctrl+V로 붙여넣기</p>
      <button type="button" onClick={handlePasteButton}>
        클립보드에서 붙여넣기
      </button>
      {error && <p className="error">{error}</p>}
    </div>
  );
}

// 사용 예시
function ImageUploader() {
  const [image, setImage] = useState<ImageFile | null>(null);

  return (
    <div>
      <ImageDropzone onImageSelect={setImage} />
      {image && (
        <div className="preview">
          <img src={image.preview} alt="미리보기" />
          <p>{image.file.name}</p>
        </div>
      )}
    </div>
  );
}
```

### 공유 URL 복사 컴포넌트

```tsx
import { useState, useCallback } from 'react';

interface ShareButtonProps {
  title: string;
  text?: string;
  url: string;
}

function ShareButton({ title, text, url }: ShareButtonProps) {
  const [showCopied, setShowCopied] = useState(false);

  // Web Share API 지원 확인
  const canShare = Boolean(navigator.share);

  const handleShare = useCallback(async () => {
    if (canShare) {
      try {
        await navigator.share({ title, text, url });
      } catch (error) {
        // 사용자가 공유 취소한 경우 무시
        if (error instanceof Error && error.name !== 'AbortError') {
          console.error('공유 실패:', error);
        }
      }
    } else {
      // 공유 API 미지원 시 클립보드에 복사
      try {
        await navigator.clipboard.writeText(url);
        setShowCopied(true);
        setTimeout(() => setShowCopied(false), 2000);
      } catch (error) {
        console.error('복사 실패:', error);
      }
    }
  }, [canShare, title, text, url]);

  return (
    <button onClick={handleShare} aria-label="공유하기">
      {showCopied ? '링크 복사됨!' : canShare ? '공유하기' : '링크 복사'}
    </button>
  );
}
```

---

## 브라우저 호환성

### 지원 현황

| API | Chrome | Firefox | Safari | Edge |
|-----|--------|---------|--------|------|
| `writeText()` | 66+ | 63+ | 13.1+ | 79+ |
| `readText()` | 66+ | 125+ | 13.1+ | 79+ |
| `write()` | 76+ | 127+ | 13.1+ | 79+ |
| `read()` | 76+ | 127+ | 13.1+ | 79+ |
| `ClipboardItem` | 76+ | 127+ | 13.1+ | 79+ |

### 기능 감지

```typescript
interface ClipboardSupport {
  clipboard: boolean;
  writeText: boolean;
  readText: boolean;
  write: boolean;
  read: boolean;
  clipboardItem: boolean;
}

function checkClipboardSupport(): ClipboardSupport {
  const clipboard = Boolean(navigator.clipboard);

  return {
    clipboard,
    writeText: clipboard && typeof navigator.clipboard.writeText === 'function',
    readText: clipboard && typeof navigator.clipboard.readText === 'function',
    write: clipboard && typeof navigator.clipboard.write === 'function',
    read: clipboard && typeof navigator.clipboard.read === 'function',
    clipboardItem: typeof ClipboardItem !== 'undefined'
  };
}

// 사용 예시
const support = checkClipboardSupport();

if (support.writeText) {
  // writeText 사용 가능
}

if (support.write && support.clipboardItem) {
  // 이미지 복사 기능 사용 가능
}
```

### 폴리필 및 대안

```typescript
// clipboard-polyfill 라이브러리 사용
// npm install clipboard-polyfill

import * as clipboard from 'clipboard-polyfill';

async function copyWithPolyfill(text: string): Promise<void> {
  await clipboard.writeText(text);
}

// 또는 조건부 폴리필
async function universalCopy(text: string): Promise<boolean> {
  // 1. Clipboard API (권장)
  if (navigator.clipboard?.writeText) {
    try {
      await navigator.clipboard.writeText(text);
      return true;
    } catch (e) {
      // 권한 오류 등으로 실패 시 폴백
    }
  }

  // 2. execCommand 폴백 (레거시)
  const textarea = document.createElement('textarea');
  textarea.value = text;
  textarea.style.cssText = 'position:fixed;opacity:0;';
  document.body.appendChild(textarea);
  textarea.select();

  let success = false;
  try {
    success = document.execCommand('copy');
  } catch (e) {
    // execCommand도 실패
  }

  document.body.removeChild(textarea);
  return success;
}
```

---

## 모범 사례와 주의점

### 사용자 경험 고려사항

```typescript
// 1. 항상 시각적 피드백 제공
async function copyWithFeedback(text: string, button: HTMLButtonElement) {
  const originalText = button.textContent;

  try {
    await navigator.clipboard.writeText(text);

    // 성공 피드백
    button.textContent = '복사됨!';
    button.classList.add('success');

    setTimeout(() => {
      button.textContent = originalText;
      button.classList.remove('success');
    }, 2000);
  } catch (error) {
    // 실패 피드백
    button.textContent = '복사 실패';
    button.classList.add('error');

    setTimeout(() => {
      button.textContent = originalText;
      button.classList.remove('error');
    }, 2000);
  }
}

// 2. 접근성 고려
function createAccessibleCopyButton(text: string): HTMLButtonElement {
  const button = document.createElement('button');
  button.textContent = '복사';
  button.setAttribute('aria-label', '클립보드에 복사');

  button.addEventListener('click', async () => {
    await navigator.clipboard.writeText(text);

    // 스크린 리더 알림
    const announcement = document.createElement('div');
    announcement.setAttribute('role', 'status');
    announcement.setAttribute('aria-live', 'polite');
    announcement.textContent = '클립보드에 복사되었습니다.';
    announcement.style.cssText = `
      position: absolute;
      width: 1px;
      height: 1px;
      padding: 0;
      margin: -1px;
      overflow: hidden;
      clip: rect(0, 0, 0, 0);
      white-space: nowrap;
      border: 0;
    `;

    document.body.appendChild(announcement);
    setTimeout(() => announcement.remove(), 1000);
  });

  return button;
}
```

### 보안 모범 사례

```typescript
// 1. 민감한 데이터 클립보드에 남기지 않기
class SecureCopy {
  private static sensitiveData: Map<string, NodeJS.Timeout> = new Map();

  static async copySecure(
    text: string,
    clearAfterMs: number = 30000
  ): Promise<void> {
    await navigator.clipboard.writeText(text);

    // 일정 시간 후 클립보드 비우기
    const timeoutId = setTimeout(async () => {
      try {
        // 현재 클립보드 내용 확인
        const currentText = await navigator.clipboard.readText();
        if (currentText === text) {
          await navigator.clipboard.writeText('');
        }
      } catch (e) {
        // 읽기 권한 없으면 무시
      }
      this.sensitiveData.delete(text);
    }, clearAfterMs);

    this.sensitiveData.set(text, timeoutId);
  }

  static cancelClear(text: string): void {
    const timeoutId = this.sensitiveData.get(text);
    if (timeoutId) {
      clearTimeout(timeoutId);
      this.sensitiveData.delete(text);
    }
  }
}

// 2. 붙여넣기 데이터 검증
function sanitizePastedContent(html: string): string {
  // DOMPurify 사용 권장
  // import DOMPurify from 'dompurify';
  // return DOMPurify.sanitize(html);

  // 간단한 검증 예시 (프로덕션에서는 DOMPurify 사용)
  const doc = new DOMParser().parseFromString(html, 'text/html');

  // 스크립트 태그 제거
  doc.querySelectorAll('script').forEach((el) => el.remove());

  // 이벤트 핸들러 속성 제거
  doc.querySelectorAll('*').forEach((el) => {
    Array.from(el.attributes).forEach((attr) => {
      if (attr.name.startsWith('on')) {
        el.removeAttribute(attr.name);
      }
    });
  });

  return doc.body.innerHTML;
}

// 3. URL 검증
function isValidUrl(text: string): boolean {
  try {
    const url = new URL(text);
    return ['http:', 'https:'].includes(url.protocol);
  } catch {
    return false;
  }
}

async function safePasteUrl(input: HTMLInputElement): Promise<void> {
  try {
    const text = await navigator.clipboard.readText();

    if (isValidUrl(text)) {
      input.value = text;
    } else {
      alert('유효한 URL이 아닙니다.');
    }
  } catch (error) {
    console.error('붙여넣기 실패:', error);
  }
}
```

### 성능 고려사항

```typescript
// 1. 대용량 데이터 복사 시 청크 처리
async function copyLargeText(
  text: string,
  onProgress?: (percent: number) => void
): Promise<void> {
  const chunkSize = 1024 * 1024; // 1MB

  if (text.length <= chunkSize) {
    await navigator.clipboard.writeText(text);
    return;
  }

  // 대용량 텍스트는 경고
  const confirmed = confirm(
    `${(text.length / 1024 / 1024).toFixed(1)}MB의 텍스트를 복사합니다. 계속하시겠습니까?`
  );

  if (!confirmed) return;

  // 직접 복사 (브라우저가 처리)
  await navigator.clipboard.writeText(text);
}

// 2. 이미지 압축 후 복사
async function copyCompressedImage(
  canvas: HTMLCanvasElement,
  quality: number = 0.8
): Promise<void> {
  return new Promise((resolve, reject) => {
    canvas.toBlob(
      async (blob) => {
        if (!blob) {
          reject(new Error('이미지 변환 실패'));
          return;
        }

        try {
          await navigator.clipboard.write([
            new ClipboardItem({ 'image/png': blob })
          ]);
          resolve();
        } catch (error) {
          reject(error);
        }
      },
      'image/png',
      quality
    );
  });
}

// 3. 디바운싱으로 과도한 클립보드 작업 방지
function createDebouncedCopy(delay: number = 300) {
  let timeoutId: NodeJS.Timeout;

  return async function debouncedCopy(text: string): Promise<void> {
    return new Promise((resolve) => {
      clearTimeout(timeoutId);

      timeoutId = setTimeout(async () => {
        await navigator.clipboard.writeText(text);
        resolve();
      }, delay);
    });
  };
}

const debouncedCopy = createDebouncedCopy();
```

### 주의사항 체크리스트

| 항목 | 설명 |
|------|------|
| HTTPS 필수 | 프로덕션 환경에서 HTTPS 사용 |
| 사용자 제스처 | 클릭 등 사용자 인터랙션 필요 |
| 문서 포커스 | 문서가 포커스된 상태에서만 작동 |
| 권한 처리 | 권한 거부 시 적절한 피드백 |
| 에러 처리 | try-catch로 모든 에러 처리 |
| 폴백 제공 | 지원하지 않는 브라우저 대응 |
| 피드백 제공 | 사용자에게 성공/실패 알림 |
| 접근성 | 스크린 리더 지원 |

---

## 정리

Clipboard API는 웹 애플리케이션에서 클립보드 작업을 안전하고 효율적으로 처리할 수 있게 해주는 현대적인 API입니다.

### 핵심 요약

| 메서드 | 용도 | 권한 |
|--------|------|------|
| `writeText()` | 텍스트 복사 | 사용자 제스처 |
| `readText()` | 텍스트 읽기 | 권한 필요 |
| `write()` | 이미지/다중 형식 복사 | 사용자 제스처 |
| `read()` | 이미지/다중 형식 읽기 | 권한 필요 |

### 모범 사례 요약

1. **비동기 처리**: Promise 기반으로 에러 처리 철저히
2. **권한 확인**: Permissions API로 사전 확인
3. **폴백 제공**: 구형 브라우저 대응
4. **사용자 피드백**: 복사 성공/실패 시각적 알림
5. **보안 준수**: HTTPS 환경, 데이터 검증

---

## 관련 글

- [JavaScript Fetch API 완벽 가이드](/posts/javascript-fetch-api-network-guide/) - 네트워크 요청 API
- [JavaScript Web Storage API 가이드](/posts/javascript-web-storage-api-guide/) - 브라우저 데이터 저장
- [JavaScript 이벤트 위임 가이드](/posts/javascript-event-delegation-bubbling-guide/) - 이벤트 처리 패턴

---

## 참고 자료

- [MDN - Clipboard API](https://developer.mozilla.org/ko/docs/Web/API/Clipboard_API)
- [MDN - Clipboard](https://developer.mozilla.org/ko/docs/Web/API/Clipboard)
- [MDN - ClipboardItem](https://developer.mozilla.org/ko/docs/Web/API/ClipboardItem)
- [MDN - ClipboardEvent](https://developer.mozilla.org/ko/docs/Web/API/ClipboardEvent)
- [W3C Clipboard API Specification](https://www.w3.org/TR/clipboard-apis/)
- [Can I use - Clipboard API](https://caniuse.com/async-clipboard)
