# Kubernetes com Minikube - Deploy de Microserviços

## Visão Geral

O Minikube foi escolhido como orquestrador Kubernetes para implementar a arquitetura de microserviços desenvolvida. Esta solução permite executar um cluster Kubernetes local, ideal para desenvolvimento, testes e demonstrações da aplicação distribuída.

## Estrutura do Projeto

Todos os microserviços foram deployados no mesmo cluster Kubernetes. Para isso, foi criada uma pasta `k8s/` com os arquivos de configuração necessários em cada um dos serviços desenvolvidos.

A estrutura básica de diretórios para o projeto é a seguinte:

```
.
├── account-service/
│   ├── k8s/
│   │   ├── deployment.yaml
│   │   └── service.yaml
│   └── ...
├── auth-service/
│   ├── k8s/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   └── secret.yaml
│   └── ...
├── gateway-service/
│   ├── k8s/
│   │   ├── deployment.yaml
│   │   └── service.yaml
│   └── ...
├── product-service/
│   ├── k8s/
│   │   ├── deployment.yaml
│   │   └── service.yaml
│   └── ...
├── order-service/
│   ├── k8s/
│   │   ├── deployment.yaml
│   │   └── service.yaml
│   └── ...
└── database/
    └── k8s/
        └── postgres.yaml
```

## Código Fonte

Os arquivos de configuração do Kubernetes são criados apenas nos repositórios de implementação das APIs, ou seja, aqueles que possuem `Service` no nome do projeto.

### Banco de Dados PostgreSQL

**Postgres ConfigMap e Deployment**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: postgres-configmap
data:
  POSTGRES_HOST: postgres
  POSTGRES_DB: store
---
apiVersion: v1
kind: Secret
metadata:
  name: postgres-secrets
type: Opaque
data:
  POSTGRES_USER: c3RvcmU=  # store (base64)
  POSTGRES_PASSWORD: c3RvcmU=  # store (base64)
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:15
        ports:
        - containerPort: 5432
        env:
        - name: POSTGRES_DB
          valueFrom:
            configMapKeyRef:
              name: postgres-configmap
              key: POSTGRES_DB
        - name: POSTGRES_USER
          valueFrom:
            secretKeyRef:
              name: postgres-secrets
              key: POSTGRES_USER
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secrets
              key: POSTGRES_PASSWORD
---
apiVersion: v1
kind: Service
metadata:
  name: postgres
spec:
  selector:
    app: postgres
  ports:
  - port: 5432
    targetPort: 5432
  type: ClusterIP
```

### Account Service

**Deployment**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: account
spec:
  replicas: 1
  selector:
    matchLabels:
      app: account
  template:
    metadata:
      labels:
        app: account
    spec:
      containers:
      - name: account
        image: 'juliaas2/account:latest'
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 8080
        env:
        - name: DATABASE_HOST
          valueFrom:
            configMapKeyRef:
              name: postgres-configmap
              key: POSTGRES_HOST
        - name: DATABASE_USER
          valueFrom:
            secretKeyRef:
              name: postgres-secrets
              key: POSTGRES_USER
        - name: DATABASE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secrets
              key: POSTGRES_PASSWORD
```

**Service**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: account
  labels:
    app: account
spec:
  type: ClusterIP
  ports:
  - port: 8080
    targetPort: 8080
  selector:
    app: account
```

### Auth Service

**Deployment**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: auth
spec:
  replicas: 1
  selector:
    matchLabels:
      app: auth
  template:
    metadata:
      labels:
        app: auth
    spec:
      containers:
      - name: auth
        image: 'juliaas2/auth:latest'
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 8080
        env:
        - name: JWT_SECRET
          valueFrom:
            secretKeyRef:
              name: auth-secrets
              key: JWT_SECRET
```

**Service**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: auth
  labels:
    app: auth
spec:
  type: ClusterIP
  ports:
  - port: 8080
    targetPort: 8080
  selector:
    app: auth
```

**Secret**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: auth-secrets
type: Opaque
data:
  JWT_SECRET: bXlTdXBlclNlY3JldEtleUZvckpXVA==  # mySuperSecretKeyForJWT (base64)
```

### Gateway Service

**Deployment**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gateway
spec:
  replicas: 1
  selector:
    matchLabels:
      app: gateway
  template:
    metadata:
      labels:
        app: gateway
    spec:
      containers:
      - name: gateway
        image: 'juliaas2/gateway:latest'
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 8080
        env:
        - name: ACCOUNT_SERVICE_URL
          value: "http://account:8080"
        - name: AUTH_SERVICE_URL
          value: "http://auth:8080"
        - name: PRODUCT_SERVICE_URL
          value: "http://product:8080"
        - name: ORDER_SERVICE_URL
          value: "http://order:8080"
```

**Service**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: gateway
  labels:
    app: gateway
spec:
  type: NodePort
  ports:
  - port: 8080
    targetPort: 8080
    nodePort: 30080
  selector:
    app: gateway
```

### Product Service

**Deployment**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: product
spec:
  replicas: 1
  selector:
    matchLabels:
      app: product
  template:
    metadata:
      labels:
        app: product
    spec:
      containers:
      - name: product
        image: 'juliaas2/product:latest'
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 8080
        env:
        - name: DATABASE_HOST
          valueFrom:
            configMapKeyRef:
              name: postgres-configmap
              key: POSTGRES_HOST
        - name: DATABASE_USER
          valueFrom:
            secretKeyRef:
              name: postgres-secrets
              key: POSTGRES_USER
        - name: DATABASE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secrets
              key: POSTGRES_PASSWORD
```

**Service**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: product
  labels:
    app: product
spec:
  type: ClusterIP
  ports:
  - port: 8080
    targetPort: 8080
  selector:
    app: product
```

### Order Service

**Deployment**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order
spec:
  replicas: 1
  selector:
    matchLabels:
      app: order
  template:
    metadata:
      labels:
        app: order
    spec:
      containers:
      - name: order
        image: 'juliaas2/order:latest'
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 8080
        env:
        - name: DATABASE_HOST
          valueFrom:
            configMapKeyRef:
              name: postgres-configmap
              key: POSTGRES_HOST
        - name: DATABASE_USER
          valueFrom:
            secretKeyRef:
              name: postgres-secrets
              key: POSTGRES_USER
        - name: DATABASE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secrets
              key: POSTGRES_PASSWORD
```

**Service**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: order
  labels:
    app: order
spec:
  type: ClusterIP
  ports:
  - port: 8080
    targetPort: 8080
  selector:
    app: order
```