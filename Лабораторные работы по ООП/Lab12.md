# Лабораторная работа №12. Интеграция с внешними API и LLM-сервисами

## 1. Паспорт работы

| Параметр | Значение |
|----------|----------|
| Тема | Интеграция с внешними API и LLM-сервисами |
| Номер занятия в модуле | 4 из 4 (модуль 3) |
| Продолжительность аудиторной части | 2 академических часа |
| Предшествующая подготовка | Лабораторные работы №9–11 (функциональные интерфейсы, Stream API, JSON) |
| Тип работы | Формирование навыков интеграции Java-приложений с внешними сервисами и большими языковыми моделями |

### 1.1. Цель работы

Освоить механизмы взаимодействия Java-приложений с внешними сервисами посредством протокола HTTP: изучить стандартный HTTP-клиент `java.net.http.HttpClient`, принципы REST-архитектуры, основы работы с JSON API, а также приобрести практические навыки интеграции с программными интерфейсами больших языковых моделей (LLM). Научиться применять средства искусственного интеллекта как для создания интеллектуальных функций в приложениях, так и для рефакторинга собственного кода.

### 1.2. Задачи работы

1. Изучить основы протокола HTTP: методы запросов, коды ответов, заголовки.
2. Освоить принципы REST-архитектуры и проектирования RESTful API.
3. Научиться выполнять HTTP-запросы посредством стандартного клиента `java.net.http.HttpClient`.
4. Освоить обработку JSON-ответов от внешних API с применением библиотеки Jackson.
5. Изучить механизмы аутентификации (API-ключи, Bearer-токены).
6. Освоить обработку ошибок HTTP и сетевых сбоев.
7. Изучить принципы взаимодействия с программными интерфейсами больших языковых моделей.
8. Освоить основы prompt-инженерии и её применение в Java-приложениях.
9. Научиться применять библиотеку LangChain4j для интеграции с LLM.
10. Освоить применение средств ИИ для рефакторинга и улучшения собственного кода.

### 1.3. Оснащение

- JDK версии 17 или выше;
- интегрированная среда разработки IntelliJ IDEA Community Edition;
- система сборки Maven или Gradle;
- система контроля версий Git;
- система модульного тестирования JUnit 5;
- библиотека Jackson для работы с JSON;
- библиотека LangChain4j (рекомендуется) или прямой HTTP-клиент;
- доступ к LLM API (OpenAI, YandexGPT, OpenRouter, Ollama или аналогичному);
- доступ к публичным REST API для демонстрационных целей.

### 1.4. Особые условия выполнения работы

В связи с тем, что работа предполагает взаимодействие с внешними сервисами, необходимо учитывать следующие условия:

1. **API-ключи.** Для работы с некоторыми LLM API требуется API-ключ. Преподаватель предоставляет учебные ключи или рекомендует бесплатные тарифы. API-ключи **не должны** фиксироваться в коде и коммититься в систему контроля версий — они передаются через переменные окружения или конфигурационные файлы, добавленные в `.gitignore`.

2. **Локальные альтернативы.** При отсутствии доступа к интернету или платным API-сервисам допускается применение локальных моделей через Ollama или LM Studio.

3. **Конфиденциальность данных.** Запрещается передавать во внешние сервисы:
   - персональные данные третьих лиц;
   - конфиденциальную информацию учебного заведения;
   - данные, нарушающие права интеллектуальной собственности;
   - API-ключи и учётные данные.

4. **Mock-серверы.** Для тестирования без реальных запросов допускается применение mock-серверов (WireMock, MockServer) или заранее подготовленных JSON-файлов.

## 2. Теоретические сведения

### 2.1. Мотивация: зачем нужна интеграция с внешними сервисами

Современные программные системы редко работают изолированно. Даже простое приложение может нуждаться в:

- получении актуальных данных (погода, курсы валют, новости);
- взаимодействии с платёжными системами;
- отправке уведомлений (email, SMS, push);
- использовании искусственного интеллекта для анализа текста, изображений, генерации контента;
- интеграции с социальными сетями и мессенджерами.

Всё это требует умения работать с внешними API посредством сетевого протокола HTTP.

### 2.2. Основы протокола HTTP

**HTTP** (HyperText Transfer Protocol) — протокол прикладного уровня для передачи данных в сети. Основан на модели «клиент-сервер»: клиент отправляет запрос, сервер возвращает ответ.

#### 2.2.1. HTTP-запрос

Запрос состоит из:
- **строки запроса** с методом, URL и версией протокола;
- **заголовков** (headers) — метаданных запроса;
- **тела** (body) — опциональных данных.

**Основные методы HTTP:**

| Метод | Назначение | Идемпотентность | Тело |
|-------|------------|:---------------:|:----:|
| `GET` | Получение ресурса | Да | Нет |
| `POST` | Создание ресурса | Нет | Да |
| `PUT` | Полное обновление ресурса | Да | Да |
| `PATCH` | Частичное обновление ресурса | Нет | Да |
| `DELETE` | Удаление ресурса | Да | Нет |

**Пример запроса:**

```http
POST /api/users HTTP/1.1
Host: api.example.com
Content-Type: application/json
Authorization: Bearer token123
Content-Length: 52

{"name": "Иванов И.И.", "email": "ivanov@example.com"}
```

#### 2.2.2. HTTP-ответ

Ответ состоит из:
- **строки состояния** с версией протокола, кодом и текстовым описанием;
- **заголовков**;
- **тела** — возвращаемых данных.

**Основные коды состояния:**

| Код | Категория | Описание |
|-----|-----------|----------|
| `200 OK` | 2xx Успех | Запрос выполнен успешно |
| `201 Created` | 2xx Успех | Ресурс создан |
| `204 No Content` | 2xx Успех | Успех без тела ответа |
| `400 Bad Request` | 4xx Ошибка клиента | Некорректный запрос |
| `401 Unauthorized` | 4xx Ошибка клиента | Требуется аутентификация |
| `403 Forbidden` | 4xx Ошибка клиента | Доступ запрещён |
| `404 Not Found` | 4xx Ошибка клиента | Ресурс не найден |
| `429 Too Many Requests` | 4xx Ошибка клиента | Превышен лимит запросов |
| `500 Internal Server Error` | 5xx Ошибка сервера | Внутренняя ошибка сервера |
| `503 Service Unavailable` | 5xx Ошибка сервера | Сервис временно недоступен |

### 2.3. REST-архитектура

**REST** (Representational State Transfer) — архитектурный стиль взаимодействия компонентов распределённых приложений. RESTful API соответствует следующим принципам:

1. **Клиент-сервер.** Разделение ответственности между клиентом и сервером.
2. **Отсутствие состояния (stateless).** Каждый запрос содержит всю необходимую информацию.
3. **Кэшируемость.** Ответы могут быть кэшируемы.
4. **Единообразие интерфейса.** Унифицированный способ взаимодействия с ресурсами.
5. **Слоистая система.** Клиент не знает, взаимодействует ли он с конечным сервером или промежуточным узлом.

**Проектирование RESTful API:**

| Операция | HTTP-метод | URL | Тело запроса |
|----------|------------|-----|--------------|
| Получить список | `GET` | `/api/users` | — |
| Получить один | `GET` | `/api/users/123` | — |
| Создать | `POST` | `/api/users` | JSON с данными |
| Обновить | `PUT` | `/api/users/123` | JSON с данными |
| Частично обновить | `PATCH` | `/api/users/123` | JSON с изменениями |
| Удалить | `DELETE` | `/api/users/123` | — |

### 2.4. HTTP-клиент в Java (`java.net.http.HttpClient`)

Начиная с Java 11, в стандартную библиотеку входит класс `HttpClient`, предоставляющий современный API для выполнения HTTP-запросов.

#### 2.4.1. Базовое использование

```java
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;

// Создание клиента
HttpClient client = HttpClient.newHttpClient();

// Создание GET-запроса
HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com/users"))
    .header("Accept", "application/json")
    .GET()
    .build();

// Выполнение запроса
HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());

// Обработка ответа
int statusCode = response.statusCode();
String body = response.body();
System.out.println("Статус: " + statusCode);
System.out.println("Тело: " + body);
```

#### 2.4.2. POST-запрос с JSON-телом

```java
String jsonBody = """
    {
        "name": "Иванов И.И.",
        "email": "ivanov@example.com"
    }
    """;

HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com/users"))
    .header("Content-Type", "application/json")
    .header("Authorization", "Bearer " + apiKey)
    .POST(HttpRequest.BodyPublishers.ofString(jsonBody))
    .build();

HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
```

#### 2.4.3. Асинхронные запросы

```java
// Асинхронное выполнение
client.sendAsync(request, HttpResponse.BodyHandlers.ofString())
    .thenApply(HttpResponse::body)
    .thenAccept(body -> System.out.println("Получено: " + body))
    .exceptionally(ex -> {
        System.err.println("Ошибка: " + ex.getMessage());
        return null;
    });
```

#### 2.4.4. Конфигурация клиента

```java
HttpClient client = HttpClient.newBuilder()
    .version(HttpClient.Version.HTTP_2)                    // HTTP/2
    .connectTimeout(Duration.ofSeconds(10))                // таймаут соединения
    .followRedirects(HttpClient.Redirect.NORMAL)           // автоматические редиректы
    .authenticator(new Authenticator() {                   // аутентификация
        @Override
        protected PasswordAuthentication getRequestAuthentication(...) {
            return new PasswordAuthentication("user", "pass".toCharArray());
        }
    })
    .build();
```

### 2.5. Обработка ошибок и таймаутов

При работе с внешними API необходимо обрабатывать различные типы ошибок:

```java
try {
    HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());

    int statusCode = response.statusCode();
    if (statusCode >= 200 && statusCode < 300) {
        // Успешный ответ
        processResponse(response.body());
    } else if (statusCode == 401) {
        throw new AuthenticationException("Неверный API-ключ");
    } else if (statusCode == 404) {
        throw new ResourceNotFoundException("Ресурс не найден");
    } else if (statusCode == 429) {
        throw new RateLimitExceededException("Превышен лимит запросов");
    } else if (statusCode >= 500) {
        throw new ServerException("Ошибка сервера: " + statusCode);
    } else {
        throw new ApiException("Неожиданный статус: " + statusCode);
    }
} catch (HttpTimeoutException e) {
    throw new ApiException("Превышен таймаут запроса", e);
} catch (InterruptedException e) {
    Thread.currentThread().interrupt();
    throw new ApiException("Запрос прерван", e);
} catch (IOException e) {
    throw new ApiException("Сетевая ошибка", e);
}
```

### 2.6. Интеграция с LLM API

**Большие языковые модели (LLM)** — нейросетевые модели, обученные на больших объёмах текстовых данных и способные генерировать и анализировать текст на естественном языке. Примеры: GPT-4 (OpenAI), YandexGPT (Яндекс), Claude (Anthropic), Qwen (Alibaba).

#### 2.6.1. Общая схема взаимодействия

Взаимодействие с LLM API обычно состоит из следующих шагов:

1. Формирование **промпта** (prompt) — текстового запроса к модели.
2. Отправка HTTP-запроса к API с промптом и параметрами (температура, максимальная длина и т. п.).
3. Получение JSON-ответа с сгенерированным текстом.
4. Разбор ответа и извлечение результата.

#### 2.6.2. Пример запроса к OpenAI API

```http
POST https://api.openai.com/v1/chat/completions
Authorization: Bearer sk-...
Content-Type: application/json

{
  "model": "gpt-4o-mini",
  "messages": [
    {"role": "system", "content": "Ты — помощник в решении задач по программированию."},
    {"role": "user", "content": "Объясни, что такое полиморфизм в Java."}
  ],
  "temperature": 0.7,
  "max_tokens": 500
}
```

Ответ:

```json
{
  "id": "chatcmpl-...",
  "choices": [{
    "message": {
      "role": "assistant",
      "content": "Полиморфизм в Java — это..."
    },
    "finish_reason": "stop"
  }],
  "usage": {
    "prompt_tokens": 20,
    "completion_tokens": 150,
    "total_tokens": 170
  }
}
```

#### 2.6.3. Роли сообщений

В диалоговых моделях различают три роли сообщений:

- **`system`** — системная инструкция, задающая поведение модели;
- **`user`** — сообщение пользователя;
- **`assistant`** — ответ модели (используется для сохранения контекста диалога).

#### 2.6.4. Параметры генерации

- **`temperature`** (0.0–2.0) — степень случайности. Низкие значения — более детерминированные ответы, высокие — более креативные.
- **`max_tokens`** — максимальная длина ответа в токенах.
- **`top_p`** — альтернатива temperature, ядерная выборка.

### 2.7. Локальные модели: Ollama

**Ollama** — инструмент для запуска LLM локально. Поддерживает модели LLaMA, Mistral, Qwen, Gemma и другие.

**Установка и запуск:**

```bash
# Установка (Linux/macOS)
curl -fsSL https://ollama.ai/install.sh | sh

# Запуск модели
ollama run llama3.2
ollama run qwen2.5:7b
```

**API Ollama** совместимо с OpenAI API и доступно по адресу `http://localhost:11434/v1/chat/completions`.

### 2.8. Библиотека LangChain4j

**LangChain4j** — Java-библиотека, упрощающая интеграцию с LLM. Предоставляет единый интерфейс для работы с различными провайдерами (OpenAI, Ollama, Azure, YandexGPT и др.).

**Подключение:**

```xml
<dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j</artifactId>
    <version>0.35.0</version>
</dependency>
<dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j-open-ai</artifactId>
    <version>0.35.0</version>
</dependency>
```

**Пример использования:**

```java
import dev.langchain4j.model.openai.OpenAiChatModel;
import dev.langchain4j.model.chat.ChatLanguageModel;

// Создание модели
ChatLanguageModel model = OpenAiChatModel.builder()
    .apiKey(System.getenv("OPENAI_API_KEY"))
    .modelName("gpt-4o-mini")
    .temperature(0.7)
    .build();

// Простой запрос
String answer = model.generate("Объясни, что такое полиморфизм в Java.");
System.out.println(answer);
```

**Использование системных промптов и AI Services:**

```java
// Определение интерфейса AI-сервиса
interface Tutor {
    @SystemMessage("Ты — репетитор по программированию. Отвечай кратко и по существу.")
    String explain(@UserMessage String topic);

    @SystemMessage("Ты — репетитор. Проверяй код и указывай на ошибки.")
    String reviewCode(@UserMessage String code);
}

// Создание прокси
Tutor tutor = AiServices.create(Tutor.class, model);

// Использование
String explanation = tutor.explain("наследование в Java");
String review = tutor.reviewCode("public class Test { public static void main(...) }");
```

### 2.9. Основы prompt-инженерии

**Prompt-инженерия** — искусство формулирования запросов к LLM для получения качественных и предсказуемых результатов.

#### 2.9.1. Принципы эффективных промптов

1. **Конкретность.** Чётко формулируйте задачу:
   ```
   Плохо: "Расскажи про Java."
   Хорошо: "Объясни различие между перегрузкой и переопределением методов в Java. Приведи по одному примеру для каждого случая."
   ```

2. **Роль.** Задавайте модели роль:
   ```
   "Ты — опытный Java-разработчик с 10-летним стажем. Проанализируй следующий код и укажи на нарушения принципов SOLID."
   ```

3. **Формат ответа.** Указывайте желаемый формат:
   ```
   "Верни ответ в формате JSON со следующими полями: error_type, line_number, description, fix."
   ```

4. **Контекст.** Предоставляйте необходимый контекст:
   ```
   "Рассмотрим класс Student с полями name, age, group. Напиши метод equals() и hashCode(), соблюдая контракт."
   ```

5. **Примеры (few-shot).** Приводите примеры желаемого поведения:
   ```
   "Преобразуй текст в список ключевых слов.
   Пример 1: 'Java — это объектно-ориентированный язык' → ['Java', 'объектно-ориентированный', 'язык']
   Пример 2: '...' → [...]
   Текст: '...'"
   ```

#### 2.9.2. Структурированные ответы

Для получения структурированных данных от LLM применяется указание формата:

```java
String prompt = """
    Проанализий следующий код и верни ответ в формате JSON:
    {
        "issues": [
            {"severity": "ERROR|WARNING|INFO", "line": N, "message": "...", "suggestion": "..."}
        ],
        "overall_quality": "GOOD|ACCEPTABLE|POOR",
        "summary": "..."
    }

    Код:
    %s
    """.formatted(code);
```

Полученный JSON затем десериализуется с помощью Jackson.

### 2.10. ИИ как инструмент рефакторинга

Средства ИИ могут применяться не только для создания интеллектуальных функций, но и для улучшения собственного кода:

**Типичные задачи рефакторинга с помощью ИИ:**

1. **Улучшение читаемости.** «Улучши читаемость следующего кода, сохранив его функциональность».
2. **Применение паттернов.** «Примени паттерн Builder для класса с большим количеством параметров».
3. **Переход к функциональному стилю.** «Перепиши следующий императивный код с использованием Stream API».
4. **Извлечение методов.** «Разбей длинный метод на несколько небольших с понятными именами».
5. **Улучшение обработки ошибок.** «Добавь корректную обработку исключений в следующий код».
6. **Оптимизация.** «Предложи более эффективную реализацию следующего алгоритма».

**Пример промпта для рефакторинга:**

```
Ты — опытный Java-разработчик, специализирующийся на рефакторинге.
Проанализируй следующий код и предложи улучшения с точки зрения:
1. Принципов SOLID
2. Чистого кода (Clean Code)
3. Производительности
4. Обработки ошибок

Верни ответ в формате JSON:
{
    "issues": [...],
    "refactored_code": "...",
    "explanation": "..."
}

Код:
[вставить код]
```

### 2.11. Безопасность API-ключей

API-ключи — конфиденциальные данные, требующие защиты:

1. **Никогда не фиксируйте ключи в коде.** Используйте переменные окружения:
   ```java
   String apiKey = System.getenv("OPENAI_API_KEY");
   if (apiKey == null || apiKey.isBlank()) {
       throw new IllegalStateException("API-ключ не установлен");
   }
   ```

2. **Добавляйте файлы с ключами в `.gitignore`:**
   ```
   .env
   application-secrets.properties
   ```

3. **Ограничивайте права ключей.** В настройках API-провайдера устанавливайте минимально необходимые права.

4. **Регулярно ротируйте ключи.** Периодически меняйте API-ключи.

5. **Используйте разные ключи для разных сред.** Отдельные ключи для разработки, тестирования и продакшена.

## 3. Примеры выполнения

### 3.1. Пример 1. Клиент для публичного REST API (Open-Meteo)

Рассмотрим интеграцию с бесплатным погодным API Open-Meteo, не требующим API-ключа:

```java
import com.fasterxml.jackson.annotation.JsonIgnoreProperties;
import com.fasterxml.jackson.databind.ObjectMapper;
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.time.Duration;
import java.util.List;

/**
 * Клиент для получения прогноза погоды от Open-Meteo API
 */
public class WeatherClient {
    private static final String BASE_URL = "https://api.open-meteo.com/v1/forecast";

    private final HttpClient httpClient;
    private final ObjectMapper objectMapper;

    public WeatherClient() {
        this.httpClient = HttpClient.newBuilder()
            .connectTimeout(Duration.ofSeconds(10))
            .build();
        this.objectMapper = new ObjectMapper();
    }

    /**
     * Запись для ответа API
     */
    @JsonIgnoreProperties(ignoreUnknown = true)
    public record WeatherResponse(
        double latitude,
        double longitude,
        CurrentWeather current_weather
    ) {}

    @JsonIgnoreProperties(ignoreUnknown = true)
    public record CurrentWeather(
        double temperature,
        double windspeed,
        String winddirection,
        String weathercode,
        String time
    ) {}

    /**
     * Получает текущую погоду для заданных координат
     *
     * @param latitude  широта
     * @param longitude долгота
     * @return ответ с данными о погоде
     */
    public WeatherResponse getCurrentWeather(double latitude, double longitude)
            throws WeatherApiException {
        String url = String.format(
            "%s?latitude=%.4f&longitude=%.4f&current_weather=true",
            BASE_URL, latitude, longitude);

        HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create(url))
            .header("Accept", "application/json")
            .GET()
            .build();

        try {
            HttpResponse<String> response = httpClient.send(
                request, HttpResponse.BodyHandlers.ofString());

            if (response.statusCode() != 200) {
                throw new WeatherApiException(
                    "Ошибка API: статус " + response.statusCode());
            }

            return objectMapper.readValue(response.body(), WeatherResponse.class);
        } catch (Exception e) {
            throw new WeatherApiException("Ошибка получения погоды", e);
        }
    }

    /**
     * Исключение для ошибок API погоды
     */
    public static class WeatherApiException extends Exception {
        public WeatherApiException(String message) { super(message); }
        public WeatherApiException(String message, Throwable cause) { super(message, cause); }
    }

    /**
     * Демонстрационный класс
     */
    public static void main(String[] args) {
        WeatherClient client = new WeatherClient();

        // Москва
        try {
            WeatherResponse weather = client.getCurrentWeather(55.7558, 37.6173);
            System.out.println("=== Погода в Москве ===");
            System.out.printf("Температура: %.1f°C%n",
                weather.current_weather().temperature());
            System.out.printf("Скорость ветра: %.1f м/с%n",
                weather.current_weather().windspeed());
            System.out.println("Время измерения: " +
                weather.current_weather().time());
        } catch (WeatherApiException e) {
            System.err.println("Ошибка: " + e.getMessage());
        }
    }
}
```

### 3.2. Пример 2. Интеграция с LLM API через LangChain4j

```java
import dev.langchain4j.model.chat.ChatLanguageModel;
import dev.langchain4j.model.openai.OpenAiChatModel;
import dev.langchain4j.service.AiServices;
import dev.langchain4j.service.SystemMessage;
import dev.langchain4j.service.UserMessage;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.annotation.JsonProperty;
import java.util.List;

/**
 * AI-сервис для анализа кода
 */
public class CodeAnalyzer {
    /**
     * Интерфейс AI-сервиса для анализа кода
     */
    interface CodeReviewService {
        @SystemMessage("""
            Ты — опытный Java-разработчик, специализирующийся на code review.
            Анализируй код и возвращай ответ СТРОГО в формате JSON:
            {
                "issues": [
                    {"severity": "ERROR|WARNING|INFO", "line": N, "message": "...", "suggestion": "..."}
                ],
                "overall_quality": "GOOD|ACCEPTABLE|POOR",
                "summary": "Краткое резюме"
            }
            Не добавляй никаких пояснений вне JSON.
            """)
        String reviewCode(@UserMessage String code);
    }

    /**
     * Запись для результата анализа
     */
    public record CodeReviewResult(
        @JsonProperty("issues") List<Issue> issues,
        @JsonProperty("overall_quality") String overallQuality,
        @JsonProperty("summary") String summary
    ) {}

    public record Issue(
        @JsonProperty("severity") String severity,
        @JsonProperty("line") int line,
        @JsonProperty("message") String message,
        @JsonProperty("suggestion") String suggestion
    ) {}

    private final CodeReviewService reviewService;
    private final ObjectMapper objectMapper;

    public CodeReviewService(ChatLanguageModel model) {
        this.reviewService = AiServices.create(CodeReviewService.class, model);
        this.objectMapper = new ObjectMapper();
    }

    /**
     * Выполняет анализ кода и возвращает структурированный результат
     */
    public CodeReviewResult analyze(String code) throws Exception {
        String jsonResponse = reviewService.reviewCode(code);

        // Извлечение JSON из ответа (на случай, если модель добавит markdown)
        String cleanJson = extractJson(jsonResponse);

        return objectMapper.readValue(cleanJson, CodeReviewResult.class);
    }

    /**
     * Извлекает JSON из ответа, удаляя возможные markdown-обёртки
     */
    private String extractJson(String response) {
        String trimmed = response.trim();
        if (trimmed.startsWith("```json")) {
            trimmed = trimmed.substring(7);
        } else if (trimmed.startsWith("```")) {
            trimmed = trimmed.substring(3);
        }
        if (trimmed.endsWith("```")) {
            trimmed = trimmed.substring(0, trimmed.length() - 3);
        }
        return trimmed.trim();
    }

    /**
     * Демонстрационный класс
     */
    public static void main(String[] args) {
        // Создание модели (API-ключ из переменной окружения)
        String apiKey = System.getenv("OPENAI_API_KEY");
        if (apiKey == null) {
            System.err.println("Установите переменную окружения OPENAI_API_KEY");
            return;
        }

        ChatLanguageModel model = OpenAiChatModel.builder()
            .apiKey(apiKey)
            .modelName("gpt-4o-mini")
            .temperature(0.2)
            .build();

        CodeAnalyzer analyzer = new CodeAnalyzer(model);

        // Код для анализа
        String codeToReview = """
            public class Calculator {
                public double divide(double a, double b) {
                    return a / b;
                }

                public int[] parseNumbers(String s) {
                    String[] parts = s.split(",");
                    int[] result = new int[parts.length];
                    for (int i = 0; i < parts.length; i++) {
                        result[i] = Integer.parseInt(parts[i]);
                    }
                    return result;
                }
            }
            """;

        try {
            CodeReviewResult result = analyzer.analyze(codeToReview);

            System.out.println("=== Результат анализа кода ===");
            System.out.println("Общее качество: " + result.overallQuality());
            System.out.println("Резюме: " + result.summary());
            System.out.println("\nЗамечания:");
            for (Issue issue : result.issues()) {
                System.out.printf("[%s] Строка %d: %s%n",
                    issue.severity(), issue.line(), issue.message());
                System.out.printf("   Предложение: %s%n", issue.suggestion());
            }
        } catch (Exception e) {
            System.err.println("Ошибка анализа: " + e.getMessage());
        }
    }
}
```

### 3.3. Пример 3. Рефакторинг кода с помощью ИИ

```java
import dev.langchain4j.model.chat.ChatLanguageModel;
import dev.langchain4j.model.openai.OpenAiChatModel;

/**
 * Утилита для рефакторинга кода с помощью ИИ
 */
public class CodeRefactorer {

    private static final String REFACTOR_PROMPT = """
        Ты — эксперт по рефакторингу Java-кода. Проанализируй следующий код и предложи улучшения.

        ТРЕБОВАНИЯ К АНАЛИЗУ:
        1. Принципы SOLID (единственная ответственность, открытость/закрытость и т. д.)
        2. Чистый код (именование, длина методов, комментарии)
        3. Обработка ошибок и исключений
        4. Производительность (где это уместно)
        5. Применение современных возможностей Java (записи, Stream API, Optional)

        ФОРМАТ ОТВЕТА (строго JSON):
        {
            "issues": [
                {
                    "category": "SOLID|CLEAN_CODE|ERROR_HANDLING|PERFORMANCE|MODERN_JAVA",
                    "description": "Описание проблемы",
                    "location": "Где находится проблема",
                    "suggestion": "Предлагаемое решение"
                }
            ],
            "refactored_code": "Полностью переписанный код",
            "summary": "Краткое резюме изменений"
        }

        КОД ДЛЯ АНАЛИЗА:
        ```java
        %s
        ```
        """;

    private final ChatLanguageModel model;

    public CodeRefactorer(ChatLanguageModel model) {
        this.model = model;
    }

    /**
     * Выполняет рефакторинг кода
     */
    public RefactoringResult refactor(String code) {
        String prompt = REFACTOR_PROMPT.formatted(code);
        String response = model.generate(prompt);

        // Разбор ответа (упрощённо)
        return parseResponse(response);
    }

    private RefactoringResult parseResponse(String response) {
        // Извлечение JSON и разбор (см. предыдущий пример)
        // ...
        return new RefactoringResult(response);
    }

    public record RefactoringResult(String fullResponse) {
        // Методы для извлечения issues, refactored_code, summary
    }

    /**
     * Демонстрационный класс
     */
    public static void main(String[] args) {
        String apiKey = System.getenv("OPENAI_API_KEY");
        if (apiKey == null) {
            System.err.println("Установите переменную окружения OPENAI_API_KEY");
            return;
        }

        ChatLanguageModel model = OpenAiChatModel.builder()
            .apiKey(apiKey)
            .modelName("gpt-4o-mini")
            .temperature(0.3)
            .build();

        CodeRefactorer refactorer = new CodeRefactorer(model);

        // «Плохой» код для рефакторинга
        String badCode = """
            public class UserService {
                public void processUser(String data) {
                    String[] parts = data.split(",");
                    String name = parts[0];
                    int age = Integer.parseInt(parts[1]);
                    String email = parts[2];

                    if (age < 18) {
                        System.out.println("Несовершеннолетний");
                        return;
                    }
                    if (email == null || email.equals("")) {
                        System.out.println("Нет email");
                        return;
                    }

                    // Сохранение в базу (упрощённо)
                    System.out.println("Сохраняем: " + name + ", " + age + ", " + email);
                }
            }
            """;

        System.out.println("=== Исходный код ===");
        System.out.println(badCode);

        System.out.println("\n=== Анализ и рефакторинг ===");
        RefactoringResult result = refactorer.refactor(badCode);
        System.out.println(result.fullResponse());
    }
}
```

## 4. Задания на паре

### Задание 4.1. Клиент для публичного REST API

Разработайте клиент для работы с публичным REST API по выбору. Возможные варианты:

- **Open-Meteo** (погода, без ключа) — https://open-meteo.com/
- **JSONPlaceholder** (тестовый API) — https://jsonplaceholder.typicode.com/
- **NumbersAPI** (факты о числах) — http://numbersapi.com/
- **Open Library** (книги) — https://openlibrary.org/
- **REST Countries** (страны) — https://restcountries.com/
- **CoinGecko** (криптовалюты, без ключа) — https://www.coingecko.com/

**Требования к реализации:**

1. Создайте класс-клиент, инкапсулирующий работу с API.
2. Реализуйте не менее 3 методов для различных операций (получение списка, получение одного элемента, поиск).
3. Применяйте записи (`record`) для представления ответов API.
4. Обрабатывайте ошибки HTTP и сетевые сбои.
5. Применяйте Jackson для десериализации JSON-ответов.
6. Реализуйте таймауты и конфигурацию клиента.
7. Создайте демонстрационный класс, показывающий работу с API.

**Пример выполнения программы:**

```
=== Клиент для REST Countries API ===

Поиск страны по коду RU:
  Название: Russia
  Столица: Moscow
  Население: 144,000,000
  Регион: Europe
  Языки: Russian

Список стран региона Europe (первые 5):
  - Germany (Berlin) — 83,000,000
  - France (Paris) — 67,000,000
  ...
```

---

### Задание 4.2. Простой чат-бот с LLM

Разработайте консольный чат-бот, использующий LLM API.

**Требования к реализации:**

1. Создайте класс `ChatBot`, инкапсулирующий взаимодействие с LLM.
2. Реализуйте поддержку диалога с сохранением истории сообщений (контекста).
3. Задайте системный промпт, определяющий роль бота (например, «репетитор по Java», «помощник в программировании», «консультант по продукту»).
4. Реализуйте команды пользователя:
   - `/clear` — очистить историю;
   - `/history` — показать историю сообщений;
   - `/save <filename>` — сохранить историю в файл;
   - `/quit` — завершить работу.
5. Обрабатывайте ошибки API (сетевые сбои, превышение лимита, некорректные ответы).
6. Применяйте переменные окружения для хранения API-ключа.
7. Реализуйте поддержку как облачных (OpenAI, YandexGPT), так и локальных (Ollama) моделей.

**Пример выполнения программы:**

```
=== Java Tutor Bot ===
Введите сообщение (или /help для списка команд):

> Что такое полиморфизм?
Бот: Полиморфизм в Java — это способность объектов с различной
внутренней структурой иметь общий интерфейс...

> Приведи пример
Бот: Рассмотрим иерархию классов Shape → Circle, Rectangle...

> /history
[system]: Ты — репетитор по Java. Отвечай кратко и с примерами.
[user]: Что такое полиморфизм?
[assistant]: Полиморфизм в Java — это...
[user]: Приведи пример
[assistant]: Рассмотрим иерархию...

> /quit
История сохранена в chat_history_2026-08-20.json
```

---

### Задание 4.3. Интеллектуальный анализатор с интеграцией LLM

Разработайте приложение, использующее LLM для анализа данных предметной области.

**Варианты применения (по выбору):**

- **Анализатор кода** — анализ Java-кода на соответствие принципам SOLID и Clean Code;
- **Генератор тестов** — создание JUnit-тестов для заданного класса;
- **Рецензент текста** — проверка текста на грамотность, стиль, связность;
- **Классификатор** — отнесение текста к определённой категории;
- **Извлекатель данных** — извлечение структурированных данных из неструктурированного текста.

**Требования к реализации:**

1. Создайте класс-сервис, инкапсулирующий взаимодействие с LLM.
2. Разработайте тщательно сформулированный системный промпт.
3. Требуйте от модели ответа в формате JSON с чёткой схемой.
4. Реализуйте десериализацию ответа модели в записи Java.
5. Обрабатывайте некорректные ответы модели (не-JSON, неожиданная структура).
6. Реализуйте повторные попытки при ошибках (retry policy).
7. Создайте демонстрационный класс с не менее чем 5 сценариями.

**Пример выполнения программы (анализатор резюме):**

```
=== Анализатор резюме ===

Введите текст резюме или путь к файлу:
> resume.txt

=== Результат анализа ===

Кандидат: Иванов Иван Иванович
Опыт: 5 лет
Навыки: Java, Spring, SQL, Git
Уровень: Middle

Рекомендация:
  Сильные стороны:
    - Опыт работы с Spring Boot
    - Знание SQL и реляционных БД
  Слабые стороны:
    - Отсутствие опыта с микросервисами
    - Не указан уровень английского
  Общая оценка: 7/10
```

## 5. Задание для самостоятельной работы

Разработать приложение, интегрирующееся с внешним API и/или LLM-сервисом, согласно своему варианту. Требования:

1. Интеграция с внешним API (публичным REST API или LLM API).
2. Применение `java.net.http.HttpClient` для выполнения HTTP-запросов.
3. Применение Jackson для сериализации и десериализации JSON.
4. Обработка ошибок HTTP и сетевых сбоев.
5. Применение переменных окружения для хранения API-ключей.
6. Не менее 5 сценариев в демонстрационном классе.
7. Применение принципов ООП (инкапсуляция, абстракция).

### Варианты заданий

**Вариант 1.** Приложение «Погодный помощник». Получение прогноза погоды для нескольких городов, сравнение погоды, рекомендации по одежде с помощью LLM.

**Вариант 2.** Приложение «Переводчик». Перевод текста между языками с помощью LLM, сохранение истории переводов, анализ качества перевода.

**Вариант 3.** Приложение «Анализатор тональности». Анализ тональности текста (позитивный, нейтральный, негативный) с помощью LLM, обработка отзывов.

**Вариант 4.** Приложение «Генератор тестов». Создание JUnit-тестов для Java-класса с помощью LLM, анализ покрытия.

**Вариант 5.** Приложение «Рецензент кода». Анализ Java-кода на соответствие стандартам, предложения по улучшению.

**Вариант 6.** Приложение «Саммари текста». Создание краткого содержания длинного текста с помощью LLM, извлечение ключевых тезисов.

**Вариант 7.** Приложение «Помощник в изучении Java». Объяснение концепций Java, генерация примеров, проверка понимания.

**Вариант 8.** Приложение «Классификатор документов». Отнесение документов к категориям с помощью LLM, извлечение метаданных.

**Вариант 9.** Приложение «Генератор описаний товаров». Создание описаний для товаров интернет-магазина с помощью LLM.

**Вариант 10.** Приложение «Помощник в написании писем». Генерация деловых писем по заданным параметрам с помощью LLM.

**Вариант 11.** Приложение «Анализатор новостей». Получение новостей из публичного API, суммаризация и категоризация с помощью LLM.

**Вариант 12.** Приложение «Помощник по рецептам». Получение рецептов из API, генерация рекомендаций по замене ингредиентов с помощью LLM.

**Вариант 13.** Приложение «Анализатор резюме». Извлечение структурированных данных из резюме с помощью LLM, оценка кандидатов.

**Вариант 14.** Приложение «Помощник в планировании». Создание плана проекта или обучения с помощью LLM, декомпозиция задач.

**Вариант 15.** Приложение «Генератор тестовых вопросов». Создание вопросов по заданной теме с помощью LLM, различных уровней сложности.

**Вариант 16.** Приложение «Анализатор тональности отзывов». Получение отзывов о продукте, анализ тональности, выявление проблем.

**Вариант 17.** Приложение «Помощник в отладке». Анализ сообщений об ошибках, предложение решений с помощью LLM.

**Вариант 18.** Приложение «Генератор документации». Создание Javadoc-комментариев для Java-кода с помощью LLM.

**Вариант 19.** Приложение «Переводчик технического текста». Перевод технической документации с сохранением терминологии.

**Вариант 20.** Приложение «Анализатор зависимостей». Анализ Maven/Gradle зависимостей, рекомендации по обновлению с помощью LLM.

**Вариант 21.** Приложение «Помощник по SQL». Генерация SQL-запросов по естественному описанию, объяснение существующих запросов.

**Вариант 22.** Приложение «Генератор тестовых данных». Создание реалистичных тестовых данных (имена, адреса, даты) с помощью LLM.

**Вариант 23.** Приложение «Анализатор логов». Анализ лог-файлов, выявление аномалий и ошибок с помощью LLM.

**Вариант 24.** Приложение «Помощник в написании README». Генерация README-файлов для GitHub-репозиториев с помощью LLM.

**Вариант 25.** Приложение «Рецензент эссе». Анализ эссе на структуру, аргументацию, грамматику с помощью LLM.

**Вариант 26.** Приложение «Генератор commit-сообщений». Создание информативных commit-сообщений по diff с помощью LLM.

**Вариант 27.** Приложение «Помощник в миграции кода». Помощь в переходе между версиями Java или фреймворками с помощью LLM.

**Вариант 28.** Приложение «Анализатор производительности». Анализ кода на предмет потенциальных проблем производительности с помощью LLM.

**Вариант 29.** Приложение «Генератор моков». Создание mock-объектов для модульного тестирования с помощью LLM.

**Вариант 30.** Приложение «Помощник в выборе технологий». Рекомендации по выбору стека технологий для проекта с помощью LLM.

## 6. Методические указания к самостоятельной работе

1. **Выбор API.** При выборе внешнего API учитывайте:
   - наличие бесплатного тарифа или возможности локального запуска;
   - качество документации;
   - стабильность сервиса;
   - ограничения (лимиты запросов, размер ответа).

2. **Архитектура клиента.** Применяйте принципы ООП при проектировании клиента API:
   - инкапсулируйте работу с HTTP в отдельном классе;
   - применяйте записи для представления DTO (Data Transfer Objects);
   - выносите обработку ошибок в отдельный слой;
   - применяйте интерфейсы для абстрагирования от конкретного провайдера.

3. **Конфигурация.** Реализуйте гибкую конфигурацию:
   - API-ключи — через переменные окружения;
   - URL API, таймауты, параметры — через конфигурационные файлы или конструктор;
   - поддержка различных провайдеров (OpenAI, Ollama, YandexGPT).

4. **Обработка ошибок.** Реализуйте надёжную обработку ошибок:
   - сетевые сбои (IOException, HttpTimeoutException);
   - ошибки API (не-2xx статусы);
   - некорректные ответы (не-JSON, неожиданная структура);
   - повторные попытки при временных ошибках (retry policy).

5. **Prompt-инженерия.** При работе с LLM:
   - формулируйте чёткие, конкретные промпты;
   - указывайте желаемый формат ответа;
   - применяйте системные промпты для задания роли;
   - тестируйте промпты на различных входных данных;
   - обрабатывайте случаи, когда модель не следует формату.

6. **Безопасность.** Соблюдайте принципы безопасности:
   - никогда не фиксируйте API-ключи в коде;
   - не передавайте конфиденциальные данные во внешние сервисы;
   - применяйте HTTPS для всех запросов;
   - валидируйте ответы API перед использованием.

7. **Тестирование.** Для тестирования без реальных запросов:
   - применяйте mock-серверы (WireMock, MockServer);
   - используйте заранее подготовленные JSON-файлы;
   - применяйте локальные модели (Ollama);
   - реализуйте интерфейсы для подмены реальных клиентов тестовыми.

8. **Производительность.** Учитывайте особенности работы с API:
   - применяйте асинхронные запросы для параллельной обработки;
   - реализуйте кэширование часто запрашиваемых данных;
   - соблюдайте лимиты запросов (rate limiting);
   - применяйте пулы соединений для множественных запросов.

9. **Применение ИИ.** При использовании ИИ-ассистентов для разработки:
   - генерируйте по отдельности клиенты API, модели данных и демонстрационные классы;
   - обязательно проверяйте сгенерированный код на корректность обработки ошибок;
   - не делегируйте ИИ проектирование архитектуры без понимания;
   - документируйте случаи, когда ИИ предложил некорректное решение.

10. **Оформление отчёта.** Отчёт должен содержать:
    - листинги всех файлов проекта с комментариями;
    - описание использованного API (ссылки на документацию);
    - примеры запросов и ответов API;
    - протокол работы демонстрационного класса;
    - описание применённых промптов (для LLM-интеграции);
    - обоснование архитектурных решений;
    - ответы на контрольные вопросы;
    - выводы по проделанной работе.

## 7. Контрольные вопросы

1. Что такое протокол HTTP? Каковы его основные методы?
2. Какие коды состояния HTTP существуют? Что означают коды 2xx, 4xx, 5xx?
3. Что такое REST-архитектура? Каковы её основные принципы?
4. Как проектируется RESTful API? Каким URL соответствуют различные операции?
5. Что такое `java.net.http.HttpClient`? Каковы его основные возможности?
6. Каким образом выполняются синхронные и асинхронные HTTP-запросы?
7. Какие типы ошибок могут возникать при работе с внешними API?
8. Что такое большие языковые модели (LLM)? Приведите примеры.
9. Какова общая схема взаимодействия с LLM API?
10. Что такое промпт? Каковы принципы эффективных промптов?
11. Какие роли сообщений существуют в диалоговых LLM?
12. Что такое температура генерации? Как она влияет на ответы модели?
13. Что такое LangChain4j? Каковы его основные возможности?
14. Что такое AI Services в LangChain4j? Как они применяются?
15. Каким образом следует хранить API-ключи?
16. Что такое mock-серверы? Для каких целей они применяются?
17. Каким образом можно получить структурированный ответ от LLM?
18. Каковы этические аспекты применения LLM в приложениях?
19. Какие альтернативы облачным LLM API существуют?
20. Каким образом ИИ может применяться для рефакторинга кода?

## 8. Рекомендуемые источники

1. Хорстманн К. *Java. Библиотека профессионала. Том 2.* — М.: Вильямс. — Глава 2 (Сетевое программирование).
2. Официальная документация Java HttpClient. URL: https://docs.oracle.com/en/java/javase/17/docs/api/java.net.http/module-summary.html
3. Документация LangChain4j. URL: https://docs.langchain4j.dev/
4. OpenAI API Documentation. URL: https://platform.openai.com/docs/api-reference
5. Ollama Documentation. URL: https://ollama.ai/library
6. REST API Tutorial. URL: https://restfulapi.net/
7. Baeldung. Java HttpClient. URL: https://www.baeldung.com/java-9-http-client
8. Baeldung. Guide to OpenAI API with Java. URL: https://www.baeldung.com/java-openai-api
9. Prompt Engineering Guide. URL: https://www.promptingguide.ai/
10. Martin R. C. *Clean Architecture.* — Prentice Hall. — Разделы о внешних сервисах и интеграции.
