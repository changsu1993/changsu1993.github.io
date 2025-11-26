---
name: list-posts
description: List recent blog posts with metadata summary. Shows title, date, categories, tags, and file path. Use to quickly see what posts exist.
---

# Recent Posts Lister

## When to Use
- Need to see recent posts
- Looking for a specific post
- Checking blog activity

## Output Format

For each post, show:
- 📄 **Title**
- 📅 Date
- 📁 Categories
- 🏷️ Tags
- 📍 File path

## Command
```bash
ls -t _posts/*.md | head -n {count}
```

Default count: 5

## Example Output
```
1. React Server Components 완벽 가이드
   📅 2025-11-25
   📁 [Frontend, React]
   🏷️ [react, server-components, nextjs]
   📍 _posts/2025-11-25-react-server-components-guide.md

2. React 18에서 19로 마이그레이션하기
   📅 2025-11-24
   ...
```
