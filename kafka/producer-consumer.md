# Producer & Consumer

## Producer 

### Producer flow

![[kafka-producer.png|<<카프카 프로듀서 - 출처:https://dzone.com/articles/take-a-deep-dive-into-kafka-producer-api >>]]

프로듀서 → 카프카로의 흐름은 다음과 같다.

1. `ProducerRecord` 를 생성하여 카프카에 메시지를 프로듀싱한다.
	- `ProducerRecord` 는 레코드를 보내고 싶은 토픽과 값을 포함한다. 필요하다면 키와 파티션을 명시할 수도 있다.
2. `ProducerRecord`를 보내고나면, 프로듀서는 제일 먼저 키-값 객체를 `ByteArray`로 직렬화하여 네트워크로 데이터를 보낼 준비를 한다.
3. 직렬화된 데이터는 파티셔너에 전달된다. `ProducerRecord`에 파티션을 명시했다면 파티셔너는 아무것도 하지 않고 `ProducerRecord`에 명시했던 파티션을 그대로 반환한다. 그렇지 않다면 파티션을 선택하게 되는데, 이는 대체로 `ProducerRecord` 키에 따라 선택된다. 
4. 프로듀서는 이제 어디에 레코드를 보낼지 알게 되었다. 이 레코드는 바로 전송되는게 아니라 잠시 모아두었다가 한 번에 같은 토픽의 파티션에 전송된다.
5. 프로듀서가 보낸 배치 전송된 레코드는 이를 처리하는 카프카의 전용 쓰레드가 처리한다. 브로커는 메시지를 받으면 응답을 전송한다. 
6. 카프카가 메시지를 쓰는데 성공하면 토픽과 파티션, 그리고 파티션 내 레코드의 오프셋이 포함된`RecordMetadata` 객체를 반환한다. 실패했다면 메시지를 다시 쓰려고 시도하며, 지정된 횟수만큼 실패하면 에러를 반환한다.

### Producer 구성하기

카프카에 메시지를 보내려면 프로듀서 객체가 필요하다. 프로듀서 객체를 만들 때 프로퍼티를 적용할 수 있다.

프로듀서에 사용할 수 있는 프로퍼티는 다음과 같다. 

- `bootstrap.servers` (Mandatory): 클라이언트가 카프카 클러스터에 처음 연결하기 위한 호스트와 포트 정보를 나타낸다. 
> 카프카 클러스터는 클러스터 마스터 개념이 없기 때문에 클러스터 내 모든 서버가 클라이언트의 요청을 받을 수 있다.
- `key.serializer` (Mandatory): 메시지 키의 `Serializer` 클래스를 지정한다.
- `value.serializer` (Mandatory): 메시지 값의 `Serializer` 클래스를 지정한다.
>  자바에서`String` 타입에 대한 기본 `Serializer`는 `org.apache.kafka.common.serialization.StringSerializer` 이다.
- `asks`: 프로듀서가 카프카 토픽의 리더 측에 메시지를 전송한 후 요청을 완료하기를 결정하는 옵션. 가능한 값은 0, 1, all(-1)이 있다.
	- `0`: 빠른 전송을 의미하며, 일부 메시지가 손실될 수 있다.
	- `1`: 리더가 메시지를 받았는지 확인한다. 그러나 모든 팔로워를 전부 확인하지는 않는다. 대부분 이 값을 사용한다.
	- `all(-1)`: 팔로워가 메시지를 받았는지 확인한다. 
- `buffer.byte`: 프로듀서가 카프카 서버로 데이터를 보내기 위해 잠시 대기(배치 전송이나 딜레이 등)할 수 있는 메모리의 크기를 지정한다.
- `enable.idempotence`: `true`를 값으로 할 경우 중복 없는 전송이 가능하다(단, `max.in.flight.requests.per.connection`은 5 이하, `retries`는 0 이상, `acks`는 `all`로 설정해야 한다).
- `max.in.flight.requests.per.connection`: 하나의 커넥션에서 프로듀서가 최대한 ACK 없이 전송할 수 있는 요청 수. 메시지의 순서가 중요하다면 1로 설정해야 하지만, 성능이 떨어질 수 있다.
- `batch.size`: 배치 크기를 설정
- `transactional.id`: '정확히 한 번 전송'을 위해 사용하는 옵션이며, 동일한 `TransactionalId`에 한해 정확히 한 번을 보장한다(단, `enable.idempotence` 값이 `true`여야 한다).

### 카프카로 메시지 보내기

세 가지 방식이 존재한다.

#### 1. Fire-and-forget

메시지를 보내지만 잘 전달되었는지는 확인하지 않는다.

```java
public class FireAndForgetProducer {
	
	private final Producer<String, String> producer;
	
	// constructor ... 
	
	public void send() {
		ProducerRecord<String, String> record = new ProducerRecord<>("topic01", "message");
		try {
			producer.send(record);
		} catch (Exception e) {
			// handle an exception
		}
	}
}

```

#### 2. Synchronous send

동기식으로 전송한다.

```java
public class SynchronousProducer {
	
	private final Producer<String, String> producer;
	
	// constructor ... 
	
	public void send() {
		ProducerRecord<String, String> record = new ProducerRecord<>("topic01", "message");
		try {
			RecordMetadata metadata = producer.send(record).get();
			// ...
		} catch (Exception e) {
			// handle an exception
		}
	}
}

```

#### 3. Asynchronous send

비동기 방식으로 전달한다. 메시지를 전달할 때 콜백을 함께 넘기는 방식이다.

```kotlin

class AsynchronousProducerCallback : Callback {
	
	override fun onCompletion(metadata: RecordMetadata, e: Exception) {
		if (e != null) {
			// handle an exception
		} else {
			// process Metadata
		}
	}
	
}

class AsynchronousProducer(
	private val producer: Producer<String, String>
) {
	
	fun send() {
		val recordz = new ProducerRecord<>("topic01", "message")
		try {
			RecordMetadata metadata = producer.send(record).get()
			
			producer.send(record, AsynchronousProducerCallback())
			// ...
		} catch (Exception e) {
			// handle an exception
		}
	}
	
}
```

## Consumer

### Behavior

토픽에 저장된 메시지를 가져오려면 컨슈머를 이용해야 한다.

컨슈머는 하나 이상의 컨슈머들이 모여 있는 컨슈머 그룹에 속한다. 컨슈머 그룹은 각 파티션의 리더에게 카프카 토픽에 저장된 메시지를 가져오기 위한 요청을 보낸다. 

파티션 수와 컨슈머의 수는 일대일로 매핑되는 것이 이상적이다. 컨슈머의 수가 파티션의 수보다 많다고 해서 더 많은 양의 메시지를 처리할 수 있는게 아니라 오히려 일부 컨슈머가 대기 상태에 있게 되기 때문이다. 
Active/Standby 개념으로 추가 컨슈머가 필요하다고 생각할 수 있다. 하지만 컨슈머 그룹에 있던 컨슈머 하나가 중단되어도 컨슈머 그룹 내에서 리밸런싱 동작으로 장애가 발생한 컨슈머 대신 다른 컨슈머가 그 역할을 계속 수행할 수 있다.

### Consumer 구성하기

컨슈머에 사용할 수 있는 주요 프로퍼티는 다음과 같다.

- `bootstrap.servers` (Mandatory): 프로듀서와 동일하게 브로커의 정보를 넣을 때 사용.
- `fetch.min.bytes`: 한 번에 가지고 올 수 있는 최소 데이터 크기를 지정한다. 지정한 크기보다 작으면 요청에 응답하지 않고 데이터가 누적될 때까지 기다린다.
- `fetch.max.bytes`: 한 번의 요청으로 가지고 올 수 있는 최대 크기를 지정한다.
- `group.id`: 컨슈머가 속한 컨슈머 그룹을 식별하는 식별자.
- `session.timeout.ms`: 이 프로퍼티로 설정된 값만큼의 시간동안 하트비트를 보내지 않으면 컨슈머가 종료된 것으로 판단하여 컨슈머 그룹에서 제외된다.
- `heartbeat.internval.ms`: 하트비트를 보내는 주기를 설정한다. `session.timeout.ms`보다 작아야 하며, 일반적으로 ${1 \over 3}$ 수준으로 설정한다.
- `enable.auto.commit`: 백그라운드로 주기적으로 오프셋을 커밋한다.
- `auto.offset.reset`: 카프카에서 초기 오프셋이 없거나 현재오프셋이 더 이상 존재하지 않는 경우 다음 옵션으로 리셋한다. 
	- `earliest`: 가장 초기의 오프셋값으로 설정한다.
	- `latest`: 가장 마지막의 오프셋값으로 설정한다.
	- `none`: 이전 오프셋값을 찾지 못하면 에러 처리한다. 
- `isolation.level`: 트랜잭션 컨슈머에서 사용되는 옵션으로, `read_uncommitted`는 기본값으로 모든 메시지를 읽고, `read_committed`는 트랜잭션이 완료된 메시지만 읽는다.

### 카프카로부터 메시지 받기

프로듀서와 마찬가지로 세 가지 방식이 있다.

#### 1. Auto commit

```java
public class AutoCommitConsumer {
	
	private final KafkaConsumer<String, String> consumer;
	
	public AutoCommitConsumer() {
		Properties props = new Properties();
		props.put("bootstrap.servers", "kafka01:9092,kafka02:9092,kafka03:9092");
		props.put("group.id", "consumer01");
		props.put("enable.auto.commit", "true");
		props.put("auto.offset.reset", "latest");
		props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
		props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
		
		consumer = new KafkaConsumer<>(props);
		consumer.subscribe(Arrays.asList("topic01"));
	}
	
	public void run() {
		try {
			while(true) {
				ConsumerRecords<String, String> records = consumer.poll(1000);
				
				for (ConsumerRecord<String, String> record : records) {
					// handle record
				}
			}
		} catch (Exception e) {
			// handle exception
		} finally {
			consumer.close();
		}
	}
}
```

`enable.auto.commit` 의 값으로 `true`를 주어 오토 커밋을 활성화 시켰다. `consumer.poll(1000)`로 1초마다 카프카로부터 데이터를 폴링하며 이 시간동안 쓰레드는 블락된다. 

컨슈머 애플리케이션이 오토 커밋을 기본값으로 많이 사용한다. 오프셋을 주기적으로 커밋하기 때문에 관리자가 오프셋을 따로 관리하지 않아도 되는 장점이 있다. 그러나 컨슈머 종료 등이 빈번하게 일어나면 일부 메시지를 못 가져오거나 메시지 중복이 발생할 수도 있다. 

#### 2. Synchronous consume

```java
public class SynchronousConsumer {
	
	private final KafkaConsumer<String, String> consumer;
	
	public AutoCommitConsumer() {
		Properties props = new Properties();
		// same as the first example
		props.put("enable.auto.commit", "false");
		
		consumer = new KafkaConsumer<>(props);
		consumer.subscribe(Arrays.asList("topic01"));
	}
	
	public void run() {
		try {
			while(true) {
				ConsumerRecords<String, String> records = consumer.poll(1000);
				
				for (ConsumerRecord<String, String> record : records) {
					// handle record
				}
				
				consumer.commitSync();
			}
		} catch (Exception e) {
			// handle exception
		} finally {
			consumer.close();
		}
	}
}
```

`enable.auto.commit` 을 비활성화하고, 가져온 메시지를 모두 처리한 다음 명시적으로 커밋하는 방식이다. 가져오는 속도는 느려지지만 메시지 손실은 거의 발생하지 않는다.

> ![NOTE]
> 메시지 손실이란, 
> 실제로 토픽에는 메시지가 존재하지만 잘못된 오프셋 커밋으로 인해 컨슈머가 메시지를 가져오지 못하는 경우를 말한다.

메시지가 손실되면 안 되는 중요한 처리 작업은 동기 방식으로 진행하는 것이 좋다. 그러나 메시지 중복 이슈는 피할 수 없다.

#### 3. Asynchronous consume

```java
public class AsynchronousConsumer {
	
	private final KafkaConsumer<String, String> consumer;
	
	public AutoCommitConsumer() {
		// Same as the second example
	}
	
	public void run() {
		try {
			while(true) {
				ConsumerRecords<String, String> records = consumer.poll(1000);
				
				for (ConsumerRecord<String, String> record : records) {
					// handle record
				}
				
				consumer.commitAsync();
			}
		} catch (Exception e) {
			// handle exception
		} finally {
			consumer.close();
		}
	}
}
```

`enable.auto.commit` 을 비활성화하고, `consumer.commitAsync()` 를 사용해 비동기적으로 커밋한다. 

`commitAsync()`는 `commitSync()`와 달리 오프셋 커밋에 실패해도 재시도하지 않는다. 그 이유는 중복 때문인데, 중복되는 메시지의 양이 매우 많아질 수 있기 때문이다.

> ![example]
> 두 번째 오프셋에서 커밋을 비동기로 시도했으나 실패했다고 가정하자. 이후 네 번째 오프셋까지 커밋을 비동기를 시도했으니 실패하고 다섯 번째 오프셋에서 성공했다면 마지막 오프셋은 5이다. 그런데 두 번째 오프셋에서 커밋 재시도에 성공하면 오프셋은 다시 2가 되며 그 사이에 있던 모든 메시지는 중복이 된다.
> 만약 오프셋이 되돌아가는 레코드 수의 단위가 천, 만 등등 커질 경우, 그만큼의 메시지 중복이 발생하게 될 것이다.

### Consumer group

컨슈머 그룹은 여러 개의 컨슈머를 묶는 단위이다. 컨슈머 그룹 내의 컨슈머는 토픽의 특정 파티션과 매핑된다. 



