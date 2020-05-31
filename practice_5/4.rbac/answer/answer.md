# RBAC

## 0. Подготовка стенда. Запускаем prepare.sh

```bash
../prepare.sh
```

## 1. Cоздайте на сервере master-1 пользователя devel и настройте ему доступ в кластер

Создайте ServiceAccount devel с доступом в неймспейс dev, с ролью edit
Получите токен и создайте контекст devel в конфигурации kubectl для пользователя devel с авторизацией по токену и кластера kubernetes

```
kubectl create serviceaccount devel --namespace dev
kubectl create rolebinding devel --namespace dev --serviceaccount dev:devel --clusterrole edit
export devel_secret_name="$(kubectl get serviceaccount devel --namespace dev -o jsonpath='{.secrets[].name}')"
export devel_token="$(kubectl get secret $devel_secret_name --namespace dev -o jsonpath='{.data.token}' | base64 -d )"

kubectl config set-credentials devel --token="$devel_token"
kubectl config set-context devel --cluster=kubernetes --user=devel
```

Проверяем работу:

```
kubectl config use-context devel
kubectl get pod # ошибка
kubectl get pod -n dev # показывает список подов

kubectl config use-context kubernetes-admin@kubernetes
```
## 2. Cоздайте на сервере master-1 пользователя qa и настройте ему доступ в кластер

Создайте ServiceAccount qa в неймспейсе prod, с доступом в неймспейс dev с ролью edit и с доступом в неймспейс prod с ролью view
Получите токен и создайте контекст qa в конфигурации kubectl для пользователя qa с авторизацией по токену и кластера kubernetes

Можно повторить команды выше, а можно создать из манифестов

```
kubectl apply -f qa/sa.yml
kubectl apply -f qa/rolebind-dev.yml
kubectl apply -f qa/rolebind-prod.yml

export qa_secret_name="$(kubectl get serviceaccount qa --namespace prod -o jsonpath='{.secrets[].name}')"
export qa_token="$(kubectl get secret $qa_secret_name --namespace prod -o jsonpath='{.data.token}' | base64 -d )"

kubectl config set-credentials qa --token="$qa_token"
kubectl config set-context qa --cluster=kubernetes --user=qa
```

```
kubectl config use-context qa
kubectl get pod # ошибка
kubectl get pod -n dev # показывает список подов
kubectl get pod -n prod # показывает список подов
kubectl create cm test --from-literal=key=data -n prod # ошибка

kubectl config use-context kubernetes-admin@kubernetes
```

## 3. Создайте ингресс для доступа к API через ингресс контроллер из манифеста ingress.yml.

Поменяйте в манифесте в двух местах название хоста s<номер студента>, на свой номер логина и примените его в кластер.

```
vi ../ingress.yml

kubectl apply -f ../ingress.yml
```

## 4. Создайте запись kube-ext в списке кластеров, указав адрес из ингресса и опцию --insecure-skip-tls-verify=true

```
kubectl config set-cluster kube-ext --insecure-skip-tls-verify=true --server=https://api.s<номер студента>.edu.slurm.io
```

## 5. Создайте контексты qa-ext и devel-ext для юзеров qa и devel с кластером kube-ext

```
kubectl config set-context devel-ext --cluster=kube-ext --user=devel
kubectl config set-context qa-ext --cluster=kube-ext --user=qa
```

## 6. Рассмотрите конфиг kubectl

Полный:
```
kubectl config view
```

Текущий контекст
```
kubectl config use-context devel
kubectl config view --minify
kubectl config use-context devel-ext
kubectl config view --minify
```

Конфиг для qa, с полными данными и доступом снаружи
```
kubectl config use-context qa-ext
kubectl config view --minify --raw
```

## 7. Выгрузите конфиг контекста qa-ext на sbox.slurm.io и проверьте доступы в кластер.

```
kubectl config use-context qa-ext
kubectl config view --minify --raw > config-qa-ext

scp config-qa-ext s<номер_студента>@sbox.slurm.io:~/.kube/config

ssh s<номер_студента>@sbox.slurm.io
kubectl get pod
kubectl get pod -n dev
kubectl get pod -n prod
kubectl create cm test --from-literal=key=data -n prod # ошибка
```

Ответы на задачи со звездочкой будут выложены в субботу.

## 8. Задача со звездочкой

На базе роли edit создайте роль edit-nosecrets, которая не позволяет изменять секреты, а позволяет только просматривать. 
Назначьте её serviceAccount qa для доступа в неймспейс dev.

## 9. Задача с двумя звездочками

Сгенерируйте сертификат и подпишите его корневым ключом кластера для авторизации пользователя
из группы `system:masters`
