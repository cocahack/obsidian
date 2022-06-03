# Resilience4j

Resilience4j는  [Netflix Hystrix](https://github.com/Netflix/Hystrix)으로부터 영감을 받아 만들어진 경량의 Fault tolerance 라이브러리로, Java 8과 함수형 프로그래밍을 위해 설계되었다(경량이라고 주장하는 근거는 [Vavr](http://www.vavr.io/) 외에 다른 외부 라이브러리가 없기 때문이라고 한다).

## Core modules

### `CircuitBreaker` 

`CLOSED`와 `OPEN`, `HALF_OPEN`등 세 개의 노말 상태와 그리고 `DISABLED` 와 `FORCED_OPEN` 두 개의 특수 상태 등으로 이루어진 FSM을 구현한 것이다.

![Circuit breaker FSM](https://files.readme.io/39cdd54-state_machine.jpg)

`CircuitBreaker`는 요청 결과를 저장하고 집계하기 위해 슬라이딩 윈도우를 사용한다. 이 때 카운트 기반 슬라이딩 윈도우(count-based sliding window)와 시간 기반 슬라이딩 윈도우(time-based sliding window) 중 하나를 선택할 수 있다.

#### 카운트 기반 슬라이딩 윈도우

마지막 $N$ 개의 요청 결과를 집계한다. 

#### 시간 기반 슬라이딩 윈도우

최근 $N$ 초간 요청 결과를 집계한다.