# 🚀 Solidary Tech — Observabilidade (Hackathon Fase 5)

[![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?style=for-the-badge&logo=prometheus&logoColor=white)](https://prometheus.io/)
[![Grafana](https://img.shields.io/badge/Grafana-F46800?style=for-the-badge&logo=grafana&logoColor=white)](https://grafana.com/)
[![Loki](https://img.shields.io/badge/Loki-F5A623?style=for-the-badge&logo=grafana&logoColor=white)](https://grafana.com/oss/loki/)
[![OpenTelemetry](https://img.shields.io/badge/OpenTelemetry-425CC7?style=for-the-badge&logo=opentelemetry&logoColor=white)](https://opentelemetry.io/)
[![ArgoCD](https://img.shields.io/badge/ArgoCD-EF7B4D?style=for-the-badge&logo=argo&logoColor=white)](https://argo-cd.readthedocs.io/)

> ⚠️ **PROJETO DIDÁTICO** - Este projeto foi desenvolvido como parte do Hackathon Fase 5 da Pós-Tech FIAP em Arquitetura Cloud e DevOps.

## 📋 Sobre o Projeto

Este projeto representa o escopo de **infraestrutura de observabilidade** da SolidaryTech, implementando a stack Opensource de monitoramento (Prometheus, Loki, Grafana) e o OpenTelemetry Collector sobre o cluster Kubernetes já provisionado em [`fiap-tech-challenge-fase-5-infra`](https://github.com/nascied/fiap-tech-challenge-fase-5-infra), com entrega 100% via GitOps (ArgoCD).

## Escopo

Este repositório contém os Helm charts e as configurações de observabilidade da SolidaryTech, entregues como `Application` do ArgoCD (padrão app-of-apps). Cobre a parte de **Observabilidade e APM** do Requisito 0 (Fundação DevOps) do enunciado do Hackathon Fase 5:

- **Prometheus** — armazenamento e consulta de métricas de infraestrutura (via `kube-prometheus-stack`)
- **Loki + Promtail** — centralização e indexação de logs de todos os containers do cluster
- **Grafana** — visualização, com dois dashboards customizados e datasources provisionados
- **OpenTelemetry Collector** — peça central de roteamento de telemetria (métricas → Prometheus, logs → Loki, traces → Datadog)
- **Alertmanager + PrometheusRule** — detecção automática por burn rate de SLO (Requisito 3, ITSM/AIOps), roteada por severidade pro Slack — ver [`docs/itsm-aiops.md`](docs/itsm-aiops.md)

Este repositório também entrega a **rota de traces para o APM** (exporter Datadog configurado no Collector) — ver seção "Status Atual" para o que já foi validado. Não fazem parte deste repositório:

- Instrumentação do código-fonte dos microsserviços (repositório [`fiap-tech-challenge-fase-5-services`](https://github.com/nascied/fiap-tech-challenge-fase-5-services) — os 3 serviços já são instrumentados com OpenTelemetry; ver seção "Observabilidade" do README de lá)
- IA do APM (Watchdog/Applied Intelligence) — depende de conta Datadog real, indisponível nesta conta educacional; ver [`docs/itsm-aiops.md`](docs/itsm-aiops.md#22-ia-do-apm-datadog-watchdog--o-que-falta-pra-ativar-de-verdade)
- PagerDuty — integrado (`pagerduty_configs` no Alertmanager, alertas `critical` do `donation-service`) e dispara self-healing via Claude (ver [`docs/itsm-aiops.md`](docs/itsm-aiops.md#31-resposta-automatizada--self-healing-com-claude), código no repo `fiap-tech-challenge-fase-5-infra/aiops/`). OpsGenie e ChatOps não implementados (Slack via Alertmanager cobre a notificação humana, ver abaixo)
- Cluster EKS, Terraform e os 3 microsserviços em si ([`fiap-tech-challenge-fase-5-infra`](https://github.com/nascied/fiap-tech-challenge-fase-5-infra) e [`fiap-tech-challenge-fase-5-services`](https://github.com/nascied/fiap-tech-challenge-fase-5-services) — pré-requisito, não escopo desta fase)

## 🛠️ Tecnologias Utilizadas

| Tecnologia | Uso |
|---|---|
| **Prometheus** | Armazenamento e consulta de métricas |
| **Grafana** | Visualização de métricas e logs |
| **Loki** | Centralização e indexação de logs |
| **Promtail** | Agente de coleta de logs dos containers |
| **OpenTelemetry Collector** | Recebimento, processamento e roteamento de telemetria |
| **ArgoCD** | Entrega contínua via GitOps |
| **Helm** | Empacotamento e parametrização dos charts |

## Componentes Provisionados

| Application (ArgoCD) | Chart | Responsabilidade |
|---|---|---|
| `kube-prometheus-stack` | `prometheus-community/kube-prometheus-stack` | Prometheus, Grafana, Alertmanager, node-exporter, kube-state-metrics |
| `loki` | `grafana/loki` | Armazenamento e indexação de logs (modo `SingleBinary`, storage em filesystem) |
| `promtail` | `grafana/promtail` | Coleta de logs de todos os pods do cluster |
| `otel-collector` | `open-telemetry/opentelemetry-collector` | Receiver OTLP + exporters para Prometheus/Loki/Datadog |
| `monitoring-dashboards` | manifesto próprio | ConfigMaps dos dashboards customizados do Grafana |
| `grafana-ingress` | manifesto próprio | Exposição do Grafana via `ingress-nginx` |
| `alert-rules` | manifesto próprio | `PrometheusRule` de burn rate de SLO (detecção — ITSM/AIOps) |

## 📁 Estrutura do Projeto

```text
├── argocd/
│   ├── root.yaml               ← Application raiz, único arquivo aplicado manualmente
│   └── application/             ← Applications reais (chart público + values deste repo)
│       ├── application-kube-prometheus-stack.yaml
│       ├── application-loki.yaml
│       ├── application-promtail.yaml
│       ├── application-otel-collector.yaml
│       ├── application-dashboards.yaml
│       ├── application-grafana-ingress.yaml
│       ├── application-alert-rules.yaml
│       └── kustomization.yaml
├── values/
│   ├── kube-prometheus-stack.yaml    ← inclui roteamento do Alertmanager (severity → Slack)
│   ├── loki.yaml
│   ├── promtail.yaml
│   └── otel-collector.yaml      ← inclui exporter Datadog (traces) via DD_API_KEY
├── dashboards/
│   ├── solidary-tech-overview.json      ← saúde dos 3 microsserviços
│   ├── solidary-tech-infra.json         ← saúde/capacidade do cluster (nodes)
│   ├── solidary-tech-sre.json            ← EXCLUSIVO: SLI/SLO/Error Budget + Golden Metrics (3 serviços, SLO por Tier)
│   └── kustomization.yaml
├── manifests/
│   └── grafana-ingress.yaml
├── alerting/
│   ├── prometheusrule-slo-burn.yaml   ← 9 regras de burn rate (SLO de latência/disponibilidade, 3 serviços)
│   └── kustomization.yaml
├── docs/
│   ├── golden-metrics.md      ← as 4 golden metrics aplicadas ao donation-service
│   ├── sli-slo-sla.md         ← definição formal de SLI, SLO e SLA + Error Budget
│   ├── mttr.md                ← como a stack ajuda a reduzir o MTTR (+ o que falta)
│   ├── itsm-aiops.md           ← ciclo de vida do incidente (detecção → resposta → post-mortem → comunicação)
│   └── postmortem-template.md ← template blameless de post-mortem
└── README.md
```

## ✅ Status Atual

| Componente | Situação | Observação |
|---|---|---|
| `kube-prometheus-stack`, `loki`+`promtail`, `otel-collector`, dashboards | 🟡 **Pendente de primeira aplicação no cluster** | Charts/values/dashboards adaptados pro Solidary Tech e validados localmente (`kubectl kustomize`, JSON válido) — nunca sincronizados contra um cluster EKS real ainda |
| `alert-rules` (PrometheusRule + roteamento Alertmanager) | 🟡 **Pendente de primeira aplicação** + Secrets do Slack/PagerDuty não criados no cluster | As 9 regras de burn rate validadas com `promtool check rules` (via `docker run prom/prometheus`) e o roteamento (Slack + PagerDuty) validado com `amtool check-config` (via `docker run prom/alertmanager`) — sintaxe confirmada de verdade, nunca sincronizado contra um cluster real ainda. Ver [`docs/itsm-aiops.md`](docs/itsm-aiops.md#6-o-que-falta-pra-isso-ser-real-honestidade) |

> Este repositório foi adaptado a partir de uma versão anterior (Fase 4, projeto "ToggleMaster") que chegou a ser validada de ponta a ponta num cluster real, incluindo traces chegando no Datadog com Service Map populado. A stack em si (charts, exporters, pipeline do Collector) é a mesma, comprovadamente funcional — só os dashboards e os `repoURL` foram adaptados pro Solidary Tech. Atualize esta tabela depois da primeira sincronização real.

### Gotchas conhecidos (herdados da versão anterior — aplicar/confirmar nesta stack)

- **Label `service_name` ausente**: o exporter Prometheus do OTel Collector mapeia `service.name` para o label `job` por padrão — que colide com o `job` do `ServiceMonitor` e é renomeado para `exported_job`. Já corrigido em `values/otel-collector.yaml` com `resource_to_telemetry_conversion.enabled: true`.
- **✅ Corrigido: `donation-service` (Go) não emitia métrica nenhuma.** O `main.go` só tinha `TracerProvider` (traces), nunca teve `MeterProvider` — o `otelhttp` grava o histograma `http.server.request.duration` automaticamente, mas só se um `MeterProvider` estiver registrado. Sem isso, **todos** os painéis de "Requisições"/SLO do `donation-service` ficariam vazios, mesmo com tudo mais configurado certo. Corrigido com `initMetrics()` em `main.go` (repositório `services`). Os serviços Python também estavam com `OTEL_METRICS_EXPORTER=none` fixo (herdado de quando o alvo local era o Jaeger, que não aceita métricas) — trocado pro mesmo toggle usado pra traces nos 3 `config.env`.
- **⚠️ Nome de métrica pode divergir entre `donation-service` (Go) e `ngo-service`/`volunteer-service` (Python) — ainda NÃO confirmado nesta stack**: no projeto anterior, a versão do `otelhttp` (Go) em uso só emitia a convenção semântica **nova** (`http.server.request.duration`), enquanto o `opentelemetry-instrumentation-flask` (Python) emitia por padrão a **antiga** (`http.server.duration`). Os dashboards deste repositório (`solidary-tech-overview.json`, `solidary-tech-sre.json`) já assumem a convenção **nova** (`http_server_request_duration_seconds_count`, label `http_response_status_code`). Antes de confiar nos painéis de "Requisições"/SLO, confirmar/aplicar `OTEL_SEMCONV_STABILITY_OPT_IN=http` nos `config.env`/`values.yaml` dos 3 serviços (repositório `services`/`gitops`) — se não estiver setado, os painéis desse serviço específico ficam vazios, não é erro do dashboard.
- **Label `namespace` inesperado**: nas métricas HTTP do OTel, o label `namespace` reflete onde o Prometheus faz o *scrape* do Collector (`monitoring`), não o namespace da aplicação (`fiap-tc-f5`) — não usar esse filtro nessas queries específicas; `service_name` já é suficiente.

## Requisitos

- Cluster Kubernetes (EKS) e ArgoCD já em execução ([`fiap-tech-challenge-fase-5-infra`](https://github.com/nascied/fiap-tech-challenge-fase-5-infra)).
- `kubectl` configurado apontando para o cluster.
- Os 3 microsserviços da SolidaryTech operacionais ([`fiap-tech-challenge-fase-5-services`](https://github.com/nascied/fiap-tech-challenge-fase-5-services) + [`fiap-tech-challenge-fase-5-gitops`](https://github.com/nascied/fiap-tech-challenge-fase-5-gitops)).
- Secret `datadog-secret` criado no namespace `monitoring` (ver seção abaixo) — obrigatório antes de sincronizar o `otel-collector`, senão o pod falha ao subir por variável de ambiente sem Secret correspondente.

Configure o `kubectl` antes da execução:

```bash
aws eks update-kubeconfig --name <nome-do-cluster> --region us-east-1
```

### Pré-requisito: Secret do Datadog

O `otel-collector` referencia `DD_API_KEY` via `secretKeyRef` (`values/otel-collector.yaml`). Esse Secret **não é versionado neste repositório** (credencial sensível) e precisa ser criado manualmente antes do ArgoCD sincronizar a Application `otel-collector`:

```bash
kubectl create namespace monitoring --dry-run=client -o yaml | kubectl apply -f -
kubectl create secret generic datadog-secret -n monitoring --from-literal=apiKey='<SUA_DD_API_KEY>'
```

Gere a chave em `https://app.datadoghq.com/organization-settings/api-keys` (ajuste o domínio se sua conta não for na região US1).

## Values por Chart

Cada `Application` usa fonte múltipla (multi-source do ArgoCD): o chart público do respectivo projeto + o `values.yaml` correspondente deste repositório.

| Arquivo | Chart afetado | Principais ajustes |
|---|---|---|
| `values/kube-prometheus-stack.yaml` | `kube-prometheus-stack` | retenção do Prometheus, descoberta de `ServiceMonitor` em qualquer namespace, sidecar de dashboards, datasource do Loki |
| `values/loki.yaml` | `loki` | modo `SingleBinary`, storage em filesystem, sem autenticação multi-tenant |
| `values/promtail.yaml` | `promtail` | endpoint de push apontando para o Service do Loki |
| `values/otel-collector.yaml` | `opentelemetry-collector` | receiver OTLP; exporter de métricas (Prometheus, com `resource_to_telemetry_conversion` para expor `service_name` como label) e logs (Loki); traces roteados para o Datadog via `DD_API_KEY` (Secret externo) |

## Como Executar

1. Configure o `kubectl` apontando para o cluster (ver comando acima).
2. Crie o Secret `datadog-secret` (ver pré-requisito acima) — sem ele, a Application `otel-collector` fica `Degraded`.
3. Acesse o diretório do repositório:

   ```bash
   cd fiap-tech-challenge-fase-5-observability
   ```

4. Aplique a Application raiz (único comando necessário — todo o resto sobe via GitOps):

   ```bash
   kubectl apply -f argocd/root.yaml
   ```

5. Acompanhe a sincronização até todas ficarem `Synced`/`Healthy`:

   ```bash
   kubectl get applications -n argocd -w
   ```

6. Valide os componentes:

   ```bash
   kubectl get pods -n monitoring
   ```

7. Acesse o Grafana (ver seção "Acesso" abaixo) e confira os dois dashboards.

## Sincronização via ArgoCD (GitOps)

A `Application` raiz (`monitoring-root`) sincroniza a pasta `argocd/application/`, que cria as 6 `Application` reais. `kube-prometheus-stack` e `loki` sobem primeiro (`sync-wave: "-1"`); `promtail`, `otel-collector`, `monitoring-dashboards` e `grafana-ingress` sobem na sequência (`sync-wave: "0"`), já que dependem do Prometheus/Loki estarem disponíveis.

| Evento | Ação | Objetivo |
|---|---|---|
| Push no branch `main` | ArgoCD detecta divergência (polling automático) | Atualizar o cluster sem intervenção manual |
| Alteração manual no cluster | `selfHeal` reverte para o estado do Git | Garantir que o Git continue sendo a única fonte da verdade |

## Acesso

- **Grafana**: `http://<domínio>/grafana` (usuário `admin`, senha em `values/kube-prometheus-stack.yaml`)
- **Dashboard "Solidary Tech - Visão Geral"**: recursos de CPU/memória por pod, taxa de requisições por microsserviço (via métrica `http_server_request_duration_seconds_count`, OTel Collector) e painel de logs em tempo real (Loki), com uma seção detalhada por microsserviço (`ngo-service`, `donation-service`, `volunteer-service`)
- **Dashboard "Solidary Tech - Infraestrutura do Cluster"**: nodes ready, pods pendentes/com restart, CPU/memória/disco/rede por node, capacidade (requests vs. alocável)
- **Dashboard "Solidary Tech - SRE"**: painel **exclusivo** de SLI/SLO/Error Budget + as 4 Golden Metrics dos 3 microsserviços — sem métricas genéricas de infra. Metas diferenciadas por Tier: `donation-service` (Tier 0/Hot Path) com SLO mais rígido (95% < 250ms, 99,9% sem 5xx); `ngo-service`/`volunteer-service` (Tier 1) com SLO mais frouxo (90% < 500ms, 99,5% sem 5xx). Definições formais em [`docs/sli-slo-sla.md`](docs/sli-slo-sla.md) e [`docs/golden-metrics.md`](docs/golden-metrics.md); como isso reduz o MTTR em [`docs/mttr.md`](docs/mttr.md).
- **Datadog (APM)**: `https://app.datadoghq.com` (ajustar domínio se a conta não for região US1)
  - **APM → Service Map**: mapa de dependências entre os 3 microsserviços, descoberto automaticamente pelos traces
  - **APM → Traces**, filtro `service:donation-service`: trace do fluxo de doação — span HTTP (`donations`) com os spans filhos `db.insert_donation` (Postgres) e `sqs.publish_donation_event` (SQS) aninhados. Os 3 serviços não têm chamada HTTP direta entre si (diferente do projeto anterior) — a "distribuição" do trace aqui é entre a aplicação e seus próprios dependentes (banco, fila), não entre microsserviços.

## Arquivos Ignorados

```text
*.tgz
.helm/
```

## Boas Práticas

- Revisar o `helm template` de cada chart localmente antes de commitar alterações de `values`.
- Nunca commitar credenciais reais (ex: senha do Grafana, `DD_API_KEY`) em texto plano — usar Secret gerenciado externamente (`datadog-secret`), fora deste repositório.
- Ao adicionar/alterar instrumentação OTel em algum microsserviço, manter a convenção semântica HTTP consistente entre serviços (`OTEL_SEMCONV_STABILITY_OPT_IN=http`) — ver "Gotchas conhecidos" acima; divergência entre serviços fragmenta os painéis de requisições.
- Usar `ServerSideApply=true` em Applications que instalam CRDs grandes (evita o limite de 256KB de annotation do `kubectl apply` padrão).

## 👨‍💻 Autor

**Edson Leandro da Silva Nascimento**
- Pós-Tech FIAP - Arquitetura Cloud e DevOps
- Hackathon Fase 5 — Observabilidade

---

## 📄 Licença

Este projeto é apenas para fins educacionais como parte do programa de pós-graduação em Arquitetura Cloud e DevOps da instituição FIAP.
