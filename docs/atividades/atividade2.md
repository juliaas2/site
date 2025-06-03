# Product API - Documentação do Projeto

## Visão Geral

Este projeto implementa uma API RESTful para uma loja, focada no gerenciamento de produtos. A API é construída usando Spring Boot 3.4.2 com Java 21 e utiliza PostgreSQL como banco de dados.

## Arquitetura

O projeto segue o padrão de microserviços com separação clara de responsabilidades:

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Gateway       │    │   Product API   │    │   Database      │
│                 │───▶│                 │───▶│   PostgreSQL    │
│   (Port 8080)   │    │   (Port 8080)   │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### Componentes Principais

- **ProductController (Interface)**: Define os contratos da API usando Feign Client
- **ProductResource**: Implementação REST dos endpoints
- **ProductService**: Lógica de negócio
- **ProductRepository**: Camada de acesso aos dados
- **ProductModel**: Entidade JPA para persistência
- **Product**: Objeto de domínio
- **ProductIn/ProductOut**: DTOs para entrada e saída

## Endpoints da API

### Autenticação
⚠️ **Importante**: Para consumir a API, o usuário deve estar autenticado.

### POST /product
Cria um novo produto.

**Request:**
```json
{
    "name": "Tomato",
    "price": 10.12,
    "unit": "kg"
}
```

**Response:**
```json
{
    "id": "0195abfb-7074-73a9-9d26-b4b9fbaab0a8",
    "name": "Tomato",
    "price": 10.12,
    "unit": "kg"
}
```
**Status Code**: `200 OK`

### GET /product
Retorna todos os produtos.

**Response:**
```json
[
    {
        "id": "0195abfb-7074-73a9-9d26-b4b9fbaab0a8",
        "name": "Tomato",
        "price": 10.12,
        "unit": "kg"
    },
    {
        "id": "0195abfe-e416-7052-be3b-27cdaf12a984",
        "name": "Cheese",
        "price": 0.62,
        "unit": "slice"
    }
]
```
**Status Code**: `200 OK`

### GET /product/{id}
Retorna um produto específico pelo ID.

**Response:**
```json
{
    "id": "0195abfb-7074-73a9-9d26-b4b9fbaab0a8",
    "name": "Tomato",
    "price": 10.12,
    "unit": "kg"
}
```
**Status Code**: `200 OK`

### DELETE /product/{id}
Remove um produto pelo ID.

**Status Code**: `204 No Content`

## Estrutura do Banco de Dados

### Schema: `product`

#### Tabela: `product`
```sql
CREATE TABLE product (
    id_product VARCHAR(36) NOT NULL,
    tx_name VARCHAR(256) NOT NULL,
    price DOUBLE PRECISION NOT NULL,
    tx_unit VARCHAR(64) NOT NULL,
    CONSTRAINT pk_product PRIMARY KEY (id_product)
);
```

## Configuração e Execução

### Variáveis de Ambiente
```yaml
DATABASE_HOST: localhost
DATABASE_PORT: 5432
DATABASE_USER: store
DATABASE_PASSWORD: store
```

### Executando Localmente
```bash
# Compilar o projeto
mvn clean package

# Executar a aplicação
java -jar target/product-service-1.0.0.jar
```

## Funcionalidades Adicionais Implementadas

### ✅ Implementado
- Estrutura completa de microserviço
- Persistência com PostgreSQL
- Migrações com Flyway
- Containerização com Docker
- Deploy no Kubernetes
- Pipeline CI/CD com Jenkins
- Separação clara de responsabilidades (Clean Architecture)


## Estrutura do Projeto

```
src/
├── main/
│   ├── java/store/product/
│   │   ├── Product.java              # Objeto de domínio
│   │   ├── ProductApplication.java   # Classe principal
│   │   ├── ProductController.java    # Interface Feign Client
│   │   ├── ProductModel.java         # Entidade JPA
│   │   ├── ProductParser.java        # Conversor de DTOs
│   │   ├── ProductRepository.java    # Repositório JPA
│   │   ├── ProductResource.java      # Controller REST
│   │   └── ProductService.java       # Serviço de negócio
│   └── resources/
│       ├── application.yaml          # Configuração da aplicação
│       └── db/migration/             # Scripts Flyway
├── Dockerfile                        # Imagem Docker
├── Jenkinsfile                       # Pipeline CI/CD
├── k8s/                             # Manifests Kubernetes
└── pom.xml                          # Configuração Maven
```