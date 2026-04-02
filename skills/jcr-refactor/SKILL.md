---
name: jcr-refactor
description: "코드 리팩토링 제안. 사용자가 리팩토링, 코드 구조 개선, 코드 정리를 요청할 때 사용. 실제 변경 코드를 포함한 구체적인 리팩토링 방안을 제시."
---

# JCR Refactor — 리팩토링 제안

코드 품질 리뷰(jcr-review)가 "문제를 찾아서 알려주는 것"이라면, 이 스킬은 "실제로 어떻게 고칠지 코드를 제안하는 것"이다.

## 리팩토링 대상 결정

인자가 없으면 `git diff`(unstaged 변경사항)를 기본 대상으로 한다.

인자가 있으면 다음 규칙으로 판단한다:
- `--staged` → `git diff --staged`
- `--branch` → 기본 브랜치 대비 diff (`git diff <base-branch>...HEAD`). 기본 브랜치는 `git symbolic-ref refs/remotes/origin/HEAD`로 자동 감지한다. `.jcr.md`에 `base-branch`가 지정되어 있으면 그것을 우선 사용한다.
- 파일/폴더 경로 → 해당 경로의 코드를 직접 읽음

## References

jcr-review의 references를 공유한다. 리팩토링 판단 기준으로 참조한다:

- `../jcr-review/references/dead-code.md` — 제거할 코드 식별
- `../jcr-review/references/duplication.md` — 추출/통합 대상 식별
- `../jcr-review/references/complexity.md` — 분리/단순화 대상 식별
- `../jcr-review/references/naming.md` — 이름 개선 제안
- `../jcr-review/references/error-handling.md` — 에러 처리 개선
- `../jcr-review/references/magic-values.md` — 상수 추출 제안
- `../jcr-review/references/comments.md` — 주석 정리
- `../jcr-review/references/style.md` — 스타일 통일

## 리팩토링 절차

1. `.jcr.md`가 있으면 먼저 읽어서 프로젝트 컨벤션과 제외 패턴을 확인한다.
2. 대상 코드를 읽고 전체 맥락을 파악한다.
3. references를 참조하여 개선 가능한 부분을 식별한다.
4. 각 리팩토링에 대해 before/after 코드를 제시한다.
5. 리팩토링의 이유와 효과를 설명한다.

## 리팩토링 유형

- **함수 추출** — 긴 함수에서 독립적인 로직을 별도 함수로 분리
- **변수/상수 추출** — 매직 값이나 복잡한 표현식에 이름 부여
- **조건 단순화** — 복잡한 조건문을 guard clause, early return 등으로 평탄화
- **중복 제거** — 반복되는 코드를 공통 함수/모듈로 통합
- **네이밍 개선** — 더 명확한 이름으로 변경
- **에러 처리 개선** — 빈 catch, 에러 삼킴 등을 적절한 처리로 변경
- **죽은 코드 제거** — 사용되지 않는 변수, import, 함수 정리

## 출력 형식

반드시 다음 형식을 따른다:

```
## 리팩토링 요약

- 총 N건의 리팩토링 제안
- 주요 유형: 유형(건수), ...

## 리팩토링 제안

### 1. [제목] — `파일경로:라인범위`

**이유**: 왜 이 리팩토링이 필요한지

**Before**:
(현재 코드)

**After**:
(개선된 코드)

**효과**: 리팩토링으로 얻는 이점
```

출력 언어는 사용자의 언어를 따른다.

## 주의사항

- 동작을 변경하지 않는 리팩토링만 제안한다. 기능 추가/수정은 이 스킬의 범위가 아니다.
- 한 번에 최대 5~7건까지 제안한다. 우선순위가 높은 것부터 제시하고, 추가 개선 가능 사항이 있으면 "N건 추가 가능"으로 안내한다.
- 테스트가 있는 코드는 테스트가 깨지지 않는 범위에서 제안한다.
- 프로젝트 컨벤션(`.jcr.md`)을 존중한다.
