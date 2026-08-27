># Лабораторная работа №3. Наследование и полиморфизм

## 1. Паспорт работы

| Параметр | Значение |
|----------|----------|
| Номер занятия | 3 из 17 |
| Блок | 1. Основы ООП |
| Продолжительность | 2 академических часа |
| Форма выполнения | Индивидуальная |
| ИИ-инструмент | YandexGPT / GigaChat / JetBrains AI Assistant |

### 1.1. Цель работы

Освоить механизм наследования, переопределение методов, полиморфизм подтипов, научиться применять абстрактные классы и интерфейсы для построения гибких иерархий классов.

### 1.2. Задачи работы

1. Изучить механизм наследования и ключевое слово `extends`.
2. Освоить переопределение методов и аннотацию `@Override`.
3. Научиться применять ключевое слово `super` для обращения к членам базового класса.
4. Изучить полиморфизм подтипов и позднее связывание.
5. Освоить абстрактные классы и методы.
6. Изучить интерфейсы и множественную реализацию.
7. Развить навыки выбора между наследованием и интерфейсами.

### 1.3. Оснащение

- JDK 17 или выше;
- IntelliJ IDEA Community Edition;
- Git;
- доступ к YandexGPT или GigaChat.

## 2. Теоретический конспект

### 2.1. Наследование

**Наследование** — механизм создания нового класса на основе существующего. Новый класс (подкласс) наследует поля и методы базового класса (суперкласса) и может добавлять свои или переопределять унаследованные.

**Отношение «является» (is-a):** подкласс является специализацией базового класса.

```java
public class Employee {
    protected String name;
    protected String department;
    protected double salary;

    public Employee(String name, String department, double salary) {
        this.name = name;
        this.department = department;
        this.salary = salary;
    }

    public double calculateBonus() {
        return salary * 0.05;  // 5% от зарплаты
    }

    @Override
    public String toString() {
        return name + " (" + department + ")";
    }
}

public class Manager extends Employee {
    private int subordinatesCount;

    public Manager(String name, String department, double salary, int subordinatesCount) {
        super(name, department, salary);  // вызов конструктора базового класса
        this.subordinatesCount = subordinatesCount;
    }

    @Override
    public double calculateBonus() {
        return salary * 0.15;  // 15% для менеджеров
    }

    @Override
    public String toString() {
        return super.toString() + " — менеджер, подчинённых: " + subordinatesCount;
    }
}
```

### 2.2. Переопределение методов

**Переопределение (overriding)** — создание в подклассе метода с той же сигнатурой, что и в базовом классе.

**Правила:**
- сигнатура должна совпадать (имя, параметры, возвращаемый тип);
- модификатор доступа не может быть строже;
- рекомендуется аннотация `@Override` для проверки компилятором.

**Различие между перегрузкой и переопределением:**
- **Перегрузка (overloading)** — несколько методов с одинаковым именем, но разными параметрами в одном классе;
- **Переопределение (overriding)** — метод с той же сигнатурой в подклассе.

### 2.3. Ключевое слово `super`

`super` используется для:
- вызова конструктора базового класса: `super(...)`;
- вызова переопределённого метода базового класса: `super.methodName()`.

```java
@Override
public String toString() {
    return super.toString() + " — менеджер";  // вызов toString() базового класса
}
```

### 2.4. Полиморфизм

**Полиморфизм** — способность объектов с различной внутренней структурой иметь общий интерфейс.

**Позднее связывание (dynamic dispatch):** конкретная версия метода определяется во время выполнения на основе фактического типа объекта.

```java
Employee[] employees = {
    new Manager("Иванов", "IT", 150000, 5),
    new Developer("Петров", "IT", 120000, "Java")
};

for (Employee e : employees) {
    System.out.println(e.calculateBonus());  // вызов конкретной реализации
}
```

### 2.5. Абстрактные классы

**Абстрактный класс** — класс, который нельзя инстанцировать. Может содержать абстрактные методы (без реализации), которые должны быть реализованы в подклассах.

```java
public abstract class Shape {
    protected String color;

    public Shape(String color) {
        this.color = color;
    }

    // Абстрактный метод — реализация в подклассах
    public abstract double getArea();

    // Конкретный метод
    public String getColor() {
        return color;
    }
}

public class Circle extends Shape {
    private double radius;

    public Circle(String color, double radius) {
        super(color);
        this.radius = radius;
    }

    @Override
    public double getArea() {
        return Math.PI * radius * radius;
    }
}
```

### 2.6. Интерфейсы

**Интерфейс** — контракт поведения. Класс может реализовывать несколько интерфейсов.

**Отношение «способен» (can-do):** класс способен выполнять определённые действия.

```java
public interface Reportable {
    String generateReport();
}

public interface Printable {
    void print();
}

public class Manager extends Employee implements Reportable, Printable {
    @Override
    public String generateReport() {
        return "Отчёт менеджера " + name;
    }

    @Override
    public void print() {
        System.out.println(toString());
    }
}
```

### 2.7. Сравнение абстрактного класса и интерфейса

| Критерий | Абстрактный класс | Интерфейс |
|----------|-------------------|-----------|
| Наследование | Одиночное (`extends`) | Множественное (`implements`) |
| Поля | Любые | Только константы (`public static final`) |
| Конструкторы | Есть | Нет |
| Методы | Абстрактные и конкретные | Абстрактные и `default` |
| Семантика | «является» (is-a) | «способен» (can-do) |
| Применение | Общая основа для родственных классов | Контракт для неродственных классов |

**Правило выбора:** если классы находятся в одной иерархии «является» и имеют общее состояние — применяется абстрактный класс. Если классы не связаны иерархией, но должны обладать общим поведением — применяется интерфейс.

### 2.8. Модификатор `final`

- `final` класс — нельзя наследовать;
- `final` метод — нельзя переопределить;
- `final` поле — константа.

## 3. Задание на паре

### Задача. Иерархия «Сотрудники»

1. **Создать базовый класс `Employee`:**
   - Поля: `name` (String), `department` (String), `salary` (double).
   - Метод `calculateBonus()` — возвращает 5% от `salary`.
   - Переопределить `toString()`.

2. **Создать подкласс `Manager`:**
   - Дополнительное поле: `subordinatesCount` (int).
   - Переопределить `calculateBonus()` — бонус 15%.
   - Переопределить `toString()`.

3. **Создать подкласс `Developer`:**
   - Дополнительные поля: `programmingLanguage` (String), `techBonus` (double).
   - Переопределить `calculateBonus()` — бонус 10% + `techBonus`.
   - Переопределить `toString()`.

4. **В `main()` создать массив `Employee[]` из 5 объектов разных типов.** В цикле вызвать `calculateBonus()` и вывести результат — убедиться, что работает полиморфизм.

5. **Добавить интерфейс `Reportable`** с методом `generateReport()`. Реализовать его в `Manager` (отчёт: список подчинённых — заглушка) и `Developer` (отчёт: список завершённых задач — заглушка).

6. **Продемонстрировать работу с интерфейсным типом:** `List<Reportable> reporters`.

**Пример выполнения:**

```
=== Полиморфизм ===
Иванов (IT) — бонус: 22500.00 руб.
Петров (IT) — бонус: 15000.00 руб.
...

=== Интерфейс Reportable ===
Отчёт менеджера Иванов: 5 подчинённых
Отчёт разработчика Петров: 10 завершённых задач
```

### Применение ИИ-инструмента

**Промпт для YandexGPT:**

```
Создай иерархию классов на Java 17:
- Абстрактный базовый класс Employee с полями name, department, salary.
- Метод calculateBonus() возвращает 5% от salary.
- Подкласс Manager: бонус 15%, поле subordinatesCount.
- Подкласс Developer: бонус 10% + techBonus, поле programmingLanguage.
- Переопредели toString() в каждом классе.
- Интерфейс Reportable с методом generateReport().
- Реализация Reportable в Manager и Developer.
```

**Анализ результата:**
- Проверить корректность вызова `super()` в конструкторах.
- Проверить аннотацию `@Override`.
- Проверить реализацию полиморфизма в цикле.

## 4. Индивидуальные задания (30 вариантов)

Каждый вариант содержит:
- базовый класс с 3 полями и 1 методом;
- 2 подкласса с переопределением метода;
- интерфейс с 1 методом;
- реализацию интерфейса в подклассах.

---

### Вариант 1. Иерархия «Сотрудники»
**Базовый класс `Employee`:** `name`, `department`, `salary`. Метод `calculateBonus()` — 5%.
**Подкласс `Manager`:** бонус 15%. Метод `generateReport()` — список подчинённых.
**Подкласс `Developer`:** бонус 10% + `techBonus`. Метод `generateReport()` — список задач.

### Вариант 2. Иерархия «Транспортные средства»
**Базовый класс `Vehicle`:** `brand`, `model`, `speed`. Метод `getFuelConsumption()` — базовый расход.
**Подкласс `Car`:** расход × 1.2. Метод `getInfo()` — тип кузова.
**Подкласс `Motorcycle`:** расход × 0.7. Метод `getInfo()` — тип двигателя.

### Вариант 3. Иерархия «Фигуры»
**Базовый класс `Shape`:** `name`, `color`. Метод `getArea()`.
**Подкласс `Circle`:** `radius`. Переопределение `getArea()`.
**Подкласс `Rectangle`:** `width`, `height`. Переопределение `getArea()`.

### Вариант 4. Иерархия «Животные»
**Базовый класс `Animal`:** `name`, `age`, `weight`. Метод `makeSound()`.
**Подкласс `Dog`:** `breed`. Переопределение `makeSound()` — «Гав!».
**Подкласс `Cat`:** `furLength`. Переопределение `makeSound()` — «Мяу!».

### Вариант 5. Иерархия «Учебные заведения»
**Базовый класс `EducationalInstitution`:** `name`, `address`, `studentsCount`. Метод `getRating()`.
**Подкласс `School`:** `teachersCount`. Переопределение `getRating()`.
**Подкласс `University`:** `facultiesCount`. Переопределение `getRating()`.

### Вариант 6. Иерархия «Банковские счета»
**Базовый класс `BankAccount`:** `accountNumber`, `owner`, `balance`. Метод `calculateInterest()`.
**Подкласс `SavingsAccount`:** `interestRate`. Переопределение `calculateInterest()`.
**Подкласс `CreditAccount`:** `creditLimit`. Переопределение `calculateInterest()`.

### Вариант 7. Иерархия «Музыкальные инструменты»
**Базовый класс `MusicalInstrument`:** `name`, `manufacturer`, `price`. Метод `play()`.
**Подкласс `Guitar`:** `stringsCount`. Переопределение `play()`.
**Подкласс `Piano`:** `keysCount`. Переопределение `play()`.

### Вариант 8. Иерархия «Мебель»
**Базовый класс `Furniture`:** `name`, `material`, `price`. Метод `assemble()`.
**Подкласс `Chair`:** `legsCount`. Переопределение `assemble()`.
**Подкласс `Table`:** `surfaceArea`. Переопределение `assemble()`.

### Вариант 9. Иерархия «Электронные устройства»
**Базовый класс `ElectronicDevice`:** `brand`, `model`, `price`. Метод `turnOn()`.
**Подкласс `Phone`:** `screenSize`. Переопределение `turnOn()`.
**Подкласс `Laptop`:** `ramSize`. Переопределение `turnOn()`.

### Вариант 10. Иерархия «Посуда»
**Базовый класс `Dishware`:** `name`, `material`, `volume`. Метод `use()`.
**Подкласс `Cup`:** `hasHandle`. Переопределение `use()`.
**Подкласс `Plate`:** `diameter`. Переопределение `use()`.

### Вариант 11. Иерархия «Одежда»
**Базовый класс `Clothing`:** `name`, `size`, `price`. Метод `wear()`.
**Подкласс `Shirt`:** `collarType`. Переопределение `wear()`.
**Подкласс `Pants`:** `length`. Переопределение `wear()`.

### Вариант 12. Иерархия «Спортивные снаряды»
**Базовый класс `SportsEquipment`:** `name`, `weight`, `price`. Метод `use()`.
**Подкласс `Dumbbell`:** `adjustable`. Переопределение `use()`.
**Подкласс `Ball`:** `diameter`. Переопределение `use()`.

### Вариант 13. Иерархия «Игровые платформы»
**Базовый класс `GamePlatform`:** `name`, `manufacturer`, `price`. Метод `launchGame()`.
**Подкласс `Console`:** `storageSize`. Переопределение `launchGame()`.
**Подкласс `PC`:** `os`. Переопределение `launchGame()`.

### Вариант 14. Иерархия «Печатные издания»
**Базовый класс `Publication`:** `title`, `publisher`, `year`. Метод `read()`.
**Подкласс `Book`:** `pagesCount`. Переопределение `read()`.
**Подкласс `Magazine`:** `issueNumber`. Переопределение `read()`.

### Вариант 15. Иерархия «Медицинские учреждения»
**Базовый класс `MedicalFacility`:** `name`, `address`, `bedsCount`. Метод `admitPatient()`.
**Подкласс `Hospital`:** `departmentsCount`. Переопределение `admitPatient()`.
**Подкласс `Clinic`:** `specialistsCount`. Переопределение `admitPatient()`.

### Вариант 16. Иерархия «Средства массовой информации»
**Базовый класс `Media`:** `name`, `owner`, `foundingYear`. Метод `broadcast()`.
**Подкласс `TVChannel`:** `audienceReach`. Переопределение `broadcast()`.
**Подкласс `RadioStation`:** `frequency`. Переопределение `broadcast()`.

### Вариант 17. Иерархия «Типы транспорта в городе»
**Базовый класс `UrbanTransport`:** `routeNumber`, `capacity`, `speed`. Метод `startRoute()`.
**Подкласс `Bus`:** `engineType`. Переопределение `startRoute()`.
**Подкласс `Tramway`:** `sectionsCount`. Переопределение `startRoute()`.

### Вариант 18. Иерархия «Типы отелей»
**Базовый класс `Hotel`:** `name`, `stars`, `roomsCount`. Метод `checkIn()`.
**Подкласс `BusinessHotel`:** `hasConferenceHall`. Переопределение `checkIn()`.
**Подкласс `ResortHotel`:** `hasPool`. Переопределение `checkIn()`.

### Вариант 19. Иерархия «Типы магазинов»
**Базовый класс `Store`:** `name`, `address`, `area`. Метод `openStore()`.
**Подкласс `Supermarket`:** `cashiersCount`. Переопределение `openStore()`.
**Подкласс `Boutique`:** `brand`. Переопределение `openStore()`.

### Вариант 20. Иерархия «Типы учебных курсов»
**Базовый класс `Course`:** `title`, `teacher`, `duration`. Метод `enroll()`.
**Подкласс `LectureCourse`:** `lecturesCount`. Переопределение `enroll()`.
**Подкласс `PracticalCourse`:** `labsCount`. Переопределение `enroll()`.

### Вариант 21. Иерархия «Типы документов»
**Базовый класс `Document`:** `number`, `issueDate`, `expiryDate`. Метод `validate()`.
**Подкласс `Passport`:** `series`. Переопределение `validate()`.
**Подкласс `License`:** `category`. Переопределение `validate()`.

### Вариант 22. Иерархия «Типы договоров»
**Базовый класс `Contract`:** `number`, `conclusionDate`, `term`. Метод `sign()`.
**Подкласс `EmploymentContract`:** `position`. Переопределение `sign()`.
**Подкласс `LeaseContract`:** `object`. Переопределение `sign()`.

### Вариант 23. Иерархия «Типы счетов в ресторане»
**Базовый класс `BillItem`:** `name`, `price`, `quantity`. Метод `calculateTotal()`.
**Подкласс `FoodItem`:** `category`. Переопределение `calculateTotal()`.
**Подкласс `DrinkItem`:** `volume`. Переопределение `calculateTotal()`.

### Вариант 24. Иерархия «Типы страховых полисов»
**Базовый класс `InsurancePolicy`:** `number`, `policyholder`, `sumInsured`. Метод `activate()`.
**Подкласс `AutoInsurance`:** `vehicle`. Переопределение `activate()`.
**Подкласс `HealthInsurance`:** `insuredPerson`. Переопределение `activate()`.

### Вариант 25. Иерархия «Типы билетов»
**Базовый класс `Ticket`:** `number`, `date`, `price`. Метод `validate()`.
**Подкласс `CinemaTicket`:** `movie`. Переопределение `validate()`.
**Подкласс `TheaterTicket`:** `performance`. Переопределение `validate()`.

### Вариант 26. Иерархия «Типы аккаунтов в системе»
**Базовый класс `UserAccount`:** `login`, `creationDate`. Метод `login()`.
**Подкласс `AdminAccount`:** `accessLevel`. Переопределение `login()`.
**Подкласс `RegularAccount`:** `subscriptionStatus`. Переопределение `login()`.

### Вариант 27. Иерархия «Типы уведомлений»
**Базовый класс `Notification`:** `id`, `creationDate`, `text`. Метод `send()`.
**Подкласс `EmailNotification`:** `recipientEmail`. Переопределение `send()`.
**Подкласс `SmsNotification`:** `phoneNumber`. Переопределение `send()`.

### Вариант 28. Иерархия «Типы платежей»
**Базовый класс `Payment`:** `number`, `amount`, `date`. Метод `process()`.
**Подкласс `CardPayment`:** `cardNumber`. Переопределение `process()`.
**Подкласс `CashPayment`:** `changeAmount`. Переопределение `process()`.

### Вариант 29. Иерархия «Типы файлов»
**Базовый класс `File`:** `name`, `size`, `creationDate`. Метод `open()`.
**Подкласс `TextFile`:** `encoding`. Переопределение `open()`.
**Подкласс `ImageFile`:** `resolution`. Переопределение `open()`.

### Вариант 30. Иерархия «Типы событий в календаре»
**Базовый класс `CalendarEvent`:** `name`, `startDate`, `duration`. Метод `schedule()`.
**Подкласс `Meeting`:** `location`. Переопределение `schedule()`.
**Подкласс `Birthday`:** `birthdayPerson`. Переопределение `schedule()`.

## 5. Методические указания

1. **Перед написанием кода** нарисуйте диаграмму иерархии классов.
2. **Вызов `super()`** должен быть первой инструкцией в конструкторе подкласса.
3. **Аннотация `@Override`** обязательна для всех переопределённых методов.
4. **Полиморфизм** демонстрируется через массив объектов базового типа.
5. **Интерфейс** применяется для поведения, не связанного с иерархией наследования.
6. **Используйте ИИ-инструмент** для генерации шаблона иерархии, но обязательно проверьте и доработайте результат.
7. **Ведите журнал применения ИИ** — это обязательная часть отчёта.

## 6. Контрольные вопросы

1. Что такое наследование? Какое отношение оно моделирует?
2. Что такое переопределение методов? Чем отличается от перегрузки?
3. Что такое полиморфизм? Приведите пример.
4. Что такое абстрактный класс? Можно ли создать его объект?
5. Что такое интерфейс? Чем отличается от абстрактного класса?
6. Может ли класс наследовать несколько классов? Реализовывать несколько интерфейсов?
7. Для чего нужна аннотация `@Override`?
8. Что такое ключевое слово `super`?
9. В каких случаях следует применять абстрактный класс, а в каких — интерфейс?
10. Что такое модификатор `final`? Какие ограничения он накладывает?

## 7. Рекомендуемые источники

1. Шилдт Г. *Java. Базовый курс.* — М.: Вильямс. — Главы 7–8.
2. Блох Дж. *Java. Эффективное программирование.* — М.: Питер. — Правило 18 (композиция vs наследование).
3. Oracle Java Tutorials: Interfaces and Inheritance. URL: https://docs.oracle.com/javase/tutorial/java/IandI/
