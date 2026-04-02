# Magic Values — 매직 넘버 / 하드코딩

## 왜 중요한가

코드에 `if (status === 3)` 같은 숫자가 나오면 아무도 3이 뭘 의미하는지 모른다. 매직 값은 코드의 의도를 숨기고, 값을 변경해야 할 때 모든 출현 위치를 찾아야 하는 문제를 만든다.

## 원칙

- 의미를 가진 값에는 이름을 붙인다 (상수, enum).
- 0, 1, -1, 빈 문자열 등 자명한 값은 매직 값으로 보지 않는다.
- 같은 값이 여러 곳에서 사용되면 반드시 상수로 추출한다.
- 설정값(timeout, retry count, URL 등)은 설정 파일이나 환경변수로 분리한다.

## 체크리스트

- [ ] 의미가 불명확한 숫자 리터럴 (예: `if (retries > 3)`, `setTimeout(fn, 86400000)`)
- [ ] 여러 곳에서 반복되는 같은 문자열 리터럴
- [ ] 하드코딩된 URL, 파일 경로, API 엔드포인트
- [ ] 하드코딩된 설정값 (timeout, 페이지 크기, 최대 재시도 횟수)
- [ ] 비트 플래그나 상태 코드를 숫자 그대로 사용
- [ ] DB 에러 코드를 문자열 리터럴로 직접 비교 (예: PostgreSQL `"23505"`)
- [ ] 시간 계산에서 `24 * 60 * 60 * 1000` 같은 변환 상수를 인라인으로 사용

## 좋은 예 / 나쁜 예

나쁜 예:
```
if (user.role === 2) {
  setTimeout(fetchData, 30000);
}
```

좋은 예:
```
const ROLE_ADMIN = 2;
const FETCH_INTERVAL_MS = 30_000;

if (user.role === ROLE_ADMIN) {
  setTimeout(fetchData, FETCH_INTERVAL_MS);
}
```

나쁜 예:
```
const response = await fetch("https://api.example.com/v2/users");
```

좋은 예:
```
const API_BASE_URL = process.env.API_BASE_URL;
const response = await fetch(`${API_BASE_URL}/users`);
```

## 안티패턴

- **모든 숫자를 상수화** — `const ONE = 1` 같은 것은 의미가 없다. 자명한 값은 그대로 써도 된다.
- **상수 이름에 값 반복** — `const THIRTY = 30` 대신 `const MAX_RETRIES = 30`처럼 의미를 담는다.
- **enum 대신 숫자 상수 나열** — 관련된 상수들이 여러 개면 enum이나 객체로 그룹화한다.
- **문자열을 enum처럼 사용** — `status === "active"` 같은 문자열 비교가 여러 곳에 있으면 상수/enum으로 정의한다.
- **DB 에러 코드 매직 스트링** — `code === "23505"` 같은 DB 에러 코드를 상수 없이 직접 비교하는 것. `const PG_UNIQUE_VIOLATION = "23505"` 처럼 의미를 담은 상수로 정의한다.
- **시간 변환 매직 넘버** — `days * 24 * 60 * 60 * 1000` 같은 밀리초 변환을 인라인으로 쓰는 것. `const MS_PER_DAY = 86_400_000` 같은 상수나 유틸리티를 사용한다.
