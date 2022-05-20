# Project Reactor
## Intro

[[reactive-streams]] 의 구현체 중 하나.

리액터는 백프레셔를 지원하는 JVM을 위한 완전한 논블로킹 리액티브 프로그래밍의 근간이다. Java 8의 `CompletableFuture`와 `Stream`, `Duration` 등 함수형 API와 직접 통합되었다.

## Core - Mono, Flux

리액터의 핵심 API는 `Mono`와 `Flux` 이다. `Mono` 는 0개 또는 1개의 엘리먼트를, `Flux`는 0개 이상의 엘리먼트를 나타낸다.

`Mono`와 `Flux` 모두 Reactive Streams의 `Publisher` 인터페이스 구현체이다. 즉, 생성된 데이터를 `subscriber`에 넘겨줄 수 있는 것이며, 그 개수는 각 클래스에 정의된 개수만큼이다.

### 생성

> [!Info]
> 아래에서 언급되지 않은 내용은 javadoc 참고
> - [Mono Javadoc](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html)
> - [Flux Javadoc](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html)

#### Mono

```java
class MonoCreation {

	public void example() {
		// 1. 빈 Mono 생성
		final var emptyMono = Mono.empty();
		
		// 2. 아무것도 하지 않는 Mono. 어떤 시그널도 emit하지 않음
		final var never = Mono.never();
		
		// 3. 객체로 초기화
		final var monoWithValue = Mono.just("foo");
		
		// 4. 예외로 초기화
		final var fluxWithException = Flux.error(new RuntimeException());
		
	}

}
```

#### Flux

```java
class FluxCreation {

	public void example() {
		// 1. 빈 Flux 생성
		final var emptyFlux = Flux.empty();
		
		// 2. 값으로 초기화
		final var fluxWithValues = Flux.just("foot", "bar");
		
		// 3. Iterable 객체로부터 초기화
		final var fluxWithIterable = Flux.just(List.of("foo", "bar"));
		
		// 4. 예외로 초기화
		final var fluxWithException = Flux.error(new RuntimeException());
		
		// 5. 특정 시간마다 생성 (0 부터 9까지 100ms마다 생성)
		final var fluxWithInterval = Flux.interval(Duration.ofMillis(100)).take(10);
	}

}
```

## Transform

데이터를 변환하는 오퍼레이터 `map` 과 `flatMap` 이 있다.

`map` 은 레이턴시가 존재하지 않을 것이라고 예상되는 변환을 1:1로 실행할 때 사용한다.

반면 `flatMap` 은 레이턴시가 존재할 것 같은 변환을 적용할 때 사용한다. 레이턴시가 존재한다면 비동기적으로 실행해야 하며, 따라서 반환 타입은 `Mono` 나 `Flux` 가 될 것이다. 따라서 `flatMap` 은 `Publisher<T>` 를 반환하는 `Function` 타입을 파라미터로 한다. 

만약 `map` 이었다면 반환되는 타입은 `Flux<Publisher<T>>` 가 될 것이며, 이것을 다루기는 매우 불편 할 것이다. 그러나 `flatMap` 은 내부의 `Publisher` 들을 구독하고 이를 하나의 전역적인 결과로 병합할 수 있다. 단, 각각의 `Publisher` 들이 도착하는 시간은 다를 수 있기 때문에, 결과로 만들어진 `Flux`는 기존과 같은 순서가 보장되지는 않는다. 


> [!Caution]
> `flatMap` 을 사용한다고 해서 인자로 전달한 함수가 비동기로 실행되는 것은 아니다. 
> `parallel()` 등의 스케줄러를 사용해야만 한다.

순서를 보장하고 싶다면 두 가지 방법이 있다. 

첫 번째는 `concatMap` 이다. `concatMap` 이 순서를 보장할 수 있는 것은 `Publisher` 를 순차적으로 처리하기 때문이다. `parallel()` 을 사용하더라도 병렬 실행은 되지 않는다(단, 쓰레드는 서로 다른 쓰레드를 사용하게 된다).

두 번째는 `flatMapSequantial()` 이다. `concatMap`과 달리 모든 `Publisher` 의 이벤트를 트리거하는데, 결과를 병합할 때는 순차적으로 병합한다. 

## Merge

여러 개의 `Publisher`를 하나의 `Flux`로 병합하는 오퍼레이션이다. 

병합하는 방법은 `merge()`와 `concat()`이 있다. 이 두 가지의 차이는 위에서 설명했던 `flatMap`과 `concatMap()`와 비슷하게, 인자로 전달한 `publisher` 의 순서를 유지하는지(그리고 `concat` 부류는 이전 작업이 끝난 뒤 실행) 여부에 차이가 있다.

## Back Pressure

Back pressure는 subscriber에서 요청 개수를 조절하는 메커니즘이다. Reactive Streams는 기본적으로 publisher가 subscriber로 푸시하는 방식이지만, subscriber가 back pressure를 사용해서 가져올 메시지를 조절하는 pull 방식도 혼용하고 있다. 

Reactor에서는 `request(long)` 메소드로 요청 개수를 조절할 수 있다. Flux, Mono에 포함된 `log()` API 를 이용하면 스트림이 어떻게 흐르는지 자세하게 로그를 찍어볼 수 있는데, 여기에 request가 몇 개 요청되고 있는지 볼 수 있다.

```text
14:03:33.730 [single-1] INFO reactor.Flux.ConcatArray.1 - onSubscribe(FluxConcatArray.ConcatArraySubscriber)
14:03:33.731 [single-1] INFO reactor.Flux.ConcatArray.1 - request(256)
14:03:35.750 [single-1] INFO reactor.Flux.ConcatArray.1 - onNext(A)
14:03:35.750 [single-1] INFO reactor.Flux.ConcatArray.1 - onNext(B)
14:03:35.750 [single-1] INFO reactor.Flux.ConcatArray.1 - onNext(C)
14:03:36.755 [single-1] INFO reactor.Flux.ConcatArray.1 - onNext(D)
14:03:36.756 [single-1] INFO reactor.Flux.ConcatArray.1 - onNext(E)
14:03:36.756 [single-1] INFO reactor.Flux.ConcatArray.1 - onNext(F)
14:03:36.756 [single-1] INFO reactor.Flux.ConcatArray.1 - onComplete()
```

## 에러 처리

스트림 처리 도중 에러가 발생했을 때, 이를 처리할 수 있는 API를 제공한다. `onErrorXXX` 류의 API가 그것이다. 

예를 들어, 예외가 발생했을 때 아래와 같이 처리할 수 있다.

```java
class ErrorExample1 {

	// IllegalStateException 이 발생하면 기본 값 2
	public Mono<Integer> exampleMono(Mono<Integer> mono) {
		return mono.onErrorReturn(IllegalStateException.class::isInstance, 2);
	}
	
	// 일단 예외가 발생하면 4, 5, 6을 emit
	public Flux<Integer> exampleFlux(Flux<Integer flux) {
		return flux.onErrorResume(e -> Flux.just(4, 5, 6));
	}
	
}
```

Checked exception의 경우는 조금 더 복잡하다. `map()` 과 같은 변환 API에 전달하는 람다식에서는 반드시 checked exception을 처리해야한다. 

가장 직관적인 방법은 `try-catch`로 감싼 다음, Checked Exception을 Unchecked Exception으로 감싸는 것이다. Reactor는 `Exceptions.propagate` 라는 편의 API를 제공한다.

```java
class ErrorExample2 {

	public Flux<Integer> exampleFlux(Flux<Integer flux) {
		return flux.map(i -> {
			try {
				return add10(i);
			} catch (Exception e) {
				throw Exceptions.propagate(e);
			}
		})
	}
	
	private int add10(int value) throw Exception {
		if (value < 0) {
			throw new Exception();
		}
		return value + 10;
	}
	
}
```

## 컨텍스트 변경 - `publishOn`, `subscribeOn`

> [!Notice]
> 스프링 공식 블로그에 올라온 [글](https://spring.io/blog/2019/12/13/flight-of-the-flux-3-hopping-threads-and-schedulers)을 바탕으로 작성했습니다.

### `publishOn`

다음과 같은 코드가 작성되어있다고 가정하자.

```java
@Slf4j
public PublishOnExample {

	public void exmaple(List<String> firstListOfUrls, List<String> secondListOfUrls) {
		Flux.fromIterable(firstListOfUrls) //contains A, B and C
			.map(url -> blockingWebClient.get(url))
			.subscribe(body -> log.info(Thread.currentThread().getName + " from first list, got " + body));

		Flux.fromIterable(secondListOfUrls) //contains D and E
			.map(url -> blockingWebClient.get(url))
			.subscribe(body -> log.info(Thread.currentThread().getName + " from second list, got " + body));
	}
}
```

인자로 URL의 리스트를 받고 하나씩 꺼내 Restful API를 호출하는 코드다. `blockingWebClient.get(url)` 부분은 blocking api를 호출한다고 가정할 때, 위 코드는 하나의 쓰레드를 이용해서 실행하게 된다. 예상되는 출력은 다음과 같다.

```text
main from first list, got A
main from first list, got B
main from first list, got C
main from second list, got D
main from second list, got E
```

Async, non-blocking 코드를 위해서는 블락되는 코드를 별도 쓰레드에 격리해야 한다. 이 때 `publishOn` 을 쓸 수 있다. `publishOn` 의 인자에 `Schduler` 객체를 넘겨주면 코드가 실행되는 컨텍스트를 변경할 수 있다. 

```java
@Slf4j
public PublishOnExample {

	public void exmaple(List<String> firstListOfUrls, List<String> secondListOfUrls) {
		Flux.fromIterable(firstListOfUrls) //contains A, B and C
			.publishOn(Schedulers.boundedElastic())
			.map(url -> blockingWebClient.get(url))
			.subscribe(body -> log.info(Thread.currentThread().getName() + " from first list, got " + body));

		Flux.fromIterable(secondListOfUrls) //contains D and E
			.publishOn(Schedulers.boundedElastic())
			.map(url -> blockingWebClient.get(url))
			.subscribe(body -> log.info(Thread.currentThread().getName() + " from second list, got " + body));
	}
}
```

이렇게 코드를 변경하면 실행 결과는 다음과 같이 바뀐다.

```
boundedElastic-1 from first list, got A
boundedElastic-2 from second list, got D
boundedElastic-1 from first list, got B
boundedElastic-2 from second list, got E
boundedElastic-1 from first list, got C
```

두 개의 Flux가 각자 할당된 쓰레드에서 실행되므로 번갈아가면서 결과가 출력된다. 

`publishOn` 으로 컨텍스트를 변경하면 이후 체이닝으로 작성한 메소드는 모두 그 스케줄러에 의해 계속 실행된다. 중간에 다른 스케줄러로 변경하는 것도 가능하다.

### `subscribeOn`

`publishOn` 은 위에서 봤던 것처럼, 소스에서 들어오는 시그널(`onNext`, `onComplete`, `onError` 등)을 특정 `Scheduler` 에서 publish하는 방법이다.

그런데 위의 예시와 다르게, 직접 조작할 수 없는 외부 API가 `Flux` 나 `Mono`를 반환하는데 코드 작성자가 실수로 publish할 `Scheduler`를 지정하지 않았을 때는 어떻게 업스트림의 쓰레드를 변경할 수 있을까?

이럴 때 사용하는 것이 `subscribeOn` 이다. 이전에 스케줄러를 지정하지 않았더라도 `subscribe` 메소드가 실행될 장소를 변경할 수 있는 것이다.

```java
@Slf4j
public SubscribeOnExample {

	final Flux<String> fetchUrls(List<String> urls) {
	  return Flux.fromIterable(urls)
		.map(url -> blockingWebClient.get(url)); // 실행 환경이 바뀌지 않는다!
	}

	public void example(List<String> firstListOfUrls, List<String> secondListOfUrls) {
		fetchUrls(firstListOfUrls)
		  .subscribeOn(Schedulers.boundedElastic())
		  .subscribe(body -> log.info(Thread.currentThread().getName() + " from first list, got " + body));

		fetchUrls(secondListOfUrls)
		  .subscribeOn(Schedulers.boundedElastic())
		  .subscribe(body -> log.info(Thread.currentThread().getName() + " from second list, got " + body));
	}

}
```

위 코드는 `publishOn` 에서 설명에 사용했던 코드와 동일하게 동작한다.

```
boundedElastic-1 from first list, got A
boundedElastic-2 from second list, got D
boundedElastic-1 from first list, got B
boundedElastic-2 from second list, got E
boundedElastic-1 from first list, got C
```

### `publishOn` 과 `subscribeOn` 을 여러 개 겹치면

결과적으로 `publishOn`은 새로 호출할 때마다 그 아래에 실행될 operation의 스케줄러가 변경되며, `subscribeOn` 은 가장 처음에 호출되는 operation에만 영향을 주고, 가장 마지막에 호출한 `subscribeOn` 의 스케줄러만 사용한다.

```java
public class ComplexExample {
	

	private Flux<String> fetchUrls(List<String> urls) {  
		return Flux.fromIterable(urls)  
				.map(url -> {  
					log.info("fetchUrls - {}", url);  
					return url;  
				});  
	}  

	public Flux<String> subscribeOn(List<String> firstListOfUrls) {  
		return fetchUrls(firstListOfUrls)  
				.publishOn(newSingle("publishOn-A", true))  
				.map(s -> {  
					log.info("after fetch - {}", s);  
					return s;  
				})  
				.publishOn(newSingle("publishOn-B", true))  
				.doOnNext(s -> log.info("doOnNext - {}", s))  
				.subscribeOn(newSingle("subscribeOn", true));  
	}  

	public void subscribeOn2(List<String> firstListOfUrls) {  
		subscribeOn(firstListOfUrls)  
				.subscribeOn(newSingle("subscribeOn2", true))  
				.subscribe(body -> log.info(Thread.currentThread().getName() + " from first list, got " + body));  
	}
	
    public static void main(String[] args) throws InterruptedException {  
    
        ComplexExample example = new ComplexExample();  
  
        List<String> firstUrls = List.of("A", "B", "C");
  
        example.subscribeOn2(firstUrls);  
  
        Thread.sleep(300);  
    }
}

```

위 코드를 실행한 결과는 다음과 같다.

```
23:16:15.281 [subscribeOn-2] INFO me.cocahack.reactordemo.ReactorDemoApplication - fetchUrls - A
23:16:15.282 [subscribeOn-2] INFO me.cocahack.reactordemo.ReactorDemoApplication - fetchUrls - B
23:16:15.282 [subscribeOn-2] INFO me.cocahack.reactordemo.ReactorDemoApplication - fetchUrls - C
23:16:15.282 [publishOn-A-4] INFO me.cocahack.reactordemo.ReactorDemoApplication - after fetch - A
23:16:15.282 [publishOn-A-4] INFO me.cocahack.reactordemo.ReactorDemoApplication - after fetch - B
23:16:15.282 [publishOn-A-4] INFO me.cocahack.reactordemo.ReactorDemoApplication - after fetch - C
23:16:15.282 [publishOn-B-3] INFO me.cocahack.reactordemo.ReactorDemoApplication - doOnNext - A
23:16:15.282 [publishOn-B-3] INFO me.cocahack.reactordemo.ReactorDemoApplication - publishOn-B-3 from first list, got A
23:16:15.282 [publishOn-B-3] INFO me.cocahack.reactordemo.ReactorDemoApplication - doOnNext - B
23:16:15.282 [publishOn-B-3] INFO me.cocahack.reactordemo.ReactorDemoApplication - publishOn-B-3 from first list, got B
23:16:15.282 [publishOn-B-3] INFO me.cocahack.reactordemo.ReactorDemoApplication - doOnNext - C
23:16:15.282 [publishOn-B-3] INFO me.cocahack.reactordemo.ReactorDemoApplication - publishOn-B-3 from first list, got C
```

## 테스트 - `StepVerifier`

리액터로 만든 Publisher를 테스트할 때 사용하는 API, `StepVerifier` 가 있다. 이 API를 사용하기 위해서는 `reactor-test` 라이브러리를 의존성으로 걸어야 한다.

`StepVerifier`는 Reactive Streams의 모든 `Publisher` 구현체를 구독할 수 있고, 이를 바탕으로 시퀀스를 검증하기 위한 assertion을 직접 정의하여 테스트할 수 있다. 유저가 정의한 assertion이 맞지 않다는 이벤트가 트리거되면 `StepVerifier`는 `AssertionError`를 생성할 것이다.

`StepVerifier`의 `.create` 스태틱 메소드로 인스턴스를 생성할 수 있다. 이 인스턴스는 expectation을 구성하기 위한 DSL을 제공한다.

**주의할 점은 반드시 `verify()` 메소드를 호출하거나, 종료 expectation과 `verify`가 합쳐진 단축 메소드(`verifyErrorMessage(String)`, `verifyComplete()` 등)를 호출해야 한다는 것이다**.

### 예시

```java
class StepVerifierExamples {

	// 1. 값 검증 - "foo", "bar" 생성 후 완료되는 flux
	void expectFooBarComplete(Flux<String> flux) {
		StepVerifier.create(flux)
			.expectNext("foo", "bar")
			.expectComplete()
			.verify();
	}
	
	// 2. 에러 검증 - "foo", "bar" 생성 후 에러 발생하는 flux
	void expectFooBarError(Flux<String> flux) {
		StepVerifier.create(flux)
			.expectNext("foo", "bar")
			.verifyError(RuntimeException.class);
	}
	
	// 3. Assertion 사용하여 복잡한 객체를 테스트 - User 객체의 username 검증
	void expectSkylerJesseComplete(Flux<User> flux) {
		StepVerifier.create(flux)
			.assertNext(u -> assertThat(u.getUsername()).isEqualTo("swhite"))
			.assertNext(u -> assertThat(u.getUsername()).isEqualTo("jpinkman"))
			.verifyComplete();
	}
	
	// 4. Flux 가 생성하는 원소의 개수 테스트
	void expect10Elements(Flux<Long> flux) {
		StepVerifier.create(flux)
			.expectNextCount(10)
			.verifyComplete();
	}
	
}
```

#### 특수 케이스 - 지연되는 Publisher 테스트하기

Publisher가 데이터를 생성하는데 시간이 걸린다면, 이를 테스트할 때도 그만큼 시간이 지연될 것이다. 예를 들어, Flux가 모든 데이터를 생성하기까지 1시간이 걸린다면, `StepVerifier`도 동일하게 1시간이 걸리는 것이다.

빠른 테스트를 위해, `StepVerifier`는 가상의 시간을 다룰 수 있는 옵션을 제공한다. `StepVerifier.withVirtualTime(Supplier<Publisher>)` 을 사용하면 된다.

`withVirtualTime()` 메소드를 사용하면 `StepVerifier`는 모든 기본 `Schedulers` 를 가상 시계를 조작할 수 있는 `VirtualTimeScheduler` 의 인스턴스 한 개로 교체한다. 오퍼레이터가 스케줄러를 선택할 수 있도록 하기 위해 `withVirtualTime()` 에 람다를 전달하여 오퍼레이터 체인이 늦게 빌드될 수 있도록 한다.

`withVirtualTime()` 을 사용했다면 시간을 임의로 감을 수 있다. `.thenAwait(Duration)`을 사용하면 인자로 전달한 시간만큼 시간이 흐르는 셈이다. 

`.expectNoEvent(Duration)` 도 있는데, 이 메소드는 인자로 전달한 시간동안 이벤트가 발생하지 않을 것이라는 assertion이다. 만약 이벤트가 발생한다면 테스트는 실패할 것이다. 시간이 흐르지 않았어도 거의 모든 시간에는 최소한 *subscription* 이벤트가 발생하기 때문에 일반적으로 `.expectNoEvent(Duration)` 다음에 `expectSubscription()` 을 사용해야 한다.

```java
class SpecialStepVerifierExamples {
	
	// 지연된 Flux를 빠르게 테스트
	void expect3600Elements(Supplier<Flux<Long>> supplier) {
		StepVerifier.withVirtualTime(supplier)
			.thenAwait(Duration.ofHours(1))
			.expectNextCount(3600)
			.verifyComplete();
	}
	
}
```

> [!Info]
> 보다 자세한 내용은 [공식 문서](https://projectreactor.io/docs/core/release/reference/index.html#testing)와 [javadoc](https://javadoc.io/doc/io.projectreactor.addons/reactor-test/latest/reactor/test/StepVerifier.html)을 참조하면 된다.