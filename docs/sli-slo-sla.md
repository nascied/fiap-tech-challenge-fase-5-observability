# SLI, SLO e SLA — 3 microsserviços

> Requisito do Hackathon Fase 5: SLIs baseados em Golden Metrics + SLO por indicador (seção *SRE*, que cita nominalmente o `donation-service`) e definição formal de SLI, SLO **e SLA** (seção *Entregáveis*, Relatório de Entrega). Estendido aqui pra `ngo-service` e `volunteer-service` também, com metas mais frouxas — o enunciado só exige formalmente pro `donation-service`, mas a mesma estrutura de indicadores se aplica igual aos outros dois.

As metas variam por **Tier** de criticidade, mesma classificação do [DRP](../../fiap-tech-challenge-fase-5-infra/docs/drp/DRP.md#3-classificação-de-criticidade):

| Serviço | Tier | Por quê |
|---|---|---|
| `donation-service` | **0 — Crítico** | Hot Path — processa a doação em si (dado explicitamente citado no enunciado do hackathon) |
| `ngo-service` | 1 — Essencial | Cadastro de ONG não bloqueia uma doação em andamento |
| `volunteer-service` | 1 — Essencial | Cadastro de voluntário não bloqueia uma doação em andamento |

## SLI 1 — Latência

**Definição**: proporção de requisições HTTP que completam abaixo de um limite de tempo.

```promql
sum(rate(http_server_request_duration_seconds_bucket{service_name="<serviço>", le="<threshold>"}[1h]))
/
sum(rate(http_server_request_duration_seconds_count{service_name="<serviço>"}[1h]))
```

`le` precisa ser um limite de bucket real do histograma `http.server.request.duration` (convenção semântica OTel) — `0.25` e `0.5` são limites padrão, não exigem configuração extra no Collector.

## SLI 2 — Disponibilidade (taxa de erro)

**Definição**: proporção de requisições que **não** retornam erro de servidor (HTTP 5xx).

```promql
sum(rate(http_server_request_duration_seconds_count{service_name="<serviço>", http_response_status_code!~"5.."}[1h]))
/
sum(rate(http_server_request_duration_seconds_count{service_name="<serviço>"}[1h]))
```

## SLO e SLA por serviço

| Serviço | Tier | SLO Latência | SLO Disponibilidade | SLA (compromisso externo às ONGs) |
|---|---|---|---|---|
| `donation-service` | 0 | 95% das requisições < **250ms** | **99,9%** sem 5xx | 99,5% disponibilidade mensal / 99% requisições < 1s |
| `ngo-service` | 1 | 90% das requisições < **500ms** | **99,5%** sem 5xx | 99% disponibilidade mensal / 95% requisições < 1,5s |
| `volunteer-service` | 1 | 90% das requisições < **500ms** | **99,5%** sem 5xx | 99% disponibilidade mensal / 95% requisições < 1,5s |

O SLA é sempre um pouco mais frouxo que o SLO interno — é o compromisso externo, com consequência contratual/reputacional; o SLO existe justamente pra dar margem de manobra antes de violar o SLA de verdade. Violação do SLA aciona a resposta formal de incidente do ciclo de vida de ITSM do projeto (documentado à parte, fora deste repositório) — sem penalidade financeira automatizada (projeto educacional), mas a estrutura é a mesma de uma relação real fornecedor↔cliente.

**Por que `donation-service` tem meta mais rígida**: é o único Tier 0 — indisponibilidade ali significa doação perdida na hora H, não só um cadastro que pode ser refeito depois.

## Error Budget

O Error Budget é o "quanto de falha ainda é aceitável" antes de violar o SLO — é o que os painéis "Error Budget restante (30d)" do [dashboard SRE](../dashboards/solidary-tech-sre.json) mostram em tempo real, um por serviço.

**Fórmula geral**:

```
taxa_de_falha_permitida = 1 - SLO
orçamento_restante (%) = 1 - (taxa_de_falha_real / taxa_de_falha_permitida)
```

| Serviço | Orçamento de falha de latência | Orçamento de falha de disponibilidade |
|---|---|---|
| `donation-service` | 1 − 0,95 = **5%** | 1 − 0,999 = **0,1%** |
| `ngo-service` / `volunteer-service` | 1 − 0,90 = **10%** | 1 − 0,995 = **0,5%** |

Quando o orçamento de qualquer serviço chega a 0%, a política padrão (Google SRE) é **congelar mudanças não-críticas** nesse serviço até o orçamento se recompor — nenhuma automação de bloqueio de deploy foi implementada ainda (ficaria no pipeline de CI/CD, outra pendência do projeto, ver `CLAUDE.md` na raiz do monorepo).

## Painel

Todos os valores acima (por serviço) estão no dashboard Grafana **"Solidary Tech - SRE"** (UID `solidary-tech-sre`, `dashboards/solidary-tech-sre.json`) — uma seção completa (Latência/Disponibilidade/Golden Metrics) por serviço, painel **exclusivo** de confiabilidade, sem métricas genéricas de infraestrutura (essas ficam no dashboard "Solidary Tech - Infraestrutura do Cluster").

## O que falta pra esses números serem reais

Este documento define os **alvos**; os números que o dashboard mostra só passam a refletir a realidade depois que:
1. O `otel-collector` estiver sincronizado no cluster (`fiap-tech-challenge-fase-5-observability`, ainda não aplicado — ver `README.md`).
2. Os 3 microsserviços estiverem com `OTEL_METRICS_EXPORTER=otlp` habilitado (hoje `none` por padrão em todos — ver [`golden-metrics.md`](./golden-metrics.md)).
3. Houver tráfego real/simulado passando pelos 3 serviços por tempo suficiente pra preencher as janelas de 1h/30d das queries.
