# Statefulset

## 1. Запустите предложенный манифест statefulset.yml и service.yml

Посмотрите в вывод подов, там будет вывод команд `date; ls /data; sleep 10`

```bash
kubectl apply -f ../statefulset.yml -f ../service.yml
kubectl get pod
kubectl get pvc

kubectl logs base-0
```

## 2. Добавьте в манифест init-container

https://kubernetes.io/docs/concepts/workloads/pods/init-containers/
init container должен записывать в том с данными файл, называющийся 'podname.current_date'.
Команда, которой можно создать файл: `touch "$(hostname).$(date)"`

> Добавляем в манифест statefulset.yaml
>
>       initContainers:
>       - image: k8s.gcr.io/busybox
>         name: touch-file
>         command: [ "sh", "-c"]
>         args:
>           - touch "/data/$(hostname).$(date)"

```bash
kubectl apply -f 2.statefulset.yaml
kubectl get pod

kubectl logs base-0
```

## 3. Добавьте в statefulset podAntiAffinity

https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#more-practical-use-cases
Так, чтобы на каждом узле мог быть запущен только один под стейтфулсета.

> Добавляем в манифест statefulset.yaml
>
>       affinity:
>         podAntiAffinity:
>           requiredDuringSchedulingIgnoredDuringExecution:
>           - labelSelector:
>               matchExpressions:
>               - key: app
>                 operator: In
>                 values:
>                 - base
>             topologyKey: "kubernetes.io/hostname"

```bash
kubectl apply -f 3.statefulset.yaml
kubectl get pod

kubectl logs base-0
```

## 4. Отскейлите statefulset до трех подов.

Сколько подов запустилось ? Разберитесь почему именно столько.
В кластере два воркер-узла, так что должно было запуститься только два пода.

```bash
kubectl scale sts base --replicas=3
kubectl get pod

kubectl logs base-0
kubectl logs base-1

kubectl describe pod base-2
```

## 5. Проверьте, что в dns есть записи, созданные headless service base

```
kubectl run -t -i --rm --image amouat/network-utils test bash

nslookup base
```

## 6. Добавьте в манифест стейтфулсета описание Readiness Probe

Эта проба должна будет возвращать False для пода с именем base-1

> Добавляем в манифест statefulset.yaml
>
>         readinessProbe:
>           exec:
>             command: ["sh", "-c", "[ $(hostname) != base-1 ]"]
>           periodSeconds: 5
>           timeoutSeconds: 1

```bash
kubectl apply -f 6.statefulset.yaml
kubectl get pod

kubectl describe pod base-1
```

## 7. Если после применения измененного манифеста поды стейтфулсета остались неизменными.

Попробуйте разобраться, почему поды не изменились.
Подсказка: pod base-2 висит в состоянии Pending, а стейтфулсет всегда применяет изменения с наибольшего индекса.
Чтобы починить, надо отскейлить стейтфулсет до двух реплик.

## 8. Проверьте, что в dns есть записи, созданные headless service base, только для подов со статусом Ready: True

```bash
kubectl run -t -i --rm --image amouat/network-utils test bash

nslookup base
```

## 9. Очищаем кластер

```bash
kubectl delete sts base
kubectl delete service base
```
