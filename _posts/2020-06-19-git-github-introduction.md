---
title: Git & GitHub 완벽 가이드 - 버전 관리와 협업의 필수 도구
description: Git의 기본 개념부터 Git Flow, 주요 명령어까지 버전 관리 시스템을 완벽하게 이해합니다. GitHub를 활용한 협업 방법도 함께 다룹니다.
author: changsu
date: 2020-06-19 10:45:31 +0900
categories: [DevOps, Git]
tags: [git, github, version-control, collaboration, git-flow, branch, merge]
---

## 개요

Git은 파일의 변경사항을 추적하고 여러 명이 협업할 수 있게 해주는 분산 버전 관리 시스템(VCS)입니다. GitHub는 Git 저장소를 클라우드에서 호스팅하고 협업 기능을 제공하는 플랫폼입니다. 현대 소프트웨어 개발에서 필수적인 도구로, 코드의 이력을 관리하고 팀원들과 효율적으로 협업할 수 있게 해줍니다.

## 1. Git이란?

### 정의

**Git**은 파일의 변경사항을 추적하고 여러 사용자 간의 작업을 조율하기 위한 **분산 버전 관리 시스템**입니다.

### 주요 특징

1. **분산 버전 관리**: 모든 개발자가 전체 이력을 가진 저장소 복사본 보유
2. **빠른 성능**: 대부분의 작업이 로컬에서 수행됨
3. **데이터 무결성**: SHA-1 해시로 모든 데이터 보장
4. **비선형 워크플로**: 브랜치를 통한 유연한 개발 흐름
5. **오픈소스**: 무료로 사용 가능

### Git이 필요한 이유

```
❌ Git 없이:
- 파일명으로 버전 관리: project_v1.js, project_v2_final.js, project_v2_final_real.js
- 이전 버전으로 돌아가기 어려움
- 협업 시 파일 충돌 해결 불가능
- 누가 무엇을 변경했는지 추적 불가

✅ Git 사용:
- 체계적인 버전 관리
- 언제든 이전 버전으로 복원 가능
- 브랜치로 독립적인 작업 공간 제공
- 변경 이력과 작성자 자동 기록
- 충돌 시 자동 감지 및 병합 도구 제공
```

## 2. Git 핵심 개념

### 1) Repository (저장소)

프로젝트의 모든 파일과 변경 이력이 저장되는 공간입니다.

**종류:**

```
┌─────────────────────────────────────┐
│   Remote Repository (원격 저장소)    │
│   - GitHub, GitLab, Bitbucket      │
│   - 여러 사람이 공유               │
│   - 클라우드에 호스팅              │
└─────────────────────────────────────┘
              ↕ (push/pull)
┌─────────────────────────────────────┐
│   Local Repository (로컬 저장소)    │
│   - 내 컴퓨터에 저장               │
│   - .git 폴더에 이력 저장          │
│   - 오프라인 작업 가능             │
└─────────────────────────────────────┘
```

### 2) Commit (커밋)

프로젝트의 특정 시점의 스냅샷입니다.

```bash
# 커밋 구성 요소
commit a1b2c3d4
Author: Alice <alice@example.com>
Date:   Mon Jun 19 10:30:00 2023 +0900

    Add user authentication feature

    - Implement login/logout
    - Add JWT token validation
    - Create user session management
```

**특징:**
- 고유한 해시 ID (SHA-1)
- 작성자, 날짜, 메시지 포함
- 이전 커밋을 가리키는 포인터
- 변경된 파일의 스냅샷

### 3) Stage (스테이징)

커밋에 포함할 변경사항을 선택하는 영역입니다.

```
┌──────────────────────────────────────┐
│  Working Directory (작업 디렉토리)    │
│  - 실제 파일들이 있는 공간           │
│  - 파일 수정, 추가, 삭제             │
└──────────────────────────────────────┘
         ↓ (git add)
┌──────────────────────────────────────┐
│  Staging Area (스테이징 영역/Index)  │
│  - 커밋 준비 영역                    │
│  - 선택적으로 파일 추가              │
└──────────────────────────────────────┘
         ↓ (git commit)
┌──────────────────────────────────────┐
│  Repository (.git directory)         │
│  - 확정된 변경사항 저장              │
│  - 영구적으로 이력 보관              │
└──────────────────────────────────────┘
```

**예시:**

```bash
# 5개의 파일 수정
# Working Directory
file1.js (수정됨)
file2.js (수정됨)
file3.js (수정됨)
file4.js (수정됨)
file5.js (수정됨)

# Staging Area에 선택적으로 추가
git add file1.js file2.js file3.js
# file1, file2, file3만 스테이징

# 커밋 (스테이징된 파일만)
git commit -m "Update feature A"
# file4, file5는 커밋되지 않음
```

### 4) Branch (브랜치)

독립적인 개발 라인으로, 포인터의 역할을 합니다.

```
      A---B---C  master
           \
            D---E  feature/login
```

**특징:**
- 가볍고 빠름 (포인터일 뿐)
- 독립적인 작업 공간 제공
- 무제한 생성 가능
- 작업 완료 후 병합 또는 삭제

**주요 브랜치:**

| 브랜치 | 설명 |
|--------|------|
| **master/main** | 메인 브랜치 (배포 가능한 상태) |
| **develop** | 개발 브랜치 (개발 중인 코드) |
| **feature/** | 기능 개발 브랜치 |
| **hotfix/** | 긴급 버그 수정 브랜치 |

### 5) Checkout (체크아웃)

브랜치나 커밋 간 이동입니다.

```bash
# 브랜치 이동
git checkout feature/login

# 이전 커밋으로 이동
git checkout a1b2c3d

# 새 브랜치 생성하며 이동
git checkout -b feature/new-feature
```

> **Info**: Git 2.23 이상에서는 `git switch`와 `git restore`로 분리되었습니다.
{: .prompt-info }

### 6) Merge (병합)

여러 브랜치를 하나로 합치는 작업입니다.

```
# Fast-forward merge (충돌 없음)
      A---B---C  master
           \
            D---E  feature

      ↓ merge

      A---B---C---D---E  master
```

```
# 3-way merge (변경사항 통합)
      A---B---C  master
           \
            D---E  feature

      ↓ merge

      A---B---C-------F  master
           \         /
            D-------E  feature
```

**Merge 종류:**

1. **Fast-forward**: 충돌 없이 자동 병합
2. **3-way merge**: 양쪽 변경사항 통합
3. **Conflict**: 충돌 발생, 수동 해결 필요

### 7) Clone (클론)

원격 저장소를 로컬로 복사합니다.

```bash
git clone https://github.com/user/repo.git
```

**결과:**
- 전체 프로젝트 다운로드
- .git 폴더 포함 (전체 이력)
- 자동으로 원격 저장소 연결

### 8) Push (푸시)

로컬 변경사항을 원격 저장소에 업로드합니다.

```bash
git push origin master
```

**동작:**
- 로컬 커밋을 원격으로 전송
- 다른 사람이 내 코드 확인 가능
- 원격 저장소 업데이트

### 9) Pull (풀)

원격 저장소의 변경사항을 로컬로 가져옵니다.

```bash
git pull origin master
```

**동작:**
- 원격 변경사항 다운로드
- 자동으로 현재 브랜치에 병합
- `git fetch` + `git merge`

**Clone vs Pull:**

```
Clone: 처음 프로젝트 전체 다운로드
├─ .git 폴더 생성
├─ 전체 이력 복사
└─ 한 번만 실행

Pull: 업데이트된 부분만 가져오기
├─ 기존 저장소 필요
├─ 변경사항만 다운로드
└─ 주기적으로 실행
```

## 3. Git Flow

### Git Flow란?

브랜치를 체계적으로 관리하기 위한 **방법론**입니다.

### 5가지 브랜치 전략

```
          hotfix/     release/
             ↓           ↓
    ┌────────●───────────●────── master (배포)
    │        ↑           ↑
    │        └─────┬─────┘
    │              │
    ├──────────────●────────────── develop (개발)
    │         ↗    ↑    ↖
    │    feature/ feature/ feature/
    │      ↓       ↓       ↓
    └──────●───────●───────●────── feature (기능)
```

#### 1. master (메인 배포)

```
역할: 프로덕션 배포 가능한 상태 유지
특징:
  - 항상 안정적
  - 릴리스된 버전만 존재
  - 태그로 버전 관리
사용: 배포 시점에만 업데이트
```

#### 2. develop (메인 개발)

```
역할: 다음 릴리스를 위한 개발 브랜치
특징:
  - 최신 개발 내용 포함
  - feature 브랜치 병합 대상
  - 테스트 서버에 배포
사용: 지속적인 개발
```

#### 3. feature (기능 개발)

```
역할: 새로운 기능 개발
생성: develop에서 분기
병합: develop으로 병합
네이밍: feature/기능명
  - feature/user-auth
  - feature/shopping-cart
  - feature/payment
삭제: 병합 후 삭제
```

#### 4. release (배포 준비)

```
역할: 배포 전 최종 테스트 및 버그 수정
생성: develop에서 분기
병합: master와 develop 모두
네이밍: release/버전
  - release/1.0.0
  - release/1.1.0
작업:
  - 버그 수정
  - 문서 정리
  - 버전 번호 업데이트
```

#### 5. hotfix (긴급 수정)

```
역할: 프로덕션 긴급 버그 수정
생성: master에서 분기
병합: master와 develop 모두
네이밍: hotfix/버그명
  - hotfix/critical-bug
  - hotfix/security-issue
특징:
  - develop/feature 거치지 않음
  - 빠른 배포 가능
```

### Git Flow 흐름도

```
1. 프로젝트 시작
   master ──┬── develop

2. 기능 개발
   develop ──┬── feature/login
             ├── feature/cart
             └── feature/payment

3. 기능 완료 및 병합
   feature/login ──→ develop (merge)
   feature/cart ──→ develop (merge)

4. 릴리스 준비
   develop ──┬── release/1.0

5. 테스트 및 버그 수정
   release/1.0에서 수정

6. 배포
   release/1.0 ──→ master (merge, tag v1.0)
   release/1.0 ──→ develop (merge)

7. 긴급 버그 발견
   master ──┬── hotfix/critical-bug

8. 긴급 수정 배포
   hotfix/critical-bug ──→ master (merge, tag v1.0.1)
   hotfix/critical-bug ──→ develop (merge)
```

### 실전 예시

```bash
# 1. develop에서 feature 브랜치 생성
git checkout develop
git checkout -b feature/user-authentication

# 2. 기능 개발
git add .
git commit -m "Add login functionality"
git commit -m "Add logout functionality"

# 3. develop에 병합
git checkout develop
git merge feature/user-authentication

# 4. feature 브랜치 삭제
git branch -d feature/user-authentication

# 5. 릴리스 준비
git checkout -b release/1.0.0

# 6. 버그 수정 후 master에 병합
git checkout master
git merge release/1.0.0
git tag -a v1.0.0 -m "Release version 1.0.0"

# 7. develop에도 병합
git checkout develop
git merge release/1.0.0

# 8. release 브랜치 삭제
git branch -d release/1.0.0
```

## 4. Fork와 Pull Request

대규모 오픈소스 프로젝트에서 사용하는 협업 방식입니다.

### Fork

```
원본 저장소 (upstream)
    ↓ Fork
내 저장소 (origin)
    ↓ Clone
로컬 저장소 (local)
```

**특징:**
- 저장소 전체를 복제
- 독립적인 저장소 생성
- 원본에 영향 없이 작업

### Pull Request (PR)

```
1. Fork된 저장소에서 작업
2. 변경사항 커밋 및 푸시
3. 원본 저장소에 PR 생성
4. 코드 리뷰
5. 원본 관리자가 병합 결정
```

**장점:**
- 코드 리뷰 가능
- 품질 관리
- 협업 및 피드백
- 권한 없이도 기여 가능

### 실전 워크플로우

```bash
# 1. 원본 저장소 Fork (GitHub 웹에서)

# 2. Fork한 저장소 Clone
git clone https://github.com/my-account/project.git

# 3. 원본 저장소를 upstream으로 추가
git remote add upstream https://github.com/original/project.git

# 4. 새 브랜치 생성
git checkout -b feature/my-feature

# 5. 작업 및 커밋
git add .
git commit -m "Add my feature"

# 6. 내 저장소에 푸시
git push origin feature/my-feature

# 7. GitHub에서 Pull Request 생성

# 8. 코드 리뷰 및 수정 요청 대응

# 9. 원본 저장소 최신 상태 유지
git fetch upstream
git merge upstream/master
```

## 5. 주요 Git 명령어

### 초기 설정

```bash
# Git 설치 확인
git --version

# 사용자 정보 설정
git config --global user.name "Your Name"
git config --global user.email "your@email.com"

# 설정 확인
git config --list
```

### 저장소 생성

```bash
# 새 저장소 초기화
git init

# 원격 저장소 복제
git clone <url>
```

### 변경사항 관리

```bash
# 상태 확인
git status

# 파일 스테이징
git add <file>           # 특정 파일
git add .                # 모든 변경사항
git add *.js             # 패턴 매칭

# 스테이징 취소
git restore --staged <file>

# 커밋
git commit -m "message"
git commit -am "message"  # add + commit

# 커밋 수정
git commit --amend
```

### 이력 확인

```bash
# 커밋 로그
git log
git log --oneline
git log --graph
git log --all --decorate --oneline --graph

# 특정 파일 이력
git log <file>

# 변경사항 확인
git diff                 # 작업 디렉토리 vs 스테이징
git diff --staged        # 스테이징 vs 저장소
git diff <commit>        # 특정 커밋과 비교
```

### 브랜치 관리

```bash
# 브랜치 목록
git branch
git branch -a            # 모든 브랜치 (원격 포함)

# 브랜치 생성
git branch <name>

# 브랜치 이동
git checkout <name>
git switch <name>        # Git 2.23+

# 생성과 동시에 이동
git checkout -b <name>
git switch -c <name>

# 브랜치 삭제
git branch -d <name>     # 병합된 브랜치만
git branch -D <name>     # 강제 삭제

# 브랜치 이름 변경
git branch -m <old> <new>
```

### 병합

```bash
# 브랜치 병합
git merge <branch>

# 충돌 해결 후
git add <resolved-files>
git commit

# 병합 취소
git merge --abort
```

### 원격 저장소

```bash
# 원격 저장소 확인
git remote -v

# 원격 저장소 추가
git remote add origin <url>

# 원격 저장소 제거
git remote remove origin

# 푸시
git push origin <branch>
git push -u origin <branch>  # 추적 브랜치 설정

# 풀
git pull origin <branch>

# 페치 (병합 안 함)
git fetch origin
```

### 실전 시나리오

#### 새 프로젝트 시작

```bash
# 1. 저장소 초기화
git init

# 2. 파일 추가
git add README.md

# 3. 첫 커밋
git commit -m "Initial commit"

# 4. 원격 저장소 연결
git remote add origin https://github.com/user/repo.git

# 5. 푸시
git push -u origin master
```

#### 기존 프로젝트 참여

```bash
# 1. 클론
git clone https://github.com/user/repo.git

# 2. 브랜치 생성
git checkout -b feature/my-feature

# 3. 작업
# ... 파일 수정 ...

# 4. 커밋
git add .
git commit -m "Add my feature"

# 5. 푸시
git push origin feature/my-feature

# 6. PR 생성 (GitHub에서)
```

#### 협업 중 충돌 해결

```bash
# 1. 최신 코드 받기
git pull origin develop

# 2. 충돌 발생
# CONFLICT (content): Merge conflict in file.js

# 3. 충돌 파일 확인
git status

# 4. 충돌 해결 (파일 수동 편집)
<<<<<<< HEAD
내 코드
=======
다른 사람 코드
>>>>>>> branch-name

# 5. 해결 후 스테이징
git add file.js

# 6. 병합 커밋
git commit -m "Resolve merge conflict"
```

## 6. GitHub란?

### 정의

**GitHub**는 Git 저장소 호스팅 서비스이자 개발자 협업 플랫폼입니다.

### Git vs GitHub

| 구분 | Git | GitHub |
|------|-----|--------|
| **타입** | 프로그램 (VCS) | 웹 서비스 |
| **위치** | 로컬 컴퓨터 | 클라우드 |
| **역할** | 버전 관리 | 호스팅 + 협업 |
| **사용** | 명령어 | 웹 인터페이스 |

### GitHub 주요 기능

#### 1. 저장소 호스팅

```
- 무제한 공개 저장소
- Private 저장소 (무료/유료)
- 자동 백업
- 웹에서 코드 탐색
```

#### 2. 협업 도구

```
- Pull Request
- Code Review
- Issues (버그 추적)
- Projects (칸반 보드)
- Wiki (문서화)
```

#### 3. 소셜 코딩

```
- 다른 개발자 팔로우
- Star (즐겨찾기)
- Fork (복제)
- Contribution 그래프
```

#### 4. CI/CD

```
- GitHub Actions
- 자동 테스트
- 자동 배포
- Workflow 관리
```

#### 5. 보안

```
- Dependabot (의존성 보안 경고)
- Code scanning
- Secret scanning
- Security advisories
```

### GitHub 활용 예시

```markdown
# README.md
프로젝트 소개, 설치 방법, 사용법

# Issues
버그 리포트, 기능 요청, 질문

# Pull Requests
코드 리뷰, 병합 요청

# Projects
작업 관리, 진행 상황 추적

# Wiki
상세 문서, 가이드
```

## 7. Best Practices

### 커밋 메시지

```bash
# ✅ 좋은 예
git commit -m "Add user authentication feature"
git commit -m "Fix login button not working on mobile"
git commit -m "Refactor database connection logic"

# ❌ 나쁜 예
git commit -m "update"
git commit -m "fix bug"
git commit -m "asdf"
```

**컨벤션:**

```
<type>: <subject>

<body>

<footer>
```

**타입:**
- `feat`: 새 기능
- `fix`: 버그 수정
- `docs`: 문서 변경
- `style`: 코드 포맷팅
- `refactor`: 리팩토링
- `test`: 테스트 추가
- `chore`: 빌드/설정 변경

### 브랜치 전략

```bash
# ✅ 명확한 브랜치명
feature/user-authentication
feature/shopping-cart
bugfix/login-error
hotfix/security-issue

# ❌ 모호한 브랜치명
test
new-feature
fix
my-branch
```

### .gitignore

```gitignore
# 의존성
node_modules/
vendor/

# 환경 변수
.env
.env.local

# 빌드 결과물
dist/
build/
*.min.js

# 로그
*.log
logs/

# OS 파일
.DS_Store
Thumbs.db

# IDE
.vscode/
.idea/
*.swp
```

### 자주 커밋하기

```bash
# ✅ 작은 단위로 자주 커밋
git commit -m "Add login form HTML"
git commit -m "Add login form validation"
git commit -m "Add login API integration"

# ❌ 큰 변경사항을 한번에
git commit -m "Complete entire login feature"
```

## 8. 일반적인 Git 에러 해결

### 1. 충돌 (Conflict)

```bash
# 문제
CONFLICT (content): Merge conflict in file.js

# 해결
# 1. 충돌 파일 열기
# 2. <<<<<<< ======= >>>>>>> 마커 찾기
# 3. 원하는 코드 선택
# 4. 마커 제거
# 5. git add file.js
# 6. git commit
```

### 2. 잘못된 커밋

```bash
# 마지막 커밋 메시지 수정
git commit --amend

# 마지막 커밋에 파일 추가
git add forgotten-file.js
git commit --amend --no-edit

# 커밋 취소 (파일은 유지)
git reset --soft HEAD~1

# 커밋 취소 (파일도 되돌림)
git reset --hard HEAD~1
```

### 3. 푸시 거부

```bash
# 문제
! [rejected] master -> master (non-fast-forward)

# 해결 1: Pull 먼저
git pull origin master
# 충돌 해결 후
git push origin master

# 해결 2: Rebase
git pull --rebase origin master
git push origin master
```

### 4. 브랜치 삭제 오류

```bash
# 문제
error: The branch 'feature' is not fully merged.

# 해결 (확실한 경우만)
git branch -D feature
```

## 9. 유용한 Git 명령어

### 히스토리 탐색

```bash
# 예쁜 로그
git log --graph --pretty=format:'%C(yellow)%h%Creset -%C(red)%d%Creset %s %C(green)(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit

# 특정 파일의 변경 이력
git log -p <file>

# 코드 작성자 확인
git blame <file>

# 파일 변경 추적
git log --follow <file>
```

### Stash (임시 저장)

```bash
# 현재 작업 임시 저장
git stash

# 저장 목록 확인
git stash list

# 복원
git stash apply
git stash pop  # apply + drop

# 삭제
git stash drop
git stash clear  # 전체 삭제
```

### 태그

```bash
# 태그 생성
git tag v1.0.0
git tag -a v1.0.0 -m "Release version 1.0.0"

# 태그 목록
git tag

# 태그 푸시
git push origin v1.0.0
git push origin --tags  # 모든 태그
```

### Cherry-pick

```bash
# 특정 커밋만 가져오기
git cherry-pick <commit-hash>
```

### Rebase

```bash
# 브랜치 재배치
git rebase master

# 인터랙티브 리베이스 (커밋 정리)
git rebase -i HEAD~3
```

## 정리

### Git 기본 워크플로우

```bash
# 1. 저장소 시작
git init / git clone

# 2. 브랜치 생성
git checkout -b feature/new-feature

# 3. 작업
# ... 파일 수정 ...

# 4. 스테이징
git add .

# 5. 커밋
git commit -m "Add new feature"

# 6. 푸시
git push origin feature/new-feature

# 7. PR 생성 (GitHub)

# 8. 병합 후 브랜치 삭제
git branch -d feature/new-feature
```

### 핵심 명령어

| 명령어 | 설명 |
|--------|------|
| `git init` | 저장소 초기화 |
| `git clone` | 저장소 복제 |
| `git add` | 스테이징 |
| `git commit` | 커밋 |
| `git push` | 원격 업로드 |
| `git pull` | 원격 다운로드 |
| `git branch` | 브랜치 관리 |
| `git checkout` | 브랜치 이동 |
| `git merge` | 브랜치 병합 |
| `git status` | 상태 확인 |

### Git Flow 브랜치

```
master    ──●───────●───── (프로덕션 배포)
             ↑       ↑
develop   ──●───●───●───── (개발)
             ↗   ↖   ↗
feature   ──●─────●─────── (기능 개발)
```

### GitHub 협업

```
1. Fork: 저장소 복제
2. Clone: 로컬로 다운로드
3. Branch: 새 브랜치 생성
4. Commit: 변경사항 커밋
5. Push: 내 저장소로 푸시
6. Pull Request: 병합 요청
7. Code Review: 리뷰 및 수정
8. Merge: 원본에 병합
```

## 참고 자료

- [Git 공식 문서](https://git-scm.com/doc)
- [Pro Git 책 (무료)](https://git-scm.com/book/ko/v2)
- [GitHub Docs](https://docs.github.com)
- [Git Flow 원문](https://nvie.com/posts/a-successful-git-branching-model/)
- [Atlassian Git Tutorials](https://www.atlassian.com/git/tutorials)
- [Learn Git Branching (시각화)](https://learngitbranching.js.org/?locale=ko)