# build-orchestrator

Orquestrador de builds.

## Requisitos

Conexão com os sistemas:

- PostgreSQL 14+
- RabbitMQ 3.11+

## Variáveis de ambiente


| Variável | Descrição | Exemplo |
|---|---|---|
| `DATABASE_URL` | String de conexão com o PostgreSQL | `postgresql://user:pass@localhost:5432/devtoolspro` |
| `RABBITMQ_URL` | `build-orchestrator` | String de conexão com o RabbitMQ | `amqp://user:pass@localhost:5672` |
| `PORT` | Porta HTTP em que o serviço escuta | `4000` |
| `NODE_ENV` | Ambiente de execução | `development`, `production` |
| `LOG_LEVEL` | Nível de verbosidade dos logs | `info`, `debug`, `error` |