## 데몬셋
- 모든 노드가 파드의 사본을 실행하도록 한다
- 노드 하나당 하나의 파드만 생성된다
- 노드가 클러스터에서 제거되면 해당 파드는 가비지로 수집된다
- 노드를 관리하는 파드라면 데몬셋으로 만드는게 가장 효율적이다
- 각 노드에서 로드 밸런서 역할을 하는 `kube-proxy` 역시 데몬셋이다

```YAML
apiVersion: apps/v1
kind: DaemonSet
metadata:
  annotations:
    deprecated.daemonset.template.generation: "2"
  generation: 2
  labels:
    addonmanager.kubernetes.io/mode: Reconcile
    k8s-app: kube-proxy
  name: kube-proxy
  namespace: kube-system
spec:
...
```

## 컨피그맵
- 키-값 쌍으로 기밀이 아닌 데이터를 저장하는 오브젝트
- 간단한 Todo 애플리케이션을 만든다고 가정한다

#### application.properties
```properties
spring.mvc.view.prefix=/WEB-INF/jsp/  
spring.mvc.view.suffix=.jsp  
logging.level.org.springframework.web=INFO  
  
management.endpoints.web.base-path=/manage  
management.endpoints.web.exposure.include=*  
  
spring.jpa.show-sql=true  
  
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver  
spring.jpa.hibernate.ddl-auto=update  
spring.datasource.url=jdbc:mysql://${RDS_HOSTNAME:localhost}:${RDS_PORT:3306}/${RDS_DB_NAME:todos}  
spring.datasource.username=${RDS_USERNAME:todos-user}  
spring.datasource.password=${RDS_PASSWORD:dummytodos}  
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL57Dialect
```
- 일반적으로 개발 환경에 따라서 데이터베이스 설정 값을 다르게 사용한다
	- RDS_HOSTNAME, RDS_PORT, RDS_USERNAME, RDS_PASSWORD
	- 위 값이 설정되지 않은 경우 `:` 이후에 나오는 값을 사용한다

### MySQL 쿠버네티스에 배포하기

#### mysql-database-data-volume-persistentvolumeclaim.yaml
```YAML
apiVersion: v1  
kind: PersistentVolumeClaim  
metadata:  
  creationTimestamp: null  
  labels:  
    io.kompose.service: mysql-database-data-volume  
  name: mysql-database-data-volume  
spec:  
  accessModes:  
    - ReadWriteOnce  
  resources:  
    requests:  
      storage: 100Mi  
status: {}
```
- 파드간 저장소를 공유하기 위해 볼륨을 사용한다
- `PersistentVolumeClaim`는 일단 볼륨에 공간을 할당 받기 위한 오브젝트라고 알아두고 일단 넘어간다

#### mysql-deployment.yaml
```YAML
apiVersion: apps/v1  
kind: Deployment  
metadata:  
  labels:  
    io.kompose.service: mysql  
  name: mysql  
spec:  
  replicas: 1  
  selector:  
    matchLabels:  
      io.kompose.service: mysql  
  strategy:  
    type: Recreate  
  template:  
    metadata:  
      labels:  
        io.kompose.network/03-todo-web-application-mysql-default: "true"  
        io.kompose.service: mysql  
    spec:  
      containers:  
        - env:  
            - name: MYSQL_DATABASE  
              value: todos  
            - name: MYSQL_PASSWORD  
              value: dummytodos  
            - name: MYSQL_ROOT_PASSWORD  
              value: dummypassword  
            - name: MYSQL_ROOT_USER  
              value: root  
            - name: MYSQL_USER  
              value: todos-user  
          image: mysql:5.7  
          args:  
            - "--ignore-db-dir=lost+found" #CHANGE  
          name: mysql  
          ports:  
            - containerPort: 3306  
          volumeMounts:  
            - mountPath: /var/lib/mysql # 호스트에 마운트
              name: mysql-database-data-volume  
      restartPolicy: Always  
      volumes:  
        - name: mysql-database-data-volume  
          persistentVolumeClaim:  
            claimName: mysql-database-data-volume # PVC 설정
```

- `apply -f mysql-deployment.yaml` 을 실행해 MySQL 디플로이먼트를 생성한다

#### mysql-service.yaml
```YAML
apiVersion: v1  
kind: Service  
metadata:  
  annotations:  
    kompose.cmd: kompose convert  
    kompose.version: 1.28.0 (HEAD)  
  labels:  
    io.kompose.service: mysql  
  name: mysql  
spec:  
  type: ClusterIP  
  ports:  
    - name: "3306"  
      port: 3306  
      targetPort: 3306  
  selector:  
    io.kompose.service: mysql
```

- `apply -f mysql-service.yaml` 을 실행해 MySQL 서비스를 생성한다

<img width="620" alt="image" src="https://github.com/seung-hun-h/record/assets/60502370/6f343567-d7d2-4b4b-b360-80ff000f9a55">\

### Todo 애플리케이션 배포하기

#### todo-web-application-deployment.yaml
```YAML
apiVersion: apps/v1  
kind: Deployment  
metadata:  
  labels:  
    io.kompose.service: todo-web-application  
  name: todo-web-application  
spec:  
  replicas: 1  
  selector:  
    matchLabels:  
      io.kompose.service: todo-web-application  
  template:  
    metadata:  
      labels:  
        io.kompose.network/03-todo-web-application-mysql-default: "true"  
        io.kompose.service: todo-web-application  
    spec:  
      containers:  
        - env:  
			- name: RDS_DB_NAME  
			  value: todos  
			- name: RDS_HOSTNAME  
			  value: mysql  
			- name: RDS_PASSWORD  
			  value: dummytodos  
			- name: RDS_PORT  
			  value: "3306"  
			- name: RDS_USERNAME  
			  value: todos-user  
          image: in28min/todo-web-application-mysql:0.0.1-SNAPSHOT  
          name: todo-web-application  
          ports:  
            - containerPort: 8080  
      restartPolicy: Always
```

#### todo-web-application-service.yaml
```YAML
apiVersion: v1  
kind: Service  
metadata:  
  labels:  
    io.kompose.service: todo-web-application  
  name: todo-web-application  
spec:  
  type: LoadBalancer  
  ports:  
    - name: "8080"  
      port: 8080  
      targetPort: 8080  
  selector:  
    io.kompose.service: todo-web-application
```

<img width="799" alt="image" src="https://github.com/seung-hun-h/record/assets/60502370/5713c815-3000-4095-8108-35bda9c0ff6c">
<img width="900" alt="image" src="https://github.com/seung-hun-h/record/assets/60502370/a542df98-eeb5-4bde-89a8-54c94b10a7c9">


- Todo 애플리케이션도 잘 배포가 되었다

### 컨피그맵 사용하기
- 데이터베이스 도메인, 포트, 계정은 모든 파드에서 동일하게 사용하기 때문에 컨피그맵을 사용할 수 있다

#### todo-web-application-config.yaml
```YAML
apiVersion: v1  
kind: ConfigMap  
metadata:  
  name: todo-web-application-config  
data:  
  RDS_DB_NAME: todos  
  RDS_HOSTNAME: mysql  
  RDS_PORT: "3306"  
  RDS_USERNAME: todos-user
```

- `kubectlt apply -f todo-web-application-config.yaml`을 실행해 컨피그맵을 생성한다
<img width="356" alt="image" src="https://github.com/seung-hun-h/record/assets/60502370/a8c78046-37fa-4b08-9adb-2d119bc22c11">

#### todo-web-application-deployment.yaml
```YAML
apiVersion: apps/v1  
kind: Deployment  
metadata:  
  labels:  
    io.kompose.service: todo-web-application  
  name: todo-web-application  
spec:  
  replicas: 1  
  selector:  
    matchLabels:  
      io.kompose.service: todo-web-application  
  template:  
    metadata:  
      labels:  
        io.kompose.network/03-todo-web-application-mysql-default: "true"  
        io.kompose.service: todo-web-application  
    spec:  
      containers:  
        - env:  
          - name: RDS_DB_NAME  
            valueFrom:  
              configMapKeyRef:  
                key: RDS_DB_NAME  
                name: todo-web-application-config  
          - name: RDS_HOSTNAME  
            valueFrom:  
              configMapKeyRef:  
                key: RDS_HOSTNAME  
                name: todo-web-application-config  
          - name: RDS_PORT  
            valueFrom:  
              configMapKeyRef:  
                key: RDS_PORT  
                name: todo-web-application-config  
          - name: RDS_USERNAME  
            valueFrom:  
              configMapKeyRef:  
                key: RDS_USERNAME  
                name: todo-web-application-config  
          - name: RDS_PASSWORD  
            value: dummytodos
          image: in28min/todo-web-application-mysql:0.0.1-SNAPSHOT  
          name: todo-web-application  
          ports:  
            - containerPort: 8080  
      restartPolicy: Always
```

### 시크릿 사용하기
- 데이터베이스 패스워드는 민감한 정보이므로 컨피그맵에 Plain Text로 저장하면 안된다
- 이를 위해서 `Secret` 오브젝트를 사용한다

#### todo-web-application-secrets
- `kubectl create secret generic todo-web-application-secrets --from-literal=RDS_PASSWORD=dummytodos`

<img width="461" alt="image" src="https://github.com/seung-hun-h/record/assets/60502370/4343bb0c-9bdc-4417-b890-7dd600ae60a2">

#### todo-web-application-deployment.yaml
```YAML
apiVersion: apps/v1  
kind: Deployment  
metadata:  
  labels:  
    io.kompose.service: todo-web-application  
  name: todo-web-application  
spec:  
  replicas: 1  
  selector:  
    matchLabels:  
      io.kompose.service: todo-web-application  
  template:  
    metadata:  
      labels:  
        io.kompose.network/03-todo-web-application-mysql-default: "true"  
        io.kompose.service: todo-web-application  
    spec:  
      containers:  
        - env:  
          - name: RDS_DB_NAME  
            valueFrom:  
              configMapKeyRef:  
                key: RDS_DB_NAME  
                name: todo-web-application-config  
          - name: RDS_HOSTNAME  
            valueFrom:  
              configMapKeyRef:  
                key: RDS_HOSTNAME  
                name: todo-web-application-config  
          - name: RDS_PASSWORD  
            valueFrom:  
              secretKeyRef:  
                key: RDS_PASSWORD  
                name: todo-web-application-secrets  
          - name: RDS_PORT  
            valueFrom:  
              configMapKeyRef:  
                key: RDS_PORT  
                name: todo-web-application-config  
          - name: RDS_USERNAME  
            valueFrom:  
              configMapKeyRef:  
                key: RDS_USERNAME  
                name: todo-web-application-config  
          image: in28min/todo-web-application-mysql:0.0.1-SNAPSHOT  
          name: todo-web-application  
          ports:  
            - containerPort: 8080  
      restartPolicy: Always
```

<img width="900" alt="image" src="https://github.com/seung-hun-h/record/assets/60502370/d6c477ab-1ed1-4789-82c1-2f0fd17033d7">
- 시크릿도 잘 적용된 것을 확인할 수 있다

## PV, PVC
- PV
	- 관리자가 프로비저닝하거나 스토리지 클래스를 사용하여 동적으로 프로비저닝한 클러스터의 스토리지
	- PV를 사용하는 개별 파드와 별도의 라이프사이클을 가진다
- PVC
	- 사용자의 스토리지에 대한 요청
	- 파드가 노드 리소스를 사용하듯 PVC는 PV 리소스를 사용한다

#### GKE Persistent Disk 생성
- PV, PVC 실습을 위해 GKE를 사용한다
- 먼저 Persistent Disk를 생성한다

```CLI
gcloud compute disks create --type=pd-standard \
 --size=10GB --zone=us-central1-a my-persistent-disk
```

<img width="838" alt="image" src="https://github.com/seung-hun-h/record/assets/60502370/965e592a-a650-4dac-8f82-66ed03010e74">

#### PV 생성
```YAML
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv-alicek106
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 5Gi
  gcePersistentDisk:
    pdName: my-persistent-disk
```
- `accessMode`를 `ReadWriteOnce`로 설정했다

##### accessMode
- ReadWriteOnce(RWO)
	- 하나의 노드에서 해당 볼륨이 읽기-쓰기로 마운트될 수 있다
	- 복수의 파드가 하나의 노드에서 실행된다면 볼륨에 접근할 수 있다
- ReadOnlyMany(ROX)
	- 볼륨이 다수의 노드에서 읽기 전용으로 마운트 될 수 있다
- ReadWriteMany(RWX)
	- 볼륨이 다수의 노드에서 읽기-쓰기 마운트될 수 있다
- ReadWriteOncePod(RWOP)
	- 볼륨이 단일 파드에서 읽기-쓰기로 마운트될 수 있다

#### PVC, Pod 생성
```YAML
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc-alicek106
spec:
  storageClassName: ""       # 1. 다이나믹 프로비저닝을 사용하지 않음을 명시합니다.
  accessModes:
    - ReadWriteOnce          # 2. accessMode가 ReadWriteOnce 인 PV를 선택합니다.
  resources:
    requests:
      storage: 5Gi           # 3. 스토리지의 크기가 5GB보다 큰 PV를 선택합니다.
---
apiVersion: v1
kind: Pod
metadata:
  name: pd-mount-container
spec:
  containers:
    - name: pd-mount-container
      image: busybox
      args: [ "tail", "-f", "/dev/null" ] # 백그라운드에서 실행되는 데몬 프로세스를 실행할 때 사용
      volumeMounts:
      - name: pd-volume
        mountPath: /mnt
  volumes:
  - name : pd-volume
    persistentVolumeClaim:
      # 4. my-pvc-alicek106 라는 이름의 pvc를 사용합니다.
      claimName: my-pvc-alicek106 
```

##### 다이나믹 프로비저닝
- 쿠버네티스에서 다이나믹 프로비저닝(Dynamic Provisioning)은 스토리지 리소스를 자동으로 생성하는 기능
- 일반적으로 다이나믹 프로비저닝은 스토리지 클래스(StorageClass)와 함께 사용 
- 스토리지 클래스는 다이나믹 프로비저닝을 위한 설정을 정의하는 객체로, PVC가 특정 스토리지 클래스를 요청하면 쿠버네티스는 해당 스토리지 클래스에 따라 적절한 스토리지를 자동으로 프로비저닝.
- 다이나믹 프로비저닝을 사용하면 클러스터 관리자는 미리 모든 스토리지를 수동으로 생성할 필요 없이, 애플리케이션에서 필요한 스토리지를 PVC로 요청할 수 있다

##### 결과 확인

<img width="439" alt="image" src="https://github.com/seung-hun-h/record/assets/60502370/0ad3616b-6e9a-448a-9d84-cb7ea0466e79">

<img width="900" alt="image" src="https://github.com/seung-hun-h/record/assets/60502370/c99cc272-795c-42cc-bb3c-cda016977a13">
- `my-pvc-alicek106 `의 상태가 `Bound` 되었다
	- PV와 PVC가 잘 연결되었다는 뜻이다
<img width="822" alt="image" src="https://github.com/seung-hun-h/record/assets/60502370/25cd0278-01b5-4d81-b217-27ac3074b778">
- `/dev/sdb`가 `/mnt`에 마운트된 것을 확인할 수 있다
	- `pd-mount-container`의 `/dev/sdb`