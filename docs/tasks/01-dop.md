### **Задание 1: Класс для работы с банковским счётом**

**Описание:**
1. Создайте класс `BankAccount` с приватными полями:
   - `accountNumber` (номер счёта),
   - `owner` (владелец счёта),
   - `balance` (баланс).
2. Реализуйте:
   - Геттеры для всех полей и сеттер только для владельца (`owner`).
   - Конструкторы:
     - Без параметров (создаёт пустой счёт с балансом 0).
     - С двумя параметрами: `accountNumber` и `owner`.
     - С тремя параметрами: `accountNumber`, `owner`, `balance`.
   - Метод `deposit(double amount)` для пополнения счёта.
   - Метод `withdraw(double amount, boolean allowOverdraft)`, который списывает деньги с учётом возможности ухода в минус.

**Пример работы:**
```java
BankAccount account1 = new BankAccount("12345678", "John Doe", 100.0);
account1.deposit(50);
account1.withdraw(30, false);
account1.withdraw(150, true);
System.out.println(account1.getBalance()); // Вывод: -30.0
```

---

### **Задание 2: Класс для обработки заказов в магазине**

**Описание:**
1. Создайте класс `Order` с приватными полями:
   - `orderId` (идентификатор заказа),
   - `customerName` (имя покупателя),
   - `totalAmount` (общая сумма заказа).
2. Реализуйте:
   - Геттеры для всех полей, сеттеры только для имени покупателя (`customerName`).
   - Конструкторы:
     - Без параметров (создаёт заказ с пустыми данными и суммой 0).
     - С двумя параметрами: `orderId` и `customerName`.
     - С тремя параметрами: `orderId`, `customerName`, `totalAmount`.
   - Методы:
     - `applyDiscount(double percentage)` — применяет скидку в процентах к общей сумме.
     - Перегруженный метод `applyDiscount(double amount, boolean isFixed)` — применяет фиксированную скидку, если `isFixed` равен `true`.

**Пример работы:**
```java
Order order = new Order("ORD001", "Alice", 200.0);
order.applyDiscount(10); // Скидка 10%
order.applyDiscount(15, true); // Скидка 15 у.е.
System.out.println(order.getTotalAmount()); // Вывод: 165.0
```

---

### **Задание 3: Класс для обработки данных о сотрудниках**

**Описание:**
1. Создайте класс `Employee` с приватными полями:
   - `id` (идентификатор сотрудника),
   - `name` (имя сотрудника),
   - `salary` (зарплата).
2. Реализуйте:
   - Геттеры для всех полей и сеттер только для зарплаты (`salary`).
   - Конструкторы:
     - Без параметров (создаёт сотрудника с пустыми данными и зарплатой 0).
     - С двумя параметрами: `id` и `name`.
     - С тремя параметрами: `id`, `name`, `salary`.
   - Методы:
     - `raiseSalary(double percentage)` — увеличивает зарплату на указанный процент.
     - Перегруженный метод `raiseSalary(double amount, boolean isFixed)` — увеличивает зарплату на фиксированную сумму, если `isFixed` равен `true`.

**Пример работы:**
```java
Employee emp = new Employee("E001", "Bob", 3000.0);
emp.raiseSalary(10); // Увеличение на 10%
emp.raiseSalary(500, true); // Увеличение на 500 у.е.
System.out.println(emp.getSalary()); // Вывод: 3800.0
```
