# Otus_Thesis
дипломная работа по Postges под управлением Kubernetes


## **(1) Создать Kubernetes кластер**

Для экономии времени, решил создать managed Kubernetes кластер в Yandex Cloud

### Создаем высокодоступный кластер в 3х разных зонах доступности
<img width="531" height="484" alt="image" src="https://github.com/user-attachments/assets/ef879d8f-cb15-4c05-9e9c-1e2f664818b1" />
Остальные настройки оставляем по умолчанию. 

кластер успешно создался
<img width="885" height="67" alt="image" src="https://github.com/user-attachments/assets/99e11dcd-d8e1-46ed-9df4-28977d90c2fa" />
<img width="483" height="290" alt="image" src="https://github.com/user-attachments/assets/5d03fdb9-0be7-4d68-8977-5f6a44adbb1f" />



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

```


### Выбрал следующую конфигурация
![image](https://github.com/user-attachments/assets/3138052b-9367-4559-b0c6-161a73647826)

### Создал Postgres cluster со следующими настройками

![image](https://github.com/user-attachments/assets/499da419-5137-4cdf-9d2b-955323a2cbed)

