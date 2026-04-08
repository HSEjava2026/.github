# Практическое задание: Разработка системы управления персоналом (HRM Lite) — Часть 3

## 1. Общее описание

**Цель задания:** Закрепить навыки работы с коллекциями, дженериками, функциональными интерфейсами, лямбда-выражениями, Stream API, а также основами модульного тестирования и ролевой модели безопасности.

Вам необходимо расширить функционал существующего консольного приложения для управления данными сотрудников компании, заменив массивы на коллекции и добавив новые возможности.

---

## 2. Архитектура приложения

Проект должен содержаться в пакете `ru.hse` и быть разделен на логические пакеты:

1. `model` — описание сущностей (POJO классы).
2. `service` — бизнес-логика приложения.
3. `presentation` — взаимодействие с пользователем через консоль.
4. `exception` — пользовательские классы исключений.
5. `repository` — работа с хранилищем данных (коллекции).
6. `util` — вспомогательные утилиты (логирование).
7. `security` — ролевая модель и аутентификация.
8. `test` — модульные тесты.

Main class - `HRMApplication`

---

## 3. Подробные требования к реализации

### Шаг 1: Переход на коллекции

**ВНИМАНИЕ:** Запрещено использование массивов для хранения данных. Используйте только коллекции из Java Collection Framework.

В пакете `repository` создайте класс `CollectionRepository` со следующими требованиями:

* Хранение сотрудников должно осуществляться в `Map<Long, Employee>` (ключ — ID сотрудника).
* Хранение задач должно осуществляться в `Map<Long, Task>` (ключ — ID задачи).
* Дополнительно должны храниться:
  * Связи «программист → список задач» (`Map<Long, List<Long>>`)
  * Связи «менеджер → список программистов» (`Map<Long, List<Long>>`)

Методы репозитория:

* `void saveEmployee(Employee employee)` — добавление/обновление сотрудника.
* `Employee findEmployeeById(Long id)` — поиск сотрудника по ID.
* `void deleteEmployee(Long id)` — удаление сотрудника.
* `List<Employee> findAllEmployees()` — получение всех сотрудников.
* `List<Programmer> findAllProgrammers()` — получение всех программистов.
* `List<Manager> findAllManagers()` — получение всех менеджеров.
* Аналогичные методы для работы с задачами.
* `void assignTaskToProgrammer(Long programmerId, Long taskId)` — назначение задачи.
* `List<Task> getTasksByProgrammer(Long programmerId)` — получение задач программиста.
* `void assignProgrammerToManager(Long managerId, Long programmerId)` — назначение программиста менеджеру.
* `List<Programmer> getProgrammersByManager(Long managerId)` — получение программистов менеджера.

---

### Шаг 2: Дженерики

Создайте параметризованный интерфейс `Repository<T, K>` в пакете `repository`:

```java
public interface Repository<T, K> {
    void save(T entity);
    T findById(K id);
    void delete(K id);
    List<T> findAll();
}
```

Класс `CollectionRepository` должен реализовать этот интерфейс для `Employee` и `Task` (можно через реализацию нескольких интерфейсов или через наследование).

Создайте параметризованный класс `ServiceResult<T>` для возврата результата операций:

```java
public class ServiceResult<T> {
    private boolean success;
    private T data;
    private String errorMessage;
    // конструкторы, геттеры
}
```

Методы сервиса должны возвращать `ServiceResult<T>` вместо выбрасывания checked исключений (или наряду с ними).

---

### Шаг 3: Функциональные интерфейсы и лямбды

В пакете `service` создайте следующие функциональные интерфейсы:

* `EmployeeFilter` — с методом `boolean test(Employee employee)`
* `TaskFilter` — с методом `boolean test(Task task)`

В классе `HRMService` добавьте следующие методы с использованием лямбда-выражений:

* `List<Employee> filterEmployees(EmployeeFilter filter)` — фильтрация сотрудников по произвольному условию.
* `List<Task> filterTasks(TaskFilter filter)` — фильтрация задач по произвольному условию.
* `void processEmployees(List<Employee> employees, Consumer<Employee> action)` — выполнение действия над каждым сотрудником.
* `<R> List<R> mapEmployees(List<Employee> employees, Function<Employee, R> mapper)` — преобразование списка сотрудников.

Пример использования в консольном меню (пункт "Show filter menu") должен быть переработан с использованием этих методов.

---

### Шаг 4: Stream API

В классе `HRMService` переработайте следующие методы с использованием Stream API:

* `List<Employee> getEmployeesWithExperienceMoreThan(int years)` — сотрудники с опытом более N лет (используйте `filter`).
* `Map<Grade, List<Programmer>> getProgrammersGroupedByGrade()` — группировка программистов по грейду.
* `double getAverageSalaryByPosition(String position)` — средняя зарплата по должности.
* `List<Task> getOverdueTasks()` — задачи с просроченным дедлайном.
* `Optional<Employee> findEmployeeWithMaxSalary()` — сотрудник с максимальной зарплатой.
* `Map<String, Long> getEmployeeCountByPosition()` — количество сотрудников по каждой должности.

Все эти методы должны быть реализованы с использованием Stream API (без явных циклов).

---

### Шаг 5: Ролевая модель и безопасность

В пакете `security` создайте:

1. **Enum `Role`** со значениями: `ADMIN`, `HR_MANAGER`, `PROJECT_MANAGER`, `EMPLOYEE`

2. **Класс `User`**:
   * `String username`
   * `String password` (хранить в открытом виде — для простоты)
   * `Role role`
   * `Long employeeId` (ссылка на сотрудника, может быть null)

3. **Класс `AuthenticationService`**:
   * `User login(String username, String password)` — аутентификация пользователя (генерирует `InvalidDataException` при ошибке)
   * `User getCurrentUser()` — получение текущего аутентифицированного пользователя
   * `void logout()` — выход из системы
   * `boolean hasRole(Role role)` — проверка роли текущего пользователя
   * `boolean hasPermission(Permission permission)` — проверка разрешения

4. **Enum `Permission`**:
   * `VIEW_EMPLOYEES` — просмотр сотрудников
   * `EDIT_EMPLOYEES` — редактирование сотрудников
   * `DELETE_EMPLOYEES` — удаление сотрудников
   * `MANAGE_TASKS` — управление задачами
   * `VIEW_REPORTS` — просмотр отчетов
   * `MANAGE_USERS` — управление пользователями

5. **Маппинг ролей и разрешений** (в классе `SecurityConfig`):
   * `ADMIN` — все разрешения
   * `HR_MANAGER` — `VIEW_EMPLOYEES`, `EDIT_EMPLOYEES`, `VIEW_REPORTS`
   * `PROJECT_MANAGER` — `VIEW_EMPLOYEES`, `MANAGE_TASKS`, `VIEW_REPORTS`
   * `EMPLOYEE` — только просмотр своих данных

При запуске приложения создайте нескольких тестовых пользователей с разными ролями.

**Требование:** Каждый метод в `HRMService` перед выполнением операции должен проверять разрешения текущего пользователя через `AuthenticationService`. При отсутствии разрешения выбрасывается `SecurityException`.

---

### Шаг 6: Модульное тестирование (JUnit)

В пакете `test` создайте следующие тестовые классы:

1. **`CollectionRepositoryTest`**:
   * Тестирование CRUD операций для сотрудников
   * Тестирование назначения задач программисту
   * Тестирование назначения программиста менеджеру

2. **`HRMServiceTest`**:
   * Тестирование фильтрации сотрудников
   * Тестирование методов с Stream API
   * Тестирование исключительных ситуаций

3. **`AuthenticationServiceTest`**:
   * Тестирование успешной и неуспешной аутентификации
   * Тестирование проверки ролей и разрешений

**Требования к тестам:**
* Используйте JUnit 5 (Jupiter).
* Тесты должны быть независимыми (каждый тест создает свои данные).
* Используйте `@BeforeEach` для инициализации.
* Покройте позитивные и негативные сценарии.
* Используйте `assertThrows` для проверки исключений.

---

### Шаг 7: Модификация консольного интерфейса (`Reporter`)

Добавьте в меню новые пункты и измените существующие:

```text
=== HRM System Menu ===
=== Current user: admin (ADMIN) ===

1. Show all employees
2. Show filter menu (lambda/stream)
3. Add new employee
4. Remove employee by ID
5. Update employee salary
6. Assign task to programmer
7. Complete task
8. Save data to file
9. Load data from file
--- Advanced Reports ---
10. Show employees grouped by grade (Stream)
11. Show average salary by position (Stream)
12. Show employee with max salary (Stream)
13. Show overdue tasks (Stream)
--- Security ---
14. Login
15. Logout
16. Show current user info
17. Exit
```

**Требования:**
* При запуске приложения пользователь не аутентифицирован. Доступны только пункты 14 (Login) и 17 (Exit).
* После успешного входа отображаются пункты меню в соответствии с ролями пользователя.
* Пункты, на которые у пользователя нет прав, должны быть скрыты или недоступны.
* Все операции из части 2 должны быть адаптированы для работы с коллекциями вместо массивов.

---

## 4. Функциональные требования

1. При работе разрешено использовать только стандартные классы библиотеки Java + JUnit (версия 5).
2. Запрещено использование массивов для хранения данных — только коллекции.
3. Все методы, работающие со списками сотрудников/задач, должны использовать Stream API где это возможно.
4. Лямбда-выражения должны использоваться как минимум в 3 различных сценариях (фильтрация, преобразование, обработка).
5. Дженерики должны быть использованы в репозитории и классе-обертке для результата.
6. Каждый публичный метод сервиса должен проверять права доступа.
7. Тесты должны проходить успешно (покрытие не менее 70% критической логики).
8. Сериализация/десериализация из части 2 должна работать с коллекциями (адаптируйте `FileRepository`).
9. При любом некорректном действии пользователя программа должна продолжать работу.

## 5. Дедлайн: 22.04.2026 23:59
