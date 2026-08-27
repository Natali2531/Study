# Лабораторная работа №8. Файловый ввод-вывод: JSON и XML

## 1. Паспорт работы

| Параметр | Значение |
|----------|----------|
| Номер занятия | 8 из 17 |
| Блок | 3. Ввод-вывод |
| Продолжительность | 2 академических часа |
| Форма выполнения | Индивидуальная |
| ИИ-инструмент | YandexGPT / GigaChat / JetBrains AI Assistant |

### 1.1. Цель работы

Освоить работу с форматами JSON и XML в Java, научиться использовать библиотеку Jackson для сериализации и десериализации объектов, реализовать простой ETL-пайплайн для конвертации данных между форматами.

### 1.2. Задачи работы

1. Изучить библиотеку Jackson для работы с JSON.
2. Освоить основные аннотации Jackson: `@JsonProperty`, `@JsonIgnore`, `@JsonFormat`.
3. Научиться читать и записывать JSON-файлы.
4. Изучить работу с XML через Jackson XmlMapper.
5. Реализовать конвертер CSV → JSON.
6. Научиться обрабатывать даты в JSON/XML.
7. Сравнить производительность различных форматов.

### 1.3. Оснащение

- JDK 17 или выше;
- IntelliJ IDEA Community Edition;
- Git;
- Maven или Gradle;
- доступ к YandexGPT или GigaChat.

---

## 2. Теоретический конспект

### 2.1. Формат JSON

**JSON (JavaScript Object Notation)** — текстовый формат обмена данными, основанный на синтаксисе JavaScript.

**Структура JSON:**
- Объект: `{ "ключ": значение }`
- Массив: `[ значение1, значение2, ... ]`
- Значения: строка, число, объект, массив, true/false, null

**Пример JSON:**
```json
{
  "id": 1,
  "title": "Война и мир",
  "author": "Толстой Л.Н.",
  "year": 1869,
  "isbn": "978-5-17-118456-0",
  "publisher": "АСТ",
  "addedDate": "2026-01-15"
}
```

### 2.2. Формат XML

**XML (eXtensible Markup Language)** — расширяемый язык разметки.

**Структура XML:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<book>
    <id>1</id>
    <title>Война и мир</title>
    <author>Толстой Л.Н.</author>
    <year>1869</year>
    <isbn>978-5-17-118456-0</isbn>
    <publisher>АСТ</publisher>
    <addedDate>2026-01-15</addedDate>
</book>
```

### 2.3. Библиотека Jackson

**Jackson** — высокопроизводительная библиотека для работы с JSON и XML в Java.

**Основные модули:**
- `jackson-databind` — основной модуль для сериализации/десериализации.
- `jackson-dataformat-xml` — поддержка XML.
- `jackson-datatype-jsr310` — поддержка Java 8 Time API.

**Подключение зависимостей (Maven):**
```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.16.1</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
    <version>2.16.1</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.datatype</groupId>
    <artifactId>jackson-datatype-jsr310</artifactId>
    <version>2.16.1</version>
</dependency>
```

### 2.4. Основные аннотации Jackson

**@JsonProperty** — задаёт имя поля в JSON/XML:
```java
@JsonProperty("book_title")
private String title;
```

**@JsonIgnore** — игнорирует поле при сериализации/десериализации:
```java
@JsonIgnore
private String internalId;
```

**@JsonIgnoreProperties** — игнорирует неизвестные поля:
```java
@JsonIgnoreProperties(ignoreUnknown = true)
public class Book { ... }
```

**@JsonFormat** — задаёт формат даты:
```java
@JsonFormat(pattern = "yyyy-MM-dd")
private LocalDate addedDate;
```

**@JsonCreator** — указывает конструктор для десериализации:
```java
@JsonCreator
public Book(@JsonProperty("id") long id, ...) { ... }
```

**@JsonInclude** — управляет включением null-значений:
```java
@JsonInclude(JsonInclude.Include.NON_NULL)
public class Book { ... }
```

### 2.5. Сериализация и десериализация JSON

**Создание ObjectMapper:**
```java
ObjectMapper mapper = new ObjectMapper();
mapper.registerModule(new JavaTimeModule()); // для LocalDate
mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
```

**Сериализация (объект → JSON):**
```java
// Объект → JSON-строка
String json = mapper.writeValueAsString(book);

// Объект → JSON-файл
mapper.writeValue(new File("book.json"), book);

// Список → JSON-файл
mapper.writeValue(new File("books.json"), books);
```

**Десериализация (JSON → объект):**
```java
// JSON-строка → объект
Book book = mapper.readValue(jsonString, Book.class);

// JSON-файл → объект
Book book = mapper.readValue(new File("book.json"), Book.class);

// JSON-файл → список
List<Book> books = mapper.readValue(
    new File("books.json"), 
    new TypeReference<List<Book>>() {}
);
```

### 2.6. Работа с XML через Jackson

**Создание XmlMapper:**
```java
XmlMapper xmlMapper = new XmlMapper();
xmlMapper.registerModule(new JavaTimeModule());
```

**Сериализация в XML:**
```java
// Объект → XML
String xml = xmlMapper.writeValueAsString(book);

// Объект → XML-файл
xmlMapper.writeValue(new File("book.xml"), book);
```

**Десериализация из XML:**
```java
// XML-файл → объект
Book book = xmlMapper.readValue(new File("book.xml"), Book.class);
```

**Настройка XML:**
```java
XmlMapper xmlMapper = new XmlMapper();
// Включить красивое форматирование
xmlMapper.enable(SerializationFeature.INDENT_OUTPUT);
// Игнорировать корневой элемент
xmlMapper.disable(SerializationFeature.WRAP_ROOT_VALUE);
```

### 2.7. Красивое форматирование

```java
ObjectMapper mapper = new ObjectMapper();
mapper.enable(SerializationFeature.INDENT_OUTPUT);
mapper.writeValue(new File("books.json"), books);
```

Результат:
```json
[
  {
    "id": 1,
    "title": "Война и мир",
    "author": "Толстой Л.Н.",
    "year": 1869,
    "isbn": "978-5-17-118456-0",
    "publisher": "АСТ",
    "addedDate": "2026-01-15"
  },
  ...
]
```

### 2.8. Обработка ошибок

```java
try {
    Book book = mapper.readValue(file, Book.class);
} catch (JsonParseException e) {
    logger.error("Ошибка парсинга JSON: {}", e.getMessage());
} catch (JsonMappingException e) {
    logger.error("Ошибка маппинга JSON: {}", e.getMessage());
} catch (IOException e) {
    logger.error("Ошибка ввода-вывода: {}", e.getMessage());
}
```

### 2.9. Jackson + Record (Java 14+)

```java
public record Book(
    @JsonProperty("id") long id,
    @JsonProperty("title") String title,
    @JsonProperty("author") String author,
    @JsonProperty("year") int year,
    @JsonProperty("isbn") String isbn,
    @JsonProperty("publisher") String publisher,
    @JsonFormat(pattern = "yyyy-MM-dd")
    @JsonProperty("addedDate") LocalDate addedDate
) {}
```

---

## 3. Задание на паре

### Задача. Работа с JSON и XML в системе управления книгами

1. **Подключить Jackson:**
   - Добавить зависимости в Maven/Gradle.
   - Разобрать основные аннотации.

2. **Создать POJO-класс `Book` с аннотациями:**
   - `@JsonProperty` для всех полей.
   - `@JsonIgnore` для одного поля (например, `internalId`).
   - `@JsonFormat` для поля `addedDate`.

3. **Прочитать JSON-файл:**
   - Чтение массива объектов в `List<Book>`.
   - Вывод статистики.

4. **Сериализовать список в JSON:**
   - Запись в файл с красивым форматированием.

5. **Реализовать конвертер CSV → JSON:**
   - На вход — CSV-файл (из лабораторной №7).
   - На выход — JSON-файл.
   - Это первый простой ETL-пайплайн.

6. **Реализовать чтение из XML:**
   - Использовать Jackson XmlMapper.
   - Прочитать тот же набор данных из XML.

7. **Добавить аннотации для дат:**
   - Формат `yyyy-MM-dd`.
   - Поле `addedDate` с текущей датой.

**Пример выполнения:**
```
=== Работа с JSON ===
Прочитано книг из JSON: 20
Записано книг в JSON: 20

=== Конвертер CSV → JSON ===
Прочитано книг из CSV: 48
Записано книг в JSON: 48
Файл: books.json

=== Работа с XML ===
Прочитано книг из XML: 20
Записано книг в XML: 20
```

### Применение ИИ-инструмента

**Промпт для YandexGPT:**
```
Создай POJO-класс Book на Java 17 для Jackson со следующими полями:
- id (long)
- title (String)
- author (String)
- year (int)
- isbn (String)
- publisher (String)
- addedDate (LocalDate)

Требования:
1. Добавить аннотацию @JsonProperty для каждого поля.
2. Добавить поле internalId с @JsonIgnore.
3. Добавить @JsonFormat для addedDate с паттерном "yyyy-MM-dd".
4. Добавить @JsonIgnoreProperties(ignoreUnknown = true).
5. Добавить конструктор с @JsonCreator.
```

**Анализ результата:**
- Проверить наличие всех аннотаций.
- Проверить корректность типов данных.
- Проверить наличие конструктора с @JsonCreator.
- Проверить обработку дат.

---

## 4. Индивидуальные задания (30 вариантов)

Каждый вариант содержит:
- POJO-класс для JSON/XML;
- Набор аннотаций Jackson;
- Дополнительные требования по форматированию.

---

### Вариант 1. Книги (Book)

**Поля:** `id`, `title`, `author`, `year`, `isbn`, `publisher`, `addedDate`

**Аннотации:**
- `@JsonProperty` для всех полей
- `@JsonFormat` для `addedDate` (yyyy-MM-dd)
- `@JsonIgnore` для `internalId`

**Дополнительно:** `@JsonInclude(Include.NON_NULL)`

---

### Вариант 2. Сотрудники (Employee)

**Поля:** `id`, `firstName`, `lastName`, `position`, `salary`, `department`, `hireDate`

**Аннотации:**
- `@JsonProperty` для всех полей
- `@JsonFormat` для `hireDate` (yyyy-MM-dd)
- `@JsonIgnore` для `password`

**Дополнительно:** `@JsonIgnoreProperties(ignoreUnknown = true)`

---

### Вариант 3. Товары (Product)

**Поля:** `id`, `name`, `category`, `price`, `quantity`, `supplier`, `lastUpdated`

**Аннотации:**
- `@JsonProperty` для всех полей
- `@JsonFormat` для `lastUpdated` (yyyy-MM-dd HH:mm:ss)
- `@JsonIgnore` для `internalCode`

---

### Вариант 4. Заказы (Order)

**Поля:** `id`, `customerName`, `totalAmount`, `status`, `createdAt`, `items`, `deliveryDate`

**Аннотации:**
- `@JsonProperty` для всех полей
- `@JsonFormat` для `createdAt` (yyyy-MM-dd'T'HH:mm:ss)
- `@JsonIgnore` для `internalStatus`

---

### Вариант 5. Пользователи (User)

**Поля:** `id`, `username`, `email`, `password`, `role`, `createdAt`, `lastLogin`

**Аннотации:**
- `@JsonProperty` для всех полей
- `@JsonFormat` для дат (yyyy-MM-dd)
- `@JsonIgnore` для `password`

---

### Вариант 6. Автомобили (Car)

**Поля:** `id`, `brand`, `model`, `year`, `price`, `vin`, `manufactureDate`

**Аннотации:**
- `@JsonProperty` для всех полей
- `@JsonFormat` для `manufactureDate` (yyyy-MM-dd)
- `@JsonIgnore` для `hiddenFeatures`

---

### Вариант 7. Студенты (Student)

**Поля:** `id`, `firstName`, `lastName`, `group`, `gpa`, `birthDate`, `enrollmentDate`

**Аннотации:**
- `@JsonProperty` для всех полей
- `@JsonFormat` для дат (yyyy-MM-dd)
- `@JsonIgnore` для `scholarship`

---

### Вариант 8. Счета (Account)

**Поля:** `id`, `accountNumber`, `ownerName`, `balance`, `currency`, `openedDate`, `lastTransaction`

**Аннотации:**
- `@JsonProperty` для всех полей
- `@JsonFormat` для дат (yyyy-MM-dd)
- `@JsonIgnore` для `pinCode`

---

### Вариант 9. Фильмы (Movie)

**Поля:** `id`, `title`, `director`, `year`, `rating`, `genre`, `releaseDate`

**Аннотации:**
- `@JsonProperty` для всех полей
- `@JsonFormat` для `releaseDate` (yyyy-MM-dd)
- `@JsonIgnore` для `budget`

---

### Вариант 10. Рестораны (Restaurant)

**Поля:** `id`, `name`, `address`, `phone`, `rating`, `cuisine`, `openingDate`

**Аннотации:**
- `@JsonProperty` для всех полей
- `@JsonFormat` для `openingDate` (yyyy-MM-dd)
- `@JsonIgnore` для `ownerInfo`

---

### Вариант 11. Транзакции (Transaction)

**Поля:** `id`, `amount`, `type`, `date`, `description`, `category`, `processedDate`

**Аннотации:**
- `@JsonProperty` для всех полей
- `@JsonFormat` для дат (yyyy-MM-dd'T'HH:mm:ss)
- `@JsonIgnore` для `confirmationCode`

---

### Вариант 12. Клиенты (Customer)

**Поля:** `id`, `firstName`, `lastName`, `email`, `phone`, `birthDate`, `registrationDate`

**Аннотации:**
- `@JsonProperty` для всех полей
- `@JsonFormat` для дат (yyyy-MM-dd)
- `@JsonIgnore` для `creditCard`

---

### Вариант 13. Договоры (Contract)

**Поля:** `id`, `number`, `client`, `signDate`, `amount`, `status`, `expiryDate`

**Аннотации:**
- `@JsonProperty` для всех полей
- `@JsonFormat` для дат (yyyy-MM-dd)
- `@JsonIgnore` для `confidential`

---

### Вариант 14. Отели (Hotel)

**Поля:** `id`, `name`, `city`, `stars`, `roomsCount`, `rating`, `establishedDate`

**Аннотации:**
- `@JsonProperty` для всех полей
- `@JsonFormat` для дат (yyyy-MM-dd)
- `@JsonIgnore` для `managementInfo`

---

### Вариант 15. Спортсмены (Athlete)

**Поля:** `id`, `firstName`, `lastName`, `sport`, `age`, `medals`, `birthDate`

**Аннотации:**
- `@JsonProperty` для всех полей
- `@JsonFormat` для `birthDate` (yyyy-MM-dd)
- `@JsonIgnore` для `dopingTest`

---

### Вариант 16. Статьи (Article)

**Поля:** `id`, `title`, `content`, `author`, `publishedDate`, `views`, `tags`

**Аннотации:**
- `@JsonProperty` для всех полей
- `@JsonFormat` для `publishedDate` (yyyy-MM-dd)
- `@JsonIgnore` для `internalId`

---

### Вариант 17. Билеты (Ticket)

**Поля:** `id`, `eventName`, `venue`, `date`, `price`, `quantity`, `purchaseDate`

**Аннотации:**
- `@JsonProperty` для всех полей
- `@JsonFormat` для дат (yyyy-MM-dd'T'HH:mm)
- `@JsonIgnore` для `barcode`

---

### Вариант 18. Курсы (Course)

**Поля:** `id`, `title`, `instructor`, `duration`, `price`, `startDate`, `endDate`

**Аннотации:**
- `@JsonProperty` для всех полей
- `@JsonFormat` для дат (yyyy-MM-dd)
- `@JsonIgnore` для `syllabus`

---

### Вариант 19. Инвентарь (Item)

**Поля:** `id`, `name`, `category`, `quantity`, `location`, `status`, `lastChecked`

**Аннотации:**
- `@JsonProperty` для всех полей
- `@JsonFormat` для `lastChecked` (yyyy-MM-dd)
- `@JsonIgnore` для `serialNumber`

---

### Вариант 20. Заявки (Request)

**Поля:** `id`, `clientName`, `description`, `priority`, `status`, `createdAt`, `resolvedAt`

**Аннотации:**
- `@JsonProperty` для всех полей
- `@JsonFormat` для дат (yyyy-MM-dd'T'HH:mm:ss)
- `@JsonIgnore` для `internalNotes`

---

### Вариант 21. Платёжные поручения (PaymentOrder)

**Поля:** `id`, `number`, `payer`, `recipient`, `amount`, `date`, `executionDate`

**Аннотации:**
- `@JsonProperty` для всех полей
- `@JsonFormat` для дат (yyyy-MM-dd)
- `@JsonIgnore` для `authorization`

---

### Вариант 22. Публикации (Publication)

**Поля:** `id`, `title`, `author`, `journal`, `year`, `doi`, `submittedDate`

**Аннотации:**
- `@JsonProperty` для всех полей
- `@JsonFormat` для дат (yyyy-MM-dd)
- `@JsonIgnore` для `peerReview`

---

### Вариант 23. Ноутбуки (Laptop)

**Поля:** `id`, `brand`, `model`, `processor`, `ram`, `price`, `releaseDate`

**Аннотации:**
- `@JsonProperty` для всех полей
- `@JsonFormat` для `releaseDate` (yyyy-MM-dd)
- `@JsonIgnore` для `warrantyCode`

---

### Вариант 24. Врачи (Doctor)

**Поля:** `id`, `firstName`, `lastName`, `specialization`, `experience`, `rating`, `certifiedDate`

**Аннотации:**
- `@JsonProperty` для всех полей
- `@JsonFormat` для дат (yyyy-MM-dd)
- `@JsonIgnore` для `licenseNumber`

---

### Вариант 25. Сертификаты (Certificate)

**Поля:** `id`, `number`, `holderName`, `issuedDate`, `expiryDate`, `type`, `issuer`

**Аннотации:**
- `@JsonProperty` для всех полей
- `@JsonFormat` для дат (yyyy-MM-dd)
- `@JsonIgnore` для `verificationCode`

---

### Вариант 26. Магазины (Shop)

**Поля:** `id`, `name`, `address`, `phone`, `openingHours`, `type`, `foundedDate`

**Аннотации:**
- `@JsonProperty` для всех полей
- `@JsonFormat` для дат (yyyy-MM-dd)
- `@JsonIgnore` для `taxNumber`

---

### Вариант 27. Здания (Building)

**Поля:** `id`, `address`, `floors`, `area`, `type`, `yearBuilt`, `renovationDate`

**Аннотации:**
- `@JsonProperty` для всех полей
- `@JsonFormat` для дат (yyyy-MM-dd)
- `@JsonIgnore` для `architect`

---

### Вариант 28. Лекции (Lecture)

**Поля:** `id`, `topic`, `speaker`, `date`, `duration`, `attendees`, `recordedDate`

**Аннотации:**
- `@JsonProperty` для всех полей
- `@JsonFormat` для дат (yyyy-MM-dd'T'HH:mm)
- `@JsonIgnore` для `internalId`

---

### Вариант 29. Отзывы (Review)

**Поля:** `id`, `userId`, `productId`, `rating`, `comment`, `createdAt`, `updatedAt`

**Аннотации:**
- `@JsonProperty` для всех полей
- `@JsonFormat` для дат (yyyy-MM-dd'T'HH:mm:ss)
- `@JsonIgnore` для `reported`

---

### Вариант 30. Чеки (Receipt)

**Поля:** `id`, `number`, `store`, `totalAmount`, `date`, `items`, `cashier`

**Аннотации:**
- `@JsonProperty` для всех полей
- `@JsonFormat` для `date` (yyyy-MM-dd'T'HH:mm:ss)
- `@JsonIgnore` для `qrCode`

---

## 5. Методические указания

### 5.1. Структура проекта

```
src/
├── main/
│   ├── java/
│   │   ├── exceptions/
│   │   │   └── FileProcessingException.java
│   │   ├── model/
│   │   │   └── Book.java
│   │   ├── parser/
│   │   │   ├── CsvParser.java
│   │   │   ├── JsonParser.java
│   │   │   └── XmlParser.java
│   │   ├── converter/
│   │   │   └── CsvToJsonConverter.java
│   │   └── Main.java
│   └── resources/
│       ├── books.csv
│       ├── books.json
│       └── books.xml
└── test/
    └── java/
        └── parser/
            └── JsonParserTest.java
```

### 5.2. Шаблон класса Book с аннотациями

```java
package model;

import com.fasterxml.jackson.annotation.*;
import com.fasterxml.jackson.databind.annotation.JsonDeserialize;
import com.fasterxml.jackson.databind.annotation.JsonSerialize;
import com.fasterxml.jackson.datatype.jsr310.deser.LocalDateDeserializer;
import com.fasterxml.jackson.datatype.jsr310.ser.LocalDateSerializer;

import java.time.LocalDate;
import java.util.Objects;

@JsonIgnoreProperties(ignoreUnknown = true)
@JsonInclude(JsonInclude.Include.NON_NULL)
public class Book {
    
    @JsonProperty("id")
    private long id;
    
    @JsonProperty("title")
    private String title;
    
    @JsonProperty("author")
    private String author;
    
    @JsonProperty("year")
    private int year;
    
    @JsonProperty("isbn")
    private String isbn;
    
    @JsonProperty("publisher")
    private String publisher;
    
    @JsonIgnore
    private String internalId;
    
    @JsonProperty("addedDate")
    @JsonFormat(pattern = "yyyy-MM-dd")
    @JsonSerialize(using = LocalDateSerializer.class)
    @JsonDeserialize(using = LocalDateDeserializer.class)
    private LocalDate addedDate;
    
    public Book() {
        this.addedDate = LocalDate.now();
    }
    
    @JsonCreator
    public Book(
            @JsonProperty("id") long id,
            @JsonProperty("title") String title,
            @JsonProperty("author") String author,
            @JsonProperty("year") int year,
            @JsonProperty("isbn") String isbn,
            @JsonProperty("publisher") String publisher,
            @JsonProperty("addedDate") LocalDate addedDate) {
        this.id = id;
        this.title = title;
        this.author = author;
        this.year = year;
        this.isbn = isbn;
        this.publisher = publisher;
        this.addedDate = addedDate != null ? addedDate : LocalDate.now();
    }
    
    // Геттеры и сеттеры
    // toString(), equals(), hashCode()
}
```

### 5.3. Шаблон JSON-парсера

```java
package parser;

import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializationFeature;
import com.fasterxml.jackson.datatype.jsr310.JavaTimeModule;
import model.Book;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.File;
import java.io.IOException;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.List;

public class JsonParser {
    private static final Logger logger = LoggerFactory.getLogger(JsonParser.class);
    private static final ObjectMapper mapper = createObjectMapper();
    
    private static ObjectMapper createObjectMapper() {
        ObjectMapper mapper = new ObjectMapper();
        mapper.registerModule(new JavaTimeModule());
        mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
        mapper.enable(SerializationFeature.INDENT_OUTPUT);
        return mapper;
    }
    
    /**
     * Чтение списка книг из JSON-файла
     */
    public static List<Book> readBooksFromJson(String filePath) throws IOException {
        logger.info("Чтение JSON-файла: {}", filePath);
        File file = new File(filePath);
        
        if (!file.exists()) {
            throw new IOException("Файл не найден: " + filePath);
        }
        
        List<Book> books = mapper.readValue(file, new TypeReference<List<Book>>() {});
        logger.info("Прочитано {} книг из JSON", books.size());
        return books;
    }
    
    /**
     * Запись списка книг в JSON-файл
     */
    public static void writeBooksToJson(List<Book> books, String filePath) throws IOException {
        logger.info("Запись JSON-файла: {}", filePath);
        
        // Создание директорий при необходимости
        Path path = Paths.get(filePath);
        if (path.getParent() != null) {
            path.getParent().toFile().mkdirs();
        }
        
        mapper.writeValue(new File(filePath), books);
        logger.info("Записано {} книг в JSON", books.size());
    }
    
    /**
     * Чтение одной книги из JSON
     */
    public static Book readBookFromJson(String filePath) throws IOException {
        return mapper.readValue(new File(filePath), Book.class);
    }
    
    /**
     * Запись одной книги в JSON
     */
    public static void writeBookToJson(Book book, String filePath) throws IOException {
        mapper.writeValue(new File(filePath), book);
    }
    
    /**
     * Преобразование JSON-строки в объект
     */
    public static Book parseJsonString(String json) throws IOException {
        return mapper.readValue(json, Book.class);
    }
    
    /**
     * Преобразование объекта в JSON-строку
     */
    public static String toJsonString(Book book) throws IOException {
        return mapper.writeValueAsString(book);
    }
}
```

### 5.4. Шаблон XML-парсера

```java
package parser;

import com.fasterxml.jackson.databind.SerializationFeature;
import com.fasterxml.jackson.dataformat.xml.XmlMapper;
import com.fasterxml.jackson.datatype.jsr310.JavaTimeModule;
import model.Book;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.File;
import java.io.IOException;
import java.util.List;

public class XmlParser {
    private static final Logger logger = LoggerFactory.getLogger(XmlParser.class);
    private static final XmlMapper xmlMapper = createXmlMapper();
    
    private static XmlMapper createXmlMapper() {
        XmlMapper xmlMapper = new XmlMapper();
        xmlMapper.registerModule(new JavaTimeModule());
        xmlMapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
        xmlMapper.enable(SerializationFeature.INDENT_OUTPUT);
        return xmlMapper;
    }
    
    /**
     * Чтение списка книг из XML-файла
     */
    public static List<Book> readBooksFromXml(String filePath) throws IOException {
        logger.info("Чтение XML-файла: {}", filePath);
        File file = new File(filePath);
        
        if (!file.exists()) {
            throw new IOException("Файл не найден: " + filePath);
        }
        
        BooksWrapper wrapper = xmlMapper.readValue(file, BooksWrapper.class);
        logger.info("Прочитано {} книг из XML", wrapper.getBooks().size());
        return wrapper.getBooks();
    }
    
    /**
     * Запись списка книг в XML-файл
     */
    public static void writeBooksToXml(List<Book> books, String filePath) throws IOException {
        logger.info("Запись XML-файла: {}", filePath);
        
        BooksWrapper wrapper = new BooksWrapper(books);
        xmlMapper.writeValue(new File(filePath), wrapper);
        logger.info("Записано {} книг в XML", books.size());
    }
    
    /**
     * Обёртка для списка книг (нужна для XML)
     */
    @JsonIgnoreProperties(ignoreUnknown = true)
    public static class BooksWrapper {
        private List<Book> books;
        
        public BooksWrapper() {}
        
        public BooksWrapper(List<Book> books) {
            this.books = books;
        }
        
        @JsonProperty("book")
        public List<Book> getBooks() { return books; }
        public void setBooks(List<Book> books) { this.books = books; }
    }
}
```

### 5.5. Шаблон конвертера CSV → JSON

```java
package converter;

import model.Book;
import parser.CsvParser;
import parser.JsonParser;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.IOException;
import java.util.List;

public class CsvToJsonConverter {
    private static final Logger logger = LoggerFactory.getLogger(CsvToJsonConverter.class);
    
    /**
     * Конвертация CSV → JSON
     */
    public static void convert(String csvFilePath, String jsonFilePath) throws IOException {
        logger.info("Начало конвертации CSV → JSON");
        logger.info("Входной файл: {}", csvFilePath);
        logger.info("Выходной файл: {}", jsonFilePath);
        
        // Extract: чтение из CSV
        long startTime = System.currentTimeMillis();
        List<Book> books = CsvParser.parseFile(csvFilePath);
        
        // Преобразование: добавление даты
        books.forEach(book -> book.setAddedDate(java.time.LocalDate.now()));
        
        // Load: запись в JSON
        JsonParser.writeBooksToJson(books, jsonFilePath);
        
        long duration = System.currentTimeMillis() - startTime;
        logger.info("Конвертация завершена за {} мс", duration);
        logger.info("Обработано {} книг", books.size());
    }
}
```

### 5.6. Основной класс

```java
import converter.CsvToJsonConverter;
import model.Book;
import parser.JsonParser;
import parser.XmlParser;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.IOException;
import java.util.List;

public class Main {
    private static final Logger logger = LoggerFactory.getLogger(Main.class);
    
    private static final String CSV_FILE = "src/main/resources/books.csv";
    private static final String JSON_FILE = "src/main/resources/books.json";
    private static final String XML_FILE = "src/main/resources/books.xml";
    
    public static void main(String[] args) {
        System.out.println("=== Работа с JSON и XML ===\n");
        
        try {
            // 1. Конвертация CSV → JSON
            System.out.println("1. Конвертация CSV → JSON:");
            CsvToJsonConverter.convert(CSV_FILE, JSON_FILE);
            System.out.println("✓ Конвертация завершена\n");
            
            // 2. Чтение из JSON
            System.out.println("2. Чтение из JSON:");
            List<Book> booksFromJson = JsonParser.readBooksFromJson(JSON_FILE);
            System.out.println("Прочитано книг: " + booksFromJson.size());
            booksFromJson.stream().limit(3).forEach(System.out::println);
            System.out.println();
            
            // 3. Запись в XML
            System.out.println("3. Запись в XML:");
            XmlParser.writeBooksToXml(booksFromJson, XML_FILE);
            System.out.println("Записано книг: " + booksFromJson.size() + "\n");
            
            // 4. Чтение из XML
            System.out.println("4. Чтение из XML:");
            List<Book> booksFromXml = XmlParser.readBooksFromXml(XML_FILE);
            System.out.println("Прочитано книг: " + booksFromXml.size());
            booksFromXml.stream().limit(3).forEach(System.out::println);
            
        } catch (IOException e) {
            logger.error("Ошибка при работе с файлами", e);
            System.err.println("Ошибка: " + e.getMessage());
        }
    }
}
```

### 5.7. Настройка Jackson для дат

```java
// Вариант 1: Глобальная настройка ObjectMapper
ObjectMapper mapper = new ObjectMapper();
mapper.registerModule(new JavaTimeModule());
mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
mapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd"));

// Вариант 2: Аннотация @JsonFormat
@JsonFormat(pattern = "yyyy-MM-dd")
private LocalDate addedDate;

// Вариант 3: Кастомный сериализатор
public class LocalDateSerializer extends StdSerializer<LocalDate> {
    public LocalDateSerializer() {
        super(LocalDate.class);
    }
    
    @Override
    public void serialize(LocalDate date, JsonGenerator gen, SerializerProvider provider) 
            throws IOException {
        gen.writeString(date.format(DateTimeFormatter.ISO_LOCAL_DATE));
    }
}
```

---

## 6. Контрольные вопросы

1. Что такое JSON? Опишите его структуру.

2. Что такое XML? Опишите его структуру.

3. Какие преимущества имеет JSON перед XML?

4. Какие преимущества имеет XML перед JSON?

5. Для чего используется библиотека Jackson?

6. Что делает аннотация `@JsonProperty`?

7. Что делает аннотация `@JsonIgnore`?

8. Что делает аннотация `@JsonFormat`?

9. Как обрабатываются даты в Jackson?

10. Как настроить красивое форматирование JSON?

11. Как прочитать массив объектов из JSON?

12. Как использовать Jackson для работы с XML?

13. В чём отличие ObjectMapper от XmlMapper?

14. Что такое сериализация и десериализация?

15. Как обрабатывать неизвестные поля в JSON?

16. Как реализовать конвертер CSV → JSON?

17. Как добавить поддержку Java 8 Time API в Jackson?

---

## 7. Пример выполнения (Вариант 1)

### 7.1. JSON-файл (`books.json`)

```json
[
  {
    "id": 1,
    "title": "Война и мир",
    "author": "Толстой Л.Н.",
    "year": 1869,
    "isbn": "978-5-17-118456-0",
    "publisher": "АСТ",
    "addedDate": "2026-01-15"
  },
  {
    "id": 2,
    "title": "Преступление и наказание",
    "author": "Достоевский Ф.М.",
    "year": 1866,
    "isbn": "978-5-17-118457-7",
    "publisher": "Эксмо",
    "addedDate": "2026-01-15"
  }
]
```

### 7.2. XML-файл (`books.xml`)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<BooksWrapper>
  <book>
    <id>1</id>
    <title>Война и мир</title>
    <author>Толстой Л.Н.</author>
    <year>1869</year>
    <isbn>978-5-17-118456-0</isbn>
    <publisher>АСТ</publisher>
    <addedDate>2026-01-15</addedDate>
  </book>
  <book>
    <id>2</id>
    <title>Преступление и наказание</title>
    <author>Достоевский Ф.М.</author>
    <year>1866</year>
    <isbn>978-5-17-118457-7</isbn>
    <publisher>Эксмо</publisher>
    <addedDate>2026-01-15</addedDate>
  </book>
</BooksWrapper>
```

### 7.3. Ожидаемый вывод

```
=== Работа с JSON и XML ===

1. Конвертация CSV → JSON:
2026-01-15 10:30:15 [main] INFO  CsvToJsonConverter - Начало конвертации CSV → JSON
2026-01-15 10:30:15 [main] INFO  CsvToJsonConverter - Входной файл: src/main/resources/books.csv
2026-01-15 10:30:15 [main] INFO  CsvToJsonConverter - Выходной файл: src/main/resources/books.json
2026-01-15 10:30:16 [main] INFO  CsvParser - Чтение CSV-файла: src/main/resources/books.csv
=== Статистика обработки ===
Всего строк: 21
Успешно прочитано: 20
Пропущено с ошибками: 1
Дубликатов по ISBN: 0
2026-01-15 10:30:16 [main] INFO  JsonParser - Запись JSON-файла: src/main/resources/books.json
2026-01-15 10:30:16 [main] INFO  JsonParser - Записано 20 книг в JSON
2026-01-15 10:30:16 [main] INFO  CsvToJsonConverter - Конвертация завершена за 125 мс
2026-01-15 10:30:16 [main] INFO  CsvToJsonConverter - Обработано 20 книг
✓ Конвертация завершена

2. Чтение из JSON:
2026-01-15 10:30:16 [main] INFO  JsonParser - Чтение JSON-файла: src/main/resources/books.json
2026-01-15 10:30:16 [main] INFO  JsonParser - Прочитано 20 книг из JSON
Прочитано книг: 20
Book{id=1, title='Война и мир', author='Толстой Л.Н.', year=1869, isbn='978-5-17-118456-0', publisher='АСТ', addedDate=2026-01-15}
Book{id=2, title='Преступление и наказание', author='Достоевский Ф.М.', year=1866, isbn='978-5-17-118457-7', publisher='Эксмо', addedDate=2026-01-15}
Book{id=3, title='Java: полное руководство', author='Шилдт Г.', year=2019, isbn='978-5-8459-1959-3', publisher='Вильямс', addedDate=2026-01-15}

3. Запись в XML:
2026-01-15 10:30:16 [main] INFO  XmlParser - Запись XML-файла: src/main/resources/books.xml
2026-01-15 10:30:16 [main] INFO  XmlParser - Записано 20 книг в XML
Записано книг: 20

4. Чтение из XML:
2026-01-15 10:30:16 [main] INFO  XmlParser - Чтение XML-файла: src/main/resources/books.xml
2026-01-15 10:30:16 [main] INFO  XmlParser - Прочитано 20 книг из XML
Прочитано книг: 20
Book{id=1, title='Война и мир', author='Толстой Л.Н.', year=1869, isbn='978-5-17-118456-0', publisher='АСТ', addedDate=2026-01-15}
Book{id=2, title='Преступление и наказание', author='Достоевский Ф.М.', year=1866, isbn='978-5-17-118457-7', publisher='Эксмо', addedDate=2026-01-15}
Book{id=3, title='Java: полное руководство', author='Шилдт Г.', year=2019, isbn='978-5-8459-1959-3', publisher='Вильямс', addedDate=2026-01-15}
```

---

## 8. Рекомендуемые источники

1. **Jackson Documentation.** — URL: https://github.com/FasterXML/jackson-docs

2. **Jackson Annotations Guide.** — URL: https://github.com/FasterXML/jackson-annotations

3. **JSON (JavaScript Object Notation).** — URL: https://www.json.org/

4. **XML (eXtensible Markup Language).** — URL: https://www.w3.org/XML/

5. **Baeldung: Jackson Tutorial.** — URL: https://www.baeldung.com/jackson

6. **Шилдт Г.** *Java. Базовый курс.* — М.: Вильямс. — Глава 15 (Ввод-вывод).

7. **Хорстманн К., Корнелл Г.** *Java. Библиотека профессионала. Том 2.* — М.: Вильямс. — Глава 2 (Ввод-вывод).
