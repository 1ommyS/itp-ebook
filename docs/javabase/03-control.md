# Управляющие операторы в Java

## Управляющие операторы позволяют изменять поток выполнения программы в зависимости от условий, повторять определенные действия или осуществлять переходы между различными частями кода. В Java управляющие операторы можно разделить на три основные категории:
1.	`Операторы выбора` <br>
Позволяют программе выбирать различные пути выполнения на основе результата выражения или состояния переменной.
2.	`Операторы итерации` <br>
Позволяют потоку выполнения повторять один или несколько операторов, формируя циклы.
3.	`Операторы перехода` <br>
Позволяют программе выполнять нелинейные переходы, такие как прерывание цикла или выход из метода.

## Операторы выбора

### if и switch

`Оператор if`

Используется для выполнения блока кода, если условие истинно. Может иметь дополнительные блоки else if и else.

```java
int number = 10;

if (number > 0) {
    System.out.println("Число положительное.");
} else if (number < 0) {
    System.out.println("Число отрицательное.");
} else {
    System.out.println("Число равно нулю.");
}
```

`Оператор switch`

Используется для выбора одного из множества вариантов выполнения на основе значения выражения.
```java
char grade = 'B';

switch (grade) {
    case 'A':
        System.out.println("Отлично!");
        break;
    case 'B':
        System.out.println("Хорошо!");
        break;
    case 'C':
        System.out.println("Удовлетворительно.");
        break;
    default:
        System.out.println("Неверная оценка.");
        break;
}

или 

switch (grade) {
    case 'A' ->{
        System.out.println("Отлично!");
    }
    case 'B': {
        System.out.println("Хорошо!");
    }
    case 'C' ->{
        System.out.println("Удовлетворительно.");
    }
    default ->
        System.out.println("Неверная оценка.")
}
```

## Примеры использования if и switch

### Пример if: Проверка возраста для доступа к ресурсу

```java
int age = 20;

if (age >= 18) {
    System.out.println("Доступ разрешен.");
} else {
    System.out.println("Доступ запрещен.");
}

```

### Пример switch: Определение дня недели

```java
int day = 3;
String dayName;

switch (day) {
    case 1:
        dayName = "Понедельник";
        break;
    case 2:
        dayName = "Вторник";
        break;
    case 3:
        dayName = "Среда";
        break;
    case 4:
        dayName = "Четверг";
        break;
    case 5:
        dayName = "Пятница";
        break;
    case 6:
        dayName = "Суббота";
        break;
    case 7:
        dayName = "Воскресенье";
        break;
    default:
        dayName = "Неверный день";
        break;
}

System.out.println("День недели: " + dayName);

```

# Операторы итерации

Циклы позволяют многократно выполнять один и тот же набор инструкций, пока не будет удовлетворено условие завершения. Условием может быть любое булевское выражение.

## Цикл while

**Выполняет тело цикла, пока условие истинно. Условие проверяется перед каждой итерацией.**

```java
int i = 0;

while (i < 5) {
    System.out.println("i = " + i);
i++;
}
```

`Пример: Нахождение суммы чисел от 1 до 100`
```java
int sum = 0;
int number = 1;

while (number <= 100) {
    sum += number;
    number++;
}

System.out.println("Сумма чисел от 1 до 100: " + sum);
```

## Цикл do-while

**Выполняет тело цикла хотя бы один раз, поскольку условие проверяется после выполнения тела.**

```java
int j = 0;

do {
    System.out.println("j = " + j);
    j++;
} while (j < 5);
```

`Пример: Запрос ввода пользователя до тех пор, пока не будет введено правильное значение`

```java
import java.util.Scanner;

public class InputExample {
    public static void main(String[] args) {
    Scanner scanner = new Scanner(System.in);
    int input;
    
    do {
        System.out.print("Введите число больше 10: ");
        input = scanner.nextInt();
    } while (input <= 10);
    
    System.out.println("Спасибо! Вы ввели: " + input);
    }
}
```

## Цикл for

**Используется, когда количество итераций известно заранее. Состоит из трех частей: инициализация, условие и итерация.**

```java
for (int k = 0; k < 5; k++) {
    System.out.println("k = " + k);
}
```

### Структура цикла for:
1.	Инициализация: Выполняется один раз при первом запуске цикла.
2.	Условие: Проверяется перед каждой итерацией. Если истинно, выполняется тело цикла.
3.	Итерация: Выполняется после тела цикла, обычно изменяет управляющую переменную.

Пример: Вывод четных чисел от 0 до 10
```java
for (int n = 0; n <= 10; n += 2) {
    System.out.println("Четное число: " + n);
}
```

### Цикл for с несколькими управляющими переменными

**Позволяет использовать несколько переменных для управления циклом, разделяя их запятыми.**
```java
for (int a = 1, b = 4; a < b; a++, b--) {
    System.out.println("a = " + a + ", b = " + b);
}
```

## В цикле for может отсутствовать часть <инициализация>, <условие> или <итерация>

Пример: Бесконечный цикл с условием выхода внутри тела
```java
int count = 0;

for (;;) { // Бесконечный цикл
    if (count >= 5) {
    break;
}
System.out.println("count = " + count);
count++;
}

```

## Цикл for-each (enhanced for)

Предназначен для прохода по коллекции объектов строго последовательно от начала до конца.

```java
int[] numbers = {1, 2, 3, 4, 5};

for (int num : numbers) {
    System.out.println("Число: " + num);
}
```

### Проход по многомерным массивам

Пример: Вывод элементов двумерного массива
```java
int[][] matrix = {
{1, 2, 3},
{4, 5, 6},
{7, 8, 9}
};

for (int[] row : matrix) {
    for (int element : row) {
        System.out.print(element + " ");
    }
    System.out.println();
}
```

# Операторы перехода

Операторы перехода позволяют изменять поток выполнения программы, прерывая циклы, пропуская итерации или возвращая управление из методов. Основные операторы перехода: break, continue, return, а также механизмы обработки исключений (try, catch, throw, finally).

## Использование оператора break

1. Завершение цикла
```java
for (int i = 0; i < 10; i++) {
    if (i == 5) {
    break; // Выход из цикла при i == 5
}
System.out.println("i = " + i);
}

```

2. Завершение оператора switch
```java
int day = 2;
String dayType;

switch (day) {
    case 1:
    case 2:
    case 3:
    case 4:
    case 5:
        dayType = "Будний день";
        break;
    case 6:
    case 7:
        dayType = "Выходной";
        break;
    default:
        dayType = "Неверный день";
        break;
}

System.out.println(dayType);
```

## Использование оператора continue

Пропуск текущей итерации и переход к следующей
```java
for (int i = 0; i < 5; i++) {
    if (i == 2) {
    continue; // Пропустить вывод при i == 2
    }
System.out.println("i = " + i);
}
```

Пример: Пропуск нечетных чисел
```java
for (int i = 1; i <= 10; i++) {
    if (i % 2 != 0) {
    continue; // Пропуск нечетных чисел
}
System.out.println("Четное число: " + i);
}
```

## Использование оператора return

Возврат из метода с значением
```java
public int add(int a, int b) {
return a + b; // Возврат суммы
}
```

Возврат из метода без значения
```java
public void printMessage(String message) {
    if (message == null) {
    return; // Выход из метода, если сообщение null
}
System.out.println(message);
}
```
# Обработка исключений (try, catch, throw, finally)

Пример: Обработка деления на ноль
```java
public void divide(int a, int b) {
    try {
        int result = a / b;
        System.out.println("Результат: " + result);
    } catch (ArithmeticException e) {
        System.out.println("Ошибка: Деление на ноль.");
    } finally {
        System.out.println("Блок finally выполняется всегда.");
    }
}
```
Использование throw для генерации исключения
```java
public void checkAge(int age) {
    if (age < 18) {
        throw new IllegalArgumentException("Возраст должен быть не менее 18 лет.");
    }
    System.out.println("Возраст принят.");
}
```
Рекомендации и лучшие практики

1. Избегайте бесконечных циклов:
Убедитесь, что циклы имеют корректные условия завершения, чтобы избежать зависания программы.
2. Используйте for-each там, где это возможно:
Он упрощает код и снижает вероятность ошибок при работе с коллекциями.
3. Правильная обработка исключений:
Обрабатывайте только те исключения, которые можете корректно обработать, и избегайте пустых блоков catch.
4. Минимизируйте использование break и continue:
Чрезмерное использование этих операторов может усложнить понимание кода. Старайтесь использовать их осмысленно и только там, где это действительно необходимо.

# Заключение

Управляющие операторы являются фундаментальной частью программирования, позволяя создавать гибкие и эффективные алгоритмы. Понимание их работы и правильное применение способствует написанию чистого, понятного и поддерживаемого кода.

## Дополнительные Примеры

Чтобы еще лучше понять использование управляющих операторов, приведем несколько дополнительных примеров.

Пример: Поиск первого положительного числа в массиве
```java
public class FindFirstPositive {
    public static void main(String[] args) {
    int[] numbers = {-5, -3, -1, 0, 2, 4, 6};
    int firstPositive = 0;

        for (int num : numbers) {
            if (num > 0) {
                firstPositive = num;
                break; // Прерываем цикл после нахождения первого положительного числа
            }
        }
        if (firstPositive > 0) {
            System.out.println("Первое положительное число: " + firstPositive);
        } else {
            System.out.println("Положительных чисел нет.");
        }
    }
}
```
Пример: Пропуск определенных элементов при переборе списка
```java

import java.util.Arrays;
import java.util.List;

public class SkipElements {
public static void main(String[] args) {
List<String> words = Arrays.asList("apple", "banana", "cherry", "date", "elderberry");

        for (String word : words) {
            if (word.startsWith("b") || word.startsWith("d")) {
                continue; // Пропускаем слова, начинающиеся на 'b' или 'd'
            }
            System.out.println("Слово: " + word);
        }
    }
}
```

Пример: Использование return для выхода из метода при обнаружении ошибки
```java
public class ValidateInput {
    public static void main(String[] args) {
        String input = "Hello, World!";
        processInput(input);
    }

    public static void processInput(String input) {
        if (input == null || input.isEmpty()) {
            System.out.println("Ошибка: Входные данные пусты.");
            return; // Выход из метода, если данные некорректны
        }
        System.out.println("Обработка данных: " + input);
    }
}
```

Пример: Использование исключений для управления потоком выполнения
```java
public class ExceptionHandlingExample {
    public static void main(String[] args) {
        try {
            performDivision(10, 0);
        } catch (ArithmeticException e) {
            System.out.println("Перехвачено исключение: " + e.getMessage());
        }
    }

    public static void performDivision(int a, int b) {
        if (b == 0) {
            throw new ArithmeticException("Деление на ноль невозможно.");
        }
        int result = a / b;
        System.out.println("Результат: " + result);
    }
}
```

Краткое Резюме

- Операторы выбора (if, switch) позволяют принимать решения в коде.
- Операторы итерации (while, do-while, for, for-each) позволяют повторять блоки кода.
- Операторы перехода (break, continue, return) управляют потоком выполнения, позволяя прерывать циклы или выходить из методов.
- Обработка исключений (try, catch, throw, finally) обеспечивает управление ошибками и исключительными ситуациями.
- Лучшие практики включают избегание ненужных меток, правильную обработку исключений и минимизацию использования операторов break и continue для улучшения читаемости кода.

# Задачи
[Учебник](https://school272.ucoz.ru/TeachersWorks/Kononov/sbornik_zadach_po_programmiroaniyu.pdf)
- 5.1
- 5.13
- 5.27 (а,г)
- 5.28 (а,г)
- 5.8
- 5.72
- 6.21
- 7.1
- 7.26
- 7.56
- 8.1
- 8.3

## Усложнение

[Доп задача №1](https://acmp.ru/index.asp?main=task&id_task=233)  <br>
[Доп задача №2](https://acmp.ru/index.asp?main=task&id_task=81) <br>
[Доп задача №3](https://acmp.ru/index.asp?main=task&id_task=331) <br>
[Доп задача №4](https://acmp.ru/index.asp?main=task&id_task=131) <br>
[Доп задача №5](https://acmp.ru/index.asp?main=task&id_task=284) <br>
[Доп задача №6](https://acmp.ru/index.asp?main=task&id_task=43) <br>
[Доп задача №7](https://acmp.ru/index.asp?main=task&id_task=264) <br>
[Доп задача №8](https://acmp.ru/index.asp?main=task&id_task=263) <br>
[Доп задача №9](https://acmp.ru/index.asp?main=task&id_task=297) <br>

# Ссылки
[Статья по ифам](https://javarush.com/groups/posts/2726-vetvlenie-v-java)
[Статья по циклам](https://javarush.com/groups/posts/cikly-java)