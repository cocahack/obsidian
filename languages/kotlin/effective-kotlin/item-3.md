# Item 3: 가능한 플랫폼 타입은 일찍 제거하라

## 요약

다른 언어에서 온 타입와 Nullability를 알 수 없는 타입은 플랫폼 타입으로 알려져 있다. 이것들은 위험하기 때문에, 가능하면 빨리 제거하여 위험이 전파되지 않도록 해야 한다. 
또한, 외부에 노출된 생성자와 메소드, 필드 등에 Nullability를 명시할 수 있는 어노테이션을 사용하여 타입을 명시하라.

## 개요

코틀린이 제공하는 null-safety 기능은 자바의 악명높은 NPE를 막을 수 있는 놀라운 기능이다. 그러나 자바나 C 등으로 작성된 코드를 그대로 사용하면 이를 보장할 수 없다. 예를 들어, `String` 타입을 반환하는 자바 메소드를 사용한다면, 이 메소드가 `null` 을 반환하지 않는지 알 수 없다.

자바의 제네릭 타입을 사용한다면 문제는 더 복잡해진다. 어노테이션으로 어떠한 표시도 하지 않은 자바 API가 `List<User>` 타입을 반환한다고 가정해보자. 코틀린이 기본적으로 nullable 타입으로 가정한다면, 리스트와 리스트가 담고 있는 유저가 `null` 이 아님을 알기 위해 리스트와 필터를 사용해서 검증해야 한다.

```java
// Java
public class UserRepo {
	public List<User> getUsers() {
		// ...
	}
}
```

```kotlin
val users: List<User> = UserRepo().users!!.filterNotNull()
```

이런 문제 때문에, 코틀린은 기본적으로 nullable로 다루는 대신에 자바로부터 온 타입과 nullability를 알 수 없는 타입을 *platform type*이라는 특수한 타입으로 본다.

플랫폼 타입은 타입 이름 뒤에 `!`를 붙여 표기한다. 이 표기법은 코드에 직접 사용할 수 없다. 플랫폼 값이 코틀린 변수나 속성에 할당되면 타입 추론은 할 수 있지만 명시적으로 설정할 수는 없다. 대신 null이거나 null이 아닌 타입을 선택할 수 있다.

```kotlin
val repo = UserRepo()
val user1 = repo.user // Type: User!
val user2: User = repo.user // Type: User
val user3: User? = repo.user // Type: User?
```

이렇게 타입을 다룰 수 있어 앞서 봤었던 문제는 발생하지 않는다. 

```kotlin
val users: List<User> = UserRepo().users
val users: List<List<User>> = UserRepo().groupedUsers
```

**그러나 not-null 로 가정한 것이 실제로는 null일 수 있는 위험성은 여전히 남아있다.** 이 것을 조심해야 하는 것이다. 

자바로 작성한 코드는 애노테이션을 사용해 null인지 아닌지를 나타내주자.

```java
// java
import org.jetbrains.annotations.NotNull;

public class UserRepo {
	public @NotNull User getUser() {
		// ...
	}
}
```

> [!NOTE]
> Jetbrains, Android, JSR-305 등 여러 프로젝트에서 애노테이션을 지원한다.

그리고 플랫폼 타입은 가능한 빨리 타입을 확정하도록 하자.


