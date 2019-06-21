---
layout: post
title: Intellij IDEA - сброс триального периода (MacOS)
tags: [apps,hack]
categories: [applications]
---

Аж в 2016 году я тут писал о том, [как удалить Idea полностью](https://dstarod.github.io/remove-idea-completely/), чтобы начать все с начала.
Но времена меняются, и что работало раньше - не работает сейчас. Вот обновленный алгоритм. Обратите внимание на версию Idea.

Удаляем файлы и каталоги:

{% highlight bash %}
cd ~/Library/Preferences
ls | grep 'jetbrains' | xargs rm
rm IntelliJIdea2019.1/eval/*.key
rm IntelliJIdea2019.1/options/other.xml
{% endhighlight %}

Теперь надо удалить записи из одного файла plist, который хранится в закодированном виде. Придется его для начала сконвертировать в текст а потом обратно:

{% highlight bash %}
cd ~/Library/Preferences
plutil -convert xml1 com.apple.java.util.prefs.plist
sed -i'' -e '/evlsprt/d' com.apple.java.util.prefs.plist
plutil -convert binary1 com.apple.java.util.prefs.plist
{% endhighlight %}

А теперь перезагрузите компьютер и можно работать еще месяц спокойно.