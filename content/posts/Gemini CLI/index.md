---
title: Консольный ИИ-агент Gemini CLI
description: Установка, настройка, примеры использования gemini-cli
date: 2025-06-29 19:10:14+03:00
slug: gemini-cli-howto
draft: false
---

Gemini CLI - консольный ИИ-агент от Google. Работает с моделью `gemini-2.5-pro`. Позволяет делать бесплатно 1000 запросов в день и дает контекстное окно в миллион токенов. Умеет работать с файловой системой на компьютере, делать запросы в интернете и пользоваться поисковиком Google.

**Что немаловажно: работает в России.**

- [Обзорная статья от Google](https://developers.google.com/gemini-code-assist/docs/overview?hl=ru)
- [Репозиторий на GitHub](https://github.com/google-gemini/gemini-cli)
- [Документация по командам и настройкам](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/commands.md)

## Установка

```bash
npm install -g @google/gemini-cli
```

## Подготовка к запуску

Чтобы использовать агента бесплатно, есть несколько путей. Опишу использование варианта с аккаунтом в Google.

- создать проект в Google по [инструкции](https://developers.google.com/workspace/guides/create-project?hl=ru)
- в новом проекте из раздела *Project Info* скопировать *Project ID*.
- добавить его в переменную (`export GOOGLE_CLOUD_PROJECT="PROJ_ID"`)
- даем доступ [Gemini for Google Cloud](https://console.cloud.google.com/apis/library/cloudaicompanion.googleapis.com)

## Запускаем

```bash
gemini
```

На просьбу авторизоваться делаем это через Google под тем-же пользователем, чей проект создавали.
Готово. Можно пользоваться.

## Полезные команды

Сохранить чат, продолжить сохраненный ранее чат

```
/chat save <tag>
/chat resume <tag>
/chat list
```

Сжатие контекста (для экономии токенов)

```
/compress
```

Статистика использования

```
/stats
```

Управление установками агента (как это делали бы через `GEMINI.md`)

```
/memory
```

Выбор темы

```
/theme
```

Выход

```
/exit
```

Все команды можно увидеть если вести `/`. Документацию покажут командой `/docs`.
