# Chapter 02 - 코틀린 기초

## 프로퍼티

자에서는 필드와 접근자를 한데 묶어 프로퍼티(property)라고 부른다. 코드로 보면 다음과 같다.

```java
public class Person {
	private final String name;
	
	public Person(String name) {
		this.name = name;
	}
	
	public String getName() {
		return this.name;
	}
}
```

이것을 코틀린에서 작성하면 다음과 같다.

```kotlin
class Person(val name);
```

생성자, getter가 생략된 모습을 볼 수 있다.

코틀린에서는 읽기 전용 프로퍼티를 `val`, setter를 가진 프로퍼티를 `var`로 정의한다. 위에서 본 코틀린 코드는 원래 자바 코드와 똑같이 구현이 숨겨져 있다. 비공개 필드가 있고, 생성자로 그 필드를 초기화하며, 게터로 그 비공개 필드에 접근하는 것이다. 이 때 자바에서 게터와 세터는 각각 get과 set이라는 접두사를 가진다.

> [!Note]
> 게터와 세터의 이름을 정하는 규칙에는 예외가 있다. `is` 로 시작하는 프로퍼티의 게터에는 원래 이름을 그대로 사용하며, 세터는 `is`를 `set`으로 바꾼 이름을 사용한다.

대부분의 프로퍼티에는 그 프로퍼티의 값을 저장하기 위한 필드가 있다. 이를 *backing field*라고 부른다. 하지만 커스텀 게터를 작성하면 프로퍼티 값을 그때그때 계산할 수도 있다.

```kotlin
// 커스텀 게터 예시
class Rectangle(val height: Int, val width: Int) {
	val isSquare: Boolean
		get() {
			return height == width
		}
}
```

위의 예시에서 `isSquare` 프로퍼티는 값을 저장하는 필드가 필요없고, 게터만 존재한다.

## 코틀린 소스코드 구조

모든 코틀린 파일의 맨 앞에 `package` 문을 넣을 수 있는데, 이렇게 하면 그 파일 안에 있는 모든 선언(클래스, 함수, 프로퍼티 등)이 해당 패키지에 들어간다.

```kotlin
package geometry.shapes

// .. Rectangle class 정의

fun createRandomRectangle(): Rectangle {
	// 구현 생략
}

```

위의 코드에서 `createRandomRectangle()` 함수를 임포트하고 싶다면 클래스를 임포트하듯이 `import geometry.shapes.createRandomRectangle`라고 쓰면 된다.

코틀린에서는 여러 클래스를 한 파일에 넣을 수 있고, 파일의 이름도 마음대로 정할 수 있다. 예를 들어 `geometry.shapes` 라는 패키지가 있다면 패키지 내부의 모든 내용을 geometry/shapes.kt 파일에 넣어도 되는 것이다.

## 	스마트 캐스트

코틀린에서는 `is`를 사용해 변수의 타입을 검사할 수 있다. 이것은 자바의 `instanceof` 와 비슷하다. 그러나 자바에서는 `instanceof`로 타입을 검사한 다음, 명시적으로 캐스팅을 하고 나서야 멤버에 접근이 가능하다. 하지만 코틀린은 `is`로 타입 검사를 하고 나면 캐스팅을 할 필요없이 그 변수가 그 타입으로 선언된 것처럼 쓸 수 있다(실제로는 컴파일러가 캐스팅을 수행). 이를 스마트 캐스트라고 한다. 

스마트 캐스트는 `is`로 변수에 든 값의 타입을 검사한 다음에 그 값이 바뀔 수 없는 경우에만 작동한다. 즉, 프로퍼티는 반드시 `val`이어야 하며 커스텀 접근자를 사용한 것도 아니어야 한다. 

명시적으로 타입 캐스팅을 하려면 `as` 키워드를 쓴다.

