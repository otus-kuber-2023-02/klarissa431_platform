# Выполнено ДЗ №1

 - [V] Основное ДЗ
 - [V] Задание со *

## В процессе сделано:
 1. Установила kind

curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.18.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

Создала кластер:
kind create cluster

$ kind get  clusters
kind

 2. Развернула statefulset minio

kubectl apply -f minio-statefulset.yaml
Сначала создала с одной репликой, проверила создание pv и pvc

Cоздала headless сервис, с указанием clusterIp none

Попробовала увеличить количество реплик до 2, проверила сервисы конечных точек:
mira@kb1:~$ kubectl get endpoints minio
NAME    ENDPOINTS                           AGE
minio   10.244.0.23:9000,10.244.0.24:9000   61m

Создала ещё один под в том же namespace, проверила, что внутри minio резолвится в список тех же адресов:
/ # host minio
minio.default.svc.cluster.local has address 10.244.0.23
minio.default.svc.cluster.local has address 10.244.0.24

Проверила, что сервис работает установкой клиента mc кубернетос:
kubectl run my-mc -i --tty --image minio/mc:latest --command -- bash

По шаблону подключилась к minio из minio client:

[root@my-mc /]# mc alias set myminio/ https://minio.default.svc.cluster.local MY-USER MY-PASSWORD
[root@my-mc /]# mc ls myminio/mybucket

[root@my-mc /]#  mc alias set minio/ http://minio.default.svc.cluster.local:9000 minio minio123
Added `minio` successfully.

Создала backet:
[root@my-mc /]# mc mb --region --region=us-west-2 minio/mynewbucket
Bucket created successfully `minio/mynewbucket`.

[root@my-mc /]# mc ls minio/
[2023-04-01 09:41:16 UTC]     0B mynewbucket/

 3. Задание с *
  Настройка secrets:
  
  get  MINIO_ACCESS_KEY и MINIO_SECRET_KEY in base64:
mira@kb1:~/klarissa431_platform/kubernetes-volumes$ echo -n 'minio' | base64
bWluaW8=

mira@kb1:~/klarissa431_platform/kubernetes-volumes$ echo -n 'minio123' | base64
bWluaW8xMjM=

Создала secret  kubectl apply -f minio-secret.yaml

Проверила, что успешно создался:
mira@kb1:$ kubectl get secrets
NAME            TYPE     DATA   AGE
minio-secrets   Opaque   2      8s

удалила существующий statefulset 

mira@kb1:$ kubectl delete statefulset minio
statefulset.apps "minio" deleted

mira@kb1:~$ kubectl get secret minio-secrets  -o jsonpath='{.data}'
{"MINIO_ACCESS_KEY":"bWluaW8=","MINIO_SECRET_KEY":"bWluaW8xMjM="}

Создала заново kubectl apply -f minio-statefulset-secrets.yaml
Внутри прописала соответствующие секретс:

        env:
        - name: MINIO_ACCESS_KEY
          valueFrom:
            secretKeyRef:
              name: minio-secrets
              key: MINIO_ACCESS_KEY
        - name: MINIO_SECRET_KEY
          valueFrom:
            secretKeyRef:
              name: minio-secrets
              key: MINIO_SECRET_KEY

Заново запустила клиент mc
Аналогично подключилась к minio командами 
mc alias set minio/ http://minio.default.svc.cluster.local:9000 minio minio123
mc ls minio/

Проверила, что клиента успешно подключился с такими данными и смог получить buckets 


## Как запустить проект:
 - Выполнить kubectl apply -f minio-statefulset.yaml
 - Выполнить kubectl apply -f minio-headless-service.yaml

c *
 - kubectl apply -f minio-secret.yaml
 - kubectl apply -f minio-statefulset-secrets.yaml
 - kubectl apply -f minio-headless-service.yaml

## Как проверить работоспособность:
 - Запуском mc, настройкой alias и созданием buckets

## PR checklist:
 - [V] Выставлен label с темой домашнего задания
