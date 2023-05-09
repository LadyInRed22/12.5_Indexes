# Домашнее задание к занятию 12.5. «Индексы» - `Виноградова Татьяна`

---

### Задание 1

Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц
``` SQL
SELECT sum(index_length) / sum(data_length) * 100 as 'index weight' 
FROM information_schema.TABLES
```

### Задание 2

Выполните explain analyze следующего запроса:
```sql
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
```
- перечислите узкие места;
- оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.
#### explain analyze:
```
-> Table scan on <temporary>  (cost=2.50..2.50 rows=0) (actual time=6366.825..6366.863 rows=391 loops=1)
    -> Temporary table with deduplication  (cost=0.00..0.00 rows=0) (actual time=6366.823..6366.823 rows=391 loops=1)
        -> Window aggregate with buffering: sum(payment.amount) OVER (PARTITION BY c.customer_id,f.title )   (actual time=2481.359..6142.669 rows=642000 loops=1)
            -> Sort: c.customer_id, f.title  (actual time=2481.329..2566.625 rows=642000 loops=1)
                -> Stream results  (cost=10194864.78 rows=15988150) (actual time=0.386..1865.039 rows=642000 loops=1)
                    -> Nested loop inner join  (cost=10194864.78 rows=15988150) (actual time=0.381..1553.754 rows=642000 loops=1)
                        -> Nested loop inner join  (cost=8592052.75 rows=15988150) (actual time=0.378..1395.870 rows=642000 loops=1)
                            -> Nested loop inner join  (cost=6989240.71 rows=15988150) (actual time=0.373..1207.961 rows=642000 loops=1)
                                -> Inner hash join (no condition)  (cost=1540174.80 rows=15400000) (actual time=0.363..54.527 rows=634000 loops=1)
                                    -> Filter: (cast(p.payment_date as date) = '2005-07-30')  (cost=1.61 rows=15400) (actual time=0.033..7.120 rows=634 loops=1)
                                        -> Table scan on p  (cost=1.61 rows=15400) (actual time=0.024..5.002 rows=16044 loops=1)
                                    -> Hash
                                        -> Covering index scan on f using idx_title  (cost=103.00 rows=1000) (actual time=0.033..0.238 rows=1000 loops=1)
                                -> Covering index lookup on r using rental_date (rental_date=p.payment_date)  (cost=0.25 rows=1) (actual time=0.001..0.002 rows=1 loops=634000)
                            -> Single-row index lookup on c using PRIMARY (customer_id=r.customer_id)  (cost=0.00 rows=1) (actual time=0.000..0.000 rows=1 loops=642000)
                        -> Single-row covering index lookup on i using PRIMARY (inventory_id=r.inventory_id)  (cost=0.00 rows=1) (actual time=0.000..0.000 rows=1 loops=642000)
```                        


``` SQL
1.
CREATE INDEX payment_date ON payment(payment_date) ;
2.
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id)
from customer c 
join rental r on r.customer_id = c.customer_id
join payment p on p.rental_id = r.rental_id
join inventory i on i.inventory_id  = r.inventory_id 
join film f on f.film_id = i.film_id 
where date(p.payment_date) = '2005-07-30'
```
#### explain analyze:
```
-> Table scan on <temporary>  (cost=2.50..2.50 rows=0) (actual time=116.108..116.154 rows=391 loops=1)
    -> Temporary table with deduplication  (cost=0.00..0.00 rows=0) (actual time=116.107..116.107 rows=391 loops=1)
        -> Window aggregate with buffering: sum(payment.amount) OVER (PARTITION BY c.customer_id )   (actual time=115.117..115.958 rows=634 loops=1)
            -> Sort: c.customer_id  (actual time=115.096..115.131 rows=634 loops=1)
                -> Stream results  (cost=24652.75 rows=16419) (actual time=64.674..114.911 rows=634 loops=1)
                    -> Nested loop inner join  (cost=24652.75 rows=16419) (actual time=64.662..114.632 rows=634 loops=1)
                        -> Nested loop inner join  (cost=18906.10 rows=16419) (actual time=0.053..62.018 rows=16044 loops=1)
                            -> Nested loop inner join  (cost=13159.45 rows=16419) (actual time=0.049..43.884 rows=16044 loops=1)
                                -> Nested loop inner join  (cost=7412.80 rows=16419) (actual time=0.045..25.298 rows=16044 loops=1)
                                    -> Covering index scan on r using rental_date  (cost=1666.15 rows=16419) (actual time=0.033..4.431 rows=16044 loops=1)
                                    -> Single-row index lookup on c using PRIMARY (customer_id=r.customer_id)  (cost=0.25 rows=1) (actual time=0.001..0.001 rows=1 loops=16044)
                                -> Single-row index lookup on i using PRIMARY (inventory_id=r.inventory_id)  (cost=0.25 rows=1) (actual time=0.001..0.001 rows=1 loops=16044)
                            -> Single-row covering index lookup on f using PRIMARY (film_id=i.film_id)  (cost=0.25 rows=1) (actual time=0.001..0.001 rows=1 loops=16044)
                        -> Filter: (cast(p.payment_date as date) = '2005-07-30')  (cost=0.25 rows=1) (actual time=0.003..0.003 rows=0 loops=16044)
                            -> Index lookup on p using fk_payment_rental (rental_id=r.rental_id)  (cost=0.25 rows=1) (actual time=0.002..0.003 rows=1 loops=16044)
```


### Задание 3*

Самостоятельно изучите, какие типы индексов используются в PostgreSQL. Перечислите те индексы, которые используются в PostgreSQL, а в MySQL — нет.

Помимо общих индексов с MySQL, в PostgreSQL используются следующие типы индексов:

* GiST -  может применяться с разными операторами, в зависимости от стратегии индексирования, например, существуют  классы операторов GiST для нескольких двумерных типов геометрических данных, что позволяет применять индексы в запросах с операторами <<   &<   &>   >>   <<|   &<|   |&>   |>>   @>   <@   ~=   &. Также GiST-индексы также могут оптимизировать поиск «ближайшего соседа».
* SP-GiST - позволяет организовывать на диске самые разные несбалансированные структуры данных, такие как деревья квадрантов, k-мерные и префиксные деревья. Индексы SP-GiST, как и GiST, поддерживают поиск ближайших соседей.
* GIN - GIN-индексы представляют собой «инвертированные индексы», в которых могут содержаться значения с несколькими ключами, например массивы. Инвертированный индекс содержит отдельный элемент для значения каждого компонента, и может эффективно работать в запросах, проверяющих присутствие определённых значений компонентов.Подобно GiST и SP-GiST, индексы GIN могут поддерживать различные определённые пользователем стратегии и в зависимости от них могут применяться с разными операторами.
* BRIN - хранят обобщённые сведения о значениях, находящихся в физически последовательно расположенных блоках таблицы. Поэтому такие индексы наиболее эффективны для столбцов, значения в которых хорошо коррелируют с физическим порядком столбцов таблицы. Подобно GiST, SP-GiST и GIN, индексы BRIN могут поддерживать определённые пользователем стратегии и в зависимости от них применяться с разными операторами.
