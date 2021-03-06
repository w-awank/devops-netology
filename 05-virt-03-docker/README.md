## Задача 1

Сценарий выполнения задачи:

- создайте свой репозиторий на https://hub.docker.com;
- выберете любой образ, который содержит веб-сервер Nginx;
- создайте свой fork образа;
- реализуйте функциональность:
запуск веб-сервера в фоне с индекс-страницей, содержащей HTML-код ниже:
```
<html>
<head>
Hey, Netology
</head>
<body>
<h1>I’m DevOps Engineer!</h1>
</body>
</html>
```
Опубликуйте созданный форк в своем репозитории и предоставьте ответ в виде ссылки на https://hub.docker.com/username_repo.
> Решение.
> HTML-код выше записал в файл netology.html. Создал dockerfile:
> ```
> FROM nginx
> COPY netology.html /usr/share/nginx/html/index.html
> ```
> Далее создал свой образ wawank/my-nginx из этого докерфайла:
> ```bash
> $ docker build -t wawank/my-nginx .
> ```
> Проверил его на локальном контейнере:
> ```bash
> $ docker run --name my-nginx -d -p 8080:80 wawank/my-nginx
> ```
> И запушил свой образ в докерхаб. Проверяйте по ссылке: https://hub.docker.com/repository/docker/wawank/my-nginx

## Задача 2

Посмотрите на сценарий ниже и ответьте на вопрос:
"Подходит ли в этом сценарии использование Docker контейнеров или лучше подойдет виртуальная машина, физическая машина? Может быть возможны разные варианты?"

Детально опишите и обоснуйте свой выбор.

--

Сценарий:

- Высоконагруженное монолитное java веб-приложение;
  > Решение. ВМ либо ФМ, т.к. есть требование высокой нагрузки. Выделить iops/cpu/ram лучше в ВМ, чем в докере.
- Nodejs веб-приложение;
  > Решение. Возможны разные варианты. Больше склоняюсь к ВМ, чем к докеру из-за тяжелого nodejs по причинам, подобным в первом случае.
- Мобильное приложение c версиями для Android и iOS;
  > Решение. Докер, т.к можно на одной ОС крутить разные версии одного и того же ПО для разработки под Android и iOS. Если мобильное приложение - значит оно легковесное.
- Шина данных на базе Apache Kafka;
  > Решение. Т.к. приложение распределенное и горизонтально масштабируемое, то вполне можно использовать и ВМ, и докер.
- Elasticsearch кластер для реализации логирования продуктивного веб-приложения - три ноды elasticsearch, два logstash и две ноды kibana;
  > Решение. По моему тут лучше использовать ВМ...
- Мониторинг-стек на базе Prometheus и Grafana;
  > Решение. ВМ, т.к. слишком крупные проекты с учетом зависимостей от компонентов. Не похоже на микросервисы.
- MongoDB, как основное хранилище данных для java-приложения;
  > Решение. Докер. Докер хорош для большого числа json файлов NoSQL БД...
- Gitlab сервер для реализации CI/CD процессов и приватный (закрытый) Docker Registry.
  > Решение. Докер вполне подойдет. Также возможна реализация и на ВМ. 

## Задача 3

- Запустите первый контейнер из образа ***centos*** c любым тэгом в фоновом режиме, подключив папку ```/data``` из текущей рабочей директории на хостовой машине в ```/data``` контейнера;
  > Решение.
  > ```bash
  > # docker pull centos
  > # mkdir data
  > # docker run -tid --name my-centos-01 -v /home/wawank/docker/05.03/data/:/data/ centos
  > ````
- Запустите второй контейнер из образа ***debian*** в фоновом режиме, подключив папку ```/data``` из текущей рабочей директории на хостовой машине в ```/data``` контейнера;
  > Решение.
  > ```bash
  > # docker pull debian
  > # docker run -tid --name my-debian-01 -v /home/wawank/docker/05.03/data/:/data/ debian
  > ```
- Подключитесь к первому контейнеру с помощью ```docker exec``` и создайте текстовый файл любого содержания в ```/data```;
  > Решение.
  > ```bash
  > # docker exec -ti my-centos-01 bash
  > [root@9df6e8d3b0da /]# echo Netology > /data/first
  > [root@9df6e8d3b0da /]# exit
  > ```
- Добавьте еще один файл в папку ```/data``` на хостовой машине;
  > Решение.
  > ```bash
  > # echo Netology2 > /home/wawank/docker/05.03/data/second
  > ```
- Подключитесь во второй контейнер и отобразите листинг и содержание файлов в ```/data``` контейнера.
  > Решение.
  > ```bash
  > # docker exec -ti my-debian-01 bash
  > root@350de7ba290a:/# ls -al /data
  > total 16
  > drwxr-xr-x 2 root root 4096 Sep  5 15:51 .
  > drwxr-xr-x 1 root root 4096 Sep  5 15:43 ..
  > -rw-r--r-- 1 root root    9 Sep  5 15:48 first
  > -rw-r--r-- 1 root root   10 Sep  5 15:51 second
  > root@350de7ba290a:/# cat /data/first
  > Netology
  > root@350de7ba290a:/# cat /data/second
  > Netology2
  > root@350de7ba290a:/# exit
  > ```

## Задача 4 (*)

Воспроизвести практическую часть лекции самостоятельно.

Соберите Docker образ с Ansible, загрузите на Docker Hub и пришлите ссылку вместе с остальными ответами к задачам.
> Решение. https://hub.docker.com/repository/docker/wawank/my-ansible
