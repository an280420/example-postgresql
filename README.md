# SQL основные команды

Данный конспект сделан по видео Алексея Голобурдина

🔥Разбираем SQL на примере PostgreSQL — SELECT, JOIN, GROUP, HAVING, Coalesce и др.
https://www.youtube.com/watch?v=WpojDncIWOw

Приводится примеры использования самых необходимых команд для работы с SQL базами данных

## install postgresql
Установка 

```
sudo apt-get install postgresql postgresql-contrib
```

Поле установки необходиом создать роли (пользователя postgres)
Вход в консоль postgresql под юзером postgres

```
psql postgres
```

Основные команды для работы в консоли с postgresql

Просмотреть список баз данных
```
postgres=# \l
```

### Create DB and requests example

Cоздадим базу данных магазина
```
postgres=# create database shop;
```
подключимся к базе данных 
```
postgres=# \c shop
```
посмотрим какие есть таблицы в данной базе
```
shop=# \d
```

#### Создание таблиц
Пока таблиц нет. Создадим таблицу  сustomer
```
shop=# create table customer(
shop(# id serial primary key,
shop(# name varchar(255),
shop(# phone varchar(30),
shop(# email varchar(255)
shop(# );
```
посмотрим созданную таблицу
```
shop=# \d
```
Что покажет нам список таблиц
```
              List of relations
 Schema |      Name       |   Type   | Owner 
--------+-----------------+----------+-------
 public | customer        | table    | anton
 public | customer_id_seq | sequence | anton
(2 rows)
```
Посмотрим саму таблицу
```
shop=# \d customer
```

```
                                    Table "public.customer"
 Column |          Type          | Collation | Nullable |               Default                
--------+------------------------+-----------+----------+--------------------------------------
 id     | integer                |           | not null | nextval('customer_id_seq'::regclass)
 name   | character varying(255) |           |          | 
 phone  | character varying(30)  |           |          | 
 email  | character varying(255) |           |          | 
Indexes:
    "customer_pkey" PRIMARY KEY, btree (id)
```
Coздадим таблицу продуктов
```
shop=# create table product(
shop(# id serial primary key,
shop(# name varchar(255),
shop(# description text,
shop(# price integer);
```

```
shop=# \d
              List of relations
 Schema |      Name       |   Type   | Owner 
--------+-----------------+----------+-------
 public | customer        | table    | anton
 public | customer_id_seq | sequence | anton
 public | product         | table    | anton
 public | product_id_seq  | sequence | anton
(4 rows)
```
Создадим таблицу product_photo
```
shop=# create table product_photo(
shop(# id serial primary key, 
shop(# url varchar(255),
shop(# product_id integer references product(id));
```
В данной строке
```
shop(# product_id integer references product(id));
```
устанавливает связь между таблицами product_photo и product

Создадим и другие таблицы
```
shop=# create table cart(
shop(# customer_id integer references customer(id),
shop(# id serial primary key);
```

```
shop=# create table cart_product(
shop(# cart_id integer references cart(id),
shop(# product_id integer references product(id));
```
Посмотрим все созданные таблицы
```
shop=# \d
                List of relations
 Schema |         Name         |   Type   | Owner 
--------+----------------------+----------+-------
 public | cart                 | table    | anton
 public | cart_id_seq          | sequence | anton
 public | cart_product         | table    | anton
 public | customer             | table    | anton
 public | customer_id_seq      | sequence | anton
 public | product              | table    | anton
 public | product_id_seq       | sequence | anton
 public | product_photo        | table    | anton
 public | product_photo_id_seq | sequence | anton
(9 rows)
```

#### Заполнение таблиц данными
Заполним наши таблицы
```
shop=# insert into customer(name, phone, email) values('Василий', 02, 'vasya@mail.com');
```
```
shop=# insert into customer(name, phone, email)
values('Петр', 03, 'petya@mail.com');
```
Выбрать все поля из таблицы customer
```
shop=# select * from customer;
 id |  name   | phone |     email      
----+---------+-------+----------------
  1 | Василий | 2     | vasya@mail.com
  2 | Петр    | 3     | petya@mail.com
(2 rows)
```
Заполним таблицу product
```
shop=# insert into product (name, description, price)
shop-# values ('iPhone', 'крутой телефон', 100000);

shop=# insert into product (name, description, price)
values ('Apple watch', 'крутые часы', 50000);
```
Посмотрим, что получилось
```
shop=# select * from product;
 id |    name     |  description   | price  
----+-------------+----------------+--------
  1 | iPhone      | крутой телефон | 100000
  2 | Apple watch | крутые часы    |  50000
(2 rows)
```
Заполним таблицу product_photo
```
shop=# insert into product_photo (url, product_id) values ('iphone_photo', 1);
```
Просмотрим существующие записи в таблице
```
shop=# select * from product_photo;
 id |     url      | product_id 
----+--------------+------------
  1 | iphone_photo |          1
(1 row)
```

#### Join

Потренируемся использовать join
```
shop=# select * from product_photo pp;
```
```
 id |     url      | product_id 
----+--------------+------------
  1 | iphone_photo |          1
(1 row)

```
pp  в данном случае alias (синоним полного названия таблицы)
Попробуем join left
```
shop=# select pp.* from product_photo pp left join product p on p.id = pp.product_id;
 id |     url      | product_id 
----+--------------+------------
  1 | iphone_photo |          1
(1 row)
```
Если захотим посмотреть все поля
```
shop=# select * from product_photo pp left join product p on p.id = pp.product_id;
 id |     url      | product_id | id |  name  |  description   | price  
----+--------------+------------+----+--------+----------------+--------
  1 | iphone_photo |          1 |  1 | iPhone | крутой телефон | 100000
(1 row)
```
Eсли захотим посмотреть выборочные поля 
```
shop=# select pp.*, p.name from product_photo pp left join product p on p.id = pp.product_id;
 id |     url      | product_id |  name  
----+--------------+------------+--------
  1 | iphone_photo |          1 | iPhone
(1 row)
```
Попробуем добавить фото на несуществующий продукт. Для этого нам необходимо удалить внешний ключ из таблицы product_photo
```
shop=# alter table product_photo drop constraint product_photo_product_id_fkey;
```

Наша таблица немного изменится
было 
```
shop=# \d product_photo
                                      Table "public.product_photo"
   Column   |          Type          | Collation | Nullable |                  Default                  
------------+------------------------+-----------+----------+-------------------------------------------
 id         | integer                |           | not null | nextval('product_photo_id_seq'::regclass)
 url        | character varying(255) |           |          | 
 product_id | integer                |           |          | 
Indexes:
    "product_photo_pkey" PRIMARY KEY, btree (id)
Foreign-key constraints:
    "product_photo_product_id_fkey" FOREIGN KEY (product_id) REFERENCES product(id)
```
Стало
```
shop=# \d product_photo
                                      Table "public.product_photo"
   Column   |          Type          | Collation | Nullable |                  Default                  
------------+------------------------+-----------+----------+-------------------------------------------
 id         | integer                |           | not null | nextval('product_photo_id_seq'::regclass)
 url        | character varying(255) |           |          | 
 product_id | integer                |           |          | 
Indexes:
    "product_photo_pkey" PRIMARY KEY, btree (id)
```
Теперь мы можем создавать фотографию на несуществующий в базе данных продукт
```
shop=# insert into product_photo (url, product_id) values ('unknown photo', 100);
```
Посмотрим, что получилось
```
shop=# select * from product_photo;
 id |      url      | product_id 
----+---------------+------------
  1 | iphone_photo  |          1
  2 | unknown photo |        100
(2 rows)
```
Посмотрим как теперь работает *left join*. Из левой таблицы выбираются все записи
```
shop=# select pp.*, p.name from product_photo pp left join product p on p.id = pp.product_id;
 id |      url      | product_id |  name  
----+---------------+------------+--------
  1 | iphone_photo  |          1 | iPhone
  2 | unknown photo |        100 | 
(2 rows)
```
И сравним это с тем как работает *right join*
```
shop=# select pp.*, p.name from product_photo pp right join product p on p.id = pp.product_id;
 id |     url      | product_id |    name     
----+--------------+------------+-------------
  1 | iphone_photo |          1 | iPhone
    |              |            | Apple watch
(2 rows)
```
Теперь из правой таблицы выбираются все записи ( в данном случае правая таблица - главная). По продукту **Apple watch** у нас не создано фото. Поэтому в полях id, url,  product_id для продукта **Apple watch**  значения - *null*

Теперь можно посмотреть как работает *inner join*. Просто заменим в запросе *left join* на *inner join*
```
shop=# select pp.*, p.name from product_photo pp inner join product p on p.id = pp.product_id;
 id |     url      | product_id |  name  
----+--------------+------------+--------
  1 | iphone_photo |          1 | iPhone
(1 row)
```

Для запоминания приведем мнемоническую картинку для *join*
![join.jpg](images/join.jpg)

#### Удаление и обновление данных
Посмотрим еще раз нашу таблицу product_photo 
```
shop=# select * from product_photo;
 id |      url      | product_id 
----+---------------+------------
  1 | iphone_photo  |          1
  2 | unknown photo |        100
(2 rows)
```
И удалим фото несуществующего товара
```
shop=# delete from product_photo where id=2;
```
Обновим фото
```
shop=# update product_photo set url='iphone_image2' where id=1;
```
```
shop=# select * from product_photo;
 id |      url      | product_id 
----+---------------+------------
  1 | iphone_image2 |          1
(1 row)
```

#### Select

Создадим заказ для одного из клиентов
```
shop=# insert into cart (customer_id) values (1);
```
```
shop=# select * from cart;
 customer_id | id 
-------------+----
           1 |  1
(1 row)
```
Наполним только что созданную корзину продуктами посредством добавления продуктов в промежуточную таблицу *cart_product*. Добавим сразу два товара одним запросом
```
shop=# insert into cart_product (cart_id, product_id) values (1, 1), (1, 2);
```
```
shop=# select * from cart_product 
;
 cart_id | product_id 
---------+------------
       1 |          1
       1 |          2
(2 rows)
```

#### Сложные запросы
Сформируем таблицу с именем клиента и с общей суммой заказов этого клиента

Достанем имена клиентов
```
shop=# select c.name from customer c;
```
```
  name   
---------
 Василий
 Петр
(2 rows)
```
Заджойним корзину
```
shop=# select c.name, cart.id as cart_id from customer c left join cart on cart.customer_id=c.id;
```
```
  name   | cart_id 
---------+---------
 Василий |       1
 Петр    |        
(2 rows)
```
Используем еще один *join*
```
shop=# select c.name, cart.id as cart_id from customer c left join cart on cart.customer_id=c.id left join cart_product cp on cp.cart_id=cart.id;
```
```
  name   | cart_id 
---------+---------
 Василий |       1
 Василий |       1
 Петр    |        
(3 rows)
```
Выведем продукты Васи
```
shop=# select c.name, cart.id as cart_id, cp.product_id from customer c left join cart on cart.customer_id=c.id left join cart_product cp on cp.cart_id=cart.id;
```
```
  name   | cart_id | product_id 
---------+---------+------------
 Василий |       1 |          1
 Василий |       1 |          2
 Петр    |         |           
(3 rows)
```
Еще добавим один *join*
```
shop=# select c.name, cart.id as cart_id, p.name from customer c left join cart on cart.customer_id=c.id left join cart_product cp on cp.cart_id=cart.id left join product p on p.id=cp.product_id;
```
```
  name   | cart_id |    name     
---------+---------+-------------
 Василий |       1 | iPhone
 Василий |       1 | Apple watch
 Петр    |         | 
(3 rows)
```

```
shop=# select c.name, cart.id as cart_id, p.name, p.price from customer c left join cart on cart.customer_id=c.id left join cart_product cp on cp.cart_id=cart.id left join product p on p.id=cp.product_id;
```
```
  name   | cart_id |    name     | price  
---------+---------+-------------+--------
 Василий |       1 | iPhone      | 100000
 Василий |       1 | Apple watch |  50000
 Петр    |         |             |       
(3 rows)
```
Теперь пришла очередь проссумировать товары в корзине. Уберем лишние поля, сгруппируем и проссумируем, используем `group by c.name` и `sum(p.price)`
```
shop=# select c.name, sum(p.price) from customer c left join cart on cart.customer_id=c.id left join cart_product cp on cp.cart_id=cart.id left join product p on p.id=cp.product_id group by c.name;
```
```
  name   |  sum   
---------+--------
 Петр    |       
 Василий | 150000
(2 rows)
```
У Петра есть поле *null*. Сделаем чтобы отображался *0*
```
shop=# select c.name, coalesce(sum(p.price), 0) as order_sum from customer c left join cart on cart.customer_id=c.id left join cart_product cp on cp.cart_id=cart.id left join product p on p.id=cp.product_id group by c.name;
```
```
  name   | order_sum 
---------+-----------
 Петр    |         0
 Василий |    150000
(2 rows)
```

Выполним сортировку по сумме заказа
```
shop=# select c.name, coalesce(sum(p.price), 0) as order_sum from customer c left join cart on cart.customer_id=c.id left join cart_product cp on cp.cart_id=cart.id left join product p on p.id=cp.product_id group by c.name order by order_sum desc;
```
```
  name   | order_sum 
---------+-----------
 Василий |    150000
 Петр    |         0
(2 rows)
```
Если нам необходимо вывести только тех клиентов, которые что-то приборели, то есть у кого сумма заказа больше нуля, воспользуемся *having*
```
shop=# select c.name, coalesce(sum(p.price), 0) as order_sum from customer c left join cart on cart.customer_id=c.id left join cart_product cp on cp.cart_id=cart.id left join product p on p.id=cp.product_id group by c.name having sum(p.price)>0;
```
```
  name   | order_sum 
---------+-----------
 Василий |    150000
(1 row)
```
#### Отличие *where* от *having*
*having* фильтрует группы
*where* фильтрует строки

Пример использования *limit*
```
shop=# select * from customer order by name limit 1;
```
```
 id |  name   | phone |     email      
----+---------+-------+----------------
  1 | Василий | 2     | vasya@mail.com
(1 row)
```
Выводится только один клиент
Для смещения применяем *offset*
```
shop=# select * from customer order by name limit 1 offset 1;
```
```
 id | name | phone |     email      
----+------+-------+----------------
  2 | Петр | 3     | petya@mail.com
(1 row)
```
Что применяется часто для пагинации - разбивки больших массивов информации на страницы


