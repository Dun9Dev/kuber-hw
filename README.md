# Домашнее задание к занятию «Хранение в K8s» - Выполнил Shestovskikh Daniil

### Примерное время выполнения задания — 180 минут

### Цель задания

Научиться работать с хранилищами в тестовой среде Kubernetes:
- обеспечить обмен файлами между контейнерами пода;
- создавать **PersistentVolume** (PV) и использовать его в подах через **PersistentVolumeClaim** (PVC);
- объявлять свой **StorageClass** (SC) и монтировать его в под через **PVC**.

------

## **Подготовка**
### **Чеклист готовности**
1. Установленное K8s-решение (MicroK8S).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключенным GitHub-репозиторием.

------

## **Задание 1. Volume: обмен данными между контейнерами в поде**

### **Файлы манифестов Задания 1:**
- [containers-data-exchange.yaml](https://github.com/Dun9Dev/kuber-hw/blob/hw4/containers-data-exchange.yaml)

### **Шаги выполнения**

1. **Создание Deployment с busybox и multitool:**
   ```bash
   kubectl apply -f containers-data-exchange.yaml
   ```
   ![Скриншот 1 - создание Deployment](https://github.com/Dun9Dev/kuber-hw/blob/hw4/img/Screenshot_20260319_165957.png)

2. **Проверка, что под запустился:**
   ```bash
   kubectl get pods
   ```
   Под `data-exchange-...` в статусе Running.

3. **Описание пода (для отчёта):**
   ```bash
   kubectl describe pod data-exchange-64484b4d76-5sk9v
   ```
   ![Скриншот 2-3 - describe pod (часть 1 и 2)](https://github.com/Dun9Dev/kuber-hw/blob/hw4/img/Screenshot_20260319_170050.png)
   *(продолжение на следующем скриншоте)*
   ![Скриншот 3 - describe pod (часть 3)](https://github.com/Dun9Dev/kuber-hw/blob/hw4/img/Screenshot_20260319_170119.png)

4. **Проверка чтения данных из общего volume:**
   ```bash
   kubectl exec -it data-exchange-64484b4d76-5sk9v -c reader -- tail -f /data/date.log
   ```
   ![Скриншот 4 - tail -f из контейнера reader](https://github.com/Dun9Dev/kuber-hw/blob/hw4/img/Screenshot_20260319_170229.png)

------
## **Задание 2. PV, PVC**

### **Файлы манифестов Задания 2:**
- [pv-pvc.yaml](https://github.com/Dun9Dev/kuber-hw/blob/hw4/pv-pvc.yaml)
- [deployment-pvc.yaml](https://github.com/Dun9Dev/kuber-hw/blob/hw4/deployment-pvc.yaml)

### **Шаги выполнения**

1. **Создание директории на ноде для PV (на рабочей ВМ):**
   ```bash
   sudo mkdir -p /mnt/data
   sudo chmod 777 /mnt/data
   ```
   ![Скриншот 5 - создание директории на ВМ](https://github.com/Dun9Dev/kuber-hw/blob/hw4/img/Screenshot_20260319_170321.png)

2. **Создание PV и PVC:**
   ```bash
   kubectl apply -f pv-pvc.yaml
   ```
   ![Скриншот 6 - создание PV и PVC](https://github.com/Dun9Dev/kuber-hw/blob/hw4/img/Screenshot_20260319_170351.png)

3. **Проверка статуса PV и PVC (должны быть Bound):**
   ```bash
   kubectl get pv
   kubectl get pvc
   ```
   ![Скриншот 7 - проверка PV и PVC](https://github.com/Dun9Dev/kuber-hw/blob/hw4/img/Screenshot_20260319_170419.png)

4. **Создание Deployment, использующего PVC:**
   ```bash
   kubectl apply -f deployment-pvc.yaml
   kubectl get pods
   ```
   ![Скриншот 8 - создание Deployment и проверка подов](https://github.com/Dun9Dev/kuber-hw/blob/hw4/img/Screenshot_20260319_170514.png)

5. **Проверка записи и чтения данных в PVC:**
   ```bash
   kubectl exec -it data-exchange-pvc-5cd7c64749-9kdqt -c reader -- tail -f /data/date.log
   ```
   ![Скриншот 9 - чтение данных из PVC](https://github.com/Dun9Dev/kuber-hw/blob/hw4/img/Screenshot_20260319_170549.png)

6. **Удаление Deployment и PVC:**
   ```bash
   kubectl delete deployment data-exchange-pvc
   kubectl delete pvc local-pvc
   ```
   ![Скриншот 10 - удаление Deployment и PVC](https://github.com/Dun9Dev/kuber-hw/blob/hw4/img/Screenshot_20260319_170650.png)

7. **Проверка состояния PV после удаления PVC:**
   ```bash
   kubectl get pv
   kubectl describe pv local-pv
   ```
   ![Скриншот 11 - PV в статусе Released](https://github.com/Dun9Dev/kuber-hw/blob/hw4/img/Screenshot_20260319_170732.png)
   
   **Пояснение:** PV перешёл в статус **Released**, потому что политика восстановления `Retain` сохраняет данные, но PV больше не может быть использован без ручной очистки.

8. **Проверка, что файл сохранился на ноде (после удаления PVC):**
   ```bash
   # На рабочей ВМ:
   ls -la /mnt/data/
   cat /mnt/data/date.log
   ```
   ![Скриншот 12 - данные сохранились на ноде](https://github.com/Dun9Dev/kuber-hw/blob/hw4/img/Screenshot_20260319_170807.png)

9. **Удаление PV:**
   ```bash
   kubectl delete pv local-pv
   ```
   ![Скриншот 13 - удаление PV](https://github.com/Dun9Dev/kuber-hw/blob/hw4/img/Screenshot_20260319_175605.png)

10. **Проверка файла на ноде после удаления PV:**
    ```bash
    # На рабочей ВМ:
    cat /mnt/data/date.log
    ```
    ![Скриншот 14 - данные сохранились после удаления PV](https://github.com/Dun9Dev/kuber-hw/blob/hw4/img/Screenshot_20260319_175634.png)
    
    **Пояснение:** Данные остались на диске ноды, так как политика `Retain` не удаляет файлы автоматически. Администратор может забрать их вручную.

------
## **Задание 3. StorageClass**

### **Файлы манифестов Задания 3:**
- [sc.yaml](https://github.com/Dun9Dev/kuber-hw/blob/hw4/sc.yaml)
- [deployment-sc.yaml](https://github.com/Dun9Dev/kuber-hw/blob/hw4/deployment-sc.yaml)
- [pv-for-sc.yaml](https://github.com/Dun9Dev/kuber-hw/blob/hw4/pv-for-sc.yaml)

### **Шаги выполнения**

1. **Создание StorageClass и PVC (с WaitForFirstConsumer):**
   ```bash
   kubectl apply -f sc.yaml
   ```
   PVC будет в статусе Pending.

2. **Проверка статуса PVC (ожидаемо Pending):**
   ```bash
   kubectl get pvc
   ```
   ![Скриншот 16 - PVC в статусе Pending](https://github.com/Dun9Dev/kuber-hw/blob/hw4/img/Screenshot_20260319_175851.png)

3. **Создание Deployment, который запросит этот PVC:**
   ```bash
   kubectl apply -f deployment-sc.yaml
   ```
   PVC всё ещё Pending, так как нет подходящего PV.

4. **Создание PV под этот StorageClass:**
   ```bash
   # На рабочей ВМ:
   sudo mkdir -p /mnt/data-sc
   sudo chmod 777 /mnt/data-sc
   
   # На ноутбуке:
   kubectl apply -f pv-for-sc.yaml
   ```
   ![Скриншот 18 - создание PV и проверка PVC (Bound)](https://github.com/Dun9Dev/kuber-hw/blob/hw4/img/Screenshot_20260319_180055.png)

5. **Проверка, что PVC стал Bound и под запустился:**
   ```bash
   kubectl get pvc
   kubectl get pods | grep data-exchange-sc
   ```
   ![Скриншот 19 - финальная проверка: под запущен, данные читаются](https://github.com/Dun9Dev/kuber-hw/blob/hw4/img/Screenshot_20260319_180208.png)

6. **Проверка чтения данных из PVC через StorageClass:**
   ```bash
   kubectl exec -it data-exchange-sc-66fff66b47-cqd74 -c reader -- tail -f /data/date.log
   ```
   (вывод виден на скриншоте 19)

------

## **Правила приёма работы**
1. Домашняя работа оформлена в Git-репозитории в файле README.md.
2. Файл README.md содержит скриншоты вывода необходимых команд `kubectl` и скриншоты результатов.
3. Репозиторий содержит файлы манифестов и ссылки на них в файле README.md.

## **Критерии оценивания задания**
1. Зачёт: Все задачи выполнены, манифесты корректны, есть доказательства работы (скриншоты) и пояснения по заданию 2.
2. Доработка (на доработку задание направляется 1 раз): основные задачи выполнены, при этом есть ошибки в манифестах или отсутствуют проверочные скриншоты.
3. Незачёт: работа выполнена не в полном объёме, есть ошибки в манифестах, отсутствуют проверочные скриншоты. Все попытки доработки израсходованы (на доработку работа направляется 1 раз). Этот вид оценки используется крайне редко.

## **Срок выполнения задания**  
1. 5 дней на выполнение задания.
2. 5 дней на доработку задания (в случае направления задания на доработку).
