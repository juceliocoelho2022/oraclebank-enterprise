# 🏦 OracleBank Enterprise

Backend bancário desenvolvido para demonstrar, na prática, a integração entre **Java, Spring Boot e Oracle Database**, explorando recursos utilizados em aplicações corporativas: **PL/SQL, transações, packages, sequences, triggers, auditoria, constraints, locking, índices e análise de performance**.

> 🚧 Projeto em desenvolvimento ativo.

## 🖥️ Conceito da interface

A imagem abaixo representa a direção visual planejada para a página inicial pública do OracleBank Enterprise, com experiência bancária premium, atendimento personalizado, produtos financeiros, segurança e acesso aos canais digitais.

![OracleBank Enterprise - Homepage](docs/oraclebank-homepage.png)

> **Nota:** este é um conceito visual da interface planejada. O frontend será implementado e integrado progressivamente à API Java/Spring Boot.

## 🚀 Stacks utilizadas no projeto

### ☕ Backend Java
- **Java 24** — linguagem principal do backend
- **Spring Boot 4.1** — framework base da aplicação
- **Spring Web** — construção da API REST
- **Spring Data JPA** — persistência e CRUD
- **Hibernate ORM** — mapeamento objeto-relacional
- **Spring JDBC / JdbcTemplate** — integração direta com PL/SQL
- **CallableStatement** — execução das procedures Oracle
- **Jakarta Bean Validation** — validação dos DTOs da API
- **Spring Boot Actuator** — health checks da aplicação e banco
- **HikariCP** — pool de conexões JDBC
- **Maven 3.9.x** — build e gerenciamento de dependências

### 🗄️ Oracle Database
- **Oracle AI Database 26ai Free**
- **Oracle JDBC / OJDBC11**
- **SQL e PL/SQL**
- **Packages, Stored Procedures e Functions**
- **Sequences e Triggers**
- **Views**
- **Primary Keys / Foreign Keys**
- **Unique Constraints / Check Constraints**
- **Transactions — COMMIT / ROLLBACK**
- **SELECT FOR UPDATE**
- **Database Auditing**

### 📊 Performance Oracle
- **Oracle Cost-Based Optimizer (CBO)**
- **EXPLAIN PLAN**
- **DBMS_XPLAN.DISPLAY / DISPLAY_CURSOR**
- **DBMS_STATS**
- **Indexes simples, compostos e covering indexes**
- **INDEX RANGE SCAN / TABLE ACCESS FULL**
- análise de **E-Rows / A-Rows / Buffers / Cost**
- seletividade, cardinalidade e clustering factor

### 🐳 Infraestrutura e desenvolvimento
- **Docker**
- **Docker Compose**
- **Oracle Database em container**
- **Git / GitHub**
- **IntelliJ IDEA**
- **PowerShell**
- **SQL*Plus**

### 🔌 Arquitetura e padrões
- **REST API**
- **Layered Architecture**
- **Controller / Service / Repository**
- **DTO Pattern**
- **Centralized Exception Handling**
- **Database Constraints como última camada de integridade**
- **Pessimistic row locking via SELECT FOR UPDATE**
- **Database-driven auditing**
- **PL/SQL para regras transacionais críticas**

## 🎯 Objetivo

O **OracleBank Enterprise** é um projeto de estudo e portfólio com foco em **Java Backend + Oracle Database**. A proposta é ir além de um CRUD tradicional, implementando operações transacionais, regras de negócio no banco e integração entre uma API REST e PL/SQL.

## 🏗️ Arquitetura

```text
Client
  │
  ▼
REST API
  │
  ▼
Controller
  │
  ▼
Service
  │
  ├────────► Spring Data JPA ───────► Oracle Database
  │
  └────────► Spring JDBC
                  │
                  ▼
           CallableStatement
                  │
                  ▼
             PL/SQL Package
                  │
                  ▼
           Oracle Database
```

O CRUD de clientes utiliza **Spring Data JPA**. Operações bancárias críticas utilizam **Spring JDBC + CallableStatement** para executar regras implementadas em PL/SQL.

## 📂 Estrutura

```text
oraclebank-enterprise/
├── backend/
│   ├── src/main/java/br/com/oraclebank/
│   │   ├── controller/
│   │   ├── domain/
│   │   ├── dto/
│   │   ├── exception/
│   │   ├── repository/
│   │   └── service/
│   ├── src/main/resources/
│   └── pom.xml
├── database/
│   ├── ddl/
│   ├── plsql/
│   ├── queries/
│   └── seed/
├── docker/
│   └── compose.yaml
├── docs/
│   └── oraclebank-homepage.png
└── README.md
```

## 🗄️ Modelo de dados

```text
CLIENTES
   │
   └── CONTAS
        ├── TRANSACOES
        └── PIX_KEYS
```

Alterações de saldo alimentam automaticamente a auditoria:

```text
CONTAS
   │ UPDATE SALDO
   ▼
TRG_AUDITORIA_CONTAS
   │
   ▼
AUDITORIA_CONTAS
```

## 👤 API de Clientes

```text
GET  /api/v1/clientes
GET  /api/v1/clientes/{id}
POST /api/v1/clientes
```

Exemplo:

```json
{
  "cpf": "45678901234",
  "nome": "Fernanda Lima",
  "email": "fernanda.lima@oraclebank.local",
  "telefone": "11999990004"
}
```

A API possui Bean Validation e tratamento centralizado de erros HTTP.

## 🛡️ Integridade dos dados

Entre as regras implementadas estão CPF e e-mail únicos, chave PIX única, foreign keys, validação de status, tipo de conta, valores positivos e limites não negativos. As validações Java melhoram a experiência da API, enquanto as **constraints do Oracle** permanecem como garantia final de integridade.

## 💰 Transferências bancárias

```http
POST /api/v1/transferencias
```

```json
{
  "contaOrigemId": 1,
  "contaDestinoId": 2,
  "valor": 100.00
}
```

Fluxo:

```text
POST /api/v1/transferencias
          │
          ▼
TransferenciaController
          │
          ▼
TransferenciaService
          │
          ▼
TransferenciaRepository
          │
          ▼
JdbcTemplate / CallableStatement
          │
          ▼
PKG_TRANSFERENCIAS.REALIZAR_TRANSFERENCIA
          │
          ▼
Oracle Database
```

## ⚙️ PL/SQL

O package `PKG_TRANSFERENCIAS` concentra operações bancárias importantes:

```text
REALIZAR_TRANSFERENCIA
CONSULTAR_SALDO
```

A transferência valida valor, contas de origem/destino, status e saldo disponível.

## 🔒 Transações e concorrência

A transferência utiliza `SELECT ... FOR UPDATE` para bloquear as linhas envolvidas durante a operação. O fluxo atual executa validações, débito, crédito, registro da transação e `COMMIT`; em caso de falha, executa `ROLLBACK`.

## 🧾 Auditoria automática

O trigger `TRG_AUDITORIA_CONTAS` registra mudanças de saldo em `AUDITORIA_CONTAS`, incluindo conta, saldo anterior, saldo novo, operação, usuário do banco e timestamp.

## 🔢 Sequences e Views

Sequences:

```text
SEQ_CLIENTES
SEQ_CONTAS
SEQ_TRANSACOES
SEQ_PIX_KEYS
SEQ_AUDITORIA_CONTAS
```

Views:

```text
VW_CONTAS_CLIENTES
VW_TRANSFERENCIAS
```

## 📊 SQL Analítico

O projeto explora `GROUP BY`, CTE, `ROW_NUMBER()`, `RANK()`, `LAG()`, `SUM() OVER()` e `PARTITION BY`.

## 🚀 Performance e otimização

Foi criado um laboratório com dezenas de milhares de transações para estudar índices, seletividade, cardinalidade, clustering factor, estatísticas, Oracle CBO e execution plans.

Índices utilizados incluem:

```text
IDX_TRANSACOES_STATUS
IDX_TRANSACOES_ORIGEM_STATUS_DATA
IDX_TRANSACOES_COVER
```

## ❤️ Health Check

```http
GET /actuator/health
```

Fluxo validado:

```text
Spring Boot → HikariCP → Oracle JDBC → Oracle Database
```

## 🐳 Oracle com Docker

```bash
docker compose -f docker/compose.yaml up -d
```

```text
Spring Boot :8080
      │
      ▼
Oracle JDBC
      │
      ▼
Oracle Database :1521
      │
      ▼
FREEPDB1
```

## 🔐 Segurança de configuração

Credenciais reais não devem ser versionadas. A aplicação utiliza configuração preparada para variáveis de ambiente.

## 🧪 Cenários validados

- Spring Boot → Oracle Database
- POST cliente → `201 Created`
- GET clientes → `200 OK`
- CPF/e-mail duplicados → `409 Conflict`
- dados inválidos → `400 Bad Request`
- cliente inexistente → `404 Not Found`
- Java → JDBC → PL/SQL
- transferência bancária concluída
- atualização atômica de saldos
- trigger de auditoria
- constraints Oracle
- rollback PL/SQL
- execution plans
- index range scan
- index-only access

## 🗺️ Roadmap

- [ ] Mapear erros `ORA-20xxx` para respostas REST de domínio
- [ ] Retornar ID da transação na API
- [ ] API de contas
- [ ] Implementar frontend baseado no conceito visual
- [ ] JUnit e Mockito
- [ ] Testes de integração
- [ ] JaCoCo
- [ ] OpenAPI / Swagger
- [ ] Revisar ownership das transações Java/PLSQL
- [ ] Prevenção de deadlocks com lock ordering determinístico
- [ ] Flyway
- [ ] Dockerizar backend
- [ ] Observabilidade e métricas
- [ ] CI/CD com GitHub Actions

## 🧠 Competências demonstradas

`Java 24` • `Spring Boot 4.1` • `REST APIs` • `Spring Data JPA` • `Hibernate` • `Spring JDBC` • `JdbcTemplate` • `CallableStatement` • `Bean Validation` • `Actuator` • `HikariCP` • `Maven` • `Oracle Database 26ai` • `Oracle JDBC` • `SQL` • `PL/SQL` • `Stored Procedures` • `Packages` • `Functions` • `Sequences` • `Triggers` • `Views` • `Constraints` • `Transactions` • `COMMIT / ROLLBACK` • `SELECT FOR UPDATE` • `Database Auditing` • `Indexes` • `Execution Plans` • `Oracle CBO` • `Query Optimization` • `Docker` • `Docker Compose` • `Git` • `GitHub`

## 👨‍💻 Autor

**Jucelio Farias Coelho**

Desenvolvedor Backend com foco em **Java, Spring Boot, APIs REST, Oracle Database, SQL, PL/SQL, Docker e Cloud**.

GitHub: **@juceliocoelho2022**

---

⭐ Projeto desenvolvido como parte da minha jornada de aprofundamento em **Java Backend e Oracle Database**, com foco em aplicações corporativas, transacionais e boas práticas de engenharia de software.
