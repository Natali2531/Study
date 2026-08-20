# Лабораторная работа №8. Исключения как часть объектно-ориентированного дизайна

## 1. Паспорт работы

| Параметр | Значение |
|----------|----------|
| Тема | Исключения как часть объектно-ориентированного дизайна |
| Номер занятия в модуле | 4 из 4 (модуль 2) |
| Продолжительность аудиторной части | 2 академических часа |
| Предшествующая подготовка | Модуль 1 (ООП), лабораторные работы №5–7 (перечисления, записи, обобщения, коллекции) |
| Тип работы | Формирование навыков проектирования иерархий исключений и надёжной обработки ошибок |

### 1.1. Цель работы

Освоить механизм обработки исключений в языке Java не как синтаксическую конструкцию, а как неотъемлемую часть объектно-ориентированного дизайна. Научиться проектировать иерархии исключений, отражающие семантику предметной области, применять проверяемые и непроверяемые исключения в соответствии с их назначением, использовать конструкцию `try-with-resources` для корректного освобождения ресурсов и применять принципы exception translation и exception chaining.

### 1.2. Задачи работы

1. Изучить иерархию классов исключений языка Java (`Throwable`, `Error`, `Exception`, `RuntimeException`).
2. Освоить различие между проверяемыми (checked) и непроверяемыми (unchecked) исключениями.
3. Научиться проектировать иерархии кастомных исключений, отражающие семантику предметной области.
4. Освоить конструкции `try`, `catch`, `finally`, `try-with-resources`.
5. Изучить механизм множественного перехвата исключений (multi-catch) и повторного выбрасывания (rethrow).
6. Освоить принципы exception translation и exception chaining.
7. Изучить интерфейс `AutoCloseable` и его применение для управления ресурсами.
8. Сформировать понимание лучших практик обработки исключений.

### 1.3. Оснащение

- JDK версии 17 или выше;
- интегрированная среда разработки IntelliJ IDEA Community Edition;
- система сборки Maven или Gradle;
- система контроля версий Git;
- система модульного тестирования JUnit 5 (рекомендуется).

## 2. Теоретические сведения

### 2.1. Мотивация: почему необходимы исключения

В лабораторных работах №1–7 для сигнализации об ошибках применялись стандартные исключения (`IllegalArgumentException`, `IllegalStateException`, `IndexOutOfBoundsException`). Однако в реальных программных системах возникает необходимость в более тонкой дифференциации ошибок:

1. **Различные причины ошибок.** Недостаточно средств, неверный номер счёта, истёк срок действия — все эти ситуации требуют различной обработки.
2. **Контекст ошибки.** Ошибка должна содержать информацию, достаточную для диагностики (идентификаторы, значения полей, время возникновения).
3. **Группировка ошибок.** Код должен иметь возможность перехватывать группы связанных ошибок единым блоком `catch`.
4. **Семантика предметной области.** Исключения должны отражать бизнес-логику системы, а не только технические проблемы.

Механизм исключений и возможность создания собственных классов исключений позволяют решить указанные задачи.

### 2.2. Иерархия исключений Java

Все классы исключений в языке Java образуют иерархию, корнем которой является класс `java.lang.Throwable`:

```
Throwable
    ├── Error (серьёзные системные ошибки)
    │       ├── OutOfMemoryError
    │       ├── StackOverflowError
    │       └── ...
    └── Exception (исключения приложения)
            ├── IOException (проверяемые)
            ├── SQLException (проверяемые)
            └── RuntimeException (непроверяемые)
                    ├── NullPointerException
                    ├── IllegalArgumentException
                    ├── IllegalStateException
                    ├── IndexOutOfBoundsException
                    └── ...
```

**Ключевые категории:**

| Категория | Проверяемость | Назначение | Действия программы |
|-----------|:-------------:|------------|---------------------|
| `Error` | Нет | Серьёзные системные проблемы | Не перехватывать, завершение работы |
| `Exception` (не `RuntimeException`) | Да | Ожидаемые внешние проблемы | Обязательная обработка или объявление |
| `RuntimeException` | Нет | Ошибки логики программы | Исправление кода, редко — перехват |

### 2.3. Проверяемые и непроверяемые исключения

**Проверяемые исключения (checked)** — наследники `Exception`, не являющиеся наследниками `RuntimeException`. Компилятор требует их обязательной обработки: либо в блоке `try-catch`, либо в сигнатуре метода через `throws`.

Примеры: `IOException`, `SQLException`, `ClassNotFoundException`.

**Непроверяемые исключения (unchecked)** — наследники `RuntimeException` и `Error`. Компилятор не требует их обработки.

Примеры: `NullPointerException`, `IllegalArgumentException`, `IllegalStateException`.

**Правило выбора:**
- **Checked** — для ситуаций, которые программа может и должна обработать (ошибки ввода-вывода, сетевые проблемы, проблемы с базой данных);
- **Unchecked** — для ситуаций, отражающих ошибки логики программы (нарушение контракта метода, некорректные аргументы).

### 2.4. Конструкции обработки исключений

#### 2.4.1. Базовая конструкция `try-catch-finally`

```java
try {
    // Код, который может выбросить исключение
    riskyOperation();
} catch (SpecificException e) {
    // Обработка конкретного типа исключения
    log.error("Ошибка: " + e.getMessage());
} catch (AnotherException e) {
    // Обработка другого типа
    throw new WrapperException("Не удалось выполнить операцию", e);
} finally {
    // Выполняется всегда (кроме System.exit)
    // Применяется для освобождения ресурсов
    cleanup();
}
```

#### 2.4.2. Множественный перехват (multi-catch)

Начиная с Java 7, несколько типов исключений могут быть перехвачены одним блоком `catch`:

```java
try {
    processFile(path);
} catch (IOException | SQLException e) {
    // Один блок для нескольких типов
    log.error("Ошибка доступа к данным: " + e.getMessage());
}
```

Переменная `e` в multi-catch неявно имеет тип `final` и является наибольшим общим супертипом перечисленных исключений.

#### 2.4.3. Конструкция `try-with-resources`

Начиная с Java 7, для автоматического освобождения ресурсов применяется конструкция `try-with-resources`. Ресурс должен реализовывать интерфейс `java.lang.AutoCloseable`:

```java
// Ресурс автоматически закрывается после выхода из try
try (BufferedReader reader = new BufferedReader(new FileReader("file.txt"))) {
    String line;
    while ((line = reader.readLine()) != null) {
        System.out.println(line);
    }
} catch (IOException e) {
    log.error("Ошибка чтения файла", e);
}
// reader.close() вызывается автоматически, даже если возникло исключение
```

Можно объявлять несколько ресурсов:

```java
try (InputStream in = new FileInputStream("input.dat");
     OutputStream out = new FileOutputStream("output.dat")) {
    // Работа с ресурсами
} catch (IOException e) {
    log.error("Ошибка копирования файла", e);
}
```

**Подавленные исключения (suppressed exceptions).** Если в блоке `try` возникает исключение, а при закрытии ресурса — другое, второе исключение не теряется, а добавляется как «подавленное»:

```java
try (Resource r = new Resource()) {
    throw new BusinessException("Основная ошибка");
} catch (BusinessException e) {
    // e.getSuppressed() содержит исключения, возникшие при закрытии ресурса
    for (Throwable suppressed : e.getSuppressed()) {
        log.warn("Подавленное исключение: " + suppressed);
    }
}
```

### 2.5. Проектирование иерархий исключений

При разработке предметно-ориентированных приложений рекомендуется создавать собственную иерархию исключений, отражающую семантику предметной области.

#### 2.5.1. Базовое исключение предметной области

Каждая предметная область должна иметь базовое исключение, от которого наследуются все специализированные исключения:

```java
/**
 * Базовое исключение для банковской системы.
 * Является checked-исключением, так как операции с банком
 * могут завершаться ожидаемыми ошибками.
 */
public class BankingException extends Exception {
    public BankingException(String message) {
        super(message);
    }

    public BankingException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

#### 2.5.2. Специализированные исключения

Дочерние исключения отражают конкретные проблемные ситуации:

```java
/**
 * Исключение: недостаточно средств на счёте
 */
public class InsufficientFundsException extends BankingException {
    private final String accountNumber;
    private final double balance;
    private final double requestedAmount;

    public InsufficientFundsException(String accountNumber,
                                       double balance,
                                       double requestedAmount) {
        super(String.format(
            "Недостаточно средств на счёте %s. Баланс: %.2f, запрошено: %.2f",
            accountNumber, balance, requestedAmount));
        this.accountNumber = accountNumber;
        this.balance = balance;
        this.requestedAmount = requestedAmount;
    }

    public String getAccountNumber() { return accountNumber; }
    public double getBalance() { return balance; }
    public double getRequestedAmount() { return requestedAmount; }
    public double getShortage() { return requestedAmount - balance; }
}

/**
 * Исключение: счёт не найден
 */
public class AccountNotFoundException extends BankingException {
    private final String accountNumber;

    public AccountNotFoundException(String accountNumber) {
        super("Счёт не найден: " + accountNumber);
        this.accountNumber = accountNumber;
    }

    public String getAccountNumber() { return accountNumber; }
}

/**
 * Исключение: счёт заблокирован
 */
public class AccountBlockedException extends BankingException {
    private final String accountNumber;
    private final String reason;

    public AccountBlockedException(String accountNumber, String reason) {
        super(String.format("Счёт %s заблокирован: %s", accountNumber, reason));
        this.accountNumber = accountNumber;
        this.reason = reason;
    }

    public String getAccountNumber() { return accountNumber; }
    public String getReason() { return reason; }
}
```

#### 2.5.3. Применение иерархии

```java
public class BankService {
    public void withdraw(String accountNumber, double amount)
            throws BankingException {
        Account account = findAccount(accountNumber);
        if (account == null) {
            throw new AccountNotFoundException(accountNumber);
        }
        if (account.isBlocked()) {
            throw new AccountBlockedException(accountNumber,
                                              account.getBlockReason());
        }
        if (account.getBalance() < amount) {
            throw new InsufficientFundsException(
                accountNumber, account.getBalance(), amount);
        }
        account.withdraw(amount);
    }
}

// Обработка с группировкой по иерархии
try {
    bankService.withdraw("1234567890", 5000.0);
} catch (InsufficientFundsException e) {
    System.out.println("Не хватает: " + e.getShortage() + " руб.");
} catch (AccountBlockedException e) {
    System.out.println("Счёт заблокирован: " + e.getReason());
} catch (AccountNotFoundException e) {
    System.out.println("Счёт не найден");
} catch (BankingException e) {
    // Перехват всех банковских исключений
    System.out.println("Ошибка банковской операции: " + e.getMessage());
}
```

### 2.6. Exception Translation и Exception Chaining

**Exception translation** — преобразование низкоуровневого исключения в высокоуровневое, соответствующее абстракции текущего уровня:

```java
public class UserRepository {
    private final DataSource dataSource;

    public User findById(String id) throws RepositoryException {
        try (Connection conn = dataSource.getConnection();
             PreparedStatement stmt = conn.prepareStatement(
                 "SELECT * FROM users WHERE id = ?")) {
            stmt.setString(1, id);
            try (ResultSet rs = stmt.executeQuery()) {
                if (rs.next()) {
                    return mapUser(rs);
                }
                throw new EntityNotFoundException("User", id);
            }
        } catch (SQLException e) {
            // Преобразование SQLException в RepositoryException
            throw new RepositoryException(
                "Не удалось найти пользователя: " + id, e);
        }
    }
}
```

**Exception chaining** — сохранение исходного исключения как причины (`cause`) нового исключения. Механизм обеспечивается конструктором `(String message, Throwable cause)`.

### 2.7. Интерфейс `AutoCloseable`

Интерфейс `AutoCloseable` определяет единственный метод `close()`, который может выбрасывать `Exception`:

```java
public interface AutoCloseable {
    void close() throws Exception;
}
```

Любой класс, реализующий `AutoCloseable`, может использоваться в конструкции `try-with-resources`. Классический пример — `InputStream`, `OutputStream`, `Connection`, `Statement`.

Собственные ресурсы:

```java
public class DatabaseConnection implements AutoCloseable {
    private final String url;
    private boolean closed = false;

    public DatabaseConnection(String url) {
        this.url = url;
        System.out.println("Соединение открыто: " + url);
    }

    public void execute(String query) {
        if (closed) {
            throw new IllegalStateException("Соединение закрыто");
        }
        System.out.println("Выполнен запрос: " + query);
    }

    @Override
    public void close() {
        if (!closed) {
            closed = true;
            System.out.println("Соединение закрыто: " + url);
        }
    }
}
```

### 2.8. Лучшие практики обработки исключений

1. **Перехватывайте конкретные типы.** Избегайте перехвата `Exception` или `Throwable` без крайней необходимости.

2. **Не «проглатывайте» исключения.** Пустой блок `catch` — признак плохого кода:
   ```java
   // ПЛОХО
   try { ... } catch (Exception e) { }
   ```

3. **Не используйте исключения для управления потоком.** Исключения должны отражать действительно исключительные ситуации.

4. **Документируйте выбрасываемые исключения.** Используйте Javadoc-тег `@throws`:
   ```java
   /**
    * Снимает средства со счёта.
    *
    * @throws InsufficientFundsException если недостаточно средств
    * @throws AccountNotFoundException если счёт не найден
    */
   public void withdraw(...) throws BankingException { ... }
   ```

5. **Используйте `finally` только для освобождения ресурсов.** В остальных случаях предпочитайте `try-with-resources`.

6. **Сохраняйте причину исключения.** При преобразовании исключений передавайте исходное как `cause`.

7. **Выбирайте правильный тип исключения:**
   - нарушение контракта метода → `IllegalArgumentException`, `IllegalStateException`;
   - ожидаемая бизнес-ошибка → собственное checked-исключение;
   - системная проблема → собственное unchecked-исключение.

8. **Включайте контекст в сообщение.** Сообщение исключения должно содержать достаточно информации для диагностики (идентификаторы, значения, время).

## 3. Примеры выполнения

### 3.1. Пример 1. Иерархия исключений для системы обработки заказов

```java
/**
 * Базовое исключение для системы обработки заказов
 */
public class OrderProcessingException extends Exception {
    private final String orderId;

    public OrderProcessingException(String orderId, String message) {
        super(message);
        this.orderId = orderId;
    }

    public OrderProcessingException(String orderId, String message, Throwable cause) {
        super(message, cause);
        this.orderId = orderId;
    }

    public String getOrderId() { return orderId; }
}

/**
 * Товар не найден в каталоге
 */
public class ProductNotFoundException extends OrderProcessingException {
    private final String productId;

    public ProductNotFoundException(String orderId, String productId) {
        super(orderId, "Товар не найден: " + productId);
        this.productId = productId;
    }

    public String getProductId() { return productId; }
}

/**
 * Товар отсутствует на складе
 */
public class OutOfStockException extends OrderProcessingException {
    private final String productId;
    private final int availableQuantity;
    private final int requestedQuantity;

    public OutOfStockException(String orderId, String productId,
                                int availableQuantity, int requestedQuantity) {
        super(orderId, String.format(
            "Товар %s отсутствует на складе. Доступно: %d, запрошено: %d",
            productId, availableQuantity, requestedQuantity));
        this.productId = productId;
        this.availableQuantity = availableQuantity;
        this.requestedQuantity = requestedQuantity;
    }

    public String getProductId() { return productId; }
    public int getAvailableQuantity() { return availableQuantity; }
    public int getRequestedQuantity() { return requestedQuantity; }
    public int getShortage() { return requestedQuantity - availableQuantity; }
}

/**
 * Оплата не прошла
 */
public class PaymentFailedException extends OrderProcessingException {
    private final String paymentMethod;
    private final String errorCode;

    public PaymentFailedException(String orderId, String paymentMethod,
                                   String errorCode, Throwable cause) {
        super(orderId, String.format(
            "Оплата не прошла. Метод: %s, код ошибки: %s",
            paymentMethod, errorCode), cause);
        this.paymentMethod = paymentMethod;
        this.errorCode = errorCode;
    }

    public String getPaymentMethod() { return paymentMethod; }
    public String getErrorCode() { return errorCode; }
}

/**
 * Заказ уже находится в терминальном состоянии
 */
public class InvalidOrderStateException extends OrderProcessingException {
    private final String currentState;
    private final String requestedAction;

    public InvalidOrderStateException(String orderId, String currentState,
                                       String requestedAction) {
        super(orderId, String.format(
            "Недопустимое действие '%s' для заказа в состоянии '%s'",
            requestedAction, currentState));
        this.currentState = currentState;
        this.requestedAction = requestedAction;
    }

    public String getCurrentState() { return currentState; }
    public String getRequestedAction() { return requestedAction; }
}
```

Сервис обработки заказов:

```java
public class OrderService {
    private final Map<String, Integer> stock = new java.util.HashMap<>();
    private final Map<String, String> orderStates = new java.util.HashMap<>();

    public OrderService() {
        stock.put("PROD-001", 10);
        stock.put("PROD-002", 5);
    }

    public void placeOrder(String orderId, String productId, int quantity)
            throws OrderProcessingException {
        // Проверка наличия товара
        if (!stock.containsKey(productId)) {
            throw new ProductNotFoundException(orderId, productId);
        }
        // Проверка остатка
        int available = stock.get(productId);
        if (available < quantity) {
            throw new OutOfStockException(orderId, productId, available, quantity);
        }
        // Уменьшение остатка
        stock.put(productId, available - quantity);
        orderStates.put(orderId, "NEW");
        System.out.println("Заказ " + orderId + " размещён");
    }

    public void cancelOrder(String orderId) throws OrderProcessingException {
        String state = orderStates.get(orderId);
        if (state == null) {
            throw new OrderProcessingException(orderId, "Заказ не найден");
        }
        if (!"NEW".equals(state)) {
            throw new InvalidOrderStateException(orderId, state, "cancel");
        }
        orderStates.put(orderId, "CANCELLED");
        System.out.println("Заказ " + orderId + " отменён");
    }
}
```

Демонстрационный класс:

```java
public class OrderServiceDemo {
    public static void main(String[] args) {
        OrderService service = new OrderService();

        // Успешное размещение заказа
        try {
            service.placeOrder("ORD-001", "PROD-001", 3);
        } catch (OrderProcessingException e) {
            System.out.println("Ошибка: " + e.getMessage());
        }

        // Попытка заказать отсутствующий товар
        try {
            service.placeOrder("ORD-002", "PROD-999", 1);
        } catch (ProductNotFoundException e) {
            System.out.println("Товар не найден: " + e.getProductId());
        } catch (OrderProcessingException e) {
            System.out.println("Ошибка: " + e.getMessage());
        }

        // Попытка заказать больше, чем есть на складе
        try {
            service.placeOrder("ORD-003", "PROD-002", 10);
        } catch (OutOfStockException e) {
            System.out.println("Не хватает " + e.getShortage() + " ед. товара " +
                               e.getProductId());
        } catch (OrderProcessingException e) {
            System.out.println("Ошибка: " + e.getMessage());
        }

        // Попытка отменить уже отменённый заказ
        try {
            service.cancelOrder("ORD-001");
            service.cancelOrder("ORD-001");   // повторная отмена
        } catch (InvalidOrderStateException e) {
            System.out.println("Нельзя " + e.getRequestedAction() +
                               ": заказ в состоянии " + e.getCurrentState());
        } catch (OrderProcessingException e) {
            System.out.println("Ошибка: " + e.getMessage());
        }

        // Группировка по базовому исключению
        System.out.println("\n=== Обработка всех ошибок заказа ===");
        String[] operations = {"ORD-001", "ORD-999"};
        for (String id : operations) {
            try {
                service.cancelOrder(id);
            } catch (OrderProcessingException e) {
                System.out.println("[" + e.getClass().getSimpleName() + "] " +
                                   e.getMessage());
            }
        }
    }
}
```

### 3.2. Пример 2. Try-with-resources для собственного ресурса

```java
/**
 * Ресурс, моделирующий транзакцию в базе данных.
 * Реализует AutoCloseable для использования в try-with-resources.
 */
public class Transaction implements AutoCloseable {
    private final String transactionId;
    private final List<String> operations = new java.util.ArrayList<>();
    private boolean committed = false;
    private boolean rolledBack = false;
    private boolean closed = false;

    public Transaction(String transactionId) {
        this.transactionId = transactionId;
        System.out.println("[TX " + transactionId + "] Транзакция начата");
    }

    public void addOperation(String operation) {
        checkState();
        operations.add(operation);
        System.out.println("[TX " + transactionId + "] Операция: " + operation);
    }

    public void commit() {
        checkState();
        committed = true;
        System.out.println("[TX " + transactionId + "] Зафиксировано операций: " +
                           operations.size());
    }

    public void rollback() {
        if (closed || committed) return;
        rolledBack = true;
        System.out.println("[TX " + transactionId + "] Откат. Операций отменено: " +
                           operations.size());
    }

    @Override
    public void close() {
        if (closed) return;
        closed = true;
        if (!committed && !rolledBack) {
            rollback();
        }
        System.out.println("[TX " + transactionId + "] Транзакция закрыта");
    }

    private void checkState() {
        if (closed) {
            throw new IllegalStateException(
                "Транзакция " + transactionId + " уже закрыта");
        }
        if (committed) {
            throw new IllegalStateException(
                "Транзакция " + transactionId + " уже зафиксирована");
        }
        if (rolledBack) {
            throw new IllegalStateException(
                "Транзакция " + transactionId + " уже откачена");
        }
    }

    public String getTransactionId() { return transactionId; }
}

/**
 * Сервис, использующий транзакции
 */
public class AccountService {
    public void transfer(String from, String to, double amount)
            throws BankingException {
        try (Transaction tx = new Transaction("TX-" + System.currentTimeMillis())) {
            tx.addOperation("Списание " + amount + " со счёта " + from);
            if (amount <= 0) {
                throw new IllegalArgumentException("Сумма должна быть положительной");
            }
            tx.addOperation("Зачисление " + amount + " на счёт " + to);
            tx.commit();
        } catch (IllegalArgumentException e) {
            // Транзакция автоматически откатится при выходе из try
            throw new BankingException("Ошибка перевода: " + e.getMessage(), e);
        }
    }
}
```

Демонстрационный класс:

```java
public class TransactionDemo {
    public static void main(String[] args) {
        AccountService service = new AccountService();

        // Успешный перевод
        try {
            service.transfer("ACC-001", "ACC-002", 1000.0);
        } catch (BankingException e) {
            System.out.println("Ошибка: " + e.getMessage());
        }

        System.out.println();

        // Перевод с ошибкой — транзакция автоматически откатится
        try {
            service.transfer("ACC-001", "ACC-002", -500.0);
        } catch (BankingException e) {
            System.out.println("Ошибка: " + e.getMessage());
            if (e.getCause() != null) {
                System.out.println("Причина: " + e.getCause().getMessage());
            }
        }
    }
}
```

### 3.3. Пример 3. Exception Translation для репозитория

```java
import java.util.*;

/**
 * Базовое исключение репозитория
 */
public class RepositoryException extends Exception {
    public RepositoryException(String message) { super(message); }
    public RepositoryException(String message, Throwable cause) { super(message, cause); }
}

/**
 * Сущность не найдена
 */
public class EntityNotFoundException extends RepositoryException {
    private final String entityType;
    private final String entityId;

    public EntityNotFoundException(String entityType, String entityId) {
        super(entityType + " с идентификатором " + entityId + " не найден");
        this.entityType = entityType;
        this.entityId = entityId;
    }

    public String getEntityType() { return entityType; }
    public String getEntityId() { return entityId; }
}

/**
 * Нарушение уникальности
 */
public class DuplicateEntityException extends RepositoryException {
    private final String entityType;
    private final String uniqueField;

    public DuplicateEntityException(String entityType, String uniqueField) {
        super(entityType + " с таким значением поля " + uniqueField + " уже существует");
        this.entityType = entityType;
        this.uniqueField = uniqueField;
    }

    public String getEntityType() { return entityType; }
    public String getUniqueField() { return uniqueField; }
}

/**
 * Запись (record) — пользователь
 */
public record User(String id, String email, String name) {
    public User {
        Objects.requireNonNull(id, "ID не может быть null");
        Objects.requireNonNull(email, "Email не может быть null");
        Objects.requireNonNull(name, "Имя не может быть null");
        if (!email.contains("@")) {
            throw new IllegalArgumentException("Некорректный email: " + email);
        }
    }
}

/**
 * Репозиторий пользователей с exception translation
 */
public class UserRepository {
    private final Map<String, User> storage = new HashMap<>();
    private final Map<String, String> emailIndex = new HashMap<>();

    public void save(User user) throws RepositoryException {
        // Имитация низкоуровневой ошибки
        if (user == null) {
            throw new NullPointerException("Пользователь не может быть null");
        }
        // Проверка уникальности email
        if (emailIndex.containsKey(user.email()) &&
            !emailIndex.get(user.email()).equals(user.id())) {
            throw new DuplicateEntityException("User", "email");
        }
        // Проверка уникальности ID
        if (storage.containsKey(user.id())) {
            throw new DuplicateEntityException("User", "id");
        }
        storage.put(user.id(), user);
        emailIndex.put(user.email(), user.id());
    }

    public User findById(String id) throws RepositoryException {
        User user = storage.get(id);
        if (user == null) {
            throw new EntityNotFoundException("User", id);
        }
        return user;
    }

    public User findByEmail(String email) throws RepositoryException {
        String id = emailIndex.get(email);
        if (id == null) {
            throw new EntityNotFoundException("User", email);
        }
        return storage.get(id);
    }

    public void delete(String id) throws RepositoryException {
        User user = findById(id);
        storage.remove(id);
        emailIndex.remove(user.email());
    }
}

/**
 * Сервисный уровень, выполняющий exception translation
 */
public class UserService {
    private final UserRepository repository = new UserRepository();

    public void registerUser(String id, String email, String name)
            throws UserRegistrationException {
        try {
            User user = new User(id, email, name);
            repository.save(user);
        } catch (IllegalArgumentException e) {
            // Преобразование в бизнес-исключение
            throw new UserRegistrationException("Некорректные данные пользователя", e);
        } catch (DuplicateEntityException e) {
            throw new UserRegistrationException(
                "Пользователь с таким " + e.getUniqueField() + " уже существует", e);
        } catch (RepositoryException e) {
            throw new UserRegistrationException("Ошибка регистрации", e);
        }
    }

    public User getUser(String id) throws UserNotFoundException {
        try {
            return repository.findById(id);
        } catch (EntityNotFoundException e) {
            throw new UserNotFoundException(id, e);
        } catch (RepositoryException e) {
            throw new UserNotFoundException(id, e);
        }
    }
}

/**
 * Исключение регистрации пользователя (unchecked — ошибка клиента)
 */
public class UserRegistrationException extends RuntimeException {
    public UserRegistrationException(String message) { super(message); }
    public UserRegistrationException(String message, Throwable cause) { super(message, cause); }
}

/**
 * Исключение «пользователь не найден» (unchecked)
 */
public class UserNotFoundException extends RuntimeException {
    private final String userId;

    public UserNotFoundException(String userId) {
        super("Пользователь не найден: " + userId);
        this.userId = userId;
    }

    public UserNotFoundException(String userId, Throwable cause) {
        super("Пользователь не найден: " + userId, cause);
        this.userId = userId;
    }

    public String getUserId() { return userId; }
}
```

Демонстрационный класс:

```java
public class UserServiceDemo {
    public static void main(String[] args) {
        UserService service = new UserService();

        // Регистрация пользователей
        try {
            service.registerUser("U001", "ivanov@example.com", "Иванов И.И.");
            service.registerUser("U002", "petrov@example.com", "Петров П.П.");
            System.out.println("Пользователи зарегистрированы");
        } catch (UserRegistrationException e) {
            System.out.println("Ошибка регистрации: " + e.getMessage());
        }

        // Попытка зарегистрировать с дублирующимся email
        try {
            service.registerUser("U003", "ivanov@example.com", "Сидоров С.С.");
        } catch (UserRegistrationException e) {
            System.out.println("Ошибка: " + e.getMessage());
            if (e.getCause() != null) {
                System.out.println("Причина: " + e.getCause().getMessage());
            }
        }

        // Попытка зарегистрировать с некорректным email
        try {
            service.registerUser("U004", "invalid-email", "Тестов Т.Т.");
        } catch (UserRegistrationException e) {
            System.out.println("Ошибка: " + e.getMessage());
        }

        // Поиск существующего пользователя
        try {
            User user = service.getUser("U001");
            System.out.println("Найден: " + user);
        } catch (UserNotFoundException e) {
            System.out.println("Ошибка: " + e.getMessage());
        }

        // Поиск несуществующего пользователя
        try {
            User user = service.getUser("U999");
        } catch (UserNotFoundException e) {
            System.out.println("Ошибка: " + e.getMessage());
            System.out.println("ID пользователя: " + e.getUserId());
        }
    }
}
```

## 4. Задания на паре

### Задание 4.1. Иерархия исключений для системы управления файлами

Разработайте иерархию исключений для системы управления файлами и примените её в файловом сервисе.

**Базовое исключение `FileSystemException`** (checked):
- Поля: `path` (String) — путь к файлу.
- Конструкторы с сообщением и причиной.

**Специализированные исключения:**

1. `FileNotFoundException` (наследует `FileSystemException`):
   - Файл не найден по указанному пути.

2. `AccessDeniedException` (наследует `FileSystemException`):
   - Отказано в доступе к файлу.
   - Дополнительное поле: `requiredPermission` (String).

3. `DirectoryNotEmptyException` (наследует `FileSystemException`):
   - Попытка удалить непустую директорию.

4. `InvalidPathException` (наследует `RuntimeException`, не `FileSystemException`):
   - Некорректный формат пути (unchecked — ошибка клиента).
   - Дополнительное поле: `invalidPart` (String).

5. `DiskFullException` (наследует `FileSystemException`):
   - Недостаточно места на диске.
   - Дополнительные поля: `requiredBytes`, `availableBytes`.

**Класс `FileService`:**
- Методы:
  - `readFile(String path)` — читает файл (может выбросить `FileNotFoundException`, `AccessDeniedException`);
  - `writeFile(String path, String content)` — записывает в файл (может выбросить `DiskFullException`, `AccessDeniedException`);
  - `deleteFile(String path)` — удаляет файл (может выбросить `FileNotFoundException`, `DirectoryNotEmptyException`);
  - `copyFile(String source, String destination)` — копирует файл.

**Демонстрация:**
- Смоделируйте различные исключительные ситуации.
- Продемонстрируйте обработку на разных уровнях иерархии.
- Покажите работу `InvalidPathException` как unchecked-исключения.

---

### Задание 4.2. Try-with-resources для пула соединений

Разработайте собственный пул соединений, реализующий `AutoCloseable`, и продемонстрируйте применение `try-with-resources`.

**Интерфейс `Connection`** (расширяет `AutoCloseable`):
- Методы: `executeQuery(String sql)`, `executeUpdate(String sql)`, `close()`.

**Класс `PooledConnection`** (реализует `Connection`):
- Поля: `id` (int), `pool` (ссылка на пул), `inUse` (boolean).
- Метод `close()` возвращает соединение в пул, а не закрывает физически.

**Класс `ConnectionPool`** (реализует `AutoCloseable`):
- Поля: `maxSize` (int), `availableConnections` (Queue<PooledConnection>), `inUseConnections` (Set<PooledConnection>).
- Методы:
  - `acquireConnection()` — получает соединение из пула (блокирует, если все заняты);
  - `releaseConnection(PooledConnection conn)` — возвращает соединение в пул;
  - `close()` — закрывает все соединения и помечает пул как закрытый.

**Класс `DatabaseService`:**
- Метод `executeInTransaction(List<String> queries)` — выполняет список запросов в транзакции, применяя `try-with-resources` для соединения.

**Демонстрация:**
- Создайте пул из 3 соединений.
- Выполните несколько операций с получением и возвратом соединений.
- Продемонстрируйте автоматический возврат соединения при выходе из `try-with-resources`.
- Продемонстрируйте закрытие пула.

---

### Задание 4.3. Комбинированное задание «Платёжный шлюз»

Разработайте систему обработки платежей с полной иерархией исключений, exception translation и `try-with-resources`.

**Иерархия исключений:**

Базовое исключение `PaymentException` (checked):
- Поля: `transactionId` (String), `amount` (double), `currency` (String).

Специализированные исключения:
1. `InvalidCardException` — некорректный номер карты.
2. `CardExpiredException` — срок действия карты истёк.
3. `InsufficientFundsException` — недостаточно средств.
4. `FraudDetectedException` — подозрение на мошенничество.
5. `PaymentGatewayUnavailableException` — шлюз недоступен.
6. `CurrencyNotSupportedException` — валюта не поддерживается (unchecked).

**Класс `PaymentGateway`** (реализует `AutoCloseable`):
- Имитирует соединение с платёжным шлюзом.
- Метод `processPayment(PaymentRequest request)` — обрабатывает платёж.
- Метод `close()` — закрывает соединение.

**Класс `PaymentService`:**
- Метод `pay(String cardNumber, double amount, String currency)` — выполняет платёж, применяя `try-with-resources` для шлюза и выполняя exception translation.

**Демонстрация:**
- Выполните не менее 6 платежей с различными сценариями (успех, отказ, мошенничество, недоступность шлюза, некорректная карта).
- Продемонстрируйте обработку на разных уровнях иерархии.
- Покажите exception chaining с сохранением причины.

## 5. Задание для самостоятельной работы

Разработать систему классов с полной иерархией исключений согласно своему варианту. Требования:

1. Базовое checked-исключение предметной области.
2. Не менее 4 специализированных исключений, наследующих от базового.
3. Как минимум одно unchecked-исключение для ошибок клиента.
4. Специализированные исключения должны содержать контекстные поля (идентификаторы, значения, причины) и соответствующие геттеры.
5. Применение `try-with-resources` для управления ресурсами.
6. Применение exception translation между слоями (репозиторий → сервис).
7. Применение exception chaining с сохранением причины.
8. Демонстрационный класс с не менее чем 6 сценариями, включая успешные и ошибочные.

### Варианты заданий

**Вариант 1.** Система «Электронная почта». Исключения: некорректный адрес, адресат не найден, ящик переполнен, спам-фильтр сработал, сервер недоступен, превышен лимит вложений. Ресурс: SMTP-соединение.

**Вариант 2.** Система «Онлайн-экзамен». Исключения: студент не допущен, время вышло, попытка списывания, вопрос не найден, ответ не принят, система мониторинга недоступна. Ресурс: сессия экзамена.

**Вариант 3.** Система «Бронирование гостиницы». Исключения: нет свободных номеров, даты недоступны, гость в чёрном списке, оплата не прошла, бронь уже существует, система бронирования недоступна. Ресурс: транзакция бронирования.

**Вариант 4.** Система «Торговая площадка». Исключения: продавец не верифицирован, товар снят с продажи, цена ниже минимальной, покупатель заблокирован, лимит сделок превышен, платёжная система недоступна. Ресурс: сессия сделки.

**Вариант 5.** Система «Управление дронами». Исключения: зона полёта запрещена, батарея критически разряжена, потеря сигнала GPS, превышена высота, столкновение неизбежно, диспетчерская система недоступна. Ресурс: сеанс связи с дроном.

**Вариант 6.** Система «Медицинская информационная система». Исключения: пациент не найден, рецепт недействителен, аллергия обнаружена, доза превышена, врач не лицензирован, система рецептов недоступна. Ресурс: защищённое соединение с базой данных.

**Вариант 7.** Система «Умный дом». Исключения: устройство не найдено, команда не поддерживается, сцена не может быть выполнена, датчик неисправен, превышен лимит энергии, контроллер недоступен. Ресурс: соединение с хабом.

**Вариант 8.** Система «Платформа видеоконференций». Исключения: комната не существует, лимит участников достигнут, камера недоступна, лицензия истекла, запись запрещена, сервер недоступен. Ресурс: медиа-сессия.

**Вариант 9.** Система «Электронный документооборот». Исключения: документ не найден, нет прав доступа, версия устарела, подпись недействительна, шаблон повреждён, хранилище недоступно. Ресурс: транзакция хранилища.

**Вариант 10.** Система «Управление парковкой». Исключения: место занято, абонемент недействителен, автомобиль в розыске, превышено время, оплата не прошла, система распознавания недоступна. Ресурс: сессия парковочного места.

**Вариант 11.** Система «Платформа обучения». Исключения: курс не найден, prerequisite не выполнен, дедлайн пропущен, плагиат обнаружен, сертификат не выдан, система тестирования недоступна. Ресурс: сессия обучения.

**Вариант 12.** Система «Криптокошелёк». Исключения: неверный адрес, недостаточно средств, транзакция отклонена сетью, комиссия слишком низкая, кошелёк заблокирован, узел сети недоступен. Ресурс: соединение с блокчейн-узлом.

**Вариант 13.** Система «Управление событиями». Исключения: площадка недоступна, дата занята, лимит гостей превышен, исполнитель отменил, оплата не прошла, система бронирования недоступна. Ресурс: транзакция бронирования.

**Вариант 14.** Система «Служба доставки». Исключения: адрес вне зоны, посылка превышает лимит, курьер недоступен, получатель не найден, оплата не прошла, система маршрутизации недоступна. Ресурс: сессия доставки.

**Вариант 15.** Система «Цифровая библиотека». Исключения: книга недоступна, лимит выдач достигнут, штраф не оплачен, читатель заблокирован, формат не поддерживается, сервер каталога недоступен. Ресурс: сессия чтения.

**Вариант 16.** Система «Платформа подкастов». Исключения: эпизод не найден, подписка истекла, регион недоступен, качество не поддерживается, скачивание запрещено, CDN недоступен. Ресурс: потоковое соединение.

**Вариант 17.** Система «Управление очередями». Исключения: талон недействителен, оператор недоступен, время ожидания истекло, приоритет оспорен, услуга не оказывается, система очередей недоступна. Ресурс: сессия обслуживания.

**Вариант 18.** Система «Платформа краудфандинга». Исключения: кампания завершена, цель не достигнута, лимит взноса превышен, проект заморожен, платёж отклонён, платформа недоступна. Ресурс: транзакция взноса.

**Вариант 19.** Система «Умное сельское хозяйство». Исключения: датчик неисправен, зона полива недоступна, удобрение закончилось, погода неблагоприятна, дрон не может взлететь, центр управления недоступен. Ресурс: сеанс связи с контроллером.

**Вариант 20.** Система «Платформа вакансий». Исключения: вакансия закрыта, резюме не соответствует, отклик уже отправлен, работодатель заблокирован, тест не пройден, система подбора недоступна. Ресурс: сессия отклика.

**Вариант 21.** Система «Цифровой пропуск». Исключения: пропуск недействителен, зона доступа запрещена, время истекло, пропуск заблокирован, биометрия не совпадает, система контроля недоступна. Ресурс: сессия верификации.

**Вариант 22.** Система «Платформа аналитики». Исключения: источник данных недоступен, запрос невалиден, лимит запросов превышен, отчёт устарел, экспорт запрещён, хранилище недоступно. Ресурс: сессия подключения к источнику.

**Вариант 23.** Система «Управление флотом». Исключения: автомобиль занят, водитель не допущен, ТО просрочено, маршрут невозможен, топливо закончилось, диспетчерская недоступна. Ресурс: сессия назначения.

**Вариант 24.** Система «Платформа онлайн-консультаций». Исключения: специалист недоступен, сессия не оплачена, время истекло, запись запрещена, качество связи низкое, платформа недоступна. Ресурс: медиа-сессия.

**Вариант 25.** Система «Управление цепочками поставок». Исключения: поставщик не верифицирован, партия забракована, срок поставки нарушен, склад переполнен, сертификат недействителен, система отслеживания недоступна. Ресурс: транзакция поставки.

**Вариант 26.** Система «Платформа киберспорта». Исключения: игрок дисквалифицирован, команда не собрана, сервер недоступен, чит обнаружен, приз не выплачен, турнирная система недоступна. Ресурс: игровая сессия.

**Вариант 27.** Система «Управление конференциями». Исключения: доклад не принят, регистрация закрыта, зал переполнен, оборудование неисправно, перевод недоступен, система регистрации недоступна. Ресурс: сессия участника.

**Вариант 28.** Система «Платформа недвижимости». Исключения: объект снят с продажи, показ невозможен, задаток не оплачен, документы недействительны, ипотека не одобрена, система регистрации недоступна. Ресурс: транзакция сделки.

**Вариант 29.** Система «Умное освещение». Исключения: лампа неисправна, сцена не найдена, расписание конфликтует, яркость вне диапазона, группа пуста, контроллер недоступен. Ресурс: сеанс связи с контроллером.

**Вариант 30.** Система «Платформа онлайн-банкинга». Исключения: операция заблокирована, лимит превышен, получатель в санкционном списке, токен недействителен, сессия истекла, банковский шлюз недоступен. Ресурс: защищённая сессия.

## 6. Методические указания к самостоятельной работе

1. **Проектирование иерархии.** Перед реализацией определите:
   - все возможные исключительные ситуации в предметной области;
   - какие из них являются ожидаемыми (checked), какие — ошибками логики (unchecked);
   - какие контекстные данные необходимо включить в каждое исключение.

2. **Конструкторы исключений.** Каждое исключение должно иметь как минимум два конструктора:
   - `(String message)` — с сообщением;
   - `(String message, Throwable cause)` — с сообщением и причиной.
   Для специализированных исключений добавляйте конструкторы с контекстными полями.

3. **Сообщения исключений.** Сообщение должно быть информативным и содержать:
   - описание проблемы;
   - идентификаторы сущностей, связанных с ошибкой;
   - фактические значения, приведшие к ошибке.

4. **Exception translation.** При преобразовании исключений между слоями:
   - всегда сохраняйте причину через конструктор с `Throwable cause`;
   - не теряйте контекстную информацию — при необходимости добавляйте её в новое исключение;
   - не выбрасывайте низкоуровневые исключения на верхних уровнях.

5. **Try-with-resources.** Применяйте для всех классов, реализующих `AutoCloseable`:
   - ресурс объявляется в скобках после `try`;
   - несколько ресурсов разделяются точкой с запятой;
   - ресурс автоматически закрывается при выходе из блока.

6. **Обработка исключений.** В демонстрационном классе:
   - перехватывайте исключения на разных уровнях иерархии;
   - демонстрируйте группировку по базовому исключению;
   - выводите контекстную информацию из специализированных исключений;
   - демонстрируйте работу exception chaining через `getCause()`.

7. **Тестирование.** Перед сдачей работы проверьте:
   - все ли исключительные ситуации покрыты;
   - корректно ли работает иерархия перехвата;
   - сохраняется ли причина при exception translation;
   - корректно ли освобождаются ресурсы в `try-with-resources`.

8. **Применение ИИ.** При использовании средств ИИ:
   - генерируйте по отдельности иерархию исключений, сервисные классы и демонстрационный класс;
   - обязательно проверяйте, что ИИ не использует `Exception` вместо специализированных типов;
   - не делегируйте ИИ проектирование иерархии без понимания семантики.

9. **Оформление отчёта.** Отчёт должен содержать:
   - листинги всех файлов проекта с комментариями;
   - диаграмму иерархии исключений;
   - протокол работы демонстрационного класса;
   - обоснование выбора checked/unchecked для каждого исключения;
   - ответы на контрольные вопросы;
   - выводы по проделанной работе.

## 7. Контрольные вопросы

1. Какова иерархия классов исключений в языке Java?
2. В чём различие между `Error` и `Exception`?
3. Что такое проверяемые (checked) и непроверяемые (unchecked) исключения?
4. В каких случаях следует применять checked-исключения, а в каких — unchecked?
5. Какова структура конструкции `try-catch-finally`?
6. Что такое множественный перехват (multi-catch)? Каковы его особенности?
7. Что такое `try-with-resources`? Какие требования предъявляются к ресурсу?
8. Что такое подавленные исключения (suppressed exceptions)?
9. Как проектируется иерархия исключений предметной области?
10. Что такое exception translation? В каких случаях он применяется?
11. Что такое exception chaining? Каким образом сохраняется причина исключения?
12. Что такое интерфейс `AutoCloseable`? Какова его роль?
13. Каковы лучшие практики обработки исключений?
14. Почему нельзя «проглатывать» исключения (пустой блок `catch`)?
15. Почему не следует использовать исключения для управления потоком выполнения?

## 8. Рекомендуемые источники

1. Блох Дж. *Java. Эффективное программирование.* — М.: Питер. — Правила 69–78 (исключения).
2. Хорстманн К. *Java. Библиотека профессионала. Том 1.* — М.: Вильямс. — Глава 7 (Обработка исключений).
3. Шилдт Г. *Java. Базовый курс.* — М.: Вильямс. — Глава 10 (Исключения).
4. Martin R. C. *Clean Code: A Handbook of Agile Software Craftsmanship.* — Prentice Hall. — Chapter 7 (Error Handling).
5. Oracle Java Tutorials. Lesson: Exceptions. URL: https://docs.oracle.com/javase/tutorial/essential/exceptions/
6. Baeldung. Exception Handling in Java. URL: https://www.baeldung.com/java-exceptions
7. Oracle Java Language Specification. Exceptions. URL: https://docs.oracle.com/javase/specs/jls/se17/html/jls-11.html
