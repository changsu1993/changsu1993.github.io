---
name: fix-code-blocks
description: Automatically fix code block language identifiers. Changes typescript to tsx when JSX is present, and json to jsonc when comments exist. Use when code shows red highlights.
---

# Code Block Fixer

## When to Use
- Code blocks show red syntax highlighting
- JSX `>` or `/>` appears red
- JSON comments appear red

## Auto-Fix Rules

| Find | Replace With | Condition |
|------|--------------|-----------|
| ` ```typescript ` | ` ```tsx ` | Contains JSX |
| ` ```json ` | ` ```jsonc ` | Contains comments |

### JSX Detection Patterns
- `<Component`
- `<div`, `<span`, `<button`
- `/>`
- `<>`
- `</`

### Comment Detection Patterns
- `//`
- `/*`
- `*/`

## Process
1. Read target file (or most recent post if not specified)
2. Find all ` ```typescript ` and ` ```json ` blocks
3. Analyze each block's content
4. Apply fixes where needed
5. Show summary: "Changed X blocks"
6. Commit with: `fix: 코드 블록 언어 식별자 수정`
