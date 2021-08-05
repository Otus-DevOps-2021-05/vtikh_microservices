# vtikh_microservices
vtikh microservices repository

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

Для запуска этого образа:
```
docker run --name reddit -d -p 9292:9292 vtikh/otus-reddit:1.0
```

Приложение будет доступно по адресу [http://localhost:9292](localhost:9292)
