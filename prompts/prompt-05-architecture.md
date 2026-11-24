## Промпт 1
```
Ось мій spec.md для онлайн-магазину книг (Лабораторна робота №5), побудуй на її основі UML Component Diagram у синтаксисі PlantUML 

## 9. Архітектура системи (Лаб 5)

### 9.1. Межа системи та зовнішні сервіси
- **Всередині системи (v1.0):** клієнтська частина (UI), API, 4 сервіси з логікою, репозиторії (Data Access) і БД
- **Зовні системи:** тільки **Payment Gateway** (через інтерфейс `IPaymentGateway`). Картки в себе не зберігаємо, оплата йде напряму через HTTPS (`NFR-2`)
- **Поза межами v1.0:** розсилки на email/SMS, вхід через соцмережі та автоматичний refund (`REQ-6` перенесли у v2.0 ще в Лаб 2)

### 9.2. Модулі та інтерфейси
- **UI (`Frontend`):** інтерфейс магазину (каталог, кошик, оформлення/скасування замовлення, відгуки). Звертається до бекенду через required-інтерфейс `IStorefrontAPI`
- **API (`API Gateway`):** приймає запити від фронтенду (дає `IStorefrontAPI`), перевіряє авторизацію (`REQ-1`, `NFR-3`) і передає виклики на потрібні сервіси
- **Сервіси (Service Layer):**
  - `AuthService` (дає `IAuthService`) - вхід, перевірка ролей (`Guest`, `Customer`, `Premium Customer`) і хешування паролів (`NFR-1`)
  - `CatalogService` (дає `ICatalogService`) - перегляд і пошук книжок, авторів та категорій (`UC-1`)
  - `OrderService` (дає `IOrderService`) - оформлення замовлення (`UC-2`), знижка `applyDiscount` для преміумів (`UC-Discount`), оплата через платіжку (`UC-3`) і скасування у `PENDING` (`UC-4`)
  - `ReviewService` (дає `IReviewService`) - перевіряє, чи купував юзер книгу, і зберігає відгук (`UC-5`)
- **Data Access (репозиторії):** `UserRepository` (`IUserRepository`), `BookRepository` (`IBookRepository`), `OrderRepository` (`IOrderRepository`), `ReviewRepository` (`IReviewRepository`). Сервіси ходять у базу тільки через них, тут же фіксується `BOOK.price` у `ORDER_ITEM.unit_price` (`REQ-2`, `NFR-4`)
- **БД (`Database`):** база в 3NF (дає `ISQLDataSource`), де лежать 7 таблиць з Лаб 1: `USER`, `BOOK`, `AUTHOR`, `CATEGORY`, `ORDER`, `ORDER_ITEM`, `REVIEW`

### 9.3. Критерії прийняття (Acceptance Criteria)
- **AC-14:** На діаграмі показані всі 5 шарів (UI, API, сервіси, Data Access, БД) і чітка межа системи
- **AC-15:** У компонентів вказані `provided` та `required` інтерфейси
- **AC-16:** 4 сервіси покривають усі юзкейси `UC-1`–`UC-5`, а репозиторії працюють з усіма 7 таблицями ERD
- **AC-17:** За межею системи тільки `Payment Gateway`, без зайвих сервісів, яких не було в Лаб 1–4
- **AC-18:** Назви сервісів і методів (`createOrder`, `requestCancellation`, `addReview`, `processPayment`) збігаються зі Sequence та Activity
```

## Відповідь ШІ
```
@startuml component-diagram
top to bottom direction
skinparam componentStyle uml2
skinparam linetype ortho
skinparam packageStyle rectangle
skinparam shadowing false

' --- Межа системи (v1.0) ---
rectangle "Система: Онлайн-магазин книг (v1.0)" as SystemBoundary {

    ' 1. Шар UI
    package "1. UI Layer (Presentation)" as UILayer {
        component "Frontend\n---\nКаталог, кошик,\nоформлення/скасування,\nвідгуки" as Frontend
    }

    interface "IStorefrontAPI" as IStorefrontAPI

    ' 2. Шар API
    package "2. API Layer" as APILayer {
        component "API Gateway\n---\nМаршрутизація запитів,\nперевірка авторизації\n(REQ-1, NFR-3)" as APIGateway
    }

    interface "IAuthService" as IAuthService
    interface "ICatalogService" as ICatalogService
    interface "IOrderService" as IOrderService
    interface "IReviewService" as IReviewService

    ' 3. Шар бізнес-логіки (4 сервіси)
    package "3. Service Layer (Business Logic)" as ServiceLayer {
        component "AuthService\n---\n+ login()\n+ getUserStatus()\n+ hashPassword() [NFR-1]" as AuthService

        component "CatalogService\n---\n+ getBooks() [UC-1]\n+ searchBooks()" as CatalogService

        component "OrderService\n---\n+ createOrder() [UC-2]\n+ applyDiscount() [UC-Discount]\n+ processPayment() [UC-3]\n+ requestCancellation() [UC-4]" as OrderService

        component "ReviewService\n---\n+ addReview() [UC-5]\n+ checkUserPurchasedBook()" as ReviewService
    }

    interface "IUserRepository" as IUserRepo
    interface "IBookRepository" as IBookRepo
    interface "IOrderRepository" as IOrderRepo
    interface "IReviewRepository" as IReviewRepo

    ' 4. Шар доступу до даних (Repositories)
    package "4. Data Access Layer (Repositories)" as DALayer {
        component "UserRepository\n---\nСутності: USER" as UserRepo

        component "BookRepository\n---\nСутності: BOOK,\nAUTHOR, CATEGORY" as BookRepo

        component "OrderRepository\n---\nСутності: ORDER, ORDER_ITEM\n(фіксація unit_price:\nREQ-2, NFR-4)" as OrderRepo

        component "ReviewRepository\n---\nСутності: REVIEW" as ReviewRepo
    }

    interface "ISQLDataSource" as ISQLDataSource

    ' 5. Шар бази даних
    package "5. Database Layer" as DBLayer {
        database "Database (3NF)\n---\nТаблиці (7):\nUSER, BOOK, AUTHOR, CATEGORY,\nORDER, ORDER_ITEM, REVIEW" as Database
    }
}

' --- Зовнішній сервіс за межею системи ---
interface "IPaymentGateway" as IPaymentGateway
component "Payment Gateway\n<<External System>>\n---\nОплата карткою та webhook\n(HTTPS / NFR-2)" as PaymentGateway

' --- Зв'язки UI <-> API ---
Frontend --( IStorefrontAPI
IStorefrontAPI -- APIGateway

' --- Зв'язки API <-> Services ---
APIGateway --( IAuthService
IAuthService -- AuthService

APIGateway --( ICatalogService
ICatalogService -- CatalogService

APIGateway --( IOrderService
IOrderService -- OrderService

APIGateway --( IReviewService
IReviewService -- ReviewService

' --- Зв'язки Services <-> Repositories ---
AuthService --( IUserRepo
IUserRepo -- UserRepo

CatalogService --( IBookRepo
IBookRepo -- BookRepo

OrderService --( IOrderRepo
OrderService --( IUserRepo
OrderService --( IBookRepo
IOrderRepo -- OrderRepo

ReviewService --( IReviewRepo
ReviewService --( IOrderRepo
IReviewRepo -- ReviewRepo

' --- Зв'язки Repositories <-> Database ---
UserRepo --( ISQLDataSource
BookRepo --( ISQLDataSource
OrderRepo --( ISQLDataSource
ReviewRepo --( ISQLDataSource
ISQLDataSource -- Database

' --- Зв'язок із зовнішнім платіжним шлюзом ---
OrderService --( IPaymentGateway
IPaymentGateway -- PaymentGateway

@enduml
```
## Аналіз відповіді ШІ
ШІ нормально розкидав усе по 5 шарах і підказав, що я у спеці забула дописати зв'язки OrderService до UserRepository і BookRepository, а ReviewService до OrderRepository. Але в самій схемі він дописав зайвий webhook у платіжку, не додав актора покупця за межею системи, а через linetype ortho стрілки до бази злиплися. Тому я дописала всі залежності в spec.md і попросила поправити ці три моменти на діаграмі