# Домашнее задание к занятию «Troubleshooting» - Выполнил Shestovskikh Daniil

### Цель задания

Устранить неисправности при деплое приложения.

------

### Чеклист готовности к домашнему заданию

1. Кластер K8s (MicroK8S).

------

## **Задание. При деплое приложение web-consumer не может подключиться к auth-db**

### **Исходный манифест**
```bash
kubectl apply -f https://raw.githubusercontent.com/netology-code/kuber-homeworks/main/3.5/files/task.yaml
```

### **Ход выполнения**

#### 1. Первая попытка установки (ошибка)

**Выявленная проблема 1:** В манифесте ресурсы создаются в namespace `web` и `data`, но сами namespace не существуют.

#### 2. Создание недостающих namespace
```bash
kubectl create namespace web
kubectl create namespace data
```

#### 3. Повторное применение манифеста (успешно)
```bash
kubectl apply -f https://raw.githubusercontent.com/netology-code/kuber-homeworks/main/3.5/files/task.yaml
```

#### 4. Проверка логов web-consumer (обнаружение второй ошибки)
```bash
kubectl logs -n web deployment/web-consumer
```

**Выявленная проблема 2:** Под `web-consumer` пытается достучаться до сервиса `auth-db` по короткому имени, но сервис находится в другом namespace (`data`). В Kubernetes короткие имена резолвятся только в пределах одного namespace.

#### 5. Исправление команды в deployment
```bash
kubectl edit deployment -n web web-consumer
```
(в редакторе заменяем `curl auth-db` на `curl auth-db.data.svc.cluster.local`)

#### 6. Принудительный перезапуск deployment
```bash
kubectl rollout restart deployment -n web web-consumer
```

#### 7. Проверка логов после исправления
```bash
kubectl logs -n web deployment/web-consumer
```
![img]()
![img]()
![img]()
![img]()
![img]()
![img]()
![img]()

### **Результат**

Проблема успешно устранена:
- ✅ Созданы недостающие namespace
- ✅ Исправлена команда в поде на полное доменное имя сервиса
- ✅ Под перезапущен и теперь успешно обращается к `auth-db`

------

### **Правила приёма работы**

1. Домашняя работа оформлена в Git-репозитории в файле README.md.
2. Файл README.md содержит скриншоты вывода необходимых команд и результатов.
3. Репозиторий содержит тексты манифестов или ссылки на них в файле README.md.
