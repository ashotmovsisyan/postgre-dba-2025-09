# №1 — Работа с уровнями изоляции транзакций в PostgreSQL

## Установка и подготовка среды

**Скриншоты:**

| ![screenshot1](../../resources/hw-1/10001-VM-dashboard.png)       | ![screenshot2](../../resources/hw-1/10002-Connection-to-the-server.png) |
|-------------------------------------------------------------------|-------------------------------------------------------------------------|
| ![screenshot3](../../resources/hw-1/10003-Connecting-to-psql.png) | ![screenshot4](../../resources/hw-1/10004-autocommit-off.png)           |


## Задания

* в первой сессии новую таблицу и наполнить ее данными 
```postgresql
create table persons (
    id serial,
    first_name text,
    second_name text
);
insert into persons (first_name, second_name) values ('ivan', 'ivanov');
insert into persons (first_name, second_name) values ('petr', 'petrov');
commit;
```

* посмотреть текущий уровень изоляции: `show transaction isolation level`

команда: 
```postgresql
show transaction isolation level;
```
результат:
```txt
 transaction_isolation 
-----------------------
 read committed
(1 row)
```

* начать новую транзакцию в обоих сессиях с дефолтным (не меняя) уровнем изоляции
* в первой сессии добавить новую запись 
```postgresql
insert into persons (first_name, second_name) values('sergey', 'sergeev');
```
* сделать `select * from persons;` во второй сессии
* видите ли вы новую запись и если да то почему?
![screenshot1-4](../../resources/hw-1/1-4.png)
> Новые добавленные строки во второй сессии не видны, потому что уровень изоляции `READ COMMITTED` не позволяет видеть незафиксированные (`not commited`) изменения в других транзакциях.

* завершить первую транзакцию - `commit;`;
* сделать `select * from persons;` во второй сессии
* видите ли вы новую запись и если да то почему?
![screenshot1-4](../../resources/hw-1/1-6.png)

> Да, новая запись видна. После того как мы успешно завершили транзакцию в первой сессии, во второй сессии становится видна новая добавленная строка. При таком уровне изоляции можно читать только те данные, которые были зафиксированы до момента выполнения запроса `SELECT`.

* начать новые, но уже `repeatable read` транзакции - 
```postgresql
set transaction isolation level repeatable read;
```

Убедимся что уровень изоляции поставленно правильно.

![screenshot2-1](../../resources/hw-1/2-1.png)

* в первой сессии добавить новую запись 
```postgresql
insert into persons (first_name, second_name) values ('sveta', 'svetova');
```
* сделать `select * from persons` во второй сессии.
![screenshot2-2](../../resources/hw-1/2-2.png)

* видите ли вы новую запись и если да то почему?
> Новую строку не видно, потому что при таком уровне изоляции также нельзя читать незафиксированные данные. 

* завершить первую транзакцию - `commit;`
* сделать `select * from persons` во второй сессии
![screenshot2-3](../../resources/hw-1/2-3.png)

* видите ли вы новую запись и если да то почему?
> В этом случае мы также не видим новые данные, потому что при данном уровне изоляции транзакция работает с одним снимком данных, который создаётся в момент её начала и не изменяется до завершения транзакции.

* завершить вторую транзакцию - `commit;`
* сделать `select * from persons` во второй сессии
![screenshot2-4](../../resources/hw-1/2-4.png)

* видите ли вы новую запись и если да то почему?
> После завершения транзакции, при выполнении нового запроса для получения данных, мы уже имеем дело с новой транзакцией, в которой должны быть видны все зафиксированные изменения.


