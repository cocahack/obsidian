# EnabledIf, DisabledIf

테스트를 어떤 조건에 따라 실행하거나 그렇지 않게 만들고 싶다면 `@EnabledIf`또는 `@DisabldIf` 애노테이션을 사용해볼 수 있다.

## 배경

테스트를 특정 프로필에서만 실행하도록 구현하고 싶어 알아보던 도중 `@IfProfileValue` 라는 애노테이션을 알게 되었다. 그러나 이 애노테이션은 JUnit4 에서만 사용할 수 있었다.

JUnit5을 사용하는 스프링 테스트 환경에서는 `@EnabledIf`또는 `@DisabldIf`을 사용하도록 변경되었다. 

> [!warning]
> JUnit5 에도  `@EnabledIf`와 `@DisabldIf`를 이름으로 하는 애노테이션이 생겼기 때문에, 스프링의 기능을 이용하고 싶다면 패키지명을 잘 확인하고 import 해야 한다.

## 사용법

이 애노테이션들은 `expression`과 `reason`, 그리고 `loadContext` 속성을 가지고 있다.

가장 중요한 속성은 `expression`이다. 조건을 SpEL, environment template, 텍스트 중 하나의 방식을 택해 사용할 수 있으며, 평가식은 `true` 또는 `false` 로 평가할 수 있어야 한다.

### Profile을 조건으로 사용하기

프로필을 조건으로 걸고 싶다면 아래와 같이 할 수 있다.

```java
@EnabledIf(expression = "#{${spring.active.profiles} == 'test'", loadContext = true)
```

`application.properties` 파일에 정의한 프로퍼티를 가져오려면 `loadContext` 속성을 `true`로 할 수 밖에 없다. 그러면 일단 모든 컨텍스트를 로드한 다음, 조건식을 확인하게 된다.

### 대안

위의 예시처럼 컨텍스트를 가져오고 싶지 않다면 다른 애노테이션을 사용해볼 수도 있다.

- `@EnabledIfEnvironmentVariable`
- `@EnabledIfSystemProperty`

각각 환경 변수와 시스템 프로퍼티를 이용하여 조건을 걸 수 있다.

```java
@EnabledIfEnvironmentVariable(named = "INTEGRATION_TESTS_ENABLED", matches = "true")
```

## References

- [junit-5-conditional-test-execution](https://www.baeldung.com/junit-5-conditional-test-execution)