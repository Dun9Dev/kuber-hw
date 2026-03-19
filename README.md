# Домашнее задание к занятию «Настройка приложений и управление доступом в Kubernetes» - Выполнил Shestovskikh Daniil

### Примерное время выполнения задания

120 минут

### Цель задания

Научиться:
- Настраивать конфигурацию приложений с помощью **ConfigMaps** и **Secrets**
- Управлять доступом пользователей через **RBAC**

Это задание поможет вам освоить ключевые механизмы Kubernetes для работы с конфигурацией и безопасностью. Эти навыки необходимы для уверенного администрирования кластеров в реальных проектах. На практике навыки используются для:
- Хранения чувствительных данных (Secrets)
- Гибкого управления настройками приложений (ConfigMaps) 
- Контроля доступа пользователей и сервисов (RBAC)

------

## **Подготовка**
### **Чеклист готовности**
- Установлен Kubernetes (MicroK8S)
- Установлен `kubectl`
- Редактор для YAML-файлов
- Утилита `openssl` для генерации сертификатов

------

## **Задание 1: Работа с ConfigMaps**

### **Файлы манифестов Задания 1:**
- [configmap-web.yaml](https://github.com/Dun9Dev/kuber-hw/blob/hw5/configmap-web.yaml)
- [deployment.yaml](https://github.com/Dun9Dev/kuber-hw/blob/hw5/deployment.yaml)

### **Шаги выполнения**
1. **Создание ConfigMap с веб-страницей и Deployment (nginx + multitool):**
   ```bash
   kubectl apply -f configmap-web.yaml
   kubectl apply -f deployment.yaml
   kubectl get pods
   ```
   ![Скриншот 1 - создание ConfigMap, Deployment и проверка подов](https://github.com/Dun9Dev/kuber-hw/blob/hw5/img/Screenshot_20260319_210704.png)

2. **Проверка доступности страницы через port-forward:**
   ```bash
   kubectl port-forward pod/web-app-74447d664f-94r5h 8080:80
   curl http://localhost:8080
   ```
   ![Скриншот 2 - curl к странице из ConfigMap](https://github.com/Dun9Dev/kuber-hw/blob/hw5/img/Screenshot_20260319_210751.png)

---
## **Задание 2: Настройка HTTPS с Secrets**

### **Файлы манифестов Задания 2:**
- [secret-tls.yaml](https://github.com/Dun9Dev/kuber-hw/blob/hw5/secret-tls.yaml)
- [service-web.yaml](https://github.com/Dun9Dev/kuber-hw/blob/hw5/service-web.yaml)
- [ingress-tls.yaml](https://github.com/Dun9Dev/kuber-hw/blob/hw5/ingress-tls.yaml)

### **Шаги выполнения**
1. **Генерация SSL-сертификата и создание Secret:**
   ```bash
   openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=myapp.example.com"
   kubectl apply -f secret-tls.yaml
   ```

2. **Создание Service и Ingress:**
   ```bash
   kubectl apply -f service-web.yaml
   kubectl apply -f ingress-tls.yaml
   kubectl get ingress
   ```

3. **Проверка HTTPS-доступа:**
   ```bash
   curl -k -H "Host: myapp.example.com" https://192.168.0.243/
   ```
   ![Скриншот 3 - проверка HTTPS (curl -k)](https://github.com/Dun9Dev/kuber-hw/blob/hw5/img/Screenshot_20260319_220344.png)
   
   ![Скриншот 4 - проверка Ingress (get ingress и curl с заголовком)](https://github.com/Dun9Dev/kuber-hw/blob/hw5/img/Screenshot_20260319_224521.png)

---
## **Задание 3: Настройка RBAC**

### **Файлы манифестов Задания 3:**
- [role-pod-reader.yaml](https://github.com/Dun9Dev/kuber-hw/blob/hw5/role-pod-reader.yaml)
- [rolebinding-developer.yaml](https://github.com/Dun9Dev/kuber-hw/blob/hw5/rolebinding-developer.yaml)

### **Шаги выполнения**
1. **Включение RBAC в MicroK8S (на рабочей ВМ):**
   ```bash
   microk8s enable rbac
   ```

2. **Генерация сертификатов для пользователя developer:**
   ```bash
   openssl genrsa -out developer.key 2048
   openssl req -new -key developer.key -out developer.csr -subj "/CN=developer"
   # Копирование CA-файлов с ВМ
   scp dun9@192.168.0.243:/var/snap/microk8s/current/certs/ca.crt .
   scp dun9@192.168.0.243:/var/snap/microk8s/current/certs/ca.key .
   openssl x509 -req -in developer.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out developer.crt -days 365
   ```

3. **Создание Role и RoleBinding:**
   ```bash
   kubectl apply -f role-pod-reader.yaml
   kubectl apply -f rolebinding-developer.yaml
   ```
   ![Скриншот 6 - создание role и rolebinding](https://github.com/Dun9Dev/kuber-hw/blob/hw5/img/Screenshot_20260319_225056.png)

4. **Проверка прав пользователя developer:**
   ```bash
   kubectl get pods --as=developer
   kubectl delete pod web-app-74447d664f-94r5h --as=developer
   ```
   ![Скриншот 5 - ошибка Forbidden при попытке удалить под](https://github.com/Dun9Dev/kuber-hw/blob/hw5/img/Screenshot_20260319_225018.png)
   
   *(вывод `kubectl get pods --as=developer` успешный, что подтверждает права на просмотр)*

---
## **Правила приёма работы**
1. Домашняя работа оформлена в Git-репозитории в файле README.md.
2. Файл README.md содержит скриншоты вывода команд `kubectl` и результаты выполнения.
3. Репозиторий содержит файлы манифестов и ссылки на них в файле README.md.
4. Для заданий с TLS приложены команды генерации сертификатов.

## **Критерии оценивания задания**
1. Зачёт: Все задачи выполнены, манифесты корректны, есть доказательства работы (скриншоты).
2. Доработка (на доработку задание направляется 1 раз): основные задачи выполнены, при этом есть ошибки в манифестах или отсутствуют проверочные скриншоты.
3. Незачёт: работа выполнена не в полном объёме, есть ошибки в манифестах, отсутствуют проверочные скриншоты. Все попытки доработки израсходованы.
