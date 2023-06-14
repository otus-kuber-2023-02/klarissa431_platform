

1. Подняла врусную kubernetes класетр в yandex и создала группу узлов
по доке:
https://cloud.yandex.ru/docs/managed-kubernetes/tutorials/new-kubernetes-project#create-sa


 Подключила yc к новому кластер:
 yc managed-kubernetes cluster get-credentials k8s-demo --external

MacBook-Pro:~ root#  yc managed-kubernetes cluster list
+----------------------+----------+---------------------+---------+---------+------------------------+----------------------+
|          ID          |   NAME   |     CREATED AT      | HEALTH  | STATUS  |   EXTERNAL ENDPOINT    |  INTERNAL ENDPOINT   |
+----------------------+----------+---------------------+---------+---------+------------------------+----------------------+
| cattbefe7ciaid4mdrra | k8s-demo | 2023-06-12 14:48:31 | HEALTHY | RUNNING | https://158.160.57.168 | https://192.168.1.29 |
+----------------------+----------+---------------------+---------+---------+------------------------+----------------------+


2. Установила helm
helm version
version.BuildInfo{Version:"v3.11.3", GitCommit:"323249351482b3bbfc9f5004f65d400aa70f9ae7", GitTreeState:"clean", GoVersion:"go1.20.3"}

Добавила репозиторий 
helm repo add stable https://charts.helm.sh/stable
helm repo update

3. Установка nginx-ingress (4.6.0)
kubectl create ns nginx-ingress
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update ingress-nginx
helm upgrade --install my-ingress ingress-nginx/ingress-nginx --namespace=nginx-ingress --version="4.6.0" --debug --wait 


MacBook-Pro:~ root# kubectl get services -n nginx-ingress
NAME                                    TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)                      AGE
my-ingress-ingress-nginx-controller             LoadBalancer   10.96.237.12   62.84.126.195   80:31838/TCP,443:31451/TCP   3m9s
my-ingress-nginx-controller-admission   ClusterIP      10.96.214.21   <none>          443/TCP                      3m9s


4. Установила cert-manager
# CRD
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.11.1/cert-manager.crds.yaml
# Cert-manager
helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --version v1.11.1
helm list --all-namespaces

# acme-issuer
kubectl apply -f cert-manager/acme-issuer.yaml

# kubectl get ClusterIssuer
NAME               READY   AGE
letsencrypt-prod   True    18s


MacBook-Pro:~ root# helm list --all-namespaces
NAME        	NAMESPACE    	REVISION	UPDATED                             	STATUS  	CHART               	APP VERSION
cert-manager	cert-manager 	1       	2023-06-12 18:12:37.107907 +0300 MSK	deployed	cert-manager-v1.11.1	v1.11.1    
my-ingress      nginx-ingress	1       	2023-06-12 18:08:04.233952 +0300 MSK	deployed	ingress-nginx-4.6.0 	1.7.0  

5.
kubectl create ns chartmuseum
helm repo add chartmuseum https://chartmuseum.github.io/charts
helm repo update chartmuseum

root@kb1:/home/mira/klarissa431_platform/kubernetes-templating# helm upgrade --install chartmuseum-release chartmuseum/chartmuseum  --wait  --namespace=chartmuseum   --version 3.9.3 -f chartmuseum/values.yaml
Release "chartmuseum-release" does not exist. Installing it now.
NAME: chartmuseum-release
LAST DEPLOYED: Mon Jun 12 15:38:09 2023
NAMESPACE: chartmuseum
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
** Please be patient while the chart is being deployed **

Get the ChartMuseum URL by running:

  export POD_NAME=$(kubectl get pods --namespace chartmuseum -l "app=chartmuseum" -l "release=chartmuseum-release" -o jsonpath="{.items[0].metadata.name}")
  echo http://127.0.0.1:8080/
  kubectl port-forward $POD_NAME 8080:8080 --namespace chartmuseum

helm ls -n chartmuseum

6. chartmuseum доступен по адресу с валидным сертификатом https://chartmuseum.62.84.126.195.sslip.io

7.
kubectl get secrets -n chartmuseum
NAME                                        TYPE                                  DATA   AGE
chartmuseum-release                         Opaque                                0      4m57s
chartmuseum.62.84.126.195.sslip.io          kubernetes.io/tls                     2      4m20s
default-token-pzmsb                         kubernetes.io/service-account-token   3      26m
sh.helm.release.v1.chartmuseum-release.v1   helm.sh/release.v1                    1      4m57s

8. Установка harbor
kubectl create ns harbor
helm repo add harbor https://helm.goharbor.io
helm repo update harbor
helm upgrade --install harbor harbor/harbor --wait --namespace=harbor --version=1.12.0 -f values.yaml

 helm ls -n harbor
NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
harbor  harbor          1               2023-06-12 15:52:24.311823373 +0000 UTC deployed        harbor-1.12.0   2.8.0

https://capture.dropbox.com/JedKkJV8MfoKSDdz

harbor работает, сертификат валиден.
# kubectl get secrets -n harbor -l owner=helm
NAME                           TYPE                 DATA   AGE
sh.helm.release.v1.harbor.v1   helm.sh/release.v1   1      5m42s

9. Создаём свой helm чарт

helm create hipster-shop
rm ./hipster-shop/values.yaml
rm -rf ./hipster-shop/templates/*
wget https://raw.githubusercontent.com/express42/otus-platform-snippets/master/Module-04/05-Templating/manifests/all-hipster-shop.yaml 
kubectl create ns hipster-shop
helm upgrade --install hipster-shop hipster-shop --namespace hipster-shop
kubectl get services -n hipster-shop

10. frontend

вынесем frontend в отдельный helm
helm upgrade --install frontend frontend --namespace hipster-shop

установим зависимость
helm dep update hipster-shop

# ls charts/
frontend-0.1.0.tgz

11.  kubecfg

root@kb1:~#  kubecfg version
kubecfg version: v0.22.0
jsonnet version: v0.17.0
client-go version: v0.0.0-master+$Format:%h$
