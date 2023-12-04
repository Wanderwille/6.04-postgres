# Домашнее задание к занятию "6.4. PostgreSQL" - Подус Сергей

## Задача 1

Используя docker поднимите инстанс PostgreSQL (версию 13). Данные БД сохраните в volume.

Подключитесь к БД PostgreSQL используя `psql`.

Воспользуйтесь командой `\?` для вывода подсказки по имеющимся в `psql` управляющим командам.

**Найдите и приведите** управляющие команды для:
- вывода списка БД
- подключения к БД
- вывода списка таблиц
- вывода описания содержимого таблиц
- выхода из psql

## Ответ:

Вывод списка БД: \l[+] - [+] подробный вывод

Подключение к БД: \connect (\c)

Вывод списка таблиц: \dt [+] - [+] подробный вывод

Вывод описания содержимого таблиц: \d tablename

Выход из psql: \q 

## Задача 2

Используя `psql` создайте БД `test_database`.

Изучите [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/master/06-db-04-postgresql/test_data).

Восстановите бэкап БД в `test_database`.

Перейдите в управляющую консоль `psql` внутри контейнера.

Подключитесь к восстановленной БД и проведите операцию ANALYZE для сбора статистики по таблице.

Используя таблицу [pg_stats](https://postgrespro.ru/docs/postgresql/12/view-pg-stats), найдите столбец таблицы `orders` 
с наибольшим средним значением размера элементов в байтах.

**Приведите в ответе** команду, которую вы использовали для вычисления и полученный результат.

## Ответ: 

create database test_database;

psql -U postgres  test_database < /backups/test_dump.sql 

psql -U postgres

ANALYZE VERBOSE orders;

select attname, avg_width from pg_stats where tablename='orders' order by avg_width desc limit 1;

```
 attname | avg_width
---------+-----------
 title   |        16
(1 row)

```


# Задача 3

Архитектор и администратор БД выяснили, что ваша таблица orders разрослась до невиданных размеров и
поиск по ней занимает долгое время. Вам, как успешному выпускнику курсов DevOps в нетологии предложили
провести разбиение таблицы на 2 (шардировать на orders_1 - price>499 и orders_2 - price<=499).

## Ответ:
```
CREATE TABLE orders_1 (CHECK (price > 499)) INHERITS (orders);
CREATE TABLE orders_2 (CHECK (price <= 499)) INHERITS (orders);

CREATE RULE rule_orders_1 AS ON INSERT TO orders WHERE (price > 499) 
DO INSTEAD INSERT INTO orders_1 VALUES (NEW.*);

CREATE RULE rule_orders_2 AS ON INSERT TO orders WHERE (price <= 499) 
DO INSTEAD INSERT INTO orders_2 VALUES (NEW.*);

INSERT INTO orders_1 (title, price) (select title, price from orders where price > 499);
INSERT INTO orders_2 (title, price) (select title, price from orders where price <= 499);
```

## Можно ли было изначально исключить "ручное" разбиение при проектировании таблицы orders?  
Можно заранее спроектировать разбиение таблицы путем [декларативного партиционирования](https://pgdash.io/blog/postgres-11-sharding.html): 
Основная таблица в таком случае светится в выводе \dt+ как "секционированная таблица" и не заполняется даннными. Данные пишутся сразу в заранее подготовленные партиции.


# Задача 4

Используя утилиту `pg_dump` создайте бекап БД `test_database`.  
 
 
Как бы вы доработали бэкап-файл, чтобы добавить уникальность значения столбца `title` для таблиц `test_database`?  

## Ответ:

`pg_dump -U postgres -O -F p  -C test_database > /backups/backups.sql` 

```
CREATE INDEX uniq_title ON orders(title);
CREATE INDEX uniq_title_1 ON orders_1(title);
CREATE INDEX uniq_title_2 ON orders_2(title);
```
