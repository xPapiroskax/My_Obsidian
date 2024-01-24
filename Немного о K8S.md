Декларативный подход - через yml файл, т.е. поддержание практики "инфраструктура как код"

Императивный подход - задание команд напрямую через командную строку. Самая плохая практика будущего, если что-то сломается, то придется вбивать все ручками.
Есть выход из ситуации, когда наши pod были изменены императивным подходам, но нам нужно это зафиксировать в yml в yml файл - поможет команда:
```
kubectl -n test get pod <my-app> -o yaml
```

#### 3.5 Контроллеры.Replicaset

Pod - это логическая абстракция, а физически он представляет из linux namespace-ов из pid namespace и сетевого namespace
В namespace может быть множество pod из разных node.
namespace и node - это не связанные вещи!
Логи можно смотреть только у pod, а у replicaSet нельзя

#### 3.6 Deployment
replicaset делает не популярным тот факт, что при обновлении приложения он автоматом не обновляет наши pods, нужно ручками удалять старые поды и только после этого replicaset поднимет pods с обновленной версией.
Теперь о самом Deployment(контроллер контроллеров - Deployment является самым популярным контролером):
Deployment контролирует replicaset:
```
kubectl -n home-dev get pods
NAME                              READY   STATUS    RESTARTS   AGE
http-server-f6nqj                 1/1     Running   0          12m
http-server-qqbwp                 1/1     Running   0          12m
go-http-server-66c9f6fb88-mswxs   1/1     Running   0          57s
66c9f6fb88 - id replicaset порожденного deployment, а id mswxs - создает репликасет для pods

kubectl -n home-dev get rs
NAME                        DESIRED   CURRENT   READY   AGE
http-server                 2         2         2       22h
go-http-server-66c9f6fb88   3         3         3       11m
```
использует стратегию RollingUpdate, которая сначала создает pods, а после удаляет старые pods
```
kubectl explain(типо help) deploy.spec.strategy
Из yaml:
strategy:
    rollingUpdate:
      maxSurge: 25% # создает новые поды
      maxUnavailable: 25% #удаляет старые поды

watch kubectl -n home-dev get po - выводит таблицу подов в реальном времени

kubectl -n home-dev rollout history deployment/go-http-server - выводит сохранненые версии

kubectl -n home-dev rollout undo deployment/go-http-server - команда отката к предыдущей версии replicaSet(откат в оба направления 0.1-0.2, 0.2-0.1 версии)

kubectl -n home-dev rollout restart deployment/go-http-server - рестарт pods подконтрольных deployment

kubectl -n home-dev delete po go-http-server-6c76584b87-99cpd - также можно сделать рестарт pods просто удалив их

kubectl -n home-dev scale deployment/go-http-server --replicas=1 - кубернетес deployment можно масштабировать, но это плохая практика, т.к. она нарушает правило "Infrastructure-as-Code; Iac"
```

#### 3.7 DaemonSet
kubectl delete node worker1 - удаление node
поднимает контейнер с новой node и синхронизирует с помощью daemonset обе nodes

#### 3.9 Job
Создание разовой задачи, которая отработает и изчезнет - поможет job
kubectl -n home-dev describe po echo-job-t4x6g - выводи описание ресурса

#### 3.9 CronJob
Позваляет создавать по расписанию разовые задачи
CronJob пораждает Job, а Job пораждает Pod: CronJob -> Job -> Pod
