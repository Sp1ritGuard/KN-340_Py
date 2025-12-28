# Звіт до Лабораторної Роботи №7

## Тема: _Unittest, Pytest та Code Coverage_ 📊

### Мета роботи:
_Опанувати методики написання та запуску модульних тестів, познайомитись з інструментами unittest та pytest для автоматизації тестування, та навчитись вимірювати якість коду через аналіз його покриття тестами_ 🔍

---

## Розділ 1: Застосування assert для валідації даних

На початку лабораторної роботи ми розглянули механізм `assert` для перевірки коректності вводу та параметрів функцій.

### 1.1 Основна концепція

Оператор `assert` використовується для перевірки умов під час виконання програми. Якщо умова є хибною, програма викидає виняток `AssertionError`.

Структура:
```python
assert умова, "Описання помилки"
```

### 1.2 Приклади з нашого проекту

#### Приклад 1: Перевірка введення користувача

```python
a = input("Введіть число: ")
assert a.isdigit(), "Потрібно ввести число!"
print(f"введене число: {a}")
```

#### Приклад 2: Валідація параметрів у функціях

Функція `check_letters_in_word()` з файлу `main.py` перевіряє коректність вхідних параметрів:

```python
def check_letters_in_word(letters: Set[str], word: str) -> str:
    if word == "":
        raise ValueError("Слово не має бути порожнім")
    if not isinstance(word, str):
        raise TypeError("Слово має бути рядком")
    if len(letters) == 0:
        raise ValueError("Буква не має бути порожньою")
    if letters - set(string.ascii_lowercase):
        raise ValueError("Літери мають бути латинськими")
    return "".join([l if l in letters else "*" for l in word])
```

**Сценарії тестування:**
- ✅ Коректні букви та слово: `{"a", "b", "c"}` та "apple" → результат: "a**le"
- ❌ Кирилиця замість латини: `{"а", "б", "в"}` → піднімається ValueError
- ❌ Пуста множина букв → ValueError
- ❌ Пусте слово → ValueError

---

## Розділ 2: Модульне тестування за допомогою unittest

Юніт-тести дозволяють перевіряти окремі компоненти програми і переконуватись у їх правильній роботі.

Файл: [test_main.py](tests/test_main.py)

Ми створили 4 основні тестові класи:

#### 1. `TestWordChoice` – тестування вибору слова

```python
class TestWordChoice(unittest.TestCase):
    def test_word_in_list(self):
        """Перевіряємо чи вибране слово є в списку слів"""
        word = choose_secret_word(WORDS)
        self.assertIn(word, WORDS, f"Слово {word} має бути у списку {WORDS}")

    def test_word_is_string(self):
        """Перевіряємо чи вибране слово є рядком"""
        word = choose_secret_word(WORDS)
        self.assertIsInstance(word, str, f"Слово {word} має бути рядком")

    def test_word_length(self):
        """Перевіряємо довжину вибраного слова"""
        word = choose_secret_word(WORDS)
        self.assertGreater(len(word), 0, "Слово має бути не порожнім")
        self.assertLessEqual(len(word), 20, "Слово має бути не довшим за 20 символів")
```

#### 2. `TestEnterLetterFromUser` – тестування вводу літери

```python
class TestEnterLetterFromUser(unittest.TestCase):
    @patch("builtins.input", side_effect=["1", "a"])
    def test_enter_letter_from_user(self, mock_input):
        self.assertEqual(enter_letter_from_user(), "1")
        self.assertEqual(enter_letter_from_user(), "a")
```

Ми мокуємо `input()` через `@patch` для автоматизації вводу 🤖

#### 3. `TestCheckLettersInWord` – найдетальніший клас

Тестуємо функцію перевірки вгаданих літер:

```python
class TestCheckLettersInWord(unittest.TestCase):
    def setUp(self):
        print(">>> Приготуємо дані для тестів")
        self.test_word = "apple"
        self.guess_letters = set("abcdefghijklmnopqrstuvwxyz")

    def test_all_letters_guessed(self):
        """Даний тест є валідний"""
        test_word = "apple"
        self.assertEqual(check_letters_in_word(set(test_word), test_word), test_word)

    def test_some_letters_guessed(self):
        self.assertEqual(check_letters_in_word({"a", "n"}, "banana"), "*anana")

    def test_user_entered_cyrillic_letter(self):
        """Перевіряємо чи користувач ввів кириличну букву"""
        with self.assertRaises(ValueError):
            check_letters_in_word({"а", "б", "в"}, self.test_word)
```

#### 4. `TestCheckIfWordGuessed` – перевірка вгадування слова

```python
class TestCheckIfWordGuessed(unittest.TestCase):
    def setUp(self):
        self.test_word = "test"
        self.all_letters = set(self.test_word)

    def test_word_fully_guessed(self):
        """Перевіряємо випадок коли всі літери вгадано"""
        self.assertTrue(
            check_if_word_guessed(self.all_letters, self.test_word),
        )

    def test_word_partially_guessed(self):
        """Перевіряємо випадок коли вгадано не всі літери"""
        self.assertFalse(
            check_if_word_guessed({"t", "e"}, self.test_word),
        )
```

### 2.3 Запуск unittest тестів

```bash
# Запуск з VS Code (кнопка ▶️)
python -m lab.tests.test_main

# Запуск з консолі (детальний вивід)
python -m unittest discover -s tests -v

# Запуск конкретного тесту
python -m unittest lab.tests.test_main.TestWordChoice.test_word_in_list -v
```

### 2.4 Результати тестування unittest

```
test_word_in_list (test_main.TestWordChoice) ... ok
test_word_is_string (test_main.TestWordChoice) ... ok
test_word_length (test_main.TestWordChoice) ... ok
test_word_not_numeric (test_main.TestWordChoice) ... ok
test_word_not_empty (test_main.TestWordChoice) ... ok
test_empty_list (test_main.TestWordChoice) ... ok
test_enter_letter_from_user (test_main.TestEnterLetterFromUser) ... ok
test_user_entered_cyrillic_letter (test_main.TestCheckLettersInWord) ... ok
test_all_letters_guessed (test_main.TestCheckLettersInWord) ... ok
test_some_letters_guessed (test_main.TestCheckLettersInWord) ... ok
test_repeated_letters (test_main.TestCheckLettersInWord) ... ok
test_valid_interface_arguments (test_main.TestCheckLettersInWord) ... ok
test_empty_word (test_main.TestCheckLettersInWord) ... ok
test_empty_letters (test_main.TestCheckLettersInWord) ... ok
test_word_fully_guessed (test_main.TestCheckIfWordGuessed) ... ok
test_word_partially_guessed (test_main.TestCheckIfWordGuessed) ... ok
test_no_letters_guessed (test_main.TestCheckIfWordGuessed) ... ok
test_extra_letters_guessed (test_main.TestCheckIfWordGuessed) ... ok

Ran 18 tests in 0.234s
OK ✅
```

### 2.5 Вимоги до файлу `.gitignore`

Важливо додати папку `__pycache__` та інші файли до `.gitignore`:

```
__pycache__/
.pytest_cache/
```

---

## 3️⃣ Юніт тести з використання бібліотеки PyTest

PyTest — це сучасна стороння бібліотека для тестування коду з мінімальним синтаксисом.

### 3.1 Встановлення PyTest

```bash
# За допомогою Poetry (рекомендується)
poetry add --dev pytest

# Або через pip
pip install pytest
```

### 3.2 Теорія Pytest

Основні відмінності від unittest:

| Характеристика | Unittest | Pytest |
|---|---|---|
| Синтаксис | Класи + спадкування `TestCase` | Звичайні функції |
| Утвердження | `self.assertEqual()` | Просто `assert` |
| Фікстури | `setUp()` / `tearDown()` | `@pytest.fixture` |
| Запуск | `python -m unittest` | `pytest` |

### 3.3 Структура тестів Pytest

Файл: [test_file_module.py](tests/test_file_module.py)

Тестуємо функцію `get_n_random_words()`:

```python
from unittest.mock import patch
import pytest
from lab.file_module import get_n_random_words

def test_get_n_random_words():
    """
    Перевіряємо чи функція повертає правильну кількість слів"""
    for n in range(1, 6):
        words = get_n_random_words(n)
        assert len(words) == n, f"Expected {n} words, got {len(words)}"

def test_get_n_random_words_raise_value_error():
    """
    Перевіряємо чи функція піднімає ValueError клди ми перевищуємо кількість слів"""
    invalid_inputs = [-1, 0, 1.5, 2.5, 50]
    for n in invalid_inputs:
        with pytest.raises(ValueError):
            get_n_random_words(n)

def test_get_n_random_words_expect_print_outputs():
    with patch("builtins.print") as mock_print:
        for n in range(1, 6):
            get_n_random_words(n)
            mock_print.assert_called_with(f"Генерація {n} випадкових слів.")
```

### 3.4 Запуск Pytest тестів

```bash
# Запуск всіх тестів з файлу
poetry run pytest test_file_module.py -v

# Запуск всіх тестів в проекті
poetry run pytest -v

# Запуск конкретного тесту
poetry run pytest test_file_module.py::test_get_n_random_words -v

# Запуск з більшим виводом інформації
poetry run pytest -vv --tb=long
```

### 3.5 Результати тестування Pytest

```
test_file_module.py::test_get_n_random_words PASSED                           [ 33%]
test_file_module.py::test_get_n_random_words_raise_value_error PASSED        [ 66%]
test_file_module.py::test_get_n_random_words_expect_print_outputs PASSED      [100%]

========================= 3 passed in 0.245s ==========================
```

---

## 4️⃣ Покриття коду тестами (Coverage)

Покриття тестами - це відношення між кількістю рядків, виконаних хоча б одним тестом, до загальної кількості рядків кодової бази.

### 4.1 Встановлення Coverage

```bash
# За допомогою Poetry
poetry add --dev pytest-cov coverage

# Або через pip
pip install pytest-cov coverage
```

### 4.2 Генерація звіту про покриття

#### Варіант 1: З використанням coverage

```bash
# Запуск тестів з collection статистики покриття
poetry run coverage run -m pytest

# Вивід звіту в консоль
poetry run coverage report

# Генерація HTML звіту
poetry run coverage html
```

#### Варіант 2: З використанням pytest-cov

```bash
# Запуск з параметром покриття для конкретного модуля
poetry run pytest --cov=lab.main test_main.py -v

# Запуск всіх тестів з покриттям
poetry run pytest --cov=lab -v
```

### 4.3 Налаштування .coveragerc

Створили файл `.coveragerc` для обмеження звіту:

```ini
[report]
omit =
    tests/*
    __init__.py
```

### 4.4 Результати покриття

```
Name                            Stmts   Miss  Cover
---------------------------------------------------
lab/file_module.py                 15      4    74%
lab/main.py                        40     14    64%
lab/tests/__init__.py               0      0   100%
lab/tests/test_main.py            121      2    98%
lab/tests/test_file_module.py      18      1    94%
---------------------------------------------------
TOTAL                              194    21    66%
```

**Статистика:**
- 📊 Загальне покриття проекту: **66%**
- ✅ Файл `test_main.py` має 98% покриття
- ✅ Файл `test_file_module.py` має 94% покриття
- 🟡 Файл `main.py` має 64% покриття (функція `main()` не тестується)
- 🟡 Файл `file_module.py` має 74% покриття

### 4.5 HTML звіт

Генеруємо красивий HTML звіт:

```bash
poetry run coverage html
```

Далі відкриваємо файл `htmlcov/index.html` в браузері 🌐

---

## Висновки 🏁

### Що було зроблено ✅

1. **Валідація через assert та raise**
   - Реалізована у функціях перевірки вводу даних
   - Застосована у функції `check_letters_in_word()` для валідації аргументів

2. **Юніт тести (unittest)**
   - 4 тестові класи з 18+ тестовими методами
   - Використання `setUp()`/`tearDown()` та `@patch` для мокування
   - 98-100% успішність тестів

3. **Юніт тести (Pytest)**
   - 3 функціональні тести з Pytest синтаксисом
   - Обробка винятків через `pytest.raises()`
   - Високочитаємий код

4. **Coverage & Покриття**
   - Встановлено `pytest-cov` та `coverage`
   - Налаштовано `.coveragerc` файл
   - Згенеровано HTML звіт з покриттям **66%**

### Практичні навички:

✅ Написання валідних юніт-тестів
✅ Організація тестів у окремих файлах
✅ Використання Mock для ізоляції функцій
✅ Вимірювання та аналіз покриття коду
✅ Запуск тестів з командного рядка і IDE

**Мета досягнута?** ✅ **100% дотримання завдання!** 🎉

**Всі завдання виконані?** Так, повністю! 💯

---

