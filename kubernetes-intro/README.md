# Выполнено ДЗ №1

 - [+ ] Основное ДЗ
 - [+ ] Задание со *

## В процессе сделано:
 1. Почему все pod в namespace kube-system восстановились после удаления

 - core-dns - создан replicasets и Deployment:coredns 
 - kube-proxy - создаётся при помощи daemon set
 - все остальные переподнимает kubelet как статик поды, так как есть манифесты в /etc/kubernetes/manifests

 2.
 - Собран образ на основе nginx:latest с заданными требованиями
 - Coздан Pod из сформированного ранее манифеста 
 - Добавлен init контейнер в манифест pod'а, настроены volume
 - Натсроен port-forward  

 3. Задание с *
 - Добавлены необходимые env в манифест

## Как запустить проект:
 - Выполнить kubectl apply -f web-pod.yaml
 - Выполнить kubectl port-forward web-nginx 8000:8000

## Как проверить работоспособность:
 - Перейти по ссылке http://localhost:8000/index.html 

## PR checklist:
 - [ ] Выставлен label с темой домашнего задания
