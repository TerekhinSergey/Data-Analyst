# Исследование данных о контенте и пользователях сервиса вопросов и ответов о программировании Stack Overflow с помощью SQL

В данном проекте работа идёт с версией базы данных, которая хранит данные о постах на сайте Stack Overflow за 2008 год, но в таблицах есть информация и о более поздних оценках, которые эти посты получили. ER-диаграмма приложена.
Всего приведено 9 задач и решений к ним.

1.
Выгрузите данные активности пользователя, который опубликовал больше всего постов за всё время. Выведите данные за октябрь 2008 года в таком виде:  
— номер недели;  
— дата и время последнего поста, опубликованного на этой неделе.

```sql
WITH
a AS(SELECT EXTRACT(WEEK FROM p.creation_date) AS week, 
            LAST_VALUE(p.creation_date) OVER (PARTITION BY EXTRACT(WEEK FROM p.creation_date)) AS last_post_dt
       FROM stackoverflow.posts AS p
       JOIN stackoverflow.users AS u ON p.user_id=u.id
      WHERE u.id IN(SELECT u.id
                      FROM stackoverflow.posts AS p
                      JOIN stackoverflow.users AS u ON p.user_id=u.id
                  GROUP BY u.id
                  ORDER BY COUNT(p.id) DESC
                     LIMIT 1)
            AND p.creation_date BETWEEN '01-10-2008' AND '01-11-2008')
  SELECT week,
       last_post_dt
    FROM a
GROUP BY week, last_post_dt
ORDER BY week
```

-------------------------------------------------------------------------


2. 
На сколько процентов менялось количество постов ежемесячно с 1 сентября по 31 декабря 2008 года? Отобразите таблицу со следующими полями:  
— номер месяца;  
— количество постов за месяц;  
— процент, который показывает, насколько изменилось количество постов в текущем месяце по сравнению с предыдущим.  
Если постов стало меньше, значение процента должно быть отрицательным, если больше — положительным. Округлите значение процента до двух знаков после запятой.

```sql
WITH
a AS(SELECT EXTRACT(MONTH FROM p.creation_date) AS month,
            COUNT(p.id) AS amount_posts 
       FROM stackoverflow.posts p
      WHERE p.creation_date BETWEEN '01-09-2008' AND '31-12-2008'
   GROUP BY month)
  SELECT month,
         amount_posts,
         ROUND(amount_posts::numeric / LEAD(amount_posts) OVER (ORDER BY amount_posts) * 100 - 100, 2) AS previous_month_posts 
    FROM a
ORDER BY month
```

-------------------------------------------------------------------------

3.

Сколько в среднем дней в период с 1 по 7 декабря 2008 года включительно пользователи взаимодействовали с платформой? Для каждого пользователя отберите дни, в которые он или она опубликовали хотя бы один пост. Нужно получить одно целое число.

```sql
WITH 
a AS(SELECT u.id,
            p.creation_date::date
       FROM stackoverflow.posts AS p
       JOIN stackoverflow.users AS u ON p.user_id=u.id
      WHERE p.creation_date BETWEEN '01-12-2008' AND '07-12-2008'
   GROUP BY u.id, p.creation_date::date), 
b AS(SELECT id,
            COUNT(creation_date)
       FROM a
   GROUP BY id
   ORDER BY id)
SELECT ROUND(AVG(count))
FROM b
```

-------------------------------------------------------------------------

4.
Используя данные о постах, выведите несколько полей:  
— идентификатор пользователя, который написал пост;  
— дата создания поста;  
— количество просмотров у текущего поста;  
— сумму просмотров постов автора с накоплением.  
Данные в таблице должны быть отсортированы по возрастанию идентификаторов пользователей, а данные об одном и том же пользователе — по возрастанию даты создания поста.

```sql
  SELECT u.id,
         p.creation_date,
         p.views_count,
         SUM(p.views_count) OVER (PARTITION BY u.id ORDER BY p.creation_date)
    FROM stackoverflow.users AS u
    JOIN stackoverflow.posts AS p ON u.id=p.user_id
ORDER BY u.id, creation_date
```

-------------------------------------------------------------------------

5.
Выведите имена самых активных пользователей, которые в первый месяц после регистрации (включая день регистрации) дали больше 100 ответов. Вопросы, которые задавали пользователи, не учитывайте. Для каждого имени пользователя выведите количество уникальных значений user_id. Отсортируйте результат по полю с именами в лексикографическом порядке.


```sql
  SELECT u.display_name,
         COUNT(DISTINCT p.user_id)
    FROM stackoverflow.users AS u
    JOIN stackoverflow.posts AS p ON u.id=p.user_id
    JOIN stackoverflow.post_types AS pt ON p.post_type_id=pt.id
   WHERE pt.type = 'Answer' AND 
         DATE_TRUNC('day', p.creation_date) BETWEEN
         DATE_TRUNC('day', u.creation_date) AND DATE_TRUNC('day', u.creation_date) + INTERVAL '1 month' 
GROUP BY u.display_name
  HAVING COUNT(p.id) > 100
ORDER BY display_name
```

-------------------------------------------------------------------------

6.
Посчитайте ежедневный прирост новых пользователей в ноябре 2008 года. Сформируйте таблицу с полями:  
— номер дня;  
— число пользователей, зарегистрированных в этот день;  
— сумму пользователей с накоплением.

```sql
  SELECT EXTRACT(DAY FROM u.creation_date::date),
         COUNT(u.id),
         SUM(COUNT(u.id)) OVER(ORDER BY u.creation_date::date)
    FROM stackoverflow.users AS u
   WHERE u.creation_date::date BETWEEN '1-11-2008' AND '30-11-2008'
GROUP BY u.creation_date::date
```

-------------------------------------------------------------------------

7.
Напишите запрос, который выгрузит данные о пользователях из США (англ. United States). Разделите пользователей на три группы в зависимости от количества просмотров их профилей:  
— пользователям с числом просмотров больше либо равным 350 присвойте группу 1;  
— пользователям с числом просмотров меньше 350, но больше либо равно 100 — группу 2;  
— пользователям с числом просмотров меньше 100 — группу 3.  
Пользователи с нулевым количеством просмотров не должны войти в итоговую таблицу. Отобразите лидеров каждой группы — пользователей, которые набрали максимальное число просмотров в своей группе. Выведите поля с идентификатором пользователя, группой и количеством просмотров. Отсортируйте таблицу по убыванию просмотров, а затем по возрастанию значения идентификатора.

```sql
WITH
a AS(SELECT u.id,
            u.views,
            CASE WHEN u.views>=350 THEN 1
                 WHEN u.views<350 AND u.views>= 100 THEN 2
                 ELSE 3
            END AS group
        FROM stackoverflow.users AS u
       WHERE u.views!=0 AND u.location LIKE '%United States%'),
b AS (SELECT a.id,
             a.views,
             a.group,
             MAX(views) OVER(PARTITION BY a.group)
        FROM a 
    GROUP BY a.id, a.views, a.group)
  SELECT b.id, b.views, b.group
    FROM b 
   WHERE b.views = b.max
ORDER BY b.views DESC, b.id
```

-------------------------------------------------------------------------

8.
Сколько в среднем очков получает пост каждого пользователя?
Сформируйте таблицу из следующих полей:  
— заголовок поста;  
— идентификатор пользователя;  
— число очков поста;  
— среднее число очков пользователя за пост, округлённое до целого числа.  
Не учитывайте посты без заголовка, а также те, что набрали ноль очков.

```sql
  SELECT p.title,
         u.id,
         p.score,
         ROUND(AVG(p.score) OVER (PARTITION BY u.id)) AS user_avg
    FROM stackoverflow.posts AS p
    JOIN stackoverflow.users AS u ON p.user_id=u.id
   WHERE p.score!=0 AND p.title!=''
GROUP BY p.title, u.id, p.score
```

-------------------------------------------------------------------------

9.
Отберите 10 пользователей по количеству значков, полученных в период с 15 ноября по 15 декабря 2008 года включительно. Отобразите несколько полей:  
— идентификатор пользователя;  
— число значков;  
— место в рейтинге — чем больше значков, тем выше рейтинг.  
Пользователям, которые набрали одинаковое количество значков, присвойте одно и то же место в рейтинге. Отсортируйте записи по количеству значков по убыванию, а затем по возрастанию значения идентификатора пользователя.

```sql
   SELECT u.id,
          COUNT(b.id),
          DENSE_RANK() OVER (ORDER BY COUNT(b.id) DESC)
     FROM stackoverflow.users AS u
     JOIN stackoverflow.badges AS b ON b.user_id=u.id
    WHERE b.creation_date::date BETWEEN '15-11-2008' AND '15-12-2008'
 GROUP BY u.id
 ORDER BY COUNT(b.id) DESC, u.id
    LIMIT 10
```

