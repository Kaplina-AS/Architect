## Практическая работа 7
### Проектирование API

1. Пользователи:

- GET/users: Получение списка пользователей .
- GET/users/{user_id}: Получение информации о конкретном пользователе.
- POST/users: Создание нового пользователя.
- PUT/users/{user_id}: Обновление информации о пользователе.
- DELETE/users/{user_id}: Удаление пользователя.
- POST/addresses: Создание нового адреса.
- GET/addresses/{address_id}: Получение информации об адресе
- PATCH/addresses/{address_id}: Обновление информации об адресе

2. Каталог товаров:

- GET/products: Получение списка товаров .
- GET/products/{product_id}: Получение информации о конкретном товаре.
- POST/products: Создание нового товара.
- PUT/products/{product_id}: Обновление информации о товаре.
- DELETE/products/{product_id}: Удаление товара.
- GET/categories: Получение списка категорий.
- GET/categories/{category_id}: Получение информации о категории.
- GET/brands: Получение списка брендов.
- GET/brands/{brand_id}: Получение информации о бренде.
- GET/materials: Получение списка материалов.
- GET/materials/{material_id}: Получение информации о материале.

3. Заказы:

- GET/orders: Получение списка заказов (с фильтрацией, сортировкой и пагинацией).
- GET/orders/{order_id}: Получение информации о конкретном заказе.
- POST/orders: Создание нового заказа.
- PUT/orders/{order_id}: Обновление информации о заказе (например, статус).
- POST/order-items: Создание нового элемента заказа.
- GET/order-items/{order_item_id}: Получение информации о элементе заказа
- POST/shipping-addresses: Создание адреса доставки
- GET/shipping-addresses/{shipping_address_id}: Получение информации об адресе доставки
- POST/billing-addresses: Создание платежного адреса
- GET/billing-addresses/{billing_address_id}: Получение информации о платежном адресе

4. Управление производством:

- GET/product-orders: Получение списка производственных заказов.
- GET/product-orders/{product_order_id}: Получение информации о производственном заказе.
- POST/product-orders: Создание производственного заказа.
- PUT//product-orders/{product_order_id}: Обновление производственного заказа.
- GETmaterial-reqs: Получение списка потребностей в материалах.
- GET/material-reqs/{material_req_id}: Получение информации о потребности в материалах.
- GET/operations: Получение списка операций.
- GET/operations/{operation_id}: Получение информации об операции.
- GET/machines: Получение списка станков.
- GET/machines/{machine_id}: Получение информации о станке.

5. Управление складом:

- GET/inventory-items: Получение списка товаров на складе.
- GET/inventory-items/{inventory_item_id}: Получение информации о товаре на складе.
- GET/batch-inventories: Получение списка партий товаров.
- GET/batch-inventories/{batch_inventory_id}: Получение информации о партии товаров.
- GET/inventory-transactions: Получение списка транзакций склада.
- GET/inventory-transactions/{inventory_transaction_id}: Получение информации о транзакции склада.

6. Платежи:

- GET/payments: Получение списка платежей (с фильтрацией по order_id, user_id, статусу).
- GET/payments/{payment_id}: Получение информации о конкретном платеже.
- POST/payments: Создание нового платежа.
- PATCH/payments/{payment_id}: Обновление статуса платежа.

7. Управление логистикой:

- GET/deliveries: Получение списка доставок (с фильтрацией по order_id, статусу).
- GET/deliveries/{delivery_id}: Получение информации о конкретной доставке.
- POST/deliveries: Создание новой доставки.
- PUT/deliveries/{delivery_id}: Обновление информации о доставке (например, статус, tracking_number).
- GET/delivery-events: Получение списка событий доставки для конкретной доставки.
- POST/delivery-events: Создание нового события доставки.

### OpenAPI 

#### API1 - Создание заказа

**Метод**: POST
**URI**: /orders
**Описание**: Создание нового заказа.  

```
openapi: 3.0.0
info:
  title: Order Service API
  version: 1.0.0
paths:
  /orders:
    post:
      summary: Create a new order
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                user_id:
                  type: string
                  format: uuid
                  description: ID of the user placing the order
                order_date:
                  type: string
                  format: date-time
                  description: Date and time the order was placed
                total_amount:
                  type: number
                  format: float
                  description: Total amount of the order
                shipping_address_id:
                  type: string
                  format: uuid
                  description: ID of the shipping address
                billing_address_id:
                  type: string
                  format: uuid
                  description: ID of the billing address
                payment_method:
                  type: string
                  enum: [credit_card, paypal, bank_transfer]
                  description: Payment method used for the order
                discount:
                  type: number
                  format: float
                  description: Discount applied to the order
                order_items:
                  type: array
                  items:
                    type: object
                    properties:
                      product_id:
                        type: string
                        format: uuid
                        description: ID of the product
                      quantity:
                        type: integer
                        description: Quantity of the product ordered
                      price:
                        type: number
                        format: float
                        description: Price of the product

      responses:
        '201':
          description: Order created successfully
          content:
            application/json:
              schema:
                type: object
                properties:
                  order_id:
                    type: string
                    format: uuid
                    description: ID of the newly created order
                  status:
                    type: string
                    description: Status of the order (e.g., pending)
        '400':
          description: Bad request (e.g., invalid data)
          content:
            application/json:
              schema:
                type: object
                properties:
                  error:
                    type: string
                    description: Error message
        '500':
          description: Internal server error
          content:
            application/json:
              schema:
                type: object
                properties:
                  error:
                    type: string
                    description: Error message

```

**JSON-запрос**
```
{
  "user_id": "a1b2c3d4-e5f6-7890-1234-567890abcdef",
  "order_date": "2024-01-01T12:00:00Z",
  "total_amount": 150.00,
  "shipping_address_id": "b1c2d3e4-f5a6-8790-2345-67890abcdef1",
  "billing_address_id": "c1d2e3f4-a5b6-9870-3456-7890abcdef21",
  "payment_method": "credit_card",
  "discount": 0.00,
  "order_items": [
    {
      "product_id": "d1e2f3a4-b5c6-0987-4567-890abcdef321",
      "quantity": 2,
      "price": 50.00
    },
    {
      "product_id": "e1f2a3b4-c5d6-1987-5678-90abcdef432",
      "quantity": 1,
      "price": 50.00
    }
  ]
}
```
Описание атрибутного состава:

| Атрибут | Описание | Поле в БД |
|-|-|-|
| user_id | Уникальный идентификатор пользователя | USER.user_id |
| order_date | Дата заказа | ORDER.order_date |
| total_amount | Описание товара | ORDER.total_amount |
| shipping_address_id | ID адреса доставки  | SHIPPINGADDRESS.shipping_address_id |
| billing_address_id | ID платежного адреса | BILLINGADDRESS.billing_address_id |
| payment_method | Метод оплаты | ORDER.payment_method |
| discount | Скидка | ORDER.discount |
| order_items | Массив товаров в заказе |  |
| product_id | ID товара |PRODUCT.product_id |
| quantity | Количествое | ORDERITEM.quantity |
| price | Цена за единицу | ORDERITEM.price |

**Успешный ответ (HTTP Code 201 Created)**

```
{
  "order_id": "f1a2b3c4-d5e6-2789-6789-abcdef012345",
  "status": "pending"
}

```
**Ошибка (HTTP Code 400 Bad Request)**

```
{
  "error": "Invalid user ID."
}
```
**Ошибка (HTTP Code 500 Internal Server Error)**

```
{
  "error": "Failed to create order."
}
```

**Логика работы интерфейса:**

1. Сервер принимает запрос на создание заказа.
2. Валидирует данные (например, существование пользователя, адресов доставки и т.д.).
3. Создает запись в таблице ORDER со статусом pending.
4. Создает записи в таблице ORDERITEM для каждого товара в заказе.
5. Запускает Saga для резервирования товаров на складе.

#### API2 - Резерв товара

**Метод**: POST
**URI**: /inventory/reserve
**Описание**:  Резервирование товаров на складе для заказа.

**JSON-запрос (пример):**
```
{
  "order_id": "f1a2b3c4-d5e6-2789-6789-abcdef012345",
  "order_items": [
    {
      "product_id": "d1e2f3a4-b5c6-0987-4567-890abcdef321",
      "quantity": 2
    },
    {
      "product_id": "e1f2a3b4-c5d6-1987-5678-90abcdef432",
      "quantity": 1
    }
  ]
}
```

**Успешный ответ (HTTP Code 201 Created)**

```
{
  "status": "reserved"
}
```
**Ошибка (HTTP Code 400 Bad Request)**

```
{
  "error": "Not enough stock for product d1e2f3a4-b5c6-0987-4567-890abcdef321."
}
```
**Ошибка (HTTP Code 500 Internal Server Error)**

```
{
  "error": "Failed to reserve inventory."
}
```

**Логика работы интерфейса:**

1. Сервер получает ID заказа и список товаров для резервирования.
2. Проверяет наличие достаточного количества каждого товара на складе (INVENTORYITEM.quantity).
2.1. Если достаточно, уменьшает quantity в INVENTORYITEM (фактически, резервирует)  и создает запись (или обновляет существующую) о резерве.  Может потребоваться отдельная таблица RESERVATION для отслеживания резервов.
Возвращает status: reserved.
2.2. Если недостаточно, возвращает ошибку и запускает компенсационную транзакцию в Order Service (отмена заказа).


#### API3 - Осуществление платежа

**Метод**: POST
**URI**: /payments/charge
**Описание**:  Проведение платежа по заказу.

**JSON-запрос (пример):**
```
{
  "order_id": "f1a2b3c4-d5e6-2789-6789-abcdef012345",
  "amount": 150.00,
  "payment_method": "credit_card",
  "credit_card_number": "1234567890123456",
  "cvv": "123"
}
```

**Успешный ответ (HTTP Code 200 OK)**

```
{
  "status": "approved",
  "transaction_id": "g1h2i3j4-k5l6-3987-7890-bcdefa012345"
}
```
**Ошибка (HTTP Code 400 Bad Request)**

```
{
  "error": "Invalid credit card number."
}
```
**Ошибка (HTTP Code 500 Internal Server Error)**

```
{
  "error": "Payment processing failed."
}
```

**Логика работы интерфейса:**

1. Сервер получает ID заказа, сумму и платежные данные.
2. Обращается к платежному шлюзу для проведения платежа.
3. Создает запись в таблице PAYMENT со статусом pending.
4. После успешного проведения платежа, обновляет запись в PAYMENT со статусом approved и transaction_id.
5. В случае неудачи, обновляет запись в PAYMENT со статусом declined и запускает компенсационную транзакцию в Order Service (отмена заказа и возврат резерва на складе)

#### API4 - Списание товара

**Метод**: POST
**URI**: /inventory/deduct
**Описание**:  Списание товара со склада.

**JSON-запрос (пример):**
```
{
  "order_id": "f1a2b3c4-d5e6-2789-6789-abcdef012345",
  "order_items": [
    {
      "product_id": "d1e2f3a4-b5c6-0987-4567-890abcdef321",
      "quantity": 2
    },
    {
      "product_id": "e1f2a3b4-c5d6-1987-5678-90abcdef432",
      "quantity": 1
    }
  ]
}
```

**Успешный ответ (HTTP Code 200 OK)**

```
{
  "status": "deducted"
}
```
**Ошибка (HTTP Code 400 Bad Request)**

```
{
  "error": "Not enough stock for product d1e2f3a4-b5c6-0987-4567-890abcdef321."
}
```
**Ошибка (HTTP Code 500 Internal Server Error)**

```
{
  "error": "Failed to deduct inventory."
}
```

**Логика работы интерфейса:**

1. Сервер получает ID заказа и список товаров для списания.
2. Уменьшает quantity в INVENTORYITEM (окончательное списание)  на указанное количество.
3. Создает запись в INVENTORYTRANSACTION с типом out.
4. Возвращает status: deducted.


### Диаграмма

>![Результат 1](/seq7.png)