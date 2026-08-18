# Лабораторная работа №3: Наследование и полиморфизм

## Варианты 

1. Банковское дело (BankAccount)
Иерархия: BankAccount  CreditAccount, SavingsAccount.
Абстракция: Метод calculateInterest().
Полиморфизм: Массив BankAccount[] с вызовом расчета процентов.
Интерфейс: TransactionLoggable (метод getHistory()).
2. Автомобильный транспорт (Car)
Иерархия: Car  Truck, SportCar.
Абстракция: Метод fuelConsumptionPer100km().
Полиморфизм: Массив Car[] для вывода расхода топлива разных типов авто.
Интерфейс: OffroadCapable (метод engage4WD()).
3. Электронная коммерция (Product)
Иерархия: Product  DigitalGood, PhysicalGood.
Абстракция: Метод calculateShippingCost(). Для цифровых товаров — 0.
Полиморфизм: Список корзины, подсчет итоговой стоимости доставки.
Интерфейс: Discountable (метод applyPromoCode()).
4. Социальные сети (UserProfile)
Иерархия: UserProfile  VerifiedUser, BusinessUser.
Абстракция: Метод getMaxUploadSize().
Полиморфизм: Массив профилей для отображения лимитов загрузки.
Интерфейс: Advertisable (метод showAdsPanel()).
5. Геолокация (GeoPoint)
Иерархия: GeoPoint  CityLocation, MountainPeak.
Абстракция: Метод getAltitude().
Полиморфизм: Расчет расстояния между массивом точек относительно базовой.
Интерфейс: TimeZoneAware (метод getLocalTime()).
6. Университет (Student)
Иерархия: Student  Undergraduate, Graduate.
Абстракция: Метод getScholarshipAmount().
Полиморфизм: Ведомость выплат стипендий из массива студентов.
Интерфейс: DormitoryResident (метод getRoomNumber()).
7. Видеоигры (VideoGame)
Иерархия: VideoGame  IndieGame, AAA_Title.
Абстракция: Метод getSystemRequirements().
Полиморфизм: Библиотека игр, проверка запуска на ПК.
Интерфейс: Multiplayer (метод connectToServer()).
8. Медицина (PatientCard)
Иерархия: PatientCard  Inpatient, Outpatient.
Абстракция: Метод calculateTreatmentCost().
Полиморфизм: Реестр пациентов стационара и амбулатории.
Интерфейс: Insured (метод getPolicyNumber())
9. Библиотечное дело (LibraryItem)
Иерархия: LibraryItem  BookCopy, MagazineIssue.
Абстракция: Метод getReturnDueDate().
Полиморфизм: Выдача читателю любого типа издания.
Интерфейс: Reservable (метод placeHold()).
10. Космонавтика (Spacecraft)
Иерархия: Spacecraft  OrbitalStation, Lander.
Абстракция: Метод currentFuelLevel().
Полиморфизм: Стартовый стол со списком кораблей.
Интерфейс: DeepSpaceProbe (метод transmitSignal()).
11. Умный дом (SmartDevice)
Иерархия: SmartDevice  Thermostat, SecurityCamera.
Абстракция: Метод updateFirmware().
Полиморфизм: Управление всеми устройствами дома через единый список.
Интерфейс: BatteryPowered (метод checkBatteryStatus()).
12. Ресторанный бизнес (MenuItem)
Иерархия: MenuItem  Beverage, MainCourse.
Абстракция: Метод isGlutenFree().
Полиморфизм: Печать меню с разделением по категориям.
Интерфейс: Spicy (метод getScovilleIndex()).
13. Музыка (MusicTrack)
Иерархия: MusicTrack  LiveRecording, StudioMix.
Абстракция: Метод getFileSizeMB().
Полиморфизм: Плейлист с расчетом общего веса файлов.
Интерфейс: ExplicitContent (метод hasParentalWarning()).
14. Управление проектами (Task)
Иерархия: Task  BugFix, FeatureDevelopment.
Абстракция: Метод estimateHours().
Полиморфизм: Спринт-борд с задачами разного типа.
Интерфейс: Blocker (метод blockProgress()).
15. Зоопарк (Animal)
Иерархия: Animal  Mammal, Reptile.
Абстракция: Метод makeSound().
Полиморфизм: «Пробуждение зоопарка» — вызов звуков у всех животных в массиве.
Интерфейс: Feedable (метод feed(Food food)).
16. Фитнес (Exercise)
Иерархия: Exercise  Cardio, StrengthTraining.
Абстракция: Метод caloriesBurnedPerHour().
Полиморфизм: Тренировочный план с подсчетом сожженных калорий.
Интерфейс: EquipmentRequired (метод getGearList()).
17. Кибербезопасность (NetworkCredential)
Иерархия: NetworkCredential  SSHKey, PasswordToken.
Абстракция: Метод isExpired().
Полиморфизм: Проверка валидности списка доступов сервера.
Интерфейс: Revocable (метод revokeAccess()).
18. Сельское хозяйство (CropField)
Иерархия: CropField  WheatField, CornField.
Абстракция: Метод yieldPerHectare().
Полиморфизм: Учет урожая со всего фермерского хозяйства.
Интерфейс: Irrigated (метод waterVolumeNeeded()).
19. Event-менеджмент (ConcertTicket)
Иерархия: ConcertTicket  VIPTicket, StandardTicket.
Абстракция: Метод getEntryGate().
Полиморфизм: Очередь на вход: определение ворот для каждого билета.
Интерфейс: Refundable (метод requestRefund()).
20. Экология (RecyclableWaste)
Иерархия: RecyclableWaste  PlasticBottle, GlassJar.
Абстракция: Метод getProcessingFee().
Полиморфизм: Сортировочная линия: расчет стоимости переработки кучи мусора.
Интерфейс: BioDegradable (метод decompositionDays()).