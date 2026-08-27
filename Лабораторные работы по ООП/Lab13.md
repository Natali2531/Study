# Лабораторная работа №13. ETL-пайплайн: Extract + Transform (данные из файлов в БД)

## 1. Паспорт работы

| Параметр | Значение |
|----------|----------|
| Номер занятия | 13 из 17 |
| Блок | 6. ETL-пайплайн |
| Продолжительность | 2 академических часа |
| Форма выполнения | Индивидуальная |
| ИИ-инструмент | YandexGPT / GigaChat / JetBrains AI Assistant |

### 1.1. Цель работы

Освоить проектирование и реализацию ETL-пайплайна (Extract, Transform, Load), научиться создавать универсальные ридеры данных из различных источников (CSV, JSON), реализовывать этапы нормализации, фильтрации и обогащения данных.

### 1.2. Задачи работы

1. Изучить концепцию ETL (Extract, Transform, Load).
2. Реализовать универсальный ридер `DataSourceReader` с поддержкой CSV и JSON (паттерн «Стратегия»).
3. Освоить нормализацию данных (приведение к единому формату).
4. Научиться фильтровать данные (исключение записей без ISBN).
5. Освоить обогащение данных (добавление категории из справочника).
6. Изучить использование цепочек промптов для проектирования.
7. Развить навыки работы с различными форматами данных.

### 1.3. Оснащение

- JDK 17 или выше;
- IntelliJ IDEA Community Edition;
- Git;
- Maven или Gradle;
- доступ к YandexGPT или GigaChat.

---

## 2. Теоретический конспект

### 2.1. Что такое ETL

**ETL (Extract, Transform, Load)** — процесс извлечения данных из источников, их преобразования и загрузки в целевую систему.

```
┌──────────────────────────────────────────────────────────────────┐
│                          ETL Pipeline                          │
│                                                                 │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐         │
│  │   EXTRACT   │───▶│  TRANSFORM  │───▶│    LOAD     │         │
│  │  Извлечение │    │Преобразование│    │  Загрузка   │         │
│  └─────────────┘    └─────────────┘    └─────────────┘         │
│        │                   │                   │                │
│        ▼                   ▼                   ▼                │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐         │
│  │ • CSV       │    │ • Фильтрация│    │ • БД        │         │
│  │ • JSON      │    │ • Нормализ. │    │ • Файл      │         │
│  │ • XML       │    │ • Обогащ.   │    │ • API       │         │
│  │ • БД        │    │ • Агрегац.  │    │             │         │
│  │ • API       │    │             │    │             │         │
│  └─────────────┘    └─────────────┘    └─────────────┘         │
└──────────────────────────────────────────────────────────────────┘
```

### 2.2. Паттерн «Стратегия» (Strategy)

Паттерн Стратегия позволяет выбирать алгоритм обработки на лету.

```
┌──────────────────────────────────────────────────────────────┐
│                   DataSourceReader                          │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  + readFile(String path): List<Book>                │   │
│  │  + setParser(ParserStrategy strategy)               │   │
│  └────────────────────┬─────────────────────────────────┘   │
│                       │                                      │
│           ┌───────────┼───────────┐                         │
│           ▼           ▼           ▼                         │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐              │
│  │ CsvStrategy│ │JsonStrategy│ │ XmlStrategy│              │
│  └────────────┘ └────────────┘ └────────────┘              │
└──────────────────────────────────────────────────────────────┘
```

**Реализация паттерна Стратегия:**

```java
// Интерфейс стратегии
public interface ParserStrategy {
    List<Book> parse(String filePath) throws IOException;
}

// Конкретная стратегия для CSV
public class CsvParserStrategy implements ParserStrategy {
    @Override
    public List<Book> parse(String filePath) throws IOException {
        // Парсинг CSV
    }
}

// Конкретная стратегия для JSON
public class JsonParserStrategy implements ParserStrategy {
    @Override
    public List<Book> parse(String filePath) throws IOException {
        // Парсинг JSON
    }
}

// Контекст
public class DataSourceReader {
    private ParserStrategy strategy;
    
    public DataSourceReader(ParserStrategy strategy) {
        this.strategy = strategy;
    }
    
    public List<Book> read(String filePath) throws IOException {
        return strategy.parse(filePath);
    }
}
```

### 2.3. Нормализация данных

**Нормализация** — приведение данных к единому формату.

**Пример нормализации имени автора:**
- Входные варианты:
  - `"Толстой Л.Н."`
  - `"Л.Н. Толстой"`
  - `"Толстой Лев Николаевич"`
  - `"Толстой Л. Н."`
  - `"Лев Николаевич Толстой"`

- Выходной формат: `"Фамилия И.О."` → `"Толстой Л.Н."`

**Алгоритм нормализации:**

```java
public class AuthorNormalizer {
    public String normalize(String input) {
        if (input == null || input.trim().isEmpty()) {
            return input;
        }
        
        // Удаление лишних пробелов
        String normalized = input.trim().replaceAll("\\s+", " ");
        
        // Проверка формата "Фамилия И.О."
        if (normalized.matches("^[А-ЯЁ][а-яё]+\\s+[А-ЯЁ]\\.[А-ЯЁ]\\.?$")) {
            return normalized;
        }
        
        // Проверка формата "И.О. Фамилия"
        if (normalized.matches("^[А-ЯЁ]\\.[А-ЯЁ]\\.?\\s+[А-ЯЁ][а-яё]+$")) {
            String[] parts = normalized.split("\\s+");
            return parts[1] + " " + parts[0];
        }
        
        // Проверка формата "Фамилия Имя Отчество"
        if (normalized.matches("^[А-ЯЁ][а-яё]+\\s+[А-ЯЁ][а-яё]+\\s+[А-ЯЁ][а-яё]+$")) {
            String[] parts = normalized.split("\\s+");
            String surname = parts[0];
            String initials = parts[1].charAt(0) + "." + parts[2].charAt(0) + ".";
            return surname + " " + initials;
        }
        
        // Проверка формата "Имя Отчество Фамилия"
        if (normalized.matches("^[А-ЯЁ][а-яё]+\\s+[А-ЯЁ][а-яё]+\\s+[А-ЯЁ][а-яё]+$")) {
            String[] parts = normalized.split("\\s+");
            String surname = parts[2];
            String initials = parts[0].charAt(0) + "." + parts[1].charAt(0) + ".";
            return surname + " " + initials;
        }
        
        // Возврат оригинального значения, если формат не распознан
        return normalized;
    }
}
```

### 2.4. Обогащение данных (Enrichment)

**Обогащение** — добавление дополнительной информации к данным.

**Пример обогащения книг категориями:**

```java
public class CategoryEnricher {
    private final Map<String, String> categoryMap;
    
    public CategoryEnricher(String categoryFile) throws IOException {
        this.categoryMap = loadCategoryMap(categoryFile);
    }
    
    private Map<String, String> loadCategoryMap(String filePath) throws IOException {
        Map<String, String> map = new HashMap<>();
        try (BufferedReader reader = new BufferedReader(
                new FileReader(filePath, StandardCharsets.UTF_8))) {
            String line;
            while ((line = reader.readLine()) != null) {
                String[] parts = line.split(",");
                if (parts.length >= 2) {
                    map.put(parts[0].toLowerCase(), parts[1]);
                }
            }
        }
        return map;
    }
    
    public String enrich(String title) {
        if (title == null || title.isEmpty()) {
            return "Unknown";
        }
        
        String lowerTitle = title.toLowerCase();
        for (Map.Entry<String, String> entry : categoryMap.entrySet()) {
            if (lowerTitle.contains(entry.getKey())) {
                return entry.getValue();
            }
        }
        return "Other";
    }
}
```

### 2.5. Фильтрация данных

**Фильтрация** — исключение записей, не соответствующих критериям.

```java
public class BookFilter {
    public boolean isValid(Book book) {
        // Проверка наличия ISBN
        if (book.getIsbn() == null || book.getIsbn().trim().isEmpty()) {
            return false;
        }
        
        // Проверка валидности года
        if (book.getYear() <= 0 || book.getYear() > Year.now().getValue()) {
            return false;
        }
        
        // Проверка наличия названия
        if (book.getTitle() == null || book.getTitle().trim().isEmpty()) {
            return false;
        }
        
        return true;
    }
}
```

### 2.6. Цепочки промптов (Prompt Chaining)

**Цепочка промптов** — разбиение сложной задачи на последовательность простых запросов.

```
┌─────────────────────────────────────────────────────────────────┐
│                     Цепочка промптов                           │
│                                                                 │
│  ┌─────────────────┐    ┌─────────────────┐                    │
│  │   Промпт 1      │    │   Промпт 2      │                    │
│  │  Проектирование │───▶│  Генерация      │                    │
│  │  архитектуры    │    │  кода модели    │                    │
│  └─────────────────┘    └─────────────────┘                    │
│                                    │                           │
│                                    ▼                           │
│                          ┌─────────────────┐                    │
│                          │   Промпт 3      │                    │
│                          │  Генерация      │                    │
│                          │  кода клиента   │                    │
│                          └─────────────────┘                    │
└─────────────────────────────────────────────────────────────────┘
```

**Пример цепочки для ETL:**

1. **Промпт 1:** "Спроектируй архитектуру ETL-пайплайна для загрузки книг из CSV и JSON в PostgreSQL."

2. **Промпт 2:** "На основе архитектуры сгенерируй интерфейс DataSourceReader с поддержкой CSV и JSON."

3. **Промпт 3:** "На основе архитектуры и интерфейса сгенерируй реализацию трансформации (нормализация автора, фильтрация по ISBN, обогащение категориями)."

---

## 3. Задание на паре

### Задача. Реализация ETL-пайплайна для книг

1. **Реализовать универсальный ридер `DataSourceReader`:**
   - Поддержка CSV и JSON.
   - Использовать паттерн «Стратегия».
   - Обработка ошибок при чтении.

2. **Реализовать этап Transform:**
   - **Нормализация:** привести `author` к формату «Фамилия И.О.».
   - **Фильтрация:** исключить книги без ISBN.
   - **Обогащение:** добавить поле `category` на основе ключевых слов в названии (справочник категорий в отдельном CSV).

3. **Результат Transform:** `List<Book>` с очищенными и обогащёнными данными.

4. **Создать справочник категорий (`categories.csv`):**
   ```
   keyword,category
   война,Военная литература
   программирование,IT
   java,IT
   python,IT
   детектив,Детектив
   фэнтези,Фэнтези
   наука,Научная литература
   история,Историческая литература
   роман,Художественная литература
   ```

5. **Использовать цепочку промптов для проектирования:**
   - Промпт 1: спроектировать DataSourceReader.
   - Промпт 2: написать метод нормализации автора.
   - Промпт 3: написать метод обогащения категориями.

**Пример выполнения:**
```
=== EXTRACT ===
Чтение файла: books.csv
Прочитано: 50 записей

Чтение файла: books.json
Прочитано: 30 записей

=== TRANSFORM ===
Нормализация авторов:
  "Толстой Л.Н." → "Толстой Л.Н."
  "Л.Н. Толстой" → "Толстой Л.Н."
  "Толстой Лев Николаевич" → "Толстой Л.Н."

Фильтрация по ISBN:
  Исключено: 8 книг без ISBN

Обогащение категориями:
  "Война и мир" → "Военная литература"
  "Java: полное руководство" → "IT"
  "Преступление и наказание" → "Художественная литература"

Результат: 72 книги
Статистика по категориям:
  IT: 12
  Художественная литература: 25
  Военная литература: 8
  Детектив: 5
  Фэнтези: 10
  Другое: 12
```

### Применение ИИ-инструмента (цепочка промптов)

**Промпт 1:**
```
Спроектируй DataSourceReader для Java, который умеет читать данные из CSV и JSON и преобразовывать их в List<Book>.

Требования:
1. Использовать паттерн «Стратегия».
2. Поддержка CSV (с учётом кавычек).
3. Поддержка JSON (с использованием Jackson).
4. Обработка ошибок при чтении.
5. Логирование процесса.
```

**Промпт 2:**
```
На основе предыдущего дизайна напиши метод нормализации имени автора на Java.

Входные данные (могут быть в разных форматах):
- "Толстой Л.Н." → "Толстой Л.Н."
- "Л.Н. Толстой" → "Толстой Л.Н."
- "Толстой Лев Николаевич" → "Толстой Л.Н."
- "Толстой Л. Н." → "Толстой Л.Н."
- "Лев Николаевич Толстой" → "Толстой Л.Н."

Выходной формат: "Фамилия И.О."
```

**Промпт 3:**
```
На основе предыдущих результатов напиши метод обогащения книг категориями на Java.

Справочник categories.csv:
keyword,category
война,Военная литература
программирование,IT
java,IT
детектив,Детектив
фэнтези,Фэнтези
наука,Научная литература
история,Историческая литература
роман,Художественная литература

Метод должен:
1. Загружать справочник из CSV.
2. Искать ключевые слова в названии книги.
3. Добавлять поле category.
4. Возвращать обогащённый объект Book.
```

---

## 4. Индивидуальные задания (30 вариантов)

Каждый вариант содержит:
- Источник данных (CSV и JSON).
- Специфические правила нормализации.
- Специфические правила фильтрации.
- Справочник для обогащения.

---

### Вариант 1. Книги (Book)

**Источники:** CSV + JSON с книгами

**Нормализация:** автор → "Фамилия И.О."

**Фильтрация:** исключить без ISBN, год ∈ [1, 2026]

**Обогащение:** категория по ключевым словам в названии

---

### Вариант 2. Сотрудники (Employee)

**Источники:** CSV + JSON с сотрудниками

**Нормализация:** ФИО → "Фамилия И.О." (разделение на части)

**Фильтрация:** исключить с зарплатой ≤ 0, email не пустой

**Обогащение:** отдел по ключевым словам в должности

---

### Вариант 3. Товары (Product)

**Источники:** CSV + JSON с товарами

**Нормализация:** название → первая буква заглавная, остальные строчные

**Фильтрация:** исключить с ценой ≤ 0, количество ≥ 0

**Обогащение:** категория по SKU (первые 3 символа)

---

### Вариант 4. Заказы (Order)

**Источники:** CSV + JSON с заказами

**Нормализация:** статус → стандартизация (NEW → NEW, IN_PROGRESS → PROCESSING)

**Фильтрация:** исключить с суммой ≤ 0, дата не пустая

**Обогащение:** приоритет по сумме заказа

---

### Вариант 5. Пользователи (User)

**Источники:** CSV + JSON с пользователями

**Нормализация:** email → нижний регистр, удаление пробелов

**Фильтрация:** исключить без email, без роли

**Обогащение:** уровень доступа по роли

---

### Вариант 6. Автомобили (Car)

**Источники:** CSV + JSON с автомобилями

**Нормализация:** VIN → верхний регистр, удаление пробелов

**Фильтрация:** исключить с ценой ≤ 0, год ∈ [1886, 2026]

**Обогащение:** класс автомобиля по цене

---

### Вариант 7. Студенты (Student)

**Источники:** CSV + JSON со студентами

**Нормализация:** ФИО → "Фамилия И.О."

**Фильтрация:** исключить с GPA < 2.0, группа не пустая

**Обогащение:** стипендия по GPA

---

### Вариант 8. Счета (Account)

**Источники:** CSV + JSON со счетами

**Нормализация:** номер счёта → формат XXXX-XXXX-XXXX-XXXX

**Фильтрация:** исключить с балансом < 0, валюта не пустая

**Обогащение:** тип счёта по балансу

---

### Вариант 9. Фильмы (Movie)

**Источники:** CSV + JSON с фильмами

**Нормализация:** рейтинг → округление до 1 знака

**Фильтрация:** исключить с рейтингом < 0 или > 10, год ∈ [1890, 2026]

**Обогащение:** категория по жанру

---

### Вариант 10. Рестораны (Restaurant)

**Источники:** CSV + JSON с ресторанами

**Нормализация:** телефон → формат +7 (XXX) XXX-XX-XX

**Фильтрация:** исключить с рейтингом < 0 или > 5

**Обогащение:** класс по рейтингу

---

### Вариант 11. Транзакции (Transaction)

**Источники:** CSV + JSON с транзакциями

**Нормализация:** тип → UPPER CASE

**Фильтрация:** исключить с суммой = 0, дата не пустая

**Обогащение:** категория по описанию

---

### Вариант 12. Клиенты (Customer)

**Источники:** CSV + JSON с клиентами

**Нормализация:** телефон → формат +7 (XXX) XXX-XX-XX

**Фильтрация:** исключить без email, без телефона

**Обогащение:** сегмент по возрасту

---

### Вариант 13. Договоры (Contract)

**Источники:** CSV + JSON с договорами

**Нормализация:** номер → формат К-XXXX-YYYY

**Фильтрация:** исключить с суммой ≤ 0, дата не пустая

**Обогащение:** статус по дате окончания

---

### Вариант 14. Отели (Hotel)

**Источники:** CSV + JSON с отелями

**Нормализация:** адрес → стандартизация (ул., д., кв.)

**Фильтрация:** исключить со звёздами < 1 или > 5, комнат > 0

**Обогащение:** класс по звёздам

---

### Вариант 15. Спортсмены (Athlete)

**Источники:** CSV + JSON со спортсменами

**Нормализация:** ФИО → "Фамилия И.О."

**Фильтрация:** исключить с возрастом < 6 или > 100

**Обогащение:** уровень по медалям

---

### Вариант 16. Статьи (Article)

**Источники:** CSV + JSON со статьями

**Нормализация:** дата → формат yyyy-MM-dd

**Фильтрация:** исключить без автора, без заголовка

**Обогащение:** тема по ключевым словам

---

### Вариант 17. Билеты (Ticket)

**Источники:** CSV + JSON с билетами

**Нормализация:** дата → формат yyyy-MM-dd HH:mm

**Фильтрация:** исключить с ценой ≤ 0, мест ≤ 0

**Обогащение:** статус по дате мероприятия

---

### Вариант 18. Курсы (Course)

**Источники:** CSV + JSON с курсами

**Нормализация:** дата → формат yyyy-MM-dd

**Фильтрация:** исключить с ценой < 0, длительность ≤ 0

**Обогащение:** уровень по цене

---

### Вариант 19. Инвентарь (Item)

**Источники:** CSV + JSON с инвентарём

**Нормализация:** название → первая буква заглавная

**Фильтрация:** исключить с количеством < 0, без категории

**Обогащение:** статус по количеству

---

### Вариант 20. Заявки (Request)

**Источники:** CSV + JSON с заявками

**Нормализация:** приоритет → UPPER CASE

**Фильтрация:** исключить без описания, без клиента

**Обогащение:** срочность по приоритету

---

### Вариант 21. Платежи (Payment)

**Источники:** CSV + JSON с платежами

**Нормализация:** номер → формат П-XXXX-YYYY

**Фильтрация:** исключить с суммой ≤ 0

**Обогащение:** статус по сумме

---

### Вариант 22. Публикации (Publication)

**Источники:** CSV + JSON с публикациями

**Нормализация:** DOI → верхний регистр

**Фильтрация:** исключить без DOI, год ∈ [1900, 2026]

**Обогащение:** тип по журналу

---

### Вариант 23. Ноутбуки (Laptop)

**Источники:** CSV + JSON с ноутбуками

**Нормализация:** RAM → формат X GB

**Фильтрация:** исключить с ценой ≤ 0, RAM < 4

**Обогащение:** класс по цене

---

### Вариант 24. Врачи (Doctor)

**Источники:** CSV + JSON с врачами

**Нормализация:** ФИО → "Фамилия И.О."

**Фильтрация:** исключить с опытом < 0, без специализации

**Обогащение:** категория по опыту

---

### Вариант 25. Сертификаты (Certificate)

**Источники:** CSV + JSON с сертификатами

**Нормализация:** номер → формат C-XXXX-YYYY

**Фильтрация:** исключить с пустой датой выдачи

**Обогащение:** статус по сроку действия

---

### Вариант 26. Магазины (Shop)

**Источники:** CSV + JSON с магазинами

**Нормализация:** телефон → формат +7 (XXX) XXX-XX-XX

**Фильтрация:** исключить без адреса, без телефона

**Обогащение:** тип по названию

---

### Вариант 27. Здания (Building)

**Источники:** CSV + JSON с зданиями

**Нормализация:** адрес → стандартизация

**Фильтрация:** исключить с этажами ≤ 0, площадью ≤ 0

**Обогащение:** класс по этажности

---

### Вариант 28. Лекции (Lecture)

**Источники:** CSV + JSON с лекциями

**Нормализация:** дата → формат yyyy-MM-dd

**Фильтрация:** исключить с длительностью ≤ 0

**Обогащение:** уровень по количеству слушателей

---

### Вариант 29. Отзывы (Review)

**Источники:** CSV + JSON с отзывами

**Нормализация:** рейтинг → округление до целого

**Фильтрация:** исключить с рейтингом < 1 или > 5

**Обогащение:** тон по рейтингу

---

### Вариант 30. Чеки (Receipt)

**Источники:** CSV + JSON с чеками

**Нормализация:** номер → формат Ч-XXXX-YYYY

**Фильтрация:** исключить с суммой ≤ 0

**Обогащение:** тип по сумме

---

## 5. Методические указания

### 5.1. Структура проекта

```
src/
├── main/
│   ├── java/
│   │   ├── model/
│   │   │   └── Book.java
│   │   ├── etl/
│   │   │   ├── reader/
│   │   │   │   ├── DataSourceReader.java
│   │   │   │   ├── ParserStrategy.java
│   │   │   │   ├── CsvParserStrategy.java
│   │   │   │   └── JsonParserStrategy.java
│   │   │   ├── transform/
│   │   │   │   ├── AuthorNormalizer.java
│   │   │   │   ├── BookFilter.java
│   │   │   │   └── CategoryEnricher.java
│   │   │   └── pipeline/
│   │   │       └── EtlPipeline.java
│   │   └── Main.java
│   └── resources/
│       ├── books.csv
│       ├── books.json
│       └── categories.csv
└── test/
    └── java/
        └── etl/
            ├── reader/
            └── transform/
```

### 5.2. Интерфейс стратегии

```java
package etl.reader;

import model.Book;
import java.io.IOException;
import java.util.List;

public interface ParserStrategy {
    /**
     * Парсит файл и возвращает список книг
     * @param filePath путь к файлу
     * @return список книг
     * @throws IOException при ошибке чтения
     */
    List<Book> parse(String filePath) throws IOException;
    
    /**
     * Возвращает поддерживаемый формат
     * @return название формата
     */
    String getFormat();
}
```

### 5.3. Реализация CSV стратегии

```java
package etl.reader;

import model.Book;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.ArrayList;
import java.util.List;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class CsvParserStrategy implements ParserStrategy {
    private static final Logger logger = LoggerFactory.getLogger(CsvParserStrategy.class);
    private static final Pattern CSV_PATTERN = 
        Pattern.compile("(?:^|,)(?:\"([^\"]*)\"|([^,]*))");
    
    @Override
    public List<Book> parse(String filePath) throws IOException {
        logger.info("Парсинг CSV-файла: {}", filePath);
        List<Book> books = new ArrayList<>();
        int lineNumber = 0;
        int errors = 0;
        
        try (BufferedReader reader = new BufferedReader(
                new FileReader(filePath, StandardCharsets.UTF_8))) {
            
            String line;
            boolean isHeader = true;
            
            while ((line = reader.readLine()) != null) {
                lineNumber++;
                
                if (isHeader) {
                    isHeader = false;
                    continue;
                }
                
                try {
                    Book book = parseLine(line);
                    if (book != null) {
                        books.add(book);
                    }
                } catch (Exception e) {
                    errors++;
                    logger.warn("Ошибка парсинга строки {}: {}", lineNumber, e.getMessage());
                }
            }
        }
        
        logger.info("CSV-парсинг завершён: {} книг, ошибок: {}", books.size(), errors);
        return books;
    }
    
    private Book parseLine(String line) {
        List<String> fields = new ArrayList<>();
        Matcher m = CSV_PATTERN.matcher(line);
        while (m.find()) {
            String field = m.group(1) != null ? m.group(1) : m.group(2);
            fields.add(field != null ? field : "");
        }
        
        if (fields.size() < 6) {
            throw new IllegalArgumentException("Недостаточно полей: " + fields.size());
        }
        
        try {
            long id = Long.parseLong(fields.get(0).trim());
            String title = fields.get(1).trim();
            String author = fields.get(2).trim();
            int year = Integer.parseInt(fields.get(3).trim());
            String isbn = fields.get(4).trim();
            String publisher = fields.get(5).trim();
            
            return new Book(id, title, author, year, isbn, publisher);
        } catch (NumberFormatException e) {
            throw new IllegalArgumentException("Ошибка парсинга числа", e);
        }
    }
    
    @Override
    public String getFormat() {
        return "CSV";
    }
}
```

### 5.4. Реализация JSON стратегии

```java
package etl.reader;

import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializationFeature;
import com.fasterxml.jackson.datatype.jsr310.JavaTimeModule;
import model.Book;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.File;
import java.io.IOException;
import java.util.List;

public class JsonParserStrategy implements ParserStrategy {
    private static final Logger logger = LoggerFactory.getLogger(JsonParserStrategy.class);
    private static final ObjectMapper mapper = createObjectMapper();
    
    private static ObjectMapper createObjectMapper() {
        ObjectMapper mapper = new ObjectMapper();
        mapper.registerModule(new JavaTimeModule());
        mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
        mapper.enable(SerializationFeature.INDENT_OUTPUT);
        return mapper;
    }
    
    @Override
    public List<Book> parse(String filePath) throws IOException {
        logger.info("Парсинг JSON-файла: {}", filePath);
        File file = new File(filePath);
        
        if (!file.exists()) {
            throw new IOException("Файл не найден: " + filePath);
        }
        
        try {
            List<Book> books = mapper.readValue(file, new TypeReference<List<Book>>() {});
            logger.info("JSON-парсинг завершён: {} книг", books.size());
            return books;
        } catch (IOException e) {
            logger.error("Ошибка парсинга JSON: {}", e.getMessage());
            throw e;
        }
    }
    
    @Override
    public String getFormat() {
        return "JSON";
    }
}
```

### 5.5. DataSourceReader

```java
package etl.reader;

import model.Book;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.IOException;
import java.util.List;

public class DataSourceReader {
    private static final Logger logger = LoggerFactory.getLogger(DataSourceReader.class);
    private ParserStrategy strategy;
    
    public DataSourceReader(ParserStrategy strategy) {
        this.strategy = strategy;
    }
    
    public void setStrategy(ParserStrategy strategy) {
        this.strategy = strategy;
    }
    
    public List<Book> read(String filePath) throws IOException {
        logger.info("Чтение данных из: {} (формат: {})", filePath, strategy.getFormat());
        
        try {
            List<Book> books = strategy.parse(filePath);
            logger.info("Успешно прочитано {} книг", books.size());
            return books;
        } catch (IOException e) {
            logger.error("Ошибка чтения файла: {}", filePath, e);
            throw e;
        }
    }
}
```

### 5.6. AuthorNormalizer

```java
package etl.transform;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.regex.Pattern;

public class AuthorNormalizer {
    private static final Logger logger = LoggerFactory.getLogger(AuthorNormalizer.class);
    
    // Регулярные выражения для различных форматов
    private static final Pattern SURNAME_INITIALS = 
        Pattern.compile("^([А-ЯЁ][а-яё]+)\\s+([А-ЯЁ])\\.([А-ЯЁ])\\.?$");
    private static final Pattern INITIALS_SURNAME = 
        Pattern.compile("^([А-ЯЁ])\\.([А-ЯЁ])\\.?\\s+([А-ЯЁ][а-яё]+)$");
    private static final Pattern SURNAME_FULL = 
        Pattern.compile("^([А-ЯЁ][а-яё]+)\\s+([А-ЯЁ][а-яё]+)\\s+([А-ЯЁ][а-яё]+)$");
    private static final Pattern FULL_SURNAME = 
        Pattern.compile("^([А-ЯЁ][а-яё]+)\\s+([А-ЯЁ][а-яё]+)\\s+([А-ЯЁ][а-яё]+)$");
    
    /**
     * Нормализация имени автора к формату "Фамилия И.О."
     */
    public String normalize(String input) {
        if (input == null || input.trim().isEmpty()) {
            logger.warn("Пустое имя автора");
            return "Unknown";
        }
        
        // Удаление лишних пробелов
        String normalized = input.trim().replaceAll("\\s+", " ");
        logger.debug("Нормализация: '{}' →", input);
        
        // Формат: "Фамилия И.О."
        Matcher m1 = SURNAME_INITIALS.matcher(normalized);
        if (m1.matches()) {
            String result = m1.group(1) + " " + m1.group(2) + "." + m1.group(3) + ".";
            logger.debug("  '{}' → '{}' (формат: Фамилия И.О.)", input, result);
            return result;
        }
        
        // Формат: "И.О. Фамилия"
        Matcher m2 = INITIALS_SURNAME.matcher(normalized);
        if (m2.matches()) {
            String result = m2.group(3) + " " + m2.group(1) + "." + m2.group(2) + ".";
            logger.debug("  '{}' → '{}' (формат: И.О. Фамилия)", input, result);
            return result;
        }
        
        // Формат: "Фамилия Имя Отчество"
        Matcher m3 = SURNAME_FULL.matcher(normalized);
        if (m3.matches()) {
            String surname = m3.group(1);
            String initial1 = String.valueOf(m3.group(2).charAt(0));
            String initial2 = String.valueOf(m3.group(3).charAt(0));
            String result = surname + " " + initial1 + "." + initial2 + ".";
            logger.debug("  '{}' → '{}' (формат: Фамилия Имя Отчество)", input, result);
            return result;
        }
        
        // Формат: "Имя Отчество Фамилия"
        Matcher m4 = FULL_SURNAME.matcher(normalized);
        if (m4.matches()) {
            String surname = m4.group(3);
            String initial1 = String.valueOf(m4.group(1).charAt(0));
            String initial2 = String.valueOf(m4.group(2).charAt(0));
            String result = surname + " " + initial1 + "." + initial2 + ".";
            logger.debug("  '{}' → '{}' (формат: Имя Отчество Фамилия)", input, result);
            return result;
        }
        
        // Если формат не распознан, возвращаем как есть
        logger.debug("  '{}' → '{}' (формат не распознан)", input, normalized);
        return normalized;
    }
}
```

### 5.7. CategoryEnricher

```java
package etl.transform;

import model.Book;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.HashMap;
import java.util.Map;

public class CategoryEnricher {
    private static final Logger logger = LoggerFactory.getLogger(CategoryEnricher.class);
    private final Map<String, String> categoryMap = new HashMap<>();
    
    public CategoryEnricher(String categoryFilePath) throws IOException {
        loadCategories(categoryFilePath);
        logger.info("Загружено {} категорий", categoryMap.size());
    }
    
    private void loadCategories(String filePath) throws IOException {
        logger.info("Загрузка справочника категорий: {}", filePath);
        
        try (BufferedReader reader = new BufferedReader(
                new FileReader(filePath, StandardCharsets.UTF_8))) {
            
            String line;
            boolean isHeader = true;
            
            while ((line = reader.readLine()) != null) {
                if (isHeader) {
                    isHeader = false;
                    continue;
                }
                
                String[] parts = line.split(",");
                if (parts.length >= 2) {
                    String keyword = parts[0].trim().toLowerCase();
                    String category = parts[1].trim();
                    categoryMap.put(keyword, category);
                    logger.debug("  '{}' → '{}'", keyword, category);
                }
            }
        }
    }
    
    /**
     * Обогащает книгу категорией на основе ключевых слов в названии
     */
    public Book enrich(Book book) {
        if (book == null || book.getTitle() == null || book.getTitle().isEmpty()) {
            book.setCategory("Unknown");
            return book;
        }
        
        String titleLower = book.getTitle().toLowerCase();
        String category = "Other";
        
        for (Map.Entry<String, String> entry : categoryMap.entrySet()) {
            if (titleLower.contains(entry.getKey())) {
                category = entry.getValue();
                logger.debug("  '{}' → категория '{}' (ключевое слово: '{}')", 
                    book.getTitle(), category, entry.getKey());
                break;
            }
        }
        
        book.setCategory(category);
        return book;
    }
}
```

### 5.8. BookFilter

```java
package etl.transform;

import model.Book;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.time.Year;

public class BookFilter {
    private static final Logger logger = LoggerFactory.getLogger(BookFilter.class);
    
    public boolean isValid(Book book) {
        if (book == null) {
            return false;
        }
        
        // Проверка ISBN
        if (book.getIsbn() == null || book.getIsbn().trim().isEmpty()) {
            logger.debug("Книга '{}' исключена: отсутствует ISBN", book.getTitle());
            return false;
        }
        
        // Проверка года
        int currentYear = Year.now().getValue();
        if (book.getYear() <= 0 || book.getYear() > currentYear) {
            logger.debug("Книга '{}' исключена: неверный год {}", book.getTitle(), book.getYear());
            return false;
        }
        
        // Проверка названия
        if (book.getTitle() == null || book.getTitle().trim().isEmpty()) {
            logger.debug("Книга исключена: отсутствует название");
            return false;
        }
        
        return true;
    }
}
```

### 5.9. ETL Pipeline

```java
package etl.pipeline;

import etl.reader.DataSourceReader;
import etl.reader.ParserStrategy;
import etl.transform.AuthorNormalizer;
import etl.transform.BookFilter;
import etl.transform.CategoryEnricher;
import model.Book;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.IOException;
import java.util.ArrayList;
import java.util.List;
import java.util.stream.Collectors;

public class EtlPipeline {
    private static final Logger logger = LoggerFactory.getLogger(EtlPipeline.class);
    
    private final DataSourceReader reader;
    private final AuthorNormalizer authorNormalizer;
    private final BookFilter filter;
    private final CategoryEnricher enricher;
    
    public EtlPipeline(DataSourceReader reader, AuthorNormalizer authorNormalizer,
                       BookFilter filter, CategoryEnricher enricher) {
        this.reader = reader;
        this.authorNormalizer = authorNormalizer;
        this.filter = filter;
        this.enricher = enricher;
    }
    
    public EtlResult run(String filePath, ParserStrategy strategy) throws IOException {
        logger.info("=== Запуск ETL-пайплайна ===");
        long startTime = System.currentTimeMillis();
        
        // 1. EXTRACT
        logger.info("=== EXTRACT ===");
        reader.setStrategy(strategy);
        List<Book> extractedBooks = reader.read(filePath);
        logger.info("Извлечено книг: {}", extractedBooks.size());
        
        // 2. TRANSFORM
        logger.info("=== TRANSFORM ===");
        
        // 2.1 Нормализация авторов
        List<Book> normalizedBooks = new ArrayList<>();
        for (Book book : extractedBooks) {
            String normalizedAuthor = authorNormalizer.normalize(book.getAuthor());
            book.setAuthor(normalizedAuthor);
            normalizedBooks.add(book);
        }
        logger.info("Нормализовано книг: {}", normalizedBooks.size());
        
        // 2.2 Фильтрация
        List<Book> filteredBooks = normalizedBooks.stream()
            .filter(filter::isValid)
            .collect(Collectors.toList());
        int filteredCount = normalizedBooks.size() - filteredBooks.size();
        logger.info("Отфильтровано книг: {}", filteredCount);
        logger.info("Осталось книг: {}", filteredBooks.size());
        
        // 2.3 Обогащение категориями
        List<Book> enrichedBooks = new ArrayList<>();
        for (Book book : filteredBooks) {
            enrichedBooks.add(enricher.enrich(book));
        }
        logger.info("Обогащено книг: {}", enrichedBooks.size());
        
        // 3. Статистика
        long duration = System.currentTimeMillis() - startTime;
        logger.info("=== ETL завершён ===");
        logger.info("Время выполнения: {} мс", duration);
        
        return new EtlResult(
            extractedBooks.size(),
            normalizedBooks.size(),
            filteredBooks.size(),
            enrichedBooks.size(),
            duration,
            enrichedBooks
        );
    }
}
```

### 5.10. EtlResult

```java
package etl.pipeline;

import model.Book;
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

public class EtlResult {
    private final int extracted;
    private final int normalized;
    private final int filtered;
    private final int loaded;
    private final long durationMs;
    private final List<Book> books;
    
    public EtlResult(int extracted, int normalized, int filtered, 
                     int loaded, long durationMs, List<Book> books) {
        this.extracted = extracted;
        this.normalized = normalized;
        this.filtered = filtered;
        this.loaded = loaded;
        this.durationMs = durationMs;
        this.books = books;
    }
    
    public int getExtracted() { return extracted; }
    public int getNormalized() { return normalized; }
    public int getFiltered() { return filtered; }
    public int getLoaded() { return loaded; }
    public long getDurationMs() { return durationMs; }
    public List<Book> getBooks() { return books; }
    
    public Map<String, Long> getCategoryStats() {
        return books.stream()
            .collect(Collectors.groupingBy(
                Book::getCategory,
                Collectors.counting()
            ));
    }
    
    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder();
        sb.append("=== Результат ETL ===\n");
        sb.append("Извлечено (Extract): ").append(extracted).append("\n");
        sb.append("Нормализовано: ").append(normalized).append("\n");
        sb.append("Отфильтровано: ").append(extracted - filtered).append("\n");
        sb.append("Загружено (Load): ").append(loaded).append("\n");
        sb.append("Время: ").append(durationMs).append(" мс\n");
        
        sb.append("\nСтатистика по категориям:\n");
        Map<String, Long> stats = getCategoryStats();
        stats.entrySet().stream()
            .sorted(Map.Entry.<String, Long>comparingByValue().reversed())
            .forEach(e -> sb.append("  ").append(e.getKey())
                .append(": ").append(e.getValue()).append("\n"));
        
        return sb.toString();
    }
}
```

### 5.11. Основной класс

```java
import etl.pipeline.EtlPipeline;
import etl.pipeline.EtlResult;
import etl.reader.CsvParserStrategy;
import etl.reader.DataSourceReader;
import etl.reader.JsonParserStrategy;
import etl.transform.AuthorNormalizer;
import etl.transform.BookFilter;
import etl.transform.CategoryEnricher;

import java.io.IOException;

public class Main {
    public static void main(String[] args) {
        try {
            // Инициализация компонентов
            DataSourceReader reader = new DataSourceReader(new CsvParserStrategy());
            AuthorNormalizer normalizer = new AuthorNormalizer();
            BookFilter filter = new BookFilter();
            CategoryEnricher enricher = new CategoryEnricher("src/main/resources/categories.csv");
            
            EtlPipeline pipeline = new EtlPipeline(reader, normalizer, filter, enricher);
            
            // Запуск ETL для CSV
            System.out.println("\n=== ETL для CSV ===\n");
            EtlResult csvResult = pipeline.run(
                "src/main/resources/books.csv",
                new CsvParserStrategy()
            );
            System.out.println(csvResult);
            
            // Запуск ETL для JSON
            System.out.println("\n=== ETL для JSON ===\n");
            EtlResult jsonResult = pipeline.run(
                "src/main/resources/books.json",
                new JsonParserStrategy()
            );
            System.out.println(jsonResult);
            
        } catch (IOException e) {
            System.err.println("Ошибка: " + e.getMessage());
            e.printStackTrace();
        }
    }
}
```

---

## 6. Контрольные вопросы

1. Что такое ETL и для чего он используется?

2. Перечислите этапы ETL и опишите каждый.

3. Что такое паттерн «Стратегия» и как он применяется в ETL?

4. Какие форматы данных могут использоваться на этапе Extract?

5. Что такое нормализация данных? Приведите пример.

6. Какие методы фильтрации данных вы знаете?

7. Что такое обогащение данных? Приведите пример.

8. Как обрабатывать ошибки при чтении данных?

9. Что такое цепочка промптов и как она применяется при проектировании?

10. Какие преимущества даёт использование паттерна «Стратегия»?

11. Как организовать логирование в ETL-пайплайне?

12. Что такое идемпотентность в контексте ETL?

13. Как проверять качество данных на этапе Transform?

14. Какие ещё преобразования можно выполнять на этапе Transform?

15. Как обеспечить гибкость ETL-пайплайна при добавлении новых источников?

---

## 7. Рекомендуемые источники

1. **Kimball R., Caserta J.** *The Data Warehouse ETL Toolkit.* — Wiley.

2. **Шилдт Г.** *Java. Базовый курс.* — М.: Вильямс. — Глава 15 (Ввод-вывод).

3. **Хорстманн К., Корнелл Г.** *Java. Библиотека профессионала. Том 2.* — М.: Вильямс. — Глава 2 (Ввод-вывод).

4. **Patterns: Strategy Pattern.** — URL: https://refactoring.guru/design-patterns/strategy

5. **Jackson Documentation.** — URL: https://github.com/FasterXML/jackson-docs

6. **OpenCSV Documentation.** — URL: http://opencsv.sourceforge.net/
