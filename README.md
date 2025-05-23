# Справочник по языку программирования Kron

## 1. Обзор языка

Kron — функциональный язык программирования с поддержкой топологических вычислений. Он предназначен для написания лаконичных алгоритмов и работы с симплициальными комплексами. Основные характеристики:
- **Парадигма**: Функциональная, с лямбда-абстракциями, сопоставлением с образцом и неизменяемыми данными.
- **Особенности**: Модульная структура, встроенные топологические операции, интеграция с shell.
- **Области применения**: Общее программирование, вычислительная топология, прототипирование математических алгоритмов.
- **Реализация**: F# на платформе .NET 8.0.


## 2 Сборка и запуск
**Сборка**
```bash
dotnet build
```

Компилирует проект с использованием .NET SDK (версия 6.0+).

**Запуск**:
```bash
dotnet run -- <путь_к_файлу>
```

## 3. Синтаксис и семантика

### 3.1 Литералы
Kron поддерживает следующие типы литералов:
| Тип                | Синтаксис            | Семантика                          | Пример          |
|--------------------|----------------------|------------------------------------|-----------------|
| Целое число        | `[-]?[0-9]+`        | 64-битное целое со знаком          | `42`, `-10`     |
| Число с плавающей точкой | `[-]?[0-9]*\.[0-9]+` | Двойная точность (IEEE 754)       | `3.14`, `-0.5`  |
| Булево значение    | `true` \| `false`   | Логическое значение                | `true`, `false` |
| Строка             | `"[^"]*"`           | Последовательность символов UTF-8  | `"привет"`      |

### 3.2 Идентификаторы
- Формат: Начинаются с буквы или `_`, далее буквы, цифры или `_`.
- Пример: `x`, `factorial`, `_temp`, `Math.pi`.

### 3.3 Выражения
Выражения возвращают значения и вычисляются в окружении, сопоставляющем идентификаторы с значениями.

#### 3.3.1 Литеральное выражение
- **Синтаксис**: `<литерал>`.
- **Семантика**: Возвращает значение литерала.
- **Пример**: `42` → `VInt 42`.

#### 3.3.2 Переменная
- **Синтаксис**: `<идентификатор>` \| `<модуль>.<идентификатор>`.
- **Семантика**: Возвращает значение, связанное с идентификатором в текущем окружении. Для `модуль.идентификатор` выполняется поиск в модуле.
- **Пример**:
  ```kron
  x  # Возвращает значение x
  Math.pi  # Возвращает значение pi из модуля Math
  ```

#### 3.3.3 Лямбда-выражение
- **Синтаксис**: `lambda <идентификатор>+ -> <выражение>`.
- **Семантика**: Создает замыкание с параметрами и телом, сохраняя текущее окружение.
- **Пример**:
  ```kron
  lambda x y -> x + y
  ```

#### 3.3.4 Применение функции
- **Синтаксис**: `<выражение> <выражение>+`.
- **Семантика**: Вычисляет функцию и аргументы, затем применяет функцию к аргументам.
- **Пример**:
  ```kron
  f x y  # Применяет f к x и y
  ```

#### 3.3.5 Привязка let
- **Синтаксис**: `let <идентификатор> = <выражение> in <выражение>`.
- **Семантика**: Связывает значение первого выражения с идентификатором в окружении для вычисления второго выражения.
- **Пример**:
  ```kron
  let x = 5 in x + 1  # Возвращает 6
  ```

#### 3.3.6 Условное выражение
- **Синтаксис**: `if <выражение> then <выражение> else <выражение>`.
- **Семантика**: Вычисляет условие; если `VBool true`, вычисляет `then`-ветку, иначе `else`-ветку.
- **Пример**:
  ```kron
  if x == 0 then 1 else 2
  ```

#### 3.3.7 Сопоставление с образцом
- **Синтаксис**:
  ```kron
  match <выражение> with
  | <образец> -> <выражение>
  | <образец> -> <выражение>
  ...
  ```
- **Семантика**: Вычисляет выражение, сопоставляет результат с образцами в порядке их объявления, вычисляет соответствующее выражение.
- **Образцы**:
  - Переменная: `x` (связывает значение).
  - Подстановочный знак: `_` (игнорирует значение).
  - Литерал: `0`, `true`, `"строка"`.
  - Конструктор: `<имя>(<образец>, ...)` (для симплексов).
- **Пример**:
  ```kron
  match n with
  | 0 -> 1
  | n -> n * fact (n - 1)
  ```

#### 3.3.8 Список
- **Синтаксис**: `[<выражение> ...]`.
- **Семантика**: Создает список значений.
- **Пример**:
  ```kron
  [1 2 3]  # Возвращает VList [VInt 1; VInt 2; VInt 2]
  ```

#### 3.3.9 Команда shell
- **Синтаксис**: `!"<строка>"`.
- **Семантика**: Выполняет команду в bash, возвращает вывод как строку (`VString`).
- **Пример**:
  ```kron
  !"ls -la"  # Возвращает вывод команды ls -la
  ```

#### 3.3.10 Топологические выражения
- **Симплекс**:
  - **Синтаксис**: `simplex [<целое> ...]`.
  - **Семантика**: Создает комплекс, содержащий один симплекс (список целых чисел).
  - **Пример**: `simplex [0 1 2]` → `VComplex (Set [[0;1;2]])`.
- **Склеивание**:
  - **Синтаксис**: `glue <выражение> <выражение>`.
  - **Семантика**: Объединяет два комплекса в один.
  - **Пример**: `glue (simplex [0 1]) (simplex [1 2])`.
- **Топологические операции**:
  - **Синтаксис**: `<операция> <выражение>`.
  - **Операции**:
    - `boundary`: Возвращает (k-1)-грани комплекса.
    - `faces`: Возвращает все грани комплекса.
    - `dimension`: Возвращает размерность комплекса.
  - **Пример**:
    ```kron
    boundary (simplex [0 1 2])  # Возвращает Complex: [[0;1]; [0;2]; [1;2]]
    ```

### 3.4 Объявления
Объявления определяют модули, импорты и глобальные привязки.

#### 3.4.1 Привязка let
- **Синтаксис**:
  ```kron
  let <идентификатор> = <выражение>
  let rec <идентификатор> = <выражение>
  ```
- **Семантика**: Связывает значение с идентификатором в глобальном окружении. `rec` позволяет рекурсию.
- **Пример**:
  ```kron
  let x = 5
  let rec fact = lambda n -> if n == 0 then 1 else n * fact (n - 1)
  ```

#### 3.4.2 Импорт
- **Синтаксис**: `import <идентификатор>`.
- **Семантика**: Импортирует модуль в текущее окружение (в текущей реализации игнорируется).
- **Пример**:
  ```kron
  import std
  ```

#### 3.4.3 Модуль
- **Синтаксис**:
  ```kron
  module <идентификатор>
    <объявление>*
  end
  ```
- **Семантика**: Создает именованное окружение, содержащее объявления.
- **Пример**:
  ```kron
  module Math
    let pi = 3.14159
  end
  ```

### 3.5 Операторы
| Оператор | Арность | Типы аргументов       | Результат         | Пример         |
|----------|---------|-----------------------|------------------|----------------|
| `+`      | 2       | `VInt`, `VFloat`      | `VInt`, `VFloat` | `2 + 3` → `5`  |
| `-`      | 2       | `VInt`, `VFloat`      | `VInt`, `VFloat` | `5 - 2` → `3`  |
| `*`      | 2       | `VInt`, `VFloat`      | `VInt`, `VFloat` | `4 * 3` → `12` |
| `/`      | 2       | `VInt`, `VFloat`      | `VFloat`         | `10 / 2` → `5.0` |
| `==`     | 2       | `VInt`, `VFloat`      | `VBool`          | `5 == 5` → `true` |

### 3.6 Типы значений
Внутренние типы значений, определённые в `Runtime.fs`:
- `VInt`: 64-битное целое.
- `VFloat`: Число с плавающей точкой.
- `VBool`: Булево значение.
- `VString`: Строка.
- `VList`: Список значений.
- `VClosure`: Замыкание (параметры, тело, окружение).
- `VBuiltin`: Встроенная функция.
- `VComplex`: Симплициальный комплекс.
- `VModule`: Окружение модуля.
- `VUnit`: Пустое значение (`()`).

### 3.7 Семантика выполнения
- **Окружение**: Карта `Map<string, Value>` для хранения привязок.
- **Вычисление**: Рекурсивное вычисление выражений с учётом окружения.
- **Ошибки**: Исключение `RuntimeError` при неопределённых переменных, несоответствии типов или неверных аргументах.
- **Модули**: Иерархические окружения, доступные через точечную нотацию.

## 4. Стандартная библиотека

Модуль `std` (`src/Std.fs`) предоставляет функции ввода-вывода:
| Функция     | Аргументы | Возвращаемое значение | Описание                          |
|-------------|-----------|----------------------|-----------------------------------|
| `print`     | `obj`     | `Unit`               | Выводит значение без новой строки |
| `println`   | `obj`     | `Unit`               | Выводит значение с новой строкой  |

Пример:
```kron
import std
let _ = println "Привет, Kron!"  # Вывод: Привет, Kron!
```

### 4.1 Встроенные функции
Доступны без импорта:
| Функция     | Аргументы                | Возвращаемое значение | Описание                          |
|-------------|--------------------------|----------------------|-----------------------------------|
| `println`   | 1 (`Value`)              | `VUnit`              | Выводит значение с новой строкой  |
| `+`, `-`, `*`, `/` | 2 (`VInt` или `VFloat`) | `VInt` или `VFloat`  | Арифметические операции           |
| `==`        | 2 (`VInt` или `VFloat`)  | `VBool`              | Сравнение на равенство            |

## 5. Топологические конструкции

Топологические операции, определённые в `src/Topology.fs`, работают с симплициальными комплексами.

### 5.1 Типы
- **Симплекс**: Упорядоченный список целых чисел (`int list`), например, `[0 1 2]`.
- **Комплекс**: Множество симплексов (`Set<Simplex>`).

### 5.2 Операции
| Операция     | Аргументы       | Возвращаемое значение | Описание                                  |
|--------------|-----------------|----------------------|-------------------------------------------|
| `dimension`  | `VComplex`      | `VInt`               | Размерность первого симплекса (длина - 1) |
| `faces`      | `VComplex`      | `VComplex`           | Все подсимплексы (удаление одной вершины) |
| `boundary`   | `VComplex`      | `VComplex`           | (k-1)-грани комплекса                     |
| `glue`       | `VComplex`, `VComplex` | `VComplex`     | Объединение двух комплексов               |

Пример:
```kron
import std
let tri = simplex [0 1 2]
let bd = boundary tri
let _ = println bd  # Вывод: Complex: [[0;1]; [0;2]; [1;2]]
```

## 6. Примеры

### 6.1 Факториал (`factorial.kron`)
```kron
import std

let factorial =
    lambda n ->
        if n == 0 then 1 else n * factorial (n - 1)

let main = factorial 5
let _ = println main  # Вывод: 120
```

### 6.2 Числа Фибоначчи (`fibonacci.kron`)
```kron
import std

let rec fib = lambda n ->
    match n with
    | 0 -> 0
    | 1 -> 1
    | n -> fib (n - 1) + fib (n - 2)

let main = fib 10
let _ = println main  # Вывод: 55
```

### 6.3 Модули (`namespace_examples.kron`)
```kron
module Math
  let pi = 3.14159
  let sin = lambda x -> builtin_sin x
end

module Main
  import std
  let _ = println (Math.pi)  # Вывод: 3.14159
end
```

### 6.4 Тор (`torus.kron`)
```kron
import std

let square = simplex [0 1 2 3]
let torus = glue square square
let result = println (dimension torus)  # Вывод: 3
```

### 6.5 Граница треугольника (`triangle.kron`)
```kron
import std

let tri = simplex [0 1 2]
let bd = boundary tri
let _ = println bd  # Вывод: Complex: [[0;1]; [0;2]; [1;2]]
```

### 6.6 Минимальный пример (`mini.kron`)
```kron
if n == 0 then 1 else 2
```

## 7. Тестирование

Файл `run_tests.fsx` содержит определения топологических операций. Для тестирования:
1. Добавьте тесты в `run_tests.fsx`, используя фреймворки F# (например, Expecto).
2. Запустите скрипт:
   ```bash
   dotnet fsi run_tests.fsx
   ```
3. Пример теста:
   ```fsharp
   let testBoundary () =
       let tri = [0; 1; 2]
       let bd = boundary tri
       let expected = Set [[0;1]; [0;2]; [1;2]]
       assert (bd = expected)
   testBoundary ()
   ```

## 8. Структура проекта
- `src/`:
  - `Ast.fs`: Определение абстрактного синтаксического дерева.
  - `Parser.fs`: Парсер для преобразования кода в AST.
  - `Interpreter.fs`: Интерпретатор для вычисления выражений.
  - `Runtime.fs`: Типы значений и обработка ошибок.
  - `Std.fs`: Стандартная библиотека.
  - `Topology.fs`: Топологические операции.
  - `Program.fs`: Точка входа.
- `samples/`: Примеры программ.
- `Kron.fsproj`: Файл проекта .NET.
- `mini.kron`: Минимальный пример.
- `run_tests.fsx`: Тестовый скрипт.
