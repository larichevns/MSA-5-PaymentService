# Реестр транзакций Saga-оркестрации OrchestrPay

## Таблица транзакций

| Транзакция | Описание | Тип | Компенсация |
|-----------|----------|-----|-------------|
| CREATE_PAYMENT | Создание записи о платеже в системе, фиксация намерения провести транзакцию | Компенсируемая | CANCEL_PAYMENT |
| RESERVE_FUNDS | Резервирование (блокировка) средств на счёте клиента | Компенсируемая | RELEASE_FUNDS |
| DEBIT_CUSTOMER | Списание средств со счёта клиента | Компенсируемая | REFUND_CUSTOMER |
| REQUEST_FRAUD_CHECK | Отправка запроса в FraudCheck сервис для проверки транзакции | Повторяемая | - |
| AWAIT_FRAUD_DECISION | Ожидание результата проверки от FraudCheck (автоматической или ручной) | Повторяемая | - |
| PROCESS_MANUAL_REVIEW | Передача транзакции на ручную проверку оператору (если FraudCheck вернул статус "ручная проверка") | Повторяемая | - |
| CREDIT_COUNTERPARTY | Перевод средств контрагенту (необратимая точка - pivot point) | Необратимая | - |
| SEND_SUCCESS_NOTIFICATION | Отправка уведомления клиенту об успешной транзакции | Повторяемая | - |
| SEND_FAILURE_NOTIFICATION | Отправка уведомления клиенту об отклонённой транзакции | Повторяемая | - |
| REFUND_CUSTOMER | Возврат средств клиенту при сбое или отклонении транзакции | Компенсируемая | - |
| RELEASE_FUNDS | Разблокировка зарезервированных средств при отмене до списания | Компенсируемая | - |
| CANCEL_PAYMENT | Отмена записи о платеже, перевод в статус "Cancelled" | Компенсируемая | - |
| NOTIFY_SECURITY | Отправка уведомления в систему безопасности о подозрительной транзакции | Повторяемая | - |

## Пояснения по типам транзакций

### Компенсируемые транзакции
Транзакции, которые можно откатить с помощью компенсирующей операции:
- **CREATE_PAYMENT** - создаёт запись, которую можно отменить через CANCEL_PAYMENT
- **RESERVE_FUNDS** - блокирует средства, которые можно разблокировать через RELEASE_FUNDS
- **DEBIT_CUSTOMER** - списывает средства, которые можно вернуть через REFUND_CUSTOMER

### Необратимые транзакции (Pivot Point)
Транзакции, которые невозможно откатить:
- **CREDIT_COUNTERPARTY** - после перевода средств контрагенту операцию нельзя отменить технически. Это критическая точка в Saga.

### Повторяемые транзакции (Retriable)
Идемпотентные операции, которые можно повторять при сбоях:
- **REQUEST_FRAUD_CHECK** - запрос можно повторить с тем же ID транзакции
- **AWAIT_FRAUD_DECISION** - ожидание результата с учётом таймаутов и cut-off-time
- **PROCESS_MANUAL_REVIEW** - передача на ручную проверку может быть повторена
- **SEND_SUCCESS_NOTIFICATION** / **SEND_FAILURE_NOTIFICATION** - отправка уведомлений идемпотентна
- **NOTIFY_SECURITY** - уведомление системы безопасности можно повторить

## Основные сценарии выполнения

### Успешный сценарий
1. CREATE_PAYMENT
2. RESERVE_FUNDS
3. DEBIT_CUSTOMER
4. REQUEST_FRAUD_CHECK
5. AWAIT_FRAUD_DECISION (approved)
6. CREDIT_COUNTERPARTY (pivot point)
7. SEND_SUCCESS_NOTIFICATION

### Сценарий с ручной проверкой
1. CREATE_PAYMENT
2. RESERVE_FUNDS
3. DEBIT_CUSTOMER
4. REQUEST_FRAUD_CHECK
5. AWAIT_FRAUD_DECISION (manual review required)
6. PROCESS_MANUAL_REVIEW (wait up to 20 min)
7. AWAIT_FRAUD_DECISION (approved/declined)
8a. Если approved: CREDIT_COUNTERPARTY → SEND_SUCCESS_NOTIFICATION
8b. Если declined: REFUND_CUSTOMER → SEND_FAILURE_NOTIFICATION → NOTIFY_SECURITY

### Сценарий с отклонением на этапе FraudCheck (до pivot point)
1. CREATE_PAYMENT
2. RESERVE_FUNDS
3. DEBIT_CUSTOMER
4. REQUEST_FRAUD_CHECK
5. AWAIT_FRAUD_DECISION (declined)
6. **Компенсация:** REFUND_CUSTOMER
7. SEND_FAILURE_NOTIFICATION
8. NOTIFY_SECURITY
9. **Компенсация:** CANCEL_PAYMENT

### Сценарий со сбоем после pivot point
1. CREATE_PAYMENT
2. RESERVE_FUNDS
3. DEBIT_CUSTOMER
4. REQUEST_FRAUD_CHECK
5. AWAIT_FRAUD_DECISION (approved)
6. CREDIT_COUNTERPARTY (pivot point - успешно)
7. SEND_SUCCESS_NOTIFICATION (сбой)
8. **Retry:** SEND_SUCCESS_NOTIFICATION (повтор до успеха)

## Критические моменты

1. **Pivot Point** - CREDIT_COUNTERPARTY является необратимой операцией. После неё компенсация невозможна, поэтому все проверки должны быть выполнены ДО этого момента.

2. **Стоимость ошибки** - лучше отклонить транзакцию, чем провести мошенническую операцию. Поэтому при любых сомнениях транзакция блокируется.

3. **Cut-off time** - если сервис проверок не отвечает в течение установленного времени, транзакция по умолчанию считается разрешённой.

4. **Идемпотентность** - все операции должны быть идемпотентными для корректной работы retry-механизмов.

5. **Зависание средств** - недопустима ситуация, когда средства списаны с клиента, но не переведены контрагенту и не возвращены клиенту.