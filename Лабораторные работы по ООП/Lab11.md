# Лабораторная работа №11. Сериализация и работа с JSON

## 1. Паспорт работы

| Параметр | Значение |
|----------|----------|
| Тема | Сериализация и работа с JSON |
| Номер занятия в модуле | 3 из 4 (модуль 3) |
| Продолжительность аудиторной части | 2 академических часа |
| Предшествующая подготовка | Лабораторные работы №9–10 (функциональные интерфейсы, Stream API) |
| Тип работы | Формирование навыков обмена данными в формате JSON с применением библиотек Jackson и Gson |

### 1.1. Цель работы

Освоить механизмы сериализации и десериализации объектов языка Java в формат JSON и обратно. Изучить возможности библиотек Jackson и Gson, научиться применять аннотации для управления процессом преобразования, обрабатывать сложные иерархические структуры, коллекции, записи (`record`) и перечисления (`enum`), а также корректно обрабатывать ошибки десериализации.

### 1.2. Задачи работы

1. Изучить формат JSON, его синтаксис и область применения.
2. Освоить базовые операции сериализации и десериализации с использованием Jackson `ObjectMapper`.
3. Изучить аннотации Jackson: `@JsonProperty`, `@JsonIgnore`, `@JsonFormat`, `@JsonCreator`, `@JsonAlias`, `@JsonInclude`.
4. Освоить работу с вложенными объектами, коллекциями и массивами.
5. Изучить сериализацию записей (`record`) и перечислений (`enum`).
6. Освоить полиморфную десериализацию с применением `@JsonTypeInfo` и `@JsonSubTypes`.
7. Изучить возможности библиотеки Gson: `Gson`, `@SerializedName`, `TypeToken`.
8. Научиться создавать кастомные сериализаторы и десериализаторы.
9. Освоить обработку ошибок десериализации и валидацию данных.

### 1.3. Оснащение

- JDK версии 17 или выше;
- интегрированная среда разработки IntelliJ IDEA Community Edition;
- система сборки Maven или Gradle;
- система контроля версий Git;
- система модульного тестирования JUnit 5;
- библиотека Jackson 2.x (`jackson-databind`, `jackson-datatype-jsr310`);
- библиотека Gson 2.x (опционально, для сравнения).

## 2. Теоретические сведения

### 2.1. Мотивация: зачем нужна сериализация

В современных распределённых системах данные необходимо передавать между различными компонентами: клиентом и сервером, микросервисами, приложениями на разных языках программирования. Для этого требуется универсальный формат обмена данными, независимый от языка программирования и платформы.

Наиболее распространёнными форматами обмена данными являются:

- **JSON** (JavaScript Object Notation) — текстовый формат, основанный на синтаксисе JavaScript;
- **XML** (eXtensible Markup Language) — древовидный формат на основе тегов;
- **YAML** (YAML Ain't Markup Language) — формат, ориентированный на читаемость;
- **Protobuf** (Protocol Buffers) — бинарный формат от Google.

В рамках настоящей работы основное внимание уделяется формату JSON как наиболее распространённому в веб-разработке и интеграции с внешними API.

### 2.2. Формат JSON

**JSON** (JavaScript Object Notation) — лёгкий текстовый формат обмена данными, легко читаемый человеком и простой для генерации и анализа машинами.

**Основные конструкции JSON:**

1. **Объект** — неупорядоченное множество пар «ключ-значение», заключённое в фигурные скобки:
   ```json
   {
     "name": "Иванов И.И.",
     "age": 20,
     "isStudent": true
   }
   ```

2. **Массив** — упорядоченный список значений, заключённый в квадратные скобки:
   ```json
   [1, 2, 3, 4, 5]
   ```

3. **Значения** могут быть следующих типов:
   - строка (в двойных кавычках): `"Привет"`;
   - число: `42`, `3.14`;
   - логическое: `true`, `false`;
   - `null` — отсутствующее значение;
   - объект: `{...}`;
   - массив: `[...]`.

**Пример сложного JSON-документа:**

```json
{
  "id": "ORD-001",
  "customer": {
    "name": "Иванов И.И.",
    "email": "ivanov@example.com",
    "phone": "+79001234567"
  },
  "items": [
    {
      "productId": "P001",
      "name": "Ноутбук",
      "quantity": 1,
      "price": 85000.00
    },
    {
      "productId": "P002",
      "name": "Мышь",
      "quantity": 2,
      "price": 1500.00
    }
  ],
  "totalAmount": 88000.00,
  "createdAt": "2026-08-20T14:30:00",
  "status": "CONFIRMED"
}
```

### 2.3. Сопоставление типов Java и JSON

| Тип Java | Представление в JSON |
|----------|----------------------|
| `String` | строка в двойных кавычках |
| `int`, `long`, `short`, `byte` | число без дробной части |
| `float`, `double`, `BigDecimal` | число (с дробной частью или без) |
| `boolean` | `true` или `false` |
| `null` (ссылочные типы) | `null` |
| Массив (`T[]`) | массив JSON |
| `List<T>`, `Set<T>` | массив JSON |
| `Map<String, V>` | объект JSON |
| Класс/запись | объект JSON (поля как ключи) |
| `enum` | строка (имя константы) |
| `LocalDate`, `LocalDateTime` | строка (по умолчанию ISO-формат) |

### 2.4. Библиотека Jackson

**Jackson** — наиболее распространённая библиотека для работы с JSON в Java. Состоит из нескольких модулей:

- `jackson-core` — базовый парсер и генератор JSON;
- `jackson-annotations` — аннотации для управления сериализацией;
- `jackson-databind` — высокоуровневый API (`ObjectMapper`);
- `jackson-datatype-jsr310` — поддержка классов даты и времени из Java 8+.

**Подключение в Maven:**

```xml
<dependencies>
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.17.0</version>
    </dependency>
    <dependency>
        <groupId>com.fasterxml.jackson.datatype</groupId>
        <artifactId>jackson-datatype-jsr310</artifactId>
        <version>2.17.0</version>
    </dependency>
</dependencies>
```

### 2.5. Класс `ObjectMapper`

**`ObjectMapper`** — центральный класс библиотеки Jackson, выполняющий сериализацию и десериализацию. Рекомендуется создавать один экземпляр `ObjectMapper` на приложение и переиспользовать его (он потокобезопасен после конфигурации).

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializationFeature;
import com.fasterxml.jackson.datatype.jsr310.JavaTimeModule;

// Создание и конфигурация
ObjectMapper mapper = new ObjectMapper();
mapper.registerModule(new JavaTimeModule());   // поддержка java.time.*
mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
mapper.enable(SerializationFeature.INDENT_OUTPUT);   // красивое форматирование

// Сериализация
String json = mapper.writeValueAsString(object);

// Десериализация
MyClass obj = mapper.readValue(json, MyClass.class);
```

### 2.6. Аннотации Jackson

#### 2.6.1. `@JsonProperty`

Управляет именем поля в JSON. Применяется, когда имя поля в Java не совпадает с именем в JSON:

```java
public class User {
    @JsonProperty("user_name")
    private String userName;

    @JsonProperty("email_address")
    private String email;
}
```

#### 2.6.2. `@JsonIgnore`

Исключает поле из сериализации и десериализации:

```java
public class User {
    private String name;

    @JsonIgnore
    private String password;   // не будет включён в JSON
}
```

#### 2.6.3. `@JsonFormat`

Определяет формат для дат и чисел:

```java
public class Event {
    @JsonFormat(pattern = "dd.MM.yyyy HH:mm")
    private LocalDateTime dateTime;

    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "#,##0.00")
    private double amount;
}
```

#### 2.6.4. `@JsonCreator` и `@JsonProperty` в конструкторе

Позволяет использовать конструктор с параметрами для десериализации (важно для неизменяемых классов и записей):

```java
public class Money {
    private final double amount;
    private final String currency;

    @JsonCreator
    public Money(
        @JsonProperty("amount") double amount,
        @JsonProperty("currency") String currency
    ) {
        this.amount = amount;
        this.currency = currency;
    }
}
```

#### 2.6.5. `@JsonAlias`

Позволяет принимать несколько имён поля при десериализации:

```java
public class User {
    @JsonAlias({"user_name", "username", "login"})
    private String userName;
}
```

#### 2.6.6. `@JsonInclude`

Управляет включением полей со значением `null` или пустых коллекций:

```java
@JsonInclude(JsonInclude.Include.NON_NULL)
public class User {
    private String name;
    private String email;   // если null — не будет включён в JSON
}
```

Варианты `Include`:
- `ALWAYS` (по умолчанию) — включать все поля;
- `NON_NULL` — исключать `null`;
- `NON_EMPTY` — исключать `null`, пустые строки и коллекции;
- `NON_DEFAULT` — исключать значения по умолчанию.

### 2.7. Сериализация записей (`record`)

Записи Java (с Java 16) сериализуются в JSON «из коробки» благодаря каноническому конструктору и геттерам. Для корректной десериализации необходимо, чтобы библиотека могла использовать конструктор:

```java
public record Address(
    String city,
    String street,
    String building,
    String apartment
) {}

public record User(
    String id,
    String name,
    Address address
) {}
```

Сериализация:

```java
User user = new User("U001", "Иванов И.И.",
    new Address("Москва", "Тверская", "1", "10"));

String json = mapper.writeValueAsString(user);
// {"id":"U001","name":"Иванов И.И.","address":{"city":"Москва","street":"Тверская","building":"1","apartment":"10"}}
```

Десериализация:

```java
User restored = mapper.readValue(json, User.class);
```

### 2.8. Сериализация перечислений (`enum`)

По умолчанию перечисления сериализуются как строки с именами констант:

```java
public enum OrderStatus {
    NEW, CONFIRMED, SHIPPED, DELIVERED, CANCELLED
}
```

Для кастомизации применяется аннотация `@JsonValue` или `@JsonProperty`:

```java
public enum OrderStatus {
    @JsonProperty("new") NEW,
    @JsonProperty("confirmed") CONFIRMED,
    @JsonProperty("shipped") SHIPPED,
    @JsonProperty("delivered") DELIVERED,
    @JsonProperty("cancelled") CANCELLED;
}
```

Или с использованием `@JsonValue`:

```java
public enum Currency {
    RUB("Russian Ruble"),
    USD("US Dollar"),
    EUR("Euro");

    private final String description;

    Currency(String description) {
        this.description = description;
    }

    @JsonValue
    public String getDescription() {
        return description;
    }

    @JsonCreator
    public static Currency fromDescription(String description) {
        for (Currency c : values()) {
            if (c.description.equalsIgnoreCase(description)) {
                return c;
            }
        }
        throw new IllegalArgumentException("Неизвестная валюта: " + description);
    }
}
```

### 2.9. Работа с коллекциями

Jackson корректно обрабатывает стандартные коллекции Java:

```java
// Сериализация списка
List<User> users = List.of(
    new User("U001", "Иванов", null),
    new User("U002", "Петров", null)
);
String json = mapper.writeValueAsString(users);
// [{"id":"U001","name":"Иванов"},{"id":"U002","name":"Петров"}]

// Десериализация списка
List<User> restored = mapper.readValue(json, new TypeReference<List<User>>() {});

// Сериализация карты
Map<String, Integer> ages = Map.of("Иванов", 20, "Петров", 22);
String json = mapper.writeValueAsString(ages);
// {"Иванов":20,"Петров":22}

// Десериализация карты
Map<String, Integer> restored = mapper.readValue(json,
    new TypeReference<Map<String, Integer>>() {});
```

### 2.10. Полиморфная десериализация

Для десериализации иерархий классов применяются аннотации `@JsonTypeInfo` и `@JsonSubTypes`:

```java
@JsonTypeInfo(
    use = JsonTypeInfo.Id.NAME,
    include = JsonTypeInfo.As.PROPERTY,
    property = "type"
)
@JsonSubTypes({
    @JsonSubTypes.Type(value = CardPayment.class, name = "card"),
    @JsonSubTypes.Type(value = WalletPayment.class, name = "wallet"),
    @JsonSubTypes.Type(value = MobilePayment.class, name = "mobile")
})
public abstract class Payment {
    private String id;
    private double amount;
    // ...
}

public class CardPayment extends Payment {
    private String cardNumber;
    private String cardHolder;
    // ...
}

public class WalletPayment extends Payment {
    private String walletId;
    // ...
}
```

JSON будет содержать поле `type`, определяющее конкретный подкласс:

```json
{
  "type": "card",
  "id": "P001",
  "amount": 1500.00,
  "cardNumber": "****1234",
  "cardHolder": "IVANOV I.I."
}
```

### 2.11. Обработка ошибок десериализации

При десериализации могут возникать следующие исключения:

- `JsonParseException` — некорректный синтаксис JSON;
- `JsonMappingException` — несоответствие структуры JSON и класса;
- `MismatchedInputException` — несоответствие типов;
- `InvalidFormatException` — некорректный формат даты или числа.

```java
try {
    User user = mapper.readValue(json, User.class);
} catch (JsonParseException e) {
    System.err.println("Некорректный JSON: " + e.getMessage());
} catch (JsonMappingException e) {
    System.err.println("Ошибка маппинга: " + e.getMessage());
    // e.getPath() содержит путь к проблемному полю
} catch (IOException e) {
    System.err.println("Ошибка ввода-вывода: " + e.getMessage());
}
```

### 2.12. Кастомные сериализаторы и десериализаторы

Для сложных случаев создаются собственные сериализаторы и десериализаторы:

```java
public class MoneySerializer extends JsonSerializer<Money> {
    @Override
    public void serialize(Money value, JsonGenerator gen,
                          SerializerProvider serializers) throws IOException {
        gen.writeStartObject();
        gen.writeNumberField("amount", value.getAmount());
        gen.writeStringField("currency", value.getCurrency());
        gen.writeStringField("formatted",
            String.format("%.2f %s", value.getAmount(), value.getCurrency()));
        gen.writeEndObject();
    }
}

// Регистрация
SimpleModule module = new SimpleModule();
module.addSerializer(Money.class, new MoneySerializer());
mapper.registerModule(module);
```

### 2.13. Библиотека Gson

**Gson** — альтернативная библиотека от Google для работы с JSON. Более простая в использовании, но менее гибкая, чем Jackson.

**Подключение:**

```xml
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.10.1</version>
</dependency>
```

**Основной класс — `Gson`:**

```java
Gson gson = new GsonBuilder()
    .setPrettyPrinting()
    .setDateFormat("yyyy-MM-dd HH:mm:ss")
    .serializeNulls()
    .create();

// Сериализация
String json = gson.toJson(object);

// Десериализация
MyClass obj = gson.fromJson(json, MyClass.class);

// Для параметризованных типов
List<User> users = gson.fromJson(json, new TypeToken<List<User>>(){}.getType());
```

**Аннотация `@SerializedName`:**

```java
public class User {
    @SerializedName("user_name")
    private String userName;

    @SerializedName(value = "email", alternate = {"mail", "e_mail"})
    private String email;
}
```

**Сравнение Jackson и Gson:**

| Характеристика | Jackson | Gson |
|----------------|---------|------|
| Производительность | Высокая | Средняя |
| Гибкость | Очень высокая | Средняя |
| Поддержка аннотаций | Богатый набор | Базовый набор |
| Потокобезопасность | `ObjectMapper` потокобезопасен | `Gson` потокобезопасен |
| Поддержка записей | Из коробки | Требует настройки |
| Полиморфизм | Полная поддержка | Ограниченная |
| Размер библиотеки | Больше | Меньше |

**Рекомендация:** для новых проектов предпочтительнее Jackson, поскольку он является стандартом де-факто в экосистеме Java (используется в Spring Boot, Quarkus и других фреймворках).

## 3. Примеры выполнения

### 3.1. Пример 1. Базовая сериализация и десериализация

```java
import com.fasterxml.jackson.annotation.*;
import com.fasterxml.jackson.databind.*;
import com.fasterxml.jackson.databind.json.JsonMapper;
import com.fasterxml.jackson.datatype.jsr310.JavaTimeModule;
import java.time.LocalDateTime;

/**
 * Запись, моделирующая пользователя
 */
@JsonInclude(JsonInclude.Include.NON_NULL)
public record User(
    String id,
    @JsonProperty("full_name") String name,
    String email,
    @JsonIgnore String password,
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss") LocalDateTime createdAt
) {}

/**
 * Демонстрация базовой сериализации и десериализации
 */
public class BasicSerializationDemo {
    public static void main(String[] args) throws Exception {
        // Создание и конфигурация ObjectMapper
        ObjectMapper mapper = JsonMapper.builder()
            .addModule(new JavaTimeModule())
            .disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS)
            .enable(SerializationFeature.INDENT_OUTPUT)
            .build();

        // Создание объекта
        User user = new User(
            "U001",
            "Иванов Иван Иванович",
            "ivanov@example.com",
            "secret123",
            LocalDateTime.of(2026, 8, 20, 14, 30)
        );

        // Сериализация
        String json = mapper.writeValueAsString(user);
        System.out.println("=== Сериализация ===");
        System.out.println(json);

        // Десериализация
        User restored = mapper.readValue(json, User.class);
        System.out.println("\n=== Десериализация ===");
        System.out.println("ID: " + restored.id());
        System.out.println("Имя: " + restored.name());
        System.out.println("Email: " + restored.email());
        System.out.println("Пароль: " + restored.password() + " (должен быть null)");
        System.out.println("Дата создания: " + restored.createdAt());

        // Десериализация с альтернативными именами
        String altJson = """
            {
              "id": "U002",
              "full_name": "Петров П.П.",
              "email": "petrov@example.com",
              "created_at": "2026-08-21 10:00:00"
            }
            """;
        User user2 = mapper.readValue(altJson, User.class);
        System.out.println("\n=== Десериализация альтернативного JSON ===");
        System.out.println("Имя: " + user2.name());
    }
}
```

### 3.2. Пример 2. Сложные структуры и коллекции

```java
import com.fasterxml.jackson.annotation.*;
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.*;
import com.fasterxml.jackson.datatype.jsr310.JavaTimeModule;
import java.time.LocalDate;
import java.util.*;

/**
 * Запись, моделирующая товар
 */
public record Product(
    String id,
    String name,
    double price,
    @JsonProperty("in_stock") boolean inStock
) {}

/**
 * Запись, моделирующая заказ
 */
public record Order(
    String id,
    @JsonProperty("customer_name") String customerName,
    @JsonProperty("customer_email") String customerEmail,
    List<Product> items,
    @JsonFormat(pattern = "yyyy-MM-dd") LocalDate orderDate,
    @JsonProperty("total_amount") double totalAmount,
    OrderStatus status
) {}

/**
 * Перечисление статусов заказа
 */
public enum OrderStatus {
    @JsonProperty("new") NEW,
    @JsonProperty("confirmed") CONFIRMED,
    @JsonProperty("shipped") SHIPPED,
    @JsonProperty("delivered") DELIVERED,
    @JsonProperty("cancelled") CANCELLED
}

/**
 * Демонстрация работы со сложными структурами
 */
public class ComplexStructureDemo {
    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = JsonMapper.builder()
            .addModule(new JavaTimeModule())
            .disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS)
            .enable(SerializationFeature.INDENT_OUTPUT)
            .build();

        // Создание заказа с несколькими товарами
        Order order = new Order(
            "ORD-001",
            "Иванов И.И.",
            "ivanov@example.com",
            List.of(
                new Product("P001", "Ноутбук", 85000.0, true),
                new Product("P002", "Мышь", 1500.0, true),
                new Product("P003", "Клавиатура", 3500.0, false)
            ),
            LocalDate.of(2026, 8, 20),
            90000.0,
            OrderStatus.CONFIRMED
        );

        // Сериализация
        String json = mapper.writeValueAsString(order);
        System.out.println("=== Сериализация заказа ===");
        System.out.println(json);

        // Десериализация
        Order restored = mapper.readValue(json, Order.class);
        System.out.println("\n=== Десериализация заказа ===");
        System.out.println("ID заказа: " + restored.id());
        System.out.println("Клиент: " + restored.customerName());
        System.out.println("Дата: " + restored.orderDate());
        System.out.println("Статус: " + restored.status());
        System.out.println("Товары:");
        for (Product p : restored.items()) {
            System.out.printf("  - %s: %.2f руб. (в наличии: %s)%n",
                p.name(), p.price(), p.inStock());
        }
        System.out.printf("Итого: %.2f руб.%n", restored.totalAmount());

        // Сериализация списка заказов
        List<Order> orders = List.of(order);
        String ordersJson = mapper.writeValueAsString(orders);
        System.out.println("\n=== Сериализация списка заказов ===");
        System.out.println(ordersJson);

        // Десериализация списка
        List<Order> restoredOrders = mapper.readValue(ordersJson,
            new TypeReference<List<Order>>() {});
        System.out.println("\nКоличество заказов: " + restoredOrders.size());

        // Сериализация карты
        Map<String, Double> prices = Map.of(
            "Ноутбук", 85000.0,
            "Мышь", 1500.0,
            "Клавиатура", 3500.0
        );
        String pricesJson = mapper.writeValueAsString(prices);
        System.out.println("\n=== Карта цен ===");
        System.out.println(pricesJson);
    }
}
```

### 3.3. Пример 3. Полиморфная десериализация

```java
import com.fasterxml.jackson.annotation.*;
import com.fasterxml.jackson.databind.*;
import com.fasterxml.jackson.datatype.jsr310.JavaTimeModule;

/**
 * Базовый абстрактный класс платёжного метода
 */
@JsonTypeInfo(
    use = JsonTypeInfo.Id.NAME,
    include = JsonTypeInfo.As.PROPERTY,
    property = "payment_type"
)
@JsonSubTypes({
    @JsonSubTypes.Type(value = CardPayment.class, name = "card"),
    @JsonSubTypes.Type(value = WalletPayment.class, name = "wallet"),
    @JsonSubTypes.Type(value = MobilePayment.class, name = "mobile")
})
public abstract class Payment {
    protected String id;
    protected double amount;
    protected String currency;

    // Конструктор по умолчанию необходим для десериализации
    protected Payment() {}

    protected Payment(String id, double amount, String currency) {
        this.id = id;
        this.amount = amount;
        this.currency = currency;
    }

    public String getId() { return id; }
    public double getAmount() { return amount; }
    public String getCurrency() { return currency; }

    public abstract String getPaymentType();

    @Override
    public String toString() {
        return String.format("%s[id=%s, amount=%.2f %s]",
            getClass().getSimpleName(), id, amount, currency);
    }
}

/**
 * Оплата банковской картой
 */
public class CardPayment extends Payment {
    private String cardNumber;
    private String cardHolder;
    private String expiryDate;

    public CardPayment() {}

    public CardPayment(String id, double amount, String currency,
                       String cardNumber, String cardHolder, String expiryDate) {
        super(id, amount, currency);
        this.cardNumber = cardNumber;
        this.cardHolder = cardHolder;
        this.expiryDate = expiryDate;
    }

    @Override
    public String getPaymentType() { return "card"; }

    public String getCardNumber() { return cardNumber; }
    public String getCardHolder() { return cardHolder; }
    public String getExpiryDate() { return expiryDate; }
}

/**
 * Оплата с электронного кошелька
 */
public class WalletPayment extends Payment {
    private String walletId;
    private String walletType;

    public WalletPayment() {}

    public WalletPayment(String id, double amount, String currency,
                         String walletId, String walletType) {
        super(id, amount, currency);
        this.walletId = walletId;
        this.walletType = walletType;
    }

    @Override
    public String getPaymentType() { return "wallet"; }

    public String getWalletId() { return walletId; }
    public String getWalletType() { return walletType; }
}

/**
 * Оплата через мобильный телефон
 */
public class MobilePayment extends Payment {
    private String phoneNumber;
    private String operator;

    public MobilePayment() {}

    public MobilePayment(String id, double amount, String currency,
                         String phoneNumber, String operator) {
        super(id, amount, currency);
        this.phoneNumber = phoneNumber;
        this.operator = operator;
    }

    @Override
    public String getPaymentType() { return "mobile"; }

    public String getPhoneNumber() { return phoneNumber; }
    public String getOperator() { return operator; }
}

/**
 * Демонстрация полиморфной десериализации
 */
public class PolymorphicDemo {
    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = JsonMapper.builder()
            .addModule(new JavaTimeModule())
            .enable(SerializationFeature.INDENT_OUTPUT)
            .build();

        // Создание платежей различных типов
        Payment card = new CardPayment("P001", 15000.0, "RUB",
            "****1234", "IVANOV I.I.", "12/28");
        Payment wallet = new WalletPayment("P002", 5000.0, "RUB",
            "W1234567890", "YooMoney");
        Payment mobile = new MobilePayment("P003", 1000.0, "RUB",
            "+79001234567", "МТС");

        // Сериализация
        System.out.println("=== Сериализация платежей ===");
        System.out.println("Карта:\n" + mapper.writeValueAsString(card));
        System.out.println("\nКошелёк:\n" + mapper.writeValueAsString(wallet));
        System.out.println("\nМобильный:\n" + mapper.writeValueAsString(mobile));

        // Десериализация с автоматическим определением типа
        String cardJson = """
            {
              "payment_type": "card",
              "id": "P004",
              "amount": 25000.0,
              "currency": "RUB",
              "cardNumber": "****5678",
              "cardHolder": "PETROV P.P.",
              "expiryDate": "06/27"
            }
            """;

        Payment restored = mapper.readValue(cardJson, Payment.class);
        System.out.println("\n=== Десериализация ===");
        System.out.println("Тип: " + restored.getClass().getSimpleName());
        System.out.println("Детали: " + restored);

        if (restored instanceof CardPayment cp) {
            System.out.println("Держатель карты: " + cp.getCardHolder());
            System.out.println("Срок действия: " + cp.getExpiryDate());
        }

        // Обработка ошибок
        System.out.println("\n=== Обработка ошибок ===");
        String invalidJson = """
            {
              "payment_type": "unknown",
              "id": "P005",
              "amount": 1000.0
            }
            """;
        try {
            mapper.readValue(invalidJson, Payment.class);
        } catch (Exception e) {
            System.out.println("Ошибка: " + e.getClass().getSimpleName());
            System.out.println("Сообщение: " + e.getMessage());
        }
    }
}
```

## 4. Задания на паре

### Задание 4.1. Сериализация системы бронирования

Разработайте систему сериализации для данных бронирования гостиницы.

**Записи:**

1. `Guest` (гость):
   - `id` (String), `fullName` (String), `email` (String), `phone` (String), `passportNumber` (String, `@JsonIgnore` при сериализации).

2. `Room` (номер):
   - `number` (String), `type` (RoomType — перечисление: STANDARD, SUPERIOR, LUXE, SUITE), `pricePerNight` (double), `capacity` (int), `amenities` (List<String>).

3. `Booking` (бронирование):
   - `bookingId` (String), `guest` (Guest), `room` (Room), `checkIn` (LocalDate), `checkOut` (LocalDate), `totalAmount` (double), `status` (BookingStatus — перечисление: PENDING, CONFIRMED, CHECKED_IN, CHECKED_OUT, CANCELLED).

**Требования к реализации:**

1. Реализуйте сериализацию одного бронирования в JSON с красивым форматированием.
2. Реализуйте десериализацию из JSON.
3. Реализуйте сериализацию списка бронирований.
4. Реализуйте десериализацию списка из JSON-файла (строки).
5. Примените аннотации `@JsonProperty`, `@JsonIgnore`, `@JsonFormat`, `@JsonInclude`.
6. Продемонстрируйте обработку ошибок десериализации (некорректный JSON, отсутствующие поля).

**Пример выполнения программы:**

```
=== Сериализация бронирования ===
{
  "booking_id" : "BK-001",
  "guest" : {
    "id" : "G001",
    "full_name" : "Иванов И.И.",
    "email" : "ivanov@example.com",
    "phone" : "+79001234567"
  },
  "room" : {
    "number" : "305",
    "type" : "LUXE",
    "price_per_night" : 8500.0,
    "capacity" : 2,
    "amenities" : [ "Wi-Fi", "Мини-бар", "Сейф" ]
  },
  "check_in" : "2026-08-20",
  "check_out" : "2026-08-27",
  "total_amount" : 59500.0,
  "status" : "CONFIRMED"
}

=== Десериализация ===
Бронирование: BK-001
Гость: Иванов И.И.
Номер: 305 (LUXE)
Период: 2026-08-20 — 2026-08-27
Сумма: 59500.0 руб.
```

---

### Задание 4.2. Полиморфная сериализация уведомлений

Разработайте систему сериализации уведомлений с поддержкой полиморфизма.

**Иерархия классов:**

Базовый абстрактный класс `Notification`:
- Поля: `id` (String), `recipient` (String), `message` (String), `createdAt` (LocalDateTime), `priority` (Priority — перечисление: LOW, NORMAL, HIGH, CRITICAL).
- Аннотации `@JsonTypeInfo` и `@JsonSubTypes` для поддержки полиморфизма.

Подклассы:
1. `EmailNotification`: `subject` (String), `cc` (List<String>), `attachments` (List<String>).
2. `SmsNotification`: `phoneNumber` (String), `maxLength` (int).
3. `PushNotification`: `title` (String), `applicationName` (String), `data` (Map<String, String>).
4. `WebhookNotification`: `url` (String), `method` (String), `headers` (Map<String, String>).

**Требования к реализации:**

1. Создайте список уведомлений различных типов.
2. Сериализуйте список в JSON (с полем-дискриминатором).
3. Десериализуйте список обратно, убедитесь в корректном восстановлении типов.
4. Продемонстрируйте обработку неизвестного типа уведомления.
5. Реализуйте метод `summarizeNotifications(List<Notification>)`, выводящий сводку по типам уведомлений.

---

### Задание 4.3. Работа с JSON API responses

Разработайте классы для работы с типичными ответами REST API.

**Структуры данных:**

1. `ApiResponse<T>` — обёртка для ответа API:
   - `success` (boolean), `data` (T), `error` (ApiError), `meta` (Map<String, Object>).

2. `ApiError`:
   - `code` (String), `message` (String), `details` (List<String>).

3. `PaginatedResponse<T>`:
   - `items` (List<T>), `page` (int), `pageSize` (int), `totalElements` (long), `totalPages` (int).

4. `User` (пример сущности):
   - `id` (String), `username` (String), `email` (String), `roles` (List<String>), `createdAt` (LocalDateTime).

**Требования к реализации:**

1. Реализуйте десериализацию ответа API с успешным результатом:
   ```json
   {
     "success": true,
     "data": { "id": "U001", "username": "ivanov", ... },
     "error": null,
     "meta": { "requestId": "req-123" }
   }
   ```

2. Реализуйте десериализацию ответа с ошибкой:
   ```json
   {
     "success": false,
     "data": null,
     "error": { "code": "USER_NOT_FOUND", "message": "...", "details": [...] }
   }
   ```

3. Реализуйте десериализацию пагинированного ответа.

4. Продемонстрируйте обработку всех сценариев с применением `Optional` и обработки исключений.

## 5. Задание для самостоятельной работы

Разработать систему классов для сериализации и десериализации согласно своему варианту. Требования:

1. Не менее 4 записей или классов, связанных между собой (вложенные структуры).
2. Не менее 1 перечисления с кастомной сериализацией.
3. Применение не менее 5 различных аннотаций Jackson.
4. Демонстрация работы с коллекциями (List, Map).
5. Обработка ошибок десериализации.
6. Демонстрационный класс с не менее чем 6 сценариями.

### Варианты заданий

**Вариант 1.** Система «Библиотека». Сериализация книг, авторов, читателей, выдач. Полиморфизм для типов изданий (книга, журнал, аудио).

**Вариант 2.** Система «Авиакомпания». Сериализация рейсов, пассажиров, бронирований. Различные типы билетов (эконом, бизнес, первый).

**Вариант 3.** Система «Банк». Сериализация счетов, клиентов, транзакций. Полиморфизм для типов транзакций (перевод, платёж, снятие).

**Вариант 4.** Система «Интернет-магазин». Сериализация товаров, заказов, покупателей. Полиморфизм для типов доставки (курьер, почта, самовывоз).

**Вариант 5.** Система «Больница». Сериализация пациентов, врачей, приёмов. Полиморфизм для типов медицинских услуг.

**Вариант 6.** Система «Университет». Сериализация студентов, курсов, оценок. Различные типы учебных занятий (лекция, практика, лабораторная).

**Вариант 7.** Система «Ресторан». Сериализация меню, заказов, столиков. Полиморфизм для типов блюд (закуска, основное, десерт, напиток).

**Вариант 8.** Система «Фитнес-клуб». Сериализация клиентов, абонементов, тренировок. Различные типы абонементов (месячный, годовой, VIP).

**Вариант 9.** Система «Автосервис». Сериализация заказов-нарядов, мастеров, услуг. Полиморфизм для типов работ (ТО, ремонт, диагностика).

**Вариант 10.** Система «Турагентство». Сериализация туров, клиентов, бронирований. Различные типы туров (пляжный, экскурсионный, круиз).

**Вариант 11.** Система «Кинотеатр». Сериализация фильмов, сеансов, билетов. Различные типы залов (стандартный, IMAX, 4DX).

**Вариант 12.** Система «Склад». Сериализация товаров, поставщиков, поступлений. Полиморфизм для типов товаров (сырьё, готовая продукция, упаковка).

**Вариант 13.** Система «Страховая компания». Сериализация полисов, клиентов, страховых случаев. Различные типы страхования (ОСАГО, КАСКО, жизнь).

**Вариант 14.** Система «Почта». Сериализация отправлений, отделений, маршрутов. Полиморфизм для типов отправлений (письмо, посылка, экспресс).

**Вариант 15.** Система «Агентство недвижимости». Сериализация объектов, клиентов, сделок. Различные типы объектов (квартира, дом, коммерческая).

**Вариант 16.** Система «Кадровое агентство». Сериализация кандидатов, вакансий, откликов. Полиморфизм для типов занятости (полная, частичная, удалённая).

**Вариант 17.** Система «Благотворительный фонд». Сериализация кампаний, пожертвований, благополучателей. Различные типы пожертвований (разовое, регулярное, целевое).

**Вариант 18.** Система «Такси». Сериализация заказов, водителей, автомобилей. Различные классы автомобилей (эконом, комфорт, бизнес).

**Вариант 19.** Система «Музей». Сериализация экспонатов, выставок, посетителей. Полиморфизм для типов экспонатов (живопись, скульптура, документ).

**Вариант 20.** Система «Спортивный клуб». Сериализация спортсменов, соревнований, результатов. Различные виды спорта с собственными правилами.

**Вариант 21.** Система «Почтовый клиент». Сериализация писем, контактов, папок. Различные типы писем (входящее, исходящее, черновик).

**Вариант 22.** Система «Календарь». Сериализация событий, напоминаний, участников. Полиморфизм для типов событий (встреча, день рождения, праздник).

**Вариант 23.** Система «Платёжная система». Сериализация merchants, платежей, возвратов. Различные методы оплаты (карта, кошелёк, мобильный).

**Вариант 24.** Система «Управление проектами». Сериализация проектов, задач, исполнителей. Полиморфизм для типов задач (разработка, дизайн, тестирование).

**Вариант 25.** Система «Ветеринарная клиника». Сериализация животных, владельцев, приёмов. Различные виды животных (собака, кошка, птица, экзотическое).

**Вариант 26.** Система «Железнодорожные перевозки». Сериализация поездов, вагонов, пассажиров. Различные типы вагонов (плацкарт, купе, СВ, люкс).

**Вариант 27.** Система «Языковые курсы». Сериализация студентов, групп, занятий. Различные уровни владения языком (A1, A2, B1, B2, C1, C2).

**Вариант 28.** Система «Фотостудия». Сериализация фотосессий, клиентов, фотографий. Полиморфизм для типов съёмок (портрет, свадьба, репортаж).

**Вариант 29.** Система «Коворкинг». Сериализация рабочих мест, резидентов, бронирований. Различные типы мест (рабочее место, переговорная, офис).

**Вариант 30.** Система «Платформа подкастов». Сериализация подкастов, эпизодов, подписчиков. Различные типы эпизодов (обычный, бонусный, трейлер).

## 6. Методические указания к самостоятельной работе

1. **Проектирование структуры JSON.** Перед реализацией:
   - определите иерархию классов и записей;
   - продумайте имена полей в JSON (snake_case или camelCase);
   - определите, какие поля должны быть исключены из сериализации;
   - спроектируйте формат дат и перечислений.

2. **Выбор между классом и записью.** Применяйте:
   - записи для неизменяемых носителей данных;
   - классы для сущностей с изменяемым состоянием или полиморфной иерархии.

3. **Применение аннотаций.** Используйте:
   - `@JsonProperty` для переименования полей;
   - `@JsonIgnore` для исключения конфиденциальных данных;
   - `@JsonFormat` для форматирования дат и чисел;
   - `@JsonInclude` для исключения `null`-значений;
   - `@JsonCreator` для десериализации через конструктор.

4. **Полиморфизм.** Для иерархий классов:
   - применяйте `@JsonTypeInfo` с указанием поля-дискриминатора;
   - перечислите все подклассы в `@JsonSubTypes`;
   - обеспечьте наличие конструктора по умолчанию в каждом подклассе.

5. **Обработка ошибок.** Реализуйте обработку:
   - `JsonParseException` — некорректный синтаксис;
   - `JsonMappingException` — несоответствие структуры;
   - `MismatchedInputException` — несоответствие типов;
   - `InvalidFormatException` — некорректный формат.

6. **Конфигурация `ObjectMapper`.** Создавайте один экземпляр `ObjectMapper` на приложение:
   - регистрируйте модуль `JavaTimeModule` для поддержки `java.time.*`;
   - отключайте `WRITE_DATES_AS_TIMESTAMPS` для читаемого формата дат;
   - включайте `INDENT_OUTPUT` только для отладки.

7. **Тестирование.** Перед сдачей работы проверьте:
   - корректность сериализации всех классов и записей;
   - корректность десериализации из JSON;
   - обработку ошибок и граничных случаев;
   - работу полиморфной десериализации;
   - отсутствие потерь данных при круговом преобразовании.

8. **Применение ИИ.** При использовании средств ИИ:
   - генерируйте по отдельности классы сущностей, конфигурацию `ObjectMapper` и демонстрационный класс;
   - обязательно проверяйте правильность аннотаций;
   - не делегируйте ИИ проектирование иерархий без понимания полиморфной десериализации.

9. **Оформление отчёта.** Отчёт должен содержать:
   - листинги всех файлов проекта с комментариями;
   - примеры JSON-документов для всех сценариев;
   - протокол работы демонстрационного класса;
   - обоснование выбора аннотаций;
   - ответы на контрольные вопросы;
   - выводы по проделанной работе.

## 7. Контрольные вопросы

1. Что такое сериализация и десериализация? Для каких целей они применяются?
2. Каковы основные конструкции формата JSON?
3. Какие типы данных поддерживает JSON?
4. Какие библиотеки для работы с JSON существуют в Java? В чём их различие?
5. Что такое `ObjectMapper` в библиотеке Jackson? Как его правильно конфигурировать?
6. Для чего применяются аннотации `@JsonProperty`, `@JsonIgnore`, `@JsonFormat`?
7. Как сериализуются записи (`record`) в JSON?
8. Как сериализуются перечисления (`enum`)? Как кастомизировать их представление?
9. Каким образом осуществляется работа с коллекциями (List, Map) в JSON?
10. Что такое полиморфная десериализация? Какие аннотации для неё применяются?
11. Какие исключения могут возникать при десериализации? Как их обрабатывать?
12. Как создать кастомный сериализатор или десериализатор?
13. Что такое `TypeReference` и для каких случаев он применяется?
14. Почему `ObjectMapper` рекомендуется создавать один раз и переиспользовать?
15. В каких случаях предпочтительнее Jackson, а в каких — Gson?

## 8. Рекомендуемые источники

1. Хорстманн К. *Java. Библиотека профессионала. Том 2.* — М.: Вильямс. — Разделы, посвящённые сериализации.
2. Официальная документация Jackson: https://github.com/FasterXML/jackson-docs
3. Baeldung. Jackson JSON Tutorial. URL: https://www.baeldung.com/jackson
4. Baeldung. Jackson Annotations. URL: https://www.baeldung.com/jackson-annotations
5. Baeldung. Jackson Polymorphic Deserialization. URL: https://www.baeldung.com/jackson-inheritance
6. Официальная документация Gson: https://github.com/google/gson/blob/main/UserGuide.md
7. JSON Specification. URL: https://www.json.org/json-ru.html
8. Oracle Java Tutorials. Lesson: Basic I/O. URL: https://docs.oracle.com/javase/tutorial/essential/io/
