# Домашнее задание к занятию «Настройка приложений и управление доступом в Kubernetes» - Студент группы FOPS-42 Власов Владимир

### Примерное время выполнения задания

120 минут

### Цель задания

Научиться:
- Настраивать конфигурацию приложений с помощью **ConfigMaps** и **Secrets**
- Управлять доступом пользователей через **RBAC**

Это задание поможет вам освоить ключевые механизмы Kubernetes для работы с конфигурацией и безопасностью. Эти навыки необходимы для уверенного администрирования кластеров в реальных проектах. На практике навыки используются для:
- Хранения чувствительных данных (Secrets)
- Гибкого управления настройками приложений (ConfigMaps) 
- Контроля доступа пользователей и сервисов (RBAC)

------

## **Подготовка**
### **Чеклист готовности**
- Установлен Kubernetes (MicroK8S, Minikube или другой)
- Установлен `kubectl`
- Редактор для YAML-файлов (VS Code, Vim и др.)
- Утилита `openssl` для генерации сертификатов

------

### Инструменты, которые пригодятся для выполнения задания

1. [Инструкция](https://microk8s.io/docs/getting-started) по установке MicroK8S
2. [Инструкция](https://minikube.sigs.k8s.io/docs/start/) по установке Minikube
3. [Инструкция](https://kubernetes.io/docs/tasks/tools/) по установке kubectl
4. [Инструкция](https://marketplace.visualstudio.com/items?itemName=ms-kubernetes-tools.vscode-kubernetes-tools) по установке VS Code

### Дополнительные материалы, которые пригодятся для выполнения задания

1. [Описание](https://kubernetes.io/docs/concepts/configuration/secret/) Secret.
2. [Описание](https://kubernetes.io/docs/concepts/configuration/configmap/) ConfigMap.
3. [Описание](https://github.com/wbitt/Network-MultiTool) Multitool.
4. [Описание](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) RBAC.
5. [Пользователи и авторизация RBAC в Kubernetes](https://habr.com/ru/company/flant/blog/470503/).
6. [RBAC with Kubernetes in Minikube](https://medium.com/@HoussemDellai/rbac-with-kubernetes-in-minikube-4deed658ea7b).

------

## **Задание 1: Работа с ConfigMaps**
### **Задача**
Развернуть приложение (nginx + multitool), решить проблему конфигурации через ConfigMap и подключить веб-страницу.

### **Шаги выполнения**
1. **Создать Deployment** с двумя контейнерами
   - `nginx`
   - `multitool`
3. **Подключить веб-страницу** через ConfigMap
4. **Проверить доступность**

### **Что сдать на проверку**
- Манифесты:
  - `deployment.yaml`
  - `configmap-web.yaml`
- Скриншот вывода `curl` или браузера

---
## **Задание 2: Настройка HTTPS с Secrets**  
### **Задача**  
Развернуть приложение с доступом по HTTPS, используя самоподписанный сертификат.

### **Шаги выполнения**  
1. **Сгенерировать SSL-сертификат**
```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key -out tls.crt -subj "/CN=myapp.example.com"
```
2. **Создать Secret**
3. **Настроить Ingress**
4. **Проверить HTTPS-доступ**

### **Что сдать на проверку**  
- Манифесты:
  - `secret-tls.yaml`
  - `ingress-tls.yaml`
- Скриншот вывода `curl -k`

---
## **Задание 3: Настройка RBAC**  
### **Задача**  
Создать пользователя с ограниченными правами (только просмотр логов и описания подов).

### **Шаги выполнения**  
1. **Включите RBAC в microk8s**
```bash
microk8s enable rbac
```
2. **Создать SSL-сертификат для пользователя**
```bash
openssl genrsa -out developer.key 2048
openssl req -new -key developer.key -out developer.csr -subj "/CN={ИМЯ ПОЛЬЗОВАТЕЛЯ}"
openssl x509 -req -in developer.csr -CA {CA серт вашего кластера} -CAkey {CA ключ вашего кластера} -CAcreateserial -out developer.crt -days 365
```
3. **Создать Role (только просмотр логов и описания подов) и RoleBinding**
4. **Проверить доступ**

### **Что сдать на проверку**  
- Манифесты:
  - `role-pod-reader.yaml`
  - `rolebinding-developer.yaml`
- Команды генерации сертификатов
- Скриншот проверки прав (`kubectl get pods --as=developer`)

---
## Шаблоны манифестов с учебными комментариями
### **1. Deployment с ConfigMap (nginx + multitool)**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        volumeMounts:
        - name: nginx-config # ПОДКЛЮЧЕНИЕ ConfigMap
          mountPath: /etc/nginx/conf.d
      volumes:
      - name: nginx-config
        configMap:
          name: nginx-config # УКАЖИТЕ имя созданного ConfigMap
```
### **2. ConfigMap для веб-страницы**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: web-content # ИЗМЕНИТЕ: Укажите имя ConfigMap
  namespace: default # ОПЦИОНАЛЬНО: Укажите namespace, если не default
data:
  # КЛЮЧЕВОЙ МОМЕНТ: index.html будет подключен как файл
  index.html: |
    <!DOCTYPE html>
    <html>
    <head>
      <title>Страница из ConfigMap</title> # ИЗМЕНИТЕ: Заголовок страницы
    </head>
    <body>
      <h1>Привет от Kubernetes!</h1> # ДОБАВЬТЕ: Свой контент страницы
    </body>
    </html>
```

### **3. Secret для TLS-сертификата**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: tls-secret # ИЗМЕНИТЕ при необходимости
type: kubernetes.io/tls
data:
  tls.crt: # ЗАМЕНИТЕ на base64-код сертификата (cat tls.crt | base64 -w 0)
  tls.key: # ЗАМЕНИТЕ на base64-код ключа (cat tls.key | base64 -w 0)
```
### **4. Role для просмотра подов**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-viewer # ИЗМЕНИТЕ: Название роли
  namespace: default # ВАЖНО: Role работает только в указанном namespace
rules:
- apiGroups: [""] # КЛЮЧЕВОЙ МОМЕНТ: "" означает core API group
  resources: # РАЗРЕШЕННЫЕ РЕСУРСЫ:
    - pods # Доступ к просмотру подов
    - pods/log # Доступ к логам подов
  verbs: # РАЗРЕШЕННЫЕ ДЕЙСТВИЯ:
    - get # Просмотр отдельных подов
    - list # Список всех подов
    - watch # Мониторинг изменений
    - describe # Просмотр деталей
# ДОПОЛНИТЕЛЬНО: Можно добавить больше правил для других ресурсов
```
---

## **Правила приёма работы**
1. Домашняя работа оформляется в своём Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать:
   - Скриншоты вывода команд `kubectl`
   - Скриншоты результатов выполнения
   - Тексты манифестов или ссылки на них
3. Для заданий с TLS приложите команды генерации сертификатов

## **Критерии оценивания задания**
1. Зачёт: Все задачи выполнены, манифесты корректны, есть доказательства работы (скриншоты).
2. Доработка (на доработку задание направляется 1 раз): основные задачи выполнены, при этом есть ошибки в манифестах или отсутствуют проверочные скриншоты.
3. Незачёт: работа выполнена не в полном объёме, есть ошибки в манифестах, отсутствуют проверочные скриншоты. Все попытки доработки израсходованы (на доработку работа направляется 1 раз). Этот вид оценки используется крайне редко.

## **Срок выполнения задания**  
1. 5 дней на выполнение задания.
2. 5 дней на доработку задания (в случае направления задания на доработку).






---

# Решение задания 1

```bash
root@ubuntu2204:/home/ubuntu/Desktop/5-storages# nano configmap-web.yaml
```
configmap-web.yaml
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
  namespace: default
data:
  index.html: |
    <!DOCTYPE html>
    <html>
    <head>
      <title>My Header</title>
    </head>
    <body>
      <h1>My Nginx App  in Kubernetes Index!</h1>
    </body>
    </html>
```
```bash
root@ubuntu2204:/home/ubuntu/Desktop/6-config_rbac# nano deployment.yaml
```
deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: multitool
        image: wbitt/network-multitool
        ports:
        - containerPort: 8080
        env:
          - name: HTTP_PORT
            value: "8080"
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        volumeMounts:
        - name: nginx-config
          mountPath: /usr/share/nginx/html/index.html
          subPath: index.html
      volumes:
      - name: nginx-config
        configMap:
          name: nginx-config
```

Применение манифестов
```bash
root@ubuntu2204:/home/ubuntu/Desktop/6-config_rbac# microk8s kubectl apply -f configmap-web.yaml
configmap/nginx-config created

root@ubuntu2204:/home/ubuntu/Desktop/6-config_rbac# microk8s kubectl apply -f deployment.yaml
deployment.apps/web-app created
```

Вывод `curl`
```bash
root@ubuntu2204:/home/ubuntu/Desktop/6-config_rbac# microk8s kubectl exec -itc multitool web-app-bc7689f67-8shtj -- curl localhost |& head
<!DOCTYPE html>
<html>
<head>
  <title>My Header</title>
</head>
<body>
  <h1>My Nginx App  in Kubernetes Index!</h1>
</body>
</html>
```

## Решение задания 2
```bash
root@ubuntu2204:/home/ubuntu/Desktop/6-config_rbac# openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key -out tls.crt -subj "/CN=myapp.example.com"
....+......+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.+..+...+...+....+........+...+...+.+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*......................+...+........+.+...........+.......+..+...+......+......+.+........+.......+..............+....+.....+....+...........+.........+.......+.....+.+..+.......+...+...........+..........+.....+......+........................+.......+...+..+.......+..+.+........+.+........+...+.........+.+..+..........+......+.....+................+...+..+.........+.+...+.....+.......+........+......+.......+..+......+.......+..+.......+......+............+..+.+...+..+............+.+............+..............+..........+.....+.+..+...+....+...+..+...............+......+..........+..+...............+...+...+...............+....+...+...+......+......+.................+..........+...............+..+.+..................+..+....+........+.+......+...+..+...............+...+......+......+......+.......+...........+....+........+.......+..+..........+...+...+..+.+..+...+.......+..+.+.....+.+......+...+............+.....+....+...+..+.........+................+...+......+......+...+...+.....+............+.+...........+.+..+...+..........+..............+.+..............+....+......+...+......+......+......+..+..........+......+..............+.......+......+..+......+.+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
.....+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.+..+.......+...+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*..+..+.......+..+......+.......+..+.........+.+...+......+.....+.......+........+.+.....................+.....+...+....+..+....+.........+...+...+...........+...+.+.....+..........+..+..........+.........+.....+.+..+.+.........+............+.....................+.....+....+........+.+.....+....+...........+.......+........+............+.......+..+.+........+.+......+..............+.+............+..+......+....+...+......+..+...+....+..+.+..+............+..........+...........+...+......+.+...+............+...+..............+..........+......+.....+....+.....+.......+......+..+.+..+..........+.....+...+......+......+...+................+........+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
-----

root@ubuntu2204:/home/ubuntu/Desktop/6-config_rbac# ll
total 24
drwxr-xr-x 2 root        root        4096 ноя 17 21:50 ./
drwxr-xr-x 7 bogatyrevam bogatyrevam 4096 ноя 17 21:41 ../
-rw-r--r-- 1 root        root         275 ноя 17 21:40 configmap-web.yaml
-rw-r--r-- 1 root        root         711 ноя 17 21:42 deployment.yaml
-rw-r--r-- 1 root        root        1135 ноя 17 21:50 tls.crt
-rw------- 1 root        root        1708 ноя 17 21:50 tls.key
```

```
root@ubuntu2204:/home/ubuntu/Desktop/6-config_rbac# microk8s kubectl create secret tls myapp-tls --key tls.key --cert tls.crt
secret/myapp-tls created
```

Читаем и переводим в `base64`
```bash
root@ubuntu2204:/home/ubuntu/Desktop/6-config_rbac# base64 -w 0 tls.key

root@ubuntu2204:/home/ubuntu/Desktop/6-config_rbac# base64 -w 0 tls.crt
```
Что получил, вставил в `secret-tls.yaml`
```bash
apiVersion: v1
kind: Secret
metadata:
  name: tls-secret
type: kubernetes.io/tls
data:
  tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ...
  tls.key: LS0tLS1CRUdJTiBQUklWQVR...
```
```bash
root@ubuntu2204:/home/ubuntu/Desktop/6-config_rbac# microk8s kubectl apply -f secret-tls.yaml
secret/tls-secret created
```
```bash
root@ubuntu2204:/home/ubuntu/Desktop/6-config_rbac# microk8s kubectl describe secret myapp-tls |& tail -4
Data
====
tls.crt:  1135 bytes
tls.key:  1708 bytes
```

Создаем и применяем `ingress-tls`
```bash
root@ubuntu2204:/home/ubuntu/Desktop/6-config_rbac# nano ingress-tls.yaml

root@ubuntu2204:/home/ubuntu/Desktop/6-config_rbac# microk8s kubectl apply -f ingress-tls.yaml
deployment.apps/myapp created
service/myapp-service created
ingress.networking.k8s.io/myapp-ingress created
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: nginx:alpine
        ports:
        - containerPort: 80
        volumeMounts:
        - name: ssl-certs
          mountPath: /etc/nginx/ssl
          readOnly: true
      volumes:
      - name: ssl-certs
        secret:
          secretName: myapp-tls
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  tls:
  - hosts:
    - myapp.example.com
    secretName: myapp-tls
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp-service
            port:
              number: 80
```

curl
```bash
root@ubuntu2204:/home/ubuntu/Desktop/6-config_rbac# curl -H 'Host: myapp.example.com' -ks 'https://localhost/' |& head -2
<!DOCTYPE html>
<html>
```

## Задание 3

```bash
root@ubuntu2204:/home/ubuntu/Desktop/6-config_rbac# microk8s enable rbac
Infer repository core for addon rbac
Enabling RBAC
Reconfiguring apiserver
Restarting apiserver
RBAC is enabled

root@ubuntu2204:/home/ubuntu/Desktop/6-config_rbac# openssl genrsa -out developer.key 2048

root@ubuntu2204:/home/ubuntu/Desktop/6-config_rbac# nano role-pod-reader.yaml
```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer-role
  namespace: default
rules:
- apiGroups: [""]
  resources:
    - pods
    - pods/log
  verbs:
    - get
    - list
    - watch
    - describe
```

```bash
root@ubuntu2204:/home/ubuntu/Desktop/6-config_rbac# nano rolebinding-developer.yaml
```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: developer_role_binding
subjects:
- kind: User
  name: developer
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer-role
  apiGroup: rbac.authorization.k8s.io
```

```bash
root@ubuntu2204:/home/ubuntu/Desktop/6-config_rbac# microk8s kubectl apply -f role-pod-reader.yaml
role.rbac.authorization.k8s.io/developer-role created

root@ubuntu2204:/home/ubuntu/Desktop/6-config_rbac# microk8s kubectl apply -f rolebinding-developer.yaml
rolebinding.rbac.authorization.k8s.io/developer_role_binding created

root@ubuntu2204:/home/ubuntu/Desktop/6-config_rbac# microk8s kubectl auth can-i get po --as=developer
yes
```
