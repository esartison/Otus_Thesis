# Otus_Thesis
дипломная работа по Postges под управлением Kubernetes


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

