---
aliases: [트랜잭션]
---
# Transaction

## 개념

데이터베이스에서 여러 개의 읽기와 쓰기를 하나의 논리적인 단위로 묶는 방법이다.

## 조금씩 다른 트랜잭션 개념

데이터베이스마다 트랜잭션의 성격은 다르다. 대표적으로 RDBMS와 NoSQL에서 트랜잭션의 개념이 존재하지만(일부 NoSQL은 트랜잭션을 아예 제공하지 않는다) 세부적인 내용은 다르다. 

### RDBMS에서의 트랜잭션

RDBMS에서 트랜잭션은 ACID 속성을 만족한다. 

- A (Atomicity, 원자성) 
- C (Consistency, 일관성)
- [I (Isolation, 격리성)](Isolation.md)
- D (Durability, 영속성)

### NoSQL에서의 트랜잭션

트랜잭션을 제공하는 대표적인 NoSQL은 MongoDB가 있다.

[MongoDB Transaction](https://docs.mongodb.com/manual/core/transactions/)

---
#db #transaction 


