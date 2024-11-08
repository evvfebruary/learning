# Solution

###  Создание ВМ и установка Postgresql    
<procedure title="Пункты условия" id="psql-hw-p1" collapsible="true">
    <step>создать новый проект в Яндекс облако или на любых ВМ, например postgres2024-, где yyyymmdd год, месяц и день вашего рождения (имя проекта должно быть уникально)</step>
    <step>далее создать инстанс виртуальной машины с дефолтными параметрами - 1-2 ядра, 2-4Гб памяти, любой линукс, на курсе Ubuntu 100%</step>
    <step>добавить свой ssh ключ</step>
    <step>зайти удаленным ssh (первая сессия), не забывайте про ssh-add</step>
    <step>поставить PostgreSQL из пакетов apt install</step>
</procedure>

Создаем машинку в Compute Cloud, выберем 4x4 конфигурацию для начала.
![postgres-yc-example.png](postgres-yc-example.png){ width="450" }

Получаем готовый к эксплуатации инстанс на Yandex Cloud.
![yc-instance.png](yc-instance.png)

Устанавливаем свежий **Postgres**:
```shell
# Зайдем на удаленную машинку:
ssh evv@89.169.136.42

sudo apt-get update
# Добавим репозиторий Postgres в apt
sudo apt install -y postgresql-common
sudo /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh

# Установим сервер бд и библиотеку клиента
sudo apt install postgresql-16 postgresql-client-16 -y

# Запустим postrgessql как сервис
sudo systemctl start postgresql
```
![postgresql-status-server.png](postgresql-status-server.png)


###  Взаимодействие на разных уровнях транзакции, манипуляции через psql
<procedure title="Пункты условия" id="psql-hw-1-p2" collapsible="true">
    <step>зайти вторым ssh (вторая сессия)</step>
    <step>запустить везде psql из под пользователя postgres</step>
    <step>выключить auto commit</step>
    <step>сделать в первой сессии новую таблицу и наполнить ее данными</step>
    <step>create table persons(id serial, first_name text, second_name text);</step>
    <step>insert into persons(first_name, second_name) values('ivan', 'ivanov');</step>
    <step>insert into persons(first_name, second_name) values('petr', 'petrov');</step>
    <step>commit;</step>
    <step>посмотреть текущий уровень изоляции: show transaction isolation level</step>
    <step>начать новую транзакцию в обоих сессиях с дефолтным (не меняя) уровнем изоляции</step>
    <step>в первой сессии добавить новую запись</step>
    <step>insert into persons(first_name, second_name) values('sergey', 'sergeev');</step>
    <step>сделать select * from persons во второй сессии</step>
    <step>видите ли вы новую запись и если да то почему?</step>
    <step>завершить первую транзакцию - commit;</step>
    <step>сделать select * from persons во второй сессии</step>
    <step>видите ли вы новую запись и если да то почему?</step>
    <step>завершите транзакцию во второй сессии</step>
    <step>начать новые но уже repeatable read транзакции - set transaction isolation level repeatable read;</step>
    <step>в первой сессии добавить новую запись</step>
    <step>insert into persons(first_name, second_name) values('sveta', 'svetova');</step>
    <step>сделать select * from persons во второй сессии</step>
    <step>видите ли вы новую запись и если да то почему?</step>
    <step>завершить первую транзакцию - commit;</step>
    <step>сделать select * from persons во второй сессии</step>
    <step>видите ли вы новую запись и если да то почему?</step>
    <step>завершить вторую транзакцию</step>
    <step>сделать select * from persons во второй сессии</step>
    <step>видите ли вы новую запись и если да то почему?</step>
    <step>ДЗ сдаем в виде миниотчета в markdown в гите</step>
</procedure>


```shell
# Заходим в psql
sudo -i -u postgres
psql
```

Проверим настройки автокоммита, выключим его, наполним таблички данными:
![psql-console-1.png](psql-console-1.png)

> Заметим, что по умолчанию Postgres работает в режиме **read commited**, 
> что подразумевает под собой либо блокирование либо версионность ( **MVCC** ) изменяемых данных.

Начав две транзакции, проверим, видим ли мы строчку которую попытались инсертнуть:
```sql
-- Строчка для инсерта
INSERT INTO PERSONS(first_name, second_name) VALUES ('sergey', 'sergeev');
```

![sql-transactions.png](sql-transactions.png)
> **Видите ли вы новую запись и если да то почему?**
> Нет, не видим, так как мы работаем в режиме `read commited`, следовательно 
> мы видим только закомичеенные изменения до начала текущего запросы ( а первая сессия не выполнила `COMMIT` )


После `COMMIT` в первой транзакции, мы увидем запись используя `SELECT`
![psql-p3](psql-p3.png)

Включаем уровень транзакций `repeatable read`, вставляем строчку и читаем
<note>
Кстати вот тут мы поменяли уровень транзакций не в рамках сессии, но только для конкретной транзакции
</note>


> **Видите ли вы новую запись и если да то почему?** 
> Нет, не видим, так как уровень `repetable read` гарантирует нам постоянный снапшот данных
> при чтение в рамках одной транзакции чтения

Делаем `COMMIT` в первой сессии, смотрим на результат:
![psql-p4.png](psql-p4.png)

> **Видите ли вы новую запись и если да то почему?** 
> И снова не видим, снова `repetable read` гарантирует нам постоянный снапшот данных
> при чтение в рамках одной транзакции чтения

А теперь мы делаем `COMMIT` для нашей транзакции чтения, вызываем новую - и видимо новые данные!
![psql-p5.png](psql-p5.png)