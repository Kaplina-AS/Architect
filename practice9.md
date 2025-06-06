## Практическая работа 7

### Интеграции

1. Inventory Service -> Shipping Service (HTTP/REST).
  После получения события OrderCreated, Inventory Service отправляет запрос в Shipping Service для подготовки к отправке товара.

2. Order Service -> Inventory Service (Message Queue).
  Order Service публикует событие OrderCreated в очередь, которую прослушивает Inventory Service для резервирования товара.

1. Интеграция Inventory Service -> Shipping Service (HTTP/REST):

#### Сценарии возникновения ошибок и способ их обработки вашей системой.

##### Inventory Service -> Shipping Service (HTTP/REST)

**Сценарий**:  Inventory Service получает событие OrderCreated, после чего должен уведомить Shipping Service о необходимости подготовить отправку товара.

**Возможные ошибки:**

- Shipping Service Unavailable (503 Service Unavailable): Shipping Service временно недоступен.
- Invalid Order Data (400 Bad Request):  Данные в запросе (например, адрес доставки) невалидны.
- Timeout (504 Gateway Timeout): Inventory Service не получает ответа от Shipping Service в течение определенного времени.

**Обработка ошибок и отказоустойчивость:**

- Retry with Exponential Backoff:  Inventory Service пытается повторить запрос к Shipping Service несколько раз с экспоненциально увеличивающимся интервалом между попытками.  Это позволяет Shipping Service восстановиться, не перегружая его повторными запросами сразу после сбоя.
- Circuit Breaker: Если Shipping Service недоступен в течение определенного времени или количество ошибок превышает заданный порог, Inventory Service "размыкает цепь" (circuit breaker).  В разомкнутом состоянии Inventory Service не пытается отправлять запросы в Shipping Service, а возвращает заранее определенный ответ (например, ставит заказ в статус "Ожидание отправки").  Периодически (например, каждые 5 минут) Inventory Service пытается "прощупать" Shipping Service, отправляя тестовый запрос.  Если запрос успешен, цепь "замыкается", и Inventory Service возобновляет отправку уведомлений об OrderCreated.
- Fallback: Если все попытки повтора и circuit breaker не помогли, Inventory Service переводит заказ в состояние "Ожидает отправки вручную" и отправляет уведомление оператору для ручной обработки.

>![Результат 1](/seq_rest.png)

##### Order Service -> Inventory Service (Message Queue - OrderCreated Event)


**Сценарий**: Order Service создает заказ и публикует событие OrderCreated в очередь. Inventory Service подписывается на эту очередь и резервирует товар при получении события.

**Возможные ошибки:**

- Message Queue Unavailable: Очередь сообщений (например, RabbitMQ, Kafka) временно недоступна.
- Invalid Message Format: Сообщение OrderCreated имеет неверный формат.
- Insufficient Inventory:  В Inventory Service недостаточно товара для резервирования.

**Обработка ошибок:**

- Message Queue Retries:  Большинство брокеров сообщений поддерживают автоматические повторные попытки доставки сообщения в случае ошибки. Order Service и Inventory Service конфигурируются с политикой повторных попыток доставки сообщения с определенным интервалом.
- Dead Letter Queue (DLQ): Если сообщение не может быть обработано после нескольких повторных попыток, оно перемещается в "мертвую очередь" (Dead Letter Queue). Это позволяет сохранить проблемные сообщения для дальнейшего анализа и ручной обработки.
- Idempotency: Inventory Service должен быть идемпотентным при обработке события OrderCreated. Это означает, что повторная обработка одного и того же события не должна приводить к дублированию резервирования товара. Inventory Service хранит историю обработанных orderId.


>![Результат 2](/seq_mq.png)

##### YAML

```
openapi: 3.0.0
info:
  title: Shipping Service API
  version: v1

paths:
  /prepare-shipping:
    post:
      summary: Prepare shipping for an order
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                orderId:
                  type: string
                  description: The ID of the order
                shippingAddress:
                  type: string
                  description: The shipping address
      responses:
        '200':
          description: Shipping preparation initiated successfully
        '400':
          description: Invalid order data (e.g., missing shipping address)
          content:
            application/json:
              schema:
                type: object
                properties:
                  error:
                    type: string
                    example: "Invalid shipping address"
        '503':
          description: Shipping Service unavailable
          content:
            application/json:
              schema:
                type: object
                properties:
                  error:
                    type: string
                    example: "Shipping service temporarily unavailable"
        '500':
          description: Internal Server Error
          content:
            application/json:
              schema:
                type: object
                properties:
                  error:
                    type: string
                    example: "Internal Server Error"
```
