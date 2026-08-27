# Лабораторная работа №14. ETL-пайплайн: Load + мониторинг (ETL завершён)

## 1. Паспорт работы

| Параметр | Значение |
|----------|----------|
| Номер занятия | 14 из 17 |
| Блок | 6. ETL-пайплайн |
| Продолжительность | 2 академических часа |
| Форма выполнения | Индивидуальная |
| ИИ-инструмент | YandexGPT / GigaChat / JetBrains AI Assistant |

### 1.1. Цель работы

Завершить реализацию ETL-пайплайна, реализовать этап загрузки (Load) данных в базу данных с обеспечением идемпотентности, добавить мониторинг, логирование и обработку ошибок, собрать все этапы в единый пайплайн с метриками.

### 1.2. Задачи работы

1. Реализовать этап Load — запись очищенных данных в PostgreSQL через DAO.
2. Обеспечить идемпотентность с использованием UPSERT.
3. Собрать весь пайплайн в единый класс `BookEtlPipeline`.
4. Создать иммутабельный объект `EtlResult` с метриками.
5. Добавить логирование каждого этапа.
6. Реализовать dead-letter-queue (сохранение ошибочных записей).
7. Протестировать пайплайн на данных с заведомо некорректными записями.
8. Проверить идемпотентность повторным запуском.

### 1.3. Оснащение

- JDK 17 или выше;
- IntelliJ IDEA Community Edition;
- Git;
- PostgreSQL (локально или в Docker);
- Maven или Gradle;
- HikariCP;
- SLF4J + Logback;
- доступ к YandexGPT или GigaChat.

---

## 2. Теоретический конспект

### 2.1. Этап Load в ETL

**Load** — этап загрузки преобразованных данных в целевую систему.

```
┌─────────────────────────────────────────────────────────────────┐
│                         Этап Load                              │
│                                                                 │
│  ┌─────────────────┐    ┌─────────────────┐                    │
│  │  Transform      │───▶│  Load           │                    │
│  │  (очищенные     │    │  (запись в БД)  │                    │
│  │   данные)       │    │                 │                    │
│  └─────────────────┘    └─────────────────┘                    │
│         │                        │                             │
│         ▼                        ▼                             │
│  ┌─────────────────────────────────────────────┐               │
│  │           Целевая БД (PostgreSQL)           │               │
│  │  ┌─────────────────────────────────────┐    │               │
│  │  │  books (id, title, author, year,   │    │               │
│  │  │  isbn, publisher, category)        │    │               │
│  │  └─────────────────────────────────────┘    │               │
│  └─────────────────────────────────────────────┘               │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2. Идемпотентность

**Идемпотентность** — свойство операции, при котором повторное выполнение даёт тот же результат, что и первое.

**Почему важна идемпотентность в ETL:**
- Пайплайн может быть перезапущен.
- Могут возникать ошибки сети.
- Данные могут дублироваться.

**Способы обеспечения идемпотентности:**
1. **UPSERT** (INSERT ... ON CONFLICT) — если запись существует, обновить.
2. **DELETE + INSERT** — удалить старые записи и вставить новые.
3. **MERGE** — операция слияния (поддерживается в некоторых СУБД).
4. **Использование хэшей** — проверка изменений перед обновлением.

```sql
-- UPSERT в PostgreSQL
INSERT INTO books (title, author, year, isbn, publisher, category)
VALUES (?, ?, ?, ?, ?, ?)
ON CONFLICT (isbn) DO UPDATE SET
    title = EXCLUDED.title,
    author = EXCLUDED.author,
    year = EXCLUDED.year,
    publisher = EXCLUDED.publisher,
    category = EXCLUDED.category
```

### 2.3. Dead-Letter Queue (DLQ)

**Dead-Letter Queue** — механизм для сохранения записей, которые не удалось обработать.

```
┌─────────────────────────────────────────────────────────────────┐
│                  Dead-Letter Queue                             │
│                                                                 │
│  ┌─────────────────┐    ┌─────────────────┐                    │
│  │  Ошибочные      │───▶│  etl_errors.csv │                    │
│  │  записи         │    │  (для анализа)  │                    │
│  └─────────────────┘    └─────────────────┘                    │
│                                                                 │
│  Структура etl_errors.csv:                                     │
│  timestamp, source_file, line_number, raw_data, error_message  │
└─────────────────────────────────────────────────────────────────┘
```

### 2.4. Мониторинг ETL-процесса

**Метрики для мониторинга:**

| Метрика | Описание |
|---------|----------|
| extracted | Количество извлечённых записей |
| transformed | Количество успешно преобразованных |
| loaded | Количество загруженных записей |
| errors | Количество ошибок |
| duration | Время выполнения |
| success_rate | Процент успешной обработки |

### 2.5. Иммутабельные объекты

**Иммутабельный объект** — объект, состояние которого не может быть изменено после создания.

**Способы создания в Java:**
1. `record` (Java 14+) — автоматически иммутабелен.
2. `final` класс с `final` полями.
3. Класс с геттерами, но без сеттеров.

```java
// Использование record (Java 14+)
public record EtlResult(
    int extracted,
    int transformed,
    int loaded,
    int errors,
    long durationMs,
    List<Book> books,
    Map<String, Long> categoryStats
) {
    // record автоматически предоставляет:
    // - конструктор со всеми полями
    // - геттеры
    // - equals(), hashCode(), toString()
}

// Или final-класс
public final class EtlResult {
    private final int extracted;
    private final int transformed;
    private final int loaded;
    private final int errors;
    private final long durationMs;
    private final List<Book> books;
    private final Map<String, Long> categoryStats;
    
    // Конструктор и геттеры
}
```

### 2.6. Стратегии обработки ошибок в ETL

| Стратегия | Описание | Когда применять |
|-----------|----------|-----------------|
| **Fail-fast** | Остановка при первой ошибке | Критические данные |
| **Skip-and-log** | Пропуск ошибочной записи, логирование | Некритические данные |
| **Dead-letter queue** | Сохранение ошибочных записей | Для последующего анализа |
| **Retry** | Повторная попытка | Временные ошибки |

---

## 3. Задание на паре

### Задача. Завершение ETL-пайплайна

1. **Реализовать этап Load:**
   - Запись очищенных данных в PostgreSQL через `BookDao`.
   - Использовать UPSERT для обеспечения идемпотентности.
   - Фиксировать количество успешно загруженных записей.

2. **Собрать весь пайплайн в единый класс `BookEtlPipeline`:**
   - Метод `run(String sourcePath, String sourceFormat)`.
   - Возвращает `EtlResult` с метриками.

3. **Создать `EtlResult` как иммутабельный объект:**
   - Использовать `record` или `final`-класс.
   - Включить метрики: extracted, transformed, loaded, errors, duration.

4. **Добавить логирование каждого этапа:**
   - Начало/завершение каждого этапа.
   - Количество обработанных записей.
   - Ошибки.

5. **Реализовать dead-letter-queue:**
   - Сохранение записей с ошибками в файл `etl_errors.csv`.
   - Включить timestamp, источник, строку, сообщение об ошибке.

6. **Протестировать пайплайн:**
   - Запустить на 500+ записях, включая некорректные.
   - Проверить идемпотентность повторным запуском.

**Пример выполнения:**
```
=== Запуск ETL-пайплайна ===
2026-01-15 10:30:15 [main] INFO  BookEtlPipeline - Начало ETL-пайплайна
2026-01-15 10:30:15 [main] INFO  BookEtlPipeline - Источник: books.csv, формат: CSV

--- EXTRACT ---
2026-01-15 10:30:15 [main] INFO  DataSourceReader - Чтение данных из: books.csv (формат: CSV)
2026-01-15 10:30:15 [main] INFO  CsvParserStrategy - Парсинг CSV-файла: books.csv
2026-01-15 10:30:15 [main] INFO  CsvParserStrategy - CSV-парсинг завершён: 550 книг, ошибок: 23
2026-01-15 10:30:15 [main] INFO  DataSourceReader - Успешно прочитано 550 книг
2026-01-15 10:30:15 [main] INFO  BookEtlPipeline - Извлечено: 550 записей

--- TRANSFORM ---
2026-01-15 10:30:15 [main] INFO  AuthorNormalizer - Нормализация авторов: 550 записей
2026-01-15 10:30:15 [main] INFO  AuthorNormalizer - Нормализовано: 550 записей
2026-01-15 10:30:15 [main] INFO  BookFilter - Фильтрация: 550 записей
2026-01-15 10:30:15 [main] WARN  BookFilter - Исключено: 15 книг без ISBN
2026-01-15 10:30:15 [main] INFO  BookFilter - Осталось: 535 записей
2026-01-15 10:30:15 [main] INFO  CategoryEnricher - Обогащение категориями: 535 записей
2026-01-15 10:30:15 [main] INFO  CategoryEnricher - Обогащено: 535 записей
2026-01-15 10:30:15 [main] INFO  BookEtlPipeline - Преобразовано: 535 записей

--- LOAD ---
2026-01-15 10:30:16 [main] INFO  BookDaoImpl - UPSERT: 535 записей
2026-01-15 10:30:16 [main] INFO  BookDaoImpl - Загружено: 535 записей
2026-01-15 10:30:16 [main] INFO  BookEtlPipeline - Загружено: 535 записей

--- DLQ ---
2026-01-15 10:30:16 [main] WARN  BookEtlPipeline - Сохранено ошибочных записей: 15
2026-01-15 10:30:16 [main] INFO  BookEtlPipeline - DLQ: etl_errors.csv

=== Результат ETL ===
Извлечено: 550
Преобразовано: 535
Загружено: 535
Ошибок: 15
Время: 125 мс

Статистика по категориям:
  IT: 125
  Художественная литература: 98
  Военная литература: 45
  Детектив: 32
  Фэнтези: 28
  Другое: 207

=== Проверка идемпотентности ===
Второй запуск (UPSERT):
  Загружено: 535 (обновлено существующих)

Идемпотентность: OK
```

### Применение ИИ-инструмента

**Промпт для YandexGPT:**
```
Проанализируй код ETL-пайплайна и предложи улучшения по обработке ошибок и читаемости.

Код:
[вставьте код EtlPipeline]

Требования к анализу:
1. Обработка исключений.
2. Логирование.
3. Читаемость кода.
4. Архитектурные решения.
5. Производительность.
6. Безопасность.
```

**Анализ результата:**
- Проверить полноту анализа.
- Проверить конкретность рекомендаций.
- Проверить практическую применимость предложений.

---

## 4. Индивидуальные задания (30 вариантов)

Каждый вариант содержит:
- Специфические правила загрузки.
- Особые требования к идемпотентности.
- Специфическую структуру DLQ.

---

### Вариант 1. Книги (Book)

**Загрузка:** UPSERT по ISBN, обновление всех полей

**Идемпотентность:** ON CONFLICT DO UPDATE

**DLQ:** etl_errors.csv с полями: timestamp, line, raw_data, error

---

### Вариант 2. Сотрудники (Employee)

**Загрузка:** UPSERT по email, обновление всех полей

**Идемпотентность:** ON CONFLICT DO UPDATE

**DLQ:** employee_errors.csv

---

### Вариант 3. Товары (Product)

**Загрузка:** UPSERT по SKU, обновление всех полей

**Идемпотентность:** ON CONFLICT DO UPDATE

**DLQ:** product_errors.csv

---

### Вариант 4. Заказы (Order)

**Загрузка:** UPSERT по order_number, обновление статуса

**Идемпотентность:** ON CONFLICT DO UPDATE

**DLQ:** order_errors.csv

---

### Вариант 5. Пользователи (User)

**Загрузка:** UPSERT по email, обновление роли

**Идемпотентность:** ON CONFLICT DO UPDATE

**DLQ:** user_errors.csv

---

### Вариант 6. Автомобили (Car)

**Загрузка:** UPSERT по VIN, обновление всех полей

**Идемпотентность:** ON CONFLICT DO UPDATE

**DLQ:** car_errors.csv

---

### Вариант 7. Студенты (Student)

**Загрузка:** UPSERT по student_id, обновление GPA

**Идемпотентность:** ON CONFLICT DO UPDATE

**DLQ:** student_errors.csv

---

### Вариант 8. Счета (Account)

**Загрузка:** UPSERT по account_number, обновление баланса

**Идемпотентность:** ON CONFLICT DO UPDATE

**DLQ:** account_errors.csv

---

### Вариант 9. Фильмы (Movie)

**Загрузка:** UPSERT по imdb_id, обновление рейтинга

**Идемпотентность:** ON CONFLICT DO UPDATE

**DLQ:** movie_errors.csv

---

### Вариант 10. Рестораны (Restaurant)

**Загрузка:** UPSERT по tax_id, обновление рейтинга

**Идемпотентность:** ON CONFLICT DO UPDATE

**DLQ:** restaurant_errors.csv

---

### Вариант 11. Транзакции (Transaction)

**Загрузка:** UPSERT по tx_id, обновление всех полей

**Идемпотентность:** ON CONFLICT DO UPDATE

**DLQ:** transaction_errors.csv

---

### Вариант 12. Клиенты (Customer)

**Загрузка:** UPSERT по customer_code, обновление телефона

**Идемпотентность:** ON CONFLICT DO UPDATE

**DLQ:** customer_errors.csv

---

### Вариант 13. Договоры (Contract)

**Загрузка:** UPSERT по contract_number, обновление статуса

**Идемпотентность:** ON CONFLICT DO UPDATE

**DLQ:** contract_errors.csv

---

### Вариант 14. Отели (Hotel)

**Загрузка:** UPSERT по hotel_code, обновление рейтинга

**Идемпотентность:** ON CONFLICT DO UPDATE

**DLQ:** hotel_errors.csv

---

### Вариант 15. Спортсмены (Athlete)

**Загрузка:** UPSERT по athlete_id, обновление медалей

**Идемпотентность:** ON CONFLICT DO UPDATE

**DLQ:** athlete_errors.csv

---

### Вариант 16. Статьи (Article)

**Загрузка:** UPSERT по DOI, обновление просмотров

**Идемпотентность:** ON CONFLICT DO UPDATE

**DLQ:** article_errors.csv

---

### Вариант 17. Билеты (Ticket)

**Загрузка:** UPSERT по ticket_code, обновление цены

**Идемпотентность:** ON CONFLICT DO UPDATE

**DLQ:** ticket_errors.csv

---

### Вариант 18. Курсы (Course)

**Загрузка:** UPSERT по course_code, обновление цены

**Идемпотентность:** ON CONFLICT DO UPDATE

**DLQ:** course_errors.csv

---

### Вариант 19. Инвентарь (Item)

**Загрузка:** UPSERT по serial_number, обновление количества

**Идемпотентность:** ON CONFLICT DO UPDATE

**DLQ:** inventory_errors.csv

---

### Вариант 20. Заявки (Request)

**Загрузка:** UPSERT по request_number, обновление статуса

**Идемпотентность:** ON CONFLICT DO UPDATE

**DLQ:** request_errors.csv

---

### Вариант 21. Платежи (Payment)

**Загрузка:** UPSERT по payment_id, обновление статуса

**Идемпотентность:** ON CONFLICT DO UPDATE

**DLQ:** payment_errors.csv

---

### Вариант 22. Публикации (Publication)

**Загрузка:** UPSERT по DOI, обновление года

**Идемпотентность:** ON CONFLICT DO UPDATE

**DLQ:** publication_errors.csv

---

### Вариант 23. Ноутбуки (Laptop)

**Загрузка:** UPSERT по serial, обновление цены

**Идемпотентность:** ON CONFLICT DO UPDATE

**DLQ:** laptop_errors.csv

---

### Вариант 24. Врачи (Doctor)

**Загрузка:** UPSERT по license, обновление рейтинга

**Идемпотентность:** ON CONFLICT DO UPDATE

**DLQ:** doctor_errors.csv

---

### Вариант 25. Сертификаты (Certificate)

**Загрузка:** UPSERT по cert_id, обновление срока действия

**Идемпотентность:** ON CONFLICT DO UPDATE

**DLQ:** certificate_errors.csv

---

### Вариант 26. Магазины (Shop)

**Загрузка:** UPSERT по shop_code, обновление рейтинга

**Идемпотентность:** ON CONFLICT DO UPDATE

**DLQ:** shop_errors.csv

---

### Вариант 27. Здания (Building)

**Загрузка:** UPSERT по building_id, обновление владельца

**Идемпотентность:** ON CONFLICT DO UPDATE

**DLQ:** building_errors.csv

---

### Вариант 28. Лекции (Lecture)

**Загрузка:** UPSERT по lecture_code, обновление слушателей

**Идемпотентность:** ON CONFLICT DO UPDATE

**DLQ:** lecture_errors.csv

---

### Вариант 29. Отзывы (Review)

**Загрузка:** UPSERT по review_id, обновление рейтинга

**Идемпотентность:** ON CONFLICT DO UPDATE

**DLQ:** review_errors.csv

---

### Вариант 30. Чеки (Receipt)

**Загрузка:** UPSERT по receipt_number, обновление суммы

**Идемпотентность:** ON CONFLICT DO UPDATE

**DLQ:** receipt_errors.csv

---

## 5. Методические указания

### 5.1. Структура проекта

```
src/
├── main/
│   ├── java/
│   │   ├── model/
│   │   │   └── Book.java
│   │   ├── dao/
│   │   │   └── BookDao.java
│   │   ├── etl/
│   │   │   ├── reader/
│   │   │   │   ├── DataSourceReader.java
│   │   │   │   ├── ParserStrategy.java
│   │   │   │   ├── CsvParserStrategy.java
│   │   │   │   └── JsonParserStrategy.java
│   │   │   ├── transform/
│   │   │   │   ├── AuthorNormalizer.java
│   │   │   │   ├── BookFilter.java
│   │   │   │   └── CategoryEnricher.java
│   │   │   ├── load/
│   │   │   │   └── BookLoader.java
│   │   │   ├── dlq/
│   │   │   │   └── DeadLetterQueue.java
│   │   │   └── pipeline/
│   │   │       ├── BookEtlPipeline.java
│   │   │       └── EtlResult.java
│   │   └── Main.java
│   └── resources/
│       ├── books.csv
│       ├── books.json
│       ├── categories.csv
│       └── logback.xml
└── test/
    └── java/
        └── etl/
            └── pipeline/
                └── BookEtlPipelineTest.java
```

### 5.2. Класс EtlResult (record)

```java
package etl.pipeline;

import model.Book;

import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

/**
 * Иммутабельный результат ETL-пайплайна
 */
public record EtlResult(
    int extracted,      // количество извлечённых записей
    int transformed,    // количество преобразованных записей
    int loaded,         // количество загруженных записей
    int errors,         // количество ошибок
    long durationMs,    // время выполнения в миллисекундах
    List<Book> books,   // загруженные книги
    List<ErrorRecord> errorRecords  // записи с ошибками
) {
    
    /**
     * Получение статистики по категориям
     */
    public Map<String, Long> getCategoryStats() {
        if (books == null || books.isEmpty()) {
            return Map.of();
        }
        return books.stream()
            .collect(Collectors.groupingBy(
                Book::getCategory,
                Collectors.counting()
            ));
    }
    
    /**
     * Получение процента успешной обработки
     */
    public double getSuccessRate() {
        if (extracted == 0) {
            return 0.0;
        }
        return (double) loaded / extracted * 100;
    }
    
    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder();
        sb.append("=== Результат ETL ===\n");
        sb.append("Извлечено (Extract): ").append(extracted).append("\n");
        sb.append("Преобразовано (Transform): ").append(transformed).append("\n");
        sb.append("Загружено (Load): ").append(loaded).append("\n");
        sb.append("Ошибок: ").append(errors).append("\n");
        sb.append("Успешность: ").append(String.format("%.2f%%", getSuccessRate())).append("\n");
        sb.append("Время: ").append(durationMs).append(" мс\n");
        
        if (!getCategoryStats().isEmpty()) {
            sb.append("\nСтатистика по категориям:\n");
            getCategoryStats().entrySet().stream()
                .sorted(Map.Entry.<String, Long>comparingByValue().reversed())
                .forEach(e -> sb.append("  ").append(e.getKey())
                    .append(": ").append(e.getValue()).append("\n"));
        }
        
        return sb.toString();
    }
}
```

### 5.3. Класс ErrorRecord

```java
package etl.pipeline;

import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

/**
 * Запись об ошибке для DLQ
 */
public record ErrorRecord(
    LocalDateTime timestamp,
    String sourceFile,
    int lineNumber,
    String rawData,
    String errorMessage
) {
    private static final DateTimeFormatter FORMATTER = 
        DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
    
    /**
     * Преобразование в CSV-строку для записи в DLQ
     */
    public String toCsv() {
        return String.format("%s,\"%s\",%d,\"%s\",\"%s\"",
            timestamp.format(FORMATTER),
            sourceFile != null ? sourceFile.replace("\"", "\"\"") : "",
            lineNumber,
            rawData != null ? rawData.replace("\"", "\"\"") : "",
            errorMessage != null ? errorMessage.replace("\"", "\"\"") : ""
        );
    }
    
    /**
     * Получение заголовка CSV для DLQ
     */
    public static String getCsvHeader() {
        return "timestamp,source_file,line_number,raw_data,error_message";
    }
}
```

### 5.4. Класс BookLoader

```java
package etl.load;

import config.DataSourceProvider;
import model.Book;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;
import java.util.List;

public class BookLoader {
    private static final Logger logger = LoggerFactory.getLogger(BookLoader.class);
    
    /**
     * Загрузка книг в БД с использованием UPSERT
     * @param books список книг для загрузки
     * @return количество успешно загруженных книг
     */
    public int load(List<Book> books) throws SQLException {
        if (books == null || books.isEmpty()) {
            logger.info("Нет данных для загрузки");
            return 0;
        }
        
        logger.info("Начало загрузки {} книг", books.size());
        long startTime = System.currentTimeMillis();
        int loaded = 0;
        
        String sql = """
            INSERT INTO books (title, author, year, isbn, publisher, category)
            VALUES (?, ?, ?, ?, ?, ?)
            ON CONFLICT (isbn) DO UPDATE SET
                title = EXCLUDED.title,
                author = EXCLUDED.author,
                year = EXCLUDED.year,
                publisher = EXCLUDED.publisher,
                category = EXCLUDED.category
        """;
        
        try (Connection conn = DataSourceProvider.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            conn.setAutoCommit(false);
            
            for (Book book : books) {
                pstmt.setString(1, book.getTitle());
                pstmt.setString(2, book.getAuthor());
                pstmt.setInt(3, book.getYear());
                pstmt.setString(4, book.getIsbn());
                pstmt.setString(5, book.getPublisher());
                pstmt.setString(6, book.getCategory());
                pstmt.addBatch();
                loaded++;
            }
            
            int[] results = pstmt.executeBatch();
            conn.commit();
            
            long duration = System.currentTimeMillis() - startTime;
            logger.info("Загрузка завершена: {} книг за {} мс", results.length, duration);
            
        } catch (SQLException e) {
            logger.error("Ошибка при загрузке книг", e);
            throw e;
        }
        
        return loaded;
    }
    
    /**
     * Проверка существования таблицы и её создание при необходимости
     */
    public void ensureTableExists() throws SQLException {
        String createTableSQL = """
            CREATE TABLE IF NOT EXISTS books (
                id SERIAL PRIMARY KEY,
                title VARCHAR(255) NOT NULL,
                author VARCHAR(255) NOT NULL,
                year INTEGER NOT NULL,
                isbn VARCHAR(20) UNIQUE NOT NULL,
                publisher VARCHAR(255),
                category VARCHAR(100)
            )
        """;
        
        try (Connection conn = DataSourceProvider.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(createTableSQL)) {
            pstmt.executeUpdate();
            logger.info("Таблица books проверена/создана");
        }
    }
}
```

### 5.5. Класс DeadLetterQueue

```java
package etl.dlq;

import etl.pipeline.ErrorRecord;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.BufferedWriter;
import java.io.FileWriter;
import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.nio.file.StandardOpenOption;
import java.util.ArrayList;
import java.util.List;

public class DeadLetterQueue {
    private static final Logger logger = LoggerFactory.getLogger(DeadLetterQueue.class);
    private final String filePath;
    private final List<ErrorRecord> errors = new ArrayList<>();
    
    public DeadLetterQueue(String filePath) {
        this.filePath = filePath;
        initFile();
    }
    
    private void initFile() {
        try {
            Path path = Paths.get(filePath);
            if (!Files.exists(path)) {
                // Создаём директорию при необходимости
                if (path.getParent() != null) {
                    Files.createDirectories(path.getParent());
                }
                // Создаём файл с заголовком
                Files.writeString(path, ErrorRecord.getCsvHeader() + "\n", 
                    StandardCharsets.UTF_8, StandardOpenOption.CREATE);
                logger.info("DLQ создан: {}", filePath);
            }
        } catch (IOException e) {
            logger.error("Ошибка создания DLQ: {}", filePath, e);
        }
    }
    
    /**
     * Добавление ошибки в DLQ
     */
    public void addError(String sourceFile, int lineNumber, String rawData, String errorMessage) {
        ErrorRecord record = new ErrorRecord(
            java.time.LocalDateTime.now(),
            sourceFile,
            lineNumber,
            rawData,
            errorMessage
        );
        errors.add(record);
        logger.debug("Добавлена ошибка в DLQ: строка {}", lineNumber);
    }
    
    /**
     * Добавление ошибки в DLQ с автоматической записью
     */
    public void addErrorAndWrite(String sourceFile, int lineNumber, String rawData, String errorMessage) {
        addError(sourceFile, lineNumber, rawData, errorMessage);
        flush();
    }
    
    /**
     * Запись всех ошибок в файл
     */
    public void flush() {
        if (errors.isEmpty()) {
            return;
        }
        
        try (BufferedWriter writer = new BufferedWriter(
                new FileWriter(filePath, StandardCharsets.UTF_8, true))) {
            
            for (ErrorRecord error : errors) {
                writer.write(error.toCsv());
                writer.newLine();
            }
            
            logger.info("Записано {} ошибок в DLQ", errors.size());
            errors.clear();
            
        } catch (IOException e) {
            logger.error("Ошибка записи в DLQ: {}", filePath, e);
        }
    }
    
    /**
     * Получение количества ошибок
     */
    public int getErrorCount() {
        return errors.size();
    }
    
    /**
     * Получение всех ошибок (без записи в файл)
     */
    public List<ErrorRecord> getErrors() {
        return new ArrayList<>(errors);
    }
}
```

### 5.6. Класс BookEtlPipeline

```java
package etl.pipeline;

import etl.dlq.DeadLetterQueue;
import etl.load.BookLoader;
import etl.reader.CsvParserStrategy;
import etl.reader.DataSourceReader;
import etl.reader.JsonParserStrategy;
import etl.reader.ParserStrategy;
import etl.transform.AuthorNormalizer;
import etl.transform.BookFilter;
import etl.transform.CategoryEnricher;
import model.Book;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.IOException;
import java.sql.SQLException;
import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.List;

public class BookEtlPipeline {
    private static final Logger logger = LoggerFactory.getLogger(BookEtlPipeline.class);
    
    private final DataSourceReader reader;
    private final AuthorNormalizer authorNormalizer;
    private final BookFilter filter;
    private final CategoryEnricher enricher;
    private final BookLoader loader;
    private final DeadLetterQueue dlq;
    
    public BookEtlPipeline(DataSourceReader reader, 
                           AuthorNormalizer authorNormalizer,
                           BookFilter filter, 
                           CategoryEnricher enricher,
                           BookLoader loader,
                           DeadLetterQueue dlq) {
        this.reader = reader;
        this.authorNormalizer = authorNormalizer;
        this.filter = filter;
        this.enricher = enricher;
        this.loader = loader;
        this.dlq = dlq;
    }
    
    /**
     * Запуск ETL-пайплайна
     * @param sourcePath путь к источнику данных
     * @param sourceFormat формат источника (CSV, JSON)
     * @return результат ETL
     */
    public EtlResult run(String sourcePath, String sourceFormat) {
        logger.info("=== Запуск ETL-пайплайна ===");
        logger.info("Источник: {}, формат: {}", sourcePath, sourceFormat);
        long startTime = System.currentTimeMillis();
        
        List<ErrorRecord> errorRecords = new ArrayList<>();
        List<Book> extractedBooks = new ArrayList<>();
        List<Book> transformedBooks = new ArrayList<>();
        List<Book> loadedBooks = new ArrayList<>();
        
        int extractedCount = 0;
        int transformedCount = 0;
        int loadedCount = 0;
        int errorCount = 0;
        
        try {
            // 1. EXTRACT
            logger.info("--- EXTRACT ---");
            ParserStrategy strategy = getStrategy(sourceFormat);
            reader.setStrategy(strategy);
            extractedBooks = reader.read(sourcePath);
            extractedCount = extractedBooks.size();
            logger.info("Извлечено: {} записей", extractedCount);
            
            // 2. TRANSFORM
            logger.info("--- TRANSFORM ---");
            List<Book> normalizedBooks = new ArrayList<>();
            for (Book book : extractedBooks) {
                try {
                    String normalizedAuthor = authorNormalizer.normalize(book.getAuthor());
                    book.setAuthor(normalizedAuthor);
                    
                    if (!filter.isValid(book)) {
                        // Запись не прошла фильтрацию
                        String errorMsg = "Не прошла валидацию (ISBN: " + book.getIsbn() + ")";
                        dlq.addError(sourcePath, 0, book.toString(), errorMsg);
                        errorCount++;
                        continue;
                    }
                    
                    Book enriched = enricher.enrich(book);
                    normalizedBooks.add(enriched);
                } catch (Exception e) {
                    errorCount++;
                    dlq.addError(sourcePath, 0, book.toString(), e.getMessage());
                    logger.warn("Ошибка трансформации: {}", e.getMessage());
                }
            }
            transformedBooks = normalizedBooks;
            transformedCount = transformedBooks.size();
            logger.info("Преобразовано: {} записей", transformedCount);
            logger.info("Отфильтровано/ошибок: {}", errorCount);
            
            // 3. LOAD
            if (!transformedBooks.isEmpty()) {
                logger.info("--- LOAD ---");
                try {
                    loader.ensureTableExists();
                    loadedCount = loader.load(transformedBooks);
                    loadedBooks = transformedBooks;
                    logger.info("Загружено: {} записей", loadedCount);
                } catch (SQLException e) {
                    logger.error("Ошибка загрузки в БД", e);
                    errorCount += transformedBooks.size();
                    // Записываем все книги в DLQ
                    for (Book book : transformedBooks) {
                        dlq.addError(sourcePath, 0, book.toString(), "Ошибка загрузки: " + e.getMessage());
                    }
                }
            }
            
            // 4. DLQ
            logger.info("--- DLQ ---");
            dlq.flush();
            logger.info("Сохранено ошибок в DLQ: {}", dlq.getErrorCount());
            
        } catch (IOException e) {
            logger.error("Критическая ошибка ETL", e);
            errorCount += extractedBooks.size();
            dlq.addError(sourcePath, 0, "", "Критическая ошибка: " + e.getMessage());
            dlq.flush();
        }
        
        long duration = System.currentTimeMillis() - startTime;
        logger.info("=== ETL завершён ===");
        logger.info("Время выполнения: {} мс", duration);
        
        // Создаём результат
        List<ErrorRecord> allErrors = dlq.getErrors();
        return new EtlResult(
            extractedCount,
            transformedCount,
            loadedCount,
            errorCount,
            duration,
            loadedBooks,
            allErrors
        );
    }
    
    private ParserStrategy getStrategy(String format) {
        return switch (format.toUpperCase()) {
            case "CSV" -> new CsvParserStrategy();
            case "JSON" -> new JsonParserStrategy();
            default -> throw new IllegalArgumentException("Неподдерживаемый формат: " + format);
        };
    }
}
```

### 5.7. Основной класс

```java
import config.DataSourceProvider;
import etl.dlq.DeadLetterQueue;
import etl.load.BookLoader;
import etl.pipeline.BookEtlPipeline;
import etl.pipeline.EtlResult;
import etl.reader.DataSourceReader;
import etl.transform.AuthorNormalizer;
import etl.transform.BookFilter;
import etl.transform.CategoryEnricher;

import java.io.IOException;

public class Main {
    public static void main(String[] args) {
        String categoriesFile = "src/main/resources/categories.csv";
        String sourceFile = "src/main/resources/books.csv";
        
        try {
            // Инициализация компонентов
            DataSourceReader reader = new DataSourceReader(null);
            AuthorNormalizer normalizer = new AuthorNormalizer();
            BookFilter filter = new BookFilter();
            CategoryEnricher enricher = new CategoryEnricher(categoriesFile);
            BookLoader loader = new BookLoader();
            DeadLetterQueue dlq = new DeadLetterQueue("logs/etl_errors.csv");
            
            BookEtlPipeline pipeline = new BookEtlPipeline(
                reader, normalizer, filter, enricher, loader, dlq
            );
            
            // Первый запуск
            System.out.println("\n=== Первый запуск ETL ===\n");
            EtlResult result1 = pipeline.run(sourceFile, "CSV");
            System.out.println(result1);
            
            // Проверка идемпотентности — повторный запуск
            System.out.println("\n=== Проверка идемпотентности (повторный запуск) ===\n");
            EtlResult result2 = pipeline.run(sourceFile, "CSV");
            System.out.println(result2);
            
            System.out.println("\n=== Идемпотентность ===");
            System.out.println("Первая загрузка: " + result1.loaded() + " записей");
            System.out.println("Вторая загрузка: " + result2.loaded() + " записей");
            System.out.println("Статус: " + (result1.loaded() == result2.loaded() ? "OK" : "ОШИБКА"));
            
        } catch (IOException e) {
            System.err.println("Ошибка: " + e.getMessage());
            e.printStackTrace();
        } finally {
            DataSourceProvider.close();
        }
    }
}
```

---

## 6. Контрольные вопросы

1. Что такое этап Load в ETL и какие задачи он решает?

2. Что такое идемпотентность и почему она важна в ETL?

3. Как обеспечить идемпотентность при загрузке данных?

4. Что такое UPSERT и как он реализуется в PostgreSQL?

5. Что такое Dead-Letter Queue и для чего она используется?

6. Какие метрики важны для мониторинга ETL-процесса?

7. Что такое иммутабельный объект и как его создать в Java?

8. В чём преимущества использования record в Java?

9. Какие стратегии обработки ошибок в ETL вы знаете?

10. Как организовать логирование в ETL-пайплайне?

11. Как проверить идемпотентность пайплайна?

12. Какие данные должны сохраняться в DLQ?

13. Как обрабатывать транзакции при загрузке данных?

14. Как обеспечить атомарность загрузки?

15. Какие ещё целевые системы могут использоваться для загрузки?

---

## 7. Пример выполнения (Вариант 1)

### 7.1. Ожидаемый вывод

```
=== Первый запуск ETL ===
2026-01-15 10:30:15 [main] INFO  BookEtlPipeline - === Запуск ETL-пайплайна ===
2026-01-15 10:30:15 [main] INFO  BookEtlPipeline - Источник: src/main/resources/books.csv, формат: CSV
2026-01-15 10:30:15 [main] INFO  BookEtlPipeline - --- EXTRACT ---
2026-01-15 10:30:15 [main] INFO  DataSourceReader - Чтение данных из: src/main/resources/books.csv (формат: CSV)
2026-01-15 10:30:15 [main] INFO  CsvParserStrategy - Парсинг CSV-файла: src/main/resources/books.csv
2026-01-15 10:30:15 [main] INFO  CsvParserStrategy - CSV-парсинг завершён: 550 книг, ошибок: 23
2026-01-15 10:30:15 [main] INFO  DataSourceReader - Успешно прочитано 550 книг
2026-01-15 10:30:15 [main] INFO  BookEtlPipeline - Извлечено: 550 записей
2026-01-15 10:30:15 [main] INFO  BookEtlPipeline - --- TRANSFORM ---
2026-01-15 10:30:15 [main] INFO  AuthorNormalizer - Нормализация авторов: 550 записей
2026-01-15 10:30:15 [main] INFO  AuthorNormalizer - Нормализовано: 550 записей
2026-01-15 10:30:15 [main] INFO  BookFilter - Фильтрация: 550 записей
2026-01-15 10:30:15 [main] WARN  BookFilter - Исключено: 15 книг без ISBN
2026-01-15 10:30:15 [main] INFO  BookFilter - Осталось: 535 записей
2026-01-15 10:30:15 [main] INFO  CategoryEnricher - Обогащение категориями: 535 записей
2026-01-15 10:30:15 [main] INFO  CategoryEnricher - Обогащено: 535 записей
2026-01-15 10:30:15 [main] INFO  BookEtlPipeline - Преобразовано: 535 записей
2026-01-15 10:30:15 [main] INFO  BookEtlPipeline - Отфильтровано/ошибок: 15
2026-01-15 10:30:15 [main] INFO  BookEtlPipeline - --- LOAD ---
2026-01-15 10:30:15 [main] INFO  BookLoader - Таблица books проверена/создана
2026-01-15 10:30:16 [main] INFO  BookLoader - Начало загрузки 535 книг
2026-01-15 10:30:16 [main] INFO  BookLoader - Загрузка завершена: 535 книг за 78 мс
2026-01-15 10:30:16 [main] INFO  BookEtlPipeline - Загружено: 535 записей
2026-01-15 10:30:16 [main] INFO  BookEtlPipeline - --- DLQ ---
2026-01-15 10:30:16 [main] INFO  DeadLetterQueue - Записано 15 ошибок в DLQ
2026-01-15 10:30:16 [main] INFO  DeadLetterQueue - Сохранено ошибок в DLQ: 15
2026-01-15 10:30:16 [main] INFO  BookEtlPipeline - === ETL завершён ===
2026-01-15 10:30:16 [main] INFO  BookEtlPipeline - Время выполнения: 125 мс

=== Результат ETL ===
Извлечено (Extract): 550
Преобразовано (Transform): 535
Загружено (Load): 535
Ошибок: 15
Успешность: 97.27%
Время: 125 мс

Статистика по категориям:
  IT: 125
  Художественная литература: 98
  Военная литература: 45
  Детектив: 32
  Фэнтези: 28
  Другое: 207

=== Проверка идемпотентности (повторный запуск) ===
2026-01-15 10:30:17 [main] INFO  BookEtlPipeline - === Запуск ETL-пайплайна ===
...
2026-01-15 10:30:18 [main] INFO  BookLoader - Загрузка завершена: 535 книг за 45 мс
...

=== Результат ETL (повторный) ===
Извлечено (Extract): 550
Преобразовано (Transform): 535
Загружено (Load): 535
Ошибок: 15
Успешность: 97.27%
Время: 98 мс

=== Идемпотентность ===
Первая загрузка: 535 записей
Вторая загрузка: 535 записей
Статус: OK

=== Проверка DLQ ===
Файл: logs/etl_errors.csv
Содержит 15 записей с ошибками
```

### 7.2. Содержимое etl_errors.csv

```csv
timestamp,source_file,line_number,raw_data,error_message
2026-01-15 10:30:15,books.csv,12,"12,,Автор,2024,,Издательство",Не прошла валидацию (ISBN: )
2026-01-15 10:30:15,books.csv,25,"25,Книга,Автор,2025,,Издательство",Не прошла валидацию (ISBN: )
...
```

---

## 8. Рекомендуемые источники

1. **Kimball R., Caserta J.** *The Data Warehouse ETL Toolkit.* — Wiley.

2. **Шилдт Г.** *Java. Базовый курс.* — М.: Вильямс. — Главы 15-16 (Ввод-вывод, JDBC).

3. **Хорстманн К., Корнелл Г.** *Java. Библиотека профессионала. Том 2.* — М.: Вильямс. — Главы 2, 4 (Ввод-вывод, Базы данных).

4. **PostgreSQL Documentation: INSERT ON CONFLICT.** — URL: https://www.postgresql.org/docs/current/sql-insert.html

5. **HikariCP GitHub.** — URL: https://github.com/brettwooldridge/HikariCP

6. **SLF4J Documentation.** — URL: https://www.slf4j.org/docs.html

7. **Logback Documentation.** — URL: https://logback.qos.ch/documentation.html
