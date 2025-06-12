# 주특기 심화 챕터
## Lv1,2,3
<details>
<summary>Lv1-1</summary>
✅ 문제 상황
회원가입 기능에서 다음과 같은 순서로 로직이 처리되고 있었습니다:

1. 클라이언트가 회원가입 요청을 보냅니다.

2. 서버는 사용자의 비밀번호를 먼저 암호화합니다. (passwordEncoder.encode(...))

3. 이후 사용자 이메일이 이미 존재하는지 DB에서 확인합니다.

4. 만약 이메일이 이미 존재한다면, 예외를 던지고 회원가입을 중단합니다.

❗ 문제의 핵심
이메일이 중복된 경우에도 비밀번호 암호화가 먼저 수행되므로, 성능적으로 불필요한 연산이 발생하게 됩니다.

passwordEncoder.encode()는 상대적으로 비용이 큰 연산입니다.

중복 이메일로 인해 회원가입이 실패할 경우, 이 연산은 결국 무의미한 처리가 됩니다.

따라서, 이메일 중복 여부를 먼저 검사하고, 통과한 경우에만 비밀번호를 암호화해야 합니다.
</details>

<details>
<summary>Lv1-2</summary>
날씨 데이터를 외부 API로부터 받아올 때, 기존 로직은 다음과 같은 구조를 가지고 있었습니다.

❗ 문제의 핵심
중첩된 if-else 구조로 인해 코드 가독성이 떨어지고,
조건 분기가 많아질수록 로직 파악이 어려워졌습니다.

특히, 실패 조건(예외 상황)이 명확할 경우에는 빠르게 종료(return/throw) 하는 방식이 더 직관적입니다.

✅ 리팩토링 방식: Guard Clause 패턴 적용
Guard Clause란, 예외 상황(예: null, 실패 응답 등)을 빠르게 처리하고 정상 로직은 들여쓰기 없이 이어가는 구조입니다.

</details>

<details>
<summary>Lv1-3</summary>
✅ 문제 상황: 비효율적인 회원가입 로직 흐름
기존 처리 순서

1. 클라이언트가 회원가입 요청을 전송한다.
2. 서버는 사용자의 비밀번호를 먼저 암호화한다. (`passwordEncoder.encode(...)`)
3. 이후, 이메일 중복 여부를 DB에서 확인한다.
4. 중복된 이메일이면 예외를 던지고 회원가입을 중단한다.
  
✅ 개선 포인트
- 이메일 중복 여부를 가장 먼저 검사한다.
- 중복되지 않을 경우에만 `passwordEncoder.encode(...)`를 수행한다.
- 이렇게 하면 불필요한 암호화 연산을 방지할 수 있어 성능상 이점이 생긴다.
</details>

<details>
<summary>Lv2</summary>
✅ 문제 상황

`TodoController`와 `TodoService`에서 전체 Todo 목록을 조회할 때,  
각 Todo와 연관된 `User` 정보(`todo.getUser().getName()`)도 함께 화면에 노출되어야 합니다.

그러나 `Todo` 엔티티는 `User`와 `@ManyToOne(fetch = FetchType.LAZY)`로 연관돼 있어,  
Todo 목록을 반복문으로 순회하면서 매번 `User` 조회 쿼리가 별도로 실행되고 있었습니다.

즉, Todo가 100개일 경우:

- `Todo` 전체 조회: 1번
- 각 `User` 조회: 100번  
  → 총 1 + N개의 쿼리 발생 (N+1 문제)

- JPA의 기본 로딩 전략이 LAZY이기 때문에, 연관 엔티티인 `User`는 실제로 접근할 때까지 쿼리를 보내지 않습니다.
- 하지만 화면에서 `Todo`와 `User` 정보를 함께 보여줘야 할 경우,  
  각 `Todo` 마다 개별적인 `User` 쿼리가 추가되어 쿼리 폭발 현상이 발생합니다.
- 이로 인해 성능 저하가 발생하고, 데이터베이스 접근 비용이 불필요하게 증가합니다.

✅ 개선 방식: @EntityGraph 사용
```@EntityGraph(attributePaths = {"user"})
Page<Todo> findAllByOrderByModifiedAtDesc(Pageable pageable);
```
</details>

<details>
<summary>Lv3-1</summary>
✅ 테스트 코드 리팩토링: 비밀번호 비교 순서 오류 수정

✅ 문제 상황

`PasswordEncoderTest.matches_메서드가_정상적으로_동작한다()` 테스트는  
**비밀번호 암호화 및 비교 기능이 정상 동작하는지 검증하는 목적**으로 작성된 테스트입니다.

기존 테스트의 흐름은 다음과 같습니다:

1. 평문 비밀번호를 암호화한다 (`encodedPassword = encode(raw)`).
2. `encodedPassword`와 `rawPassword`를 `matches()` 메서드로 비교한다.
3. 일치 여부가 `true`일 것을 기대한다.

하지만 테스트가 실패하며, 예상과 다른 결과를 반환하고 있었습니다.

✅ 수정된 테스트 코드
BCryptPasswordEncoder는 단방향 해시이므로,
암호화된 비밀번호를 복호화하지 않고 raw → hash 다시 암호화 후 비교합니다.

순서를 바르게 지정하면 matches()는 내부적으로 해시 비교를 수행하여 일치 여부를 올바르게 판단합니다.
</details>