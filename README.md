# Лабораторная работа №4.1. Чтение данных из файла и их обработка. Запись результатов обработки в файл
## Цель работы: 
Научиться работать с файлами, выполнять их чтение, запись и манипуляцию, а также освоить методы сериализации и десериализации данных с использованием популярных форматов хранения, таких как CSV и JSON.
## Задачи:
1. Изучить структуру файловой системы, свойства файлов, виды путей к файлам (абсолютный и относительный).
2. Освоить операции с файлами в Python: открытие, чтение, запись, закрытие.
3. Разобраться с файловыми объектами и их основными свойствами и методами.
4. Изучить методы работы с файлами: чтение файла целиком, построчное чтение, запись строк в файл.
5. Познакомиться с концепцией сериализации и десериализации данных.
6. Изучить популярные форматы сериализации: CSV и JSON.
7. Научиться использовать модуль pickle для сериализации данных в Python.
8. Решить практические задачи по обработке файлов и сериализации данных.

## Вариант 7
На сайте Всемирного банка (WB) в разделе Data доступна экономическая статистика о валовом внутреннем продукте (ВВП) на душу населения в долларах США 1.
Используя подготовленный CSV-файл (GDP per capita (current US$)): CSV-файл Всемирного банка), реализуйте:
- загрузку данных;
- поиск государства по названию, а также государства с максимальным, минимальным ВВП на душу населения;
- сохранение данных в новый CSV-файл с фильтром по определенному условию (например, топ-10 государств по объему ВВП на душу населения).
```
# Выполнила: Паньшина З.А.
# Группа: АБП-231

import csv

class NoSuchCountryError(Exception):
    def __init__(self, message):
        super().__init__(message)

class IllegalArgumentError(ValueError):
    pass

def load_data(filename):
    """Загрузить данные ВВП на душу населения из csv-файла 'filename'.

    Если значения для какого-либо государства не известно, строка должна
    быть пропущена и отсутствовать в результате.

    Параметры:
        - filename (str): имя файла.

    Результат:
        - list of dict: [
            {
              - "name": str: название государства;
              - "gdp": float: ВВП на душу населения.
            }
            ...
          ]

    Функция не обрабатывает исключения."""
    data = []
    with open(filename, 'r', encoding='utf-8') as f:
        reader = csv.DictReader(f)
        for row in reader:
            country_name = row['name']
            gdp_value = row['gdp']
            if gdp_value and gdp_value != '':
                try:
                    gdp = float(gdp_value)
                    data.append({
                        "name": country_name,
                        "gdp": gdp
                    })
                except ValueError:
                    continue
    return data

def search(data, criteria):
    """Выполнить поиск государства-значения в 'data' по критерию 'criteria'.

    Параметры:
        - data (list of dict): структура данных формата 'load_data()';
        - criteria (string): критерий поиска; допустимые значения:
            - "-max-": государство с максимальным ВВП на душу населения;
            - "-min-": государство с мнимальным ВВП на душу населения;
            - "Russian Federation": название государства.

    Результат:
        - dict: [
            {
              - "name": str: название государства;
              - "gdp": float: ВВП на душу населения.
            }
        или
        - NoSuchCountryError: если такой страны нет.
    """

    if criteria == "-max-":
        if not data:
            raise NoSuchCountryError("Нет данных для поиска")
        max_country = max(data, key=lambda x: x["gdp"])
        return max_country
    elif criteria == "-min-":
        if not data:
            raise NoSuchCountryError("Нет данных для поиска")
        min_country = min(data, key=lambda x: x["gdp"])
        return min_country
    else:
        # Поиск по названию страны
        for country in data:
            if country["name"] == criteria:
                return country
        raise NoSuchCountryError(f"Страна '{criteria}' не найдена")

def save_data(filename, data, criteria):
    """Сохранить данные 'data' в csv-файл 'filename' по критерию 'criteria'.

    Параметры:
        - filename (str): имя файла;
        - data (list of dict): структура данных формата 'load_data()';
        - criteria (string): критерий поиска; допустимые значения:
            - "top=X": первые X государств по ВВП на душу населения
                         (целое число > 0, по убыванию значения);
            - "tail=X": последние X государств по ВВП на душу населения
                          (целое число > 0, по возрастанию значения);
            - "greater=X": список государств с ВВП на душу населения, больше
                           чем X (вещ. число, по убыванию значения);
            - "less=X": список государств с ВВП на душу населения, меньше
                          чем X (вещ. число, по возрастанию значения).

    Исключения:
        - IllegalArgumentError: 'criteria' содержит недопустимое значение.
    """
    try:
        method, value = criteria.split("=")
        filtered_data = []

        if method == "top":
            x = int(value)
            if x <= 0:
                raise IllegalArgumentError("Значение X должно быть > 0")
            sorted_data = sorted(data, key=lambda item: item["gdp"], reverse=True)
            filtered_data = sorted_data[:x]
        elif method == "tail":
            x = int(value)
            if x <= 0:
                raise IllegalArgumentError("Значение X должно быть > 0")
            sorted_data = sorted(data, key=lambda item: item["gdp"])
            filtered_data = sorted_data[:x]
        elif method == "greater":
            threshold = float(value)
            filtered_data = [item for item in data if item["gdp"] > threshold]
            filtered_data.sort(key=lambda item: item["gdp"], reverse=True)
        elif method == "less":
            threshold = float(value)
            filtered_data = [item for item in data if item["gdp"] < threshold]
            filtered_data.sort(key=lambda item: item["gdp"])
        else:
            raise IllegalArgumentError("Неизвестный метод")

        with open(filename, 'w', encoding='utf-8', newline='') as f:
            writer = csv.DictWriter(f, fieldnames=["name", "gdp"])
            writer.writeheader()
            for country in filtered_data:
                writer.writerow({
                    "name": country["name"],
                    "gdp": country["gdp"]
                })

    except Exception as err:
        raise IllegalArgumentError(
            "Значение параметра 'criteria' может быть "
            "одним из:\n"
            '- "top=X": первые X государств по ВВП на душу населения'
            ' (целое число > 0, по убыванию значения);\n'
            '- "tail=X": последние X государств по ВВП на душу населения'
            ' (целое число > 0, по возрастанию значения);\n'
            '- "greater=X": список государств с ВВП на душу населения, больше'
            ' чем X (вещ. число, по убыванию значения);\n'
            '- "less=X": список государств с ВВП на душу населения, меньше'
            ' чем X (вещ. число, по возрастанию значения).')

try:
    filename = input("Введите имя файла: ")

    save_filename = input("Введите имя файла для сохранения: ")

    data = load_data(filename)

    print("Максимальный ВВП:", search(data, criteria="-max-"))
    print("Минимальный ВВП:", search(data, criteria="-min-"))
    print("Россия:", search(data, criteria="Russian Federation"))

    save_data(save_filename, data, criteria="top=5")
    print(f"Данные сохранены в файл: {save_filename}")

except FileNotFoundError:
    print("Ошибка: Файл не найден!")
except NoSuchCountryError as e:
    print(f"Ошибка поиска страны: {e}")
except IllegalArgumentError as e:
    print(f"Ошибка аргумента: {e}")
except Exception as e:
    print(f"Произошла непредвиденная ошибка: {e}")
```

<img width="796" height="151" alt="image" src="https://github.com/user-attachments/assets/45f29e62-6d5e-431c-80d4-9e16b676e84a" />

## Выводы:
В ходе выполнения данной лабораторной работы научились работать с файлами, выполнять их чтение, запись и манипуляцию, а также освоили методы сериализации и десериализации данных с использованием популярных форматов хранения, таких как CSV и JSON.
