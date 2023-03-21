# Выполнено ДЗ №

 - [x] Основное ДЗ
 - [x] Задание со *

## В процессе сделано:
 - Изучены возможности контроллера Replicaset, важность selector и matchLabels
 - Изменение количества реплик
 - Почему обновление ReplicaSet не влечёт обновление запущенных pod:replicaset может обновить поды только при пересоздании (темплейтинг)
 - Изучен контроллер Deployment, обновление и откат
 - Реализован аналог blue-green и reverse rolling update

## Как запустить проект:
 - kubectl apply -f frontend-replicaset.yaml
 - kubectl apply -f paymentservice-replicaset.yaml
 - kubectl apply -f paymentservice-deployment.yaml

## Как проверить работоспособность:

## PR checklist:
 - [x] Выставлен label с темой домашнего задания
