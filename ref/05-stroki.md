# Строки

> Работа с текстом, поиск, разбиение на слова
>
> Это [раздел 10](#10-stdstring) справочника. Нумерация сквозная во всех файлах: ссылка «см. [раздел 18](07-algoritmy.md#18-алгоритмы)» ведёт в [Итераторы и алгоритмы](07-algoritmy.md), а полный список — в [оглавлении](../00-НАЧНИ-ОТСЮДА.md).

[← Начни отсюда](../00-НАЧНИ-ОТСЮДА.md) · [← Функции](04-funkcii.md) · [Контейнеры: vector, map, set, pair →](06-konteynery.md) · [Примеры программ](../examples/README.md)

---

## Что в этом файле

- **[10. `std::string`](#10-stdstring)**
  - [Основные операции](#основные-операции)
  - [Вставка, удаление, замена](#вставка-удаление-замена)
  - [Поиск](#поиск)
  - [`starts_with` и `ends_with` — начало и конец строки](#starts_with-и-ends_with--начало-и-конец-строки)
  - [Преобразования числа и строки](#преобразования-числа-и-строки)
  - [Разбить строку по разделителю](#разбить-строку-по-разделителю)
  - [Перебор символов](#перебор-символов)
  - [Типовые задачи](#типовые-задачи)
  - [Обрезать пробелы (trim)](#обрезать-пробелы-trim)
  - [Сравнение строк](#сравнение-строк)

---

## 10. `std::string`

```cpp
#include <string>

std::string s = "Привет";
std::string empty;                    // пустая строка, не мусор — это безопасно
std::string repeated(5, 'x');         // → xxxxx   пять одинаковых символов
```

### Основные операции

```cpp
std::string s = "Привет";

s += " мир";                          // приклеить в конец
std::cout << s << "\n";               // → Привет мир

std::cout << s.length() << "\n";      // → 19  ⚠️ БАЙТЫ, не буквы (см. предупреждение ниже)
std::cout << s.size() << "\n";        // → 19  size() и length() — одно и то же
std::cout << s.empty() << "\n";       // → 0   пустая ли строка

std::string part = s.substr(0, 12);   // подстрока: с позиции 0, длиной 12 байт
std::cout << part << "\n";            // → Привет

s.clear();                            // очистить
std::cout << s.empty() << "\n";       // → 1
```

> **Кириллица в `std::string` считается байтами, а не буквами.** У строки `"Привет"` `length()` вернёт **12**, а `s[0]` даст половину буквы: в UTF-8 русская буква занимает два байта. Для латиницы и цифр всё как ожидается. Практический вывод: перебирать русский текст по `s[i]` и резать его через `substr` **нельзя** — получишь битые символы. Для учебных задач с посимвольной обработкой бери латиницу.

### Вставка, удаление, замена

Строку можно править на месте — у неё есть готовые методы, не надо собирать новую вручную:

```cpp
#include <iostream>
#include <string>

int main() {
  std::string s = "Hello";
  s.push_back('!');       // добавить один символ в конец  → Hello!
  s.append(" мир");       // добавить строку в конец       → Hello! мир
  s.insert(5, ",");       // вставить "," на позицию 5      → Hello,! мир

  std::string t = "abcdef";
  t.erase(2, 3);          // удалить 3 символа с позиции 2 (cde) → abf
  t.replace(0, 1, "X");   // заменить 1 символ с позиции 0       → Xbf

  std::cout << s << "\n" << t << "\n";   // → Hello,! мир   и   Xbf
  return 0;
}
```

| Метод | Что делает |
|---|---|
| `s += "..."` / `s.append("...")` | приклеить строку в конец |
| `s.push_back(c)` / `s.pop_back()` | добавить / убрать один символ с конца |
| `s.insert(pos, "...")` | вставить на позицию `pos` |
| `s.erase(pos, n)` | удалить `n` символов с позиции `pos` |
| `s.replace(pos, n, "...")` | заменить `n` символов на другую строку |
| `s.front()` / `s.back()` | первый / последний символ |

> `s[i]` не проверяет границы (за пределами — мусор или порча памяти), а `s.at(i)` проверяет и при выходе бросает исключение. Пока учишься — держи флаг `-D_GLIBCXX_ASSERTIONS` (см. [основы](01-osnovy.md#флаг-который-ловит-выход-за-границы)), он ловит и `s[i]` за границей.

### Поиск

```cpp
std::string s = "hello world";

std::size_t pos = s.find("world");             // возвращает позицию начала
if (pos != std::string::npos)                  // npos = «не найдено», особое значение
  std::cout << "Найдено на позиции " << pos;   // → Найдено на позиции 6
else
  std::cout << "Не найдено";

std::cout << s.find('o') << "\n";              // → 4   первое вхождение символа
std::cout << s.rfind('o') << "\n";             // → 7   последнее вхождение
std::cout << s.find("xyz") << "\n";            // → 18446744073709551615 (это и есть npos)
```

> `find` **никогда** не возвращает `-1`. Сравнивай только с `std::string::npos`.

### `starts_with` и `ends_with` — начало и конец строки

Частый вопрос — «начинается ли строка с…» или «кончается ли на…». Через `find` это писать неудобно и легко ошибиться (`find(x) == 0` для начала — да, но для конца формула громоздкая). В C++20 для этого есть прямые методы:

```cpp
std::string s = "hello.txt";

std::cout << s.starts_with("hello") << "\n";   // → 1
std::cout << s.ends_with(".txt") << "\n";      // → 1
std::cout << s.ends_with(".cpp") << "\n";      // → 0
std::cout << s.starts_with('h') << "\n";       // → 1   можно проверить и один символ
```

Читается как предложение и не требует возиться с позициями. Типичное применение — разбор команд и имён файлов:

```cpp
if (command.starts_with("add "))          // команда вида "add молоко"
  std::cout << "добавляем: " << command.substr(4) << "\n";

if (fileName.ends_with(".cpp"))
  std::cout << "это исходник C++\n";
```

> Работает по тем же байтам, что и остальная строка, — для сравнения с латинскими префиксами (команды, расширения файлов) это ровно то, что нужно. С русскими префиксами помни про [двухбайтовость](#10-stdstring), но там обычно и не проверяют «начинается ли с буквы».

### Преобразования числа и строки

```cpp
int n = 42;
double d = 3.14;

std::string s1 = std::to_string(n);      // → "42"
std::string s2 = std::to_string(d);      // → "3.140000"

int back = std::stoi("42");              // строка → int
double backD = std::stod("3.14");        // строка → double
long long backLL = std::stoll("9999999999");   // строка → long long

std::cout << back * 2 << "\n";           // → 84

int bad = std::stoi("abc");              // ❌ бросит исключение std::invalid_argument
```

### Разбить строку по разделителю

Строку часто нужно разложить на части. Если разделитель — **пробел**, работает `istringstream` и `>>` (как из `cin`):

```cpp
std::string line = "10 20 30 40";
std::istringstream stream(line);       // строка как поток
int x = 0;
int sum = 0;
while (stream >> x)                     // читаем числа по одному, как из cin
  sum += x;
// sum == 100
```

А если разделитель — **не пробел** (`;`, `,`, `-`), у `getline` есть третий аргумент — символ-разделитель. Так разбирают CSV:

```cpp
#include <iostream>
#include <sstream>
#include <string>
#include <vector>

int main() {
  std::string csv = "Аня;21;Москва";
  std::istringstream stream(csv);
  std::string field;
  std::vector<std::string> fields;

  while (std::getline(stream, field, ';'))   // третий аргумент — разделитель полей
    fields.push_back(field);

  std::cout << fields.size() << "\n";        // → 3
  std::cout << fields[0] << "\n";            // → Аня
  return 0;
}
```

Разделять по пробелам через `getline(..., ' ')` **не стоит**: два пробела подряд дадут пустое поле. Для пробелов — `>>` (он сам пропускает лишние), для остального — `getline` с разделителем. Разбиение на слова целиком — в [типовых задачах](#типовые-задачи) ниже.

### Перебор символов

```cpp
std::string s = "hello";

// Чтение
for (char c : s)
  std::cout << c << "-";                 // → h-e-l-l-o-

// Изменение — нужна ссылка
for (char &c : s)
  c = static_cast<char>(std::toupper(static_cast<unsigned char>(c)));
std::cout << s;                             // → HELLO

// По индексам, если нужна позиция
for (std::size_t i = 0; i < s.length(); ++i)
  std::cout << i << ":" << s[i] << " ";     // → 0:H 1:E 2:L 3:L 4:O
```

> **Почему у `toupper` два преобразования подряд.** Внутрь надо передать `unsigned char`: у обычного `char` для не-ASCII байта значение отрицательное, а `toupper` от отрицательного числа — неопределённое поведение (может и упасть). Наружу `toupper` отдаёт `int`, и обратно в `char` его кладём явно — иначе `-Wconversion` справедливо поругается. Выглядит громоздко, зато это единственная запись, которая верна всегда:
> `c = static_cast<char>(std::toupper(static_cast<unsigned char>(c)));`
>
> На кириллицу это всё равно не подействует: `toupper` работает только с латиницей.

### Типовые задачи

**Подсчёт символов:**

```cpp
std::string s = "programming";
int count = 0;

for (char c : s)
  if (c == 'm')
    ++count;
std::cout << count;                     // → 2
```

**Переворот строки:**

```cpp
#include <algorithm>

std::string s = "hello";
std::reverse(s.begin(), s.end());       // меняет строку на месте
std::cout << s;                         // → olleh
```

**Проверка на палиндром:**

```cpp
bool isPalindrome(const std::string &s) {
  std::size_t left = 0;
  std::size_t right = s.length();               // ⚠️ не length()-1: для пустой строки будет беда
  while (left + 1 < right) {                    // идём навстречу друг другу
    if (s[left] != s[right - 1])
      return false;                             // нашли расхождение — не палиндром
    ++left;
    --right;
  }
  return true;
}

std::cout << isPalindrome("level") << "\n";     // → 1
std::cout << isPalindrome("hello") << "\n";     // → 0
```

**Разбиение строки на слова:**

```cpp
#include <sstream>
#include <vector>

std::string text = "один два три";
std::istringstream stream(text);         // превращаем строку в поток
std::vector<std::string> words;
std::string word;

while (stream >> word)                   // читаем как из cin — по пробелам
  words.push_back(word);

std::cout << words.size() << "\n";       // → 3
std::cout << words[1] << "\n";           // → два
```

**Замена всех вхождений:**

```cpp
std::string s = "a-b-c-d";
std::size_t pos = 0;

while ((pos = s.find('-', pos)) != std::string::npos) {   // ищем начиная с pos
  s[pos] = '+';                                           // заменяем
  ++pos;                                                  // сдвигаемся, иначе зациклимся
}
std::cout << s;                                           // → a+b+c+d
```

### Обрезать пробелы (trim)

Ввод часто приходит с лишними пробелами по краям («` да `» вместо «`да`»), и они ломают сравнения. Готового `trim` в стандартной библиотеке нет — вот короткая надёжная заготовка на `find_first_not_of` / `find_last_not_of`:

```cpp
#include <iostream>
#include <string>

std::string trim(const std::string &s) {
  std::size_t start = s.find_first_not_of(" \t\n\r");   // первый непробельный символ
  if (start == std::string::npos)
    return "";                                          // строка из одних пробелов
  std::size_t end = s.find_last_not_of(" \t\n\r");      // последний непробельный
  return s.substr(start, end - start + 1);
}

int main() {
  std::cout << "[" << trim("   привет   ") << "]\n";     // → [привет]
  std::cout << "[" << trim("\t да \n") << "]\n";         // → [да]
  return 0;
}
```

`find_first_not_of(" \t\n\r")` находит первый символ, которого **нет** в наборе пробельных, а `find_last_not_of` — последний; между ними и лежит полезная часть. Пробелы внутри строки функция не трогает — только по краям.

### Сравнение строк

```cpp
std::string a = "apple";
std::string b = "banana";

std::cout << (a == b) << "\n";     // → 0   сравнение по содержимому, всё как ожидаешь
std::cout << (a < b) << "\n";      // → 1   лексикографически: "apple" раньше "banana"
std::cout << (a != b) << "\n";     // → 1

// Сравнение без учёта регистра (для латиницы) — своей функцией
bool equalIgnoreCase(const std::string &x, const std::string &y) {
  if (x.length() != y.length())            // разной длины — точно не равны, дальше не смотрим
    return false;
  for (std::size_t i = 0; i < x.length(); ++i) {
    unsigned char a = static_cast<unsigned char>(x[i]);   // см. заметку про toupper выше
    unsigned char b = static_cast<unsigned char>(y[i]);
    if (std::tolower(a) != std::tolower(b))
      return false;                        // нашли различие — выходим сразу
  }
  return true;                             // дошли до конца — все символы совпали
}
std::cout << equalIgnoreCase("Hello", "hELLO");    // → 1
```

---

---

[← Начни отсюда](../00-НАЧНИ-ОТСЮДА.md) · [← Функции](04-funkcii.md) · [Контейнеры: vector, map, set, pair →](06-konteynery.md) · [Примеры программ](../examples/README.md)
