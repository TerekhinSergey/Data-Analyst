# Исследование данных об инвестиции венчурных фондов в компании-стартапы с помощью SQL

В данном проекте работа идёт с базой данных, которая хранит информацию о венчурных фондах и инвестициях в компании-стартапы. Эта база данных основана на датасете Startup Investments, опубликованном на популярной платформе для соревнований по исследованию данных Kaggle. ER-диаграмма приложена.
Всего приведено 8 задач и решений к ним.

1.
Выгрузите таблицу с десятью самыми активными инвестирующими странами. Активность страны определите по среднему количеству компаний, в которые инвестируют фонды этой страны.
Для каждой страны посчитайте минимальное, максимальное и среднее число компаний, в которые инвестировали фонды, основанные с 2010 по 2012 год включительно.
Исключите из таблицы страны с фондами, у которых минимальное число компаний, получивших инвестиции, равно нулю. Отсортируйте таблицу по среднему количеству компаний от большего к меньшему.

```sql
SELECT country_code,
       MIN(invested_companies),
       MAX(invested_companies),
       AVG(invested_companies)
FROM fund
WHERE EXTRACT (YEAR FROM CAST(founded_at AS date)) BETWEEN 2010 AND 2012
GROUP BY country_code
HAVING MIN(invested_companies) <> 0 
ORDER BY AVG(invested_companies) DESC
LIMIT 10
```

-------------------------------------------------------------------------


2. 
Отобразите имя и фамилию всех сотрудников стартапов. Добавьте поле с названием учебного заведения, которое окончил сотрудник, если эта информация известна.

```sql
SELECT p.first_name,
       p.last_name,
       e.instituition
FROM people AS p
LEFT JOIN education AS e ON p.id = e.person_id
```

-------------------------------------------------------------------------

3.
Для каждой компании найдите количество учебных заведений, которые окончили её сотрудники. Выведите название компании и число уникальных названий учебных заведений. Составьте топ-5 компаний по количеству университетов.

```sql
SELECT c.name,
       COUNT(DISTINCT e.instituition)
FROM company AS c
JOIN people AS p ON c.id = p.company_id
JOIN education AS e ON p.id = e.person_id
GROUP BY c.name
ORDER BY COUNT(DISTINCT e.instituition) DESC
LIMIT 5
```

-------------------------------------------------------------------------

4.
Составьте таблицу из полей:
name_of_fund — название фонда;
name_of_company — название компании;
amount — сумма инвестиций, которую привлекла компания в раунде.
В таблицу войдут данные о компаниях, в истории которых было больше шести важных этапов, а раунды финансирования проходили с 2012 по 2013 год включительно.

```sql
SELECT f.name AS name_of_fund,
       c.name AS name_of_company,
       fr.raised_amount AS amount
FROM company AS c
LEFT JOIN investment AS i ON c.id = i.company_id
LEFT JOIN fund AS f ON f.id = i.fund_id
LEFT JOIN funding_round AS fr ON fr.id = i.funding_round_id
WHERE c.milestones > 6
  AND EXTRACT (YEAR FROM funded_at) BETWEEN 2012 AND 2013
```

-------------------------------------------------------------------------

5.
Выгрузите таблицу, в которой будут такие поля:
название компании-покупателя;
сумма сделки;
название компании, которую купили;
сумма инвестиций, вложенных в купленную компанию;
доля, которая отображает, во сколько раз сумма покупки превысила сумму вложенных в компанию инвестиций, округлённая до ближайшего целого числа.
Не учитывайте те сделки, в которых сумма покупки равна нулю. Если сумма инвестиций в компанию равна нулю, исключите такую компанию из таблицы.
Отсортируйте таблицу по сумме сделки от большей к меньшей, а затем по названию купленной компании в алфавитном порядке. Ограничьте таблицу первыми десятью записями.

```sql
SELECT c.name AS acquiring_company,
       a.price_amount,
       c1.name AS acquired_company,
       c1.funding_total,
       ROUND(a.price_amount/c1.funding_total)
FROM company AS c
RIGHT JOIN acquisition AS a ON c.id = a.acquiring_company_id
LEFT JOIN company AS c1 ON c1.id = a.acquired_company_id      
WHERE a.price_amount <> 0
  AND c1.funding_total <> 0
ORDER BY a.price_amount DESC, c1.name
LIMIT 10
```

-------------------------------------------------------------------------

6.
Выгрузите таблицу, в которую войдут названия компаний из категории social, получившие финансирование с 2010 по 2013 год. Выведите также номер месяца, в котором проходил раунд финансирования.

```sql
SELECT c.name,
       EXTRACT (MONTH FROM fr.funded_at)
FROM company AS c
LEFT JOIN funding_round AS fr ON c.id = fr.company_id
WHERE c.category_code = 'social'
  AND EXTRACT (YEAR FROM fr.funded_at) BETWEEN 2010 AND 2013  
```

-------------------------------------------------------------------------

7.
Отберите данные по месяцам с 2010 по 2013 год, когда проходили инвестиционные раунды. Сгруппируйте данные по номеру месяца и получите таблицу, в которой будут поля:
номер месяца, в котором проходили раунды;
количество уникальных названий фондов из США, которые инвестировали в этом месяце;
количество компаний, купленных за этот месяц;
общая сумма сделок по покупкам в этом месяце.

```sql
WITH 
a AS (SELECT EXTRACT (MONTH FROM fr.funded_at) AS month,
             COUNT(DISTINCT name) AS number_funds
      FROM funding_round AS fr
      LEFT JOIN investment AS i ON fr.id = i.funding_round_id
      LEFT JOIN fund AS f ON i.fund_id = f.id
      WHERE EXTRACT (YEAR FROM fr.funded_at) BETWEEN 2010 AND 2013
        AND f.country_code = 'USA'
      GROUP BY month),
b AS (SELECT EXTRACT (MONTH FROM a.acquired_at) AS month,
             COUNT(a.acquired_company_id) AS number_companies,
             SUM(a.price_amount) AS sum_price_amount
      FROM acquisition AS a
      WHERE EXTRACT (YEAR FROM a.acquired_at) BETWEEN 2010 AND 2013
      GROUP BY month)

SELECT a.month,
       a.number_funds,
       b.number_companies,
       b.sum_price_amount
FROM a
LEFT JOIN b ON a.month = b.month
```

-------------------------------------------------------------------------

8.
Составьте сводную таблицу и выведите среднюю сумму инвестиций для стран, в которых есть стартапы, зарегистрированные в 2011, 2012 и 2013 годах. Данные за каждый год должны быть в отдельном поле. Отсортируйте таблицу по среднему значению инвестиций за 2011 год от большего к меньшему.


```sql
WITH
year2011 AS (SELECT c.country_code AS country,
             AVG(c.funding_total) AS average_funding_2011
             FROM company AS c
             WHERE EXTRACT (YEAR FROM c.founded_at) = 2011
             GROUP BY c.country_code),

year2012 AS (SELECT c.country_code AS country,
             AVG(c.funding_total) AS average_funding_2012
             FROM company AS c
             WHERE EXTRACT (YEAR FROM c.founded_at) = 2012
             GROUP BY c.country_code),

year2013 AS (SELECT c.country_code AS country,
             AVG(c.funding_total) AS average_funding_2013
             FROM company AS c
             WHERE EXTRACT (YEAR FROM c.founded_at) = 2013
             GROUP BY c.country_code)

SELECT year2011.country,
       year2011.average_funding_2011,
       year2012.average_funding_2012,
       year2013.average_funding_2013
FROM year2011
INNER JOIN year2012 ON year2011.country = year2012.country
INNER JOIN year2013 ON year2012.country = year2013.country
ORDER BY average_funding_2011 DESC
```
