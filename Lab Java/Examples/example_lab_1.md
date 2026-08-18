# Лабораторная работа №1. Настройка окружения. Первый класс

> **Назначение примера.** `Book` — обезличенная учебная сущность, показывающая
> только техническую форму решения. В сдаваемой работе используйте класс, поля
> и предметные данные назначенного варианта. Не копируйте пример как готовое
> решение.

## Тема

**Установка JDK, IntelliJ IDEA (Community Edition), Git. Создание класса с инкапсуляцией**

## Задачи

1. Установить среду выполнения JDK 17 или выше и интегрированную среду разработки IntelliJ IDEA Community Edition.
2. Инициализировать пустой Git-репозиторий для проекта, настроить .gitignore для игнорирования служебных файлов IDE и скомпилированного кода (*.class, /out/), выполнить первый коммит.
3. Создать проект в IntelliJ IDEA и реализовать класс Book, соблюдая принципы инкапсуляции:

    - все поля объявлены как private.
    - для каждого поля реализованы публичные методы доступа (getter) и изменения (setter).
    - реализовать метод getDescription(), формирующий строковое представление объекта по заданному шаблону.

4. Написать код в методе main() класса для инстанцирования трех различных объектов типа Book и вывода результата их метода getDescription() в консоль.
5. Протестировать работу программы, зафиксировать промежуточное состояние через коммиты в Git.
6. Подготовить с помощью ИИ черновик README.md с кратким описанием проекта, требованиями к окружению и инструкцией по запуску.

## Стек технологий

- **Язык:** Java (версия 17+)
- **Сборка:** Встроенная система сборки IntelliJ IDEA / Maven / Gradle (опционально)
- **IDE:** JetBrains IntelliJ IDEA Community Edition
- **VCS:** Git
- **ИИ-инструмент:** доступный чат или coding agent; конкретный сервис не обязателен

***

## Инструкции

### 1. Подготовка окружения

- Установите любой совместимый дистрибутив JDK версии 17 или выше. Проверьте установку командами `java -version` и `javac -version`.
- Скачайте и установите IntelliJ IDEA Community Edition.
- Убедитесь, что установлен Git. Проверьте командой git --version. При необходимости выполните базовую настройку пользователя:

```bash

git config --global user.name "Ваше Имя"
git config --global user.email "your_email@example.com"

```

### 2. Создание репозитория

- Создайте новую папку для проекта.
- Откройте терминал в этой папке и выполните

```bash
git init
touch .gitignore
```

- Добавьте в .gitignore следующие строки:

```text
# IntelliJ files
.idea/
*.iml
out/
target/

# Compiled class files
*.class

# OS generated files
.DS_Store
Thumbs.db

```

- Выполните первый коммит настроек:

```bash
git add .
git commit -m "chore: initial project setup and .gitignore"

```

### 3. Проект в IDEA

- Запустите IntelliJ IDEA -> New Project -> Java.
- Выберите установленный JDK 17+.
- Создайте новый пакет (например, `example.course`) и внутри него — класс `Book.java`.

### 4. Ручная реализация класса Book

- Создайте файл `src/example/course/Book.java` со следующей структурой:

```java
package example.course;

public class Book {
    private long id;
    private String title;
    private String author;
    private int year;

    // Getters and Setters
    public long getId() {
        return id;
    }

    public void setId(long id) {
        this.id = id;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getAuthor() {
        return author;
    }

    public void setAuthor(String author) {
        this.author = author;
    }

    public int getYear() {
        return year;
    }

    public void setYear(int year) {
        this.year = year;
    }

    /**
     * Возвращает описание книги в формате «ID: "Название" — Автор (Год)».
     */
    public String getDescription() {
        return id + ": \"" + title + "\" — " + author + " (" + year + ")";
    }
}
```

### 5. Класс Main для запуска

- Создайте класс Main.java (или Lab1.java) в том же пакете:

```java
package example.course;

public class Main {
    public static void main(String[] args) {
        Book book1 = new Book();
        book1.setId(1L);
        book1.setTitle("Война и мир");
        book1.setAuthor("Толстой Л.Н.");
        book1.setYear(1869);

        Book book2 = new Book();
        book2.setId(2L);
        book2.setTitle("Преступление и наказание");
        book2.setAuthor("Достоевский Ф.М.");
        book2.setYear(1866);

        Book book3 = new Book();
        book3.setId(3L);
        book3.setTitle("Мастер и Маргарита");
        book3.setAuthor("Булгаков М.А.");
        book3.setYear(1967);

        System.out.println(book1.getDescription());
        System.out.println(book2.getDescription());
        System.out.println(book3.getDescription());
    }
}
```

### 6. Подготовка README с ИИ-инструментом

- Передайте ИИ-агенту название проекта, назначение реализованного класса, используемую версию Java и фактический способ запуска
- Попросите подготовить черновик README.md с кратким описанием проекта, требованиями к окружению и инструкцией по запуску
- Сопоставьте инструкцию с проектом, исправьте неточные команды и удалите неподтверждённые утверждения

### 7. Работа с системой контроля версий

- После завершения реализации добавьте файлы в индекс и зафиксируйте прогресс:

```bash
git add src/
git commit -m "feat: implemented Book entity and Main runner"
```

***

## Ожидаемый результат

- исходный код проекта;
- .gitignore;
- проверенный README.md;
- текст промпта и краткий список внесённых в ответ ИИ-агента исправлений;
- ссылку на использованный шаблон или инструкцию по промпту;
- точные тексты промптов, существенные ответы ИИ-агента и журнал внесённых исправлений.
