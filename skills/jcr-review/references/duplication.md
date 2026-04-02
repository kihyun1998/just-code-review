# Duplication — 코드 중복 / 재사용

## 왜 중요한가

중복 코드는 변경 비용을 곱절로 늘린다. 한 곳을 수정하면 다른 곳도 수정해야 하고, 하나를 놓치면 버그가 된다. 하지만 모든 중복이 나쁜 것은 아니다. 섣부른 추상화보다 약간의 중복이 나은 경우도 있다.

## 원칙

- 동일한 코드가 3번 이상 반복되면 추출을 고려한다 (Rule of Three).
- 2번 반복은 아직 추출하지 않아도 된다. 패턴이 확립될 때까지 기다린다.
- 코드가 비슷해 보여도 변경 이유가 다르면 중복이 아니다 (우연의 중복).
- 추상화의 비용(복잡도 증가)이 중복의 비용보다 크면 중복을 유지한다.

## 체크리스트

- [ ] 3회 이상 반복되는 동일한 코드 블록
- [ ] 복사-붙여넣기로 만든 유사한 함수 (파라미터만 다름)
- [ ] 여러 파일에 걸쳐 동일한 유틸리티 로직
- [ ] 같은 검증/변환 로직이 여러 곳에 존재
- [ ] 상수 문자열/설정값이 여러 곳에 하드코딩 (magic-values 참조)
- [ ] API route handler 전체에 걸쳐 동일한 보일러플레이트(인증, JSON 파싱, 에러 응답) 반복
- [ ] 공통 유틸리티 함수가 존재함에도 각 호출처에서 인라인 반복 (dead-code 참조)
- [ ] 에러 응답 파싱 패턴 등 클라이언트 측 동일 로직이 여러 컴포넌트에서 반복

## 좋은 예 / 나쁜 예

나쁜 예 — 동일 로직 반복:
```
// 파일 A
const fullName = `${user.firstName} ${user.lastName}`;
const initials = `${user.firstName[0]}${user.lastName[0]}`;

// 파일 B (동일 로직)
const displayName = `${member.firstName} ${member.lastName}`;
const memberInitials = `${member.firstName[0]}${member.lastName[0]}`;
```

좋은 예 — 공통 함수 추출:
```
function formatFullName(person) {
  return `${person.firstName} ${person.lastName}`;
}

function getInitials(person) {
  return `${person.firstName[0]}${person.lastName[0]}`;
}
```

## 안티패턴

- **섣부른 추상화 (Premature Abstraction)** — 2번 나왔다고 바로 추출하는 것. 패턴이 확립되기 전에 추상화하면 잘못된 경계로 나뉠 수 있다.
- **과도한 DRY** — 모든 중복을 제거하려다 코드가 오히려 이해하기 어려워지는 것. 읽는 사람이 여러 파일을 넘나들어야 하는 추상화는 중복보다 나쁘다.
- **우연의 중복 제거** — 현재 코드가 비슷해 보여도 서로 다른 이유로 변경되는 코드를 억지로 합치는 것. 나중에 분리할 때 더 큰 비용이 든다.
- **상속으로 중복 제거** — 중복 제거만을 위해 상속 계층을 만드는 것. 구성(composition)이 대부분의 경우 더 적절하다.
- **유틸리티 무시 인라인 반복 (Utility Bypass)** — 공통 함수가 이미 존재하는데 각 호출처에서 인라인으로 동일 로직을 반복 작성하는 것. 유틸리티와 인라인 버전의 동작이 미묘하게 달라져 버그의 원인이 된다.
- **보일러플레이트 복사-붙여넣기** — API route handler의 인증/파싱/에러 처리 패턴을 매번 복사하는 것. 미들웨어나 wrapper 함수로 추출을 검토한다.
