> `spring.jpa.properties.hibernate` 에서 설정할 수 있는 프로퍼티를 정리한다.
 사용할 수 있는 hibernate 설정은 `org.hibernate.cfg.AvailableSettings` 클래스를 참조하면 된다.

## `hibernate.format_sql`

`spring.jpa.properties.hibernate.show_sql` 을 `true` 로 설정하면 사용되는 쿼리가 콘솔에 출력된다. 이 때는 콘솔 창에 한 줄로 찍히게 되므로 가독성이 떨어진다.
그 때 `hibernate.format_sql`  옵션을 사용하면 보기좋게 쿼리문이 출력된다.

## `hibernate.query.in_clause_parameter_padding`

기본적으로, `IN` 절은 바인딩된 파라미터 값을 모두 포함할 수 있도록 늘어나게 된다. 그런데 실행 계획 캐싱을 지원하는 DBMS의 경우 `IN` 절의 파라미터 수가 줄어들면 캐시가 적중할 확률이 더 높아진다. 따라서 이 프로퍼티를 `true` 로 설정하면 바인딩 된 파라미터의 수를 2의 배수로 확장한다.

## `hibernate.default_batch_fetch_size` 

Lazy loading에 의해 추가 쿼리가 발생하는 경우, 컬렉션 등을 가져올 때 프로퍼티에 적은 개수 씩 `IN` 절을 사용하여 조회하게 된다. 이렇게 하면 흔히 알려진 N+1 문제에서 실행하는 쿼리 개수를 줄일 수 있다.

## `hibernate.connection.provider_disables_autocommit`

이 프로퍼티를 `true`로 설정하면 Connection Pool의 autocommit 설정을 무조건 따르겠다는 의미가 된다. 원래는 트랜잭션을 맺고나면 `setAutoCommit()` 을 실행하면서 추가적인 네트워크 오버헤드가 발생하는데, 이 프로퍼티를 사용하면 이러한 오버헤드를 제거할 수 있는 것이다.

[참고](https://pkgonan.github.io/2019/01/hibrnate-autocommit-tuning)

