# Лабораторная работа №11. JDBC: подключение к БД и CRUD

## 1. Паспорт работы

| Параметр | Значение |
|----------|----------|
| Номер занятия | 11 из 17 |
| Блок | 5. Базы данных и JDBC |
| Продолжительность | 2 академических часа |
| Форма выполнения | Индивидуальная |
| ИИ-инструмент | YandexGPT / GigaChat / JetBrains AI Assistant |

### 1.1. Цель работы

Освоить работу с реляционными базами данных в Java через JDBC, научиться выполнять CRUD-операции, использовать PreparedStatement для защиты от SQL-инъекций и применять паттерн DAO для отделения логики работы с БД от бизнес-логики.

### 1.2. Задачи работы

1. Изучить архитектуру JDBC (DriverManager, Connection, Statement, ResultSet).
2. Освоить подключение к PostgreSQL.
3. Научиться создавать таблицы в БД.
4. Изучить CRUD-операции (Create, Read, Update, Delete).
5. Освоить пакетные операции (batch insert).
6. Изучить PreparedStatement и защиту от SQL-инъекций.
7. Реализовать паттерн DAO (Data Access Object).
8. Освоить закрытие ресурсов с try-with-resources.

### 1.3. Оснащение

- JDK 17 или выше;
- IntelliJ IDEA Community Edition;
- Git;
- PostgreSQL (локально или в Docker);
- Maven или Gradle;
- доступ к YandexGPT или GigaChat.

---

## 2. Теоретический конспект

### 2.1. Что такое JDBC

**JDBC (Java Database Connectivity)** — стандартный API для взаимодействия Java-приложений с реляционными базами данных.

**Архитектура JDBC:**

```
Java Application
       ↓
   JDBC API
       ↓
   Driver Manager
       ↓
   JDBC Driver
       ↓
   Database
```

**Основные компоненты:**

| Компонент | Описание |
|-----------|----------|
| `DriverManager` | Управляет драйверами БД |
| `Connection` | Представляет соединение с БД |
| `Statement` | Выполняет SQL-запросы |
| `PreparedStatement` | Выполняет параметризованные запросы |
| `CallableStatement` | Вызывает хранимые процедуры |
| `ResultSet` | Представляет результат запроса |
| `SQLException` | Исключение при работе с БД |

### 2.2. Подключение к базе данных

**Добавление зависимости (Maven):**

```xml
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>42.7.1</version>
</dependency>
```

**Подключение через DriverManager:**

```java
// Загрузка драйвера (не обязательно в Java 6+)
Class.forName("org.postgresql.Driver");

// URL для подключения
String url = "jdbc:postgresql://localhost:5432/mydb";
String user = "postgres";
String password = "password";

// Создание соединения
try (Connection conn = DriverManager.getConnection(url, user, password)) {
    System.out.println("Подключено к БД");
} catch (SQLException e) {
    e.printStackTrace();
}
```

### 2.3. Создание таблицы

```java
String createTableSQL = """
    CREATE TABLE IF NOT EXISTS books (
        id SERIAL PRIMARY KEY,
        title VARCHAR(255) NOT NULL,
        author VARCHAR(255) NOT NULL,
        year INTEGER NOT NULL,
        isbn VARCHAR(20) UNIQUE NOT NULL,
        publisher VARCHAR(255)
    )
""";

try (Connection conn = getConnection();
     Statement stmt = conn.createStatement()) {
    stmt.execute(createTableSQL);
    System.out.println("Таблица создана");
} catch (SQLException e) {
    e.printStackTrace();
}
```

### 2.4. CRUD-операции

**CREATE (INSERT):**

```java
// Обычный Statement (не рекомендуется)
String sql = "INSERT INTO books (title, author, year, isbn, publisher) VALUES ('" + 
             title + "', '" + author + "', " + year + ", '" + isbn + "', '" + publisher + "')";
stmt.executeUpdate(sql);

// PreparedStatement (рекомендуется)
String sql = "INSERT INTO books (title, author, year, isbn, publisher) VALUES (?, ?, ?, ?, ?)";
try (PreparedStatement pstmt = conn.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS)) {
    pstmt.setString(1, book.getTitle());
    pstmt.setString(2, book.getAuthor());
    pstmt.setInt(3, book.getYear());
    pstmt.setString(4, book.getIsbn());
    pstmt.setString(5, book.getPublisher());
    
    int affectedRows = pstmt.executeUpdate();
    
    // Получение сгенерированного ID
    if (affectedRows > 0) {
        try (ResultSet rs = pstmt.getGeneratedKeys()) {
            if (rs.next()) {
                long id = rs.getLong(1);
                book.setId(id);
            }
        }
    }
}
```

**READ (SELECT):**

```java
String sql = "SELECT * FROM books WHERE id = ?";
try (PreparedStatement pstmt = conn.prepareStatement(sql)) {
    pstmt.setLong(1, id);
    try (ResultSet rs = pstmt.executeQuery()) {
        if (rs.next()) {
            Book book = new Book();
            book.setId(rs.getLong("id"));
            book.setTitle(rs.getString("title"));
            book.setAuthor(rs.getString("author"));
            book.setYear(rs.getInt("year"));
            book.setIsbn(rs.getString("isbn"));
            book.setPublisher(rs.getString("publisher"));
            return book;
        }
    }
}
```

**UPDATE:**

```java
String sql = "UPDATE books SET title = ?, year = ? WHERE id = ?";
try (PreparedStatement pstmt = conn.prepareStatement(sql)) {
    pstmt.setString(1, book.getTitle());
    pstmt.setInt(2, book.getYear());
    pstmt.setLong(3, book.getId());
    return pstmt.executeUpdate() > 0;
}
```

**DELETE:**

```java
String sql = "DELETE FROM books WHERE id = ?";
try (PreparedStatement pstmt = conn.prepareStatement(sql)) {
    pstmt.setLong(1, id);
    return pstmt.executeUpdate() > 0;
}
```

### 2.5. Пакетная вставка (Batch Insert)

```java
String sql = "INSERT INTO books (title, author, year, isbn, publisher) VALUES (?, ?, ?, ?, ?)";
try (PreparedStatement pstmt = conn.prepareStatement(sql)) {
    for (Book book : books) {
        pstmt.setString(1, book.getTitle());
        pstmt.setString(2, book.getAuthor());
        pstmt.setInt(3, book.getYear());
        pstmt.setString(4, book.getIsbn());
        pstmt.setString(5, book.getPublisher());
        pstmt.addBatch();
    }
    int[] results = pstmt.executeBatch();
    return results.length;
}
```

### 2.6. Паттерн DAO

**DAO (Data Access Object)** — паттерн для отделения логики доступа к данным от бизнес-логики.

```
┌─────────────────────────────────────────────────────────┐
│                    Business Layer                       │
│  ┌─────────────────────────────────────────────────┐    │
│  │               BookService                       │    │
│  └────────────────────┬────────────────────────────┘    │
│                       │                                 │
│  ┌────────────────────▼────────────────────────────┐    │
│  │            BookDao (interface)                  │    │
│  └────────────────────┬────────────────────────────┘    │
│                       │                                 │
│  ┌────────────────────▼────────────────────────────┐    │
│  │         BookDaoImpl (implementation)            │    │
│  └─────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────┘
                         │
                         ▼
                 PostgreSQL Database
```

**Интерфейс DAO:**

```java
public interface BookDao {
    Book save(Book book) throws SQLException;
    Optional<Book> findById(long id) throws SQLException;
    List<Book> findAll() throws SQLException;
    List<Book> findByAuthor(String author) throws SQLException;
    boolean update(Book book) throws SQLException;
    boolean deleteById(long id) throws SQLException;
    int batchInsert(List<Book> books) throws SQLException;
}
```

### 2.7. Закрытие ресурсов

Всегда закрывайте ресурсы в обратном порядке создания:

```java
// Плохо (может привести к утечкам)
Connection conn = null;
Statement stmt = null;
ResultSet rs = null;
try {
    conn = DriverManager.getConnection(url, user, password);
    stmt = conn.createStatement();
    rs = stmt.executeQuery("SELECT * FROM books");
    // обработка
} finally {
    if (rs != null) rs.close();
    if (stmt != null) stmt.close();
    if (conn != null) conn.close();
}

// Хорошо (try-with-resources)
try (Connection conn = DriverManager.getConnection(url, user, password);
     Statement stmt = conn.createStatement();
     ResultSet rs = stmt.executeQuery("SELECT * FROM books")) {
    // обработка
} // ресурсы автоматически закрыты
```

---

## 3. Задание на паре

### Задача. CRUD-операции с книгами в PostgreSQL

1. **Установить PostgreSQL:**
   - Локально или через Docker:
   ```bash
   docker run --name pg-java -e POSTGRES_PASSWORD=pass -p 5432:5432 -d postgres:16
   ```

2. **Создать базу данных и таблицу:**
   ```sql
   CREATE TABLE books (
       id SERIAL PRIMARY KEY,
       title VARCHAR(255) NOT NULL,
       author VARCHAR(255) NOT NULL,
       year INTEGER NOT NULL,
       isbn VARCHAR(20) UNIQUE NOT NULL,
       publisher VARCHAR(255)
   );
   ```

3. **Реализовать BookDao с операциями:**
   - `save(Book book)` — вставка одной книги.
   - `findById(long id)` — поиск по ID.
   - `findAll()` — выборка всех книг.
   - `findByAuthor(String author)` — поиск по автору.
   - `update(Book book)` — обновление года.
   - `deleteByIsbn(String isbn)` — удаление по ISBN.

4. **ВСЕ ресурсы (Connection, Statement, ResultSet) закрывать в try-with-resources.**

5. **Добавить пакетную вставку:**
   - `batchInsert(List<Book> books)` — вставка 1000 книг.

6. **Протестировать все операции в main():**
   - Создать 10 книг.
   - Сохранить их.
   - Найти по ID.
   - Найти по автору.
   - Обновить год.
   - Удалить по ISBN.
   - Вывести все книги.

**Пример выполнения:**
```
=== Создание таблицы ===
Таблица books создана

=== Вставка книг ===
Книга сохранена: id=1, title='Война и мир'
Книга сохранена: id=2, title='Преступление и наказание'
...
10 книг сохранено

=== Поиск по ID ===
Найдена книга: Book{id=1, title='Война и мир', ...}

=== Поиск по автору ===
Найдено 2 книги автора 'Толстой Л.Н.'

=== Обновление года ===
Книга с id=1 обновлена: 1869 → 1870

=== Удаление по ISBN ===
Книга с ISBN '978-5-17-118457-7' удалена

=== Все книги ===
Всего книг: 9
1. Война и мир (Толстой Л.Н., 1870)
2. Java: полное руководство (Шилдт Г., 2019)
...
```

### Применение ИИ-инструмента

**Промпт для YandexGPT:**

```
Сгенерируй CRUD-методы для BookDao в Java с использованием JDBC и PostgreSQL.

Класс Book:
- id (long)
- title (String)
- author (String)
- year (int)
- isbn (String)
- publisher (String)

Требования:
1. Использовать PreparedStatement.
2. Закрывать ресурсы в try-with-resources.
3. Обрабатывать SQLException.
4. Возвращать Optional для findById.
5. Поддерживать batchInsert для списка книг.
6. Использовать RETURN_GENERATED_KEYS для получения id.
```

**Анализ результата:**
- Проверить корректность SQL-запросов.
- Проверить закрытие ресурсов.
- Проверить обработку SQLException.
- Проверить получение сгенерированного ключа.

---

## 4. Индивидуальные задания (30 вариантов)

Каждый вариант содержит:
- Название таблицы.
- Структуру таблицы.
- DAO-интерфейс с методами.

---

### Вариант 1. Книги (Book)

**Таблица:** `books`

**Поля:** `id, title, author, year, isbn, publisher`

**DAO методы:** save, findById, findAll, findByAuthor, update, deleteByIsbn, batchInsert

---

### Вариант 2. Сотрудники (Employee)

**Таблица:** `employees`

**Поля:** `id, first_name, last_name, position, salary, department, hire_date`

**DAO методы:** save, findById, findAll, findByDepartment, update, deleteById, batchInsert

---

### Вариант 3. Товары (Product)

**Таблица:** `products`

**Поля:** `id, name, category, price, quantity, supplier`

**DAO методы:** save, findById, findAll, findByCategory, updatePrice, deleteById, batchInsert

---

### Вариант 4. Заказы (Order)

**Таблица:** `orders`

**Поля:** `id, customer_name, total_amount, status, created_at, items_count`

**DAO методы:** save, findById, findAll, findByStatus, updateStatus, deleteById, batchInsert

---

### Вариант 5. Пользователи (User)

**Таблица:** `users`

**Поля:** `id, username, email, password_hash, role, created_at, is_active`

**DAO методы:** save, findById, findAll, findByEmail, updateRole, deleteById, batchInsert

---

### Вариант 6. Автомобили (Car)

**Таблица:** `cars`

**Поля:** `id, brand, model, year, price, vin, color`

**DAO методы:** save, findById, findAll, findByBrand, updatePrice, deleteByVin, batchInsert

---

### Вариант 7. Студенты (Student)

**Таблица:** `students`

**Поля:** `id, first_name, last_name, group_name, gpa, birth_date, scholarship`

**DAO методы:** save, findById, findAll, findByGroup, updateGpa, deleteById, batchInsert

---

### Вариант 8. Счета (Account)

**Таблица:** `accounts`

**Поля:** `id, account_number, owner_name, balance, currency, opened_date`

**DAO методы:** save, findById, findAll, findByOwner, updateBalance, deleteById, batchInsert

---

### Вариант 9. Фильмы (Movie)

**Таблица:** `movies`

**Поля:** `id, title, director, year, rating, genre, duration`

**DAO методы:** save, findById, findAll, findByGenre, updateRating, deleteById, batchInsert

---

### Вариант 10. Рестораны (Restaurant)

**Таблица:** `restaurants`

**Поля:** `id, name, address, phone, rating, cuisine, opening_time`

**DAO методы:** save, findById, findAll, findByCuisine, updateRating, deleteById, batchInsert

---

### Вариант 11. Транзакции (Transaction)

**Таблица:** `transactions`

**Поля:** `id, amount, type, date, description, category`

**DAO методы:** save, findById, findAll, findByCategory, updateDescription, deleteById, batchInsert

---

### Вариант 12. Клиенты (Customer)

**Таблица:** `customers`

**Поля:** `id, first_name, last_name, email, phone, birth_date, city`

**DAO методы:** save, findById, findAll, findByCity, updatePhone, deleteById, batchInsert

---

### Вариант 13. Договоры (Contract)

**Таблица:** `contracts`

**Поля:** `id, number, client_name, sign_date, amount, status, expiry_date`

**DAO методы:** save, findById, findAll, findByStatus, updateAmount, deleteById, batchInsert

---

### Вариант 14. Отели (Hotel)

**Таблица:** `hotels`

**Поля:** `id, name, city, stars, rooms_count, rating, phone`

**DAO методы:** save, findById, findAll, findByCity, updateRating, deleteById, batchInsert

---

### Вариант 15. Спортсмены (Athlete)

**Таблица:** `athletes`

**Поля:** `id, first_name, last_name, sport, age, medals, country`

**DAO методы:** save, findById, findAll, findBySport, updateMedals, deleteById, batchInsert

---

### Вариант 16. Статьи (Article)

**Таблица:** `articles`

**Поля:** `id, title, content, author, published_date, views, tags`

**DAO методы:** save, findById, findAll, findByAuthor, updateViews, deleteById, batchInsert

---

### Вариант 17. Билеты (Ticket)

**Таблица:** `tickets`

**Поля:** `id, event_name, venue, date, price, quantity, status`

**DAO методы:** save, findById, findAll, findByEvent, updatePrice, deleteById, batchInsert

---

### Вариант 18. Курсы (Course)

**Таблица:** `courses`

**Поля:** `id, title, instructor, duration, price, start_date, end_date`

**DAO методы:** save, findById, findAll, findByInstructor, updatePrice, deleteById, batchInsert

---

### Вариант 19. Инвентарь (Item)

**Таблица:** `inventory`

**Поля:** `id, name, category, quantity, location, status, last_checked`

**DAO методы:** save, findById, findAll, findByCategory, updateQuantity, deleteById, batchInsert

---

### Вариант 20. Заявки (Request)

**Таблица:** `requests`

**Поля:** `id, client_name, description, priority, status, created_at, resolved_at`

**DAO методы:** save, findById, findAll, findByStatus, updatePriority, deleteById, batchInsert

---

### Вариант 21. Платежи (Payment)

**Таблица:** `payments`

**Поля:** `id, number, payer, recipient, amount, date, status`

**DAO методы:** save, findById, findAll, findByPayer, updateStatus, deleteById, batchInsert

---

### Вариант 22. Публикации (Publication)

**Таблица:** `publications`

**Поля:** `id, title, author, journal, year, doi, submitted_date`

**DAO методы:** save, findById, findAll, findByJournal, updateYear, deleteById, batchInsert

---

### Вариант 23. Ноутбуки (Laptop)

**Таблица:** `laptops`

**Поля:** `id, brand, model, processor, ram, price, release_date`

**DAO методы:** save, findById, findAll, findByBrand, updatePrice, deleteById, batchInsert

---

### Вариант 24. Врачи (Doctor)

**Таблица:** `doctors`

**Поля:** `id, first_name, last_name, specialization, experience, rating, phone`

**DAO методы:** save, findById, findAll, findBySpecialization, updateRating, deleteById, batchInsert

---

### Вариант 25. Сертификаты (Certificate)

**Таблица:** `certificates`

**Поля:** `id, number, holder_name, issued_date, expiry_date, type, issuer`

**DAO методы:** save, findById, findAll, findByHolder, updateExpiry, deleteById, batchInsert

---

### Вариант 26. Магазины (Shop)

**Таблица:** `shops`

**Поля:** `id, name, address, phone, opening_hours, type, rating`

**DAO методы:** save, findById, findAll, findByType, updateRating, deleteById, batchInsert

---

### Вариант 27. Здания (Building)

**Таблица:** `buildings`

**Поля:** `id, address, floors, area, type, year_built, owner`

**DAO методы:** save, findById, findAll, findByType, updateOwner, deleteById, batchInsert

---

### Вариант 28. Лекции (Lecture)

**Таблица:** `lectures`

**Поля:** `id, topic, speaker, date, duration, attendees, room`

**DAO методы:** save, findById, findAll, findBySpeaker, updateAttendees, deleteById, batchInsert

---

### Вариант 29. Отзывы (Review)

**Таблица:** `reviews`

**Поля:** `id, user_id, product_id, rating, comment, created_at, helpful_count`

**DAO методы:** save, findById, findAll, findByProduct, updateHelpful, deleteById, batchInsert

---

### Вариант 30. Чеки (Receipt)

**Таблица:** `receipts`

**Поля:** `id, number, store, total_amount, date, cashier, items_count`

**DAO методы:** save, findById, findAll, findByStore, updateTotal, deleteById, batchInsert

---

## 5. Методические указания

### 5.1. Структура проекта

```
src/
├── main/
│   ├── java/
│   │   ├── config/
│   │   │   └── DatabaseConfig.java
│   │   ├── model/
│   │   │   └── Book.java
│   │   ├── dao/
│   │   │   ├── BookDao.java
│   │   │   └── BookDaoImpl.java
│   │   ├── exception/
│   │   │   └── DatabaseException.java
│   │   └── Main.java
│   └── resources/
│       └── application.properties
└── test/
    └── java/
        └── dao/
            └── BookDaoTest.java
```

### 5.2. Конфигурация базы данных

```java
package config;

import java.io.IOException;
import java.io.InputStream;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.util.Properties;

public class DatabaseConfig {
    private static final Properties properties = new Properties();
    
    static {
        try (InputStream input = DatabaseConfig.class
                .getClassLoader()
                .getResourceAsStream("application.properties")) {
            if (input != null) {
                properties.load(input);
            } else {
                // Значения по умолчанию
                properties.setProperty("db.url", "jdbc:postgresql://localhost:5432/postgres");
                properties.setProperty("db.user", "postgres");
                properties.setProperty("db.password", "pass");
                properties.setProperty("db.driver", "org.postgresql.Driver");
            }
        } catch (IOException e) {
            throw new RuntimeException("Не удалось загрузить конфигурацию БД", e);
        }
    }
    
    public static String getUrl() {
        return properties.getProperty("db.url");
    }
    
    public static String getUser() {
        return properties.getProperty("db.user");
    }
    
    public static String getPassword() {
        return properties.getProperty("db.password");
    }
    
    public static String getDriver() {
        return properties.getProperty("db.driver");
    }
    
    public static Connection getConnection() throws SQLException {
        return DriverManager.getConnection(getUrl(), getUser(), getPassword());
    }
}
```

### 5.3. Файл application.properties

```properties
# PostgreSQL Configuration
db.url=jdbc:postgresql://localhost:5432/postgres
db.user=postgres
db.password=pass
db.driver=org.postgresql.Driver

# Connection Pool Settings (для будущих лабораторных)
db.pool.size=10
db.pool.maxWait=5000
```

### 5.4. Интерфейс BookDao

```java
package dao;

import model.Book;

import java.sql.SQLException;
import java.util.List;
import java.util.Optional;

public interface BookDao {
    /**
     * Сохраняет книгу в БД
     * @param book объект книги (id будет заполнен автоматически)
     * @return сохранённый объект с заполненным id
     */
    Book save(Book book) throws SQLException;
    
    /**
     * Находит книгу по ID
     * @param id идентификатор книги
     * @return Optional с книгой или пустой Optional
     */
    Optional<Book> findById(long id) throws SQLException;
    
    /**
     * Возвращает все книги
     * @return список всех книг
     */
    List<Book> findAll() throws SQLException;
    
    /**
     * Находит книги по автору
     * @param author имя автора (частичное совпадение)
     * @return список книг автора
     */
    List<Book> findByAuthor(String author) throws SQLException;
    
    /**
     * Обновляет информацию о книге
     * @param book обновлённый объект книги
     * @return true если обновление успешно
     */
    boolean update(Book book) throws SQLException;
    
    /**
     * Удаляет книгу по ISBN
     * @param isbn ISBN книги
     * @return true если удаление успешно
     */
    boolean deleteByIsbn(String isbn) throws SQLException;
    
    /**
     * Пакетная вставка книг
     * @param books список книг
     * @return количество вставленных книг
     */
    int batchInsert(List<Book> books) throws SQLException;
}
```

### 5.5. Реализация BookDaoImpl

```java
package dao;

import config.DatabaseConfig;
import model.Book;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.sql.*;
import java.util.ArrayList;
import java.util.List;
import java.util.Optional;

public class BookDaoImpl implements BookDao {
    private static final Logger logger = LoggerFactory.getLogger(BookDaoImpl.class);
    
    @Override
    public Book save(Book book) throws SQLException {
        String sql = """
            INSERT INTO books (title, author, year, isbn, publisher)
            VALUES (?, ?, ?, ?, ?)
        """;
        
        try (Connection conn = DatabaseConfig.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS)) {
            
            pstmt.setString(1, book.getTitle());
            pstmt.setString(2, book.getAuthor());
            pstmt.setInt(3, book.getYear());
            pstmt.setString(4, book.getIsbn());
            pstmt.setString(5, book.getPublisher());
            
            int affectedRows = pstmt.executeUpdate();
            
            if (affectedRows == 0) {
                throw new SQLException("Создание книги не удалось, ни одна строка не затронута");
            }
            
            try (ResultSet generatedKeys = pstmt.getGeneratedKeys()) {
                if (generatedKeys.next()) {
                    long id = generatedKeys.getLong(1);
                    book.setId(id);
                    logger.info("Книга сохранена: id={}, title='{}'", id, book.getTitle());
                } else {
                    throw new SQLException("Создание книги не удалось, ID не получен");
                }
            }
        }
        
        return book;
    }
    
    @Override
    public Optional<Book> findById(long id) throws SQLException {
        String sql = "SELECT * FROM books WHERE id = ?";
        
        try (Connection conn = DatabaseConfig.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            pstmt.setLong(1, id);
            
            try (ResultSet rs = pstmt.executeQuery()) {
                if (rs.next()) {
                    return Optional.of(mapRowToBook(rs));
                }
            }
        }
        
        return Optional.empty();
    }
    
    @Override
    public List<Book> findAll() throws SQLException {
        String sql = "SELECT * FROM books ORDER BY id";
        List<Book> books = new ArrayList<>();
        
        try (Connection conn = DatabaseConfig.getConnection();
             Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery(sql)) {
            
            while (rs.next()) {
                books.add(mapRowToBook(rs));
            }
        }
        
        logger.info("Найдено {} книг", books.size());
        return books;
    }
    
    @Override
    public List<Book> findByAuthor(String author) throws SQLException {
        String sql = "SELECT * FROM books WHERE author ILIKE ? ORDER BY year";
        List<Book> books = new ArrayList<>();
        
        try (Connection conn = DatabaseConfig.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            pstmt.setString(1, "%" + author + "%");
            
            try (ResultSet rs = pstmt.executeQuery()) {
                while (rs.next()) {
                    books.add(mapRowToBook(rs));
                }
            }
        }
        
        logger.info("Найдено {} книг автора '{}'", books.size(), author);
        return books;
    }
    
    @Override
    public boolean update(Book book) throws SQLException {
        String sql = "UPDATE books SET title = ?, author = ?, year = ?, publisher = ? WHERE id = ?";
        
        try (Connection conn = DatabaseConfig.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            pstmt.setString(1, book.getTitle());
            pstmt.setString(2, book.getAuthor());
            pstmt.setInt(3, book.getYear());
            pstmt.setString(4, book.getPublisher());
            pstmt.setLong(5, book.getId());
            
            int affectedRows = pstmt.executeUpdate();
            boolean updated = affectedRows > 0;
            
            if (updated) {
                logger.info("Книга обновлена: id={}", book.getId());
            } else {
                logger.warn("Книга не найдена для обновления: id={}", book.getId());
            }
            
            return updated;
        }
    }
    
    @Override
    public boolean deleteByIsbn(String isbn) throws SQLException {
        String sql = "DELETE FROM books WHERE isbn = ?";
        
        try (Connection conn = DatabaseConfig.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            pstmt.setString(1, isbn);
            
            int affectedRows = pstmt.executeUpdate();
            boolean deleted = affectedRows > 0;
            
            if (deleted) {
                logger.info("Книга удалена по ISBN: {}", isbn);
            } else {
                logger.warn("Книга не найдена для удаления по ISBN: {}", isbn);
            }
            
            return deleted;
        }
    }
    
    @Override
    public int batchInsert(List<Book> books) throws SQLException {
        String sql = "INSERT INTO books (title, author, year, isbn, publisher) VALUES (?, ?, ?, ?, ?)";
        int count = 0;
        
        try (Connection conn = DatabaseConfig.getConnection();
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
            
            logger.info("Пакетная вставка: {} книг", results.length);
        } catch (SQLException e) {
            logger.error("Ошибка при пакетной вставке", e);
            throw e;
        }
        
        return count;
    }
    
    private Book mapRowToBook(ResultSet rs) throws SQLException {
        Book book = new Book();
        book.setId(rs.getLong("id"));
        book.setTitle(rs.getString("title"));
        book.setAuthor(rs.getString("author"));
        book.setYear(rs.getInt("year"));
        book.setIsbn(rs.getString("isbn"));
        book.setPublisher(rs.getString("publisher"));
        return book;
    }
}
```

### 5.6. Создание таблицы (инициализация)

```java
package init;

import config.DatabaseConfig;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.sql.Connection;
import java.sql.SQLException;
import java.sql.Statement;

public class DatabaseInitializer {
    private static final Logger logger = LoggerFactory.getLogger(DatabaseInitializer.class);
    
    public static void createTable() throws SQLException {
        String createTableSQL = """
            CREATE TABLE IF NOT EXISTS books (
                id SERIAL PRIMARY KEY,
                title VARCHAR(255) NOT NULL,
                author VARCHAR(255) NOT NULL,
                year INTEGER NOT NULL,
                isbn VARCHAR(20) UNIQUE NOT NULL,
                publisher VARCHAR(255)
            )
        """;
        
        try (Connection conn = DatabaseConfig.getConnection();
             Statement stmt = conn.createStatement()) {
            
            stmt.execute(createTableSQL);
            logger.info("Таблица books создана (или уже существует)");
        }
    }
    
    public static void dropTable() throws SQLException {
        String dropTableSQL = "DROP TABLE IF EXISTS books CASCADE";
        
        try (Connection conn = DatabaseConfig.getConnection();
             Statement stmt = conn.createStatement()) {
            
            stmt.execute(dropTableSQL);
            logger.info("Таблица books удалена");
        }
    }
}
```

### 5.7. Основной класс

```java
import config.DatabaseConfig;
import dao.BookDao;
import dao.BookDaoImpl;
import init.DatabaseInitializer;
import model.Book;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.sql.SQLException;
import java.util.Arrays;
import java.util.List;
import java.util.Optional;

public class Main {
    private static final Logger logger = LoggerFactory.getLogger(Main.class);
    
    public static void main(String[] args) {
        BookDao bookDao = new BookDaoImpl();
        
        try {
            // 1. Создание таблицы
            System.out.println("=== Создание таблицы ===");
            DatabaseInitializer.createTable();
            System.out.println("✓ Таблица создана\n");
            
            // 2. Вставка книг
            System.out.println("=== Вставка книг ===");
            List<Book> books = createBooks();
            for (Book book : books) {
                bookDao.save(book);
                System.out.println("✓ Книга сохранена: id=" + book.getId() + 
                    ", title='" + book.getTitle() + "'");
            }
            System.out.println();
            
            // 3. Поиск по ID
            System.out.println("=== Поиск по ID ===");
            Optional<Book> found = bookDao.findById(1L);
            found.ifPresentOrElse(
                book -> System.out.println("✓ Найдена: " + book),
                () -> System.out.println("✗ Книга не найдена")
            );
            System.out.println();
            
            // 4. Поиск по автору
            System.out.println("=== Поиск по автору ===");
            List<Book> authorBooks = bookDao.findByAuthor("Толстой");
            System.out.println("✓ Найдено книг: " + authorBooks.size());
            authorBooks.forEach(System.out::println);
            System.out.println();
            
            // 5. Обновление года
            System.out.println("=== Обновление года ===");
            if (found.isPresent()) {
                Book book = found.get();
                book.setYear(2020);
                boolean updated = bookDao.update(book);
                System.out.println("✓ Обновлено: " + updated);
                System.out.println("  " + bookDao.findById(book.getId()).orElse(null));
            }
            System.out.println();
            
            // 6. Удаление по ISBN
            System.out.println("=== Удаление по ISBN ===");
            String isbnToDelete = "978-5-17-118457-7";
            boolean deleted = bookDao.deleteByIsbn(isbnToDelete);
            System.out.println("✓ Удалено: " + deleted);
            System.out.println();
            
            // 7. Все книги
            System.out.println("=== Все книги ===");
            List<Book> allBooks = bookDao.findAll();
            System.out.println("✓ Всего книг: " + allBooks.size());
            allBooks.forEach(System.out::println);
            
        } catch (SQLException e) {
            logger.error("Ошибка при работе с БД", e);
            System.err.println("Ошибка: " + e.getMessage());
        }
    }
    
    private static List<Book> createBooks() {
        return Arrays.asList(
            new Book("Война и мир", "Толстой Л.Н.", 1869, "978-5-17-118456-0", "АСТ"),
            new Book("Преступление и наказание", "Достоевский Ф.М.", 1866, "978-5-17-118457-7", "Эксмо"),
            new Book("Java: полное руководство", "Шилдт Г.", 2019, "978-5-8459-1959-3", "Вильямс"),
            new Book("Чистый код", "Мартин Р.", 2012, "978-5-8459-1961-6", "Вильямс"),
            new Book("Рефакторинг", "Фаулер М.", 2012, "978-5-8459-1962-3", "Вильямс")
        );
    }
}
```

---

## 6. Контрольные вопросы

1. Что такое JDBC и для чего он используется?

2. Опишите архитектуру JDBC.

3. Какие основные компоненты JDBC вы знаете?

4. Как подключиться к базе данных в Java?

5. В чём отличие Statement от PreparedStatement?

6. Почему PreparedStatement безопаснее Statement?

7. Что такое SQL-инъекция и как её предотвратить?

8. Как получить сгенерированный ключ при вставке?

9. Что такое ResultSet и как его использовать?

10. Как обрабатывать исключения в JDBC?

11. Что такое паттерн DAO и для чего он нужен?

12. Как выполнить пакетную вставку (batch insert)?

13. Почему важно закрывать ресурсы в try-with-resources?

14. Какие типы ResultSet существуют?

15. Как выполнить транзакцию в JDBC?

16. Что такое Connection Pool и зачем он нужен?

17. Как обрабатывать большие объёмы данных из ResultSet?

---

## 7. Рекомендуемые источники

1. **Шилдт Г.** *Java. Базовый курс.* — М.: Вильямс. — Глава 16 (Работа с базами данных).

2. **Хорстманн К., Корнелл Г.** *Java. Библиотека профессионала. Том 2.* — М.: Вильямс. — Глава 4 (Базы данных).

3. **Oracle Java Tutorials: JDBC.** — URL: https://docs.oracle.com/javase/tutorial/jdbc/

4. **PostgreSQL JDBC Driver Documentation.** — URL: https://jdbc.postgresql.org/documentation/

5. **Baeldung: JDBC Tutorial.** — URL: https://www.baeldung.com/java-jdbc

6. **Блох Дж.** *Java. Эффективное программирование.* — М.: Питер. — Правило 9 (try-with-resources).
