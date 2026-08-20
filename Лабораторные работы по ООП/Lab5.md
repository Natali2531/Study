# Лабораторная работа №5. Перечисления, вложенные типы и записи

## 1. Паспорт работы

| Параметр | Значение |
|----------|----------|
| Тема | Перечисления, вложенные типы и записи |
| Номер занятия в модуле | 1 из 4 (модуль 2) |
| Продолжительность аудиторной части | 2 академических часа |
| Предшествующая подготовка | Модуль 1 (базовый синтаксис, инкапсуляция, наследование, полиморфизм, контракт `Object`) |
| Тип работы | Освоение современных языковых конструкций для моделирования предметной области |

### 1.1. Цель работы

Освоить современные языковые конструкции языка Java, предназначенные для моделирования ограниченных множеств значений, вложенных структур данных и неизменяемых объектов данных: перечисления (`enum`), вложенные и внутренние классы, а также записи (`record`). Научиться применять указанные конструкции в соответствии с их семантикой для повышения выразительности и надёжности программного кода.

### 1.2. Задачи работы

1. Изучить возможности перечислений (`enum`) как полноценных классов с полями, методами и конструкторами.
2. Освоить применение перечислений для моделирования ограниченных множеств значений предметной области.
3. Изучить виды вложенных типов: статические вложенные классы, внутренние классы, локальные классы, анонимные классы.
4. Освоить применение паттерна «Строитель» (Builder) посредством вложенных классов.
5. Изучить записи (`record`) как механизм создания неизменяемых объектов данных.
6. Освоить компактные конструкторы записей и применение валидации в записях.
7. Развить навыки выбора между классом, перечислением и записью при моделировании предметной области.

### 1.3. Оснащение

- JDK версии 17 или выше (для поддержки записей и `sealed`-классов);
- интегрированная среда разработки IntelliJ IDEA Community Edition;
- система сборки Maven или Gradle;
- система контроля версий Git.

## 2. Теоретические сведения

### 2.1. Мотивация: моделирование ограниченных множеств значений

Во многих предметных областях встречаются сущности, принимающие ограниченное число значений. Например, дни недели принимают одно из семи значений, состояние заказа — одно из нескольких фиксированных состояний, тип валюты — одно из поддерживаемых обозначений. Использование строковых или целочисленных констант для представления таких множеств имеет существенные недостатки:

1. **Отсутствие типобезопасности.** Переменной можно присвоить любое значение, в том числе недопустимое.
2. **Отсутствие пространства имён.** Константы из различных множеств могут конфликтовать.
3. **Хрупкость кода.** Изменение значения константы требует перекомпиляции всех использующих её модулей.
4. **Отсутствие поведения.** Константы не могут содержать методы и дополнительные данные.

Указанные недостатки устраняются применением перечислений (`enum`).

### 2.2. Перечисления (`enum`)

**Перечисление** — специальный тип класса, экземплярами которого является фиксированный набор именованных констант. В языке Java перечисления являются полноценными классами и обладают значительно большими возможностями, чем в большинстве других языков программирования.

#### 2.2.1. Базовое перечисление

Простейшее перечисление объявляется с использованием ключевого слова `enum`:

```java
/**
 * Перечисление дней недели
 */
public enum DayOfWeek {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
}
```

Каждая константа перечисления является единственным экземпляром соответствующего класса. Перечисления неявно наследуют от класса `java.lang.Enum` и, следовательно, не могут наследовать от других классов (однако могут реализовывать интерфейсы).

Использование перечисления:

```java
DayOfWeek today = DayOfWeek.MONDAY;

if (today == DayOfWeek.SATURDAY || today == DayOfWeek.SUNDAY) {
    System.out.println("Выходной день");
}

// Обход всех значений
for (DayOfWeek day : DayOfWeek.values()) {
    System.out.println(day);
}

// Получение константы по имени
DayOfWeek parsed = DayOfWeek.valueOf("FRIDAY");
```

#### 2.2.2. Перечисления с полями и методами

Перечисление может содержать поля, конструкторы и методы, что позволяет каждой константе обладать собственным состоянием и поведением:

```java
/**
 * Перечисление планет Солнечной системы с физическими характеристиками
 */
public enum Planet {
    MERCURY(3.303e+23, 2.4397e6),
    VENUS  (4.869e+24, 6.0518e6),
    EARTH  (5.976e+24, 6.37814e6),
    MARS   (6.421e+23, 3.3972e6),
    JUPITER(1.9e+27,   7.1492e7),
    SATURN (5.688e+26, 6.0268e7),
    URANUS (8.686e+25, 2.5559e7),
    NEPTUNE(1.024e+26, 2.4746e7);

    private final double mass;           // масса в килограммах
    private final double radius;         // радиус в метрах

    // Гравитационная постоянная (м^3 кг^-1 с^-2)
    private static final double G = 6.67300E-11;

    /**
     * Конструктор перечисления.
     * Вызывается автоматически для каждой константы.
     */
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
    }

    public double getMass() { return mass; }
    public double getRadius() { return radius; }

    /**
     * Вычисляет ускорение свободного падения на поверхности планеты
     */
    public double surfaceGravity() {
        return G * mass / (radius * radius);
    }

    /**
     * Вычисляет вес объекта массой mass на поверхности данной планеты
     */
    public double surfaceWeight(double objectMass) {
        return objectMass * surfaceGravity();
    }
}
```

Использование:

```java
double earthWeight = 75.0;
double mass = earthWeight / Planet.EARTH.surfaceGravity();

for (Planet planet : Planet.values()) {
    System.out.printf("Вес на %s: %.2f Н%n",
                      planet, planet.surfaceWeight(mass));
}
```

#### 2.2.3. Перечисления с абстрактными методами

Перечисление может объявлять абстрактные методы, которые должны быть реализованы каждой константой индивидуально. Это позволяет каждой константе обладать собственным поведением:

```java
/**
 * Перечисление операций калькулятора.
 * Каждая константа реализует собственную логику вычисления.
 */
public enum Operation {
    PLUS("+") {
        @Override
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        @Override
        public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
        @Override
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDES("/") {
        @Override
        public double apply(double x, double y) {
            if (y == 0) {
                throw new ArithmeticException("Деление на ноль");
            }
            return x / y;
        }
    };

    private final String symbol;

    Operation(String symbol) {
        this.symbol = symbol;
    }

    public String getSymbol() { return symbol; }

    /**
     * Абстрактный метод — каждая константа реализует свою логику
     */
    public abstract double apply(double x, double y);

    @Override
    public String toString() { return symbol; }
}
```

Использование:

```java
double result = Operation.PLUS.apply(10.0, 5.0);   // 15.0
System.out.println("10 " + Operation.TIMES + " 5 = " +
                   Operation.TIMES.apply(10.0, 5.0));
```

#### 2.2.4. Перечисления и интерфейсы

Перечисления могут реализовывать один или несколько интерфейсов:

```java
/**
 * Интерфейс, определяющий поведение «идентифицируемый»
 */
public interface Identifiable {
    String getCode();
    String getDescription();
}

/**
 * Перечисление валют, реализующее интерфейс Identifiable
 */
public enum Currency implements Identifiable {
    RUB("RUB", "Российский рубль"),
    USD("USD", "Доллар США"),
    EUR("EUR", "Евро"),
    CNY("CNY", "Китайский юань");

    private final String code;
    private final String description;

    Currency(String code, String description) {
        this.code = code;
        this.description = description;
    }

    @Override
    public String getCode() { return code; }

    @Override
    public String getDescription() { return description; }
}
```

#### 2.2.5. Специализированные коллекции для перечислений

Библиотека Java предоставляет специализированные коллекции для эффективной работы с перечислениями:

- `EnumSet<E>` — множество, оптимизированное для хранения элементов перечисления (реализовано посредством битового вектора);
- `EnumMap<K, V>` — карта, ключами которой являются элементы перечисления.

```java
// Множество выходных дней
EnumSet<DayOfWeek> weekend = EnumSet.of(DayOfWeek.SATURDAY, DayOfWeek.SUNDAY);

// Множество рабочих дней (все, кроме выходных)
EnumSet<DayOfWeek> workdays = EnumSet.complementOf(weekend);

// Карта сопоставления дня недели и количества рабочих часов
EnumMap<DayOfWeek, Integer> workHours = new EnumMap<>(DayOfWeek.class);
workHours.put(DayOfWeek.MONDAY, 8);
workHours.put(DayOfWeek.FRIDAY, 7);
```

### 2.3. Вложенные типы

Язык Java позволяет объявлять классы внутри других классов. Такие классы называются **вложенными** (nested). Различают четыре вида вложенных типов:

| Вид | Модификатор `static` | Доступ к членам внешнего класса | Типичное применение |
|-----|:--------------------:|:-------------------------------:|---------------------|
| Статический вложенный класс | Да | Только к статическим | Вспомогательные классы, паттерн Builder |
| Внутренний класс (нестатический) | Нет | Ко всем, включая нестатические | События, итераторы, адаптеры |
| Локальный класс | — | К эффективным `final` переменным метода | Одноразовая логика внутри метода |
| Анонимный класс | — | Как локальный класс | Реализация функциональных интерфейсов, коллбэки |

#### 2.3.1. Статические вложенные классы

**Статический вложенный класс** объявляется с модификатором `static` и не имеет неявной ссылки на экземпляр внешнего класса. Может обращаться только к статическим членам внешнего класса.

```java
/**
 * Класс, моделирующий компьютер с конфигурацией
 */
public class Computer {
    private String brand;
    private String model;
    private Configuration configuration;

    /**
     * Статический вложенный класс, моделирующий конфигурацию компьютера.
     * Может существовать независимо от экземпляра Computer.
     */
    public static class Configuration {
        private final String cpu;
        private final int ramGB;
        private final int storageGB;

        public Configuration(String cpu, int ramGB, int storageGB) {
            this.cpu = cpu;
            this.ramGB = ramGB;
            this.storageGB = storageGB;
        }

        public String getCpu() { return cpu; }
        public int getRamGB() { return ramGB; }
        public int getStorageGB() { return storageGB; }

        @Override
        public String toString() {
            return String.format("CPU: %s, RAM: %d ГБ, Накопитель: %d ГБ",
                                 cpu, ramGB, storageGB);
        }
    }

    public Computer(String brand, String model, Configuration configuration) {
        this.brand = brand;
        this.model = model;
        this.configuration = configuration;
    }

    // Создание вложенного объекта не требует экземпляра внешнего класса
    public static void main(String[] args) {
        Configuration config = new Configuration("Intel i7", 16, 512);
        Computer computer = new Computer("Dell", "XPS 15", config);
    }
}
```

#### 2.3.2. Внутренние классы (нестатические)

**Внутренний класс** не имеет модификатора `static` и обладает неявной ссылкой на экземпляр внешнего класса. Может обращаться ко всем членам внешнего класса, включая приватные.

```java
/**
 * Класс, моделирующий связный список.
 * Внутренний класс Node представляет элемент списка.
 */
public class LinkedList {
    /**
     * Внутренний класс — узел списка.
     * Имеет доступ к членам внешнего класса.
     */
    private class Node {
        int data;
        Node next;

        Node(int data) {
            this.data = data;
            this.next = null;
        }
    }

    private Node head;
    private int size;

    public void add(int data) {
        Node newNode = new Node(data);
        if (head == null) {
            head = newNode;
        } else {
            Node current = head;
            while (current.next != null) {
                current = current.next;
            }
            current.next = newNode;
        }
        size++;
    }

    public int size() { return size; }
}
```

#### 2.3.3. Паттерн «Строитель» (Builder)

Наиболее распространённым применением статических вложенных классов является реализация паттерна «Строитель», позволяющего конструировать объекты с большим количеством параметров:

```java
/**
 * Класс, моделирующий заказ в интернет-магазине.
 * Применяется паттерн Builder для гибкой инициализации.
 */
public class Order {
    private final String orderId;
    private final String customerName;
    private final String deliveryAddress;
    private final List<String> items;
    private final double discount;
    private final String promoCode;
    private final boolean isGiftWrapped;

    /**
     * Приватный конструктор — создание объекта только через Builder
     */
    private Order(Builder builder) {
        this.orderId = builder.orderId;
        this.customerName = builder.customerName;
        this.deliveryAddress = builder.deliveryAddress;
        this.items = List.copyOf(builder.items);
        this.discount = builder.discount;
        this.promoCode = builder.promoCode;
        this.isGiftWrapped = builder.isGiftWrapped;
    }

    // Геттеры
    public String getOrderId() { return orderId; }
    public String getCustomerName() { return customerName; }
    public String getDeliveryAddress() { return deliveryAddress; }
    public List<String> getItems() { return items; }
    public double getDiscount() { return discount; }
    public String getPromoCode() { return promoCode; }
    public boolean isGiftWrapped() { return isGiftWrapped; }

    /**
     * Статический вложенный класс Builder
     */
    public static class Builder {
        // Обязательные параметры
        private final String orderId;
        private final String customerName;

        // Необязательные параметры (значения по умолчанию)
        private String deliveryAddress = "";
        private final List<String> items = new ArrayList<>();
        private double discount = 0.0;
        private String promoCode = "";
        private boolean isGiftWrapped = false;

        public Builder(String orderId, String customerName) {
            this.orderId = orderId;
            this.customerName = customerName;
        }

        public Builder deliveryAddress(String address) {
            this.deliveryAddress = address;
            return this;
        }

        public Builder addItem(String item) {
            this.items.add(item);
            return this;
        }

        public Builder discount(double discount) {
            this.discount = discount;
            return this;
        }

        public Builder promoCode(String code) {
            this.promoCode = code;
            return this;
        }

        public Builder giftWrapped(boolean wrapped) {
            this.isGiftWrapped = wrapped;
            return this;
        }

        public Order build() {
            if (items.isEmpty()) {
                throw new IllegalStateException("Заказ должен содержать хотя бы один товар");
            }
            return new Order(this);
        }
    }

    @Override
    public String toString() {
        return String.format("Заказ %s для %s: %d товаров, скидка %.0f%%",
                             orderId, customerName, items.size(), discount * 100);
    }
}
```

Использование:

```java
Order order = new Order.Builder("ORD-001", "Иванов И.И.")
    .deliveryAddress("г. Москва, ул. Ленина, д. 1")
    .addItem("Ноутбук")
    .addItem("Мышь")
    .addItem("Клавиатура")
    .discount(0.1)
    .promoCode("SUMMER2026")
    .giftWrapped(true)
    .build();

System.out.println(order);
```

#### 2.3.4. Локальные и анонимные классы

**Локальный класс** объявляется внутри метода и доступен только в пределах этого метода:

```java
public void processData() {
    /**
     * Локальный класс, доступный только в рамках метода
     */
    class DataValidator {
        boolean isValid(String data) {
            return data != null && !data.isEmpty();
        }
    }

    DataValidator validator = new DataValidator();
    if (validator.isValid("test")) {
        System.out.println("Данные корректны");
    }
}
```

**Анонимный класс** — локальный класс без имени, объявляемый и инстанцируемый в одном выражении:

```java
public void demonstrateAnonymous() {
    Runnable task = new Runnable() {
        @Override
        public void run() {
            System.out.println("Выполнение задачи");
        }
    };
    task.run();
}
```

### 2.4. Записи (`record`)

**Запись** — специальный вид класса, предназначенный для представления неизменяемых носителей данных. Записи были представлены в Java 14 (как предварительная версия) и стали стандартной возможностью в Java 16.

Запись автоматически генерирует:
- приватные `final`-поля для каждого компонента;
- канонический конструктор, соответствующий списку компонентов;
- геттеры (с именами компонентов, а не `get...`);
- методы `equals()`, `hashCode()`, `toString()`;
- деструкторизацию (pattern matching).

#### 2.4.1. Базовая запись

```java
/**
 * Запись, моделирующая точку на плоскости.
 * Компилятор автоматически генерирует конструктор, геттеры,
 * equals(), hashCode() и toString().
 */
public record Point(double x, double y) {
    // Дополнительный метод
    public double distanceToOrigin() {
        return Math.sqrt(x * x + y * y);
    }

    public double distanceTo(Point other) {
        double dx = this.x - other.x;
        double dy = this.y - other.y;
        return Math.sqrt(dx * dx + dy * dy);
    }
}
```

Использование:

```java
Point p1 = new Point(3.0, 4.0);
Point p2 = new Point(6.0, 8.0);

// Геттеры называются по имени компонента (не getX, а x)
System.out.println("Координаты: (" + p1.x() + "; " + p1.y() + ")");
System.out.println("Расстояние до начала координат: " + p1.distanceToOrigin());
System.out.println("Расстояние между точками: " + p1.distanceTo(p2));

// Автоматически сгенерированные методы
System.out.println(p1);                              // Point[x=3.0, y=4.0]
System.out.println("Равны: " + p1.equals(new Point(3.0, 4.0)));  // true
System.out.println("Хеш-код: " + p1.hashCode());
```

#### 2.4.2. Компактный конструктор и валидация

Записи поддерживают **компактный конструктор** — особую форму конструктора, не объявляющую параметры и не выполняющую присваивание полей (оно происходит неявно после тела конструктора). Компактный конструктор применяется для валидации и нормализации значений:

```java
import java.time.LocalDate;

/**
 * Запись, моделирующая денежную сумму в определённой валюте
 */
public record Money(double amount, String currency) {
    /**
     * Компактный конструктор с валидацией
     */
    public Money {
        if (amount < 0) {
            throw new IllegalArgumentException(
                "Сумма не может быть отрицательной: " + amount);
        }
        if (currency == null || currency.trim().isEmpty()) {
            throw new IllegalArgumentException("Валюта не может быть пустой");
        }
        if (currency.length() != 3) {
            throw new IllegalArgumentException(
                "Код валюты должен состоять из 3 символов: " + currency);
        }
        // Нормализация: приведение кода валюты к верхнему регистру
        currency = currency.toUpperCase();
    }

    /**
     * Дополнительные методы
     */
    public Money add(Money other) {
        if (!this.currency.equals(other.currency)) {
            throw new IllegalArgumentException(
                "Нельзя складывать суммы в разных валютах");
        }
        return new Money(this.amount + other.amount, this.currency);
    }

    @Override
    public String toString() {
        return String.format("%.2f %s", amount, currency);
    }
}
```

#### 2.4.3. Записи и интерфейсы

Записи могут реализовывать интерфейсы, но не могут наследовать классы (поскольку неявно наследуют от `java.lang.Record`):

```java
/**
 * Интерфейс, определяющий поведение «имеющий стоимость»
 */
public interface Valuable {
    double getValue();
    String getUnit();
}

/**
 * Запись, реализующая интерфейс
 */
public record Product(String name, double price, String currency)
        implements Valuable {

    @Override
    public double getValue() {
        return price;
    }

    @Override
    public String getUnit() {
        return currency;
    }
}
```

#### 2.4.4. Ограничения записей

Записи имеют следующие ограничения:
- неявно являются `final` — не могут быть наследованы;
- неявно наследуют от `java.lang.Record` — не могут наследовать другие классы;
- поля неявно являются `private final` — записи неизменяемы;
- не могут содержать нестатические поля;
- не могут содержать инициализаторы экземпляра (блоки `{}`).

### 2.5. Выбор между классом, перечислением и записью

| Критерий | Класс | Перечисление (`enum`) | Запись (`record`) |
|----------|-------|----------------------|-------------------|
| Количество экземпляров | Произвольное | Фиксированный набор | Произвольное |
| Изменяемость | Изменяемый или неизменяемый | Неизменяемый (константы) | Неизменяемый |
| Состояние | Произвольное | Фиксированное для каждой константы | Только поля-компоненты |
| Поведение | Произвольное | Методы констант и перечисления | Методы записи |
| Наследование | Да | Только интерфейсы | Только интерфейсы |
| Типичное применение | Сущности с изменяемым состоянием | Ограниченные множества значений | Носители данных (DTO, значения) |

**Общее правило:**
- если сущность имеет фиксированный набор состояний — применяйте `enum`;
- если сущность является неизменяемым носителем данных — применяйте `record`;
- если сущность имеет изменяемое состояние или требует наследования — применяйте обычный класс.

## 3. Примеры выполнения

### 3.1. Пример 1. Перечисление «Статусы заказа» с логикой переходов

```java
/**
 * Перечисление, моделирующее статусы заказа в интернет-магазине.
 * Каждая константа содержит описание и набор допустимых переходов.
 */
public enum OrderStatus {
    NEW("Новый заказ") {
        @Override
        public boolean canTransitionTo(OrderStatus target) {
            return target == CONFIRMED || target == CANCELLED;
        }
    },
    CONFIRMED("Подтверждён") {
        @Override
        public boolean canTransitionTo(OrderStatus target) {
            return target == PROCESSING || target == CANCELLED;
        }
    },
    PROCESSING("В обработке") {
        @Override
        public boolean canTransitionTo(OrderStatus target) {
            return target == SHIPPED || target == CANCELLED;
        }
    },
    SHIPPED("Отправлен") {
        @Override
        public boolean canTransitionTo(OrderStatus target) {
            return target == DELIVERED;
        }
    },
    DELIVERED("Доставлен") {
        @Override
        public boolean canTransitionTo(OrderStatus target) {
            return target == COMPLETED || target == RETURNED;
        }
    },
    COMPLETED("Завершён") {
        @Override
        public boolean canTransitionTo(OrderStatus target) {
            return false;   // терминальное состояние
        }
    },
    CANCELLED("Отменён") {
        @Override
        public boolean canTransitionTo(OrderStatus target) {
            return false;   // терминальное состояние
        }
    },
    RETURNED("Возвращён") {
        @Override
        public boolean canTransitionTo(OrderStatus target) {
            return target == COMPLETED;
        }
    };

    private final String description;

    OrderStatus(String description) {
        this.description = description;
    }

    public String getDescription() {
        return description;
    }

    /**
     * Абстрактный метод проверки допустимости перехода.
     * Каждая константа реализует собственную логику.
     */
    public abstract boolean canTransitionTo(OrderStatus target);

    @Override
    public String toString() {
        return description;
    }
}

/**
 * Класс, моделирующий заказ
 */
public class Order {
    private String orderId;
    private OrderStatus status;

    public Order(String orderId) {
        this.orderId = orderId;
        this.status = OrderStatus.NEW;
    }

    /**
     * Изменение статуса заказа с проверкой допустимости перехода
     */
    public void changeStatus(OrderStatus newStatus) {
        if (!status.canTransitionTo(newStatus)) {
            throw new IllegalStateException(
                "Недопустимый переход: " + status + " → " + newStatus);
        }
        System.out.println("Заказ " + orderId + ": " + status + " → " + newStatus);
        this.status = newStatus;
    }

    public OrderStatus getStatus() { return status; }
}
```

Демонстрационный класс:

```java
public class OrderDemo {
    public static void main(String[] args) {
        Order order = new Order("ORD-001");
        System.out.println("Начальный статус: " + order.getStatus());

        order.changeStatus(OrderStatus.CONFIRMED);
        order.changeStatus(OrderStatus.PROCESSING);
        order.changeStatus(OrderStatus.SHIPPED);
        order.changeStatus(OrderStatus.DELIVERED);
        order.changeStatus(OrderStatus.COMPLETED);

        // Попытка недопустимого перехода
        try {
            order.changeStatus(OrderStatus.NEW);
        } catch (IllegalStateException e) {
            System.out.println("Ошибка: " + e.getMessage());
        }
    }
}
```

### 3.2. Пример 2. Паттерн Builder для конфигурации HTTP-запроса

```java
import java.util.Collections;
import java.util.HashMap;
import java.util.Map;

/**
 * Неизменяемый класс, моделирующий HTTP-запрос.
 * Конструирование осуществляется посредством вложенного Builder.
 */
public final class HttpRequest {
    private final String method;
    private final String url;
    private final Map<String, String> headers;
    private final String body;
    private final int timeoutMs;

    private HttpRequest(Builder builder) {
        this.method = builder.method;
        this.url = builder.url;
        this.headers = Collections.unmodifiableMap(new HashMap<>(builder.headers));
        this.body = builder.body;
        this.timeoutMs = builder.timeoutMs;
    }

    public String getMethod() { return method; }
    public String getUrl() { return url; }
    public Map<String, String> getHeaders() { return headers; }
    public String getBody() { return body; }
    public int getTimeoutMs() { return timeoutMs; }

    @Override
    public String toString() {
        return String.format("%s %s (headers: %d, timeout: %d мс)",
                             method, url, headers.size(), timeoutMs);
    }

    /**
     * Статический вложенный класс Builder
     */
    public static class Builder {
        private final String method;
        private final String url;
        private final Map<String, String> headers = new HashMap<>();
        private String body = "";
        private int timeoutMs = 30_000;

        public Builder(String method, String url) {
            if (method == null || method.trim().isEmpty()) {
                throw new IllegalArgumentException("Метод не может быть пустым");
            }
            if (url == null || url.trim().isEmpty()) {
                throw new IllegalArgumentException("URL не может быть пустым");
            }
            this.method = method.toUpperCase();
            this.url = url;
        }

        public Builder header(String name, String value) {
            headers.put(name, value);
            return this;
        }

        public Builder body(String body) {
            this.body = body;
            return this;
        }

        public Builder timeout(int timeoutMs) {
            if (timeoutMs <= 0) {
                throw new IllegalArgumentException(
                    "Таймаут должен быть положительным");
            }
            this.timeoutMs = timeoutMs;
            return this;
        }

        public HttpRequest build() {
            return new HttpRequest(this);
        }
    }
}
```

Использование:

```java
public class HttpRequestDemo {
    public static void main(String[] args) {
        HttpRequest request = new HttpRequest.Builder("POST", "https://api.example.com/data")
            .header("Content-Type", "application/json")
            .header("Authorization", "Bearer token123")
            .body("{\"key\": \"value\"}")
            .timeout(5000)
            .build();

        System.out.println(request);
        System.out.println("Заголовки: " + request.getHeaders());

        // Попытка модификации возвращённой карты (неизменяемая)
        try {
            request.getHeaders().put("New-Header", "value");
        } catch (UnsupportedOperationException e) {
            System.out.println("Карта заголовков неизменяема");
        }
    }
}
```

### 3.3. Пример 3. Записи для моделирования предметной области

```java
import java.time.LocalDate;
import java.util.List;
import java.util.Objects;

/**
 * Запись, моделирующая адрес
 */
public record Address(String city, String street, String building, String apartment) {
    public Address {
        Objects.requireNonNull(city, "Город не может быть null");
        Objects.requireNonNull(street, "Улица не может быть null");
        Objects.requireNonNull(building, "Номер дома не может быть null");
        if (city.trim().isEmpty()) {
            throw new IllegalArgumentException("Город не может быть пустым");
        }
    }

    @Override
    public String toString() {
        return apartment == null || apartment.isEmpty()
            ? String.format("%s, %s, д. %s", city, street, building)
            : String.format("%s, %s, д. %s, кв. %s", city, street, building, apartment);
    }
}

/**
 * Запись, моделирующая период дат
 */
public record DateRange(LocalDate from, LocalDate to) {
    public DateRange {
        Objects.requireNonNull(from, "Дата начала не может быть null");
        Objects.requireNonNull(to, "Дата окончания не может быть null");
        if (to.isBefore(from)) {
            throw new IllegalArgumentException(
                "Дата окончания не может быть раньше даты начала");
        }
    }

    public long getDays() {
        return java.time.temporal.ChronoUnit.DAYS.between(from, to) + 1;
    }

    public boolean contains(LocalDate date) {
        return !date.isBefore(from) && !date.isAfter(to);
    }
}

/**
 * Запись, моделирующая бронирование номера в отеле
 */
public record HotelBooking(String bookingId,
                           String guestName,
                           Address hotelAddress,
                           DateRange stayPeriod,
                           String roomType,
                           double pricePerNight) {
    public HotelBooking {
        Objects.requireNonNull(bookingId, "ID бронирования не может быть null");
        Objects.requireNonNull(guestName, "Имя гостя не может быть null");
        Objects.requireNonNull(hotelAddress, "Адрес не может быть null");
        Objects.requireNonNull(stayPeriod, "Период проживания не может быть null");
        Objects.requireNonNull(roomType, "Тип номера не может быть null");
        if (pricePerNight <= 0) {
            throw new IllegalArgumentException(
                "Стоимость за ночь должна быть положительной");
        }
    }

    /**
     * Вычисляет общую стоимость проживания
     */
    public double getTotalPrice() {
        return stayPeriod.getDays() * pricePerNight;
    }

    @Override
    public String toString() {
        return String.format(
            "Бронирование %s: %s, %s, %s, %d ночей, итого: %.2f руб.",
            bookingId, guestName, hotelAddress, roomType,
            stayPeriod.getDays(), getTotalPrice());
    }
}
```

Демонстрационный класс:

```java
public class BookingDemo {
    public static void main(String[] args) {
        Address address = new Address("Санкт-Петербург", "Невский пр.", "28", "15");
        DateRange period = new DateRange(
            LocalDate.of(2026, 8, 20),
            LocalDate.of(2026, 8, 27));

        HotelBooking booking = new HotelBooking(
            "BK-001",
            "Иванов И.И.",
            address,
            period,
            "Люкс",
            8500.0);

        System.out.println(booking);

        // Демонстрация неизменяемости
        System.out.println("\nПроверка равенства:");
        HotelBooking sameBooking = new HotelBooking(
            "BK-001", "Иванов И.И.", address, period, "Люкс", 8500.0);
        System.out.println("booking.equals(sameBooking): " +
                           booking.equals(sameBooking));

        // Проверка попадания даты в период
        LocalDate checkDate = LocalDate.of(2026, 8, 23);
        System.out.println("\nДата " + checkDate + " попадает в период: " +
                           period.contains(checkDate));
    }
}
```

## 4. Задания на паре

### Задание 4.1. Перечисление «Типы данных» с валидацией

Разработайте перечисление `DataType`, моделирующее типы данных, которые могут храниться в ячейке электронной таблицы.

**Требования к перечислению:**

Каждая константа перечисления должна содержать:
- `name` (String) — отображаемое имя типа;
- `storageSize` (int) — размер хранения в байтах;
- `pattern` (String) — регулярное выражение для проверки корректности значения.

Константы:
- `INTEGER` — целое число, 4 байта, шаблон `^-?\d+$`;
- `DOUBLE` — вещественное число, 8 байт, шаблон `^-?\d+(\.\d+)?$`;
- `BOOLEAN` — логическое значение, 1 байт, шаблон `^(true|false)$`;
- `STRING` — строка, 40 байт (среднее), шаблон `^.*$`;
- `DATE` — дата, 8 байт, шаблон `^\d{4}-\d{2}-\d{2}$`;
- `CURRENCY` — денежная сумма, 8 байт, шаблон `^\d+(\.\d{2})?$`.

Методы перечисления:
- `boolean isValid(String value)` — проверяет, соответствует ли значение шаблону типа;
- `Object parse(String value)` — преобразует строку в значение соответствующего типа (для `INTEGER` — `Integer`, для `DOUBLE` — `Double`, для `BOOLEAN` — `Boolean`, для `DATE` — `java.time.LocalDate`, для остальных — `String`);
- `static DataType detectType(String value)` — определяет тип по значению (перебирает типы в порядке приоритета).

**Демонстрация:**
- Создайте массив строк, содержащий значения различных типов.
- Для каждого значения определите его тип посредством `detectType`.
- Выполните разбор значения и выведите результат.
- Продемонстрируйте обработку некорректных значений.

**Пример выполнения программы:**

```
=== Определение типов данных ===
Значение: "42"              → Тип: INTEGER, разобрано: 42 (java.lang.Integer)
Значение: "3.14159"         → Тип: DOUBLE, разобрано: 3.14159 (java.lang.Double)
Значение: "true"            → Тип: BOOLEAN, разобрано: true (java.lang.Boolean)
Значение: "2026-08-20"      → Тип: DATE, разобрано: 2026-08-20 (java.time.LocalDate)
Значение: "1234.56"         → Тип: CURRENCY, разобрано: 1234.56 (java.lang.Double)
Значение: "Привет, мир!"    → Тип: STRING, разобрано: Привет, мир! (java.lang.String)

=== Размеры хранения ===
INTEGER: 4 байт
DOUBLE: 8 байт
BOOLEAN: 1 байт
STRING: 40 байт
DATE: 8 байт
CURRENCY: 8 байт
```

---

### Задание 4.2. Паттерн Builder для «Конфигурации отчёта»

Разработайте класс `Report`, моделирующий отчёт, с применением паттерна Builder.

**Класс `Report` должен содержать:**
- `title` (String) — заголовок отчёта (обязательный);
- `author` (String) — автор отчёта (обязательный);
- `createdAt` (LocalDate) — дата создания (устанавливается автоматически);
- `sections` (List<String>) — список разделов (по умолчанию пустой);
- `format` (ReportFormat — перечисление: PDF, DOCX, HTML, TXT) — формат (по умолчанию PDF);
- `isConfidential` (boolean) — признак конфиденциальности (по умолчанию false);
- `maxPages` (int) — максимальное количество страниц (по умолчанию 100).

**Перечисление `ReportFormat`:**
- Содержит поле `extension` (String) — расширение файла;
- Содержит поле `mimeType` (String) — MIME-тип;
- Метод `getFileName(String baseName)` — формирует имя файла.

**Вложенный класс `Report.Builder`:**
- Реализует fluent-интерфейс (возвращает `this` во всех методах);
- Метод `addSection(String section)` — добавляет раздел;
- Метод `build()` выполняет валидацию: заголовок не пуст, автор не пуст, список разделов не пуст.

**Демонстрация:**
- Создайте не менее трёх отчётов с различными конфигурациями.
- Выведите информацию о каждом отчёте.
- Сформируйте имена файлов для каждого отчёта в различных форматах.

---

### Задание 4.3. Записи для «Платёжных операций»

Разработайте систему записей, моделирующих платёжные операции.

**Запись `Account`:**
- Компоненты: `accountNumber` (String), `ownerName` (String), `balance` (double).
- Компактный конструктор: проверка корректности номера счёта (10 цифр), имени, неотрицательности начального баланса.

**Перечисление `TransactionType`:**
- Константы: `DEPOSIT`, `WITHDRAW`, `TRANSFER`.
- Поле `description` (String).
- Метод `isIncreasesBalance()` — возвращает `true`, если операция увеличивает баланс.

**Запись `Transaction`:**
- Компоненты: `transactionId` (String), `type` (TransactionType), `amount` (double), `timestamp` (LocalDateTime), `description` (String).
- Компактный конструктор: валидация всех полей.
- Метод `getSignedAmount()` — возвращает сумму со знаком (положительную для DEPOSIT, отрицательную для WITHDRAW и TRANSFER).

**Запись `TransferOperation`:**
- Компоненты: `transaction` (Transaction), `sourceAccount` (Account), `targetAccount` (Account).
- Компактный конструктор: проверка совпадения валют (если применимо), проверка достаточности средств.
- Метод `execute()` — возвращает пару новых состояний счетов после выполнения перевода (используя `withBalance()`).

**Дополнительно:**
- Реализуйте в записи `Account` метод `withBalance(double newBalance)`, возвращающий новую запись с изменённым балансом (записи неизменяемы).

**Демонстрация:**
- Создайте два счёта.
- Выполните не менее трёх операций различных типов.
- Выведите историю операций и итоговые балансы счетов.
- Продемонстрируйте неизменяемость записей.

## 5. Задание для самостоятельной работы

Разработать систему классов, перечислений и записей согласно своему варианту. Требования:

1. Не менее одного перечисления с полями, методами и (при необходимости) абстрактными методами.
2. Не менее одного вложенного класса (статического или внутреннего) с осмысленным применением.
3. Не менее двух записей с компактным конструктором и валидацией.
4. Демонстрация всех возможностей в демонстрационном классе.
5. Обработка граничных случаев и некорректных данных.

### Варианты заданий

**Вариант 1.** Система «Библиотека». Перечисление `BookGenre` (жанры с возрастными ограничениями). Записи `Book` (ISBN, название, автор, год, жанр), `Reader` (номер билета, ФИО, адрес). Вложенный класс `Library.Card` (карточка читателя с историей выдач).

**Вариант 2.** Система «Авиакомпания». Перечисление `SeatClass` (эконом, бизнес, первый с множителями цены). Записи `Flight` (номер, маршрут, дата, время), `Passenger` (ФИО, паспорт, место). Вложенный класс `Booking.Builder` (бронирование с опциями).

**Вариант 3.** Система «Банк». Перечисление `AccountType` (текущий, сберегательный, кредитный, депозитный со ставками). Записи `Client` (ИНН, ФИО, адрес), `Transaction` (ID, тип, сумма, дата). Вложенный класс `Bank.Statement` (выписка по счёту).

**Вариант 4.** Система «Интернет-магазин». Перечисление `OrderStatus` (статусы с допустимыми переходами). Записи `Product` (артикул, название, цена, категория), `CartItem` (товар, количество). Вложенный класс `Order.Builder` (формирование заказа).

**Вариант 5.** Система «Больница». Перечисление `DiagnosisCode` (коды МКБ с описаниями). Записи `Patient` (номер карты, ФИО, дата рождения), `Appointment` (дата, врач, кабинет, диагноз). Вложенный класс `MedicalRecord.Entry` (запись в медицинской карте).

**Вариант 6.** Система «Учебный процесс». Перечисление `Grade` (оценки с баллами и описаниями). Записи `Student` (номер зачётки, ФИО, группа), `Course` (код, название, семестр, преподаватель). Вложенный класс `Transcript` (зачётная ведомость с методами расчёта среднего балла).

**Вариант 7.** Система «Ресторан». Перечисление `MealType` (завтрак, обед, ужин, десерт с временными диапазонами). Записи `MenuItem` (код, название, цена, категория), `Order` (номер, стол, список блюд, время). Вложенный класс `Restaurant.Receipt` (чек с расчётом НДС).

**Вариант 8.** Система «Фитнес-клуб». Перечисление `MembershipType` (месячный, квартальный, годовой, VIP со скидками). Записи `Client` (номер, ФИО, телефон), `Visit` (дата, время входа, время выхода). Вложенный класс `Membership.Builder` (конструирование абонемента с доп. услугами).

**Вариант 9.** Система «Автосервис». Перечисление `ServiceType` (ТО, ремонт кузова, ремонт двигателя, шиномонтаж с нормо-часами). Записи `Vehicle` (госномер, марка, модель, VIN), `ServiceOrder` (номер, автомобиль, услуги, мастер). Вложенный класс `Invoice` (счёт на оплату).

**Вариант 10.** Система «Турагентство». Перечисление `TourType` (пляжный, экскурсионный, горнолыжный, круиз с сезонностью). Записи `Tour` (код, страна, тип, цена, длительность), `Tourist` (паспорт, телефон, email). Вложенный класс `Booking.Builder` (бронирование тура с опциями).

**Вариант 11.** Система «Кинопрокат». Перечисление `MovieGenre` (жанры с возрастными рейтингами). Записи `Movie` (код, название, режиссёр, длительность, жанр), `Session` (номер зала, время, фильм). Вложенный класс `Ticket` (билет с рядом, местом, ценой).

**Вариант 12.** Система «Склад». Перечисление `StorageCondition` (обычное, холодильное, морозильное, опасные грузы с диапазонами температур). Записи `Product` (артикул, название, категория, вес), `Shipment` (номер, список товаров, дата). Вложенный класс `Warehouse.Location` (ячейка хранения с зоной и стеллажом).

**Вариант 13.** Система «Страховая компания». Перечисление `InsuranceType` (ОСАГО, КАСКО, страхование жизни, страхование имущества с коэффициентами). Записи `Policy` (номер, тип, страхователь, срок), `InsuranceCase` (номер, дата, описание). Вложенный класс `Claim` (страховое требование с расчётом выплаты).

**Вариант 14.** Система «Почтовая служба». Перечисление `DeliveryType` (стандартная, экспресс, международная, курьерская со сроками). Записи `Parcel` (трек-номер, вес, габариты), `Address` (индекс, город, улица, дом). Вложенный класс `TrackingEvent` (событие отслеживания с датой и статусом).

**Вариант 15.** Система «Недвижимость». Перечисление `PropertyType` (квартира, дом, коммерческая, земельный участок с налоговыми ставками). Записи `Property` (кадастровый номер, адрес, площадь, тип), `Agent` (номер лицензии, ФИО, агентство). Вложенный класс `Contract` (договор купли-продажи с расчётом стоимости).

**Вариант 16.** Система «Кадровое агентство». Перечисление `EmploymentType` (полная занятость, частичная, удалённая, стажировка с условиями). Записи `Candidate` (резюме, ФИО, навыки), `Vacancy` (код, должность, компания, зарплата). Вложенный класс `JobOffer.Builder` (предложение о работе с условиями).

**Вариант 17.** Система «Благотворительный фонд». Перечисление `DonationType` (разовая, регулярная, целевая, корпоративная с налоговыми вычетами). Записи `Donor` (ИНН/паспорт, ФИО, контакты), `Campaign` (код, название, целевая сумма, срок). Вложенный класс `Donation` (пожертвование с назначением и чеком).

**Вариант 18.** Система «Такси». Перечисление `CarClass` (эконом, комфорт, бизнес, минивэн с тарифами). Записи `Driver` (номер лицензии, ФИО, автомобиль), `Trip` (номер, адрес отправления, адрес назначения, расстояние). Вложенный класс `RideCost` (расчёт стоимости с учётом класса, времени, коэффициентов).

**Вариант 19.** Система «Музей». Перечисление `ExhibitType` (живопись, скульптура, археология, документы, нумизматика с условиями хранения). Записи `Exhibit` (инвентарный номер, название, автор, год, тип), `Visitor` (билет, ФИО, возраст). Вложенный класс `Exhibition` (выставка с датами и списком экспонатов).

**Вариант 20.** Система «Спортивный клуб». Перечисление `SportType` (футбол, баскетбол, теннис, плавание, единоборства с правилами). Записи `Athlete` (номер, ФИО, дата рождения, разряд), `Competition` (название, дата, место, тип спорта). Вложенный класс `Result` (результат выступления с местом и очками).

**Вариант 21.** Система «Электронная почта». Перечисление `Priority` (низкий, обычный, высокий, срочный с маркерами). Записи `EmailAddress` (адрес, домен, отображаемое имя), `Attachment` (имя файла, размер, MIME-тип). Вложенный класс `Email.Builder` (конструирование письма с получателями, копиями, вложениями).

**Вариант 22.** Система «Календарь событий». Перечисление `EventRecurrence` (ежедневно, еженедельно, ежемесячно, ежегодно, не повторяется). Записи `Event` (название, дата начала, длительность, место), `Participant` (email, ФИО, статус). Вложенный класс `Reminder` (напоминание с временем и типом уведомления).

**Вариант 23.** Система «Платёжная система». Перечисление `PaymentMethod` (банковская карта, электронный кошелёк, мобильный платёж, криптовалюта с комиссиями). Записи `Merchant` (ID, название, категория), `Payment` (ID, сумма, валюта, метод). Вложенный класс `Refund` (возврат средств с причиной и статусом).

**Вариант 24.** Система «Управление проектами». Перечисление `TaskPriority` (критическая, высокая, средняя, низкая со сроками). Записи `Project` (код, название, менеджер, сроки), `Task` (ID, название, исполнитель, приоритет). Вложенный класс `Sprint` (спринт с датами и списком задач).

**Вариант 25.** Система «Ветеринарная клиника». Перечисление `AnimalType` (собака, кошка, птица, грызун, рептилия с особенностями приёма). Записи `Pet` (кличка, вид, порода, возраст, владелец), `Visit` (дата, диагноз, назначения). Вложенный класс `Vaccination` (прививка с названием, датой, ревакцинацией).

**Вариант 26.** Система «Железнодорожные перевозки». Перечисление `CarriageType` (плацкарт, купе, СВ, люкс с вместимостью и ценами). Записи `Train` (номер, маршрут, состав), `Passenger` (ФИО, паспорт, документ). Вложенный класс `Ticket` (билет с поездом, вагоном, местом).

**Вариант 27.** Система «Курсы иностранных языков». Перечисление `LanguageLevel` (A1, A2, B1, B2, C1, C2 с требованиями). Записи `Student` (номер, ФИО, уровень), `Course` (язык, уровень, преподаватель, часы). Вложенный класс `ExamResult` (результат экзамена с баллами по разделам).

**Вариант 28.** Система «Фотограф». Перечисление `ShootType` (портрет, свадьба, репортаж, предметная съёмка с базовыми ценами). Записи `Client` (ФИО, телефон, email), `Booking` (дата, тип съёмки, длительность, локация). Вложенный класс `PhotoPackage` (пакет фотографий с количеством и обработкой).

**Вариант 29.** Система «Коворкинг». Перечисение `WorkspaceType` (рабочее место, переговорная, офис,_event-зал с тарифами). Записи `Member` (номер, ФИО, компания), `Reservation` (дата, время, тип, длительность). Вложенный класс `Invoice` (счёт с расчётом по часам и доп. услугами).

**Вариант 30.** Система «Платформа подкастов». Перечисление `EpisodeType` (обычный, бонусный, трейлер, интервью с доступом). Записи `Podcast` (ID, название, автор, категория), `Episode` (номер, название, длительность, тип). Вложенный класс `Subscription` (подписка с уровнем и датами).

## 6. Методические указания к самостоятельной работе

1. **Проектирование перечисления.** Перед реализацией перечисления определите:
   - полный набор констант и их порядок;
   - поля, необходимые для каждой константы;
   - набор методов, в том числе абстрактных, если поведение констант различается;
   - необходимость реализации интерфейсов.

2. **Применение записей.** Записи следует применять для неизменяемых носителей данных. При проектировании записи:
   - определите набор компонентов и их типы;
   - реализуйте компактный конструктор с валидацией всех ограничений;
   - при необходимости добавьте дополнительные методы, использующие компоненты;
   - помните о неявной генерации `equals()`, `hashCode()`, `toString()`.

3. **Паттерн Builder.** Применяйте Builder, когда класс имеет более четырёх параметров конструирования, часть из которых необязательна. Реализация Builder:
   - объявляется как `public static` вложенный класс;
   - содержит обязательные параметры в конструкторе, необязательные — с значениями по умолчанию;
   - все методы-мутаторы возвращают `this` для fluent-интерфейса;
   - метод `build()` выполняет итоговую валидацию и создаёт объект.

4. **Неизменяемость.** Записи и объекты, созданные Builder'ом, должны быть неизменяемыми:
   - все поля — `final`;
   - коллекции — защищённые от модификации (`Collections.unmodifiableList`, `List.copyOf`);
   - при необходимости изменения — создание нового объекта (методы вида `withXxx()`).

5. **Валидация.** Валидация должна выполняться:
   - в компактных конструкторах записей;
   - в конструкторах Builder;
   - в методах-мутаторах Builder;
   - с использованием `IllegalArgumentException` для некорректных значений;
   - с использованием `IllegalStateException` для нарушений состояния.

6. **Тестирование.** Перед сдачей работы проверьте:
   - корректность всех констант перечисления и их поведения;
   - работу валидации в записях и Builder;
   - неизменяемость созданных объектов;
   - обработку граничных случаев (пустые строки, `null`, отрицательные значения).

7. **Применение ИИ.** При использовании средств ИИ:
   - генерируйте по отдельности перечисления, записи и Builder;
   - обязательно проверяйте сгенерированный код на соответствие контрактам;
   - не делегируйте ИИ проектирование иерархий и выбор конструкций.

8. **Оформление отчёта.** Отчёт должен содержать:
   - листинги всех файлов проекта с комментариями;
   - протокол работы демонстрационного класса;
   - обоснование выбора между классом, перечислением и записью для каждой сущности;
   - ответы на контрольные вопросы;
   - выводы по проделанной работе.

## 7. Контрольные вопросы

1. Что такое перечисление (`enum`) в языке Java? Чем оно отличается от перечислений в других языках?
2. Какие члены может содержать перечисление? Может ли перечисление иметь конструктор?
3. Как реализовать различное поведение для разных констант перечисления?
4. Может ли перечисление реализовывать интерфейсы? Может ли оно наследовать классы?
5. Какие специализированные коллекции предоставляет Java для работы с перечислениями?
6. Какие виды вложенных типов существуют в языке Java? В чём их различие?
7. Что такое статический вложенный класс? Чем он отличается от внутреннего класса?
8. Для каких целей применяется паттерн Builder? Какова роль в нём вложенного класса?
9. Что такое запись (`record`)? Какие методы генерируются компилятором автоматически?
10. Что такое компактный конструктор записи? Для каких целей он применяется?
11. Каковы ограничения записей? Почему записи являются неизменяемыми?
12. В каких случаях следует применять перечисление, запись, обычный класс?
13. Каким образом обеспечивается неизменяемость объектов, созданных посредством Builder?

## 8. Рекомендуемые источники

1. Блох Дж. *Java. Эффективное программирование.* — М.: Питер. — Правило 34 (использование интерфейсов для определения типов), правило 2 (конструкторы с множеством параметров — Builder).
2. Хорстманн К. *Java. Библиотека профессионала. Том 1.* — М.: Вильямс. — Разделы, посвящённые перечислениям и записям.
3. Oracle Java Tutorials. Enum Types. URL: https://docs.oracle.com/javase/tutorial/java/javaOO/enum.html
4. Oracle Java Language Specification. Records. URL: https://docs.oracle.com/javase/specs/jls/se17/html/jls-8.html#jls-8.10
5. Baeldung. Guide to Java Records. URL: https://www.baeldung.com/java-record-keyword
