# Лабораторная работа №6: Исключения и логирование

## Варианты

1. Банковское дело (BankAccount)
Exception: InsufficientFundsException при попытке списания суммы больше баланса.
Logging: INFO — успешный перевод; ERROR — недостаточно средств.
2. Автомобильный транспорт (Car)
Exception: DuplicateVINException при добавлении авто с существующим VIN в реестр.
Logging: WARN — попытка регистрации дубля; INFO — регистрация пройдена.
3. Электронная коммерция (Product)
Exception: OutOfStockException при заказе товара, которого нет на складе.
Logging: INFO — заказ оформлен; WARN — товар закончился во время оформления.
4. Социальные сети (UserProfile)
Exception: UsernameTakenException при занятом логине.
Logging: ERROR — неудачная попытка подбора/регистрации; INFO — вход выполнен.
5. Геолокация (GeoPoint)
Exception: InvalidCoordinatesException при выходе за пределы широты/долготы.
Logging: WARN — некорректные GPS-данные от устройства.
6. Университет (Student)
Exception: GradeFormatException при передаче неверного формата оценки (например, > 100).
Logging: ERROR — ошибка импорта ведомости из файла.
7. Видеоигры (VideoGame)
Exception: UnsupportedPlatformException при запуске игры на несовместимой ОС.
Logging: INFO — игра запущена; DEBUG — проверка системных требований.
8. Медицина (PatientCard)
Exception: MedicalDataAccessException при ошибке доступа к защищенным данным пациента.
Logging: ERROR — несанкционированный доступ к базе СНИЛС.
9. Библиотечное дело (LibraryItem)
Exception: LoanLimitExceededException при превышении лимита книг у читателя.
Logging: INFO — книга выдана; WARN — читатель имеет задолженность.
10. Космонавтика (Spacecraft)
Exception: FuelDepletionException при недостатке топлива для маневра.
Logging: FATAL — критическая потеря связи или топлива.
11. Умный дом (SmartDevice)
Exception: ConnectionTimeoutException при потере связи с устройством.
Logging: WARN — устройство оффлайн; INFO — переподключение успешно.
12. Ресторанный бизнес (MenuItem)
Exception: AllergenNotDeclaredException если в блюде есть аллерген без маркировки.
Logging: ERROR — жалоба клиента на состав блюда.
13. Музыка (MusicTrack)

Exception: LicenseExpiredException при попытке воспроизведения трека по истекшей подписке.
Logging: INFO — трек проигрывается; WARN — регион заблокирован.
14. Управление проектами (Task)
Exception: CircularDependencyException при создании циклической зависимости задач.
Logging: ERROR — нарушение структуры спринта.
15. Зоопарк (Animal)
Exception: DietMismatchException при попытке кормления хищника травой.
Logging: WARN — животное отказывается от еды.
16. Фитнес (Exercise)
Exception: EquipmentInUseException при попытке занять занятый тренажер.
Logging: INFO — начало подхода; DEBUG — расчет пульса.
17. Кибербезопасность (NetworkCredential)
Exception: BruteForceDetectedException при множественных неудачных попытках входа.
Logging: SECURITY — блокировка IP-адреса.
18. Сельское хозяйство (CropField)
Exception: SoilContaminationException при обнаружении вредителей выше нормы.
Logging: WARN — требуется обработка пестицидами.
19. Event-менеджмент (ConcertTicket)
Exception: SeatAlreadyOccupiedException при двойном бронировании места.
Logging: INFO — скан билета прошел; WARN — попытка повторного прохода.
20. Экология (RecyclableWaste)
Exception: HazardousMaterialException при попадании запрещенного мусора в бак.
Logging: ERROR — остановка конвейера сортировочной ленты.