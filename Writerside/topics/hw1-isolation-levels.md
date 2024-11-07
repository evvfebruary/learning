# hw1-isolation-levels

### Домашнее задание
Работа с уровнями изоляции транзакции в PostgreSQL.

**Цель**:
- научиться работать с Google Cloud Platform на уровне Google Compute Engine (IaaS) /ЯО
- научиться управлять уровнем изолции транзации в PostgreSQL и понимать особенность работы уровней read commited и repeatable read

<procedure title="Описание/Пошаговая инструкция выполнения домашнего задания" id="some_procedure" collapsible="true">
    <step>создать новый проект в Яндекс облако или на любых ВМ, например postgres2024-, где yyyymmdd год, месяц и день вашего рождения (имя проекта должно быть уникально)</step>
    <step>далее создать инстанс виртуальной машины с дефолтными параметрами - 1-2 ядра, 2-4Гб памяти, любой линукс, на курсе Ubuntu 100%</step>
    <step>добавить свой ssh ключ</step>
    <step>зайти удаленным ssh (первая сессия), не забывайте про ssh-add</step>
    <step>поставить PostgreSQL из пакетов apt install</step>
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

