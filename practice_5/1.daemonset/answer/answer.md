# Daemon set

## 1. Изменить предложенный манифест deployment.yml так, чтобы получился daemonset.

Документация на daemonset: https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/

> Меняем:
> kind: Deployment на  kind: DaemonSet

> Удаляем:
>
>  replicas: 3
>
>  strategy:
>    rollingUpdate:
>      maxSurge: 25%
>      maxUnavailable: 25%
>    type: RollingUpdate

```bash
vim 1.daemonset.yml
```

## 2. Запустите получившийся daemonset.yml в кластере.

Посмотрите на каких узлах запустились поды, если не на всех, тогда выполните пункт 3

```bash
kubectl apply -f 1.daemonset.yml
kubectl get pod -o wide
```

## 3. Добавьте необходимые настройки, чтобы поды были запущены на всех узлах кластера.

> Добавляем toleration:
>
>      tolerations:
>      - effect: NoSchedule
>        operator: Exists

```bash
kubectl apply -f 3.daemonset.yml
kubectl get pod -o wide
```

## 4. Добавьте вывод текущей даты в цикл, выполняемый внутри контейнера. Измените настройки rollingUpdate, чтобы одновременно обновлялось по 50% запущенных подов.

Документация: https://kubernetes.io/docs/tasks/manage-daemon/update-daemon-set/

> Добавляем
>
>  updateStrategy:
>    type: RollingUpdate
>    rollingUpdate:
>      maxUnavailable: 50%

```bash
kubectl apply -f 4.daemonset.yml
kubectl get pod -o wide
```

## 5. Измените настройки rollingUpdate, чтобы обновлялось только по одному поду.

> Меняем значение поля maxUnavailable:
>
>       maxUnavailable: 1

```bash
kubectl apply -f 5.daemonset.yml
kubectl get pod -o wide
```

## 6. Очищаем кластер

```bash
kubectl delete ds node-name
```
