# Домашнее задание к занятию «Сетевое взаимодействие в K8S. Часть 1»

## Цель задания

Научиться создавать и работать с сетевыми ресурсами в Kubernetes.

## Чеклист готовности к домашнему заданию

1.  Установленное k8s-решение (например, MicroK8S).
2.  Установленный локальный kubectl.
3.  Редактор YAML-файлов с подключённым git-репозиторием.
x
## Задание 1. Создать Deployment и обеспечить доступ к контейнерам приложения по разным портам из другого Pod внутри кластера

### 1. Создание Deployment

Манифест [deployment.yaml](task1/deployment.yaml):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-multitool
  labels:
    app: nginx-multitool
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-multitool
  template:
    metadata:
      labels:
        app: nginx-multitool
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
      - name: multitool
        image: wbitt/network-multitool:latest
        env:
        - name: HTTP_PORT
          value: "8080"
        ports:
        - containerPort: 8080
```

Результат применения:
```bash
kubectl get pods -l app=nginx-multitool
NAME                               READY   STATUS    RESTARTS   AGE
nginx-multitool-5bc776b6c6-9drsr   2/2     Running   0          15s
nginx-multitool-5bc776b6c6-lmzvx   2/2     Running   0          28s
nginx-multitool-5bc776b6c6-vb2gh   2/2     Running   0          42s
```

### 2. Создание Service

Манифест [service.yaml](task1/service.yaml):

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-multitool-svc
spec:
  selector:
    app: nginx-multitool
  ports:
    - name: nginx
      port: 9001
      targetPort: 80
    - name: multitool
      port: 9002
      targetPort: 8080
```

### 3. Проверка доступности

Создание тестового пода:
```bash
kubectl run test-multitool --image=wbitt/network-multitool
```

Проверка доступа к nginx (порт 9001):
```bash
kubectl exec test-multitool -- curl nginx-multitool-svc:9001
```
```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
</html>
```

Проверка доступа к multitool (порт 9002):
```bash
kubectl exec test-multitool -- curl nginx-multitool-svc:9002
```
```
WBITT Network MultiTool (with NGINX) - nginx-multitool-5bc776b6c6-vb2gh - 10.1.46.11 - HTTP: 8080 , HTTPS: 443
```

## Задание 2. Создать Service и обеспечить доступ к приложениям снаружи кластера

### 1. Создание NodePort Service

Манифест [service-nodeport.yaml](task2/service-nodeport.yaml):

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-multitool-nodeport
spec:
  type: NodePort
  selector:
    app: nginx-multitool
  ports:
    - name: nginx
      port: 80
      targetPort: 80
      nodePort: 30080
```

Результат создания:
```bash
kubectl get service nginx-multitool-nodeport
NAME                       TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
nginx-multitool-nodeport   NodePort   10.152.183.187   <none>        80:30080/TCP   9s
```

### 2. Проверка доступности снаружи кластера

```bash
curl 10.0.2.15:30080
```
```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
</html>
```

## Выполненные задачи

1. Создан Deployment с 3 репликами и 2 контейнерами
2. Создан Service для доступа к контейнерам по портам 9001 и 9002
3. Проверен доступ к обоим контейнерам через curl
4. Создан NodePort Service для доступа извне кластера
5. Проверен доступ к приложению извне через NodePort
