---
date: 2025-08-12T21:10:47+03:00
description: Как в консоли изменить теги для mp3 массово скриптом
draft: false
slug: console-mp3-tags
tags:
    - utils
    - cli
title: Скрипт массового изменения ID3 для mp3
---

Задача: поменять в каталоге теги всем mp3-файлам, основываясь на именах файлов и папок, используя консольные утилиты.

Окружение: MacOS (тут без разницы, линукс идентично), [fishshell](https://fishshell.com), [Homebrew](https://brew.sh), [mp3info](https://formulae.brew.sh/formula/mp3info), [eyeD3](https://eyed3.readthedocs.io/en/v0.9.8/). Fish у меня основная оболочка, для других синтаксис чуть изменится, но не суть.

Итак. У меня каталог, который называется как дата в формате YYYY-MM-DD, в нем mp3-файлы с именами в формате "НомерПробелИмятрека.mp3". Нужно будет получить имя папки для даты, кусочек имени файла до пробела для номера трека, все остальное кроме расширения - имя трека.
Создаем функцию (конечно в файлике `~/.config/fish/config.fish`), заходим в нужную папку, запускаем. Текст функции:

```shell
function tags
for i in *.mp3
        set basename $(basename $i .mp3)
        set num $(echo $basename | cut -c1-2)
        set name $(echo $basename | cut -c4- | xargs)
        set reldate $(echo $PWD | awk -F/ '{print $NF}')
        mp3info -d $i
        eyeD3 --remove-all --remove-all-images --remove-all-objects $i
        eyeD3 \
                --track $num \
                --title $name \
                --artist 'Библейская церковь' \
                --recording-date $reldate \
                --album "Богослужение $reldate" \
                --add-image /path/to/cover.jpg:FRONT_COVER \
                --encoding utf8 --force-update --to-v2.4 \
                $i
end
end
```

Зачем тут `mp3info`? eyeD3 обладает неприятным багом с кодировками в ID3 первой версии. `mp3info` с этим справляется, поэтому чистим теги именно им.