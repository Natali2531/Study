# Лабораторная работа №4. Полиморфизм, абстрактные классы и интерфейсы

## 1. Паспорт работы

| Параметр | Значение |
|----------|----------|
| Тема | Полиморфизм, абстрактные классы и интерфейсы |
| Номер занятия в модуле | 4 из 4 |
| Продолжительность аудиторной части | 2 академических часа |
| Предшествующая подготовка | Лабораторные работы №1–3 (базовый синтаксис, инкапсуляция, наследование и композиция) |
| Тип работы | Формирование навыков проектирования гибких и расширяемых объектно-ориентированных систем |

### 1.1. Цель работы

Освоить механизмы абстрагирования и полиморфизма как фундаментальных принципов объектно-ориентированного программирования: научиться проектировать иерархии классов с использованием абстрактных классов и интерфейсов, применять полиморфизм для создания универсального кода, а также корректно переопределять методы базового класса `Object`.

### 1.2. Задачи работы

1. Изучить принцип полиморфизма и механизм динамического связывания.
2. Освоить различие между перегрузкой и переопределением методов.
3. Научиться проектировать абстрактные классы и применять абстрактные методы.
4. Освоить механизм интерфейсов, включая `default` и `static` методы.
5. Научиться применять множественную реализацию интерфейсов.
6. Изучить контракт класса `Object`: переопределение `equals`, `hashCode`, `toString`.
7. Развить навыки проектирования расширяемых систем с использованием принципов абстракции.

### 1.3. Оснащение

- JDK версии 17 или выше;
- интегрированная среда разработки IntelliJ IDEA Community Edition;
- система контроля версий Git.

## 2. Теоретические сведения

### 2.1. Полиморфизм

**Полиморфизм** — принцип объектно-ориентированного программирования, позволяющий объектам с различной внутренней структурой иметь общий интерфейс и единообразно обрабатываться в коде. Полиморфизм обеспечивает гибкость и расширяемость программных систем.

В языке Java полиморфизм реализуется посредством механизма **динамического (позднего) связывания**: конкретная версия переопределённого метода определяется не на этапе компиляции, а во время выполнения программы — в зависимости от фактического типа объекта, на который ссылается переменная.

```java
/**
 * Демонстрация полиморфизма
 */
public class PolymorphismDemo {
    public static void main(String[] args) {
        // Переменная базового типа может ссылаться на объекты подклассов
        Shape shape1 = new Circle(5.0);
        Shape shape2 = new Rectangle(4.0, 6.0);
        Shape shape3 = new Triangle(3.0, 4.0, 5.0);

        // Вызов переопределённого метода определяется типом объекта,
        // а не типом ссылки (динамическое связывание)
        Shape[] shapes = {shape1, shape2, shape3};
        for (Shape shape : shapes) {
            System.out.println(shape.getInfo());   // вызов конкретной реализации
        }
    }
}
```

### 2.2. Перегрузка и переопределение методов

В объектно-ориентированном программировании различают два механизма изменения поведения методов:

**Перегрузка (overloading)** — объявление в классе нескольких методов с одинаковым именем, но различными списками параметров. Разрешается на этапе компиляции (**раннее связывание**).

```java
public class Calculator {
    public int add(int a, int b) {
        return a + b;
    }

    public double add(double a, double b) {
        return a + b;
    }

    public int add(int a, int b, int c) {
        return a + b + c;
    }
}
```

**Переопределение (overriding)** — объявление в подклассе метода с той же сигнатурой, что и в базовом классе, с целью предоставления специализированной реализации. Разрешается во время выполнения (**позднее связывание**).

```java
public class Animal {
    public void makeSound() {
        System.out.println("Животное издаёт звук");
    }
}

public class Dog extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Собака лает: Гав-гав!");
    }
}
```

Сводная таблица различий:

| Критерий | Перегрузка | Переопределение |
|----------|------------|-----------------|
| Расположение | В одном классе | В иерархии наследования |
| Сигнатура | Имя совпадает, параметры различаются | Полное совпадение сигнатуры |
| Возвращаемый тип | Может различаться | Совпадает или ковариантен |
| Модификатор доступа | Может различаться | Не может быть строже |
| Связывание | Раннее (компиляция) | Позднее (выполнение) |
| Аннотация | Не применяется | Рекомендуется `@Override` |

### 2.3. Абстрактные классы и методы

**Абстрактный класс** — класс, один или несколько методов которого не имеют реализации и объявлены как абстрактные. Абстрактный класс не может быть инстанцирован (нельзя создать объект абстрактного класса посредством `new`).

**Абстрактный метод** — метод, объявленный без реализации (без тела). Реализация абстрактного метода обеспечивается в подклассах.

Синтаксис:

```java
/**
 * Абстрактный класс, моделирующий геометрическую фигуру
 */
public abstract class Shape {
    protected String name;
    protected String color;

    public Shape(String name, String color) {
        this.name = name;
        this.color = color;
    }

    // Абстрактные методы — реализация в подклассах
    public abstract double getArea();
    public abstract double getPerimeter();

    // Конкретный метод — общая реализация для всех подклассов
    public String getInfo() {
        return String.format("%s [%s]: площадь = %.2f, периметр = %.2f",
                             name, color, getArea(), getPerimeter());
    }
}

/**
 * Подкласс, реализующий абстрактные методы
 */
public class Circle extends Shape {
    private double radius;

    public Circle(double radius, String color) {
        super("Круг", color);
        this.radius = radius;
    }

    @Override
    public double getArea() {
        return Math.PI * radius * radius;
    }

    @Override
    public double getPerimeter() {
        return 2 * Math.PI * radius;
    }
}
```

**Правила применения абстрактных классов:**

1. Подкласс обязан реализовать все абстрактные методы базового класса, иначе он сам должен быть объявлен абстрактным.
2. Абстрактный класс может содержать как абстрактные, так и конкретные методы.
3. Абстрактный класс может иметь конструкторы, вызываемые из конструкторов подклассов.
4. Абстрактный класс может содержать поля, в том числе с инициализацией.

### 2.4. Интерфейсы

**Интерфейс** — абстрактный тип, определяющий контракт поведения без реализации (за исключением `default` и `static` методов). Класс может реализовывать несколько интерфейсов, что обеспечивает множественное наследование поведения.

Синтаксис:

```java
/**
 * Интерфейс, определяющий поведение «изменяемый размер»
 */
public interface Resizable {
    // Абстрактный метод (неявно public abstract)
    void resize(double factor);

    // Default-метод (с реализацией по умолчанию)
    default void doubleSize() {
        resize(2.0);
    }

    // Static-метод
    static Resizable createDefault() {
        return new DefaultResizable();
    }
}

/**
 * Класс, реализующий интерфейс
 */
public class Rectangle implements Resizable {
    private double width;
    private double height;

    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }

    @Override
    public void resize(double factor) {
        width *= factor;
        height *= factor;
    }
}
```

**Особенности интерфейсов:**

1. Все методы интерфейса неявно являются `public`.
2. Поля интерфейса неявно являются `public static final` (константы).
3. Начиная с Java 8, интерфейсы могут содержать `default`-методы с реализацией по умолчанию.
4. Начиная с Java 8, интерфейсы могут содержать `static`-методы.
5. Начиная с Java 9, интерфейсы могут содержать `private`-методы для повторного использования кода в `default`-методах.
6. Класс может реализовывать несколько интерфейсов, перечисляя их через запятую.

### 2.5. Множественная реализация интерфейсов

В отличие от наследования классов (в Java допускается только одиночное наследование), класс может реализовывать произвольное количество интерфейсов:

```java
public interface Printable {
    void print();
}

public interface Saveable {
    void save();
}

public interface Serializable {
    byte[] serialize();
}

/**
 * Класс реализует три интерфейса одновременно
 */
public class Document implements Printable, Saveable, Serializable {
    private String content;

    public Document(String content) {
        this.content = content;
    }

    @Override
    public void print() {
        System.out.println("Печать: " + content);
    }

    @Override
    public void save() {
        System.out.println("Сохранение: " + content);
    }

    @Override
    public byte[] serialize() {
        return content.getBytes();
    }
}
```

### 2.6. Абстрактные классы vs Интерфейсы

| Критерий | Абстрактный класс | Интерфейс |
|----------|-------------------|-----------|
| Наследование | Одиночное (`extends`) | Множественное (`implements`) |
| Поля | Любые (с состоянием) | Только константы (`public static final`) |
| Конструкторы | Есть | Нет |
| Методы | Абстрактные и конкретные | Абстрактные, `default`, `static` |
| Модификаторы доступа | Любые | Только `public` |
| Семантика | «является» (is-a) | «может» (can-do) |
| Применение | Общая основа для родственных классов | Контракт для неродственных классов |

**Общее правило:** если классы находятся в одной иерархии «является» и имеют общее состояние — применяется абстрактный класс. Если классы не связаны иерархией, но должны обладать общим поведением — применяется интерфейс.

### 2.7. Контракт класса `Object`

Класс `java.lang.Object` является базовым для всех классов в языке Java. При переопределении его методов необходимо соблюдать определённые контракты.

#### 2.7.1. Метод `toString()`

Метод `toString()` возвращает строковое представление объекта. Рекомендуется переопределять его во всех классах, моделирующих предметные сущности:

```java
@Override
public String toString() {
    return String.format("Student[name=%s, age=%d, group=%s]",
                         name, age, group);
}
```

#### 2.7.2. Метод `equals(Object obj)`

Метод `equals()` определяет отношение эквивалентности объектов. При переопределении необходимо соблюдать пять свойств:

1. **Рефлексивность:** `x.equals(x)` возвращает `true`.
2. **Симметричность:** если `x.equals(y)`, то `y.equals(x)`.
3. **Транзитивность:** если `x.equals(y)` и `y.equals(z)`, то `x.equals(z)`.
4. **Непротиворечивость:** многократные вызовы возвращают одинаковый результат.
5. **Согласованность с `null`:** `x.equals(null)` возвращает `false` для любого ненулевого `x`.

Шаблон корректной реализации:

```java
@Override
public boolean equals(Object obj) {
    // 1. Проверка ссылочного равенства
    if (this == obj) return true;
    // 2. Проверка на null и типа
    if (obj == null || getClass() != obj.getClass()) return false;
    // 3. Приведение типа
    Student other = (Student) obj;
    // 4. Сравнение полей
    return age == other.age &&
           Objects.equals(name, other.name) &&
           Objects.equals(group, other.group);
}
```

#### 2.7.3. Метод `hashCode()`

Метод `hashCode()` возвращает целочисленное хеш-значение объекта. Контракт между `equals()` и `hashCode()`:

- если два объекта равны согласно `equals()`, их `hashCode()` должен быть одинаковым;
- если `hashCode()` различается, объекты не могут быть равны (но обратное неверно).

```java
@Override
public int hashCode() {
    return Objects.hash(name, age, group);
}
```

**Важно:** при переопределении `equals()` **обязательно** следует переопределять и `hashCode()`. В противном случае объект некорректно функционирует в хеш-коллекциях (`HashMap`, `HashSet`).

### 2.8. Записи (Records) — современная альтернатива

Начиная с Java 16, для классов, основным назначением которых является хранение данных, рекомендуется применять **записи (records)**. Запись автоматически генерирует конструктор, геттеры, `equals()`, `hashCode()` и `toString()`:

```java
/**
 * Запись, моделирующая точку на плоскости.
 * Компилятор автоматически генерирует все необходимые методы.
 */
public record Point(double x, double y) {
    // Компактный конструктор с валидацией
    public Point {
        if (Double.isNaN(x) || Double.isNaN(y)) {
            throw new IllegalArgumentException("Координаты не могут быть NaN");
        }
    }

    // Дополнительный метод
    public double distanceToOrigin() {
        return Math.sqrt(x * x + y * y);
    }
}
```

Записи являются `final`-классами, неявно наследующими от `java.lang.Record`. Поля записи неявно являются `private final`.

## 3. Примеры выполнения

### 3.1. Пример 1. Абстрактный класс «Фигуры» с полиморфизмом

```java
/**
 * Абстрактный базовый класс, моделирующий геометрическую фигуру
 */
public abstract class Shape {
    protected String name;
    protected String color;

    public Shape(String name, String color) {
        this.name = name;
        setColor(color);
    }

    public String getName() { return name; }
    public String getColor() { return color; }

    public void setColor(String color) {
        if (color == null || color.trim().isEmpty()) {
            throw new IllegalArgumentException("Цвет не может быть пустым");
        }
        this.color = color;
    }

    // Абстрактные методы — реализация предоставляется подклассами
    public abstract double getArea();
    public abstract double getPerimeter();

    // Конкретный метод, использующий абстрактные (шаблонный метод)
    public String getInfo() {
        return String.format("%s [%s]: площадь = %.2f, периметр = %.2f",
                             name, color, getArea(), getPerimeter());
    }

    @Override
    public String toString() {
        return getInfo();
    }
}

/**
 * Подкласс «Круг»
 */
public class Circle extends Shape {
    private double radius;

    public Circle(double radius, String color) {
        super("Круг", color);
        setRadius(radius);
    }

    public double getRadius() { return radius; }

    public void setRadius(double radius) {
        if (radius <= 0) {
            throw new IllegalArgumentException("Радиус должен быть положительным");
        }
        this.radius = radius;
    }

    @Override
    public double getArea() {
        return Math.PI * radius * radius;
    }

    @Override
    public double getPerimeter() {
        return 2 * Math.PI * radius;
    }
}

/**
 * Подкласс «Прямоугольник»
 */
public class Rectangle extends Shape {
    private double width;
    private double height;

    public Rectangle(double width, double height, String color) {
        super("Прямоугольник", color);
        setWidth(width);
        setHeight(height);
    }

    public double getWidth() { return width; }
    public double getHeight() { return height; }

    public void setWidth(double width) {
        if (width <= 0) {
            throw new IllegalArgumentException("Ширина должна быть положительной");
        }
        this.width = width;
    }

    public void setHeight(double height) {
        if (height <= 0) {
            throw new IllegalArgumentException("Высота должна быть положительной");
        }
        this.height = height;
    }

    @Override
    public double getArea() {
        return width * height;
    }

    @Override
    public double getPerimeter() {
        return 2 * (width + height);
    }
}

/**
 * Подкласс «Треугольник»
 */
public class Triangle extends Shape {
    private double sideA;
    private double sideB;
    private double sideC;

    public Triangle(double sideA, double sideB, double sideC, String color) {
        super("Треугольник", color);
        setSides(sideA, sideB, sideC);
    }

    public double getSideA() { return sideA; }
    public double getSideB() { return sideB; }
    public double getSideC() { return sideC; }

    public void setSides(double sideA, double sideB, double sideC) {
        if (sideA <= 0 || sideB <= 0 || sideC <= 0) {
            throw new IllegalArgumentException("Стороны должны быть положительными");
        }
        if (sideA + sideB <= sideC || sideA + sideC <= sideB || sideB + sideC <= sideA) {
            throw new IllegalArgumentException(
                "Нарушено неравенство треугольника: сумма любых двух сторон " +
                "должна превышать третью");
        }
        this.sideA = sideA;
        this.sideB = sideB;
        this.sideC = sideC;
    }

    @Override
    public double getArea() {
        double p = (sideA + sideB + sideC) / 2;   // полупериметр
        return Math.sqrt(p * (p - sideA) * (p - sideB) * (p - sideC));
    }

    @Override
    public double getPerimeter() {
        return sideA + sideB + sideC;
    }
}
```

Демонстрационный класс:

```java
/**
 * Демонстрация полиморфизма на иерархии фигур
 */
public class ShapeDemo {
    public static void main(String[] args) {
        // Полиморфный массив: переменные базового типа ссылаются
        // на объекты различных подклассов
        Shape[] shapes = {
            new Circle(5.0, "красный"),
            new Rectangle(4.0, 6.0, "синий"),
            new Triangle(3.0, 4.0, 5.0, "зелёный"),
            new Circle(2.5, "жёлтый"),
            new Rectangle(10.0, 3.0, "фиолетовый")
        };

        // Полиморфный вызов: конкретная реализация определяется
        // типом объекта во время выполнения
        System.out.println("=== Геометрические фигуры ===");
        for (Shape shape : shapes) {
            System.out.println(shape.getInfo());
        }

        // Агрегированные вычисления
        double totalArea = 0;
        double totalPerimeter = 0;
        for (Shape shape : shapes) {
            totalArea += shape.getArea();
            totalPerimeter += shape.getPerimeter();
        }

        System.out.printf("%nСуммарная площадь: %.2f%n", totalArea);
        System.out.printf("Суммарный периметр: %.2f%n", totalPerimeter);

        // Поиск фигуры с максимальной площадью
        Shape largest = shapes[0];
        for (Shape shape : shapes) {
            if (shape.getArea() > largest.getArea()) {
                largest = shape;
            }
        }
        System.out.println("\nФигура с наибольшей площадью: " + largest.getName());
    }
}
```

### 3.2. Пример 2. Интерфейсы и множественная реализация

```java
/**
 * Интерфейс «Печатаемый»
 */
public interface Printable {
    void print();

    default void printWithHeader(String header) {
        System.out.println("=== " + header + " ===");
        print();
        System.out.println("==================");
    }
}

/**
 * Интерфейс «Сохраняемый»
 */
public interface Saveable {
    void saveToFile(String filename);
    void loadFromFile(String filename);
}

/**
 * Интерфейс «Сериализуемый в JSON»
 */
public interface JsonSerializable {
    String toJson();

    static String wrapJson(String content) {
        return "{" + content + "}";
    }
}

/**
 * Класс «Документ», реализующий три интерфейса
 */
public class Document implements Printable, Saveable, JsonSerializable {
    private String title;
    private String content;

    public Document(String title, String content) {
        setTitle(title);
        setContent(content);
    }

    public String getTitle() { return title; }
    public String getContent() { return content; }

    public void setTitle(String title) {
        if (title == null || title.trim().isEmpty()) {
            throw new IllegalArgumentException("Заголовок не может быть пустым");
        }
        this.title = title;
    }

    public void setContent(String content) {
        if (content == null) {
            throw new IllegalArgumentException("Содержимое не может быть null");
        }
        this.content = content;
    }

    @Override
    public void print() {
        System.out.println("Документ: " + title);
        System.out.println(content);
    }

    @Override
    public void saveToFile(String filename) {
        System.out.println("Сохранение документа '" + title + "' в файл " + filename);
        // Реальная реализация записи в файл
    }

    @Override
    public void loadFromFile(String filename) {
        System.out.println("Загрузка документа из файла " + filename);
        // Реальная реализация чтения из файла
    }

    @Override
    public String toJson() {
        return JsonSerializable.wrapJson(
            "\"title\":\"" + title + "\"," +
            "\"content\":\"" + content + "\""
        );
    }

    @Override
    public String toString() {
        return "Document[title=" + title + "]";
    }
}

/**
 * Демонстрация работы с интерфейсами
 */
public class InterfacesDemo {
    public static void main(String[] args) {
        Document doc = new Document("Отчёт", "Содержимое отчёта за 2026 год");

        // Вызов методов различных интерфейсов
        doc.print();
        doc.printWithHeader("Печать документа");
        doc.saveToFile("report.txt");
        System.out.println("JSON: " + doc.toJson());

        // Полиморфизм через интерфейсы
        Printable printable = doc;
        printable.print();

        Saveable saveable = doc;
        saveable.saveToFile("backup.txt");

        // Статический метод интерфейса
        String wrapped = JsonSerializable.wrapJson("\"key\":\"value\"");
        System.out.println("\nСтатический метод: " + wrapped);
    }
}
```

### 3.3. Пример 3. Контракт `Object`: `equals`, `hashCode`, `toString`

```java
import java.util.HashSet;
import java.util.Objects;
import java.util.Set;

/**
 * Класс, моделирующий студента.
 * Демонстрирует корректное переопределение методов Object.
 */
public class Student {
    private String name;
    private int age;
    private String group;
    private double averageGrade;

    public Student(String name, int age, String group, double averageGrade) {
        setName(name);
        setAge(age);
        setGroup(group);
        setAverageGrade(averageGrade);
    }

    // Геттеры
    public String getName() { return name; }
    public int getAge() { return age; }
    public String getGroup() { return group; }
    public double getAverageGrade() { return averageGrade; }

    // Сеттеры с валидацией
    public void setName(String name) {
        if (name == null || name.trim().isEmpty()) {
            throw new IllegalArgumentException("Имя не может быть пустым");
        }
        this.name = name;
    }

    public void setAge(int age) {
        if (age < 16 || age > 100) {
            throw new IllegalArgumentException(
                "Возраст должен быть в диапазоне от 16 до 100");
        }
        this.age = age;
    }

    public void setGroup(String group) {
        if (group == null || group.trim().isEmpty()) {
            throw new IllegalArgumentException("Группа не может быть пустой");
        }
        this.group = group;
    }

    public void setAverageGrade(double averageGrade) {
        if (averageGrade < 0.0 || averageGrade > 5.0) {
            throw new IllegalArgumentException(
                "Средний балл должен быть в диапазоне от 0.0 до 5.0");
        }
        this.averageGrade = averageGrade;
    }

    /**
     * Переопределение toString() для удобного вывода
     */
    @Override
    public String toString() {
        return String.format("Student{name='%s', age=%d, group='%s', averageGrade=%.2f}",
                             name, age, group, averageGrade);
    }

    /**
     * Переопределение equals().
     * Два студента считаются равными, если совпадают имя, возраст и группа.
     * Средний балл не учитывается (студент — тот же человек, даже если балл изменился).
     */
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        Student other = (Student) obj;
        return age == other.age &&
               Objects.equals(name, other.name) &&
               Objects.equals(group, other.group);
    }

    /**
     * Переопределение hashCode().
     * Использует те же поля, что и equals(), для соблюдения контракта.
     */
    @Override
    public int hashCode() {
        return Objects.hash(name, age, group);
    }
}

/**
 * Демонстрация работы контракта Object
 */
public class ObjectContractDemo {
    public static void main(String[] args) {
        // Создание объектов
        Student s1 = new Student("Иванов И.И.", 19, "ПИ-21", 4.5);
        Student s2 = new Student("Иванов И.И.", 19, "ПИ-21", 4.8);   // тот же студент, другой балл
        Student s3 = new Student("Петров П.П.", 20, "ПИ-21", 4.5);

        // Проверка equals()
        System.out.println("=== Проверка equals() ===");
        System.out.println("s1 == s2 (ссылочное): " + (s1 == s2));
        System.out.println("s1.equals(s2): " + s1.equals(s2));   // true (один студент)
        System.out.println("s1.equals(s3): " + s1.equals(s3));   // false (разные студенты)

        // Проверка hashCode()
        System.out.println("\n=== Проверка hashCode() ===");
        System.out.println("s1.hashCode(): " + s1.hashCode());
        System.out.println("s2.hashCode(): " + s2.hashCode());   // совпадает с s1
        System.out.println("s3.hashCode(): " + s3.hashCode());   // отличается

        // Проверка toString()
        System.out.println("\n=== Проверка toString() ===");
        System.out.println(s1);
        System.out.println(s3);

        // Демонстрация работы в HashSet
        System.out.println("\n=== Работа в HashSet ===");
        Set<Student> students = new HashSet<>();
        students.add(s1);
        students.add(s2);   // не добавится, так как equals() = true
        students.add(s3);
        System.out.println("Количество элементов в множестве: " + students.size());   // 2
        System.out.println("Содержимое множества: " + students);
    }
}
```

## 4. Задания на паре

### Задание 4.1. Абстрактный класс «Платёжные системы»

Разработайте иерархию классов, моделирующих различные платёжные системы.

**Абстрактный базовый класс `PaymentSystem`:**
- Поля: `name` (String), `commissionPercent` (double).
- Абстрактные методы:
  - `pay(double amount)` — выполняет платёж на указанную сумму;
  - `getTransactionInfo(String transactionId)` — возвращает информацию о транзакции;
  - `calculateFee(double amount)` — возвращает размер комиссии.
- Конкретные методы:
  - `getInfo()` — возвращает общую информацию о платёжной системе;
  - геттеры и сеттеры с валидацией.

**Подкласс `CardPayment` (оплата банковской картой):**
- Дополнительные поля: `cardNumber` (String, последние 4 цифры), `cardType` (String — Visa, MasterCard, МИР).
- Реализация абстрактных методов: комиссия 1.5% для Visa/MasterCard, 1% для МИР.

**Подкласс `ElectronicWallet` (электронный кошелёк):**
- Дополнительные поля: `walletId` (String), `walletType` (String — YooMoney, QIWI, WebMoney).
- Реализация абстрактных методов: комиссия 2% для всех типов.

**Подкласс `MobilePayment` (мобильный платёж):**
- Дополнительные поля: `phoneNumber` (String), `operator` (String — МТС, Билайн, МегаФон, Теле2).
- Реализация абстрактных методов: комиссия 3% для всех операторов.

**Демонстрация:**
- Создайте массив `PaymentSystem[]` с объектами различных типов.
- Выполните платежи на различные суммы.
- Выведите информацию о каждой транзакции.
- Вычислите суммарную комиссию по всем платежам.
- Определите платёжную систему с наименьшей комиссией для заданной суммы.

**Пример выполнения программы:**

```
=== Платёжные системы ===
Оплата картой Visa ****1234: сумма 1000.00 руб., комиссия 15.00 руб.
Оплата с кошелька YooMoney W1234567: сумма 2000.00 руб., комиссия 40.00 руб.
Мобильный платёж +7***1234567 (МТС): сумма 500.00 руб., комиссия 15.00 руб.

Суммарная комиссия: 70.00 руб.
Платёжная система с наименьшей комиссией для суммы 1000 руб.: CardPayment (10.00 руб.)
```

---

### Задание 4.2. Интерфейсы «Сортируемые и сравниваемые элементы»

Разработайте систему классов, реализующих интерфейсы для сортировки и сравнения объектов.

**Интерфейс `Comparable<T>`** (используется стандартный из `java.lang`):
- Метод `int compareTo(T other)` — сравнивает текущий объект с другим.

**Интерфейс `Filterable`:**
- Метод `boolean matches(String criteria)` — проверяет соответствие объекта заданному критерию.

**Интерфейс `Displayable`:**
- Метод `String getDisplayString()` — возвращает строку для отображения;
- `default`-метод `printFormatted()` — выводит объект в формате с разделителями.

**Класс `Book`** (реализует `Comparable<Book>`, `Filterable`, `Displayable`):
- Поля: `title` (String), `author` (String), `year` (int), `price` (double), `pages` (int).
- `compareTo()` — сравнение по году издания (по возрастанию);
- `matches(criteria)` — проверка вхождения критерия в название или автора (без учёта регистра);
- `getDisplayString()` — форматированная строка с информацией о книге.

**Класс `Product`** (реализует `Comparable<Product>`, `Filterable`, `Displayable`):
- Поля: `name` (String), `category` (String), `price` (double), `rating` (double).
- `compareTo()` — сравнение по цене (по возрастанию);
- `matches(criteria)` — проверка вхождения критерия в название или категорию;
- `getDisplayString()` — форматированная строка с информацией о товаре.

**Демонстрационный класс `Catalog<T>`:**
- Поле: `items` (массив объектов типа `T`, ограниченного `Comparable<T> & Displayable & Filterable`).
- Методы:
  - `addItem(T item)` — добавить элемент;
  - `sortItems()` — отсортировать элементы (используя `compareTo`);
  - `filterByCriteria(String criteria)` — вернуть список элементов, соответствующих критерию;
  - `displayAll()` — вывести все элементы в отформатированном виде.

**Демонстрация:**
- Создайте каталог книг и каталог товаров.
- Добавьте в каждый не менее 5 элементов.
- Отсортируйте элементы.
- Выполните фильтрацию по различным критериям.
- Выведите результаты.

---

### Задание 4.3. Комбинированное задание «Система уведомлений»

Разработайте систему уведомлений, использующую абстрактные классы, интерфейсы и контракт `Object`.

**Интерфейс `Sendable`:**
- Метод `send(String recipient)` — отправляет уведомление получателю;
- Метод `boolean isDelivered()` — возвращает статус доставки.

**Интерфейс `Trackable`:**
- Метод `String getTrackingId()` — возвращает идентификатор для отслеживания;
- `default`-метод `getTrackingInfo()` — возвращает строку с информацией об отслеживании.

**Абстрактный класс `Notification`** (реализует `Sendable`, `Trackable`):
- Поля: `id` (String), `message` (String), `createdAt` (long — время создания в миллисекундах), `delivered` (boolean).
- Абстрактные методы:
  - `getDeliveryChannel()` — возвращает канал доставки (Email, SMS, Push);
  - `formatMessage()` — форматирует сообщение для конкретного канала.
- Конкретные методы:
  - реализация `send()` — устанавливает `delivered = true` и выводит информацию;
  - реализация `isDelivered()`, `getTrackingId()`, `getTrackingInfo()`;
  - переопределение `equals()`, `hashCode()`, `toString()`.

**Подкласс `EmailNotification`:**
- Дополнительные поля: `subject` (String), `recipientEmail` (String).
- Реализация `getDeliveryChannel()` — возвращает `"Email"`.
- Реализация `formatMessage()` — возвращает сообщение в формате письма.

**Подкласс `SmsNotification`:**
- Дополнительные поля: `recipientPhone` (String), `maxLength` (int, по умолчанию 160).
- Реализация `getDeliveryChannel()` — возвращает `"SMS"`.
- Реализация `formatMessage()` — возвращает сообщение, обрезанное до `maxLength`.

**Подкласс `PushNotification`:**
- Дополнительные поля: `title` (String), `applicationName` (String), `priority` (String — LOW, NORMAL, HIGH).
- Реализация `getDeliveryChannel()` — возвращает `"Push"`.
- Реализация `formatMessage()` — возвращает сообщение в формате push-уведомления.

**Демонстрация:**
- Создайте массив `Notification[]` с объектами различных типов.
- Отправьте все уведомления.
- Выведите информацию об отслеживании каждого.
- Создайте `HashSet<Notification>` и продемонстрируйте работу `equals()` и `hashCode()`.
- Определите количество уведомлений по каждому каналу доставки.

## 5. Задание для самостоятельной работы

Разработать систему классов согласно своему варианту, включающую:

1. Абстрактный базовый класс с не менее чем 2 абстрактными методами и не менее чем 2 конкретными методами.
2. Минимум 3 подкласса, реализующих абстрактные методы.
3. Не менее 2 интерфейсов, реализуемых классами иерархии (один из интерфейсов должен содержать `default`-метод).
4. Корректное переопределение `equals()`, `hashCode()`, `toString()` в базовом классе.
5. Демонстрацию полиморфизма через массив объектов базового класса.
6. Демонстрацию работы с хеш-коллекцией (`HashSet` или `HashMap`).

### Варианты заданий

**Вариант 1.** Система «Медиаплеер». Абстрактный класс `MediaFile` (название, длительность, размер). Подклассы: `AudioFile` (исполнитель, битрейт), `VideoFile` (разрешение, кодек), `ImageFile` (разрешение, формат). Интерфейсы: `Playable` (воспроизведение, пауза), `Shareable` (поделиться, получить ссылку).

**Вариант 2.** Система «Электронная библиотека». Абстрактный класс `LibraryResource` (идентификатор, название, год). Подклассы: `EBook` (автор, формат), `Audiobook` (рассказчик, длительность), `VideoLecture` (преподаватель, качество). Интерфейсы: `Downloadable` (скачать, размер файла), `Borrowable` (взять, вернуть).

**Вариант 3.** Система «Служба доставки». Абстрактный класс `Delivery` (номер, адрес, стоимость). Подклассы: `StandardDelivery` (срок), `ExpressDelivery` (срок, приоритет), `InternationalDelivery` (страна, таможня). Интерфейсы: `Trackable` (отследить, статус), `Insurable` (застраховать, сумма).

**Вариант 4.** Система «Платформа онлайн-обучения». Абстрактный класс `LearningContent` (название, автор, длительность). Подклассы: `VideoLesson` (разрешение, субтитры), `InteractiveTask` (сложность, баллы), `Quiz` (количество вопросов, проходной балл). Интерфейсы: `Gradable` (оценить, балл), `CertificateEligible` (проверить право на сертификат).

**Вариант 5.** Система «Умный дом». Абстрактный класс `SmartDevice` (название, производитель, статус). Подклассы: `SmartLight` (яркость, цвет), `SmartThermostat` (температура, режим), `SmartLock` (статус блокировки, код). Интерфейсы: `Controllable` (включить, выключить), `Scheduleable` (запланировать, расписание).

**Вариант 6.** Система «Банковские операции». Абстрактный класс `BankOperation` (номер, сумма, дата). Подклассы: `Transfer` (счёт получателя, назначение), `Payment` (получатель, реквизиты), `Withdrawal` (тип: наличные/перевод). Интерфейсы: `Reversible` (откатить, проверить возможность), `Verifiable` (проверить, статус верификации).

**Вариант 7.** Система «Платформа электронной коммерции». Абстрактный класс `MarketplaceItem` (название, продавец, цена). Подклассы: `PhysicalProduct` (вес, габариты), `DigitalProduct` (формат, ссылка), `Service` (длительность, исполнитель). Интерфейсы: `Reviewable` (добавить отзыв, рейтинг), `Discountable` (применить скидку, итоговая цена).

**Вариант 8.** Система «Платформа для мероприятий». Абстрактный класс `Event` (название, дата, место). Подклассы: `Conference` (спикеры, тематика), `Concert` (исполнители, жанр), `Workshop` (ведущий, количество мест). Интерфейсы: `Bookable` (забронировать, отменить), `Ticketed` (цена билета, количество мест).

**Вариант 9.** Система «Социальная сеть». Абстрактный класс `Content` (автор, дата, текст). Подклассы: `Post` (количество лайков, комментарии), `Story` (длительность, фильтры), `Reel` (длительность, музыка). Интерфейсы: `Shareable` (поделиться, платформы), `Moderatable` (проверить, скрыть).

**Вариант 10.** Система «Платформа бронирования». Абстрактный класс `Reservation` (номер, клиент, дата). Подклассы: `HotelReservation` (номер, ночи), `RestaurantReservation` (столик, время), `CarRental` (автомобиль, дни). Интерфейсы: `Cancellable` (отменить, штраф), `Modifiable` (изменить, новые параметры).

**Вариант 11.** Система «Платформа аналитики». Абстрактный класс `Report` (название, период, автор). Подклассы: `FinancialReport` (доход, расход), `MarketingReport` (охват, конверсия), `TechnicalReport` (метрики, инциденты). Интерфейсы: `Exportable` (экспорт, формат), `Shareable` (поделиться, доступ).

**Вариант 12.** Система «Управление проектами». Абстрактный класс `ProjectTask` (название, исполнитель, срок). Подклассы: `DevelopmentTask` (сложность, технологии), `DesignTask` (макет, инструменты), `TestingTask` (количество тестов, покрытие). Интерфейсы: `Assignable` (назначить, переназначить), `Estimatable` (оценить, часы).

**Вариант 13.** Система «Платформа подкастов». Абстрактный класс `PodcastContent` (название, автор, длительность). Подклассы: `Episode` (номер, описание), `Trailer` (длительность, анонс), `BonusEpisode` (уровень подписки). Интерфейсы: `Downloadable` (скачать, качество), `Streamable` (воспроизвести, буферизация).

**Вариант 14.** Система «Медицинская информационная система». Абстрактный класс `MedicalRecord` (пациент, дата, врач). Подклассы: `DiagnosisRecord` (диагноз, код МКБ), `PrescriptionRecord` (лекарства, дозировка), `ProcedureRecord` (процедура, результат). Интерфейсы: `Confidential` (проверить доступ, зашифровать), `Archivable` (архивировать, срок хранения).

**Вариант 15.** Система «Платформа путешествий». Абстрактный класс `TravelItem` (название, направление, стоимость). Подклассы: `Tour` (длительность, отель), `Flight` (авиакомпания, пересадки), `Excursion` (гид, длительность). Интерфейсы: `Bookable` (забронировать, дата), `Reviewable` (отзыв, рейтинг).

**Вариант 16.** Система «Управление складом». Абстрактный класс `WarehouseItem` (артикул, название, количество). Подклассы: `RawMaterial` (поставщик, срок годности), `FinishedProduct` (цена, категория), `Packaging` (тип, размер). Интерфейсы: `Trackable` (отследить, местоположение), `Orderable` (заказать, минимальное количество).

**Вариант 17.** Система «Платформа фриланса». Абстрактный класс `FreelanceItem` (название, автор, цена). Подклассы: `Project` (длительность, навыки), `Contest` (призовой фонд, срок), `Consultation` (длительность, специализация). Интерфейсы: `Biddable` (сделать ставку, сумма), `Reviewable` (отзыв, рейтинг).

**Вариант 18.** Система «Платформа новостей». Абстрактный класс `NewsContent` (заголовок, автор, дата). Подклассы: `Article` (текст, категория), `Interview` (респондент, вопросы), `Investigation` (длительность, источники). Интерфейсы: `Publishable` (опубликовать, канал), `Moderatable` (проверить, статус).

**Вариант 19.** Система «Управление автопарком». Абстрактный класс `FleetUnit` (номер, марка, статус). Подклассы: `PassengerCar` (вместимость, класс), `CargoVehicle` (грузоподъёмность, тип кузова), `SpecialVehicle` (назначение, оборудование). Интерфейсы: `Serviceable` (ТО, пробег), `Trackable` (GPS, местоположение).

**Вариант 20.** Система «Платформа благотворительности». Абстрактный класс `CharityItem` (название, организация, цель). Подклассы: `Campaign` (срок, целевая сумма), `Donation` (сумма, жертвователь), `VolunteerActivity` (дата, количество волонтёров). Интерфейсы: `Fundable` (пожертвовать, сумма), `Reportable` (отчёт, использование средств).

**Вариант 21.** Система «Платформа видеостриминга». Абстрактный класс `VideoContent` (название, длительность, качество). Подклассы: `Movie` (режиссёр, жанр), `Series` (количество сезонов, статус), `Documentary` (тема, страна). Интерфейсы: `Streamable` (воспроизвести, буферизация), `Rateable` (оценить, рейтинг).

**Вариант 22.** Система «Управление недвижимостью». Абстрактный класс `Property` (адрес, площадь, стоимость). Подклассы: `Apartment` (комнаты, этаж), `House` (участок, этажи), `CommercialProperty` (назначение, арендная ставка). Интерфейсы: `Leasable` (сдать в аренду, срок), `Sellable` (продать, условия).

**Вариант 23.** Система «Платформа онлайн-банкинга». Абстрактный класс `FinancialProduct` (название, ставка, срок). Подклассы: `Deposit` (сумма, капитализация), `Loan` (сумма, тип платежа), `InvestmentAccount` (портфель, риск). Интерфейсы: `Calculable` (рассчитать доход, формула), `Comparable` (сравнить по доходности).

**Вариант 24.** Система «Управление персоналом». Абстрактный класс `HRDocument` (номер, сотрудник, дата). Подклассы: `EmploymentOrder` (должность, оклад), `VacationRequest` (дни, тип), `BusinessTrip` (место, длительность). Интерфейсы: `Approvable` (согласовать, статус), `Archivable` (архивировать, срок).

**Вариант 25.** Система «Платформа киберспорта». Абстрактный класс `EsportsContent` (название, игра, дата). Подклассы: `Tournament` (призовой фонд, участники), `Match` (команды, формат), `Training` (длительность, тренер). Интерфейсы: `Streamable` (транслировать, платформа), `Betable` (сделать ставку, коэффициент).

**Вариант 26.** Система «Управление цепочками поставок». Абстрактный класс `SupplyChainElement` (идентификатор, этап, статус). Подклассы: `Supplier` (страна, рейтинг), `Shipment` (вес, маршрут), `Warehouse` (локация, вместимость). Интерфейсы: `Trackable` (отследить, местоположение), `Optimizable` (оптимизировать, метрика).

**Вариант 27.** Система «Платформа онлайн-консультаций». Абстрактный класс `Consultation` (специалист, длительность, цена). Подклассы: `MedicalConsultation` (специализация, лицензия), `LegalConsultation` (отрасль права, опыт), `PsychologicalConsultation` (методика, образование). Интерфейсы: `Bookable` (записаться, время), `Recordable` (записать, согласие).

**Вариант 28.** Система «Управление событиями». Абстрактный класс `EventItem` (название, дата, место). Подклассы: `CorporateEvent` (бренд, бюджет), `PrivateEvent` (тип, количество гостей), `PublicEvent` (тематика, вместимость). Интерфейсы: `Organizable` (организовать, план), `Cancellable` (отменить, условия).

**Вариант 29.** Система «Платформа краудфандинга». Абстрактный класс `FundingProject` (название, автор, цель). Подклассы: `CreativeProject` (категория, прототип), `SocialProject` (благотворительная цель, бенефициары), `StartupProject` (бизнес-план, доля). Интерфейсы: `Fundable` (профинансировать, сумма), `Updatable` (обновить, отчёт).

**Вариант 30.** Система «Управление образовательными программами». Абстрактный класс `EducationalProgram` (название, уровень, длительность). Подклассы: `BachelorProgram` (специальность, аккредитация), `MasterProgram` (направление, профиль), `ProfessionalProgram` (квалификация, часы). Интерфейсы: `Accreditable` (аккредитовать, срок), `Modifiable` (изменить, учебный план).

## 6. Методические указания к самостоятельной работе

1. **Проектирование иерархии.** Перед написанием кода определите:
   - общие характеристики всех сущностей (поля и конкретные методы абстрактного класса);
   - специфические характеристики каждого подвида (поля и абстрактные методы);
   - поведение, которое должно быть общим для неродственных классов (интерфейсы).

2. **Применение абстрактных классов.** Абстрактный класс должен содержать:
   - не менее двух абстрактных методов, реализация которых существенно различается в подклассах;
   - не менее двух конкретных методов, реализация которых является общей для всех подклассов;
   - защищённые поля и конструктор для инициализации общих полей.

3. **Применение интерфейсов.** Интерфейсы должны определять:
   - поведение, не связанное с иерархией наследования (например, «сохраняемый», «печатаемый»);
   - не менее одного `default`-метода с реализацией по умолчанию;
   - при необходимости — `static`-методы для вспомогательных операций.

4. **Контракт `Object`.** При переопределении `equals()`, `hashCode()`, `toString()`:
   - определите, какие поля участвуют в проверке равенства (обычно — поля, идентифицирующие сущность);
   - используйте те же поля в `hashCode()`, что и в `equals()`;
   - применяйте `Objects.equals()` для полей ссылочных типов;
   - применяйте `Objects.hash()` для вычисления хеш-кода.

5. **Демонстрация полиморфизма.** В демонстрационном классе:
   - создайте массив объектов абстрактного базового класса;
   - заполните его объектами различных подклассов;
   - в цикле вызывайте абстрактные и конкретные методы;
   - продемонстрируйте работу интерфейсов через ссылки интерфейсного типа.

6. **Работа с хеш-коллекциями.** Создайте `HashSet` или `HashMap` с объектами классов иерархии и продемонстрируйте:
   - корректность работы `equals()` и `hashCode()`;
   - невозможность добавления эквивалентных объектов в множество;
   - корректный поиск объектов в карте.

7. **Тестирование.** Перед сдачей работы убедитесь, что:
   - абстрактный класс не может быть инстанцирован;
   - все подклассы реализуют абстрактные методы;
   - полиморфные вызовы работают корректно;
   - контракт `equals`/`hashCode` соблюдён;
   - интерфейсные методы работают как через ссылки базового класса, так и через интерфейсные ссылки.

8. **Оформление отчёта.** Отчёт должен содержать:
   - листинги всех файлов проекта с комментариями;
   - протокол работы программы;
   - ответы на контрольные вопросы;
   - обоснование выбора между абстрактным классом и интерфейсом для каждого случая;
   - выводы о применении полиморфизма в конкретном варианте.

## 7. Контрольные вопросы

1. Что такое полиморфизм? Какие виды полиморфизма существуют в языке Java?
2. В чём различие между перегрузкой и переопределением методов?
3. Что такое динамическое связывание? На каком этапе определяется вызываемая версия метода?
4. Что такое абстрактный класс? Можно ли создать объект абстрактного класса?
5. Что такое абстрактный метод? Обязан ли подкласс реализовывать все абстрактные методы базового класса?
6. Что такое интерфейс? В чём его отличие от абстрактного класса?
7. Что такое `default`-методы в интерфейсах? Для каких целей они применяются?
8. Может ли класс реализовывать несколько интерфейсов? Может ли класс наследовать несколько классов?
9. В каких случаях следует применять абстрактный класс, а в каких — интерфейс?
10. Какие методы класса `Object` рекомендуется переопределять в пользовательских классах?
11. Каков контракт между методами `equals()` и `hashCode()`? Почему его необходимо соблюдать?
12. Какие свойства должно удовлетворять отношение эквивалентности, реализуемое методом `equals()`?
13. Что такое записи (records)? В каких случаях их целесообразно применять?

