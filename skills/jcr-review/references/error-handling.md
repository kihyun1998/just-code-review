# Error Handling — 에러 처리 패턴

## 왜 중요한가

잘못된 에러 처리는 디버깅을 불가능하게 만든다. 빈 catch 블록은 에러를 삼켜서 문제의 원인을 감추고, 과도한 try-catch는 정상적인 코드 흐름을 가린다. 에러 처리는 프로그램의 안정성과 유지보수성을 직접적으로 결정한다.

## 원칙

- 에러를 삼키지 않는다. catch했으면 로그를 남기거나, 상위로 전파하거나, 의미 있는 처리를 한다.
- 가능한 좁은 범위를 try-catch로 감싼다.
- 에러 메시지는 "무엇이 왜 실패했는지"를 포함한다.
- 예외적 상황에만 예외를 사용한다. 정상적인 흐름 제어에 예외를 쓰지 않는다.

## 체크리스트

- [ ] 빈 catch 블록 (에러를 삼키는 것)
- [ ] catch에서 에러를 무시하고 null/undefined/기본값만 반환
- [ ] 너무 넓은 예외 포착 (최상위 `catch(Exception e)` 또는 `catch(e)`)
- [ ] 에러 메시지가 없거나 불명확한 경우 (`throw new Error("error")`)
- [ ] 전체 함수를 감싸는 과도한 try-catch
- [ ] catch에서 원본 에러 정보를 잃어버리는 경우
- [ ] 정상적인 흐름 제어에 예외를 사용하는 경우
- [ ] async/await에서 에러 처리 누락 (unhandled promise rejection)
- [ ] 비동기 IIFE `(async () => { ... })()` 에 `.catch()` 없이 사용 (unhandled rejection 위험)
- [ ] 부수 효과(로깅, 알림 등)의 실패가 메인 트랜잭션까지 실패시키는 구조

## 좋은 예 / 나쁜 예

나쁜 예 — 에러 삼킴:
```
try {
  await saveUser(user);
} catch (e) {
  // ignore
}
```

좋은 예 — 적절한 처리:
```
try {
  await saveUser(user);
} catch (error) {
  logger.error("Failed to save user", { userId: user.id, error });
  throw new UserSaveError(`사용자 저장 실패: ${user.id}`, { cause: error });
}
```

나쁜 예 — 너무 넓은 try-catch:
```
try {
  const config = readConfig();
  const data = fetchData(config.url);
  const result = processData(data);
  await saveResult(result);
  sendNotification(result);
} catch (e) {
  console.log("something went wrong");
}
```

좋은 예 — 필요한 곳만 감싸기:
```
const config = readConfig();

let data;
try {
  data = await fetchData(config.url);
} catch (error) {
  throw new DataFetchError(`데이터 조회 실패: ${config.url}`, { cause: error });
}

const result = processData(data);
await saveResult(result);
```

## 안티패턴

- **Pokemon Exception Handling** — "Gotta catch 'em all". 모든 예외를 최상위에서 하나의 catch로 잡는 것. 에러 유형별로 적절히 처리해야 한다.
- **에러 삼킴 (Swallowing)** — catch 블록에서 아무것도 하지 않는 것. 의도적으로 무시해야 한다면 주석으로 이유를 명시한다 (예: `catch { /* JSON 파싱 실패 시 기본 메시지 사용 */ }`). 주석이 있는 의도적 빈 catch는 허용하되, 주석 없는 빈 catch는 error로 취급한다.
- **에러 변환 시 원인 손실** — 새 에러를 던지면서 원본 에러를 `cause`에 포함하지 않는 것.
- **흐름 제어용 예외** — `try { findUser() } catch { createUser() }` 같은 패턴. 조건 검사로 대체한다.
- **console.log만 하는 catch** — 로그만 찍고 에러를 전파하지 않으면 상위 호출자는 성공으로 인식한다.
- **부수 효과 미보호 (Unprotected Side Effect)** — 감사 로그, 알림 발송 등 부수 효과를 메인 try-catch 안에서 await하면, 부수 효과 실패가 전체 요청을 500으로 만든다. 비즈니스 로직은 이미 성공했는데 클라이언트는 실패로 인식하게 된다. 부수 효과는 별도 try-catch로 감싸거나 fire-and-forget으로 처리한다.
