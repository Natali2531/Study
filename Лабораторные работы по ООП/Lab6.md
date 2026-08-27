# Лабораторная работа №6. Исключения и логирование

## 1. Паспорт работы

| Параметр | Значение |
|----------|----------|
| Номер занятия | 6 из 17 |
| Блок | 2. Коллекции, обобщения и исключения |
| Продолжительность | 2 академических часа |
| Форма выполнения | Индивидуальная |
| ИИ-инструмент | YandexGPT / GigaChat / JetBrains AI Assistant |

### 1.1. Цель работы

Освоить механизм обработки исключений в Java, научиться создавать собственные исключения, применять логирование для отслеживания работы приложения и диагностики ошибок.

### 1.2. Задачи работы

1. Изучить иерархию исключений в Java (Throwable, Exception, RuntimeException, Error).
2. Освоить различия между checked и unchecked исключениями.
3. Научиться использовать конструкцию try-catch-finally и try-with-resources.
4. Освоить создание собственных классов исключений.
5. Изучить подключение и настройку библиотеки логирования SLF4J + Logback.
6. Научиться расставлять логирование разных уровней (INFO, WARN, ERROR, DEBUG).
7. Развить навыки обработки ошибок в приложениях.

### 1.3. Оснащение

- JDK 17 или выше;
- IntelliJ IDEA Community Edition;
- Git;
- Maven или Gradle;
- доступ к YandexGPT или GigaChat.

---

## 2. Теоретический конспект

### 2.1. Иерархия исключений

В Java все исключения наследуются от класса `Throwable`. Иерархия выглядит следующим образом:

```
Throwable
├── Error (непредсказуемые ошибки JVM)
│   ├── OutOfMemoryError
│   ├── StackOverflowError
│   └── ...
├── Exception (обрабатываемые исключения)
│   ├── IOException
│   ├── SQLException
│   ├── ClassNotFoundException
│   └── RuntimeException (исключения времени выполнения)
│       ├── NullPointerException
│       ├── IndexOutOfBoundsException
│       ├── ArithmeticException
│       └── ...
```

**Error** — критические ошибки, которые не должны обрабатываться в приложении (например, `OutOfMemoryError`).

**Exception** — исключения, которые должны обрабатываться.

**RuntimeException** — исключения времени выполнения, которые могут возникать из-за ошибок программиста.

### 2.2. Checked vs Unchecked исключения

| Характеристика | Checked | Unchecked |
|----------------|---------|-----------|
| Проверка компилятором | Да — должны быть обработаны или проброшены | Нет |
| Наследование | `Exception` (не `RuntimeException`) | `RuntimeException` |
| Примеры | `IOException`, `SQLException` | `NullPointerException`, `ArithmeticException` |
| Обработка | try-catch или throws | По желанию |

```java
// Checked исключение — требует обработки
try {
    FileReader reader = new FileReader("file.txt");
} catch (FileNotFoundException e) {
    e.printStackTrace();
}

// Unchecked исключение — может не обрабатываться
int result = 10 / 0; // ArithmeticException
```

### 2.3. Конструкция try-catch-finally

```java
try {
    // Код, который может выбросить исключение
    int result = 10 / 0;
} catch (ArithmeticException e) {
    // Обработка конкретного исключения
    System.err.println("Ошибка деления на ноль: " + e.getMessage());
} catch (Exception e) {
    // Обработка всех остальных исключений
    System.err.println("Общая ошибка: " + e.getMessage());
} finally {
    // Выполняется всегда (даже если было исключение)
    System.out.println("Блок finally выполнен");
}
```

### 2.4. Try-with-resources (Java 7+)

Автоматическое закрытие ресурсов, реализующих `AutoCloseable`:

```java
// До Java 7
BufferedReader reader = null;
try {
    reader = new BufferedReader(new FileReader("file.txt"));
    String line = reader.readLine();
} catch (IOException e) {
    e.printStackTrace();
} finally {
    if (reader != null) {
        try {
            reader.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

// Java 7+ (try-with-resources)
try (BufferedReader reader = new BufferedReader(new FileReader("file.txt"))) {
    String line = reader.readLine();
} catch (IOException e) {
    e.printStackTrace();
}
// reader автоматически закрыт
```

### 2.5. Ключевое слово throw

Используется для явного выброса исключения:

```java
public void validateAge(int age) {
    if (age < 0 || age > 150) {
        throw new IllegalArgumentException("Некорректный возраст: " + age);
    }
}
```

### 2.6. Создание собственных исключений

**Checked исключение:**

```java
public class BookNotFoundException extends Exception {
    public BookNotFoundException(String message) {
        super(message);
    }
    
    public BookNotFoundException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

**Unchecked исключение:**

```java
public class InvalidBookDataException extends RuntimeException {
    public InvalidBookDataException(String message) {
        super(message);
    }
    
    public InvalidBookDataException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

### 2.7. Логирование с SLF4J и Logback

**Подключение зависимости (Maven):**

```xml
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
```

**Использование:**

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class BookService {
    private static final Logger logger = LoggerFactory.getLogger(BookService.class);
    
    public void saveBook(Book book) {
        logger.info("Сохранение книги: {}", book.getTitle());
        try {
            // сохранение в БД
            logger.info("Книга успешно сохранена: id={}", book.getId());
        } catch (Exception e) {
            logger.error("Ошибка при сохранении книги: {}", book.getTitle(), e);
            throw new RuntimeException("Не удалось сохранить книгу", e);
        }
    }
    
    public Book findById(long id) {
        Book book = repository.findById(id);
        if (book == null) {
            logger.warn("Книга с id {} не найдена", id);
            throw new BookNotFoundException("Книга с id " + id + " не найдена");
        }
        return book;
    }
}
```

**Конфигурация logback.xml:**

```xml
<configuration>
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/application.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>logs/application.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    
    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="FILE"/>
    </root>
</configuration>
```

### 2.8. Уровни логирования

| Уровень | Использование |
|---------|---------------|
| ERROR | Ошибки, которые требуют вмешательства |
| WARN | Потенциальные проблемы, которые не критичны |
| INFO | Важные события (успешные операции) |
| DEBUG | Детальная отладочная информация |
| TRACE | Ещё более детальная информация |

---

## 3. Задание на паре

### Задача. Обработка исключений и логирование в системе управления книгами

1. **Создать checked-исключение `BookNotFoundException`:**
   - Наследуется от `Exception`;
   - Конструктор с сообщением;
   - Конструктор с сообщением и причиной.

2. **Создать unchecked-исключение `InvalidBookDataException`:**
   - Наследуется от `RuntimeException`;
   - Конструктор с сообщением;
   - Конструктор с сообщением и причиной.

3. **Доработать `Repository<Book>`:**
   - Метод `findById(long id)` выбрасывает `BookNotFoundException` при отсутствии книги;
   - Добавить логирование WARN при попытке получить несуществующую книгу;
   - Метод `save(T entity)` логирует INFO при успешном сохранении;
   - Метод `delete(long id)` логирует INFO при успешном удалении.

4. **Добавить валидацию в сеттеры `Book`:**
   - `title` — не null и не пустой;
   - `author` — не null и не пустой;
   - `year` — > 0 и ≤ текущий год;
   - При нарушении — бросать `InvalidBookDataException` с описанием ошибки;
   - Логировать ERROR при попытке создания/обновления с некорректными данными.

5. **Подключить SLF4J + Logback:**
   - Настроить логирование в файл и в консоль;
   - Логировать успешные операции (INFO);
   - Логировать предупреждения (WARN);
   - Логировать ошибки (ERROR).

6. **Написать код с try-catch-finally и try-with-resources:**
   - Чтение/запись лога в файл;
   - Попытка загрузки книг из файла с обработкой ошибок.

**Пример выполнения:**

```
=== Сохранение книг ===
2026-01-15 10:30:15 [main] INFO  BookRepository - Сохранение книги: Война и мир
2026-01-15 10:30:15 [main] INFO  BookRepository - Книга успешно сохранена: id=1

=== Попытка получения несуществующей книги ===
2026-01-15 10:30:16 [main] WARN  BookRepository - Попытка получить книгу с id 999
BookNotFoundException: Книга с id 999 не найдена

=== Валидация книги ===
2026-01-15 10:30:17 [main] ERROR BookService - Ошибка валидации: year должно быть > 0
InvalidBookDataException: year должно быть > 0
```

### Применение ИИ-инструмента

**Промпт для YandexGPT:**

```
Создай на Java 17:

1. Checked-исключение BookNotFoundException, наследующее Exception.
   - Конструктор с сообщением
   - Конструктор с сообщением и причиной

2. Unchecked-исключение InvalidBookDataException, наследующее RuntimeException.
   - Конструктор с сообщением
   - Конструктор с сообщением и причиной

Добавь конструкторы с цепочкой причин (Throwable cause).
```

**Анализ результата:**

- Проверить, что оба класса корректно наследуются.
- Проверить наличие обоих конструкторов.
- Проверить, что конструкторы вызывают `super()` с соответствующими параметрами.
- Оценить, добавил ли ИИ конструктор с цепочкой причин.

---

## 4. Индивидуальные задания (30 вариантов)

Каждый вариант содержит:
- 2 собственных исключения (1 checked, 1 unchecked);
- Класс-сущность с валидацией в сеттерах;
- Репозиторий с выбрасыванием исключений;
- Логирование всех операций.

---

### Вариант 1. Иерархия «Книги»

**Сущность `Book`:** `id`, `title`, `author`, `year`, `isbn`, `publisher`

**Исключения:**
- `BookNotFoundException` (checked)
- `InvalidBookDataException` (unchecked)

**Валидация:** title не null/пустой, author не null/пустой, year ∈ [1, текущий]

**Дополнительно:** isbn уникальный, publisher не null

---

### Вариант 2. Иерархия «Сотрудники»

**Сущность `Employee`:** `id`, `fullName`, `position`, `salary`, `department`

**Исключения:**
- `EmployeeNotFoundException` (checked)
- `InvalidEmployeeDataException` (unchecked)

**Валидация:** fullName не null/пустой, salary > 0, position не null/пустой

---

### Вариант 3. Иерархия «Товары»

**Сущность `Product`:** `id`, `name`, `category`, `price`, `quantity`, `supplier`

**Исключения:**
- `ProductNotFoundException` (checked)
- `InvalidProductDataException` (unchecked)

**Валидация:** name не null/пустой, price > 0, quantity ≥ 0

---

### Вариант 4. Иерархия «Заказы»

**Сущность `Order`:** `id`, `customerName`, `totalAmount`, `status`, `createdAt`

**Исключения:**
- `OrderNotFoundException` (checked)
- `InvalidOrderDataException` (unchecked)

**Валидация:** customerName не null/пустой, totalAmount > 0, status не null

---

### Вариант 5. Иерархия «Пользователи»

**Сущность `User`:** `id`, `username`, `email`, `password`, `role`

**Исключения:**
- `UserNotFoundException` (checked)
- `InvalidUserDataException` (unchecked)

**Валидация:** username не null/пустой, email содержит @, password длина ≥ 8

---

### Вариант 6. Иерархия «Автомобили»

**Сущность `Car`:** `id`, `brand`, `model`, `year`, `price`, `vin`

**Исключения:**
- `CarNotFoundException` (checked)
- `InvalidCarDataException` (unchecked)

**Валидация:** brand не null/пустой, year ∈ [1886, текущий], price > 0

---

### Вариант 7. Иерархия «Студенты»

**Сущность `Student`:** `id`, `firstName`, `lastName`, `group`, `gpa`, `birthDate`

**Исключения:**
- `StudentNotFoundException` (checked)
- `InvalidStudentDataException` (unchecked)

**Валидация:** firstName не null/пустой, gpa ∈ [0.0, 5.0], birthDate не null

---

### Вариант 8. Иерархия «Счета»

**Сущность `Account`:** `id`, `accountNumber`, `ownerName`, `balance`, `currency`

**Исключения:**
- `AccountNotFoundException` (checked)
- `InvalidAccountDataException` (unchecked)

**Валидация:** accountNumber не null/пустой, balance ≥ 0, currency не null

---

### Вариант 9. Иерархия «Фильмы»

**Сущность `Movie`:** `id`, `title`, `director`, `year`, `rating`, `genre`

**Исключения:**
- `MovieNotFoundException` (checked)
- `InvalidMovieDataException` (unchecked)

**Валидация:** title не null/пустой, year ∈ [1890, текущий], rating ∈ [0.0, 10.0]

---

### Вариант 10. Иерархия «Рестораны»

**Сущность `Restaurant`:** `id`, `name`, `address`, `phone`, `rating`, `cuisine`

**Исключения:**
- `RestaurantNotFoundException` (checked)
- `InvalidRestaurantDataException` (unchecked)

**Валидация:** name не null/пустой, phone не null/пустой, rating ∈ [0.0, 5.0]

---

### Вариант 11. Иерархия «Транзакции»

**Сущность `Transaction`:** `id`, `amount`, `type`, `date`, `description`, `category`

**Исключения:**
- `TransactionNotFoundException` (checked)
- `InvalidTransactionDataException` (unchecked)

**Валидация:** amount ≠ 0, type не null, date не null

---

### Вариант 12. Иерархия «Клиенты»

**Сущность `Customer`:** `id`, `firstName`, `lastName`, `email`, `phone`, `birthDate`

**Исключения:**
- `CustomerNotFoundException` (checked)
- `InvalidCustomerDataException` (unchecked)

**Валидация:** email содержит @, phone не null/пустой, birthDate не null

---

### Вариант 13. Иерархия «Договоры»

**Сущность `Contract`:** `id`, `number`, `client`, `signDate`, `amount`, `status`

**Исключения:**
- `ContractNotFoundException` (checked)
- `InvalidContractDataException` (unchecked)

**Валидация:** number не null/пустой, amount > 0, signDate не null

---

### Вариант 14. Иерархия «Отели»

**Сущность `Hotel`:** `id`, `name`, `city`, `stars`, `roomsCount`, `rating`

**Исключения:**
- `HotelNotFoundException` (checked)
- `InvalidHotelDataException` (unchecked)

**Валидация:** name не null/пустой, stars ∈ [1, 5], roomsCount > 0

---

### Вариант 15. Иерархия «Спортсмены»

**Сущность `Athlete`:** `id`, `firstName`, `lastName`, `sport`, `age`, `medals`

**Исключения:**
- `AthleteNotFoundException` (checked)
- `InvalidAthleteDataException` (unchecked)

**Валидация:** firstName не null/пустой, age ∈ [6, 100], medals ≥ 0

---

### Вариант 16. Иерархия «Статьи»

**Сущность `Article`:** `id`, `title`, `content`, `author`, `publishedDate`, `views`

**Исключения:**
- `ArticleNotFoundException` (checked)
- `InvalidArticleDataException` (unchecked)

**Валидация:** title не null/пустой, content не null/пустой, views ≥ 0

---

### Вариант 17. Иерархия «Билеты»

**Сущность `Ticket`:** `id`, `eventName`, `venue`, `date`, `price`, `quantity`

**Исключения:**
- `TicketNotFoundException` (checked)
- `InvalidTicketDataException` (unchecked)

**Валидация:** eventName не null/пустой, price > 0, quantity > 0

---

### Вариант 18. Иерархия «Курсы»

**Сущность `Course`:** `id`, `title`, `instructor`, `duration`, `price`, `startDate`

**Исключения:**
- `CourseNotFoundException` (checked)
- `InvalidCourseDataException` (unchecked)

**Валидация:** title не null/пустой, duration > 0, price ≥ 0

---

### Вариант 19. Иерархия «Инвентарь»

**Сущность `Item`:** `id`, `name`, `category`, `quantity`, `location`, `status`

**Исключения:**
- `ItemNotFoundException` (checked)
- `InvalidItemDataException` (unchecked)

**Валидация:** name не null/пустой, quantity ≥ 0, category не null

---

### Вариант 20. Иерархия «Заявки»

**Сущность `Request`:** `id`, `clientName`, `description`, `priority`, `status`, `createdAt`

**Исключения:**
- `RequestNotFoundException` (checked)
- `InvalidRequestDataException` (unchecked)

**Валидация:** clientName не null/пустой, priority ∈ [1, 5], createdAt не null

---

### Вариант 21. Иерархия «Платёжные поручения»

**Сущность `PaymentOrder`:** `id`, `number`, `payer`, `recipient`, `amount`, `date`

**Исключения:**
- `PaymentOrderNotFoundException` (checked)
- `InvalidPaymentOrderException` (unchecked)

**Валидация:** number не null/пустой, amount > 0, date не null

---

### Вариант 22. Иерархия «Публикации»

**Сущность `Publication`:** `id`, `title`, `author`, `journal`, `year`, `doi`

**Исключения:**
- `PublicationNotFoundException` (checked)
- `InvalidPublicationException` (unchecked)

**Валидация:** title не null/пустой, year ∈ [1900, текущий], doi не null/пустой

---

### Вариант 23. Иерархия «Ноутбуки»

**Сущность `Laptop`:** `id`, `brand`, `model`, `processor`, `ram`, `price`

**Исключения:**
- `LaptopNotFoundException` (checked)
- `InvalidLaptopDataException` (unchecked)

**Валидация:** brand не null/пустой, ram ∈ [4, 128], price > 0

---

### Вариант 24. Иерархия «Врачи»

**Сущность `Doctor`:** `id`, `firstName`, `lastName`, `specialization`, `experience`, `rating`

**Исключения:**
- `DoctorNotFoundException` (checked)
- `InvalidDoctorDataException` (unchecked)

**Валидация:** firstName не null/пустой, experience ≥ 0, rating ∈ [0.0, 5.0]

---

### Вариант 25. Иерархия «Сертификаты»

**Сущность `Certificate`:** `id`, `number`, `holderName`, `issuedDate`, `expiryDate`, `type`

**Исключения:**
- `CertificateNotFoundException` (checked)
- `InvalidCertificateDataException` (unchecked)

**Валидация:** number не null/пустой, holderName не null, expiryDate после issuedDate

---

### Вариант 26. Иерархия «Магазины»

**Сущность `Shop`:** `id`, `name`, `address`, `phone`, `openingHours`, `type`

**Исключения:**
- `ShopNotFoundException` (checked)
- `InvalidShopDataException` (unchecked)

**Валидация:** name не null/пустой, address не null, phone не null/пустой

---

### Вариант 27. Иерархия «Здания»

**Сущность `Building`:** `id`, `address`, `floors`, `area`, `type`, `yearBuilt`

**Исключения:**
- `BuildingNotFoundException` (checked)
- `InvalidBuildingDataException` (unchecked)

**Валидация:** address не null/пустой, floors > 0, area > 0

---

### Вариант 28. Иерархия «Лекции»

**Сущность `Lecture`:** `id`, `topic`, `speaker`, `date`, `duration`, `attendees`

**Исключения:**
- `LectureNotFoundException` (checked)
- `InvalidLectureDataException` (unchecked)

**Валидация:** topic не null/пустой, duration > 0, attendees ≥ 0

---

### Вариант 29. Иерархия «Отзывы»

**Сущность `Review`:** `id`, `userId`, `productId`, `rating`, `comment`, `createdAt`

**Исключения:**
- `ReviewNotFoundException` (checked)
- `InvalidReviewDataException` (unchecked)

**Валидация:** userId > 0, productId > 0, rating ∈ [1, 5]

---

### Вариант 30. Иерархия «Чеки»

**Сущность `Receipt`:** `id`, `number`, `store`, `totalAmount`, `date`, `items`

**Исключения:**
- `ReceiptNotFoundException` (checked)
- `InvalidReceiptDataException` (unchecked)

**Валидация:** number не null/пустой, totalAmount > 0, date не null

---

## 5. Методические указания

### 5.1. Общие правила

1. **Создавайте исключения в отдельном пакете** (например, `exceptions`).

2. **Все конструкторы исключений** должны вызывать `super()` с соответствующими параметрами.

3. **Валидация данных** должна выполняться в сеттерах и конструкторах.

4. **Логирование** должно быть расставлено на всех уровнях:
   - INFO — успешные операции;
   - WARN — некритичные проблемы;
   - ERROR — критичные ошибки.

5. **Ведите журнал применения ИИ** — это обязательная часть отчёта.

### 5.2. Шаблон создания исключения

```java
/**
 * Checked исключение, выбрасываемое при отсутствии сущности.
 */
public class BookNotFoundException extends Exception {
    
    public BookNotFoundException(String message) {
        super(message);
    }
    
    public BookNotFoundException(String message, Throwable cause) {
        super(message, cause);
    }
}

/**
 * Unchecked исключение, выбрасываемое при неверных данных сущности.
 */
public class InvalidBookDataException extends RuntimeException {
    
    public InvalidBookDataException(String message) {
        super(message);
    }
    
    public InvalidBookDataException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

### 5.3. Шаблон валидации в сеттерах

```java
public void setTitle(String title) {
    if (title == null || title.trim().isEmpty()) {
        logger.error("Ошибка валидации: title не может быть null или пустым");
        throw new InvalidBookDataException("title не может быть null или пустым");
    }
    this.title = title;
}
```

### 5.4. Шаблон репозитория с логированием

```java
public class BookRepository {
    private static final Logger logger = LoggerFactory.getLogger(BookRepository.class);
    private final Map<Long, Book> storage = new HashMap<>();
    private long nextId = 1;
    
    public Book save(Book book) {
        logger.info("Сохранение книги: {}", book.getTitle());
        book.setId(nextId++);
        storage.put(book.getId(), book);
        logger.info("Книга успешно сохранена: id={}", book.getId());
        return book;
    }
    
    public Book findById(long id) {
        Book book = storage.get(id);
        if (book == null) {
            logger.warn("Попытка получить книгу с id {}", id);
            throw new BookNotFoundException("Книга с id " + id + " не найдена");
        }
        return book;
    }
    
    public void delete(long id) {
        Book removed = storage.remove(id);
        if (removed == null) {
            logger.warn("Попытка удалить книгу с id {}", id);
            throw new BookNotFoundException("Книга с id " + id + " не найдена");
        }
        logger.info("Книга успешно удалена: id={}, title={}", id, removed.getTitle());
    }
}
```

### 5.5. Конфигурация logback.xml

Расположите файл `logback.xml` в `src/main/resources/`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <!-- Консольный вывод -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    
    <!-- Вывод в файл -->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/application.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>logs/application.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>10MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    
    <!-- Отдельный файл для ошибок -->
    <appender name="ERROR_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/errors.log</file>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>ERROR</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>logs/errors.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>10MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    
    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="FILE"/>
        <appender-ref ref="ERROR_FILE"/>
    </root>
</configuration>
```

### 5.6. Пример использования try-with-resources

```java
public void processFile(String filePath) {
    logger.info("Обработка файла: {}", filePath);
    int successCount = 0;
    int errorCount = 0;
    
    try (BufferedReader reader = new BufferedReader(new FileReader(filePath))) {
        String line;
        while ((line = reader.readLine()) != null) {
            try {
                processLine(line);
                successCount++;
            } catch (InvalidBookDataException e) {
                errorCount++;
                logger.error("Ошибка при обработке строки: {}", line, e);
            }
        }
    } catch (FileNotFoundException e) {
        logger.error("Файл не найден: {}", filePath, e);
        throw new RuntimeException("Файл не найден", e);
    } catch (IOException e) {
        logger.error("Ошибка при чтении файла: {}", filePath, e);
        throw new RuntimeException("Ошибка при чтении файла", e);
    }
    
    logger.info("Обработка завершена: успешно={}, ошибок={}", successCount, errorCount);
}
```

---

## 6. Контрольные вопросы

1. Что такое иерархия исключений в Java? Назовите основные классы.

2. В чём разница между checked и unchecked исключениями?

3. Что такое try-with-resources и как он работает?

4. В чём отличие между ключевыми словами `throw` и `throws`?

5. Как создать собственное checked-исключение?

6. Как создать собственное unchecked-исключение?

7. Что такое SLF4J и Logback?

8. Перечислите уровни логирования и опишите, когда какой использовать.

9. Почему важно логировать операции в приложении?

10. Что такое цепочка причин (cause) в исключениях?

11. В чём отличие `finally` от `try-with-resources`?

12. Как настроить логирование в файл?

---

## 7. Пример выполнения (Вариант 1)

### 7.1. Классы исключений

```java
// exceptions/BookNotFoundException.java
package exceptions;

public class BookNotFoundException extends Exception {
    
    public BookNotFoundException(String message) {
        super(message);
    }
    
    public BookNotFoundException(String message, Throwable cause) {
        super(message, cause);
    }
}

// exceptions/InvalidBookDataException.java
package exceptions;

public class InvalidBookDataException extends RuntimeException {
    
    public InvalidBookDataException(String message) {
        super(message);
    }
    
    public InvalidBookDataException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

### 7.2. Класс Book

```java
package model;

import exceptions.InvalidBookDataException;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.time.Year;

public class Book {
    private static final Logger logger = LoggerFactory.getLogger(Book.class);
    
    private long id;
    private String title;
    private String author;
    private int year;
    private String isbn;
    private String publisher;
    
    public Book() {}
    
    public Book(String title, String author, int year, String isbn, String publisher) {
        setTitle(title);
        setAuthor(author);
        setYear(year);
        setIsbn(isbn);
        setPublisher(publisher);
    }
    
    // Геттеры и сеттеры с валидацией
    
    public long getId() { return id; }
    public void setId(long id) { this.id = id; }
    
    public String getTitle() { return title; }
    public void setTitle(String title) {
        if (title == null || title.trim().isEmpty()) {
            logger.error("Ошибка валидации: title не может быть null или пустым");
            throw new InvalidBookDataException("title не может быть null или пустым");
        }
        this.title = title;
    }
    
    public String getAuthor() { return author; }
    public void setAuthor(String author) {
        if (author == null || author.trim().isEmpty()) {
            logger.error("Ошибка валидации: author не может быть null или пустым");
            throw new InvalidBookDataException("author не может быть null или пустым");
        }
        this.author = author;
    }
    
    public int getYear() { return year; }
    public void setYear(int year) {
        int currentYear = Year.now().getValue();
        if (year <= 0 || year > currentYear) {
            logger.error("Ошибка валидации: year должно быть в диапазоне (0, {}]", currentYear);
            throw new InvalidBookDataException("year должно быть в диапазоне (0, " + currentYear + "]");
        }
        this.year = year;
    }
    
    public String getIsbn() { return isbn; }
    public void setIsbn(String isbn) {
        if (isbn == null || isbn.trim().isEmpty()) {
            logger.error("Ошибка валидации: isbn не может быть null или пустым");
            throw new InvalidBookDataException("isbn не может быть null или пустым");
        }
        this.isbn = isbn;
    }
    
    public String getPublisher() { return publisher; }
    public void setPublisher(String publisher) {
        if (publisher == null || publisher.trim().isEmpty()) {
            logger.error("Ошибка валидации: publisher не может быть null или пустым");
            throw new InvalidBookDataException("publisher не может быть null или пустым");
        }
        this.publisher = publisher;
    }
    
    @Override
    public String toString() {
        return String.format("Book{id=%d, title='%s', author='%s', year=%d, isbn='%s', publisher='%s'}",
                id, title, author, year, isbn, publisher);
    }
}
```

### 7.3. Класс BookRepository

```java
package repository;

import exceptions.BookNotFoundException;
import model.Book;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.HashMap;
import java.util.Map;

public class BookRepository {
    private static final Logger logger = LoggerFactory.getLogger(BookRepository.class);
    private final Map<Long, Book> storage = new HashMap<>();
    private long nextId = 1;
    
    public Book save(Book book) {
        logger.info("Сохранение книги: {}", book.getTitle());
        book.setId(nextId++);
        storage.put(book.getId(), book);
        logger.info("Книга успешно сохранена: id={}, title={}", book.getId(), book.getTitle());
        return book;
    }
    
    public Book findById(long id) throws BookNotFoundException {
        Book book = storage.get(id);
        if (book == null) {
            logger.warn("Попытка получить книгу с id {}", id);
            throw new BookNotFoundException("Книга с id " + id + " не найдена");
        }
        return book;
    }
    
    public void delete(long id) throws BookNotFoundException {
        Book removed = storage.remove(id);
        if (removed == null) {
            logger.warn("Попытка удалить книгу с id {}", id);
            throw new BookNotFoundException("Книга с id " + id + " не найдена");
        }
        logger.info("Книга успешно удалена: id={}, title={}", id, removed.getTitle());
    }
    
    public Map<Long, Book> findAll() {
        logger.debug("Получение всех книг. Количество: {}", storage.size());
        return new HashMap<>(storage);
    }
}
```

### 7.4. Main-класс для демонстрации

```java
import exceptions.BookNotFoundException;
import exceptions.InvalidBookDataException;
import model.Book;
import repository.BookRepository;

public class Main {
    public static void main(String[] args) {
        System.out.println("=== Обработка исключений и логирование ===\n");
        
        BookRepository repository = new BookRepository();
        
        // 1. Успешное создание и сохранение книги
        try {
            System.out.println("1. Сохранение книги:");
            Book book = new Book("Война и мир", "Толстой Л.Н.", 1869, "978-5-17-118456-0", "АСТ");
            repository.save(book);
            System.out.println("✓ Книга успешно сохранена\n");
        } catch (InvalidBookDataException e) {
            System.err.println("✗ Ошибка при создании книги: " + e.getMessage());
        }
        
        // 2. Попытка получения несуществующей книги
        System.out.println("2. Попытка получения несуществующей книги:");
        try {
            Book found = repository.findById(999L);
            System.out.println("Найдена книга: " + found);
        } catch (BookNotFoundException e) {
            System.err.println("✗ " + e.getMessage());
        }
        System.out.println();
        
        // 3. Создание книги с неверными данными
        System.out.println("3. Создание книги с неверными данными:");
        try {
            Book invalidBook = new Book("", "Автор", 2026, "ISBN", "Издательство");
            // Не должно достигнуто
            System.err.println("✗ Ошибка: книга с неверными данными была создана!");
        } catch (InvalidBookDataException e) {
            System.err.println("✗ Ошибка валидации: " + e.getMessage());
        }
        System.out.println();
        
        // 4. try-with-resources для чтения файла
        System.out.println("4. Чтение данных из файла:");
        processFile("books.csv");
    }
    
    private static void processFile(String filePath) {
        // Имитация обработки файла
        System.out.println("Обработка файла: " + filePath);
        System.out.println("✓ Файл успешно обработан (имитация)");
    }
}
```

### 7.5. Зависимости Maven

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
</dependencies>
```

---

## 8. Рекомендуемые источники

1. **Шилдт Г.** *Java. Базовый курс.* — М.: Вильямс. — Главы 10–11 (Обработка исключений).

2. **Хорстманн К., Корнелл Г.** *Java. Библиотека профессионала. Том 1.* — М.: Вильямс. — Глава 7 (Обработка исключений, логирование).

3. **Блох Дж.** *Java. Эффективное программирование.* — М.: Питер. — Правило 73 (Пробрасывайте исключения, соответствующие абстракции).

4. **Oracle Java Tutorials: Exceptions.** — URL: https://docs.oracle.com/javase/tutorial/essential/exceptions/

5. **SLF4J Documentation.** — URL: https://www.slf4j.org/docs.html

6. **Logback Documentation.** — URL: https://logback.qos.ch/documentation.html
