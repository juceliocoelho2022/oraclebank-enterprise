# 🏦 OracleBank Enterprise

Backend bancário desenvolvido para demonstrar, na prática, a integração entre **Java, Spring Boot e Oracle Database**, explorando recursos utilizados em aplicações corporativas: **PL/SQL, transações, packages, sequences, triggers, auditoria, constraints, locking, índices e análise de performance**.

> 🚧 Projeto em desenvolvimento ativo.

## 🎯 Objetivo

O **OracleBank Enterprise** é um projeto de estudo e portfólio com foco em **Java Backend + Oracle Database**. A proposta é ir além de um CRUD tradicional, implementando operações transacionais, regras de negócio no banco e integração entre uma API REST e PL/SQL.

## 🛠️ Stack

### Backend
- Java 24
- Spring Boot 4.1
- Spring Web
- Spring Data JPA
- Spring JDBC
- Bean Validation
- Spring Boot Actuator
- Maven
- HikariCP

### Database
- Oracle AI Database 26ai Free
- Oracle JDBC
- SQL e PL/SQL
- Packages, Procedures e Functions
- Sequences e Triggers
- Views e Constraints
- Indexes
- Cost-Based Optimizer (CBO)

### Infraestrutura
- Docker
- Docker Compose
- Git / GitHub

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

Alterações de saldo também alimentam a estrutura de auditoria:

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

Endpoint base:

```http
/api/v1/clientes
```

Operações implementadas:

```text
GET  /api/v1/clientes
GET  /api/v1/clientes/{id}
POST /api/v1/clientes
```

Exemplo de criação:

```json
{
  "cpf": "45678901234",
  "nome": "Fernanda Lima",
  "email": "fernanda.lima@oraclebank.local",
  "telefone": "11999990004"
}
```

A API possui validações de entrada e tratamento centralizado de erros HTTP.

## 🛡️ Integridade dos dados

O Oracle atua como camada final de garantia da consistência dos dados. Entre as regras implementadas estão:

- CPF único
- e-mail único
- chave PIX única
- relacionamentos por foreign keys
- validação de status
- validação do tipo de conta
- valor de transação maior que zero
- limite da conta não negativo

As validações Java melhoram a experiência da API, enquanto as **constraints do Oracle** permanecem como garantia final de integridade.

## 💰 Transferências bancárias

Endpoint:

```http
POST /api/v1/transferencias
```

Exemplo:

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

O package `PKG_TRANSFERENCIAS` concentra operações bancárias importantes, incluindo:

```text
REALIZAR_TRANSFERENCIA
CONSULTAR_SALDO
```

A transferência valida, entre outros pontos:

- valor maior que zero
- origem diferente do destino
- existência das contas
- status das contas
- saldo disponível

## 🔒 Transações e concorrência

Durante uma transferência, o projeto utiliza:

```sql
SELECT ... FOR UPDATE
```

Isso permite bloquear as linhas envolvidas enquanto a operação financeira está em andamento.

```text
SELECT FOR UPDATE
       │
       ▼
Validar regras
       │
       ▼
Debitar origem
       │
       ▼
Creditar destino
       │
       ▼
Registrar transação
       │
       ▼
COMMIT
```

Em caso de falha, a operação executa `ROLLBACK`, preservando a atomicidade.

## 🧾 Auditoria automática

O trigger `TRG_AUDITORIA_CONTAS` registra automaticamente mudanças de saldo em `AUDITORIA_CONTAS`, incluindo:

- conta
- saldo anterior
- saldo novo
- operação
- usuário do banco
- timestamp

Uma transferência executada pela API Java confirmou o fluxo completo entre REST, PL/SQL, atualização das contas e auditoria automática.

## 🔢 Sequences

O projeto utiliza sequences dedicadas, incluindo:

```text
SEQ_CLIENTES
SEQ_CONTAS
SEQ_TRANSACOES
SEQ_PIX_KEYS
SEQ_AUDITORIA_CONTAS
```

Os laboratórios também exploram o comportamento de `CACHE` em sequences Oracle.

## 👁️ Views

Views implementadas:

```text
VW_CONTAS_CLIENTES
VW_TRANSFERENCIAS
```

## 📊 SQL Analítico

O projeto explora recursos como:

```sql
GROUP BY
CTE
ROW_NUMBER()
RANK()
LAG()
SUM() OVER()
PARTITION BY
```

## 🚀 Performance e otimização

Foi criado um laboratório de performance com dezenas de milhares de transações para estudar:

- índices simples e compostos
- covering indexes
- seletividade
- cardinalidade
- clustering factor
- estatísticas
- Cost-Based Optimizer
- execution plans

Recursos utilizados:

```sql
EXPLAIN PLAN
DBMS_XPLAN.DISPLAY
DBMS_XPLAN.DISPLAY_CURSOR
DBMS_STATS
```

Métricas analisadas incluem `E-Rows`, `A-Rows`, `Buffers` e `Cost`.

Índices utilizados nos laboratórios incluem:

```text
IDX_TRANSACOES_STATUS
IDX_TRANSACOES_ORIGEM_STATUS_DATA
IDX_TRANSACOES_COVER
```

Os testes permitem comparar estratégias como `TABLE ACCESS FULL` e `INDEX RANGE SCAN`.

## ❤️ Health Check

A aplicação utiliza Spring Boot Actuator:

```http
GET /actuator/health
```

O health check confirma o fluxo:

```text
Spring Boot → HikariCP → Oracle JDBC → Oracle Database
```

## 🐳 Oracle com Docker

```bash
docker compose -f docker/compose.yaml up -d
```

Arquitetura local:

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

Credenciais reais não devem ser versionadas. A configuração do backend pode utilizar variáveis de ambiente:

```yaml
spring:
  datasource:
    url: ${DB_URL:jdbc:oracle:thin:@//localhost:1521/FREEPDB1}
    username: ${DB_USERNAME:oraclebank}
    password: ${DB_PASSWORD}
```

## 🧪 Cenários validados

- Spring Boot → Oracle Database
- POST cliente → `201 Created`
- GET clientes → `200 OK`
- CPF duplicado → `409 Conflict`
- e-mail duplicado → `409 Conflict`
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

`Java` • `Spring Boot` • `REST APIs` • `Spring Data JPA` • `Spring JDBC` • `Oracle Database` • `SQL` • `PL/SQL` • `Stored Procedures` • `Packages` • `Sequences` • `Triggers` • `Views` • `Constraints` • `Transactions` • `COMMIT / ROLLBACK` • `SELECT FOR UPDATE` • `Database Auditing` • `Indexes` • `Execution Plans` • `Oracle CBO` • `Query Optimization` • `Docker`

## 👨‍💻 Autor

**Jucelio Farias Coelho**

Desenvolvedor Backend com foco em **Java, Spring Boot, APIs REST, Oracle Database, SQL, PL/SQL, Docker e Cloud**.

GitHub: **@juceliocoelho2022**

---

⭐ Projeto desenvolvido como parte da minha jornada de aprofundamento em **Java Backend e Oracle Database**, com foco em aplicações corporativas, transacionais e boas práticas de engenharia de software.
