# Домашнее задание к занятию «Сетевое взаимодействие в Kubernetes» - Выполнил Shestovskikh Daniil

### Примерное время выполнения задания

120 минут

### Цель задания

Научиться настраивать доступ к приложениям в Kubernetes:
- Внутри кластера через **Service** (ClusterIP, NodePort).
- Снаружи кластера через **Ingress**.

Это задание поможет вам освоить базовые принципы сетевого взаимодействия в Kubernetes — ключевого навыка для работы с кластерами.
На практике Service и Ingress используются для доступа к приложениям, балансировки нагрузки и маршрутизации трафика. Понимание этих механизмов поможет вам упростить управление сервисами в рабочих окружениях и снизит риски ошибок при развёртывании.

------

## **Подготовка**
### **Чеклист готовности**
- Установлен Kubernetes (MicroK8S, Minikube или другой).
- Установлен `kubectl`.
- Редактор для YAML-файлов (VS Code, Vim и др.).

------

### Инструменты, которые пригодятся для выполнения задания

1. [Инструкция](https://microk8s.io/docs/getting-started) по установке MicroK8S.
2. [Инструкция](https://minikube.sigs.k8s.io/docs/start/?arch=%2Fwindows%2Fx86-64%2Fstable%2F.exe+download) по установке Minikube. 
3. [Инструкция](https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/)по установке kubectl.
4. [Инструкция](https://marketplace.visualstudio.com/items?itemName=ms-kubernetes-tools.vscode-kubernetes-tools) по установке VS Code

### Дополнительные материалы, которые пригодятся для выполнения задания

1. [Описание](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) Deployment и примеры манифестов.
2. [Описание](https://kubernetes.io/docs/concepts/services-networking/service/) Описание Service.
3. [Описание](https://kubernetes.io/docs/concepts/services-networking/ingress/) Ingress.
4. [Описание](https://github.com/wbitt/Network-MultiTool) Multitool.

------

## **Задание 1: Настройка Service (ClusterIP и NodePort)**

### **Файлы манифестов Задания 1:**
- [deployment-multi-container.yaml](https://github.com/Dun9Dev/kuber-hw/blob/hw3/deployment-multi-container.yaml)
- [service-clusterip.yaml](https://github.com/Dun9Dev/kuber-hw/blob/hw3/service-clusterip.yaml)
- [service-nodeport.yaml](https://github.com/Dun9Dev/kuber-hw/blob/hw3/service-nodeport.yaml)

### **Шаги выполнения**
1. **Создать Deployment** с двумя контейнерами (nginx и multitool), 3 реплики:
   ```bash
   kubectl apply -f deployment-multi-container.yaml
   ```
   ![Скриншот 1 - создание Deployment и проверка подов](https://github.com/Dun9Dev/kuber-hw/blob/hw3/img/Screenshot_20260319_153026.png)

2. **Создать Service типа ClusterIP** (порты 9001 для nginx, 9002 для multitool) и проверить доступ из временного пода:
   ```bash
   kubectl apply -f service-clusterip.yaml
   kubectl run test-pod --image=wbitt/network-multitool --rm -it -- sh
   curl multi-app-clusterip:9001
   curl multi-app-clusterip:9002
   ```
   ![Скриншот 2 - создание ClusterIP и проверка доступа](https://github.com/Dun9Dev/kuber-hw/blob/hw3/img/Screenshot_20260319_153616.png)

3. **Создать Service типа NodePort** для доступа к nginx снаружи (порт 30080) и проверить доступ с локального компьютера:
   ```bash
   kubectl apply -f service-nodeport.yaml
   kubectl get nodes -o wide
   curl http://192.168.0.243:30080
   ```
   ![Скриншот 3 - создание NodePort и проверка доступа](https://github.com/Dun9Dev/kuber-hw/blob/hw3/img/Screenshot_20260319_154144.png)

---
## **Задание 2: Настройка Ingress**

### **Файлы манифестов Задания 2:**
- [deployment-frontend.yaml](https://github.com/Dun9Dev/kuber-hw/blob/hw3/deployment-frontend.yaml)
- [deployment-backend.yaml](https://github.com/Dun9Dev/kuber-hw/blob/hw3/deployment-backend.yaml)
- [service-frontend.yaml](https://github.com/Dun9Dev/kuber-hw/blob/hw3/service-frontend.yaml)
- [service-backend.yaml](https://github.com/Dun9Dev/kuber-hw/blob/hw3/service-backend.yaml)
- [ingress.yaml](https://github.com/Dun9Dev/kuber-hw/blob/hw3/ingress.yaml)

### **Шаги выполнения**
1. **Развернуть два Deployment** (frontend на nginx, backend на multitool) и создать для них Service:
   ```bash
   kubectl apply -f deployment-frontend.yaml
   kubectl apply -f deployment-backend.yaml
   kubectl apply -f service-frontend.yaml
   kubectl apply -f service-backend.yaml
   ```

2. **Включить Ingress-контроллер** (выполнено на рабочей ВМ):
   ```bash
   microk8s enable ingress
   ```

3. **Создать Ingress**, который направляет `/` на frontend, а `/api` на backend, и проверить доступ:
   ```bash
   kubectl apply -f ingress.yaml
   kubectl get ingress
   curl -H "Host: example.com" http://192.168.0.243/
   curl -H "Host: example.com" http://192.168.0.243/api
   ```
   ![Скриншот 4 - создание Ingress и проверка доступа](https://github.com/Dun9Dev/kuber-hw/blob/hw3/img/Screenshot_20260319_154658.png)

---
## Шаблоны манифестов с учебными комментариями
### **1. Deployment (nginx + multitool)**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: # ПРИМЕР: "multi-container-app"
spec:
  replicas: # ЗАДАНИЕ: Укажите количество реплик
  selector:
    matchLabels:
      app: # ДОПОЛНИТЕ: Метка для селектора
  template:
    metadata:
      labels:
        app: # ПОВТОРИТЕ метку из selector.matchLabels
    spec:
      containers:
 - name: # ЗАДАНИЕ: Название первого контейнера
        image: nginx
        ports:
 - containerPort: 80
 - name: multitool
        image: wbitt/network-multitool
        ports:
 - containerPort: 8080
        env:
 - name: HTTP_PORT
          value: "8080" # КЛЮЧЕВОЙ МОМЕНТ: Порт должен совпадать с containerPort
```
### **2. Ingress (для frontend и backend)**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: # ЗАДАНИЕ: Придумайте имя, допустим example-ingress
  annotations:  # ВАЖНО: Эта аннотация нужна для rewrite правил
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
 - http:
      paths:
 - path: /
        pathType: Prefix
        backend:
          service:
            name: # УКАЖИТЕ: Имя frontend Service
            port:
              number: 80
 - path: /api # КЛЮЧЕВОЙ ПУТЬ: API endpoint
        pathType: Prefix
        backend:
          service:
            name: # УКАЖИТЕ: Имя backend Service
            port:
              number: 80
```
---

## **Правила приёма работы**
1. Домашняя работа оформляется в своём Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl` и скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.

## **Критерии оценивания задания**
1. Зачёт: Все задачи выполнены, манифесты корректны, есть доказательства работы (скриншоты).
2. Доработка (на доработку задание направляется 1 раз): основные задачи выполнены, при этом есть ошибки в манифестах или отсутствуют проверочные скриншоты.
3. Незачёт: работа выполнена не в полном объёме, есть ошибки в манифестах, отсутствуют проверочные скриншоты. Все попытки доработки израсходованы (на доработку работа направляется 1 раз). Этот вид оценки используется крайне редко.

## **Срок выполнения задания**  
1. 5 дней на выполнение задания.
2. 5 дней на доработку задания (в случае направления задания на доработку).
```
