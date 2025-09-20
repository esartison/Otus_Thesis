# Otus_Thesis
дипломная работа по Postges под управлением Kubernetes


## **(1) Создать Kubernetes кластер**

Для экономии времени, решил создать managed Kubernetes кластер в Yandex Cloud

### Создаем высокодоступный кластер в 3х разных зонах доступности
<img width="531" height="484" alt="image" src="https://github.com/user-attachments/assets/ef879d8f-cb15-4c05-9e9c-1e2f664818b1" />
Остальные настройки оставляем по умолчанию. 

### Ресурсы: 1 vCPU, 1 ГБ RAM
Создадим VM с Ubuntu для управления нашим Kubernetes cluster



### Выбрал следующую конфигурация
![image](https://github.com/user-attachments/assets/3138052b-9367-4559-b0c6-161a73647826)

### Создал Postgres cluster со следующими настройками

![image](https://github.com/user-attachments/assets/499da419-5137-4cdf-9d2b-955323a2cbed)

