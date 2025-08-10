# 📚 Онлайн-книгарня — API на NestJS

Це бекенд для онлайн-книгарні, створений за допомогою [NestJS](https://nestjs.com/).

# 🚀 Запуск проєкту

## Встановлення залежностей
npm install

## Запуск сервера у режимі розробки
npm run start:dev

## 📦 Основні залежності

@nestjs/common, @nestjs/core, @nestjs/swagger

cookie-parser - Робота з cookie (читання, видалення)

class-validator - Валідація DTO

typeorm — для роботи з БД

bcrypt — для хешування паролів

passport, jwt — для аутентифікації

supabase — для зберігання зображень

redis — для керування сесіями токенів

Swagger - API-документація

# 🛠️ Змінні середовища

### PORT – порт серверу (за замовчуванням 4000)

### CLIENT_ORIGIN – дозволене походження для CORS (за замовчуванням http://localhost:3000)

# 🧠 RedisService 

Сервіс для роботи з Redis у проєкті на NestJS. Використовується для кешування, зберігання токенів. Автоматично підключається до Redis при ініціалізації модуля та закриває з’єднання при завершенні.

>Redis-сервер має бути запущений перед стартом NestJS-додатку.

### ⚙️ Конфігурація Redis

this.client = new Redis({
host: 'localhost',
port: 6379,
});

# ☁️ Supabase Service

Ініціалізує Supabase клієнт для зберігання файлів (наприклад, аватарів користувачів, обкладинок новин, тощо).

### ⚙️ Приклад .env 

CONFIG__SUPABASE__URL=https://your-project.supabase.co

CONFIG__SUPABASE__KEY=your-secret-api-key

>Використовуйте service role key лише на бекенді. Не виводьте ключ у фронтенд.

# 📧 Email Service 

Сервіс для генерації та надсилання email-повідомлень через Gmail SMTP з використанням шаблонів Handlebars. Підтримується структура з layout, partial'ами (header, footer) та динамічним наповненням контенту.

### 🛠️ Конфігурація .env

CONFIG__SMTP__SMTP_EMAIL=your_email@gmail.com 

CONFIG__SMTP__SMTP_PASSWORD=your_gmail_app_password

>Gmail вимагає створення App Passwords при включеній 2FA.

# 🔐Auth

Реалізовано повний цикл аутентифікації користувача, включаючи завантаження аватарів, реєстрацію, логін, оновлення токенів, вихід із системи.

Авторизація
Для захищених маршрутів використовується @UseGuards(AuthGuard()) із JWT.

>refreshToken зберігається в HTTP-only cookie

>mail-нотифікації
Після реєстрації користувачу відправляється вітальний email за допомогою EmailService.

### Аутентифікація
Реєстрація
POST /auth/registration

multipart/form-data

Тіло: email, password, firstName, lastName, age, phone, image (файл) (опціонально)

📥 Завантажене зображення зберігається у Supabase. Якщо не передано — використовується аватар за замовчуванням.


### Логін
POST /auth/login

Тіло: email, password

📤 Відповідь:

json
{
  "accessToken": "jwt-token",
  "refreshToken": "refresh-token"
}

### Вихід
POST /auth/logout
🔐 Requires Bearer Token

### Оновлення токенів
POST /auth/refresh

📤 Відповідь:
json
{
  "refreshToken": "..."
}

### Скидання пароля (через email)

PATCH /auth/resetPassword

Тіло:
email: string;

>Цей маршрут не вимагає токена — викликається користувачем, який забув пароль.
На пошту приходить випадково згенерований пароль.

### Зміна пароля

PATCH /auth/newPassword

Authorization: Bearer <accessToken>

Тіло:
{
"oldPassword": "currentPassword",
"newPassword": "newSecurePassword"
}

📤 Відповідь:
{
"message": "Password updated successfully"
}

# 🛒 Basket

Функціонал дозволяє авторизованим користувачам додавати книги до кошика, видаляти окремі книги, очищати весь кошик і переглядати вміст.

> Усі маршрути вимагають авторизації (JWT Bearer Token).

### Отримати поточний кошик

ET /basket/find

Headers:
Authorization: Bearer <accessToken>

📤 Відповідь: Об'єкт кошика користувача:

json
{
"id": "basket-id",
"user": { ... },
"items": [
{
"id": "item-id",
"book": {
"id": "book-id",
"title": "Назва книги",
...
},
"quantity": 2
},
...
]
}

### Додати книгу до кошика

POST /basket/add

Headers:
Authorization: Bearer <accessToken>

Тіло:
json
  {
    "bookId": "string",
    "quantity": 2 // опціонально, за замовчуванням 1
  }
📤 Відповідь: Оновлений об'єкт кошика

### Видалити одну книгу з кошика

DELETE /basket/remove/:id

:id — bookId

Headers:
Authorization: Bearer <accessToken>

📤 Відповідь: Оновлений об'єкт кошика

### Очистити весь кошик

DELETE /basket/clear

Headers:
Authorization: Bearer <accessToken>

📤 Відповідь: Оновлений (порожній) об'єкт кошика

### 📘 Примітки

>Якщо книга вже є в кошику — її кількість збільшується.

>Якщо передати від'ємну кількість, вона буде приведена до 0.

>Якщо кошика в користувача ще немає — він створюється автоматично.

# 📚 Books

Модуль для керування книгами: створення, оновлення, публікація, отримання, фільтрація та видалення.

### Створення книги

POST /books/create-book

Headers:
Authorization: Bearer <accessToken>

Поле зображення: image

Тіло:
title: string;
price: number;
description?: string;
author?: string;
gift: boolean;
cover: 'soft' | 'firm';
categories: string[];

### Оновлення книги

PATCH /books/update/:id

Headers:
Authorization: Bearer <accessToken>

Тіло: (всі поля опціональні) title?: string;
price?: number;
description?: string;
author?: string;
image?: string;
gift?: boolean;
cover?: 'soft' | 'firm';
categories?: string[];

### Зміна статусу публікації

PUT /books/published/:id

Headers:
Authorization: Bearer <accessToken>

### Отримання списку книг

GET /books/list

### Отримання однієї книги

GET /books/find/:id

### Видалення книги

DELETE /books/delete/:id

Headers:
Authorization: Bearer <accessToken>

# 🧩 Category
Модуль Category забезпечує CRUD-операції для категорій книг з підтримкою пагінації, сортування, фільтрації та вкладених категорій.

### Створення категорії

POST /category/create

>Тільки Admin

Authorization: Bearer <accessToken>

Тіло:
name: string,
parentId?: string

### Оновлення категорій

PATCH /category/update/:id

>Тільки Admin

Authorization: Bearer <accessToken>

Тіло: (усі поля опціональні)
name?: string,
parentId?:string

### Видалення категорії

DELETE /category/delete/:id

Authorization: Bearer <accessToken>

>Тільки Admin

### Отримання списку категорій

GET /category/list

Query-параметри:

| Параметр   | Тип    | Опис                                         |
| ---------- | ------ | -------------------------------------------- |
| `page`     | string | Номер сторінки (за замовчуванням: 1)         |
| `limit`    | string | Кількість на сторінку (за замовчуванням: 10) |
| `sort`     | string | Поле для сортування (наприклад, `name`)      |
| `order`    | string | `'ASC'` або `'DESC'`                         |
| `search`   | string | Пошук по назві                               |
| `parentId` | string | Фільтрація по батьківській категорії         |
| `name`     | string | Точна назва категорії                        |
| `id`       | string | Пошук по ID                                  |

### Отримання головних категорій

GET /category/mainCategories/list

>Повертає лише категорії, у яких немає parentId.

### Отримання категорії по ID

GET /category/find/id/:id

# 💬 Comments 

Модуль для керування коментарями користувачів до книг: створення, оновлення, фільтрація, перегляд та видалення. Підтримує авторизацію та фільтрацію за book_id і user_id. Тільки авторизовані користувачі можуть:
створювати коментарі
оновлювати свої коментарі
видаляти свої коментарі

### Створення коментаря

POST /comments/create-comment

Authorization: Bearer <accessToken>

Тіло:
book_id: string,
text: string

Відповідь:
{
"id": "b3a6c35e-4bb3-4f29-b8fc-720be24a3fd9",
"user_id": "USER_ID",
"book_id": "7b6d6402-e7f2-42e1-9445-19ec1bd154a2",
"text": "Дуже сподобалась книга!",
"createdAt": "2025-06-25T08:00:00.000Z"
}

### Оновлення коментаря

PATCH /comments/update/:id

Authorization: Bearer <accessToken>

Тіло:
text: string

>Користувач може редагувати лише свої коментарі.

### Видалення коментаря

DELETE /comments/delete/:id

Authorization: Bearer <accessToken>

>Користувач може видалити лише свої коментарі.

### Отримання списку коментарів

GET /comments/list

Query-параметри:

| Параметр  | Тип    | Опис                                     |
| --------- | ------ | ---------------------------------------- |
| `page`    | string | Номер сторінки (default: `1`)            |
| `limit`   | string | Кількість на сторінку (default: `10`)    |
| `sort`    | string | Поле сортування (наприклад, `createdAt`) |
| `order`   | string | `'ASC'` або `'DESC'` (default: `ASC`)    |
| `book_id` | string | Фільтр по ID книги                       |
| `user_id` | string | Фільтр по ID користувача                 |

### Отримання одного коментаря

GET /comments/find/:id

# 📰 News

Модуль для керування новинами: створення, оновлення, отримання списку, фільтрація, перегляд по ID та видалення. Підтримується завантаження зображень та категоризація новин на типи: general, promotion, event. Тільки адміністратор (Admin) має доступ до створення, редагування та видалення новин.

### Створення новини

POST /news/create

Authorization: Bearer <accessToken>

Тіло:
title:string
content:string	
category:string	'general' | 'promotion' | 'event'
image?:file (опціональне)

Відповідь:
{
"id": "1",
"title": "Новий розділ відкрито",
"content": "...",
"category": "event",
"image": "https://your-storage.com/news-images/123.jpg",
"createdAt": "2025-06-25T10:00:00.000Z"
}

### Оновлення новини

PATCH /news/update/:id

Authorization: Bearer <accessToken>

Тіло(всі поля опціональні):
title?: string,
content?: string,
category?: string 'general' | 'promotion' | 'event'

### Видалення новини

DELETE /news/delete/:id

Authorization: Bearer <accessToken>

###  Отримання списку новин

GET /news/list

Query-параметри:

| Параметр   | Тип    | Опис                                            |
| ---------- | ------ | ----------------------------------------------- |
| `page`     | string | Номер сторінки (default: `1`)                   |
| `limit`    | string | Кількість новин на сторінку (default: `10`)     |
| `sort`     | string | Поле для сортування (`title`, `createdAt` тощо) |
| `order`    | string | `'ASC'` або `'DESC'` (default: `'ASC'`)         |
| `title`    | string | Пошук новин за частиною заголовку               |
| `category` | string | `'general'` \| `'promotion'` \| `'event'`       |

### Отримання новини за ID

GET /news/find/id/:id

# 👤 Users

Модуль для роботи з користувачами: отримання списку, перегляд профілю, оновлення персональних даних, зміна ролі (для адмінів) та видалення.

### Отримання списку користувачів

GET /users/list

Query-параметри:

| Параметр    | Тип    | Опис                                           |
| ----------- | ------ | ---------------------------------------------- |
| `page`      | string | Номер сторінки (default: `1`)                  |
| `limit`     | string | Кількість на сторінку (default: `10`)          |
| `sort`      | string | Поле для сортування (`firstName`, `age`, тощо) |
| `order`     | string | `'ASC'` або `'DESC'` (default: `'ASC'`)        |
| `firstName` | string | Пошук по імені                                 |
| `lastName`  | string | Пошук по прізвищу                              |
| `email`     | string | Фільтр по email                                |
| `phone`     | string | Фільтр по номеру телефону                      |
| `age`       | number | Фільтр по віку                                 |

### Отримання користувача за ID

GET /users/find/:id

### Отримання користувача з JWT (себе)

GET /users/find

Authorization: Bearer <accessToken>

>Повертає дані про авторизованого користувача на основі токена. 

### Оновлення користувач

PATCH /users/update/:id

Authorization: Bearer <accessToken>

Тіло: (Всі поля опціональні)
firstName?: string;
lastName?: string;
phone?: string;
age?: number;

>Користувач може оновлювати лише свій профіль.

### Зміна ролі користувача

PATCH /users/role/:id

Authorization: Bearer <accessToken>

>Тільки адміністратор має доступ до зміни ролей.
Внутрішньо змінюється роль користувача (наприклад, з User на Admin або навпаки).

### Видалення користувача

DELETE /users/delete/:id

Authorization: Bearer <accessToken>

>Користувач може видалити свій профіль.

### Видалення користувача адмінстратором

DELETE /users/exclude/:id

Authorization: Bearer <accessToken>

>Адміністратор може видаляти тільки користувачів, role = User.

# 👍 Likes

Модуль дозволяє користувачам додавати/видаляти улюблені книги, переглядати свої лайки та бачити кількість лайків для кожної книги.

>При додаванні/видаленні лайку оновлюється поле likesCount у сутності Book.

### Додати лайк

POST /likes/:bookId

Authorization: Bearer <accessToken>

>Якщо лайк вже існував — буде видалено

### Видаляє лайкy

DELETE /likes/:bookId

Authorization: Bearer <accessToken>

Видаляє лайк для вказаної книги.

Відповідь:
{
"message": "Like removed"
}

### Список вподобаних книг

GET /likes/list

Authorization: Bearer <accessToken>

Повертає список усіх книг, які користувач вподобав.

Параметри запиту (необов'язково) – доступні всі поля з BookQueryDto,

>Запити до /likes/list використовують QueryBuilder з повною підтримкою фільтрації, сортування та пагінації.

Відповідь:
{
"page": 1,
"pages": 3,
"countItems": 25,
"entities": [
{
"id": "bookId",
"title": "Book title",
...
}
]
}

### Кількість лайків

GET /likes/count/:bookId

Authorization: Bearer <accessToken>

Повертає кількість лайків для конкретної книги.

Відповідь:
{
"count": 12
}