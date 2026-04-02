# Naming — 네이밍 컨벤션

## 왜 중요한가

이름은 코드에서 가장 많이 읽히는 부분이다. 좋은 이름은 주석 없이도 코드의 의도를 전달하고, 나쁜 이름은 매번 코드를 다시 읽게 만든다. 네이밍은 코드 가독성의 80%를 결정한다.

## 원칙

- 이름은 그것이 "무엇인지" 또는 "무엇을 하는지"를 드러내야 한다.
- 축약하지 않는다. `usr` 대신 `user`, `btn` 대신 `button`.
- 스코프가 넓을수록 이름은 구체적이어야 한다. 루프 변수 `i`는 괜찮지만, 모듈 수준 변수 `d`는 안 된다.
- 일관성이 중요하다. 같은 개념에는 같은 단어를 쓴다. `fetch`/`get`/`retrieve`를 혼용하지 않는다.
- 언어/프레임워크의 컨벤션을 따른다.

## 체크리스트

- [ ] 한 글자 변수명 (루프 인덱스, 1-2줄 콜백의 단일 파라미터 제외. 단, 같은 파일에서 동일 문자가 다른 의미로 사용되면 지적)
- [ ] 의미 없는 이름 (`data`, `info`, `temp`, `result`, `obj`, `val` 단독 사용)
- [ ] 타입 정보를 이름에 포함 (`strName`, `arrItems` — 헝가리안 표기법)
- [ ] boolean 변수가 `is`, `has`, `can`, `should` 등으로 시작하지 않는 경우
- [ ] 함수명이 동사로 시작하지 않는 경우 (예: `userData()` → `fetchUserData()`)
- [ ] 부정형 boolean (`isNotValid` → `isInvalid` 또는 `isValid`의 반전)
- [ ] 같은 개념에 다른 단어 혼용 (`get`/`fetch`/`retrieve`, `remove`/`delete`/`destroy`)
- [ ] 약어가 보편적이지 않은 경우 (`cnt` → `count`, `mgr` → `manager`)
- [ ] 컨벤션 불일치 (같은 코드베이스에서 camelCase와 snake_case 혼용)

## 좋은 예 / 나쁜 예

나쁜 예:
```
const d = new Date();
const ul = getUsers();
const flag = check(ul);
```

좋은 예:
```
const createdAt = new Date();
const activeUsers = getActiveUsers();
const hasPermission = checkPermission(activeUsers);
```

나쁜 예:
```
function process(data) { ... }
```

좋은 예:
```
function validatePaymentRequest(paymentRequest) { ... }
```

## 안티패턴

- **숫자 접미사** — `user1`, `user2`, `data2`. 각각이 무엇인지 구분되는 이름을 붙인다.
- **과도한 축약** — 몇 글자 아끼려다 가독성을 잃는 것. IDE의 자동완성이 길이 문제를 해결해준다.
- **과도하게 긴 이름** — `getUserFromDatabaseByEmailAddress()`처럼 너무 긴 것도 문제다. 적절한 맥락이 있으면 `findByEmail()`로 충분하다.
- **오해를 유발하는 이름** — `list`라는 이름인데 실제로는 Map인 경우, `count`인데 boolean인 경우.
- **동사 없는 함수명** — `userData()`, `emailValidation()`. 함수는 행위이므로 `fetchUserData()`, `validateEmail()`로 동사를 앞에 둔다.
- **콜백 파라미터 혼동** — 같은 파일에서 `(e) =>`가 에러 콜백과 일반 콜백 양쪽에서 사용되는 경우. 짧은 이름이라도 파일 내에서 의미가 충돌하면 구체적 이름을 사용한다.
