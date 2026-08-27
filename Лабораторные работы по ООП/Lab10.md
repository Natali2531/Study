# Лабораторная работа №10. Многопоточность: ExecutorService и Future

## 1. Паспорт работы

| Параметр | Значение |
|----------|----------|
| Номер занятия | 10 из 17 |
| Блок | 4. Многопоточность |
| Продолжительность | 2 академических часа |
| Форма выполнения | Индивидуальная |
| ИИ-инструмент | YandexGPT / GigaChat / JetBrains AI Assistant |

### 1.1. Цель работы

Освоить высокоуровневые примитивы для работы с многопоточностью в Java: пулы потоков (ExecutorService), асинхронное выполнение задач с возвратом результата (Callable и Future), научиться реализовывать параллельную обработку данных с контролем времени выполнения.

### 1.2. Задачи работы

1. Изучить ExecutorService и пулы потоков.
2. Освоить интерфейсы Callable и Future для асинхронного выполнения задач с возвратом результата.
3. Научиться собирать результаты параллельной работы.
4. Освоить установку таймаутов для задач.
5. Изучить отмену задач (cancel()).
6. Научиться сравнивать производительность последовательной и параллельной обработки.
7. Развить навыки обработки исключений в многопоточных приложениях.

### 1.3. Оснащение

- JDK 17 или выше;
- IntelliJ IDEA Community Edition;
- Git;
- доступ к YandexGPT или GigaChat.

---

## 2. Теоретический конспект

### 2.1. ExecutorService и пулы потоков

**ExecutorService** — высокоуровневый интерфейс для управления пулами потоков.

**Преимущества перед Thread:**
- Переиспользование потоков (экономия ресурсов).
- Управление жизненным циклом потоков.
- Ограничение количества одновременно работающих потоков.
- Удобный API для асинхронного выполнения задач.

**Создание пулов потоков:**

```java
// Фиксированный пул (фиксированное количество потоков)
ExecutorService fixedPool = Executors.newFixedThreadPool(4);

// Кеширующий пул (создаёт потоки по мере необходимости)
ExecutorService cachedPool = Executors.newCachedThreadPool();

// Одиночный поток (выполняет задачи последовательно)
ExecutorService singlePool = Executors.newSingleThreadExecutor();

// Пул с планированием (для отложенных и периодических задач)
ScheduledExecutorService scheduledPool = Executors.newScheduledThreadPool(4);

// ForkJoinPool (для рекурсивных задач)
ForkJoinPool forkJoinPool = new ForkJoinPool(4);

// Пул с привязкой к виртуальным потокам (Java 21+)
ExecutorService virtualPool = Executors.newVirtualThreadPerTaskExecutor();
```

### 2.2. Runnable vs Callable

| Характеристика | Runnable | Callable |
|----------------|----------|----------|
| Возвращаемое значение | void | V (любой тип) |
| Исключения | Не может пробрасывать checked исключения | Может пробрасывать любые исключения |
| Метод | `void run()` | `V call() throws Exception` |
| Использование | `executor.execute(runnable)` | `executor.submit(callable)` |

**Пример Callable:**

```java
Callable<Integer> task = () -> {
    Thread.sleep(1000);
    return 42;
};

Future<Integer> future = executor.submit(task);
Integer result = future.get(); // Блокирует до получения результата
```

### 2.3. Future

**Future** — интерфейс, представляющий результат асинхронной операции.

**Основные методы:**

```java
// Проверка завершения задачи
boolean isDone();

// Проверка отмены задачи
boolean isCancelled();

// Ожидание результата (блокирующий)
V get() throws InterruptedException, ExecutionException;

// Ожидание результата с таймаутом
V get(long timeout, TimeUnit unit) 
    throws InterruptedException, ExecutionException, TimeoutException;

// Отмена задачи
boolean cancel(boolean mayInterruptIfRunning);
```

**Пример использования:**

```java
ExecutorService executor = Executors.newFixedThreadPool(4);

List<Future<Integer>> futures = new ArrayList<>();
for (int i = 0; i < 10; i++) {
    final int value = i;
    Callable<Integer> task = () -> {
        Thread.sleep(1000);
        return value * value;
    };
    futures.add(executor.submit(task));
}

// Сбор результатов
int sum = 0;
for (Future<Integer> future : futures) {
    try {
        // Ожидание с таймаутом
        Integer result = future.get(5, TimeUnit.SECONDS);
        sum += result;
    } catch (TimeoutException e) {
        future.cancel(true);
        System.err.println("Задача не завершилась за 5 секунд");
    }
}

executor.shutdown();
```

### 2.4. Пул потоков и управление ресурсами

**Важные методы ExecutorService:**

```java
// Плавное завершение (ожидание завершения всех задач)
executor.shutdown();
executor.awaitTermination(30, TimeUnit.SECONDS);

// Принудительное завершение
List<Runnable> notExecuted = executor.shutdownNow();

// Проверка статуса
boolean isShutdown = executor.isShutdown();
boolean isTerminated = executor.isTerminated();
```

### 2.5. Обработка исключений

**Исключения в Callable:**

```java
Callable<Integer> task = () -> {
    if (someCondition) {
        throw new IOException("Ошибка чтения файла");
    }
    return 42;
};

Future<Integer> future = executor.submit(task);
try {
    Integer result = future.get();
} catch (ExecutionException e) {
    // Причина — оригинальное исключение из task
    Throwable cause = e.getCause();
    if (cause instanceof IOException) {
        System.err.println("Ошибка ввода-вывода: " + cause.getMessage());
    }
}
```

**Обработка через try-catch в задаче:**

```java
Callable<Integer> safeTask = () -> {
    try {
        // Рискованный код
        return 42;
    } catch (Exception e) {
        logger.error("Ошибка в задаче", e);
        return null; // или пробросить исключение
    }
};
```

### 2.6. CompletableFuture (Java 8+)

Более мощная альтернатива Future с композицией задач:

```java
CompletableFuture<Integer> future = CompletableFuture
    .supplyAsync(() -> 42, executor)
    .thenApply(result -> result * 2)
    .thenApply(result -> result + 10)
    .exceptionally(throwable -> {
        System.err.println("Ошибка: " + throwable.getMessage());
        return -1;
    });

// Синхронное ожидание
Integer result = future.join(); // или future.get()

// Комбинация нескольких CompletableFuture
CompletableFuture<Integer> future1 = CompletableFuture.supplyAsync(() -> 10);
CompletableFuture<Integer> future2 = CompletableFuture.supplyAsync(() -> 20);
CompletableFuture<Integer> combined = future1
    .thenCombine(future2, (a, b) -> a + b);
```

### 2.7. CountDownLatch

Синхронизатор для ожидания завершения группы задач:

```java
CountDownLatch latch = new CountDownLatch(5);

for (int i = 0; i < 5; i++) {
    executor.submit(() -> {
        try {
            // Выполнение работы
            Thread.sleep(1000);
        } finally {
            latch.countDown(); // Уменьшаем счётчик
        }
    });
}

latch.await(); // Ожидаем, пока счётчик станет 0
System.out.println("Все задачи завершены");
```

### 2.8. Шаблон параллельной загрузки данных

```
Параллельная загрузка:
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│   ┌──────────┐    ┌──────────┐    ┌──────────┐             │
│   │ Файл 1   │    │ Файл 2   │    │ Файл 3   │             │
│   └────┬─────┘    └────┬─────┘    └────┬─────┘             │
│        │               │               │                    │
│   ┌────▼─────┐    ┌────▼─────┐    ┌────▼─────┐             │
│   │ Поток 1  │    │ Поток 2  │    │ Поток 3  │             │
│   └────┬─────┘    └────┬─────┘    └────┬─────┘             │
│        │               │               │                    │
│   ┌────▼─────┐    ┌────▼─────┐    ┌────▼─────┐             │
│   │List<Book>│    │List<Book>│    │List<Book>│             │
│   └────┬─────┘    └────┬─────┘    └────┬─────┘             │
│        └───────────────┼───────────────┘                    │
│                        ▼                                    │
│              ┌─────────────────┐                           │
│              │ Объединение и   │                           │
│              │ удаление        │                           │
│              │ дубликатов      │                           │
│              └─────────────────┘                           │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. Задание на паре

### Задача. Параллельная загрузка данных из CSV-файлов

1. **Подготовить 5 CSV-файлов с книгами:**
   - Каждый файл содержит ~10 000 строк.
   - Поля: `id,title,author,year,isbn,publisher`.

2. **Создать задачу загрузки (Callable<List<Book>>):**
   - Чтение CSV-файла.
   - Парсинг и создание объектов Book.
   - Возврат списка книг.

3. **Создать ExecutorService с фиксированным пулом из 4 потоков.**

4. **Загрузить все файлы параллельно:**
   - Каждый файл загружается в отдельном Callable.
   - Результаты собираются через Future.get().

5. **Объединить списки и удалить дубликаты по ISBN.**

6. **Сравнить время последовательной и параллельной загрузки.**

7. **Реализовать таймаут для Future.get():**
   - Если файл загружается дольше 30 секунд, отменять задачу.
   - Логировать ошибку.

**Пример выполнения:**
```
=== Последовательная загрузка ===
Файл 1: 10000 книг, 125 мс
Файл 2: 10000 книг, 118 мс
Файл 3: 10000 книг, 132 мс
Файл 4: 10000 книг, 121 мс
Файл 5: 10000 книг, 130 мс
Всего книг: 50000
Время: 626 мс

=== Параллельная загрузка (пул из 4 потоков) ===
Файл 1: 10000 книг, 125 мс (поток 1)
Файл 2: 10000 книг, 118 мс (поток 2)
Файл 3: 10000 книг, 132 мс (поток 3)
Файл 4: 10000 книг, 121 мс (поток 4)
Файл 5: 10000 книг, 130 мс (поток 1)
Всего книг: 50000
После удаления дубликатов: 48000
Время: 145 мс

=== Сравнение ===
Ускорение: 4.32x
```

### Применение ИИ-инструмента

**Промпт для YandexGPT:**
```
Сгенерируй обёртку для параллельной загрузки CSV-файлов в Java с использованием ExecutorService.

Требования:
1. Метод loadFiles(List<String> filePaths) возвращает List<Book>.
2. Использовать фиксированный пул потоков.
3. Каждый файл загружается в отдельном Callable.
4. Результаты собираются через Future.
5. Таймаут 30 секунд на каждый файл.
6. Обработка ошибок и логирование.
7. Удаление дубликатов по ISBN после загрузки.
```

**Анализ результата:**
- Проверить корректность использования ExecutorService.
- Проверить обработку исключений и таймаутов.
- Проверить закрытие ресурсов (shutdown()).
- Проверить обработку дубликатов.

---

## 4. Индивидуальные задания (30 вариантов)

Каждый вариант содержит:
- Количество файлов для загрузки.
- Количество потоков в пуле.
- Ожидаемое время загрузки.
- Специфические требования.

---

### Вариант 1. Книги (5 файлов, 4 потока)

**Файлы:** 5 CSV с книгами (~10 000 строк каждый)

**Потоки:** 4

**Таймаут:** 30 секунд

**Особенность:** Удаление дубликатов по ISBN

---

### Вариант 2. Товары (3 файла, 3 потока)

**Файлы:** 3 CSV с товарами (~15 000 строк каждый)

**Потоки:** 3

**Таймаут:** 20 секунд

**Особенность:** Фильтрация товаров с ценой > 0

---

### Вариант 3. Сотрудники (6 файлов, 3 потока)

**Файлы:** 6 CSV с сотрудниками (~5 000 строк каждый)

**Потоки:** 3

**Таймаут:** 25 секунд

**Особенность:** Группировка по отделам

---

### Вариант 4. Заказы (4 файла, 2 потока)

**Файлы:** 4 CSV с заказами (~20 000 строк каждый)

**Потоки:** 2

**Таймаут:** 40 секунд

**Особенность:** Вычисление общей суммы

---

### Вариант 5. Пользователи (8 файлов, 4 потока)

**Файлы:** 8 CSV с пользователями (~3 000 строк каждый)

**Потоки:** 4

**Таймаут:** 15 секунд

**Особенность:** Фильтрация активных пользователей

---

### Вариант 6. Автомобили (4 файла, 4 потока)

**Файлы:** 4 CSV с автомобилями (~12 000 строк каждый)

**Потоки:** 4

**Таймаут:** 30 секунд

**Особенность:** Группировка по бренду

---

### Вариант 7. Студенты (5 файлов, 5 потоков)

**Файлы:** 5 CSV со студентами (~8 000 строк каждый)

**Потоки:** 5

**Таймаут:** 20 секунд

**Особенность:** Вычисление среднего GPA

---

### Вариант 8. Счета (3 файла, 3 потока)

**Файлы:** 3 CSV со счетами (~20 000 строк каждый)

**Потоки:** 3

**Таймаут:** 35 секунд

**Особенность:** Подсчёт общего баланса

---

### Вариант 9. Фильмы (6 файлов, 3 потока)

**Файлы:** 6 CSV с фильмами (~6 000 строк каждый)

**Потоки:** 3

**Таймаут:** 20 секунд

**Особенность:** Фильтр по рейтингу > 7.0

---

### Вариант 10. Рестораны (4 файла, 4 потока)

**Файлы:** 4 CSV с ресторанами (~10 000 строк каждый)

**Потоки:** 4

**Таймаут:** 25 секунд

**Особенность:** Группировка по кухне

---

### Вариант 11. Транзакции (5 файлов, 3 потока)

**Файлы:** 5 CSV с транзакциями (~15 000 строк каждый)

**Потоки:** 3

**Таймаут:** 30 секунд

**Особенность:** Сумма по категориям

---

### Вариант 12. Клиенты (7 файлов, 4 потока)

**Файлы:** 7 CSV с клиентами (~4 000 строк каждый)

**Потоки:** 4

**Таймаут:** 20 секунд

**Особенность:** Фильтр по возрасту

---

### Вариант 13. Договоры (4 файла, 4 потока)

**Файлы:** 4 CSV с договорами (~12 000 строк каждый)

**Потоки:** 4

**Таймаут:** 30 секунд

**Особенность:** Активные договоры

---

### Вариант 14. Отели (5 файлов, 3 потока)

**Файлы:** 5 CSV с отелями (~8 000 строк каждый)

**Потоки:** 3

**Таймаут:** 25 секунд

**Особенность:** Отели с рейтингом > 4.0

---

### Вариант 15. Спортсмены (6 файлов, 4 потока)

**Файлы:** 6 CSV со спортсменами (~5 000 строк каждый)

**Потоки:** 4

**Таймаут:** 20 секунд

**Особенность:** Медали > 0

---

### Вариант 16. Статьи (4 файла, 2 потока)

**Файлы:** 4 CSV со статьями (~15 000 строк каждый)

**Потоки:** 2

**Таймаут:** 40 секунд

**Особенность:** Фильтр по просмотрам > 1000

---

### Вариант 17. Билеты (5 файлов, 5 потоков)

**Файлы:** 5 CSV с билетами (~10 000 строк каждый)

**Потоки:** 5

**Таймаут:** 25 секунд

**Особенность:** Вычисление выручки

---

### Вариант 18. Курсы (3 файла, 3 потока)

**Файлы:** 3 CSV с курсами (~20 000 строк каждый)

**Потоки:** 3

**Таймаут:** 35 секунд

**Особенность:** Бесплатные курсы

---

### Вариант 19. Инвентарь (6 файлов, 4 потока)

**Файлы:** 6 CSV с инвентарём (~5 000 строк каждый)

**Потоки:** 4

**Таймаут:** 20 секунд

**Особенность:** Товары с количеством > 0

---

### Вариант 20. Заявки (4 файла, 4 потока)

**Файлы:** 4 CSV с заявками (~12 000 строк каждый)

**Потоки:** 4

**Таймаут:** 30 секунд

**Особенность:** Заявки с высоким приоритетом

---

### Вариант 21. Платежи (5 файлов, 3 потока)

**Файлы:** 5 CSV с платежами (~10 000 строк каждый)

**Потоки:** 3

**Таймаут:** 25 секунд

**Особенность:** Группировка по плательщикам

---

### Вариант 22. Публикации (7 файлов, 4 потока)

**Файлы:** 7 CSV с публикациями (~4 000 строк каждый)

**Потоки:** 4

**Таймаут:** 20 секунд

**Особенность:** Публикации по годам

---

### Вариант 23. Ноутбуки (4 файла, 3 потока)

**Файлы:** 4 CSV с ноутбуками (~15 000 строк каждый)

**Потоки:** 3

**Таймаут:** 30 секунд

**Особенность:** Фильтр по цене < 50000

---

### Вариант 24. Врачи (6 файлов, 4 потока)

**Файлы:** 6 CSV с врачами (~5 000 строк каждый)

**Потоки:** 4

**Таймаут:** 20 секунд

**Особенность:** Опыт > 5 лет

---

### Вариант 25. Сертификаты (5 файлов, 3 потока)

**Файлы:** 5 CSV с сертификатами (~8 000 строк каждый)

**Потоки:** 3

**Таймаут:** 25 секунд

**Особенность:** Не истекшие сертификаты

---

### Вариант 26. Магазины (4 файла, 4 потока)

**Файлы:** 4 CSV с магазинами (~10 000 строк каждый)

**Потоки:** 4

**Таймаут:** 30 секунд

**Особенность:** Группировка по типу

---

### Вариант 27. Здания (6 файлов, 3 потока)

**Файлы:** 6 CSV с зданиями (~6 000 строк каждый)

**Потоки:** 3

**Таймаут:** 20 секунд

**Особенность:** Здания с лифтами

---

### Вариант 28. Лекции (5 файлов, 5 потоков)

**Файлы:** 5 CSV с лекциями (~8 000 строк каждый)

**Потоки:** 5

**Таймаут:** 20 секунд

**Особенность:** Лекции с > 100 слушателей

---

### Вариант 29. Отзывы (7 файлов, 4 потока)

**Файлы:** 7 CSV с отзывами (~3 000 строк каждый)

**Потоки:** 4

**Таймаут:** 15 секунд

**Особенность:** Отзывы с рейтингом > 4

---

### Вариант 30. Чеки (4 файла, 3 потока)

**Файлы:** 4 CSV с чеками (~15 000 строк каждый)

**Потоки:** 3

**Таймаут:** 35 секунд

**Особенность:** Вычисление среднего чека

---

## 5. Методические указания

### 5.1. Структура проекта

```
src/
├── main/
│   ├── java/
│   │   ├── model/
│   │   │   └── Book.java
│   │   ├── loader/
│   │   │   ├── BookLoader.java
│   │   │   ├── SequentialLoader.java
│   │   │   └── ParallelLoader.java
│   │   ├── executor/
│   │   │   └── LoaderExecutor.java
│   │   └── Main.java
│   └── resources/
│       └── books/
│           ├── books1.csv
│           ├── books2.csv
│           ├── books3.csv
│           ├── books4.csv
│           └── books5.csv
└── test/
    └── java/
        └── loader/
            └── LoaderTest.java
```

### 5.2. Шаблон BookLoader

```java
package loader;

import model.Book;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.Callable;

public class BookLoader implements Callable<List<Book>> {
    private static final Logger logger = LoggerFactory.getLogger(BookLoader.class);
    private final String filePath;
    
    public BookLoader(String filePath) {
        this.filePath = filePath;
    }
    
    @Override
    public List<Book> call() throws Exception {
        String threadName = Thread.currentThread().getName();
        logger.info("{} начал загрузку файла: {}", threadName, filePath);
        long startTime = System.currentTimeMillis();
        
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
                    logger.warn("Ошибка парсинга строки {} в файле {}: {}", 
                        lineNumber, filePath, e.getMessage());
                }
            }
            
        } catch (IOException e) {
            logger.error("Ошибка чтения файла: {}", filePath, e);
            throw e;
        }
        
        long duration = System.currentTimeMillis() - startTime;
        logger.info("{} завершил загрузку файла: {} книг, время: {} мс, ошибок: {}", 
            threadName, books.size(), duration, errors);
        
        return books;
    }
    
    private Book parseLine(String line) {
        // Парсинг CSV-строки
        String[] fields = line.split(",");
        if (fields.length < 6) {
            return null;
        }
        
        try {
            long id = Long.parseLong(fields[0].trim());
            String title = fields[1].trim();
            String author = fields[2].trim();
            int year = Integer.parseInt(fields[3].trim());
            String isbn = fields[4].trim();
            String publisher = fields[5].trim();
            
            return new Book(id, title, author, year, isbn, publisher);
        } catch (NumberFormatException e) {
            return null;
        }
    }
}
```

### 5.3. Шаблон ParallelLoader

```java
package loader;

import model.Book;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.*;
import java.util.concurrent.*;
import java.util.stream.Collectors;

public class ParallelLoader {
    private static final Logger logger = LoggerFactory.getLogger(ParallelLoader.class);
    private final int threadPoolSize;
    private final long timeoutMillis;
    
    public ParallelLoader(int threadPoolSize, long timeoutMillis) {
        this.threadPoolSize = threadPoolSize;
        this.timeoutMillis = timeoutMillis;
    }
    
    /**
     * Параллельная загрузка файлов
     */
    public List<Book> loadFiles(List<String> filePaths) throws InterruptedException {
        logger.info("Начало параллельной загрузки {} файлов", filePaths.size());
        long startTime = System.currentTimeMillis();
        
        ExecutorService executor = Executors.newFixedThreadPool(threadPoolSize);
        List<Future<List<Book>>> futures = new ArrayList<>();
        
        // Отправка задач на выполнение
        for (String filePath : filePaths) {
            Callable<List<Book>> loader = new BookLoader(filePath);
            Future<List<Book>> future = executor.submit(loader);
            futures.add(future);
        }
        
        // Сбор результатов с таймаутом
        List<Book> allBooks = new ArrayList<>();
        List<String> failedFiles = new ArrayList<>();
        
        for (int i = 0; i < futures.size(); i++) {
            Future<List<Book>> future = futures.get(i);
            String filePath = filePaths.get(i);
            
            try {
                List<Book> books = future.get(timeoutMillis, TimeUnit.MILLISECONDS);
                allBooks.addAll(books);
                logger.info("Файл {} загружен успешно: {} книг", filePath, books.size());
            } catch (TimeoutException e) {
                logger.error("Таймаут при загрузке файла: {}", filePath);
                future.cancel(true);
                failedFiles.add(filePath);
            } catch (ExecutionException e) {
                logger.error("Ошибка при загрузке файла: {}", filePath, e.getCause());
                failedFiles.add(filePath);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                logger.error("Загрузка прервана");
                break;
            }
        }
        
        // Завершение пула
        executor.shutdown();
        try {
            if (!executor.awaitTermination(5, TimeUnit.SECONDS)) {
                executor.shutdownNow();
            }
        } catch (InterruptedException e) {
            executor.shutdownNow();
            Thread.currentThread().interrupt();
        }
        
        long duration = System.currentTimeMillis() - startTime;
        logger.info("Параллельная загрузка завершена за {} мс", duration);
        
        if (!failedFiles.isEmpty()) {
            logger.warn("Не удалось загрузить файлы: {}", failedFiles);
        }
        
        // Удаление дубликатов по ISBN
        return removeDuplicates(allBooks);
    }
    
    /**
     * Удаление дубликатов по ISBN
     */
    private List<Book> removeDuplicates(List<Book> books) {
        int originalSize = books.size();
        
        Map<String, Book> uniqueBooks = new LinkedHashMap<>();
        for (Book book : books) {
            String isbn = book.getIsbn();
            if (!uniqueBooks.containsKey(isbn)) {
                uniqueBooks.put(isbn, book);
            } else {
                logger.warn("Дубликат ISBN: {}", isbn);
            }
        }
        
        List<Book> result = new ArrayList<>(uniqueBooks.values());
        int removed = originalSize - result.size();
        
        if (removed > 0) {
            logger.info("Удалено {} дубликатов по ISBN", removed);
        }
        
        return result;
    }
}
```

### 5.4. Шаблон SequentialLoader

```java
package loader;

import model.Book;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.ArrayList;
import java.util.List;

public class SequentialLoader {
    private static final Logger logger = LoggerFactory.getLogger(SequentialLoader.class);
    
    public List<Book> loadFiles(List<String> filePaths) {
        logger.info("Начало последовательной загрузки {} файлов", filePaths.size());
        long startTime = System.currentTimeMillis();
        
        List<Book> allBooks = new ArrayList<>();
        int errors = 0;
        
        for (String filePath : filePaths) {
            try {
                BookLoader loader = new BookLoader(filePath);
                List<Book> books = loader.call();
                allBooks.addAll(books);
                logger.info("Файл {} загружен: {} книг", filePath, books.size());
            } catch (Exception e) {
                errors++;
                logger.error("Ошибка загрузки файла {}: {}", filePath, e.getMessage());
            }
        }
        
        long duration = System.currentTimeMillis() - startTime;
        logger.info("Последовательная загрузка завершена за {} мс, ошибок: {}", duration, errors);
        
        return removeDuplicates(allBooks);
    }
    
    private List<Book> removeDuplicates(List<Book> books) {
        // Реализация удаления дубликатов (аналогично ParallelLoader)
        // ...
        return books;
    }
}
```

### 5.5. Шаблон PerformanceTest

```java
package executor;

import model.Book;
import loader.ParallelLoader;
import loader.SequentialLoader;

import java.util.ArrayList;
import java.util.List;

public class PerformanceTest {
    private final List<String> filePaths;
    private final int threadPoolSize;
    private final long timeoutMillis;
    
    public PerformanceTest(List<String> filePaths, int threadPoolSize, long timeoutMillis) {
        this.filePaths = filePaths;
        this.threadPoolSize = threadPoolSize;
        this.timeoutMillis = timeoutMillis;
    }
    
    public TestResult run() throws InterruptedException {
        SequentialLoader sequentialLoader = new SequentialLoader();
        ParallelLoader parallelLoader = new ParallelLoader(threadPoolSize, timeoutMillis);
        
        // Последовательная загрузка
        System.out.println("\n=== Последовательная загрузка ===");
        long seqStart = System.currentTimeMillis();
        List<Book> seqBooks = sequentialLoader.loadFiles(filePaths);
        long seqDuration = System.currentTimeMillis() - seqStart;
        
        // Параллельная загрузка
        System.out.println("\n=== Параллельная загрузка ===");
        long parStart = System.currentTimeMillis();
        List<Book> parBooks = parallelLoader.loadFiles(filePaths);
        long parDuration = System.currentTimeMillis() - parStart;
        
        return new TestResult(
            seqBooks.size(), seqDuration,
            parBooks.size(), parDuration,
            (double) seqDuration / parDuration
        );
    }
    
    public static class TestResult {
        private final int seqBooksCount;
        private final long seqDuration;
        private final int parBooksCount;
        private final long parDuration;
        private final double speedup;
        
        public TestResult(int seqBooksCount, long seqDuration, 
                          int parBooksCount, long parDuration, double speedup) {
            this.seqBooksCount = seqBooksCount;
            this.seqDuration = seqDuration;
            this.parBooksCount = parBooksCount;
            this.parDuration = parDuration;
            this.speedup = speedup;
        }
        
        @Override
        public String toString() {
            return String.format(
                "\n=== Сравнение производительности ===\n" +
                "Последовательная загрузка:\n" +
                "  Книг: %d\n" +
                "  Время: %d мс\n" +
                "Параллельная загрузка:\n" +
                "  Книг: %d\n" +
                "  Время: %d мс\n" +
                "Ускорение: %.2fx\n" +
                "Экономия времени: %d мс (%.1f%%)",
                seqBooksCount, seqDuration,
                parBooksCount, parDuration,
                speedup,
                seqDuration - parDuration,
                (1 - (double) parDuration / seqDuration) * 100
            );
        }
    }
}
```

### 5.6. Основной класс

```java
import executor.PerformanceTest;
import java.util.Arrays;
import java.util.List;

public class Main {
    private static final int THREAD_POOL_SIZE = 4;
    private static final long TIMEOUT_MILLIS = 30000; // 30 секунд
    
    public static void main(String[] args) throws InterruptedException {
        // Подготовка списка файлов
        List<String> filePaths = Arrays.asList(
            "src/main/resources/books/books1.csv",
            "src/main/resources/books/books2.csv",
            "src/main/resources/books/books3.csv",
            "src/main/resources/books/books4.csv",
            "src/main/resources/books/books5.csv"
        );
        
        System.out.println("=== Параллельная загрузка данных из CSV ===\n");
        
        // Запуск теста производительности
        PerformanceTest test = new PerformanceTest(filePaths, THREAD_POOL_SIZE, TIMEOUT_MILLIS);
        PerformanceTest.TestResult result = test.run();
        
        System.out.println(result);
    }
}
```

### 5.7. Обработка ошибок и таймаутов

```java
// Обработка ошибок при загрузке
private List<Book> loadWithErrorHandling(String filePath) {
    try {
        BookLoader loader = new BookLoader(filePath);
        return loader.call();
    } catch (Exception e) {
        logger.error("Критическая ошибка при загрузке файла: {}", filePath, e);
        return Collections.emptyList();
    }
}

// Обработка таймаута с повторной попыткой
private List<Book> loadWithRetry(String filePath, int maxRetries) {
    for (int attempt = 1; attempt <= maxRetries; attempt++) {
        try {
            Future<List<Book>> future = executor.submit(new BookLoader(filePath));
            return future.get(timeoutMillis, TimeUnit.MILLISECONDS);
        } catch (TimeoutException e) {
            logger.warn("Таймаут при загрузке файла {}, попытка {}/{}", 
                filePath, attempt, maxRetries);
            if (attempt == maxRetries) {
                logger.error("Не удалось загрузить файл {} после {} попыток", 
                    filePath, maxRetries);
                return Collections.emptyList();
            }
        } catch (Exception e) {
            logger.error("Ошибка при загрузке файла {}: {}", filePath, e.getMessage());
            return Collections.emptyList();
        }
    }
    return Collections.emptyList();
}
```

---

## 6. Контрольные вопросы

1. Что такое ExecutorService и зачем он нужен?

2. Какие способы создания пулов потоков существуют?

3. В чём отличие между Runnable и Callable?

4. Что такое Future и какие методы он предоставляет?

5. Как получить результат асинхронной задачи?

6. Как установить таймаут для Future?

7. Как отменить выполняющуюся задачу?

8. Что происходит при вызове future.cancel(true)?

9. Как обрабатывать исключения в Callable?

10. В чём отличие shutdown() от shutdownNow()?

11. Как дождаться завершения всех задач в ExecutorService?

12. Что такое CompletableFuture и чем он лучше Future?

13. Как удалить дубликаты после параллельной загрузки?

14. Как сравнить производительность последовательной и параллельной загрузки?

15. Какие факторы влияют на ускорение при параллельной обработке?

16. Как обрабатывать таймауты при параллельной загрузке?

17. Что такое ForkJoinPool и для каких задач он предназначен?

---

## 7. Пример выполнения (Вариант 1)

### 7.1. Ожидаемый вывод

```
=== Параллельная загрузка данных из CSV ===

=== Последовательная загрузка ===
2026-01-15 10:30:15 [main] INFO  SequentialLoader - Начало последовательной загрузки 5 файлов
2026-01-15 10:30:15 [main] INFO  BookLoader - main начал загрузку файла: src/main/resources/books/books1.csv
2026-01-15 10:30:15 [main] INFO  BookLoader - main завершил загрузку файла: 10000 книг, время: 125 мс, ошибок: 0
2026-01-15 10:30:15 [main] INFO  SequentialLoader - Файл src/main/resources/books/books1.csv загружен: 10000 книг
2026-01-15 10:30:15 [main] INFO  BookLoader - main начал загрузку файла: src/main/resources/books/books2.csv
2026-01-15 10:30:15 [main] INFO  BookLoader - main завершил загрузку файла: 10000 книг, время: 118 мс, ошибок: 0
...
2026-01-15 10:30:16 [main] INFO  SequentialLoader - Последовательная загрузка завершена за 636 мс, ошибок: 0

=== Параллельная загрузка ===
2026-01-15 10:30:16 [main] INFO  ParallelLoader - Начало параллельной загрузки 5 файлов
2026-01-15 10:30:16 [pool-1-thread-1] INFO  BookLoader - pool-1-thread-1 начал загрузку файла: src/main/resources/books/books1.csv
2026-01-15 10:30:16 [pool-1-thread-2] INFO  BookLoader - pool-1-thread-2 начал загрузку файла: src/main/resources/books/books2.csv
2026-01-15 10:30:16 [pool-1-thread-3] INFO  BookLoader - pool-1-thread-3 начал загрузку файла: src/main/resources/books/books3.csv
2026-01-15 10:30:16 [pool-1-thread-4] INFO  BookLoader - pool-1-thread-4 начал загрузку файла: src/main/resources/books/books4.csv
2026-01-15 10:30:16 [pool-1-thread-1] INFO  BookLoader - pool-1-thread-1 завершил загрузку файла: 10000 книг, время: 125 мс, ошибок: 0
2026-01-15 10:30:16 [pool-1-thread-1] INFO  BookLoader - pool-1-thread-1 начал загрузку файла: src/main/resources/books/books5.csv
2026-01-15 10:30:16 [pool-1-thread-2] INFO  BookLoader - pool-1-thread-2 завершил загрузку файла: 10000 книг, время: 118 мс, ошибок: 0
2026-01-15 10:30:16 [pool-1-thread-3] INFO  BookLoader - pool-1-thread-3 завершил загрузку файла: 10000 книг, время: 132 мс, ошибок: 0
2026-01-15 10:30:16 [pool-1-thread-4] INFO  BookLoader - pool-1-thread-4 завершил загрузку файла: 10000 книг, время: 121 мс, ошибок: 0
2026-01-15 10:30:16 [pool-1-thread-1] INFO  BookLoader - pool-1-thread-1 завершил загрузку файла: 10000 книг, время: 130 мс, ошибок: 0
2026-01-15 10:30:16 [main] WARN  ParallelLoader - Дубликат ISBN: 978-5-17-118456-0
2026-01-15 10:30:16 [main] WARN  ParallelLoader - Дубликат ISBN: 978-5-17-118457-7
...
2026-01-15 10:30:16 [main] INFO  ParallelLoader - Удалено 2000 дубликатов по ISBN
2026-01-15 10:30:16 [main] INFO  ParallelLoader - Параллельная загрузка завершена за 148 мс

=== Сравнение производительности ===
Последовательная загрузка:
  Книг: 50000
  Время: 636 мс
Параллельная загрузка:
  Книг: 48000
  Время: 148 мс
Ускорение: 4.30x
Экономия времени: 488 мс (76.7%)
```

### 7.2. График сравнения

```
Сравнение времени загрузки (мс):
┌─────────────────────────────────────────────────────────────┐
│ Последовательная  ████████████████████████████████ 636 мс │
│ Параллельная      █████████ 148 мс                        │
└─────────────────────────────────────────────────────────────┘

Ускорение: 4.30x

Анализ:
- Теоретическое ускорение: 4.00x (при 4 потоках)
- Фактическое ускорение: 4.30x (лучше теоретического за счёт кеширования)
- Накладные расходы на создание потоков: ~15 мс
- Эффективность использования процессора: ~85%
```

---

## 8. Рекомендуемые источники

1. **Шилдт Г.** *Java. Базовый курс.* — М.: Вильямс. — Глава 11 (Многопоточное программирование).

2. **Хорстманн К., Корнелл Г.** *Java. Библиотека профессионала. Том 1.* — М.: Вильямс. — Глава 12 (Параллелизм).

3. **Гоetz Б. и др.** *Java Concurrency in Practice.* — М.: Питер.

4. **Oracle Java Tutorials: Executors.** — URL: https://docs.oracle.com/javase/tutorial/essential/concurrency/executors.html

5. **Baeldung: ExecutorService Guide.** — URL: https://www.baeldung.com/java-executor-service-tutorial

6. **Baeldung: Future and CompletableFuture.** — URL: https://www.baeldung.com/java-future

7. **Блох Дж.** *Java. Эффективное программирование.* — М.: Питер. — Правила 78-86 (Параллелизм).
