# Домашнее задание к занятию "6.4. PostgreSQL"

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

```
- вывода списка БД
\l
- подключения к БД
\c
- вывода списка таблиц
\dt
- вывода описания содержимого таблиц
\d table_name
- выхода из psql
exit

```

## Задача 2

Используя `psql` создайте БД `test_database`.

Изучите [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/master/06-db-04-postgresql/test_data).

Восстановите бэкап БД в `test_database`.

Перейдите в управляющую консоль `psql` внутри контейнера.

Подключитесь к восстановленной БД и проведите операцию ANALYZE для сбора статистики по таблице.

Используя таблицу [pg_stats](https://postgrespro.ru/docs/postgresql/12/view-pg-stats), найдите столбец таблицы `orders` 
с наибольшим средним значением размера элементов в байтах.

**Приведите в ответе** команду, которую вы использовали для вычисления и полученный результат.
```
select attname, avg_width from pg_stats where(tablename = 'orders' and avg_width=(select max(avg_width) from pg_stats where tablename = 'orders'));
```
![image](https://user-images.githubusercontent.com/48878229/151705710-1a20199e-5ad2-4bd6-842b-f6bea04d8901.png)

## Задача 3

Архитектор и администратор БД выяснили, что ваша таблица orders разрослась до невиданных размеров и
поиск по ней занимает долгое время. Вам, как успешному выпускнику курсов DevOps в нетологии предложили
провести разбиение таблицы на 2 (шардировать на orders_1 - price>499 и orders_2 - price<=499).

Предложите SQL-транзакцию для проведения данной операции.

```
CREATE TABLE orders_1 (LIKE orders);
CREATE TABLE orders_2 (LIKE orders);

INSERT INTO orders_1 SELECT * FROM orders WHERE
	price > 499;
DELETE FROM orders WHERE price > 499;

INSERT INTO orders_2 SELECT * FROM orders WHERE
	price <= 499;
DELETE FROM orders WHERE price <= 499;
```

Можно ли было изначально исключить "ручное" разбиение при проектировании таблицы orders?

```
CREATE TABLE orders (
    id integer NOT NULL,
    title character varying(80) NOT NULL,
    price integer DEFAULT 0
) PARTITION BY RANGE(price); 

CREATE TABLE orders_1
    PARTITION OF orders
    FOR VALUES FROM (0) TO (499);

CREATE TABLE orders_2
    PARTITION OF orders
    FOR VALUES FROM (499) TO (1000000) # не нашел спопоба указать диапазон только с начальным значением (499);
```

## Задача 4

Используя утилиту `pg_dump` создайте бекап БД `test_database`.

Как бы вы доработали бэкап-файл, чтобы добавить уникальность значения столбца `title` для таблиц `test_database`?

```
Объявеление таблицы исправил бы на следующее:
CREATE TABLE public.orders (
    id integer NOT NULL,
    title character varying(80) NOT NULL UNIQUE,
    price integer DEFAULT 0
);
```
---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
