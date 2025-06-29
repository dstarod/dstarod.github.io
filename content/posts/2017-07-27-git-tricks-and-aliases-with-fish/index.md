---
title: Удобные команды и алиасы для GIT
tags:
- memo
date: 2017-07-27 00:00:00+03:00
slug: git-tricks-and-aliases-with-fish
draft: false
---

Несколько удобных алиасов для git, чтобы не потерять, положу это здесь.
Я использую оболочку [fish](https://fishshell.com/), поэтому возможно какие-то команды потребуется адаптировать для вашей.
Шикарный набор советов [здесь](https://github.com/git-tips/tips).


Лог в виде дерева:

```bash
function gl
    git log --oneline --decorate --graph --all
end
```

Интерактивный rebase с указанного в аргументах коммита:

```bash
function gri
    git rebase -i $argv
end
```

Перезапись локальной ветки свежей веткой с origin:

```bash
function grho
    git fetch origin (gb)
    git reset --hard FETCH_HEAD
end
```

Имя текущей ветки:

```bash
function gb
    git branch ^/dev/null | grep \* | sed 's/* //'
end
```

Интерактивный rebase с указанного в аргументах коммита:

```bash
function gri
    git rebase -i $argv
end
```

Первый коммит фичи:

```bash
function gfs
    git log devel..(gb) --oneline | tail -n 1 | awk '{print $1}'
end
```

Интерактивный rebase с первого коммита фичи:

```bash
function grif
    gri (gfs)~
end
```