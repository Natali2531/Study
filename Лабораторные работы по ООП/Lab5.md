# Лабораторная работа №5. Generics: универсальные структуры данных

## 1. Паспорт работы

| Параметр | Значение |
|----------|----------|
| Номер занятия | 5 из 17 |
| Блок | 2. Коллекции, обобщения и исключения |
| Продолжительность | 2 академических часа |
| Форма выполнения | Индивидуальная |
| ИИ-инструмент | YandexGPT / GigaChat |

### 1.1. Цель работы

Освоить механизм обобщённого программирования (generics), научиться создавать параметризованные классы и методы, применять ограничители типов.

### 1.2. Задачи работы

1. Изучить синтаксис обобщённых классов и методов.
2. Освоить ограничители типов (`extends`).
3. Научиться создавать универсальные структуры данных.
4. Понять механизм стирания типов (type erasure).

## 2. Теоретический конспект

### 2.1. Обобщённые классы

```java
public class Pair<A, B> {
    private A first;
    private B second;

    public Pair(A first, B second) {
        this.first = first;
        this.second = second;
    }

    public A getFirst() { return first; }
    public B getSecond() { return second; }
}
```

### 2.2. Обобщённые методы

```java
public static <T> void swap(T[] array, int i, int j) {
    T temp = array[i];
    array[i] = array[j];
    array[j] = temp;
}
```

### 2.3. Ограничители типов

```java
public class Pair<A extends Comparable<A>, B> {
    public A min() {
        return first.compareTo(second) <= 0 ? first : second;
    }
}
```

### 2.4. Универсальное хранилище

```java
public class Repository<T> {
    private Map<Long, T> storage = new HashMap<>();
    private long nextId = 1;

    public long save(T entity) {
        long id = nextId++;
        storage.put(id, entity);
        return id;
    }

    public T findById(long id) {
        return storage.get(id);
    }

    public boolean delete(long id) {
        return storage.remove(id) != null;
    }

    public List<T> findAll() {
        return new ArrayList<>(storage.values());
    }
}
```

## 3. Задание на паре

1. Реализовать обобщённый класс `Pair<A, B>` с полями `first` и `second`, конструктором и getter'ами.
2. Реализовать обобщённый метод `swap`, меняющий местами два элемента в массиве любого типа.
3. Переписать `Pair` так, чтобы поля допускали только наследников `Comparable`. Реализовать метод `min()` и `max()`.
4. Создать `Repository<T>` — минималистичное хранилище с методами `save(T)`, `findById(long)`, `delete(long)`, `findAll()`.

## 4. Индивидуальные задания (30 вариантов)

Каждый вариант — реализация собственного обобщённого класса:

### Варианты 1-10. Базовые структуры данных

1. **Stack<T>** — стек с методами `push`, `pop`, `peek`, `isEmpty`, `size`.
2. **Queue<T>** — очередь с методами `enqueue`, `dequeue`, `peek`, `isEmpty`.
3. **LinkedList<T>** — связный список с методами `add`, `get`, `remove`, `size`.
4. **Matrix<T extends Number>** — матрица с методами `get`, `set`, `transpose`.
5. **TreeMap<K extends Comparable<K>, V>** — упрощённое дерево поиска.
6. **Cache<K, V>** — кэш с ограниченным размером (LRU).
7. **EventBus<T>** — шина событий с методами `subscribe`, `publish`.
8. **Result<T, E>** — результат операции (успех или ошибка).
9. **Table<R, C, V>** — таблица с типизированными строками, столбцами и значениями.
10. **Observable<T>** — наблюдаемый объект (паттерн Observer).

### Варианты 11-20. Продвинутые структуры

11. **Range<T extends Comparable<T>>** — диапазон значений.
12. **Pool<T>** — пул объектов.
13. **Tuple3<A, B, C>** — кортеж из трёх элементов.
14. **Converter<S, T>** — преобразователь из типа S в тип T.
15. **Multimap<K, V>** — карта, сопоставляющая ключу коллекцию значений.
16. **Lazy<T>** — ленивая инициализация значения.
17. **StateMachine<S, E>** — конечный автомат.
18. **PaginatedResult<T>** — результат постраничного запроса.
19. **Grouping<K, V>** — группировка элементов по ключу.
20. **Validator<T>** — валидатор объектов.

### Варианты 21-30. Специализированные структуры

21. **Diff<T>** — результат сравнения двух коллекций.
22. **CircuitBreaker<T>** — паттерн «предохранитель».
23. **RetryPolicy<T>** — политика повторных попыток.
24. **BatchProcessor<T>** — пакетный обработчик.
25. **Index<T, K>** — индекс для быстрого поиска.
26. **RateLimiter<T>** — ограничитель частоты операций.
27. **Snapshot<T>** — снимок состояния.
28. **Chain<T>** — цепочка обработчиков.
29. **BloomFilter<T>** — фильтр Блума.
30. **Accumulator<T, R>** — накопитель значений с агрегацией.

## 5. Контрольные вопросы

1. Что такое generics? Какую проблему они решают?
2. Что такое параметр типа? Какие соглашения об именовании существуют?
3. Что такое ограничители типов (`extends`)?
4. Что такое стирание типов (type erasure)?
5. Можно ли создать экземпляр параметра типа (`new T()`)?

## 6. Рекомендуемые источники

1. Хорстманн К. *Java. Библиотека профессионала. Том 1.* — М.: Вильямс. — Глава 6 (Обобщения).
2. Oracle Java Tutorials: Generics. URL: https://docs.oracle.com/javase/tutorial/extra/generics/

1. Шилдт Г. *Java. Базовый курс.* — М.: Вильямс. — Глава 10 (Исключения).
2. Baeldung: Introduction to SLF4J. URL: https://www.baeldung.com/slf4j-logging-tutorial
