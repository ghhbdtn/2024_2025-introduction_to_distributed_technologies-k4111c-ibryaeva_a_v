#### University: [ITMO University](https://itmo.ru/ru/)
#### Faculty: [FPIN](https://fict.itmo.ru)
#### Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)
#### Year: 2024/2025
#### Group: K4111c
#### Author: Ibryaeva Alina Vadimovna
#### Lab: Lab1
#### Date of create: 24.11.2024
#### Date of finished: xx.xx.2024

---

# Лабораторная работа №1 "Установка Docker и Minikube, мой первый манифест."

## Описание
Это первая лабораторная работа в которой вы сможете протестировать Docker, установить Minikube и развернуть свой первый "под".

## Цель работы
Ознакомиться с инструментами Minikube и Docker, развернуть свой первый "под".

---

## Ход работы

### 1. Установлен Docker на рабочий компьютер

### 2. Установлен Minikube используя оригинальную инструкцию

### 3. Создание контейнера vault
Скачан образ vault командой `docker pull hashicorp/vault`.  
Проверено, что образ vault появился - `docker images`.

![Образ vault]( 'Образ vault')

Создан контейнер на основе образа vault - `docker run -d hashicorp/vault`.  
Проверено, что контейнер vault появился - `docker ps -a`.

![Контейнер vault]( 'Контейнер vault')

### 4. Создание Pod'a
Запущен minikube командой `minikube start`.  
Проверено, что появился узел - `kubectl get nodes`.

Создан **manifest file**, в котором будет описан под:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: vault
  labels:
    environment: dev
    tier: vault
spec:
  containers:
    - name: vault
      image: hashicorp/vault
      ports:
        - containerPort: 8200
```
В директории с .yaml-файлом выполнена команда `kubectl create -f firstmanifest.yaml`.

Проверено, что Pod появился - `kubectl get pods`.

![Pod vault]( 'Pod vault')

### 5. Создание сервиса
Создаем сервис для доступа к Pod - `minikube kubectl -- expose pod vault --type=NodePort --port=8200`.

![Service vault]( 'Service vault')

Перенаправляем трафик с Pod на локальный - `minikube kubectl -- port-forward service/vault 8200:8200`.

![Port-forward]( 'Port-forward')

Перешли на страницу авторизации Vault `http://localhost:8200`:

![Vault page]( 'Vault page')

### 6. Поиск токена
Чтобы найти токен для авторизации, открываем второй терминал и используем команду `minikube kubectl -- logs service/vault`.

Получили следующий токен:

![Successful authorization]( 'Successful authorization')

Работа выполнена - остановили узел командой `minikube stop`.

### Диаграмма

