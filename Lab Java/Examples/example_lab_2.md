# Лабораторная работа №2. Конструкторы, перегрузка, static

> **Назначение примера.** `Book` — обезличенная учебная сущность. Перенесите
> показанные механизмы на класс и правила создания своего варианта; пример не
> заменяет КИМ и перечень вариантов.

## Тема

**Перегрузка конструкторов и методов. Статические члены класса.**

## Задачи

1. Доработать класс Book: добавить конструктор без параметров (id=0, title="Без названия", author="Неизвестен", year=0), конструктор со всеми параметрами, конструктор без id (где полю id присваивается значение 0).
2. Организовать цепочку вызовов конструкторов с использованием ключевого слова this(...).
3. Добавить статическое поле-счетчик counter и статический метод getCounter(), возвращающий общее количество созданных объектов.
4. Реализовать статический фабричный метод createBook(String title, String author, int year), который автоматически инкрементирует счетчик ID и возвращает новый объект Book.
5. Реализовать перегруженный метод getDescription(boolean shortFormat). Если флаг равен true, возвращается краткий формат (например, только «Автор — Название»), если false — полный формат из первой лабораторной.
6. Протестировать все варианты создания объектов в методе main() и зафиксировать код через коммиты Git.
7. Передать ИИ-агенту сигнатуры готовых элементов и описание их фактического поведения, подготовить Javadoc для открытых конструкторов и методов, а также короткие примеры их использования.

## Стек технологий

- **Язык:** Java (версия 17+)
- **IDE:** JetBrains IntelliJ IDEA Community Edition
- **VCS:** Git
- **ИИ-инструмент:** доступный чат или coding agent; конкретный сервис не обязателен

## Инструкция по выполнению

### 1. Подготовка

Убедитесь, что репозиторий инициализирован. Создайте новую ветку для чистоты истории:

```bash
git checkout -b feature/lab2-overloading-statics
```

### 2. Модификация класса Book.java

- Конструкторы и this(...)
- Реализуйте три конструктора, используя цепочку вызовов для исключения дублирования кода:

```java
package example.course;

public class Book {
    private long id;
    private String title;
    private String author;
    private int year;
    private static long counter = 0;
    private static long nextId = 1;

    // 1. Конструктор без параметров
    public Book() {
        this(0, "Без названия", "Неизвестен", 0);
    }

    // 2. Конструктор без id (делегируем полному конструктору, id будет присвоен внутри или останется 0)
    public Book(String title, String author, int year) {
        this(0, title, author, year);
    }

    // 3. Полный конструктор
    public Book(long id, String title, String author, int year) {
        this.id = id;
        this.title = title;
        this.author = author;
        this.year = year;
        counter++;
        if (id >= nextId) {
            nextId = id + 1;
        }
    }

    public long getId() {
        return id;
    }

    public String getTitle() {
        return title;
    }

    public String getAuthor() {
        return author;
    }

    public int getYear() {
        return year;
    }

    public static long getCounter() {
        return counter;
    }

    // Конструктор сам учитывает объект, поэтому фабрика counter не изменяет.
    public static Book createBook(String title, String author, int year) {
        return new Book(nextId++, title, author, year);
    }

    public String getDescription() {
        return id + ": \"" + title + "\" — " + author + " (" + year + ")";
    }

    /**
     * Возвращает краткое или полное описание книги.
     */
    public String getDescription(boolean shortFormat) {
        if (shortFormat) {
            return author + " — \"" + title + "\"";
        }
        return getDescription();
    }
}
```

Счётчик означает количество созданных объектов, поэтому он увеличивается ровно
один раз в полном конструкторе. Не используйте `finalize()`: сборка мусора не
определяет факт создания объекта и не должна менять такой счётчик.

### 3. Класс Main.java для тестирования

- Запустите IntelliJ IDEA -> New Project -> Java.
- Выберите установленный JDK 17+.
- Продолжайте работу в пакете предыдущей лабораторной.

### 4. Демонстрационный запуск

```java
package example.course;

public class Main {
    public static void main(String[] args) {
        System.out.println("Всего книг создано до начала: " + Book.getCounter());

        // Тест конструкторов
        Book b1 = new Book(); // По умолчанию
        Book b2 = new Book("Война и мир", "Толстой Л.Н.", 1869); // Без явного id
        Book b3 = new Book(99L, "Мастер и Маргарита", "Булгаков М.А.", 1967); // С полным id

        // Тест фабрики
        Book b4 = Book.createBook("Преступление и наказание", "Достоевский Ф.М.", 1866);

        System.out.println("\n--- Описания (полные) ---");
        System.out.println(b1.getDescription(false));
        System.out.println(b2.getDescription(false));
        System.out.println(b3.getDescription(false));
        System.out.println(b4.getDescription(false));

        System.out.println("\n--- Описания (краткие) ---");
        System.out.println(b1.getDescription(true));
        System.out.println(b2.getDescription(true));
        System.out.println(b3.getDescription(true));
        System.out.println(b4.getDescription(true));

        System.out.println("\nИтоговое количество созданных книг: " + Book.getCounter());
        if (Book.getCounter() != 4) {
            throw new AssertionError("Каждый из четырёх объектов должен быть учтён один раз");
        }
    }
}
```

### 5. Работа с системой контроля версий

- Зафиксируйте изменения:

```bash
git add src/
git commit -m "feat: constructor overloading, static counter and factory method"

```

### 6. Генерация с помощью ИИ-агента

- передайте ИИ-агенту сигнатуры готовых элементов и описание их фактического поведения
- подготовьте Javadoc для открытых конструкторов и методов, а также короткие примеры их использования

## Ожидаемый результат

- обновлённый исходный код;
- Javadoc для открытого API класса;
- примеры использования всех конструкторов и фабричного метода;
- текст промпта и перечень исправлений документации;
- ссылку на использованный шаблон или инструкцию по промпту;
- точные тексты промптов, существенные ответы ИИ-агента и журнал внесённых исправлений.
