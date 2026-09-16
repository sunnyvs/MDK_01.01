# Ошибки компилятора: словарь

> Двадцать сообщений, которые ты увидишь чаще всего. Все они **воспроизведены** на твоём g++ 15.2 —
> тексты приведены дословно, а не пересказаны.
> Как устроено сообщение вообще — [раздел 1](01-osnovy.md#как-читать-ошибку-компилятора).

[← Начни отсюда](../00-НАЧНИ-ОТСЮДА.md) · [← Словарь терминов](10-slovar.md) · [Как ловить баги →](12-lovim-bagi.md) · [Примеры программ](../examples/README.md)

---

## Что в этом файле

- **[27. Что означают сообщения компилятора](#27-что-означают-сообщения-компилятора)**
  - [Три правила, которые экономят больше всего времени](#три-правила-которые-экономят-больше-всего-времени)
- **[Имена и опечатки](#имена-и-опечатки)**
  - [`'cout' was not declared in this scope`](#cout-was-not-declared-in-this-scope)
  - [`'vector' is not a member of 'std'`](#vector-is-not-a-member-of-std)
- **[Знаки препинания](#знаки-препинания)**
  - [`expected ';' after struct definition`](#expected--after-struct-definition)
  - [`expected '}' at end of input`](#expected--at-end-of-input)
  - [`jump to case label`](#jump-to-case-label)
- **[Функции](#функции)**
  - [`too many arguments to function`](#too-many-arguments-to-function)
  - [`undefined reference to 'calc(int)'`](#undefined-reference-to-calcint)
  - [`multiple definition of 'main'`](#multiple-definition-of-main)
  - [`control reaches end of non-void function`](#control-reaches-end-of-non-void-function)
  - [`reference to local variable returned`](#reference-to-local-variable-returned)
- **[Типы и `const`](#типы-и-const)**
  - [`assignment of read-only variable`](#assignment-of-read-only-variable)
  - [`narrowing conversion`](#narrowing-conversion)
  - [`invalid conversion from 'const char*' to 'int'`](#invalid-conversion-from-const-char-to-int)
  - [`no match for 'operator+=' ... 'const std::string'`](#no-match-for-operator--const-stdstring)
  - [`'int Box::size' is private within this context`](#int-boxsize-is-private-within-this-context)
- **[Страшные ошибки на пол-экрана](#страшные-ошибки-на-пол-экрана)**
- **[Предупреждения, которые нельзя игнорировать](#предупреждения-которые-нельзя-игнорировать)**
  - [`'count' is used uninitialized`](#count-is-used-uninitialized)
  - [`suggest parentheses around assignment used as truth value`](#suggest-parentheses-around-assignment-used-as-truth-value)
  - [`comparison of integer expressions of different signedness`](#comparison-of-integer-expressions-of-different-signedness)
  - [`conversion from 'double' to 'int' may change value`](#conversion-from-double-to-int-may-change-value)
  - [`unused variable 'unused'`](#unused-variable-unused)
- **[Куда смотреть, если сообщения нет вовсе](#куда-смотреть-если-сообщения-нет-вовсе)**

---

## 27. Что означают сообщения компилятора

### Три правила, которые экономят больше всего времени

**1. Читай только первую ошибку.** Остальные почти всегда — её последствия. Исправил первую, собрал заново, посмотрел, что осталось.

**2. Ошибка часто на строку выше, чем показано.** Компилятор сообщает о месте, где *заметил* проблему, а не где ты её *сделал*. Классика — забытая точка с запятой:

```cpp
int a = 5      // ← ошибка сделана здесь
int b = 6;     // ← а компилятор показывает сюда
```

```
error: expected ',' or ';' before 'int'
    4 |   int b = 6;
      |   ^~~
```

**3. Слово `note:` — это подсказка, а не вторая ошибка.** Компилятор объясняет, почему ругается, и часто прямо называет причину.

---

## Имена и опечатки

### `'cout' was not declared in this scope`

```
error: 'cout' was not declared in this scope; did you mean 'std::cout'?
```

Имени в этом месте не существует. Три причины по частоте: **опечатка**, забыт **`std::`**, переменная объявлена **ниже** или в другом блоке.

Компилятор часто подсказывает `did you mean` — и обычно угадывает верно.

```cpp
std::cout << cout;        // ❌ забыли std::
std::cout << std::cout;   // а вот так — бессмыслица, но имя существует
```

Если имя точно есть, а компилятор его не видит — проверь [область видимости](01-osnovy.md#область-видимости): переменная из цикла снаружи не существует.

### `'vector' is not a member of 'std'`

```
error: 'vector' is not a member of 'std'
```

Имя правильное, но **забыт `#include`**. Компилятор знает `std`, но в нём пока нет `vector`.

| Что не найдено | Чего не хватает |
|---|---|
| `vector` | `#include <vector>` |
| `sort`, `count_if`, `max_element` | `#include <algorithm>` |
| `map`, `set` | `#include <map>` / `<set>` |
| `accumulate` | `#include <numeric>` |
| `format` | `#include <format>` |
| `setw`, `setprecision` | `#include <iomanip>` |

Полная таблица — [раздел 23](09-spravka.md#23-заголовки--что-откуда).

Иногда подсказка сбивает с толку:

```
error: 'sort' is not a member of 'std'; did you mean 'qsort'?
```

`qsort` — это старая функция из C. Она **не** то, что тебе нужно: подключи `<algorithm>` и пиши `std::sort`.

## Знаки препинания

### `expected ';' after struct definition`

```
error: expected ';' after struct definition
    5 | }
      |  ^
      |  ;
```

После закрывающей скобки `struct` нужна точка с запятой. Компилятор даже показывает, что именно вставить.

```cpp
struct Point {
  int x = 0;
};              // ← вот она
```

### `expected '}' at end of input`

```
error: expected '}' at end of input
    6 | }
      |  ^
skobki.cpp:2:12: note: to match this '{'
    2 | int main() {
      |            ^
```

Не закрыта фигурная скобка. Ценность здесь в строке `note`: она показывает, **какая именно** скобка осталась открытой. В примере — та, что открыла `main`.

Быстрый способ найти: поставь курсор на скобку — VS Code подсветит парную. Если парной нет, подсветки не будет.

### `jump to case label`

```
error: jump to case label
    9 |   case 2:
      |        ^
note:   crosses initialization of 'int x'
    6 |     int x = 5;
```

Внутри `case` объявлена переменная, а фигурных скобок нет. Лечится скобками вокруг этого `case`:

```cpp
case 1: {                  // ← открыли
  int x = 5;
  std::cout << x;
  break;
}                          // ← закрыли
```

Подробнее — [раздел 7](03-logika-cikly.md#switch--выбор-из-фиксированного-списка).

## Функции

### `too many arguments to function`

```
error: too many arguments to function 'int square(int)'
```

Вызвал с большим числом аргументов, чем принимает функция. Компилятор тут же показывает её настоящую сигнатуру — сверься с ней.

Бывает и наоборот: `too few arguments to function`.

### `undefined reference to 'calc(int)'`

```
ld.exe: main.cpp:(.text+0x13): undefined reference to `calc(int)'
collect2.exe: error: ld returned 1 exit status
```

**Это ошибка не компилятора, а компоновщика** — и она устроена иначе. Обрати внимание: в ней нет номера строки твоего кода, зато есть `ld.exe` и `collect2.exe`.

Смысл: функция **объявлена**, компилятор поверил, что где-то есть её тело, — а тела нет.

Причины:

```cpp
int calc(int x);          // объявили (прототип)
// ...тело так и не написали → undefined reference

int Calc(int x) { ... }   // написали, но имя с другой буквы → тоже undefined reference
```

### `multiple definition of 'main'`

```
ld.exe: two_main.cpp:(.text+0x0): multiple definition of `main';
        net_tela.cpp:(.text+0x0): first defined here
collect2.exe: error: ld returned 1 exit status
```

Собираются сразу два файла, и в каждом свой `main`. В программе он должен быть один.

Обычная причина — сборка целой папки (`Ctrl+K B`) вместо одного файла. Запускай через `Ctrl+Alt+R`.

### `control reaches end of non-void function`

```
warning: control reaches end of non-void function [-Wreturn-type]
```

Функция обещает вернуть значение, но существует путь, на котором `return` не выполнится:

```cpp
int getValue(bool flag) {
  if (flag)
    return 1;
              // ❌ а если flag == false? Вернётся мусор
}
```

Формально это предупреждение, по сути — ошибка: значение будет случайным. Дописывай `return` для всех веток.

### `reference to local variable returned`

```
warning: reference to local variable 'result' returned [-Wreturn-local-addr]
```

Функция вернула **ссылку** на свою локальную переменную, а та исчезает вместе с функцией (её «тарелка» снимается со [стека вызовов](04-funkcii.md#стек-вызовов-как-я-сюда-попал)). Ссылка повисает — читать по ней нельзя.

```cpp
const std::string &bad() {
  std::string result = "привет";
  return result;        // ❌ result умрёт на выходе — ссылка укажет в никуда
}

std::string good() {    // ✅ верни по ЗНАЧЕНИЮ: копия/move живёт дальше, это не тормозит (RVO)
  std::string result = "привет";
  return result;
}
```

Формально предупреждение, по сути — гарантированный баг. Возвращай по значению; ссылку возвращают только на то, что живёт дольше функции (например, на элемент переданного контейнера).

## Типы и `const`

### `assignment of read-only variable`

```
error: assignment of read-only variable 'SIZE'
```

Попытка изменить `const`. Либо значение и правда должно меняться (тогда убери `const`), либо ты меняешь не ту переменную.

### `narrowing conversion`

```
error: narrowing conversion of '3.1400000000000001e+0' from 'double' to 'int' [-Wnarrowing]
```

Появляется при инициализации через **фигурные скобки** `{}`, когда значение не влезает без потери. Это не придирка, а защита — `{}` заставляет написать преобразование явно:

```cpp
int a{3.14};      // ❌ фигурные скобки запрещают терять дробную часть
int b = 3.14;     // соберётся (b = 3), но с -Wconversion предупредит о потере
int c{3};         // ✅ ровно влезает
```

Подробнее про это поведение — [раздел 3](01-osnovy.md#инициализация-в-фигурных-скобках-ловит-сужение).

### `invalid conversion from 'const char*' to 'int'`

```
error: invalid conversion from 'const char*' to 'int' [-fpermissive]
```

Присваиваешь текст числу или наоборот. `const char*` в сообщении означает «строковый литерал в кавычках».

```cpp
int n = "текст";              // ❌
int n = std::stoi("42");      // ✅ если нужно превратить текст в число
```

### `no match for 'operator+=' ... 'const std::string'`

```
error: no match for 'operator+=' (operand types are 'const std::string' and 'const char [2]')
```

Слово `const` в типе — главная подсказка. Ты пытаешься **изменить то, что получил только для чтения**:

```cpp
void show(const std::string &s) {
  s += "!";        // ❌ параметр помечен const
}
```

Либо убери `const` (если менять действительно надо), либо не меняй. Про выбор — [раздел 9](04-funkcii.md#три-способа-передать-аргумент).

### `'int Box::size' is private within this context`

```
error: 'int Box::size' is private within this context
```

Поле спрятано внутри класса. У `class` всё приватно по умолчанию, у `struct` — открыто. Для учебных задач достаточно писать `struct`.

## Страшные ошибки на пол-экрана

Иногда одна строка твоего кода порождает простыню из системных заголовков:

```
In file included from D:/msys64/ucrt64/include/c++/15.2.0/bits/stl_tree.h:67,
                 from D:/msys64/ucrt64/include/c++/15.2.0/map:64,
                 from main.cpp:1:
.../stl_function.h: In instantiation of 'constexpr bool std::less<_Tp>::operator()...
  554 |         if (__i == end() || key_comp()(__k, (*__i).first))
```

Пугаться не надо — читается это по одному правилу:

> **Найди в простыне первую строку, где упомянут ТВОЙ файл.** Всё остальное — внутренности библиотеки, туда лезть незачем.

В примере выше настоящая причина спрятана дальше: `no match for 'operator<'` для типа `Point`. То есть `struct` используется ключом `map`, а сравнивать его компилятор не умеет.

```cpp
struct Point {
  int x = 0;
  int y = 0;
  bool operator<(const Point &other) const {   // ← вот чего не хватало
    if (x != other.x) return x < other.x;
    return y < other.y;
  }
};
```

Подробнее — [раздел 12](06-konteynery.md#свой-struct-как-ключ).

**Признак этой группы ошибок:** в тексте есть `In instantiation of`, `required from` и пути внутрь `include/c++`. Значит дело в шаблоне — контейнере или алгоритме, — и почти всегда причина в том, что твой тип чего-то не умеет.

## Предупреждения, которые нельзя игнорировать

Формально программа собралась. По сути — три из них означают настоящую ошибку.

### `'count' is used uninitialized`

```
warning: 'count' is used uninitialized [-Wuninitialized]
```

Переменная используется до того, как ей задали значение. Внутри — мусор. Правило простое: **задавай значение при объявлении**.

### `suggest parentheses around assignment used as truth value`

```
warning: suggest parentheses around assignment used as truth value [-Wparentheses]
```

Написал `=` вместо `==` в условии:

```cpp
if (x = 5)      // ❌ присвоили 5, условие всегда истинно
if (x == 5)     // ✅
```

### `comparison of integer expressions of different signedness`

```
warning: comparison of integer expressions of different signedness:
         'int' and 'std::vector<int>::size_type' [-Wsign-compare]
```

Сравниваешь `int` с размером контейнера. Почему это опасно и три способа починить — [раздел 2](01-osnovy.md#stdsize_t--почему-компилятор-ругается-на-vsize).

### `conversion from 'double' to 'int' may change value`

```
warning: conversion from 'double' to 'int' may change value [-Wfloat-conversion]
```

Дробное кладётся в целое, дробная часть теряется. Если так и задумано — напиши это явно, и предупреждение исчезнет:

```cpp
int n = static_cast<int>(3.99);   // ✅ 3, и видно, что отбрасывание намеренное
```

### `unused variable 'unused'`

```
warning: unused variable 'unused' [-Wunused-variable]
```

Самое безобидное — но полезное. Часто означает, что ты завёл переменную и забыл её использовать, а считаешь в другом месте по ошибке.

---

## Куда смотреть, если сообщения нет вовсе

Программа собралась, но работает не так. Компилятор молчит — значит, ошибка не в синтаксисе, а в логике. Это уже другой инструмент: [Как ловить баги](12-lovim-bagi.md).

---

[← Начни отсюда](../00-НАЧНИ-ОТСЮДА.md) · [← Словарь терминов](10-slovar.md) · [Как ловить баги →](12-lovim-bagi.md) · [Примеры программ](../examples/README.md)
