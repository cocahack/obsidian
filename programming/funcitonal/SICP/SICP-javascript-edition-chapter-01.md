# SICP 1장

## 1.1 The Elements of Programming

단순한 아이디어를 결합하여 더 복잡한 아이디어를 만드는 것이 프로그래밍이다. 모든 강력한 프로그래밍 언어는 이를 달성할 수 있도록 세 가지의 메커니즘을 제공한다.

> 언어와 관련있는 가장 단순한 개체를 나타내는 **primitive expression** 과 좀 더 단순한 것으로부터 구축되는 **means of combination**, 복합 요소가 단위로 명명되고 조작될 수 있는 **means of abstraction**.


#### expression evaluation methods

**normal-order evaluation**은 완전히 펼쳤다가 줄여나가는(fully expand and then reduce)" 평가 방식이다.

```text

// Expend phase

f(5)

sum_of_squares(5 + 1, 5 * 2)

square(5 + 1) * square(5 + 1) + square(5 * 2) + square(5 * 2)

(5 + 1) * (5 + 1) + (5 * 2) * (5 * 2)

// Reduce phase

6 * 6 + 10 * 10

36 + 100

136

```


**applicative-order evaluation** 은 