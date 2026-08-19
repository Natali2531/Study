# Лабораторная работа №3. Наследование и композиция

## 1. Паспорт работы

| Параметр | Значение |
|----------|----------|
| Тема | Наследование и композиция |
| Номер занятия в модуле | 3 из 4 |
| Продолжительность аудиторной части | 2 академических часа |
| Предшествующая подготовка | Лабораторные работы №1–2 (базовый синтаксис Java, инкапсуляция) |
| Тип работы | Формирование навыков повторного использования кода и моделирования отношений между классами |

### 1.1. Цель работы

Освоить механизмы повторного использования кода в объектно-ориентированном программировании: научиться применять наследование для построения иерархий классов, переопределять методы базового класса, использовать ключевое слово `super`, а также отличать наследование от композиции и применять их в соответствии с семантикой предметной области.

### 1.2. Задачи работы

1. Изучить механизм наследования и ключевое слово `extends`.
2. Освоить переопределение методов базового класса и аннотацию `@Override`.
3. Научиться применять ключевое слово `super` для обращения к членам базового класса.
4. Изучить порядок вызова конструкторов в иерархии наследования.
5. Освоить принцип композиции и отношение «has-a».
6. Развить навыки выбора между наследованием и композицией при проектировании классов.
7. Изучить модификатор `final` и его применение.

### 1.3. Оснащение

- JDK версии 17 или выше;
- интегрированная среда разработки IntelliJ IDEA Community Edition;
- система контроля версий Git.

## 2. Теоретические сведения

### 2.1. Мотивация: проблема дублирования кода

В лабораторных работах №1–2 были рассмотрены классы, каждый из которых содержал собственный набор полей и методов. Однако во многих предметных областях существуют сущности, имеющие общие характеристики. Например, классы `Student`, `Teacher` и `Administrator` могут иметь общие поля: `name`, `age`, `email`. Без механизма повторного использования кода эти поля придётся объявлять в каждом классе, что приведёт к дублированию и усложнит сопровождение.

Наследование и композиция — два основных механизма, позволяющих решать указанную проблему.

### 2.2. Наследование (отношение «is-a»)

**Наследование** — механизм языка, позволяющий создавать новый класс (**подкласс**, **производный класс**) на основе существующего класса (**базового класса**, **суперкласса**, **родительского класса**) путём заимствования его полей и методов с возможностью добавления собственных полей и методов или переопределения унаследованных.

Наследование моделирует отношение **«является» (is-a)**: подкласс является специализацией базового класса. Например, `Student` **является** `Person`, `Circle` **является** `Shape`.

Синтаксис наследования:

```java
public class Subclass extends Superclass {
    // Дополнительные поля и методы подкласса
}
```

Пример иерархии:

```java
/**
 * Базовый класс, моделирующий персону
 */
public class Person {
    protected String name;
    protected int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    public String getInfo() {
        return "Имя: " + name + ", возраст: " + age;
    }
}

/**
 * Подкласс, моделирующий студента
 */
public class Student extends Person {
    private String group;
    private double averageGrade;

    public Student(String name, int age, String group, double averageGrade) {
        super(name, age);   // вызов конструктора базового класса
        this.group = group;
        this.averageGrade = averageGrade;
    }

    public String getGroup() {
        return group;
    }

    public double getAverageGrade() {
        return averageGrade;
    }

    /**
     * Переопределение метода базового класса
     */
    @Override
    public String getInfo() {
        return super.getInfo() + ", группа: " + group +
               ", средний балл: " + averageGrade;
    }
}
```

### 2.3. Модификатор доступа `protected`

В приведённом примере поля базового класса объявлены с модификатором `protected`. Данный модификатор обеспечивает видимость членов класса в пределах:

- самого класса;
- классов того же пакета;
- подклассов, расположенных в других пакетах.

Модификатор `protected` применяется, когда необходимо предоставить подклассам прямой доступ к полям базового класса, сохранив при этом сокрытие от внешнего кода. В большинстве случаев предпочтительнее использовать `private`-поля и предоставлять к ним доступ через `protected`-методы.

### 2.4. Переопределение методов

**Переопределение (overriding)** — объявление в подклассе метода с той же сигнатурой (именем, списком параметров и возвращаемым типом), что и в базовом классе, с целью предоставления специализированной реализации.

Правила переопределения:

1. Имя метода, список параметров и возвращаемый тип должны совпадать с методом базового класса (допускается ковариантность возвращаемого типа — возврат подкласса).
2. Модификатор доступа переопределённого метода не может быть строже, чем у метода базового класса.
3. Переопределённый метод не может выбрасывать новые проверяемые исключения по сравнению с методом базового класса.
4. Рекомендуется использовать аннотацию `@Override` для проверки корректности переопределения на этапе компиляции.

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

Если аннотация `@Override` указана, но метод фактически не переопределяет метод базового класса (например, из-за опечатки в имени), компилятор выдаст ошибку.

### 2.5. Ключевое слово `super`

Ключевое слово `super` представляет ссылку на базовый класс и применяется в следующих случаях:

**1. Вызов конструктора базового класса.** Вызов `super(...)` должен быть первой инструкцией в конструкторе подкласса:

```java
public class Student extends Person {
    private String group;

    public Student(String name, int age, String group) {
        super(name, age);   // вызов конструктора Person(String, int)
        this.group = group;
    }
}
```

Если в конструкторе подкласса не указан явный вызов `super(...)`, компилятор автоматически добавляет вызов `super()` — конструктора базового класса без параметров. Если базовый класс не имеет конструктора без параметров, возникнет ошибка компиляции.

**2. Вызов переопределённого метода базового класса:**

```java
@Override
public String getInfo() {
    return super.getInfo() + ", группа: " + group;
}
```

**3. Доступ к скрытым полям базового класса** (если в подклассе объявлено поле с тем же именем):

```java
public class Employee extends Person {
    private String name;   // скрывает поле name базового класса

    public void displayNames() {
        System.out.println("Поле подкласса: " + this.name);
        System.out.println("Поле базового класса: " + super.name);
    }
}
```

### 2.6. Порядок вызова конструкторов

При создании объекта подкласса конструкторы вызываются в порядке от базового класса к подклассу (**цепочка конструкторов**):

1. Конструктор `java.lang.Object` (неявно);
2. Конструктор базового класса;
3. Конструктор подкласса.

```java
public class A {
    public A() {
        System.out.println("Конструктор A");
    }
}

public class B extends A {
    public B() {
        System.out.println("Конструктор B");
    }
}

public class C extends B {
    public C() {
        System.out.println("Конструктор C");
    }
}

// Создание объекта:
// new C();
// Вывод:
// Конструктор A
// Конструктор B
// Конструктор C
```

### 2.7. Композиция (отношение «has-a»)

**Композиция** — отношение между классами, при котором один класс содержит объекты других классов в качестве полей. Композиция моделирует отношение **«имеет» (has-a)**: класс является составным и включает в себя другие классы как части.

```java
/**
 * Класс, моделирующий двигатель
 */
public class Engine {
    private int horsepower;
    private String type;

    public Engine(int horsepower, String type) {
        this.horsepower = horsepower;
        this.type = type;
    }

    public void start() {
        System.out.println("Двигатель " + type + " (" + horsepower + " л.с.) запущен");
    }

    public void stop() {
        System.out.println("Двигатель остановлен");
    }
}

/**
 * Класс, моделирующий автомобиль.
 * Автомобиль ИМЕЕТ двигатель (композиция).
 */
public class Car {
    private String model;
    private Engine engine;   // композиция: Car has-a Engine

    public Car(String model, int horsepower, String engineType) {
        this.model = model;
        this.engine = new Engine(horsepower, engineType);
    }

    public void start() {
        System.out.println("Автомобиль " + model + ":");
        engine.start();   // делегирование вызова объекту Engine
    }
}
```

### 2.8. Наследование vs Композиция

Выбор между наследованием и композицией осуществляется на основании семантики отношения между классами:

| Критерий | Наследование | Композиция |
|----------|--------------|------------|
| Отношение | «является» (is-a) | «имеет» (has-a) |
| Пример | Студент **является** персоной | Автомобиль **имеет** двигатель |
| Связь | Жёсткая, фиксированная на этапе компиляции | Гибкая, может изменяться во время выполнения |
| Повторное использование | Через расширение функциональности | Через делегирование вызовов |
| Количество базовых классов | Один (в Java) | Неограниченное количество |

**Общее правило:** «Предпочитайте композицию наследованию» (Дж. Блох, «Java. Эффективное программирование»). Наследование следует применять только тогда, когда подкласс действительно является специализацией базового класса и планируется использование полиморфизма.

### 2.9. Модификатор `final`

Модификатор `final` запрещает дальнейшее изменение или расширение:

- **`final` класс** — класс, который нельзя наследовать (например, `String`);
- **`final` метод** — метод, который нельзя переопределить в подклассе;
- **`final` поле** — константа, значение которой не может быть изменено после инициализации.

```java
public final class MathConstants {
    public static final double PI = 3.14159265358979;

    public final double calculateCircleArea(double radius) {
        return PI * radius * radius;
    }
}

// public class Derived extends MathConstants { }  // ОШИБКА: нельзя наследовать final-класс
```

### 2.10. Класс `Object` — предок всех классов

Все классы в языке Java неявно наследуют от класса `java.lang.Object`, который определяет базовый набор методов:

- `toString()` — строковое представление объекта;
- `equals(Object obj)` — сравнение объектов на равенство;
- `hashCode()` — хеш-код объекта;
- `getClass()` — получение класса объекта во время выполнения.

При необходимости эти методы переопределяются в пользовательских классах (подробнее — в лабораторной работе №4).

## 3. Примеры выполнения

### 3.1. Пример 1. Иерархия «Сотрудники организации»

```java
/**
 * Базовый класс, моделирующий сотрудника организации
 */
public class Employee {
    protected String name;
    protected int id;
    protected double baseSalary;

    public Employee(String name, int id, double baseSalary) {
        setName(name);
        setId(id);
        setBaseSalary(baseSalary);
    }

    public String getName() { return name; }
    public int getId() { return id; }
    public double getBaseSalary() { return baseSalary; }

    public void setName(String name) {
        if (name == null || name.trim().isEmpty()) {
            throw new IllegalArgumentException("Имя не может быть пустым");
        }
        this.name = name;
    }

    public void setId(int id) {
        if (id <= 0) {
            throw new IllegalArgumentException("ID должен быть положительным");
        }
        this.id = id;
    }

    public void setBaseSalary(double baseSalary) {
        if (baseSalary < 0) {
            throw new IllegalArgumentException("Зарплата не может быть отрицательной");
        }
        this.baseSalary = baseSalary;
    }

    /**
     * Расчёт заработной платы.
     * В базовом классе возвращается базовая зарплата.
     */
    public double calculateSalary() {
        return baseSalary;
    }

    @Override
    public String toString() {
        return String.format("Сотрудник [ID: %d, Имя: %s, Зарплата: %.2f руб.]",
                             id, name, calculateSalary());
    }
}

/**
 * Подкласс «Менеджер»
 */
public class Manager extends Employee {
    private int subordinatesCount;
    private double bonusPercentage;

    public Manager(String name, int id, double baseSalary,
                   int subordinatesCount, double bonusPercentage) {
        super(name, id, baseSalary);
        setSubordinatesCount(subordinatesCount);
        setBonusPercentage(bonusPercentage);
    }

    public int getSubordinatesCount() { return subordinatesCount; }
    public double getBonusPercentage() { return bonusPercentage; }

    public void setSubordinatesCount(int subordinatesCount) {
        if (subordinatesCount < 0) {
            throw new IllegalArgumentException(
                "Количество подчинённых не может быть отрицательным");
        }
        this.subordinatesCount = subordinatesCount;
    }

    public void setBonusPercentage(double bonusPercentage) {
        if (bonusPercentage < 0 || bonusPercentage > 100) {
            throw new IllegalArgumentException(
                "Процент бонуса должен быть в диапазоне от 0 до 100");
        }
        this.bonusPercentage = bonusPercentage;
    }

    /**
     * Переопределение: менеджер получает бонус за каждого подчинённого
     */
    @Override
    public double calculateSalary() {
        double bonus = baseSalary * (bonusPercentage / 100.0) * subordinatesCount;
        return baseSalary + bonus;
    }

    @Override
    public String toString() {
        return String.format(
            "Менеджер [ID: %d, Имя: %s, Зарплата: %.2f руб., Подчинённых: %d]",
            id, name, calculateSalary(), subordinatesCount);
    }
}

/**
 * Подкласс «Разработчик»
 */
public class Developer extends Employee {
    private String programmingLanguage;
    private int yearsOfExperience;

    public Developer(String name, int id, double baseSalary,
                     String programmingLanguage, int yearsOfExperience) {
        super(name, id, baseSalary);
        setProgrammingLanguage(programmingLanguage);
        setYearsOfExperience(yearsOfExperience);
    }

    public String getProgrammingLanguage() { return programmingLanguage; }
    public int getYearsOfExperience() { return yearsOfExperience; }

    public void setProgrammingLanguage(String programmingLanguage) {
        if (programmingLanguage == null || programmingLanguage.trim().isEmpty()) {
            throw new IllegalArgumentException("Язык программирования не может быть пустым");
        }
        this.programmingLanguage = programmingLanguage;
    }

    public void setYearsOfExperience(int yearsOfExperience) {
        if (yearsOfExperience < 0) {
            throw new IllegalArgumentException("Стаж не может быть отрицательным");
        }
        this.yearsOfExperience = yearsOfExperience;
    }

    /**
     * Переопределение: разработчик получает надбавку за каждый год опыта
     */
    @Override
    public double calculateSalary() {
        double experienceBonus = baseSalary * 0.1 * yearsOfExperience;
        return baseSalary + experienceBonus;
    }

    @Override
    public String toString() {
        return String.format(
            "Разработчик [ID: %d, Имя: %s, Язык: %s, Опыт: %d лет, Зарплата: %.2f руб.]",
            id, name, programmingLanguage, yearsOfExperience, calculateSalary());
    }
}

/**
 * Подкласс «Стажёр»
 */
public class Intern extends Employee {
    private String university;
    private int internshipDurationMonths;

    public Intern(String name, int id, double baseSalary,
                  String university, int internshipDurationMonths) {
        super(name, id, baseSalary);
        setUniversity(university);
        setInternshipDurationMonths(internshipDurationMonths);
    }

    public String getUniversity() { return university; }
    public int getInternshipDurationMonths() { return internshipDurationMonths; }

    public void setUniversity(String university) {
        if (university == null || university.trim().isEmpty()) {
            throw new IllegalArgumentException("Название университета не может быть пустым");
        }
        this.university = university;
    }

    public void setInternshipDurationMonths(int internshipDurationMonths) {
        if (internshipDurationMonths <= 0) {
            throw new IllegalArgumentException("Стажировка должна иметь положительную длительность");
        }
        this.internshipDurationMonths = internshipDurationMonths;
    }

    /**
     * Переопределение: стажёр получает 50% от базовой зарплаты
     */
    @Override
    public double calculateSalary() {
        return baseSalary * 0.5;
    }

    @Override
    public String toString() {
        return String.format(
            "Стажёр [ID: %d, Имя: %s, Университет: %s, Зарплата: %.2f руб.]",
            id, name, university, calculateSalary());
    }
}
```

Демонстрационный класс:

```java
/**
 * Демонстрационный класс, отображающий работу иерархии сотрудников
 */
public class EmployeeDemo {
    public static void main(String[] args) {
        // Создание объектов различных подклассов
        Employee employee = new Employee("Иванов Иван", 1, 50000);
        Manager manager = new Manager("Петров Пётр", 2, 70000, 5, 10);
        Developer developer = new Developer("Сидоров Алексей", 3, 80000, "Java", 3);
        Intern intern = new Intern("Кузнецов Дмитрий", 4, 30000, "МГУ", 6);

        // Вывод информации (демонстрация переопределения toString)
        System.out.println("=== Сотрудники организации ===");
        System.out.println(employee);
        System.out.println(manager);
        System.out.println(developer);
        System.out.println(intern);

        // Демонстрация полиморфного поведения
        System.out.println("\n=== Проверка типов ===");
        System.out.println("manager является Employee: " + (manager instanceof Employee));
        System.out.println("manager является Manager: " + (manager instanceof Manager));
        System.out.println("manager является Developer: " + (manager instanceof Developer));
    }
}
```

### 3.2. Пример 2. Композиция: «Компьютер и его компоненты»

```java
/**
 * Класс, моделирующий процессор
 */
public class CPU {
    private String model;
    private int cores;
    private double frequency;

    public CPU(String model, int cores, double frequency) {
        setModel(model);
        setCores(cores);
        setFrequency(frequency);
    }

    public String getModel() { return model; }
    public int getCores() { return cores; }
    public double getFrequency() { return frequency; }

    public void setModel(String model) {
        if (model == null || model.trim().isEmpty()) {
            throw new IllegalArgumentException("Модель процессора не может быть пустой");
        }
        this.model = model;
    }

    public void setCores(int cores) {
        if (cores <= 0) {
            throw new IllegalArgumentException("Количество ядер должно быть положительным");
        }
        this.cores = cores;
    }

    public void setFrequency(double frequency) {
        if (frequency <= 0) {
            throw new IllegalArgumentException("Частота должна быть положительной");
        }
        this.frequency = frequency;
    }

    @Override
    public String toString() {
        return String.format("%s (%d ядер, %.2f ГГц)", model, cores, frequency);
    }
}

/**
 * Класс, моделирующий оперативную память
 */
public class RAM {
    private int capacityGB;
    private String type;

    public RAM(int capacityGB, String type) {
        setCapacityGB(capacityGB);
        setType(type);
    }

    public int getCapacityGB() { return capacityGB; }
    public String getType() { return type; }

    public void setCapacityGB(int capacityGB) {
        if (capacityGB <= 0) {
            throw new IllegalArgumentException("Объём памяти должен быть положительным");
        }
        this.capacityGB = capacityGB;
    }

    public void setType(String type) {
        if (type == null || type.trim().isEmpty()) {
            throw new IllegalArgumentException("Тип памяти не может быть пустым");
        }
        this.type = type;
    }

    @Override
    public String toString() {
        return capacityGB + " ГБ " + type;
    }
}

/**
 * Класс, моделирующий накопитель
 */
public class Storage {
    private int capacityGB;
    private String type;

    public Storage(int capacityGB, String type) {
        setCapacityGB(capacityGB);
        setType(type);
    }

    public int getCapacityGB() { return capacityGB; }
    public String getType() { return type; }

    public void setCapacityGB(int capacityGB) {
        if (capacityGB <= 0) {
            throw new IllegalArgumentException("Объём накопителя должен быть положительным");
        }
        this.capacityGB = capacityGB;
    }

    public void setType(String type) {
        if (type == null || type.trim().isEmpty()) {
            throw new IllegalArgumentException("Тип накопителя не может быть пустым");
        }
        this.type = type;
    }

    @Override
    public String toString() {
        return capacityGB + " ГБ " + type;
    }
}

/**
 * Класс, моделирующий компьютер.
 * Компьютер ИМЕЕТ процессор, оперативную память и накопитель (композиция).
 */
public class Computer {
    private String brand;
    private String model;
    private CPU cpu;
    private RAM ram;
    private Storage storage;

    public Computer(String brand, String model,
                    CPU cpu, RAM ram, Storage storage) {
        setBrand(brand);
        setModel(model);
        this.cpu = cpu;
        this.ram = ram;
        this.storage = storage;
    }

    public String getBrand() { return brand; }
    public String getModel() { return model; }
    public CPU getCpu() { return cpu; }
    public RAM getRam() { return ram; }
    public Storage getStorage() { return storage; }

    public void setBrand(String brand) {
        if (brand == null || brand.trim().isEmpty()) {
            throw new IllegalArgumentException("Марка не может быть пустой");
        }
        this.brand = brand;
    }

    public void setModel(String model) {
        if (model == null || model.trim().isEmpty()) {
            throw new IllegalArgumentException("Модель не может быть пустой");
        }
        this.model = model;
    }

    /**
     * Метод, делегирующий вызовы компонентам (композиция)
     */
    public void displayConfiguration() {
        System.out.println("=== Конфигурация компьютера ===");
        System.out.println("Марка: " + brand);
        System.out.println("Модель: " + model);
        System.out.println("Процессор: " + cpu);
        System.out.println("Оперативная память: " + ram);
        System.out.println("Накопитель: " + storage);
    }
}
```

Демонстрационный класс:

```java
public class ComputerDemo {
    public static void main(String[] args) {
        // Создание компонентов
        CPU cpu = new CPU("Intel Core i7-12700", 12, 3.6);
        RAM ram = new RAM(32, "DDR5");
        Storage storage = new Storage(1000, "NVMe SSD");

        // Создание компьютера с использованием композиции
        Computer computer = new Computer("Dell", "XPS 15", cpu, ram, storage);

        computer.displayConfiguration();
    }
}
```

### 3.3. Пример 3. Комбинирование наследования и композиции

```java
/**
 * Базовый класс для всех транспортных средств
 */
public class Vehicle {
    protected String brand;
    protected String model;
    protected int year;

    public Vehicle(String brand, String model, int year) {
        setBrand(brand);
        setModel(model);
        setYear(year);
    }

    public String getBrand() { return brand; }
    public String getModel() { return model; }
    public int getYear() { return year; }

    public void setBrand(String brand) {
        if (brand == null || brand.trim().isEmpty()) {
            throw new IllegalArgumentException("Марка не может быть пустой");
        }
        this.brand = brand;
    }

    public void setModel(String model) {
        if (model == null || model.trim().isEmpty()) {
            throw new IllegalArgumentException("Модель не может быть пустой");
        }
        this.model = model;
    }

    public void setYear(int year) {
        int currentYear = java.time.Year.now().getValue();
        if (year < 1900 || year > currentYear + 1) {
            throw new IllegalArgumentException(
                "Год выпуска должен быть в диапазоне от 1900 до " + (currentYear + 1));
        }
        this.year = year;
    }

    public int getAge() {
        return java.time.Year.now().getValue() - year;
    }

    @Override
    public String toString() {
        return brand + " " + model + " (" + year + ")";
    }
}

/**
 * Класс, моделирующий двигатель (используется в композиции)
 */
public class Engine {
    private int horsepower;
    private double volume;
    private String fuelType;

    public Engine(int horsepower, double volume, String fuelType) {
        setHorsepower(horsepower);
        setVolume(volume);
        setFuelType(fuelType);
    }

    public int getHorsepower() { return horsepower; }
    public double getVolume() { return volume; }
    public String getFuelType() { return fuelType; }

    public void setHorsepower(int horsepower) {
        if (horsepower <= 0) {
            throw new IllegalArgumentException("Мощность должна быть положительной");
        }
        this.horsepower = horsepower;
    }

    public void setVolume(double volume) {
        if (volume <= 0) {
            throw new IllegalArgumentException("Объём должен быть положительным");
        }
        this.volume = volume;
    }

    public void setFuelType(String fuelType) {
        if (fuelType == null || fuelType.trim().isEmpty()) {
            throw new IllegalArgumentException("Тип топлива не может быть пустым");
        }
        this.fuelType = fuelType;
    }

    @Override
    public String toString() {
        return String.format("%.1f л, %d л.с., %s", volume, horsepower, fuelType);
    }
}

/**
 * Подкласс «Автомобиль» — наследует Vehicle, использует композицию с Engine
 */
public class Car extends Vehicle {
    private int numberOfDoors;
    private Engine engine;   // композиция

    public Car(String brand, String model, int year,
               int numberOfDoors, int horsepower, double volume, String fuelType) {
        super(brand, model, year);
        setNumberOfDoors(numberOfDoors);
        this.engine = new Engine(horsepower, volume, fuelType);
    }

    public int getNumberOfDoors() { return numberOfDoors; }
    public Engine getEngine() { return engine; }

    public void setNumberOfDoors(int numberOfDoors) {
        if (numberOfDoors < 2 || numberOfDoors > 5) {
            throw new IllegalArgumentException(
                "Количество дверей должно быть от 2 до 5");
        }
        this.numberOfDoors = numberOfDoors;
    }

    @Override
    public String toString() {
        return super.toString() + ", дверей: " + numberOfDoors +
               ", двигатель: " + engine;
    }
}

/**
 * Подкласс «Мотоцикл» — наследует Vehicle, использует композицию с Engine
 */
public class Motorcycle extends Vehicle {
    private String type;
    private Engine engine;   // композиция

    public Motorcycle(String brand, String model, int year,
                      String type, int horsepower, double volume, String fuelType) {
        super(brand, model, year);
        setType(type);
        this.engine = new Engine(horsepower, volume, fuelType);
    }

    public String getType() { return type; }
    public Engine getEngine() { return engine; }

    public void setType(String type) {
        if (type == null || type.trim().isEmpty()) {
            throw new IllegalArgumentException("Тип мотоцикла не может быть пустым");
        }
        this.type = type;
    }

    @Override
    public String toString() {
        return super.toString() + ", тип: " + type + ", двигатель: " + engine;
    }
}
```

Демонстрационный класс:

```java
public class VehicleDemo {
    public static void main(String[] args) {
        Car car = new Car("Toyota", "Camry", 2022, 4, 200, 2.5, "Бензин");
        Motorcycle motorcycle = new Motorcycle("Yamaha", "MT-09", 2023,
                                               "Спорт", 117, 0.847, "Бензин");

        System.out.println("=== Транспортные средства ===");
        System.out.println(car);
        System.out.println(motorcycle);

        System.out.println("\n=== Возраст ===");
        System.out.println("Автомобиль: " + car.getAge() + " лет");
        System.out.println("Мотоцикл: " + motorcycle.getAge() + " лет");

        // Демонстрация полиморфизма
        System.out.println("\n=== Полиморфизм ===");
        Vehicle[] vehicles = {car, motorcycle};
        for (Vehicle v : vehicles) {
            System.out.println(v);
        }
    }
}
```

## 4. Задания на паре

### Задание 4.1. Иерархия «Геометрические фигуры»

Разработайте иерархию классов для геометрических фигур.

**Базовый класс `Shape`:**
- Поля: `name` (String), `color` (String).
- Методы:
  - `getArea()` — возвращает площадь фигуры (в базовом классе возвращает 0.0);
  - `getPerimeter()` — возвращает периметр фигуры (в базовом классе возвращает 0.0);
  - `displayInfo()` — выводит информацию о фигуре (название, цвет, площадь, периметр).

**Подкласс `Circle` (наследует `Shape`):**
- Дополнительное поле: `radius` (double).
- Переопределите методы `getArea()` (π·r²) и `getPerimeter()` (2·π·r).

**Подкласс `Rectangle` (наследует `Shape`):**
- Дополнительные поля: `width` (double), `height` (double).
- Переопределите методы `getArea()` (w·h) и `getPerimeter()` (2·(w+h)).

**Подкласс `Triangle` (наследует `Shape`):**
- Дополнительные поля: `sideA`, `sideB`, `sideC` (double).
- Переопределите методы `getArea()` (формула Герона) и `getPerimeter()` (сумма сторон).
- Реализуйте проверку существования треугольника (сумма любых двух сторон должна превышать третью).

**Демонстрация:**
- Создайте массив `Shape[]` с объектами различных типов.
- В цикле выведите информацию о каждой фигуре.
- Вычислите суммарную площадь всех фигур.
- Определите фигуру с наибольшим периметром.

**Пример выполнения программы:**

```
=== Геометрические фигуры ===
Круг [красный]: площадь = 78.54, периметр = 31.42
Прямоугольник [синий]: площадь = 50.00, периметр = 30.00
Треугольник [зелёный]: площадь = 6.00, периметр = 12.00

Суммарная площадь: 134.54
Фигура с наибольшим периметром: Прямоугольник (30.00)
```

---

### Задание 4.2. Композиция: «Смартфон и его компоненты»

Разработайте класс `Smartphone`, моделирующий смартфон, используя принцип композиции.

**Компоненты (отдельные классы):**

1. **`Processor`** (процессор):
   - Поля: `model` (String), `cores` (int), `frequency` (double).
   - Метод: `getPerformanceScore()` — возвращает произведение количества ядер на частоту.

2. **`Camera`** (камера):
   - Поля: `resolution` (int, в мегапикселях), `hasFlash` (boolean), `opticalStabilization` (boolean).
   - Метод: `getQualityScore()` — возвращает оценку качества (resolution × 10, +50 за вспышку, +100 за стабилизацию).

3. **`Battery`** (батарея):
   - Поля: `capacity` (int, мА·ч), `chargingSpeed` (int, Вт).
   - Метод: `getEstimatedHours()` — возвращает примерное время работы (capacity / 300).

**Класс `Smartphone`:**
- Поля: `brand` (String), `model` (String), `price` (double), `processor` (Processor), `camera` (Camera), `battery` (Battery).
- Методы:
  - `getOverallScore()` — возвращает общий рейтинг (сумма оценок компонентов);
  - `displayConfiguration()` — выводит полную конфигурацию смартфона.

**Демонстрация:**
- Создайте не менее трёх смартфонов различных конфигураций.
- Выведите конфигурацию каждого.
- Определите смартфон с наилучшим общим рейтингом.
- Определите самый дешёвый смартфон с рейтингом выше заданного порога.

---

### Задание 4.3. Комбинирование наследования и композиции: «Библиотека»

Разработайте систему классов, моделирующих библиотеку.

**Базовый класс `LibraryItem` (элемент библиотеки):**
- Поля: `id` (String), `title` (String), `year` (int), `isBorrowed` (boolean).
- Методы:
  - `borrow()` — пометить как выданный (если уже выдан — выбросить `IllegalStateException`);
  - `returnItem()` — пометить как возвращённый (если не выдан — выбросить `IllegalStateException`);
  - `getInfo()` — строковое представление элемента.

**Подкласс `Book` (наследует `LibraryItem`):**
- Дополнительные поля: `author` (String), `isbn` (String), `pages` (int).
- Переопределите метод `getInfo()`.

**Подкласс `Magazine` (наследует `LibraryItem`):**
- Дополнительные поля: `issueNumber` (int), `publisher` (String).
- Переопределите метод `getInfo()`.

**Подкласс `DVD` (наследует `LibraryItem`):**
- Дополнительные поля: `director` (String), `durationMinutes` (int).
- Переопределите метод `getInfo()`.

**Класс `Library` (использует композицию):**
- Поля: `name` (String), `items` (массив `LibraryItem[]`), `capacity` (int).
- Методы:
  - `addItem(LibraryItem item)` — добавить элемент в библиотеку;
  - `findItemById(String id)` — найти элемент по идентификатору;
  - `borrowItem(String id)` — выдать элемент по идентификатору;
  - `returnItem(String id)` — принять элемент по идентификатору;
  - `displayAllItems()` — вывести список всех элементов;
  - `countAvailableItems()` — подсчитать количество доступных элементов.

**Демонстрация:**
- Создайте библиотеку и добавьте в неё не менее 6 элементов различных типов.
- Выведите список всех элементов.
- Выдайте несколько элементов.
- Снова выведите список, отобразив статус элементов.
- Подсчитайте количество доступных элементов.

## 5. Задание для самостоятельной работы

Разработать иерархию классов согласно своему варианту. Требования:

1. Базовый класс с общими полями (не менее 3) и методами (не менее 2, включая переопределяемые).
2. Минимум 3 подкласса, наследующих от базового, каждый с не менее чем 2 дополнительными полями.
3. Переопределение методов в каждом подклассе с применением `@Override` и `super`.
4. Применение инкапсуляции: все поля — `private` или `protected`, валидация в сеттерах.
5. Демонстрация полиморфизма: массив объектов базового класса с вызовом переопределённых методов.

### Варианты заданий

**Вариант 1.** Иерархия «Транспортные средства»: базовый класс `Vehicle` (марка, модель, год выпуска, скорость). Подклассы: `Car` (количество дверей, тип кузова), `Motorcycle` (тип, объём двигателя), `Truck` (грузоподъёмность, количество осей). Методы: `start()`, `stop()`, `getInfo()`.

**Вариант 2.** Иерархия «Домашние животные»: базовый класс `Pet` (кличка, возраст, вес). Подклассы: `Dog` (порода, обученность командам), `Cat` (длина шерсти, характер), `Parrot` (вид, словарный запас). Методы: `feed()`, `makeSound()`, `getInfo()`.

**Вариант 3.** Иерархия «Банковские счета»: базовый класс `BankAccount` (номер счёта, владелец, баланс). Подклассы: `SavingsAccount` (процентная ставка, срок вклада), `CheckingAccount` (лимит овердрафта), `CreditAccount` (кредитный лимит, процентная ставка). Методы: `deposit()`, `withdraw()`, `calculateInterest()`, `getInfo()`.

**Вариант 4.** Иерархия «Учебные заведения»: базовый класс `EducationalInstitution` (название, адрес, количество учащихся). Подклассы: `School` (количество классов, директор), `University` (количество факультетов, ректор), `College` (количество специальностей). Методы: `enroll()`, `expel()`, `getInfo()`.

**Вариант 5.** Иерархия «Музыкальные инструменты»: базовый класс `MusicalInstrument` (название, производитель, год выпуска). Подклассы: `Guitar` (количество струн, тип), `Piano` (количество клавиш, тип), `Violin` (размер, материал). Методы: `play()`, `tune()`, `getInfo()`.

**Вариант 6.** Иерархия «Мебель»: базовый класс `Furniture` (название, материал, цвет). Подклассы: `Chair` (количество ножек, наличие подлокотников), `Table` (форма, количество мест), `Bed` (размер, наличие матраса). Методы: `assemble()`, `disassemble()`, `getInfo()`.

**Вариант 7.** Иерархия «Электронные устройства»: базовый класс `ElectronicDevice` (марка, модель, цена). Подклассы: `Phone` (диагональ экрана, ёмкость батареи), `Laptop` (диагональ экрана, вес), `Tablet` (диагональ экрана, наличие стилуса). Методы: `turnOn()`, `turnOff()`, `getInfo()`.

**Вариант 8.** Иерархия «Посуда»: базовый класс `Dishware` (название, материал, объём). Подклассы: `Cup` (наличие ручки, тип), `Plate` (форма, глубина), `Pot` (наличие крышки, тип покрытия). Методы: `use()`, `clean()`, `getInfo()`.

**Вариант 9.** Иерархия «Одежда»: базовый класс `Clothing` (название, размер, цвет). Подклассы: `Shirt` (тип воротника, длина рукава), `Pants` (тип застёжки, длина), `Dress` (длина, стиль). Методы: `wear()`, `wash()`, `getInfo()`.

**Вариант 10.** Иерархия «Спортивные снаряды»: базовый класс `SportsEquipment` (название, вес, материал). Подклассы: `Dumbbell` (тип, регулируемый или нет), `Ball` (диаметр, вид спорта), `Racket` (размер головки, вес). Методы: `use()`, `maintain()`, `getInfo()`.

**Вариант 11.** Иерархия «Игровые платформы»: базовый класс `GamePlatform` (название, производитель, год выпуска). Подклассы: `Console` (тип, объём памяти), `PC` (операционная система, объём памяти), `Mobile` (операционная система, диагональ). Методы: `launchGame()`, `shutdown()`, `getInfo()`.

**Вариант 12.** Иерархия «Печатные издания»: базовый класс `Publication` (название, издательство, год издания). Подклассы: `Book` (автор, количество страниц), `Magazine` (номер выпуска, периодичность), `Newspaper` (дата выпуска, тираж). Методы: `read()`, `archive()`, `getInfo()`.

**Вариант 13.** Иерархия «Медицинские учреждения»: базовый класс `MedicalFacility` (название, адрес, количество коек). Подклассы: `Hospital` (количество отделений, главный врач), `Clinic` (количество специалистов, профиль), `Pharmacy` (количество препаратов, режим работы). Методы: `admitPatient()`, `dischargePatient()`, `getInfo()`.

**Вариант 14.** Иерархия «Средства массовой информации»: базовый класс `Media` (название, владелец, год основания). Подклассы: `TVChannel` (охват аудитории, тематика), `RadioStation` (частота вещания, тематика), `OnlinePortal` (посещаемость, тематика). Методы: `broadcast()`, `publish()`, `getInfo()`.

**Вариант 15.** Иерархия «Типы транспорта в городе»: базовый класс `UrbanTransport` (номер маршрута, вместимость). Подклассы: `Bus` (тип двигателя, наличие кондиционера), `Tramway` (количество секций, наличие низкого пола), `Trolleybus` (тип контактной сети). Методы: `startRoute()`, `finishRoute()`, `getInfo()`.

**Вариант 16.** Иерархия «Типы отелей»: базовый класс `Hotel` (название, количество звёзд, количество номеров). Подклассы: `BusinessHotel` (наличие конференц-зала, бизнес-центра), `ResortHotel` (наличие бассейна, пляжа), `Hostel` (количество мест в номере, наличие общей кухни). Методы: `checkIn()`, `checkOut()`, `getInfo()`.

**Вариант 17.** Иерархия «Типы магазинов»: базовый класс `Store` (название, адрес, площадь). Подклассы: `Supermarket` (количество касс, количество отделов), `Boutique` (бренд, ценовая категория), `OnlineStore` (сайт, количество товаров). Методы: `openStore()`, `closeStore()`, `getInfo()`.

**Вариант 18.** Иерархия «Типы учебных курсов»: базовый класс `Course` (название, преподаватель, длительность). Подклассы: `LectureCourse` (количество лекций, форма контроля), `PracticalCourse` (количество лабораторных, форма контроля), `OnlineCourse` (платформа, наличие сертификата). Методы: `enroll()`, `complete()`, `getInfo()`.

**Вариант 19.** Иерархия «Типы документов»: базовый класс `Document` (номер, дата выдачи, срок действия). Подклассы: `Passport` (серия, место выдачи), `License` (категория, место выдачи), `Certificate` (тип, организация). Методы: `validate()`, `renew()`, `getInfo()`.

**Вариант 20.** Иерархия «Типы договоров»: базовый класс `Contract` (номер, дата заключения, срок действия). Подклассы: `EmploymentContract` (должность, оклад), `LeaseContract` (объект аренды, ежемесячная плата), `ServiceContract` (вид услуги, стоимость). Методы: `sign()`, `terminate()`, `getInfo()`.

**Вариант 21.** Иерархия «Типы счетов в ресторане»: базовый класс `BillItem` (название, цена, количество). Подклассы: `FoodItem` (категория блюда, вес порции), `DrinkItem` (объём, тип напитка), `ServiceItem` (тип услуги, процент). Методы: `calculateTotal()`, `applyDiscount()`, `getInfo()`.

**Вариант 22.** Иерархия «Типы страховых полисов»: базовый класс `InsurancePolicy` (номер, страхователь, страховая сумма). Подклассы: `AutoInsurance` (автомобиль, срок), `HealthInsurance` (застрахованный, срок), `PropertyInsurance` (объект, срок). Методы: `activate()`, `deactivate()`, `getInfo()`.

**Вариант 23.** Иерархия «Типы билетов»: базовый класс `Ticket` (номер, дата, цена). Подклассы: `CinemaTicket` (фильм, место), `TheaterTicket` (спектакль, место), `TransportTicket` (маршрут, тип). Методы: `validate()`, `cancel()`, `getInfo()`.

**Вариант 24.** Иерархия «Типы аккаунтов в системе»: базовый класс `UserAccount` (логин, дата создания). Подклассы: `AdminAccount` (уровень доступа), `ModeratorAccount` (зона модерации), `RegularAccount` (статус подписки). Методы: `login()`, `logout()`, `getInfo()`.

**Вариант 25.** Иерархия «Типы уведомлений»: базовый класс `Notification` (идентификатор, дата создания, текст). Подклассы: `EmailNotification` (адрес получателя, тема), `SmsNotification` (номер телефона), `PushNotification` (заголовок, приложение). Методы: `send()`, `markAsRead()`, `getInfo()`.

**Вариант 26.** Иерархия «Типы платежей»: базовый класс `Payment` (номер, сумма, дата). Подклассы: `CardPayment` (номер карты, тип), `CashPayment` (сумма сдачи), `ElectronicPayment` (электронный кошелёк). Методы: `process()`, `refund()`, `getInfo()`.

**Вариант 27.** Иерархия «Типы файлов»: базовый класс `File` (имя, размер, дата создания). Подклассы: `TextFile` (кодировка, количество строк), `ImageFile` (разрешение, формат), `AudioFile` (длительность, битрейт). Методы: `open()`, `close()`, `getInfo()`.

**Вариант 28.** Иерархия «Типы событий в календаре»: базовый класс `CalendarEvent` (название, дата начала, длительность). Подклассы: `Meeting` (место проведения, участники), `Birthday` (именинник, периодичность), `Holiday` (страна, тип). Методы: `schedule()`, `cancel()`, `getInfo()`.

**Вариант 29.** Иерархия «Типы заказов в доставке»: базовый класс `Order` (номер, дата, сумма). Подклассы: `FoodOrder` (ресторан, адрес доставки), `GroceryOrder` (супермаркет, временной слот), `PharmacyOrder` (аптека, наличие рецепта). Методы: `confirm()`, `cancel()`, `getInfo()`.

**Вариант 30.** Иерархия «Типы комментариев в системе»: базовый класс `Comment` (автор, дата, текст). Подклассы: `ReviewComment` (оценка, товар), `ForumComment` (тема, количество лайков), `SocialComment` (платформа, количество репостов). Методы: `publish()`, `delete()`, `getInfo()`.

## 6. Методические указания к самостоятельной работе

1. **Анализ предметной области.** Перед проектированием иерархии определите:
   - общие характеристики всех сущностей (будут полями базового класса);
   - специфические характеристики каждого подвида (будут полями подклассов);
   - поведение, общее для всех сущностей (методы базового класса);
   - поведение, специфичное для подвидов (переопределённые методы).

2. **Применение `super`.** В конструкторах подклассов обязательно вызывайте конструктор базового класса посредством `super(...)`. В переопределённых методах при необходимости вызывайте реализацию базового класса посредством `super.methodName(...)`.

3. **Аннотация `@Override`.** Каждый переопределённый метод должен быть помечен аннотацией `@Override` для обеспечения проверки корректности на этапе компиляции.

4. **Инкапсуляция в иерархии.** Поля базового класса, к которым необходим доступ из подклассов, объявляйте с модификатором `protected`. Остальные поля — с модификатором `private`. Валидация должна выполняться во всех сеттерах, включая сеттеры базового класса.

5. **Полиморфизм.** В демонстрационном классе создайте массив объектов базового класса, заполните его объектами различных подклассов и в цикле вызывайте переопределённые методы. Это продемонстрирует механизм динамического связывания.

6. **Тестирование.** Проверьте:
   - корректность вызова конструкторов в иерархии;
   - работу переопределённых методов;
   - корректность валидации во всех классах иерархии;
   - работу полиморфных вызовов.

7. **Оформление отчёта.** Отчёт должен содержать листинги всех файлов проекта, протокол работы программы, ответы на контрольные вопросы и выводы о применении наследования в конкретном варианте.

## 7. Контрольные вопросы

1. Что такое наследование? Какое отношение оно моделирует?
2. Каков синтаксис наследования в языке Java?
3. Что такое переопределение методов? Каковы его правила?
4. Для чего применяется аннотация `@Override`?
5. Для каких целей используется ключевое слово `super`?
6. Каков порядок вызова конструкторов в иерархии наследования?
7. Что такое композиция? Какое отношение она моделирует?
8. В чём различие между наследованием и композицией?
9. В каких случаях предпочтительнее применять композицию?
10. Что такое модификатор `final`? Какие ограничения он накладывает?
11. Что такое модификатор `protected`? В каких случаях он применяется?
12. Какой класс является базовым для всех классов в языке Java?

