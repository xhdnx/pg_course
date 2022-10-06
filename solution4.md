## поставить Docker Engine

> hdn@denispg:/$ curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
rm get-docker.sh
sudo usermod -aG docker $USER


## развернуть контейнер с PostgreSQL 13 смонтировав в него /var/lib/postgres

> hdn@denispg:/$ sudo docker run --name pg-dockeras --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5433:5433 -v /var/lib/postgres:/var/lib/postgresql/data postgres:13
> cdedf7d59905cbb650cf5a5bd15609691e2793716c7be8b2c75ecaf9b61f3882

Создан контейнер с сервером Postgresql 13. Возникли проблемы при попытке создать контейнер для postgresql 14 версии. Также пришлось изменить порт на 5433, т.к 5432 используется для pg кластера 14 версии 

## развернуть контейнер с клиентом postgres
> hdn@denispg:/$ sudo docker run -it --rm --network pg-net --name pg-client postgres:13 psql -h pg-dockeras -U p
ostgres
>Password for user postgres:
>psql (13.8 (Debian 13.8-1.pgdg110+1))
>Type "help" for help.
>postgres=#

Успешно удалось смонтировать еще один контейнер в котором выполняется подключение через клиент pqsl к серверной части из первого контейнера. Можно сразу выполнить создание тестовой таблицы и записей, но есть возможность зайти в контейнер повторно. Для этого стартуем контейнер и проваливаемся в него

> hdn@denispg:/$ sudo docker container start f42f1a6595be
> hdn@denistt:/$ sudo docker ps -a
CONTAINER ID   IMAGE         COMMAND                  CREATED          STATUS          PORTS                                       NAMES
f42f1a6595be   postgres:13   "docker-entrypoint.s…"   7 minutes ago    Up 2 seconds    5432/tcp                                    pg-client
661dd4c1022a   postgres:13   "docker-entrypoint.s…"   10 minutes ago   Up 10 minutes   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   pg-docker

>hdn@denistt:/$ sudo docker exec -it pg-client bash 
> root@f42f1a6595be:/# psql -h pg-docker -U postgres
Password for user postgres: *postgres*
psql (13.8 (Debian 13.8-1.pgdg110+1))
Type "help" for help.
postgres=#

## подключится из контейнера с клиентом к контейнеру с сервером и сделать

> postgres=# CREATE TABLE test1 (col1 int);
CREATE TABLE
>postgres=# INSERT INTO test1 values(5);
INSERT 0 1
>postgres=# INSERT INTO test1 values(55);
INSERT 0 1
>postgres=# select * from test1;
 col1
------
    5
    55
    (2 rows)
>postgres=#

Создаем таблицу и делаем в нее две записи

## подключится к контейнеру с сервером с ноутбука/компьютера извне инстансов GCP

Сначала провалимся в контейнер с сервером pg и отредактируем hba файла и откроем все порты 0.0.0.0/0. Подключаться будем со второй ВМ созданной на Яндекс облаке 158.160.15.78

>hdn@denispg:~$ psql -h 84.201.162.179 -p 5432 -U postgres
Password for user postgres:
psql (14.5 (Ubuntu 14.5-1.pgdg22.04+1), server 13.8 (Debian 13.8-1.pgdg110+1))
Type "help" for help.
postgres=# select * from test1;
col1
    5
   55
(2 rows)
postgres=#

Убедились что попали куда надо, выведя записи из тестовой таблицы

## удалить контейнер с сервером

> hdn@denistt:/$ sudo docker ps -a
CONTAINER ID   IMAGE         COMMAND                  CREATED          STATUS          PORTS                                       NAMES
f42f1a6595be   postgres:13   "docker-entrypoint.s…"   35 minutes ago   Up 28 minutes   5432/tcp                                    pg-client
661dd4c1022a   postgres:13   "docker-entrypoint.s…"   38 minutes ago   Up 38 minutes   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   pg-docker
> hdn@denistt:/$ sudo docker container stop 661dd4c1022a
661dd4c1022a
> hdn@denistt:/$ sudo docker container rm 661dd4c1022a
661dd4c1022a
> hdn@denistt:/$ sudo docker ps -a
CONTAINER ID   IMAGE         COMMAND                  CREATED          STATUS          PORTS      NAMES
f42f1a6595be   postgres:13   "docker-entrypoint.s…"   36 minutes ago   Up 28 minutes   5432/tcp   pg-client
hdn@denistt:/$

Остается только контейнер с клиентом pg - psql.

## создать его заново

> hdn@denistt:/$ sudo docker run --name pg-docker --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:13
3519a786744c94759c21c360fcca5d819b590137765b768416e3148999fb457f
> hdn@denistt:/$ sudo docker ps -a
CONTAINER ID   IMAGE         COMMAND                  CREATED          STATUS          PORTS                                       NAMES
3519a786744c   postgres:13   "docker-entrypoint.s…"   10 seconds ago   Up 9 seconds    0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   pg-docker
f42f1a6595be   postgres:13   "docker-entrypoint.s…"   38 minutes ago   Up 30 minutes   5432/tcp                                    pg-client
hdn@denistt:/$

Контейнер поднялся 

## подключится снова из контейнера с клиентом к контейнеру с сервером проверить, что данные остались на месте

> hdn@denistt:/$ sudo docker exec -it pg-client bash
> root@f42f1a6595be:/# psql -h pg-docker -U postgres
psql (13.8 (Debian 13.8-1.pgdg110+1))
Type "help" for help.

postgres=# select * from test1;
col1
    5
   55
(2 rows)

postgres=#

Подключаемся к контейнеру pg-client и выполняем вход на pg_server из контейнера pg-docker. Таблица с данными осталась на месте.

## Выводы
Не простое дз, т.к впервые столкнулся с docker. 
1. Много заморочек, с портами (был развернут pg на машине на порту 5432 и пытался прокинуть из контейнера порт pg на 5432). 
2. Разбирался с ключами docker. Все команды писал вручную чтобы плотнее улеглось в голове.

Суммарно потратил на ДЗ около 5 часов (не считая просмотра лекции). Урок очень понравился
