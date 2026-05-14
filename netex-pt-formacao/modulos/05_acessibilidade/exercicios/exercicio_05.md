# Exercício 5 — Auditar a Acessibilidade da Linha 200

> **Módulo 5 — Acessibilidade**

---

## Contexto

A Operadora Exemplo está a preparar a submissão de dados de acessibilidade ao NAP Portugal, conforme exigido pelo Regulamento (UE) 2024/490. O ponto de partida é o feed GTFS atual da Linha 200, que contém os campos `wheelchair_accessible` (em `trips.txt`) e `wheelchair_boarding` (em `stops.txt`).

### Dados GTFS disponíveis

**stops.txt** (extrato das paragens do início da Linha 200):

| stop_id | stop_name | stop_lat | stop_lon | wheelchair_boarding |
|---------|-----------|----------|----------|---------------------|
| BLRB1 | Bolhão | 41.151868 | -8.607115 | 1 |
| CMO | Carmo | 41.146210 | -8.614560 | 1 |
| PRG4 | Praça da Galiza | 41.156670 | -8.629470 | 0 |
| MFZ1 | Mercado da Foz | 41.157610 | -8.636440 | 2 |
| CQ10 | Castelo do Queijo | 41.168930 | -8.676240 | 1 |

**trips.txt** (extrato — `wheelchair_accessible`):

| trip_id | service_id | wheelchair_accessible |
|---------|------------|----------------------|
| 200_DU_0600 | DU | 1 |
| 200_DU_0740 | DU | 1 |
| 200_SAB_0604 | SAB | 0 |
| 200_DF_0604 | DF | 2 |

---

## Parte A — Converter wheelchair_boarding para NeTEx

Para cada paragem da tabela acima, crie um `Quay` (dentro de um `StopPlace`) com o `AccessibilityAssessment` correto.

**Regras de mapeamento:**
- `wheelchair_boarding=1` → `WheelchairAccess=yes`, `MobilityImpairedAccess=yes`
- `wheelchair_boarding=2` → `WheelchairAccess=no`, `MobilityImpairedAccess=no`
- `wheelchair_boarding=0` → `WheelchairAccess=unknown`, `MobilityImpairedAccess=unknown`

Para os campos `StepFreeAccess`, `AudibleSignalsAvailable` e `VisualSignsAvailable`, use `unknown` quando a informação não estiver disponível no GTFS.

---

## Parte B — Converter wheelchair_accessible para NeTEx

Para cada viagem da tabela acima, adicione um `AccessibilityAssessment` na `ServiceJourney`.

**Regras de mapeamento:**
- `wheelchair_accessible=1` → `WheelchairAccess=yes`, `MobilityImpairedAccess=yes`
- `wheelchair_accessible=2` → `WheelchairAccess=no`, `MobilityImpairedAccess=no`
- `wheelchair_accessible=0` → `WheelchairAccess=unknown`, `MobilityImpairedAccess=unknown`

As `ServiceJourney` devem referenciar o `JourneyPatternRef` e `DayTypeRef` já definidos nos Módulos 2 e 3.

---

## Parte C — Cenário de Auditoria

A Operadora Exemplo detetou que a paragem MFZ1 (Marechal Fontaine) foi renovada em março de 2026: o lancil foi rebaixado, foi instalado piso tátil e um botão sonoro.

Modele esta alteração usando o mecanismo de versionamento do NeTEx:
1. O `Quay` original (version="1") com `WheelchairAccess=no` válido até `2026-03-01`
2. O `Quay` renovado (version="2") com `WheelchairAccess=yes`, `AudibleSignalsAvailable=yes`, `VisualSignsAvailable=partial` a partir de `2026-03-01`

---

## Questões de Reflexão

**1. O problema do "parcial"**
A paragem PRG4 (Praça da República) tem `wheelchair_boarding=0` (desconhecido). Após visita de campo, descobre-se que existe um lancil rebaixado mas o passeio tem um poste de eletricidade a bloquear parte da passagem — uma cadeira de rodas larga não consegue passar, mas uma cadeira estreita consegue.

Como representaria esta situação em NeTEx? Qual o valor mais correto para `WheelchairAccess`? Justifique.

**2. Acessibilidade da cadeia completa**
A viagem `200_DF_0604` tem `wheelchair_accessible=2` (veículo não acessível). No entanto, a paragem inicial (BLRB1) tem `wheelchair_boarding=1` (paragem acessível).

Como é que um sistema de pesquisa de percursos (tipo Moovit ou Google Maps) deve interpretar esta combinação? Deveria mostrar este serviço a um utilizador que filtrou por "serviços acessíveis"?

**3. Manutenção de dados ao longo do tempo**
O elevador da estação de metro de Campanhã (modelada no Módulo 4) avariou temporariamente de 2026-05-01 a 2026-05-15. Como representaria esta situação em NeTEx sem apagar a informação de acessibilidade original?

---

## Autoavaliação

- [ ] O mapeamento `wheelchair_boarding=0` usa `unknown` (não `no`)?
- [ ] O mapeamento `wheelchair_boarding=2` usa `no` (não `unknown`)?
- [ ] Cada `AccessibilityAssessment` tem `MobilityImpairedAccess` como campo geral?
- [ ] Os campos `StepFreeAccess`, `AudibleSignalsAvailable`, `VisualSignsAvailable` são `unknown` quando não há informação GTFS?
- [ ] A Parte C usa `validBetween` com `FromDate`/`ToDate` para separar as versões do Quay?
- [ ] As `ServiceJourney` da Parte B têm `DayTypeRef` e `JourneyPatternRef` corretos?

---

## Solução

Consulte [`solucoes/exercicio_05_solucao.xml`](solucoes/exercicio_05_solucao.xml)
