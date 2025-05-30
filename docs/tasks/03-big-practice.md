# Большое задание (расчитано на 2 недели)
Какой объем вы должны отсюда выполнить? Лично я настоятельно рекомендую минимум 30%, что эквивалетно 10 задачам.
При этом учтите, вы обязательно должны сделать по задаче из первых трех блоков и попытаться сделать хотя бы одну из блока `хардкор`.

## Оглавление

1) [Задачи на повторение основ](#основы)
2) [База, которую нужно знать и понимать](#База)
3) [Усложнение базы](#Усложнение)
4) [ХАРДКОР!!!!!](#хардкор)

Итак, посмотрим на сами задачи)

### 1) [Задачи на повторение основ](#основы)
В данном блоке предполагается отработка самых базовых навыков: цикл `for`, цикл `while`, оператор `if/else`, оператор `switch`, написание простых методов. Всё это будет встречаться в более сложных задачах позже.

1) **Работа с циклами и условиями.**
   - Программа запрашивает у пользователя целое число `n`.
   - Если `n < 0`, выведите на экран: «Число отрицательное».
   - Если `n > 0`, выведите на экран: «Число положительное».
   - Если `n = 0`, выведите на экран: «Число равно нулю».
   - После этого выведите все числа от `-n` до `n` включительно с помощью цикла `for`.
   - Реализуйте также вариант вывода этих же чисел с помощью цикла `while`.

2) **Простые числа в диапазоне.**
   - Программа запрашивает у пользователя два числа: `start` и `end`. Гарантируется, что `start <= end`.
   - Необходимо вывести все простые числа из диапазона `[start; end]`.
   - Используйте простую проверку деления на все числа от 2 до \(\sqrt{N}\).
   - Результат выведите в одну строку, числа разделяйте пробелами.

3) **Использование `switch` для меню.**
   - Создайте простое консольное «меню»:
      1) Вывести «Hello»
      2) Вывести «World»
      3) Вывести «Hello World»
      4) Выход
   - Пользователь вводит пункт меню (число от 1 до 4).
   - Используя оператор `switch`, выполните соответствующие действия.
   - При выборе пункта 4 программа завершается.

4) **Сумма цифр числа**
   - Запросить у пользователя целое число `n`.
   - С помощью цикла `while` найти сумму всех цифр этого числа (учитывая знак числа).
   - Если число отрицательное, сделайте его положительным и посчитайте сумму цифр.
   - Выведите результат.

5) **Подсчёт гласных и согласных**
   - Приложение запрашивает у пользователя строку (только латиница, пусть это будет упрощение).
   - С помощью цикла `for` подсчитать количество гласных (a, e, i, o, u, в верхнем и нижнем регистре) и согласных букв.
   - Вывести результат.

6) **Проверка на палиндром** 
   - Запросить у пользователя строку.
   - Удалить из неё все пробелы и знаки препинания (оставить только буквы/цифры).
   - Привести к одному регистру.
   - Проверить, является ли полученная строка палиндромом (читается одинаково слева направо и справа налево).
   - Вывести результат.
7) Вам дан массив `целых чисел`. Размер массива задается `рандомом`. Вам необходимо заполнить его `рандомными числами`.
   После этого в массиве необходимо найти и вывести на экран
   - все четные элементы
   - все нечетные элементы
   - максимальный элеметн
   - минимальный элемент
   - медиану массива (элемент, который либо равен среднему арифметическому всех чисел в массиве, либо ближе всего к
     нему по значению, то есть разница между ср.арифм и этим элементом - минимальная)
   - количество четных чисел
   - количество чисел кратных 2 и 3 одновременно.
   - количество чисел, которые является квадратом другого числа (то есть их квадратный корень - целое число)
8) Вам необходимо реализовать приложение, которое будет получать на вход `строку` и:
   - Если длина строки больше 15 символов, будет выводить первые 15 символов, а вместо остальных будет выводить ...
   - Будет всю выходную строку приводить к UPPERCASE.
   - Будет заменять в строке все символы `.` на `,`.


## 2) [База, которую нужно знать и понимать](#База)
В этом разделе важно освоить написание классов, методов, понимание конструктора, геттеров и сеттеров, перегрузку методов, рекурсию и взаимодействие простых классов (как один класс может содержать объекты другого). Также здесь важен метод `toString()`.

1) **Простейший класс `Person`.**
   - Создайте класс `Person` c полями: `name` (String) и `age` (int).
   - Реализуйте конструктор, геттеры и сеттеры.
   - Переопределите метод `toString()`, чтобы при выводе объекта `Person` на экран печаталась строка вида: `Person{name='...', age=...}`.
   - Напишите метод `increaseAge()`, который увеличивает возраст на 1.
   - В отдельном классе `Main` создайте несколько объектов типа `Person`, выведите их на экран и протестируйте все методы.

2) **Перегрузка методов (Overloading).**
   - Создайте класс `Calculator`.
   - В нём сделайте несколько методов `add(...)` с одинаковым названием, но разной сигнатурой:
      - `add(int a, int b)` – возвращает сумму двух целых чисел.
      - `add(double a, double b)` – возвращает сумму двух чисел с плавающей точкой.
      - `add(String a, String b)` – соединяет две строки через пробел и возвращает их.
   - В методе `main` проверьте работу всех вариантов `add(...)`.

3) **Рекурсивные методы.**
   - Напишите рекурсивный метод `factorial(int n)`, который вычисляет факториал числа `n`. Если `n < 0`, метод может выбрасывать исключение или возвращать -1 (на ваш выбор). Если вы получаете отрицательное число, все окей, вы не влезаете по размеру числа в инт(лонг)
   - Дополнительно реализуйте рекурсивный метод `fibonacci(int n)`, который возвращает n-е число Фибоначчи. *ВНИМАНИЕ* НОВЫЙ КЛАСС СОЗДАВАТЬ `НЕ НУЖНО`
   - Проверьте работу обоих методов в `main`.

4) **Класс `Book` и взаимодействие классов.**
   - Создайте класс `Book` с полями: `title` (String), `author` (String), `year` (int).
   - Создайте класс `Library`, в котором есть массив (или список) объектов `Book`.
      - В `Library` добавьте метод `addBook(Book book)`, который добавляет книгу в коллекцию.
      - Добавьте метод `printAllBooks()`, который выводит список всех книг.
   - В `main` создайте несколько книг, заполните их данными, добавьте в библиотеку, выведите результат.

5) **Практика с методами и возвратом значений.**
   - Создайте класс `MathUtils` c методами:
      - `isPrime(int n)`: проверяет, простое ли число.
      - `gcd(int a, int b)`: находит наибольший общий делитель (НОД).
      - `lcm(int a, int b)`: находит наименьшее общее кратное (НОК).
   - В `main` проверьте эти методы на разных наборах входных данных.

6) **Перегрузка конструкторов**
   - Создайте класс `Car` с полями `brand`, `model`, `year`.
   - Реализуйте два конструктора:
      - Без параметров (по умолчанию `brand = "Unknown"`, `model = "Unknown"`, `year = 0`).
      - С тремя параметрами (brand, model, year).
   - Создайте несколько объектов `Car` используя оба конструктора и выведите их (`toString()`).

7) **Статические методы и поля**
   - Добавьте в класс `Car` статическое поле `totalCars` (количество созданных объектов). Увеличивайте его в каждом конструкторе.
   - Добавьте статический метод `getTotalCars()`, возвращающий значение `totalCars`.
   - В `main` создайте несколько `Car`, а затем выведите `Car.getTotalCars()`.

8) **Реализация собственных исключений**
   - Создайте свой класс исключения `AgeException`, который наследуется от `Exception`.
   - В классе `Person` сделайте сеттер для возраста, который выбрасывает это исключение, если возраст < 0 или > 150.
   - В `main` обработайте это исключение.

9) **Методы сравнения (`equals` и `hashCode`)**
   - В класс `Book` добавьте методы `equals(Object o)` и `hashCode()`, чтобы книги считались равными, если у них совпадают `title` и `author`.
   - Проверьте работу в `main`, создав несколько книг с одинаковыми полями и сравнив их.

10) **Класс, содержащий другие классы (составной объект)**
- Создайте класс `Address` с полями `city`, `street`, `houseNumber`.
- В классе `Person` добавьте поле `Address address`.
- Реализуйте соответствующие геттеры/сеттеры, а также логику вывода в `toString()`.
- В `main` создайте объекты `Address` и присвойте их разным `Person`.

---

## 3) [Усложнение базы](#Усложнение)
Здесь подключаются темы наследования, полиморфизма, а также базовой работы с файлами (чтение и запись). За счёт этого вы научитесь проектировать более гибкие структуры, а также сохранять результаты своей работы в файл.

### Усложнение базы

1) **Наследование и полиморфизм (Фигуры).**
   - Создайте абстрактный класс `Shape` с абстрактным методом `double getArea()` (площадь).
   - Создайте классы-наследники: `Circle` (круг) и `Rectangle` (прямоугольник). У круга поле `radius`, у прямоугольника поля `width` и `height`.
   - Переопределите в каждом из них метод `getArea()`.
   - В `main` создайте массив `Shape[] shapes` и добавьте туда несколько кругов и прямоугольников.
   - Пробегитесь по массиву и выведите площади всех фигур (демонстрация полиморфизма).

2) **Чтение и запись в текстовый файл.**
   - Создайте класс `FileIOExample` с методами:
      - `writeToFile(String filename, String text)`, который записывает строку в указанный файл (создает новый файл или перезаписывает существующий).
      - `readFromFile(String filename)`, который считывает содержимое файла и возвращает его как строку.
   - В `main` создайте объект `FileIOExample` и:
      - Запишите в файл любую строку.
      - Считайте эту строку из файла и выведите на экран.

3) **Расширение взаимодействия классов (Наследование для библиотеки).**
   - Вернитесь к классу `Book` из предыдущего раздела. Создайте наследника `EBook` с дополнительным полем `fileSize` (размер файла в мегабайтах) и `format` (например, `PDF`, `EPUB` и т.п.).
   - Переопределите метод `toString()`, чтобы он учитывал новые поля.
   - В классе `Library` сделайте метод `printOnlyEBooks()`, который выводит информацию только об электронных книгах.
   - В `main` добавьте несколько `Book` и несколько `EBook` в библиотеку, потом вызовите метод `printOnlyEBooks()` и проверьте вывод.

4) **Интерфейсы**
   - Создайте интерфейс `Printable` с методом `printInfo()`.
   - Реализуйте этот интерфейс в классах `Book` и `EBook`, чтобы метод `printInfo()` выводил информацию о книге.
   - В `Library` сделайте метод `printAllPrintable()`, который вызывает `printInfo()` для всех `Printable` объектов.

5) **Чтение и запись объектов построчно**
   - Модифицируйте `FileIOExample`, добавив метод `writeBookList(ArrayList<Book> books, String filename)`, который записывает каждую книгу на новой строке в формате: `title;author;year`.
   - Добавьте метод `readBookList(String filename)`, который читает файл построчно, парсит данные и создаёт объекты `Book`. Возвращает `ArrayList<Book>`.
   - Протестируйте запись и чтение списка книг.

6) **Абстракция и несколько уровней наследования**
   - Создайте абстрактный класс `Employee` с полями `name`, `salary` и абстрактным методом `work()`.
   - Создайте несколько наследников: `Manager`, `Developer`, `Designer` и переопределите `work()`.
   - Добавьте общий метод `increaseSalary(double percent)`.
   - В `main` создайте список сотрудников разных типов, пройдитесь по списку и вызовите `work()`, а также увеличьте зарплату всем на определённый процент.

7) **Практика с исключениями и файлами**
   - В методе чтения файла из задания выше предусмотрите выброс исключений (например, `IOException`).
   - Обработайте их в `main`.
   - Попробуйте создать ситуацию, когда файл не существует, и посмотреть, как всё сработает.
8) **В нашем проекте ЭДО**
   - Добавьте метод для проверки наличия пользователя в базе данных
9) **Реализуйте метод восстановления пароля**
   - Добавьте в меню авторизации логику восстановления пароля пользователя по ключевому слову
   - Вы должны ожидать на вход от пользователя логин, а он должен ввести ключевое слово, которое он должен просто знать
   - В коде ключевое слово должно быть константой
   ```java
   private final String keyWord = "Значение ключевого слова";
   ```
10) **Метод для вычитывания данных с консоли**
   - Сделайте в классе utils метод, который позволит удобно читать значения из консоли. 
```java
   String login = Utils.readValue();
```
---

## 4) [ХАРДКОР!!!!!](#хардкор)
Самые сложные темы: более продвинутая работа с файлами (чтение/запись разных форматов), полиморфизм на более серьёзном уровне, реализация собственного `ArrayList`, дженерики и т.д. Эти задачи призваны максимально прокачать навыки.

### Хардкор

1) **Полиморфизм и иерархия классов (Животные).**
   - Создайте абстрактный класс `Animal` c методами:
      - `void makeSound()`,
      - `void move()`.
   - Создайте несколько классов-наследников: `Cat`, `Dog`, `Bird` и т.д.
      - Переопределите методы `makeSound()` (звук), `move()` (способ перемещения).
   - Создайте класс `Zoo` с коллекцией животных.
      - Добавьте в `Zoo` методы, позволяющие добавлять животное (`addAnimal`), удалять (`removeAnimal`) и выводить список всех животных (`printAllAnimals`).
      - Продемонстрируйте работу в `main`.
   - (Дополнительно) Реализуйте у некоторых животных поля, уникальные для них (например, у `Bird` – размах крыльев).

2) **Реализация собственного списка (упрощённый `ArrayList`).**
   - Создайте класс `MyArrayList<T>`. В нём должно быть:
      - Внутренний массив `Object[] elements`.
      - Поле `size` – текущий размер списка (количество реальных элементов).
   - Реализуйте методы:
      - `add(T value)`: добавляет элемент в конец списка, при переполнении массива увеличивать массив вдвое.
      - `get(int index)`: возвращает элемент по индексу (учитывая проверки на выход за границы).
      - `remove(int index)`: удаляет элемент по индексу со сдвигом элементов.
      - `size()`: возвращает текущее количество элементов.
   - В `main` протестируйте `MyArrayList`: добавьте несколько элементов, выведите их, удалите пару элементов и посмотрите, корректно ли сдвигаются оставшиеся.

3) **Файлы и дженерики: чтение/запись списка объектов.**
   - Создайте класс `User` с полями `id` (int), `name` (String), `email` (String). Переопределите `toString()`.
   - Создайте класс `UserManager<T extends User>` (используем дженерики), который будет хранить список пользователей типа `T`.
      - Метод `addUser(T user)` – добавляет пользователя в список.
      - Метод `saveToFile(String filename)` – записывает всех пользователей в текстовый файл (каждый пользователь в новой строке).
      - Метод `loadFromFile(String filename)` – читает всех пользователей из файла и добавляет их в список (парсит строки и создает объекты `User`).
   - Напишите `main`, который создаёт несколько объектов `User`, добавляет их в `UserManager`, сохраняет в файл, потом очищает список в памяти, загружает из файла и выводит прочитанные данные на экран.

4) **Реализация собственного связанного списка (`MyLinkedList<T>`)**
   - Создайте класс `Node<T>` с полями `value`, `Node<T> next`.
   - Класс `MyLinkedList<T>` будет хранить `head` (первый элемент) и `size`.
   - Реализуйте методы:
      - `add(T value)`: добавляет элемент в конец списка (проходим до последнего элемента и ставим `next`).
      - `remove(int index)`: удаляет элемент по индексу.
      - `get(int index)`: возвращает элемент по индексу.
      - `size()`.
   - В `main` протестируйте на разных данных, включая крайние случаи (удаление первого элемента, последнего, и т.п.).

5) **В нашем проекте ЭДО**
   - Реализуйте FileMenu, UserMenu 
   - Добавьте возможность бесконечного ввода команд
   - Добавьте возможность редактировать информацию любого пользователя
   - Доделайте сериализацию и десериализацию для хранения файлов
   - Добавьте простейшее шифрование пароля (как угодно)
   - Донастройте работу с паролями, создав отдельный класс, который будет заниматься их валидацией
---

## Заключение
Все эти задачи призваны последовательно прокачивать навыки: от базовых структур (циклы, условия, массивы, методы) к объектно-ориентированному программированию (классы, наследование, полиморфизм), а затем к более серьёзным вещам (файлы, дженерики, собственные структуры данных).

Старайтесь делать задачи последовательно, вдумчиво, тщательно разбираясь в каждом шаге. Если какая-то задача кажется слишком лёгкой – обязательно попробуйте усложнить её: добавьте дополнительную проверку, интересную логику или красивый вывод. Если задача, наоборот, кажется слишком сложной – разбивайте её на маленькие части и тестируйте постепенно каждую часть.

Удачи вам!
