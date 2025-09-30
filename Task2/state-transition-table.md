# Таблица переходов State Machine для платёжной системы OrchestrPay

## Таблица переходов состояний

| Исходное состояние | Переходное состояние | Событие |
|-------------------|---------------------|---------|
| - | CREATED | PaymentInitiated |
| CREATED | FUNDS_RESERVED | FundsReserved |
| CREATED | CANCELLED | PaymentCreationFailed |
| FUNDS_RESERVED | FUNDS_DEBITED | FundsDebited |
| FUNDS_RESERVED | CANCELLED | FundsReservationFailed |
| FUNDS_DEBITED | FRAUD_CHECK_PENDING | FraudCheckRequested |
| FUNDS_DEBITED | REFUNDING | DebitFailed |
| FRAUD_CHECK_PENDING | FRAUD_CHECK_APPROVED | FraudCheckApproved |
| FRAUD_CHECK_PENDING | FRAUD_CHECK_DECLINED | FraudCheckDeclined |
| FRAUD_CHECK_PENDING | MANUAL_REVIEW_REQUIRED | ManualReviewRequested |
| FRAUD_CHECK_PENDING | FRAUD_CHECK_APPROVED | CutOffTimeExpired |
| FRAUD_CHECK_PENDING | FRAUD_CHECK_PENDING | FraudCheckRetry |
| MANUAL_REVIEW_REQUIRED | FRAUD_CHECK_APPROVED | ManualReviewApproved |
| MANUAL_REVIEW_REQUIRED | FRAUD_CHECK_DECLINED | ManualReviewDeclined |
| MANUAL_REVIEW_REQUIRED | FRAUD_CHECK_APPROVED | ManualReviewCutOffExpired |
| FRAUD_CHECK_APPROVED | COUNTERPARTY_CREDITED | CounterpartyCredited |
| FRAUD_CHECK_APPROVED | REFUNDING | CounterpartyCreditFailed |
| FRAUD_CHECK_DECLINED | REFUNDING | RefundInitiated |
| COUNTERPARTY_CREDITED | NOTIFICATION_SENT | SuccessNotificationSent |
| COUNTERPARTY_CREDITED | COUNTERPARTY_CREDITED | NotificationRetry |
| REFUNDING | REFUNDED | RefundCompleted |
| REFUNDING | REFUND_FAILED | RefundFailed |
| REFUNDED | NOTIFICATION_SENT | FailureNotificationSent |
| REFUNDED | REFUNDED | NotificationRetry |
| NOTIFICATION_SENT | COMPLETED_SUCCESS | PaymentCompleted |
| NOTIFICATION_SENT | COMPLETED_DECLINED | PaymentDeclined |
| REFUND_FAILED | MANUAL_INTERVENTION_REQUIRED | ManualInterventionTriggered |
| FRAUD_CHECK_DECLINED | SECURITY_NOTIFIED | SecurityNotificationSent |
| SECURITY_NOTIFIED | REFUNDING | RefundInitiated |
| CANCELLED | FUNDS_RELEASED | FundsReleased |
| FUNDS_RELEASED | COMPLETED_CANCELLED | PaymentCancelled |

## Описание состояний

### Основные состояния обработки

1. **CREATED** - Платёж создан, запись в системе зафиксирована
2. **FUNDS_RESERVED** - Средства заблокированы на счёте клиента
3. **FUNDS_DEBITED** - Средства списаны со счёта клиента
4. **FRAUD_CHECK_PENDING** - Транзакция проверяется в FraudCheck сервисе
5. **MANUAL_REVIEW_REQUIRED** - Транзакция направлена на ручную проверку оператором
6. **FRAUD_CHECK_APPROVED** - Проверка пройдена успешно
7. **FRAUD_CHECK_DECLINED** - Проверка не пройдена, транзакция отклонена
8. **COUNTERPARTY_CREDITED** - Средства переведены контрагенту (Pivot Point)
9. **NOTIFICATION_SENT** - Уведомление отправлено клиенту

### Состояния компенсации

10. **REFUNDING** - Выполняется возврат средств клиенту
11. **REFUNDED** - Возврат средств завершён
12. **REFUND_FAILED** - Возврат средств не удался
13. **SECURITY_NOTIFIED** - Система безопасности уведомлена о подозрительной транзакции
14. **CANCELLED** - Платёж отменён до списания средств
15. **FUNDS_RELEASED** - Зарезервированные средства разблокированы

### Финальные состояния

16. **COMPLETED_SUCCESS** - Транзакция успешно завершена
17. **COMPLETED_DECLINED** - Транзакция завершена с отклонением
18. **COMPLETED_CANCELLED** - Транзакция отменена
19. **MANUAL_INTERVENTION_REQUIRED** - Требуется ручное вмешательство (критическая ошибка)

## Описание событий

### События инициации и обработки

- **PaymentInitiated** - Пользователь инициировал платёж
- **FundsReserved** - Средства успешно зарезервированы
- **FundsDebited** - Средства успешно списаны
- **FraudCheckRequested** - Запрос на проверку отправлен в FraudCheck
- **FraudCheckRetry** - Повторная попытка проверки при временной недоступности

### События результатов проверки

- **FraudCheckApproved** - FraudCheck одобрил транзакцию
- **FraudCheckDeclined** - FraudCheck отклонил транзакцию
- **ManualReviewRequested** - FraudCheck требует ручной проверки
- **CutOffTimeExpired** - Истёк таймаут ожидания ответа от FraudCheck, транзакция одобрена по умолчанию

### События ручной проверки

- **ManualReviewApproved** - Оператор одобрил транзакцию
- **ManualReviewDeclined** - Оператор отклонил транзакцию
- **ManualReviewCutOffExpired** - Истекли 20 минут ожидания решения оператора, транзакция одобрена по умолчанию

### События завершения основного потока

- **CounterpartyCredited** - Средства успешно переведены контрагенту
- **SuccessNotificationSent** - Уведомление об успехе отправлено клиенту
- **PaymentCompleted** - Платёж успешно завершён

### События компенсации

- **RefundInitiated** - Инициирован возврат средств
- **RefundCompleted** - Возврат средств выполнен
- **RefundFailed** - Возврат средств не удался
- **FailureNotificationSent** - Уведомление об отклонении отправлено клиенту
- **SecurityNotificationSent** - Уведомление системе безопасности отправлено
- **PaymentDeclined** - Платёж завершён с отклонением

### События отмены

- **PaymentCreationFailed** - Не удалось создать платёж
- **FundsReservationFailed** - Не удалось зарезервировать средства
- **DebitFailed** - Не удалось списать средства
- **CounterpartyCreditFailed** - Не удалось перевести средства контрагенту
- **FundsReleased** - Средства разблокированы
- **PaymentCancelled** - Платёж отменён

### События повторов

- **NotificationRetry** - Повтор отправки уведомления
- **ManualInterventionTriggered** - Требуется ручное вмешательство службы поддержки

## Детализация ключевых сценариев

### 1. Успешный сценарий (Happy Path)

```
CREATED
  → (FundsReserved) → FUNDS_RESERVED
  → (FundsDebited) → FUNDS_DEBITED
  → (FraudCheckRequested) → FRAUD_CHECK_PENDING
  → (FraudCheckApproved) → FRAUD_CHECK_APPROVED
  → (CounterpartyCredited) → COUNTERPARTY_CREDITED [PIVOT POINT]
  → (SuccessNotificationSent) → NOTIFICATION_SENT
  → (PaymentCompleted) → COMPLETED_SUCCESS
```

### 2. Сценарий с ручной проверкой и одобрением

```
CREATED
  → (FundsReserved) → FUNDS_RESERVED
  → (FundsDebited) → FUNDS_DEBITED
  → (FraudCheckRequested) → FRAUD_CHECK_PENDING
  → (ManualReviewRequested) → MANUAL_REVIEW_REQUIRED
  → (ManualReviewApproved) → FRAUD_CHECK_APPROVED
  → (CounterpartyCredited) → COUNTERPARTY_CREDITED [PIVOT POINT]
  → (SuccessNotificationSent) → NOTIFICATION_SENT
  → (PaymentCompleted) → COMPLETED_SUCCESS
```

### 3. Сценарий с ручной проверкой и отклонением

```
CREATED
  → (FundsReserved) → FUNDS_RESERVED
  → (FundsDebited) → FUNDS_DEBITED
  → (FraudCheckRequested) → FRAUD_CHECK_PENDING
  → (ManualReviewRequested) → MANUAL_REVIEW_REQUIRED
  → (ManualReviewDeclined) → FRAUD_CHECK_DECLINED
  → (SecurityNotificationSent) → SECURITY_NOTIFIED
  → (RefundInitiated) → REFUNDING
  → (RefundCompleted) → REFUNDED
  → (FailureNotificationSent) → NOTIFICATION_SENT
  → (PaymentDeclined) → COMPLETED_DECLINED
```

### 4. Сценарий с отклонением FraudCheck

```
CREATED
  → (FundsReserved) → FUNDS_RESERVED
  → (FundsDebited) → FUNDS_DEBITED
  → (FraudCheckRequested) → FRAUD_CHECK_PENDING
  → (FraudCheckDeclined) → FRAUD_CHECK_DECLINED
  → (SecurityNotificationSent) → SECURITY_NOTIFIED
  → (RefundInitiated) → REFUNDING
  → (RefundCompleted) → REFUNDED
  → (FailureNotificationSent) → NOTIFICATION_SENT
  → (PaymentDeclined) → COMPLETED_DECLINED
```

### 5. Сценарий с cut-off time (автоматическое одобрение)

```
CREATED
  → (FundsReserved) → FUNDS_RESERVED
  → (FundsDebited) → FUNDS_DEBITED
  → (FraudCheckRequested) → FRAUD_CHECK_PENDING
  → [Timeout] → (CutOffTimeExpired) → FRAUD_CHECK_APPROVED
  → (CounterpartyCredited) → COUNTERPARTY_CREDITED [PIVOT POINT]
  → (SuccessNotificationSent) → NOTIFICATION_SENT
  → (PaymentCompleted) → COMPLETED_SUCCESS
```

### 6. Сценарий с cut-off time для ручной проверки

```
CREATED
  → (FundsReserved) → FUNDS_RESERVED
  → (FundsDebited) → FUNDS_DEBITED
  → (FraudCheckRequested) → FRAUD_CHECK_PENDING
  → (ManualReviewRequested) → MANUAL_REVIEW_REQUIRED
  → [20 minutes timeout] → (ManualReviewCutOffExpired) → FRAUD_CHECK_APPROVED
  → (CounterpartyCredited) → COUNTERPARTY_CREDITED [PIVOT POINT]
  → (SuccessNotificationSent) → NOTIFICATION_SENT
  → (PaymentCompleted) → COMPLETED_SUCCESS
```

### 7. Сценарий с ошибкой перевода контрагенту (до pivot point)

```
CREATED
  → (FundsReserved) → FUNDS_RESERVED
  → (FundsDebited) → FUNDS_DEBITED
  → (FraudCheckRequested) → FRAUD_CHECK_PENDING
  → (FraudCheckApproved) → FRAUD_CHECK_APPROVED
  → (CounterpartyCreditFailed) → REFUNDING
  → (RefundCompleted) → REFUNDED
  → (FailureNotificationSent) → NOTIFICATION_SENT
  → (PaymentDeclined) → COMPLETED_DECLINED
```

### 8. Сценарий с ошибкой возврата (критическая ситуация)

```
CREATED
  → (FundsReserved) → FUNDS_RESERVED
  → (FundsDebited) → FUNDS_DEBITED
  → (FraudCheckRequested) → FRAUD_CHECK_PENDING
  → (FraudCheckDeclined) → FRAUD_CHECK_DECLINED
  → (SecurityNotificationSent) → SECURITY_NOTIFIED
  → (RefundInitiated) → REFUNDING
  → (RefundFailed) → REFUND_FAILED
  → (ManualInterventionTriggered) → MANUAL_INTERVENTION_REQUIRED
```

### 9. Сценарий с повторной отправкой уведомления

```
...
  → COUNTERPARTY_CREDITED [PIVOT POINT]
  → (SuccessNotificationSent - failed) → COUNTERPARTY_CREDITED
  → (NotificationRetry) → COUNTERPARTY_CREDITED
  → (SuccessNotificationSent - success) → NOTIFICATION_SENT
  → (PaymentCompleted) → COMPLETED_SUCCESS
```

### 10. Сценарий отмены на ранней стадии

```
CREATED
  → (FundsReserved) → FUNDS_RESERVED
  → (FundsReservationFailed) → CANCELLED
  → (FundsReleased) → FUNDS_RELEASED
  → (PaymentCancelled) → COMPLETED_CANCELLED
```

## Важные особенности State Machine

### Pivot Point - точка невозврата
Состояние **COUNTERPARTY_CREDITED** является критической точкой (Pivot Point). После перевода средств контрагенту:
- Компенсация через возврат невозможна
- Все проверки должны быть выполнены ДО этого состояния
- При ошибках отправки уведомлений после Pivot Point используется retry без компенсации

### Идемпотентность
Все события должны быть идемпотентными для корректной работы retry-механизмов:
- Повторный FraudCheckRetry не создаёт новую проверку
- NotificationRetry отправляет то же уведомление с тем же ID

### Таймауты и Cut-off
- **FraudCheck Cut-off**: если сервис не отвечает → автоматическое одобрение (CutOffTimeExpired)
- **Manual Review Cut-off**: если оператор не принял решение за 20 минут → автоматическое одобрение (ManualReviewCutOffExpired)

### Критические состояния
- **MANUAL_INTERVENTION_REQUIRED** - требует участия службы поддержки для ручного разрешения ситуации (например, когда не удалось выполнить возврат средств)
- **REFUND_FAILED** - недопустимая ситуация "зависших" средств, требует немедленного реагирования

### Финальные состояния
Финальные состояния являются терминальными, из них нет переходов:
- **COMPLETED_SUCCESS** - успешное завершение
- **COMPLETED_DECLINED** - завершение с отклонением
- **COMPLETED_CANCELLED** - завершение с отменой
- **MANUAL_INTERVENTION_REQUIRED** - требует ручного вмешательства (технически финальное для автоматического процесса)