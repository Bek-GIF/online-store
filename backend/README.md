# Backend Documentation — Online Store API

## 1. Описание проекта

Backend интернет-магазина — REST API для управления пользователями, товарами, категориями, корзиной и заказами.

Основные возможности:

- Регистрация и авторизация пользователей.
- JWT Access Token и Refresh Token.
- Разграничение доступа по ролям `USER` и `ADMIN`.
- CRUD товаров и категорий.
- Поиск, фильтрация, сортировка и пагинация.
- Управление корзиной.
- Оформление и отслеживание заказов.
- Административное управление пользователями и заказами.
- Валидация данных и обработка ошибок.
- Автоматизированные миграции базы данных через Prisma.

## 2. Технологический стек

| Технология        | Назначение                  |
| ----------------- | --------------------------- |
| Node.js           | Серверная среда выполнения  |
| Express           | Создание REST API           |
| TypeScript        | Типизация кода              |
| PostgreSQL        | Реляционная база данных     |
| Supabase          | Размещение PostgreSQL       |
| Prisma ORM        | Работа с БД и миграции      |
| JWT               | Авторизация                 |
| bcrypt            | Хеширование паролей         |
| Zod               | Валидация данных            |
| Swagger / OpenAPI | Документация API            |

## 3. Архитектура backend

Каждый запрос должен проходить через соответствующие уровни приложения.

```
Client
  ↓
Express App
  ↓
Routes
  ↓
Middleware
  ↓
Controller
  ↓
Service
  ↓
Prisma Client
  ↓
Supabase PostgreSQL
```

### Routes

Отвечают за:

- Определение HTTP-методов и URL.
- Подключение middleware.
- Передачу управления контроллерам.

Routes не должны содержать бизнес-логику и запросы к базе данных.

### Controllers

Отвечают за:

- Получение параметров запроса.
- Вызов методов сервисов.
- Формирование HTTP-ответов.
- Выбор подходящего HTTP-статуса.

Контроллеры не должны содержать сложные бизнес-правила.

### Services

Содержат бизнес-логику приложения:

- Проверку наличия товара.
- Расчёт стоимости заказа.
- Проверку остатков.
- Создание заказа.
- Изменение статусов.
- Работу с пользователями и категориями.

Все операции с базой данных выполняются через Prisma.

### Middlewares

Используются для:

- Проверки JWT.
- Проверки ролей.
- Валидации запросов.
- Обработки ошибок.
- Логирования запросов при необходимости.

## 4. Структура папок

```
backend/
├── src/
│   ├── config/
│   │   ├── env.ts
│   │   └── swagger.ts
│   ├── lib/
│   │   └── prisma.ts
│   ├── routes/
│   │   ├── index.ts
│   │   ├── auth.routes.ts
│   │   ├── product.routes.ts
│   │   ├── category.routes.ts
│   │   ├── cart.routes.ts
│   │   ├── order.routes.ts
│   │   ├── user.routes.ts
│   │   └── admin.routes.ts
│   ├── controllers/
│   │   ├── auth.controller.ts
│   │   ├── product.controller.ts
│   │   ├── category.controller.ts
│   │   ├── cart.controller.ts
│   │   ├── order.controller.ts
│   │   └── admin.controller.ts
│   ├── services/
│   │   ├── auth.service.ts
│   │   ├── product.service.ts
│   │   ├── category.service.ts
│   │   ├── cart.service.ts
│   │   ├── order.service.ts
│   │   └── admin.service.ts
│   ├── middlewares/
│   │   ├── auth.middleware.ts
│   │   ├── role.middleware.ts
│   │   ├── validate.middleware.ts
│   │   ├── error.middleware.ts
│   │   └── not-found.middleware.ts
│   ├── schemas/
│   │   ├── auth.schema.ts
│   │   ├── product.schema.ts
│   │   ├── category.schema.ts
│   │   ├── cart.schema.ts
│   │   └── order.schema.ts
│   ├── utils/
│   ├── types/
│   ├── app.ts
│   └── server.ts
├── prisma/
│   ├── schema.prisma
│   ├── migrations/
│   └── seed.ts
├── tests/
├── .env.example
├── .gitignore
├── package.json
├── tsconfig.json
└── README.md
```

Создавайте дополнительные файлы по мере необходимости. Не добавляйте пустые слои и файлы только ради соответствия структуре.

## 5. База данных

База данных — PostgreSQL в Supabase. Prisma используется для описания моделей, связей, запросов и миграций.

### User

| Поле      | Тип           | Описание          |
| --------- | ------------- | ----------------- |
| id        | String / UUID | Первичный ключ    |
| name      | String        | Имя пользователя  |
| email     | String        | Уникальный email  |
| password  | String        | Хеш пароля        |
| role      | Enum          | `USER` или `ADMIN` |
| createdAt | DateTime      | Дата регистрации  |
| updatedAt | DateTime      | Дата изменения    |

### Product

| Поле        | Тип           | Описание           |
| ----------- | ------------- | ------------------ |
| id          | String / UUID | Первичный ключ     |
| title       | String        | Название товара    |
| description | String        | Описание           |
| price       | Decimal       | Цена               |
| image       | String?       | URL изображения    |
| stock       | Int           | Остаток            |
| categoryId  | String        | Связь с категорией |
| createdAt   | DateTime      | Дата создания      |
| updatedAt   | DateTime      | Дата изменения     |

### Category

| Поле      | Тип           | Описание             |
| --------- | ------------- | -------------------- |
| id        | String / UUID | Первичный ключ       |
| name      | String        | Уникальное название  |
| createdAt | DateTime      | Дата создания        |

### Cart

Корзина принадлежит одному пользователю.

Поля: `id`, `userId`, `createdAt`, `updatedAt`.

### CartItem

Поля: `id`, `cartId`, `productId`, `quantity`.

Одна и та же корзина не должна содержать несколько отдельных записей одного товара. Используйте уникальное ограничение на пару `cartId + productId`.

### Order

Поля: `id`, `userId`, `totalPrice`, `status`, `createdAt`, `updatedAt`.

Статусы:

- `PENDING`
- `CONFIRMED`
- `SHIPPED`
- `DELIVERED`
- `CANCELLED`

### OrderItem

Поля: `id`, `orderId`, `productId`, `quantity`, `price`.

`price` — цена единицы товара на момент покупки. Историческая цена не должна меняться при последующем редактировании товара.

### Основные связи

```
User 1 ─── 1 Cart
Cart 1 ─── N CartItem
Product 1 ─── N CartItem

Category 1 ─── N Product

User 1 ─── N Order
Order 1 ─── N OrderItem
Product 1 ─── N OrderItem
```

Для внешних ключей, уникальности email и названий категорий, ограничений количества и денежных полей настройте подходящие ограничения на уровне Prisma и PostgreSQL.

## 6. Миграции Prisma

`schema.prisma` является источником истины для структуры базы данных, управляемой Prisma.

Создание миграции в разработке:

```shell
npx prisma migrate dev --name init
```

Генерация Prisma Client:

```shell
npx prisma generate
```

Применение миграций в production:

```shell
npx prisma migrate deploy
```

Просмотр базы данных:

```shell
npx prisma studio
```

Правила:

1. Сначала измените `schema.prisma`.
2. Создайте миграцию.
3. Проверьте SQL миграции.
4. Сгенерируйте Prisma Client.
5. Проверьте работу затронутых endpoints.

Не используйте `prisma db push` как замену версионируемым миграциям в основном командном workflow. Не удаляйте историю миграций ради устранения ошибки.

Для Supabase используйте подходящие параметры подключения: `DATABASE_URL` для рабочего соединения и `DIRECT_URL`, если выбранный способ подключения требует прямого соединения для миграций. Конкретные URL и настройки должны соответствовать конфигурации Supabase и используемой версии Prisma.

## 7. Переменные окружения

Файл `.env.example`:

```shell
PORT=5000
NODE_ENV=development

DATABASE_URL=
DIRECT_URL=

JWT_ACCESS_SECRET=
JWT_REFRESH_SECRET=

CLIENT_URL=http://localhost:3000
```

Не добавляйте реальные секреты в `.env.example` или Git.

Дополнительные правила:

- Проверяйте наличие обязательных переменных при старте сервера.
- Не передавайте секреты в Next.js-клиент.
- Не публикуйте строки подключения к базе данных.
- Не используйте одинаковые секреты JWT для access и refresh токенов.

## 8. API Endpoints

Все endpoints располагаются под префиксом `/api`.

### Authentication

| Method | Endpoint             | Доступ                      |
| ------ | -------------------- | --------------------------- |
| POST   | `/api/auth/register` | Public                      |
| POST   | `/api/auth/login`    | Public                      |
| POST   | `/api/auth/refresh`  | Refresh token               |
| POST   | `/api/auth/logout`   | Авторизованный пользователь |
| GET    | `/api/auth/profile`  | USER / ADMIN                |

Регистрация создаёт пользователя с ролью `USER`. Роль `ADMIN` нельзя назначить через публичную регистрацию.

### Products

| Method | Endpoint            | Доступ |
| ------ | ------------------- | ------ |
| GET    | `/api/products`     | Public |
| GET    | `/api/products/:id` | Public |
| POST   | `/api/products`     | ADMIN  |
| PATCH  | `/api/products/:id` | ADMIN  |
| DELETE | `/api/products/:id` | ADMIN  |

Поддерживаемые query-параметры:

```
search
categoryId
minPrice
maxPrice
sort
page
limit
```

Пример:

```
GET /api/products?search=iphone&minPrice=100&maxPrice=1000&page=1&limit=12&sort=price_asc
```

Поддерживаемые варианты сортировки должны быть явно определены в API, например `price_asc`, `price_desc`, `newest`.

### Categories

| Method | Endpoint              | Доступ |
| ------ | --------------------- | ------ |
| GET    | `/api/categories`     | Public |
| GET    | `/api/categories/:id` | Public |
| POST   | `/api/categories`     | ADMIN  |
| PATCH  | `/api/categories/:id` | ADMIN  |
| DELETE | `/api/categories/:id` | ADMIN  |

При удалении категории необходимо обработать товары, связанные с ней. Не допускайте случайного удаления связанных данных.

### Cart

| Method | Endpoint              | Доступ |
| ------ | --------------------- | ------ |
| GET    | `/api/cart`           | USER   |
| POST   | `/api/cart/items`     | USER   |
| PATCH  | `/api/cart/items/:id` | USER   |
| DELETE | `/api/cart/items/:id` | USER   |
| DELETE | `/api/cart`           | USER   |

Пользователь может управлять только собственной корзиной. Цена берётся из базы данных, а не из переданных клиентом значений.

### Orders

| Method | Endpoint          | Доступ                    |
| ------ | ----------------- | ------------------------- |
| POST   | `/api/orders`     | USER                      |
| GET    | `/api/orders`     | USER                      |
| GET    | `/api/orders/:id` | Владелец заказа или ADMIN |

Пользователь должен видеть только собственные заказы.

### Admin

| Method | Endpoint                       | Доступ |
| ------ | ------------------------------ | ------ |
| GET    | `/api/admin/orders`            | ADMIN  |
| PATCH  | `/api/admin/orders/:id/status` | ADMIN  |
| GET    | `/api/admin/users`             | ADMIN  |

Для списков предусмотрите пагинацию и фильтры, где они нужны.

## 9. Авторизация и безопасность

### JWT

Используются два типа токенов:

- **Access Token** — для авторизации API-запросов.
- **Refresh Token** — для получения нового Access Token.

Access Token должен иметь ограниченный срок действия. Refresh Token должен проверяться отдельно, иметь срок действия и поддерживать безопасный logout.

При использовании cookies:

- Refresh Token храните в `HttpOnly` cookie.
- Настройте `Secure` для HTTPS в production.
- Настройте `SameSite` с учётом архитектуры приложения.
- Учитывайте CSRF-риски и правила CORS.
- При необходимости поддерживайте отзыв или ротацию refresh-токенов.

### Middleware

`authMiddleware`:

- Получает Access Token.
- Проверяет подпись и срок действия.
- Получает идентификатор пользователя.
- Передаёт данные авторизации дальше.

`roleMiddleware`:

- Проверяет наличие авторизованного пользователя.
- Проверяет его роль.
- Возвращает `403 Forbidden`, если доступа недостаточно.

Проверки выполняются на backend, даже если frontend скрывает административные кнопки.

### Пароли

- Хешируйте пароли через bcrypt.
- Никогда не возвращайте пароль или его хеш клиенту.
- Не храните пароль в открытом виде.
- Не выводите токены и пароли в логах.

## 10. Создание заказа

Оформление заказа должно выполняться в транзакции.

Последовательность:

```
Получить корзину пользователя
        ↓
Проверить наличие товаров
        ↓
Проверить количество на складе
        ↓
Получить актуальные цены из БД
        ↓
Рассчитать итоговую стоимость
        ↓
Создать Order
        ↓
Создать OrderItems
        ↓
Уменьшить остатки товаров
        ↓
Очистить корзину
```

Все перечисленные изменения должны быть атомарными: если операция не удалась, изменения заказа, остатков и корзины не должны частично сохраниться.

Для предотвращения продажи одного остатка нескольким покупателям используйте безопасное обновление остатков с проверкой количества, учитывая параллельные запросы.

Правила:

- Не доверяйте цене, переданной frontend.
- Не допускайте отрицательного количества.
- Проверяйте, что товаров достаточно.
- Сохраняйте цену товара в `OrderItem`.
- Проверяйте допустимость переходов между статусами заказа.
- Не разрешайте пользователю получать чужие заказы.

## 11. Валидация и обработка ошибок

Используйте Zod для проверки:

- `req.body`
- `req.params`
- `req.query`

Пример схемы товара:

```ts
import { z } from "zod";

export const createProductSchema = z.object({
  title: z.string().trim().min(2).max(150),
  description: z.string().trim().min(1),
  price: z.number().positive(),
  image: z.string().url().optional(),
  stock: z.number().int().min(0),
  categoryId: z.string().min(1),
});
```

При необходимости адаптируйте типы под выбранную схему идентификаторов и способ передачи данных.

### HTTP-статусы

| Код | Значение                                                           |
| --- | ------------------------------------------------------------------ |
| 200 | Успешный запрос                                                    |
| 201 | Ресурс создан                                                      |
| 204 | Успешная операция без тела ответа                                  |
| 400 | Неверные данные запроса                                            |
| 401 | Не авторизован                                                     |
| 403 | Нет прав доступа                                                   |
| 404 | Ресурс не найден                                                   |
| 409 | Конфликт, например дублирующийся email                             |
| 422 | Семантически недопустимые данные, если используется такая договорённость |
| 500 | Внутренняя ошибка сервера                                          |

Используйте единый формат ошибок:

```json
{
  "message": "Product not found"
}
```
