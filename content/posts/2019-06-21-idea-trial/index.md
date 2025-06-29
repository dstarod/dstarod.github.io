---
title: Intellij IDEA - сброс триального периода
tags:
- apps
- hack
slug: idea-trial
date: 2019-06-21 00:00:00+03:00
draft: false
---

Аж в 2016 году я тут писал о том, [как удалить Idea полностью](https://dstarod.github.io/remove-idea-completely/), чтобы начать все с начала.
Но времена меняются, и что работало раньше - не работает сейчас. Вот обновленный алгоритм для MacOS. Обратите внимание на версию Idea.

Удаляем файлы и каталоги:

```bash
cd ~/Library/Preferences
ls | grep 'jetbrains' | xargs rm
rm IntelliJIdea2019.1/eval/*.key
rm IntelliJIdea2019.1/options/other.xml
```

Теперь надо удалить записи из одного файла plist, который хранится в закодированном виде. Придется его для начала сконвертировать в текст а потом обратно:

```bash
cd ~/Library/Preferences
plutil -convert xml1 com.apple.java.util.prefs.plist
sed -i'' -e '/evlsprt/d' com.apple.java.util.prefs.plist
plutil -convert binary1 com.apple.java.util.prefs.plist
```

А теперь перезагрузите компьютер и можно работать еще месяц спокойно.

Для Linux всё проще, достаточно вот этого (обратите внимание на каталог .IntelliJIdea2019.2, с версиями может быть другим):

```bash
cd
rm .IntelliJIdea2019.2/config/options/other.xml
rm .IntelliJIdea2019.2/config/eval/*
rm -rf .java/.userPrefs
```
