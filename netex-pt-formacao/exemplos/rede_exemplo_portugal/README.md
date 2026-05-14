# Rede de Exemplo — Operadora Exemplo (Linha 200)

Dataset de referência para todos os exemplos XML ao longo do curso.

> **Decisão (Task 1.3 — Abril 2026):** Este curso usa dados reais da **Operadora Exemplo (Sociedade de Transportes Colectivos do Porto)** como rede âncora, em vez de uma rede fictícia. Justificação: o feed GTFS oficial da Operadora Exemplo está disponível e é atual (2026-04-30 → 2027-04-29), os dados são relevantes para o público-alvo português, e usar dados reais aumenta a credibilidade pedagógica dos exemplos.

**Fonte:** [DATA-03] Operadora Exemplo. "Feed GTFS Operadora Exemplo (versão Escolar 228, 2026-04-30)." Feed oficial Operadora Exemplo.

---

## Linha Âncora: Linha 200 — Bolhão → Castelo do Queijo

A **Linha 200** é usada como fio condutor de todos os módulos do curso porque:

- É uma das linhas mais emblemáticas do Porto (percorre a cidade de Este para Oeste)
- Liga o Mercado do Bolhão (centro histórico) ao Castelo do Queijo (junto ao mar)
- Tem uma topologia simples (1 linha, 2 direções) adequada para aprendizagem
- Tem dados de acessibilidade (`wheelchair_accessible=1`) em todas as viagens

### Dados da Linha 200

| Campo GTFS | Valor |
|------------|-------|
| `agency_id` | Operadora Exemplo |
| `route_id` | 200 |
| `route_short_name` | 200 |
| `route_long_name` | Bolhão - Cast.queijo |
| `route_type` | 3 (autocarro) |
| `route_color` | #187EC2 (azul Operadora Exemplo) |

### Paragens Principais (direção → Castelo do Queijo)

| Sequência | `stop_id` | Nome | Lat | Lon | Zona |
|-----------|-----------|------|-----|-----|------|
| 1 | BLRB1 | Bolhão | 41.151868 | -8.607115 | PRT1 |
| 6 | CMO | Carmo | 41.147223 | -8.616926 | PRT1 |
| 9 | PRG4 | Praça da Galiza | 41.152531 | -8.627041 | PRT1 |
| 23 | MFZ1 | Mercado da Foz | 41.155048 | -8.673343 | PRT2 |
| 30 | CQ10 | Castelo do Queijo | 41.167422 | -8.689127 | PRT2 |

### Calendário de Serviço

| `service_id` | Descrição |
|--------------|-----------|
| `DIAS UTEIS` | Segunda a sexta (exceto feriados) |
| `SÁBADOS` | Sábados |
| `DOMINGOS\|FERIADOS` | Domingos e feriados nacionais |

### Dimensão do Dataset Extraído

| Ficheiro | Registos |
|----------|----------|
| `gtfs/agency.txt` | 1 agência (Operadora Exemplo) |
| `gtfs/routes.txt` | 1 rota (linha 200) |
| `gtfs/stops.txt` | 59 paragens |
| `gtfs/trips.txt` | 341 viagens |
| `gtfs/stop_times.txt` | ~10.000 tempos de passagem |
| `gtfs/calendar_dates.txt` | Calendário completo (Abr 2026 – Abr 2027) |

---

## Como os Dados São Usados nos Módulos

| Módulo | Dados Usados |
|--------|-------------|
| Módulo 0 | Apresentação do feed (ficheiros, estrutura) |
| Módulo 1 | Estrutura geral de um ficheiro NeTEx para a linha 200 |
| Módulo 2 | Conversão de `routes.txt` + `stops.txt` → `Line`, `Route`, `ScheduledStopPoint` |
| Módulo 3 | Conversão de `trips.txt` + `stop_times.txt` + `calendar_dates.txt` → `ServiceJourney`, `DayType` |
| Módulo 4 | Enriquecimento das paragens com `StopPlace`, `Quay` (não disponível no GTFS) |
| Módulo 5 | Dados de acessibilidade (`wheelchair_accessible`) → `AccessibilityAssessment` |

---

## Nota sobre Direitos

Os dados GTFS da Operadora Exemplo são disponibilizados pela operadora. Todos os exemplos XML derivados destes dados são criados para fins educativos, com as devidas atribuições à fonte original.
