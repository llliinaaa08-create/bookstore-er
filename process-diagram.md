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