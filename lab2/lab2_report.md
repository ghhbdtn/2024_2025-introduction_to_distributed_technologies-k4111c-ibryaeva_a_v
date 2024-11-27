#### University: [ITMO University](https://itmo.ru/ru/)
#### Faculty: [FPIN](https://fict.itmo.ru)
#### Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)
#### Year: 2024/2025
#### Group: K4111c
#### Author: Ibryaeva Alina Vadimovna
#### Lab: Lab2
#### Date of create: 24.11.2024
#### Date of finished: xx.xx.2024

---

# Лабораторная работа №2 "Развертывание веб сервиса в Minikube, доступ к веб интерфейсу сервиса. Мониторинг сервиса."

## Описание
В данной лабораторной работе вы познакомитесь с развертыванием полноценного веб сервиса с несколькими репликами.

## Цель работы
Ознакомиться с типами "контроллеров" развертывания контейнеров, ознакомится с сетевыми сервисами и развернуть свое веб приложение.

---

## Ход работы

### 1. Создание контейнера frontend-container
Скачан образ `ifilyaninitmo/itdt-contained-frontend:master` командой `docker pull ifilyaninitmo/itdt-contained-frontend:master`.

Проверено, что скачанный образ появился командой `docker images`.

![Образ itdt-contained-frontend]( 'Образ itdt-contained-frontend')

Создаем контейнер на основе полученного образа - `docker run -d --name frontend-container ifilyaninitmo/itdt-contained-frontend:master`.

Проверено, что контейнер создан - `docker ps -a`.

![Контейнер frontend-container]( 'Контейнер frontend-container')

### 2. Создание Deployment

Запускаем minikube - `minikube start`.

Создан манифест для развертывания:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  labels:
    app: frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend_pod
  template:
    metadata:
      labels:
        app: frontend_pod
    spec:
      containers:
        - name: frontend-container
          image: ifilyaninitmo/itdt-contained-frontend:master
          ports:
            - containerPort: 3000
          env:
            - name: REACT_APP_USERNAME
              value: Alina
            - name: REACT_APP_COMPANY_NAME
              value: ITMO_MASTERS
```

Для запуска 2-х экземпляров пода, использовано свойство-значение `replicas: 2`.

Задан шаблон пода в объекте `template`.
С помощью свойства `env` объявлены внутри подов переменные окружения `REACT_APP_USERNAME` и `REACT_APP_COMPANY_NAME` со значениями `Alina` и `ITMO_MASTERS`, соответственно.

В директории с .yaml-файлом выполнена команда `kubectl create -f lab2.yaml`.

Проверено, что появился `deployment` командой `kubectl get deployments`.

![frontend deployment]( 'frontend deployment')

### 3. Создание сервиса frontend-service

Создан сервис для доступа к развертыванию - `minikube kubectl -- expose deployment frontend --port=3000 --target-port=3000 --name=frontend-service --type=LoadBalancer`.

Тип сервиса `LoadBalancer` будет решать задачу балансировки нагрузки между подами.

![frontend service]( 'frontend service')

Проброшен локальный порт на порт контейнера - `minikube kubectl -- port-forward service/frontend-service 3000:3000`.

![Port-forward]( 'Port-forward')

Перешли по ссылке `http://localhost:3000`:

![Frontend page]( 'Frontend page')

### 4. Логи подов

Получили список всех подов - `minikube kubectl get pods`. Как и ожидалось, deployment запустил 2 пода.

![Pods]( 'Pods')

Получены логи первого пода командой `minikube kubectl -- logs pod/frontend-6545595896-lx78l`.

![Log pod 1]( 'Log pod 1')

Получены логи второго пода командой `minikube kubectl -- logs pod/frontend-6545595896-x8c62`.

![Log pod 2]( 'Log pod 2')

### Вывод
1. Ресурс deployment позволяет создавать несколько подов на основе одного контейнера.
2. Сервис типа LoadBalancer - эта абстракция, которая позволяет нам воспринимать группу подов (с одинаковой меткой) как единую сущность и работать с ними, используя сервис как единую точку доступа к ним.

   Именно поэтому логи первого и второго пода идентичны.

### Диаграмма

