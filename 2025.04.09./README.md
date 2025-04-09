# 📌 Kafka 적용 가이드

## 📝 Kafka란?

Kafka는 **실시간 데이터 스트리밍을 처리하기 위한 분산 메시징 플랫폼**이다. 대용량 데이터를 빠르게 전달하고, 메시지를 안정적으로 처리할 수 있도록 설계되어 있다.

> 핵심 개념  
- **Producer**: 데이터를 Kafka로 전송하는 주체  
- **Broker**: 메시지를 저장하고 중개하는 Kafka 서버  
- **Consumer**: 메시지를 읽어가는 주체  
- **Topic**: 메시지를 구분하는 논리적 공간 (게시판처럼 분리됨)

---

## ⚙️ Kafka가 사용되는 이유

Kafka는 다음과 같은 상황에서 유용하게 쓰인다:

- 마이크로서비스 간 **비동기 통신**
- 사용자 활동 로그, 시스템 로그 등의 **실시간 수집 및 분석**
- **이벤트 기반 아키텍처**에서의 메시지 전달
- **결제/주문/알림** 시스템처럼 데이터 순서가 중요하거나 지연 없이 처리돼야 하는 경우

---

## 📂 Kafka 적용 순서 및 실무 흐름

Kafka를 프로젝트에 적용하기 위해서는 다음 단계대로 구성하면 된다.

---

### ✅ 1. Kafka 설치 및 실행

Kafka는 로컬에서 테스트 가능하며, Docker로도 쉽게 실행할 수 있다. 여기서는 기본 설치 기준으로 설명한다.

```bash
# Kafka 다운로드 및 압축 해제
wget https://downloads.apache.org/kafka/3.6.0/kafka_2.13-3.6.0.tgz
tar -xzf kafka_2.13-3.6.0.tgz
cd kafka_2.13-3.6.0

# Zookeeper 실행
bin/zookeeper-server-start.sh config/zookeeper.properties

# Kafka Broker 실행
bin/kafka-server-start.sh config/server.properties
```

---

### ✅ 2. Topic 생성

Kafka는 데이터를 저장할 논리 공간인 **Topic**을 기준으로 메시지를 주고받는다.

```bash
bin/kafka-topics.sh --create \
  --bootstrap-server localhost:9092 \
  --replication-factor 1 \
  --partitions 1 \
  --topic test-topic
```

---

### ✅ 3. Spring Boot 프로젝트에 Kafka 적용

#### 1) 의존성 추가 (Gradle 기준)

```groovy
implementation 'org.springframework.kafka:spring-kafka'
```

#### 2) application.yml 설정

```yaml
spring:
  kafka:
    bootstrap-servers: localhost:9092
    consumer:
      group-id: my-group
      auto-offset-reset: earliest
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
```

---

### ✅ 4. Producer/Consumer 구현

#### Kafka 메시지 전송 (Producer)

```java
@Service
@RequiredArgsConstructor
public class KafkaMessageProducer {
    private final KafkaTemplate<String, String> kafkaTemplate;

    public void send(String topic, String message) {
        kafkaTemplate.send(topic, message);
    }
}
```

#### Kafka 메시지 수신 (Consumer)

```java
@Component
@Slf4j
public class KafkaMessageConsumer {

    @KafkaListener(topics = "test-topic", groupId = "my-group")
    public void listen(String message) {
        log.info("Kafka 메시지 수신: {}", message);
        // 처리 로직
    }
}
```

---

### ✅ 5. 메시지 전송 및 확인

```java
kafkaMessageProducer.send("test-topic", "Kafka 메시지 테스트");
```

Consumer 로그에서 메시지 수신 확인:
```
Kafka 메시지 수신: Kafka 메시지 테스트
```

---

## 🔎 실무에서 주의할 점

| 항목 | 설명 |
|------|------|
| Topic 관리 | 비즈니스 단위별로 Topic을 구분하여 관리해야 유지보수에 유리함 |
| 메시지 포맷 | 대부분 JSON을 사용하며, DTO 직렬화 필요 |
| 중복 방지 | 멱등성을 위한 메시지 키 사용 필요 |
| 장애 대비 | Kafka는 메시지를 디스크에 저장함 → 복구 가능 |
| 테스트 | 개발 환경에서 Kafka 서버가 없을 경우 Docker 또는 Embedded Kafka 사용 가능 |

---

## 💡 확장 시 고려할 내용

Kafka는 다양한 시스템과 조합하여 활용할 수 있다:

- **Kafka + Redis**: 실시간 캐시 저장
- **Kafka + Elasticsearch**: 로그 및 이벤트 검색용 저장소
- **Kafka + Spring Cloud Stream**: 마이크로서비스 간 이벤트 기반 아키텍처 구축
- **Kafka Connect / Schema Registry**: 외부 시스템과의 연동 및 스키마 관리

---

## 🧾 요약

- Kafka는 **대용량, 고속, 실시간 데이터 전달**을 위한 메시지 브로커 시스템이다.
- Producer → Topic → Consumer의 구조로 동작하며, Spring Boot에서 쉽게 연동 가능하다.
- 설정과 흐름을 이해하면 다양한 시스템에 유연하게 적용할 수 있다.
- 실무에서는 **Topic 설계, 장애 처리, 메시지 포맷 정합성**을 반드시 고려해야 한다.
