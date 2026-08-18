# Лабораторная работа №4: Коллекции: List, Set, Map

> **Назначение примера.** `Book` и книжные данные — обезличенный сценарий.
> Используйте сущность, группировку, фильтр и устойчивый уникальный признак
> своего варианта. Пример показывает технику, но не заменяет КИМ.

## Тема

**Работа с основными коллекциями. Сортировка. Итерация.**

## Задачи

1. Создать List<Book> с 10 книгами. Выполнить: сортировку по году (Collections.sort + Comparator), сортировку по автору, фильтрацию (книги после 2000 года).
2. Создать Set<Book> — проверить, что дубликаты не добавляются (потребуется переопределить equals() и hashCode()).
3. Создать Map<String, List<Book>> — группировка книг по автору. Вывести автора и количество его книг.
4. Использовать Stream API: filter(), map(), collect(Collectors.groupingBy(...)), sorted().
5. Замерить время выполнения операции поиска в ArrayList и HashSet для 100 000 элементов.
6. Передайте ИИ-агенту требования и список реализованных операций. Запросите тест-план без программного кода. После утверждения плана запросите JUnit-тесты, явно передав фактические сигнатуры методов. Запустите тесты и проверьте результат.

## Стек технологий

- **Язык:** Java (версия 17+)
- **IDE:** JetBrains IntelliJ IDEA Community Edition
- **VCS:** Git
- **ИИ-инструмент:** доступный чат или coding agent; конкретный сервис не обязателен

## Инструкция по выполнению

### 1. Подготовка

Используя код Лабораторной работы №3, создайте новую ветку:

```bash
git checkout -b feature/lab4-collections
```

### 2. Подготовка (Equals & HashCode)

В классе Book.java обязательно должны быть переопределены эти методы. Без них HashSet будет считать две разные книги с одинаковым ID как разные объекты.

Реализуйте методы самостоятельно по устойчивому идентификатору. В этой работе
ИИ-агенту разрешены тест-план и JUnit-тесты, но не реализация предметного кода.

```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (!(o instanceof Book)) return false;
    Book book = (Book) o;
    return getId() == book.getId();
}

@Override
public int hashCode() {
    return Long.hashCode(getId());
}
```

### 3. Реализация Main.java

```java
package example.course;

import java.util.*;
import java.util.stream.Collectors;

public class Main {

    public static void main(String[] args) {

        // --- 1. Создание данных ---
        List<Book> library = new ArrayList<>();
        String[] titles = {
                "Чистый код",
                "Дом, в котором...",
                "Мастер и Маргарита",
                "Атлант расправил плечи",
                "Война и мир",
                "1984",
                "Пикник на обочине",
                "Гарри Поттер и Кубок огня",
                "Марсианин"
        };
        String[] authors = {
                "Мартин",
                "Петросян",
                "Булгаков",
                "Рэнд",
                "Толстой",
                "Оруэлл",
                "Стругацкие",
                "Роулинг",
                "Вейер"
        };
        int[] years = {2008, 2009, 1967, 1957, 1869, 1949, 1972, 2000, 2011};
        for (int i = 1; i <= 9; i++) {
            library.add(new Ebook(
                    i,
                    titles[i - 1],
                    authors[i - 1],
                    years[i - 1],
                    "PDF",
                    false));
        }
        // Отдельный объект с тем же устойчивым ID — логический дубликат.
        library.add(new Ebook(1L, "Чистый код, другое издание", "Мартин", 2008, "EPUB", true));

        System.out.println("=== РАБОТА С LIST ===");

        // --- 2. Сортировка List ---
        Collections.sort(library, Comparator.comparingInt(Book::getYear));
        System.out.println("\nСортировка по году:");
        library.forEach(System.out::println);

        library.sort(Comparator.comparing(Book::getAuthor));
        System.out.println("\nСортировка по автору:");
        library.forEach(System.out::println);

        // --- 3. Фильтрация ---
        List<Book> modernBooks = new ArrayList<>();
        for (Book book : library) {
            if (book.getYear() > 2000) {
                modernBooks.add(book);
            }
        }
        System.out.println("\nКниги после 2000 года: " + modernBooks.size());

        // --- 4. Работа с SET (Дубликаты) ---
        Set<Book> uniqueBooks = new HashSet<>(library);
        System.out.println("\nРазмер списка: " + library.size()); // 10
        System.out.println("Размер множества (дубликат отсечён): " + uniqueBooks.size()); // 9

        // --- 5. Группировка MAP ---
        Map<String, List<Book>> byAuthor = new HashMap<>();
        for (Book b : library) {
            byAuthor.computeIfAbsent(b.getAuthor(), k -> new ArrayList<>()).add(b);
        }

        System.out.println("\n=== ГРУППИРОВКА ПО АВТОРУ (MAP) ===");
        byAuthor.forEach((author, books) ->
            System.out.println(author + ": " + books.size() + " шт."));

        // --- 6. STREAM API (аналог группировки) ---
        System.out.println("\n=== STREAM GROUPINGBY ===");
        Map<String, Long> countByAuthor = library.stream()
                .collect(Collectors.groupingBy(
                        Book::getAuthor,
                        Collectors.counting()));
        countByAuthor.forEach((a, c) -> System.out.println(a + " -> " + c));

        System.out.println("\n=== STREAM: FILTER + SORTED + MAP ===");
        List<String> streamDescriptions = library.stream()
                .filter(book -> book.getYear() > 2000)
                .sorted(Comparator.comparingInt(Book::getYear))
                .map(book -> book.getDescription(true))
                .toList();
        streamDescriptions.forEach(System.out::println);

        // Проверки пустой коллекции, одного элемента и границы фильтра
        System.out.println("Пустой список: " +
                List.<Book>of().stream().filter(b -> b.getYear() > 2000).count());
        System.out.println("Один элемент: " +
                List.of(library.get(0)).stream().count());
        System.out.println("Граница 2000 не проходит строгий фильтр: " +
                library.stream().filter(b -> b.getYear() == 2000)
                        .filter(b -> b.getYear() > 2000).count());

        // --- 7. БЕНЧМАРК производительности ---
        performanceTest();
    }

    private static void performanceTest() {
        final int ELEMENTS = 100_000;
        final int SEARCHES = 200;
        final int RUNS = 5;

        List<Long> bigList = new ArrayList<>(ELEMENTS);
        Set<Long> bigSet = new HashSet<>(ELEMENTS);

        // Подготовка коллекций не входит в измеряемый участок.
        for (int i = 0; i < ELEMENTS; i++) {
            long key = i;
            bigList.add(key);
            bigSet.add(key);
        }

        Long searchTarget = (long) ELEMENTS - 1;

        // Короткий прогрев до измерений.
        bigList.contains(searchTarget);
        bigSet.contains(searchTarget);

        for (int run = 1; run <= RUNS; run++) {
            long start = System.nanoTime();
            int listHits = 0;
            for (int i = 0; i < SEARCHES; i++) {
                if (bigList.contains(searchTarget)) {
                    listHits++;
                }
            }
            long listTime = System.nanoTime() - start;

            start = System.nanoTime();
            int setHits = 0;
            for (int i = 0; i < SEARCHES; i++) {
                if (bigSet.contains(searchTarget)) {
                    setHits++;
                }
            }
            long setTime = System.nanoTime() - start;

            if (listHits != SEARCHES || setHits != SEARCHES) {
                throw new AssertionError("Существующий ключ должен быть найден в каждом поиске");
            }
            System.out.printf(
                    "Прогон %d: ArrayList %.2f мс; HashSet %.2f мс%n",
                    run,
                    listTime / 1_000_000.0,
                    setTime / 1_000_000.0);
        }
    }
}
```

`Main` служит демонстрационным запуском. В своей работе вынесите сортировку,
фильтрацию и группировку в методы или отдельный класс с явными сигнатурами,
чтобы JUnit-тесты проверяли возвращаемые значения, а не текст в консоли.

### 4. Работа с  ИИ-агентом.

- Передайте ИИ-агенту требования и список реализованных операций. Запросите тест-план без программного кода.
- Проверьте план: он должен охватывать обычные, граничные и ошибочные случаи, обе реализации операций и правило идентичности. Следующим промптом передайте свои замечания и получите уточнённый план.
- После утверждения плана запросите JUnit-тесты, явно передав фактические сигнатуры методов.
- Запустите тесты. Передайте ИИ-агенту результаты запуска и попросите объяснить причины ошибок, не поручая автоматическое изменение проекта.
- Самостоятельно определите, ошибочен тест или реализация, и внесите окончательные исправления.

### 5. Работа с системой контроля версий

- Зафиксируйте изменения:

```bash
git add src/
git commit -m "feat(lab4): implemented collections, streams and performance benchmark"

```

## Ожидаемый результат

- исходный код операций и JUnit-тестов;
- результаты измерения поиска;
- исходный и уточнённый тест-планы;
- последовательность промптов, результаты тестов и обоснование окончательных исправлений.
- ссылку на использованный шаблон или инструкцию по промпту;
- точные тексты промптов, существенные ответы ИИ-агента и журнал внесённых исправлений.
