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

-Проблема
Не все ноды видны из-за того, что CoreDns процесс не работает. Использовал релизный канал Stable
>kubectl get pods -n kube-system -l k8s-app=kube-dns -o wide
<img width="982" height="52" alt="image" src="https://github.com/user-attachments/assets/6a0a8c3a-69b7-4ffa-968a-609e3d095102" />

При попытке открыть консоль из GUI тоже получал ошибку
<img width="1035" height="257" alt="image" src="https://github.com/user-attachments/assets/f08567a9-8288-440d-80b7-7b30b85dafa7" />
<img width="1089" height="346" alt="image" src="https://github.com/user-attachments/assets/2fe7d1c1-1ff4-435a-aac5-f6995244f763" />

-Решение
Попробовал перезапустить Kubernetes Cluster через GUI
<img width="1018" height="200" alt="image" src="https://github.com/user-attachments/assets/bfa49b01-a983-4102-97a7-e0086009c6d3" />
Не помогло, создаем Kubernetes cluster на одной ноде.


### Создаем НЕвысокодоступный Kubernetes кластер в 1-й зоне
Создаем базовый Kuberntes кластер в 1 зоне доступности
<img width="910" height="373" alt="image" src="https://github.com/user-attachments/assets/a4d3c7b0-5ed7-49f8-83df-24f7337416b2" />


Тоже ошибка 
<img width="979" height="52" alt="image" src="https://github.com/user-attachments/assets/d7525ac1-a835-436e-82df-3b66a1f6bd27" />
Gui консоль не открывается
<img width="756" height="269" alt="image" src="https://github.com/user-attachments/assets/41beaf26-5838-473e-86f9-a2524e73bec7" />


### Создаем НЕвысокодоступный Kubernetes кластер в 1-й зоне с версией 1.30
Пробую создать с самой низкой из доступных версий
<img width="794" height="400" alt="image" src="https://github.com/user-attachments/assets/0f9e7310-240e-48b5-b044-a69c83394b62" />

 



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
esartison@kubermgt01:~$ kubectl version --client --output=yaml
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
esartison@kubermgt01:~$ yc managed-kubernetes cluster get-credentials --id catfgm5va6vueb71unfh --external
Context 'yc-otuskubercluster1' was added as default to kubeconfig '/home/esartison/.kube/config'. Check connection to cluster using 'kubectl cluster-info --kubeconfig /home/esartison/.kube/config'.
Note, that authentication depends on 'yc' and its config profile 'default'. To access clusters using the Kubernetes API, please use Kubernetes Service Account.

esartison@kubermgt01:~$ kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://158.160.179.174
  name: yc-managed-k8s-cat1bhkp5006a840e3f0
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://51.250.42.68
  name: yc-managed-k8s-catfgm5va6vueb71unfh
contexts:
- context:
    cluster: yc-managed-k8s-cat1bhkp5006a840e3f0
    user: yc-managed-k8s-cat1bhkp5006a840e3f0
  name: yc-otuskubercluster
- context:
    cluster: yc-managed-k8s-catfgm5va6vueb71unfh
    user: yc-managed-k8s-catfgm5va6vueb71unfh
  name: yc-otuskubercluster1
current-context: yc-otuskubercluster1
kind: Config
users:
- name: yc-managed-k8s-cat1bhkp5006a840e3f0
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
- name: yc-managed-k8s-catfgm5va6vueb71unfh
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




yc container cluster list
yc container cluster list-node-groups cat1bhkp5006a840e3f0
yc compute disk list
```


### Выбрал следующую конфигурация
![image](https://github.com/user-attachments/assets/3138052b-9367-4559-b0c6-161a73647826)

### Создал Postgres cluster со следующими настройками

![image](https://github.com/user-attachments/assets/499da419-5137-4cdf-9d2b-955323a2cbed)

