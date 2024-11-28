#### University: [ITMO University](https://itmo.ru/ru/)
#### Faculty: [FPIN](https://fict.itmo.ru)
#### Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)
#### Year: 2024/2025
#### Group: K4111c
#### Author: Ibryaeva Alina Vadimovna
#### Lab: Lab4
#### Date of create: 24.11.2024
#### Date of finished: xx.xx.2024

---

# Лабораторная работа №4 "Сети связи в Minikube, CNI и CoreDNS"

## Описание
Это последняя лабораторная работа в которой вы познакомитесь с сетями связи в Minikube. Особенность Kubernetes заключается в том, что у него одновременно работают underlay и overlay сети, а управление может быть организованно различными CNI.

## Цель работы
Познакомиться с CNI Calico и функцией IPAM Plugin, изучить особенности работы CNI и CoreDNS.

---

## Ход работы

### 1. Calico и Multi-Node Clusters
При запуске minikube устанавливаем [плагин](https://projectcalico.docs.tigera.io/getting-started/kubernetes/minikube) `CNI=calico`, режим работы `Multi-Node Clusters` и разворачиваем [2 Node](https://minikube.sigs.k8s.io/docs/tutorials/multi_node/) командой:

```
minikube start --network-plugin=cni --cni=calico --nodes 2 -p multinode-demo
```

Проверяем, что запустились 2 Node: `kubectl get nodes`.

![get_nodes](https://github.com/ghhbdtn/2024_2025-introduction_to_distributed_technologies-k4111c-ibryaeva_a_v/blob/master/lab4/images/get_nodes.png 'get_nodes')

Чтобы проверить работу CNI Calico, посмотрим Pod'ы с меткой **calico-node**:
```
kubectl get pods -l k8s-app=calico-node -A
```

![get_pods](https://github.com/ghhbdtn/2024_2025-introduction_to_distributed_technologies-k4111c-ibryaeva_a_v/blob/master/lab4/images/get_pods.png 'get_pods')

### 2. calicoctl и IPPool
Для назначения IP адресов в Calico необходимо написать манифест для **IPPool** ресурса.

С помощью IPPool можно создать **IP-pool (блок IP-адресов)**, который выделяет IP-адреса только для узлов с определенной **меткой (label)**.

Чтобы назначить метки узлам, используем следующие команды, задав метки, согласно географическому признаку:
```
kubectl label nodes multinode-demo zone=east  
kubectl label nodes multinode-demo-m02 zone=west
```

Написан манифест IPPool:
```yaml
apiVersion: projectcalico.org/v3
kind: IPPool
metadata:
   name: zone-east-ippool
spec:
   cidr: 192.168.0.0/24
   ipipMode: Always
   natOutgoing: true
   nodeSelector: zone == "east"
---
apiVersion: projectcalico.org/v3
kind: IPPool
metadata:
   name: zone-west-ippool
spec:
   cidr: 192.168.1.0/24
   ipipMode: Always
   natOutgoing: true
   nodeSelector: zone == "west"
```

Чтобы применить манифест для IPPool, надо установить **calicoctl**, для этого скачаем [config-файл](https://github.com/projectcalico/calico/blob/master/manifests/calicoctl.yaml) с официального репозитория и выполним команду: `kubectl create -f calicoctl.yaml`.

Перед тем, как добавить собственные IPPool'ы, проверим созданные по-умолчанию:
```
kubectl exec -i -n kube-system calicoctl -- /calicoctl --allow-version-mismatch get ippools -o wide
```

Удаляем IPPool по-умолчанию:
```
kubectl delete ippools default-ipv4-ippool
```

Созданы IPPool'ы следующей командой:
```
kubectl exec -i -n kube-system calicoctl -- /calicoctl --allow-version-mismatch create -f - < lab4-ippool.yaml
```
![create_ippools]( 'create_ippools')

Проверено, что появилось два pool'а командой:
```
kubectl exec -i -n kube-system calicoctl -- /calicoctl --allow-version-mismatch get ippool -o wide
```

![ippools](https://github.com/ghhbdtn/2024_2025-introduction_to_distributed_technologies-k4111c-ibryaeva_a_v/blob/master/lab4/images/ippools.png 'ippools')

### 3. Deployment и Service
Манифест для развертывания берем из 2 лабораторной работы и заменяем метку на `lab4-frontend`.

Также написан манифест сервиса:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: lab4-service
spec:
  selector:
    app: lab4-frontend
  ports:
    - port: 3000
      targetPort: 3000
  type: LoadBalancer
```

В директории с .yaml файлом выполнена команда:
```
kubectl apply -f lab4-deployment.yaml -f lab4-service.yaml
```

Проверено, что появились развертывание и сервис: `kubectl get deployments`, `kubectl get services`.

![create_deployment_and_service](https://github.com/ghhbdtn/2024_2025-introduction_to_distributed_technologies-k4111c-ibryaeva_a_v/blob/master/lab4/images/ippools.png 'create_deployment_and_service')

![create_deployment_and_service](https://github.com/ghhbdtn/2024_2025-introduction_to_distributed_technologies-k4111c-ibryaeva_a_v/blob/master/lab4/images/services.png 'create_deployment_and_service')

Проверены IP-адреса созданных Pod'ов: `kubectl get pods -o wide`.

![pods_ip](https://github.com/ghhbdtn/2024_2025-introduction_to_distributed_technologies-k4111c-ibryaeva_a_v/blob/master/lab4/images/pods_ip.png 'pods_ip')

### 3. Проброс порта
Проброшен порт для подключения к сервису через браузер командой: `kubectl port-forward service/lab4-service 8200:3000`.

При переходе по ссылке: `http://localhost:8200/`, видим следующее:

![browser](https://github.com/ghhbdtn/2024_2025-introduction_to_distributed_technologies-k4111c-ibryaeva_a_v/blob/master/lab4/images/web.png 'browser')

### 4. Ping

Выполнена команда 'ping' с контейнера `lab4-deployment-757648f768-9d5c4` контейнеру с IP-адресом: `ping 192.168.0.69` с помощью команды:

```
kubectl exec -ti lab4-deployment-757648f768-9d5c4 -- sh
```

![ping_1](https://github.com/ghhbdtn/2024_2025-introduction_to_distributed_technologies-k4111c-ibryaeva_a_v/blob/master/lab4/images/ping1.png 'ping_1')

Чтобы выйти из контейнера, использована команда `exit`.

Выполнена команда 'ping' с контейнера `lab4-deployment-757648f768-p5qq8` контейнеру с IP-адресом: `ping 192.168.1.195` с помощью команды:

```
kubectl exec -ti lab4-deployment-757648f768-p5qq8 -- sh
```

![ping_2](https://github.com/ghhbdtn/2024_2025-introduction_to_distributed_technologies-k4111c-ibryaeva_a_v/blob/master/lab4/images/ping2.png 'ping_2')

### Диаграмма
![diagram](https://github.com/ghhbdtn/2024_2025-introduction_to_distributed_technologies-k4111c-ibryaeva_a_v/blob/master/lab4/images/diagram.png 'diagram')
