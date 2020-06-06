**!!!Всю практику выполняем на своем `master-1.sXXXXXX` ПОД РУТОМ (sudo -i) , предварительно зайдя на него по SSH!!!**

# Разбираемся с DNS в кластере

0) На `master-1` клонируем git-репо:

```
cd /srv
git clone git@gitlab.slurm.io:school/slurm.git

cd slurm/8.DNS_Services_Ingress/
```

1) Запускаем Deployment с Centos и заходим в его Pod:

```
kubectl run centos --image=centos:7 -it bash
```

2) Внутри Pod'а устанавливаем необходимые для практики пакеты:

```
yum install bind-utils tcpdump screen -y
```

3) Запускаем `screen`, переходим в `bash` и запускаем `tcpdump` на 53 порту:

```
screen
bash
tcpdump -n -p -i eth0 port 53
Ctrl+A+D
```

4) В основной оболочке `bash` Pod'а делаем DNS-запрос:

```
nslookup yandex.ru
```

5) Возвращаемся в `screen` и видим множество пакетов. Останавливаем tcpdump и выходим из `screen`:

```
screen -x
Ctrl+C
Ctrl+A+D
```

6) Пофиксим эту проблему. Для этого нам надо выйти из Pod'а и открыть на редактирование Configmap CoreDNS:

```
exit
kubectl edit cm -n kube-system coredns
```

7) В открывшемся файле меняем

```bash
pods insecure
```

на

```bash
pods verified
```

и дописываем

```bash
autopath @kubernetes
```

**В итоге должно получиться**

```bash
Corefile: |
  .:53 {
      errors
      health
      ready
      autopath @kubernetes  <---
      kubernetes cluster.local in-addr.arpa ip6.arpa {
         pods verified  <---
         fallthrough in-addr.arpa ip6.arpa
         ttl 30
      }
      prometheus :9153
      forward . /etc/resolv.conf
      cache 30
      loop
      reload
      loadbalance
  }
```

8) Сохраняем Configmap, ждем пока Pod CoreDNS сделает reload конфига(1-2 минуты). Затем заходим в Pod c Centos и повторяем эксперимент. Видим что проблема пофиксилась:

```
*Если установленные ранее пакеты пропали, значит Pod был рестартован. Нужно просто установить их заново*

kubectl exec *имя-пода* -it bash

screen -x
tcpdump -n -p -i eth0 port 53
Ctrl+A+D

nslookup google.com
screen -x

Ctrl+A+D

exit
```

9) Прибираемся за собой:

```
kubectl delete deploy centos
```

# Смотрим на Service'ы Kubernetes'а

1) Деплоим "основное" приложение

```
cd slurm/8.DNS_Services_Ingress/

kubectl apply -f app
```

2) Запустим тестовое приложение в namespace'е `test`, с которого мы будем обращаться к основному:

```
kubectl create ns test

kubectl run test -n test --image=amouat/network-utils -it bash

exit
```

3) Создаем Service типа ClusterIP:

```
kubectl apply -f clusterip.yaml
```

4) Убедимся, что Service работает. Узнаем его IP, зайдем внутрь нашего тестового Pod'а и обратимся к основному приложению, используя имя сервиса и IP:

```
kubectl get svc
kubectl exec -n test *имя-пода* -it bash

curl *ip-адрес сервиса*
curl my-service.default

exit
```

5) Создаем Service типа Nodeport:

```
kubectl apply -f nodeport.yaml
```

6) Проверяем что все ОК. Смотрим наши Service'ы, находим NodePort. Фиксируем какой порт нам открылся и проверяем работу Service'а:

```
kubectl get svc

curl node-1.s<ваш номер логина>.slurm.io:<ваш номер порта>

curl master-1.s<ваш номер логина>.slurm.io:<ваш номер порта>
```

7) Создаем Service LoadBalancer:

```
kubectl create -f loadbalancer.yaml

kubectl get svc
```

8) Подчищаем за собой:

```
kubectl delete svc my-service-lb my-service-np

kubectl delete ns test
```

# Разбираемся с Ingress'ами

1) Создадим Ingress без указания хоста:

```
kubectl apply -f nginx-ingress.yaml
kubectl get ing
```

2) Попробуем покурлить разные домены:

```
curl my.sXXXXXX.edu.slurm.io <- Подставляем свой номер логина

curl notmy.sXXXXXX.edu.slurm.io 
```

**САМОСТОЯТЕЛЬНАЯ РАБОТА:**
- Подправить Ingress таким образом, чтобы он работал только на домене `my.sXXXXXX.edu.slurm.io`

Правильный ответ лежит в `right_answers/`

# Переходим к установке `cert-manager`

```
cd cert-manager
cat README.md
```
