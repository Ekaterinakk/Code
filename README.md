```openapi: 3.0.3

info:
  title: API цветочного магазина «Cactus»
  description: |
    API для каталога цветов, букетов, оформления заказов и управления доставкой.
  version: 1.0.0
  contact:
    name: Поддержка «Cactus»
    email: support@floramir.example

servers:
  - url: https://api.floramir.example/v1
    description: Production server
  - url: https://sandbox.floramir.example/v1
    description: Sandbox server

tags:
  - name: Catalog
    description: Каталог цветов и букетов
  - name: Orders
    description: Работа с заказами
  - name: Customers
    description: Покупатели
  - name: Delivery
    description: Расчёт доставки

paths:

  /products:
    get:
      tags:
        - Catalog
      summary: Получить список товаров
      parameters:
        - $ref: '#/components/parameters/Page'
        - $ref: '#/components/parameters/Limit'
        - name: category
          in: query
          description: Категория товара
          schema:
            type: string
            enum:
              - bouquets
              - roses
              - tulips
              - gifts
              - plants
        - name: color
          in: query
          description: Основной цвет
          schema:
            type: string
        - name: minPrice
          in: query
          schema:
            type: number
            format: float
            minimum: 0
        - name: maxPrice
          in: query
          schema:
            type: number
            format: float
            minimum: 0
        - name: search
          in: query
          description: Поиск по названию
          schema:
            type: string
      responses:
        '200':
          description: Список товаров
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ProductListResponse'
        '400':
          $ref: '#/components/responses/BadRequest'

    post:
      tags:
        - Catalog
      summary: Создать товар
      security:
        - bearerAuth: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/ProductCreateRequest'
      responses:
        '201':
          description: Товар создан
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Product'
        '400':
          $ref: '#/components/responses/BadRequest'
        '401':
          $ref: '#/components/responses/Unauthorized'

  /products/{productId}:
    get:
      tags:
        - Catalog
      summary: Получить товар по идентификатору
      parameters:
        - $ref: '#/components/parameters/ProductId'
      responses:
        '200':
          description: Информация о товаре
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Product'
        '404':
          $ref: '#/components/responses/NotFound'

    patch:
      tags:
        - Catalog
      summary: Обновить товар
      security:
        - bearerAuth: []
      parameters:
        - $ref: '#/components/parameters/ProductId'
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/ProductUpdateRequest'
      responses:
        '200':
          description: Товар обновлён
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Product'
        '401':
          $ref: '#/components/responses/Unauthorized'
        '404':
          $ref: '#/components/responses/NotFound'

    delete:
      tags:
        - Catalog
      summary: Удалить товар
      security:
        - bearerAuth: []
      parameters:
        - $ref: '#/components/parameters/ProductId'
      responses:
        '204':
          description: Товар удалён
        '401':
          $ref: '#/components/responses/Unauthorized'
        '404':
          $ref: '#/components/responses/NotFound'

  /orders:
    get:
      tags:
        - Orders
      summary: Получить список заказов
      security:
        - bearerAuth: []
      parameters:
        - $ref: '#/components/parameters/Page'
        - $ref: '#/components/parameters/Limit'
        - name: status
          in: query
          schema:
            $ref: '#/components/schemas/OrderStatus'
        - name: customerId
          in: query
          schema:
            type: string
            format: uuid
      responses:
        '200':
          description: Список заказов
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/OrderListResponse'
        '401':
          $ref: '#/components/responses/Unauthorized'

    post:
      tags:
        - Orders
      summary: Оформить заказ
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/OrderCreateRequest'
      responses:
        '201':
          description: Заказ создан
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Order'
        '400':
          $ref: '#/components/responses/BadRequest'
        '409':
          description: Недостаточно товара на складе
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'

  /orders/{orderId}:
    get:
      tags:
        - Orders
      summary: Получить заказ
      security:
        - bearerAuth: []
      parameters:
        - $ref: '#/components/parameters/OrderId'
      responses:
        '200':
          description: Информация о заказе
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Order'
        '401':
          $ref: '#/components/responses/Unauthorized'
        '404':
          $ref: '#/components/responses/NotFound'

  /orders/{orderId}/status:
    patch:
      tags:
        - Orders
      summary: Изменить статус заказа
      security:
        - bearerAuth: []
      parameters:
        - $ref: '#/components/parameters/OrderId'
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/OrderStatusUpdateRequest'
      responses:
        '200':
          description: Статус обновлён
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Order'
        '400':
          $ref: '#/components/responses/BadRequest'
        '401':
          $ref: '#/components/responses/Unauthorized'
        '404':
          $ref: '#/components/responses/NotFound'

  /customers:
    post:
      tags:
        - Customers
      summary: Зарегистрировать покупателя
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CustomerCreateRequest'
      responses:
        '201':
          description: Покупатель зарегистрирован
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Customer'
        '400':
          $ref: '#/components/responses/BadRequest'
        '409':
          description: Покупатель с таким email уже существует
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'

  /customers/{customerId}:
    get:
      tags:
        - Customers
      summary: Получить данные покупателя
      security:
        - bearerAuth: []
      parameters:
        - $ref: '#/components/parameters/CustomerId'
      responses:
        '200':
          description: Данные покупателя
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Customer'
        '401':
          $ref: '#/components/responses/Unauthorized'
        '404':
          $ref: '#/components/responses/NotFound'

  /delivery/calculate:
    post:
      tags:
        - Delivery
      summary: Рассчитать стоимость доставки
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/DeliveryCalculateRequest'
      responses:
        '200':
          description: Стоимость доставки рассчитана
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/DeliveryCalculation'
        '400':
          $ref: '#/components/responses/BadRequest'

components:

  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT

  parameters:
    ProductId:
      name: productId
      in: path
      required: true
      description: Идентификатор товара
      schema:
        type: string
        format: uuid

    OrderId:
      name: orderId
      in: path
      required: true
      description: Идентификатор заказа
      schema:
        type: string
        format: uuid

    CustomerId:
      name: customerId
      in: path
      required: true
      description: Идентификатор покупателя
      schema:
        type: string
        format: uuid

    Page:
      name: page
      in: query
      description: Номер страницы
      schema:
        type: integer
        minimum: 1
        default: 1

    Limit:
      name: limit
      in: query
      description: Количество элементов на странице
      schema:
        type: integer
        minimum: 1
        maximum: 100
        default: 20

  responses:
    BadRequest:
      description: Некорректные данные запроса
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'

    Unauthorized:
      description: Требуется авторизация
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'

    NotFound:
      description: Ресурс не найден
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'

  schemas:

    Product:
      type: object
      required:
        - id
        - name
        - category
        - price
        - currency
        - available
      properties:
        id:
          type: string
          format: uuid
          example: 7f8c0f92-7e21-4d8d-94dc-95f034a8d501
        name:
          type: string
          example: Букет «Весеннее настроение»
        description:
          type: string
          example: Нежный букет из тюльпанов и эвкалипта
        category:
          type: string
          enum:
            - bouquets
            - roses
            - tulips
            - gifts
            - plants
          example: bouquets
        price:
          type: number
          format: float
          minimum: 0
          example: 3490
        currency:
          type: string
          example: RUB
        color:
          type: string
          example: Розовый
        imageUrl:
          type: string
          format: uri
          example: https://cdn.floramir.example/products/spring-mood.jpg
        available:
          type: boolean
          example: true
        stock:
          type: integer
          minimum: 0
          example: 12
        createdAt:
          type: string
          format: date-time
          example: 2025-03-08T10:00:00Z

    ProductCreateRequest:
      type: object
      required:
        - name
        - category
        - price
      properties:
        name:
          type: string
          minLength: 2
          example: Букет «Весеннее настроение»
        description:
          type: string
          example: Нежный букет из тюльпанов и эвкалипта
        category:
          type: string
          enum:
            - bouquets
            - roses
            - tulips
            - gifts
            - plants
        price:
          type: number
          format: float
          minimum: 0
          example: 3490
        currency:
          type: string
          default: RUB
          example: RUB
        color:
          type: string
          example: Розовый
        imageUrl:
          type: string
          format: uri
        stock:
          type: integer
          minimum: 0
          default: 0

    ProductUpdateRequest:
      type: object
      properties:
        name:
          type: string
        description:
          type: string
        price:
          type: number
          format: float
          minimum: 0
        color:
          type: string
        imageUrl:
          type: string
          format: uri
        stock:
          type: integer
          minimum: 0
        available:
          type: boolean

    ProductListResponse:
      type: object
      required:
        - items
        - pagination
      properties:
        items:
          type: array
          items:
            $ref: '#/components/schemas/Product'
        pagination:
          $ref: '#/components/schemas/Pagination'

    Order:
      type: object
      required:
        - id
        - customer
        - items
        - total
        - status
        - delivery
        - createdAt
      properties:
        id:
          type: string
          format: uuid
          example: 4b7e84c9-ea4e-46ae-9910-6ed5cc1d54a8
        customer:
          $ref: '#/components/schemas/Customer'
        items:
          type: array
          items:
            $ref: '#/components/schemas/OrderItem'
        subtotal:
          type: number
          format: float
          example: 3490
        deliveryCost:
          type: number
          format: float
          example: 400
        total:
          type: number
          format: float
          example: 3890
        currency:
          type: string
          example: RUB
        status:
          $ref: '#/components/schemas/OrderStatus'
        delivery:
          $ref: '#/components/schemas/Delivery'
        comment:
          type: string
          example: Позвонить получателю перед доставкой
        createdAt:
          type: string
          format: date-time
        updatedAt:
          type: string
          format: date-time

    OrderCreateRequest:
      type: object
      required:
        - customerId
        - items
        - delivery
      properties:
        customerId:
          type: string
          format: uuid
        items:
          type: array
          minItems: 1
          items:
            $ref: '#/components/schemas/OrderItemRequest'
        delivery:
          $ref: '#/components/schemas/Delivery'
        comment:
          type: string
          maxLength: 500

    OrderItem:
      type: object
      required:
        - product
        - quantity
        - price
        - total
      properties:
        product:
          $ref: '#/components/schemas/Product'
        quantity:
          type: integer
          minimum: 1
          example: 2
        price:
          type: number
          format: float
          example: 3490
        total:
          type: number
          format: float
          example: 6980

    OrderItemRequest:
      type: object
      required:
        - productId
        - quantity
      properties:
        productId:
          type: string
          format: uuid
        quantity:
          type: integer
          minimum: 1
          example: 1

    OrderStatus:
      type: string
      enum:
        - pending
        - confirmed
        - assembling
        - delivering
        - completed
        - cancelled
      example: pending

    OrderStatusUpdateRequest:
      type: object
      required:
        - status
      properties:
        status:
          $ref: '#/components/schemas/OrderStatus'

    OrderListResponse:
      type: object
      properties:
        items:
          type: array
          items:
            $ref: '#/components/schemas/Order'
        pagination:
          $ref: '#/components/schemas/Pagination'

    Customer:
      type: object
      required:
        - id
        - name
        - phone
        - email
      properties:
        id:
          type: string
          format: uuid
          example: 6ab8f6db-b7ef-46df-ae25-b2ec5fcf6bc3
        name:
          type: string
          example: Анна Петрова
        phone:
          type: string
          example: +79991234567
        email:
          type: string
          format: email
          example: anna@example.com
        createdAt:
          type: string
          format: date-time

    CustomerCreateRequest:
      type: object
      required:
        - name
        - phone
        - email
      properties:
        name:
          type: string
          example: Анна Петрова
        phone:
          type: string
          pattern: '^\+?[0-9 ()-]{10,20}$'
          example: +79991234567
        email:
          type: string
          format: email
          example: anna@example.com

    Delivery:
      type: object
      required:
        - address
        - date
        - timeSlot
      properties:
        address:
          $ref: '#/components/schemas/Address'
        date:
          type: string
          format: date
          example: 2025-04-15
        timeSlot:
          type: string
          enum:
            - "09:00-12:00"
            - "12:00-15:00"
            - "15:00-18:00"
            - "18:00-21:00"
          example: "15:00-18:00"
        recipientName:
          type: string
          example: Мария Иванова
        recipientPhone:
          type: string
          example: +79990001122

    Address:
      type: object
      required:
        - city
        - street
        - house
      properties:
        city:
          type: string
          example: Москва
        street:
          type: string
          example: Тверская
        house:
          type: string
          example: 12
        apartment:
          type: string
          example: 45
        entrance:
          type: string
          example: 2
        floor:
          type: string
          example: 5
        comment:
          type: string
          example: Домофон не работает

    DeliveryCalculateRequest:
      type: object
      required:
        - address
      properties:
        address:
          $ref: '#/components/schemas/Address'

    DeliveryCalculation:
      type: object
      required:
        - cost
        - currency
        - estimatedMinutes
      properties:
        cost:
          type: number
          format: float
          example: 400
        currency:
          type: string
          example: RUB
        estimatedMinutes:
          type: integer
          example: 90
        available:
          type: boolean
          example: true

    Pagination:
      type: object
      properties:
        page:
          type: integer
          example: 1
        limit:
          type: integer
          example: 20
        totalItems:
          type: integer
          example: 48
        totalPages:
          type: integer
          example: 3

    Error:
      type: object
      required:
        - code
        - message
      properties:
        code:
          type: string
          example: PRODUCT_NOT_FOUND
        message:
          type: string
          example: Товар не найден
        details:
          type: object
          additionalProperties: true
```
