# Лабораторная работа №17. Мини-проект: реализация, тестирование и защита

## 1. Паспорт работы

| Параметр | Значение |
|----------|----------|
| Номер занятия | 17 из 17 |
| Блок | 7. Промпт-инжиниринг |
| Продолжительность | 2 академических часа |
| Форма выполнения | Командная (2-3 человека) |
| ИИ-инструмент | Copilot / JetBrains AI Assistant / ChatGPT / YandexGPT / GigaChat |

### 1.1. Цель работы

Завершить реализацию командного мини-проекта «Сервис загрузки и очистки прайс-листов», провести интеграционное тестирование, подготовить документацию и защитить проект. Провести ретроспективу использования ИИ-инструментов в процессе разработки.

### 1.2. Задачи работы

1. Завершить реализацию всех модулей согласно архитектуре.
2. Интегрировать модули в единое приложение.
3. Провести сквозное тестирование на тестовых данных.
4. Подготовить README с описанием архитектуры и инструкцией по запуску.
5. Задокументировать все промпты, использованные в процессе разработки.
6. Провести защиту проекта с живой демонстрацией.
7. Провести ретроспективу использования ИИ.

### 1.3. Оснащение

- JDK 17 или выше;
- IntelliJ IDEA Community Edition;
- Git/GitHub/GitLab;
- Maven или Gradle;
- PostgreSQL (локально или в Docker);
- Доступ к ИИ-инструментам.

---

## 2. Теоретический конспект

### 2.1. Интеграционное тестирование

**Интеграционное тестирование** — проверка взаимодействия между модулями системы.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    Интеграционное тестирование                             │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                     Сквозной тест (End-to-End)                      │   │
│  │                                                                     │   │
│  │  Вход: CSV/JSON файл → Приложение → Выход: БД + Отчёт              │   │
│  │                                                                     │   │
│  │  ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐         │   │
│  │  │  Файл   │───▶│ Extract │───▶│Transform│───▶│  Load   │         │   │
│  │  │(тестовый)│    │         │    │         │    │         │         │   │
│  │  └─────────┘    └─────────┘    └─────────┘    └────┬────┘         │   │
│  │                                                      │              │   │
│  │                                                      ▼              │   │
│  │                                               ┌─────────┐         │   │
│  │                                               │  БД     │         │   │
│  │                                               │(результат)│        │   │
│  │                                               └─────────┘         │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 2.2. Тестовые данные

**Требования к тестовым данным:**
1. **Не менее 3 файлов** разных форматов (CSV, JSON).
2. **Разные валюты** (USD, EUR, RUB).
3. **Заведомо некорректные записи** (пустые поля, неверные форматы).
4. **Дубликаты** для проверки идемпотентности.

**Пример тестовых данных:**

```csv
article,name,category,price,currency,date,supplier
A001,Ноутбук ASUS,Электроника,85000.00,RUB,2024-01-15,ООО Техно
A002,Смартфон Samsung,Электроника,120.00,USD,2024-01-15,ООО Техно
A003,Книга "Java",Книги,1500.00,RUB,2024-01-15,ООО Книги
A004,Пустая строка,,100.00,USD,2024-01-15,
A005,Неверная дата,Книги,1000.00,RUB,2024-13-40,ООО Книги
```

### 2.3. Документирование

**Структура README:**

```markdown
# Price List Service

## Описание
Сервис загрузки и очистки прайс-листов поставщиков.

## Требования
- Java 17+
- PostgreSQL 16+
- Maven 3.8+

## Установка и запуск
1. Клонировать репозиторий
2. Настроить БД
3. Собрать проект
4. Запустить

## Архитектура
[Диаграмма классов, описание модулей]

## Использование
[Инструкция по использованию]

## Тестирование
[Описание тестовых данных и результатов]

## Использованные промпты
[Документация всех промптов]

## Команда
[Список участников и их роли]
```

### 2.4. Ретроспектива использования ИИ

**Вопросы для ретроспективы:**

1. **На каких этапах ИИ помог?**
   - Генерация шаблонного кода.
   - Написание тестов.
   - Рефакторинг.
   - Документация.

2. **На каких этапах ИИ помешал?**
   - Галлюцинации (несуществующие API).
   - Неполный код (требуется доработка).
   - Устаревшие подходы.

3. **Чему научились в части взаимодействия с ИИ?**
   - Составлять эффективные промпты.
   - Проверять сгенерированный код.
   - Использовать цепочки промптов.
   - Критически оценивать результаты.

### 2.5. Критерии оценки проекта

| Критерий | Вес | Описание |
|----------|-----|----------|
| Корректность ETL | 25% | Правильность извлечения, преобразования и загрузки |
| Стабильность при конкурентной загрузке | 20% | Отсутствие ошибок при параллельной обработке |
| Качество кода (code review) | 25% | Чистота кода, соблюдение принципов SOLID |
| Полнота документации | 15% | README, промпты, инструкция |
| Демонстрация и ответы на вопросы | 15% | Качество презентации, понимание проекта |

---

## 3. Задание на паре

### Задача. Завершение проекта и защита

### 3.1. Этап 1. Завершение реализации (30 мин)

1. **Завершить реализацию всех модулей:**
   - `PriceListPipeline` — основной пайплайн.
   - `PriceListDaoImpl` — полная реализация.
   - `CurrencyNormalizer` — нормализация валют.
   - `CategoryEnricher` — обогащение категориями.
   - `DataSourceReader` — чтение CSV/JSON.

2. **Интегрировать модули** в единое приложение.

3. **Добавить консольный интерфейс** (аргументы командной строки).

### 3.2. Этап 2. Тестирование (20 мин)

1. **Подготовить тестовые данные:**
   - 3 файла разных форматов.
   - Разные валюты.
   - Некорректные записи.

2. **Провести сквозное тестирование:**
   - Запуск приложения на тестовых данных.
   - Проверка результатов в БД.
   - Проверка DLQ (файл с ошибками).

3. **Проверить идемпотентность:**
   - Повторный запуск на тех же данных.
   - Отсутствие дублей.

### 3.3. Этап 3. Документация (20 мин)

1. **Подготовить README:**
   - Описание проекта.
   - Инструкция по установке и запуску.
   - Архитектура (диаграмма).
   - Документированные промпты.

2. **Задокументировать промпты:**
   - Промпт для генерации каркаса.
   - Промпты для отдельных модулей.
   - Промпты для тестов.

### 3.4. Этап 4. Подготовка к защите (10 мин)

1. **Подготовить демонстрацию:**
   - Запуск приложения на тестовых данных.
   - Демонстрация результатов в БД.
   - Демонстрация DLQ.

2. **Подготовиться к вопросам:**
   - Архитектура.
   - Реализация.
   - Использование ИИ.

### 3.5. Этап 5. Защита проекта (время на паре)

1. **Живая демонстрация:**
   - Запуск приложения на тестовых данных.
   - Демонстрация результатов в БД.
   - Ответы на вопросы.

2. **Ретроспектива:**
   - Каждый студент описывает:
     - На каких этапах ИИ помог.
     - На каких этапах ИИ помешал.
     - Чему научились в части взаимодействия с ИИ.

### Применение ИИ-инструмента (ретроспектива)

**Промпт для рефлексии:**

```markdown
**Роль:** Ты — Java-разработчик, завершивший проект "Сервис загрузки и очистки прайс-листов".

**Задача:** Проведи ретроспективу использования ИИ-инструментов в проекте.

**Вопросы для рефлексии:**
1. На каких этапах разработки ИИ был наиболее полезен?
2. Какие проблемы возникли при использовании ИИ?
3. Какие промпты оказались наиболее эффективными?
4. Чему ты научился в части взаимодействия с ИИ?
5. Как бы ты изменил подход к использованию ИИ в следующем проекте?

**Формат вывода:** Структурированный текст с примерами.
```

---

## 4. Методические указания

### 4.1. Полная реализация PriceListPipeline

```java
package com.example.pricelist.pipeline;

import com.example.pricelist.dao.PriceListDao;
import com.example.pricelist.model.PriceItem;
import com.example.pricelist.reader.DataSourceReader;
import com.example.pricelist.reader.ParserStrategy;
import com.example.pricelist.transformer.CategoryEnricher;
import com.example.pricelist.transformer.CurrencyNormalizer;
import com.example.pricelist.transformer.PriceItemFilter;
import com.example.pricelist.util.DeadLetterQueue;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.IOException;
import java.sql.SQLException;
import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.*;

public class PriceListPipeline implements Pipeline {
    private static final Logger logger = LoggerFactory.getLogger(PriceListPipeline.class);
    
    private final DataSourceReader reader;
    private final CurrencyNormalizer normalizer;
    private final CategoryEnricher enricher;
    private final PriceItemFilter filter;
    private final PriceListDao dao;
    private final DeadLetterQueue dlq;
    private final int threadPoolSize;
    
    public PriceListPipeline(DataSourceReader reader,
                             CurrencyNormalizer normalizer,
                             CategoryEnricher enricher,
                             PriceItemFilter filter,
                             PriceListDao dao,
                             DeadLetterQueue dlq,
                             int threadPoolSize) {
        this.reader = reader;
        this.normalizer = normalizer;
        this.enricher = enricher;
        this.filter = filter;
        this.dao = dao;
        this.dlq = dlq;
        this.threadPoolSize = threadPoolSize;
    }
    
    @Override
    public EtlResult run(String filePath, String format) throws IOException, SQLException {
        logger.info("=== Запуск ETL-пайплайна ===");
        logger.info("Файл: {}, формат: {}", filePath, format);
        long startTime = System.currentTimeMillis();
        
        // 1. EXTRACT
        logger.info("--- EXTRACT ---");
        ParserStrategy strategy = getStrategy(format);
        reader.setStrategy(strategy);
        List<PriceItem> extracted = reader.read(filePath);
        logger.info("Извлечено: {} записей", extracted.size());
        
        // 2. TRANSFORM
        logger.info("--- TRANSFORM ---");
        List<PriceItem> transformed = new ArrayList<>();
        int errors = 0;
        
        for (PriceItem item : extracted) {
            try {
                // Валидация
                if (!filter.isValid(item)) {
                    errors++;
                    dlq.addError(filePath, 0, item.toString(), "Не прошёл валидацию");
                    continue;
                }
                
                // Нормализация валюты
                PriceItem normalized = normalizer.normalize(item);
                
                // Обогащение категорией
                PriceItem enriched = enricher.enrich(normalized);
                
                transformed.add(enriched);
            } catch (Exception e) {
                errors++;
                dlq.addError(filePath, 0, item.toString(), e.getMessage());
                logger.warn("Ошибка трансформации: {}", e.getMessage());
            }
        }
        logger.info("Преобразовано: {} записей, ошибок: {}", transformed.size(), errors);
        
        // 3. LOAD
        logger.info("--- LOAD ---");
        int loaded = 0;
        if (!transformed.isEmpty()) {
            try {
                // Сначала создаём таблицу
                dao.ensureTableExists();
                // Загружаем с UPSERT
                loaded = dao.saveAll(transformed);
                logger.info("Загружено: {} записей", loaded);
            } catch (SQLException e) {
                logger.error("Ошибка загрузки в БД", e);
                for (PriceItem item : transformed) {
                    dlq.addError(filePath, 0, item.toString(), "Ошибка загрузки: " + e.getMessage());
                }
                errors += transformed.size();
            }
        }
        
        // 4. DLQ
        logger.info("--- DLQ ---");
        dlq.flush();
        
        long duration = System.currentTimeMillis() - startTime;
        logger.info("=== ETL завершён за {} мс ===", duration);
        
        return new EtlResult(
            extracted.size(),
            transformed.size(),
            loaded,
            errors,
            duration,
            transformed,
            dlq.getErrors()
        );
    }
    
    @Override
    public EtlResult runParallel(List<String> filePaths) throws IOException, SQLException {
        logger.info("=== Запуск параллельного ETL ===");
        logger.info("Файлов: {}", filePaths.size());
        long startTime = System.currentTimeMillis();
        
        ExecutorService executor = Executors.newFixedThreadPool(threadPoolSize);
        List<Future<EtlResult>> futures = new ArrayList<>();
        
        for (String filePath : filePaths) {
            String format = detectFormat(filePath);
            Future<EtlResult> future = executor.submit(() -> run(filePath, format));
            futures.add(future);
        }
        
        // Сбор результатов
        List<PriceItem> allItems = new ArrayList<>();
        int totalExtracted = 0;
        int totalTransformed = 0;
        int totalLoaded = 0;
        int totalErrors = 0;
        
        for (Future<EtlResult> future : futures) {
            try {
                EtlResult result = future.get(60, TimeUnit.SECONDS);
                totalExtracted += result.extracted();
                totalTransformed += result.transformed();
                totalLoaded += result.loaded();
                totalErrors += result.errors();
                allItems.addAll(result.items());
            } catch (TimeoutException e) {
                logger.error("Таймаут при выполнении задачи", e);
                future.cancel(true);
            } catch (Exception e) {
                logger.error("Ошибка при выполнении задачи", e);
            }
        }
        
        executor.shutdown();
        try {
            if (!executor.awaitTermination(10, TimeUnit.SECONDS)) {
                executor.shutdownNow();
            }
        } catch (InterruptedException e) {
            executor.shutdownNow();
            Thread.currentThread().interrupt();
        }
        
        long duration = System.currentTimeMillis() - startTime;
        logger.info("=== Параллельный ETL завершён за {} мс ===", duration);
        
        return new EtlResult(
            totalExtracted,
            totalTransformed,
            totalLoaded,
            totalErrors,
            duration,
            allItems,
            new ArrayList<>()
        );
    }
    
    private ParserStrategy getStrategy(String format) {
        return switch (format.toUpperCase()) {
            case "CSV" -> new CsvParserStrategy();
            case "JSON" -> new JsonParserStrategy();
            default -> throw new IllegalArgumentException("Неподдерживаемый формат: " + format);
        };
    }
    
    private String detectFormat(String filePath) {
        if (filePath.endsWith(".csv")) return "CSV";
        if (filePath.endsWith(".json")) return "JSON";
        return "CSV"; // по умолчанию
    }
}
```

### 4.2. Консольный интерфейс (Main)

```java
package com.example.pricelist;

import com.example.pricelist.config.DataSourceProvider;
import com.example.pricelist.dao.PriceListDao;
import com.example.pricelist.dao.PriceListDaoImpl;
import com.example.pricelist.pipeline.EtlResult;
import com.example.pricelist.pipeline.PriceListPipeline;
import com.example.pricelist.reader.DataSourceReader;
import com.example.pricelist.transformer.CategoryEnricher;
import com.example.pricelist.transformer.CurrencyNormalizer;
import com.example.pricelist.transformer.PriceItemFilter;
import com.example.pricelist.util.CurrencyApiClient;
import com.example.pricelist.util.DeadLetterQueue;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.IOException;
import java.sql.SQLException;
import java.util.Arrays;
import java.util.List;

public class Main {
    private static final Logger logger = LoggerFactory.getLogger(Main.class);
    private static final String CATEGORIES_FILE = "src/main/resources/categories.csv";
    private static final String DLQ_FILE = "logs/price_list_errors.csv";
    private static final int THREAD_POOL_SIZE = 4;
    
    public static void main(String[] args) {
        System.out.println("=== Сервис загрузки и очистки прайс-листов ===\n");
        
        if (args.length == 0) {
            printUsage();
            return;
        }
        
        try {
            // Инициализация компонентов
            DataSourceReader reader = new DataSourceReader(null);
            CurrencyApiClient currencyApi = new CurrencyApiClient();
            CurrencyNormalizer normalizer = new CurrencyNormalizer(currencyApi);
            CategoryEnricher enricher = new CategoryEnricher(CATEGORIES_FILE);
            PriceItemFilter filter = new PriceItemFilter();
            PriceListDao dao = new PriceListDaoImpl();
            DeadLetterQueue dlq = new DeadLetterQueue(DLQ_FILE);
            
            PriceListPipeline pipeline = new PriceListPipeline(
                reader, normalizer, enricher, filter, dao, dlq, THREAD_POOL_SIZE
            );
            
            // Обработка аргументов
            if (args.length == 1) {
                // Одиночный файл
                String filePath = args[0];
                String format = detectFormat(filePath);
                EtlResult result = pipeline.run(filePath, format);
                System.out.println(result);
            } else if (args.length >= 2 && args[0].equals("--parallel")) {
                // Параллельная загрузка
                List<String> filePaths = Arrays.asList(args).subList(1, args.length);
                EtlResult result = pipeline.runParallel(filePaths);
                System.out.println(result);
            } else {
                printUsage();
            }
            
        } catch (Exception e) {
            logger.error("Ошибка выполнения", e);
            System.err.println("Ошибка: " + e.getMessage());
        } finally {
            DataSourceProvider.close();
        }
    }
    
    private static void printUsage() {
        System.out.println("Использование:");
        System.out.println("  java -jar price-list-service.jar <file>");
        System.out.println("  java -jar price-list-service.jar --parallel <file1> <file2> ...");
        System.out.println();
        System.out.println("Примеры:");
        System.out.println("  java -jar price-list-service.jar price.csv");
        System.out.println("  java -jar price-list-service.jar --parallel price1.csv price2.json");
    }
    
    private static String detectFormat(String filePath) {
        if (filePath.endsWith(".csv")) return "CSV";
        if (filePath.endsWith(".json")) return "JSON";
        return "CSV";
    }
}
```

### 4.3. EtlResult (иммутабельный)

```java
package com.example.pricelist.pipeline;

import com.example.pricelist.model.PriceItem;
import com.example.pricelist.util.ErrorRecord;

import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

public record EtlResult(
    int extracted,
    int transformed,
    int loaded,
    int errors,
    long durationMs,
    List<PriceItem> items,
    List<ErrorRecord> errorRecords
) {
    public double getSuccessRate() {
        if (extracted == 0) return 0.0;
        return (double) loaded / extracted * 100;
    }
    
    public Map<String, Long> getCategoryStats() {
        if (items == null || items.isEmpty()) {
            return Map.of();
        }
        return items.stream()
            .collect(Collectors.groupingBy(
                PriceItem::category,
                Collectors.counting()
            ));
    }
    
    public Map<String, Long> getCurrencyStats() {
        if (items == null || items.isEmpty()) {
            return Map.of();
        }
        return items.stream()
            .collect(Collectors.groupingBy(
                PriceItem::currency,
                Collectors.counting()
            ));
    }
    
    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder();
        sb.append("\n=== Результат ETL ===\n");
        sb.append("Извлечено (Extract): ").append(extracted).append("\n");
        sb.append("Преобразовано (Transform): ").append(transformed).append("\n");
        sb.append("Загружено (Load): ").append(loaded).append("\n");
        sb.append("Ошибок: ").append(errors).append("\n");
        sb.append("Успешность: ").append(String.format("%.2f%%", getSuccessRate())).append("\n");
        sb.append("Время: ").append(durationMs).append(" мс\n");
        
        Map<String, Long> categoryStats = getCategoryStats();
        if (!categoryStats.isEmpty()) {
            sb.append("\nСтатистика по категориям:\n");
            categoryStats.entrySet().stream()
                .sorted(Map.Entry.<String, Long>comparingByValue().reversed())
                .forEach(e -> sb.append("  ").append(e.getKey())
                    .append(": ").append(e.getValue()).append("\n"));
        }
        
        Map<String, Long> currencyStats = getCurrencyStats();
        if (!currencyStats.isEmpty()) {
            sb.append("\nСтатистика по валютам:\n");
            currencyStats.entrySet().stream()
                .sorted(Map.Entry.<String, Long>comparingByValue().reversed())
                .forEach(e -> sb.append("  ").append(e.getKey())
                    .append(": ").append(e.getValue()).append("\n"));
        }
        
        if (errorRecords != null && !errorRecords.isEmpty()) {
            sb.append("\nОшибки сохранены в: logs/price_list_errors.csv\n");
            sb.append("Количество ошибок: ").append(errorRecords.size()).append("\n");
        }
        
        return sb.toString();
    }
}
```

### 4.4. Шаблон README

```markdown
# Price List Service

## 📋 Описание

Сервис загрузки и очистки прайс-листов поставщиков. Реализует ETL-пайплайн для извлечения данных из CSV/JSON, нормализации валют, обогащения категориями и загрузки в PostgreSQL.

## 🚀 Требования

- Java 17+
- PostgreSQL 16+
- Maven 3.8+
- Docker (опционально)

## 📦 Установка и запуск

### 1. Клонирование репозитория

```bash
git clone https://github.com/your-team/price-list-service.git
cd price-list-service
```

### 2. Настройка базы данных

```bash
docker run --name price-list-db -e POSTGRES_PASSWORD=password -p 5432:5432 -d postgres:16
```

### 3. Сборка проекта

```bash
mvn clean package
```

### 4. Запуск

```bash
java -jar target/price-list-service-1.0.jar data/sample.csv
```

## 🏗️ Архитектура

```
┌─────────────────────────────────────────────────────────────┐
│                    Price List Service                       │
│                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │   Extract   │──▶│  Transform  │──▶│    Load     │         │
│  │ (CSV/JSON)  │  │ (Currency,  │  │ (PostgreSQL)│         │
│  │             │  │  Category)  │  │             │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              Dead Letter Queue                      │    │
│  │         (etl_errors.csv)                           │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

## 📝 Использование

### Одиночный файл

```bash
java -jar price-list-service.jar data/price.csv
java -jar price-list-service.jar data/price.json
```

### Параллельная загрузка

```bash
java -jar price-list-service.jar --parallel data/price1.csv data/price2.json data/price3.csv
```

### Пример вывода

```
=== Результат ETL ===
Извлечено: 500
Преобразовано: 480
Загружено: 480
Ошибок: 20
Успешность: 96.00%
Время: 125 мс

Статистика по категориям:
  Электроника: 150
  Книги: 120
  Одежда: 100
  Другое: 110
```

## 🤖 Использованные промпты

### Промпт 1: Генерация каркаса проекта

```markdown
**Роль:** Ты — senior Java-архитектор.

**Задача:** Спроектируй структуру проекта "Сервис загрузки и очистки прайс-листов".

**Требования:**
1. Консольное приложение на Java 17.
2. Чтение CSV и JSON файлов.
3. ETL-пайплайн...
```

### Промпт 2: Генерация PriceItem

```markdown
**Роль:** Ты — senior Java-разработчик.

**Задача:** Создай record PriceItem для модели данных прайс-листа.

**Поля:** article, name, category, price, currency, date, supplier
```

## 👥 Команда

| Роль | Имя |
|------|-----|
| Team Lead | [Имя] |
| Разработчик 1 | [Имя] |
| Разработчик 2 | [Имя] |

## 📚 Источники

1. Шилдт Г. Java. Базовый курс.
2. Хорстманн К. Java. Библиотека профессионала.
3. PostgreSQL Documentation.
4. Jackson Documentation.
```

---

## 5. Контрольные вопросы для защиты

1. **Архитектура проекта:**
   - Как организована структура приложения?
   - Какие паттерны использованы?

2. **ETL-пайплайн:**
   - Как реализован этап Extract?
   - Как реализован этап Transform?
   - Как реализован этап Load?

3. **Многопоточность:**
   - Как организована параллельная загрузка?
   - Какие проблемы возникли при конкурентной загрузке?

4. **Идемпотентность:**
   - Как обеспечивается идемпотентность?
   - Что будет при повторном запуске?

5. **Обработка ошибок:**
   - Как обрабатываются ошибки?
   - Что такое dead-letter queue?

6. **Использование ИИ:**
   - Какие промпты использовались?
   - Какие проблемы возникли с ИИ?

7. **Тестирование:**
   - Какие тестовые данные использовались?
   - Как проверялась корректность?

---

## 6. Критерии оценки защиты

| Критерий | Отлично | Хорошо | Удовлетворительно |
|----------|---------|--------|-------------------|
| **Корректность ETL** | Все этапы работают корректно, данные загружены правильно | Есть незначительные ошибки | Есть критические ошибки |
| **Стабильность** | Работает без ошибок при параллельной загрузке | Редкие ошибки | Частые ошибки |
| **Качество кода** | SOLID, чистый код, документация | Есть нарушения | Много нарушений |
| **Документация** | Полная, понятная, включает промпты | Частичная | Минимальная |
| **Демонстрация** | Уверенная демонстрация, полные ответы | Хорошая демонстрация | Слабая демонстрация |

---

## 7. Рекомендуемые источники

1. **Шилдт Г.** *Java. Базовый курс.* — М.: Вильямс.

2. **Хорстманн К., Корнелл Г.** *Java. Библиотека профессионала. Том 1-2.* — М.: Вильямс.

3. **Fowler M.** *Patterns of Enterprise Application Architecture.* — Addison-Wesley.

4. **Kimball R., Caserta J.** *The Data Warehouse ETL Toolkit.* — Wiley.

5. **PostgreSQL Documentation.** — URL: https://www.postgresql.org/docs/

6. **Jackson Documentation.** — URL: https://github.com/FasterXML/jackson-docs

7. **HikariCP GitHub.** — URL: https://github.com/brettwooldridge/HikariCP

8. **Baeldung: Java Tutorials.** — URL: https://www.baeldung.com/
