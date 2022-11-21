## Kubernetes quick guide

### Что такое K8s?
Kubernetes - проект с открытым исходным кодом, предназначенным для управления кластером контейнеров

+ Помогает управлять контейнеризированными приложениями в различных окружениях (physical, virtual, cloud)

Плюсы:
+ Хорошая надежность & zero downtime
+ Высокая расширяемость 
+ Аварийное восстановление 

### Компоненты K8s:
__K8s__ содержит в себе огромное количество компонентов, но опишем основные используемые компоненты.

__Worker Node__ - рабочая машина, являющаяся физической или виртуальной машиной

__Pod__ - наименьший компонент в K8s, представляющий собой «обертку» для одного или группы контейнеров
+ является абстракцией над контейнером (например Docker container)
+ обычно на каждый Pod по одному приложению (например db, java application, etc)
+ каждый юнит имеет свой личный IP-address и при пересодзании получает новый
+ эти юниты легко порождаются и убиваются при необходимости

__Service__ - абстрактный объект, который определяет логический набор подов и политику доступа к ним. 
 + перманнентный IP-адрес
 + жизненные циклы Service и Pod никак не связаны
 + Работает как балансировщик нагрузки
 + Репликации подключаются к одному и тому же сервису
 + Если один pod сервиса "отвалится", то он автоматически будет направлять запросы на реплику
 
__Ingress__ - объект API, который управляет внешним доступом к объектам Service в проекте
 + этот компонент получает внешние запросы и перенаправляет в нужные сервисы

__ConfigMap__ - позволяет отделить конфигурационные файлы от содержимого образа
 + Внешнее хранилище конфигурации для приложения

__Secrets__ - позволяет хранить и управлять конфиденциальной информацией, такой как пароли, токены OAuth и ключи SSH
 + Хранилище конфигурации для секьюрных данных приложения (пароли, ключи)

__Volumes__ - это единицы хранения, подготовленные админом
 + Подключает физическое хранилище к вашему поду
 + Хранилище может быть как локальным, так и удаленным
 + Используется, например, для базы данных, чтобы данные не удалялись при рестарте приложения, так как сам K8s не заботится о сохранении данных

__Deployment__ - представляет работающее приложение в кластере
 + Вы не создаете напрямую сервисы, а описываете конфигурацию, с помощью которой определяются зависимости, количество реплик
 + Работает со Stateless приложениями

__StatefulSet__ - используется для управления приложениями и отслеживания их состояния.
 + Используется для Stateful-приложений или баз данных (но они часто находятся вне k8s кластера)
 
__Абстракция слоев:__
1. Deployment управляет ReplicaSet
2. ReplicaSet управляет Pod
3. Pod абстракция над Container
4. Container
Все, что ниже Deployment управляется Kubernetes'ом

### Архитектура K8s
Главным компонентом K8s является `worker node` и каждый нод будет иметь несколько подов с контейнерами, работающими на нем. На каждом ноде работает 3 процесса, которые работают для управления этими нодами, а также самой фактической требуемой работы от них, поэтому они и называются `worker node`.

Процессы, работающие на каждой `Worker node`:
1. `Container runtime`
2. `Kubelet` -  взаимодействует с контейнером и нодой. Ответственный за получение настроек и фактического запуска пода с контейнерами внутри с определенным выделением ресурсов поду
3. `Kubeproxy` - ответственный за переадресацию запросов сервисов до модулей. Старается избежать накладных расходов ресурсов, например тем, что отправляет запросы приоритетно в свой же node.

`Master node` - имеют совсем другие процессы, используемые для мониторинга, запуска, остановки, подов, подключения к нодам.
1. `Api server` - когда вы как пользователь k8s хотите сделать что-то, то обращаетесь через клиент(UI, API, CLI (kubectl)) к `api-server`'у, который далее уже направляет запросы в необходимые процессы. По сути `cluster gateway`. Запросы, приходящие к нему проходят проверку на аутентификацию, валидность. Единственная входная точка кластера.
2. `Scheduler` - планировщик, получает запросы от Api server и создает задания на выполнение чего либо. Например при задаче создания пода, просмотрит имеющиеся ресурсы на нодах и выберет самый малозагруженный node, после чего передаст задачу в `Kubelet` ноды на создание пода.
3. `Controller manager` - следит за изменением состояния кластеров. 

   Например: `pod` умер > `controller manager` обнаружил это > Передает в `scheduler` задание на пересоздание упавших подов > `scheduler` осуществляет свою работу.

4. `etcd` - Key, Value хранилище кластера, каждое изменение кластера сохраняется в `etcd`. Все остальные процессы работают на данных `etcd`, вся информация берется отсюда.

### Minikube & Kubectl

`Minikube` - Используется в локальной разработке. Для полного K8s кластера необходимо несколько `Master/Worker node`, что требует несколько физических машин или разделения на виртуалки, но `Minikube` размещает на одной ноде и `Master node` и `Worker node`, для этого он использует Virtual Box.

`Kubectl` - Инструмент, для взаимодействия с кластером K8s через командную строку. Общается с ApiServer k8s.

Установка Minikube и Kubectl: https://kubernetes.io/docs/tasks/tools/ и https://minikube.sigs.k8s.io/docs/start/

Для запуска Minikube cluster: ```minikube start``` или ```minikube start --vm-driver=hyperkit```, чтобы запустить кластер на драйвере виртуализации hyperkit.

### Основные команды K8s

• `kubectl create deployment [name] --image=[image] [opts]` - создать deployment

• `kubectl edit deployment [name]` - редактировать deployment

• `kubectl delete deployment [name]` - удалить deployment

• `kubectl describe [pod name]` - возвращает расширенную информацию о поде

• `kubectl get nodes | pod | services | replicaset | deployment [-o wide для доп инфы]` - для просмотра статуса различных компонентов

• `kubectl logs [pod name]` - для просмотра логов пода

• `kubectl exec -it [pod name]` - для подключения к поду через консоль

• `kubectl apply -f [file name]` - подключает файл конфигурации

Больше команд команд на [официальном сайте](//kubernetes.io/ru/docs/reference/kubectl/overview/) или `kubectl --help`

### K8s YAML файл

В файле `.yaml` создаваемого объекта Kubernetes необходимо указать значения для следующих полей:

• __apiVersion__ — используемая для создания объекта версия API Kubernete

• __metadata__ — данные, позволяющие идентифицировать объект (`name`, `UID` и необязательное поле `namespace`)

• __spec__ — требуемое состояние объекта

• __kind__ — тип создаваемого объекта

Конкретный формат поля-объекта spec зависит от типа объекта Kubernetes и содержит вложенные поля, предназначенные только для используемого объекта.

Пример конфигурации:

__Deployment__
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template: # template описывает конфигурацию для pod'а
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.16
        ports:
        - containerPort: 8080
```
__Service__
```yml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```
### Демо
__Deployment:__
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongodb-deployment
  labels:
    app: mongodb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
      - name: mongodb
        image: mongo
        ports:
        - containerPort: 27017
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          valueFrom:
            secretKeyRef:
              name: mongodb-secret
              key: mongo-root-username
        - name: MONGO_INITDB_ROOT_PASSWORD
          valueFrom: 
            secretKeyRef:
              name: mongodb-secret
              key: mongo-root-password
```
__Service:__
```yml
apiVersion: v1
kind: Service
metadata:
  name: mongodb-service
spec:
  selector:
    app: mongodb
  ports:
    - protocol: TCP
      port: 27017
      targetPort: 27017
```
__Config Map:__
```yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mongodb-configmap
data:
  database_url: mongodb-service
```
__Secrets:__
```yml
apiVersion: v1
kind: Secret
metadata:
    name: mongodb-secret
type: Opaque
data:
    mongo-root-username: dXNlcm5hbWU=
    mongo-root-password: cGFzc3dvcmQ=
```
__Another deployment & service__
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo-express
  labels:
    app: mongo-express
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongo-express
  template:
    metadata:
      labels:
        app: mongo-express
    spec:
      containers:
      - name: mongo-express
        image: mongo-express
        ports:
        - containerPort: 8081
        env:
        - name: ME_CONFIG_MONGODB_ADMINUSERNAME
          valueFrom:
            secretKeyRef:
              name: mongodb-secret
              key: mongo-root-username
        - name: ME_CONFIG_MONGODB_ADMINPASSWORD
          valueFrom: 
            secretKeyRef:
              name: mongodb-secret
              key: mongo-root-password
        - name: ME_CONFIG_MONGODB_SERVER
          valueFrom: 
            configMapKeyRef:
              name: mongodb-configmap
              key: database_url
---
apiVersion: v1
kind: Service
metadata:
  name: mongo-express-service
spec:
  selector:
    app: mongo-express
  type: LoadBalancer  # assigns service an external ip address and so accepts ext. requests
  ports:
    - protocol: TCP
      port: 8081
      targetPort: 8081
      nodePort: 30000 # port for external ip address (30000-32767)
```

### K8s namespaces

`Namespaces` - можно рассмотреть как виртуальный кластер внутри вашего кластера `Kubernetes`. Вы можете иметь несколько изолированных друг от друга пространств имен внутри одного кластера `Kubernetes`. Они реально могут помочь вам и вашим командам с организацией, безопасностью и даже производительностью системы.

__Причины использования:__
 
 1. Без них - все в одном стандартном `namespace`.
 2. Много команд работают в одном окружения - появятся конфликты.
 3. Распределение ресурсов: `dev`, `stage` окружения.
 4. `Blue/green deployment` - бесшовный деплоймент
 5. Разграничивание доступов для команд

__Какие требования нужны для внедрения namespaces? __

1. Необходима структуризация компонентов
2. Возникают конфликты в окружении между командами
3. Необходимость шаринга сервисов между окружениями
4. Ограничение доступов и ресурсов для различных окружений

__Особенности:__

- Каждый `namespace` должен определять свой `ConfigMap/Secret`

- `Node/Volume` виден на весь кластер, вы не можете его изолировать

- По умолчанию, компоненты создаются в стандартном `namespace`

__Для определения компонента в `namespace`:__
1. `kubectl apply -f mysql-configmap.yaml --namespace=my-namespace`
2. Или через файл
```yml
metadata: 
  namespace: my-namespace
```

__Стандартные namespaces:__

- kubesystem:
1. Системные процессы
2. Не изменяйте и не модифицируйте этот `namespace`

- kube public:
1. Публично доступная информация
2. `ConfigMap`, которая содержит в себе информацию о кластере

- kube-node-lease:

1. "Сердцебиение" всех нод
2. Каждый узел имеет связанный объект в пространстве имен
3. Определяет доступность узлов

- default:
1. Все создаваемые компоненты по умолчанию попадают сюда

__Создание нового namespace__
```
 kubectl create namespace my-namespace
```
Но лучше создавать их с помощью yaml конфигураций

__Как изменить активный `namespace`?__

Из коробки в k8s нет удобного решения, но можно установить `kubens` или использовать стандартные возможности:
```
kubectl get pods --namespace=test
```

Установка `kubens`:
```
snap install kubectx
```
Применение:
```
kubens - отобразить все namespaces

kubens [name] для смены namespace
```

### K8s ingress

__Ingress__ — это объект API, который управляет внешним доступом к сервисам в кластере, главным образом через HTTP / HTTPS. Чтобы Ingress-ресурс заработал, нужен Ingress-контроллер. Если вы используете GCE, то контроллер Ingress уже развернут в мастер. Однако, если вы самостоятельно загрузили кластер, например с kops на AWS, вам нужно развернуть Ingress-контроллер самостоятельно. На minikube это решается включением надстройки Ingress.

__Ingress-контроллеры__

Роль Ingress-контроллера могут выполнять NGINX Ingress Controller, Kong, Octavia Ingress Controller и др. В этой статье мы рассмотрим такой инструмент как Traefik и посмотрим, как можно использовать его в качестве Ingress-контроллера для сервисов в кластере.

__Зачем?__

Зачем использовать Ingress-контроллер, если можно предоставить доступ к каждому сервису через NodePort или LoadBalancer? Если кратко, это позволяет получить одну центральную точку для проксирования всего трафика. То есть, с использованием Ingress-контроллера вам понадобится всего один LoadBalancer для Traefik и ничего больше. Эта связка и будет разруливать весь трафик.

### Helm - package manager

__Helm__ - пакетный менеджер для запуска приложений в кластере. С его помощью можно создавать единые шаблоны для описания приложений, и `helm` будет дальше готовить и запускать все остальное сам. По сути можно сказать, что аналог `Docker Repository`, но для кластеров

- Установим `helm`

```
snap install helm
```

- После установки самого `helm`, надо подключить репозиторий. По-умолчанию ни один из них не подключен.

```
helm repo add stable https://kubernetes-charts.storage.googleapis.com/
```

- Теперь можно его проверить, выведя всех доступных чартов. По сути это набор приложений, которые может развернуть в `kubernetes helm`.

```
helm search repo stable
```
- Обновить репозиторий можно командой.

```
helm repo update
```

- Все как в привычных пакетных менеджерах. Рекомендуется обновлять репозиторий перед установкой, чтобы забрать самую свежую версию приложения.

Для поиска по репозиторию `helm` можно использовать search

```
helm search hub wordpress
```

__Установка приложений через Helm__

Для примера давайте поставим интересную панельку для управления и установки чартов helm через браузер. Называется она Kubeapps. Для ее установки подключим репозиторий разработчика.
```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```
- Kubeapps использует отдельный namespace, создадим его.
```
kubectl create namespace kubeapps
```

- Ставим приложение Kubeapps через Helm.
```
helm install kubeapps --namespace kubeapps bitnami/kubeapps
```

- Наблюдать за созданием подом можно с помощью команды.
```
kubectl get pods -w --namespace kubeapps
```

__Подробнее:__
https://serveradmin.ru/rabota-s-helm-3-v-kubernetes/

### Volumes - хранилище данных для K8s

__Виды `Volumes`:__
- `Persistent volume` 
- `Persistent volume claim`
- `Storage class`

__Persistent volume:__
- ресурс кластера
- создается с помощью yaml конфигурации
- физическое или облачное хранилище
- `Persistent volume` не используются в `namespaces` и доступны для всего кластера
		
  TODO.. 
