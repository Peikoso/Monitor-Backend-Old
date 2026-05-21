# Monitor Backend (Old)

Sistema de monitoramento e execução automatizada de regras SQL com notificações em tempo real.

## 📋 Pré-requisitos

- **Node.js** 18+ 
- **PostgreSQL** 14+
- **pnpm** (gerenciador de pacotes)
- **Firebase Admin SDK** configurado

## 🚀 Configuração Inicial

### 1. Instalar Dependências

```bash
pnpm install
```

### 2. Configurar Variáveis de Ambiente

Copie o arquivo `.env-example` para `.env`:

```bash
cp .env-example .env
```

Configure as variáveis no arquivo `.env`:

```env
DATABASE_URL=postgresql://usuario:senha@localhost:5432/qqmonitor
PORT=8000
SERVICE_PATH=./plantao-monitor-firebase-adminsdk.json
DEFAULT_PASSWORD=sua_senha_padrao_para_criacao_de_usuarios
ENABLE_RUNNER_WORKER=true
```

### 3. Aplicar Migrations no Banco de Dados

Execute os scripts SQL na seguinte ordem:

```bash
# 1. Criar estrutura do banco
psql -h localhost -U seu_usuario -d qqmonitor -f script.sql

# 2. Popular com dados de teste
psql -h localhost -U seu_usuario -d qqmonitor -f some-inserts.sql
```

**Ou via DATABASE_URL:**

```bash
psql $DATABASE_URL -f script.sql
psql $DATABASE_URL -f some-inserts.sql
```

### 4. Configurar Firebase Admin SDK

Coloque o arquivo JSON do Firebase Admin SDK na raiz do projeto com o nome:
```
plantao-monitor-firebase-adminsdk.json
```

## ▶️ Executar Aplicação

```bash
node src/main.js
```

A API estará disponível em: `http://localhost:8000`

## 🧪 Testar Endpoint /db-test

Use o comando curl abaixo, substituindo `SEU_TOKEN_FIREBASE` pelo token válido do Firebase:
O usuário deve estar autenticado no Firebase e criado no banco de dados com profile = 'admin' para acessar este endpoint.

```bash
curl -H "Authorization: Bearer SEU_TOKEN_FIREBASE" http://localhost:8000/api/v1/db-test
```

### Resposta esperada:

```json
{
  "currentTime": "2025-11-28T03:58:42.451Z",
  "pgVersion": "PostgreSQL 17.4 on x86_64-windows, compiled by msvc-19.42.34436, 64-bit",
  "tableCounts": {
    "users": 5,
    "user_preferences": 3,
    "user_preferences_channels": 4,
    "channels": 4,
    "rules": 5,
    "rules_roles": 5,
    "roles": 5,
    "users_roles": 3,
    "incidents": 6,
    "schedules": 5,
    "incidents_events": 5,
    "runners": 5,
    "runner_queue": 33,
    "runner_logs": 94,
    "audit_logs": 5,
    "notifications": 5,
    "sql_test_logs": 5,
    "escalation_policy": 5,
    "app_settings": 5,
    "plantao_monitor": 1
  }
}
```

## 🤖 Sistema Runner

O sistema Runner é iniciado automaticamente com a aplicação quando `ENABLE_RUNNER_WORKER=true`.

### Componentes:

- **Runner Scheduler**: Agenda regras para execução baseado no intervalo configurado
- **Runner Worker**: Executa as queries SQL e grava resultados em `runner_logs`

### Logs do Runner:

Você verá logs no console como:

```
[Runner Scheduler] Verificando 5 runner(s) para agendamento...
[Runner Scheduler] Runner agendado para regra: "Verificar CPU Alta"
[Runner Worker] 1 runner(s) encontrado(s).
[Runner Worker] Job abc-123 concluído com sucesso. Linhas afetadas: 3
[Runner Metrics] {"timestamp":"2025-11-28T...", "queueId":"abc-123", ...}
```

### Verificar Execuções:

```sql
-- Ver últimas execuções
SELECT * FROM runner_logs ORDER BY executed_at DESC LIMIT 10;

-- Ver fila de runners
SELECT * FROM runner_queue WHERE status = 'PENDING';

-- Ver status dos runners
SELECT r.*, ru.name FROM runners r 
JOIN rules ru ON r.rule_id = ru.id;
```

## 📁 Estrutura do Projeto

```
qqtech9-backend/
├── src/
│   ├── config/          # Configurações (DB, Firebase)
│   ├── controllers/     # Controladores das rotas
│   ├── dto/             # Data Transfer Objects
│   ├── middleware/      # Middlewares (Auth, Errors, Validação)
│   ├── models/          # Modelos de negócio
│   ├── repositories/    # Acesso a dados (SQL)
│   ├── routers/         # Definição de rotas
│   ├── services/        # Lógica de negócio
│   ├── utils/           # Utilitários
│   ├── worker/          # Runner Scheduler e Worker
│   └── main.js          # Ponto de entrada
├── script.sql           # Migration principal
├── some-inserts.sql     # Dados de teste
└── package.json
```

## 📊 Métricas e Monitoramento

O sistema gera métricas em formato JSON no console:

```json
{
  "timestamp": "2025-11-28T10:30:00.000Z",
  "queueId": "abc-123",
  "runnerId": "def-456",
  "ruleId": "ghi-789",
  "runTimeMs": 1250,
  "status": "SUCCESS",
  "rowsAffected": 3
}
```

---

**Desenvolvido para QQTech** | Versão 1.0.0
