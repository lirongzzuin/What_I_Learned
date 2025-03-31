# 📌 한국투자증권 API 기반 실시간 체결 및 수익률 알림 봇 개발 기록

## 📝 개요
한국투자증권 OpenAPI를 활용해, **보유 종목의 수익률 정보를 정기적으로 디스코드로 전송하고, 체결 내역을 실시간으로 감지하는 자동화 알림 봇**을 개발했다.  
초기에는 REST API 기반으로 구현했으며, 이후 웹소켓 기반 실시간 체결 수신 기능도 시도하고 있다.

---

## 🚀 주요 기능 및 흐름

### ✅ 1. 인증 및 토큰 발급
OpenAPI 호출을 위한 `access_token`을 먼저 발급받아야 하며, 별도로 실시간 체결 수신을 위한 `approval_key`도 별도로 발급 필요.

```python
def get_kis_access_token():
    url = "https://openapi.koreainvestment.com:9443/oauth2/tokenP"
    ...
    response = requests.post(url, headers=headers, data=json.dumps(data))
```

```python
def get_approval_key():
    url = "https://openapi.koreainvestment.com:9443/oauth2/Approval"
    ...
```

### ✅ 2. 보유 종목 수익률 계산 및 알림
잔고 정보를 가져와서 보유 수량, 평균 단가, 현재가 등을 활용해 평가금액과 수익률을 계산하고 디스코드에 메시지를 전송한다.

```python
def get_account_profit():
    for item in res["output1"]:
        hold_qty = int(item["hldg_qty"])
        avg_price = float(item["pchs_avg_pric"])
        current_price = float(item["prpr"])
        eval_amt = hold_qty * current_price
        invest_amt = hold_qty * avg_price
        profit_amt = eval_amt - invest_amt
        profit_rate = ((current_price - avg_price) / avg_price) * 100
```

📌 메시지 예시:
```
📌 삼성전자
┗ 수량: 10주 | 평균단가: 60,000원 | 현재가: 63,000원
┗ 평가금액: 630,000원 | 수익금: 30,000원 | 수익률: 5.00%
```

### ✅ 3. 체결 내역 조회 및 중복 방지 처리
10초 간격으로 체결 내역을 조회하고, 이전에 감지된 주문번호를 기준으로 중복 알림을 방지하는 방식으로 구현.

```python
last_order_ids = set()
def check_and_notify_order():
    orders = get_order_list()
    for order in orders:
        odno = order.get("odno")
        if odno not in last_order_ids:
            type_str = "매수" if order.get("sll_buy_dvsn_cd") == "02" else "매도"
            msg = f"[{type_str} 체결 알림] ..."
            send_discord_message(msg)
            last_order_ids.add(odno)
```

### ✅ 4. 디스코드 메시지 전송
```python
def send_discord_message(content):
    requests.post(DISCORD_WEBHOOK_URL, json={"content": content})
```

---

## 🧪 웹소켓 기반 실시간 체결 수신 시도
현재 REST 방식은 10초 간격으로 체결 상태를 조회하지만, 실시간으로 알림을 받기 위해 웹소켓 연결을 시도 중이다.

```python
def on_message(ws, message):
    data = json.loads(message)
    ... # 체결 내용 파싱 및 알림 전송

def start_websocket():
    ws = websocket.WebSocketApp(...)
    ws.run_forever()
```

- 연결은 정상적으로 이루어지나, 체결 알림 메시지가 수신되지 않아 원인 분석 중
- `approval_key`, `tr_id`, `tr_key` 등의 조합이 정확한지 점검 필요
- SSL 설정 또는 헤더 정보 부족 가능성도 의심

---

## 📌 작업하면서 배운 점
- **API 보안 구조에 대한 이해가 필수적이다.**
  - REST와 웹소켓 모두에서 인증 흐름과 파라미터 구성이 상이함
  - 인증 실패 시 디버깅을 위한 로그 기록이 중요

- **웹소켓 통신에서의 이벤트 흐름을 정확히 설계해야 한다.**
  - `on_open`, `on_message`, `on_error`, `on_close` 등 이벤트 핸들링 구조 숙지 필요

- **주문 체결 감지를 위한 고유 식별자 추적이 중요하다.**
  - 주문번호(`odno`) 기반 중복 필터링 처리로 불필요한 알림 방지

- **디스코드 웹훅은 빠른 알림 전달에 효과적이며, 메시지 포맷을 꾸며주는 것도 중요하다.**

---

## 🎯 향후 개선 방향
- 웹소켓 실시간 체결 수신 로직 완성 및 안정화
- 실패 시 fallback 방식으로 REST 조회 유지
- 체결 뿐 아니라 미체결, 예약 주문 알림까지 확장
- 수익률 리포트에 기술적 지표(RSI, 이동평균선 등) 추가 검토
- 주기적 실행 로직을 스케줄러 대신 백그라운드 서비스 형태로 개선

---