# README Técnico - DevToolsPro

## Metadados do Documento
- Documento: README Técnico - DevToolsPro
- Versão: 1.0.0
- Status: Em revisão
- Responsável (owner): Equipe DevToolsPro
- Aprovador: A definir
- Última atualização: 2026-03-27
- Próxima revisão: 2026-06-27
- Público-alvo: Desenvolvedores, analistas de qualidade e equipes de operações
- Classificação da informação: Interna

## Premissas, Lacunas e Riscos (preenchimento obrigatório)

### Premissas
- A stack descrita (Node.js, React, PostgreSQL, RabbitMQ) está em uso em todos os módulos da plataforma.
- Os três módulos documentados (`quality-gate-service`, `build-orchestrator`, `dev-portal`) representam o escopo completo do produto em sua versão atual.
- O ambiente de execução é baseado em containers Docker com orquestração via Docker Compose ou equivalente.
- As equipes consumidoras vão consumir os produtos via interfaces agnósticas de tecnologias (YAML para descrever pipelines do build-orchestrator, HTTP para enviar comandos aos produtos)

### Lacunas de informação
- Não há documentação oficial das variáveis de ambiente utilizadas por cada módulo. Os valores descritos neste documento foram inferidos a partir da stack e das convenções comuns do ecossistema Node.js.
- Não existem registros formais de incidentes anteriores para compor a seção de troubleshooting. Os cenários listados foram elaborados com base nos desafios conhecidos.
- Detalhes sobre a estratégia de deploy em produção (CI/CD, ambientes de staging, rollback) não estão disponíveis.
- Métricas e alertas de observabilidade não possuem especificação formal; as recomendações apresentadas são baseadas em boas práticas para a stack utilizada.

### Riscos identificados
| Risco | Impacto | Mitigação sugerida |
|---|---|---|
| Variáveis de ambiente não padronizadas entre módulos | Falhas em deploys e inconsistência entre ambientes | Criar um esquema centralizado (`.env.example`) para cada módulo com validação no startup |
| Ausência de fluxo único de troubleshooting | Aumento do tempo de resolução de incidentes pelo suporte | Adotar e manter a tabela de troubleshooting deste documento como referência oficial |
| README desatualizado após novas releases | Onboarding prejudicado e retrabalho de suporte | Incluir revisão do README como item obrigatório no checklist de release |
| Dependência de RabbitMQ sem redundância documentada | Perda de mensagens e interrupção de pipelines | Documentar configuração de alta disponibilidade e dead-letter queues |

---

## 1. Visão Geral
- **Nome do produto:** DevToolsPro
- **Problema que resolve:** Equipes de desenvolvimento carecem de uma plataforma unificada para automação de qualidade de código e gestão de builds. Sem essa centralização, os processos de análise estática, execução de pipelines e acompanhamento de builds ficam fragmentados, gerando retrabalho e dificultando a visibilidade sobre a saúde do código.
- **Público-alvo:** Desenvolvedores de software, analistas de qualidade (QA) e equipes de operações (Ops/SRE) que participam do ciclo de entrega de software.
- **Escopo funcional:**
  - Consolidação de resultados de análise estática de código.
  - Orquestração e execução de pipelines de build internos.
  - Interface web para acompanhamento de builds, métricas de qualidade e status de pipelines.

## 2. Arquitetura em Alto Nível

### Componentes principais

| Módulo | Responsabilidade | Tecnologia |
|---|---|---|
| `quality-gate-service` | Consolida resultados de análise estática de código e aplica critérios de aprovação (quality gates) | Node.js, PostgreSQL |
| `build-orchestrator` | Coordena a execução de pipelines internos de build, gerenciando filas e status de execução | Node.js, RabbitMQ, PostgreSQL |
| `dev-portal` | Interface web para que os times acompanhem builds, métricas de qualidade e status de pipelines | React |

### Integrações
- **PostgreSQL:** Persistência de dados de builds, resultados de quality gates e configurações.
- **RabbitMQ:** Mensageria assíncrona entre o `build-orchestrator` e os demais serviços, garantindo desacoplamento na execução de pipelines.
- **Ferramentas de análise estática:** O `quality-gate-service` consome relatórios de ferramentas como ESLint, SonarQube ou equivalentes.
- **Prometheus** e **Grafana:** para coletar os dados de observabilidade e exibir os dashboards.

### Fluxo resumido de execução

```
Desenvolvedor faz push
        │
        ▼
build-orchestrator recebe evento
        │
        ▼
Pipeline de build é enfileirado (RabbitMQ)
        │
        ▼
Build é executado e resultados são persistidos (PostgreSQL)
        │
        ▼
quality-gate-service consolida análise estática
        │
        ▼
Resultado aprovado/reprovado é registrado
        │
        ▼
dev-portal exibe status atualizado para o time
```

## 3. Pré-requisitos

### Requisitos de ambiente
- Sistema operacional: Linux, macOS ou Windows (com WSL2 recomendado)
- Docker 20.10+ e Docker Compose 2.x
- Node.js 18 LTS ou superior
- npm 9+ ou yarn 1.22+

### Dependências essenciais
- PostgreSQL 14+
- RabbitMQ 3.11+
- Navegador moderno (Chrome, Firefox, Edge) para acesso ao `dev-portal`

### Permissões necessárias
- Acesso de leitura/escrita ao repositório do projeto.
- Credenciais de conexão ao banco de dados PostgreSQL.
- Credenciais de acesso ao RabbitMQ (usuário e senha).
- Permissão de rede para comunicação entre os serviços (portas padrão: 5432 para PostgreSQL, 5672/15672 para RabbitMQ, 3000 para dev-portal, 4000 para APIs backend).

## 4. Configuração

### Variáveis de ambiente relevantes

| Variável | Módulo | Descrição | Exemplo |
|---|---|---|---|
| `DATABASE_URL` | Todos os serviços backend | String de conexão com o PostgreSQL | `postgresql://user:pass@localhost:5432/devtoolspro` |
| `RABBITMQ_URL` | `build-orchestrator` | String de conexão com o RabbitMQ | `amqp://user:pass@localhost:5672` |
| `PORT` | Cada serviço | Porta HTTP em que o serviço escuta | `4000` |
| `NODE_ENV` | Todos | Ambiente de execução | `development`, `production` |
| `REACT_APP_API_URL` | `dev-portal` | URL base da API consumida pelo frontend | `http://localhost:4000/api` |
| `LOG_LEVEL` | Todos os serviços backend | Nível de verbosidade dos logs | `info`, `debug`, `error` |

> **Nota:** Devido à lacuna identificada na padronização de variáveis, recomenda-se criar um arquivo `.env.example` na raiz de cada módulo como referência.

### Arquivos de configuração
- `.env` (ou `.env.local`): variáveis de ambiente por módulo (não versionado).
- `docker-compose.yml`: orquestração dos serviços e dependências de infraestrutura.
- `package.json`: dependências e scripts de cada módulo.

### Padrões de segurança aplicados
- Variáveis sensíveis (senhas, tokens) devem ser fornecidas exclusivamente via variáveis de ambiente, nunca hardcoded no código.
- Arquivos `.env` devem constar no `.gitignore`.
- Conexões com PostgreSQL e RabbitMQ devem utilizar TLS em ambientes de produção.

## 5. Operação (uso técnico)

### Como iniciar o sistema

**Com Docker Compose (recomendado):**
```bash
# Subir toda a infraestrutura e os serviços
docker compose up -d

# Verificar status dos containers
docker compose ps
```

**Desenvolvimento local (por módulo):**
```bash
# Instalar dependências
cd src/quality-gate-service && npm install
cd src/build-orchestrator && npm install
cd src/dev-portal && npm install

# Iniciar cada serviço
npm run dev   # em cada diretório de módulo
```

### Fluxos mais comuns
1. **Acompanhar um build:** Acessar o `dev-portal` no navegador, navegar até o painel de builds e filtrar pelo repositório ou branch desejado.
2. **Consultar quality gate:** No `dev-portal`, acessar a seção de qualidade para visualizar resultados de análise estática consolidados pelo `quality-gate-service`.
3. **Disparar pipeline manualmente:** Enviar requisição à API do `build-orchestrator` para enfileirar uma nova execução de pipeline.

### Exemplo de uso (descritivo)
Um desenvolvedor realiza push em uma branch de feature. O `build-orchestrator` detecta o evento, enfileira o build via RabbitMQ e inicia a execução do pipeline. Ao término, o `quality-gate-service` processa os relatórios de análise estática e registra se o código atende aos critérios de qualidade configurados. O resultado (aprovado ou reprovado) fica disponível no `dev-portal`, onde o time pode verificar o status antes de prosseguir com o merge.

## 6. Build e Entrega

### Como o produto é empacotado
- Cada módulo backend é empacotado como imagem Docker independente.
- O `dev-portal` (React) é compilado em assets estáticos via `npm run build` e servido por um servidor web (Nginx ou equivalente) dentro de um container.

### Como ocorre a publicação
- As imagens Docker são construídas e publicadas em um registry interno.
- O deploy é realizado via atualização das imagens nos ambientes-alvo (staging e produção).

### Critérios de qualidade antes da entrega
- Todos os testes unitários e de integração devem passar (`npm test`).
- O quality gate do `quality-gate-service` deve estar aprovado (sem violações críticas).
- Revisão de código (code review) concluída e aprovada.
- Build reproduzível a partir da branch de release.

## 7. Observabilidade e Suporte

### Logs e métricas principais
- **Logs:** Cada serviço backend emite logs estruturados (JSON) para stdout, capturáveis pelo runtime de containers. Nível configurável via `LOG_LEVEL`.
- **Métricas recomendadas:**
  - Tempo médio de execução de builds.
  - Taxa de aprovação/reprovação nos quality gates.
  - Tamanho da fila de builds pendentes no RabbitMQ.
  - Latência de resposta das APIs.
  - Uso de conexões no pool do PostgreSQL.

### Alertas relevantes
| Alerta | Condição | Severidade |
|---|---|---|
| Fila de builds acumulada | Mais de 50 mensagens pendentes no RabbitMQ por mais de 10 minutos | Alta |
| Falha de conexão com PostgreSQL | Serviço não consegue conectar ao banco | Crítica |
| Taxa de erros HTTP elevada | Mais de 5% de respostas 5xx em janela de 5 minutos | Alta |
| Build travado | Execução de build sem atualização de status por mais de 30 minutos | Média |

### Procedimento de suporte inicial
1. Verificar o status dos containers: `docker compose ps`.
2. Consultar logs do serviço afetado: `docker compose logs <serviço> --tail=100`.
3. Verificar conectividade com PostgreSQL e RabbitMQ.
4. Consultar a tabela de troubleshooting abaixo.
5. Escalar para a equipe de desenvolvimento caso o problema persista após os passos acima.

## 8. Troubleshooting

| Sintoma | Causa provável | Ação recomendada |
|---|---|---|
| `dev-portal` não carrega dados | API backend indisponível ou `REACT_APP_API_URL` incorreta | Verificar se os serviços backend estão rodando e se a variável aponta para a URL correta |
| Build permanece em status "pendente" | RabbitMQ indisponível ou consumer do `build-orchestrator` parado | Verificar status do RabbitMQ (`docker compose logs rabbitmq`) e reiniciar o `build-orchestrator` |
| Erro de conexão com o banco de dados | PostgreSQL fora do ar ou `DATABASE_URL` incorreta | Validar se o container do PostgreSQL está ativo e se a string de conexão está correta |
| Quality gate não retorna resultado | Relatório de análise estática não foi gerado ou não foi recebido pelo serviço | Verificar se a ferramenta de análise estática executou corretamente e se o relatório está acessível |
| Tempo de build excessivamente longo | Recursos insuficientes no host ou muitos builds enfileirados | Verificar uso de CPU/memória do host; considerar escalar workers do `build-orchestrator` |
| Mensagens perdidas no RabbitMQ | Filas não configuradas como duráveis | Configurar filas como `durable: true` e habilitar dead-letter queues para mensagens não processadas |

## 9. Limitações e Débitos Técnicos

### Limitações conhecidas
- A plataforma não suporta execução de builds em ambientes multi-cloud; a orquestração é limitada à infraestrutura interna.
- O `dev-portal` não possui autenticação/autorização própria; depende de integração com provedor de identidade externo.
- Não há suporte nativo a notificações (e-mail, Slack) sobre status de builds ou quality gates.

### Débitos técnicos
- **Variáveis de ambiente não padronizadas:** Cada módulo define suas variáveis de forma independente, sem schema compartilhado ou validação centralizada.
- **README técnico desatualizado:** Até a elaboração deste documento, a documentação não acompanhava a evolução dos módulos (este documento visa resolver esta lacuna).
- **Ausência de fluxo único de troubleshooting:** O time de suporte não possuía referência centralizada para diagnóstico de problemas (a seção 8 deste documento é o primeiro passo para resolver isso).
- **Testes de integração limitados:** Cobertura focada em testes unitários; testes end-to-end entre os módulos ainda são manuais.

### Riscos operacionais
- Falha no RabbitMQ pode causar perda de mensagens de build caso as filas não estejam configuradas como duráveis.
- Ausência de mecanismo de retry automático em caso de falha parcial de pipelines.
- Escalabilidade horizontal do `build-orchestrator` não está documentada nem testada.

## 10. Roadmap e Evolução

### Melhorias planejadas
- Padronização de variáveis de ambiente com schema compartilhado e validação automática no startup de cada serviço.
- Implementação de notificações (Slack, e-mail) para eventos de build e quality gate.
- Dashboard de métricas de observabilidade integrado ao `dev-portal`.
- Autenticação e controle de acesso no `dev-portal` via SSO/OIDC.

### Prioridades de curto prazo
1. Padronizar e documentar variáveis de ambiente (`.env.example` por módulo).
2. Consolidar fluxo de troubleshooting e disponibilizar para o time de suporte.
3. Adicionar testes de integração automatizados entre os módulos.

### Itens fora de escopo
- Migração de stack (substituição de Node.js, React ou PostgreSQL).
- Suporte a pipelines de terceiros (Jenkins, GitHub Actions) como substituto do `build-orchestrator`.
- Funcionalidades de gerenciamento de repositórios (fora do escopo do produto).

---

## Anexos e Referências
- Contexto técnico do produto: [`src/contexto-produto.md`](../src/contexto-produto.md)
- Template de referência: [`docs/readme-template.md`](readme-template.md)
- Instruções da atividade: [`docs/README-INSTRUCOES.md`](README-INSTRUCOES.md)
- Documentação do Node.js: https://nodejs.org/docs
- Documentação do RabbitMQ: https://www.rabbitmq.com/documentation.html
- Documentação do PostgreSQL: https://www.postgresql.org/docs/

## Checklist de Qualidade (pré-entrega)
- [x] Objetivo, escopo e público-alvo claros.
- [x] Pré-requisitos e setup descritos de forma executável.
- [x] Fluxo de uso e comandos essenciais documentados.
- [x] Troubleshooting com sintomas e ações recomendadas.
- [x] Premissas, lacunas e riscos preenchidos.
