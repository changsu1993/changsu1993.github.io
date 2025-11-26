---
name: check-seo
description: Analyze blog post SEO using seo-analyzer subagent. Checks meta tags, content structure, and provides optimization recommendations. Use before publishing important posts.
---

# SEO Analyzer

## When to Use
- Before publishing important posts
- When optimizing existing posts for search
- When checking meta tag quality

## Analysis Areas

### Meta Tags
- **title**: Under 60 characters recommended
- **description**: Under 155 characters recommended
- Contains target keywords

### Content Structure
- Proper heading hierarchy (H2 → H3)
- Minimum 1000 characters recommended
- Images have alt text
- Code examples have explanations

### Technical SEO
- URL slug contains keywords
- Categories/tags are relevant
- Internal links to related posts

## Instructions
1. Use Task tool with `subagent_type="seo-analyzer"`
2. Analyze target post (or most recent if not specified)
3. Provide recommendations with priority:
   - 🔴 High: Must fix
   - 🟡 Medium: Should fix
   - 🟢 Low: Nice to have
