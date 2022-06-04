# 이펙티브 코틀린 - Item 2: 변수의 스코프를 최소화하라

## 요약

- 변수를 가능한 한 가장 가까운 스코프에 선언하도록 하라. 또한 지역 변수는 `val`을 활용하라. 
- 변수는 항상 람다 안에 캡쳐됨을 명심하라.

## 개요

상태를 정의할 때, 변수와 프로퍼티의 스코프를 다음과 같은 방법으로 좁히는 것이 좋다.

- 프로퍼티 보다 로컬 변수를 사용하라
- 가능한 가장 좁은 스코프에서 변수를 사용하라. 예를 들면, 루프 안에서 사용되는 변수는 루프 안에 정의하는 것이 있다.

이렇게 변수의 스코프를 조일 수록, 프로그램을 추적하고 관리하는 것이 보다 간단해진다.

![[Pasted image 20220604163259.png]]

### Capturing

에라토스테네스의 체를 무한 시퀀스로 구현한 코드의 예시는 다음과 같다.

```kotlin
val primes: Sequence<Int> = sequence {
	var numbers = generateSequence(2) { it + 1 }
	
	while (true) {
		val prime = numbers.first()
		yield(prime)
		numbers = numbers.drop(1).filter { it % prime != 0 }
	}
}

print(primes.take(10).toList()) 
// [2, 3, 5, 7, 11, 13, 17, 19, 23, 29]
```

그러나 코드를 다음과 같이 변경하면 틀린 답이 출력된다.

```kotlin
val primes: Sequence<Int> = sequence {
	var numbers = generateSequence(2) { it + 1 }
	
	var prime: Int
	while (true) {
		prime = numbers.first()
		yield(prime)
		numbers = numbers.drop(1).filter { it % prime != 0 }
	}
}

print(primes.take(10).toList()) 
// [2, 3, 5, 6, 7, 8, 9, 10, 11, 12]
```


이런 문제가 발생한 이유는 람다 상에서 변수가 캡쳐되었기 때문이다. 시퀀스를 사용했기 때문에 필터링은 지연 실행된다. 가변 프로퍼티 `prime`를 참조하는 필터를 추가한다. 그러므로 `prime` 의 마지막 값이 항상 필터되는 것이다.

의도하지 않은 캡처링은 위와 같이 잘못된 코드를 작성할 위험이 있다. 따라서 가변적인 상황을 피하고 변수의 스코프를 좁혀야 하는 것이다.