# vtikh_microservices
vtikh microservices repository

## ДЗ 13 - docker-4

Кроме запуска контейнеров в разных типах docker-сетей в задании изучается docker-compose.

Для запуска кода необходимо:
1. установить docker-compose
2. заполнить файл переменных `src/.env` основываясь на примере.
3. если необходимо, присоединиться к удаленной docker-machine (см. предыдущие hw)
4. выполнить:

```
cd src
docker-compose up -d
```

Приложение доступно по адресу хоста, где запущены контейнеры.



## ДЗ 12 - docker-3

В задании приложение разбивается на микросервисы

### Сборка образов:


```
docker build -t vtikh/post:1.0 ./post-py
docker build -t vtikh/comment:1.0 ./comment
docker build -t vtikh/ui:2.0 ./ui
```

## Запуск контейнеров

Создать сеть и хранилище для контейнера с БД:
```
docker network create reddit
docker volume create reddit_db
```

Запуск:

```
docker run -d --network=reddit --network-alias=post_db --network-alias=comment_db -v reddit_db:/data/db mongo:latest
docker run -d --network=reddit --network-alias=post <your-dockerhub-login>/post:1.0
docker run -d --network=reddit --network-alias=comment <your-dockerhub-login>/comment:1.0
docker run -d --network=reddit -p 9292:9292 <your-dockerhub-login>/ui:2.0
```


## ДЗ 11 - docker-2

В задании изучается **docker**. Сделано:

- Локально развернут docker, опробованы простые команды управления контейнерами и образами
- Развернута docker-machine на базе ВМ в Яндекс.облаке
- Собран образ docker с приложением
- Образ проверен на docker-machine и загружен на docker hub
- Проверено развертывание приложения из образа, загружженого на docker hub

Развертывание docker-machine и подключение к ней:
```
yc compute instance create \
  --name docker-host \
  --zone ru-central1-a \
  --network-interface subnet-name=default-ru-central1-a,nat-ip-version=ipv4 \
  --create-boot-disk image-folder-id=standard-images,image-family=ubuntu-1804-lts,size=15 \
  --ssh-key ~/.ssh/id_rsa.pub \
  --core-fraction=5

docker-machine create \
  --driver generic \
  --generic-ip-address=<ip> \
  --generic-ssh-user yc-user \
  --generic-ssh-key ~/.ssh/id_rsa \
  docker-host

eval $(docker-machine env docker-host)
```

Для запуска этого образа:
```
docker run --name reddit -d -p 9292:9292 vtikh/otus-reddit:1.0
```

Приложение будет доступно по адресу [http://localhost:9292](localhost:9292)
