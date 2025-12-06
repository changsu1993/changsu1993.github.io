# Auto Post - Automated Blog Post Creation

Automatically create a new blog post on a topic that doesn't overlap with existing posts.

## Instructions

### Step 1: Analyze Existing Posts (CRITICAL)

**IMPORTANT: This step MUST be thorough to prevent duplicate content.**

1. First, get the list of all existing posts:
```bash
ls -t _posts/*.md
```

2. **Read the front matter of ALL existing posts** to extract:
   - `title`: The post title
   - `categories`: Main topic areas
   - `tags`: Specific technologies/concepts covered

3. Create a structured list of already-covered topics:

```
## Already Covered Topics (DO NOT DUPLICATE):

### React
- [ ] React Hooks 성능 최적화
- [ ] React Server Components
- [ ] React 19 새 기능
- [ ] React 18→19 마이그레이션
- [ ] React Error Boundary
- [ ] React Suspense
- [ ] React Portal
- [ ] React Custom Hooks 패턴
- [ ] React Hook Form
- [ ] React Query (TanStack Query)
- [ ] React Actions 패턴
... (list ALL React-related posts)

### TypeScript
- [ ] TypeScript 고급 타입 패턴
- [ ] Zod 유효성 검사
... (list ALL TypeScript-related posts)

### Testing
- [ ] 프론트엔드 테스팅 전략
- [ ] TDD 실전 가이드
- [ ] E2E 테스팅
- [ ] Visual Regression Testing
- [ ] MSW API 모킹
... (list ALL testing-related posts)

### 기타 주제들
... (categorize ALL other posts)
```

4. **Verify the topic is truly new** by checking:
   - Title doesn't overlap with existing posts
   - Core concept isn't already covered (even with different title)
   - Not a subset of an existing comprehensive guide

### Step 2: Select Topic

Choose a frontend development topic that:
- ✅ Is NOT in the "Already Covered Topics" list
- ✅ Does NOT overlap significantly with existing content
- ✅ Brings genuinely new value to readers

**Topic Categories (check existing posts FIRST):**
- React (hooks, patterns, optimization, new features)
- TypeScript (advanced types, patterns)
- Next.js features
- Testing strategies
- Web performance
- CSS/Styling solutions
- Build tools (Vite, Webpack, etc.)
- State management
- API patterns
- Developer tools
- Browser APIs
- Security
- Accessibility

**Before proceeding, explicitly state:**
> "선택한 주제: [주제명]"
> "기존 포스트와의 차별점: [설명]"
> "유사 포스트 확인: [없음 / 있다면 어떻게 다른지 설명]"

### Step 3: Write Blog Post

Use Task tool with `subagent_type="blog-content-expert"` to write the post.

**CRITICAL: Include this in the prompt to the agent:**
```
## 주의사항 - 중복 방지
아래 주제들은 이미 블로그에 작성된 내용입니다. 절대 중복되지 않도록 해주세요:

[Step 1에서 정리한 기존 포스트 목록 전체를 여기에 포함]

작성할 주제: [선택한 주제]
이 주제가 기존 포스트와 다른 점: [차별점 설명]
```

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
