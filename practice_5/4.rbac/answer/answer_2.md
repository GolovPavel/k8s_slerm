## 8. Задача со звездочкой

На базе роли edit создайте роль edit-nosecrets, которая не позволяет изменять секреты, а позволяет только просматривать. 
Назначьте её serviceAccount qa для доступа в неймспейс dev.

```
kubectl get clusterrole edit -o yaml > edit1.yml
vim edit1.yml
```

Удаляем слово secrets в правиле:

```
- apiGroups:
  - ""
  resources:
  - configmaps
  - endpoints
  - persistentvolumeclaims
  - replicationcontrollers
  - replicationcontrollers/scale
--- - secrets
  - serviceaccounts
  - services
  - services/proxy
  verbs:
  - create
  - delete
  - deletecollection
  - patch
  - update
```

Меняем название роли и удаляем лишние поля
```
--- aggregationRule:
---   clusterRoleSelectors:
---   - matchLabels:
---       rbac.authorization.k8s.io/aggregate-to-edit: "true"
metadata:
  name: edit-nosecrets
---  annotations:
---    rbac.authorization.kubernetes.io/autoupdate: "true"
---  creationTimestamp: "2020-05-24T12:40:15Z"
---  resourceVersion: "332"
---  selfLink: /apis/rbac.authorization.k8s.io/v1/clusterroles/edit
---  uid: fd9ccc99-f985-49e0-8f69-5dfd00ea60db
---  labels:
---    kubernetes.io/bootstrapping: rbac-defaults
---    rbac.authorization.k8s.io/aggregate-to-admin: "true"
```

Пересоздаем rolebinding
```
kubectl delete rolebindings.rbac.authorization.k8s.io qa-dev -n dev
kubectl create rolebinding qa-dev --namespace dev --serviceaccount prod:qa --clusterrole edit-nosecrets
```

Проверяем, должно выдать ошибку на обе команды:
```
kubectl config use-context qa
kubectl -n dev create secret generic db2 --from-literal=k=d
kubectl -n dev delete secret generic db
```

## 9. Задача с двумя звездочками

Сгенерируйте сертификат и подпишите его корневым ключом кластера для авторизации пользователя
из группы `system:masters`

```
openssl genrsa -out admin.key 2048
openssl req -new -key admin.key -out admin.csr -subj "/CN=admin/O=system:masters"  # Генерим CSR
openssl x509 -req -in admin.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out admin.crt -days 3

kubectl config set-credentials admin --client-certificate=admin.crt --client-key=admin.key --embed-certs
kubectl config set-context admin --cluster=kubernetes --user=admin
```

Проверяем, должно работать:
```
kubectl config use-context admin
kubectl config view --minify --raw

kubectl get pod -A
```
