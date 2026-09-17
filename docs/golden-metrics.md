# Golden Metrics — os 3 microsserviços

> Requisito do Hackathon Fase 5 (seção *SRE: Confiabilidade e Golden Metrics*): *"defina e documente, no mínimo, dois SLIs baseados nas Golden Metrics (ex: Latência e Taxa de Erros)"*. O enunciado cita nominalmente só o `donation-service`; aqui as 4 métricas são aplicadas aos 3 serviços, cada um com sua seção no [dashboard SRE](../dashboards/solidary-tech-sre.json).

As **4 métricas de ouro** (Google SRE Book) usadas para avaliar a saúde de qualquer serviço voltado a requisição/resposta.

| Golden Metric | O que mede | Fonte (Prometheus) | Painel no dashboard SRE |
|---|---|---|---|
| **Latência** | Tempo de resposta das requisições HTTP | `http_server_request_duration_seconds_bucket{service_name="<serviço>"}` (histograma, via OTel Collector) | "Latência (p50/p95/p99)" |
| **Tráfego** | Volume de requisições por segundo | `http_server_request_duration_seconds_count{service_name="<serviço>"}` | "Tráfego (requisições/s)" |
| **Erros** | Taxa de requisições que falham (HTTP 4xx/5xx) | `http_server_request_duration_seconds_count{service_name="<serviço>", http_response_status_code=~"4..\|5.."}` | "Erros (rate por status)" |
| **Saturação** | Quão perto o serviço está do limite de recursos alocados | `container_cpu_usage_seconds_total` / `container_memory_working_set_bytes` vs. `kube_pod_container_resource_limits` (namespace `fiap-tc-f5`, por pod) | "Saturação — CPU/Memória (% do limit)" |

`<serviço>` = `donation-service`, `ngo-service` ou `volunteer-service` — cada um tem sua própria seção completa no dashboard, com essas 4 métricas repetidas.

## Por que só Latência e Erros viram SLI formal

As 4 métricas ficam visíveis no dashboard, mas só **Latência** e **Erros** (via disponibilidade) foram promovidas a SLI/SLO formais (ver [`sli-slo-sla.md`](./sli-slo-sla.md)) — é a orientação padrão de SRE (Google SRE Workbook): nem toda métrica de ouro precisa virar um compromisso formal, só as que refletem diretamente a experiência do usuário final (doador, ONG, voluntário). Tráfego e Saturação continuam monitoradas porque explicam *por quê* a latência ou a taxa de erro se degradou (ex.: saturação de CPU alta correlacionada com aumento de latência), mas não são, sozinhas, o que a SolidaryTech promete.

## Origem técnica da métrica

`http_server_request_duration_seconds_*` é gerada pelo `otelhttp` (Go, `donation-service`) e pela auto-instrumentação Flask (`ngo-service`/`volunteer-service`), exportada via OTLP pro `otel-collector` ([`fiap-tech-challenge-fase-5-observability/values/otel-collector.yaml`](../values/otel-collector.yaml)), que expõe pro Prometheus com `resource_to_telemetry_conversion.enabled: true` — é isso que torna `service.name` disponível como label `service_name` nas queries acima.

**Pré-requisito**: os 3 microsserviços precisam estar com `OTEL_METRICS_EXPORTER=otlp` (hoje `none` por padrão — ver `config.env` de cada serviço no repositório [`fiap-tech-challenge-fase-5-services`](https://github.com/nascied/fiap-tech-challenge-fase-5-services)) e `OTEL_EXPORTER_OTLP_ENDPOINT` apontando pro `otel-collector`. Sem isso, todos os painéis acima ficam vazios — não é bug do dashboard.
