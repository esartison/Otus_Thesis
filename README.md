Дипломная работа по Postges под управлением Kubernetes


## **(1) Создать Kubernetes кластер**

Для экономии времени, решил создать managed Kubernetes кластер в Yandex Cloud

### Создаем высокодоступный Kubernetes кластер в 3х разных зонах доступности
<img width="531" height="484" alt="image" src="https://github.com/user-attachments/assets/ef879d8f-cb15-4c05-9e9c-1e2f664818b1" />

Остальные настройки оставляем по умолчанию. 

кластер успешно создался
<img width="885" height="67" alt="image" src="https://github.com/user-attachments/assets/99e11dcd-d8e1-46ed-9df4-28977d90c2fa" />
<img width="483" height="290" alt="image" src="https://github.com/user-attachments/assets/5d03fdb9-0be7-4d68-8977-5f6a44adbb1f" />

Создал кластер чтобы проверить что все работает, для экономии средств не буду его использовать. 


### Создаем НЕвысокодоступный Kubernetes кластер в 1-й зоне
Создаем базовый Kuberntes кластер в 1 зоне доступности
<img width="910" height="373" alt="image" src="https://github.com/user-attachments/assets/a4d3c7b0-5ed7-49f8-83df-24f7337416b2" />


### Создаем группу рабочих узлов workerkub в Kubernetes кластере otuskuberes
В группе создаем 3 узла(node)
<img width="1039" height="112" alt="image" src="https://github.com/user-attachments/assets/fcd8f2cf-998d-4d7d-8a38-bbe3ded26d74" />

<img width="521" height="564" alt="image" src="https://github.com/user-attachments/assets/26ff7c77-a081-47fe-aaf1-30e813e23bf8" />

<img width="1583" height="333" alt="image" src="https://github.com/user-attachments/assets/3fbd7906-14b8-437a-9ab6-8ea9a390a940" />


### Создание VM с Ubuntu для управления нашим Kubernetes cluster
<img width="452" height="209" alt="image" src="https://github.com/user-attachments/assets/a6070c02-1317-46a4-b95e-61f21d9be579" />





## **(2) Настройка kubermgt01 для управления нашим Kubernetes cluster**
Нужна простая virtual machine на Ubuntu для управления нашим Kubernetes кластером. Разворачиваю в облаке.


### Создание VM с Ubuntu для управления нашим Kubernetes cluster
<img width="452" height="209" alt="image" src="https://github.com/user-attachments/assets/a6070c02-1317-46a4-b95e-61f21d9be579" />


### Настройка утилит для управления Kubernetes cluster
```
# установка утилиты yc
curl -sSL https://storage.yandexcloud.net/yandexcloud-yc/install.sh | bash
exit
ssh connect
yc init

# установка kubectl
sudo apt-get update && sudo apt-get install -y apt-transport-https
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
esartison@kubermgt01:~$ kubectl version --client
Client Version: v1.34.1
Kustomize Version: v5.7.1
esartison@kubermgt01:~$  kubectl version --client --output=yaml
clientVersion:
  buildDate: "2025-09-09T19:44:50Z"
  compiler: gc
  gitCommit: 93248f9ae092f571eb870b7664c534bfc7d00f03
  gitTreeState: clean
  gitVersion: v1.34.1
  goVersion: go1.24.6
  major: "1"
  minor: "34"
  platform: linux/amd64
kustomizeVersion: v5.7.1

# установка Postgres клиента
sudo apt install postgresql-client


# Настройка kubectl
esartison@kubermgt01:~$ yc managed-kubernetes cluster get-credentials --id catofhs7gt4b2i448mdg --external

esartison@kubermgt01:~$ kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://51.250.13.148
  name: yc-managed-k8s-catofhs7gt4b2i448mdg
contexts:
- context:
    cluster: yc-managed-k8s-catofhs7gt4b2i448mdg
    user: yc-managed-k8s-catofhs7gt4b2i448mdg
  name: yc-otuskuberes
current-context: yc-otuskuberes
kind: Config
users:
- name: yc-managed-k8s-catofhs7gt4b2i448mdg
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1beta1
      args:
      - k8s
      - create-token
      - --profile=default
      command: /home/esartison/yandex-cloud/bin/yc
      env: null
      interactiveMode: IfAvailable
      provideClusterInfo: false


# Проверка состояния Kubernetes Cluster
esartison@kubermgt01:~$ yc container cluster list
+----------------------+-------------+---------------------+---------+---------+-----------------------+---------------------+
|          ID          |    NAME     |     CREATED AT      | HEALTH  | STATUS  |   EXTERNAL ENDPOINT   |  INTERNAL ENDPOINT  |
+----------------------+-------------+---------------------+---------+---------+-----------------------+---------------------+
| catofhs7gt4b2i448mdg | otuskuberes | 2025-09-21 04:47:53 | HEALTHY | RUNNING | https://51.250.13.148 | https://10.128.0.23 |
+----------------------+-------------+---------------------+---------+---------+-----------------------+---------------------+

esartison@kubermgt01:~$ yc container cluster list-node-groups catofhs7gt4b2i448mdg
+----------------------+-----------+----------------------+---------------------+---------+------+
|          ID          |   NAME    |  INSTANCE GROUP ID   |     CREATED AT      | STATUS  | SIZE |
+----------------------+-----------+----------------------+---------------------+---------+------+
| cat0rvr566p70e22316d | workerkub | cl1o2cumtlodjr8887jm | 2025-09-21 05:09:51 | RUNNING |    3 |
+----------------------+-----------+----------------------+---------------------+---------+------+

esartison@kubermgt01:~$ yc compute disk list
+----------------------+-------------------------------------+--------------+---------------+--------+----------------------+-----------------+-------------+
|          ID          |                NAME                 |     SIZE     |     ZONE      | STATUS |     INSTANCE IDS     | PLACEMENT GROUP | DESCRIPTION |
+----------------------+-------------------------------------+--------------+---------------+--------+----------------------+-----------------+-------------+
| epdkkd84di8galak0lpl |                                     | 103079215104 | ru-central1-b | READY  | epd6dvet7p0u6s2ego74 |                 |             |
| epdnjfjbcvnbkrbhrqbf |                                     | 103079215104 | ru-central1-b | READY  | epdah132nbki3j48iukf |                 |             |
| epdt4mqd7ik32dr6r5nb |                                     | 103079215104 | ru-central1-b | READY  | epdklnfli43m5hmb3oqp |                 |             |
| epdvmi8ssrai708jqce4 | disk-ubuntu-24-04-lts-1758442581983 |  21474836480 | ru-central1-b | READY  | epd3vtkpnbg6erpsu0m2 |                 |             |
+----------------------+-------------------------------------+--------------+---------------+--------+----------------------+-----------------+-------------+

esartison@kubermgt01:~$ kubectl get nodes
NAME                        STATUS   ROLES    AGE     VERSION
cl1o2cumtlodjr8887jm-apez   Ready    <none>   3h16m   v1.32.1
cl1o2cumtlodjr8887jm-ubor   Ready    <none>   3h17m   v1.32.1
cl1o2cumtlodjr8887jm-utaq   Ready    <none>   3h17m   v1.32.1

esartison@kubermgt01:~$ kubectl get nodes
NAME                        STATUS   ROLES    AGE     VERSION
cl1o2cumtlodjr8887jm-apez   Ready    <none>   3h16m   v1.32.1
cl1o2cumtlodjr8887jm-ubor   Ready    <none>   3h17m   v1.32.1
cl1o2cumtlodjr8887jm-utaq   Ready    <none>   3h17m   v1.32.1

 get allesartison@kubermgt01:~$ kubectl get all --ignore-not-found
NAME                 TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.128.1   <none>        443/TCP   3h32m

esartison@kubermgt01:~$ kubectl get nodes --ignore-not-found
NAME                        STATUS   ROLES    AGE     VERSION
cl1o2cumtlodjr8887jm-apez   Ready    <none>   3h17m   v1.32.1
cl1o2cumtlodjr8887jm-ubor   Ready    <none>   3h17m   v1.32.1
cl1o2cumtlodjr8887jm-utaq   Ready    <none>   3h17m   v1.32.1
```
Kubernetes cluster готов для работы и можно проводить работы по установке Postgres-а.





## **(3) Установки Postgres через Helm**
В этом пункте будем устанавливать Postgres с помощью Helm 

Установка утилиты Helm
```
esartison@kubermgt01:~$ sudo apt install snapd
esartison@kubermgt01:~$ sudo snap install helm --classic
2025-09-21T09:55:25Z INFO Waiting for automatic snapd restart...
helm 3.19.0 from Snapcrafters✪ installed
esartison@kubermgt01:~$
```

Установка Helm Chart-а Postgresql-ha
```
esartison@kubermgt01:~$ helm install my-release oci://registry-1.docker.io/bitnamicharts/postgresql-ha
Pulled: registry-1.docker.io/bitnamicharts/postgresql-ha:16.3.2
Digest: sha256:78ae138a11c4f6f058fcb18c94e57b0bb52b1b557f92975f70c0dd74b4eb2d94
NAME: my-release
LAST DEPLOYED: Sun Sep 21 09:58:05 2025
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: postgresql-ha
CHART VERSION: 16.3.2
APP VERSION: 17.6.0

⚠ WARNING: Since August 28th, 2025, only a limited subset of images/charts are available for free.
    Subscribe to Bitnami Secure Images to receive continued support and security updates.
    More info at https://bitnami.com and https://github.com/bitnami/containers/issues/83267

** Please be patient while the chart is being deployed **
PostgreSQL can be accessed through Pgpool-II via port 5432 on the following DNS name from within your cluster:

    my-release-postgresql-ha-pgpool.default.svc.cluster.local

Pgpool-II acts as a load balancer for PostgreSQL and forward read/write connections to the primary node while read-only connections are forwarded to standby nodes.

To get the password for "postgres" user run:

    export PASSWORD=$(kubectl get secret --namespace "default" my-release-postgresql-ha-postgresql -o jsonpath="{.data.password}" | base64 -d)

To connect to your database run the following command:

    kubectl run my-release-postgresql-ha-client --rm --tty -i --restart='Never' --namespace default \
        --image docker.io/bitnami/postgresql-repmgr:17.6.0-debian-12-r2 --env="PGPASSWORD=$PASSWORD"  \
        --command -- psql -h my-release-postgresql-ha-pgpool -p 5432 -U postgres -d postgres

To connect to your database from outside the cluster execute the following commands:

    kubectl port-forward --namespace default svc/my-release-postgresql-ha-pgpool 5432:5432 &
    psql -h 127.0.0.1 -p 5432 -U postgres -d postgres

WARNING: There are "resources" sections in the chart not set. Using "resourcesPreset" is not recommended for production. For production installations, please set the following values according to your workload needs:
  - postgresql.resources
  - pgpool.resources
+info https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/
```

Прверка POD-в
```
esartison@kubermgt01:~$ kubectl get all
NAME                                                   READY   STATUS    RESTARTS        AGE
pod/my-release-postgresql-ha-pgpool-6fc4b5d57c-xxhkx   1/1     Running   1 (3m24s ago)   4m25s
pod/my-release-postgresql-ha-postgresql-0              1/1     Running   0               4m25s
pod/my-release-postgresql-ha-postgresql-1              1/1     Running   0               4m25s
pod/my-release-postgresql-ha-postgresql-2              1/1     Running   0               4m25s

NAME                                                   TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
service/kubernetes                                     ClusterIP   10.96.128.1    <none>        443/TCP    5h6m
service/my-release-postgresql-ha-pgpool                ClusterIP   10.96.183.88   <none>        5432/TCP   4m25s
service/my-release-postgresql-ha-postgresql            ClusterIP   10.96.248.95   <none>        5432/TCP   4m25s
service/my-release-postgresql-ha-postgresql-headless   ClusterIP   None           <none>        5432/TCP   4m25s

NAME                                              READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/my-release-postgresql-ha-pgpool   1/1     1            1           4m25s

NAME                                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/my-release-postgresql-ha-pgpool-6fc4b5d57c   1         1         1       4m25s

NAME                                                   READY   AGE
statefulset.apps/my-release-postgresql-ha-postgresql   3/3     4m25s
```

Проверим как POD-ы распределились по узлам(node), видим что мастер и 2 реплики были распределены по разным узлам
```
esartison@kubermgt01:~$ kubectl get all -o wide
NAME                                                   READY   STATUS    RESTARTS     AGE   IP             NODE                        NOMINATED NODE   READINESS GATES
pod/my-release-postgresql-ha-pgpool-6fc4b5d57c-xxhkx   1/1     Running   1 (9m ago)   10m   10.112.129.4   cl1o2cumtlodjr8887jm-ubor   <none>           <none>
pod/my-release-postgresql-ha-postgresql-0              1/1     Running   0            10m   10.112.130.4   cl1o2cumtlodjr8887jm-apez   <none>           <none>
pod/my-release-postgresql-ha-postgresql-1              1/1     Running   0            10m   10.112.129.5   cl1o2cumtlodjr8887jm-ubor   <none>           <none>
pod/my-release-postgresql-ha-postgresql-2              1/1     Running   0            10m   10.112.128.6   cl1o2cumtlodjr8887jm-utaq   <none>           <none>

NAME                                                   TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE     SELECTOR
service/kubernetes                                     ClusterIP   10.96.128.1    <none>        443/TCP    5h11m   <none>
service/my-release-postgresql-ha-pgpool                ClusterIP   10.96.183.88   <none>        5432/TCP   10m     app.kubernetes.io/component=pgpool,app.kubernetes.io/instance=my-release,app.kubernetes.io/name=postgresql-ha
service/my-release-postgresql-ha-postgresql            ClusterIP   10.96.248.95   <none>        5432/TCP   10m     app.kubernetes.io/component=postgresql,app.kubernetes.io/instance=my-release,app.kubernetes.io/name=postgresql-ha,role=data
service/my-release-postgresql-ha-postgresql-headless   ClusterIP   None           <none>        5432/TCP   10m     app.kubernetes.io/component=postgresql,app.kubernetes.io/instance=my-release,app.kubernetes.io/name=postgresql-ha,role=data

NAME                                              READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                        SELECTOR
deployment.apps/my-release-postgresql-ha-pgpool   1/1     1            1           10m   pgpool       docker.io/bitnami/pgpool:4.6.3-debian-12-r0   app.kubernetes.io/component=pgpool,app.kubernetes.io/instance=my-release,app.kubernetes.io/name=postgresql-ha

NAME                                                         DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES                                        SELECTOR
replicaset.apps/my-release-postgresql-ha-pgpool-6fc4b5d57c   1         1         1       10m   pgpool       docker.io/bitnami/pgpool:4.6.3-debian-12-r0   app.kubernetes.io/component=pgpool,app.kubernetes.io/instance=my-release,app.kubernetes.io/name=postgresql-ha,pod-template-hash=6fc4b5d57c

NAME                                                   READY   AGE   CONTAINERS   IMAGES
statefulset.apps/my-release-postgresql-ha-postgresql   3/3     10m   postgresql   docker.io/bitnami/postgresql-repmgr:17.6.0-debian-12-r2
```


Подключимся к нашей базе данных
```
# выставим переменные 
export POSTGRES_PASSWORD=$(kubectl get secret --namespace default my-release-postgresql-ha-postgresql -o jsonpath="{.data.password}" | base64 -d)
export REPMGR_PASSWORD=$(kubectl get secret --namespace default my-release-postgresql-ha-postgresql -o jsonpath="{.data.repmgr-password}" | base64 -d)

# включим перенаправление портов, чтобы можно было подключаться с локального клиента, добавим в конце & чтобы команда выполнялась в фоне
kubectl port-forward --namespace default svc/my-release-postgresql-ha-pgpool 5432:5432 &

# в этом же окне подключиться к базе
psql -h 127.0.0.1 -p 5432 -U postgres -d postgres 
```
<img width="648" height="161" alt="image" src="https://github.com/user-attachments/assets/cd00c8cf-b42f-4081-9075-e092a47b3a33" />


Эмуляция отказа мастер узла
(a) подключаемся и узнаем какой из POD-в является мастером. В нашем случае мастер my-release-postgresql-ha-postgresql-0+
<img width="618" height="341" alt="image" src="https://github.com/user-attachments/assets/f2e09650-1c7a-4a7e-82e9-83093d3964f5" />
(b) Удаляем под для мастера
<img width="931" height="369" alt="image" src="https://github.com/user-attachments/assets/63535931-b472-4f09-8fb6-d58088b6eda2" />
(c) Пробуем еще раз подключиться и нас перенаправляет на my-release-postgresql-ha-postgresql-1+
<img width="614" height="335" alt="image" src="https://github.com/user-attachments/assets/52ddaf35-767f-403c-a147-98cd8f3e7643" />

Автоматический failover работает. 

Удаление Helm Chart-а
<img width="1163" height="294" alt="image" src="https://github.com/user-attachments/assets/bd2adf53-dfe5-4baf-8b8c-bb483ff063ce" />

Недавно, было анонсировано что Bitnami вводит плату за пользование их чартами
<img width="811" height="73" alt="image" src="https://github.com/user-attachments/assets/d85d42cc-7e10-417d-bc83-a78d7da5a5f3" />


## **(2) Установки Postgres через CloudNativePG**

Установка Оператора
```
esartison@kubermgt01:~/CloudNativePG$ kubectl apply -f https://raw.githubusercontent.com/cloudnative-pg/cloudnative-pg/release-1.22/releases/cnpg-1.22.1.yaml
namespace/cnpg-system configured
customresourcedefinition.apiextensions.k8s.io/backups.postgresql.cnpg.io configured
customresourcedefinition.apiextensions.k8s.io/clusters.postgresql.cnpg.io configured
Warning: resource customresourcedefinitions/poolers.postgresql.cnpg.io is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
customresourcedefinition.apiextensions.k8s.io/poolers.postgresql.cnpg.io configured
customresourcedefinition.apiextensions.k8s.io/scheduledbackups.postgresql.cnpg.io configured
serviceaccount/cnpg-manager configured
clusterrole.rbac.authorization.k8s.io/cnpg-manager configured
clusterrolebinding.rbac.authorization.k8s.io/cnpg-manager-rolebinding configured
configmap/cnpg-default-monitoring configured
service/cnpg-webhook-service configured
deployment.apps/cnpg-controller-manager unchanged
mutatingwebhookconfiguration.admissionregistration.k8s.io/cnpg-mutating-webhook-configuration configured
validatingwebhookconfiguration.admissionregistration.k8s.io/cnpg-validating-webhook-configuration configured

esartison@kubermgt01:~/CloudNativePG$ kubectl get deployment -n cnpg-system
NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
cnpg-controller-manager   1/1     1            1           6h14m
```


Устанока Longhorn storage class-а
```
# установка longhorn
esartison@kubermgt01:~$ kubectl apply -f https://raw.githubusercontent.com/longhorn/longhorn/v1.9.1/deploy/longhorn.yaml
namespace/longhorn-system unchanged
priorityclass.scheduling.k8s.io/longhorn-critical configured
serviceaccount/longhorn-service-account unchanged
serviceaccount/longhorn-ui-service-account unchanged
serviceaccount/longhorn-support-bundle unchanged
configmap/longhorn-default-resource unchanged
configmap/longhorn-default-setting unchanged
configmap/longhorn-storageclass unchanged
customresourcedefinition.apiextensions.k8s.io/backingimagedatasources.longhorn.io unchanged
customresourcedefinition.apiextensions.k8s.io/backingimagemanagers.longhorn.io unchanged
customresourcedefinition.apiextensions.k8s.io/backingimages.longhorn.io unchanged
customresourcedefinition.apiextensions.k8s.io/backupbackingimages.longhorn.io unchanged
customresourcedefinition.apiextensions.k8s.io/backups.longhorn.io unchanged
customresourcedefinition.apiextensions.k8s.io/backuptargets.longhorn.io unchanged
customresourcedefinition.apiextensions.k8s.io/backupvolumes.longhorn.io unchanged
customresourcedefinition.apiextensions.k8s.io/engineimages.longhorn.io unchanged
customresourcedefinition.apiextensions.k8s.io/engines.longhorn.io unchanged
customresourcedefinition.apiextensions.k8s.io/instancemanagers.longhorn.io unchanged
customresourcedefinition.apiextensions.k8s.io/nodes.longhorn.io unchanged
customresourcedefinition.apiextensions.k8s.io/orphans.longhorn.io unchanged
customresourcedefinition.apiextensions.k8s.io/recurringjobs.longhorn.io unchanged
customresourcedefinition.apiextensions.k8s.io/replicas.longhorn.io unchanged
customresourcedefinition.apiextensions.k8s.io/settings.longhorn.io unchanged
customresourcedefinition.apiextensions.k8s.io/sharemanagers.longhorn.io unchanged
customresourcedefinition.apiextensions.k8s.io/snapshots.longhorn.io unchanged
customresourcedefinition.apiextensions.k8s.io/supportbundles.longhorn.io unchanged
customresourcedefinition.apiextensions.k8s.io/systembackups.longhorn.io unchanged
customresourcedefinition.apiextensions.k8s.io/systemrestores.longhorn.io unchanged
customresourcedefinition.apiextensions.k8s.io/volumeattachments.longhorn.io unchanged
customresourcedefinition.apiextensions.k8s.io/volumes.longhorn.io unchanged
clusterrole.rbac.authorization.k8s.io/longhorn-role unchanged
clusterrolebinding.rbac.authorization.k8s.io/longhorn-bind unchanged
clusterrolebinding.rbac.authorization.k8s.io/longhorn-support-bundle unchanged
service/longhorn-backend unchanged
service/longhorn-frontend configured
service/longhorn-conversion-webhook unchanged
service/longhorn-admission-webhook unchanged
service/longhorn-recovery-backend unchanged
daemonset.apps/longhorn-manager unchanged
deployment.apps/longhorn-driver-deployer unchanged
deployment.apps/longhorn-ui unchanged

# создание ConfigMap-а
esartison@kubermgt01:~$ cat apply.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: longhorn-storageclass-1r
  namespace: longhorn-system
  labels:
    app.kubernetes.io/name: longhorn
    app.kubernetes.io/instance: longhorn
    app.kubernetes.io/version: v1.5.3
data:
  storageclass.yaml: |
    kind: StorageClass
    apiVersion: storage.k8s.io/v1
    metadata:
      name: longhorn-1r
      annotations:
        storageclass.kubernetes.io/is-default-class: "true"
    provisioner: driver.longhorn.io
    allowVolumeExpansion: true
    reclaimPolicy: "Delete"
    volumeBindingMode: Immediate
    parameters:
      numberOfReplicas: "1"
      staleReplicaTimeout: "30"
      fromBackup: ""
      fsType: "ext4"
      dataLocality: "disabled"
	  
	  
esartison@kubermgt01:~$ kubectl apply -f apply.yaml
configmap/longhorn-storageclass-1r created
```

Создание namespace-а
```
esartison@kubermgt01:~/CloudNativePG$ kubectl create namespace cnpg-kuber
namespace/cnpg-kuber created
```


# Создание Postgres кластера с минимальным набором настроек и отработка простых сценариев

Создание Postgres кластера и подключение к базе данных
```
# Yaml файл
esartison@kubermgt01:~/CloudNativePG$ cat cluster-example.yaml
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: pg-simple-cluster
spec:
  instances: 3

  storage:
    size: 1Gi

# Создание кластера посредством применения yaml файла
esartison@kubermgt01:~/CloudNativePG$ kubectl apply -f cluster-example.yaml
cluster.postgresql.cnpg.io/pg-simple-cluster created

#Проверка состояния POD-в
esartison@kubermgt01:~/CloudNativePG$  kubectl get all -o wide
NAME                                   READY   STATUS      RESTARTS   AGE   IP             NODE                        NOMINATED NODE   READINESS GATES
pod/pg-simple-cluster-1                1/1     Running     0          31s   10.112.129.9   cl1o2cumtlodjr8887jm-ubor   <none>           <none>
pod/pg-simple-cluster-1-initdb-h54xj   0/1     Completed   0          86s   10.112.129.8   cl1o2cumtlodjr8887jm-ubor   <none>           <none>
pod/pg-simple-cluster-2-join-bs4v2     0/1     Init:0/1    0          14s   <none>         cl1o2cumtlodjr8887jm-apez   <none>           <none>

NAME                           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE   SELECTOR
service/kubernetes             ClusterIP   10.96.128.1     <none>        443/TCP    21h   <none>
service/pg-simple-cluster-r    ClusterIP   10.96.165.64    <none>        5432/TCP   87s   cnpg.io/cluster=pg-simple-cluster,cnpg.io/podRole=instance
service/pg-simple-cluster-ro   ClusterIP   10.96.176.150   <none>        5432/TCP   87s   cnpg.io/cluster=pg-simple-cluster,role=replica
service/pg-simple-cluster-rw   ClusterIP   10.96.156.86    <none>        5432/TCP   87s   cnpg.io/cluster=pg-simple-cluster,role=primary

NAME                                   STATUS     COMPLETIONS   DURATION   AGE   CONTAINERS   IMAGES                                   SELECTOR
job.batch/pg-simple-cluster-1-initdb   Complete   1/1           55s        86s   initdb       ghcr.io/cloudnative-pg/postgresql:16.1   batch.kubernetes.io/controller-uid=eeb75b7b-094b-4168-81d8-459982628442
job.batch/pg-simple-cluster-2-join     Running    0/1           14s        14s   join         ghcr.io/cloudnative-pg/postgresql:16.1   batch.kubernetes.io/controller-uid=a6b82fd2-88dc-4ca3-b930-82e70299bbf5
esartison@kubermgt01:~/CloudNativePG$
esartison@kubermgt01:~/CloudNativePG$
esartison@kubermgt01:~/CloudNativePG$
esartison@kubermgt01:~/CloudNativePG$ kubectl apply -f cluster-example.yaml
cluster.postgresql.cnpg.io/pg-simple-cluster unchanged
esartison@kubermgt01:~/CloudNativePG$ kubectl get all -o wide
NAME                      READY   STATUS    RESTARTS   AGE     IP              NODE                        NOMINATED NODE   READINESS GATES
pod/pg-simple-cluster-1   1/1     Running   0          2m43s   10.112.129.9    cl1o2cumtlodjr8887jm-ubor   <none>           <none>
pod/pg-simple-cluster-2   1/1     Running   0          93s     10.112.130.10   cl1o2cumtlodjr8887jm-apez   <none>           <none>
pod/pg-simple-cluster-3   1/1     Running   0          23s     10.112.128.10   cl1o2cumtlodjr8887jm-utaq   <none>           <none>

NAME                           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE     SELECTOR
service/kubernetes             ClusterIP   10.96.128.1     <none>        443/TCP    21h     <none>
service/pg-simple-cluster-r    ClusterIP   10.96.165.64    <none>        5432/TCP   3m39s   cnpg.io/cluster=pg-simple-cluster,cnpg.io/podRole=instance
service/pg-simple-cluster-ro   ClusterIP   10.96.176.150   <none>        5432/TCP   3m39s   cnpg.io/cluster=pg-simple-cluster,role=replica
service/pg-simple-cluster-rw   ClusterIP   10.96.156.86    <none>        5432/TCP   3m39s   cnpg.io/cluster=pg-simple-cluster,role=primary

# Создание Load Balancer для подключения из вне
esartison@kubermgt01:~/CloudNativePG$ kubectl -n default expose service pg-simple-cluster-rw  --name=test-cnpg-bones-lb --port=5432 --type=LoadBalancer

# Получение секретов для подключения к базе данных
esartison@kubermgt01:~/CloudNativePG$ kubectl get secret pg-simple-cluster-app -o jsonpath='{.data}'
{"dbname":"YXBw","host":"cGctc2ltcGxlLWNsdXN0ZXItcnc=","jdbc-uri":"amRiYzpwb3N0Z3Jlc3FsOi8vcGctc2ltcGxlLWNsdXN0ZXItcnc6NTQzMi9hcHA/cGFzc3dvcmQ9TFNyWjJoTktQaWFxT0lWSjZjbFVDdFJhNUVTNEppdXFrM2FRMHdwOUwzSHh3dWlKS1VRUk4wSHVBOE5aVGJPbiZ1c2VyPWFwcA==","password":"TFNyWjJoTktQaWFxT0lWSjZjbFVDdFJhNUVTNEppdXFrM2FRMHdwOUwzSHh3dWlKS1VRUk4wSHVBOE5aVGJPbg==","pgpass":"cGctc2ltcGxlLWNsdXN0ZXItcnc6NTQzMjphcHA6YXBwOkxTcloyaE5LUGlhcU9JVko2Y2xVQ3RSYTVFUzRKaXVxazNhUTB3cDlMM0h4d3VpSktVUVJOMEh1QThOWlRiT24K","port":"NTQzMg==","uri":"cG9zdGdyZXNxbDovL2FwcDpMU3JaMmhOS1BpYXFPSVZKNmNsVUN0UmE1RVM0Sml1cWszYVEwd3A5TDNIeHd1aUpLVVFSTjBIdUE4TlpUYk9uQHBnLXNpbXBsZS1jbHVzdGVyLXJ3OjU0MzIvYXBw","user":"YXBw","username":"YXBw"}esartison@kubermgt01:~/CloudNativePG$ echo 'cGctc2ltcGxlLWNsdXN0ZXItcnc6NTQzMjphcHA6YXBwOkxTcloyaE5LUGlhcU9JVko2Y2xVQ3RSYTVFUzRKaXVxazNhUTB3cDlMM0h4d3VpSktVUVJOMEh1QThOWlRiT24K'  | base64 --decode
pg-simple-cluster-rw:5432:app:app:LSrZ2hNKPiaqOIVJ6clUCtRa5ES4Jiuqk3aQ0wp9L3HxwuiJKUQRN0HuA8NZTbOn
esartison@kubermgt01:~/CloudNativePG$ export PGPASSWORD=LSrZ2hNKPiaqOIVJ6clUCtRa5ES4Jiuqk3aQ0wp9L3HxwuiJKUQRN0HuA8NZTbOn

# Подключение к базе данных
esartison@kubermgt01:~/CloudNativePG$ psql -h 158.160.195.68 -U app -d app
app=>
app=>
app=> \l
                                                  List of databases
   Name    |  Owner   | Encoding | Locale Provider | Collate | Ctype | ICU Locale | ICU Rules |   Access privileges
-----------+----------+----------+-----------------+---------+-------+------------+-----------+-----------------------
 app       | app      | UTF8     | libc            | C       | C     |            |           |
 postgres  | postgres | UTF8     | libc            | C       | C     |            |           |
 template0 | postgres | UTF8     | libc            | C       | C     |            |           | =c/postgres          +
           |          |          |                 |         |       |            |           | postgres=CTc/postgres
 template1 | postgres | UTF8     | libc            | C       | C     |            |           | =c/postgres          +
           |          |          |                 |         |       |            |           | postgres=CTc/postgres
(4 rows)
```
В итоге было создано 1 мастер база и 2 реплики.


Сокращение кол-ва экземпляров с 3 до 2х
```
esartison@kubermgt01:~/CloudNativePG$ diff cluster-example.yaml cluster-example2.yaml
6c6
<   instances: 3
---
>   instances: 2

esartison@kubermgt01:~/CloudNativePG$ kubectl apply -f cluster-example2.yaml
cluster.postgresql.cnpg.io/pg-simple-cluster configured
esartison@kubermgt01:~/CloudNativePG$  kubectl get all -o wide
NAME                      READY   STATUS    RESTARTS   AGE   IP              NODE                        NOMINATED NODE   READINESS GATES
pod/pg-simple-cluster-1   1/1     Running   0          92m   10.112.129.9    cl1o2cumtlodjr8887jm-ubor   <none>           <none>
pod/pg-simple-cluster-2   1/1     Running   0          91m   10.112.130.10   cl1o2cumtlodjr8887jm-apez   <none>           <none>

NAME                           TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)          AGE   SELECTOR
service/kubernetes             ClusterIP      10.96.128.1     <none>           443/TCP          23h   <none>
service/pg-simple-cluster-r    ClusterIP      10.96.165.64    <none>           5432/TCP         93m   cnpg.io/cluster=pg-simple-cluster,cnpg.io/podRole=instance
service/pg-simple-cluster-ro   ClusterIP      10.96.176.150   <none>           5432/TCP         93m   cnpg.io/cluster=pg-simple-cluster,role=replica
service/pg-simple-cluster-rw   ClusterIP      10.96.156.86    <none>           5432/TCP         93m   cnpg.io/cluster=pg-simple-cluster,role=primary
service/test-cnpg-bones-lb     LoadBalancer   10.96.169.23    158.160.195.68   5432:32621/TCP   58m   cnpg.io/cluster=pg-simple-cluster,role=primary
```
Кол-во экземпляров сократилось до 2х. 1 мастер и 1 реплика. 



Минорный апгрейди увеличение реплик с 1й до 2х
```
# проверка версии ДО - 16.2
esartison@kubermgt01:~/CloudNativePG$ psql -h 158.160.195.68 -U app -d app
app=> select version();
                                                           version
-----------------------------------------------------------------------------------------------------------------------------
 PostgreSQL 16.2 (Debian 16.2-1.pgdg110+2) on x86_64-pc-linux-gnu, compiled by gcc (Debian 10.2.1-6) 10.2.1 20210110, 64-bit
(1 row)

# Правки в YAML файле
esartison@kubermgt01:~/CloudNativePG$ diff cluster-example-upgrade.yaml cluster-example2.yaml
6,7c6
<   imageName: ghcr.io/cloudnative-pg/postgresql:16.10
<   instances: 3
---
>   instances: 2

# приминение конфигурации
esartison@kubermgt01:~/CloudNativePG$ kubectl apply -f cluster-example-upgrade.yaml
cluster.postgresql.cnpg.io/pg-simple-cluster configured

# проверка версии ПОСЛЕ - 16.10
esartison@kubermgt01:~/CloudNativePG$ psql -h 158.160.195.68 -U app -d app
app=> select version();
                                                           version
------------------------------------------------------------------------------------------------------------------------------
 PostgreSQL 16.10 (Debian 16.10-1.pgdg11+1) on x86_64-pc-linux-gnu, compiled by gcc (Debian 10.2.1-6) 10.2.1 20210110, 64-bit
(1 row)

# проверка что кол-во версий увеличилось
esartison@kubermgt01:~/CloudNativePG$ kubectl get all -o wide
NAME                      READY   STATUS    RESTARTS   AGE     IP              NODE                        NOMINATED NODE   READINESS GATES
pod/pg-simple-cluster-1   1/1     Running   0          8m15s   10.112.129.10   cl1o2cumtlodjr8887jm-ubor   <none>           <none>
pod/pg-simple-cluster-2   1/1     Running   0          8m47s   10.112.130.11   cl1o2cumtlodjr8887jm-apez   <none>           <none>
pod/pg-simple-cluster-4   1/1     Running   0          9m8s    10.112.128.12   cl1o2cumtlodjr8887jm-utaq   <none>           <none>

NAME                           TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)          AGE    SELECTOR
service/kubernetes             ClusterIP      10.96.128.1     <none>           443/TCP          23h    <none>
service/pg-simple-cluster-r    ClusterIP      10.96.165.64    <none>           5432/TCP         122m   cnpg.io/cluster=pg-simple-cluster,cnpg.io/podRole=instance
service/pg-simple-cluster-ro   ClusterIP      10.96.176.150   <none>           5432/TCP         122m   cnpg.io/cluster=pg-simple-cluster,role=replica
service/pg-simple-cluster-rw   ClusterIP      10.96.156.86    <none>           5432/TCP         122m   cnpg.io/cluster=pg-simple-cluster,role=primary
service/test-cnpg-bones-lb     LoadBalancer   10.96.169.23    158.160.195.68   5432:32621/TCP   88m    cnpg.io/cluster=pg-simple-cluster,role=primary
```

Удаление кластера
```
esartison@kubermgt01:~/CloudNativePG$     kubectl get cluster --all-namespaces
NAMESPACE   NAME                AGE   INSTANCES   READY   STATUS                                       PRIMARY
default     pg-simple-cluster   16h   3           1       Waiting for the instances to become active   pg-simple-cluster-1
esartison@kubermgt01:~/CloudNativePG$  kubectl delete cluster pg-simple-cluster n default
cluster.postgresql.cnpg.io "pg-simple-cluster" deleted from default namespace
```


#Создание Postgres кластера с более продвинутым набором настроек и отработка простых сценариев
```
esartison@kubermgt01:~/CloudNativePG$ cat advanced_PG_cluster.yaml

apiVersion: v1
data:
  password: VHhWZVE0bk44MlNTaVlIb3N3cU9VUlp2UURhTDRLcE5FbHNDRUVlOWJ3RHhNZDczS2NrSWVYelM1Y1U2TGlDMg==
  username: YXBw
kind: Secret
metadata:
  name: pgotuscluster-app-user
type: kubernetes.io/basic-auth
---
apiVersion: v1
data:
  password: dU4zaTFIaDBiWWJDYzRUeVZBYWNCaG1TemdxdHpxeG1PVmpBbjBRSUNoc0pyU211OVBZMmZ3MnE4RUtLTHBaOQ==
  username: cG9zdGdyZXM=
kind: Secret
metadata:
  name: pgotuscluster-superuser
type: kubernetes.io/basic-auth
---
apiVersion: v1
kind: Secret
metadata:
  name: backup-creds
data:
  ACCESS_KEY_ID: a2V5X2lk
  ACCESS_SECRET_KEY: c2VjcmV0X2tleQ==
---
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: pgotuscluster-full
spec:
  description: "Example of cluster"
  imageName: ghcr.io/cloudnative-pg/postgresql:17.5
  # imagePullSecret is only required if the images are located in a private registry
  # imagePullSecrets:
  #   - name: private_registry_access
  instances: 3
  startDelay: 300
  stopDelay: 300
  primaryUpdateStrategy: unsupervised

  postgresql:
    parameters:
      shared_buffers: 256MB
      pg_stat_statements.max: '10000'
      pg_stat_statements.track: all
      auto_explain.log_min_duration: '10s'
    pg_hba:
      - host all all 10.244.0.0/16 md5

  bootstrap:
    initdb:
      database: app
      owner: app
      secret:
        name: pgotuscluster-app-user
    # Alternative bootstrap method: start from a backup
    #recovery:
    #  backup:
    #    name: backup-example

  enableSuperuserAccess: true
  superuserSecret:
    name: pgotuscluster-superuser

  storage:
    size: 1Gi

  backup:
    barmanObjectStore:
      destinationPath: s3://pgotuscluster-full-backup/
      endpointURL: http://custom-endpoint:1234
      s3Credentials:
        accessKeyId:
          name: backup-creds
          key: ACCESS_KEY_ID
        secretAccessKey:
          name: backup-creds
          key: ACCESS_SECRET_KEY
      wal:
        compression: gzip
        encryption: AES256
      data:
        compression: gzip
        encryption: AES256
        immediateCheckpoint: false
        jobs: 2
    retentionPolicy: "30d"

  resources:
    requests:
      memory: "512Mi"
      cpu: "1"
    limits:
      memory: "1Gi"
      cpu: "2"

  affinity:
    enablePodAntiAffinity: false
    topologyKey: failure-domain.beta.kubernetes.io/zone

  nodeMaintenanceWindow:
    inProgress: false
    reusePVC: false


esartison@kubermgt01:~/CloudNativePG$ kubectl apply -f advanced_PG_cluster.yaml
secret/pgotuscluster-app-user created
secret/pgotuscluster-superuser created
secret/backup-creds created
cluster.postgresql.cnpg.io/pgotuscluster-full created
esartison@kubermgt01:~/CloudNativePG$
```


```
esartison@kubermgt01:~/CloudNativePG$ kubectl get all -o wide
NAME                       READY   STATUS    RESTARTS   AGE     IP              NODE                        NOMINATED NODE   READINESS GATES
pod/pgotuscluster-full-1   1/1     Running   0          4m20s   10.112.130.16   cl1o2cumtlodjr8887jm-apez   <none>           <none>
pod/pgotuscluster-full-2   1/1     Running   0          2m47s   10.112.129.18   cl1o2cumtlodjr8887jm-ubor   <none>           <none>
pod/pgotuscluster-full-3   1/1     Running   0          75s     10.112.128.25   cl1o2cumtlodjr8887jm-utaq   <none>           <none>

NAME                            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE     SELECTOR
service/kubernetes              ClusterIP   10.96.128.1     <none>        443/TCP    5m47s   <none>
service/pgotuscluster-full-r    ClusterIP   10.96.151.151   <none>        5432/TCP   5m24s   cnpg.io/cluster=pgotuscluster-full,cnpg.io/podRole=instance
service/pgotuscluster-full-ro   ClusterIP   10.96.170.28    <none>        5432/TCP   5m24s   cnpg.io/cluster=pgotuscluster-full,role=replica
service/pgotuscluster-full-rw   ClusterIP   10.96.244.224   <none>        5432/TCP   5m24s   cnpg.io/cluster=pgotuscluster-full,role=primary
```






#Изменение параметра в Postgres в бегущем экземпляре


esartison@kubermgt01:~/CloudNativePG$ kubectl exec -it pod/pgotuscluster-full-1 -- psql -U postgres
Defaulted container "postgres" out of: postgres, bootstrap-controller (init)
psql (17.5 (Debian 17.5-1.pgdg110+1))
Type "help" for help.

postgres=# show log_statement;
 log_statement
---------------
 none
(1 row)


esartison@kubermgt01:~/CloudNativePG$ cat cluster-example-update_parameter.yaml
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: pgotuscluster-full
spec:
  description: "Example of cluster"
  imageName: ghcr.io/cloudnative-pg/postgresql:17.5

  postgresql:
    parameters:
      log_statement: ddl

  storage:
    size: 1Gi

esartison@kubermgt01:~/CloudNativePG$ kubectl apply -f cluster-example-update_parameter.yaml
cluster.postgresql.cnpg.io/pgotuscluster-full configured

esartison@kubermgt01:~/CloudNativePG$ kubectl exec -it pod/pgotuscluster-full-1 -- psql -U postgres
Defaulted container "postgres" out of: postgres, bootstrap-controller (init)
psql (17.5 (Debian 17.5-1.pgdg110+1))
Type "help" for help.

postgres=# show log_statement;
 log_statement
---------------
 ddl
(1 row)






#Monitoring
https://grafana.com/grafana/dashboards/20417-cloudnativepg/


esartison@kubermgt01:~/CloudNativePG$ kubectl apply -f cluster-example-update_monitoring.yaml
cluster.postgresql.cnpg.io/pgotuscluster-full configured

esartison@kubermgt01:~/CloudNativePG$ kubectl apply -f cluster-example-update_monitoring.yaml
cluster.postgresql.cnpg.io/pgotuscluster-full configured
esartison@kubermgt01:~/CloudNativePG$ cat cluster-example-update_monitoring.yaml
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: pgotuscluster-full
spec:
  description: "Example of cluster"
  imageName: ghcr.io/cloudnative-pg/postgresql:17.5

  storage:
    size: 1Gi
  monitoring:
    enablePodMonitor: true
esartison@kubermgt01:~/CloudNativePG$








esartison@kubermgt01:~/CloudNativePG$ cat cluster-example-update_add_metric.yaml
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: pgotuscluster-full
spec:
  description: "Example of cluster"
  imageName: ghcr.io/cloudnative-pg/postgresql:17.5
  instances: 3

  storage:
    size: 1Gi
  monitoring:
    customQueriesConfigMap:
      - name: additional-monitoring
        key:  custom-queries
esartison@kubermgt01:~/CloudNativePG$ kubectl apply -f cluster-example-update_add_metric.yaml
cluster.postgresql.cnpg.io/pgotuscluster-full configured









esartison@kubermgt01:~/CloudNativePG$ cat cluster-example-update_add_conf_map.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: additional-monitoring
  namespace: default
  labels:
    cnpg.io/reload: ""
data:
  custom-queries: |
    pg_replication:
      query: "SELECT CASE WHEN NOT pg_is_in_recovery()
              THEN 0
              ELSE GREATEST (0,
                EXTRACT(EPOCH FROM (now() - pg_last_xact_replay_timestamp())))
              END AS lag,
              pg_is_in_recovery() AS in_recovery,
              EXISTS (TABLE pg_stat_wal_receiver) AS is_wal_receiver_up,
              (SELECT count(*) FROM pg_stat_replication) AS streaming_replicas"

      metrics:
        - lag:
            usage: "GAUGE"
            description: "Replication lag behind primary in seconds"
        - in_recovery:
            usage: "GAUGE"
            description: "Whether the instance is in recovery"
        - is_wal_receiver_up:
            usage: "GAUGE"
            description: "Whether the instance wal_receiver is up"
        - streaming_replicas:
            usage: "GAUGE"
            description: "Number of streaming replicas connected to the instance"

esartison@kubermgt01:~/CloudNativePG$ kubectl apply -f cluster-example-update_add_conf_map.yaml
configmap/additional-monitoring created





https://raw.githubusercontent.com/cloudnative-pg/cloudnative-pg/main/config/manager/default-monitoring.yaml
