---
layout: post
title: Удобные команды и алиасы для GIT
tags: [memo]
categories: [git,development]
---

Несколько удобных алиасов для git, чтобы не потерять, положу это здесь.
Я использую оболочку [fish](https://fishshell.com/), поэтому возможно какие-то команды потребуется адаптировать для вашей.
Шикарный набор советов [здесь](https://github.com/git-tips/tips).


Лог в виде дерева:

{% highlight bash %}
function gl
    git log --oneline --decorate --graph --all
end
{% endhighlight %}

Интерактивный rebase с указанного в аргументах коммита:

{% highlight bash %}
function gri
    git rebase -i $argv
end
{% endhighlight %}

Перезапись локальной ветки свежей веткой с origin:

{% highlight bash %}
function grho
    git fetch origin (gb)
    git reset --hard FETCH_HEAD
end
{% endhighlight %}

Имя текущей ветки:

{% highlight bash %}
function gb
    git branch ^/dev/null | grep \* | sed 's/* //'
end
{% endhighlight %}

Интерактивный rebase с указанного в аргументах коммита:

{% highlight bash %}
function gri
    git rebase -i $argv
end
{% endhighlight %}

Первый коммит фичи:

{% highlight bash %}
function gfs
    git log devel..(gb) --oneline | tail -n 1 | awk '{print $1}'
end
{% endhighlight %}

Интерактивный rebase с первого коммита фичи:

{% highlight bash %}
function grif
    gri (gfs)~
end
{% endhighlight %}