#### University: [ITMO University](https://itmo.ru/ru/)
#### Faculty: [FPIN](https://fict.itmo.ru)
#### Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)
#### Year: 2024/2025
#### Group: K4111c
#### Author: Ibryaeva Alina Vadimovna
#### Lab: Lab3
#### Date of create: 24.11.2024
#### Date of finished: 28.11.2024

---

# Лабораторная работа №3 "Сертификаты и "секреты" в Minikube, безопасное хранение данных."

## Описание
В данной лабораторной работе вы познакомитесь с сертификатами и "секретами" в Minikube, правилами безопасного хранения данных в Minikube.

## Цель работы
Познакомиться с сертификатами и "секретами" в Minikube, правилами безопасного хранения данных в Minikube.

---

## Ход работы

### 1. Создание ConfigMap
Запущен minikube командой `minikube start`.

Создан манифест для ConfigMap:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: frontend-configmap
data:
  react_app_user_name: "Alina"
  react_app_company_name: "ITMO_MASTERS"
```
Ключи `react_app_user_name` и `react_app_company_name` имеют значения для переменных REACT_APP_USERNAME и REACT_APP_COMPANY_NAME, соответственно.

В директории с .yaml файлом и выполнена команда `kubectl create -f lab3-configmap.yaml`.

Проверено, что появился ConfigMap командой `kubectl get configmaps`.

![configmap](https://github.com/ghhbdtn/2024_2025-introduction_to_distributed_technologies-k4111c-ibryaeva_a_v/blob/master/lab3/images/configmaps.png 'configmap')

### 2. Создание ReplicaSet
Создан манифест для ReplicaSet:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend-replicaset
  labels:
    app: lab3-frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: lab3-frontend
  template:
    metadata:
      labels:
        app: lab3-frontend
    spec:
      containers:
        - name: frontend-container
          image: ifilyaninitmo/itdt-contained-frontend:master
          ports:
            - containerPort: 3000
          env:
            - name: REACT_APP_USERNAME
              valueFrom:
                configMapKeyRef:
                  name: frontend-configmap
                  key: react_app_user_name
            - name: REACT_APP_COMPANY_NAME
              valueFrom:
                configMapKeyRef:
                  name: frontend-configmap
                  key: react_app_company_name
```

Шаблон пода задается в объекте `Template`. С помощью свойства `env` внутри подов объявлены переменные окружения `REACT_APP_USERNAME` и `REACT_APP_COMPANY_NAME`, а их значения берутся из ConfigMap frontend-configmap, который создан на предыдущем этапе.

Создан контроллер командой - `kubectl create -f lab3-replicaset.yaml`.

Проверено, что появился контроллер командой `kubectl get rs`.

![replicaset](https://github.com/ghhbdtn/2024_2025-introduction_to_distributed_technologies-k4111c-ibryaeva_a_v/blob/master/lab3/images/replicasets.png 'replicaset')

### 3. Создание сервиса
Для создания Inngress ресурса потребуется сервис, поэтому написан манифест для сервиса:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
  labels:
    app: lab3-frontend
spec:
  type: NodePort
  selector:
    app: lab3-frontend
  ports:
    - protocol: TCP
      port: 3000
      targetPort: 3000
      nodePort: 30555
```

> Доступны только порты 30000–32767, пусть `nodePort` 30555.

Сервис создан командой `kubectl create -f lab3-service.yaml`. 
Проверено, что появился сервис командой `kubectl get services`.

![service](https://github.com/ghhbdtn/2024_2025-introduction_to_distributed_technologies-k4111c-ibryaeva_a_v/blob/master/lab3/images/services.png 'service')

### 4. Генерация TLS
Для генерации TLS сертификата использована утилита [OpenSSL](https://losst.pro/sozdanie-sertifikata-openssl).

Сгенерированы приватный ключ и сертификат командой `openssl req -new -newkey rsa:4096 -x509 -sha256 -days 365 -nodes -out lab3-frontend.crt -keyout lab3-frontend.key -subj "/CN=lab3-frontend.iav"`. 
Опция `-out` указывает на имя файла для сохранения ключа, а число `2048` - размер ключа в битах (по умолчанию 512).

В опции `-subj` указано доменное имя, по которому с помощью Ingress, мы будем заходить на сервер - `lab3-frontend.iav`.
![Key](https://github.com/ghhbdtn/2024_2025-introduction_to_distributed_technologies-k4111c-ibryaeva_a_v/blob/master/lab3/images/openssl.png 'key')
### 5. Создание Secret
Создаем секрет командой - `kubectl -- create secret tls lab3-frontend-tls --key lab3-frontend.key --cert lab3-frontend.crt`.

![secret](https://github.com/ghhbdtn/2024_2025-introduction_to_distributed_technologies-k4111c-ibryaeva_a_v/blob/master/lab3/images/secret.png 'secret')

### 6. Создание Ingress

Подключен Ingress в minikube:

```
minikube addons enable ingress
minikube addons enable ingress-dns
```

Создан манифест для Ingress:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: frontend-ingress
spec:
  tls:
    - hosts:
        - lab3-frontend.iav
      secretName: lab3-frontend-tls
  rules:
    - host: lab3-frontend.iav
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-service
                port:
                  number: 3000
```

В полях **hosts** и **host** указано **FQDN** - `lab3-frontend.iav`.

При активации Ingress было написано:

> After the addon is enabled, please run "minikube tunnel" and your ingress resources would be available at "127.0.0.1"

Соответственно, добавляем **IP адрес ingress** и **FQDN**, то есть `127.0.0.1 lab3-frontend.iav` в [hosts файл](), который лежит по пути: `C:\Windows\System32\drivers\etc`.

![hosts](https://github.com/ghhbdtn/2024_2025-introduction_to_distributed_technologies-k4111c-ibryaeva_a_v/blob/master/lab3/images/localhost.png 'hosts')

Создана точка входа в кластер minikube командой `kubectl create -f frontend-ingress.yaml`.

Проверено, что появился Ingress командой `kubectl get ingress`.

![ingress create](https://github.com/ghhbdtn/2024_2025-introduction_to_distributed_technologies-k4111c-ibryaeva_a_v/blob/master/lab3/images/ingresses.png 'ingress create')

Подключились к Ingress командой `minikube tunnel`.

При открытии страницы `https://lab3-frontend.iav/`, видны **параметры**, переданные через ConfigMap:

![web](https://github.com/ghhbdtn/2024_2025-introduction_to_distributed_technologies-k4111c-ibryaeva_a_v/blob/master/lab3/images/web.png 'web')

Данные сертификата:

![cert check](https://github.com/ghhbdtn/2024_2025-introduction_to_distributed_technologies-k4111c-ibryaeva_a_v/blob/master/lab3/images/cert.png 'cert check')

### Диаграмма
![diagram](https://github.com/ghhbdtn/2024_2025-introduction_to_distributed_technologies-k4111c-ibryaeva_a_v/blob/master/lab3/images/diagram.png 'diagram')
