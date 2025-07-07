---
date: 2025-07-07T14:54:57+03:00
description: Shell commands - плагин для Obsidian, который дает возможность бесконечно расширять функционал Obsidian через вызовы консольных утилит. Кратко о его возможностях с примерами.
draft: false
slug: obsidian-plugin-shell-commands
tags:
    - obsidian
title: Obsidian плагин Shell commands
---

Если вы хотели расширить возможности [Obsidian](https://obsidian.md), подходящих готовых плагинов не нашли, писать на Typescript морально не готовы, а писать консольные утилиты любите умеете и практикуете, то этот пост для вас.
Не так давно натолкнулся у какого-то блогера упоминание о плагине [Shell commans](https://publish.obsidian.md/shellcommands/Index), с помощью которого можно вызвать консольную команду. Взглянув на него повнимательнее я понял - это неограненный алмаз, недооцененный артефакт, и даже удивительно, почему так мало о нем говорят.

Мы имеем возможность:

- создавать именованную группу команд, которую вы вставили бы в терминал
- выбирать, в каком окружении запускать эти команды
- использовать готовые весьма полезные [переменные](https://publish.obsidian.md/shellcommands/Variables/Variables+-+general+principles)
- создавать собственные [переменные](https://publish.obsidian.md/shellcommands/Variables/Custom+variables)
- работать с данными заметок
- и прочее, и прочее

---

В качестве примера покажу, как я организовал вызов [скрипта для экспорта заметок Obsidian в движок Hugo](https://dstarod.github.io/posts/obsidian2hugo/).

1. Устанавливаем: Сторонние плагины -> Обзор -> Shell commands
2. Добавляем команду:

```bash
VAULT_PATH="/path/to/vault"
CACHE_PATH="$VAULT_PATH/_service/Cache"
HUGO_ROOT="/path/to/hugo/dstarod.github.io"
HUGO_POSTS="$HUGO_ROOT/content/posts"
rm -r $HUGO_POSTS/*
/path/to/obsidian2hugo --notes-dir "$VAULT_PATH" --attachments-dir "$CACHE_PATH" --hugo-posts-dir "$HUGO_POSTS" --log-level WARNING --filter-tag blog --remove-filter-tag --exclude-dirs _service
cd $HUGO_ROOT
/opt/homebrew/bin/hugo --minify --cleanDestinationDir --gc --quiet
git add .
git commit -m "Update"
git push origin master
```

Что мы тут делаем:

- добавляем переменные чтоб не хардкодить длинные пути
- удаляем все из каталога с постами
- вызываем скрипт `obsidian2hugo` с параметрами
- генерируем контент утилитой `hugo`
- добавляем новый вариант заполнения блога в git и отправляем в github

Теперь даем скрипту имя, например *hugo_generate* (где текст команды найдите шестеренку - там указать Alias), и можно вызывать командой **Shell commands: Execute: hugo_generate** (через сочетание клавиш `Cmd+P`, ну вы в курсе).

---

P.S. Я упоминал о том, что в **Shell commands** есть готовые переменные и можно добавлять свои. Они находятся в настройках плагина, раздел *Variables*. Среди готовых есть `{{vault_path}}` - полный путь к хранилищу. Добавим парочку своих (там-же подраздел *Custom variables*):

- `{{_cache_dir}}` со значением `{{vault_path}}/_service/Cache`
- `{{_hugo_root}}` со значением `/path/to/hugo/dstarod.github.io` (у вас свое конечно)
- `{{_hugo_posts}}` со значением `{{_hugo_root}}/content/posts`

Теперь скрипт будет выглядеть поаккуратнее

```shell
rm -r {{_hugo_posts}}/*
/path/to/obsidian2hugo --notes-dir {{vault_path}} --attachments-dir {{_cache_dir}} --hugo-posts-dir {{_hugo_posts}} --log-level WARNING --filter-tag blog --remove-filter-tag --exclude-dirs _service
cd {{_hugo_root}}
/opt/homebrew/bin/hugo --minify --cleanDestinationDir --gc --quiet
git add .
git commit -m "Update"
git push origin master
```

Конечно, это вершина айсберга, чтобы разжечь аппетит.