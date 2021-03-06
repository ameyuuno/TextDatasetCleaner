# Text Dataset Cleaner

Настраиваемый пайплайн для очистки текстовых датасетов от мусора (некорректный язык, бранная речь, HTML-теги и т.д.).
Список обработчиков для очистки можно комбинировать и настраивать на свой вкус с помощью конфигурационного файла в YAML формате.

## Запуск

Использовалось на Ubuntu 18.04 с Python 3.7.

Для запуска нужно установить этот пакет, чтобы получить cli-команду, с помощью которой можно творить чудеса:

```bash
tdc -c путь_к_вашему_конфигу.yml -i input_file.txt -o output_file.txt
```

## Стадии обработки

В данном инструменте предусмотрено 3 стадии обработки:

1. **PRE_PROCESSING** - запуск предварительной обработки, например: удаление дубликатов.
В ней могут запускаться только "файловые" обработчики.
2. **PROCESSING** - запуск основной обработки. В ней могут запускаться только "построчные" обработчики.
3. **POST_PROCESSING** - запуск постобработки, например: перемешивание строк (полезно, если ваша сеть может запомнить порядок
классов на выборке в момент обучения). В ней могут запускаться только "файловые" обработчики.

## Список обработчиков

Все обработчики (processors) делятся на 2 типа:

- **Файловые** – обрабатывают весь файл целиком.
- **Построчные** – обрабатывают каждую строку в файле последовательно.

У любого из них могут быть обязательные и опциональные параметры, которые задаются в конфигурационном файле.

### clean_html

Построчный обработчик для очистки HTML-тегов. По умолчанию включается только если в строке есть оба символа: `<` и `>`.

Параметр | Обязательный? | Тип данных | Значение по-умолчанию | Описание
---------|---------------|------------|-----------------------|----------
or_condition | Нет | `bool` | `False` | `True` - обрабатывать есть ли есть хотя бы один из символов: `<` и `>`. `False` - когда обязательно есть оба.

### clean_symbols

Построчный обработчик для очистки различных utf-8 и непечатных символов и замены их на ASCII-эквиваленты.

Обрабатывает следующие виды символов:

- исправляет двойные кавычки
- исправляет одинарные кавычки
- исправляет тире
- исправляет пробелы
- исправляет восклицательный знак
- исправляет вопросительный знак
- исправляет дублирующиеся тире
- исправляет пробелы перед точкой
- удаляет непечатные символы

### detect_language

Построчный обработчик для определения языка текста через библиотеку fastText, с использованием их предобученной или собственной модели.
Если строка имеет язык отличный от заданого, то она пропускается (выкидывается) и не будет участвовать в других обработчиках.

Параметр | Обязательный? | Тип данных | Значение по-умолчанию | Описание
---------|---------------|------------|-----------------------|----------
language_code | Да | `str` | - | Ожидаемый код языка (`ru`, `en` и т.д.). [Список поддерживаемых языков](https://fasttext.cc/docs/en/language-identification.html)
threshold | Нет | `float` | `0.9` | Пороговое значение, _ниже которого_ считается что язык определён некорректно.
model_path | Нет | `str` | `` | Путь на диске к модели (если не указано, то скачается официальная).
model_url | Нет | `str` | `` | URL кастомной модели (если указано, то скачается за место официальной).
delimiter | Нет | `str` | `` | Разделитель в тексте. Если у вас TSV и нужно определить язык только выбранного столбца.
delimited_position | Нет | `int` | `-1` | Позиция столбца после разделения строки с помощью `.split`. По дефолту - последний.

### filter_currency_symbols

Построчный обработчик для замены или удаления строк, которые имеют один из символов валюты.

Параметр | Обязательный? | Тип данных | Значение по-умолчанию | Описание
---------|---------------|------------|-----------------------|----------
mode | Нет | `str` | `remove_line` | Что делать со строкой: `remove_line` - удалить (выбросить), `replace` - заменить найденные совпадения.
replace_with | Нет | `str` | `[пробел]` | На что заменять, если выбран режим `replace`.

### filter_email

Построчный обработчик для замены или удаления строк, содержащих email-адрес.

Параметр | Обязательный? | Тип данных | Значение по-умолчанию | Описание
---------|---------------|------------|-----------------------|----------
mode | Нет | `str` | `remove_line` | Что делать со строкой: `remove_line` - удалить (выбросить), `replace` - заменить найденные совпадения.
replace_with | Нет | `str` | `[пробел]` | На что заменять, если выбран режим `replace`.

### filter_emoji

Построчный обработчик для замены или удаления строк, содержащих emoji.

Параметр | Обязательный? | Тип данных | Значение по-умолчанию | Описание
---------|---------------|------------|-----------------------|----------
mode | Нет | `str` | `remove_line` | Что делать со строкой: `remove_line` - удалить (выбросить), `replace` - заменить найденные совпадения.
replace_with | Нет | `str` | `[пробел]` | На что заменять, если выбран режим `replace`.

### filter_hashtags

Построчный обработчик для замены или удаления строк, содержащих `#хэштеги`. 

Параметр | Обязательный? | Тип данных | Значение по-умолчанию | Описание
---------|---------------|------------|-----------------------|----------
mode | Нет | `str` | `remove_line` | Что делать со строкой: `remove_line` - удалить (выбросить), `replace` - заменить найденные совпадения.
replace_with | Нет | `str` | `[пробел]` | На что заменять, если выбран режим `replace`.

### filter_numbers

Построчный обработчик для замены или удаления строк, содержащих числа вне контекста слов.

Параметр | Обязательный? | Тип данных | Значение по-умолчанию | Описание
---------|---------------|------------|-----------------------|----------
mode | Нет | `str` | `remove_line` | Что делать со строкой: `remove_line` - удалить (выбросить), `replace` - заменить найденные совпадения.
replace_with | Нет | `str` | `[пробел]` | На что заменять, если выбран режим `replace`.

### filter_phone_number

Построчный обработчик для замены или удаления строк, содержащих телефонные номера.

Параметр | Обязательный? | Тип данных | Значение по-умолчанию | Описание
---------|---------------|------------|-----------------------|----------
mode | Нет | `str` | `remove_line` | Что делать со строкой: `remove_line` - удалить (выбросить), `replace` - заменить найденные совпадения.
replace_with | Нет | `str` | `[пробел]` | На что заменять, если выбран режим `replace`.

### filter_stop_words

Построчный обработчик для замены или удаления строк, содержащих стоп-слова (список языков и слов см. ниже).

Параметр | Обязательный? | Тип данных | Значение по-умолчанию | Описание
---------|---------------|------------|-----------------------|----------
language_code | Да | `str` | - | Ожидаемый код языка (`ru`, `en` и т.д.). [Список поддерживаемых языков](https://github.com/6/stopwords-json/tree/master/dist).
mode | Нет | `str` | `remove_line` | Что делать со строкой: `remove_line` - удалить (выбросить), `replace` - заменить найденные совпадения.
replace_with | Нет | `str` | `[пробел]` | На что заменять, если выбран режим `replace`.

### filter_url

Построчный обработчик для замены или удаления строк, содержащих URL-адреса.

Параметр | Обязательный? | Тип данных | Значение по-умолчанию | Описание
---------|---------------|------------|-----------------------|----------
mode | Нет | `str` | `remove_line` | Что делать со строкой: `remove_line` - удалить (выбросить), `replace` - заменить найденные совпадения.
replace_with | Нет | `str` | `[пробел]` | На что заменять, если выбран режим `replace`.

### filter_user_handle

Построчный обработчик для замены или удаления строк, содержащих `@юзернеймы`.

Параметр | Обязательный? | Тип данных | Значение по-умолчанию | Описание
---------|---------------|------------|-----------------------|----------
mode | Нет | `str` | `remove_line` | Что делать со строкой: `remove_line` - удалить (выбросить), `replace` - заменить найденные совпадения.
replace_with | Нет | `str` | `[пробел]` | На что заменять, если выбран режим `replace`.

### line_convert_case

Построчный обработчик для смены регистра.

Параметр | Обязательный? | Тип данных | Значение по-умолчанию | Описание
---------|---------------|------------|-----------------------|----------
mode | Да | `str` | - | Варианты: `title` - делает первую букву во всех словах заглавной; `lower` - понижает регистр; `upper` - повышает регистр.

### line_strip

Построчный обработчик для удаления стартовых и конечных пробельных символов.

### normalize_hyphenated_words

Построчный обработчик для исправления слов в тексте, которые были разделены дефисом для переноса слов по слогам в конце строки.
Объединяет кусочки слов воедино, убирая дефис и пробелы.

### normalize_quotation_marks

Построчный обработчик для исправления одинарных и двойных кавычек, а также апострофов до их ASCII-эквивалентов.

### normalize_repeating_chars

Построчный обработчик для удаления повторяющихся пунктуационных символов.
Обрабатывает следующий список символов: `!"#$%&\'()*+,-/:;<=>?@[\\]^_``{|}~`, а также исправляет многоточие в строках
(превращает `..` в `...` и если есть 4 подряд повторяющихся точки, то заменяет их в многоточие).

### normalize_unicode

Построчный обработчик для исправления юникодных символов в тексте в их канонический вид.

Параметр | Обязательный? | Тип данных | Значение по-умолчанию | Описание
---------|---------------|------------|-----------------------|----------
form | Да | `str` | `NFKC` | Формы преобразования: `NFC`, `NFKC`, `NFD`, `NFKD`. [Подробнее о формах](https://docs.python.org/3/library/unicodedata.html#unicodedata.normalize).

### normalize_whitespace

Построчный обработчик для исправления:
смежных пробелов нулевой ширины на пустую строку;
перевода каретки в Windows-стиле (`\r\n`) и перевода каретки с вертикальной табуляцией (`\n\v`) на простой перевод каретки `\n`;
пробельных символов без переноса каретки на одинарный пробел;
и удаления стартовых/конечных пробелов в строке.

### remove_accents

Построчный обработчик для транслитерации юникодных символов в ASCII-эквиваленты.

### remove_profanity

Построчный обработчик для удаления строк с матерной речью на _английском языке_.

Параметр | Обязательный? | Тип данных | Значение по-умолчанию | Описание
---------|---------------|------------|-----------------------|----------
threshold | Нет | `float` | `0.9` | Пороговое значение, _выше которого_ считается что в строке есть бранная речь.

### shuffle

Файловый обработчик для перемешивания строк. Используется системная реалзация GNU `shuf`.

### unique

Файловый обработчик для удаления повторов в строках. Используются системные реализации BSD `sort` и `uniq`.

## TODO

В идеале хочется собрать всё в виде docker-контейнера, чтоб можно было сделать `docker pull ...`, выставить input и output директории через volume и задать какой-то yaml-подобный файл конфигурации для определения необходимых шагов препроцессинга. А при запуске контейнера чтобы стартовала очистка, которая в итоге брала все файлы из input-директории и собирала результаты в output.

К существующим скриптам препроцессинга хочется допилить:

- [ ] Сделать документацию на rtfd.
- [ ] Перевести всё на английский язык.
- [ ] Поддержка других входных/выходных форматов файлов (jsonl/стрим из hdfs).
- [ ] Провести бенчмарки для того чтоб понимать сколько вообще оно работает (можно сделать отдельный бенч-тул и запускать в GHA).
- [ ] Проверку и исправление орфографии через standalone-версию [LanguageTool](https://github.com/languagetool-org/languagetool)
- [ ] Исправление знаков пунктуации в строках
- [ ] Сделать анализ тональности и удалять негативные строки
- [ ] Попробовать прикрутить классификацию тематики текста
- [ ] Удаление знаков препинания
- [x] Понижение регистра (если вдруг кому-то это нужно)
- [ ] Сделать удаление нечётких дубликатов через шинглы (лучше через какое-то готовое решение)
- [x] Удаление всех строк, которые содержат стоп-слова из заданного словаря
- [ ] Проверка метрик читаемости текста (через [pattern.metrics](https://www.clips.uantwerpen.be/pages/pattern-metrics) или [ruTS](https://github.com/SergeyShk/ruTS))
- [ ] Оценка натуральности/логичности текста через [BERT (режим №2)](https://colab.research.google.com/github/blade1780/bert/blob/master/BERT.ipynb)
- [ ] Использование готовых API для проверки орфографии через Google/Bing
- [ ] Прикрутить использование RAMDisk для ускорения обработки
- [ ] Загрузка процессоров из других директорий (чтобы не форкать реп, если нужно прикрутить свой)
- [x] Удаление emoji

## Contribute?

Хочется что-то предложить или поделиться своим опытом? - Welcome in issues! Мы будем рады обсудить ваши идеи и опыт ;-)

Вы тоже понимаете, что сейчас нет "серебряной пули" для решения такого рода задачи и абсолютно каждый городит что-то своё? - Присылайте PR, давайте сделаем вместе что-то прекрасное и полезное для всего DS-сообщества!
