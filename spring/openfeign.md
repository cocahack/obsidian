# OpenFeign

[OpenFeign](https://github.com/OpenFeign/feign)은 자바 진영에서 사용하는 HTTP 클라이언트 라이브러리이다. 

애노테이션을 기반으로 Request를 정의할 수 있어 사용하기 쉽고, HTTP 통신과 관련된 세부 사항은 감춰져 있어 개발자는 정의에만 집중하면 된다.

## Spring Cloud OpenFeign

스프링 클라우드에 포함된 프로젝트로 자동 설정 및 바인딩을 통해 스프링 부트 애플리케이션과 OpenFeign을 통합할 수 있게 해준다.

### Example

스프링에서 컨트롤러를 정의하듯이 클라이언트 요청 부분을 정의할 수 있다.

```kotlin
@FeignClient(name = "external-service", url = "http://10.10.10.10:8080")
interface ExternalServiceClient {

	@PostMapping("/hello")
	fun hello(@RequestBody body: Body): HelloResponse;
}
```

정의한 인터페이스는 애플리케이션 구동 시  `@Repository`처럼 자동으로 빈이 생성되고 다른 빈에 주입도 될 수 있다.

