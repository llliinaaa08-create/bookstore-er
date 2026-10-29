# Prompt - Lab 3 

## Вхідний промпт
```
Згенеруй Sequence-діаграми у синтаксисі Mermaid для 3 сценаріїв на основі ось цього спеку який я надіслала нижче

### Учасники
- **`Customer`** — покупець
- **`Frontend`** — інтерфейс веб-сайту
- **`OrderService`** — сервіс замовлень
- **`ReviewService`** — сервіс відгуків
- **`Database`** — БД (`USER`, `BOOK`, `ORDER`, `ORDER_ITEM`, `REVIEW`)
- **`PaymentGateway`** — платіжний шлюз

---

### Сценарії

### Сценарій 1: Оформлення та оплата (з Gherkin 1)
1. `Customer` натискає «Оформити» -> `Frontend.submitOrder()`
2. `Frontend` викликає `OrderService.createOrder()`
3. `OrderService` бере з `Database` статус юзера (`getUserStatus`)
4. **[opt]** Якщо юзер `Premium Customer` — рахується знижка `applyDiscount()`
5. `OrderService` створює замовлення в `Database` зі статусом `PENDING` та фіксує `unit_price` у таблиці `ORDER_ITEM`
6. `OrderService` відправляє запит у `PaymentGateway.processPayment()`
7. **[alt]** Якщо шлюз повертає `SUCCESS` -> `Database` оновлює статус замовлення на `PAID`

### Сценарій 2: Скасування замовлення (з Gherkin 2)
1. `Customer` натискає «Скасувати» -> `Frontend.cancelOrder()`
2. `Frontend` перенаправляє запит у `OrderService.requestCancellation()`
3. `OrderService` перевіряє в `Database` поточний статус (`getOrderStatus`)
4. **[alt]** Якщо статус `PENDING` -> `Database` змінює статус на `CANCELLED`
5. **[alt]** Якщо статус не `PENDING` -> повертається помилка «Скасування неможливе»

### Сценарій 3: Додавання відгуку (з Gherkin 3)
1. `Customer` залишає оцінку й коментар -> `Frontend.submitReview()`
2. `Frontend` передає дані в `ReviewService.addReview()`
3. `ReviewService` робить запит до `Database` (`checkUserPurchasedBook`), чи купував юзер цю книгу
4. **[alt]** Якщо книга купувалася -> `Database` зберігає новий запис у `REVIEW`
5. **[alt]** Якщо не купувалася -> повертається помилка «Відгук може залишити лише покупець»

---

### Критерії прийняття
1. **AC-1:** Сценарії 1:1 відповідають Gherkin-сценаріям з Розділу 4
2. **AC-2:** Учасники збігаються з таблицями ERD (`USER`, `BOOK`, `ORDER`, `ORDER_ITEM`, `REVIEW`) та акторами Use Case
3. **AC-3:** Виклики мають реалістичні назви (`createOrder()`, `requestCancellation()`, `addReview()`)
4. **AC-4:** У кожному сценарії використовуються умовні блоки `alt` або `opt`"
```
## Відповідь ШІ
Сценарій 1: Оформлення та оплата
Фрагмент коду
sequenceDiagram
    autonumber
    actor Customer
    participant Frontend
    participant OrderService
    participant Database
    participant PaymentGateway

    Customer->>Frontend: submitOrder()
    Frontend->>OrderService: createOrder()
    OrderService->>Database: getUserStatus()
    Database-->>OrderService: userStatus

    opt User is Premium Customer
        OrderService->>OrderService: applyDiscount()
    end

    OrderService->>Database: createOrder(status: PENDING, unit_price in ORDER_ITEM)
    Database-->>OrderService: orderCreated

    OrderService->>PaymentGateway: processPayment()
    PaymentGateway-->>OrderService: paymentResult

    alt Payment result == SUCCESS
        OrderService->>Database: updateOrderStatus(PAID)
        Database-->>OrderService: statusUpdated
        OrderService-->>Frontend: orderSuccess
        Frontend-->>Customer: Замовлення успішно оплачено
    else Payment result == FAILED / ERROR
        OrderService->>Database: updateOrderStatus(PAYMENT_FAILED)
        Database-->>OrderService: statusUpdated
        OrderService-->>Frontend: orderFailed
        Frontend-->>Customer: Помилка оплати
    end
Сценарій 2: Скасування замовлення
Фрагмент коду
sequenceDiagram
    autonumber
    actor Customer
    participant Frontend
    participant OrderService
    participant Database

    Customer->>Frontend: cancelOrder()
    Frontend->>OrderService: requestCancellation()
    OrderService->>Database: getOrderStatus()
    Database-->>OrderService: orderStatus

    alt Status == PENDING
        OrderService->>Database: updateOrderStatus(CANCELLED)
        Database-->>OrderService: statusUpdated
        OrderService-->>Frontend: cancellationSuccess
        Frontend-->>Customer: Замовлення скасовано
    else Status != PENDING
        OrderService-->>Frontend: error("Скасування неможливе")
        Frontend-->>Customer: Помилка: Скасування неможливе
    end
Сценарій 3: Додавання відгуку
Фрагмент коду
sequenceDiagram
    autonumber
    actor Customer
    participant Frontend
    participant ReviewService
    participant Database

    Customer->>Frontend: submitReview()
    Frontend->>ReviewService: addReview()
    ReviewService->>Database: checkUserPurchasedBook()
    Database-->>ReviewService: isPurchased

    alt Book was purchased (isPurchased == true)
        ReviewService->>Database: insertReview(REVIEW)
        Database-->>ReviewService: reviewSaved
        ReviewService-->>Frontend: reviewSuccess
        Frontend-->>Customer: Відгук успішно опубліковано
    else Book was NOT purchased (isPurchased == false)
        ReviewService-->>Frontend: error("Відгук може залишити лише покупець")
        Frontend-->>Customer: Помилка: Відгук може залишити лише покупець
    end
    
## Аудит відповіді ШІ
1. **(AC-5):** ШІ проігнорував `activate` та `deactivate`, через це лінії активності процесів просто зникли й не видно фокус контролю
2. **Розмовна мова в коді (AC-6):** У першому сценарії ШІ додав текст `unit_price in ORDER_ITEM` прямо в аргументи методу замість нормальної сигнатури

## Зворотний зв'язок та корекція специфікації
На основі цих помилок у розділ 7 файлу `spec.md` було додано додаткові критерії прийняття (**AC-5** та **AC-6**)

## Вхідний промпт 2
```
Перероби ці діаграми з урахуванням доданих критеріїв прийняття які я кинула нижче, учасники й сценарії ті ж самі 

5. **AC-5:** Обов'язкова наявність блоків `activate` / `deactivate` для відображення фокусу контролю
6. **AC-6:** Сигнатури методів повинні використовувати суворий синтаксис (без пояснювального тексту мовою на зразок `unit_price in ORDER_ITEM`)
```
## Відповідь ШІ 2
Ось оновлені Sequence-діаграми з урахуванням доданих критеріїв AC-5 (блоків activate / deactivate) та AC-6 (суворого синтаксису сигнатур методів):
Сценарій 1: Оформлення та оплата
Фрагмент коду

sequenceDiagram
    autonumber
    actor Customer
    participant Frontend
    participant OrderService
    participant Database
    participant PaymentGateway

    Customer->>Frontend: submitOrder(cartData)
    activate Frontend
    Frontend->>OrderService: createOrder(userId, cartItems)
    activate OrderService

    OrderService->>Database: getUserStatus(userId)
    activate Database
    Database-->>OrderService: userStatus
    deactivate Database

    opt userStatus == "PREMIUM"
        OrderService->>OrderService: applyDiscount(cartItems)
    end

    OrderService->>Database: createOrder(userId, cartItems, "PENDING")
    activate Database
    Database-->>OrderService: orderId
    deactivate Database

    OrderService->>PaymentGateway: processPayment(orderId, totalAmount)
    activate PaymentGateway
    PaymentGateway-->>OrderService: paymentResult
    deactivate PaymentGateway

    alt paymentResult == "SUCCESS"
        OrderService->>Database: updateOrderStatus(orderId, "PAID")
        activate Database
        Database-->>OrderService: success
        deactivate Database
        OrderService-->>Frontend: orderSuccessResponse
        Frontend-->>Customer: displaySuccessMessage()
    else paymentResult == "FAILED"
        OrderService->>Database: updateOrderStatus(orderId, "PAYMENT_FAILED")
        activate Database
        Database-->>OrderService: success
        deactivate Database
        OrderService-->>Frontend: orderFailedResponse
        Frontend-->>Customer: displayErrorMessage()
    end

    deactivate OrderService
    deactivate Frontend
Сценарій 2: Скасування замовлення
Фрагмент коду

sequenceDiagram
    autonumber
    actor Customer
    participant Frontend
    participant OrderService
    participant Database

    Customer->>Frontend: cancelOrder(orderId)
    activate Frontend
    Frontend->>OrderService: requestCancellation(orderId)
    activate OrderService

    OrderService->>Database: getOrderStatus(orderId)
    activate Database
    Database-->>OrderService: orderStatus
    deactivate Database

    alt orderStatus == "PENDING"
        OrderService->>Database: updateOrderStatus(orderId, "CANCELLED")
        activate Database
        Database-->>OrderService: success
        deactivate Database
        OrderService-->>Frontend: cancellationSuccess
        Frontend-->>Customer: displayCancellationSuccess()
    else orderStatus != "PENDING"
        OrderService-->>Frontend: cancellationError
        Frontend-->>Customer: displayCancellationError()
    end

    deactivate OrderService
    deactivate Frontend
Сценарій 3: Додавання відгуку
Фрагмент коду

sequenceDiagram
    autonumber
    actor Customer
    participant Frontend
    participant ReviewService
    participant Database

    Customer->>Frontend: submitReview(bookId, rating, comment)
    activate Frontend
    Frontend->>ReviewService: addReview(userId, bookId, rating, comment)
    activate ReviewService

    ReviewService->>Database: checkUserPurchasedBook(userId, bookId)
    activate Database
    Database-->>ReviewService: isPurchased
    deactivate Database

    alt isPurchased == true
        ReviewService->>Database: insertReview(userId, bookId, rating, comment)
        activate Database
        Database-->>ReviewService: reviewId
        deactivate Database
        ReviewService-->>Frontend: reviewSuccess
        Frontend-->>Customer: displayReviewSuccess()
    else isPurchased == false
        ReviewService-->>Frontend: reviewError
        Frontend-->>Customer: displayReviewError()
    end

    deactivate ReviewService
    deactivate Frontend

## Аудит повторної відповіді ШІ (Ітерація 2)

ШІ додав `activate` та `deactivate` для кожного сервісу, тож лінії активності тепер відображаються суцільно й без розривів (AC-5). З дужок методів зник увесь зайвий текст, а замість нього з'явилися нормальні параметри на зразок `createOrder(userId, cartItems, "PENDING")` (AC-6). Згенеровані діаграми повністю покривають AC-1 - AC-6