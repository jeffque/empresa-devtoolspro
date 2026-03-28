# quality-gate-service

Serviço de gate de qualidade


## Requisitos

Conexão com os sistemas:

- PostgreSQL 14+

## Variáveis de ambiente


| Variável | Descrição | Exemplo |
|---|---|---|
| `DATABASE_URL` | String de conexão com o PostgreSQL | `postgresql://user:pass@localhost:5432/devtoolspro` |
| `PORT` | Porta HTTP em que o serviço escuta | `4000` |
| `NODE_ENV` | Ambiente de execução | `development`, `production` |
| `LOG_LEVEL` | Nível de verbosidade dos logs | `info`, `debug`, `error` |