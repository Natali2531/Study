# Лабораторная работа №5: Generics: универсальные структуры данных

## Варианты

1. Банковское дело (BankAccount)
Класс: Repository<BankAccount> с использованием Map<Long, BankAccount>.
Stream: Подсчет общей суммы процентов: .mapToDouble(BankAccount::calculateInterest).sum().
2. Автомобильный транспорт (Car)
Класс: Pair<String, Double> (Модель и расход топлива).
Set: HashSet<Car> для хранения уникальных VIN-номеров.
Stream: Фильтрация внедорожников: .filter(Car::isOffroadCapable).
3. Электронная коммерция (Product)
Класс: Обобщенный Box<T extends Product> для корзины.
Set: Удаление дубликатов товаров из List<Product> через new HashSet<>(list).
Stream: Расчет цены: .peek(p -> p.applyPromoCode()).map(Product::getPrice).sum().
4. Социальные сети (UserProfile)
Класс: ComparablePair<UserProfile> для сравнения пользователей по лимиту загрузки.
Set: Set<String> логинов для проверки уникальности при регистрации.
Stream: Профили с рекламой: .filter(UserProfile::isAdvertisable).
5. Геолокация (GeoPoint)
Класс: ArrayList<GeoPoint> как путь.
Set: Set<GeoPoint> для исключения повторных координат.
Stream: Поиск максимума: .max(Comparator.comparingDouble(GeoPoint::getAltitude)).
6. Университет (Student)
Класс: Универсальный статический метод <T extends Comparable<T>> T findMax(List<T> list).
Set: Set<Integer> номеров зачеток.
Stream: Студенты в общежитии: .filter(s -> s instanceof DormitoryResident).
7. Видеоигры (VideoGame)
Класс: Pair<VideoGame, SystemRequirements>.
Set: Проверка наличия игры перед покупкой через library.contains(game).
Stream: Игры с мультиплеером: .filter(g -> g instanceof Multiplayer).
8. Медицина (PatientCard)
Класс: Repository<PatientCard>.
Set: Set<Long> СНИЛС/ID пациентов.
Stream: Пациенты со страховкой: .filter(Insured.class::isInstance).
9. Библиотечное дело (LibraryItem)
Класс: ComparablePair<LocalDate> для поиска самой ранней даты возврата.
Set: Set<Reservable> забронированных книг.
Stream: Просроченные издания: .filter(i -> i.getReturnDueDate().isBefore(LocalDate.now())).
10. Космонавтика (Spacecraft)
Класс: Pair<Spacecraft, Double> (Корабль и топливо).
Set: Реестр ID запущенных аппаратов.
Stream: Вызов метода у зондов: .filter(d -> d instanceof DeepSpaceProbe).forEach(DeepSpaceProbe::transmitSignal).
11. Умный дом (SmartDevice)
Класс: Pair<String, SmartDevice> (Комната и устройство).
Set: Set<String> IP-адресов для обнаружения конфликтов.
Stream: Устройства на батарейках: .filter(BatteryPowered.class::isInstance).
12. Ресторанный бизнес (MenuItem)
Класс: Box<MenuItem> заказа.
Set: Set<String> ингредиентов блюда.
Stream: Острые блюда: .filter(Spicy::isGlutenFree) (опечатка в примере выше, должно быть Spicy или проверка остроты).
13. Музыка (MusicTrack)
Класс: Repository<MusicTrack>.
Set: Плейлист без дублей (LinkedHashSet для сохранения порядка).
Stream: Скрытие треков: .filter(t -> !t.hasParentalWarning()).
14. Управление проектами (Task)
Класс: Pair<Task, Employee> (Задача и исполнитель).
Set: Список заблокированных задач (Blocker).
Stream: Оценка времени спринта: .mapToInt(Task::estimateHours).sum().
15. Зоопарк (Animal)
Класс: Class<? extends Animal> как параметр фабрики.
Set: Учет чипированных животных.
Stream: Кормление млекопитающих: .filter(Mammal.class::isInstance).forEach(a -> a.feed(food)).
16. Фитнес (Exercise)
Класс: Pair<Exercise, Integer> (Упражнение и подходы).
Set: Доступные упражнения дня.
Stream: Суммарный вес: .mapToDouble(e -> e.caloriesBurnedPerHour() * reps).sum().
17. Кибербезопасность (NetworkCredential)
Класс: Repository<NetworkCredential>.
Set: Отозванные токены.
Stream: Очистка: .filter(NetworkCredential::isExpired).collect(Collectors.toList()).
18. Сельское хозяйство (CropField)
Класс: Pair<CropField, Double> (Поле и урожайность).
Set: Карта засеянных участков.
Stream: Потребность в воде: .mapToDouble(Irrigated::waterVolumeNeeded).sum().
19. Event-менеджмент (ConcertTicket)
Класс: ComparablePair<Double> (Цена билета).
Set: Возвращенные билеты (Refundable).
Stream: VIP-проход: .filter(VIPTicket.class::isInstance).
20. Экология (RecyclableWaste)
Класс: Box<RecyclableWaste> мусоровоза.
Set: Типы материалов в баке.
Stream: Баллы: .mapToInt(RecyclableWaste::getEcoPoints).sum().