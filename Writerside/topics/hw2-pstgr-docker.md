# hw2-pstgr-docker

### Домашнее задание
Работа с уровнями изоляции транзакции в PostgreSQL.

**Цель:**
- развернуть ВМ в GCP/ЯО/Аналоги
- установить туда докер
- установить PostgreSQL в Docker контейнере
- настроить контейнер для внешнего подключения

<procedure title="Описание/Пошаговая инструкция выполнения домашнего задания" id="some_procedure" collapsible="true">
    <step>сделать в GCE/ЯО/Аналоги инстанс с Ubuntu 20.04</step>
    <step>поставить на нем Docker Engine</step>
    <step>сделать каталог /var/lib/postgres</step>
    <step>развернуть контейнер с PostgreSQL 14 смонтировав в него /var/lib/postgres</step>
    <step>развернуть контейнер с клиентом postgres</step>
    <step>подключится из контейнера с клиентом к контейнеру с сервером и сделать</step>
    <step>таблицу с парой строк</step>
    <step>подключится к контейнеру с сервером с ноутбука/компьютера извне инстансов GCP/ЯО/Аналоги</step>
    <step>удалить контейнер с сервером</step>
    <step>создать его заново</step>
    <step>подключится снова из контейнера с клиентом к контейнеру с сервером</step>
    <step>проверить, что данные остались на месте</step>
    <step>оставляйте в ЛК ДЗ комментарии что и как вы делали и как боролись с проблемами</step>
</procedure>


### Решение

Создадим `Docker Compose` с указанными требованиями, легкой рукой дополнительно потюнем `shm_size`, чтобы запомнилось.    

Все файлы можно посмотреть в [этом репозитории](https://github.com/evvfebruary/learning/tree/main/databases/postgres/otus/homeworks/hw2-pstgr-docker).

```yaml
version: '3.9'

services:

  postgres:
    image: postgres:14
    container_name: postgres-server
    restart: always
#    Tune shared memory size
    shm_size: 128mb
#    Add volumes for data
    volumes:
      - pgdata:/var/lib/postgresql/data
#      Add connections from any host
      - ./pg_hba.conf:/var/lib/postgresql/data/pg_hba.conf
    ports:
      - '5432:5432'
    environment:
#      Override some environments
      POSTGRES_USER: postgres_user
      POSTGRES_PASSWORD: postgres_password

# Create separate container for psql client
  pgclient:
    image: postgres:14
    container_name: postgres-client
    depends_on:
      - postgres
#    Keep it running with some dirty trick
    entrypoint: ["tail", "-f", "/dev/null"]

volumes:
  pgdata:
    driver: local
```

Видим, что сервисы поднялись успешно!
![hw2-postgres-cmps-ports.png](hw2-postgres-cmps-ports.png)

Сделаем тестовую табличку и записями с нашего второго контейнере.
![hw2-psql-tables.png](hw2-psql-tables.png)

А вот для того чтобы подключиться к нашему инстансу `Postgres` извне, надо будет:
<procedure title="Открываем доступ извне">
<step>открыть порты наружу ( да, есть некоторый оверхед на перформанс из за <b>iptables</b>, но сейчас не важно)</step>
<step>поправить <b>pg_hba.conf</b></step>
<step>добавить правило для сети в <b>YC</b>, чтобы пробиться к порту <b>5432</b>
<img src="hw2-yc-rule.png" alt=""/>
</step>
</procedure>

Успешно пробиваемся к нашей базе через `DataGrip`.
![hw2-datagrip.png](hw2-datagrip.png)


Удаляем контейнер, создаем его заного и проверям, остались ли наши данные.
![hw2-get-data-after-removal.png](hw2-get-data-after-removal.png)





