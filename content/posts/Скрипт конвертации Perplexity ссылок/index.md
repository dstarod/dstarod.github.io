---
date: 2025-07-01T20:45:42+03:00
description: Конвертация ссылок из исследования Perplexity в ссылки формата Markdown
draft: false
slug: perplexity-copy-links-markdown-convert
tags:
    - python
title: Скрипт конвертации Perplexity ссылок
---

К сожалению, когда копируешь (кнопкой "Копировать") исследования Perplexity он генерирует документы, которые не совсем Markdown - ссылки на источники он помещает в конце документа, а в тексте остаются только некликабельные указатели типа `[10]`. Это неудобно, если хочешь поместить исследование например в Obsidian.
Нагенерил скрипт, который это исправляет. Не сильно фонтан, но может кому сэкономит немного времени.

```python
import logging
import re
import sys

logging.basicConfig(
    filename="perplexity.log",
    level=logging.INFO,
    format="%(asctime)s - %(levelname)s - %(message)s",
)


def process_file(filepath):
    """
    Обрабатывает файл, заменяя ссылки на источники в тексте ссылками HTML.

    Args:
        filepath (str): Путь к файлу, содержащему текст и источники.

    Returns:
        str: Обработанный текст или None, если файл не найден или формат неправильный.
    """

    try:
        with open(
            filepath, "r", encoding="utf-8"
        ) as f:  # Указываем кодировку для поддержки русских символов
            content = f.read()

        # Разделяем текст и источники
        parts = content.split(
            "\n\nИсточники\n", 1
        )  # Разделяем только один раз, чтобы не сломать текст
        if len(parts) != 2:
            print("Ошибка: Файл не содержит раздел 'Источники'.")
            return None

        text = parts[0]
        sources_section = parts[1]

        # Извлекаем источники и создаем словарь
        sources = {}
        for line in sources_section.splitlines():
            if (
                line.strip() and "[" in line and "]" in line
            ):  # Проверяем, что строка не пустая и содержит квадратные скобки
                match = re.match(r"\[(\d+)\]\s+(.*)\s+(http.*)", line)
                if match:
                    key = match.group(1)
                    name = match.group(2).strip()  # Удаляем лишние пробелы в названии
                    link = match.group(3)  # Добавляем префикс https://
                    sources[key] = (name, link)

        # Заменяем ссылки в тексте
        def replace_link(match):
            key = match.group(1)
            if key in sources:
                _, link = sources[key]
                return f"{key}({link})"
            else:
                return match.group(
                    0
                )  # Возвращаем исходный текст, если ссылка не найдена

        processed_text = re.sub(r"\[(\d+)\]", replace_link, text)

        return processed_text

    except FileNotFoundError:
        print(f"Ошибка: Файл не найден по пути {filepath}")
        return None
    except Exception as e:
        print(f"Произошла ошибка при обработке файла: {e}")
        return None


if __name__ == "__main__":
    if len(sys.argv) > 1:
        filepath = sys.argv[1]  # Получаем путь к файлу из аргументов командной строки
    else:
        print("Использование: python perplexity.py <путь_к_файлу>")
        sys.exit(1)

    processed_text = process_file(filepath)

    if processed_text:
        # Сохраняем результат в новый файл
        with open("output.md", "w", encoding="utf-8") as f:
            f.write(processed_text)

```