# Домашнее задание к занятию «Индексы»

### Задание 1

### Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

### Решение
### Если я правильно понял, что нужно сделать, то вот так
### ![](https://github.com/Berezhok/hw_index/blob/main/img/zad1.png)


### Задание 2

### Выполните explain analyze следующего запроса:
### ```sql
### select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
### from payment p, rental r, customer c, inventory i, film f
### where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
### ```
### - перечислите узкие места;
### - оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.
### Решение
### Запрос довольно долго обрабатывается. При некоторых обработках запроса происходит опрос 642000 строк. При этом для чего столько указано лишних данных в запросе, если нужно только две таблицы. Попробуем убрать все лишнее.
###  так все это выглядело до оптимизации, каких-то куча строк, время обработки около 4 секунд.
### -> Table scan on <temporary>  (cost=2.5..2.5 rows=0) (actual time=4465..4465 rows=391 loops=1)
###    -> Temporary table with deduplication  (cost=0..0 rows=0) (actual time=4465..4465 rows=391 loops=1)
###        -> Window aggregate with buffering: sum(payment.amount) OVER (PARTITION BY c.customer_id,f.title )   (actual time=1829..4319 rows=642000 loops=1)
###            -> Sort: c.customer_id, f.title  (actual time=1829..1877 rows=642000 loops=1)
###                -> Stream results  (cost=22.8e+6 rows=17.1e+6) (actual time=1.19..1285 rows=642000 loops=1)
###                    -> Nested loop inner join  (cost=22.8e+6 rows=17.1e+6) (actual time=1.18..1108 rows=642000 loops=1)
###                        -> Nested loop inner join  (cost=21.1e+6 rows=17.1e+6) (actual time=0.84..982 rows=642000 loops=1)
###                            -> Nested loop inner join  (cost=19.3e+6 rows=17.1e+6) (actual time=0.831..852 rows=642000 loops=1)
###                                -> Inner hash join (no condition)  (cost=1.65e+6 rows=16.5e+6) (actual time=0.564..33.1 rows=634000 loops=1)
###                                    -> Filter: (cast(p.payment_date as date) = '2005-07-30')  (cost=1.72 rows=16500) (actual time=0.0247..6.08 rows=634 loops=1)
###                                        -> Table scan on p  (cost=1.72 rows=16500) (actual time=0.016..3.94 rows=16044 loops=1)
###                                    -> Hash
###                                        -> Covering index scan on f using idx_title  (cost=103 rows=1000) (actual time=0.339..0.476 rows=1000 loops=1)
###                                -> Covering index lookup on r using rental_date (rental_date=p.payment_date)  (cost=0.969 rows=1.04) (actual time=858e-6..0.0012 rows=1.01 loops=634000)
###                            -> Single-row index lookup on c using PRIMARY (customer_id=r.customer_id)  (cost=250e-6 rows=1) (actual time=94.5e-6..109e-6 rows=1 loops=642000)
###                        -> Single-row covering index lookup on i using PRIMARY (inventory_id=r.inventory_id)  (cost=250e-6 rows=1) (actual time=80.2e-6..94.8e-6 rows=1 loops=642000)
<<<<<<< HEAD
### -----------------------+++++++++++++++++++++++++++++++++++----------------------------------------
=======
### -----------------------+++++++++++++++++++++++++++++++++++-----------------------------------------
>>>>>>> 4292ef0d66443754727d18d9f504ee69a7afed65
### Так стало после изменений. Время обработки значительно сократилось, строк стало в разы меньше при обработке оконной функции с 642000 снизилось до 634.
### Я не совсем понимаю что это, но этих петель loops тоже стало на порядки меньше. 
### -> Table scan on <temporary>  (cost=2.5..2.5 rows=0) (actual time=4.33..4.35 rows=391 loops=1)
###    -> Temporary table with deduplication  (cost=0..0 rows=0) (actual time=4.33..4.33 rows=391 loops=1)
###        -> Window aggregate with buffering: sum(payment.amount) OVER (PARTITION BY c.customer_id )   (actual time=3.71..4.22 rows=634 loops=1)
###            -> Sort: c.customer_id  (actual time=3.69..3.71 rows=634 loops=1)
###                -> Stream results  (cost=7449 rows=16500) (actual time=0.0551..3.61 rows=634 loops=1)
###                    -> Nested loop inner join  (cost=7449 rows=16500) (actual time=0.0499..3.51 rows=634 loops=1)
###                        -> Filter: (cast(p.payment_date as date) = '2005-07-30')  (cost=1674 rows=16500) (actual time=0.0387..3.11 rows=634 loops=1)
###                            -> Table scan on p  (cost=1674 rows=16500) (actual time=0.0314..2.4 rows=16044 loops=1)
###                        -> Single-row index lookup on c using PRIMARY (customer_id=p.customer_id)  (cost=0.25 rows=1) (actual time=524e-6..537e-6 rows=1 loops=634)
### Так выглядит новый запрос. Ну и как видно полученная таблица ничем не отличается от изначальной.
### ![](https://github.com/Berezhok/hw_index/blob/main/img/zad2.png)
### +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
### Доработка
### Я попытался переделать с вашими предложениями, но все равно не очень понятно. Вроде я создал index1 в таблице payment для payment_date, но при запросе explain analyze не могу понять работает он с ним или нет.
### ![](https://github.com/Berezhok/hw_index/blob/main/img/index1.png)
### ![](https://github.com/Berezhok/hw_index/blob/main/img/explain.png)

