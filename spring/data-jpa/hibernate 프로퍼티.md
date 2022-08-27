> `spring.jpa.properties.hibernate` 에서 설정할 수 있는 프로퍼티를 정리한다.
 사용할 수 있는 hibernate 설정은 `org.hibernate.cfg.AvailableSettings` 클래스를 참조하면 된다.

## `hibernate.format_sql`

`spring.jpa.properties.hibernate.show_sql` 을 `true` 로 설정하면 사용되는 쿼리가 콘솔에 출력된다. 이 때는 콘솔 창에 한 줄로 찍히게 되므로 가독성이 떨어진다.
그 때 `hibernate.format_sql`  옵션을 사용하면 보기좋게 쿼리문이 출력된다.

## `hibernate.query.in_clause_parameter_padding`

