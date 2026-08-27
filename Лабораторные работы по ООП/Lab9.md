# Лабораторная работа №9. Многопоточность: Runnable, Thread, synchronized

## 1. Паспорт работы

| Параметр | Значение |
|----------|----------|
| Номер занятия | 9 из 17 |
| Блок | 4. Многопоточность |
| Продолжительность | 2 академических часа |
| Форма выполнения | Индивидуальная |
| ИИ-инструмент | YandexGPT / GigaChat / JetBrains AI Assistant |

### 1.1. Цель работы

Освоить базовые механизмы создания и управления потоками в Java, изучить проблему гонки данных (race condition) и способы её решения с помощью синхронизации, научиться сравнивать производительность синхронизированного и несинхронизированного кода.

### 1.2. Задачи работы

1. Изучить способы создания потоков: `Thread` и `Runnable`.
2. Освоить запуск потоков методом `start()`.
3. Изучить проблему гонки данных при конкурентном доступе.
4. Освоить ключевое слово `synchronized` для синхронизации.
5. Изучить альтернативу синхронизации через `ReentrantLock`.
6. Научиться измерять производительность многопоточного кода.
7. Развить навыки отладки многопоточных приложений.

### 1.3. Оснащение

- JDK 17 или выше;
- IntelliJ IDEA Community Edition;
- Git;
- доступ к YandexGPT или GigaChat.

---

## 2. Теоретический конспект

### 2.1. Процесс vs Поток

**Процесс** — экземпляр выполняющейся программы, имеющий собственное адресное пространство памяти.

**Поток** — легковесный процесс, выполняющийся в рамках одного процесса. Потоки одного процесса разделяют общую память.

**Зачем нужна многопоточность:**
- **CPU-bound задачи** — использование нескольких ядер процессора для ускорения вычислений.
- **I/O-bound задачи** — выполнение операций ввода-вывода (чтение файлов, сетевые запросы) без блокировки основного потока.
- **Параллельная обработка** данных.

### 2.2. Создание потоков

**Способ 1: Наследование от Thread**

```java
class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("Поток " + Thread.currentThread().getName() + " выполняется");
    }
}

// Запуск
MyThread thread = new MyThread();
thread.start(); // Запускает новый поток
// thread.run(); // НЕ ЗАПУСКАЕТ новый поток! Выполняется в текущем потоке
```

**Способ 2: Реализация Runnable**

```java
class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("Поток " + Thread.currentThread().getName() + " выполняется");
    }
}

// Запуск
Thread thread = new Thread(new MyRunnable());
thread.start();
```

**Способ 3: Лямбда-выражение (Java 8+)**

```java
Thread thread = new Thread(() -> {
    System.out.println("Поток " + Thread.currentThread().getName() + " выполняется");
});
thread.start();
```

### 2.3. Состояния потока

```
NEW → RUNNABLE → BLOCKED/WAITING/TIMED_WAITING → TERMINATED
```

| Состояние | Описание |
|-----------|----------|
| NEW | Поток создан, но не запущен |
| RUNNABLE | Поток выполняется или готов к выполнению |
| BLOCKED | Поток заблокирован (ожидает монитор) |
| WAITING | Поток ожидает уведомления (wait()) |
| TIMED_WAITING | Поток ожидает с таймаутом (sleep(), wait(timeout)) |
| TERMINATED | Поток завершил выполнение |

### 2.4. Проблема гонки данных (Race Condition)

Гонка данных возникает, когда несколько потоков одновременно обращаются к общим данным, и хотя бы один поток выполняет запись.

```java
public class Counter {
    private int count = 0;
    
    public void increment() {
        count++; // НЕ АТОМАРНО! Три операции:
        // 1. Чтение count
        // 2. Увеличение на 1
        // 3. Запись count
    }
    
    public int getCount() { return count; }
}
```

**Проблема:** Если два потока одновременно вызывают `increment()`, возможна потеря обновления:

```
Поток A: читает count = 5
Поток B: читает count = 5
Поток A: записывает count = 6
Поток B: записывает count = 6
Итог: 6 (хотя должно быть 7!)
```

### 2.5. Синхронизация с synchronized

**Синхронизированный метод:**

```java
public synchronized void increment() {
    count++; // Теперь атомарно
}
```

**Синхронизированный блок:**

```java
public void increment() {
    synchronized (this) {
        count++;
    }
}
```

**Синхронизированный статический метод:**

```java
public static synchronized void incrementStatic() {
    // Синхронизация на объекте класса
}
```

### 2.6. volatile

`volatile` гарантирует видимость изменений между потоками, но не обеспечивает атомарность.

```java
private volatile boolean running = true;

public void stop() {
    running = false; // Изменение видно всем потокам
}
```

**Когда использовать volatile:**
- Только для простых операций чтения/записи.
- Для флагов состояния.
- Для публикации неизменяемых объектов.

### 2.7. wait(), notify(), notifyAll()

Используются для взаимодействия между потоками:

```java
// Поток-производитель
synchronized (queue) {
    while (queue.isFull()) {
        queue.wait(); // Ожидание места
    }
    queue.add(item);
    queue.notifyAll(); // Уведомление потребителей
}

// Поток-потребитель
synchronized (queue) {
    while (queue.isEmpty()) {
        queue.wait(); // Ожидание данных
    }
    item = queue.remove();
    queue.notifyAll(); // Уведомление производителей
}
```

### 2.8. ReentrantLock

Альтернатива `synchronized` с большей гибкостью:

```java
import java.util.concurrent.locks.ReentrantLock;

public class BankAccount {
    private final ReentrantLock lock = new ReentrantLock();
    private double balance;
    
    public void deposit(double amount) {
        lock.lock();
        try {
            balance += amount;
        } finally {
            lock.unlock(); // Важно: unlock в finally
        }
    }
}
```

**Преимущества ReentrantLock:**
- Можно прервать ожидание (`lockInterruptibly()`)
- Таймаут при попытке захвата (`tryLock(timeout, unit)`)
- Более гибкие условия (`newCondition()`)

### 2.9. Паттерн Producer-Consumer

```
Producer → [Buffer/Queue] → Consumer
   ↓            ↓              ↓
Создаёт      Хранит       Обрабатывает
данные       данные        данные
```

**Требования:**
- Producer не должен добавлять данные в полную очередь.
- Consumer не должен забирать данные из пустой очереди.
- Доступ к очереди должен быть синхронизирован.

---

## 3. Задание на паре

### Задача. Банковский счёт и гонка данных

1. **Создать класс `BankAccount`:**
   - Поле `balance` (double).
   - Методы `deposit(double amount)` и `withdraw(double amount)`.
   - Метод `getBalance()`.

2. **Создать класс `Client` (Runnable):**
   - Выполняет 1000 операций: случайное пополнение или снятие.
   - Сумма операции: 1-100.
   - При снятии проверять, что баланс не уходит в минус.

3. **Реализовать без синхронизации:**
   - Запустить 10 потоков-клиентов.
   - Продемонстрировать некорректный итоговый баланс (race condition).

4. **Добавить синхронизацию:**
   - Использовать `synchronized` для методов `deposit` и `withdraw`.
   - Убедиться, что баланс сходится.

5. **Сравнить производительность:**
   - Замерить время выполнения синхронизированной версии.
   - Сравнить с несинхронизированной.

6. **Реализовать альтернативу через `ReentrantLock`:**
   - Сравнить по времени и читаемости.

**Пример выполнения:**
```
=== Без синхронизации ===
Итоговый баланс: 15423.50 (ожидалось: ~15500.00)
Ошибка: ~76.50

=== С synchronized ===
Итоговый баланс: 15499.50 (ожидалось: ~15500.00)
Время выполнения: 125 мс

=== С ReentrantLock ===
Итоговый баланс: 15499.50 (ожидалось: ~15500.00)
Время выполнения: 132 мс
```

### Применение ИИ-инструмента

**Промпт для YandexGPT:**
```
Объясни причину расхождения баланса в несинхронизированной версии банковского счёта.

Код:
public class BankAccount {
    private double balance;
    
    public void deposit(double amount) {
        balance += amount;
    }
    
    public void withdraw(double amount) {
        if (balance >= amount) {
            balance -= amount;
        }
    }
}

Запускается 10 потоков, каждый выполняет 1000 операций.
Почему итоговый баланс не совпадает с ожидаемым?
```

**Анализ результата:**
- Проверить качество объяснения.
- Проверить, упоминает ли ИИ операции чтения-изменения-записи.
- Проверить, упоминает ли ИИ проблему видимости (cache).
- Проверить, даёт ли ИИ рекомендации по исправлению.

---

## 4. Индивидуальные задания (30 вариантов)

Каждый вариант содержит:
- Класс с общими данными.
- Задачу для параллельного выполнения.
- Количество потоков и операций.

---

### Вариант 1. Банковский счёт

**Класс:** `BankAccount` (balance)

**Операция:** Пополнение или снятие (случайно)

**Потоки:** 10

**Операций:** 1000 каждый

**Особенность:** Не уходить в минус

---

### Вариант 2. Счётчик

**Класс:** `Counter` (count)

**Операция:** Инкремент

**Потоки:** 5

**Операций:** 100000 каждый

**Особенность:** Сравнение volatile vs synchronized

---

### Вариант 3. Список

**Класс:** `SharedList` (List<Integer>)

**Операция:** Добавление элемента

**Потоки:** 8

**Операций:** 5000 каждый

**Особенность:** ArrayList vs Vector

---

### Вариант 4. Банкомат

**Класс:** `ATM` (balance, transactionCount)

**Операция:** Пополнение и снятие

**Потоки:** 5

**Операций:** 2000 каждый

**Особенность:** ReentrantLock vs synchronized

---

### Вариант 5. Склад

**Класс:** `Warehouse` (stock)

**Операция:** Добавление и удаление товара

**Потоки:** 3 производителя, 3 потребителя

**Операций:** 1000 каждый

**Особенность:** Producer-Consumer

---

### Вариант 6. Матрица

**Класс:** `Matrix` (double[][])

**Операция:** Сложение матриц

**Потоки:** 4

**Операций:** 1 (разбиение на части)

**Особенность:** Параллельные вычисления

---

### Вариант 7. Гоночная машина

**Класс:** `Car` (speed, fuel)

**Операция:** Ускорение и расход топлива

**Потоки:** 10

**Операций:** 500 каждый

**Особенность:** Проверка ограничений

---

### Вариант 8. Биржа

**Класс:** `StockExchange` (price, volume)

**Операция:** Покупка и продажа акций

**Потоки:** 8

**Операций:** 1000 каждый

**Особенность:** Цена зависит от объёма

---

### Вариант 9. Очередь

**Класс:** `Queue` (LinkedList)

**Операция:** Добавление и извлечение

**Потоки:** 4 производителя, 4 потребителя

**Операций:** 2000 каждый

**Особенность:** wait/notify

---

### Вариант 10. Парковка

**Класс:** `Parking` (spots)

**Операция:** Заезд и выезд

**Потоки:** 15

**Операций:** 500 каждый

**Особенность:** Ограниченное количество мест

---

### Вариант 11. Библиотека

**Класс:** `Library` (books)

**Операция:** Взятие и возврат книги

**Потоки:** 10

**Операций:** 300 каждый

**Особенность:** Уникальные книги (Set)

---

### Вариант 12. Касса

**Класс:** `Cashier` (total, count)

**Операция:** Продажа билета

**Потоки:** 6

**Операций:** 1000 каждый

**Особенность:** Лимит билетов

---

### Вариант 13. Игровой счёт

**Класс:** `GameScore` (score, level)

**Операция:** Начисление очков

**Потоки:** 8

**Операций:** 2000 каждый

**Особенность:** Повышение уровня

---

### Вариант 14. Менеджер задач

**Класс:** `TaskManager` (tasks)

**Операция:** Добавление и выполнение

**Потоки:** 5

**Операций:** 1000 каждый

**Особенность:** Пул потоков

---

### Вариант 15. Теплица

**Класс:** `Greenhouse` (temperature, humidity)

**Операция:** Изменение параметров

**Потоки:** 4

**Операций:** 1500 каждый

**Особенность:** Диапазон значений

---

### Вариант 16. Футбольный матч

**Класс:** `FootballMatch` (score1, score2)

**Операция:** Гол

**Потоки:** 5

**Операций:** 100 каждый

**Особенность:** Команды-потоки

---

### Вариант 17. Лифт

**Класс:** `Elevator` (floor, passengers)

**Операция:** Перемещение

**Потоки:** 6

**Операций:** 300 каждый

**Особенность:** Ограничение пассажиров

---

### Вариант 18. Кафе

**Класс:** `Cafe` (orders, cooks)

**Операция:** Приём и выполнение заказов

**Потоки:** 4

**Операций:** 500 каждый

**Особенность:** Producer-Consumer

---

### Вариант 19. Светофор

**Класс:** `TrafficLight` (state)

**Операция:** Смена состояния

**Потоки:** 3

**Операций:** 1000 каждый

**Особенность:** Очерёдность состояний

---

### Вариант 20. Автобус

**Класс:** `Bus` (passengers)

**Операция:** Вход и выход

**Потоки:** 10

**Операций:** 500 каждый

**Особенность:** Ограничение мест

---

### Вариант 21. Ресторан

**Класс:** `Restaurant` (tables, waiters)

**Операция:** Посадка и обслуживание

**Потоки:** 8

**Операций:** 200 каждый

**Особенность:** ReentrantLock

---

### Вариант 22. Банк

**Класс:** `Bank` (accounts)

**Операция:** Перевод между счетами

**Потоки:** 5

**Операций:** 500 каждый

**Особенность:** Два счёта (deadlock риск)

---

### Вариант 23. Поиск

**Класс:** `ArraySearch` (int[])

**Операция:** Поиск элемента

**Потоки:** 4

**Операций:** 1

**Особенность:** Разбиение массива

---

### Вариант 24. Космический корабль

**Класс:** `Spaceship` (fuel, health)

**Операция:** Маневры

**Потоки:** 6

**Операций:** 400 каждый

**Особенность:** Ограничения ресурсов

---

### Вариант 25. Строительство

**Класс:** `Construction` (materials, workers)

**Операция:** Использование материалов

**Потоки:** 8

**Операций:** 300 каждый

**Особенность:** wait/notify

---

### Вариант 26. Билеты в кино

**Класс:** `Cinema` (seats)

**Операция:** Бронирование места

**Потоки:** 10

**Операций:** 50 каждый

**Особенность:** Уникальные места

---

### Вариант 27. Онлайн-магазин

**Класс:** `Shop` (items)

**Операция:** Покупка товара

**Потоки:** 5

**Операций:** 1000 каждый

**Особенность:** Ограничение количества

---

### Вариант 28. Спортзал

**Класс:** `Gym` (machines)

**Операция:** Занятие тренажёром

**Потоки:** 12

**Операций:** 200 каждый

**Особенность:** Таймаут ожидания

---

### Вариант 29. Почта

**Класс:** `PostOffice` (letters)

**Операция:** Отправка и доставка

**Потоки:** 4

**Операций:** 1000 каждый

**Особенность:** BlockingQueue

---

### Вариант 30. Завод

**Класс:** `Factory` (details)

**Операция:** Производство и сборка

**Потоки:** 6

**Операций:** 500 каждый

**Особенность:** Два этапа производства

---

## 5. Методические указания

### 5.1. Структура проекта

```
src/
├── main/
│   ├── java/
│   │   ├── model/
│   │   │   └── BankAccount.java
│   │   ├── thread/
│   │   │   └── ClientThread.java
│   │   ├── executor/
│   │   │   └── AccountSimulator.java
│   │   └── Main.java
│   └── resources/
│       └── logback.xml
└── test/
    └── java/
        └── model/
            └── BankAccountTest.java
```

### 5.2. Шаблон BankAccount (без синхронизации)

```java
package model;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class BankAccount {
    private static final Logger logger = LoggerFactory.getLogger(BankAccount.class);
    private double balance;
    private final String accountNumber;
    
    public BankAccount(String accountNumber, double initialBalance) {
        this.accountNumber = accountNumber;
        this.balance = initialBalance;
        logger.info("Счёт {} открыт с балансом {}", accountNumber, initialBalance);
    }
    
    public void deposit(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Сумма должна быть положительной");
        }
        balance += amount;
        logger.debug("Счёт {}: пополнение на {}, баланс: {}", accountNumber, amount, balance);
    }
    
    public void withdraw(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Сумма должна быть положительной");
        }
        if (balance >= amount) {
            balance -= amount;
            logger.debug("Счёт {}: снятие на {}, баланс: {}", accountNumber, amount, balance);
        } else {
            logger.warn("Счёт {}: недостаточно средств для снятия {}", accountNumber, amount);
        }
    }
    
    public double getBalance() {
        return balance;
    }
    
    @Override
    public String toString() {
        return String.format("BankAccount{accountNumber='%s', balance=%.2f}", accountNumber, balance);
    }
}
```

### 5.3. Шаблон ClientThread

```java
package thread;

import model.BankAccount;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.Random;

public class ClientThread implements Runnable {
    private static final Logger logger = LoggerFactory.getLogger(ClientThread.class);
    private final BankAccount account;
    private final int operationsCount;
    private final Random random = new Random();
    private int successfulOperations = 0;
    private int failedOperations = 0;
    
    public ClientThread(BankAccount account, int operationsCount) {
        this.account = account;
        this.operationsCount = operationsCount;
    }
    
    @Override
    public void run() {
        String threadName = Thread.currentThread().getName();
        logger.info("{} начал работу", threadName);
        
        long startTime = System.currentTimeMillis();
        
        for (int i = 0; i < operationsCount; i++) {
            double amount = 1 + random.nextDouble() * 99; // 1-100
            boolean isDeposit = random.nextBoolean();
            
            try {
                if (isDeposit) {
                    account.deposit(amount);
                    successfulOperations++;
                } else {
                    account.withdraw(amount);
                    successfulOperations++;
                }
            } catch (Exception e) {
                failedOperations++;
                logger.error("{} ошибка при операции: {}", threadName, e.getMessage());
            }
        }
        
        long duration = System.currentTimeMillis() - startTime;
        logger.info("{} завершил работу. Операций: {}, успешно: {}, ошибок: {}, время: {} мс",
                threadName, operationsCount, successfulOperations, failedOperations, duration);
    }
    
    public int getSuccessfulOperations() { return successfulOperations; }
    public int getFailedOperations() { return failedOperations; }
}
```

### 5.4. Шаблон BankAccount (с synchronized)

```java
package model;

public class BankAccount {
    private double balance;
    private final String accountNumber;
    
    public BankAccount(String accountNumber, double initialBalance) {
        this.accountNumber = accountNumber;
        this.balance = initialBalance;
    }
    
    public synchronized void deposit(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Сумма должна быть положительной");
        }
        balance += amount;
    }
    
    public synchronized void withdraw(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Сумма должна быть положительной");
        }
        if (balance >= amount) {
            balance -= amount;
        }
    }
    
    public synchronized double getBalance() {
        return balance;
    }
}
```

### 5.5. Шаблон BankAccount (с ReentrantLock)

```java
package model;

import java.util.concurrent.locks.ReentrantLock;

public class BankAccount {
    private final ReentrantLock lock = new ReentrantLock();
    private double balance;
    private final String accountNumber;
    
    public BankAccount(String accountNumber, double initialBalance) {
        this.accountNumber = accountNumber;
        this.balance = initialBalance;
    }
    
    public void deposit(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Сумма должна быть положительной");
        }
        lock.lock();
        try {
            balance += amount;
        } finally {
            lock.unlock();
        }
    }
    
    public void withdraw(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Сумма должна быть положительной");
        }
        lock.lock();
        try {
            if (balance >= amount) {
                balance -= amount;
            }
        } finally {
            lock.unlock();
        }
    }
    
    public double getBalance() {
        lock.lock();
        try {
            return balance;
        } finally {
            lock.unlock();
        }
    }
}
```

### 5.6. Шаблон симулятора

```java
package executor;

import model.BankAccount;
import thread.ClientThread;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

public class AccountSimulator {
    private final BankAccount account;
    private final int threadCount;
    private final int operationsPerThread;
    private final String mode;
    
    public AccountSimulator(BankAccount account, int threadCount, int operationsPerThread, String mode) {
        this.account = account;
        this.threadCount = threadCount;
        this.operationsPerThread = operationsPerThread;
        this.mode = mode;
    }
    
    public SimulationResult run() {
        System.out.println("\n=== " + mode + " ===");
        
        ExecutorService executor = Executors.newFixedThreadPool(threadCount);
        CountDownLatch latch = new CountDownLatch(threadCount);
        List<ClientThread> clients = new ArrayList<>();
        
        long startTime = System.currentTimeMillis();
        
        for (int i = 0; i < threadCount; i++) {
            ClientThread client = new ClientThread(account, operationsPerThread);
            clients.add(client);
            executor.submit(() -> {
                try {
                    client.run();
                } finally {
                    latch.countDown();
                }
            });
        }
        
        try {
            latch.await(30, TimeUnit.SECONDS);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
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
        
        int totalOperations = clients.stream()
            .mapToInt(ClientThread::getSuccessfulOperations)
            .sum();
        int totalErrors = clients.stream()
            .mapToInt(ClientThread::getFailedOperations)
            .sum();
        
        return new SimulationResult(
            mode,
            account.getBalance(),
            totalOperations,
            totalErrors,
            duration
        );
    }
    
    public static class SimulationResult {
        private final String mode;
        private final double finalBalance;
        private final int totalOperations;
        private final int totalErrors;
        private final long durationMs;
        
        public SimulationResult(String mode, double finalBalance, int totalOperations, 
                                int totalErrors, long durationMs) {
            this.mode = mode;
            this.finalBalance = finalBalance;
            this.totalOperations = totalOperations;
            this.totalErrors = totalErrors;
            this.durationMs = durationMs;
        }
        
        @Override
        public String toString() {
            return String.format(
                "%s:\n" +
                "  Итоговый баланс: %.2f\n" +
                "  Всего операций: %d\n" +
                "  Ошибок: %d\n" +
                "  Время: %d мс",
                mode, finalBalance, totalOperations, totalErrors, durationMs
            );
        }
    }
}
```

### 5.7. Основной класс

```java
import model.BankAccount;
import executor.AccountSimulator;

public class Main {
    private static final int THREAD_COUNT = 10;
    private static final int OPERATIONS_PER_THREAD = 1000;
    private static final double INITIAL_BALANCE = 10000.0;
    
    public static void main(String[] args) {
        System.out.println("=== Многопоточность: банковский счёт ===\n");
        
        // 1. Без синхронизации
        BankAccount accountNoSync = new BankAccount("ACC001", INITIAL_BALANCE);
        AccountSimulator simulatorNoSync = new AccountSimulator(
            accountNoSync, THREAD_COUNT, OPERATIONS_PER_THREAD, "Без синхронизации"
        );
        AccountSimulator.SimulationResult resultNoSync = simulatorNoSync.run();
        System.out.println(resultNoSync);
        
        // 2. С synchronized
        BankAccount accountSync = new BankAccount("ACC002", INITIAL_BALANCE);
        AccountSimulator simulatorSync = new AccountSimulator(
            accountSync, THREAD_COUNT, OPERATIONS_PER_THREAD, "С synchronized"
        );
        AccountSimulator.SimulationResult resultSync = simulatorSync.run();
        System.out.println(resultSync);
        
        // 3. С ReentrantLock
        BankAccount accountLock = new BankAccount("ACC003", INITIAL_BALANCE);
        AccountSimulator simulatorLock = new AccountSimulator(
            accountLock, THREAD_COUNT, OPERATIONS_PER_THREAD, "С ReentrantLock"
        );
        AccountSimulator.SimulationResult resultLock = simulatorLock.run();
        System.out.println(resultLock);
        
        // Сравнение
        System.out.println("\n=== Сравнение ===");
        System.out.printf("Ожидаемый баланс: ~%.2f%n", INITIAL_BALANCE);
        System.out.printf("Без синхронизации: %.2f (отклонение: %.2f)%n", 
            resultNoSync.getFinalBalance(), 
            Math.abs(resultNoSync.getFinalBalance() - INITIAL_BALANCE));
        System.out.printf("С synchronized: %.2f (отклонение: %.2f)%n",
            resultSync.getFinalBalance(),
            Math.abs(resultSync.getFinalBalance() - INITIAL_BALANCE));
        System.out.printf("С ReentrantLock: %.2f (отклонение: %.2f)%n",
            resultLock.getFinalBalance(),
            Math.abs(resultLock.getFinalBalance() - INITIAL_BALANCE));
        System.out.printf("Время (synchronized): %d мс%n", resultSync.getDurationMs());
        System.out.printf("Время (ReentrantLock): %d мс%n", resultLock.getDurationMs());
    }
}
```

---

## 6. Контрольные вопросы

1. Что такое процесс и что такое поток? В чём их различие?

2. Какие способы создания потоков существуют в Java?

3. Почему нельзя просто вызвать `run()` для запуска потока?

4. Какие состояния может принимать поток?

5. Что такое гонка данных (race condition)? Приведите пример.

6. Как `synchronized` решает проблему гонки данных?

7. В чём разница между `synchronized` методом и `synchronized` блоком?

8. Что делает ключевое слово `volatile`? Когда его использовать?

9. Для чего используются методы `wait()`, `notify()`, `notifyAll()`?

10. Чем `ReentrantLock` отличается от `synchronized`?

11. Почему в `ReentrantLock` нужно вызывать `unlock()` в `finally`?

12. Что такое взаимная блокировка (deadlock)? Как её избежать?

13. Как измерить производительность многопоточного кода?

14. Как отлаживать многопоточные приложения?

15. Что такое паттерн Producer-Consumer?

---

## 7. Пример выполнения (Вариант 1)

### 7.1. Ожидаемый вывод

```
=== Многопоточность: банковский счёт ===

=== Без синхронизации ===
Thread-1 начал работу
Thread-0 начал работу
Thread-1 завершил работу. Операций: 1000, успешно: 984, ошибок: 16, время: 15 мс
Thread-0 завершил работу. Операций: 1000, успешно: 982, ошибок: 18, время: 16 мс
...
Без синхронизации:
  Итоговый баланс: 15423.50
  Всего операций: 9850
  Ошибок: 150
  Время: 125 мс

=== С synchronized ===
pool-1-thread-1 начал работу
pool-1-thread-0 начал работу
...
С synchronized:
  Итоговый баланс: 15499.50
  Всего операций: 10000
  Ошибок: 0
  Время: 289 мс

=== С ReentrantLock ===
pool-2-thread-1 начал работу
pool-2-thread-0 начал работу
...
С ReentrantLock:
  Итоговый баланс: 15499.50
  Всего операций: 10000
  Ошибок: 0
  Время: 312 мс

=== Сравнение ===
Ожидаемый баланс: ~15500.00
Без синхронизации: 15423.50 (отклонение: 76.50)
С synchronized: 15499.50 (отклонение: 0.50)
С ReentrantLock: 15499.50 (отклонение: 0.50)
Время (synchronized): 289 мс
Время (ReentrantLock): 312 мс
```

### 7.2. График сравнения производительности

```
Производительность (меньше — лучше):
┌────────────────────────────────────────────────────────┐
│ Без синхронизации    ████████████████ 125 мс          │
│ С synchronized       ████████████████████████████ 289 мс │
│ С ReentrantLock      █████████████████████████████ 312 мс │
└────────────────────────────────────────────────────────┘

Точность (отклонение меньше — лучше):
┌────────────────────────────────────────────────────────┐
│ Без синхронизации    ████████████████████████ 76.50   │
│ С synchronized       ██ 0.50                          │
│ С ReentrantLock      ██ 0.50                          │
└────────────────────────────────────────────────────────┘
```

---

## 8. Рекомендуемые источники

1. **Шилдт Г.** *Java. Базовый курс.* — М.: Вильямс. — Глава 11 (Многопоточное программирование).

2. **Хорстманн К., Корнелл Г.** *Java. Библиотека профессионала. Том 1.* — М.: Вильямс. — Глава 12 (Параллелизм).

3. **Гоetz Б. и др.** *Java Concurrency in Practice.* — М.: Питер. — Полное руководство по параллелизму.

4. **Oracle Java Tutorials: Concurrency.** — URL: https://docs.oracle.com/javase/tutorial/essential/concurrency/

5. **Baeldung: Java Concurrency.** — URL: https://www.baeldung.com/java-concurrency

6. **Блох Дж.** *Java. Эффективное программирование.* — М.: Питер. — Правила 78-86 (Параллелизм).
