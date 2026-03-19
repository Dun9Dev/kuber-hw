# Домашнее задание к занятию «Helm» - Выполнил Shestovskikh Daniil

### Цель задания

В тестовой среде Kubernetes необходимо установить и обновить приложения с помощью Helm.

------

### Чеклист готовности к домашнему заданию

1. Установленное k8s-решение (MicroK8S).
2. Установленный локальный kubectl.
3. Установленный локальный Helm.
4. Редактор YAML-файлов с подключенным репозиторием GitHub.

------

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Инструкция](https://helm.sh/docs/intro/install/) по установке Helm. [Helm completion](https://helm.sh/docs/helm/helm_completion/).

------

## **Задание 1. Подготовить Helm-чарт для приложения**

### **Файлы манифестов:**
- [myapp-chart/](https://github.com/Dun9Dev/kuber-hw/tree/hw6/myapp-chart) - полная структура Helm-чарта
  - [Chart.yaml](https://github.com/Dun9Dev/kuber-hw/blob/hw6/myapp-chart/Chart.yaml)
  - [values.yaml](https://github.com/Dun9Dev/kuber-hw/blob/hw6/myapp-chart/values.yaml)
  - [templates/deployment.yaml](https://github.com/Dun9Dev/kuber-hw/blob/hw6/myapp-chart/templates/deployment.yaml)

### **Шаги выполнения**

1. **Создание базовой структуры Helm-чарта:**
   ```bash
   helm create myapp-chart
   ```
   ![Скриншот 1 - создание Helm-чарта](https://github.com/Dun9Dev/kuber-hw/blob/hw6/img/Screenshot_20260319_231656.png)

2. **Настройка чарта под приложение с двумя контейнерами (nginx + multitool):**
   - Добавлены параметры для multitool в `values.yaml`
   - Обновлён шаблон `deployment.yaml` для двух контейнеров

3. **Проверка корректности чарта:**
   ```bash
   helm lint myapp-chart/
   ```
   ![Скриншот 2 - успешный lint чарта](https://github.com/Dun9Dev/kuber-hw/blob/hw6/img/Screenshot_20260319_232453.png)

------
## **Задание 2. Запустить две версии в разных неймспейсах**

### **Шаги выполнения**

1. **Создание namespace:**
   ```bash
   kubectl create namespace app1
   kubectl create namespace app2
   ```
   ![Скриншот 3 - создание namespace](https://github.com/Dun9Dev/kuber-hw/blob/hw6/img/Screenshot_20260319_232550.png)

2. **Установка первой версии приложения в namespace app1:**
   ```bash
   helm install myapp-release-1 ./myapp-chart --namespace app1
   ```
   ![Скриншот 4 - установка myapp-release-1](https://github.com/Dun9Dev/kuber-hw/blob/hw6/img/Screenshot_20260319_232613.png)

3. **Установка второй версии приложения в тот же namespace app1:**
   ```bash
   helm install myapp-release-2 ./myapp-chart --namespace app1
   ```
   ![Скриншот 5 - установка myapp-release-2](https://github.com/Dun9Dev/kuber-hw/blob/hw6/img/Screenshot_20260319_232644.png)

4. **Установка третьей версии приложения в namespace app2:**
   ```bash
   helm install myapp-release-3 ./myapp-chart --namespace app2
   ```
   ![Скриншот 6 - установка myapp-release-3](https://github.com/Dun9Dev/kuber-hw/blob/hw6/img/Screenshot_20260319_232743.png)

5. **Проверка, что все поды запущены:**
   ```bash
   kubectl get pods -n app1
   kubectl get pods -n app2
   ```
   ![Скриншот 7 - проверка подов в обоих namespace](https://github.com/Dun9Dev/kuber-hw/blob/hw6/img/Screenshot_20260319_232807.png)

### **Результат:**
- В namespace `app1` запущено 2 версии приложения (myapp-release-1 и myapp-release-2)
- В namespace `app2` запущена 1 версия приложения (myapp-release-3)
- Все поды в статусе `Running` с двумя контейнерами (nginx и multitool)

### **Правила приёма работы**

1. Домашняя работа оформлена в Git-репозитории в файле README.md.
2. Файл README.md содержит скриншоты вывода необходимых команд `kubectl` и `helm`.
3. Репозиторий содержит файлы манифестов (структуру чарта) и ссылки на них в файле README.md.

