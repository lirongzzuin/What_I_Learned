# 📌 Redis 정리 및 적용 가이드

## 📝 Redis란?

Redis는 **Remote Dictionary Server**의 약자로, **Key-Value 구조의 인메모리 데이터 저장소**이다.  
모든 데이터를 메모리에 저장하기 때문에 **읽기·쓰기 속도가 매우 빠르며**, 다양한 자료구조를 지원한다.  
주로 **캐시, 세션 저장소, 메시지 브로커, 실시간 데이터 처리 등**의 용도로 사용된다.

---

## 🔍 Redis 주요 특징

| 항목 | 설명 |
|------|------|
| **인메모리 기반** | 모든 데이터를 메모리에 저장하므로 디스크 접근보다 훨씬 빠름 |
| **다양한 자료구조** | String, List, Hash, Set, Sorted Set 등 다양한 타입 지원 |
| **퍼시스턴스 기능** | 데이터를 디스크에 저장하는 AOF/RDB 기능 제공 |
| **Pub/Sub 기능** | 메시지 브로커처럼 사용할 수 있는 publish/subscribe 기능 |
| **원자성 보장** | 하나의 명령은 독립적으로 처리되어 race condition 방지 가능 |

---

## 📂 Redis 활용 사례

| 목적 | 사용 예시 |
|------|-----------|
| **캐싱** | DB 조회 결과를 Redis에 저장해 성능 개선 |
| **세션 관리** | 로그인 세션을 Redis에 저장하고 공유 |
| **속도 제한 (Rate Limiting)** | 특정 사용자 요청 횟수를 제한 |
| **분산 락** | Redisson 등을 통해 동시성 제어 |
| **실시간 상태 저장** | Kafka 컨슈머 처리 상태 저장, 채팅방 사용자 목록 저장 등 |

---

## 🛠️ Redis 적용 순서

### ✅ 1. Redis 서버 실행

- **로컬 설치 (Mac 기준)**
```bash
brew install redis
brew services start redis
```

- **Docker 실행**
```bash
docker run -d -p 6379:6379 --name redis redis
```

- **정상 동작 확인**
```bash
redis-cli ping
# → PONG
```

---

### ✅ 2. 프로젝트에 Redis 의존성 추가

**Spring Boot - build.gradle**
```groovy
implementation 'org.springframework.boot:spring-boot-starter-data-redis'
implementation 'org.springframework.boot:spring-boot-starter-cache'
```

---

### ✅ 3. application.yml 설정

```yaml
spring:
  redis:
    host: localhost
    port: 6379
    timeout: 60000
```

---

### ✅ 4. RedisTemplate 설정 (선택)

```java
@Configuration
public class RedisConfig {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(connectionFactory);
        template.setKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(new GenericJackson2JsonRedisSerializer());
        return template;
    }
}
```

---

## 📦 Redis 사용 예시

### 1. 캐시 저장 및 조회

```java
@Autowired
private RedisTemplate<String, Object> redisTemplate;

public void saveToRedis(String key, Object value) {
    redisTemplate.opsForValue().set(key, value);
}

public Object getFromRedis(String key) {
    return redisTemplate.opsForValue().get(key);
}
```

### 2. 캐시 저장 시 만료 시간 설정

```java
redisTemplate.opsForValue().set("myKey", "value", 10, TimeUnit.MINUTES);
```

### 3. 리스트나 해시 구조 사용도 가능

```java
// 리스트에 추가
redisTemplate.opsForList().rightPush("listKey", "item1");

// 해시 삽입
redisTemplate.opsForHash().put("hashKey", "field", "value");
```

---

## 🔒 실무에서 주의할 점

| 항목 | 설명 |
|------|------|
| **TTL 설정 필수** | 만료 시간을 설정하지 않으면 메모리를 계속 사용하게 됨 |
| **Key 네이밍 전략** | `user:{id}`, `chat:{roomId}:users` 등으로 명확한 규칙 설정 필요 |
| **Null 캐싱 방지** | DB에 값이 없는 경우 Redis에 저장하지 않도록 처리 |
| **보안 설정** | 인증 비밀번호 설정, 포트 접근 제한, TLS 적용 필요 |
| **동시성 처리** | 분산 환경에서는 Redisson 등으로 분산 락 처리 고려 |

---

## 💡 실무 팁

| 상황 | 적용 방식 |
|------|-----------|
| **로그인 세션 공유** | 서버 간 세션 공유를 위해 Redis에 세션 저장 |
| **API 요청 제한** | 사용자 ID 또는 IP 기준으로 요청 횟수 체크 |
| **Kafka 소비 처리 상태 관리** | Redis에 마지막 처리 offset 저장 |
| **JWT 블랙리스트 처리** | 로그아웃 시 해당 토큰을 Redis에 저장하여 만료 전 차단 |

---

## ✅ 정리

- Redis는 **빠른 응답속도**, **다양한 자료구조**, **확장성**을 바탕으로 실무에서 다양한 역할을 수행한다.
- Spring Boot에서는 `spring-boot-starter-data-redis`를 활용해 손쉽게 연동할 수 있다.
- 실무에서는 TTL 설정, key 네이밍 전략, 데이터 정합성 관리 등을 반드시 고려해야 한다.
- Kafka와 함께 사용할 경우 메시지 소비 상태를 Redis에 저장하는 방식으로 안정성을 높일 수 있다.
