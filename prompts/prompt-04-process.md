# Prompt - Lab 4

## Вхідний промпт
```
Згенеруй Mermaid-код для Activity-діаграми бізнес-процесу оформлення замовлення на основі того, що я дала нижче
## 5. Опис бізнес-процесу 

### 5.1. Мета та контекст
- **Мета:** Провести замовлення від кошика покупця до передачі в доставку — без втрати товарів на складі та з чіткою перевіркою оплати
- **Пов'язаний Use Case:** UC-1 (Оформлення замовлення)
- **Нотація:** Activity Diagram (Mermaid Flowchart)
- **Чому вона:** Схема пишеться звичайним текстом прямо в Git. На ній зручно показувати одночасні дії (`fork/join`), перевірки умов (`if/else`) та відправку повідомлень без використання важких XML-файлів BPMN чи сторонніх програм

### 5.2. Учасники та Ролі
- `Customer` - покупець, який купує книжки та отримує email
- `OrderService` - головний сервіс, який керує процесом і міняє статуси замовлення (`PENDING`, `PAID`, `CANCELLED`)
- `InventoryService` - складський сервіс (перевіряє наявність та резервує книги)
- `PaymentGateway` - зовнішній платіжний шлюз для зняття грошей

### 5.3. Повідомлення (Message Flow)
1. `PaymentRequest(orderId, amount)` - `OrderService` шле запит на `PaymentGateway` і чекає на вебхук `PaymentCallback(status)`. Якщо оплата не пройшла або платіжка не відповіла, резерв товарів знімається, а замовлення скасовується
2. `OrderConfirmedNotification(orderId)` - після успішної оплати `OrderService` відправляє покупцю email і одразу йде далі за процесом, не чекаючи на відповідь

### 5.4. Критерії прийняття (Acceptance Criteria)
- **AC-7:** Процес 1:1 повторює кроки з UC-1 (Лаб 2) та Sequence-діаграми (Лаб 3)
- **AC-8:** На діаграмі є розгалуження (`if/else`) та паралельні гілки (`fork/join`) для одночасної обробки складу й оплати
- **AC-9:** Показано два типи повідомлень: із чеканням відповіді (`PaymentCallback`) та звичайне сповіщення в один бік (`OrderConfirmedNotification`)
- **AC-10:** Усі кроки на схемі логічно зв'язані, немає неправильних вузлів або висячих переходів
```
## Відповідь ШІ
Ось Mermaid-код Activity-діаграми (Flowchart з дорожками/swimlanes), що реалізує бізнес-процес оформлення замовлення з урахуванням усіх критеріїв прийняття:

Фрагмент коду
flowchart TD
    %% --- Вузли початку та завершення ---
    Start((Початок))
    EndSuccess(((Успішне завершення)))
    EndFailed(((Скасовано)))

    %% --- Дорожка Customer ---
    subgraph Customer ["👤 Customer (Покупець)"]
        Start --> Act_Submit[Натискає 'Оформити замовлення']
        Act_RecvSuccess[Отримує підтвердження замовлення] --> EndSuccess
        Act_RecvFail[Отримує сповіщення про помилку/скасування] --> EndFailed
    end

    %% --- Дорожка OrderService ---
    subgraph OrderService ["⚙️ OrderService (Сервіс замовлень)"]
        Act_Submit --> Act_CheckUser[Перевірка статусу користувача]
        Act_CheckUser --> Cond_User{Premium Customer?}
        
        Cond_User -- Так --> Act_ApplyDiscount[Застосування преміум-знижки]
        Cond_User -- Ні --> Act_CalcTotal[Розрахунок стандартної суми]

        Act_ApplyDiscount --> Fork1[=== FORK: Паралельна обробка ===]
        Act_CalcTotal --> Fork1

        Fork1 --> Act_CreatePending[Створення ORDER зі статусом PENDING\nта фіксація unit_price в ORDER_ITEM]

        Join1[=== JOIN: Перевірка готовності ===] --> Cond_Stock{Товари зарезервовано?}
        
        Cond_Stock -- Ні --> Act_SetCancelStock[Зміна статусу ORDER на CANCELLED]
        Act_SetCancelStock --> Act_RecvFail

        Cond_Stock -- Так --> Act_SendPayReq["PaymentRequest(orderId, amount)\n[Запит із очікуванням відповіді]"]

        Act_RecvCallback --> Cond_Payment{PaymentCallback status?}

        Cond_Payment -- FAILED / Timeout --> Act_SetCancelPay[Зміна статусу ORDER на CANCELLED]
        Cond_Payment -- SUCCESS --> Act_SetPaid[Зміна статусу ORDER на PAID]

        Act_SetPaid --> Fork2[=== FORK: Паралельні дії після оплати ===]
        
        Fork2 --> Act_SendEmail["OrderConfirmedNotification(orderId)\n[Одностороннє асинхронне сповіщення]"]
        Fork2 --> Act_PassDelivery[Передача замовлення в доставку]

        Act_SendEmail -.-> Act_RecvSuccess
        Act_PassDelivery --> EndSuccess
    end

    %% --- Дорожка InventoryService ---
    subgraph InventoryService ["📦 InventoryService (Складський сервіс)"]
        Fork1 --> Act_CheckStock[Перевірка наявності товарів]
        Act_CheckStock --> Act_ReserveStock[Резервування товарів на складі]
        
        Act_CreatePending --> Join1
        Act_ReserveStock --> Join1

        Act_SetCancelPay --> Act_ReleaseStock[Зняття резерву товарів]
        Act_ReleaseStock --> Act_RecvFail

        Act_SetPaid --> Act_DebitStock[Остаточне списання товарів зі складу]
    end

    %% --- Дорожка PaymentGateway ---
    subgraph PaymentGateway ["💳 PaymentGateway (Платіжний шлюз)"]
        Act_SendPayReq --> Act_ProcessPayment[Обробка транзакції]
        Act_ProcessPayment --> Act_SendCallback["PaymentCallback(status)\n[Надсилання Webhook]"]
        Act_SendCallback --> Act_RecvCallback[Отримання Webhook]
    end

    %% --- Стилізація ---
    style Start fill:#2ecc71,stroke:#27ae60,stroke-width:2px,color:#fff
    style EndSuccess fill:#2ecc71,stroke:#27ae60,stroke-width:2px,color:#fff
    style EndFailed fill:#e74c3c,stroke:#c0392b,stroke-width:2px,color:#fff
    style Fork1 fill:#34495e,stroke:#2c3e50,color:#fff
    style Join1 fill:#34495e,stroke:#2c3e50,color:#fff
    style Fork2 fill:#34495e,stroke:#2c3e50,color:#fff

## Аудит відповіді ШІ
1. **Зайва логіка зі знижками:** ШІ вставив перевірку преміум аккаунту та знижок. У мене у БД (ERD з Лаб 1) та в описі Use Case (UC-1 з Лаб 2) цього немає
2. **Два фінали після `Fork2`:** ШІ зв'язав відправку email напряму з кінцем схеми. Через це процес завершувався двічі. Email має просто надсилатися в фоні, а єдиний фінал має бути передачею в доставку
3. **Забутий резерв на складі:** Якщо якихось товарів не вистачило, процес одразу видавав помилку й забував скасувати резерв для тих книг, які вже встигли заблокуватися 

## Зворотний зв'язок та корекція специфікації
На основі цих помилок у `spec.md` було додано додаткові критерії прийняття (**AC-11** та **AC-13**)

## Вхідний промпт 2
```
Виправи код діаграми для бізнес-процесу з урахуванням ось цих нових критеріїв AC-11 - AC-13
- **AC-11** Процес не містить вигаданих дій (знижки, преміум-акаунти), яких немає в ERD та UC-1
- **AC-12** Відправка email іде як фонова дія без власного кінця, а єдиний фінал - передача в доставку
- **AC-13** При будь-якому збої (нестача товарів або помилка оплати) зарезервовані книги обов'язково знімаються з резерву
```
## Відповідь ШІ
flowchart TD
    %% --- Вузли початку та завершення ---
    Start((Початок))
    EndSuccess(((Успішне завершення)))
    EndFailed(((Скасовано)))

    %% --- Дорожка Customer ---
    subgraph Customer ["👤 Customer (Покупець)"]
        Start --> Act_Submit[Натискає 'Оформити замовлення']
        Act_RecvFail[Отримує сповіщення про помилку/скасування] --> EndFailed
    end

    %% --- Дорожка OrderService ---
    subgraph OrderService ["⚙️ OrderService (Сервіс замовлень)"]
        Act_Submit --> Act_CalcTotal[Розрахунок суми замовлення]
        Act_CalcTotal --> Fork1[=== FORK: Паралельна обробка ===]

        Fork1 --> Act_CreatePending[Створення ORDER зі статусом PENDING\nта фіксація unit_price у ORDER_ITEM]

        Join1[=== JOIN: Перевірка готовності ===] --> Cond_Stock{Товари зарезервовано?}

        %% AC-13: Скасування та зняття резерву при нестачі товарів
        Cond_Stock -- Ні --> Act_SetCancelStock[Зміна статусу ORDER на CANCELLED]
        Act_SetCancelStock --> Act_ReleaseStockFail

        %% Оплата
        Cond_Stock -- Так --> Act_SendPayReq["PaymentRequest(orderId, amount)\n[Запит із очікуванням відповіді]"]

        Act_RecvCallback --> Cond_Payment{PaymentCallback status?}

        %% AC-13: Скасування та обов'язкове зняття резерву при помилці оплати
        Cond_Payment -- FAILED / Timeout --> Act_SetCancelPay[Зміна статусу ORDER на CANCELLED]
        Act_SetCancelPay --> Act_ReleaseStockPay

        %% Успішна оплата
        Cond_Payment -- SUCCESS --> Act_SetPaid[Зміна статусу ORDER на PAID]

        %% AC-12: Фонова відправка email без власного кінця
        Act_SetPaid --> Act_SendEmail["OrderConfirmedNotification(orderId)\n[Фонове асинхронне сповіщення]"]

        %% AC-12: Єдиний фінал успіху — передача в доставку
        Act_SetPaid --> Act_PassDelivery[Передача замовлення в доставку]
        Act_PassDelivery --> EndSuccess
    end

    %% --- Дорожка InventoryService ---
    subgraph InventoryService ["📦 InventoryService (Складський сервіс)"]
        Fork1 --> Act_CheckReserve[Перевірка наявності та резервування книг]

        Act_CreatePending --> Join1
        Act_CheckReserve --> Join1

        %% AC-13: Обов'язкове зняття з резерву
        Act_ReleaseStockFail[Зняття резерву товарів] --> Act_RecvFail
        Act_ReleaseStockPay[Зняття резерву товарів] --> Act_RecvFail

        Act_SetPaid --> Act_DebitStock[Остаточне списання товарів зі складу]
    end

    %% --- Дорожка PaymentGateway ---
    subgraph PaymentGateway ["💳 PaymentGateway (Платіжний шлюз)"]
        Act_SendPayReq --> Act_ProcessPayment[Обробка транзакції]
        Act_ProcessPayment --> Act_SendCallback["PaymentCallback(status)\n[Надсилання Webhook]"]
        Act_SendCallback --> Act_RecvCallback[Отримання Webhook]
    end

    %% --- Стилізація ---
    style Start fill:#2ecc71,stroke:#27ae60,stroke-width:2px,color:#fff
    style EndSuccess fill:#2ecc71,stroke:#27ae60,stroke-width:2px,color:#fff
    style EndFailed fill:#e74c3c,stroke:#c0392b,stroke-width:2px,color:#fff
    style Fork1 fill:#34495e,stroke:#2c3e50,color:#fff
    style Join1 fill:#34495e,stroke:#2c3e50,color:#fff

## Аудит відповіді ШІ
- **AC-11 виконала:** Тепер сума рахується стандартно, як в ERD та UC-1
- **AC-12 виконала:** Email надсилається у фоні, процес більше не завершується двічі. Єдиний кінець тепер  це передача в доставку
- **AC-13 виконала:** Додали зняття резерву товарів при будь-якій помилці (і якщо товару не вистачило, і якщо відхилили оплату)