# Лабораторная работа №10. Stream API: обработка потоков данных

## 1. Паспорт работы

| Параметр | Значение |
|----------|----------|
| Тема | Stream API: обработка потоков данных |
| Номер занятия в модуле | 2 из 4 (модуль 3) |
| Продолжительность аудиторной части | 2 академических часа |
| Предшествующая подготовка | Лабораторная работа №9 (функциональные интерфейсы, лямбда-выражения, Optional) |
| Тип работы | Формирование навыков функциональной обработки коллекций с применением Stream API |

### 1.1. Цель работы

Освоить стандартный API потоков данных (Stream API) языка Java: научиться создавать потоки из различных источников, применять промежуточные и терминальные операции, использовать коллекторы для агрегации данных, понимать механизм ленивых вычислений и особенности параллельных потоков. Сформировать навыки перехода от императивного стиля обработки коллекций к функциональному.

### 1.2. Задачи работы

1. Изучить понятие потока данных (Stream) и его отличие от коллекции.
2. Освоить способы создания потоков из коллекций, массивов, файлов и других источников.
3. Изучить промежуточные операции: `filter`, `map`, `flatMap`, `distinct`, `sorted`, `peek`, `limit`, `skip`, `mapToInt`, `mapToDouble`, `mapToLong`.
4. Освоить терминальные операции: `forEach`, `collect`, `reduce`, `count`, `min`, `max`, `anyMatch`, `allMatch`, `noneMatch`, `findFirst`, `findAny`.
5. Изучить стандартные коллекторы класса `Collectors`: `toList`, `toSet`, `toMap`, `groupingBy`, `partitioningBy`, `joining`, `summarizingInt`, `summarizingLong`, `summarizingDouble`.
6. Понять принцип ленивых вычислений (lazy evaluation) и его значение для производительности.
7. Изучить параллельные потоки и их применение.
8. Развить навыки построения эффективных конвейеров обработки данных.

### 1.3. Оснащение

- JDK версии 17 или выше;
- интегрированная среда разработки IntelliJ IDEA Community Edition;
- система сборки Maven или Gradle;
- система контроля версий Git;
- система модульного тестирования JUnit 5.

## 2. Теоретические сведения

### 2.1. Мотивация: от императивного стиля к функциональному

Рассмотрим задачу фильтрации и преобразования списка строк. Императивный подход:

```java
List<String> names = Arrays.asList("Анна", "Борис", "Александр", "Виктор", "Антон");

// Императивный стиль
List<String> result = new ArrayList<>();
for (String name : names) {
    if (name.startsWith("А") && name.length() > 4) {
        result.add(name.toUpperCase());
    }
}
Collections.sort(result);
```

Функциональный подход с Stream API:

```java
// Функциональный стиль
List<String> result = names.stream()
    .filter(name -> name.startsWith("А"))
    .filter(name -> name.length() > 4)
    .map(String::toUpperCase)
    .sorted()
    .collect(Collectors.toList());
```

**Преимущества Stream API:**

1. **Выразительность.** Код описывает, *что* нужно сделать, а не *как*.
2. **Декларативность.** Операции образуют читаемый конвейер.
3. **Отсутствие промежуточных коллекций.** Не создаются временные списки.
4. **Возможность параллелизации.** Переход к параллельной обработке — одна строка кода.
5. **Ленивые вычисления.** Операции выполняются только при необходимости.

### 2.2. Понятие потока данных

**Поток (Stream)** — это последовательность элементов, поддерживающая промежуточные и терминальные операции для обработки данных. Поток не хранит данные — он передаёт элементы из источника через конвейер операций.

**Ключевые отличия от коллекции:**

| Характеристика | Коллекция | Поток |
|----------------|-----------|-------|
| Хранение данных | Хранит элементы | Не хранит, передаёт |
| Изменение | Может изменяться | Не изменяется |
| Размер | Фиксирован | Не применимо |
| Доступ к элементам | По индексу/ключу | Только последовательный |
| Итерация | Внешняя (цикл) | Внутренняя (конвейер) |
| Выполнение | Немедленное | Ленивое (при терминальной операции) |

### 2.3. Создание потоков

#### 2.3.1. Из коллекций

```java
List<String> list = Arrays.asList("Анна", "Борис", "Виктор");
Stream<String> stream = list.stream();

// Параллельный поток
Stream<String> parallelStream = list.parallelStream();
```

#### 2.3.2. Из массивов

```java
int[] numbers = {1, 2, 3, 4, 5};
IntStream intStream = Arrays.stream(numbers);

// Или через Stream.of
Stream<Integer> stream = Stream.of(1, 2, 3, 4, 5);
```

#### 2.3.3. Из отдельных значений

```java
Stream<String> stream = Stream.of("Анна", "Борис", "Виктор");
Stream<String> empty = Stream.empty();
```

#### 2.3.4. Генерация потоков

```java
// Бесконечный поток (требует limit для терминальной операции)
Stream<Integer> infinite = Stream.iterate(0, n -> n + 2);  // 0, 2, 4, 6, ...

// Поток с генерацией значений
Stream<Double> random = Stream.generate(Math::random);

// Диапазон чисел
IntStream range = IntStream.range(1, 10);        // 1..9
IntStream rangeClosed = IntStream.rangeClosed(1, 10);  // 1..10
```

#### 2.3.5. Из файлов

```java
try (Stream<String> lines = Files.lines(Paths.get("file.txt"))) {
    lines.filter(line -> !line.isEmpty())
         .forEach(System.out::println);
}
```

### 2.4. Промежуточные операции

Промежуточные операции преобразуют поток в другой поток. Они **ленивы** — не выполняются до вызова терминальной операции.

#### 2.4.1. `filter(Predicate<T>)` — фильтрация

```java
List<String> names = List.of("Анна", "Борис", "Александр", "Виктор");
List<String> longNames = names.stream()
    .filter(name -> name.length() > 5)
    .collect(Collectors.toList());
// [Александр]
```

#### 2.4.2. `map(Function<T, R>)` — преобразование

```java
List<String> names = List.of("Анна", "Борис", "Виктор");
List<Integer> lengths = names.stream()
    .map(String::length)
    .collect(Collectors.toList());
// [4, 5, 6]
```

#### 2.4.3. `flatMap(Function<T, Stream<R>>)` — плоское отображение

```java
List<List<String>> nested = List.of(
    List.of("Анна", "Борис"),
    List.of("Виктор", "Глеб")
);
List<String> flat = nested.stream()
    .flatMap(Collection::stream)
    .collect(Collectors.toList());
// [Анна, Борис, Виктор, Глеб]
```

#### 2.4.4. `distinct()` — удаление дубликатов

```java
List<String> names = List.of("Анна", "Борис", "Анна", "Виктор", "Борис");
List<String> unique = names.stream()
    .distinct()
    .collect(Collectors.toList());
// [Анна, Борис, Виктор]
```

#### 2.4.5. `sorted()` и `sorted(Comparator)` — сортировка

```java
List<String> names = List.of("Виктор", "Анна", "Борис");
List<String> sorted = names.stream()
    .sorted()
    .collect(Collectors.toList());
// [Анна, Борис, Виктор]

// Сортировка по длине
List<String> byLength = names.stream()
    .sorted(Comparator.comparingInt(String::length))
    .collect(Collectors.toList());
```

#### 2.4.6. `peek(Consumer<T>)` — просмотр элементов (для отладки)

```java
List<String> result = List.of("Анна", "Борис", "Виктор").stream()
    .peek(name -> System.out.println("До фильтра: " + name))
    .filter(name -> name.length() > 4)
    .peek(name -> System.out.println("После фильтра: " + name))
    .collect(Collectors.toList());
```

#### 2.4.7. `limit(long)` и `skip(long)` — ограничение и пропуск

```java
List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
List<Integer> first3 = numbers.stream()
    .limit(3)
    .collect(Collectors.toList());
// [1, 2, 3]

List<Integer> after5 = numbers.stream()
    .skip(5)
    .collect(Collectors.toList());
// [6, 7, 8, 9, 10]
```

#### 2.4.8. Специализированные потоки: `mapToInt`, `mapToDouble`, `mapToLong`

```java
List<String> names = List.of("Анна", "Борис", "Александр");
IntStream lengths = names.stream()
    .mapToInt(String::length);
// IntStream можно использовать для примитивных операций
int sum = lengths.sum();
OptionalInt max = lengths.max();
```

### 2.5. Терминальные операции

Терминальные операции завершают конвейер и возвращают результат. После терминальной операции поток не может быть использован повторно.

#### 2.5.1. `forEach(Consumer<T>)` — выполнение действия

```java
List<String> names = List.of("Анна", "Борис", "Виктор");
names.stream()
    .forEach(System.out::println);
```

#### 2.5.2. `collect(Collector)` — сбор результатов

```java
List<String> names = List.of("Анна", "Борис", "Виктор");
List<String> list = names.stream()
    .filter(n -> n.length() > 4)
    .collect(Collectors.toList());

Set<String> set = names.stream()
    .collect(Collectors.toSet());

String joined = names.stream()
    .collect(Collectors.joining(", "));
// "Анна, Борис, Виктор"
```

#### 2.5.3. `reduce(BinaryOperator<T>)` — свёртка

```java
List<Integer> numbers = List.of(1, 2, 3, 4, 5);

// Сумма
int sum = numbers.stream()
    .reduce(0, Integer::sum);
// 15

// Произведение
int product = numbers.stream()
    .reduce(1, (a, b) -> a * b);
// 120

// Максимум
Optional<Integer> max = numbers.stream()
    .reduce(Integer::max);
// Optional[5]
```

#### 2.5.4. `count()`, `min()`, `max()`

```java
List<Integer> numbers = List.of(5, 2, 8, 1, 9);

long count = numbers.stream().count();  // 5
Optional<Integer> min = numbers.stream().min(Integer::compareTo);  // Optional[1]
Optional<Integer> max = numbers.stream().max(Integer::compareTo);  // Optional[9]
```

#### 2.5.5. `anyMatch()`, `allMatch()`, `noneMatch()`

```java
List<String> names = List.of("Анна", "Борис", "Александр");

boolean anyLong = names.stream()
    .anyMatch(n -> n.length() > 5);  // true (Александр)

boolean allLong = names.stream()
    .allMatch(n -> n.length() > 3);  // true

boolean noneEmpty = names.stream()
    .noneMatch(String::isEmpty);  // true
```

#### 2.5.6. `findFirst()`, `findAny()`

```java
List<String> names = List.of("Анна", "Борис", "Виктор");

Optional<String> first = names.stream()
    .filter(n -> n.startsWith("Б"))
    .findFirst();
// Optional[Борис]

Optional<String> any = names.parallelStream()
    .filter(n -> n.length() > 4)
    .findAny();
// Может вернуть любой подходящий элемент
```

### 2.6. Коллекторы

Класс `Collectors` предоставляет набор готовых коллекторов для наиболее распространённых операций сбора.

#### 2.6.1. Базовые коллекторы

```java
// В список
List<String> list = stream.collect(Collectors.toList());

// В множество
Set<String> set = stream.collect(Collectors.toSet());

// В карту
Map<String, Integer> map = stream.collect(
    Collectors.toMap(
        Person::getName,      // ключ
        Person::getAge        // значение
    )
);
```

#### 2.6.2. Группировка

```java
List<Person> people = List.of(
    new Person("Анна", 25, "Москва"),
    new Person("Борис", 30, "Санкт-Петербург"),
    new Person("Виктор", 25, "Москва")
);

// Группировка по городу
Map<String, List<Person>> byCity = people.stream()
    .collect(Collectors.groupingBy(Person::getCity));
// {Москва=[Анна, Виктор], Санкт-Петербург=[Борис]}

// Группировка с подсчётом
Map<String, Long> countByCity = people.stream()
    .collect(Collectors.groupingBy(
        Person::getCity,
        Collectors.counting()
    ));
// {Москва=2, Санкт-Петербург=1}
```

#### 2.6.3. Разделение

```java
// Разделение по предикату
Map<Boolean, List<Person>> partitioned = people.stream()
    .collect(Collectors.partitioningBy(p -> p.getAge() > 25));
// {false=[Анна], true=[Борис, Виктор]}
```

#### 2.6.4. Соединение строк

```java
List<String> names = List.of("Анна", "Борис", "Виктор");

String joined = names.stream()
    .collect(Collectors.joining(", "));
// "Анна, Борис, Виктор"

String withPrefix = names.stream()
    .collect(Collectors.joining(", ", "[", "]"));
// "[Анна, Борис, Виктор]"
```

#### 2.6.5. Статистика

```java
List<Person> people = List.of(
    new Person("Анна", 25),
    new Person("Борис", 30),
    new Person("Виктор", 35)
);

// Статистика по возрасту
IntSummaryStatistics stats = people.stream()
    .collect(Collectors.summarizingInt(Person::getAge));

System.out.println("Количество: " + stats.getCount());      // 3
System.out.println("Сумма: " + stats.getSum());             // 90
System.out.println("Минимум: " + stats.getMin());           // 25
System.out.println("Максимум: " + stats.getMax());          // 35
System.out.println("Среднее: " + stats.getAverage());       // 30.0
```

### 2.7. Ленивые вычисления

Промежуточные операции выполняются **лениво** — они не обрабатывают элементы до вызова терминальной операции. Более того, терминальная операция может запросить только необходимое количество элементов.

```java
List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

List<Integer> result = numbers.stream()
    .filter(n -> {
        System.out.println("filter: " + n);
        return n > 5;
    })
    .map(n -> {
        System.out.println("map: " + n);
        return n * 2;
    })
    .limit(2)  // Запрашиваем только 2 элемента
    .collect(Collectors.toList());

// Вывод:
// filter: 1
// filter: 2
// ...
// filter: 6
// map: 6
// filter: 7
// map: 7
// Результат: [12, 14]
// Элементы 8, 9, 10 не обрабатывались!
```

### 2.8. Параллельные потоки

Параллельные потоки разделяют данные на части и обрабатывают их в нескольких потоках. Применяются для ускорения обработки больших объёмов данных.

```java
List<Integer> numbers = IntStream.range(0, 1_000_000)
    .boxed()
    .collect(Collectors.toList());

// Последовательная обработка
long start = System.currentTimeMillis();
long sum = numbers.stream()
    .mapToLong(Integer::longValue)
    .sum();
long sequential = System.currentTimeMillis() - start;

// Параллельная обработка
start = System.currentTimeMillis();
long parallelSum = numbers.parallelStream()
    .mapToLong(Integer::longValue)
    .sum();
long parallel = System.currentTimeMillis() - start;

System.out.println("Последовательно: " + sequential + " мс");
System.out.println("Параллельно: " + parallel + " мс");
```

**Важно:** параллельные потоки не всегда быстрее. Они эффективны для:
- больших объёмов данных (тысячи элементов и более);
- операций, не зависящих от порядка;
- операций без общего изменяемого состояния.

## 3. Примеры выполнения

### 3.1. Пример 1. Анализ списка студентов

```java
import java.util.*;
import java.util.stream.*;

/**
 * Демонстрация применения Stream API для анализа данных студентов
 */
public class StudentAnalysis {
    record Student(String name, String group, double averageGrade, int age) {}

    public static void main(String[] args) {
        List<Student> students = List.of(
            new Student("Иванов И.", "ПИ-21", 4.5, 19),
            new Student("Петров П.", "ПИ-21", 3.8, 20),
            new Student("Сидоров С.", "ПИ-22", 4.9, 19),
            new Student("Кузнецов К.", "ПИ-22", 4.2, 21),
            new Student("Смирнов С.", "ПИ-21", 4.7, 20),
            new Student("Попов П.", "ПИ-23", 3.5, 18),
            new Student("Волков В.", "ПИ-23", 4.0, 19),
            new Student("Зайцев З.", "ПИ-22", 4.6, 20)
        );

        // 1. Стипендиаты (средний балл >= 4.5)
        System.out.println("=== Стипендиаты ===");
        List<String> scholarshipStudents = students.stream()
            .filter(s -> s.averageGrade() >= 4.5)
            .map(Student::name)
            .sorted()
            .collect(Collectors.toList());
        scholarshipStudents.forEach(System.out::println);

        // 2. Средний балл по группам
        System.out.println("\n=== Средний балл по группам ===");
        Map<String, Double> averageByGroup = students.stream()
            .collect(Collectors.groupingBy(
                Student::group,
                Collectors.averagingDouble(Student::averageGrade)
            ));
        averageByGroup.forEach((group, avg) ->
            System.out.printf("Группа %s: %.2f%n", group, avg));

        // 3. Топ-3 студента по среднему баллу
        System.out.println("\n=== Топ-3 студента ===");
        students.stream()
            .sorted(Comparator.comparingDouble(Student::averageGrade).reversed())
            .limit(3)
            .forEach(s -> System.out.printf("%s (%s): %.2f%n",
                s.name(), s.group(), s.averageGrade()));

        // 4. Статистика по возрасту
        System.out.println("\n=== Статистика по возрасту ===");
        IntSummaryStatistics ageStats = students.stream()
            .collect(Collectors.summarizingInt(Student::age));
        System.out.println("Количество: " + ageStats.getCount());
        System.out.println("Минимальный возраст: " + ageStats.getMin());
        System.out.println("Максимальный возраст: " + ageStats.getMax());
        System.out.println("Средний возраст: " + ageStats.getAverage());

        // 5. Количество студентов в каждой группе
        System.out.println("\n=== Количество студентов по группам ===");
        Map<String, Long> countByGroup = students.stream()
            .collect(Collectors.groupingBy(
                Student::group,
                Collectors.counting()
            ));
        countByGroup.forEach((group, count) ->
            System.out.printf("Группа %s: %d студентов%n", group, count));

        // 6. Список имён через запятую
        System.out.println("\n=== Все имена ===");
        String allNames = students.stream()
            .map(Student::name)
            .collect(Collectors.joining(", "));
        System.out.println(allNames);

        // 7. Проверка условий
        System.out.println("\n=== Проверка условий ===");
        boolean allPassing = students.stream()
            .allMatch(s -> s.averageGrade() >= 3.0);
        System.out.println("Все студенты учатся на '3' и выше: " + allPassing);

        boolean anyExcellent = students.stream()
            .anyMatch(s -> s.averageGrade() >= 4.9);
        System.out.println("Есть отличники (4.9+): " + anyExcellent);

        // 8. Сумма средних баллов
        System.out.println("\n=== Сумма средних баллов ===");
        double totalAverage = students.stream()
            .mapToDouble(Student::averageGrade)
            .sum();
        System.out.printf("Сумма: %.2f%n", totalAverage);
    }
}
```

### 3.2. Пример 2. Обработка вложенных структур с `flatMap`

```java
import java.util.*;
import java.util.stream.*;

/**
 * Демонстрация применения flatMap для обработки вложенных структур
 */
public class FlatMapDemo {
    record Department(String name, List<Employee> employees) {}
    record Employee(String name, double salary, List<String> skills) {}

    public static void main(String[] args) {
        List<Department> departments = List.of(
            new Department("Разработка", List.of(
                new Employee("Иванов", 150000, List.of("Java", "Spring")),
                new Employee("Петров", 180000, List.of("Java", "Kotlin"))
            )),
            new Department("Аналитика", List.of(
                new Employee("Сидоров", 120000, List.of("SQL", "Python")),
                new Employee("Кузнецов", 140000, List.of("Python", "R"))
            ))
        );

        // 1. Все сотрудники всех отделов (flatMap)
        System.out.println("=== Все сотрудники ===");
        departments.stream()
            .flatMap(dept -> dept.employees().stream())
            .forEach(emp -> System.out.println(emp.name()));

        // 2. Все уникальные навыки (flatMap + distinct)
        System.out.println("\n=== Все уникальные навыки ===");
        List<String> allSkills = departments.stream()
            .flatMap(dept -> dept.employees().stream())
            .flatMap(emp -> emp.skills().stream())
            .distinct()
            .sorted()
            .collect(Collectors.toList());
        System.out.println(allSkills);

        // 3. Сотрудники с зарплатой выше 130000
        System.out.println("\n=== Высокооплачиваемые сотрудники ===");
        departments.stream()
            .flatMap(dept -> dept.employees().stream())
            .filter(emp -> emp.salary() > 130000)
            .forEach(emp -> System.out.printf("%s: %.0f руб.%n",
                emp.name(), emp.salary()));

        // 4. Средняя зарплата по отделам
        System.out.println("\n=== Средняя зарплата по отделам ===");
        Map<String, Double> avgSalaryByDept = departments.stream()
            .collect(Collectors.toMap(
                Department::name,
                dept -> dept.employees().stream()
                    .mapToDouble(Employee::salary)
                    .average()
                    .orElse(0.0)
            ));
        avgSalaryByDept.forEach((dept, avg) ->
            System.out.printf("%s: %.0f руб.%n", dept, avg));

        // 5. Количество сотрудников со знанием Java
        System.out.println("\n=== Сотрудники со знанием Java ===");
        long javaDevelopers = departments.stream()
            .flatMap(dept -> dept.employees().stream())
            .filter(emp -> emp.skills().contains("Java"))
            .count();
        System.out.println("Количество: " + javaDevelopers);
    }
}
```

### 3.3. Пример 3. Сложная аналитика с коллекторами

```java
import java.util.*;
import java.util.stream.*;

/**
 * Демонстрация сложных операций с коллекторами
 */
public class AdvancedCollectors {
    record Sale(String product, String category, double amount, LocalDate date) {}

    public static void main(String[] args) {
        List<Sale> sales = List.of(
            new Sale("Ноутбук", "Электроника", 80000, LocalDate.of(2026, 1, 15)),
            new Sale("Мышь", "Электроника", 1500, LocalDate.of(2026, 1, 20)),
            new Sale("Книга", "Книги", 800, LocalDate.of(2026, 2, 5)),
            new Sale("Клавиатура", "Электроника", 3500, LocalDate.of(2026, 2, 10)),
            new Sale("Роман", "Книги", 600, LocalDate.of(2026, 3, 1)),
            new Sale("Монитор", "Электроника", 25000, LocalDate.of(2026, 3, 15))
        );

        // 1. Общая сумма продаж по категориям
        System.out.println("=== Сумма продаж по категориям ===");
        Map<String, Double> totalByCategory = sales.stream()
            .collect(Collectors.groupingBy(
                Sale::category,
                Collectors.summingDouble(Sale::amount)
            ));
        totalByCategory.forEach((cat, total) ->
            System.out.printf("%s: %.0f руб.%n", cat, total));

        // 2. Количество продаж по месяцам
        System.out.println("\n=== Количество продаж по месяцам ===");
        Map<Integer, Long> salesByMonth = sales.stream()
            .collect(Collectors.groupingBy(
                sale -> sale.date().getMonthValue(),
                TreeMap::new,
                Collectors.counting()
            ));
        salesByMonth.forEach((month, count) ->
            System.out.printf("Месяц %d: %d продаж%n", month, count));

        // 3. Топ-3 товара по сумме продаж
        System.out.println("\n=== Топ-3 товара ===");
        sales.stream()
            .collect(Collectors.groupingBy(
                Sale::product,
                Collectors.summingDouble(Sale::amount)
            ))
            .entrySet().stream()
            .sorted(Map.Entry.<String, Double>comparingByValue().reversed())
            .limit(3)
            .forEach(entry ->
                System.out.printf("%s: %.0f руб.%n", entry.getKey(), entry.getValue()));

        // 4. Статистика продаж по категориям
        System.out.println("\n=== Статистика по категориям ===");
        Map<String, DoubleSummaryStatistics> statsByCategory = sales.stream()
            .collect(Collectors.groupingBy(
                Sale::category,
                Collectors.summarizingDouble(Sale::amount)
            ));
        statsByCategory.forEach((cat, stats) -> {
            System.out.printf("%s: средняя=%.0f, мин=%.0f, макс=%.0f%n",
                cat, stats.getAverage(), stats.getMin(), stats.getMax());
        });

        // 5. Разделение продаж на крупные (>10000) и мелкие
        System.out.println("\n=== Разделение по сумме ===");
        Map<Boolean, List<Sale>> partitioned = sales.stream()
            .collect(Collectors.partitioningBy(sale -> sale.amount() > 10000));
        System.out.println("Крупные продажи:");
        partitioned.get(true).forEach(sale ->
            System.out.printf("  %s: %.0f руб.%n", sale.product(), sale.amount()));
        System.out.println("Мелкие продажи:");
        partitioned.get(false).forEach(sale ->
            System.out.printf("  %s: %.0f руб.%n", sale.product(), sale.amount()));

        // 6. Все названия товаров через запятую
        System.out.println("\n=== Все товары ===");
        String allProducts = sales.stream()
            .map(Sale::product)
            .distinct()
            .sorted()
            .collect(Collectors.joining(", ", "[", "]"));
        System.out.println(allProducts);
    }
}
```

## 4. Задания на паре

### Задание 4.1. Анализ данных о продажах

Разработайте программу для анализа данных о продажах с применением Stream API.

**Запись `Sale`:**
- Компоненты: `id` (String), `product` (String), `category` (String), `amount` (double), `quantity` (int), `date` (LocalDate), `region` (String).

**Требования к реализации:**

1. Создайте список не менее чем из 15 продаж.
2. Реализуйте следующие операции с использованием Stream API:
   - общая сумма продаж по категориям;
   - количество продаж по регионам;
   - топ-5 товаров по сумме продаж;
   - средняя сумма продажи по месяцам;
   - товары, проданные более 10 раз;
   - статистика продаж по регионам (сумма, среднее, мин, макс);
   - разделение продаж на крупные (>50000) и мелкие;
   - список уникальных категорий, отсортированный по алфавиту.
3. Все операции должны выполняться с использованием Stream API без явных циклов.
4. Применяйте различные коллекторы: `groupingBy`, `summingDouble`, `averagingInt`, `counting`, `partitioningBy`, `joining`.

**Пример выполнения программы:**

```
=== Сумма продаж по категориям ===
Электроника: 450000 руб.
Книги: 25000 руб.
Одежда: 180000 руб.

=== Топ-5 товаров ===
Ноутбук: 320000 руб.
Смартфон: 280000 руб.
...

=== Статистика по регионам ===
Москва: средняя=45000, мин=1500, макс=120000
Санкт-Петербург: средняя=38000, мин=2000, макс=95000
```

---

### Задание 4.2. Обработка текстовых данных

Разработайте программу для обработки текстовых данных с применением Stream API.

**Требования к реализации:**

1. Реализуйте метод `analyzeText(String text)`, возвращающий объект `TextAnalysisResult`, содержащий:
   - количество слов;
   - количество уникальных слов;
   - топ-10 наиболее частых слов;
   - среднюю длину слова;
   - самые длинные слова;
   - количество предложений;
   - количество абзацев.

2. Все операции должны выполняться с использованием Stream API.
3. Слова определяются как последовательности букв, разделённые пробелами и знаками препинания.
4. Применяйте `flatMap`, `filter`, `map`, `collect`, `groupingBy`, `comparing`.

**Пример выполнения программы:**

```
=== Анализ текста ===
Количество слов: 150
Уникальных слов: 87
Средняя длина слова: 5.2

=== Топ-10 частых слов ===
1. и — 15
2. в — 12
3. на — 10
...

=== Самые длинные слова ===
программирование (14)
объектно-ориентированный (22)
```

---

### Задание 4.3. Система отчётов с группировкой и агрегацией

Разработайте систему формирования отчётов с применением Stream API и коллекторов.

**Запись `Employee`:**
- Компоненты: `id` (String), `name` (String), `department` (String), `position` (String), `salary` (double), `hireDate` (LocalDate), `performanceScore` (int, от 1 до 10).

**Требования к реализации:**

1. Создайте список не менее чем из 20 сотрудников.
2. Реализуйте следующие отчёты:
   - средняя зарплата по отделам;
   - количество сотрудников по должностям;
   - топ-3 сотрудника по performance score в каждом отделе;
   - сотрудники, проработавшие более 5 лет;
   - распределение сотрудников по возрастным группам (20-29, 30-39, 40-49, 50+);
   - общая сумма зарплат по отделам с сортировкой по убыванию;
   - сотрудники с зарплатой выше средней по компании;
   - статистика performance score по отделам.
3. Все отчёты должны формироваться с использованием Stream API.
4. Применяйте сложные коллекторы: `groupingBy`, `mapping`, `collectingAndThen`, `summarizingDouble`.

## 5. Задание для самостоятельной работы

Разработать систему согласно своему варианту с активным применением Stream API. Требования:

1. Применение не менее 8 различных операций Stream API (промежуточных и терминальных).
2. Применение не менее 4 различных коллекторов.
3. Применение `flatMap` для обработки вложенных структур.
4. Применение `groupingBy` и `partitioningBy` для группировки данных.
5. Применение `reduce` для агрегации.
6. Демонстрация ленивых вычислений (например, с `limit`).
7. Демонстрационный класс с не менее чем 8 различными аналитическими операциями.

### Варианты заданий

**Вариант 1.** Система «Библиотека». Анализ выдач книг: статистика по жанрам, топ популярных авторов, среднее количество страниц, анализ задолженностей.

**Вариант 2.** Система «Авиакомпания». Анализ рейсов: загруженность направлений, статистика задержек, топ популярных маршрутов, анализ пассажиропотока.

**Вариант 3.** Система «Банк». Анализ транзакций: обороты по счетам, топ клиентов, статистика по типам операций, анализ комиссий.

**Вариант 4.** Система «Интернет-магазин». Анализ заказов: статистика по товарам, топ покупателей, средний чек, анализ возвратов.

**Вариант 5.** Система «Больница». Анализ приёмов: статистика по врачам, загруженность отделений, среднее время приёма, анализ диагнозов.

**Вариант 6.** Система «Университет». Анализ успеваемости: средний балл по группам, топ студентов, статистика по предметам, анализ стипендиатов.

**Вариант 7.** Система «Ресторан». Анализ заказов: статистика по блюдам, среднее время приготовления, топ популярных позиций, анализ выручки по часам.

**Вариант 8.** Система «Фитнес-клуб». Анализ посещений: статистика по абонементам, загруженность по времени, топ тренеров, анализ продлений.

**Вариант 9.** Система «Автосервис». Анализ заказов-нарядов: статистика по услугам, средняя стоимость ремонта, топ мастеров, анализ загруженности.

**Вариант 10.** Система «Турагентство». Анализ продаж туров: топ направлений, средняя стоимость, статистика по сезонам, анализ возвратов.

**Вариант 11.** Система «Кинотеатр». Анализ продаж билетов: статистика по фильмам, загруженность сеансов, топ жанров, анализ выручки.

**Вариант 12.** Система «Склад». Анализ движения товаров: статистика по категориям, топ поставщиков, анализ остатков, скорость оборачиваемости.

**Вариант 13.** Система «Страховая компания». Анализ страховых случаев: статистика по типам, средние выплаты, топ регионов, анализ частоты.

**Вариант 14.** Система «Почта». Анализ отправлений: статистика по типам, среднее время доставки, топ направлений, анализ потерь.

**Вариант 15.** Система «Агентство недвижимости». Анализ сделок: статистика по районам, средняя стоимость, топ агентов, анализ времени продажи.

**Вариант 16.** Система «Кадровое агентство». Анализ подбора персонала: статистика по вакансиям, среднее время закрытия, топ кандидатов, анализ отказов.

**Вариант 17.** Система «Благотворительный фонд». Анализ пожертвований: статистика по кампаниям, средний размер взноса, топ жертвователей, анализ динамики.

**Вариант 18.** Система «Такси». Анализ поездок: статистика по водителям, средняя стоимость, топ районов, анализ часов пик.

**Вариант 19.** Система «Музей». Анализ посещений: статистика по выставкам, средний возраст посетителей, топ экспонатов, анализ выручки.

**Вариант 20.** Система «Спортивный клуб». Анализ соревнований: статистика по видам спорта, топ спортсменов, средние результаты, анализ рекордов.

**Вариант 21.** Система «Почтовый клиент». Анализ корреспонденции: статистика по папкам, топ отправителей, среднее время ответа, анализ непрочитанных.

**Вариант 22.** Система «Календарь». Анализ событий: статистика по типам, средняя длительность, топ участников, анализ загруженности дней.

**Вариант 23.** Система «Платёжная система». Анализ платежей: статистика по методам, средние суммы, топ merchants, анализ отказов.

**Вариант 24.** Система «Управление проектами». Анализ задач: статистика по статусам, среднее время выполнения, топ исполнителей, анализ просрочек.

**Вариант 25.** Система «Ветеринарная клиника». Анализ приёмов: статистика по видам животных, средние чеки, топ врачей, анализ диагнозов.

**Вариант 26.** Система «Железнодорожные перевозки». Анализ перевозок: статистика по направлениям, средняя загруженность, топ поездов, анализ опозданий.

**Вариант 27.** Система «Языковые курсы». Анализ обучения: статистика по уровням, средние баллы, топ преподавателей, анализ отсева.

**Вариант 28.** Система «Фотостудия». Анализ фотосессий: статистика по типам, среднее количество фотографий, топ клиентов, анализ выручки.

**Вариант 29.** Система «Коворкинг». Анализ бронирований: статистика по типам мест, средняя длительность, топ резидентов, анализ загруженности.

**Вариант 30.** Система «Платформа подкастов». Анализ прослушиваний: статистика по подкастам, среднее время прослушивания, топ эпизодов, анализ подписчиков.

## 6. Методические указания к самостоятельной работе

1. **Проектирование конвейера.** Перед реализацией определите:
   - источник данных (коллекция, файл, генератор);
   - последовательность промежуточных операций;
   - терминальную операцию и тип результата;
   - необходимые коллекторы для агрегации.

2. **Порядок операций.** Соблюдайте оптимальный порядок операций:
   - фильтрацию (`filter`) выполняйте как можно раньше — это уменьшает объём данных;
   - преобразование (`map`) — после фильтрации;
   - сортировку (`sorted`) — перед `limit`;
   - `distinct` — после фильтрации, но до сортировки.

3. **Избегание побочных эффектов.** Лямбда-выражения в Stream API не должны изменять внешнее состояние:
   ```java
   // ПЛОХО: изменение внешнего списка
   List<String> result = new ArrayList<>();
   stream.forEach(s -> result.add(s.toUpperCase()));

   // ХОРОШО: использование collect
   List<String> result = stream
       .map(String::toUpperCase)
       .collect(Collectors.toList());
   ```

4. **Применение `flatMap`.** Используйте `flatMap` для обработки вложенных коллекций:
   - если элемент содержит коллекцию — `flatMap(e -> e.getCollection().stream())`;
   - для «разглаживания» иерархической структуры.

5. **Применение коллекторов.** Для агрегации данных используйте специализированные коллекторы:
   - `groupingBy` — группировка по ключу;
   - `partitioningBy` — разделение по предикату;
   - `summarizingInt/Long/Double` — статистика;
   - `joining` — соединение строк;
   - `toMap` — преобразование в карту.

6. **Ленивые вычисления.** Помните, что промежуточные операции не выполняются до терминальной. Это означает:
   - операции выполняются «за один проход»;
   - `limit` может прервать обработку раньше;
   - `peek` применяется только для отладки, не для бизнес-логики.

7. **Параллельные потоки.** Применяйте `parallelStream()` только при:
   - большом объёме данных (тысячи элементов);
   - отсутствии общего изменяемого состояния;
   - независимости результатов от порядка.

8. **Тестирование.** Перед сдачей работы проверьте:
   - корректность всех 8+ аналитических операций;
   - отсутствие явных циклов (`for`, `while`) в логике обработки;
   - применение `flatMap` для вложенных структур;
   - корректность группировок и агрегаций;
   - обработку пустых коллекций.

9. **Применение ИИ.** При использовании средств ИИ:
   - генерируйте по отдельности конвейеры Stream API для каждой операции;
   - обязательно проверяйте порядок операций и корректность коллекторов;
   - не делегируйте ИИ проектирование конвейеров без понимания ленивых вычислений.

10. **Оформление отчёта.** Отчёт должен содержать:
    - листинги всех файлов проекта с комментариями;
    - протокол работы демонстрационного класса;
    - обоснование выбора операций и коллекторов;
    - ответы на контрольные вопросы;
    - выводы по проделанной работе.

## 7. Контрольные вопросы

1. Что такое поток данных (Stream)? Чем он отличается от коллекции?
2. Какие способы создания потоков существуют в Java?
3. Что такое промежуточные операции? Приведите примеры.
4. Что такое терминальные операции? Приведите примеры.
5. В чём различие между `map` и `flatMap`?
6. Что такое ленивые вычисления? Как они проявляются в Stream API?
7. Какие коллекторы предоставляет класс `Collectors`?
8. Для чего применяется `groupingBy`? Чем он отличается от `partitioningBy`?
9. Что такое свёртка (`reduce`)? Как она применяется?
10. Для чего применяются методы `anyMatch`, `allMatch`, `noneMatch`?
11. Что такое параллельные потоки? В каких случаях их следует применять?
12. Почему нельзя повторно использовать поток после терминальной операции?
13. Каковы типичные ошибки при использовании Stream API?
14. Почему не рекомендуется использовать `peek` для бизнес-логики?
15. В каких случаях Stream API даёт выигрыш в производительности по сравнению с циклами?

## 8. Рекомендуемые источники

1. Хорстманн К. *Java. Библиотека профессионала. Том 2.* — М.: Вильямс. — Глава 6 (Потоки данных).
2. Блох Дж. *Java. Эффективное программирование.* — М.: Питер. — Правила 45–48 (Stream API).
3. Урма Р., Фуко М., Уорбертон А. *Java 8 в действии.* — М.: Вильямс. — Главы 4–7.
4. Тафти Н. *Java 8: Лямбда-выражения и Stream API.* — СПб.: БХВ-Петербург.
5. Oracle Java Tutorials. Lesson: Aggregate Operations. URL: https://docs.oracle.com/javase/tutorial/collections/streams/
6. Baeldung. Java 8 Stream API Tutorial. URL: https://www.baeldung.com/java-8-streams
7. Baeldung. Java Collectors. URL: https://www.baeldung.com/java-8-collectors
