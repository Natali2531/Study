# Лабораторная работа №6. Исключения и логирование

## 1. Паспорт работы

| Параметр | Значение |
|----------|----------|
| Номер занятия | 6 из 17 |
| Блок | 2. Коллекции, обобщения и исключения |
| Продолжительность | 2 академических часа |
| Форма выполнения | Индивидуальная |
| ИИ-инструмент | YandexGPT / GigaChat |

### 1.1. Цель работы

Освоить механизм обработки исключений, научиться создавать собственные классы исключений, применять библиотеку логирования SLF4J + Logback.

## 2. Теоретический конспект

### 2.1. Иерархия исключений

```
Throwable
├── Error (системные ошибки)
└── Exception
    ├── IOException (checked)
    └── RuntimeException (unchecked)
        ├── NullPointerException
        └── IllegalArgumentException
```

### 2.2. Checked vs Unchecked

**Checked** — компилятор требует обработки (`try-catch` или `throws`).
**Unchecked** — обработка не требуется.

### 2.3. Создание собственных исключений

```java
public class BookNotFoundException extends Exception {
    private final long bookId;

    public BookNotFoundException(long bookId) {
        super("Книга с ID " + bookId + " не найдена");
        this.bookId = bookId;
    }

    public long getBookId() { return bookId; }
}
```

### 2.4. Логирование с SLF4J + Logback

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class BookRepository {
    private static final Logger logger = LoggerFactory.getLogger(BookRepository.class);

    public void save(Book book) {
        logger.info("Сохранение книги: {}", book.getTitle());
    }
}
```

## 3. Задание на паре

1. Создать checked-исключение `BookNotFoundException` и unchecked `InvalidBookDataException`.
2. В `Repository<Book>` выбрасывать `BookNotFoundException` при отсутствии книги в `findById`.
3. В сеттерах `Book` валидировать входные данные — при нарушении бросать `InvalidBookDataException`.
4. Подключить SLF4J + Logback. Расставить логирование: INFO, WARN, ERROR.
5. Написать `try-catch-finally` и `try-with-resources` для работы с файлами.

## 4. Индивидуальные задания (30 вариантов)

Каждый вариант основан на классе из лабораторной работы №1. Необходимо:

1. Создать checked-исключение `XxxNotFoundException` (с полем идентификатора).
2. Создать unchecked-исключение `InvalidXxxDataException` (для ошибок валидации).
3. В `Repository<Xxx>` выбрасывать `XxxNotFoundException` при отсутствии объекта.
4. В сеттерах валидировать данные, бросая `InvalidXxxDataException`.
5. Подключить SLF4J + Logback, расставить логирование INFO/WARN/ERROR.

**Варианты 1-30** соответствуют вариантам из лабораторной работы №1.

**Пример для варианта 1 (Студент):**
- `StudentNotFoundException` (checked) — с полем `studentId`.
- `InvalidStudentDataException` (unchecked) — для некорректного возраста, среднего балла.
- Логирование: INFO при сохранении, WARN при поиске несуществующего, ERROR при некорректных данных.

## 5. Контрольные вопросы

1. Чем checked-исключение отличается от unchecked?
2. Как создать собственное исключение?
3. Что такое `try-with-resources`?
4. Зачем нужно логирование? Какие уровни логирования существуют?
5. Что такое `finally`? Всегда ли он выполняется?

## 6. Рекомендуемые источники
