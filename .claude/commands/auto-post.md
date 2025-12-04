# Auto Post - Automated Blog Post Creation

Automatically create a new blog post on a topic that doesn't overlap with existing posts.

## Instructions

### Step 1: Analyze Existing Posts
Check existing posts in `_posts/` folder to find non-duplicate topics.

```bash
ls -t _posts/*.md | head -30
```

Analyze the titles and identify topics NOT yet covered.

### Step 2: Select Topic
Choose a frontend development topic that hasn't been covered yet.
Consider these categories:
- React (hooks, patterns, optimization, new features)
- TypeScript (advanced types, patterns)
- Next.js features
- Testing strategies
- Web performance
- CSS/Styling solutions
- Build tools (Vite, Webpack, etc.)
- State management
- API patterns

### Step 3: Write Blog Post
Use Task tool with `subagent_type="blog-content-expert"` to write the post.

Requirements:
- File path: `_posts/YYYY-MM-DD-slug.md` (use today's date)
- Language: Korean (all content must be in Korean)
- Code blocks: Use `tsx` for JSX, `jsonc` for JSON with comments
- No manual TOC (Chirpy theme auto-generates it)
- Include practical code examples
- Target audience: beginner to intermediate developers

### Step 4: Validate Post
Use the `validate-post` skill to check:
- Front Matter fields (title, description, date, categories, tags)
- Code block language identifiers
- Liquid template conflicts (wrap `{{ }}` with `{% raw %}{% endraw %}`)

### Step 5: SEO Optimization
Use Task tool with `subagent_type="seo-analyzer"` to:
- Optimize description (120-155 characters)
- Expand tags with relevant keywords
- Add internal links to related existing posts

Apply the recommended improvements.

### Step 6: Commit and Push
```bash
git add _posts/YYYY-MM-DD-*.md
git commit -m "feat(blog): [Post Title in Korean] 포스트 작성

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>"
git push origin main
```

### Step 7: Report
After completion, report:
- Created file path
- Post title and topic
- Deployment URL: `https://changsu1993.github.io/posts/[slug]/`
- Note: GitHub Actions will auto-deploy in ~3 minutes
