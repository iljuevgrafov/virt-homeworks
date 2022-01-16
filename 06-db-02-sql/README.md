# Домашнее задание к занятию "6.2. SQL"

## Введение

Перед выполнением задания вы можете ознакомиться с 
[дополнительными материалами](https://github.com/netology-code/virt-homeworks/tree/master/additional/README.md).

## Задача 1

Используя docker поднимите инстанс PostgreSQL (версию 12) c 2 volume, 
в который будут складываться данные БД и бэкапы.

Приведите получившуюся команду или docker-compose манифест.
```
Создаем 2 тома  
docker volume create vol1  
docker volume create vol2  

Содержание docker-compose:  
services:  
  postgresdb:  
    image: postgres  
    environment:  
      POSTGRES_DB: "test_db"  
      POSTGRES_USER: "test-admin-user"  
      POSTGRES_PASSWORD: "123456"  
    volumes:  
        - vol1:/var/lib/postgresql/data  
        - vol2:/var/lib/postgresql/backup  
    ports:  
      - "5432:5432"  
volumes:  
    vol1:  
        external:  
            name: vol1  
   vol2:  
        external:  
            name: vol2  
```
## Задача 2

В БД из задачи 1: 
- создайте пользователя test-admin-user и БД test_db
- в БД test_db создайте таблицу orders и clients (спeцификация таблиц ниже)
- предоставьте привилегии на все операции пользователю test-admin-user на таблицы БД test_db
- создайте пользователя test-simple-user  
- предоставьте пользователю test-simple-user права на SELECT/INSERT/UPDATE/DELETE данных таблиц БД test_db

Таблица orders:
- id (serial primary key)
- наименование (string)
- цена (integer)

Таблица clients:
- id (serial primary key)
- фамилия (string)
- страна проживания (string, index)
- заказ (foreign key orders)

Приведите:
- итоговый список БД после выполнения пунктов выше,
- описание таблиц (describe)
- SQL-запрос для выдачи списка пользователей с правами над таблицами test_db
- список пользователей с правами над таблицами test_db

![image](https://user-images.githubusercontent.com/48878229/149655120-eb8e5096-e142-468c-8f8b-cacb81a4f6ab.png)
```
SQL-запрос
SELECT table_name, privilege_type, grantee
FROM   information_schema.table_privileges 
WHERE table_schema = 'public';

Список пользователей
table_name | privilege_type |     grantee     
------------+----------------+-----------------
 orders     | INSERT         | test-admin-user
 orders     | SELECT         | test-admin-user
 orders     | UPDATE         | test-admin-user
 orders     | DELETE         | test-admin-user
 orders     | TRUNCATE       | test-admin-user
 orders     | REFERENCES     | test-admin-user
 orders     | TRIGGER        | test-admin-user
 clients    | INSERT         | test-admin-user
 clients    | SELECT         | test-admin-user
 clients    | UPDATE         | test-admin-user
 clients    | DELETE         | test-admin-user
 clients    | TRUNCATE       | test-admin-user
 clients    | REFERENCES     | test-admin-user
 clients    | TRIGGER        | test-admin-user
```

## Задача 3

Используя SQL синтаксис - наполните таблицы следующими тестовыми данными:

Таблица orders

|Наименование|цена|
|------------|----|
|Шоколад| 10 |
|Принтер| 3000 |
|Книга| 500 |
|Монитор| 7000|
|Гитара| 4000|

Таблица clients

|ФИО|Страна проживания|
|------------|----|
|Иванов Иван Иванович| USA |
|Петров Петр Петрович| Canada |
|Иоганн Себастьян Бах| Japan |
|Ронни Джеймс Дио| Russia|
|Ritchie Blackmore| Russia|

Используя SQL синтаксис:
- вычислите количество записей для каждой таблицы 
- приведите в ответе:
    - запросы 
    - результаты их выполнения.
```
select count(*) from orders;
select count(*) from clients;
```
<img width="252" alt="image" src="https://user-images.githubusercontent.com/48878229/149674001-92abedbd-1392-4091-a067-b8484b5c52ea.png">


## Задача 4

Часть пользователей из таблицы clients решили оформить заказы из таблицы orders.

Используя foreign keys свяжите записи из таблиц, согласно таблице:

|ФИО|Заказ|
|------------|----|
|Иванов Иван Иванович| Книга |
|Петров Петр Петрович| Монитор |
|Иоганн Себастьян Бах| Гитара |

Приведите SQL-запросы для выполнения данных операций.
```
update clients set заказ = 3 where ФИО = 'Иванов Иван Иванович';
```
Приведите SQL-запрос для выдачи всех пользователей, которые совершили заказ, а также вывод данного запроса.
```
select * from clients where заказ IS NOT NULL;
```
 <img width="363" alt="image" src="https://user-images.githubusercontent.com/48878229/149674357-5403f86b-2208-4e43-a705-f49d9c82bfe9.png">

Подсказк - используйте директиву `UPDATE`.

## Задача 5

Получите полную информацию по выполнению запроса выдачи всех пользователей из задачи 4 
(используя директиву EXPLAIN).

Приведите получившийся результат и объясните что значат полученные значения.

## Задача 6

Создайте бэкап БД test_db и поместите его в volume, предназначенный для бэкапов (см. Задачу 1).

Остановите контейнер с PostgreSQL (но не удаляйте volumes).

Поднимите новый пустой контейнер с PostgreSQL.

Восстановите БД test_db в новом контейнере.

Приведите список операций, который вы применяли для бэкапа данных и восстановления. 

---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
