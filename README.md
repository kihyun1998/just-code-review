# Just Code Review (JCR)

Claude Code용 코드 품질 리뷰 스킬. 보안이나 버그 탐지가 아닌, **"이 코드를 더 깔끔하게 만들 수 있는가"**에 집중한다.

## 스킬

### `/jcr-review` — 코드 품질 리뷰

코드의 품질 이슈를 찾아서 심각도별로 보고한다.

```
/jcr-review                    # git diff (unstaged 변경사항)
/jcr-review --staged           # staged 변경사항
/jcr-review --branch           # 브랜치 전체 변경사항
/jcr-review src/utils.ts       # 특정 파일
/jcr-review src/components/    # 특정 폴더
```

리뷰 관점:
- 불필요한 코드 (미사용 변수, import, 죽은 코드)
- 주석 품질 (부정확, 불필요, 누락)
- 네이밍 (변수, 함수, 클래스)
- 코드 중복 / 재사용 가능 패턴
- 함수 복잡도 / 책임 분리
- 매직 넘버 / 하드코딩
- 에러 처리 패턴
- 코드 스타일 일관성

### `/jcr-refactor` — 리팩토링 제안

문제를 찾는 것에서 더 나아가, **실제 변경 코드를 포함한 리팩토링 방안**을 제시한다. before/after 코드와 함께 이유와 효과를 설명한다.

```
/jcr-refactor                  # git diff 대상
/jcr-refactor src/views/       # 특정 폴더 대상
```

## 설치

### 방법 1: 프로젝트에 직접 복사

```bash
# 프로젝트 루트에서
mkdir -p .claude/skills
cp -r path/to/just-code-review/skills/* .claude/skills/
```

### 방법 2: 글로벌 설치

```bash
mkdir -p ~/.claude/skills
cp -r path/to/just-code-review/skills/* ~/.claude/skills/
```

### 방법 3: git clone 후 심볼릭 링크

```bash
git clone https://github.com/<your-username>/just-code-review.git ~/.jcr

# 프로젝트별
mkdir -p .claude/skills
ln -s ~/.jcr/skills/jcr-review .claude/skills/jcr-review
ln -s ~/.jcr/skills/jcr-refactor .claude/skills/jcr-refactor

# 또는 글로벌
mkdir -p ~/.claude/skills
ln -s ~/.jcr/skills/jcr-review ~/.claude/skills/jcr-review
ln -s ~/.jcr/skills/jcr-refactor ~/.claude/skills/jcr-refactor
```

## 프로젝트별 설정 (`.jcr.md`)

프로젝트 루트에 `.jcr.md` 파일을 두면 팀/프로젝트별 컨벤션을 적용할 수 있다.

```markdown
# 프로젝트 리뷰 설정

## base-branch
- develop

## 네이밍 컨벤션
- 변수/함수: camelCase
- 컴포넌트: PascalCase
- 상수: UPPER_SNAKE_CASE

## 무시할 패턴
- `src/components/ui/` 하위 파일 (자동 생성)
- `src/generated/` 하위 파일

## 스타일 컨벤션
- 따옴표: 큰따옴표
- 세미콜론: 사용

## 추가 룰
- 모든 public 함수에 JSDoc 필수
```

`.jcr.md`에 명시된 룰은 기본 룰보다 우선 적용된다.

## 출력 예시

```
## 리뷰 요약

- 총 5건 (error: 1, warning: 3, info: 1)
- 주요 영역: 네이밍(2), 에러 처리(2), 중복(1)

## 리뷰 결과

### 에러 처리
- [error] `src/api/handler.ts:42` - 빈 catch 블록으로 에러를 삼키고 있음
  → catch에서 로그를 남기거나 상위로 전파할 것

### 네이밍
- [warning] `src/utils.ts:15` - `d`는 의미가 불명확
  → `duration` 또는 `delay`로 변경 권장
```

## 구조

```
skills/
├── jcr-review/
│   ├── SKILL.md                  # 리뷰 스킬 정의
│   └── references/
│       ├── dead-code.md          # 불필요한 코드
│       ├── comments.md           # 주석 품질
│       ├── naming.md             # 네이밍 컨벤션
│       ├── duplication.md        # 중복 / 재사용
│       ├── complexity.md         # 복잡도 / 책임 분리
│       ├── magic-values.md       # 매직 넘버 / 하드코딩
│       ├── error-handling.md     # 에러 처리 패턴
│       └── style.md              # 스타일 일관성
└── jcr-refactor/
    └── SKILL.md                  # 리팩토링 제안 스킬
```

## 라이선스

MIT
