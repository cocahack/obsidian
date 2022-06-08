# Item 17: named argument를 사용하라

다음의 코드를 비교해보자.

```kotlin
val text1 = (1..10).joinToString("|")
val text2 = (1..10).joinToString(separator = "|")
```

두 코드는 같은 결과를 나타내지만, 아래에 있는 코드가 더 가독성이 좋다. Parameter가 뭘 뜻하는지 바로 알 수 있기 떄문이다.

## 언제 사용해야 하는가?

코드가 길어지지만 두 가지 장점이 있다.

- 값이 무엇을 나타내는지 알 수 있어 가독성이 좋다.
- 파라미터 순서와 상관없다.

이에 따라, 세 가지 경우에 대해 named argument를 사용하는 것이 권장된다.

### Default argument

일반적으로 함수 이름은 필수 파라미터하고만 밀접하게 관련이 있어 optional 파라미터들에 대한 설명이 부족하다. named argument로 이를 나타내면 좀 더 명확할 것이다.

### 같은 타입의 파라미터가 많을 때

같은 파라미터가 많이, 그리고 연속적으로 나올 경우 argument 순서를 잘못 넣는 실수를 빨리 알아차리기가 어렵다.

### 함수 타입의 파라미터가 있을 떄

일반적으로, 함수 타입 파라미터는 가장 마지막에 오는 것이 좋다. 다음과 같이 나타낼 수 있기 떄문이다.

```kotlin

thread {
	// ..code
}
```

이외에 모든 함수 타입 파라미터에 대해서는 named argument를 사용하는 것이 훨씬 이해하기 쉽다.

예를 들면 다음과 같다.

```kotlin
observable.getUsers()
	.subscribeBy(
		onNext = { users: List<User> -> 
			// ...
		},
		onError = { throwable: Throwable ->
			// ...
		},
		onCompleted = {
			// ...
		}
	)
```



