# Лабораторная работа №16. Мини-проект: проектирование и начало реализации

## 1. Паспорт работы

| Параметр | Значение |
|----------|----------|
| Номер занятия | 16 из 17 |
| Блок | 7. Промпт-инжиниринг |
| Продолжительность | 2 академических часа |
| Форма выполнения | Командная (2-3 человека) |
| ИИ-инструмент | Copilot / JetBrains AI Assistant / ChatGPT / YandexGPT / GigaChat |

### 1.1. Цель работы

Спроектировать и начать реализацию командного мини-проекта «Сервис загрузки и очистки прайс-листов», применить на практике все знания, полученные в ходе курса: ООП, коллекции, исключения, ввод-вывод, многопоточность, JDBC, ETL, промпт-инжиниринг.

### 1.2. Задачи работы

1. Изучить постановку задачи и требования к проекту.
2. Распределить роли в команде.
3. Спроектировать архитектуру приложения (диаграмма классов).
4. Определить интерфейсы между модулями.
5. Инициализировать Git-репозиторий.
6. Настроить Maven/Gradle и подключить зависимости.
7. Сгенерировать каркас проекта с помощью ИИ.
8. Начать реализацию базовых модулей.

### 1.3. Оснащение

- JDK 17 или выше;
- IntelliJ IDEA Community Edition;
- Git/GitHub/GitLab;
- Maven или Gradle;
- PostgreSQL (локально или в Docker);
- Доступ к ИИ-инструментам.

---

## 2. Теоретический конспект

### 2.1. Постановка задачи

**Сервис загрузки и очистки прайс-листов**

Разработать консольное Java-приложение, которое:

1. **Принимает на вход** CSV или JSON-файл с прайс-листом поставщика.
2. **Выполняет ETL:**
   - **Очистка** — удаление дубликатов.
   - **Нормализация** — приведение валюты к единому стандарту (RUB) по курсу ЦБ на дату.
   - **Обогащение** — добавление категории товара из справочника.
3. **Загружает результат** в базу данных PostgreSQL.
4. **Поддерживает конкурентную загрузку** нескольких файлов (ExecutorService).

**Поля прайс-листа:**
- `article` — артикул (String)
- `name` — название товара (String)
- `category` — категория (String)
- `price` — цена (BigDecimal)
- `currency` — валюта (String) — USD, EUR, RUB
- `date` — дата (LocalDate)
- `supplier` — поставщик (String)

**Выходные данные:**
- Очищенные данные в БД.
- Статистика обработки (всего записей, очищено, загружено, ошибки).
- Файл с ошибками (dead-letter queue).

### 2.2. Архитектура приложения

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         Архитектура приложения                             │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                      Console Application                            │   │
│  │  ┌─────────────────────────────────────────────────────────────┐   │   │
│  │  │                      Main / CLI                              │   │   │
│  │  └─────────────────────────┬───────────────────────────────────┘   │   │
│  └────────────────────────────┼──────────────────────────────────────┘   │
│                               │                                           │
│  ┌────────────────────────────▼──────────────────────────────────────┐   │
│  │                    PriceListPipeline                              │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │   │
│  │  │   Extract   │──▶│  Transform  │──▶│    Load     │              │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘              │   │
│  └────────────────────────────┬──────────────────────────────────────┘   │
│                               │                                           │
│  ┌────────────────────────────▼──────────────────────────────────────┐   │
│  │                        Модули                                     │   │
│  ├───────────────────────────────────────────────────────────────────┤   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────┐│   │
│  │  │   Reader    │  │  Normalizer │  │   Enricher  │  │   DAO   ││   │
│  │  │ (CSV/JSON)  │  │ (Currency)  │  │ (Category)  │  │ (JDBC)  ││   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────┘│   │
│  └───────────────────────────────────────────────────────────────────┘   │
│                               │                                           │
│  ┌────────────────────────────▼──────────────────────────────────────┐   │
│  │                     Внешние зависимости                           │   │
│  ├───────────────────────────────────────────────────────────────────┤   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │   │
│  │  │  PostgreSQL │  │  API ЦБ РФ  │  │  CSV/JSON   │              │   │
│  │  │   Database  │  │  (Currency) │  │   Files     │              │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘              │   │
│  └───────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 2.3. Диаграмма классов (базовая)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         Диаграмма классов                                   │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                         PriceListPipeline                           │   │
│  │  ────────────────────────────────────────────────────────────────── │   │
│  │  - reader: DataSourceReader                                         │   │
│  │  - normalizer: CurrencyNormalizer                                   │   │
│  │  - enricher: CategoryEnricher                                       │   │
│  │  - loader: PriceListLoader                                          │   │
│  │  - dlq: DeadLetterQueue                                             │   │
│  │  ────────────────────────────────────────────────────────────────── │   │
│  │  + run(filePath: String, format: String): EtlResult                │   │
│  │  + runParallel(filePaths: List<String>): EtlResult                 │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                    │                                        │
│          ┌─────────────────────────┼─────────────────────────┐              │
│          │                         │                         │              │
│          ▼                         ▼                         ▼              │
│  ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐       │
│  │ DataSourceReader │     │ CurrencyNormalizer│   │ CategoryEnricher│       │
│  │ ──────────────── │     │ ──────────────── │     │ ──────────────── │       │
│  │ - strategy:      │     │ - apiClient:     │     │ - categoryMap:   │       │
│  │   ParserStrategy │     │   CurrencyApi    │     │   Map<String,    │       │
│  │ ──────────────── │     │ ──────────────── │     │   String>        │       │
│  │ + read():        │     │ + normalize():   │     │ ──────────────── │       │
│  │   List<PriceItem>│     │   PriceItem      │     │ + enrich():      │       │
│  └─────────────────┘     └─────────────────┘     │   PriceItem      │       │
│          │                                        └─────────────────┘       │
│          │                         ┌─────────────────────────────────────┐  │
│          ▼                         │       PriceItem (record)           │  │
│  ┌─────────────────┐               │  ────────────────────────────────── │  │
│  │ ParserStrategy  │               │  + article: String                  │  │
│  │ (interface)     │               │  + name: String                     │  │
│  │ ──────────────── │               │  + category: String                 │  │
│  │ + parse():      │               │  + price: BigDecimal                │  │
│  │   List<PriceItem>│               │  + currency: String                 │  │
│  └────────┬────────┘               │  + date: LocalDate                  │  │
│           │                        │  + supplier: String                 │  │
│  ┌────────┴────────┐               └─────────────────────────────────────┘  │
│  │                 │                                                        │
│  ▼                 ▼                                                        │
│  ┌──────────────┐ ┌──────────────┐     ┌─────────────────────────────────┐ │
│  │CsvParser    │ │JsonParser    │     │       PriceListDao (interface)   │ │
│  │Strategy     │ │Strategy      │     │  ────────────────────────────────── │ │
│  └──────────────┘ └──────────────┘     │  + save(PriceItem): PriceItem     │ │
│                                        │  + saveAll(List<PriceItem>): int  │ │
│                                        │  + findByArticle(String): ...     │ │
│                                        └─────────────────────────────────┘ │
│                                                    │                        │
│                                                    ▼                        │
│                                        ┌─────────────────────────────────┐ │
│                                        │     PriceListDaoImpl            │ │
│                                        │  ────────────────────────────────── │ │
│                                        │  + save(): PriceItem              │ │
│                                        │  + saveAll(): int                 │ │
│                                        │  + upsert(): PriceItem            │ │
│                                        └─────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 2.4. Распределение ролей в команде

| Роль | Ответственность |
|------|-----------------|
| **Team Lead / Архитектор** | Координация работы, архитектура, интеграция модулей, code review |
| **Разработчик 1** | DataSourceReader, парсеры CSV/JSON, нормализация валют |
| **Разработчик 2** | Обогащение категориями, валидация, DAO, загрузка в БД |
| **Разработчик 3** (если есть) | Многопоточность, DLQ, логирование, тестирование |

### 2.5. Зависимости проекта (Maven)

```xml
<dependencies>
    <!-- Логирование -->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>2.0.9</version>
    </dependency>
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.4.11</version>
    </dependency>

    <!-- Jackson для JSON -->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.16.1</version>
    </dependency>
    <dependency>
        <groupId>com.fasterxml.jackson.datatype</groupId>
        <artifactId>jackson-datatype-jsr310</artifactId>
        <version>2.16.1</version>
    </dependency>

    <!-- PostgreSQL JDBC -->
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
        <version>42.7.1</version>
    </dependency>

    <!-- HikariCP -->
    <dependency>
        <groupId>com.zaxxer</groupId>
        <artifactId>HikariCP</artifactId>
        <version>5.0.1</version>
    </dependency>

    <!-- Тестирование -->
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>5.10.1</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

---

## 3. Задание на паре

### Задача. Проектирование и начало реализации мини-проекта

### 3.1. Этап 1. Распределение ролей и планирование (15 мин)

1. **Сформировать команду** (2-3 человека).
2. **Распределить роли** (Team Lead, разработчики).
3. **Согласовать** интерфейсы между модулями.
4. **Создать** Git-репозиторий и дать доступ всем участникам.

### 3.2. Этап 2. Проектирование архитектуры (20 мин)

1. **Нарисовать диаграмму классов** (можно в Miro, Draw.io или просто текстом).
2. **Определить интерфейсы** для каждого модуля.
3. **Согласовать** структуру пакетов.
4. **Составить промпт** для генерации каркаса проекта.

### 3.3. Этап 3. Настройка проекта (15 мин)

1. **Инициализировать** Maven/Gradle проект.
2. **Настроить** структуру пакетов.
3. **Подключить** зависимости.
4. **Настроить** logback.xml.
5. **Создать** файлы `application.properties`.

### 3.4. Этап 4. Генерация каркаса с помощью ИИ (20 мин)

1. **Всей командой** составить промпт для генерации каркаса проекта.
2. **Проанализировать** сгенерированный код.
3. **Скорректировать** при необходимости.
4. **Сохранить** промпт в документацию.

### 3.5. Этап 5. Начало реализации (30 мин)

1. **Создать** модель данных `PriceItem` (record).
2. **Реализовать** базовые интерфейсы.
3. **Реализовать** `CsvParserStrategy` и `JsonParserStrategy`.
4. **Реализовать** `PriceListDao` (интерфейс).

### Применение ИИ-инструмента

**Промпт для генерации каркаса проекта:**

```markdown
**Роль:** Ты — senior Java-архитектор.

**Задача:** Спроектируй структуру проекта "Сервис загрузки и очистки прайс-листов".

**Требования:**
1. Консольное приложение на Java 17.
2. Чтение CSV и JSON файлов с прайс-листами.
3. ETL-пайплайн: Extract (чтение), Transform (нормализация валют, обогащение категориями), Load (запись в PostgreSQL).
4. Многопоточная загрузка нескольких файлов.
5. Dead-letter queue для ошибочных записей.

**Формат вывода:**
- Структура пакетов
- Список основных классов и интерфейсов
- Базовый код основных классов

**Ограничения:**
- Используй Maven.
- Используй record для модели данных.
- Используй паттерн «Стратегия» для парсеров.
- Добавь логирование.
```

---

## 4. Индивидуальные задания (30 вариантов)

Каждый вариант добавляет специфические требования к проекту:

### Вариант 1. Базовый

**Требования:** Стандартные (см. постановку задачи)

**Дополнительно:** API ЦБ РФ для курса валют

---

### Вариант 2. С API ЦБ РФ

**Требования:** Курс валют брать из API ЦБ РФ

**Дополнительно:** Кэширование курса на 1 час

---

### Вариант 3. С JSON-конфигурацией

**Требования:** Настройки приложения в JSON-файле

**Дополнительно:** Поддержка нескольких поставщиков

---

### Вариант 4. С веб-интерфейсом

**Требования:** REST API вместо консоли (Spring Boot)

**Дополнительно:** Swagger документация

---

### Вариант 5. С Excel поддержкой

**Требования:** Поддержка .xlsx файлов (Apache POI)

**Дополнительно:** Обработка больших файлов

---

### Вариант 6. С Kafka

**Требования:** Результат отправлять в Kafka

**Дополнительно:** Spring Cloud Stream

---

### Вариант 7. С Email-уведомлениями

**Требования:** Отправка отчёта по email

**Дополнительно:** JavaMail

---

### Вариант 8. С графическим интерфейсом

**Требования:** JavaFX интерфейс

**Дополнительно:** Визуализация статистики

---

### Вариант 9. С MongoDB

**Требования:** Загрузка в MongoDB вместо PostgreSQL

**Дополнительно:** Spring Data MongoDB

---

### Вариант 10. С Apache Camel

**Требования:** Использовать Apache Camel для маршрутов

**Дополнительно:** Маршруты в XML

---

### Вариант 11. С Redis кэшем

**Требования:** Кэширование справочников в Redis

**Дополнительно:** Spring Data Redis

---

### Вариант 12. С CSV с кавычками

**Требования:** Поддержка CSV с кавычками и запятыми внутри

**Дополнительно:** OpenCSV библиотека

---

### Вариант 13. С XML поддержкой

**Требования:** Поддержка XML файлов (Jackson XmlMapper)

**Дополнительно:** XSD валидация

---

### Вариант 14. С Apache Kafka Streams

**Требования:** Обработка через Kafka Streams

**Дополнительно:** Stateful обработка

---

### Вариант 15. С Elasticsearch

**Требования:** Загрузка в Elasticsearch

**Дополнительно:** REST Client

---

### Вариант 16. С Flyway миграциями

**Требования:** Управление схемой БД через Flyway

**Дополнительно:** Миграции для всех версий

---

### Вариант 17. С Docker Compose

**Требования:** Docker Compose для всех компонентов

**Дополнительно:** Multi-stage сборка

---

### Вариант 18. С мониторингом

**Требования:** Prometheus + Grafana метрики

**Дополнительно:** Micrometer

---

### Вариант 19. С OAuth2

**Требования:** Аутентификация через OAuth2

**Дополнительно:** Spring Security

---

### Вариант 20. С S3 хранилищем

**Требования:** Чтение файлов из S3

**Дополнительно:** AWS SDK

---

### Вариант 21. С Data Warehouse

**Требования:** Загрузка в DWH (ClickHouse)

**Дополнительно:** ClickHouse JDBC

---

### Вариант 22. С ML классификацией

**Требования:** Автоматическая категоризация товаров (ML)

**Дополнительно:** Weka / TensorFlow

---

### Вариант 23. С WebSocket

**Требования:** Веб-сокет для прогресса загрузки

**Дополнительно:** Spring WebSocket

---

### Вариант 24. С Apache Spark

**Требования:** Обработка через Spark

**Дополнительно:** Java API

---

### Вариант 25. С NoSQL (Cassandra)

**Требования:** Загрузка в Cassandra

**Дополнительно:** Datastax Driver

---

### Вариант 26. С Apache Airflow

**Требования:** Управление пайплайнами через Airflow

**Дополнительно:** Python API

---

### Вариант 27. С gRPC

**Требования:** gRPC API вместо REST

**Дополнительно:** Protobuf

---

### Вариант 28. С Apache Flink

**Требования:** Обработка через Flink

**Дополнительно:** Java API

---

### Вариант 29. С HashiCorp Vault

**Требования:** Секреты через Vault

**Дополнительно:** Spring Cloud Vault

---

### Вариант 30. С Kubernetes

**Требования:** Kubernetes деплоймент

**Дополнительно:** Helm charts

---

## 5. Методические указания

### 5.1. Структура проекта

```
price-list-service/
├── pom.xml
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/example/pricelist/
│   │   │       ├── Main.java
│   │   │       ├── config/
│   │   │       │   ├── DatabaseConfig.java
│   │   │       │   └── DataSourceProvider.java
│   │   │       ├── model/
│   │   │       │   └── PriceItem.java
│   │   │       ├── dao/
│   │   │       │   ├── PriceListDao.java
│   │   │       │   └── PriceListDaoImpl.java
│   │   │       ├── reader/
│   │   │       │   ├── DataSourceReader.java
│   │   │       │   ├── ParserStrategy.java
│   │   │       │   ├── CsvParserStrategy.java
│   │   │       │   └── JsonParserStrategy.java
│   │   │       ├── transformer/
│   │   │       │   ├── CurrencyNormalizer.java
│   │   │       │   ├── CategoryEnricher.java
│   │   │       │   └── PriceItemFilter.java
│   │   │       ├── loader/
│   │   │       │   └── PriceListLoader.java
│   │   │       ├── pipeline/
│   │   │       │   ├── PriceListPipeline.java
│   │   │       │   └── EtlResult.java
│   │   │       ├── exception/
│   │   │       │   └── PriceListException.java
│   │   │       └── util/
│   │   │           ├── DeadLetterQueue.java
│   │   │           └── CurrencyApiClient.java
│   │   └── resources/
│   │       ├── application.properties
│   │       ├── categories.csv
│   │       └── logback.xml
│   └── test/
│       └── java/
│           └── com/example/pricelist/
│               ├── dao/
│               └── pipeline/
└── README.md
```

### 5.2. Модель данных (PriceItem)

```java
package com.example.pricelist.model;

import com.fasterxml.jackson.annotation.JsonFormat;
import com.fasterxml.jackson.annotation.JsonProperty;
import java.math.BigDecimal;
import java.time.LocalDate;

/**
 * Иммутабельная модель данных прайс-листа
 */
public record PriceItem(
    @JsonProperty("article")
    String article,
    
    @JsonProperty("name")
    String name,
    
    @JsonProperty("category")
    String category,
    
    @JsonProperty("price")
    BigDecimal price,
    
    @JsonProperty("currency")
    String currency,
    
    @JsonProperty("date")
    @JsonFormat(pattern = "yyyy-MM-dd")
    LocalDate date,
    
    @JsonProperty("supplier")
    String supplier
) {
    /**
     * Валидация полей
     */
    public PriceItem {
        if (article == null || article.trim().isEmpty()) {
            throw new IllegalArgumentException("Артикул не может быть пустым");
        }
        if (name == null || name.trim().isEmpty()) {
            throw new IllegalArgumentException("Название не может быть пустым");
        }
        if (price == null || price.compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException("Цена должна быть положительной");
        }
        if (currency == null || currency.trim().isEmpty()) {
            throw new IllegalArgumentException("Валюта не может быть пустой");
        }
        if (date == null) {
            throw new IllegalArgumentException("Дата не может быть пустой");
        }
    }
}
```

### 5.3. Интерфейсы модулей

```java
// Reader
public interface ParserStrategy {
    List<PriceItem> parse(String filePath) throws IOException;
    String getFormat();
}

// Transformer
public interface Normalizer {
    PriceItem normalize(PriceItem item);
}

public interface Enricher {
    PriceItem enrich(PriceItem item);
}

public interface Filter {
    boolean isValid(PriceItem item);
}

// DAO
public interface PriceListDao {
    PriceItem save(PriceItem item) throws SQLException;
    int saveAll(List<PriceItem> items) throws SQLException;
    PriceItem upsert(PriceItem item) throws SQLException;
    Optional<PriceItem> findByArticle(String article) throws SQLException;
    List<PriceItem> findAll() throws SQLException;
}

// Pipeline
public interface Pipeline {
    EtlResult run(String filePath, String format) throws IOException, SQLException;
    EtlResult runParallel(List<String> filePaths) throws IOException, SQLException;
}
```

---

## 6. Контрольные вопросы

1. Опишите постановку задачи проекта.

2. Какие этапы ETL реализованы в проекте?

3. Какие роли распределены в команде?

4. Какие паттерны проектирования используются в проекте?

5. Как организована структура пакетов?

6. Какие зависимости подключены в проекте?

7. Как реализована поддержка CSV и JSON?

8. Как обеспечивается идемпотентность загрузки?

9. Как организована многопоточная загрузка?

10. Как обрабатываются ошибки в проекте?

11. Что такое dead-letter queue и как она реализована?

12. Как происходит нормализация валют?

13. Как происходит обогащение категориями?

14. Как организовано логирование?

15. Какие метрики собираются в процессе ETL?

---

## 7. Рекомендуемые источники

1. **Шилдт Г.** *Java. Базовый курс.* — М.: Вильямс. — Весь курс.

2. **Хорстманн К., Корнелл Г.** *Java. Библиотека профессионала. Том 1-2.* — М.: Вильямс.

3. **Fowler M.** *Patterns of Enterprise Application Architecture.* — Addison-Wesley.

4. **Kimball R., Caserta J.** *The Data Warehouse ETL Toolkit.* — Wiley.

5. **Maven Documentation.** — URL: https://maven.apache.org/guides/

6. **PostgreSQL Documentation.** — URL: https://www.postgresql.org/docs/

7. **Jackson Documentation.** — URL: https://github.com/FasterXML/jackson-docs

8. **HikariCP GitHub.** — URL: https://github.com/brettwooldridge/HikariCP

9. **SLF4J Documentation.** — URL: https://www.slf4j.org/docs.html

10. **Baeldung: Java Tutorials.** — URL: https://www.baeldung.com/
