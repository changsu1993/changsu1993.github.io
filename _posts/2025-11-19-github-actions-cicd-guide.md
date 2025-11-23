---
title: "GitHub Actions로 CI/CD 파이프라인 구축하기 - 실전 완벽 가이드"
description: 프론트엔드 프로젝트를 위한 완벽한 CI/CD 파이프라인 구축 방법을 다룹니다. GitHub Actions 기초부터 테스트 자동화, 빌드 최적화, Docker 배포, 환경별 전략까지. React와 Next.js 프로젝트의 실전 워크플로우로 CI/CD를 완벽하게 마스터하세요.
author: changsu
date: 2025-11-19 10:05:00 +0900
categories: [DevOps, CI/CD]
tags: [github-actions, ci-cd, devops, docker, automation, deployment, testing, 지속적통합, 배포자동화]
toc: true
comments: true
---

## 들어가며

"코드 푸시할 때마다 수동으로 테스트하고, 빌드하고, 배포하는 게 너무 번거로워요." 이런 경험 있으시죠?

**CI/CD(Continuous Integration/Continuous Deployment)**는 이런 반복 작업을 자동화하여 개발자가 코드 작성에만 집중할 수 있게 해줍니다. GitHub Actions를 사용하면 별도의 서버 설정 없이도 강력한 CI/CD 파이프라인을 무료로 구축할 수 있습니다.

이 글에서는 프론트엔드 프로젝트(React, Next.js)를 중심으로 실무에서 바로 적용 가능한 CI/CD 파이프라인을 단계별로 구축해봅니다.

> CI/CD는 단순한 자동화가 아닙니다. 빠른 피드백, 안정적인 배포, 팀 생산성 향상의 핵심입니다.
{: .prompt-tip }

## CI/CD란 무엇인가?

### CI (Continuous Integration) - 지속적 통합

**코드를 자주 통합하고 자동으로 검증하는 프로세스**

```yaml
# CI의 핵심 단계
코드 커밋
  ↓
자동 빌드
  ↓
자동 테스트 (Unit, Integration, E2E)
  ↓
코드 품질 검사 (Lint, Type Check)
  ↓
통과 ✅ / 실패 ❌
```

**CI의 이점:**
- 버그를 조기에 발견
- 통합 문제를 빠르게 해결
- 코드 품질 향상
- 팀 간 충돌 최소화

### CD (Continuous Deployment) - 지속적 배포

**검증된 코드를 자동으로 프로덕션에 배포하는 프로세스**

```yaml
# CD의 핵심 단계
CI 통과
  ↓
자동 빌드 & 최적화
  ↓
스테이징 환경 배포
  ↓
자동/수동 승인
  ↓
프로덕션 배포
  ↓
모니터링 & 롤백 준비
```

**CD의 이점:**
- 빠른 릴리스 주기
- 수동 배포 오류 제거
- 일관된 배포 프로세스
- 롤백 용이성

### CI/CD가 필요한 이유

**실제 시나리오:**

```text
// ❌ CI/CD 없는 경우
개발자A: "코드 푸시했어요!"
개발자B: "앗, 제 브랜치랑 충돌나요..."
PM: "긴급 수정 배포해야 하는데 누가 배포할 수 있나요?"
개발자C: "테스트 돌려봤는데 로컬에선 되는데..."
DevOps: "배포 스크립트가 어디 있죠?"

// ✅ CI/CD 있는 경우
개발자: PR 생성
GitHub Actions: 자동 테스트 실행 ✅
팀: 코드 리뷰
GitHub Actions: main 브랜치 머지 시 자동 배포 🚀
팀: 슬랙으로 배포 완료 알림 받음 📬
```

## GitHub Actions 핵심 개념

### Workflow, Job, Step 이해하기

```yaml
# .github/workflows/ci.yml

# Workflow: 전체 자동화 프로세스
name: CI Pipeline

# Trigger: 언제 실행될지 정의
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

# Jobs: 병렬로 실행될 수 있는 작업 그룹
jobs:
  # Job 1: 테스트
  test:
    runs-on: ubuntu-latest

    # Steps: 순차적으로 실행되는 개별 명령
    steps:
      - name: 코드 체크아웃
        uses: actions/checkout@v4

      - name: Node.js 설정
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: 의존성 설치
        run: npm ci

      - name: 테스트 실행
        run: npm test

  # Job 2: 린트 (test와 병렬 실행)
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      - run: npm run lint
```

### Workflow 구성 요소

**1. Trigger (on):**

```yaml
# Push 이벤트
on:
  push:
    branches:
      - main
      - develop
    paths:
      - 'src/**'      # src 폴더만 감시
      - '**.ts'       # TypeScript 파일만
    paths-ignore:
      - 'docs/**'     # docs는 무시

# Pull Request 이벤트
on:
  pull_request:
    types:
      - opened        # PR 생성 시
      - synchronize   # 새 커밋 푸시 시
      - reopened      # PR 재오픈 시
    branches:
      - main

# 스케줄 (Cron)
on:
  schedule:
    - cron: '0 0 * * *'  # 매일 자정

# 수동 트리거
on:
  workflow_dispatch:
    inputs:
      environment:
        description: '배포 환경'
        required: true
        type: choice
        options:
          - development
          - staging
          - production

# 여러 이벤트 결합
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  workflow_dispatch:
```

**2. Jobs와 의존성:**

```yaml
jobs:
  # Job 1: 빌드
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm run build

      # 빌드 결과물 저장
      - uses: actions/upload-artifact@v4
        with:
          name: build-artifacts
          path: dist/

  # Job 2: 테스트 (build와 병렬 실행)
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm test

  # Job 3: 배포 (build와 test가 성공해야 실행)
  deploy:
    needs: [build, test]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: build-artifacts
          path: dist/
      - run: npm run deploy
```

**3. Matrix Strategy (여러 환경 테스트):**

```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node-version: [18, 20, 22]
        # ubuntu-18, ubuntu-20, ubuntu-22
        # windows-18, windows-20, windows-22
        # macos-18, macos-20, macos-22
        # 총 9개의 조합이 병렬로 실행됨

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm ci
      - run: npm test
```

### 환경 변수와 시크릿

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest

    env:
      # Workflow 레벨 환경 변수
      NODE_ENV: production
      API_URL: https://api.example.com

    steps:
      - uses: actions/checkout@v4

      - name: 빌드
        env:
          # Step 레벨 환경 변수
          BUILD_NUMBER: ${{ github.run_number }}
        run: |
          echo "Building version: $BUILD_NUMBER"
          npm run build

      - name: 배포
        env:
          # GitHub Secrets 사용
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          # GitHub 기본 제공 변수
          GITHUB_SHA: ${{ github.sha }}
          GITHUB_REF: ${{ github.ref }}
        run: npm run deploy
```

**GitHub Secrets 설정:**

```bash
# Repository Settings > Secrets and variables > Actions

# 시크릿 추가:
AWS_ACCESS_KEY_ID: AKIA...
AWS_SECRET_ACCESS_KEY: wJalr...
DOCKER_USERNAME: myusername
DOCKER_PASSWORD: mypassword
```

## 테스트 자동화

### Unit 테스트 (Jest)

```yaml
# .github/workflows/test.yml
name: Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  unit-test:
    runs-on: ubuntu-latest

    steps:
      - name: 코드 체크아웃
        uses: actions/checkout@v4

      - name: Node.js 설정
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'  # npm 캐시 활성화

      - name: 의존성 설치
        run: npm ci

      - name: Unit 테스트 실행
        run: npm run test:unit -- --coverage

      - name: 커버리지 리포트 업로드
        uses: codecov/codecov-action@v4
        with:
          files: ./coverage/coverage-final.json
          flags: unittests
          name: codecov-umbrella
```

**Jest 설정 (jest.config.js):**

```javascript
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'jsdom',
  roots: ['<rootDir>/src'],
  testMatch: ['**/__tests__/**/*.test.ts(x)?'],
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.d.ts',
    '!src/**/*.stories.tsx',
    '!src/index.tsx',
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
  },
  setupFilesAfterEnv: ['<rootDir>/jest.setup.js'],
};
```

### Integration 테스트

```yaml
jobs:
  integration-test:
    runs-on: ubuntu-latest

    # 서비스 컨테이너 (DB, Redis 등)
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: testdb
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

      redis:
        image: redis:7
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 6379:6379

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - run: npm ci

      - name: Integration 테스트
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/testdb
          REDIS_URL: redis://localhost:6379
        run: npm run test:integration
```

### E2E 테스트 (Playwright)

```yaml
jobs:
  e2e-test:
    runs-on: ubuntu-latest
    timeout-minutes: 60

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - run: npm ci

      - name: Playwright 설치
        run: npx playwright install --with-deps

      - name: 앱 빌드
        run: npm run build

      - name: E2E 테스트 실행
        run: npm run test:e2e

      - name: 테스트 실패 시 스크린샷 업로드
        uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 30
```

**Playwright 설정 (playwright.config.ts):**

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: process.env.CI ? 'html' : 'list',

  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
  },

  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] },
    },
  ],

  webServer: {
    command: 'npm run start',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

## 린트 및 코드 품질 검사

### ESLint + Prettier

```yaml
# .github/workflows/code-quality.yml
name: Code Quality

on:
  pull_request:
    branches: [main, develop]

jobs:
  lint:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - run: npm ci

      - name: ESLint 실행
        run: npm run lint

      - name: Prettier 체크
        run: npm run format:check

      - name: TypeScript 타입 체크
        run: npm run type-check
```

**package.json 스크립트:**

```json
{
  "scripts": {
    "lint": "eslint . --ext .ts,.tsx --max-warnings 0",
    "lint:fix": "eslint . --ext .ts,.tsx --fix",
    "format": "prettier --write \"src/**/*.{ts,tsx,json,css,md}\"",
    "format:check": "prettier --check \"src/**/*.{ts,tsx,json,css,md}\"",
    "type-check": "tsc --noEmit"
  }
}
```

### 코드 품질 자동 수정

```yaml
jobs:
  auto-fix:
    runs-on: ubuntu-latest
    permissions:
      contents: write

    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.head_ref }}

      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - run: npm ci

      - name: ESLint 자동 수정
        run: npm run lint:fix
        continue-on-error: true

      - name: Prettier 자동 포맷팅
        run: npm run format

      - name: 변경사항 커밋
        uses: stefanzweifel/git-auto-commit-action@v5
        with:
          commit_message: "style: auto-fix lint and format issues"
          commit_user_name: GitHub Actions
          commit_user_email: actions@github.com
```

## 빌드 최적화

### 캐싱 전략

```yaml
jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      # Node.js 캐싱
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'  # node_modules 캐싱

      # 커스텀 캐싱
      - name: 캐시 복원
        id: cache
        uses: actions/cache@v4
        with:
          path: |
            ~/.npm
            node_modules
            .next/cache
          key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-node-

      - name: 의존성 설치
        if: steps.cache.outputs.cache-hit != 'true'
        run: npm ci

      - name: 빌드
        run: npm run build

      # 빌드 결과 캐싱
      - name: 빌드 결과 캐시
        uses: actions/cache/save@v4
        with:
          path: dist/
          key: build-${{ github.sha }}
```

### 병렬 처리

```yaml
jobs:
  # Job 1: 빌드
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: dist
          path: dist/

  # Job 2-4: 병렬 테스트
  test-unit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run test:unit

  test-integration:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run test:integration

  test-e2e:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npx playwright install --with-deps
      - run: npm run test:e2e

  # Job 5: 배포 (모든 테스트와 빌드 성공 후)
  deploy:
    needs: [build, test-unit, test-integration, test-e2e]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: dist
      - run: npm run deploy
```

### 조건부 실행

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    # main 브랜치에 푸시되고, 커밋 메시지에 [skip-deploy]가 없을 때만 실행
    if: |
      github.ref == 'refs/heads/main' &&
      !contains(github.event.head_commit.message, '[skip-deploy]')

    steps:
      - name: 프로덕션 배포
        run: npm run deploy:prod

  notify:
    runs-on: ubuntu-latest
    # deploy job이 실패했을 때만 실행
    if: failure()
    needs: [deploy]

    steps:
      - name: 슬랙 알림
        uses: slackapi/slack-github-action@v1
        with:
          webhook-url: ${{ secrets.SLACK_WEBHOOK }}
          payload: |
            {
              "text": "❌ 배포 실패: ${{ github.repository }}"
            }
```

## Docker를 활용한 프론트엔드 배포

### 멀티 스테이지 Dockerfile

```dockerfile
# Dockerfile

# Stage 1: 빌드 스테이지
FROM node:20-alpine AS builder

# 작업 디렉토리 설정
WORKDIR /app

# package.json과 package-lock.json 복사
COPY package*.json ./

# 의존성 설치 (ci는 package-lock.json을 엄격히 따름)
RUN npm ci

# 소스 코드 복사
COPY . .

# 빌드
ARG NODE_ENV=production
ENV NODE_ENV=${NODE_ENV}

RUN npm run build

# 불필요한 devDependencies 제거
RUN npm prune --production

# Stage 2: 프로덕션 스테이지
FROM nginx:alpine AS production

# Nginx 설정 복사
COPY nginx.conf /etc/nginx/conf.d/default.conf

# 빌드 결과물 복사
COPY --from=builder /app/dist /usr/share/nginx/html

# 헬스체크
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget --quiet --tries=1 --spider http://localhost:80/ || exit 1

# 포트 노출
EXPOSE 80

# Nginx 실행
CMD ["nginx", "-g", "daemon off;"]
```

**Nginx 설정 (nginx.conf):**

```nginx
server {
    listen 80;
    server_name _;
    root /usr/share/nginx/html;
    index index.html;

    # Gzip 압축
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types text/plain text/css text/xml text/javascript
               application/x-javascript application/xml+rss
               application/javascript application/json;

    # 보안 헤더
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;

    # SPA를 위한 라우팅
    location / {
        try_files $uri $uri/ /index.html;
    }

    # 정적 리소스 캐싱
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # 헬스체크 엔드포인트
    location /health {
        access_log off;
        return 200 "healthy\n";
        add_header Content-Type text/plain;
    }
}
```

### Docker Compose

```yaml
# docker-compose.yml
version: '3.8'

services:
  frontend:
    build:
      context: .
      dockerfile: Dockerfile
      args:
        NODE_ENV: production
    image: myapp-frontend:latest
    container_name: myapp-frontend
    ports:
      - "80:80"
    environment:
      - NODE_ENV=production
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "wget", "--quiet", "--tries=1", "--spider", "http://localhost:80/health"]
      interval: 30s
      timeout: 3s
      retries: 3
      start_period: 40s
    networks:
      - app-network

networks:
  app-network:
    driver: bridge
```

### GitHub Actions에서 Docker 빌드 및 배포

```yaml
# .github/workflows/docker-deploy.yml
name: Docker Build & Deploy

on:
  push:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write

    steps:
      - name: 코드 체크아웃
        uses: actions/checkout@v4

      - name: Docker Buildx 설정
        uses: docker/setup-buildx-action@v3

      - name: GitHub Container Registry 로그인
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: 이미지 메타데이터 추출
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            type=sha,prefix={{branch}}-
            type=raw,value=latest,enable={{is_default_branch}}

      - name: Docker 이미지 빌드 및 푸시
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          build-args: |
            NODE_ENV=production
            BUILD_DATE=${{ github.event.head_commit.timestamp }}
            VCS_REF=${{ github.sha }}

  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest

    steps:
      - name: SSH로 서버 배포
        uses: appleboy/ssh-action@v1.0.0
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            # 컨테이너 중지 및 제거
            docker compose down

            # 새 이미지 풀
            docker compose pull

            # 컨테이너 실행
            docker compose up -d

            # 이전 이미지 정리
            docker image prune -f
```

## 환경별 배포 전략

### Development, Staging, Production 분리

```yaml
# .github/workflows/deploy-multi-env.yml
name: Multi-Environment Deploy

on:
  push:
    branches:
      - develop      # Development
      - staging      # Staging
      - main         # Production
  workflow_dispatch:
    inputs:
      environment:
        description: '배포 환경'
        required: true
        type: choice
        options:
          - development
          - staging
          - production

jobs:
  determine-environment:
    runs-on: ubuntu-latest
    outputs:
      environment: ${{ steps.set-env.outputs.environment }}

    steps:
      - name: 환경 결정
        id: set-env
        run: |
          if [ "${{ github.event_name }}" = "workflow_dispatch" ]; then
            echo "environment=${{ github.event.inputs.environment }}" >> $GITHUB_OUTPUT
          elif [ "${{ github.ref }}" = "refs/heads/main" ]; then
            echo "environment=production" >> $GITHUB_OUTPUT
          elif [ "${{ github.ref }}" = "refs/heads/staging" ]; then
            echo "environment=staging" >> $GITHUB_OUTPUT
          else
            echo "environment=development" >> $GITHUB_OUTPUT
          fi

  build:
    needs: determine-environment
    runs-on: ubuntu-latest
    environment: ${{ needs.determine-environment.outputs.environment }}

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - run: npm ci

      - name: 빌드 (${{ needs.determine-environment.outputs.environment }})
        env:
          NODE_ENV: production
          VITE_API_URL: ${{ vars.API_URL }}
          VITE_APP_ENV: ${{ needs.determine-environment.outputs.environment }}
        run: npm run build

      - uses: actions/upload-artifact@v4
        with:
          name: build-${{ needs.determine-environment.outputs.environment }}
          path: dist/

  deploy:
    needs: [determine-environment, build]
    runs-on: ubuntu-latest
    environment:
      name: ${{ needs.determine-environment.outputs.environment }}
      url: ${{ steps.deploy.outputs.url }}

    steps:
      - uses: actions/download-artifact@v4
        with:
          name: build-${{ needs.determine-environment.outputs.environment }}
          path: dist/

      - name: 배포
        id: deploy
        run: |
          # 환경별 배포 로직
          case "${{ needs.determine-environment.outputs.environment }}" in
            production)
              echo "url=https://app.example.com" >> $GITHUB_OUTPUT
              # 프로덕션 배포 명령
              ;;
            staging)
              echo "url=https://staging.example.com" >> $GITHUB_OUTPUT
              # 스테이징 배포 명령
              ;;
            development)
              echo "url=https://dev.example.com" >> $GITHUB_OUTPUT
              # 개발 배포 명령
              ;;
          esac
```

**GitHub Environment 설정:**

```bash
# Settings > Environments

# 1. Production 환경 생성
- Environment name: production
- Protection rules:
  ✅ Required reviewers (필수 승인자)
  ✅ Wait timer (배포 대기 시간)
- Environment secrets:
  API_URL: https://api.example.com
  DEPLOY_KEY: xxxxx

# 2. Staging 환경 생성
- Environment name: staging
- Protection rules: (선택적)
- Environment secrets:
  API_URL: https://staging-api.example.com
  DEPLOY_KEY: yyyyy

# 3. Development 환경 생성
- Environment name: development
- Environment secrets:
  API_URL: https://dev-api.example.com
  DEPLOY_KEY: zzzzz
```

### Blue-Green 배포

```yaml
jobs:
  blue-green-deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: 현재 활성 환경 확인
        id: current
        run: |
          ACTIVE=$(curl -s https://example.com/active)
          if [ "$ACTIVE" = "blue" ]; then
            echo "target=green" >> $GITHUB_OUTPUT
          else
            echo "target=blue" >> $GITHUB_OUTPUT
          fi

      - name: 타겟 환경에 배포
        run: |
          # ${{ steps.current.outputs.target }} 환경에 배포
          docker tag myapp:latest myapp:${{ steps.current.outputs.target }}
          docker push myapp:${{ steps.current.outputs.target }}

      - name: 헬스체크
        run: |
          for i in {1..30}; do
            if curl -f https://${{ steps.current.outputs.target }}.example.com/health; then
              echo "Health check passed"
              exit 0
            fi
            sleep 10
          done
          exit 1

      - name: 트래픽 전환
        run: |
          # 로드 밸런서 설정 변경
          curl -X POST https://api.example.com/switch-active \
            -H "Authorization: Bearer ${{ secrets.DEPLOY_TOKEN }}" \
            -d '{"active": "${{ steps.current.outputs.target }}"}'

      - name: 이전 환경 중지 (선택적)
        run: |
          # 필요시 이전 환경을 대기 상태로 유지하거나 중지
          echo "Old environment remains as standby for rollback"
```

## 보안 고려사항

### Secrets 관리

```yaml
jobs:
  secure-deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: 시크릿 사용
        env:
          # ✅ 올바른 방법: 환경 변수로 전달
          API_KEY: ${{ secrets.API_KEY }}
          DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
        run: |
          # 시크릿은 자동으로 마스킹됨
          echo "API Key: ***"
          npm run deploy

      - name: 잘못된 시크릿 사용
        run: |
          # ❌ 위험: 명령어에 직접 노출
          curl -H "Authorization: ${{ secrets.API_KEY }}" https://api.example.com
```

### OIDC를 통한 AWS 인증

```yaml
jobs:
  deploy-to-aws:
    runs-on: ubuntu-latest
    permissions:
      id-token: write  # OIDC 토큰 발급을 위해 필요
      contents: read

    steps:
      - uses: actions/checkout@v4

      - name: AWS 자격 증명 획득
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/GitHubActionsRole
          aws-region: ap-northeast-2

      - name: S3에 배포
        run: |
          aws s3 sync dist/ s3://my-bucket/ --delete

      - name: CloudFront 캐시 무효화
        run: |
          aws cloudfront create-invalidation \
            --distribution-id EDFDVBD6EXAMPLE \
            --paths "/*"
```

**AWS IAM Role Trust Policy:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::123456789012:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
        },
        "StringLike": {
          "token.actions.githubusercontent.com:sub": "repo:username/repo-name:*"
        }
      }
    }
  ]
}
```

### 의존성 보안 검사

```yaml
jobs:
  security-scan:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: npm audit
        run: npm audit --audit-level=high

      - name: Snyk 보안 검사
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high

      - name: 라이선스 검사
        run: npx license-checker --production --onlyAllow 'MIT;Apache-2.0;BSD-3-Clause;ISC'
```

## 모니터링 및 롤백 전략

### 배포 알림

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: 배포 시작 알림
        uses: slackapi/slack-github-action@v1
        with:
          webhook-url: ${{ secrets.SLACK_WEBHOOK }}
          payload: |
            {
              "text": "🚀 배포 시작",
              "blocks": [
                {
                  "type": "section",
                  "text": {
                    "type": "mrkdwn",
                    "text": "*배포 시작*\n환경: Production\n커밋: ${{ github.sha }}\n작성자: ${{ github.actor }}"
                  }
                }
              ]
            }

      - name: 배포 실행
        id: deploy
        run: npm run deploy

      - name: 배포 성공 알림
        if: success()
        uses: slackapi/slack-github-action@v1
        with:
          webhook-url: ${{ secrets.SLACK_WEBHOOK }}
          payload: |
            {
              "text": "✅ 배포 성공",
              "blocks": [
                {
                  "type": "section",
                  "text": {
                    "type": "mrkdwn",
                    "text": "*배포 성공* ✅\n환경: Production\nURL: https://app.example.com\n소요 시간: ${{ steps.deploy.outputs.duration }}"
                  }
                }
              ]
            }

      - name: 배포 실패 알림
        if: failure()
        uses: slackapi/slack-github-action@v1
        with:
          webhook-url: ${{ secrets.SLACK_WEBHOOK }}
          payload: |
            {
              "text": "❌ 배포 실패",
              "blocks": [
                {
                  "type": "section",
                  "text": {
                    "type": "mrkdwn",
                    "text": "*배포 실패* ❌\n환경: Production\n로그: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}"
                  }
                }
              ]
            }
```

### 자동 롤백

```yaml
jobs:
  deploy-with-rollback:
    runs-on: ubuntu-latest

    steps:
      - name: 이전 버전 백업
        id: backup
        run: |
          PREV_VERSION=$(docker ps --filter "name=myapp" --format "{{.Image}}")
          echo "previous=$PREV_VERSION" >> $GITHUB_OUTPUT

      - name: 새 버전 배포
        id: deploy
        run: |
          docker pull myapp:${{ github.sha }}
          docker stop myapp || true
          docker rm myapp || true
          docker run -d --name myapp myapp:${{ github.sha }}

      - name: 헬스체크
        id: health
        run: |
          sleep 10
          for i in {1..30}; do
            if curl -f http://localhost/health; then
              echo "Health check passed"
              exit 0
            fi
            sleep 5
          done
          echo "Health check failed"
          exit 1

      - name: 자동 롤백
        if: failure() && steps.health.outcome == 'failure'
        run: |
          echo "헬스체크 실패, 이전 버전으로 롤백"
          docker stop myapp || true
          docker rm myapp || true
          docker run -d --name myapp ${{ steps.backup.outputs.previous }}

          # 슬랙 알림
          curl -X POST ${{ secrets.SLACK_WEBHOOK }} \
            -H 'Content-Type: application/json' \
            -d '{"text": "⚠️ 배포 실패로 자동 롤백됨"}'
```

### 성능 모니터링

```yaml
jobs:
  performance-monitoring:
    runs-on: ubuntu-latest

    steps:
      - name: Lighthouse CI
        uses: treosh/lighthouse-ci-action@v10
        with:
          urls: |
            https://app.example.com
            https://app.example.com/dashboard
          uploadArtifacts: true
          temporaryPublicStorage: true

      - name: 성능 임계값 체크
        run: |
          SCORE=$(jq '.[] | select(.url == "https://app.example.com") | .summary.performance' lighthouse-results.json)
          if (( $(echo "$SCORE < 90" | bc -l) )); then
            echo "Performance score below threshold: $SCORE"
            exit 1
          fi
```

## 실전 예제: React + Next.js 프로젝트

### 완전한 CI/CD 파이프라인

```yaml
# .github/workflows/main.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  NODE_VERSION: '20'
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # Job 1: 코드 품질 검사
  quality:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - name: 의존성 설치
        run: npm ci

      - name: TypeScript 타입 체크
        run: npm run type-check

      - name: ESLint
        run: npm run lint

      - name: Prettier 체크
        run: npm run format:check

  # Job 2: 단위 테스트
  test-unit:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - run: npm ci
      - run: npm run test:unit -- --coverage

      - name: 커버리지 업로드
        uses: codecov/codecov-action@v4
        with:
          files: ./coverage/coverage-final.json

  # Job 3: E2E 테스트
  test-e2e:
    runs-on: ubuntu-latest
    timeout-minutes: 60

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - run: npm ci
      - run: npx playwright install --with-deps
      - run: npm run build
      - run: npm run test:e2e

      - uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: playwright-report
          path: playwright-report/

  # Job 4: 빌드
  build:
    needs: [quality, test-unit, test-e2e]
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - run: npm ci

      - name: Next.js 빌드
        env:
          NEXT_PUBLIC_API_URL: ${{ vars.API_URL }}
        run: npm run build

      - uses: actions/upload-artifact@v4
        with:
          name: nextjs-build
          path: .next/

  # Job 5: Docker 이미지 빌드 및 푸시
  docker:
    needs: build
    runs-on: ubuntu-latest
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    permissions:
      contents: read
      packages: write

    steps:
      - uses: actions/checkout@v4

      - uses: docker/setup-buildx-action@v3

      - uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - uses: docker/metadata-action@v5
        id: meta
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=sha,prefix={{branch}}-
            type=raw,value=latest

      - uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  # Job 6: 배포
  deploy:
    needs: docker
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://app.example.com

    steps:
      - name: 배포 시작 알림
        uses: slackapi/slack-github-action@v1
        with:
          webhook-url: ${{ secrets.SLACK_WEBHOOK }}
          payload: |
            {
              "text": "🚀 프로덕션 배포 시작\n커밋: ${{ github.sha }}\n작성자: ${{ github.actor }}"
            }

      - name: SSH 배포
        uses: appleboy/ssh-action@v1.0.0
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd /app
            docker compose pull
            docker compose up -d
            docker image prune -f

      - name: 헬스체크
        run: |
          sleep 10
          for i in {1..30}; do
            if curl -f https://app.example.com/health; then
              echo "Deployment successful"
              exit 0
            fi
            sleep 5
          done
          exit 1

      - name: 배포 성공 알림
        if: success()
        uses: slackapi/slack-github-action@v1
        with:
          webhook-url: ${{ secrets.SLACK_WEBHOOK }}
          payload: |
            {
              "text": "✅ 프로덕션 배포 성공\nURL: https://app.example.com"
            }

      - name: 배포 실패 알림
        if: failure()
        uses: slackapi/slack-github-action@v1
        with:
          webhook-url: ${{ secrets.SLACK_WEBHOOK }}
          payload: |
            {
              "text": "❌ 프로덕션 배포 실패\n로그: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}"
            }
```

**Next.js Dockerfile:**

```dockerfile
# Dockerfile
FROM node:20-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci

FROM node:20-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
ENV NEXT_TELEMETRY_DISABLED 1
RUN npm run build

FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV production
ENV NEXT_TELEMETRY_DISABLED 1

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000

ENV PORT 3000

CMD ["node", "server.js"]
```

**next.config.js:**

```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  output: 'standalone',
  reactStrictMode: true,
  swcMinify: true,

  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'images.example.com',
      },
    ],
  },

  async headers() {
    return [
      {
        source: '/:path*',
        headers: [
          {
            key: 'X-Frame-Options',
            value: 'DENY',
          },
          {
            key: 'X-Content-Type-Options',
            value: 'nosniff',
          },
          {
            key: 'Referrer-Policy',
            value: 'strict-origin-when-cross-origin',
          },
        ],
      },
    ];
  },
};

module.exports = nextConfig;
```

## 트러블슈팅 및 베스트 프랙티스

### 일반적인 문제와 해결책

**1. 의존성 설치 실패**

```yaml
# ❌ 문제: package-lock.json 불일치
- run: npm install

# ✅ 해결: npm ci 사용
- run: npm ci
```

**2. 캐시 미적용**

```yaml
# ❌ 문제: 매번 전체 의존성 재설치
- uses: actions/setup-node@v4
  with:
    node-version: '20'

# ✅ 해결: 캐시 활성화
- uses: actions/setup-node@v4
  with:
    node-version: '20'
    cache: 'npm'
```

**3. 시크릿 노출**

```yaml
# ❌ 위험: 시크릿 로그 노출
- run: echo ${{ secrets.API_KEY }}

# ✅ 안전: 환경 변수로 전달
- env:
    API_KEY: ${{ secrets.API_KEY }}
  run: npm run deploy
```

**4. 타임아웃**

```yaml
# ❌ 문제: 긴 작업이 타임아웃
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - run: npm run deploy  # 20분 걸림

# ✅ 해결: 타임아웃 연장
jobs:
  deploy:
    runs-on: ubuntu-latest
    timeout-minutes: 30
    steps:
      - run: npm run deploy
```

**5. 불필요한 워크플로우 실행**

```yaml
# ❌ 문제: 문서 변경에도 테스트 실행
on:
  push:
    branches: [main]

# ✅ 해결: 경로 필터링
on:
  push:
    branches: [main]
    paths:
      - 'src/**'
      - 'package*.json'
    paths-ignore:
      - 'docs/**'
      - '**.md'
```

### 베스트 프랙티스

**1. 최소 권한 원칙**

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read        # 코드 읽기만 허용
      packages: write      # 패키지 쓰기만 필요한 경우
      id-token: write      # OIDC 필요한 경우만
```

**2. 재사용 가능한 워크플로우**

```yaml
# .github/workflows/reusable-build.yml
name: Reusable Build

on:
  workflow_call:
    inputs:
      node-version:
        required: true
        type: string
      environment:
        required: true
        type: string
    outputs:
      artifact-id:
        value: ${{ jobs.build.outputs.artifact-id }}

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      artifact-id: ${{ steps.upload.outputs.artifact-id }}

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
          cache: 'npm'
      - run: npm ci
      - run: npm run build

      - id: upload
        uses: actions/upload-artifact@v4
        with:
          name: build-${{ inputs.environment }}
          path: dist/
```

**사용:**

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  build-prod:
    uses: ./.github/workflows/reusable-build.yml
    with:
      node-version: '20'
      environment: 'production'

  deploy:
    needs: build-prod
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying artifact ${{ needs.build-prod.outputs.artifact-id }}"
```

**3. 조건부 스텝**

```yaml
steps:
  - name: 테스트 실행
    id: test
    run: npm test
    continue-on-error: true

  - name: 테스트 실패 처리
    if: steps.test.outcome == 'failure'
    run: |
      echo "Tests failed but continuing..."
      # 슬랙 알림 등

  - name: 배포 (테스트 성공 시만)
    if: steps.test.outcome == 'success'
    run: npm run deploy
```

**4. 컨테이너 서비스 활용**

```yaml
jobs:
  integration-test:
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - run: npm run test:integration
```

**5. 아티팩트 정리**

```yaml
- uses: actions/upload-artifact@v4
  with:
    name: test-results
    path: test-results/
    retention-days: 7  # 7일 후 자동 삭제
```

## FAQ

### Q1: GitHub Actions는 무료인가요?

**A:** Public 레포지토리는 완전 무료입니다. Private 레포지토리는 월 2,000분 무료이며, 초과 시 분당 $0.008입니다. Self-hosted runner를 사용하면 비용을 절감할 수 있습니다.

### Q2: npm install과 npm ci의 차이는 무엇인가요?

**A:**
- `npm install`: package.json 기반으로 설치, package-lock.json 업데이트 가능
- `npm ci`: package-lock.json을 엄격히 따름, 더 빠르고 재현 가능, CI에 권장

```bash
# 로컬 개발
npm install

# CI/CD
npm ci
```

### Q3: 캐시를 사용해도 빌드가 느린데요?

**A:** 캐시 키를 확인하세요. `hashFiles('**/package-lock.json')`을 사용하면 의존성 변경 시만 캐시를 무효화합니다.

```yaml
- uses: actions/cache@v4
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-
```

### Q4: 특정 브랜치에만 워크플로우를 실행하려면?

**A:**

```yaml
on:
  push:
    branches:
      - main
      - 'release/**'  # release/v1, release/v2 등
  pull_request:
    branches:
      - main
```

### Q5: Docker 이미지 빌드가 너무 오래 걸려요

**A:** 멀티 스테이지 빌드와 캐싱을 활용하세요:

```yaml
- uses: docker/build-push-action@v5
  with:
    context: .
    push: true
    tags: myapp:latest
    cache-from: type=gha
    cache-to: type=gha,mode=max
```

### Q6: 시크릿과 환경 변수의 차이는?

**A:**
- **Secrets**: 민감한 정보 (API 키, 비밀번호), GitHub이 암호화하여 저장
- **Variables**: 일반 설정 값, 암호화되지 않음

```yaml
env:
  API_URL: ${{ vars.API_URL }}          # Variable (공개 가능)
  API_KEY: ${{ secrets.API_KEY }}       # Secret (민감 정보)
```

### Q7: Pull Request마다 배포 환경을 만들고 싶어요

**A:** Preview 배포를 활용하세요:

```yaml
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  preview-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm run build

      - name: Vercel Preview 배포
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.ORG_ID }}
          vercel-project-id: ${{ secrets.PROJECT_ID }}
```

### Q8: 워크플로우 실행 시간을 어떻게 단축하나요?

**A:**

1. **병렬 실행**: 독립적인 Job을 병렬로 실행
2. **캐싱**: 의존성과 빌드 결과 캐싱
3. **조건부 실행**: 변경된 파일만 트리거
4. **Self-hosted Runner**: 더 빠른 하드웨어 사용

```yaml
jobs:
  test-unit:
    # 병렬 실행
  test-e2e:
    # 병렬 실행

  deploy:
    needs: [test-unit, test-e2e]  # 병렬 테스트 완료 후
```

## 마치며

GitHub Actions를 활용한 CI/CD 파이프라인은 단순한 자동화를 넘어 **팀의 개발 문화를 바꿉니다**. 수동 배포의 부담에서 벗어나 코드 작성에 집중할 수 있고, 자동화된 테스트로 품질을 보장하며, 빠른 피드백 루프로 생산성을 향상시킵니다.

**핵심 포인트:**

1. **작게 시작하기**: 간단한 린트와 테스트부터 시작
2. **점진적 개선**: 필요에 따라 단계적으로 복잡도 증가
3. **보안 우선**: Secrets 관리와 최소 권한 원칙
4. **모니터링**: 배포 알림과 롤백 전략 필수
5. **문서화**: 팀원들이 이해할 수 있도록 워크플로우 문서화

처음에는 복잡해 보일 수 있지만, 한 번 구축하면 엄청난 생산성 향상을 경험할 수 있습니다. 이 가이드를 참고하여 여러분의 프로젝트에 맞는 CI/CD 파이프라인을 구축해보세요!

> 완벽한 CI/CD는 없습니다. 팀의 필요에 맞게 지속적으로 개선하는 것이 중요합니다.
{: .prompt-info }

## 참고 자료

### GitHub Actions 공식 문서
- [GitHub Actions 문서](https://docs.github.com/en/actions)
- [Workflow 문법](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
- [Marketplace](https://github.com/marketplace?type=actions)
- [Security Hardening](https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions)

### CI/CD 베스트 프랙티스
- [CI/CD Best Practices (GitLab)](https://about.gitlab.com/topics/ci-cd/)
- [Continuous Delivery (Martin Fowler)](https://martinfowler.com/bliki/ContinuousDelivery.html)
- [The Twelve-Factor App](https://12factor.net/)

### Docker
- [Docker Documentation](https://docs.docker.com/)
- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)
- [Dockerfile Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

### 테스팅
- [Jest Documentation](https://jestjs.io/docs/getting-started)
- [Playwright Documentation](https://playwright.dev/)
- [Testing Library](https://testing-library.com/)

### 보안
- [OWASP CI/CD Security](https://owasp.org/www-project-devsecops-guideline/)
- [GitHub Security Best Practices](https://docs.github.com/en/code-security)
- [Snyk](https://snyk.io/)

### 모니터링 및 알림
- [Slack Incoming Webhooks](https://api.slack.com/messaging/webhooks)
- [Sentry](https://sentry.io/)
- [Datadog](https://www.datadoghq.com/)

### 예제 프로젝트
- [Next.js Examples](https://github.com/vercel/next.js/tree/canary/examples)
- [Awesome Actions](https://github.com/sdras/awesome-actions)
- [GitHub Actions Examples](https://github.com/actions/starter-workflows)
