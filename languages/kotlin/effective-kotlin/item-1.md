# 이펙티브 코틀린 - Item 1: 가변성을 제한하라

## 요약

- `var` 보다 `val` 사용
- 가변 프로퍼티보다 불변 프로퍼티
- 오브젝트와 클래스를 불변으로 선언하자
- 변경이 필요하다면 불변 data 클래스와 `copy` 를 활용하자.
- 상태를 유지해야 한다면 읽기 전용 컬렉션을 사용하자.
- Mutation point를 현명하게 설계하고 불필요하게 만들지 말라.
- 가변 오브젝트를 노출하지 마라.

## 개요

클래스가 상태를 가지게 하는 것은 양날의 검이다. 요소가 변경되는 것을 잘 나타내어 주지만, 상태를 관리하기가 매우 까다롭다. 상태 관리의 어려운 점은 다음과 같다.

1. 프로그램의 많은 mutating point를 이해하고 디버깅하기가 더 어려워진다.
2. 코드를 이해하고 추론하는 것도 어려워진다.
3. 멀티쓰레드 프로그램에서의 동기화를 지원하기가 힘들다.
4. 테스트하기 힘들다. 테스트할 지점이 더 많아지기 때문이다.
5. 상태가 바뀌면 다른 클래스에게 이를 알려야할 수도 있다.

코틀린에는 이런 가변성을 잘 제한하는 방법을 제공하고 있다.

## 가변성 제한하기

여러 기능과 특징 중, 가장 중요한 것이라고 할 수 있는 것들은 다음과 같다.

- 읽기 전용 프로퍼티 `val`
- 쓰기, 읽기 전용 컬렉션의 분리
- 데이터 클래스의 `copy`

### `val`

`val`로 선언된 프로퍼티는 한 번 할당되면 값을 변경할 수 없다.

```kotlin
val a = 10  
a = 20//ERROR
```

그러나 반드시 불변하거나 final일 필요는 없다. 

또, `val`은 가변적인 오브젝트를 가지고 있을 수 있다.

```kotlin
val list = mutableListOf(1,2,3)
list.add(4)

print(list)//[1,2,3,4]
```

읽기 전용 프로퍼티는 커스텀 getter를 가질 수도 있다.

```kotlin
var name: String = "Marcin"
var surname: String = "Moskała"
val fullName
	get() = "$name $surname"

fun main(){  
	println(fullName) // Marcin Moskała
	name = "Maja"  
	println(fullName) // Maja Moskała
}
```

**`val` 은 mutation point를 제공하지 않는 것이 핵심이다.** 따라서 외부에서는 getter만 쓸 수 있다. 이런 특성에 따라 `val` 프로퍼티를 `var` 로 오버라이딩하는 것도 가능하다.

```kotlin
interface Element {
	val active: Boolean
}

class ActualElement : Element {
	override var active: Boolean = false
}

```

실제 java 코드는 다음과 같다.

```java
public interface Element {  
   boolean getActual();  
} 

// ActualElement.java  
public final class ActualElement implements Element {  
   private boolean actual;  
  
   public boolean getActual() {  
      return this.actual;  
   }  
  
   public void setActual(boolean var1) {  
      this.actual = var1;  
   }  
}
```

그러나 `val` 이 불변성을 제공하는 도구라고 해석해서는 절대 안된다.

### 가변 컬렉션과 읽기 전용 컬렉션의 분리

![[kotlin-collections.png]]

`Iterable`, `Collection` 등의 인터페이스가 읽기 전용과 그렇지 않은 것으로 분리되어 있으며, mutable한 인터페이스가 read-only 인터페이스를 상속하고 있는 구조이다. 이는 프로퍼티가 동작하는 방식과 유사하다.

Read-only 컬렉션은 반드시 불변하지 않다. 거의 대부분 가변적이겠지만, read-only 인터페이스에 가려져서 수정이 불가능한 것이다.

#### Data 클래스의 Copy

가변적인 오브젝트는 예상치 못한 곳에서 그 상태가 변경될 수 있다. 반변 불변 오브젝트는 데이터를 변경할 필요가 있을 때 문제가 된다. 이런 문제를 해결하기 위해 불변 오브젝트는 변경된 상태가 반영된 새로운 오브젝트를 반환한다.

```kotlin
class User(
	val name: String,
	val surname: String,
) {
	fun withSurname(surname: String) = User(name, surname)
}

var user = User("A", "B")
user = user.withSurname("C")
print(user) // User(name = "A", surname = "C")
```

그러나 위와 같은 메소드를 작성할 때, 많은 프로퍼티 중 하나만 변경해야 한다면 매우 귀찮은 작업이 될 것이다. 이를 위해 `copy` 를 사용할 수 있다.

```kotlin
// 위 코드와 동일


var user = User("A", "B")
user = user.copy(surname = "C")
print(user) // User(name = "A", surname = "C")

```

### 변경될 수 있는 부분을 노출하지 마라

```kotlin
data class User(val name: String)

class UserRepository {
	private val storedUsers: MutableMap<Int, String> = 
		mutableMapOf()
	
	fun loadAll(): MutableMap<Int, String> {
		return storedUsers
	}
}
```

위 코드를 보면 `UserRepository` 내부에서 사용하는 상태가 외부로 노출되는 것을 볼 수 있다. 가변 컬렉션으로 반환하고 있기 때문에 캡슐화가 깨지는 것이다. 

외부에 노출할 때는 read-only 컬렉션 등으로 노출할 수 있도록 습관화하자.

  

