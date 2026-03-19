# Домашнее задание к занятию «Запуск приложений в K8S» - Выполнил Shestovakik Daniil

### Цель задания

В тестовой среде для работы с Kubernetes, установленной в предыдущем ДЗ, необходимо развернуть Deployment с приложением, состоящим из нескольких контейнеров, и масштабировать его.

------

### Чеклист готовности к домашнему заданию

1. Установленное k8s-решение (например, MicroK8S).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключённым git-репозиторием.

------

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Описание](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) Deployment и примеры манифестов.
2. [Описание](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/) Init-контейнеров.
3. [Описание](https://github.com/wbitt/Network-MultiTool) Multitool.

------

### Задание 1. Создать Deployment и обеспечить доступ к репликам приложения из другого Pod

1. Создать Deployment приложения, состоящего из двух контейнеров — nginx и multitool. Решить возникшую ошибку.
2. После запуска увеличить количество реплик работающего приложения до 2.
3. Продемонстрировать количество подов до и после масштабирования.
4. Создать Service, который обеспечит доступ до реплик приложений из п.1.
5. Создать отдельный Pod с приложением multitool и убедиться с помощью `curl`, что из пода есть доступ до приложений из п.1.

**Файлы манифестов Задания 1:**
- [deployment-app.yaml](https://github.com/Dun9Dev/kuber-hw/blob/hw2/deployment-app.yaml)
- [service-app.yaml](https://github.com/Dun9Dev/kuber-hw/blob/hw2/service-app.yaml)
- [test-pod.yaml](https://github.com/Dun9Dev/kuber-hw/blob/hw2/test-pod.yaml)

![img](https://github.com/Dun9Dev/kuber-hw/blob/hw2/img/Screenshot_20260318_002507.png)
![img](https://github.com/Dun9Dev/kuber-hw/blob/hw2/img/Screenshot_20260318_002705.png)
![img](https://github.com/Dun9Dev/kuber-hw/blob/hw2/img/Screenshot_20260318_004826.png)
![img](https://github.com/Dun9Dev/kuber-hw/blob/hw2/img/Screenshot_20260318_004900.png)
![img](https://github.com/Dun9Dev/kuber-hw/blob/hw2/img/Screenshot_20260319_125839.png)

------

### Задание 2. Создать Deployment и обеспечить старт основного контейнера при выполнении условий

1. Создать Deployment приложения nginx и обеспечить старт контейнера только после того, как будет запущен сервис этого приложения.
2. Убедиться, что nginx не стартует. В качестве Init-контейнера взять busybox.
3. Создать и запустить Service. Убедиться, что Init запустился.
4. Продемонстрировать состояние пода до и после запуска сервиса.

**Файлы манифестов Задания 2:**
- [deployment-init.yaml](https://github.com/Dun9Dev/kuber-hw/blob/hw2/deployment-init.yaml)
- [service-nginx.yaml](https://github.com/Dun9Dev/kuber-hw/blob/hw2/service-nginx.yaml)

![img](https://github.com/Dun9Dev/kuber-hw/blob/hw2/img/Screenshot_20260319_132147.png)
![img](https://github.com/Dun9Dev/kuber-hw/blob/hw2/img/Screenshot_20260319_132817.png)
------

### Правила приема работы

1. Домашняя работа оформляется в своем Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl` и скриншоты результатов.
3. Репозиторий должен содержать файлы манифестов и ссылки на них в файле README.md.

------
