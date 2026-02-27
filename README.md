# jacehwang/harness

코딩 에이전트를 위한 skill과 subagent 모음입니다.

## 설치

### Skills만 사용하는 경우

[Skills CLI](https://skills.sh)로 간편하게 설치할 수 있습니다.

```bash
bunx skills add jacehwang/harness
```

설치하면 `/commit`, `/pr`, `/address-reviews` 등의 명령을 바로 사용할 수 있습니다.

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

| Skill | 설명 |
|-------|------|
| commit | 컨벤션에 맞는 커밋 메시지로 git commit 생성 |
| pr | GitHub PR 생성 및 업데이트 |
| address-reviews | PR 리뷰 코멘트 분류 및 대응 계획 수립 |

## Subagents

> Plugin으로 설치한 경우에만 사용할 수 있습니다.

| Agent | 설명 |
|-------|------|
| prompt-doctor | 프롬프트 분석 및 개선 (experimental) |

prompt-doctor는 아직 활발히 다듬고 있는 단계라, 동작이나 출력이 바뀔 수 있습니다.

## 개발 및 테스트

```bash
claude --plugin-dir ./
```
