# 📌 Bootstrap Table 필터링 개선 및 코인 가격 알림 기능 업데이트

## 📝 개요
### 1️⃣ **Bootstrap Table 필터링 기능 개선**
Bootstrap Table에서 `solution.name`을 직접 사용했을 때 필터링 오류가 발생하는 문제가 있었다. 이를 해결하기 위해 `solution_name`을 별도로 분리하여 테이블 필터링이 정상적으로 작동하도록 수정했다.

### 2️⃣ **코인 가격 실시간 알림 기능 개선**
기존에는 설정한 가격에 도달했을 때만 알림을 보내는 방식이었으나, 이번 개선을 통해 다음 기능을 추가했다:
- **알림 시작 및 종료 메시지 추가**: 모니터링이 정상 작동 중인지 확인 가능
- **1시간마다 가격 요약 메시지 전송**: 전체적인 가격 흐름을 확인할 수 있도록 개선

---

## 🚀 구현 내용

### 🔹 1. Bootstrap Table 필터링 문제 해결
#### ✅ 문제점
- 기존에는 `solution.name` 값을 `solution_name` 컬럼에 직접 매핑하여 사용했음.
- Bootstrap Table의 필터링 기능을 적용하는 과정에서 데이터가 정상적으로 매핑되지 않음.

#### ✅ 해결 방법
- `solution_name`을 `solution` 객체에서 분리하여 별도 필드로 추가.
- 데이터 로드 시, `solution.name` 값을 `solution_name`으로 매핑하여 사용.

```javascript
appinfoItem.solution_name = appinfoItem.solution ? appinfoItem.solution.name : "N/A";
```

---

### 🔹 2. 코인 가격 실시간 알림 기능 개선
#### ✅ 기존 기능
- 사용자가 설정한 가격(상한/하한)에 도달하면 Slack 알림을 보내는 방식

#### ✅ 추가된 기능
- **모니터링 시작 및 종료 시 알림** → 스크립트 정상 작동 확인 가능
- **1시간마다 가격 요약 알림** → 전체적인 시장 흐름을 한눈에 파악 가능

#### ✅ 개선된 코드
```python
# 모니터링 시작 알림
send_slack_message("📢 코인 가격 모니터링이 시작되었습니다. (실시간 감지 + 1시간 간격 요약)")

# 종료 신호 처리
signal.signal(signal.SIGINT, signal_handler)  # Ctrl + C
signal.signal(signal.SIGTERM, signal_handler) # 시스템 종료

def monitor_prices():
    last_summary_time = time()
    while running:
        summary_message = "📊 **현재 코인 가격 요약**\n"
        for alert in crypto_alerts:
            symbol = alert["symbol"]
            current_price = get_crypto_price(symbol)
            summary_message += f"- {symbol}: {current_price} USDT\n"
        
        # 1시간마다 가격 요약 전송
        if time() - last_summary_time >= 3600:
            send_slack_message(summary_message)
            last_summary_time = time()
        
        sleep(5)  # 5초마다 가격 확인
```

---

## 📌 작업하면서 배운 점
- **데이터 필터링을 위해 컬럼 값을 명확하게 지정하는 것이 중요하다.**
  - 기존 `solution.name`을 직접 참조하면 Bootstrap Table에서 필터링 시 오류가 발생할 수 있음.
  - `solution_name`을 별도로 분리하여 필터링 기능과 호환되도록 처리.
  
- **JSON 데이터를 다룰 때 예외 처리가 필요하다.**
  - `define` 값이 비어 있을 경우 예외가 발생하므로, 비어 있는 경우에 대한 예외 처리를 추가함.
  
- **실시간 모니터링 시스템에서는 정상 작동 여부를 알 수 있도록 피드백을 주는 것이 중요하다.**
  - 알림이 작동 중인지 확인하기 위해 시작/종료 메시지를 추가하고, 1시간마다 상태를 점검하는 기능을 추가함.
  
- **주기적인 데이터 요약을 제공하는 것이 사용자 경험 개선에 도움이 된다.**
  - 1시간마다 가격 요약 메시지를 보내도록 설정하여 사용자에게 지속적인 정보를 제공.

---

## 🎯 향후 개선 방향
- **Bootstrap Table 필터링 기능을 더욱 확장하여 검색 편의성 향상**
- **코인 가격 감시 기능을 다양한 채널 (이메일, Discord 등)로 확장**
- **데이터 매핑 시 일관된 방식 적용을 위한 공통 함수 도입**
- **예외 처리 로직을 더욱 개선하여 에러 발생 시 명확한 로그 출력**

