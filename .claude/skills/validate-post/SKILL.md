---
name: validate-post
description: Validate blog post for syntax errors, code block issues, and formatting problems. Use when checking post quality before publishing or when red highlights appear in code blocks.
---

# Blog Post Validator

## When to Use
- Before publishing a new post
- When code blocks show red highlights
- When checking post formatting

## Validation Checklist

### 1. Front Matter
- [ ] Required fields: title, description, date, categories, tags
- [ ] Date format: `YYYY-MM-DD HH:MM:SS +0900`
- [ ] Categories and tags are arrays

### 2. Code Block Issues (CRITICAL)
Find these patterns and report:

| Problem | Fix |
|---------|-----|
| `typescript` with JSX (`<`, `/>`, `<>`) | Change to `tsx` |
| `json` with comments (`//`, `/*`) | Change to `jsonc` |

### 3. Liquid Template Conflicts
- `{{ }}` or `{% %}` in code blocks need wrapping with `{% raw %}{% endraw %}`

### 4. Manual TOC
- Remove any manual table of contents (Chirpy auto-generates)

## Output
1. List all issues with line numbers
2. Provide fix suggestions
3. Ask if user wants auto-fix applied
