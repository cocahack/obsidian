# Race condition
## 개념

멀티쓰레드 환경에서 공유된 자원에 여러 쓰레드가 동시에 접근할 경우 발생할 수 있는 이상현상을 말한다.

프로그램이나 시스템 등의 실행/출력 결과가 일정하지 않고, 입력의 타이밍, 실행되는 순서나 시간 등에 영향을 받게 되는 상황이 발생한다.

### 예시

A 쓰레드와 B 쓰레드가 동시에 `x` 라는 변수를 읽고 쓴다고 하자.

이 경우 다음과 같은 상황이 발생할 수 있다.

```text
Thread A: read x, value is 7
Thread A: add 1 to x, value is now 8
Thread B: read x, value is 7
Thread A: stores 8 in x
Thread B: add 1 to x, value is now 8
Thread B: stores 8 in x 
```

두 쓰레드가 동시에 값을 바꿨지만 결과적으로 쓰레드 B의 연산결과는 무시된 것이나 다름없다. 

A 쓰레드와 B 쓰레드가 변경한 값을 모두 반영할 수 있으려면 

- 락을 사용한다.
- 하드웨어 수준에서 원자성을 제공하는 연산을 사용한다.

와 같은 방법이 있다.

### 원인

왜 이런 현상이 발생하는가?

