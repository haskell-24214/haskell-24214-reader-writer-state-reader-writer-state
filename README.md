# Домашняя работа №1. Монады Reader, Writer и State

Задачи на 3 и 4 -- на использование монад Reader и Writer,
задача на 5 -- на Writer и State.

## Как запустить тесты

```bash
stack test --test-arguments "tier0" # Тесты для задачи на тройку
stack test --test-arguments "tier1" # Тесты для задачи на четвёрку
stack test --test-arguments "tier2" # Тесты для задачи на пятерку
```

## Задача на 3

1. В файле `src/Tier0/Reader.hs` заданы следующие типы данных:

```haskell
data Environment = Environment
  { username :: String
  , isSuperUser :: Bool
  , host :: String
  , currentDir :: String
  } deriving Eq

type EnvironmentM = Reader Environment
```

Тип `Environment` очень грубо представляет часть окружения оболочки командной строки,
тип `EnvironmentM` немного упрощает работу с ним с помощью монады `Reader`.

Требуется написать код для форматирования приглашения командной строки на основе данных из некоторого
окружения с использованием монады `Reader`.

Приглашение командной строки -- текст, который отображается перед вводом какой-либо команды в оболочке
командной строки, например, для пользователя с именем `kilroy`, работающего на устройстве `sandbox`,
находящегося в папке `/home/kilroy/places-where-i-have-been`, приглашение может быть отформатировано
следующим образом:

```
kilroy@sandbox:/home/kilroy/places-where-i-have-been$
```

Для выполнения задания в том же файле `src/Tier0/Reader.hs` необходимо реализовать следующие функции:

* `formatUserName :: EnvironmentM String` -- читает из окружения информацию об имени пользователя и о том, является ли он [суперпользователем](https://ru.wikipedia.org/wiki/Root). Если пользователь не является суперпользователем, просто возвращает его имя, иначе возвращает `root` (будем считать, что в какой-нибудь операционной системе раздача прав это допускает).
* `formatHost :: EnvironmentM String` -- читает из окружения имя устройства, на котором работает пользователь, и просто возвращает его.
* `formatCurrentDir :: EnvironmentM String` -- читает из окружения и возвращает название текущей папки.
* `formatPrompt :: EnvironmentM String` -- возвращает приглашение ввода, отформатированное согласно следующему шаблону: `{username}@{hostname}:{cwd}$`,
  где `username`, `hostname`, `cwd` -- имя пользователя, имя устройства и расположение рабочей папки соответственно, полученные из окружения. Для суперпользователя в качестве имени пользователя должен выводиться `root`.
  
2. В файле `src/Tier0/Writer.hs` определен тип данных, представляющий [двоичное дерево](https://en.wikipedia.org/wiki/Binary_tree):

```
data Tree a = Leaf a | Branch (Tree a) a (Tree a) deriving Eq
```

Требуется реализовать функцию `sumAndTraceInOrder :: Num a => Tree a -> Writer [a] a`, которая при участии монады `Writer`:

* вычисляет сумму значений в вершинах дерева;
* выписывает значения в вершинах в порядке [центрированного обхода двоичного дерева](https://en.wikipedia.org/wiki/Tree_traversal#In-order,_LNR).

Например, для дерева `tree`

```
    4
   / \
  2   5
 / \
1   3
```

Вычисление внутри монады `Writer`

```haskell
runWriter sumAndTraceInOrder tree
```

должно породить значение `(15, [1,2,3,4,5])`.

## Задача на 4

_(Чтобы получить четвёрку, нужно также решить задачу на тройку)_

Предлагается реализовать следующие функции, определённые в файле `src/Tier1/Reader.hs`:

* ```haskell
  cd :: String -> EnvironmentM a -> EnvironmentM a
  cd dir env = undefined
  ```
Модифицирует некоторое окружение так, что к расположению рабочей папки справа приписывается строка `/${dir}`.
Например, если в некотором окружении `env` текущая папка называлась `/home/kilroy`, то в результате
применения `cd "places-where-i-have-been" env` получится новое окружение с текущей папкой `/home/kilroy/places-where-i-have-been`.

* ```haskell
  su :: EnvironmentM a -> EnvironmentM a
  ```

Модифицирует некоторое окружение так, что пользователь получает права суперпользователя (значение поля `isSuperUser` меняется на `True`).

## Задача на 5

_(Чтобы получить пятёрку, нужно также решить задачи на тройку и четвёрку)_

1. Предлагается реализовать функцию, определённую в файле `src/Tier2/Writer.hs`:

* ```haskell
  collectAndSumInOrder :: Num a => Tree a -> Writer (Sum a) [a]
  collectAndSumInOrder = undefined
  ```
  
  Поведение данной функции аналогично `sumAndTraceInOrder` с тем отличием, что основное вычисление и побочный эффект здесь меняются местами:
  * список значений в вершинах дерева в порядке центрированного обхода получается как _главный_ результат функции;
  * сумма значений накапливается как __побочный__ эффект.

2. С помощью монады `State` предлагается реализовать регистровый калькулятор, определенный в `src/Tier2/State.hs`, в соответствии с описанием его кодирующих инструкций и принципа
работы.

### Описание калькулятора

Калькулятор содержит регистры, предназначенные для хранения промежуточных
результатов вычислений и операндов арифметических команд, а именно:

- регистр `acc` -- аккумулятор, хранящий результат вычисления
  очередной арифметической команды;
- регистр `A`, в который записываются "левые" операнды бинарных операций;
- регистр `B`, в который записываются "правые" операнды бинарных операций;
- регистр `blink`, состоящий из одного бита, показывающего, в какой из набора
  регистров `{A, B}` следует произвести запись значения, закодированного
  специальными инструкциями. 

Регистры `acc`, `A`, `B` могут быть представлены программно в виде
целых чисел типа `Int`, регистр `blink` -- в виде значения типа `Bool`.

Перед началом процесса декодирования инструкций и исполнения последовательности
команд, регистры `acc`, `A`, `B` содержат значение `0`, в регистр `blink` записан
нулевой бит.

Далее начинается процесс разбора команд и выполнения соответствующих
им действий.  Программно команды могут быть представлены в виде
значений типа `String`. Представлены следующие команды:

- `+`, записывает в регистр `acc` сумму значений в регистрах `A`, `B`, затем снимает бит в регистре `blink` (записывает в него нулевой бит);
- `-`, записывает в регистр `acc` разность значений в регистрах `A` и `B`, затем снимает бит в регистре `blink`;
- `*`, записывает в регистр `acc` произведение значений в регистрах `A` и `B`, затем снимает бит в регистре `blink`;
- `/`, записывает в регистр `acc` частное от деления значения регистра `A` на значение регистра `B`, затем снимает бит в регистре `blink`; в случае деления на `0` все регистры зануляются;
- `swap`, меняет содержимое регистров `A` и `B` местами;
- `blink`, меняет значение бита в регистре `blink` на противоположное;
- `acc`, записывает содержимое регистра `acc` в один из регистров `{A, B}`: в `A`, если в регистре `blink` выставлен бит `0`, в `B`, в обратном случае; затем меняет значение бита в регистре `blink` на противоположное;
- `X`, где `X` - произвольное целое число -- записывает число `X` в один из регистров `{A, B}`: в `A`, если в регистре `blink` выставлен бит `0`, в `B`, в обратном случае; затем меняет значение бита в регистре `blink` на противоположное.

После исполнения последней команды, калькулятор печатает содержимое регистра `acc`. Программно такое поведение может быть представлено записью `acc` в стандартный поток вывода.
