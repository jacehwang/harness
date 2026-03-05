# jacehwang/harness

코딩 에이전트를 위한 skill 모음입니다.

## 설치

### Skills CLI

[Skills CLI](https://skills.sh)로 간편하게 설치할 수 있습니다.

```bash
bunx skills add jacehwang/harness
npx skills add jacehwang/harness
```

설치하면 `/prompt-doctor`, `/explore-test`, `/code-review`, `/address-findings` 등의 명령을 바로 사용할 수 있습니다.

### Claude Code Marketplace

Claude Code의 plugin marketplace를 통해 설치할 수도 있습니다.

```bash
# marketplace 등록 (최초 1회)
/plugin marketplace add jacehwang/harness

# plugin 설치
/plugin install jace@harness
```

Plugin으로 설치하면 skill 명령에 네임스페이스가 붙습니다 (예: `/jace:commit`).

## Skills

| Skill | 설명 |
|-------|------|
| commit | 컨벤션에 맞는 커밋 메시지로 git commit 생성 |
| pr | GitHub PR 생성 및 업데이트 |
| address-findings | code-review 결과를 Core vs Peripheral 분석하여 우선순위별 수정 계획 수립 |
| plan-ticket | Linear 티켓에 목표·구현 계획을 작성하여 코딩 에이전트가 자율 작업 가능하도록 준비 |
| explore-test | 현재 파일 변경사항에서 탐색적 테스팅 시나리오 도출 (Charter + Oracle + Tour) |
| internalize | 코딩 세션 중 인간 개입을 분석하여 에이전트 프롬프트 파일 강화 |
| code-review | git 변경 파일의 정확성 버그, 보안 취약점, 결함 리스크 리뷰 |
| prompt-doctor | 8개 학술 프레임워크 기반 프롬프트 분석 및 최적화 |

## 개발 및 테스트

```bash
claude --plugin-dir ./
```
