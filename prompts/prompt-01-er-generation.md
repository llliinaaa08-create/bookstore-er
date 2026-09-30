# Лог генерації ER-діаграми 

## Prompt 1
> Зроби ER-діаграму для онлайн-магазину книг на Mermaid (`erDiagram`) за цими вимогами:
> 
> 1. Опис сутностей, атрибутів та зв'язків:
> - USER (Користувач): має 'id', 'email', 'name', 'role'
>   - Створює багато ORDER (1:N)
>   - Залишає багато REVIEW (1:N)
> - BOOK (Книга): має 'id', 'title', 'isbn', 'price', 'category_id'
>   - Належить одній CATEGORY (N:1)
>   - Може мати кількох авторів AUTHOR (N:M)
>   - Міститься у багатьох ORDER_ITEM (1:N)
>   - Має багато REVIEW (1:N)
> - AUTHOR (Автор): має 'id', 'name', 'bio'
>   - Є автором багатьох BOOK (N:M)
> - CATEGORY (Категорія): має 'id', 'name'
>   - Містить багато BOOK (1:N)
> - ORDER (Замовлення): має 'id', 'user_id', 'status', 'total_amount'
>   - Належить одному USER (N:1)
>   - Містить багато ORDER_ITEM (1:N)
> - ORDER_ITEM (Позиція замовлення): має 'id', 'order_id', 'book_id', 'quantity', 'unit_price'
>   - Належить одному ORDER (N:1)
>   - Вказує на одну BOOK (N:1)
> - REVIEW (Відгук): має 'id', 'user_id', 'book_id', 'rating', 'comment'
>   - Залишений одним USER (N:1)
>   - Стосується однієї BOOK (N:1)
> 
> 2. Критерії прийняття:
> 1. Підсумкова діаграма повинна бути згенерована у коді Mermaid (`erDiagram`)
> 2. Усі 7 сутностей мають первинний ключ з однаковою назвою `id`, типом `UUID` та позначкою `PK`
> 3. Зв'язок N:M між `BOOK` та `AUTHOR` показуємо напряму між сутностями, без додаткової таблиці-зв'язки
> 4. Усі зв'язки 1:N повинні мати явно вказані зовнішні ключі з позначкою `FK` на боці «багато» (`category_id`, `user_id`, `order_id`, `book_id`)
> 5. Структура даних має суворо відповідати 3-й нормальній формі (3NF)
> 6. Для типів даних використовувати лише концептуальні назви (`string`, `number`), без SQL-типів (`VARCHAR`, `INT` тощо)
> 7. Заборонено додавати будь-які атрибути, не згадані в цьому описі (наприклад, `created_at` або `discount`)

---

## Перша відповідь AI

```
erDiagram USER { UUID id PK string email string name string role }

AUTHOR {
    UUID id PK
    string name
    string bio
}

CATEGORY {
    UUID id PK
    string name
}

BOOK {
    UUID id PK
    string title
    string isbn
    number price
    UUID category_id FK
}

ORDER {
    UUID id PK
    UUID user_id FK
    string status
    number total_amount
}

ORDER_ITEM {
    UUID id PK
    UUID order_id FK
    UUID book_id FK
    number quantity
    number unit_price
}

REVIEW {
    UUID id PK
    UUID user_id FK
    UUID book_id FK
    number rating
    string comment
}

USER ||--o{ ORDER : "creates"
USER ||--o{ REVIEW : "leaves"
CATEGORY ||--o{ BOOK : "contains"
BOOK }|--|{ AUTHOR : "has_author"
ORDER ||--o{ ORDER_ITEM : "contains"
BOOK ||--o{ ORDER_ITEM : "included_in"
BOOK ||--o{ REVIEW : "has"
```
## Prompt 2
> Нормально в цілому, але треба виправити типи ключів. Ти вказав UUID як тип поля, а за вимогами має бути концептуальний string і вже після PK/FK писати "UUID" у лапках
>
> Поправ Mermaid-код:
> - Усі UUID id PK зміни на string id PK "UUID"
> - Усі зовнішні ключі (як-от UUID category_id FK) зміни на string category_id FK "UUID"
> - Інші поля залиш тільки з типами string або number
## Друга віжповідь АІ
```
erDiagram
    USER {
        string id PK "UUID"
        string email
        string name
        string role
    }

    AUTHOR {
        string id PK "UUID"
        string name
        string bio
    }

    CATEGORY {
        string id PK "UUID"
        string name
    }

    BOOK {
        string id PK "UUID"
        string title
        string isbn
        number price
        string category_id FK "UUID"
    }

    ORDER {
        string id PK "UUID"
        string user_id FK "UUID"
        string status
        number total_amount
    }

    ORDER_ITEM {
        string id PK "UUID"
        string order_id FK "UUID"
        string book_id FK "UUID"
        number quantity
        number unit_price
    }

    REVIEW {
        string id PK "UUID"
        string user_id FK "UUID"
        string book_id FK "UUID"
        number rating
        string comment
    }

    USER ||--o{ ORDER : "creates"
    USER ||--o{ REVIEW : "leaves"
    CATEGORY ||--o{ BOOK : "contains"
    BOOK }|--|{ AUTHOR : "has_author"
    ORDER ||--o{ ORDER_ITEM : "contains"
    BOOK ||--o{ ORDER_ITEM : "included_in"
    BOOK ||--o{ REVIEW : "has"
```