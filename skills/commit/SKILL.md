---
name: commit
description: Creates a git commit with proper message formatting. Use when committing staged changes with a descriptive commit message.
allowed-tools: Bash(git add:*) Bash(git status:*) Bash(git commit:*)
---

## 컨텍스트

- 현재 git 상태: !`git status`
- 현재 diff (staged + unstaged): !`git diff HEAD`
- 추적되지 않는 파일: !`git ls-files --others --exclude-standard`
- 현재 브랜치: !`git branch --show-current`
- 최근 커밋: !`git log --oneline -10`

## 작업

위 변경사항을 기반으로 단일 git commit을 생성한다.

## 가이드라인

### 1. 추적되지 않는 파일 처리

- 위 추적되지 않는 파일 목록을 검토한다
- 현재 변경과 관련된 추적되지 않는 파일은 커밋에 포함한다
- `.claude/` 디렉토리 파일은 항상 포함한다
- 포함 기준: 새 소스 파일, 설정 파일, 문서
- 제외 기준: 빌드 산출물, 임시 파일, .env 파일
- 판단 기준: "이 파일이 커밋하려는 논리적 변경의 일부인가?"
- 특수문자(괄호, 공백 등)가 포함된 경로는 반드시 따옴표로 감싼다: `git add "path/with[brackets]/file.ts"`

### 2. 커밋 메시지 형식

**제목 줄: 50자 이내.** 짧은 동사(Add, Fix, Update, Remove, Refactor) + 간결한 명사구를 사용한다.

좋은 예시:
- "Add user auth module" (20)
- "Fix login form validation" (25)
- "Refactor database connection pool" (34)

길면 줄인다:
- "Implement investment proposal management feature" (48) → "Add proposal management" (22)
- "Reorganize skills into subdirectory and improve metadata" (56) → "Reorganize skills directory" (26)

규칙:
1. 제목 줄은 50자 이내로 작성한다
2. *무엇*이 변경되었는지 기술한다 (이유는 본문에 작성)
3. 불필요한 단어(the, a, for the, in the 등)를 생략한다
4. 접두사 없이 순수 변경 설명만 작성한다 — 브랜치명, 티켓번호, conventional commit 접두사(feat:, fix: 등) 모두 제외

**커밋 전 확인:** 제목이 50자를 초과하면 형용사를 제거하고 명사구를 압축하여 줄인다.

### 3. 커밋 본문 (선택)

제목만으로 충분하지 않을 때 본문을 추가한다:
- 제목 뒤에 빈 줄 하나를 남긴다
- 본문에 변경 *이유*를 설명한다
- 본문 줄은 72자에서 줄바꿈한다
