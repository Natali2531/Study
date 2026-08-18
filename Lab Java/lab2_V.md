# Лабораторная работа №2: Конструкторы, перегрузка, static

## Варианты

1. Банковское дело (BankAccount)
Конструкторы: По умолчанию (баланс 0), с номером счета, со всеми данными.
Статика: Статическая переменная nextAccountNumber для автогенерации номеров счетов в фабричном методе.
Фабрика: createSavingsAccount(String owner) — создает счет с типом «Сберегательный» и уникальным номером.
Перегрузка: getStatement(boolean includeTransactions). Если false — только итоговый баланс, если true — полная выписка.
2. Автомобильный транспорт (Car)
Конструкторы: Дефолтный («Безымянный», год 2000), полный, конструктор копирования (Car anotherCar).
Статика: Поле totalCarsCreated.
Фабрика: fromJson(String jsonString) — парсит строку вида "brand":"Lada","model":"Vesta" и возвращает объект.
Перегрузка: drive(double kilometers) и drive(double distance, String unit), где unit может быть "km" или "miles".
3. Электронная коммерция (Product)
Конструкторы: Без ID, с ID, со скидкой по умолчанию.
Статика: Счетчик activeProducts (увеличивается в конструкторе, уменьшается при вызове метода archive()).
Фабрика: createBundle(Product p1, Product p2, double discount) — собирает два товара в один бандл.
Перегрузка: getPrice(), возвращающий double, и getPrice(Locale locale), форматирующий цену под валюту региона.
4. Социальные сети (UserProfile)
Конструкторы: Приватный основной, публичный без параметров (заглушка «Гость»).
Статика: Set<String> existingLogins для проверки уникальности на этапе создания.
Фабрика: registerViaOAuth(String provider, String externalId).
Перегрузка: updateEmail(String newEmail) и updateEmail(String newEmail, String confirmationCode).
5. Геолокация (GeoPoint)
Конструкторы: Градусы/минуты/секунды, десятичные градусы, пустой (0,0).
Статика: Константа EARTH_RADIUS_KM.
Фабрика: randomPointInRadius(GeoPoint center, double radiusKm).
Перегрузка: distanceTo(GeoPoint other) (в км) и distanceTo(GeoPoint other, DistanceUnit unit).
6. Университет (Student)
Конструкторы: Только ФИО, ФИО + номер зачетки, все данные.
Статика: Map<Integer, Student> registry для поиска студента по номеру зачетки.
Фабрика: expelledStudent(String name, int id) — помечает статус как «Отчислен».
Перегрузка: addGrade(int grade) и addGrades(List<Integer> grades).
7. Видеоигры (VideoGame)
Конструкторы: Название/жанр, полное издание (Collector's Edition), клон существующей игры.
Статика: Список List<String> bannedGenres.
Фабрика: createIndieGame(String title, String developer).
Перегрузка: isCompatible(OS os) и isCompatible(OS os, boolean checkDrivers).
8. Медицина (PatientCard)
Конструкторы: Анонимный пациент, с паспортными данными.
Статика: Пороговые значения температуры и давления (CRITICAL_TEMP).
Фабрика: emergencyCase(String symptoms) — заполняет карту экстренного пациента.
Перегрузка: takeMedication(String drugName) и takeMedication(String drugName, double dosageMg).
9. Библиотечное дело (LibraryItem)
Конструкторы: Базовый класс; подклассы вызывают super(...).
Статика: Общее количество экземпляров во всей библиотеке.
Фабрика: createFromBarcode(String barcodeData).
Перегрузка: renewLoan(User user) и renewLoan(User user, int additionalDays).
10. Космонавтика (Spacecraft)
Конструкторы: Прототип, готовая к запуску ракета.
Статика: Очередь запуска Queue<Spacecraft> launchPad.
Фабрика: cargoMission(String destination, double payloadKg).
Перегрузка: statusReport() (текстом) и statusReport(OutputStream stream) (запись в лог-файл).
11. Умный дом (SmartDevice)
Конструкторы: Только MAC-адрес, адрес + тип устройства.
Статика: Эмуляция задержки сети PROCESSING_DELAY_MS.
Фабрика: discoverOnNetwork(String ipRange).
Перегрузка: setPower(boolean onOff) и setPower(PowerLevel level) (где уровень — enum LOW/MEDIUM/HIGH).
12. Ресторанный бизнес (MenuItem)
Конструкторы: Простое блюдо, комбо-обед.
Статика: Налоговая ставка SALES_TAX_RATE.
Фабрика: dailySpecial(LocalDate date).
Перегрузка: calculateCost() и calculateCost(int guestCount).
13. Музыка (MusicTrack)
Конструкторы: Инструментальный трек (без слов), трек с вокалом.
Статика: Максимальная длительность трека в сервисе (например, 10 часов).
Фабрика: generateSilence(long durationMs).
Перегрузка: play() и play(double speedFactor) (ускоренное воспроизведение).
14. Управление проектами (Task)
Конструкторы: Задача без срока, задача с высоким приоритетом.
Статистика: Счетчик просроченных задач overdueCounter.
Фабрика: subtaskOf(Task parent).
Перегрузка: markComplete() и markComplete(String completionNotes).
15. Зоопарк (Animal)
Конструкторы: Новорожденное животное (возраст 0), взрослое.
Статика: Общее число калорий еды в зоопарке на сегодня.
Фабрика: rescuedAnimal(String species, LocalDate rescueDate).
Перегрузка: feed(Food food) и feed(List<Food> dailyRation).
16. Фитнес (Exercise)
Конструкторы: Упражнение с собственным весом, со штангой.
Статика: Словарь коэффициентов сложности упражнений.
Фабрика: warmupRoutine().
Перегрузка: logSet(int reps) и logSet(int reps, double weightKg).
17. Кибербезопасность (NetworkCredential)
Конструкторы: Создание нового пользователя, восстановление доступа.
Статика: История хэшей паролей List<String> passwordHistory для запрета старых паролей.
Фабрика: serviceAccount(String botName).
Перегрузка: rotatePassword() (автогенерация) и rotatePassword(String newPlainText).
18. Сельское хозяйство (CropField)
Конструкторы: Прямоугольное поле (длина, ширина), квадратное (сторона).
Статика: Средняя урожайность по региону.
Фабрика: irrigatedZone(double area).
Перегрузка: harvest() (убирает всё) и harvest(double hectares).
19. Event-менеджмент (ConcertTicket)
Конструкторы: Обычное место, VIP-ложа.
Статика: AtomicInteger nextSeatId (потокобезопасная нумерация).
Фабрика: buyBackReturnedTicket(int originalId).
Перегрузка интерфейса: compareTo(Ticket o) сортирует сначала по сектору, затем по ряду, затем по месту.
20. Экология (RecyclableWaste)
Конструкторы: Неизвестный тип мусора, отсортированный мусор.
Статика: Тарифная сетка баллов Map<MaterialType, Integer> ecoRates.
Фабрика: fromHouseholdWeight(double totalWeight) (автоклассификация).
Перегрузка: processBatch(double kg) и processBatch(double kg, boolean isCleanSort).
