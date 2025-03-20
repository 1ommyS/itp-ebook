
## 1. Принципы SOLID

### 1.1 Single Responsibility Principle (Принцип Единственной Ответственности)
Каждый класс или модуль в программе должен иметь только одну причину для изменения. То есть, класс выполняет строго одну задачу. Если в классе появляется логика, не связанная напрямую с основной задачей, её следует вынести в другой класс.

**Пример (нарушение принципа)**:
```java
public class UserManager {
    public void createUser(String name) {
        // Логика создания пользователя
    }

    public void validateUserData(String name) {
        // Логика проверки корректности имени
    }

    public void printUserReport(String name) {
        // Логика вывода данных о пользователе в консоль/файл
    }
}
```
Здесь `UserManager` делает сразу три разные вещи: создаёт пользователя, валидирует данные и выводит отчёт.

**Пример (исправление)**:
```java
public class UserManager {
    public void createUser(String name) {
        // Логика создания пользователя
    }
}

public class UserValidator {
    public boolean isValidName(String name) {
        // Логика проверки имени
        return name != null && !name.isEmpty();
    }
}

public class UserReportPrinter {
    public void printUserReport(String name) {
        // Логика вывода данных о пользователе
    }
}
```
Теперь каждый класс отвечает за свою отдельную задачу.

---

### 1.2 Open/Closed Principle (Принцип Открытости/Закрытости)
Классы, модули и функции должны быть открыты для расширения, но закрыты для изменения. Другими словами, мы стремимся добавлять новую функциональность через наследование или композицию, не меняя уже протестированный код.

**Пример**:
Представим, что у нас есть класс `PaymentProcessor` для обработки платежей. Если в нём изначально жёстко зашит лишь один тип оплаты (скажем, кредитной картой), расширять такой класс будет сложно. Лучше воспользоваться полиморфизмом.

```java
public abstract class Payment {
    public abstract void pay(double amount);
}

public class CreditCardPayment extends Payment {
    @Override
    public void pay(double amount) {
        System.out.println("Оплата кредитной картой: " + amount);
    }
}

public class PayPalPayment extends Payment {
    @Override
    public void pay(double amount) {
        System.out.println("Оплата через PayPal: " + amount);
    }
}

public class PaymentProcessor {
    public void processPayment(Payment payment, double amount) {
        payment.pay(amount);
    }
}
```
Таким образом, чтобы добавить новый способ оплаты (например, `CryptoPayment`), мы не меняем `PaymentProcessor`, а лишь создаём новый класс, который расширяет `Payment`.

---

### 1.3 Liskov Substitution Principle (Принцип Подстановки Лисков)
Подклассы должны быть заменяемы базовыми классами, не нарушая функциональность. Если у нас есть класс-родитель и его наследник, то объект наследника должен корректно работать там, где ожидается объект родителя.

**Пример (нарушение принципа)**:
```java
public class Rectangle {
    protected int width;
    protected int height;

    public void setWidth(int width) {
        this.width = width;
    }
    public void setHeight(int height) {
        this.height = height;
    }

    public int getArea() {
        return width * height;
    }
}

public class Square extends Rectangle {
    // Переопределение сеттеров так, что при изменении ширины, меняется и высота.
    @Override
    public void setWidth(int width) {
        this.width = width;
        this.height = width;
    }
    @Override
    public void setHeight(int height) {
        this.width = height;
        this.height = height;
    }
}
```
Если мы используем `Square` вместо `Rectangle`, то логика вычисления площади может “ломаться” в тех местах, где ожидается независимая установка ширины и высоты.

**Исправление**: часто в таких случаях лучше разделять абстракции (возможно, сделать отдельные интерфейсы или использовать композицию) и не пытаться подсовывать `Square` как подтип `Rectangle`.

---

### 1.4 Interface Segregation Principle (Принцип Разделения Интерфейсов)
Клиенты не должны зависеть от интерфейсов, которые они не используют. Лучше делать несколько маленьких интерфейсов с чётким набором функциональности, чем один "большой" интерфейс.

**Пример (нарушение)**:
```java
public interface MultifunctionalDevice {
    void print();
    void scan();
    void fax();
}
```
Если какому-то устройству не нужен факс, оно всё равно будет вынуждено реализовывать метод `fax`.

**Пример (исправление)**:
```java
public interface Printer {
    void print();
}

public interface Scanner {
    void scan();
}

public interface Fax {
    void fax();
}
```
Теперь можно реализовывать интерфейсы только с нужной функциональностью.

---

### 1.5 Dependency Inversion Principle (Принцип Инверсии Зависимостей)
Высокоуровневые модули не должны зависеть от низкоуровневых модулей. Оба типа модулей должны зависеть от абстракций.  
Проще говоря, вместо того, чтобы внутри класса создавать конкретные объекты, следует зависеть от интерфейсов или абстрактных классов и передавать реализацию извне (через DI-контейнер, фабрики и т.п.).

**Пример**:
```java
public interface NotificationSender {
    void send(String message);
}

public class EmailSender implements NotificationSender {
    @Override
    public void send(String message) {
        System.out.println("Отправка email: " + message);
    }
}

public class SMSSender implements NotificationSender {
    @Override
    public void send(String message) {
        System.out.println("Отправка SMS: " + message);
    }
}

public class UserNotifier {
    private final NotificationSender notificationSender;

    public UserNotifier(NotificationSender notificationSender) {
        this.notificationSender = notificationSender;
    }

    public void notifyUser(String message) {
        notificationSender.send(message);
    }
}
```
Класс `UserNotifier` не зависит от конкретного способа отправки уведомлений. Вместо этого он получает `NotificationSender` в конструкторе, что упрощает тестирование и замену реализаций.

---

## 2. DRY (Don’t Repeat Yourself)
Принцип гласит: “Не повторяйся”. Любая логика должна быть определена только в одном месте. Если в коде встречаются повторы, следует подумать, как их вынести в общую функцию, базовый класс или метод.

**Пример (нарушение)**:
```java
public class OrderService {
    public double calculateTotalPrice(List<Product> products) {
        double total = 0;
        for (Product p : products) {
            total += p.getPrice();
        }
        return total;
    }
}

public class CartService {
    public double calculateTotalPrice(List<Product> products) {
        double total = 0;
        for (Product p : products) {
            total += p.getPrice();
        }
        return total;
    }
}
```
Одинаковая логика подсчёта цены дублируется в двух классах.

**Пример (исправление)**:
```java
public class PriceCalculator {
    public static double calculateTotalPrice(List<Product> products) {
        double total = 0;
        for (Product p : products) {
            total += p.getPrice();
        }
        return total;
    }
}

public class OrderService {
    public double calculateTotalPrice(List<Product> products) {
        return PriceCalculator.calculateTotalPrice(products);
    }
}

public class CartService {
    public double calculateTotalPrice(List<Product> products) {
        return PriceCalculator.calculateTotalPrice(products);
    }
}
```
Теперь логика сосредоточена в одном месте — `PriceCalculator`.

---

## 3. KISS (Keep It Simple, Stupid)
Идея: “Делай проще, не усложняй”. Если задачу можно решить простой структурой данных или простым алгоритмом, лучше так и сделать. Старайтесь избегать чрезмерной гибкости и абстракций, если они не нужны.

Пример: иногда достаточно использовать список (List) для хранения объектов вместо создания сложной структуры с наследованием, если у вас элементарная задача упорядоченного хранения данных.

---

## 4. YAGNI (You Ain’t Gonna Need It)
“Вам это не понадобится”. Не надо писать код и создавать функциональность “на будущее”, если нет конкретной задачи, требующей этого. Лишний, неиспользуемый код приводит к усложнению поддержки и тестирования.

---

## 5. Фабричный метод (Factory Method)
**Идея**: фабричный метод позволяет делегировать логику создания объектов наследникам суперкласса. Вместо того, чтобы порождать объекты напрямую через `new`, мы вызываем специализированный метод, который может быть переопределён в подклассе.

**Ситуация**: у нас есть базовый класс, который содержит логику работы с определёнными продуктами, но нам нужно дать возможность наследникам определять, какой именно продукт создавать.

**Структура**:

1. Абстрактный класс или интерфейс, описывающий продукт.
2. Конкретные реализации этого продукта.
3. Абстрактный класс (или интерфейс) "создатель" (creator), содержащий фабричный метод `createProduct()`.
4. Конкретные классы-"создатели", переопределяющие фабричный метод, чтобы возвращать нужный тип продукта.

**Пример**:

```java
// 1. Абстрактный продукт
public interface Button {
    void render();
}

// 2. Конкретные продукты
public class WindowsButton implements Button {
    @Override
    public void render() {
        System.out.println("Рисуем кнопку в стиле Windows");
    }
}

public class HTMLButton implements Button {
    @Override
    public void render() {
        System.out.println("Рисуем кнопку в стиле HTML");
    }
}

// 3. Абстрактный создатель
public abstract class Dialog {
    public void renderWindow() {
        // ... некоторая логика ...
        Button okButton = createButton();
        okButton.render();
        // ... ещё логика ...
    }

    // Фабричный метод
    protected abstract Button createButton();
}

// 4. Конкретные создатели
public class WindowsDialog extends Dialog {
    @Override
    protected Button createButton() {
        return new WindowsButton();
    }
}

public class WebDialog extends Dialog {
    @Override
    protected Button createButton() {
        return new HTMLButton();
    }
}
```
Таким образом, клиентский код работает с `Dialog`, а какой конкретно тип кнопки будет создан — решают подклассы `WindowsDialog` или `WebDialog`.

---

## 6. Простой объект “Фабрика” и “Абстрактная фабрика”

### 6.1 Простая Фабрика (Simple Factory)
Не является официальным паттерном из GoF, но очень распространена. Цель — сосредоточить логику создания разных объектов в одном месте (обычно в отдельном классе с фабричными методами).

**Пример**:
```java
public class ShapeFactory {
    public static Shape createShape(String shapeType) {
        switch (shapeType) {
            case "CIRCLE":
                return new Circle();
            case "SQUARE":
                return new Square();
            default:
                throw new IllegalArgumentException("Неизвестный тип фигуры: " + shapeType);
        }
    }
}
```
При этом в коде мы можем вызывать:
```java
Shape circle = ShapeFactory.createShape("CIRCLE");
```
и получить объект нужного типа без прямого вызова `new Circle()`.

### 6.2 Абстрактная фабрика (Abstract Factory)
**Идея**: Позволяет создавать целые семейства связанных объектов без указания их конкретных классов. Часто используется для обеспечения совместимости продуктов внутри "семейства".

**Ситуация**: например, при разработке графического интерфейса под разные ОС нам нужно создавать связанные элементы (кнопки, чекбоксы, окна) в одном стиле.

**Пример**:
```java
// 1. Абстрактные продукты
public interface Button {
    void paint();
}

public interface Checkbox {
    void paint();
}

// 2. Конкретные продукты
public class WinButton implements Button {
    @Override
    public void paint() {
        System.out.println("Вы нарисовали WinButton.");
    }
}

public class WinCheckbox implements Checkbox {
    @Override
    public void paint() {
        System.out.println("Вы нарисовали WinCheckbox.");
    }
}

public class MacButton implements Button {
    @Override
    public void paint() {
        System.out.println("Вы нарисовали MacButton.");
    }
}

public class MacCheckbox implements Checkbox {
    @Override
    public void paint() {
        System.out.println("Вы нарисовали MacCheckbox.");
    }
}

// 3. Абстрактная фабрика
public interface GUIFactory {
    Button createButton();
    Checkbox createCheckbox();
}

// 4. Конкретные фабрики
public class WinFactory implements GUIFactory {
    @Override
    public Button createButton() {
        return new WinButton();
    }

    @Override
    public Checkbox createCheckbox() {
        return new WinCheckbox();
    }
}

public class MacFactory implements GUIFactory {
    @Override
    public Button createButton() {
        return new MacButton();
    }

    @Override
    public Checkbox createCheckbox() {
        return new MacCheckbox();
    }
}

// 5. Клиентский код
public class Application {
    private Button button;
    private Checkbox checkbox;

    public Application(GUIFactory factory) {
        button = factory.createButton();
        checkbox = factory.createCheckbox();
    }

    public void paint() {
        button.paint();
        checkbox.paint();
    }
}

// Пример использования
public class Demo {
    public static void main(String[] args) {
        GUIFactory factory;
        String osName = System.getProperty("os.name").toLowerCase();
        if (osName.contains("win")) {
            factory = new WinFactory();
        } else {
            factory = new MacFactory();
        }

        Application app = new Application(factory);
        app.paint();
    }
}
```
Таким образом, мы можем легко переключаться между разными “семействами” элементов (Windows-стиль, Mac-стиль и т.д.).

---

## 7. Singleton (Синглтон)
**Идея**: гарантировать, что у класса есть только один экземпляр, и предоставить глобальную точку доступа к этому экземпляру. Чаще всего используется для глобальных настроек, кэшей, логгеров и т.д.

**Пример**:
```java
public class Singleton {
    private static Singleton instance;

    private Singleton() {
        // Приватный конструктор запрещает создание новых экземпляров извне
    }

    public static synchronized Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }

    public void doSomething() {
        System.out.println("Делаем что-то важное в синглтоне...");
    }
}
```
Здесь используется ленивый (lazy) подход к созданию объекта и синхронизация, чтобы избежать проблем в многопоточном окружении. Однако синхронизация в `getInstance()` может снижать производительность. Более эффективный вариант — “Double-Checked Locking” или статическая инициализация.

---

## 8. Adapter (Адаптер)
**Идея**: паттерн позволяет объектам с несовместимыми интерфейсами работать вместе. Адаптер “оборачивает” один объект и преобразует его интерфейс в нужный для другого объекта/кода.

**Ситуация**: у нас есть класс `RoundHole`, который работает с круглыми “колышками” (`RoundPeg`), но поступили квадратные “колышки” (`SquarePeg`), и их нужно как-то использовать без изменения существующего кода для `RoundHole`.

**Структура**:

1. Имеется целевой интерфейс/класс, с которым работает клиент (`RoundHole` + `RoundPeg`).
2. Есть несовместимый класс (`SquarePeg`).
3. Адаптер (`SquarePegAdapter`), который реализует интерфейс “круглого колышка” и переводит его вызовы в методы “квадратного колышка”.

**Пример**:
```java
// Целевой класс, работающий с круглыми колышками
public class RoundHole {
    private double radius;

    public RoundHole(double radius) {
        this.radius = radius;
    }

    public double getRadius() {
        return radius;
    }

    public boolean fits(RoundPeg peg) {
        return this.getRadius() >= peg.getRadius();
    }
}

// Целевой интерфейс
public class RoundPeg {
    private double radius;

    public RoundPeg() {}

    public RoundPeg(double radius) {
        this.radius = radius;
    }

    public double getRadius() {
        return radius;
    }
}

// Несовместимый класс
public class SquarePeg {
    private double width;

    public SquarePeg(double width) {
        this.width = width;
    }

    public double getWidth() {
        return width;
    }
}

// Адаптер
public class SquarePegAdapter extends RoundPeg {
    private SquarePeg squarePeg;

    public SquarePegAdapter(SquarePeg squarePeg) {
        this.squarePeg = squarePeg;
    }

    @Override
    public double getRadius() {
        // Рассчитываем "радиус" исходя из ширины квадрата
        // Диагональ квадрата = width * sqrt(2)
        // Радиус вписанного круга = диагональ / 2 = (width * sqrt(2)) / 2 = width * sqrt(2)/2
        return (squarePeg.getWidth() * Math.sqrt(2)) / 2;
    }
}

// Пример использования
public class DemoAdapter {
    public static void main(String[] args) {
        RoundHole hole = new RoundHole(5);
        RoundPeg rpeg = new RoundPeg(5);
        System.out.println("Round peg fits? " + hole.fits(rpeg)); // true

        SquarePeg smallSqPeg = new SquarePeg(5);
        SquarePeg largeSqPeg = new SquarePeg(10);

        // hole.fits(smallSqPeg); // Ошибка, несовместимые типы

        // Теперь используем адаптер
        SquarePegAdapter smallSqPegAdapter = new SquarePegAdapter(smallSqPeg);
        SquarePegAdapter largeSqPegAdapter = new SquarePegAdapter(largeSqPeg);
        System.out.println("Square peg with width 5 fits? " + hole.fits(smallSqPegAdapter)); // true
        System.out.println("Square peg with width 10 fits? " + hole.fits(largeSqPegAdapter)); // false
    }
}
```
Таким образом, `SquarePegAdapter` позволяет объекту `SquarePeg` работать там, где ожидается `RoundPeg`.

---

## Заключение

### Почему важны принципы и паттерны?

1. **SOLID** принципы помогают писать код, который легче сопровождать и расширять.
2. **DRY** уменьшает дублирование, что снижает риск багов и упрощает рефакторинг.
3. **KISS** помогает избежать чрезмерной сложности, а значит, уменьшает риски запутанности проекта.
4. **YAGNI** напоминает не тратить ресурсы на “предполагаемые” будущие задачи.
5. **Паттерны** (Фабричный метод, Фабрика, Абстрактная фабрика, Синглтон, Адаптер и др.) помогают решать типовые архитектурные задачи, делая систему более гибкой и понятной. Они дают общий язык для разработчиков, позволяя мгновенно распознавать знакомые решения.
