---
aliases: [캐싱 전략]
---

# 캐싱 전략

## Cache-Aside

애플리케이션이 직접 캐시에 접근하는 방식이다.

1. 애플리케이션은 캐시를 먼저 확인한다.
2. 캐시에 데이터가 있다면 이를 반환한다.
3. 캐시에 데이터가 없다면 데이터베이스에서 데이터를 가져오는 등의 추가 작업을 한다. 그 결과물을 클라이언트에게 반환하고 캐시에 저장한다.

### 장단점

## Read-Through Cache

## Write-Through Cache

## Write-Around

## Write-Back


## References

- [Caching Strategies and How to Choose the Right One](https://codeahoy.com/2017/08/11/caching-strategies-and-how-to-choose-the-right-one/)