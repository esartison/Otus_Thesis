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


### Настройка kubermgt01 для управления нашим Kubernetes cluster
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



## **(2) Установки Postgres через Helm**
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
