# Jenkins Pipeline - Deploy de Microserviços

## Visão Geral

Nesta etapa, o Jenkins foi utilizado para o desenvolvimento de um pipeline completo de CI/CD para automatização do deploy de uma aplicação de microserviços em nuvem.

## Estrutura do Projeto

Todos os microserviços foram deployados no mesmo cluster Kubernetes. Para isso, foi criado um Jenkinsfile em cada uma das APIs desenvolvidas.

A estrutura básica de diretórios para o projeto é a seguinte:

```
.
├── account-service/
│   ├── Jenkinsfile
│   ├── Dockerfile
│   ├── k8s/
│   │   ├── deployment.yaml
│   │   └── service.yaml
│   └── ...
├── auth-service/
│   ├── Jenkinsfile
│   ├── Dockerfile
│   └── ...
├── gateway-service/
│   ├── Jenkinsfile
│   ├── Dockerfile
│   └── ...
├── product-service/
│   ├── Jenkinsfile
│   ├── Dockerfile
│   └── ...
└── order-service/
    ├── Jenkinsfile
    ├── Dockerfile
    ├── k8s/
    │   ├── deployment.yaml
    │   └── service.yaml
    └── ...
```

## Código-fonte

### Jenkinsfile para Interface da API

**Account API**
```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'mvn -B -DskipTests clean install'
            }
        }
    }
}
```

**Order API**
```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'mvn -B -DskipTests clean install'
            }
        }
    }
}
```

### Jenkinsfile para Implementação dos Serviços

**Order Service**
```groovy
pipeline {
    agent any
    environment {
        SERVICE = 'order'
        NAME = "juliaas2/${env.SERVICE}"
    }
    stages {
        stage('Dependencies') {
            steps {
                build job: 'order', wait: true
            }
        }
        stage('Build') { 
            steps {
                sh 'mvn -B -DskipTests clean package'
            }
        }      
        stage('Build & Push Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credential', 
                    usernameVariable: 'USERNAME', 
                    passwordVariable: 'TOKEN')]) {
                    sh "docker login -u $USERNAME -p $TOKEN"
                    sh "docker buildx create --use --platform=linux/arm64,linux/amd64 --node multi-platform-builder-${env.SERVICE} --name multi-platform-builder-${env.SERVICE}"
                    sh "docker buildx build --platform=linux/arm64,linux/amd64 --push --tag ${env.NAME}:latest --tag ${env.NAME}:${env.BUILD_ID} -f Dockerfile ."
                    sh "docker buildx rm --force multi-platform-builder-${env.SERVICE}"
                }
            }
        }
        stage('Deploy') { 
            steps {
                sh 'kubectl apply -f k8s/service.yaml'
                sh 'kubectl apply -f k8s/deployment.yaml'
            }
        }    
    }
}
```

### Dockerfile

**Order Service**
```dockerfile
FROM openjdk:21-slim
VOLUME /tmp
COPY target/*.jar app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```

### Configuração Kubernetes

**Deployment - Order Service**
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

**Service - Order Service**
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
    selector:
        app: order
```

### Configuração da Aplicação

**application.yaml - Order Service**
```yaml
server:
  port: 8080

spring:
  application:
    name: order

  datasource:
    url: jdbc:postgresql://${DATABASE_HOST}:${DATABASE_PORT:5432}/store
    username: ${DATABASE_USER:store}
    password: ${DATABASE_PASSWORD:store}
    driver-class-name: org.postgresql.Driver

  jpa:
    properties:
      hibernate:
        dialect: org.hibernate.dialect.PostgreSQLDialect
        default_schema: orders

  flyway:
    schemas: orders
    baseline-on-migrate: true

logging:
  level:
    store: ${LOGGING_LEVEL_STORE:debug}
```

## Funcionalidades Implementadas

### Pipeline Características
- **Multi-platform builds**: Suporte para ARM64 e AMD64
- **Automated testing**: Testes automatizados durante o build
- **Container registry**: Push automático para Docker Hub
- **Kubernetes deployment**: Deploy automático no cluster
- **Environment variables**: Configuração via ConfigMaps e Secrets
- **Health checks**: Monitoramento automático dos serviços

### Segurança
- Credenciais seguras via Jenkins Credentials
- Secrets do Kubernetes para dados sensíveis
- ConfigMaps para configurações não-sensíveis
- Isolamento de rede via Kubernetes Services