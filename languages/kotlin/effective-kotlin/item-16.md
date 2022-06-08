# Item 16: 프로퍼티는 동작이 아니라 상태를 나타내야 한다.

## Prerequisition - Property

언뜻 보기에, 코틀린의 프로퍼티는 자바의 필드와 비슷해 보인다. 하지만 전혀 다른 개념이다.

둘 다 데이터를 저장하는 점만 같으며, 프로퍼티는 그 보다 더 많은 기능을 가지고 있다. 기본적으로, 사용자 정의 getter, setter를 가질 수 있다.

```kotlin

var name: String? = null
	get() = field?.toUpperCase()
	set(value) {
		if(!value.isNullOrBlank()) {
			field = value
		}
	}
```

> [!note]
> **field 식별자**
> 
> 프로퍼티의 데이터를 저장해주는 backing field에 대한 레퍼런스이다. getter, setter의 디폴트 구현에 사용되므로, 자동으로 만들어진다. `val` 프로퍼티는 만들어지지 않는다.

프로퍼티는 필드가 반드시 필요하지 않다. 오히려 개념적으로 접근자(`val`은 getter, `var`는 getter와 setter)를 나타낸다. 이런 점 때문에 인터페이스에서 프로퍼티를 정의할 수 있다. 인터페이스로 정의할 수 있으므로 오버라이드가 당연히 가능하고, 프로퍼티를 위임할 수도 있다.

```kotlin
// override Property
interface Person {
	val name: String
}

class PersonImpl(override val name: String) : Person


// delegate
val db: Database by lazy { connectToDb() }
```

## 본론

프로퍼티를 함수 대신 사용할 수도 있으나, 완전히 대체해서 사용하는 것은 좋지 않다. 판단하는 기준을 '이 프로퍼티를 함수로 정의할 경우, 접두사로 get 또는 set을 붙이는게 자연스러운가?'로 두면 어느정도 명확할 것이다.

프로퍼티가 아닌 함수로 구현해야 하는 구체적인 예시를 들면,

- 연산 비용이 높거나, 복잡도가 $O(1)$ 보다 큰 경우
- 비즈니스 로직을 포함하는 경우
- 함수의 결과가 non-deterministic 인 경우
- 변환 함수의 경우
- 프로퍼티의 상태 변경이 일어나는 로직의 경우


반대로 상태를 추출/설정할 때는 프로퍼티를 사용하자.


