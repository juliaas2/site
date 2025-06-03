# Serviço de Pedidos - Documentação do Projeto

## Visão Geral

Este projeto implementa um Serviço de Pedidos como parte de uma arquitetura de microsserviços para uma loja de e-commerce. O serviço gerencia a criação, recuperação e gestão de pedidos para usuários autenticados.

## Arquitetura

O Serviço de Pedidos segue um padrão de arquitetura de microsserviços com os seguintes componentes:

```
Camada Confiável
request → gateway → account → auth
                 ↓
Database ← exchange → product
                 ↓
                order
                 ↓
           3rd-party API
                 ↓
              internet
```

### Componentes Principais

- **Serviço de Pedidos**: Serviço principal para gestão de pedidos
- **Serviço de Produtos**: Serviço externo para informações de produtos
- **Serviço de Contas**: Serviço externo para autenticação de usuários
- **Banco PostgreSQL**: Armazenamento persistente para pedidos e itens
- **Kubernetes**: Plataforma de orquestração de contêineres
- **Jenkins**: Automação de pipeline CI/CD

## Documentação da API

### Autenticação

**Importante**: Todos os endpoints da API requerem autenticação. O usuário deve estar autenticado e fornecer o cabeçalho `id-account`.

### Endpoints

#### POST /order
Criar um novo pedido para o usuário autenticado atual.

**Cabeçalhos da Requisição:**
```
id-account: <user-id>
Content-Type: application/json
```

**Corpo da Requisição:**
```json
{
    "itens": [
        {
            "idProduct": "0195abfb-7074-73a9-9d26-b4b9fbaab0a8",
            "qtd": 2
        },
        {
            "idProduct": "0195abfe-e416-7052-be3b-27cdaf12a984",
            "qtd": 1
        }
    ]
}
```

**Resposta:**
```json
{
    "id": "0195ac33-73e5-7cb3-90ca-7b5e7e549569",
    "date": "2025-02-21 12:30:00",
    "itens": [
        {
            "id": "01961b9a-bca2-78c4-9be1-7092b261f217",
            "product": {
                "id": "0195abfb-7074-73a9-9d26-b4b9fbaab0a8",
                "name": "Nome do Produto",
                "price": 10.12
            },
            "qtd": 2,
            "total": 20.24
        },
        {
            "id": "01961b9b-08fd-76a5-8508-cdb6cd5c27ab",
            "product": {
                "id": "0195abfe-e416-7052-be3b-27cdaf12a984",
                "name": "Outro Produto",
                "price": 6.20
            },
            "qtd": 1,
            "total": 6.20
        }
    ],
    "total": 26.44
}
```

**Códigos de Resposta:**
- `200 OK`: Pedido criado com sucesso
- `400 Bad Request`: Dados da requisição inválidos
- `401 Unauthorized`: Autenticação necessária

#### GET /order
Recuperar todos os pedidos para o usuário autenticado atual.

**Cabeçalhos da Requisição:**
```
id-account: <user-id>
```

**Resposta:**
```json
[
    {
        "id": "0195ac33-73e5-7cb3-90ca-7b5e7e549569",
        "date": "2025-02-21 12:30:00",
        "itens": [
            {
                "id": "01961b9a-bca2-78c4-9be1-7092b261f217",
                "product": {
                    "id": "0195abfb-7074-73a9-9d26-b4b9fbaab0a8"
                },
                "qtd": 2,
                "total": 20.24
            }
        ],
        "total": 26.44
    },
    {
        "id": "0195ac33-cbbd-7a6e-a15b-b85402cf143f",
        "date": "2025-02-21 15:21:57",
        "itens": [
            {
                "id": "01961b9b-08fd-76a5-8508-cdb6cd5c27ab",
                "product": {
                    "id": "0195abfe-e416-7052-be3b-27cdaf12a984"
                },
                "qtd": 3,
                "total": 18.60
            }
        ],
        "total": 18.60
    }
]
```

**Códigos de Resposta:**
- `200 OK`: Pedidos recuperados com sucesso
- `401 Unauthorized`: Autenticação necessária

#### GET /order/{id}
Recuperar detalhes do pedido por ID. O pedido deve pertencer ao usuário autenticado atual.

**Cabeçalhos da Requisição:**
```
id-account: <user-id>
```

**Parâmetros de Caminho:**
- `id`: ID do Pedido (UUID)

**Resposta:**
```json
{
    "id": "0195ac33-73e5-7cb3-90ca-7b5e7e549569",
    "date": "2025-02-21 12:30:00",
    "itens": [
        {
            "id": "01961b9a-bca2-78c4-9be1-7092b261f217",
            "product": {
                "id": "0195abfb-7074-73a9-9d26-b4b9fbaab0a8"
            },
            "qtd": 2,
            "total": 20.24
        },
        {
            "id": "01961b9b-08fd-76a5-8508-cdb6cd5c27ab",
            "product": {
                "id": "0195abfe-e416-7052-be3b-27cdaf12a984"
            },
            "qtd": 1,
            "total": 6.20
        }
    ],
    "total": 26.44
}
```

**Códigos de Resposta:**
- `200 OK`: Pedido encontrado e retornado
- `404 Not Found`: Pedido não encontrado ou não pertence ao usuário
- `401 Unauthorized`: Autenticação necessária

#### DELETE /order/{id}
Excluir um pedido por ID.

**Cabeçalhos da Requisição:**
```
id-account: <user-id>
```

**Parâmetros de Caminho:**
- `id`: ID do Pedido (UUID)

**Códigos de Resposta:**
- `200 OK`: Pedido excluído com sucesso
- `404 Not Found`: Pedido não encontrado
- `401 Unauthorized`: Autenticação necessária

## Esquema do Banco de Dados

### Tabela Orders
```sql
CREATE TABLE orders (
    id_order VARCHAR(36) NOT NULL,
    id_user VARCHAR(36) NOT NULL,
    date_order TIMESTAMP NOT NULL,
    total DOUBLE PRECISION NOT NULL,
    CONSTRAINT pk_order PRIMARY KEY (id_order)
);
```

### Tabela Items
```sql
CREATE TABLE item (
    id_item VARCHAR(36) NOT NULL,
    id_product VARCHAR(36) NOT NULL,
    id_order VARCHAR(36) NOT NULL,
    qtd INT NOT NULL,
    total DOUBLE PRECISION NOT NULL,
    CONSTRAINT pk_item PRIMARY KEY (id_item)
);
```

## Estrutura do Projeto

```
src/
├── main/
│   ├── java/store/order-service/
│   │   ├── Item.java              # Modelo de domínio Item
│   │   ├── ItemModel.java         # Entidade JPA Item
│   │   ├── ItemParser.java        # Conversores DTO Item
│   │   ├── ItemRepository.java    # Repositório de dados Item
│   │   ├── Order.java             # Modelo de domínio Order
│   │   ├── OrderApplication.java  # Classe principal da aplicação
│   │   ├── OrderModel.java        # Entidade JPA Order
│   │   ├── OrderParser.java       # Conversores DTO Order
│   │   ├── OrderRepository.java   # Repositório de dados Order
│   │   ├── OrderResource.java     # Controlador REST
│   │   └── OrderService.java      # Serviço de lógica de negócio
│   └── resources/
│       ├── application.yaml       # Configuração da aplicação
│       └── db/migrations/         # Migrações Flyway do banco
├── Dockerfile                     # Definição da imagem do contêiner
├── Jenkinsfile                    # Pipeline CI/CD
├── k8s/                          # Manifestos Kubernetes
│   ├── deployment.yaml
│   └── service.yaml
└── pom.xml                       # Configuração Maven
```