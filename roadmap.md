# Just Code Review (JCR) - 기획서

## 개요

Claude Code에서 사용하는 코드 리뷰 스킬 모음.
코드 품질에 집중하며, 언어/프레임워크에 관계없이 동작한다.

## 목표

- 불필요한 코드, 미흡한 주석, 네이밍, 중복, 복잡도 등 코드 품질 이슈를 체계적으로 리뷰
- 리뷰 관점별로 references 파일을 두어 일관된 기준 제공
- 리뷰 대상을 유연하게 지정 (git diff, 파일, 폴더)

## 스킬 구성

### `/jcr-review` - 코드 품질 리뷰 (핵심)

대상 코드의 품질을 종합적으로 리뷰한다.

```yaml
---
name: jcr-review
description: "코드 품질 리뷰. 사용자가 코드 리뷰, 코드 품질 체크, 코드 점검, 코드 검토를 요청할 때 사용. git diff, 파일, 폴더 단위로 리뷰 가능."
---
```

- 리뷰 대상:
  - `git diff` (기본, unstaged 변경사항)
  - `git diff --staged` (staged 변경사항)
  - `git diff main...HEAD` (브랜치 전체 변경사항)
  - 파일 / 폴더 경로 직접 지정
  - 인자로 대상 지정: `/jcr-review src/utils.ts`, `/jcr-review --staged`
- 리뷰 관점:
  - 불필요한 변수, import, 죽은 코드
  - 주석 품질 (부정확, 불필요, 누락)
  - 네이밍 (변수, 함수, 클래스)
  - 코드 중복 / 재사용 가능 패턴
  - 함수 복잡도 / 책임 분리
  - 매직 넘버 / 하드코딩된 값
  - 에러 처리 (빈 catch, 에러 삼킴, 과도한 예외 포착)
  - 코드 스타일 일관성
- 리뷰 제외 대상:
  - 자동 생성 코드 (*.generated.*, *.g.*)
  - vendor / node_modules / lock 파일
  - 테스트 fixture / snapshot
- 출력: 심각도 + 파일:라인번호 + 문제 설명 + 개선 제안 + 요약 통계
- 프로젝트별 설정: 프로젝트 루트의 `.jcr.md` 파일을 참조하여 팀/프로젝트별 컨벤션 적용

### `/jcr-refactor` - 리팩토링 제안

리뷰를 넘어서 구체적인 리팩토링 방안을 제시한다.
jcr-review의 references 파일을 공유하여 동일한 기준으로 판단한다.

```yaml
---
name: jcr-refactor
description: "코드 리팩토링 제안. 사용자가 리팩토링, 코드 구조 개선, 코드 정리를 요청할 때 사용. 실제 변경 코드를 포함한 구체적인 리팩토링 방안을 제시."
---
```

- 추출 가능한 함수/컴포넌트
- 단순화 가능한 로직
- 패턴 적용 기회
- 실제 리팩토링 코드 제안 포함
- references 참조: `../jcr-review/references/` 경로의 파일들을 공유

## 디렉토리 구조

```
skills/
├── jcr-review/
│   ├── SKILL.md                  ← 종합 리뷰 스킬 정의 (frontmatter + 메타 룰 포함)
│   └── references/
│       ├── dead-code.md          ← 불필요한 변수/import/코드
│       ├── comments.md           ← 주석 품질
│       ├── naming.md             ← 네이밍 컨벤션
│       ├── duplication.md        ← 중복/재사용 가능 패턴
│       ├── complexity.md         ← 함수 복잡도/책임 분리
│       ├── magic-values.md       ← 매직 넘버/하드코딩
│       ├── error-handling.md     ← 에러 처리 패턴
│       └── style.md              ← 코드 스타일 일관성
└── jcr-refactor/
    └── SKILL.md                  ← 리팩토링 제안 스킬 (jcr-review/references/ 공유)
```

## references 파일 설계

각 references 파일은 다음 구조를 따른다:

```markdown
# [관점 이름]

## 왜 중요한가
- 이 관점이 코드 품질에 미치는 영향과 이유

## 원칙
- 이 관점에서 지켜야 할 핵심 원칙

## 체크리스트
- 구체적인 검사 항목들

## 좋은 예 / 나쁜 예
- 언어 중립적인 예시 (필요시 주요 언어별 예시)

## 안티패턴
- 흔히 발생하는 실수들
```

## SKILL.md의 references 참조 방식

SKILL.md body에서 각 references 파일을 나열하고, 어떤 상황에서 참조할지 안내한다:

```markdown
## References

리뷰 시 대상 코드의 특성에 맞는 references를 선택적으로 참조한다:

- `references/dead-code.md` — 불필요한 변수, 미사용 import, 도달 불가 코드 탐지 시
- `references/comments.md` — 주석의 정확성, 필요성, 누락 여부 판단 시
- `references/naming.md` — 변수, 함수, 클래스 네이밍 평가 시
- `references/duplication.md` — 코드 중복, 재사용 가능 패턴 식별 시
- `references/complexity.md` — 함수 길이, 분기 복잡도, 책임 분리 판단 시
- `references/magic-values.md` — 매직 넘버, 하드코딩된 문자열 탐지 시
- `references/error-handling.md` — 에러 처리 패턴 평가 시
- `references/style.md` — 코드 스타일 일관성 확인 시
```

## 프로젝트별 설정 (`.jcr.md`)

프로젝트 루트에 `.jcr.md` 파일을 두면 팀/프로젝트별 컨벤션을 적용할 수 있다:

```markdown
# 프로젝트 리뷰 설정

## base-branch
- main

## 프로젝트 정보
- Next.js 15 (App Router) + TypeScript (strict 모드)
- Tailwind CSS + shadcn/ui
- pnpm workspace monorepo

## 네이밍 컨벤션
- 변수/함수: camelCase
- 컴포넌트: PascalCase
- 상수: UPPER_SNAKE_CASE

## 무시할 패턴
- `src/components/ui/` 하위 파일 (shadcn 자동 생성)
- `src/generated/` 하위 파일
- `*.test.ts`의 매직 넘버

## 스타일 컨벤션
- 따옴표: 큰따옴표
- 세미콜론: 사용
- import 순서: 외부 패키지 → @/ 경로 → 상대 경로

## 추가 룰
- 모든 public 함수에 JSDoc 필수
```

## 출력 형식

리뷰 결과는 다음 형식으로 출력한다:

```
## 리뷰 요약

- 총 7건 (error: 2, warning: 3, info: 2)
- 주요 영역: 네이밍(3), 불필요한 코드(2), 에러 처리(2)

## 리뷰 결과

### 불필요한 코드
- [error] `src/utils.ts:42` - `tempVar`가 선언 후 사용되지 않음
  → 제거 권장

### 네이밍
- [warning] `src/api/handler.ts:15` - `d`는 의미가 불명확
  → `duration` 또는 `delay`로 변경 권장

### 중복
- [info] `src/components/Card.tsx:30-45`와 `src/components/Panel.tsx:20-35`
  → 공통 레이아웃 컴포넌트로 추출 가능
```

이슈가 없는 경우:

```
## 리뷰 요약

리뷰 대상 코드에서 특별한 품질 이슈가 발견되지 않았습니다.
```

출력 언어는 사용자의 언어를 따른다.

## 로드맵

### Phase 1 - 핵심 스킬 + 전체 references

- [x] `jcr-review/SKILL.md` 작성 (frontmatter + 메타 룰 + references 참조 가이드)
- [x] references 파일 전체 작성:
  - [x] dead-code.md
  - [x] comments.md
  - [x] naming.md
  - [x] duplication.md
  - [x] complexity.md
  - [x] magic-values.md
  - [x] error-handling.md
  - [x] style.md
- [x] `jcr-refactor/SKILL.md` 작성
- [x] 테스트 프롬프트로 동작 검증 (just-apps-homepage 대상, 피드백 반영 완료)

### Phase 2 - 실전 적용 및 보완

- [x] 실제 프로젝트에 적용하며 룰 보완 (just-apps-homepage 대상, 8개 reference 전체 보강)
- [x] 안티패턴 사례 축적 (실전 사례 기반 체크리스트/안티패턴 추가)
- [x] `.jcr.md` 설정 파일 지원 검증 (공식 섹션 목록 정의, 탐색 위치 명시, 리뷰 절차 순서 개선)
- [x] description 최적화 (jcr-refactor: "코드 개선" → "코드 구조 개선"으로 트리거 충돌 방지)

### Phase 3 - 배포

- [x] README.md 작성 (설치 방법, 사용법)
- [x] 설치 가이드 (직접 복사, 글로벌 설치, 심볼릭 링크 3가지 방법)
