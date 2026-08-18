# Лабораторная работа №3. Наследование и полиморфизм

> **Назначение примера.** Иерархия `Book` показывает обезличенный технический
> сценарий. Названия классов, поля, способность и предметную операцию берите из
> своего варианта. Пример не является готовой сдаваемой работой.

## Тема

**Иерархия классов. Переопределение методов. Позднее связывание**

## Задачи

1. Модифицировать класс Book, созданный в Лабораторной работе №2, превратив его в абстрактный базовый класс.
2. Создать два класса-наследника:

    - Ebook — наследник Book. Добавить специфичные поля: fileFormat (String) и isDrmProtected (boolean).
    - PrintedEdition — наследник Book. Добавить специфичные поля: bindingType (String), pageCount (int) и stockCount (int).
3. Переопределение методов (@Override):

    - Реализовать абстрактный метод calculateDiscount() в классе Book.
        - В Ebook: скидка всегда фиксированная (например, 20%).
        - В PrintedEdition: скидка зависит от объема (например, > 500 страниц = 15%, иначе 5%).

    - Переопределить методы getDescription(boolean shortFormat) и toString() в каждом классе с учетом новых полей.
4. В методе main() создать массив базового типа Book[] library из 5 объектов (смесь Ebook и PrintedEdition).
5. В цикле вызвать System.out.println(book.getDescription(false)) и System.out.printf("Скидка: %.0f%%%n", book.calculateDiscount() * 100). Убедиться, что вызываются версии методов конкретных подклассов.
6. Добавить интерфейс Reportable с методом generateStockReport(). Реализовать этот интерфейс только в классе PrintedEdition (так как складской учет актуален для физических копий). Метод должен возвращать строку вида: "Склад [ID: X]: Печать 'Hardcover', Кол-во: 100 шт."
7. Создать список `List<Reportable> reporters`. Пройти по массиву `library`, проверить объекты через `instanceof Reportable` и добавить подходящие элементы в список. Вывести результаты `generateStockReport()` единым циклом.
8. Проверьте, что клиентский код работает через базовый тип и интерфейс, а не содержит отдельную ветку бизнес-логики для каждого подкласса. Полностью реализуйте и проверьте иерархию самостоятельно.
9. Передайте ИИ-агенту готовые объявления классов, их связи и открытые методы. Постройте с помощью ИИ-агента диаграмму классов в Mermaid или PlantUML без изменения архитектуры.

## Стек технологий

- **Язык:** Java (версия 17+)
- **IDE:** JetBrains IntelliJ IDEA Community Edition
- **VCS:** Git
- **ИИ-инструмент:** доступный чат или coding agent; конкретный сервис не обязателен

## Инструкция по выполнению

### 1. Подготовка

Используя код Лабораторной работы №2, создайте новую ветку:

```bash
git checkout -b feature/lab3-inheritance
```

### 2. Код реализации

- **Базовый класс Book.java (Abstract)**
Обратите внимание: добавлен абстрактный метод скидки, а общие поля остаются
закрытыми; наследники читают их через открытые методы:

```java
package example.course;

public abstract class Book {
    private long id;
    private String title;
    private String author;
    private int year;

    protected Book(long id, String title, String author, int year) {
        this.id = id;
        this.title = title;
        this.author = author;
        this.year = year;
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

    public String getDescription(boolean shortFormat) {
        if (shortFormat) {
            return author + " — \"" + title + "\"";
        }
        return id + ": \"" + title + "\" — " + author + " (" + year + ")";
    }

    public abstract double calculateDiscount();

    @Override
    public String toString() {
        return "Book{id=" + id + ", title='" + title + '\'' + '}';
    }
}
```

После превращения `Book` в абстрактный класс прежний фабричный метод не может
содержать `new Book(...)`. Если фабрика из лабораторной №2 сохраняется в
накопительном проекте, перенесите создание в фабрики конкретных подклассов;
счётчик можно оставить в общем защищённом конструкторе базового класса.

- **Класс Ebook.java**


```java
package example.course;

public class Ebook extends Book {
    private String fileFormat;
    private boolean isDrmProtected;

    // Конструктор делегирует вызов родительскому
    public Ebook(long id, String title, String author, int year,
                 String format, boolean drm) {
        super(id, title, author, year);
        this.fileFormat = format;
        this.isDrmProtected = drm;
    }

    @Override
    public double calculateDiscount() {
        return 0.20; // Фиксированные 20%
    }

    @Override
    public String getDescription(boolean shortFormat) {
        if (shortFormat) return super.getDescription(true);
        return "[EBOOK] " + super.getDescription(false) +
               " | Format: " + fileFormat +
               " | DRM: " + isDrmProtected;
    }

    @Override
    public String toString() {
        return getDescription(false);
    }
}
```

- **Класс PrintedEdition.java**

```java
package example.course;

public class PrintedEdition extends Book implements Reportable {
    private String bindingType;
    private int pageCount;
    private int stockCount;

    public PrintedEdition(long id, String title, String author, int year,
                          String binding, int pages, int stockCount) {
        super(id, title, author, year);
        this.bindingType = binding;
        this.pageCount = pages;
        this.stockCount = stockCount;
    }

    @Override
    public double calculateDiscount() {
        if (pageCount > 500) return 0.15;
        return 0.05;
    }

    @Override
    public String getDescription(boolean shortFormat) {
        if (shortFormat) return super.getDescription(true);
        return "[PRINTED] " + super.getDescription(false) +
               " | Binding: " + bindingType +
               " | Pages: " + pageCount +
               " | Stock: " + stockCount;
    }

    @Override
    public String generateStockReport() {
        return "Склад [ID: " + getId() + "]: Печать '" +
               bindingType + "', Кол-во: " + stockCount + " шт.";
    }

    @Override
    public String toString() {
        return getDescription(false);
    }
}
```

- **Интерфейс Reportable.java**

```java
package example.course;

public interface Reportable {
    String generateStockReport();
}
```

### 3. Демонстрация в Main.java

```java
package example.course;

import java.util.ArrayList;
import java.util.List;

public class Main {
    public static void main(String[] args) {
        System.out.println("=== ДЕМОНСТРАЦИЯ ПОЛИМОРФИЗМА ===\n");

        Book[] library = new Book[5];
        library[0] = new Ebook(1L, "Чистый код", "Мартин", 2008, "PDF", true);
        library[1] = new PrintedEdition(2L, "Дом, в котором...", "Петросян", 2009, "Hardcover", 992, 40);
        library[2] = new PrintedEdition(3L, "Мастер и Маргарита", "Булгаков", 1967, "Paperback", 512, 25);
        library[3] = new Ebook(4L, "Атлант расправил плечи", "Рэнд", 1957, "EPUB", false);
        library[4] = new PrintedEdition(5L, "Война и мир", "Толстой", 1869, "Hardcover", 1225, 10);

        List<Reportable> reporters = new ArrayList<>();

        for (Book b : library) {
            System.out.println(b.getDescription(false));
            System.out.printf("Скидка: %.0f%%%n", b.calculateDiscount() * 100);

            // Проверка возможности генерации отчета
            if (b instanceof Reportable r) {
                reporters.add(r);
            }
            System.out.println("---------------------------------");
        }

        System.out.println("\n=== ОТЧЕТЫ ПО СКЛАДУ ===");
        for (Reportable rep : reporters) {
            System.out.println(rep.generateStockReport());
        }
    }
}

```

### 4. Генерация через ИИ

- Передайте ИИ-агенту готовые объявления классов, их связи и открытые методы. Постройте с помощью ИИ-агента диаграмму классов в Mermaid или PlantUML без изменения архитектуры.
- Сопоставьте диаграмму с кодом: проверьте наследование, реализацию интерфейсов, поля, методы и модификаторы доступа. Исправьте найденные расхождения в диаграмме либо, если выявлена реальная ошибка, в коде.

### 5. Работа с системой контроля версий

- Зафиксируйте изменения:

```bash
git add src/
git commit -m "feat(lab3): implemented inheritance hierarchy Book -> Ebook/Printed"

```

## Ожидаемый результат

- исходный код иерархии и демонстрационного запуска;
- диаграмму классов в текстовом формате Mermaid или PlantUML;
- текст промпта;
- краткий перечень расхождений и принятых исправлений.
- ссылку на использованный шаблон или инструкцию по промпту;
- точные тексты промптов, существенные ответы ИИ-агента и журнал внесённых исправлений.
