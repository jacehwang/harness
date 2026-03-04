# jacehwang/harness

코딩 에이전트를 위한 skill과 subagent 모음입니다.

## 설치

### Skills만 사용하는 경우

[Skills CLI](https://skills.sh)로 간편하게 설치할 수 있습니다.

```bash
bunx skills add jacehwang/harness
npx skills add jacehwang/harness
```

설치하면 `/commit`, `/pr`, `/address-reviews`, `/plan-ticket`, `/explore-test` 등의 명령을 바로 사용할 수 있습니다.

### Subagent도 함께 사용하는 경우

Subagent는 현재 Claude Code에서만 지원되며, plugin으로 설치해야 사용할 수 있습니다.

```bash
# marketplace 등록 (최초 1회)
/plugin marketplace add jacehwang/harness

# plugin 설치
/plugin install jace@harness
```

Plugin으로 설치하면 skill 명령에 네임스페이스가 붙습니다 (예: `/jace:commit`).

## Skills

| Skill | 설명 | 업데이트 |
|-------|------|---------|
| commit | 컨벤션에 맞는 커밋 메시지로 git commit 생성 | [![updated][commit-badge]][commit-log] |
| pr | GitHub PR 생성 및 업데이트 | [![updated][pr-badge]][pr-log] |
| address-reviews | PR 리뷰 코멘트 분류 및 대응 계획 수립 | [![updated][ar-badge]][ar-log] |
| plan-ticket | Linear 티켓에 목표·구현 계획을 작성하여 코딩 에이전트가 자율 작업 가능하도록 준비 | [![updated][pt-badge]][pt-log] |
| explore-test | 현재 파일 변경사항에서 탐색적 테스팅 시나리오 도출 (Charter + Oracle + Tour) | [![updated][et-badge]][et-log] |

## Subagents

> Plugin으로 설치한 경우에만 사용할 수 있습니다.

| Agent | 설명 | 업데이트 |
|-------|------|---------|
| prompt-doctor | 프롬프트 분석 및 개선 | [![updated][pd-badge]][pd-log] |

## 개발 및 테스트

```bash
claude --plugin-dir ./
```

<!-- Badges -->
[commit-badge]: https://img.shields.io/github/last-commit/jacehwang/harness?path=skills/commit&style=flat-square&label=updated
[commit-log]: https://github.com/jacehwang/harness/commits/main/skills/commit
[pr-badge]: https://img.shields.io/github/last-commit/jacehwang/harness?path=skills/pr&style=flat-square&label=updated
[pr-log]: https://github.com/jacehwang/harness/commits/main/skills/pr
[ar-badge]: https://img.shields.io/github/last-commit/jacehwang/harness?path=skills/address-reviews&style=flat-square&label=updated
[ar-log]: https://github.com/jacehwang/harness/commits/main/skills/address-reviews
[pt-badge]: https://img.shields.io/github/last-commit/jacehwang/harness?path=skills/plan-ticket&style=flat-square&label=updated
[pt-log]: https://github.com/jacehwang/harness/commits/main/skills/plan-ticket
[et-badge]: https://img.shields.io/github/last-commit/jacehwang/harness?path=skills/explore-test&style=flat-square&label=updated
[et-log]: https://github.com/jacehwang/harness/commits/main/skills/explore-test
[pd-badge]: https://img.shields.io/github/last-commit/jacehwang/harness?path=agents/prompt-doctor.md&style=flat-square&label=updated
[pd-log]: https://github.com/jacehwang/harness/commits/main/agents/prompt-doctor.md
