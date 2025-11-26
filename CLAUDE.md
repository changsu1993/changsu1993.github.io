# Development Blog Project

## Project Overview
Jekyll-based technical blog (Chirpy 7.4.1 theme)
- URL: https://changsu1993.github.io/
- Hosting: GitHub Pages
- Main Topics: Frontend Development, React, TypeScript, Web Performance
- Language: All blog posts must be written in **Korean**

## Tech Stack
- Jekyll 4.3
- Ruby 3.x
- Chirpy Theme 7.4.1
- GitHub Actions (auto deployment)

## Directory Structure
```
_posts/          # Blog posts (YYYY-MM-DD-slug.md)
_layouts/        # Custom layouts
_tabs/           # Navigation tabs
assets/          # Static files (images, JS, CSS)
```

## Blog Post Writing Rules

### Front Matter (Required)
```yaml
---
title: "Title"
description: "Description (under 155 chars recommended)"
date: YYYY-MM-DD HH:MM:SS +0900
categories: [Category1, Category2]
tags: [tag1, tag2, tag3]
---
```

### Code Block Language Identifiers (CRITICAL!)
Rouge syntax highlighter rules:

| Situation | Correct | Wrong |
|-----------|---------|-------|
| Code with JSX | `tsx` | `typescript` |
| JSON with comments | `jsonc` | `json` |
| React components | `tsx` | `typescript` |
| Pure TypeScript | `typescript` | - |
| Shell commands | `bash` | `shell` |

**JSX detection patterns**: `<Component`, `<div`, `/>`, `<>`
**JSON comment detection**: `//`, `/*`

### Liquid Template Conflict Prevention
When using `{{ }}` or `{% %}` inside code blocks:
```liquid
{% raw %}
{{ variable }}
{% endraw %}
```

### Table of Contents
Chirpy theme auto-generates TOC - do NOT add manual TOC

## Subagent Usage

### blog-content-expert
- **Purpose**: Write new blog posts, review/improve existing posts
- **Auto-trigger**: When user requests blog post creation
- **Features**: Korean writing, official docs based, working code examples

### seo-analyzer
- **Purpose**: SEO analysis and optimization recommendations
- **Auto-trigger**: When user requests SEO-related tasks
- **Features**: Meta tags, Core Web Vitals, structured data analysis

## Available Skills

Skills are located in `.claude/skills/` directory.

| Skill | Purpose | When to Use |
|-------|---------|-------------|
| `new-post` | Create new blog post with blog-content-expert | User requests new article |
| `validate-post` | Check syntax, code blocks, formatting | Before publishing, when issues found |
| `fix-code-blocks` | Auto-fix language identifiers (tsx, jsonc) | Red highlights in code blocks |
| `check-seo` | SEO analysis with seo-analyzer | Before publishing important posts |
| `list-posts` | Show recent posts with metadata | Need to see existing posts |

## Common Issues & Solutions

### 1. Red Highlighted Code
**Symptom**: JSX `>` or JSON comments shown in red
**Fix**: Change `typescript` → `tsx`, `json` → `jsonc`

### 2. Dark Mode Styles Not Applied
**Symptom**: Custom styles don't work in dark mode
**Fix**: Use `html[data-mode="dark"]` selector, leverage theme defaults

### 3. Search Lag
**Symptom**: Blog search is slow
**Fix**: Limit content to 500 chars in `assets/js/data/search.json`

### 4. ASCII Box Misalignment
**Symptom**: Text art boxes look broken
**Fix**: Convert to markdown tables

## Commit Message Convention
```
feat(blog): New feature/post
fix: Bug fix
refactor: Code refactoring
style: Style changes
docs: Documentation
```
