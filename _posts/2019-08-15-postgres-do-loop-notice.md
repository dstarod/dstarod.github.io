---
layout: post
title: Использование цикла в PostgreSQL с выводом в консоль
tags: [postgresql]
categories: [development]
---

Небольшая заметка на полях на будущее. Иногда в PostgreSQL нужно выполнить ряд повторяющихся действий с выводом результатов в лог.
Одним примером покажу, как это сделать легко и просто.

{% highlight sql %}

-- Делаем вывод сообщений в лог видимым
SET client_min_messages TO notice;

CREATE TEMP TABLE tmp(id SERIAL, name TEXT);

-- Анонимный блок DO
DO $$
BEGIN
    -- Цикл LOOP
    LOOP
        INSERT INTO tmp(name) VALUES(gen_random_uuid()::TEXT);
        -- Условный оператор IF
        IF (SELECT MAX(id) FROM tmp) > 100 THEN
            RAISE NOTICE 'max id == %', (SELECT COUNT(*) FROM tmp);
            EXIT;
        END IF;
    END LOOP;
END $$;

{% endhighlight %}
