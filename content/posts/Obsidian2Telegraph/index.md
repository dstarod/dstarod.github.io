---
date: 2025-07-05T13:21:43+03:00
description: Конвертация заметок Obsidian в посты для Telegraph с использованием плагина Copy document as HTML
draft: false
slug: obsidian-to-telegraph
title: Превращаем заметки Obsidian в посты Telegraph
---

Чтобы писать посты в [Obsidian](https://obsidian.md) и постить в Telegraph прям с картинками, таблицами, Dataview и прочими плюшками, достаточно копировать заметку в HTML с помощью плагина [Copy Document as HTML](https://github.com/mvdkwast/obsidian-copy-as-html)

Ставим плагин и настраиваем как хочется:

- убираем front-matter (Remove properties)
- вшиваем картинки (Embed external images)
- вики-ссылки делаем просто текстом (Link handling: Don't link)
- убираем лишние добавки к заметке (Footnote handling: Remove everything)

Чтобы запустить вызываем команду (`Cmd-P`): **Copy document as HTML: Copy selection or document to clipboard**. Это отправит в буфер обмена или выделенный фрагмент (если выделено что-то) или весь документ.

Чтобы ускорить процесс ставим на горячую клавишу (например, `Cmd+Shift+C`).

Всё, теперь просто вставляем в Telegraph.