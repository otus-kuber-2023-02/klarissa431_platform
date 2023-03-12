1. Почему все pod в namespace kube-system восстановились после удаления
core-dns - создан replicasets и Deployment:coredns 
kube-proxy - создаётся при помощи daemon set
все остальные переподнимает kubelet как статик поды, так как есть манифесты в /etc/kubernetes/manifests

2.
- create nginx:latest
- docker build -t klarissa451/web . 
- docker push klarissa451/web:latest
- add web-pod.yaml
- kubectl apply -f web-pod.yaml
- create and add init container with volumes
- kubectl port-forward --address 0.0.0.0 pod/web 8000:8000
- curl https://localhost:8000/index.html

Задание *
Добавила нужные env в манифест

