# CineFlow - Diagramas UML - Documentação Completa

Documentação de todos os diagramas PlantUML do projeto CineFlow, uma plataforma de compra e reserva de ingressos de cinema usando arquitetura de **microsserviços**.

---

## 📋 Índice de Diagramas

### **Diagramas de Requisitos e Análise**
- `actors.puml`: Identificação de atores/stakeholders
- `use_cases.puml`: Casos de uso do sistema
- `booking_sequence.puml`: Sequência detalhada de reserva

### **Diagramas de Projeto (FOCO)**
- **`components.puml`** ⭐: Diagrama de Componentes - Arquitetura de Software
- **`deployment.puml`** ⭐: Diagrama de Implantação - Infraestrutura e Serviços em Nuvem
- **`communication_booking.puml`** ⭐: Diagrama de Comunicação - Fluxo Completo de Reserva e Pagamento

### **Diagramas de Implementação**
- `classes.puml`: Estrutura de classes (OOP)
- `booking_state.puml`: Máquina de estados de uma Reserva
- `data_model.puml`: Modelo de Dados (ER)
- `system_sequence.puml`: Sequência do sistema
- `architecture.puml`: Visão geral da arquitetura

---

## 🏗️ **TRÊS PILARES PRINCIPAIS (Diagramas Solicitados)**

### 1️⃣ **Diagrama de Componentes** (`components.puml`)

**FOCO**: Arquitetura de Software e Componentes

Representa a estrutura interna do sistema em nível de componentes, módulos, responsabilidades e integrações.

#### **Camadas Representadas**:

```
┌─────────────────────────────────────────────────────────┐
│         Camada de Apresentação (UI / Mobile)            │
│  Frontend Web SPA (React/Vue) | Mobile App (Flutter)    │
└────────────────┬────────────────────────────────────────┘
                 │
            ┌────▼─────┐
            │ API       │ ← Roteamento, Autenticação, Rate Limit
            │ Gateway   │
            └────┬──────┘
                 │
    ┌────────────┼────────────┬────────────┬──────────┐
    │            │            │            │          │
┌───▼────┐  ┌───▼────┐  ┌───▼────┐  ┌───▼────┐  ┌──▼────┐
│Catalog │  │ Sales  │  │  Auth  │  │Payment │  │Notif  │
│Service │  │Service │  │Service │  │Service │  │Service│
└────┬───┘  └───┬────┘  └────┬───┘  └───┬───┘  └───┬────┘
     │          │            │          │          │
   ┌─▼───┐   ┌─▼───┐     ┌──▼──┐   ┌──▼──┐   ┌──▼──┐
   │ PostgreSQL   │      │      │   │     │
   │ DB_Catalogo  │      │      │   │     │
   └───────┘      └──┴──┘     └──┴──┘
   
   ┌────────────────────────────────────────────┐
   │   Message Broker (RabbitMQ / SQS)          │
   │   Padrão Event-Driven (Assíncrono)         │
   └────────────────────────────────────────────┘
```

#### **Componentes Principais**:

| Serviço | Responsabilidade | Banco | Protocolo |
|---------|------------------|-------|-----------|
| **Catalog** | Catálogo de filmes, sessões, disponibilidade | PostgreSQL | REST |
| **Sales** | Reservas, ingressos, orquestração | Aurora MySQL | REST + Async |
| **Auth** | Autenticação (JWT), autorização | PostgreSQL | REST |
| **Payment** | Processamento de pagamentos, gateway | PostgreSQL | REST + Webhook |
| **Notification** | Emails, SMS, push notifications | N/A | Async Queue |
| **Search** | Full-text search, filtros avançados | OpenSearch/Elastic | REST |

#### **Padrões**:
- ✅ **REST APIs** para comunicação síncrona
- ✅ **Message Broker** (RabbitMQ/SQS) para assíncrono
- ✅ **Database per Service** (isolamento de dados)
- ✅ **DTOs** para serialização
- ✅ **Repository Pattern** para acesso a dados
- ✅ **Service Layer** para lógica de negócio

---

### 2️⃣ **Diagrama de Implantação** (`deployment.puml`)

**FOCO**: Infraestrutura Física/Lógica e Serviços em Nuvem

Representa como o sistema é implantado em ambiente de produção (AWS).

#### **Arquitetura de Infraestrutura**:

```
┌──────────────────────────────────────────────────────────┐
│                    AWS Cloud (us-east-1)                │
├──────────────────────────────────────────────────────────┤
│  ┌──────────────────────────────────────────────────┐   │
│  │  Public Subnet (Internet-Facing)                 │   │
│  │  ┌─────────┐  ┌──────────┐  ┌──────────┐        │   │
│  │  │CloudFront│ │ Route 53 │  │   ALB    │        │   │
│  │  │   CDN    │ │   DNS    │  │Load Bal. │        │   │
│  │  └─────────┘  └──────────┘  └──────────┘        │   │
│  │                │                                  │   │
│  │             ┌──▼──────┐                          │   │
│  │             │ API GW  │                          │   │
│  │             └─────────┘                          │   │
│  └──────────────────────────────────────────────────┘   │
│                 ▼                                        │
│  ┌──────────────────────────────────────────────────┐   │
│  │  Private Subnets (Multi-AZ)                      │   │
│  │  ┌─────────────────────────┐                    │   │
│  │  │   ECS Fargate Cluster   │  (Auto-scaling)    │   │
│  │  │  ┌────────────────────┐ │                    │   │
│  │  │  │ Task 1: Catalog    │ │                    │   │
│  │  │  │ Task 2: Sales      │ │                    │   │
│  │  │  │ Task 3: Auth       │ │                    │   │
│  │  │  │ Task 4: Payment    │ │                    │   │
│  │  │  │ Task 5: Notif      │ │                    │   │
│  │  │  │ Task 6: Search     │ │                    │   │
│  │  │  └────────────────────┘ │                    │   │
│  │  └─────────────────────────┘                    │   │
│  │                                                  │   │
│  │  ┌──────────────────────────────────────────┐  │   │
│  │  │    Data Layer (Database, Cache, Search)  │  │   │
│  │  │  ┌──────┐ ┌──────┐ ┌──────────────────┐ │  │   │
│  │  │  │RDS   │ │Redis │ │OpenSearch/Elastic│ │  │   │
│  │  │  │Aurora│ │Cache │ │Search Index      │ │  │   │
│  │  │  └──────┘ └──────┘ └──────────────────┘ │  │   │
│  │  └──────────────────────────────────────────┘  │   │
│  │                                                  │   │
│  │  ┌──────────────────────────────────────────┐  │   │
│  │  │    Message Queue                          │  │   │
│  │  │  ┌────────────┐  ┌───────────────────┐  │  │   │
│  │  │  │Amazon SQS  │  │  RabbitMQ (EC2)   │  │  │   │
│  │  │  └────────────┘  └───────────────────┘  │  │   │
│  │  └──────────────────────────────────────────┘  │   │
│  └──────────────────────────────────────────────────┘   │
│                                                         │
│  ┌──────────────────────────────────────────────────┐   │
│  │    Observabilidade                               │   │
│  │  ┌─────────────┐ ┌──────┐ ┌────────────┐       │   │
│  │  │ CloudWatch  │ │ X-Ray│ │EventBridge │       │   │
│  │  │Logs, Metrics│ │ Trace│ │ Events     │       │   │
│  │  └─────────────┘ └──────┘ └────────────┘       │   │
│  └──────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────┐
│           Serviços Externos (Third-party)               │
│  ┌────────────────┐  ┌──────────────┐  ┌─────────────┐ │
│  │Payment Gateway │  │Email Service │  │SMS Service  │ │
│  │(Stripe/PayPal) │  │(AWS SES)     │  │(Twilio)     │ │
│  └────────────────┘  └──────────────┘  └─────────────┘ │
└──────────────────────────────────────────────────────────┘
```

#### **Componentes de Infraestrutura**:

| Componente | Função | Tecnologia |
|-----------|--------|-----------|
| **CloudFront** | CDN para assets estáticos | AWS CloudFront |
| **Route 53** | DNS e Load Balancing | AWS Route 53 |
| **ALB** | Load Balancer da aplicação | AWS ALB |
| **API Gateway** | Roteamento de APIs REST | AWS API Gateway |
| **ECS Fargate** | Orquestração de containers (serverless) | AWS ECS Fargate |
| **Aurora RDS** | Banco de dados gerenciado (Multi-AZ) | AWS Aurora |
| **ElastiCache** | Cache distribuído | AWS ElastiCache (Redis) |
| **OpenSearch** | Search engine (full-text) | AWS OpenSearch |
| **Amazon SQS** | Fila de mensagens | AWS SQS |
| **RabbitMQ** | Message broker (self-managed) | EC2 + RabbitMQ |
| **S3** | Object storage (assets, backups) | AWS S3 |
| **CloudWatch** | Logs, métricas, alarms | AWS CloudWatch |
| **X-Ray** | Distributed tracing | AWS X-Ray |

#### **Características de Infraestrutura**:
- ✅ **Multi-AZ** para alta disponibilidade
- ✅ **Auto-scaling** baseado em CPU/Memory
- ✅ **Failover automático** (RDS, ECS)
- ✅ **Criptografia** em trânsito (HTTPS/TLS) e em repouso
- ✅ **VPC Privada** com subnets isoladas
- ✅ **Backup & Disaster Recovery** (RDS, S3)
- ✅ **Observabilidade** completa (CloudWatch, X-Ray)

---

### 3️⃣ **Diagrama de Comunicação** (`communication_booking.puml`)

**FOCO**: Fluxos de Interação e Sequência de Negócio

Representa o fluxo completo de uma transação de reserva + pagamento com 10 fases:

#### **Fases do Fluxo**:

```
1. Autenticação (Login)
2. Navegação no Catálogo
3. Verificação de Disponibilidade
4. Criação da Reserva (Status: PENDENTE)
   └─ Lock otimista na base de dados
   └─ Transação ACID (BEGIN/COMMIT)
5. Publicação de Evento no Message Broker
6. Processamento de Pagamento
   └─ Chamada ao Gateway Externo (Stripe/PayPal)
   └─ Tokenização (PCI-DSS)
7. Confirmação da Reserva (Status: CONFIRMADA)
8. Notificação Pós-Pagamento
   └─ Email com ingressos e QR codes
9. Exibição de Confirmação ao Cliente
10. Caso de Erro: Timeout + Cancelamento Automático
```

#### **Padrões Utilizados**:
- ✅ **Saga Distribuída** (Compensating Transactions)
- ✅ **Event-Driven Architecture** (Publish-Subscribe)
- ✅ **Transações ACID** (Vendas Service)
- ✅ **Retry Logic** com backoff exponencial
- ✅ **Timeout handling** (15 minutos)
- ✅ **PCI-DSS Compliance** para pagamentos

---

## 🔑 **Diferenças Principais: Componentes vs. Implantação**

| Aspecto | Componentes | Implantação |
|---------|-----------|------------|
| **Escopo** | Software Architecture | Infrastructure |
| **O que mostra** | Serviços, módulos, responsabilidades | Servidores, containers, cloud services |
| **Nível de Abstração** | Alto (lógica de negócio) | Baixo (tecnologia física) |
| **Público-alvo** | Arquitetos, Desenvolvedores | DevOps, Infraestrutura, Arquitetos |
| **Detalhes** | APIs, DTOs, Services, Repos | AWS, Docker, Kubernetes, Ports |
| **Relacionamentos** | Chamadas síncronas/assíncronas | Conexões de rede, protocolos |

### **Exemplo Prático**:
- **Componentes**: "Sales Service chama Auth Service via REST para validar usuário"
- **Implantação**: "ECS Task da Sales (port 8080) conecta-se ao ECS Task da Auth (port 8080) via ALB em Private Subnet"

---

## 📚 **Outros Diagramas (Complementares)**

### `actors.puml`
- Identifica **stakeholders**: Usuários, Admin, Sistema de Pagamento, etc.

### `use_cases.puml`
- Mapeia **funcionalidades** do ponto de vista do usuário
- Casos: Buscar filmes, Reservar, Pagar, Receber notificações

### `classes.puml`
- Estrutura **OOP** das classes Java/Kotlin
- Entidades, DTOs, Services, Repositories

### `booking_state.puml`
- **Máquina de Estados** de uma Reserva
- Estados: PENDENTE → CONFIRMADA → CONCLUÍDA / CANCELADA

### `booking_sequence.puml`
- Sequência detalhada de **booking** entre microsserviços

### `data_model.puml`
- **Modelo ER** dos bancos de dados
- Tabelas e relacionamentos

### `system_sequence.puml`
- Fluxo **simplificado** de um caso de uso

### `architecture.puml`
- Visão **geral** da arquitetura

---

## 🛠️ **Tecnologias Utilizadas**

### **Backend**
- Spring Boot / Micronaut / Quarkus
- Java 17+ / Kotlin
- PostgreSQL, MySQL (Aurora)

### **Frontend**
- React / Vue / Angular (SPA Web)
- React Native / Flutter (Mobile)

### **Infraestrutura**
- AWS (VPC, ECS, RDS, S3, CloudFront, CloudWatch)
- Docker (Containerização)
- RabbitMQ / Amazon SQS (Message Queue)
- OpenSearch / Elasticsearch (Search)
- Redis (Cache)

### **DevOps & CI/CD**
- Docker / ECS
- Terraform / CloudFormation (IaC)
- GitHub Actions / AWS CodePipeline
- CloudWatch / X-Ray (Observabilidade)

---

## 📝 **Padrões de Design Implementados**

### **Arquitetura**
- ✅ **Microsserviços**: Independentes, escaláveis
- ✅ **API Gateway**: Roteamento centralizado
- ✅ **Database per Service**: Isolamento de dados
- ✅ **Event-Driven**: Comunicação assíncrona

### **Comunicação**
- ✅ **REST APIs**: Síncrono
- ✅ **Message Queues**: Assíncrono
- ✅ **JWT**: Autenticação stateless
- ✅ **Circuit Breaker**: Resiliência

### **Data**
- ✅ **ACID Transactions**: Consistência
- ✅ **Event Sourcing** (futura)
- ✅ **CQRS** (Command Query Responsibility Segregation)

### **Segurança**
- ✅ **PCI-DSS**: Compliance para pagamentos
- ✅ **Tokenização**: Dados sensíveis protegidos
- ✅ **Encryption**: TLS + AES
- ✅ **VPC Private**: Isolamento de rede

---

## 🚀 **Como Usar**

### **Para Desenvolvedores**:
1. Ler `components.puml` para entender arquitetura
2. Ler `communication_booking.puml` para fluxos
3. Ler `classes.puml` para estrutura OOP

### **Para DevOps/Infraestrutura**:
1. Ler `deployment.puml` para infraestrutura
2. Usar para scaling e disaster recovery

### **Para Arquitetos/Líderes**:
1. Ler `components.puml` + `deployment.puml`
2. Consultar `communication_booking.puml` para fluxos críticos

---

## 📞 **Versão e Status**

- **Última Atualização**: Maio 2025
- **Versão**: 2.0 (Completo e Validado)
- **Status**: ✅ Pronto para uso em produção

---

## 📖 **Tabela de Conteúdo da Documentação**

Este conjunto de diagramas mapeia as seções solicitadas:

```
2. Modelos de Usuário e Requisitos
   └─ 2.1 Descrição de Atores .................... actors.puml
   └─ 2.2 Modelo de Casos de Uso ................ use_cases.puml
   └─ 2.3 Diagrama de Sequência do Sistema ..... system_sequence.puml

3. Modelos de Projeto
   └─ 3.1 Arquitetura .......................... architecture.puml
   └─ 3.2 Diagrama de Componentes (✅ COMPLETO) . components.puml
   └─ 3.3 Diagrama de Classes .................. classes.puml
   └─ 3.4 Diagrama de Sequência ................ booking_sequence.puml
   └─ 3.5 Diagrama de Comunicação (✅ COMPLETO) . communication_booking.puml
   └─ 3.6 Diagrama de Estados .................. booking_state.puml

   👉 PLUS: Diagrama de Implantação (✅ COMPLETO) .. deployment.puml

4. Modelos de Dados
   └─ 4.1 Modelo de Dados ...................... data_model.puml
```

---

Desenvolvido para o projeto **CineFlow - Sistema de Reserva de Ingressos de Cinema**
