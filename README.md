# vtikh_microservices
vtikh microservices repository
## ДЗ 17 - kubernetes-1

В Я.облаке запущены 2 вм, в каждой из ни:

- установлены необходимые пакеты:


```
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
sudo apt update
sudo apt-get install docker-ce=5:19.03.15~3-0~ubuntu-bionic docker-ce-cli=5:19.03.15~3-0~ubuntu-bionic containerd.io kubelet=1.19.14-00 kubeadm=1.19.15-00 kubectl=1.19.15-00
```

- на первой (мастер) развернут kubernetes:

```
kubeadm init --apiserver-cert-extra-sans=<IP> --apiserver-advertise-address=0.0.0.0 --control-plane-endpoint=<IP> --pod-network-cidr=10.128.0.0/24
```

- вторая подключена к мастер-узлу:

```
sudo kubeadm join <IP>:6443 --token 9gzb3g.ob9xyzqow7kf4bj4 --discovery-token-ca-cert-hash sha256:07adfcf7e90e162059ac3cc50cc6e18ab6490499d61d70e572fc1174811d1e80
```

- на мастер узле создана конфигурация

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

- скачан и применен манифест Calico:

```
curl https://docs.projectcalico.org/manifests/calico.yaml -O
kubectl apply -f calico.yaml
```

- а также запущен один из подов:

```
kubectl apply -f ui-deployment.yml
...
$ kubectl get pods
NAME                             READY   STATUS    RESTARTS   AGE
ui-deployment-864cd78b97-jf6fh   1/1     Running   0          95m
```


## ДЗ 15 - monitoring

Добавляем мониторинг к микросервисному приложению.

Для запуска:

Создаем ВМ
```
yc compute instance create \
  --name docker-host \
  --zone ru-central1-a \
  --network-interface subnet-name=default-ru-central1-a,nat-ip-version=ipv4 \
  --create-boot-disk image-folder-id=standard-images,image-family=ubuntu-1804-lts,size=15 \
  --ssh-key ~/.ssh/id_rsa.pub
```

Настроить docker-machine и переключить окружение на нее

```
docker-machine create \
  --driver generic \
  --generic-ip-address=84.201.135.110 \
  --generic-ssh-user yc-user \
  --generic-ssh-key ~/.ssh/id_rsa \
  docker-host

eval $(docker-machine env docker-host)
```

Cобрать образы приложения:

```
for i in ui post-py comment; do cd src/$i; bash docker_build.sh; cd -; done
```

Далее перейти в директорию docker, заполнить файл .env, как показано в примере. Собрать образ prometheus:

```
export USER_NAME=username
docker build -t $USER_NAME/prometheus .
```

Запустить приложение:

```
docker-compose up -d
```

Приложение доступно по протоколу http и адресу ВМ на порту 9292. Мониторинг - на 9090

Образы загружены на docker hub: [https://hub.docker.com/u/vtikh](https://hub.docker.com/u/vtikh)

## ДЗ 14 - gitlab CI

В задании разворачивается сервер gitlab на ВМ в Я.облаке, который настраивается как дополнительный удаленный репозиторий. В gitlab настраивается пайплайн и окружения.

### Подготовка

Необходимо создать ВМ на ubuntu 18.04 в Я.облаке (2cpu/4RAM), в ней:

```
mkdir -p /srv/gitlab/config /srv/gitlab/data /srv/gitlab/logs
```

разместить в ВМ по пути `srv/gitlab` файл `gitlab-ci/docker-compose.yml`. Запустить контейнер:

```
docker-compose up -d
```

### Настройка CI

После успешного развертывания добавить удаленный репозиторий к этому проекту и отправить туда код:

```
git remote add gitlab http://<gitlab-vm-ip>/homework/example.git
git push gitlab gitlab-ci-1
```

Репозиторий уже содержит описание пайплайна `.gitlab-ci.yml`, поэтому требуется настроить только серверную часть. На ВМ-сервере gitlab создать воркер:

```
sudo docker run -d --name gitlab-runner --restart always -v /srv/gitlab-runner/config:/etc/gitlab-runner -v /var/run/docker.sock:/var/run/docker.sockgitlab/gitlab-runner:latest
```

и зарегистрировать его:

```
docker exec -it gitlab-runner gitlab-runner register \
    --url http://<your-ip>/ \
    --non-interactive \
    --locked=false \
    --name DockerRunner \
    --executor docker \
    --docker-image alpine:latest \
    --registration-token <your-token> \
    --tag-list "linux,xenial,ubuntu,docker" \
    --run-untagged
```

### Окружения

Также в пайплайне `.gitlab-ci.yml` объявлены окружения *dev*, *beta* и *production*. Последние два запускаются вручную.

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
