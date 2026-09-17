# Post-Mortem de Incidente — Template

> Parte do ciclo de vida de incidente descrito em [`itsm-aiops.md`](./itsm-aiops.md). Copie este arquivo para `incidents/AAAA-MM-DD-<service>.md` a cada incidente com `severity=critical` (ou `warning` que exigiu intervenção manual).

Blameless: o objetivo é entender o que o **processo/sistema** permitiu que desse errado, não quem executou o quê. Nomes de pessoas não entram neste documento — só papéis (ver [DRP, seção 8](../../fiap-tech-challenge-fase-5-infra/docs/drp/DRP.md#8-papéis-e-responsabilidades)).

## Metadados

| Campo | Valor |
|---|---|
| Serviço afetado | `donation-service` / `ngo-service` / `volunteer-service` |
| Tier | 0 / 1 |
| Severidade do alerta | `critical` / `warning` |
| Alerta(s) que dispararam | ex.: `SLOErrorBudgetBurnFast` |
| Impacto em doações | Sim/Não — se sim, quantas/qual período |

## Linha do tempo

| Horário (UTC) | Evento |
|---|---|
| T0 | Início real do problema (se souber, muitas vezes só descoberto depois) |
| | Alerta disparado (Alertmanager/Watchdog) |
| | Triagem iniciada |
| | Causa identificada |
| | Mitigação aplicada |
| | Serviço confirmado normal (SLI voltou ao patamar do SLO) |
| | Incidente encerrado |

## Causa raiz

Use os "5 porquês" ou o método que fizer sentido — o objetivo é chegar à causa sistêmica, não só ao sintoma imediato.

## O que funcionou bem

- 

## O que não funcionou / atrasou a resposta

- 

## Ações de melhoria

| Ação | Dono (papel) | Prazo | Status |
|---|---|---|---|
| | | | |

## Isso deveria ter sido pego antes?

- [ ] Não havia como prever
- [ ] Sim — falta de teste/alerta específico (qual?)
- [ ] Sim — runbook existia mas não foi seguido (por quê?)

## Comunicação

- [ ] Equipe técnica notificada (automático via Slack)
- [ ] Resumo executivo enviado à diretoria/stakeholders (obrigatório se Tier 0)
- [ ] ONGs/doadores avisados, se aplicável
