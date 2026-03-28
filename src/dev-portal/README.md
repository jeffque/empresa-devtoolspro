# dev-portal

Portal do desenvolvedor.

## Requisitos

Conexão com os sistemas:

- PostgreSQL 14+

## Variáveis de ambiente


| Variável | Descrição | Exemplo |
|---|---|---|
| `DATABASE_URL` | String de conexão com o PostgreSQL | `postgresql://user:pass@localhost:5432/devtoolspro` |
| `PORT` | Porta HTTP em que o serviço escuta | `4000` |
| `NODE_ENV` | Ambiente de execução | `development`, `production` |
| `REACT_APP_API_URL` | `dev-portal` | URL base da API consumida pelo frontend | `http://localhost:4000/api` |
| `LOG_LEVEL` | Nível de verbosidade dos logs | `info`, `debug`, `error` |