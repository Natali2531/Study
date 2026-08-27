# Лабораторная работа №7. Файловый ввод-вывод: чтение CSV

## 1. Паспорт работы

| Параметр | Значение |
|----------|----------|
| Номер занятия | 7 из 17 |
| Блок | 3. Ввод-вывод |
| Продолжительность | 2 академических часа |
| Форма выполнения | Индивидуальная |
| ИИ-инструмент | YandexGPT / GigaChat / JetBrains AI Assistant |

### 1.1. Цель работы

Освоить работу с файловым вводом-выводом в Java, научиться читать и парсить структурированные текстовые файлы в формате CSV, обрабатывать ошибки при чтении данных и формировать статистику обработки.

### 1.2. Задачи работы

1. Изучить классы для работы с файлами: `FileReader`, `BufferedReader`, `Files`, `Path`.
2. Освоить построчное чтение и парсинг CSV-файлов.
3. Научиться корректно обрабатывать поля с кавычками и разделителями внутри.
4. Изучить обработку некорректных строк при парсинге.
5. Освоить запись данных в новый CSV-файл.
6. Сравнить подходы: классический `BufferedReader` vs `Files.readAllLines()` + Stream API.
7. Развить навыки работы с регулярными выражениями для парсинга строк.

### 1.3. Оснащение

- JDK 17 или выше;
- IntelliJ IDEA Community Edition;
- Git;
- доступ к YandexGPT или GigaChat.

---

## 2. Теоретический конспект

### 2.1. Потоки ввода-вывода в Java

Java предоставляет две иерархии потоков:

**Байтовые потоки** — для работы с бинарными данными:

```
InputStream (байтовый ввод)
├── FileInputStream
├── ByteArrayInputStream
├── BufferedInputStream
└── ...

OutputStream (байтовый вывод)
├── FileOutputStream
├── ByteArrayOutputStream
├── BufferedOutputStream
└── ...
```

**Символьные потоки** — для работы с текстовыми данными (с учётом кодировки):

```
Reader (символьный ввод)
├── FileReader
├── StringReader
├── BufferedReader (буферизация)
└── ...

Writer (символьный вывод)
├── FileWriter
├── StringWriter
├── BufferedWriter (буферизация)
└── ...
```

### 2.2. Основные классы для работы с текстовыми файлами

**FileReader / FileWriter** — базовые классы для чтения/записи текстовых файлов:

```java
// Чтение
try (FileReader fr = new FileReader("file.txt")) {
    int ch;
    while ((ch = fr.read()) != -1) {
        System.out.print((char) ch);
    }
}

// Запись
try (FileWriter fw = new FileWriter("file.txt")) {
    fw.write("Hello, World!");
}
```

**BufferedReader / BufferedWriter** — с буферизацией для повышения производительности:

```java
// Чтение построчно
try (BufferedReader br = new BufferedReader(new FileReader("file.txt"))) {
    String line;
    while ((line = br.readLine()) != null) {
        System.out.println(line);
    }
}

// Запись с буферизацией
try (BufferedWriter bw = new BufferedWriter(new FileWriter("file.txt"))) {
    bw.write("Hello, World!");
    bw.newLine();
}
```

### 2.3. Современный подход: java.nio.file

Пакет `java.nio.file` предоставляет более удобный API для работы с файлами:

```java
import java.nio.file.*;

// Чтение всех строк
List<String> lines = Files.readAllLines(Paths.get("file.txt"), StandardCharsets.UTF_8);

// Чтение как поток строк
try (Stream<String> stream = Files.lines(Paths.get("file.txt"))) {
    stream.forEach(System.out::println);
}

// Копирование файла
Files.copy(Paths.get("source.txt"), Paths.get("dest.txt"), StandardCopyOption.REPLACE_EXISTING);

// Перемещение
Files.move(Paths.get("source.txt"), Paths.get("dest.txt"), StandardCopyOption.REPLACE_EXISTING);

// Проверка существования
boolean exists = Files.exists(Paths.get("file.txt"));

// Создание директорий
Files.createDirectories(Paths.get("data/backup"));
```

### 2.4. Формат CSV

CSV (Comma-Separated Values) — текстовый формат для представления табличных данных.

**Особенности формата:**

1. Каждая строка — одна запись.
2. Поля разделяются разделителем (обычно запятая или точка с запятой).
3. Поля могут быть заключены в кавычки.
4. Внутри кавычек могут содержаться разделители и переводы строк.
5. Для экранирования кавычек используется удвоение (`""`).

**Пример CSV:**

```csv
id,title,author,year,isbn,publisher
1,"Война и мир","Толстой Л.Н.",1869,"978-5-17-118456-0","АСТ"
2,"Преступление и наказание","Достоевский Ф.М.",1866,"978-5-17-118457-7","Эксмо"
3,"Java: полное руководство","Шилдт Г.",2019,"978-5-8459-1959-3","Вильямс"
```

### 2.5. Парсинг CSV с учётом кавычек

**Простой вариант (без учёта кавычек):**

```java
String[] fields = line.split(",");
```

**Вариант с учётом кавычек (регулярное выражение):**

```java
// Разбиение с учётом кавычек
private static final Pattern CSV_PATTERN = 
    Pattern.compile("(?:^|,)(?:\"([^\"]*)\"|([^,]*))");

private static List<String> parseCsvLine(String line) {
    List<String> fields = new ArrayList<>();
    Matcher m = CSV_PATTERN.matcher(line);
    while (m.find()) {
        String field = m.group(1) != null ? m.group(1) : m.group(2);
        fields.add(field != null ? field : "");
    }
    return fields;
}
```

**Вариант с кастомным парсером:**

```java
public static List<String> parseCsvLine(String line) {
    List<String> result = new ArrayList<>();
    StringBuilder field = new StringBuilder();
    boolean inQuotes = false;
    
    for (int i = 0; i < line.length(); i++) {
        char c = line.charAt(i);
        
        if (inQuotes) {
            if (c == '"') {
                if (i + 1 < line.length() && line.charAt(i + 1) == '"') {
                    // Экранированная кавычка
                    field.append('"');
                    i++; // пропускаем вторую кавычку
                } else {
                    inQuotes = false;
                }
            } else {
                field.append(c);
            }
        } else {
            if (c == '"') {
                inQuotes = true;
            } else if (c == ',') {
                result.add(field.toString());
                field.setLength(0);
            } else {
                field.append(c);
            }
        }
    }
    result.add(field.toString());
    return result;
}
```

### 2.6. Кодировки

Важно явно указывать кодировку при работе с текстовыми файлами:

```java
// Неправильно (использует системную кодировку)
new FileReader("file.txt");

// Правильно (явное указание кодировки)
new InputStreamReader(new FileInputStream("file.txt"), StandardCharsets.UTF_8);
new FileReader("file.txt", StandardCharsets.UTF_8); // Java 11+

// Чтение через Files
Files.readAllLines(Paths.get("file.txt"), StandardCharsets.UTF_8);
```

**Часто используемые кодировки:**
- `UTF-8` — универсальная, поддерживает все символы.
- `Windows-1251` — кириллица в Windows.
- `KOI8-R` — кириллица в Unix.

### 2.7. Stream API для работы с файлами

```java
// Фильтрация, преобразование, группировка
try (Stream<String> lines = Files.lines(Paths.get("books.csv"))) {
    Map<String, List<Book>> booksByAuthor = lines
        .skip(1) // пропускаем заголовок
        .map(BookCsvParser::parseLine)
        .filter(Optional::isPresent)
        .map(Optional::get)
        .filter(book -> book.getYear() > 2000)
        .collect(Collectors.groupingBy(Book::getAuthor));
}
```

---

## 3. Задание на паре

### Задача. Парсер CSV-файла с книгами

1. **Подготовить CSV-файл `books.csv` с 50+ строками:**
   - Поля: `id,title,author,year,isbn,publisher`.
   - Включить строки с кавычками (например, названия с запятыми).
   - Включить строки с ошибками (недостаточно полей, нечисловой год, пустой isbn).

2. **Написать парсер CSV:**
   - Чтение через `BufferedReader` построчно.
   - Разбиение строки с учётом кавычек (поля, содержащие запятые внутри кавычек).
   - Создание объектов `Book` из каждой строки.

3. **Обработать некорректные строки:**
   - Недостаточно полей.
   - Нечисловой год.
   - Пустой isbn.
   - Дубликаты по ISBN.

4. **Вывести статистику:**
   - Всего строк в файле.
   - Успешно прочитано.
   - Пропущено с ошибками.
   - Дубликатов по ISBN.

5. **Реализовать то же самое через `Files.readAllLines()` + Stream API:**
   - Сравнить читаемость кода.

6. **Записать результат в новый CSV-файл `books_clean.csv`:**
   - Только успешно прочитанные книги.
   - Сохранить заголовок.

**Пример выполнения:**

```
=== Чтение CSV-файла ===
Файл: books.csv

Статистика:
- Всего строк: 57
- Успешно прочитано: 48
- Пропущено с ошибками: 9
  - Недостаточно полей: 3
  - Нечисловой год: 2
  - Пустой ISBN: 4
- Дубликатов по ISBN: 2

Результат сохранён в: books_clean.csv

=== Сравнение подходов ===
BufferedReader: просто и понятно, подходит для больших файлов.
Files.readAllLines(): удобно, но загружает весь файл в память.
```

### Применение ИИ-инструмента

**Промпт для YandexGPT:**

```
Напиши регулярное выражение на Java для разбиения CSV-строки на поля с учётом кавычек.

Примеры строк:
1. "1","Война и мир","Толстой Л.Н.",1869,"978-5-17-118456-0","АСТ"
2. "2","Java: полное руководство","Шилдт Г.",2019,"978-5-8459-1959-3","Вильямс"
3. 3,Программирование на Java,Эккель Б.,2018,978-5-8459-1959-3,Питер

Результат должен быть в виде Pattern и метода parseLine.
```

**Анализ результата:**

- Проверить корректность разбиения на тестовых примерах.
- Проверить обработку пустых полей.
- Проверить обработку кавычек внутри строки.
- Проверить обработку экранированных кавычек.

---

## 4. Индивидуальные задания (30 вариантов)

Каждый вариант содержит:
- CSV-файл с определённой структурой;
- Требования по обработке данных;
- Дополнительные условия валидации.

---

### Вариант 1. Книги

**Структура:** `id,title,author,year,isbn,publisher`

**Валидация:** год ∈ [1, 2026], isbn не пустой, title не пустой

**Дополнительно:** сортировка по году перед записью

---

### Вариант 2. Сотрудники

**Структура:** `id,fullName,position,salary,department,hireDate`

**Валидация:** salary > 0, fullName не пустой, hireDate в формате yyyy-MM-dd

**Дополнительно:** фильтр по department

---

### Вариант 3. Товары

**Структура:** `id,name,category,price,quantity,supplier`

**Валидация:** price > 0, quantity ≥ 0, name не пустой

**Дополнительно:** группировка по категориям

---

### Вариант 4. Заказы

**Структура:** `id,customerName,totalAmount,status,createdAt,itemsCount`

**Валидация:** totalAmount > 0, customerName не пустой, status ∈ [NEW, PROCESSING, COMPLETED, CANCELLED]

**Дополнительно:** фильтр по статусу

---

### Вариант 5. Пользователи

**Структура:** `id,username,email,password,role,createdAt`

**Валидация:** email содержит @, password длина ≥ 8, role не пустой

**Дополнительно:** маскировка пароля при записи

---

### Вариант 6. Автомобили

**Структура:** `id,brand,model,year,price,vin`

**Валидация:** year ∈ [1886, 2026], price > 0, vin длина 17 символов

**Дополнительно:** выборка по бренду

---

### Вариант 7. Студенты

**Структура:** `id,firstName,lastName,group,gpa,birthDate`

**Валидация:** gpa ∈ [0.0, 5.0], birthDate в формате yyyy-MM-dd, group не пустой

**Дополнительно:** сортировка по GPA

---

### Вариант 8. Счета

**Структура:** `id,accountNumber,ownerName,balance,currency,openedDate`

**Валидация:** balance ≥ 0, accountNumber не пустой, currency ∈ [USD, EUR, RUB]

**Дополнительно:** конвертация валют

---

### Вариант 9. Фильмы

**Структура:** `id,title,director,year,rating,genre`

**Валидация:** year ∈ [1890, 2026], rating ∈ [0.0, 10.0], title не пустой

**Дополнительно:** фильтр по жанру

---

### Вариант 10. Рестораны

**Структура:** `id,name,address,phone,rating,cuisine`

**Валидация:** rating ∈ [0.0, 5.0], phone не пустой, name не пустой

**Дополнительно:** сортировка по рейтингу

---

### Вариант 11. Транзакции

**Структура:** `id,amount,type,date,description,category`

**Валидация:** amount ≠ 0, type ∈ [INCOME, EXPENSE], date в формате yyyy-MM-dd

**Дополнительно:** сумма по категориям

---

### Вариант 12. Клиенты

**Структура:** `id,firstName,lastName,email,phone,birthDate`

**Валидация:** email содержит @, phone не пустой, birthDate в формате yyyy-MM-dd

**Дополнительно:** подсчёт возраста

---

### Вариант 13. Договоры

**Структура:** `id,number,client,signDate,amount,status`

**Валидация:** amount > 0, number не пустой, signDate в формате yyyy-MM-dd

**Дополнительно:** фильтр по статусу

---

### Вариант 14. Отели

**Структура:** `id,name,city,stars,roomsCount,rating`

**Валидация:** stars ∈ [1, 5], roomsCount > 0, name не пустой

**Дополнительно:** выборка по городу

---

### Вариант 15. Спортсмены

**Структура:** `id,firstName,lastName,sport,age,medals`

**Валидация:** age ∈ [6, 100], medals ≥ 0, sport не пустой

**Дополнительно:** сортировка по медалям

---

### Вариант 16. Статьи

**Структура:** `id,title,content,author,publishedDate,views`

**Валидация:** title не пустой, views ≥ 0, publishedDate в формате yyyy-MM-dd

**Дополнительно:** выборка по автору

---

### Вариант 17. Билеты

**Структура:** `id,eventName,venue,date,price,quantity`

**Валидация:** price > 0, quantity > 0, eventName не пустой

**Дополнительно:** расчёт общей выручки

---

### Вариант 18. Курсы

**Структура:** `id,title,instructor,duration,price,startDate`

**Валидация:** duration > 0, price ≥ 0, startDate в формате yyyy-MM-dd

**Дополнительно:** выборка по дате начала

---

### Вариант 19. Инвентарь

**Структура:** `id,name,category,quantity,location,status`

**Валидация:** quantity ≥ 0, name не пустой, location не пустой

**Дополнительно:** сортировка по количеству

---

### Вариант 20. Заявки

**Структура:** `id,clientName,description,priority,status,createdAt`

**Валидация:** priority ∈ [1, 5], clientName не пустой, createdAt в формате yyyy-MM-dd

**Дополнительно:** фильтр по приоритету

---

### Вариант 21. Платёжные поручения

**Структура:** `id,number,payer,recipient,amount,date`

**Валидация:** amount > 0, number не пустой, date в формате yyyy-MM-dd

**Дополнительно:** сумма по плательщикам

---

### Вариант 22. Публикации

**Структура:** `id,title,author,journal,year,doi`

**Валидация:** year ∈ [1900, 2026], doi не пустой, title не пустой

**Дополнительно:** выборка по журналу

---

### Вариант 23. Ноутбуки

**Структура:** `id,brand,model,processor,ram,price`

**Валидация:** ram ∈ [4, 128], price > 0, brand не пустой

**Дополнительно:** сортировка по RAM

---

### Вариант 24. Врачи

**Структура:** `id,firstName,lastName,specialization,experience,rating`

**Валидация:** experience ≥ 0, rating ∈ [0.0, 5.0], specialization не пустой

**Дополнительно:** выборка по специализации

---

### Вариант 25. Сертификаты

**Структура:** `id,number,holderName,issuedDate,expiryDate,type`

**Валидация:** number не пустой, issuedDate < expiryDate, type не пустой

**Дополнительно:** проверка на истечение срока

---

### Вариант 26. Магазины

**Структура:** `id,name,address,phone,openingHours,type`

**Валидация:** name не пустой, phone не пустой, type не пустой

**Дополнительно:** группировка по типу

---

### Вариант 27. Здания

**Структура:** `id,address,floors,area,type,yearBuilt`

**Валидация:** floors > 0, area > 0, address не пустой

**Дополнительно:** сортировка по этажности

---

### Вариант 28. Лекции

**Структура:** `id,topic,speaker,date,duration,attendees`

**Валидация:** duration > 0, attendees ≥ 0, topic не пустой

**Дополнительно:** сумма посещаемости

---

### Вариант 29. Отзывы

**Структура:** `id,userId,productId,rating,comment,createdAt`

**Валидация:** userId > 0, productId > 0, rating ∈ [1, 5]

**Дополнительно:** фильтр по рейтингу

---

### Вариант 30. Чеки

**Структура:** `id,number,store,totalAmount,date,items`

**Валидация:** totalAmount > 0, number не пустой, date в формате yyyy-MM-dd

**Дополнительно:** расчёт среднего чека

---

## 5. Методические указания

### 5.1. Структура проекта

```
src/
├── main/
│   ├── java/
│   │   ├── exceptions/
│   │   │   ├── CsvParseException.java
│   │   │   └── BookNotFoundException.java
│   │   ├── model/
│   │   │   └── Book.java
│   │   ├── parser/
│   │   │   ├── CsvParser.java
│   │   │   ├── CsvParserBufferedReader.java
│   │   │   └── CsvParserStream.java
│   │   ├── statistics/
│   │   │   └── CsvStatistics.java
│   │   └── Main.java
│   └── resources/
│       ├── books.csv
│       └── logback.xml
└── test/
    └── java/
        └── parser/
            └── CsvParserTest.java
```

### 5.2. Шаблон класса Book

```java
package model;

public class Book {
    private long id;
    private String title;
    private String author;
    private int year;
    private String isbn;
    private String publisher;

    // Конструкторы, геттеры, сеттеры
    // toString(), equals(), hashCode()
}
```

### 5.3. Шаблон парсера CSV

```java
package parser;

import model.Book;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.*;
import java.nio.charset.StandardCharsets;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.*;
import java.util.stream.Stream;

public class CsvParserBufferedReader {
    private static final Logger logger = LoggerFactory.getLogger(CsvParserBufferedReader.class);
    
    // Регулярное выражение для разбиения CSV с учётом кавычек
    private static final Pattern CSV_PATTERN = 
        Pattern.compile("(?:^|,)(?:\"([^\"]*)\"|([^,]*))");
    
    public static List<Book> parseFile(String filePath) throws IOException {
        List<Book> books = new ArrayList<>();
        int lineNumber = 0;
        
        try (BufferedReader reader = new BufferedReader(
                new InputStreamReader(new FileInputStream(filePath), StandardCharsets.UTF_8))) {
            
            String line;
            boolean isHeader = true;
            
            while ((line = reader.readLine()) != null) {
                lineNumber++;
                
                if (isHeader) {
                    isHeader = false;
                    continue; // пропускаем заголовок
                }
                
                try {
                    List<String> fields = parseLine(line);
                    Book book = createBook(fields, lineNumber);
                    books.add(book);
                    logger.debug("Строка {} успешно обработана: {}", lineNumber, book.getTitle());
                } catch (CsvParseException e) {
                    logger.warn("Строка {} пропущена: {}", lineNumber, e.getMessage());
                }
            }
        }
        
        return books;
    }
    
    private static List<String> parseLine(String line) {
        List<String> fields = new ArrayList<>();
        Matcher m = CSV_PATTERN.matcher(line);
        while (m.find()) {
            String field = m.group(1) != null ? m.group(1) : m.group(2);
            fields.add(field != null ? field : "");
        }
        return fields;
    }
    
    private static Book createBook(List<String> fields, int lineNumber) 
            throws CsvParseException {
        if (fields.size() < 6) {
            throw new CsvParseException("Недостаточно полей: ожидается 6, получено " + fields.size());
        }
        
        try {
            long id = Long.parseLong(fields.get(0).trim());
            String title = fields.get(1).trim();
            String author = fields.get(2).trim();
            int year = Integer.parseInt(fields.get(3).trim());
            String isbn = fields.get(4).trim();
            String publisher = fields.get(5).trim();
            
            // Валидация
            if (title.isEmpty()) {
                throw new CsvParseException("Пустой title");
            }
            if (year <= 0 || year > 2026) {
                throw new CsvParseException("Некорректный год: " + year);
            }
            if (isbn.isEmpty()) {
                throw new CsvParseException("Пустой ISBN");
            }
            
            return new Book(id, title, author, year, isbn, publisher);
        } catch (NumberFormatException e) {
            throw new CsvParseException("Ошибка парсинга числа: " + e.getMessage());
        }
    }
}
```

### 5.4. Шаблон для статистики

```java
package statistics;

import java.util.HashMap;
import java.util.Map;

public class CsvStatistics {
    private int totalLines = 0;
    private int successCount = 0;
    private int errorCount = 0;
    private int duplicateCount = 0;
    private final Map<String, Integer> errorTypes = new HashMap<>();
    private final Map<String, Long> duplicateIsbns = new HashMap<>();
    
    public void incrementTotalLines() { totalLines++; }
    public void incrementSuccess() { successCount++; }
    public void incrementError(String type) { 
        errorCount++; 
        errorTypes.put(type, errorTypes.getOrDefault(type, 0) + 1);
    }
    public void addDuplicateIsbn(String isbn) {
        duplicateCount++;
        duplicateIsbns.put(isbn, duplicateIsbns.getOrDefault(isbn, 0L) + 1);
    }
    
    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder();
        sb.append("=== Статистика обработки ===\n");
        sb.append("Всего строк: ").append(totalLines).append("\n");
        sb.append("Успешно прочитано: ").append(successCount).append("\n");
        sb.append("Пропущено с ошибками: ").append(errorCount).append("\n");
        sb.append("Дубликатов по ISBN: ").append(duplicateCount).append("\n");
        
        if (!errorTypes.isEmpty()) {
            sb.append("\nТипы ошибок:\n");
            for (Map.Entry<String, Integer> entry : errorTypes.entrySet()) {
                sb.append("  - ").append(entry.getKey()).append(": ").append(entry.getValue()).append("\n");
            }
        }
        
        return sb.toString();
    }
}
```

### 5.5. Сравнение BufferedReader vs Stream API

**BufferedReader (классический):**

```java
// Простой, понятный, построчная обработка
try (BufferedReader reader = new BufferedReader(new FileReader(filePath))) {
    String line;
    while ((line = reader.readLine()) != null) {
        processLine(line);
    }
}
```

**Files.readAllLines() + Stream API (современный):**

```java
// Декларативный стиль, цепочки операций
try (Stream<String> lines = Files.lines(Paths.get(filePath), StandardCharsets.UTF_8)) {
    List<Book> books = lines
        .skip(1) // пропустить заголовок
        .map(CsvParserStream::parseLine)
        .filter(Optional::isPresent)
        .map(Optional::get)
        .collect(Collectors.toList());
}
```

**Сравнение:**

| Критерий | BufferedReader | Stream API |
|----------|---------------|------------|
| Читаемость | Простой императивный | Декларативный, цепочки |
| Производительность | Хорошая | Хорошая (ленивые операции) |
| Память | Минимальная (построчно) | Загружает всё (при collect) |
| Сложность | Низкая | Средняя |
| Гибкость | Ручная обработка | Мощные операции (filter, map) |
| Java версия | Любая | Java 8+ |

### 5.6. Обработка ошибок

```java
public static List<Book> parseFileWithErrorHandling(String filePath) {
    List<Book> books = new ArrayList<>();
    CsvStatistics stats = new CsvStatistics();
    
    try (BufferedReader reader = new BufferedReader(
            new FileReader(filePath, StandardCharsets.UTF_8))) {
        
        // Чтение заголовка
        String header = reader.readLine();
        stats.incrementTotalLines();
        
        String line;
        int lineNumber = 1;
        Set<String> uniqueIsbns = new HashSet<>();
        
        while ((line = reader.readLine()) != null) {
            lineNumber++;
            stats.incrementTotalLines();
            
            try {
                Book book = parseBook(line, lineNumber);
                
                // Проверка дубликатов по ISBN
                if (!uniqueIsbns.add(book.getIsbn())) {
                    stats.addDuplicateIsbn(book.getIsbn());
                    logger.warn("Дубликат ISBN: {} в строке {}", book.getIsbn(), lineNumber);
                    continue;
                }
                
                books.add(book);
                stats.incrementSuccess();
                
            } catch (CsvParseException e) {
                stats.incrementError(e.getType());
                logger.warn("Ошибка в строке {}: {}", lineNumber, e.getMessage());
            }
        }
        
    } catch (IOException e) {
        logger.error("Ошибка при чтении файла: {}", filePath, e);
        throw new RuntimeException("Не удалось прочитать файл", e);
    }
    
    System.out.println(stats);
    return books;
}
```

### 5.7. Запись в CSV

```java
public static void writeBooksToCsv(List<Book> books, String filePath) throws IOException {
    try (BufferedWriter writer = new BufferedWriter(
            new OutputStreamWriter(new FileOutputStream(filePath), StandardCharsets.UTF_8))) {
        
        // Запись заголовка
        writer.write("id,title,author,year,isbn,publisher");
        writer.newLine();
        
        // Запись данных
        for (Book book : books) {
            writer.write(String.format("%d,\"%s\",\"%s\",%d,\"%s\",\"%s\"",
                book.getId(),
                escapeCsv(book.getTitle()),
                escapeCsv(book.getAuthor()),
                book.getYear(),
                escapeCsv(book.getIsbn()),
                escapeCsv(book.getPublisher())
            ));
            writer.newLine();
        }
        
        logger.info("Записано {} книг в файл {}", books.size(), filePath);
    }
}

private static String escapeCsv(String value) {
    if (value == null) return "";
    // Экранирование кавычек удвоением
    return value.replace("\"", "\"\"");
}
```

---

## 6. Контрольные вопросы

1. В чём разница между байтовыми и символьными потоками в Java?

2. Для чего нужен `BufferedReader`? В чём его преимущество перед `FileReader`?

3. Что такое буферизация и зачем она нужна при работе с файлами?

4. Как явно указать кодировку при чтении файла?

5. Какие сложности возникают при парсинге CSV-файлов?

6. Как обрабатывать поля, содержащие запятые внутри кавычек?

7. Как экранировать кавычки внутри поля CSV?

8. В чём разница между `Files.readAllLines()` и `Files.lines()`?

9. Как обрабатывать ошибки при парсинге (некорректные строки)?

10. Почему важно проверять дубликаты при загрузке данных?

11. Какие преимущества даёт Stream API при работе с файлами?

12. Как в Java указывается кодировка при чтении/записи файлов?

---

## 7. Пример выполнения (Вариант 1)

### 7.1. Подготовка CSV-файла (`books.csv`)

```csv
id,title,author,year,isbn,publisher
1,"Война и мир","Толстой Л.Н.",1869,"978-5-17-118456-0","АСТ"
2,"Преступление и наказание","Достоевский Ф.М.",1866,"978-5-17-118457-7","Эксмо"
3,"Java: полное руководство","Шилдт Г.",2019,"978-5-8459-1959-3","Вильямс"
4,"Программирование на Java","Эккель Б.",2018,"978-5-8459-1960-9","Питер"
5,"Чистый код","Мартин Р.",2012,"978-5-8459-1961-6","Вильямс"
6,"Рефакторинг","Фаулер М.",2012,"978-5-8459-1962-3","Вильямс"
7,"Идиот","Достоевский Ф.М.",1868,"978-5-17-118458-4","АСТ"
8,"Анна Каренина","Толстой Л.Н.",1877,"978-5-17-118459-1","АСТ"
9,"Совершенный код","Макконнелл С.",2004,"978-5-8459-1963-0","Питер"
10,"Эффективное программирование","Блох Дж.",2008,"978-5-8459-1964-7","Вильямс"
11,"Властелин колец","Толкин Дж.Р.Р.",1954,"978-5-17-118460-7","АСТ"
12,"Гарри Поттер и философский камень","Роулинг Дж.К.",1997,"978-5-17-118461-4","РОСМЭН"
13,"1984","Оруэлл Дж.",1949,"978-5-17-118462-1","АСТ"
14,"Мастер и Маргарита","Булгаков М.А.",1967,"978-5-17-118463-8","Эксмо"
15,"Три товарища","Ремарк Э.М.",1936,"978-5-17-118464-5","АСТ"
16,"Гордость и предубеждение","Остин Дж.",1813,"978-5-17-118465-2","Эксмо"
17,"Война миров","Уэллс Г.",1898,"978-5-17-118466-9","АСТ"
18,"О дивный новый мир","Хаксли О.",1932,"978-5-17-118467-6","АСТ"
19,"Фауст","Гёте И.В.",1808,"978-5-17-118468-3","Эксмо"
20,"Дон Кихот","Сервантес М.",1605,"978-5-17-118469-0","АСТ"
```

### 7.2. Основной класс

```java
import model.Book;
import parser.CsvParserBufferedReader;
import parser.CsvParserStream;

import java.io.IOException;
import java.util.List;

public class Main {
    private static final String INPUT_FILE = "src/main/resources/books.csv";
    private static final String OUTPUT_FILE = "src/main/resources/books_clean.csv";
    
    public static void main(String[] args) {
        System.out.println("=== Чтение CSV-файла ===\n");
        
        try {
            // Чтение через BufferedReader
            System.out.println("1. Чтение через BufferedReader:");
            long startTime = System.nanoTime();
            List<Book> books = CsvParserBufferedReader.parseFile(INPUT_FILE);
            long endTime = System.nanoTime();
            System.out.println("Время выполнения: " + (endTime - startTime) / 1_000_000 + " мс");
            System.out.println("Успешно прочитано: " + books.size() + " книг");
            
            // Чтение через Stream API
            System.out.println("\n2. Чтение через Stream API:");
            startTime = System.nanoTime();
            List<Book> books2 = CsvParserStream.parseFile(INPUT_FILE);
            endTime = System.nanoTime();
            System.out.println("Время выполнения: " + (endTime - startTime) / 1_000_000 + " мс");
            System.out.println("Успешно прочитано: " + books2.size() + " книг");
            
            // Запись результата
            System.out.println("\n3. Запись результата:");
            CsvParserBufferedReader.writeBooksToCsv(books, OUTPUT_FILE);
            System.out.println("Результат сохранён в: " + OUTPUT_FILE);
            
        } catch (IOException e) {
            System.err.println("Ошибка при работе с файлом: " + e.getMessage());
        }
    }
}
```

### 7.3. Ожидаемый вывод

```
=== Чтение CSV-файла ===

1. Чтение через BufferedReader:
=== Статистика обработки ===
Всего строк: 21
Успешно прочитано: 20
Пропущено с ошибками: 1
Дубликатов по ISBN: 0

Типы ошибок:
  - Пустой title: 1

Время выполнения: 15 мс
Успешно прочитано: 20 книг

2. Чтение через Stream API:
=== Статистика обработки ===
Всего строк: 21
Успешно прочитано: 20
Пропущено с ошибками: 1
Дубликатов по ISBN: 0

Типы ошибок:
  - Пустой title: 1

Время выполнения: 18 мс
Успешно прочитано: 20 книг

3. Запись результата:
Результат сохранён в: src/main/resources/books_clean.csv
```

---

## 8. Рекомендуемые источники

1. **Шилдт Г.** *Java. Базовый курс.* — М.: Вильямс. — Главы 13, 15 (Ввод-вывод, NIO).

2. **Хорстманн К., Корнелл Г.** *Java. Библиотека профессионала. Том 2.* — М.: Вильямс. — Глава 2 (Ввод-вывод).

3. **Oracle Java Tutorials: Basic I/O.** — URL: https://docs.oracle.com/javase/tutorial/essential/io/

4. **RFC 4180 — Common Format and MIME Type for CSV Files.** — URL: https://tools.ietf.org/html/rfc4180

5. **OpenCSV Documentation.** — URL: http://opencsv.sourceforge.net/
