---
name: new-post
description: Create a new blog post using blog-content-expert subagent. Use when user requests writing a new technical blog article. Automatically applies correct code block identifiers and Korean language.
---

# New Blog Post Creator

## When to Use
- User requests a new blog post
- User provides a topic for writing

## Instructions

1. Use Task tool with `subagent_type="blog-content-expert"`
2. Generate file path: `_posts/YYYY-MM-DD-slug.md` (use today's date)
3. Ensure all code blocks use correct language identifiers:
   - JSX code → `tsx` (not `typescript`)
   - JSON with comments → `jsonc` (not `json`)
4. Do NOT include manual TOC (Chirpy auto-generates it)
5. After completion, ask user if they want to commit and push

## Front Matter Template
```yaml
---
title: "Title here"
description: "Description under 155 chars"
date: YYYY-MM-DD HH:MM:SS +0900
categories: [Category1, Category2]
tags: [tag1, tag2, tag3]
---
```
