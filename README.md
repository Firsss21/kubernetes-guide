## Kubernetes quick guide

### Что такое K8s?
Kubernetes - проект с открытым исходным кодом, предназначенным для управления кластером контейнеров

+ Помогает управлять контейнеризированными приложениями в различных окружениях (physical, virtual, cloud)

Плюсы:
+ Хорошая надежность & zero downtime
+ Высокая расширяемость 
+ Аварийное восстановление 

### Компоненты K8s:
K8s содержит в себе огромное количество компонентов, но опишем основные используемые компоненты.

Worker Node - рабочая машина, являющаяся физической или виртуальной машиной

Pod - наименьший компонент в K8s
+ является абстракцией над контейнером (например Docker container)
+ обычно на каждый Pod по одному приложению (например db, java application, etc)
+ каждый юнит имеет свой личный IP-address и при пересодзании получает новый
+ эти юниты легко порождаются и убиваются при необходимости

Service 
 + перманнентный IP-адрес
 + жизненные циклы Service и Pod никак не связаны
 + Работает как балансировщик нагрузки
 + Репликации подключаются к одному и тому же сервису
 + Если один pod сервиса "отвалится", то он автоматически будет направлять запросы на реплику
 
Ingress
 + этот компонент получает внешние запросы и перенаправляет в нужные сервисы

ConfigMap
 + Внешнее хранилище конфигурации для приложения

Secrets 
 + Хранилище конфигурации для секьюрных данных приложения (пароли, ключи)

Volumes
 + Подключает физическое хранилище к вашему поду
 + Хранилище может быть как локальным, так и удаленным
 + Используется, например, для базы данных, чтобы данные не удалялись при рестарте приложения, так как сам K8s не заботится о сохранении данных

Deployment 
 + Вы не создаете напрямую сервисы, а описываете конфигурацию, с помощью которой определяются зависимости, количество реплик
 + Работает со Stateless приложениями

StatefulSet 
 + Используется для Stateful-приложений или баз данных (но они часто находятся вне k8s кластера)
 
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

`kubectl create deployment [name] --image=[image] [opts]` - создать deployment
`kubectl edit deployment [name]` - редактировать deployment
`kubectl delete deployment [name]` - удалить deployment
`kubectl get nodes | pod | services | replicaset | deployment` - для просмотра статуса различных компонентов
`kubectl logs [pod name]` - для просмотра логов пода
`kubectl exec -it [pod name]` - для подключения к поду через консоль

### K8s YAML файл
### Демо


### K8s namespaces
### K8s ingress
### Helm - package manager
### Volumes - persisting data in K8s
### K8s StatefulSet
### K8s Services
