# Лабораторная работа №12. JDBC: транзакции и пакетная обработка

## 1. Паспорт работы

| Параметр | Значение |
|----------|----------|
| Номер занятия | 12 из 17 |
| Блок | 5. Базы данных и JDBC |
| Продолжительность | 2 академических часа |
| Форма выполнения | Индивидуальная |
| ИИ-инструмент | YandexGPT / GigaChat / JetBrains AI Assistant |

### 1.1. Цель работы

Освоить управление транзакциями в JDBC, научиться выполнять пакетную обработку данных, использовать пул соединений для повышения производительности, реализовать UPSERT-операции для обеспечения идемпотентности.

### 1.2. Задачи работы

1. Изучить управление транзакциями в JDBC (setAutoCommit, commit, rollback).
2. Освоить пакетную обработку (batch insert) для массовой вставки данных.
3. Изучить использование пула соединений HikariCP.
4. Реализовать UPSERT (INSERT ... ON CONFLICT DO UPDATE).
5. Научиться замерять производительность различных подходов.
6. Реализовать транзакционный метод для переназначения книг.
7. Сравнить производительность с пулом и без пула соединений.

### 1.3. Оснащение

- JDK 17 или выше;
- IntelliJ IDEA Community Edition;
- Git;
- PostgreSQL (локально или в Docker);
- Maven или Gradle;
- HikariCP;
- доступ к YandexGPT или GigaChat.

---

## 2. Теоретический конспект

### 2.1. Транзакции в JDBC

**Транзакция** — группа операций, которые выполняются как единое целое (ACID).

**ACID:**
- **Atomicity** — атомарность (всё или ничего).
- **Consistency** — согласованность.
- **Isolation** — изоляция.
- **Durability** — долговечность.

**Управление транзакциями в JDBC:**

```java
// По умолчанию autoCommit = true
Connection conn = DriverManager.getConnection(url, user, password);

// Отключение autoCommit (начало транзакции)
conn.setAutoCommit(false);

try {
    // Выполнение операций
    stmt.executeUpdate("UPDATE accounts SET balance = balance - 100 WHERE id = 1");
    stmt.executeUpdate("UPDATE accounts SET balance = balance + 100 WHERE id = 2");
    
    // Фиксация транзакции
    conn.commit();
    System.out.println("Транзакция успешно выполнена");
} catch (SQLException e) {
    // Откат при ошибке
    conn.rollback();
    System.err.println("Транзакция откачена: " + e.getMessage());
} finally {
    // Восстановление autoCommit
    conn.setAutoCommit(true);
}
```

### 2.2. Точки сохранения (Savepoints)

```java
conn.setAutoCommit(false);

try {
    stmt.executeUpdate("INSERT INTO books (title) VALUES ('Book 1')");
    
    // Установка точки сохранения
    Savepoint sp = conn.setSavepoint("before_book2");
    
    stmt.executeUpdate("INSERT INTO books (title) VALUES ('Book 2')");
    // Если ошибка, откат к точке сохранения
    conn.rollback(sp);
    System.out.println("Откат к точке сохранения");
    
    conn.commit();
} catch (SQLException e) {
    conn.rollback();
}
```

### 2.3. Пакетная обработка (Batch)

**Преимущества:**
- Меньше сетевых вызовов.
- Выше производительность при массовых вставках.
- Транзакционная целостность.

```java
String sql = "INSERT INTO books (title, author, year, isbn, publisher) VALUES (?, ?, ?, ?, ?)";

try (Connection conn = getConnection();
     PreparedStatement pstmt = conn.prepareStatement(sql)) {
    
    conn.setAutoCommit(false);
    
    for (Book book : books) {
        pstmt.setString(1, book.getTitle());
        pstmt.setString(2, book.getAuthor());
        pstmt.setInt(3, book.getYear());
        pstmt.setString(4, book.getIsbn());
        pstmt.setString(5, book.getPublisher());
        pstmt.addBatch();
    }
    
    int[] results = pstmt.executeBatch();
    conn.commit();
    
    System.out.println("Вставлено: " + results.length + " записей");
}
```

### 2.4. UPSERT (INSERT ON CONFLICT)

**UPSERT** — операция, которая вставляет запись, если она не существует, или обновляет, если существует.

**PostgreSQL синтаксис:**
```sql
INSERT INTO books (title, author, year, isbn, publisher)
VALUES (?, ?, ?, ?, ?)
ON CONFLICT (isbn) DO UPDATE SET
    title = EXCLUDED.title,
    author = EXCLUDED.author,
    year = EXCLUDED.year,
    publisher = EXCLUDED.publisher
```

**В Java:**
```java
String sql = """
    INSERT INTO books (title, author, year, isbn, publisher)
    VALUES (?, ?, ?, ?, ?)
    ON CONFLICT (isbn) DO UPDATE SET
        title = EXCLUDED.title,
        author = EXCLUDED.author,
        year = EXCLUDED.year,
        publisher = EXCLUDED.publisher
""";

try (PreparedStatement pstmt = conn.prepareStatement(sql)) {
    pstmt.setString(1, book.getTitle());
    pstmt.setString(2, book.getAuthor());
    pstmt.setInt(3, book.getYear());
    pstmt.setString(4, book.getIsbn());
    pstmt.setString(5, book.getPublisher());
    pstmt.executeUpdate();
}
```

### 2.5. Пул соединений HikariCP

**HikariCP** — высокопроизводительная библиотека для пула соединений.

**Преимущества:**
- Переиспользование соединений (уменьшение накладных расходов).
- Ограничение количества соединений.
- Управление временем жизни соединений.

**Подключение зависимости:**
```xml
<dependency>
    <groupId>com.zaxxer</groupId>
    <artifactId>HikariCP</artifactId>
    <version>5.0.1</version>
</dependency>
```

**Настройка HikariCP:**
```java
import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;

public class DataSourceProvider {
    private static HikariDataSource dataSource;
    
    public static synchronized HikariDataSource getDataSource() {
        if (dataSource == null) {
            HikariConfig config = new HikariConfig();
            config.setJdbcUrl("jdbc:postgresql://localhost:5432/postgres");
            config.setUsername("postgres");
            config.setPassword("pass");
            config.setDriverClassName("org.postgresql.Driver");
            
            // Настройки пула
            config.setMaximumPoolSize(10);
            config.setMinimumIdle(5);
            config.setConnectionTimeout(30000);
            config.setIdleTimeout(600000);
            config.setMaxLifetime(1800000);
            
            dataSource = new HikariDataSource(config);
        }
        return dataSource;
    }
    
    public static Connection getConnection() throws SQLException {
        return getDataSource().getConnection();
    }
}
```

### 2.6. Сравнение производительности

```java
public void comparePerformance() {
    // Без пула соединений
    long startNoPool = System.currentTimeMillis();
    for (int i = 0; i < 100; i++) {
        try (Connection conn = DriverManager.getConnection(url, user, pass)) {
            // запрос
        }
    }
    long timeNoPool = System.currentTimeMillis() - startNoPool;
    
    // С пулом соединений
    long startPool = System.currentTimeMillis();
    for (int i = 0; i < 100; i++) {
        try (Connection conn = DataSourceProvider.getConnection()) {
            // запрос
        }
    }
    long timePool = System.currentTimeMillis() - startPool;
    
    System.out.printf("Без пула: %d мс%n", timeNoPool);
    System.out.printf("С пулом: %d мс%n", timePool);
    System.out.printf("Ускорение: %.2fx%n", (double) timeNoPool / timePool);
}
```

### 2.7. Транзакции и конкурентность

```java
// Транзакционный метод с проверкой конфликтов
public void transferBooks(long fromAuthorId, long toAuthorId) throws SQLException {
    String checkIsbnSql = "SELECT isbn FROM books WHERE author_id = ? AND isbn IN (SELECT isbn FROM books WHERE author_id = ?)";
    String updateSql = "UPDATE books SET author_id = ? WHERE author_id = ?";
    
    try (Connection conn = DataSourceProvider.getConnection()) {
        conn.setAutoCommit(false);
        conn.setTransactionIsolation(Connection.TRANSACTION_READ_COMMITTED);
        
        // Проверка на конфликт
        try (PreparedStatement checkStmt = conn.prepareStatement(checkIsbnSql)) {
            checkStmt.setLong(1, fromAuthorId);
            checkStmt.setLong(2, toAuthorId);
            try (ResultSet rs = checkStmt.executeQuery()) {
                if (rs.next()) {
                    throw new SQLException("Конфликт ISBN: книга с таким ISBN уже существует у целевого автора");
                }
            }
        }
        
        // Обновление
        try (PreparedStatement updateStmt = conn.prepareStatement(updateSql)) {
            updateStmt.setLong(1, toAuthorId);
            updateStmt.setLong(2, fromAuthorId);
            int count = updateStmt.executeUpdate();
            System.out.println("Переназначено книг: " + count);
        }
        
        conn.commit();
        System.out.println("Транзакция успешно выполнена");
    } catch (SQLException e) {
        // rollback выполняется автоматически, если не вызван commit
        throw e;
    }
}
```

---

## 3. Задание на паре

### Задача. Транзакции и пакетная обработка в JDBC

1. **Реализовать метод `transferBooks(long fromAuthorId, long toAuthorId)`:**
   - Все книги одного автора переназначить другому.
   - Вся операция — в одной транзакции.
   - Если у целевого автора уже есть книга с таким же ISBN — откатывать транзакцию.

2. **Реализовать метод `batchInsert(List<Book> books)`:**
   - Вставка 10 000 записей в одной транзакции.
   - Использовать `PreparedStatement.addBatch()` и `executeBatch()`.
   - Замерить время пакетной вставки vs построчной вставки.

3. **Реализовать метод `saveOrUpdate(Book book)`:**
   - Если книга с таким ISBN уже существует — обновить её.
   - Иначе — вставить (UPSERT).
   - Использовать SQL `ON CONFLICT`.

4. **Настроить HikariCP как пул соединений:**
   - Сравнить время 100 последовательных запросов с пулом и без пула.
   - Проанализировать результаты.

5. **Протестировать все операции в main():**
   - Создать тестовые данные.
   - Выполнить все методы.
   - Вывести результаты.

**Пример выполнения:**
```
=== Переназначение книг ===
Автор 1: 15 книг
Автор 2: 10 книг
Переназначено книг: 15
Транзакция выполнена успешно

=== Пакетная вставка ===
Построчная вставка: 10000 записей за 5234 мс
Пакетная вставка: 10000 записей за 245 мс
Ускорение: 21.36x

=== UPSERT ===
Книга с ISBN '978-5-17-118456-0' обновлена
Книга с ISBN '978-5-17-118457-7' вставлена

=== Сравнение производительности (100 запросов) ===
Без пула: 1856 мс
С пулом (HikariCP): 156 мс
Ускорение: 11.90x
```

### Применение ИИ-инструмента

**Промпт для YandexGPT:**
```
Сгенерируй SQL-запрос INSERT ... ON CONFLICT DO UPDATE для таблицы books в PostgreSQL.

Таблица:
CREATE TABLE books (
    id SERIAL PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    author VARCHAR(255) NOT NULL,
    year INTEGER NOT NULL,
    isbn VARCHAR(20) UNIQUE NOT NULL,
    publisher VARCHAR(255)
);

Требования:
1. Конфликт по полю isbn.
2. При конфликте обновлять все поля кроме id.
3. Возвращать id вставленной/обновлённой записи.
4. Использовать синтаксис PostgreSQL.
```

**Анализ результата:**
- Проверить корректность синтаксиса ON CONFLICT.
- Проверить использование EXCLUDED.
- Проверить возврат id через RETURNING.
- Проверить соответствие полей.

---

## 4. Индивидуальные задания (30 вариантов)

Каждый вариант содержит:
- Два метода с транзакциями.
- Особые условия для конфликтов.
- Особую структуру таблицы.

---

### Вариант 1. Книги (Book)

**Таблица:** `books` (id, title, author, year, isbn, publisher)

**Метод 1:** `transferBooks(long fromAuthorId, long toAuthorId)` — конфликт по isbn

**Метод 2:** `batchInsert(List<Book> books)` — вставка 10000 записей

**UPSERT:** по isbn, обновлять все поля

---

### Вариант 2. Сотрудники (Employee)

**Таблица:** `employees` (id, first_name, last_name, position, salary, department, email)

**Метод 1:** `transferEmployees(long fromDeptId, long toDeptId)` — конфликт по email

**Метод 2:** `batchInsert(List<Employee> employees)` — вставка 5000 записей

**UPSERT:** по email, обновлять все поля

---

### Вариант 3. Товары (Product)

**Таблица:** `products` (id, name, category, price, quantity, supplier, sku)

**Метод 1:** `mergeProducts(long fromSupplierId, long toSupplierId)` — конфликт по sku

**Метод 2:** `batchInsert(List<Product> products)` — вставка 15000 записей

**UPSERT:** по sku, обновлять все поля

---

### Вариант 4. Заказы (Order)

**Таблица:** `orders` (id, customer_name, total_amount, status, created_at, order_number)

**Метод 1:** `mergeOrders(long fromCustomerId, long toCustomerId)` — конфликт по order_number

**Метод 2:** `batchInsert(List<Order> orders)` — вставка 8000 записей

**UPSERT:** по order_number, обновлять все поля

---

### Вариант 5. Пользователи (User)

**Таблица:** `users` (id, username, email, password_hash, role, is_active)

**Метод 1:** `mergeUsers(long fromRoleId, long toRoleId)` — конфликт по email

**Метод 2:** `batchInsert(List<User> users)` — вставка 7000 записей

**UPSERT:** по email, обновлять все поля

---

### Вариант 6. Автомобили (Car)

**Таблица:** `cars` (id, brand, model, year, price, vin, color)

**Метод 1:** `transferCars(long fromBrandId, long toBrandId)` — конфликт по vin

**Метод 2:** `batchInsert(List<Car> cars)` — вставка 12000 записей

**UPSERT:** по vin, обновлять все поля

---

### Вариант 7. Студенты (Student)

**Таблица:** `students` (id, first_name, last_name, group_name, gpa, student_id, birth_date)

**Метод 1:** `transferStudents(long fromGroupId, long toGroupId)` — конфликт по student_id

**Метод 2:** `batchInsert(List<Student> students)` — вставка 10000 записей

**UPSERT:** по student_id, обновлять все поля

---

### Вариант 8. Счета (Account)

**Таблица:** `accounts` (id, account_number, owner_name, balance, currency, opened_date)

**Метод 1:** `transferAccounts(long fromOwnerId, long toOwnerId)` — конфликт по account_number

**Метод 2:** `batchInsert(List<Account> accounts)` — вставка 6000 записей

**UPSERT:** по account_number, обновлять все поля

---

### Вариант 9. Фильмы (Movie)

**Таблица:** `movies` (id, title, director, year, rating, genre, imdb_id)

**Метод 1:** `transferMovies(long fromGenreId, long toGenreId)` — конфликт по imdb_id

**Метод 2:** `batchInsert(List<Movie> movies)` — вставка 9000 записей

**UPSERT:** по imdb_id, обновлять все поля

---

### Вариант 10. Рестораны (Restaurant)

**Таблица:** `restaurants` (id, name, address, phone, rating, cuisine, tax_id)

**Метод 1:** `transferRestaurants(long fromCuisineId, long toCuisineId)` — конфликт по tax_id

**Метод 2:** `batchInsert(List<Restaurant> restaurants)` — вставка 5000 записей

**UPSERT:** по tax_id, обновлять все поля

---

### Вариант 11. Транзакции (Transaction)

**Таблица:** `transactions` (id, amount, type, date, description, category, tx_id)

**Метод 1:** `mergeTransactions(long fromCategoryId, long toCategoryId)` — конфликт по tx_id

**Метод 2:** `batchInsert(List<Transaction> transactions)` — вставка 20000 записей

**UPSERT:** по tx_id, обновлять все поля

---

### Вариант 12. Клиенты (Customer)

**Таблица:** `customers` (id, first_name, last_name, email, phone, city, customer_code)

**Метод 1:** `transferCustomers(long fromCityId, long toCityId)` — конфликт по customer_code

**Метод 2:** `batchInsert(List<Customer> customers)` — вставка 8000 записей

**UPSERT:** по customer_code, обновлять все поля

---

### Вариант 13. Договоры (Contract)

**Таблица:** `contracts` (id, number, client_name, sign_date, amount, status, contract_number)

**Метод 1:** `mergeContracts(long fromStatusId, long toStatusId)` — конфликт по contract_number

**Метод 2:** `batchInsert(List<Contract> contracts)` — вставка 7000 записей

**UPSERT:** по contract_number, обновлять все поля

---

### Вариант 14. Отели (Hotel)

**Таблица:** `hotels` (id, name, city, stars, rooms_count, rating, hotel_code)

**Метод 1:** `transferHotels(long fromCityId, long toCityId)` — конфликт по hotel_code

**Метод 2:** `batchInsert(List<Hotel> hotels)` — вставка 6000 записей

**UPSERT:** по hotel_code, обновлять все поля

---

### Вариант 15. Спортсмены (Athlete)

**Таблица:** `athletes` (id, first_name, last_name, sport, age, medals, athlete_id)

**Метод 1:** `transferAthletes(long fromSportId, long toSportId)` — конфликт по athlete_id

**Метод 2:** `batchInsert(List<Athlete> athletes)` — вставка 5000 записей

**UPSERT:** по athlete_id, обновлять все поля

---

### Вариант 16. Статьи (Article)

**Таблица:** `articles` (id, title, content, author, published_date, views, doi)

**Метод 1:** `mergeArticles(long fromAuthorId, long toAuthorId)` — конфликт по doi

**Метод 2:** `batchInsert(List<Article> articles)` — вставка 8000 записей

**UPSERT:** по doi, обновлять все поля

---

### Вариант 17. Билеты (Ticket)

**Таблица:** `tickets` (id, event_name, venue, date, price, quantity, ticket_code)

**Метод 1:** `transferTickets(long fromVenueId, long toVenueId)` — конфликт по ticket_code

**Метод 2:** `batchInsert(List<Ticket> tickets)` — вставка 12000 записей

**UPSERT:** по ticket_code, обновлять все поля

---

### Вариант 18. Курсы (Course)

**Таблица:** `courses` (id, title, instructor, duration, price, start_date, course_code)

**Метод 1:** `mergeCourses(long fromInstructorId, long toInstructorId)` — конфликт по course_code

**Метод 2:** `batchInsert(List<Course> courses)` — вставка 6000 записей

**UPSERT:** по course_code, обновлять все поля

---

### Вариант 19. Инвентарь (Item)

**Таблица:** `inventory` (id, name, category, quantity, location, status, serial_number)

**Метод 1:** `transferItems(long fromCategoryId, long toCategoryId)` — конфликт по serial_number

**Метод 2:** `batchInsert(List<Item> items)` — вставка 10000 записей

**UPSERT:** по serial_number, обновлять все поля

---

### Вариант 20. Заявки (Request)

**Таблица:** `requests` (id, client_name, description, priority, status, created_at, request_number)

**Метод 1:** `mergeRequests(long fromStatusId, long toStatusId)` — конфликт по request_number

**Метод 2:** `batchInsert(List<Request> requests)` — вставка 7000 записей

**UPSERT:** по request_number, обновлять все поля

---

### Вариант 21. Платежи (Payment)

**Таблица:** `payments` (id, number, payer, recipient, amount, date, payment_id)

**Метод 1:** `transferPayments(long fromPayerId, long toPayerId)` — конфликт по payment_id

**Метод 2:** `batchInsert(List<Payment> payments)` — вставка 15000 записей

**UPSERT:** по payment_id, обновлять все поля

---

### Вариант 22. Публикации (Publication)

**Таблица:** `publications` (id, title, author, journal, year, doi, pub_id)

**Метод 1:** `mergePublications(long fromJournalId, long toJournalId)` — конфликт по doi

**Метод 2:** `batchInsert(List<Publication> publications)` — вставка 5000 записей

**UPSERT:** по doi, обновлять все поля

---

### Вариант 23. Ноутбуки (Laptop)

**Таблица:** `laptops` (id, brand, model, processor, ram, price, serial)

**Метод 1:** `transferLaptops(long fromBrandId, long toBrandId)` — конфликт по serial

**Метод 2:** `batchInsert(List<Laptop> laptops)` — вставка 8000 записей

**UPSERT:** по serial, обновлять все поля

---

### Вариант 24. Врачи (Doctor)

**Таблица:** `doctors` (id, first_name, last_name, specialization, experience, rating, license)

**Метод 1:** `mergeDoctors(long fromSpecializationId, long toSpecializationId)` — конфликт по license

**Метод 2:** `batchInsert(List<Doctor> doctors)` — вставка 6000 записей

**UPSERT:** по license, обновлять все поля

---

### Вариант 25. Сертификаты (Certificate)

**Таблица:** `certificates` (id, number, holder_name, issued_date, expiry_date, cert_id)

**Метод 1:** `transferCertificates(long fromHolderId, long toHolderId)` — конфликт по cert_id

**Метод 2:** `batchInsert(List<Certificate> certificates)` — вставка 7000 записей

**UPSERT:** по cert_id, обновлять все поля

---

### Вариант 26. Магазины (Shop)

**Таблица:** `shops` (id, name, address, phone, opening_hours, type, shop_code)

**Метод 1:** `mergeShops(long fromTypeId, long toTypeId)` — конфликт по shop_code

**Метод 2:** `batchInsert(List<Shop> shops)` — вставка 5000 записей

**UPSERT:** по shop_code, обновлять все поля

---

### Вариант 27. Здания (Building)

**Таблица:** `buildings` (id, address, floors, area, type, year_built, building_id)

**Метод 1:** `transferBuildings(long fromTypeId, long toTypeId)` — конфликт по building_id

**Метод 2:** `batchInsert(List<Building> buildings)` — вставка 4000 записей

**UPSERT:** по building_id, обновлять все поля

---

### Вариант 28. Лекции (Lecture)

**Таблица:** `lectures` (id, topic, speaker, date, duration, attendees, lecture_code)

**Метод 1:** `mergeLectures(long fromSpeakerId, long toSpeakerId)` — конфликт по lecture_code

**Метод 2:** `batchInsert(List<Lecture> lectures)` — вставка 5000 записей

**UPSERT:** по lecture_code, обновлять все поля

---

### Вариант 29. Отзывы (Review)

**Таблица:** `reviews` (id, user_id, product_id, rating, comment, created_at, review_id)

**Метод 1:** `transferReviews(long fromProductId, long toProductId)` — конфликт по review_id

**Метод 2:** `batchInsert(List<Review> reviews)` — вставка 10000 записей

**UPSERT:** по review_id, обновлять все поля

---

### Вариант 30. Чеки (Receipt)

**Таблица:** `receipts` (id, number, store, total_amount, date, cashier, receipt_number)

**Метод 1:** `mergeReceipts(long fromStoreId, long toStoreId)` — конфликт по receipt_number

**Метод 2:** `batchInsert(List<Receipt> receipts)` — вставка 8000 записей

**UPSERT:** по receipt_number, обновлять все поля

---

## 5. Методические указания

### 5.1. Структура проекта

```
src/
├── main/
│   ├── java/
│   │   ├── config/
│   │   │   ├── DatabaseConfig.java
│   │   │   └── DataSourceProvider.java
│   │   ├── model/
│   │   │   └── Book.java
│   │   ├── dao/
│   │   │   ├── BookDao.java
│   │   │   └── BookDaoImpl.java
│   │   ├── service/
│   │   │   └── BookService.java
│   │   ├── util/
│   │   │   └── PerformanceUtil.java
│   │   └── Main.java
│   └── resources/
│       └── application.properties
└── test/
    └── java/
        └── service/
            └── BookServiceTest.java
```

### 5.2. Настройка HikariCP

```java
package config;

import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;

import java.sql.Connection;
import java.sql.SQLException;

public class DataSourceProvider {
    private static HikariDataSource dataSource;
    private static final String URL = "jdbc:postgresql://localhost:5432/postgres";
    private static final String USER = "postgres";
    private static final String PASSWORD = "pass";
    
    public static synchronized HikariDataSource getDataSource() {
        if (dataSource == null) {
            HikariConfig config = new HikariConfig();
            config.setJdbcUrl(URL);
            config.setUsername(USER);
            config.setPassword(PASSWORD);
            config.setDriverClassName("org.postgresql.Driver");
            
            // Оптимальные настройки для производительности
            config.setMaximumPoolSize(10);
            config.setMinimumIdle(5);
            config.setConnectionTimeout(30000);
            config.setIdleTimeout(600000);
            config.setMaxLifetime(1800000);
            
            // Дополнительные настройки
            config.addDataSourceProperty("cachePrepStmts", "true");
            config.addDataSourceProperty("prepStmtCacheSize", "250");
            config.addDataSourceProperty("prepStmtCacheSqlLimit", "2048");
            config.setLeakDetectionThreshold(15000);
            
            dataSource = new HikariDataSource(config);
        }
        return dataSource;
    }
    
    public static Connection getConnection() throws SQLException {
        return getDataSource().getConnection();
    }
    
    public static void close() {
        if (dataSource != null) {
            dataSource.close();
        }
    }
}
```

### 5.3. Сервисный слой с транзакциями

```java
package service;

import config.DataSourceProvider;
import dao.BookDao;
import model.Book;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.List;

public class BookService {
    private static final Logger logger = LoggerFactory.getLogger(BookService.class);
    private final BookDao bookDao;
    
    public BookService(BookDao bookDao) {
        this.bookDao = bookDao;
    }
    
    /**
     * Переназначение книг от одного автора к другому
     */
    public int transferBooks(long fromAuthorId, long toAuthorId) throws SQLException {
        logger.info("Начало переназначения книг: fromAuthor={}, toAuthor={}", fromAuthorId, toAuthorId);
        
        String checkSql = """
            SELECT b1.isbn 
            FROM books b1 
            JOIN books b2 ON b1.isbn = b2.isbn 
            WHERE b1.author_id = ? AND b2.author_id = ?
        """;
        
        String updateSql = "UPDATE books SET author_id = ? WHERE author_id = ?";
        
        try (Connection conn = DataSourceProvider.getConnection()) {
            // Начало транзакции
            conn.setAutoCommit(false);
            conn.setTransactionIsolation(Connection.TRANSACTION_READ_COMMITTED);
            
            // Проверка конфликтов
            try (PreparedStatement checkStmt = conn.prepareStatement(checkSql)) {
                checkStmt.setLong(1, fromAuthorId);
                checkStmt.setLong(2, toAuthorId);
                
                try (ResultSet rs = checkStmt.executeQuery()) {
                    if (rs.next()) {
                        String conflictingIsbn = rs.getString("isbn");
                        logger.error("Конфликт ISBN: {}", conflictingIsbn);
                        throw new SQLException(
                            "Книга с ISBN '" + conflictingIsbn + 
                            "' уже существует у целевого автора"
                        );
                    }
                }
            }
            
            // Выполнение переназначения
            int transferred;
            try (PreparedStatement updateStmt = conn.prepareStatement(updateSql)) {
                updateStmt.setLong(1, toAuthorId);
                updateStmt.setLong(2, fromAuthorId);
                transferred = updateStmt.executeUpdate();
            }
            
            // Фиксация транзакции
            conn.commit();
            logger.info("Переназначено {} книг", transferred);
            return transferred;
            
        } catch (SQLException e) {
            logger.error("Ошибка при переназначении книг: {}", e.getMessage());
            throw e;
        }
    }
    
    /**
     * Пакетная вставка книг с сравнением производительности
     */
    public BatchInsertResult batchInsertWithComparison(List<Book> books) throws SQLException {
        logger.info("Сравнение производительности вставки {} книг", books.size());
        
        // Построчная вставка
        long startRow = System.currentTimeMillis();
        int rowCount = bookDao.insertRowByRow(books);
        long rowTime = System.currentTimeMillis() - startRow;
        
        // Пакетная вставка
        long startBatch = System.currentTimeMillis();
        int batchCount = bookDao.batchInsert(books);
        long batchTime = System.currentTimeMillis() - startBatch;
        
        logger.info("Построчная: {} мс, Пакетная: {} мс, Ускорение: {:.2f}x", 
            rowTime, batchTime, (double) rowTime / batchTime);
        
        return new BatchInsertResult(rowCount, rowTime, batchCount, batchTime);
    }
    
    /**
     * UPSERT — вставка или обновление книги
     */
    public Book saveOrUpdate(Book book) throws SQLException {
        logger.info("UPSERT: isbn={}, title={}", book.getIsbn(), book.getTitle());
        
        String sql = """
            INSERT INTO books (title, author, year, isbn, publisher)
            VALUES (?, ?, ?, ?, ?)
            ON CONFLICT (isbn) DO UPDATE SET
                title = EXCLUDED.title,
                author = EXCLUDED.author,
                year = EXCLUDED.year,
                publisher = EXCLUDED.publisher
            RETURNING id
        """;
        
        try (Connection conn = DataSourceProvider.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            pstmt.setString(1, book.getTitle());
            pstmt.setString(2, book.getAuthor());
            pstmt.setInt(3, book.getYear());
            pstmt.setString(4, book.getIsbn());
            pstmt.setString(5, book.getPublisher());
            
            try (ResultSet rs = pstmt.executeQuery()) {
                if (rs.next()) {
                    long id = rs.getLong(1);
                    book.setId(id);
                    logger.info("UPSERT выполнен: id={}, isbn={}", id, book.getIsbn());
                }
            }
        }
        
        return book;
    }
    
    /**
     * Сравнение производительности с пулом и без пула
     */
    public PoolComparisonResult comparePoolPerformance(int queryCount) throws SQLException {
        logger.info("Сравнение производительности: {} запросов", queryCount);
        
        // Без пула
        long startNoPool = System.currentTimeMillis();
        executeQueriesNoPool(queryCount);
        long timeNoPool = System.currentTimeMillis() - startNoPool;
        
        // С пулом
        long startPool = System.currentTimeMillis();
        executeQueriesWithPool(queryCount);
        long timePool = System.currentTimeMillis() - startPool;
        
        logger.info("Без пула: {} мс, С пулом: {} мс", timeNoPool, timePool);
        
        return new PoolComparisonResult(timeNoPool, timePool);
    }
    
    private void executeQueriesNoPool(int count) throws SQLException {
        String url = "jdbc:postgresql://localhost:5432/postgres";
        String user = "postgres";
        String pass = "pass";
        
        for (int i = 0; i < count; i++) {
            try (Connection conn = DriverManager.getConnection(url, user, pass);
                 PreparedStatement pstmt = conn.prepareStatement("SELECT 1")) {
                pstmt.executeQuery();
            }
        }
    }
    
    private void executeQueriesWithPool(int count) throws SQLException {
        for (int i = 0; i < count; i++) {
            try (Connection conn = DataSourceProvider.getConnection();
                 PreparedStatement pstmt = conn.prepareStatement("SELECT 1")) {
                pstmt.executeQuery();
            }
        }
    }
    
    // Вспомогательные классы для результатов
    public static class BatchInsertResult {
        public final int rowCount;
        public final long rowTime;
        public final int batchCount;
        public final long batchTime;
        
        public BatchInsertResult(int rowCount, long rowTime, int batchCount, long batchTime) {
            this.rowCount = rowCount;
            this.rowTime = rowTime;
            this.batchCount = batchCount;
            this.batchTime = batchTime;
        }
        
        @Override
        public String toString() {
            return String.format(
                "Построчная вставка: %d записей за %d мс\n" +
                "Пакетная вставка: %d записей за %d мс\n" +
                "Ускорение: %.2fx",
                rowCount, rowTime, batchCount, batchTime, (double) rowTime / batchTime
            );
        }
    }
    
    public static class PoolComparisonResult {
        public final long timeNoPool;
        public final long timePool;
        
        public PoolComparisonResult(long timeNoPool, long timePool) {
            this.timeNoPool = timeNoPool;
            this.timePool = timePool;
        }
        
        @Override
        public String toString() {
            return String.format(
                "Без пула: %d мс\nС пулом (HikariCP): %d мс\nУскорение: %.2fx",
                timeNoPool, timePool, (double) timeNoPool / timePool
            );
        }
    }
}
```

### 5.4. Расширенный DAO с пакетной вставкой

```java
package dao;

import config.DataSourceProvider;
import model.Book;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;
import java.util.List;

public class BookDaoImpl implements BookDao {
    private static final Logger logger = LoggerFactory.getLogger(BookDaoImpl.class);
    
    @Override
    public int batchInsert(List<Book> books) throws SQLException {
        String sql = "INSERT INTO books (title, author, year, isbn, publisher) VALUES (?, ?, ?, ?, ?)";
        int count = 0;
        
        try (Connection conn = DataSourceProvider.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            conn.setAutoCommit(false);
            
            for (Book book : books) {
                pstmt.setString(1, book.getTitle());
                pstmt.setString(2, book.getAuthor());
                pstmt.setInt(3, book.getYear());
                pstmt.setString(4, book.getIsbn());
                pstmt.setString(5, book.getPublisher());
                pstmt.addBatch();
                count++;
            }
            
            int[] results = pstmt.executeBatch();
            conn.commit();
            
            logger.info("Пакетная вставка: {} записей", results.length);
        } catch (SQLException e) {
            logger.error("Ошибка при пакетной вставке", e);
            throw e;
        }
        
        return count;
    }
    
    @Override
    public int insertRowByRow(List<Book> books) throws SQLException {
        String sql = "INSERT INTO books (title, author, year, isbn, publisher) VALUES (?, ?, ?, ?, ?)";
        int count = 0;
        
        try (Connection conn = DataSourceProvider.getConnection()) {
            conn.setAutoCommit(true);
            
            for (Book book : books) {
                try (PreparedStatement pstmt = conn.prepareStatement(sql)) {
                    pstmt.setString(1, book.getTitle());
                    pstmt.setString(2, book.getAuthor());
                    pstmt.setInt(3, book.getYear());
                    pstmt.setString(4, book.getIsbn());
                    pstmt.setString(5, book.getPublisher());
                    pstmt.executeUpdate();
                    count++;
                }
            }
        }
        
        logger.info("Построчная вставка: {} записей", count);
        return count;
    }
}
```

### 5.5. Основной класс для демонстрации

```java
import config.DataSourceProvider;
import dao.BookDao;
import dao.BookDaoImpl;
import model.Book;
import service.BookService;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.List;

public class Main {
    public static void main(String[] args) {
        BookDao bookDao = new BookDaoImpl();
        BookService bookService = new BookService(bookDao);
        
        try {
            // 1. Тестирование переназначения
            System.out.println("=== Переназначение книг ===\n");
            try {
                int transferred = bookService.transferBooks(1L, 2L);
                System.out.println("✓ Переназначено книг: " + transferred);
            } catch (SQLException e) {
                System.out.println("✗ Ошибка: " + e.getMessage());
            }
            System.out.println();
            
            // 2. Тестирование пакетной вставки
            System.out.println("=== Пакетная вставка ===\n");
            List<Book> books = generateTestBooks(10000);
            BookService.BatchInsertResult batchResult = 
                bookService.batchInsertWithComparison(books);
            System.out.println(batchResult);
            System.out.println();
            
            // 3. Тестирование UPSERT
            System.out.println("=== UPSERT ===\n");
            Book existingBook = new Book("Обновлённая книга", "Новый автор", 2024, "978-5-17-118456-0", "Новое издательство");
            Book savedBook = bookService.saveOrUpdate(existingBook);
            System.out.println("✓ UPSERT выполнен: id=" + savedBook.getId());
            System.out.println();
            
            // 4. Сравнение производительности пула
            System.out.println("=== Сравнение производительности (100 запросов) ===\n");
            BookService.PoolComparisonResult poolResult = 
                bookService.comparePoolPerformance(100);
            System.out.println(poolResult);
            
        } catch (SQLException e) {
            System.err.println("Ошибка: " + e.getMessage());
            e.printStackTrace();
        } finally {
            DataSourceProvider.close();
        }
    }
    
    private static List<Book> generateTestBooks(int count) {
        List<Book> books = new ArrayList<>(count);
        for (int i = 1; i <= count; i++) {
            books.add(new Book(
                "Book " + i,
                "Author " + (i % 100),
                2000 + (i % 24),
                "ISBN-" + String.format("%010d", i),
                "Publisher " + (i % 10)
            ));
        }
        return books;
    }
}
```

---

## 6. Контрольные вопросы

1. Что такое транзакция и какие свойства ACID она обеспечивает?

2. Как управлять транзакциями в JDBC?

3. Что произойдёт, если не вызвать commit или rollback?

4. Что такое точки сохранения (Savepoint) и когда они применяются?

5. В чём преимущество пакетной вставки перед построчной?

6. Как работает метод addBatch() и executeBatch()?

7. Что такое UPSERT и для чего он нужен?

8. В чём отличие INSERT ... ON CONFLICT от отдельного SELECT + INSERT/UPDATE?

9. Что такое пул соединений и зачем он нужен?

10. Как HikariCP повышает производительность?

11. Какие настройки пула соединений важны для производительности?

12. В чём отличие соединения с пулом и без пула?

13. Как обрабатывать конфликты при переназначении данных?

14. Что такое уровень изоляции транзакций?

15. Какие проблемы могут возникнуть при параллельных транзакциях?

16. Как выполнить откат транзакции при ошибке?

17. Как измерить производительность различных подходов к работе с БД?

---

## 7. Ожидаемый результат выполнения (Вариант 1)

```
=== Переназначение книг ===
2026-01-15 10:30:15 [main] INFO  BookService - Начало переназначения книг: fromAuthor=1, toAuthor=2
2026-01-15 10:30:15 [main] INFO  BookService - Переназначено 15 книг
✓ Переназначено книг: 15

=== Пакетная вставка ===
2026-01-15 10:30:15 [main] INFO  BookService - Сравнение производительности вставки 10000 книг
2026-01-15 10:30:20 [main] INFO  BookDaoImpl - Построчная вставка: 10000 записей
2026-01-15 10:30:20 [main] INFO  BookDaoImpl - Пакетная вставка: 10000 записей
2026-01-15 10:30:20 [main] INFO  BookService - Построчная: 5234 мс, Пакетная: 245 мс, Ускорение: 21.36x

Построчная вставка: 10000 записей за 5234 мс
Пакетная вставка: 10000 записей за 245 мс
Ускорение: 21.36x

=== UPSERT ===
2026-01-15 10:30:20 [main] INFO  BookService - UPSERT: isbn=978-5-17-118456-0, title=Обновлённая книга
2026-01-15 10:30:20 [main] INFO  BookService - UPSERT выполнен: id=1, isbn=978-5-17-118456-0
✓ UPSERT выполнен: id=1

=== Сравнение производительности (100 запросов) ===
2026-01-15 10:30:20 [main] INFO  BookService - Сравнение производительности: 100 запросов
2026-01-15 10:30:22 [main] INFO  BookService - Без пула: 1856 мс, С пулом: 156 мс

Без пула: 1856 мс
С пулом (HikariCP): 156 мс
Ускорение: 11.90x
```

---

## 8. Рекомендуемые источники

1. **Шилдт Г.** *Java. Базовый курс.* — М.: Вильямс. — Глава 16 (Работа с базами данных).

2. **Хорстманн К., Корнелл Г.** *Java. Библиотека профессионала. Том 2.* — М.: Вильямс. — Глава 4 (Базы данных).

3. **PostgreSQL Documentation: INSERT ON CONFLICT.** — URL: https://www.postgresql.org/docs/current/sql-insert.html

4. **HikariCP GitHub.** — URL: https://github.com/brettwooldridge/HikariCP

5. **Baeldung: HikariCP Tutorial.** — URL: https://www.baeldung.com/hikaricp

6. **Oracle Java Tutorials: JDBC Transactions.** — URL: https://docs.oracle.com/javase/tutorial/jdbc/basics/transactions.html
