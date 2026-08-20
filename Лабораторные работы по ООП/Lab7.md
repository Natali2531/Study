# Лабораторная работа №7. Стандартная библиотека коллекций

## 1. Паспорт работы

| Параметр | Значение |
|----------|----------|
| Тема | Стандартная библиотека коллекций (Collections Framework) |
| Номер занятия в модуле | 3 из 4 (модуль 2) |
| Продолжительность аудиторной части | 2 академических часа |
| Предшествующая подготовка | Модуль 1 (базовый синтаксис, ООП), лабораторные работы №5–6 (перечисления, записи, обобщения) |
| Тип работы | Формирование навыков работы со стандартной библиотекой коллекций Java |

### 1.1. Цель работы

Освоить стандартную библиотеку коллекций языка Java (Collections Framework): изучить иерархию интерфейсов и реализаций, научиться выбирать подходящую структуру данных в зависимости от решаемой задачи, применять алгоритмы класса `Collections`, а также использовать интерфейсы `Comparable` и `Comparator` для сортировки и сравнения объектов.

### 1.2. Задачи работы

1. Изучить иерархию интерфейсов библиотеки коллекций: `Collection`, `List`, `Set`, `Queue`, `Map`.
2. Освоить основные реализации коллекций: `ArrayList`, `LinkedList`, `HashSet`, `TreeSet`, `LinkedHashSet`, `HashMap`, `TreeMap`, `LinkedHashMap`, `PriorityQueue`, `ArrayDeque`.
3. Понять контракты коллекций и различия между реализациями.
4. Изучить интерфейсы `Comparable` и `Comparator` для сортировки объектов.
5. Освоить алгоритмы класса `Collections` (сортировка, перемешивание, поиск, частота).
6. Научиться применять неизменяемые коллекции (`List.of`, `Map.of`, `Collections.unmodifiable*`).
7. Изучить механизм итераторов и fail-fast поведение.
8. Развить навыки выбора структуры данных в зависимости от требований задачи.

### 1.3. Оснащение

- JDK версии 17 или выше;
- интегрированная среда разработки IntelliJ IDEA Community Edition;
- система сборки Maven или Gradle;
- система контроля версий Git;
- система модульного тестирования JUnit 5 (рекомендуется).

## 2. Теоретические сведения

### 2.1. Мотивация: зачем нужны коллекции

В лабораторных работах №1–6 для хранения групп объектов применялись массивы. Массивы имеют ряд ограничений:

1. **Фиксированный размер.** Размер массива определяется при создании и не может быть изменён.
2. **Отсутствие готовых алгоритмов.** Поиск, сортировка, фильтрация должны реализовываться вручную.
3. **Отсутствие семантики.** Массив не различает множества, списки, очереди, карты.
4. **Неэффективность для некоторых операций.** Вставка в начало или середина массива требует сдвига элементов.

Библиотека коллекций устраняет указанные недостатки, предоставляя богатый набор структур данных и готовых алгоритмов.

### 2.2. Иерархия интерфейсов коллекций

Библиотека коллекций Java построена на системе интерфейсов, определяющих контракты различных типов коллекций:

```
Iterable
    └── Collection
            ├── List          (упорядоченная коллекция с индексами)
            ├── Set           (множество уникальных элементов)
            ├── Queue         (очередь)
            │       └── Deque (двусторонняя очередь)
            └── (отдельно) Map (карта «ключ-значение»)
```

**Ключевые интерфейсы:**

| Интерфейс | Назначение | Допускает дубликаты | Упорядоченность |
|-----------|------------|:-------------------:|:---------------:|
| `Collection<E>` | Базовый интерфейс для всех коллекций | Зависит от реализации | Зависит от реализации |
| `List<E>` | Упорядоченный список с доступом по индексу | Да | По порядку вставки |
| `Set<E>` | Множество уникальных элементов | Нет | Зависит от реализации |
| `Queue<E>` | Очередь (FIFO) | Да | По порядку вставки |
| `Deque<E>` | Двусторонняя очередь | Да | По порядку вставки |
| `Map<K, V>` | Карта «ключ-значение» | Ключи — нет | Зависит от реализации |

### 2.3. Реализации списков (`List`)

#### 2.3.1. `ArrayList`

Динамический массив на основе обычного массива. Обеспечивает быстрый доступ по индексу (`O(1)`), но медленную вставку/удаление в середину (`O(n)`).

**Применение:** большинство задач, когда преобладают операции чтения и добавления в конец.

```java
List<String> list = new ArrayList<>();
list.add("Анна");          // O(1) амортизированное
list.add("Борис");
list.add("Виктор");

String name = list.get(1); // O(1) — быстрый доступ по индексу
list.remove(0);            // O(n) — сдвиг элементов
```

#### 2.3.2. `LinkedList`

Двусвязный список. Обеспечивает быструю вставку/удаление в начало и конец (`O(1)`), но медленный доступ по индексу (`O(n)`).

**Применение:** когда преобладают операции вставки/удаления в начало и конец, или когда требуется реализация `Deque`.

```java
List<String> list = new LinkedList<>();
list.add("Анна");          // O(1)
list.add(0, "Первый");     // O(1) — вставка в начало
// list.get(5);            // O(n) — медленный доступ по индексу
```

#### 2.3.3. Сравнение реализаций

| Операция | `ArrayList` | `LinkedList` |
|----------|:-----------:|:------------:|
| Доступ по индексу | O(1) | O(n) |
| Вставка в конец | O(1) ам. | O(1) |
| Вставка в начало | O(n) | O(1) |
| Вставка в середину | O(n) | O(n) |
| Удаление по индексу | O(n) | O(n) |
| Потребление памяти | Меньше | Больше (узел + ссылки) |

**Общее правило:** в большинстве случаев предпочитайте `ArrayList`. `LinkedList` оправдан только при частых вставках/удалениях в начало списка или при использовании в качестве `Deque`.

### 2.4. Реализации множеств (`Set`)

#### 2.4.1. `HashSet`

Множество на основе хеш-таблицы (`HashMap`). Обеспечивает операции `add`, `remove`, `contains` за `O(1)`. Не гарантирует порядок элементов.

**Требования к элементам:** корректная реализация `equals()` и `hashCode()`.

```java
Set<String> names = new HashSet<>();
names.add("Анна");
names.add("Борис");
names.add("Анна");   // дубликат не добавится

System.out.println(names.contains("Анна"));  // true
System.out.println(names.size());             // 2
```

#### 2.4.2. `LinkedHashSet`

Множество на основе хеш-таблицы с поддержкой порядка вставки. Сохраняет порядок, в котором элементы были добавлены.

```java
Set<String> names = new LinkedHashSet<>();
names.add("Виктор");
names.add("Анна");
names.add("Борис");

System.out.println(names);   // [Виктор, Анна, Борис] — порядок вставки
```

#### 2.4.3. `TreeSet`

Множество на основе красно-чёрного дерева. Элементы упорядочены согласно `Comparable` или `Comparator`. Операции `add`, `remove`, `contains` выполняются за `O(log n)`.

**Требования к элементам:** реализация `Comparable` или передача `Comparator` в конструктор.

```java
Set<String> names = new TreeSet<>();
names.add("Виктор");
names.add("Анна");
names.add("Борис");

System.out.println(names);   // [Анна, Борис, Виктор] — естественный порядок
```

#### 2.4.4. Сравнение реализаций

| Операция | `HashSet` | `LinkedHashSet` | `TreeSet` |
|----------|:---------:|:---------------:|:---------:|
| `add`, `remove`, `contains` | O(1) | O(1) | O(log n) |
| Порядок | Не определён | По вставке | По сравнению |
| Требования к элементам | `equals`, `hashCode` | `equals`, `hashCode` | `Comparable` или `Comparator` |
| Потребление памяти | Среднее | Больше | Больше |

### 2.5. Реализации карт (`Map`)

#### 2.5.1. `HashMap`

Карта на основе хеш-таблицы. Обеспечивает операции `put`, `get`, `remove` за `O(1)`. Не гарантирует порядок.

```java
Map<String, Integer> ages = new HashMap<>();
ages.put("Анна", 20);
ages.put("Борис", 22);
ages.put("Виктор", 19);

System.out.println(ages.get("Анна"));         // 20
System.out.println(ages.containsKey("Борис")); // true
```

#### 2.5.2. `LinkedHashMap`

Карта на основе хеш-таблицы с поддержкой порядка вставки (или порядка доступа — при специальном конструкторе).

```java
Map<String, Integer> ages = new LinkedHashMap<>();
ages.put("Виктор", 19);
ages.put("Анна", 20);
ages.put("Борис", 22);

System.out.println(ages);   // {Виктор=19, Анна=20, Борис=22}
```

#### 2.5.3. `TreeMap`

Карта на основе красно-чёрного дерева. Ключи упорядочены согласно `Comparable` или `Comparator`. Операции выполняются за `O(log n)`.

```java
Map<String, Integer> ages = new TreeMap<>();
ages.put("Виктор", 19);
ages.put("Анна", 20);
ages.put("Борис", 22);

System.out.println(ages);   // {Анна=20, Борис=22, Виктор=19}
```

#### 2.5.4. Сравнение реализаций

| Операция | `HashMap` | `LinkedHashMap` | `TreeMap` |
|----------|:---------:|:---------------:|:---------:|
| `put`, `get`, `remove` | O(1) | O(1) | O(log n) |
| Порядок ключей | Не определён | По вставке | По сравнению |
| Требования к ключам | `equals`, `hashCode` | `equals`, `hashCode` | `Comparable` или `Comparator` |

### 2.6. Очереди и деки

#### 2.6.1. `PriorityQueue`

Очередь с приоритетом. Элементы извлекаются в порядке приоритета (определяется `Comparable` или `Comparator`). По умолчанию — минимальный элемент первым.

```java
Queue<Integer> pq = new PriorityQueue<>();
pq.add(5);
pq.add(1);
pq.add(3);

System.out.println(pq.poll());   // 1
System.out.println(pq.poll());   // 3
System.out.println(pq.poll());   // 5
```

#### 2.6.2. `ArrayDeque`

Двусторонняя очередь на основе массива. Быстрее `LinkedList` для большинства операций. Может использоваться как стек (LIFO) и как очередь (FIFO).

```java
Deque<String> deque = new ArrayDeque<>();
deque.addFirst("Первый");   // в начало
deque.addLast("Последний"); // в конец
deque.push("Стек");         // как стек

System.out.println(deque.pop());    // Стек (LIFO)
System.out.println(deque.poll());   // Первый (FIFO)
```

### 2.7. Интерфейсы `Comparable` и `Comparator`

#### 2.7.1. `Comparable<T>` — естественный порядок

Интерфейс определяет **естественный порядок** элементов. Класс реализует метод `compareTo`:

```java
public class Student implements Comparable<Student> {
    private String name;
    private double averageGrade;

    // ...

    /**
     * Естественный порядок — по среднему баллу (по убыванию)
     */
    @Override
    public int compareTo(Student other) {
        return Double.compare(other.averageGrade, this.averageGrade);
    }
}
```

#### 2.7.2. `Comparator<T>` — внешний порядок

Интерфейс определяет **внешний порядок** — порядок, задаваемый извне, независимо от класса:

```java
// Сортировка по имени
Comparator<Student> byName = Comparator.comparing(Student::getName);

// Сортировка по возрасту, затем по имени
Comparator<Student> byAgeThenName = Comparator
    .comparing(Student::getAge)
    .thenComparing(Student::getName);

// Обратный порядок
Comparator<Student> reversed = byName.reversed();
```

#### 2.7.3. Применение

```java
List<Student> students = new ArrayList<>();
// ... заполнение

// Сортировка по естественному порядку (Comparable)
Collections.sort(students);

// Сортировка по внешнему порядку (Comparator)
students.sort(Comparator.comparing(Student::getName));

// Сортировка по нескольким полям
students.sort(Comparator
    .comparing(Student::getGroup)
    .thenComparing(Comparator.comparing(Student::getAverageGrade).reversed()));
```

### 2.8. Алгоритмы класса `Collections`

Класс `java.util.Collections` предоставляет набор статических методов для работы с коллекциями:

| Метод | Назначение |
|-------|------------|
| `sort(List<T>)` | Сортировка списка по естественному порядку |
| `sort(List<T>, Comparator)` | Сортировка по внешнему порядку |
| `shuffle(List<?>)` | Случайное перемешивание |
| `reverse(List<?>)` | Обращение порядка элементов |
| `rotate(List<?>, int)` | Циклический сдвиг |
| `binarySearch(List<? extends Comparable>, Object)` | Бинарный поиск в отсортированном списке |
| `min(Collection)`, `max(Collection)` | Минимальный/максимальный элемент |
| `frequency(Collection, Object)` | Частота элемента в коллекции |
| `disjoint(Collection, Collection)` | Проверка непересечения |
| `unmodifiableList(List)` | Неизменяемое представление |
| `synchronizedList(List)` | Синхронизированное представление |

### 2.9. Неизменяемые коллекции

Начиная с Java 9, для создания неизменяемых коллекций применяются фабричные методы:

```java
// Неизменяемые коллекции (Java 9+)
List<String> names = List.of("Анна", "Борис", "Виктор");
Set<Integer> numbers = Set.of(1, 2, 3, 4, 5);
Map<String, Integer> ages = Map.of("Анна", 20, "Борис", 22);

// Попытка модификации выбросит UnsupportedOperationException
// names.add("Глеб");  // Ошибка во время выполнения
```

До Java 9 применялся класс `Collections`:

```java
List<String> mutable = new ArrayList<>(List.of("Анна", "Борис"));
List<String> immutable = Collections.unmodifiableList(mutable);
// immutable.add("Виктор");  // UnsupportedOperationException
```

### 2.10. Итераторы и fail-fast поведение

**Итератор** — объект, предоставляющий последовательный доступ к элементам коллекции:

```java
List<String> list = new ArrayList<>(List.of("Анна", "Борис", "Виктор"));

// Использование итератора
Iterator<String> it = list.iterator();
while (it.hasNext()) {
    String name = it.next();
    if (name.equals("Борис")) {
        it.remove();   // безопасное удаление через итератор
    }
}
```

**Fail-fast поведение.** Большинство коллекций Java являются fail-fast: если коллекция модифицируется извне во время итерации (не через метод `remove()` итератора), итератор выбрасывает `ConcurrentModificationException`:

```java
List<String> list = new ArrayList<>(List.of("Анна", "Борис", "Виктор"));
for (String name : list) {
    if (name.equals("Борис")) {
        // list.remove(name);  // ОШИБКА: ConcurrentModificationException
        // Правильно: использовать итератор или removeIf
    }
}

// Безопасное удаление
list.removeIf(name -> name.equals("Борис"));
```

## 3. Примеры выполнения

### 3.1. Пример 1. Подсчёт частоты слов в тексте

```java
import java.util.*;

/**
 * Пример демонстрирует применение Map для подсчёта частоты слов
 * и сортировку результатов по различным критериям.
 */
public class WordFrequency {
    /**
     * Подсчитывает частоту каждого слова в тексте.
     * Слова приводятся к нижнему регистру, punctuation удаляется.
     */
    public static Map<String, Integer> countWords(String text) {
        if (text == null || text.isEmpty()) {
            return Collections.emptyMap();
        }

        // Разбиение текста на слова
        String[] words = text.toLowerCase()
                             .replaceAll("[^а-яёa-z0-9\\s]", " ")
                             .split("\\s+");

        // Подсчёт частоты с использованием Map.merge
        Map<String, Integer> frequency = new HashMap<>();
        for (String word : words) {
            if (!word.isEmpty()) {
                frequency.merge(word, 1, Integer::sum);
            }
        }
        return frequency;
    }

    /**
     * Возвращает список слов, отсортированный по убыванию частоты.
     */
    public static List<Map.Entry<String, Integer>> getTopWords(
            Map<String, Integer> frequency, int topN) {
        List<Map.Entry<String, Integer>> entries = new ArrayList<>(frequency.entrySet());

        // Сортировка: сначала по частоте (убывание), затем по алфавиту
        entries.sort(Map.Entry.<String, Integer>comparingByValue().reversed()
                             .thenComparing(Map.Entry.comparingByKey()));

        // Возвращаем только top N элементов
        return entries.subList(0, Math.min(topN, entries.size()));
    }

    public static void main(String[] args) {
        String text = "Мама мыла раму. Мама мыла окно. " +
                      "Окно было чистым. Рама была чистой. " +
                      "Мама была довольна.";

        Map<String, Integer> frequency = countWords(text);

        System.out.println("=== Частота слов ===");
        frequency.forEach((word, count) ->
            System.out.printf("%-15s : %d%n", word, count));

        System.out.println("\n=== Топ-5 наиболее частых слов ===");
        List<Map.Entry<String, Integer>> topWords = getTopWords(frequency, 5);
        for (int i = 0; i < topWords.size(); i++) {
            Map.Entry<String, Integer> entry = topWords.get(i);
            System.out.printf("%d. %-15s : %d%n",
                              i + 1, entry.getKey(), entry.getValue());
        }

        System.out.println("\n=== Уникальных слов: " + frequency.size());
    }
}
```

### 3.2. Пример 2. Система группировки студентов по среднему баллу

```java
import java.util.*;
import java.util.stream.Collectors;

/**
 * Класс, моделирующий студента
 */
public class Student implements Comparable<Student> {
    private final String name;
    private final String group;
    private final double averageGrade;

    public Student(String name, String group, double averageGrade) {
        this.name = Objects.requireNonNull(name);
        this.group = Objects.requireNonNull(group);
        if (averageGrade < 0.0 || averageGrade > 5.0) {
            throw new IllegalArgumentException(
                "Средний балл должен быть в диапазоне [0.0, 5.0]");
        }
        this.averageGrade = averageGrade;
    }

    public String getName() { return name; }
    public String getGroup() { return group; }
    public double getAverageGrade() { return averageGrade; }

    @Override
    public int compareTo(Student other) {
        return Double.compare(other.averageGrade, this.averageGrade);
    }

    @Override
    public String toString() {
        return String.format("%s (%s, %.2f)", name, group, averageGrade);
    }
}

/**
 * Демонстрация применения различных коллекций для работы со студентами
 */
public class StudentGrouping {
    public static void main(String[] args) {
        List<Student> students = List.of(
            new Student("Иванов И.", "ПИ-21", 4.5),
            new Student("Петров П.", "ПИ-21", 3.8),
            new Student("Сидоров С.", "ПИ-22", 4.9),
            new Student("Кузнецов К.", "ПИ-22", 4.2),
            new Student("Смирнов С.", "ПИ-21", 4.7),
            new Student("Попов П.", "ПИ-23", 3.5),
            new Student("Волков В.", "ПИ-23", 4.0),
            new Student("Зайцев З.", "ПИ-22", 4.6)
        );

        // 1. Группировка по группе (Map<String, List<Student>>)
        System.out.println("=== Группировка по учебной группе ===");
        Map<String, List<Student>> byGroup = new TreeMap<>();
        for (Student s : students) {
            byGroup.computeIfAbsent(s.getGroup(), k -> new ArrayList<>()).add(s);
        }
        byGroup.forEach((group, groupStudents) -> {
            System.out.println("Группа " + group + ":");
            for (Student s : groupStudents) {
                System.out.println("  " + s);
            }
        });

        // 2. Топ-3 студента по среднему баллу
        System.out.println("\n=== Топ-3 студента по среднему баллу ===");
        List<Student> sorted = new ArrayList<>(students);
        Collections.sort(sorted);   // естественный порядок — по убыванию балла
        for (int i = 0; i < Math.min(3, sorted.size()); i++) {
            System.out.printf("%d. %s%n", i + 1, sorted.get(i));
        }

        // 3. Средний балл по каждой группе
        System.out.println("\n=== Средний балл по группам ===");
        Map<String, Double> averageByGroup = new LinkedHashMap<>();
        for (Map.Entry<String, List<Student>> entry : byGroup.entrySet()) {
            double sum = 0;
            for (Student s : entry.getValue()) {
                sum += s.getAverageGrade();
            }
            averageByGroup.put(entry.getKey(), sum / entry.getValue().size());
        }
        averageByGroup.forEach((group, avg) ->
            System.out.printf("Группа %s: %.2f%n", group, avg));

        // 4. Студенты-стипендиаты (балл >= 4.5)
        System.out.println("\n=== Студенты-стипендиаты (балл >= 4.5) ===");
        Set<String> scholarshipStudents = new TreeSet<>();
        for (Student s : students) {
            if (s.getAverageGrade() >= 4.5) {
                scholarshipStudents.add(s.getName());
            }
        }
        scholarshipStudents.forEach(name -> System.out.println("  " + name));

        // 5. Подсчёт студентов по группам
        System.out.println("\n=== Количество студентов в группах ===");
        Map<String, Integer> countByGroup = new HashMap<>();
        for (Student s : students) {
            countByGroup.merge(s.getGroup(), 1, Integer::sum);
        }
        countByGroup.forEach((group, count) ->
            System.out.printf("Группа %s: %d студентов%n", group, count));
    }
}
```

### 3.3. Пример 3. Очередь задач с приоритетами и отложенным выполнением

```java
import java.util.*;

/**
 * Класс, моделирующий задачу с приоритетом
 */
public class Task implements Comparable<Task> {
    private final String name;
    private final int priority;     // чем больше, тем важнее
    private final long createdAt;

    public Task(String name, int priority) {
        this.name = Objects.requireNonNull(name);
        if (priority < 1 || priority > 10) {
            throw new IllegalArgumentException(
                "Приоритет должен быть в диапазоне [1, 10]");
        }
        this.priority = priority;
        this.createdAt = System.currentTimeMillis();
    }

    public String getName() { return name; }
    public int getPriority() { return priority; }
    public long getCreatedAt() { return createdAt; }

    /**
     * Естественный порядок: по убыванию приоритета,
     * при равенстве — по времени создания (старые раньше).
     */
    @Override
    public int compareTo(Task other) {
        int cmp = Integer.compare(other.priority, this.priority);
        if (cmp != 0) return cmp;
        return Long.compare(this.createdAt, other.createdAt);
    }

    @Override
    public String toString() {
        return String.format("Task[%s, priority=%d]", name, priority);
    }
}

/**
 * Планировщик задач, использующий PriorityQueue
 */
public class TaskScheduler {
    private final Queue<Task> queue = new PriorityQueue<>();
    private final List<Task> completed = new ArrayList<>();

    /**
     * Добавляет задачу в очередь
     */
    public void schedule(Task task) {
        queue.add(task);
        System.out.println("Запланирована: " + task);
    }

    /**
     * Добавляет несколько задач
     */
    public void scheduleAll(Collection<? extends Task> tasks) {
        for (Task task : tasks) {
            queue.add(task);
        }
    }

    /**
     * Выполняет следующую задачу с наивысшим приоритетом
     */
    public Task executeNext() {
        if (queue.isEmpty()) {
            throw new NoSuchElementException("Очередь задач пуста");
        }
        Task task = queue.poll();
        completed.add(task);
        System.out.println("Выполнена: " + task);
        return task;
    }

    /**
     * Просмотр следующей задачи без извлечения
     */
    public Task peekNext() {
        if (queue.isEmpty()) {
            return null;
        }
        return queue.peek();
    }

    /**
     * Возвращает список выполненных задач
     */
    public List<Task> getCompletedTasks() {
        return Collections.unmodifiableList(completed);
    }

    /**
     * Возвращает количество задач в очереди
     */
    public int pendingCount() {
        return queue.size();
    }

    public static void main(String[] args) {
        TaskScheduler scheduler = new TaskScheduler();

        // Планирование задач в произвольном порядке
        scheduler.schedule(new Task("Отправить отчёт", 5));
        scheduler.schedule(new Task("Исправить критический баг", 10));
        scheduler.schedule(new Task("Обновить документацию", 2));
        scheduler.schedule(new Task("Провести код-ревью", 7));
        scheduler.schedule(new Task("Созвон с командой", 8));
        scheduler.schedule(new Task("Написать тесты", 6));

        System.out.println("\n=== Задач в очереди: " + scheduler.pendingCount());
        System.out.println("Следующая к выполнению: " + scheduler.peekNext());

        System.out.println("\n=== Выполнение задач ===");
        while (scheduler.pendingCount() > 0) {
            scheduler.executeNext();
        }

        System.out.println("\n=== История выполненных задач ===");
        for (Task task : scheduler.getCompletedTasks()) {
            System.out.println("  " + task);
        }
    }
}
```

## 4. Задания на паре

### Задание 4.1. Анализ текста с использованием коллекций

Разработайте программу, выполняющую комплексный анализ текста с применением различных коллекций.

**Требования к реализации:**

1. Реализуйте метод `analyzeText(String text)`, возвращающий объект `TextAnalysisResult`, содержащий:
   - `wordCount` (int) — общее количество слов;
   - `uniqueWords` (Set<String>) — множество уникальных слов (в нижнем регистре);
   - `wordFrequency` (Map<String, Integer>) — частота каждого слова;
   - `topWords` (List<Map.Entry<String, Integer>>) — топ-N наиболее частых слов;
   - `averageWordLength` (double) — средняя длина слова;
   - `longestWords` (Set<String>) — множество самых длинных слов.

2. Слова определяются как последовательности букв и цифр, разделённые пробелами и знаками препинания.

3. Топ-N определяется с использованием сортировки записей карты по убыванию частоты.

4. Реализуйте метод `findCommonWords(Map<String, Integer> freq1, Map<String, Integer> freq2)`, возвращающий множество слов, присутствующих в обоих текстах (пересечение ключей).

**Демонстрация:**
- Проанализируйте два различных текста.
- Выведите результаты анализа для каждого.
- Найдите общие слова в обоих текстах.
- Выведите статистику: количество слов, уникальных слов, средняя длина слова.

---

### Задание 4.2. Система управления инвентарём магазина

Разработайте класс `Inventory`, моделирующий инвентарь магазина, с применением различных коллекций.

**Класс `Product`:**
- Поля: `id` (String), `name` (String), `category` (String), `price` (double), `quantity` (int).
- Реализует `Comparable<Product>` по наименованию.

**Класс `Inventory`:**
- Поля:
  - `productsById` (Map<String, Product>) — быстрый доступ по ID;
  - `productsByCategory` (Map<String, Set<Product>>) — группировка по категориям;
  - `allProducts` (Set<Product>) — все товары для итерации.
- Методы:
  - `addProduct(Product p)` — добавить товар (с проверкой уникальности ID);
  - `removeProduct(String id)` — удалить товар по ID;
  - `getProductById(String id)` — получить товар по ID;
  - `getProductsByCategory(String category)` — получить товары категории;
  - `getCategories()` — получить множество всех категорий;
  - `getMostExpensive(int n)` — получить топ-N самых дорогих товаров;
  - `getTotalValue()` — вычислить суммарную стоимость инвентаря;
  - `updateQuantity(String id, int delta)` — изменить количество товара.

**Демонстрация:**
- Создайте инвентарь с не менее чем 10 товарами в 3-4 категориях.
- Выполните операции добавления, удаления, поиска.
- Выведите товары по категориям.
- Найдите топ-3 самых дорогих товара.
- Вычислите суммарную стоимость инвентаря.

---

### Задание 4.3. Система бронирования аудиторий

Разработайте систему бронирования аудиторий в учебном заведении.

**Класс `Room`:**
- Поля: `number` (String), `capacity` (int), `hasProjector` (boolean), `floor` (int).
- Реализует `Comparable<Room>` по номеру.

**Запись `Booking`:**
- Компоненты: `roomNumber` (String), `teacher` (String), `subject` (String), `date` (LocalDate), `timeSlot` (String, например "09:00-10:30"), `group` (String).

**Класс `BookingSystem`:**
- Поля:
  - `rooms` (Map<String, Room>) — все аудитории;
  - `bookings` (Map<LocalDate, List<Booking>>) — бронирования по датам;
  - `roomBookings` (Map<String, Set<Booking>>`) — бронирования по аудиториям.
- Методы:
  - `addRoom(Room room)` — добавить аудиторию;
  - `book(Booking booking)` — забронировать (с проверкой конфликтов);
  - `cancel(String roomNumber, LocalDate date, String timeSlot)` — отменить бронь;
  - `getAvailableRooms(LocalDate date, String timeSlot, int minCapacity)` — найти свободные аудитории;
  - `getBookingsForDate(LocalDate date)` — получить все бронирования на дату;
  - `getRoomUtilization(String roomNumber)` — вычислить загруженность аудитории.

**Демонстрация:**
- Создайте не менее 5 аудиторий.
- Выполните не менее 10 бронирований на разные даты и время.
- Продемонстрируйте обнаружение конфликта бронирований.
- Найдите свободные аудитории на заданное время с требуемой вместимостью.
- Выведите загруженность каждой аудитории.

## 5. Задание для самостоятельной работы

Разработать систему классов согласно своему варианту с активным применением различных коллекций. Требования:

1. Применение не менее трёх различных типов коллекций (List, Set, Map, Queue и т. п.).
2. Применение не менее одной реализации с упорядоченностью (TreeSet, TreeMap, LinkedHashMap).
3. Применение `Comparable` и/или `Comparator` для сортировки.
4. Применение алгоритмов класса `Collections` или методов коллекций (sort, reverse, frequency и т. п.).
5. Применение неизменяемых коллекций там, где это уместно.
6. Обработка исключительных ситуаций и граничных случаев.

### Варианты заданий

**Вариант 1.** Система «Библиотека». Хранение книг, читателей, выдач. Операции: выдача/возврат книги, поиск книг по автору/жанру, топ популярных книг, список должников.

**Вариант 2.** Система «Авиакомпания». Хранение рейсов, пассажиров, бронирований. Операции: регистрация на рейс, поиск свободных мест, отмена бронирования, статистика загруженности рейсов.

**Вариант 3.** Система «Банк». Хранение счетов, клиентов, транзакций. Операции: проведение транзакции, выписка по счёту, поиск клиентов по балансу, отчёт о движении средств за период.

**Вариант 4.** Система «Интернет-магазин». Хранение товаров, заказов, покупателей. Операции: добавление в корзину, оформление заказа, поиск товаров по категории, топ продаж.

**Вариант 5.** Система «Больница». Хранение пациентов, врачей, приёмов. Операции: запись на приём, отмена записи, расписание врача, список пациентов с диагнозом.

**Вариант 6.** Система «Университет». Хранение студентов, курсов, оценок. Операции: запись на курс, выставление оценки, расписание, рейтинг студентов.

**Вариант 7.** Система «Ресторан». Хранение меню, заказов, столиков. Операции: приём заказа, бронирование столика, формирование счёта, статистика популярных блюд.

**Вариант 8.** Система «Фитнес-клуб». Хранение клиентов, абонементов, посещений. Операции: регистрация посещения, продление абонемента, список клиентов с истекающим абонементом, статистика посещений.

**Вариант 9.** Система «Автосервис». Хранение заказов-нарядов, мастеров, услуг. Операции: создание заказ-наряда, назначение мастера, список ожидающих обслуживания, статистика по услугам.

**Вариант 10.** Система «Турагентство». Хранение туров, клиентов, бронирований. Операции: бронирование тура, отмена, поиск туров по стране, топ направлений сезона.

**Вариант 11.** Система «Кинотеатр». Хранение фильмов, сеансов, билетов. Операции: покупка билета, возврат, расписание на день, статистика продаж по фильмам.

**Вариант 12.** Система «Склад». Хранение товаров, поставщиков, поступлений. Операции: приёмка товара, отгрузка, инвентаризация, товары с низким остатком.

**Вариант 13.** Система «Страховая компания». Хранение полисов, клиентов, страховых случаев. Операции: оформление полиса, регистрация страхового случая, расчёт выплаты, статистика по типам страхования.

**Вариант 14.** Система «Почта». Хранение отправлений, отделений, маршрутов. Операции: регистрация отправления, отслеживание, расчёт стоимости, статистика по направлениям.

**Вариант 15.** Система «Агентство недвижимости». Хранение объектов, клиентов, сделок. Операции: добавление объекта, поиск по параметрам, регистрация сделки, статистика по районам.

**Вариант 16.** Система «Кадровое агентство». Хранение кандидатов, вакансий, откликов. Операции: размещение вакансии, отклик кандидата, подбор кандидатов, статистика по вакансиям.

**Вариант 17.** Система «Благотворительный фонд». Хранение кампаний, пожертвований, благополучателей. Операции: регистрация пожертвования, распределение средств, отчёт по кампании, топ жертвователей.

**Вариант 18.** Система «Такси». Хранение водителей, заказов, автомобилей. Операции: создание заказа, назначение водителя, отмена, статистика по водителям.

**Вариант 19.** Система «Музей». Хранение экспонатов, выставок, посетителей. Операции: добавление экспоната, формирование выставки, продажа билетов, статистика посещений.

**Вариант 20.** Система «Спортивный клуб». Хранение спортсменов, соревнований, результатов. Операции: регистрация спортсмена, запись на соревнование, ввод результата, рейтинг спортсменов.

**Вариант 21.** Система «Почтовый клиент». Хранение писем, контактов, папок. Операции: отправка письма, перемещение в папку, поиск по контактам, статистика по папкам.

**Вариант 22.** Система «Календарь». Хранение событий, напоминаний, участников. Операции: создание события, добавление участника, поиск событий на дату, список предстоящих событий.

**Вариант 23.** Система «Платёжная система». Хранение merchants, платежей, возвратов. Операции: проведение платежа, возврат, статистика по merchants, отчёт за период.

**Вариант 24.** Система «Управление проектами». Хранение проектов, задач, исполнителей. Операции: создание задачи, назначение исполнителя, изменение статуса, отчёт по загрузке.

**Вариант 25.** Система «Ветеринарная клиника». Хранение животных, владельцев, приёмов. Операции: регистрация животного, запись на приём, ведение карты, статистика по видам животных.

**Вариант 26.** Система «Железнодорожные перевозки». Хранение поездов, вагонов, пассажиров. Операции: покупка билета, возврат, поиск поездов, статистика по направлениям.

**Вариант 27.** Система «Языковые курсы». Хранение студентов, групп, занятий. Операции: запись на курс, формирование группы, отметка посещаемости, статистика по уровням.

**Вариант 28.** Система «Фотостудия». Хранение фотосессий, клиентов, фотографий. Операции: бронирование фотосессии, загрузка фотографий, выдача клиенту, статистика по типам съёмок.

**Вариант 29.** Система «Коворкинг». Хранение рабочих мест, резидентов, бронирований. Операции: бронирование места, регистрация резидента, учёт посещений, статистика загруженности.

**Вариант 30.** Система «Платформа подкастов». Хранение подкастов, эпизодов, подписчиков. Операции: публикация эпизода, подписка, статистика прослушиваний, топ подкастов.

## 6. Методические указания к самостоятельной работе

1. **Выбор коллекций.** Для каждой сущности определите оптимальную структуру данных:
   - для быстрого доступа по ключу — `HashMap` или `LinkedHashMap`;
   - для упорядоченного доступа — `TreeMap` или `TreeSet`;
   - для хранения уникальных элементов — `HashSet` или `LinkedHashSet`;
   - для упорядоченного списка — `ArrayList`;
   - для очереди задач — `PriorityQueue` или `ArrayDeque`.

2. **Обоснование выбора.** В отчёте поясните, почему для каждой сущности выбрана именно эта структура данных (со ссылкой на сложность операций и требования задачи).

3. **Применение `Comparable` и `Comparator`.** Определите естественный порядок для основных сущностей (по идентификатору, имени, дате). Для альтернативных порядков применяйте `Comparator`.

4. **Неизменяемые коллекции.** Применяйте `List.of`, `Map.of`, `Collections.unmodifiable*` для:
   - возврата результатов поиска;
   - хранения константных наборов данных;
   - защиты внутреннего состояния объектов.

5. **Группировка данных.** Для группировки элементов по какому-либо признаку применяйте `Map<K, List<V>>` с методом `computeIfAbsent`:

   ```java
   map.computeIfAbsent(key, k -> new ArrayList<>()).add(value);
   ```

6. **Обработка исключений.** Реализуйте обработку:
   - дубликатов ключей при добавлении;
   - отсутствия элементов при поиске;
   - конфликтов бронирований и т. п.

7. **Тестирование.** Перед сдачей работы проверьте:
   - корректность работы всех операций;
   - сохранение инвариантов (уникальность, упорядоченность);
   - обработку граничных случаев (пустые коллекции, отсутствие элементов).

8. **Применение ИИ.** При использовании средств ИИ:
   - генерируйте по отдельности классы сущностей, коллекции и демонстрационный класс;
   - обязательно проверяйте выбор структур данных;
   - не делегируйте ИИ обоснование выбора коллекций.

9. **Оформление отчёта.** Отчёт должен содержать:
   - листинги всех файлов проекта с комментариями;
   - протокол работы демонстрационного класса;
   - обоснование выбора структур данных;
   - ответы на контрольные вопросы;
   - выводы по проделанной работе.

## 7. Контрольные вопросы

1. Какие интерфейсы составляют основу библиотеки коллекций Java?
2. В чём различие между `List`, `Set`, `Queue` и `Map`?
3. Чем `ArrayList` отличается от `LinkedList`? В каких случаях применяется каждая из этих реализаций?
4. Какие реализации `Set` существуют в Java? В чём их различие?
5. Чем `HashMap` отличается от `TreeMap` и `LinkedHashMap`?
6. Что такое `PriorityQueue`? Как определяется порядок элементов?
7. Для чего предназначен интерфейс `Comparable`? Как реализуется метод `compareTo`?
8. Для чего предназначен интерфейс `Comparator`? В каких случаях он применяется?
9. Какие алгоритмы предоставляет класс `Collections`?
10. Как создать неизменяемую коллекцию в Java?
11. Что такое fail-fast поведение итератора? Когда возникает `ConcurrentModificationException`?
12. Каким образом осуществляется группировка элементов по ключу?
13. Как выбрать подходящую реализацию коллекции для конкретной задачи?

## 8. Рекомендуемые источники

1. Хорстманн К. *Java. Библиотека профессионала. Том 2. Расширенные средства программирования.* — М.: Вильямс. — Глава 1 (Библиотека коллекций).
2. Блох Дж. *Java. Эффективное программирование.* — М.: Питер. — Правила 43–56 (коллекции и потоки).
3. Oracle Java Tutorials. Lesson: Collections. URL: https://docs.oracle.com/javase/tutorial/collections/
4. Baeldung. Java Collections Guide. URL: https://www.baeldung.com/java-collections
5. Schildt H. *Java: The Complete Reference.* — McGraw-Hill Education. — Chapter 19 (Collections Framework).
