# «Запуск приложений в K8S» - Дрибноход Давид

### Задание 1. Создать Deployment и обеспечить доступ к репликам приложения из другого Pod

1. Создать Deployment приложения, состоящего из двух контейнеров — nginx и multitool. Решить возникшую ошибку.

#### Ответ:

Создаю отдельный неймспейс ``` kubectl create namespace 12-03-hw ```

Создаю deployment.yaml, указываю порт 8087 так как 80 уже занят
``` YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-multitool
spec:
  selector:
    matchLabels:
      app: ngxmt
  replicas: 1
  template:
    metadata:
      labels:
        app: ngxmt
    spec:
      containers:
      - name: nginx
        image: nginx:1.25.4
        ports:
        - containerPort: 80
      - name: multitool
        image: wbitt/network-multitool
        ports:
        - containerPort: 8080
        env: 
          - name: HTTP_PORT
            value: "8087"
```
Применяю deployment.yaml
![image](https://github.com/DrDavidN/12-03-hw/assets/128225763/7cd80a60-0feb-4676-af53-91cb9a333da4)

2. После запуска увеличить количество реплик работающего приложения до 2.

#### Ответ:

Увеличиваю кол-во реплик до 2  и выполняю проверку
![image](https://github.com/DrDavidN/12-03-hw/assets/128225763/7fa41f48-7b19-463e-8689-350ca944131f)

3. Продемонстрировать количество подов до и после масштабирования.

#### Ответ:

До
![image](https://github.com/DrDavidN/12-03-hw/assets/128225763/afa1efe9-c071-4e97-af1a-e1784b5276b2)

После
![image](https://github.com/DrDavidN/12-03-hw/assets/128225763/8be0c92a-3fb9-43e5-b783-a366d28cd8f3)

4. Создать Service, который обеспечит доступ до реплик приложений из п.1.

#### Ответ:

Создаю service.yaml с именем nginx-multitool-svc в namespace 12-03-hw. Применяю service.yaml и выполняю проверку:
``` YAML
apiVersion: v1
kind: Service
metadata:
  name: nginx-multitool-svc
  namespace: 12-03-hw
spec:
  selector:
    app: ngxmt
  ports:
    - protocol: TCP
      name: nginx
      port: 80
      targetPort: 80
    - protocol: TCP
      name: multitool
      port: 8080
      targetPort: 8087
```
![image](https://github.com/DrDavidN/12-03-hw/assets/128225763/963e3820-9423-4891-9bb2-6043c5d24db3)

5. Создать отдельный Pod с приложением multitool и убедиться с помощью `curl`, что из пода есть доступ до приложений из п.1.

#### Ответ:

Создаю отдельный pod multitool, применю его, получаю список pods
``` YAML
apiVersion: v1
kind: Pod
metadata:
   name: multitool
   namespace: 12-03-hw
spec:
   containers:
     - name: multitool
       image: wbitt/network-multitool
       ports:
        - containerPort: 8080
```
![image](https://github.com/DrDavidN/12-03-hw/assets/128225763/96c1b865-b6c0-47d9-9888-fde4b90c01d8)

Проверяю доступ с помощью curl до приложения из пункта 1
![image](https://github.com/DrDavidN/12-03-hw/assets/128225763/a43c1a42-119f-4c7b-a5c5-a2901d9e31ae)

------

### Задание 2. Создать Deployment и обеспечить старт основного контейнера при выполнении условий

1. Создать Deployment приложения nginx и обеспечить старт контейнера только после того, как будет запущен сервис этого приложения.

#### Ответ:
``` YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-init-deploy
  namespace: 12-03-hw
spec:
  selector:
    matchLabels:
      app: nginx-init
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx-init
    spec:
      containers:
      - name: nginx
        image: nginx:1.25.4
        ports:
        - containerPort: 80
      initContainers:
      - name: delay
        image: busybox
        command: ['sh', '-c', "until nslookup nginx-init-svc.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for nginx-init-svc; sleep 2; done"]
```
![image](https://github.com/DrDavidN/12-03-hw/assets/128225763/5f84bc22-4999-4c0a-9134-d59c204423b7)

2. Убедиться, что nginx не стартует. В качестве Init-контейнера взять busybox.

#### Ответ:
![image](https://github.com/DrDavidN/12-03-hw/assets/128225763/899ff582-afc2-4ddd-b0e1-928a6e5ee9a5)

pod не запущен и находится в состоянии ```Init:0/1```

3. Создать и запустить Service. Убедиться, что Init запустился.

#### Ответ:

Создаю nginx-init-svc.yaml и применяю его
``` YAML
apiVersion: v1
kind: Service
metadata:
  name: nginx-init-svc
  namespace: 12-03-hw
spec:
  ports:
    - name: nginx-init
      port: 80
  selector:
    app: nginx-init
```
![image](https://github.com/DrDavidN/12-03-hw/assets/128225763/243ddb0b-0ee2-44b5-9ab8-a9560aa2eff0)

4. Продемонстрировать состояние пода до и после запуска сервиса.

#### Ответ:

После запусука service pod также успешно стартовал
![image](https://github.com/DrDavidN/12-03-hw/assets/128225763/6cca80cb-f45c-4535-95be-cfa6f89c3bf3)

------
