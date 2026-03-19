# Домашнее задание к занятию «Как работает сеть в K8s» - Выполнил Shestovskikh Daniil

### Цель задания

Настроить сетевую политику доступа к подам.

------

### Чеклист готовности к домашнему заданию

1. Кластер K8s с установленным сетевым плагином Calico (MicroK8S).

------

## **Задание 1. Создать сетевую политику для обеспечения доступа**

### **Файлы манифестов:**

**Deployment'ы:**
- [frontend-deployment.yaml](https://github.com/Dun9Dev/kuber-hw/blob/hw9/frontend-deployment.yaml)
- [backend-deployment.yaml](https://github.com/Dun9Dev/kuber-hw/blob/hw9/backend-deployment.yaml)
- [cache-deployment.yaml](https://github.com/Dun9Dev/kuber-hw/blob/hw9/cache-deployment.yaml)

**Сервисы:**
- [frontend-service.yaml](https://github.com/Dun9Dev/kuber-hw/blob/hw9/frontend-service.yaml)
- [backend-service.yaml](https://github.com/Dun9Dev/kuber-hw/blob/hw9/backend-service.yaml)
- [cache-service.yaml](https://github.com/Dun9Dev/kuber-hw/blob/hw9/cache-service.yaml)

**Сетевые политики:**
- [network-policy-frontend-to-backend.yaml](https://github.com/Dun9Dev/kuber-hw/blob/hw9/network-policy-frontend-to-backend.yaml)
- [network-policy-backend-to-cache.yaml](https://github.com/Dun9Dev/kuber-hw/blob/hw9/network-policy-backend-to-cache.yaml)

### **Ход выполнения**

1. **Создание namespace `app` и развёртывание всех ресурсов:**
   ```bash
   kubectl create namespace app
   kubectl apply -f frontend-deployment.yaml
   kubectl apply -f backend-deployment.yaml
   kubectl apply -f cache-deployment.yaml
   kubectl apply -f frontend-service.yaml
   kubectl apply -f backend-service.yaml
   kubectl apply -f cache-service.yaml
   ```

2. **Проверка, что все поды запустились:**
   ```bash
   kubectl get pods -n app
   ```
   *(на скриншоте видно 3 пода в статусе Running)*

3. **Проверка доступа без политик (всё разрешено):**
   ```bash
   kubectl exec -it -n app frontend-67c67f6c68-xdl4m -- curl backend-service
   kubectl exec -it -n app frontend-67c67f6c68-xdl4m -- curl cache-service
   ```
   *(на скриншоте видно успешные ответы от backend и cache)*

4. **Применение сетевых политик:**
   ```bash
   kubectl apply -f network-policy-frontend-to-backend.yaml
   kubectl apply -f network-policy-backend-to-cache.yaml
   ```

5. **Проверка разрешённого трафика (frontend → backend):**
   ```bash
   kubectl exec -it -n app frontend-67c67f6c68-xdl4m -- curl backend-service
   ```
   *(на скриншоте видно успешный ответ от backend)*

6. **Проверка запрещённого трафика (frontend → cache):**
   ```bash
   kubectl exec -it -n app frontend-67c67f6c68-xdl4m -- curl cache-service --connect-timeout 5
   ```
   *(на скриншоте видно таймаут соединения)*

7. **Проверка разрешённого трафика (backend → cache):**
   ```bash
   kubectl exec -it -n app backend-9bb96bd8d-q4gmz -- curl cache-service
   ```
   *(на скриншоте видно успешный ответ от cache)*

8. **Проверка запрещённого трафика (cache → backend):**
   ```bash
   kubectl exec -it -n app cache-6dfb8d5d78-h45gx -- curl backend-service --connect-timeout 5
   ```
   *(на скриншоте видно таймаут соединения)*

### **Итоговый скриншот выполнения**

![Весь процесс выполнения ДЗ №9](https://github.com/Dun9Dev/kuber-hw/blob/hw9/img/Screenshot_20260320_001131.png)

### **Результат**

Сетевые политики настроены корректно и обеспечивают требуемую цепочку доступа:
- **frontend → backend** — разрешено
- **backend → cache** — разрешено
- **frontend → cache** — запрещено
- **cache → backend** — запрещено
- любые другие подключения также запрещены (по умолчанию)

------

### Правила приёма работы

1. Домашняя работа оформлена в Git-репозитории в файле README.md.
2. Файл README.md содержит скриншоты вывода необходимых команд и результатов.
3. Репозиторий содержит тексты манифестов и ссылки на них в файле README.md.

