---
date: 2016-01-19T00:00:00+03:00
draft: false
slug: postgres-generate-most-large-followers
tags:
    - postgres
title: 'Postgres: generate the most large followers intersections'
---

Задача: получить список пользователей твиттера, с которыми у одного из них есть общие фоловеры, и отсортировать по их количеству. Лучше всего продемонстрировать на примере. Создадим таблицу с идентификаторами пользователей и фоловеров:

```sql
CREATE TEMP TABLE f(uid INT, fid INT);
INSERT INTO f(uid, fid) VALUES
    (1, 10), (1, 11), (1, 12), (1, 13), (1, 14),
    (2, 10), (2, 11), (2, 12),
    (3, 13), (3, 14),
    (4, 13), (4, 14), (4, 10), (4, 11)
;
```

А вот собственно и запрос, интересуют пересечения с пользователем 2, самые большие сверху.

```sql
SELECT uid, array_agg(fid) followers_intersect
FROM f GROUP BY uid
ORDER BY cardinality(
    (
        SELECT array_agg(a1) FROM unnest(array_agg(fid)) a1
        INNER JOIN unnest(
            (SELECT array_agg(fid) FROM f WHERE uid=2)::INT[]
        ) a2
        ON a1=a2
    )::INT[]
) DESC NULLS LAST;
```

Результат, как и ожидалось:

```
1. "{10,11,12,13,14}"
2. "{10,11,12}"
4. "{13,14,10,11}"
3. "{13,14}"
```