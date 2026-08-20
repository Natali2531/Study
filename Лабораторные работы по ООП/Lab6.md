# Лабораторная работа №6. Обобщённое программирование (Generics)

## 1. Паспорт работы

| Параметр | Значение |
|----------|----------|
| Тема | Обобщённое программирование (Generics) |
| Номер занятия в модуле | 2 из 4 (модуль 2) |
| Продолжительность аудиторной части | 2 академических часа |
| Предшествующая подготовка | Модуль 1 (базовый синтаксис, инкапсуляция, наследование, полиморфизм), лабораторная работа №5 (перечисления, записи, вложенные типы) |
| Тип работы | Формирование навыков типобезопасного параметризованного программирования |

### 1.1. Цель работы

Освоить механизм обобщённого программирования (generics) в языке Java: научиться проектировать типобезопасные обобщённые классы и методы, применять ограничители типов и wildcards, понимать принцип PECS (Producer Extends, Consumer Super) и механизм стирания типов (type erasure). Сформировать навыки создания переиспользуемых компонентов, работающих с произвольными типами данных при сохранении строгой типизации.

### 1.2. Задачи работы

1. Изучить мотивацию введения механизма обобщений в язык Java.
2. Освоить синтаксис объявления и использования обобщённых классов.
3. Научиться проектировать обобщённые методы, в том числе статические.
4. Изучить ограничители типов (`bounded types`) — `extends` и `super`.
5. Освоить применение wildcards (`? extends`, `? super`, `?`) и принцип PECS.
6. Изучить механизм стирания типов (type erasure) и его последствия.
7. Понять ограничения механизма обобщений в языке Java.
8. Развить навыки создания переиспользуемых типобезопасных компонентов.

### 1.3. Оснащение

- JDK версии 17 или выше;
- интегрированная среда разработки IntelliJ IDEA Community Edition;
- система сборки Maven или Gradle;
- система контроля версий Git.

## 2. Теоретические сведения

### 2.1. Мотивация: проблема отсутствия типобезопасности до Java 5

До появления механизма обобщений в Java 5 для создания универсальных контейнеров применялся тип `Object`, поскольку любой ссылочный тип является наследником `Object`. Такой подход приводил к серьёзным проблемам:

```java
// Код до Java 5 — без обобщений
List list = new ArrayList();
list.add("Привет");
list.add(Integer.valueOf(42));   // Компилятор не возражает

// Ошибка обнаруживается только во время выполнения
String s = (String) list.get(1); // ClassCastException!
```

**Недостатки подхода без обобщений:**

1. **Отсутствие типобезопасности.** Компилятор не может проверить корректность типов, ошибки проявляются во время выполнения.
2. **Необходимость явных приведений.** Каждое извлечение элемента требует приведения типа.
3. **Хрупкость кода.** Изменение типа данных в одном месте может привести к ошибкам в совершенно другом участке программы.
4. **Снижение выразительности.** По сигнатуре метода невозможно понять, с какими типами он работает.

Механизм обобщений устраняет указанные недостатки, перенося проверку типов на этап компиляции.

### 2.2. Обобщённые классы

**Обобщённый класс** — класс, объявленный с одним или несколькими параметрами типа, которые могут быть использованы в объявлении полей, методов и внутренних типов.

Синтаксис объявления:

```java
public class ClassName<T> {
    // T — формальный параметр типа
}
```

Пример обобщённого класса — контейнер `Box`:

```java
/**
 * Обобщённый класс, моделирующий контейнер для одного объекта.
 * @param <T> тип объекта, хранящегося в контейнере
 */
public class Box<T> {
    private T content;

    public Box() {
        this.content = null;
    }

    public Box(T content) {
        this.content = content;
    }

    public T getContent() {
        return content;
    }

    public void setContent(T content) {
        this.content = content;
    }

    public boolean isEmpty() {
        return content == null;
    }

    @Override
    public String toString() {
        return "Box[" + (content != null ? content : "пусто") + "]";
    }
}
```

Использование обобщённого класса:

```java
// Создание экземпляров с различными типами
Box<String> stringBox = new Box<>("Привет, мир!");
Box<Integer> intBox = new Box<>(42);
Box<List<String>> listBox = new Box<>(new ArrayList<>());

// Извлечение значения не требует приведения типа
String s = stringBox.getContent();   // тип String гарантирован компилятором
Integer n = intBox.getContent();     // тип Integer гарантирован компилятором

// Попытка нарушения типобезопасности пресекается на этапе компиляции
// intBox.setContent("строка");  // Ошибка компиляции!
```

**Соглашения об именовании параметров типа:**

| Параметр | Расшифровка | Назначение |
|----------|-------------|------------|
| `T` | Type | Общий тип |
| `E` | Element | Тип элемента (обычно в коллекциях) |
| `K` | Key | Тип ключа |
| `V` | Value | Тип значения |
| `N` | Number | Числовой тип |
| `R` | Result | Тип возвращаемого значения |

### 2.3. Обобщённые классы с несколькими параметрами типа

Класс может иметь несколько параметров типа, разделённых запятыми:

```java
/**
 * Обобщённый класс, моделирующий пару объектов.
 * @param <K> тип первого элемента
 * @param <V> тип второго элемента
 */
public class Pair<K, V> {
    private final K first;
    private final V second;

    public Pair(K first, V second) {
        this.first = first;
        this.second = second;
    }

    public K getFirst() { return first; }
    public V getSecond() { return second; }

    /**
     * Обобщённый статический метод-фабрика.
     * Типы K и V выводятся компилятором из аргументов.
     */
    public static <K, V> Pair<K, V> of(K first, V second) {
        return new Pair<>(first, second);
    }

    /**
     * Метод, создающий новую пару с переставленными местами элементов.
     */
    public Pair<V, K> swap() {
        return new Pair<>(second, first);
    }

    @Override
    public String toString() {
        return "Pair[" + first + ", " + second + "]";
    }
}
```

Использование:

```java
Pair<String, Integer> nameToAge = Pair.of("Иванов И.И.", 20);
Pair<Integer, String> ageToName = nameToAge.swap();

System.out.println(nameToAge);   // Pair[Иванов И.И., 20]
System.out.println(ageToName);   // Pair[20, Иванов И.И.]
```

### 2.4. Обобщённые методы

**Обобщённый метод** — метод, объявленный с собственным параметром типа, независимым от параметров типа класса. Параметр типа указывается перед возвращаемым типом.

```java
/**
 * Утилитарный класс с обобщёнными методами
 */
public class ArrayUtils {

    /**
     * Обобщённый статический метод: поиск элемента в массиве.
     * Параметр типа T объявлен для данного метода.
     */
    public static <T> int indexOf(T[] array, T target) {
        if (array == null) {
            throw new IllegalArgumentException("Массив не может быть null");
        }
        for (int i = 0; i < array.length; i++) {
            if (java.util.Objects.equals(array[i], target)) {
                return i;
            }
        }
        return -1;
    }

    /**
     * Обобщённый метод: преобразование массива в список.
     */
    public static <T> java.util.List<T> toList(T[] array) {
        java.util.List<T> list = new java.util.ArrayList<>(array.length);
        for (T element : array) {
            list.add(element);
        }
        return list;
    }

    /**
     * Обобщённый метод: обмен элементов в списке.
     */
    public static <T> void swap(java.util.List<T> list, int i, int j) {
        if (i < 0 || j < 0 || i >= list.size() || j >= list.size()) {
            throw new IndexOutOfBoundsException("Некорректные индексы");
        }
        T temp = list.get(i);
        list.set(i, list.get(j));
        list.set(j, temp);
    }
}
```

Использование:

```java
String[] names = {"Анна", "Борис", "Виктор"};
int index = ArrayUtils.indexOf(names, "Борис");   // 1
java.util.List<String> nameList = ArrayUtils.toList(names);
ArrayUtils.swap(nameList, 0, 2);
```

### 2.5. Ограничители типов (Bounded Types)

**Ограничители типов** позволяют наложить ограничения на параметры типа, указав, что тип должен быть наследником определённого класса или реализовывать определённые интерфейсы.

#### 2.5.1. Верхняя граница (`extends`)

Синтаксис: `<T extends SuperType>` — тип `T` должен быть `SuperType` или его подклассом.

```java
/**
 * Обобщённый класс с ограничителем типа: T должен быть наследником Number.
 */
public class NumberBox<T extends Number> {
    private T value;

    public NumberBox(T value) {
        this.value = value;
    }

    public T getValue() { return value; }

    /**
     * Благодаря ограничителю можно вызывать методы класса Number
     */
    public double doubleValue() {
        return value.doubleValue();
    }

    public int intValue() {
        return value.intValue();
    }

    /**
     * Метод, сравнивающий значение с другим числом
     */
    public boolean isGreaterThan(T other) {
        return value.doubleValue() > other.doubleValue();
    }
}
```

Использование:

```java
NumberBox<Integer> intBox = new NumberBox<>(42);
NumberBox<Double> doubleBox = new NumberBox<>(3.14);
// NumberBox<String> strBox = new NumberBox<>("text");  // Ошибка компиляции!

System.out.println(intBox.doubleValue());       // 42.0
System.out.println(intBox.isGreaterThan(10));   // true
```

#### 2.5.2. Несколько ограничителей

Тип может быть ограничен одним классом и несколькими интерфейсами:

```java
/**
 * Тип T должен быть наследником Number и реализовывать Comparable.
 * Первый ограничитель — класс, последующие — интерфейсы.
 */
public class SortableNumberBox<T extends Number & Comparable<T>> {
    private T value;

    public SortableNumberBox(T value) {
        this.value = value;
    }

    public T getValue() { return value; }

    /**
     * Сравнение значений (доступно благодаря Comparable)
     */
    public int compareTo(SortableNumberBox<T> other) {
        return this.value.compareTo(other.value);
    }

    /**
     * Числовое значение (доступно благодаря Number)
     */
    public double doubleValue() {
        return value.doubleValue();
    }
}
```

#### 2.5.3. Обобщённые методы с ограничителями

Ограничители могут применяться и к параметрам типов методов:

```java
public class NumberUtils {

    /**
     * Метод принимает список чисел (или наследников Number)
     * и возвращает их сумму.
     */
    public static double sum(java.util.List<? extends Number> numbers) {
        double result = 0.0;
        for (Number n : numbers) {
            result += n.doubleValue();
        }
        return result;
    }

    /**
     * Метод находит максимальное значение в списке сравнимых элементов.
     */
    public static <T extends Comparable<T>> T max(java.util.List<T> list) {
        if (list == null || list.isEmpty()) {
            throw new IllegalArgumentException("Список не может быть пустым");
        }
        T max = list.get(0);
        for (T element : list) {
            if (element.compareTo(max) > 0) {
                max = element;
            }
        }
        return max;
    }
}
```

### 2.6. Wildcards (неизвестные типы)

**Wildcard** (`?`) — специальный синтаксис, обозначающий «неизвестный тип». Применяется в ситуациях, когда конкретный тип не важен или должен быть определён динамически.

#### 2.6.1. Неограниченный wildcard (`?`)

Обозначает любой тип. Применяется, когда методы работают с объектами независимо от их типа:

```java
/**
 * Метод выводит элементы коллекции произвольного типа
 */
public static void printCollection(java.util.Collection<?> collection) {
    for (Object element : collection) {
        System.out.println(element);
    }
}

// Работает с коллекциями любых типов
printCollection(java.util.List.of(1, 2, 3));
printCollection(java.util.List.of("А", "Б", "В"));
```

#### 2.6.2. Ограниченный wildcard сверху (`? extends T`)

Обозначает «неизвестный тип, являющийся `T` или его подклассом». Применяется, когда данные **читаются** из структуры (producer):

```java
/**
 * Метод суммирует числа из коллекции, тип элементов —
 * Number или его подкласс.
 */
public static double sum(java.util.List<? extends Number> list) {
    double result = 0.0;
    for (Number n : list) {
        result += n.doubleValue();   // чтение допустимо
    }
    // list.add(42);  // ОШИБКА компиляции: запись запрещена!
    return result;
}

// Работает с List<Integer>, List<Double>, List<Number> и т. д.
System.out.println(sum(java.util.List.of(1, 2, 3)));           // 6.0
System.out.println(sum(java.util.List.of(1.5, 2.5, 3.0)));     // 7.0
```

#### 2.6.3. Ограниченный wildcard снизу (`? super T`)

Обозначает «неизвестный тип, являющийся `T` или его суперклассом». Применяется, когда данные **записываются** в структуру (consumer):

```java
/**
 * Метод добавляет целые числа в коллекцию, способную принимать Integer
 * (List<Integer>, List<Number>, List<Object>).
 */
public static void addIntegers(java.util.List<? super Integer> list) {
    list.add(1);   // запись допустима
    list.add(2);
    list.add(3);
    // Integer x = list.get(0);  // ОШИБКА: чтение возвращает Object
}

java.util.List<Integer> intList = new java.util.ArrayList<>();
java.util.List<Number> numList = new java.util.ArrayList<>();
java.util.List<Object> objList = new java.util.ArrayList<>();

addIntegers(intList);   // OK
addIntegers(numList);   // OK
addIntegers(objList);   // OK
```

### 2.7. Принцип PECS (Producer Extends, Consumer Super)

**PECS** — мнемоническое правило, сформулированное Джошуа Блохом в книге «Java. Эффективное программирование»:

- **Producer Extends** — если структура данных **производит** значения (из неё читают), используйте `? extends T`;
- **Consumer Super** — если структура данных **потребляет** значения (в неё записывают), используйте `? super T`.

Рассмотрим метод, который одновременно читает из одной коллекции и записывает в другую:

```java
/**
 * Метод копирует элементы из src в dest.
 * src — producer (читаем), поэтому ? extends T.
 * dest — consumer (записываем), поэтому ? super T.
 */
public static <T> void copy(java.util.List<? extends T> src,
                             java.util.List<? super T> dest) {
    for (T element : src) {
        dest.add(element);
    }
}

// Использование
java.util.List<Integer> source = java.util.List.of(1, 2, 3);
java.util.List<Number> destination = new java.util.ArrayList<>();
copy(source, destination);   // OK: Integer extends Number
```

### 2.8. Механизм стирания типов (Type Erasure)

При компиляции обобщённого кода компилятор Java выполняет **стирание типов** — заменяет все параметры типа их ограничителями (или `Object`, если ограничитель не указан). Информация о конкретных типах не сохраняется в байт-коде.

Последствия стирания типов:

```java
// Исходный код
Box<String> stringBox = new Box<>("Привет");
Box<Integer> intBox = new Box<>(42);

// После стирания типов оба класса становятся Box<Object>
// stringBox.getClass() == intBox.getClass() → true!
```

**Ограничения, вытекающие из стирания типов:**

1. **Нельзя создать экземпляр параметра типа:**
   ```java
   public <T> void method() {
       // T obj = new T();     // ОШИБКА компиляции
   }
   ```

2. **Нельзя создать массив параметризованного типа:**
   ```java
   // T[] array = new T[10];  // ОШИБКА компиляции
   ```

3. **Нельзя использовать примитивные типы в качестве параметров:**
   ```java
   // Box<int> intBox = new Box<>();  // ОШИБКА: только ссылочные типы
   Box<Integer> intBox = new Box<>(); // OK: используйте обёртки
   ```

4. **Нельзя выполнить instanceof для параметризованного типа:**
   ```java
   // if (obj instanceof Box<String>) { }  // ОШИБКА компиляции
   if (obj instanceof Box) { }             // OK: без параметра
   ```

5. **Нельзя объявлять статические поля параметризованного типа:**
   ```java
   public class Box<T> {
       // private static T field;  // ОШИБКА
   }
   ```

### 2.9. Reifiable types и suppression

Некоторые обобщённые типы сохраняют информацию о типах во время выполнения (**reifiable types**):
- неограниченные wildcards (`List<?>`, `Map<?, ?>`);
- необобщённые типы (`List`, `Map`);
- «сырые» типы (raw types).

Для обхода ограничений стирания типов в некоторых случаях применяется аннотация `@SuppressWarnings("unchecked")`:

```java
@SuppressWarnings("unchecked")
public <T> T[] toArray(java.util.List<T> list, Class<T> type) {
    T[] array = (T[]) java.lang.reflect.Array.newInstance(type, list.size());
    for (int i = 0; i < list.size(); i++) {
        array[i] = list.get(i);
    }
    return array;
}
```

## 3. Примеры выполнения

### 3.1. Пример 1. Обобщённый связный список

```java
/**
 * Обобщённый связный список.
 * Демонстрирует применение параметра типа E для элемента.
 *
 * @param <E> тип элементов списка
 */
public class GenericLinkedList<E> {
    /**
     * Внутренний класс узла списка
     */
    private class Node {
        E data;
        Node next;

        Node(E data) {
            this.data = data;
            this.next = null;
        }
    }

    private Node head;
    private int size;

    public GenericLinkedList() {
        this.head = null;
        this.size = 0;
    }

    /**
     * Добавление элемента в конец списка
     */
    public void add(E element) {
        if (element == null) {
            throw new IllegalArgumentException("Элемент не может быть null");
        }
        Node newNode = new Node(element);
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

    /**
     * Получение элемента по индексу
     */
    public E get(int index) {
        checkIndex(index);
        Node current = head;
        for (int i = 0; i < index; i++) {
            current = current.next;
        }
        return current.data;
    }

    /**
     * Удаление элемента по индексу
     */
    public E remove(int index) {
        checkIndex(index);
        E removed;
        if (index == 0) {
            removed = head.data;
            head = head.next;
        } else {
            Node current = head;
            for (int i = 0; i < index - 1; i++) {
                current = current.next;
            }
            removed = current.next.data;
            current.next = current.next.next;
        }
        size--;
        return removed;
    }

    /**
     * Поиск элемента в списке
     */
    public boolean contains(E element) {
        Node current = head;
        while (current != null) {
            if (java.util.Objects.equals(current.data, element)) {
                return true;
            }
            current = current.next;
        }
        return false;
    }

    public int size() { return size; }
    public boolean isEmpty() { return size == 0; }

    private void checkIndex(int index) {
        if (index < 0 || index >= size) {
            throw new IndexOutOfBoundsException(
                "Индекс " + index + " вне диапазона [0, " + size + ")");
        }
    }

    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder("[");
        Node current = head;
        while (current != null) {
            sb.append(current.data);
            if (current.next != null) sb.append(", ");
            current = current.next;
        }
        sb.append("]");
        return sb.toString();
    }
}
```

Демонстрационный класс:

```java
public class LinkedListDemo {
    public static void main(String[] args) {
        // Список строк
        GenericLinkedList<String> names = new GenericLinkedList<>();
        names.add("Анна");
        names.add("Борис");
        names.add("Виктор");
        System.out.println("Имена: " + names);
        System.out.println("Содержит 'Борис': " + names.contains("Борис"));
        System.out.println("Элемент по индексу 1: " + names.get(1));

        // Список целых чисел
        GenericLinkedList<Integer> numbers = new GenericLinkedList<>();
        for (int i = 1; i <= 5; i++) {
            numbers.add(i * 10);
        }
        System.out.println("\nЧисла: " + numbers);
        System.out.println("Удалён элемент: " + numbers.remove(2));
        System.out.println("После удаления: " + numbers);

        // Типобезопасность: компилятор не позволит добавить String в список Integer
        // numbers.add("строка");  // Ошибка компиляции
    }
}
```

### 3.2. Пример 2. Обобщённые методы с ограничителями

```java
import java.util.List;
import java.util.ArrayList;

/**
 * Утилитарный класс с обобщёнными методами,
 * использующими ограничители типов.
 */
public class GenericAlgorithms {

    /**
     * Находит максимальный элемент в списке.
     * Тип T должен реализовывать Comparable<T>.
     */
    public static <T extends Comparable<T>> T max(List<T> list) {
        if (list == null || list.isEmpty()) {
            throw new IllegalArgumentException("Список не может быть пустым");
        }
        T max = list.get(0);
        for (T element : list) {
            if (element.compareTo(max) > 0) {
                max = element;
            }
        }
        return max;
    }

    /**
     * Находит минимальный элемент в списке.
     */
    public static <T extends Comparable<T>> T min(List<T> list) {
        if (list == null || list.isEmpty()) {
            throw new IllegalArgumentException("Список не может быть пустым");
        }
        T min = list.get(0);
        for (T element : list) {
            if (element.compareTo(min) < 0) {
                min = element;
            }
        }
        return min;
    }

    /**
     * Суммирует числа из списка.
     * Принимает список любых наследников Number.
     */
    public static double sum(List<? extends Number> list) {
        double result = 0.0;
        for (Number n : list) {
            result += n.doubleValue();
        }
        return result;
    }

    /**
     * Подсчитывает количество элементов, удовлетворяющих предикату.
     */
    public static <T> int countIf(List<T> list, java.util.function.Predicate<T> predicate) {
        int count = 0;
        for (T element : list) {
            if (predicate.test(element)) {
                count++;
            }
        }
        return count;
    }

    /**
     * Фильтрует список, возвращая новые список с элементами,
     * удовлетворяющими предикату.
     */
    public static <T> List<T> filter(List<T> list, java.util.function.Predicate<T> predicate) {
        List<T> result = new ArrayList<>();
        for (T element : list) {
            if (predicate.test(element)) {
                result.add(element);
            }
        }
        return result;
    }

    /**
     * Преобразует список элементов одного типа в список другого типа.
     */
    public static <T, R> List<R> map(List<T> list, java.util.function.Function<T, R> mapper) {
        List<R> result = new ArrayList<>(list.size());
        for (T element : list) {
            result.add(mapper.apply(element));
        }
        return result;
    }
}
```

Демонстрационный класс:

```java
public class AlgorithmsDemo {
    public static void main(String[] args) {
        // Работа со строками
        List<String> names = List.of("Анна", "Борис", "Виктор", "Антон");
        System.out.println("Максимальная строка: " + GenericAlgorithms.max(names));
        System.out.println("Минимальная строка: " + GenericAlgorithms.min(names));

        // Работа с числами
        List<Integer> numbers = List.of(5, 2, 8, 1, 9, 3);
        List<Double> doubles = List.of(1.5, 2.7, 3.1, 0.5);

        System.out.println("\nМаксимальное целое: " + GenericAlgorithms.max(numbers));
        System.out.println("Сумма целых: " + GenericAlgorithms.sum(numbers));
        System.out.println("Сумма вещественных: " + GenericAlgorithms.sum(doubles));

        // Фильтрация
        List<Integer> evenNumbers = GenericAlgorithms.filter(numbers, n -> n % 2 == 0);
        System.out.println("\nЧётные числа: " + evenNumbers);

        // Подсчёт по условию
        int countGreaterThan5 = GenericAlgorithms.countIf(numbers, n -> n > 5);
        System.out.println("Количество чисел > 5: " + countGreaterThan5);

        // Преобразование
        List<String> asStrings = GenericAlgorithms.map(numbers, n -> "Число " + n);
        System.out.println("Преобразованный список: " + asStrings);
    }
}
```

### 3.3. Пример 3. Принцип PECS на примере обобщённого контейнера

```java
import java.util.List;
import java.util.ArrayList;
import java.util.Collection;

/**
 * Обобщённый контейнер, демонстрирующий принцип PECS.
 *
 * @param <T> тип элементов
 */
public class GenericContainer<T> {
    private final List<T> items;

    public GenericContainer() {
        this.items = new ArrayList<>();
    }

    public GenericContainer(Collection<? extends T> source) {
        this.items = new ArrayList<>(source);
    }

    /**
     * Добавление одного элемента (consumer — принимаем T или подтип)
     */
    public void add(T item) {
        items.add(item);
    }

    /**
     * Добавление всех элементов из коллекции.
     * Источник — producer, поэтому ? extends T.
     */
    public void addAll(Collection<? extends T> source) {
        items.addAll(source);
    }

    /**
     * Получение элемента по индексу (producer — возвращаем T)
     */
    public T get(int index) {
        return items.get(index);
    }

    /**
     * Передача всех элементов в коллекцию-приёмник.
     * Приёмник — consumer, поэтому ? super T.
     */
    public void copyTo(Collection<? super T> destination) {
        destination.addAll(items);
    }

    /**
     * Применение операции ко всем элементам.
     * Consumer принимает T или супертип.
     */
    public void forEach(java.util.function.Consumer<? super T> action) {
        for (T item : items) {
            action.accept(item);
        }
    }

    /**
     * Преобразование элементов с помощью функции.
     * Источник — producer (? extends T), результат — новый контейнер.
     */
    public <R> GenericContainer<R> map(java.util.function.Function<? super T, ? extends R> mapper) {
        GenericContainer<R> result = new GenericContainer<>();
        for (T item : items) {
            result.add(mapper.apply(item));
        }
        return result;
    }

    public int size() { return items.size(); }
    public boolean isEmpty() { return items.isEmpty(); }

    @Override
    public String toString() {
        return "GenericContainer" + items;
    }
}
```

Демонстрационный класс:

```java
public class ContainerDemo {
    public static void main(String[] args) {
        // Создание контейнера целых чисел
        GenericContainer<Integer> intContainer = new GenericContainer<>();
        intContainer.add(1);
        intContainer.add(2);
        intContainer.add(3);
        System.out.println("Целочисленный контейнер: " + intContainer);

        // Копирование в контейнер чисел (Number — супертип Integer)
        GenericContainer<Number> numberContainer = new GenericContainer<>();
        intContainer.copyTo(numberContainer);   // PECS: consumer super T
        numberContainer.add(4.5);               // Можно добавить Double
        numberContainer.add(6L);                // Можно добавить Long
        System.out.println("Числовой контейнер: " + numberContainer);

        // Инициализация из коллекции (? extends T)
        List<Integer> source = List.of(10, 20, 30);
        GenericContainer<Integer> fromList = new GenericContainer<>(source);
        System.out.println("Из списка: " + fromList);

        // Преобразование: Integer -> String
        GenericContainer<String> stringContainer =
            intContainer.map(n -> "Число " + n);
        System.out.println("Преобразованный: " + stringContainer);

        // forEach с consumer, принимающим Number (супертип Integer)
        System.out.print("Печать через forEach: ");
        intContainer.forEach((Number n) -> System.out.print(n + " "));
        System.out.println();
    }
}
```

## 4. Задания на паре

### Задание 4.1. Обобщённый стек

Разработайте обобщённый класс `Stack<E>`, моделирующий структуру данных «стек» (LIFO — Last In, First Out).

**Требования к реализации:**

1. Класс должен быть параметризован типом `E` — типом хранимых элементов.
2. Внутреннее хранение — на основе массива (с автоматическим расширением при переполнении) или связного списка.
3. Методы:
   - `void push(E element)` — поместить элемент на вершину стека;
   - `E pop()` — извлечь элемент с вершины стека (бросить `EmptyStackException`, если стек пуст);
   - `E peek()` — просмотреть элемент на вершине без извлечения;
   - `boolean isEmpty()` — проверить пустоту стека;
   - `int size()` — количество элементов;
   - `void clear()` — очистить стек;
   - `boolean contains(E element)` — проверить наличие элемента.
4. Валидация: метод `push()` не должен принимать `null`.
5. Переопределите метод `toString()` для вывода содержимого стека.

**Демонстрация:**
- Создайте стек строк и стек целых чисел.
- Выполните не менее 5 операций `push` для каждого.
- Извлеките несколько элементов посредством `pop`.
- Продемонстрируйте работу `peek`, `contains`, `clear`.
- Продемонстрируйте обработку исключительных ситуаций (извлечение из пустого стека).

**Пример выполнения программы:**

```
=== Стек строк ===
Push: "Анна"
Push: "Борис"
Push: "Виктор"
Содержимое стека: [Анна, Борис, Виктор]
Peek: Виктор
Pop: Виктор
Pop: Борис
Содержимое стека: [Анна]

=== Стек чисел ===
Push: 10, 20, 30, 40, 50
Содержимое: [10, 20, 30, 40, 50]
Содержит 30: true
Содержит 100: false

=== Обработка исключений ===
Попытка извлечения из пустого стека: Стек пуст
```

---

### Задание 4.2. Обобщённые утилиты для работы с числовыми коллекциями

Разработайте утилитарный класс `NumberCollectionUtils` с обобщёнными статическими методами, работающими с коллекциями числовых типов.

**Требования к реализации:**

Все методы должны использовать wildcards `? extends Number` для обеспечения совместимости с `Integer`, `Double`, `Long`, `Float`, `Short`, `Byte`, `BigDecimal` и другими наследниками `Number`.

Методы:

1. `double sum(Collection<? extends Number> collection)` — сумма элементов;
2. `double average(Collection<? extends Number> collection)` — среднее арифметическое;
3. `<T extends Number & Comparable<T>> T min(Collection<T> collection)` — минимальный элемент;
4. `<T extends Number & Comparable<T>> T max(Collection<T> collection)` — максимальный элемент;
5. `<T extends Number> List<T> filterGreaterThan(Collection<T> collection, double threshold)` — фильтрация по порогу;
6. `<T extends Number> Map<String, Double> getStatistics(Collection<T> collection)` — статистика (сумма, среднее, мин, макс, количество) в виде карты.

**Демонстрация:**
- Создайте коллекции `List<Integer>`, `List<Double>`, `List<Long>`.
- Примените все методы утилит к каждой коллекции.
- Продемонстрируйте работу с пустой коллекцией (обработка исключений).
- Выведите статистику для каждой коллекции.

---

### Задание 4.3. Обобщённый контейнер «Очередь с приоритетом»

Разработайте обобщённый класс `PriorityQueue<T extends Comparable<T>>`, моделирующий очередь с приоритетом (элементы с наибольшим приоритетом извлекаются первыми).

**Требования к реализации:**

1. Параметр типа `T` ограничен интерфейсом `Comparable<T>` — элементы должны поддерживать сравнение.
2. Внутреннее хранение — на основе массива или связного списка, поддерживаемого в отсортированном порядке.
3. Методы:
   - `void enqueue(T element)` — добавить элемент с сохранением порядка;
   - `T dequeue()` — извлечь элемент с наивысшим приоритетом;
   - `T peek()` — просмотреть элемент с наивысшим приоритетом;
   - `boolean isEmpty()`, `int size()`, `void clear()`;
   - `List<T> toSortedList()` — вернуть все элементы в виде отсортированного списка.
4. Метод `addAll(Collection<? extends T> collection)` — добавить все элементы из коллекции (применение PECS).
5. Метод `copyTo(Collection<? super T> destination)` — скопировать все элементы в коллекцию-приёмник (применение PECS).

**Демонстрация:**
- Создайте очередь строк (приоритет — лексикографический порядок).
- Создайте очередь целых чисел.
- Создайте очередь объектов класса `Task` (с реализацией `Comparable` по полю «приоритет»).
- Для каждой очереди выполните операции `enqueue`, `dequeue`, `peek`.
- Продемонстрируйте применение `addAll` и `copyTo`.

**Пример выполнения программы:**

```
=== Очередь целых чисел ===
Enqueue: 5, 2, 8, 1, 9, 3
Отсортированный список: [1, 2, 3, 5, 8, 9]
Dequeue: 1
Dequeue: 2
Peek: 3

=== Очередь задач ===
Enqueue: Task[низкий], Task[высокий], Task[средний], Task[критический]
Dequeue: Task[критический]
Dequeue: Task[высокий]
```

## 5. Задание для самостоятельной работы

Разработать обобщённый класс и/или набор обобщённых методов согласно своему варианту. Требования:

1. Обобщённый класс должен содержать не менее одного параметра типа.
2. Реализация должна включать не менее 5 методов, использующих параметр типа.
3. Применение ограничителей типов (`extends`) — не менее чем в одном методе.
4. Применение wildcards (`? extends`, `? super`) — не менее чем в одном методе с обоснованием выбора (PECS).
5. Валидация входных данных и обработка исключительных ситуаций.
6. Демонстрационный класс должен показывать работу с различными типами (минимум тремя различными параметризациями).

### Варианты заданий

**Вариант 1.** Обобщённый класс `SparseArray<T>` — разреженный массив (хранит только ненулевые элементы). Методы: `get`, `set`, `remove`, `containsKey`, `size`, `keys`. Дополнительно: статический метод `merge` с wildcards.

**Вариант 2.** Обобщённый класс `Matrix<T extends Number>` — матрица числовых значений. Методы: `get`, `set`, `getRows`, `getColumns`, `transpose`, `multiply`. Дополнительно: метод `sum` с `? extends Number`.

**Вариант 3.** Обобщённый класс `Cache<K, V>` — кэш с ограниченным размером (LRU-стратегия). Методы: `put`, `get`, `remove`, `containsKey`, `size`, `clear`. Дополнительно: метод `copyTo` с `? super`.

**Вариант 4.** Обобщённый класс `EventBus<T>` — шина событий. Методы: `subscribe`, `unsubscribe`, `publish`, `getSubscribers`, `clear`. Дополнительно: метод `publishAll` с `? extends T`.

**Вариант 5.** Обобщённый класс `TreeNode<T extends Comparable<T>>` — узел бинарного дерева поиска. Методы: `insert`, `search`, `remove`, `inOrder`, `min`, `max`, `height`. Дополнительно: метод `copyTo` с `? super`.

**Вариант 6.** Обобщённый класс `Result<T, E>` — результат операции (успех с значением типа `T` или ошибка типа `E`). Методы: `success`, `failure`, `isSuccess`, `getValue`, `getError`, `map`, `flatMap`. Дополнительно: метод `merge` с wildcards.

**Вариант 7.** Обобщённый класс `Table<R, C, V>` — таблица (двумерная структура с типизированными строками, столбцами и значениями). Методы: `put`, `get`, `remove`, `containsCell`, `getRow`, `getColumn`, `clear`. Дополнительно: метод `copyFrom` с `? extends`.

**Вариант 8.** Обобщённый класс `Observable<T>` — наблюдаемый объект (паттерн Observer). Методы: `addListener`, `removeListener`, `notifyListeners`, `getListeners`, `clear`. Дополнительно: метод `addListener` с `? super` для listener'а.

**Вариант 9.** Обобщённый класс `Range<T extends Comparable<T>>` — диапазон значений. Методы: `contains`, `overlaps`, `intersection`, `union`, `isEmpty`, `getLower`, `getUpper`. Дополнительно: статический метод `merge` с wildcards.

**Вариант 10.** Обобщённый класс `Pool<T>` — пул объектов (переиспользование). Методы: `acquire`, `release`, `size`, `availableCount`, `clear`. Дополнительно: метод `releaseAll` с `? extends T`.

**Вариант 11.** Обобщённый класс `Tuple3<A, B, C>` — кортеж из трёх элементов. Методы: `getFirst`, `getSecond`, `getThird`, `mapFirst`, `mapSecond`, `mapThird`, `equals`, `hashCode`. Дополнительно: статический метод `from` с wildcards.

**Вариант 12.** Обобщённый класс `Repository<T extends Identifiable>` — хранилище сущностей. Методы: `save`, `findById`, `findAll`, `delete`, `count`, `exists`. Дополнительно: метод `saveAll` с `? extends T`.

**Вариант 13.** Обобщённый класс `Converter<S, T>` — преобразователь из типа `S` в тип `T`. Методы: `convert`, `convertAll`, `andThen`, `compose`. Дополнительно: метод `convertAll` с `? extends` и `? super`.

**Вариант 14.** Обобщённый класс `Multimap<K, V>` — карта, сопоставляющая ключу коллекцию значений. Методы: `put`, `get`, `remove`, `containsKey`, `keySet`, `size`, `clear`. Дополнительно: метод `putAll` с `? extends`.

**Вариант 15.** Обобщённый класс `Lazy<T>` — ленивая инициализация значения. Методы: `get`, `isInitialized`, `reset`, `of`, `ofSupplier`. Дополнительно: метод `map` с wildcards.

**Вариант 16.** Обобщённый класс `StateMachine<S, E>` — конечный автомат (состояния `S`, события `E`). Методы: `addTransition`, `fireEvent`, `getCurrentState`, `getAvailableEvents`, `reset`. Дополнительно: метод `addTransitions` с `? extends`.

**Вариант 17.** Обобщённый класс `PaginatedResult<T>` — результат постраничного запроса. Методы: `getItems`, `getPageNumber`, `getPageSize`, `getTotalElements`, `getTotalPages`, `hasNext`, `hasPrevious`. Дополнительно: метод `map` с wildcards.

**Вариант 18.** Обобщённый класс `Grouping<K, V>` — группировка элементов по ключу. Методы: `add`, `getGroup`, `getGroups`, `containsGroup`, `size`, `clear`. Дополнительно: метод `merge` с `? extends`.

**Вариант 19.** Обобщённый класс `Validator<T>` — валидатор объектов. Методы: `addRule`, `validate`, `isValid`, `getErrors`, `clearRules`. Дополнительно: метод `combine` с wildcards.

**Вариант 20.** Обобщённый класс `Diff<T>` — результат сравнения двух коллекций (добавленные, удалённые, изменённые). Методы: `getAdded`, `getRemoved`, `getChanged`, `hasChanges`, `isEmpty`. Дополнительно: статический метод `compare` с `? extends`.

**Вариант 21.** Обобщённый класс `CircuitBreaker<T>` — паттерн «предохранитель». Методы: `execute`, `getState`, `reset`, `getFailureCount`, `getSuccessCount`. Дополнительно: метод `executeAll` с `? extends`.

**Вариант 22.** Обобщённый класс `RetryPolicy<T>` — политика повторных попыток. Методы: `execute`, `getMaxAttempts`, `getDelay`, `getAttemptsMade`, `isExhausted`. Дополнительно: метод `executeAll` с wildcards.

**Вариант 23.** Обобщённый класс `BatchProcessor<T>` — пакетный обработчик. Методы: `add`, `process`, `getProcessedCount`, `getFailedCount`, `clear`. Дополнительно: метод `processAll` с `? extends T`.

**Вариант 24.** Обобщённый класс `Index<T, K>` — индекс для быстрого поиска. Методы: `add`, `findByKey`, `remove`, `containsKey`, `size`, `clear`. Дополнительно: метод `addAll` с `? extends`.

**Вариант 25.** Обобщённый класс `RateLimiter<T>` — ограничитель частоты операций. Методы: `tryAcquire`, `acquire`, `getAvailablePermits`, `reset`, `getMaxPermits`. Дополнительно: метод `tryAcquireAll` с wildcards.

**Вариант 26.** Обобщённый класс `Snapshot<T>` — снимок состояния. Методы: `capture`, `restore`, `getTimestamp`, `getData`, `isValid`. Дополнительно: метод `merge` с `? extends`.

**Вариант 27.** Обобщённый класс `Chain<T>` — цепочка обработчиков (паттерн Chain of Responsibility). Методы: `addHandler`, `removeHandler`, `execute`, `getHandlers`, `clear`. Дополнительно: метод `addHandlers` с `? extends`.

**Вариант 28.** Обобщённый класс `BloomFilter<T>` — фильтр Блума (приближённое множество). Методы: `add`, `mightContain`, `clear`, `getExpectedInsertions`, `getFalseProbability`. Дополнительно: метод `addAll` с `? extends T`.

**Вариант 29.** Обобщённый класс `Accumulator<T, R>` — накопитель значений с агрегацией. Методы: `accumulate`, `getResult`, `reset`, `getCount`, `isAccumulating`. Дополнительно: метод `merge` с wildcards.

**Вариант 30.** Обобщённый класс `Timeline<T extends Temporal>` — временная шкала событий. Методы: `addEvent`, `getEventsInRange`, `getEventsBefore`, `getEventsAfter`, `size`, `clear`. Дополнительно: метод `merge` с `? extends`.

## 6. Методические указания к самостоятельной работе

1. **Проектирование параметра типа.** Перед реализацией определите:
   - сколько параметров типа требуется классу;
   - необходимы ли ограничители (extends);
   - какие методы должны использовать wildcards и по какому принципу (PECS).

2. **Применение ограничителей.** Ограничители типов применяются, когда:
   - необходимо вызывать методы, определённые в базовом классе или интерфейсе;
   - необходимо обеспечить сравнение элементов (`Comparable`);
   - необходимо обеспечить числовую природу типа (`Number`).

3. **Применение wildcards.** Используйте wildcards в методах, а не в классах:
   - `? extends T` — когда метод читает данные из аргумента (producer);
   - `? super T` — когда метод записывает данные в аргумент (consumer);
   - `?` — когда тип не важен.

4. **Обоснование PECS.** В отчёте обязательно укажите, почему для каждого метода с wildcards выбран именно такой ограничитель, со ссылкой на принцип PECS.

5. **Избегание «сырых» типов.** Запрещается использование обобщённых классов без параметров типа (raw types). Все экземпляры должны быть параметризованы.

6. **Обработка исключений.** Реализуйте обработку:
   - нарушения ограничений (null-элементы, пустые коллекции);
   - выхода за границы (индексы, размеры);
   - нарушений состояния (операции с пустыми структурами).

7. **Тестирование.** Перед сдачей работы проверьте:
   - работу класса с различными типами (минимум тремя);
   - типобезопасность (попытки нарушения должны пресекаться компилятором);
   - корректность работы wildcards и ограничителей;
   - обработку всех граничных случаев.

8. **Применение ИИ.** При использовании средств ИИ:
   - генерируйте по отдельности обобщённый класс, методы с wildcards и демонстрационный класс;
   - обязательно проверяйте сгенерированный код на типобезопасность;
   - не делегируйте ИИ выбор ограничителей и wildcards без понимания PECS.

9. **Оформление отчёта.** Отчёт должен содержать:
   - листинги всех файлов проекта с комментариями;
   - протокол работы демонстрационного класса;
   - обоснование выбора ограничителей типов и wildcards;
   - ответы на контрольные вопросы;
   - выводы по проделанной работе.

## 7. Контрольные вопросы

1. Какую проблему решает механизм обобщений в языке Java?
2. Что такое параметр типа? Какие соглашения об именовании параметров типа существуют?
3. Чем обобщённый класс отличается от обычного класса?
4. Что такое обобщённый метод? Чем он отличается от метода обобщённого класса?
5. Что такое ограничители типов (bounded types)? Как они объявляются?
6. Что такое wildcard? Какие виды wildcards существуют?
7. В чём различие между `? extends T` и `? super T`?
8. Что такое принцип PECS? Приведите примеры его применения.
9. Что такое стирание типов (type erasure)? Каковы его последствия?
10. Какие ограничения накладывает стирание типов на использование обобщений?
11. Почему в качестве параметров типа нельзя использовать примитивные типы?
12. Что такое reifiable types? Какие обобщённые типы являются reifiable?
13. Можно ли создать массив параметризованного типа? Почему?

## 8. Рекомендуемые источники

1. Блох Дж. *Java. Эффективное программирование.* — М.: Питер. — Правила 26–35 (обобщения).
2. Хорстманн К. *Java. Библиотека профессионала. Том 1.* — М.: Вильямс. — Глава 6 (Обобщения).
3. Шилдт Г. *Java. Базовый курс.* — М.: Вильямс. — Глава 13 (Обобщения).
4. Oracle Java Tutorials. Lesson: Generics. URL: https://docs.oracle.com/javase/tutorial/extra/generics/
5. Oracle Java Language Specification. Types and Values. URL: https://docs.oracle.com/javase/specs/jls/se17/html/jls-4.html
6. Baeldung. Java Generics Tutorial. URL: https://www.baeldung.com/java-generics
7. Naftalin M., Wadler P. *Java Generics and Collections.* — O'Reilly Media.
