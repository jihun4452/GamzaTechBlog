# JUnit 단위 테스트 핵심 정리

![](https://gamzatech-bucket.s3.ap-northeast-2.amazonaws.com/7728f4ca-4608-4ba2-b90c-231486748891_e1e73a2d-2a42-4e33-acf3-4e95d34688ae_image.png)


## 1. 스택 트레이스  
- 자바에서 오류가 발생하면 스택 트레이스로 원인을 파악한다.  
- 인수가 하나 많거나 적어도 한 끗 차이의 스택 트레이스로 표시된다.

## 2. 회귀 테스트 vs. 단위 테스트  
- **단위 테스트**: 메서드나 클래스 단위로 고립된 실행  
- **회귀 테스트**: 누적된 단위 테스트를 모두 실행해 변경점이 기존 기능을 망가뜨리지 않는지 검증  

## 3. 테스트 구조 (AAA 패턴)  
1. **Arrange(준비)**  
   - 테스트 대상 객체 생성  
   - 필요한 상태·데이터 초기화  
2. **Act(실행)**  
   - 단일 메서드 호출로 테스트 동작 수행  
3. **Assert(단언)**  
   - 반환값·상태·사이드 이펙트를 검증  
- Given–When–Then 형식으로도 표현한다.

## 4. 테스트 독립성 유지  
- 각 테스트는 고유한 맥락을 가져야 하며, **순서에 의존하면 안 된다.**  
- `static` 필드를 사용하면 테스트 간 상태가 공유될 수 있으므로 주의한다.

## 5. @Before / @After  
- `@Before` (JUnit 4) 또는 `@BeforeEach` (JUnit 5)로 **공통 초기화** 수행  
- 여러 개의 `@Before`는 순서가 보장되지 않으므로, 가급적 하나로 합쳐 순서대로 초기화한다.  
- `@After` / `@AfterEach`는 리소스 해제용으로 드물게 사용하며, **테스트 실패 시에도 실행**된다.

## 6. 단언 (Assertions)  
- 기본 단언 메서드  
  - `assertTrue(condition)` / `assertFalse(condition)`  
  - `assertEquals(expected, actual)`  
  - `fail("메시지")`: 의도적으로 실패 지점을 표시  
- **Hamcrest 매처** 활용 (CoreMatchers)  
  ```java
  assertThat(actual, is(expected));   // is()는 가독성 장식자
  assertThat(list, hasSize(3));
  ```
## 7. 예외 테스트
- **구버전(JUnit 4)**
  ```java
  @Test(expected = IllegalArgumentException.class)
  public void 테스트명() {
      // 예외가 발생해야 성공
  }
  
  Throwable ex = assertThrows(
    IllegalArgumentException.class,
    () -> service.doSomething(null)
);
assertThat(ex.getMessage(), containsString("…"));

## 8. TDD와 테스트 작성 원칙
- 빠르고 고립적이며 반복 가능해야 한다.  
- 프로덕션 코드와 완전히 분리된 테스트 모듈로 관리한다.  
- 공개 인터페이스만 호출해 검증하고, 내부 구현에 과도하게 의존하지 않는다.  
- 단일 책임 원칙(SRP) 준수: 하나의 테스트는 하나의 동작만 검증한다.  

## 9. 테스트 네이밍 컨벤션
- `Given_조건_When_동작_Then_결과` 형식 권장  
- 테스트 메서드 이름만 보고 **무엇을 검증하는지 한눈에** 알 수 있어야 한다.  
- 예: `givenNullUser_whenJoin_thenThrowsException`  

## 10. 유지보수와 최적화
- 의존성을 최소화해 실행 속도 개선  
- 테스트가 너무 길어지면 **작게 분리**  
- 실패 시 원인 파악이 쉽도록 **명확한 단언 메시지** 작성  
- 변경이 발생하면 테스트도 함께 **리팩토링**  

## 11. Mockito를 활용한 외부 의존성 분리
- `@Mock` / `@InjectMocks` + `@RunWith(MockitoJUnitRunner.class)`로 서비스·레포지토리 의존성 모킹  
- DB나 네트워크 호출 등 외부 리소스를 격리해 **진정한 단위 테스트** 구현  

## 12. 파라미터화 테스트 (JUnit 5)
- `@ParameterizedTest` + `@ValueSource`, `@CsvSource` 등으로 **여러 케이스**를 한 번에 검증  
- 중복된 테스트 코드를 줄이고 **가독성 향상**  

---

**좋은 테스트란**  
- 다른 테스트에 의존하지 않고, 순서와 상관없이 실행 가능  
- 적절한 범위로 쪼개져 있어 실패 시 문제 지점을 쉽게 파악  
- 지속적으로 유지·관리되며, 테스트 자체가 신뢰할 수 있는 문서 역할  
