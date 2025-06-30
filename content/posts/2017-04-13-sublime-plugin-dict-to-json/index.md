---
date: 2017-04-13T00:00:00+03:00
draft: false
slug: sublime-plugin-dict-to-json
tags:
    - sublime
title: Плагин для Sublime Text своими руками
---

Не секрет, что одним из самых популярных редакторов кода у разработчиков на сегодняшний день
является [Sublime Text](https://www.sublimetext.com/). Кроме своих индивидуальных киллер-фич,
которые вскоре стали копировать разработчики других продуктов той-же ниши
(например, мультиредактирование), он хорош своей расширяемостью плагинами, коих великое множество на любой вкус. Думаю, не последней причиной такого их количества является простота создания. Для примера возьмем и напишем полезный инструмент, который будет брать как попало отформатированный Python-словарь и выдавать JSON стройными рядами.

Создаем новый плагин по шаблону: "Tools->Developer->New Plugin" и сохраняем в папку, которую предложат, например как my_plugin.py

```python
import re
import json

import sublime
import sublime_plugin

class DsonCommand(sublime_plugin.TextCommand):
    """ Convert python dict to json """
    def run(self, edit):
        # For every selected region
        for region in self.view.sel():
            # Extract text from selection
            line = self.view.line(region)
            content = self.view.substr(line)
            # Restore python dict from string
            python_dict = eval(content)
            # Prepare pretty-formatted JSON
            json_doc = json.dumps(
                python_dict,
                indent=4, sort_keys=True
            )
            # Replace selected region with prepared JSON
            self.view.replace(
                edit=edit, r=region, text=json_doc
            )
```

Теперь у нас есть команда `dson`, можно ее вызвать из консоли (Ctrl+~) (выделив текст со словарем), набрав там

```python
view.run_command('dson')
```

Эту команду имеет смысл повесить на горячую клавишу, если она нужна часто (Preferences->Key Bindings), например так:

```
{ "keys": ["ctrl+shift+d"], "command": "dson" }
```