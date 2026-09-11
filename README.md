# ngo-service

Cadastro e gestão de ONGs parceiras da SolidaryTech.

- Linguagem: Python 3.11 / Flask
- Banco: PostgreSQL (`ngo_db`)
- Porta: 8081

## Endpoints

| Método | Rota | Descrição |
|---|---|---|
| GET | `/health` | Health check |
| POST | `/ngos` | Cadastra uma ONG (`name`, `email`, `cause`, `city`) |
| GET | `/ngos` | Lista ONGs |

## Variáveis de ambiente

| Variável | Obrigatória | Descrição |
|---|---|---|
| `PORT` | não (default 8081) | Porta HTTP |
| `DATABASE_URL` | sim | Connection string do PostgreSQL |
| `OTEL_*` | não | Configuração padrão do OpenTelemetry SDK (ver `fiap-tc-5-gitops/apps/ngo`) |

## Observabilidade

Auto-instrumentado via `opentelemetry-instrument` (ver `Dockerfile`) +
métricas customizadas `solidarytech_http_requests_total` e
`solidarytech_http_request_duration_seconds` (mesmo padrão dos demais
serviços — alimentam o dashboard Overview e o dashboard SRE).

## Nota de correção

O `requirements.txt` original não fixava a versão do `Werkzeug`. Com
`Flask==2.2.2` e um `Werkzeug` mais recente instalado, a aplicação nem
sobe (`ImportError: cannot import name 'url_quote' from 'werkzeug.urls'`
— a função foi removida em versões novas do Werkzeug). Fixado em
`Werkzeug==2.2.3` (mesma versão já usada nos serviços da Fase 3).

## CI/CD

`.github/workflows/build-push.yaml`: Lint (flake8) → Test → SonarQube
(SAST) → Trivy (SCA, ignora CVEs sem fix disponível) → Build/Push no ACR →
atualiza o GitOps (`fiap-tc-5-gitops`).

`.github/workflows/self-heal.yml`: disparado pelo Datadog via
`repository_dispatch` (se `ngo-service` estiver em `monitored_services` no
Terraform) — executa `kubectl rollout restart`.

### Secrets necessários no GitHub

| Secret | Uso |
|---|---|
| `ACR_USERNAME` / `ACR_PASSWORD` | push da imagem |
| `GITOPS_TOKEN` | commit no repo `fiap-tc-5-gitops` |
| `SONAR_TOKEN` / `SONAR_HOST_URL` | SonarQube |
| `AZURE_CREDENTIALS` / `AKS_RESOURCE_GROUP` / `AKS_CLUSTER_NAME` | self-healing |
