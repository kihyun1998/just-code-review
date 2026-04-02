# Complexity — 함수 복잡도 / 책임 분리

## 왜 중요한가

복잡한 함수는 읽기 어렵고, 테스트하기 어렵고, 버그가 숨기 좋은 곳이다. 함수가 여러 일을 하면 변경 시 의도하지 않은 부작용이 생긴다. 단순한 함수 여러 개가 복잡한 함수 하나보다 낫다.

## 원칙

- 함수는 한 가지 일만 한다 (Single Responsibility).
- 함수의 추상화 수준을 일관되게 유지한다. 고수준 로직과 저수준 디테일을 섞지 않는다.
- 중첩(nesting)이 깊어지면 함수를 분리하거나 early return을 사용한다.
- 파라미터가 많으면 객체로 묶거나 함수를 분리한다.

## 체크리스트

- [ ] 함수 길이가 과도한 경우 (언어에 따라 다르지만, 40줄 이상이면 검토)
- [ ] 중첩 깊이가 3단계 이상 (if 안에 for 안에 if)
- [ ] 하나의 함수에서 여러 독립적인 작업 수행
- [ ] 파라미터가 4개 이상인 함수
- [ ] boolean 파라미터로 함수 동작 분기 (flag argument)
- [ ] 조건문이 복잡한 경우 (3개 이상의 조건 조합)
- [ ] 한 파일/클래스가 너무 많은 책임을 가진 경우
- [ ] 중첩 try-catch (try 안에 try)로 보상/롤백 로직이 구현된 경우
- [ ] 하나의 API handler가 인증, 검증, 비즈니스 로직, 부수 효과를 모두 처리하는 경우

## 좋은 예 / 나쁜 예

나쁜 예 — 여러 책임이 섞인 함수:
```
function processOrder(order) {
  // 검증
  if (!order.items || order.items.length === 0) {
    throw new Error("Empty order");
  }
  if (!order.address) {
    throw new Error("No address");
  }

  // 가격 계산
  let total = 0;
  for (const item of order.items) {
    let price = item.price * item.quantity;
    if (item.discount) {
      price = price * (1 - item.discount);
    }
    total += price;
  }

  // 저장
  db.save({ ...order, total });

  // 이메일 발송
  sendEmail(order.email, `주문 완료: ${total}원`);
}
```

좋은 예 — 책임 분리:
```
function processOrder(order) {
  validateOrder(order);
  const total = calculateTotal(order.items);
  db.save({ ...order, total });
  notifyOrderComplete(order.email, total);
}
```

나쁜 예 — 깊은 중첩:
```
function findEligible(users) {
  const result = [];
  for (const user of users) {
    if (user.active) {
      if (user.age >= 18) {
        if (user.verified) {
          result.push(user);
        }
      }
    }
  }
  return result;
}
```

좋은 예 — early return / 필터링:
```
function findEligible(users) {
  return users.filter(user =>
    user.active && user.age >= 18 && user.verified
  );
}
```

## 안티패턴

- **God Function** — 수백 줄짜리 함수 하나가 모든 것을 처리. 분리할 경계를 찾아 나눈다.
- **Flag Argument** — `process(data, true, false)`. boolean 파라미터로 분기하는 대신 별도 함수로 분리한다.
- **Arrow Anti-Pattern** — 중첩된 조건/루프가 화살표 모양(→)을 만드는 것. guard clause와 early return으로 평탄화한다.
- **과도한 분리** — 2줄짜리 로직을 별도 함수로 추출하는 것도 문제다. 추상화 비용이 인라인 코드보다 크면 분리하지 않는다.
- **중첩 try-catch 보상 패턴 (Nested Try-Catch Compensation)** — 비즈니스 로직 실패 시 보상(rollback)을 위해 try 안에 또 다른 try를 넣는 구조. 트랜잭션 레벨 지원이나 별도 함수 분리가 더 적절하다.
