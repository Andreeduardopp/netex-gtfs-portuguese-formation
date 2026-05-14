# Comparação GTFS vs NeTEx

Uma análise lado-a-lado dos dois standards para técnicos que já conhecem GTFS.

> Esta página é um recurso de consulta rápida. Para a narrativa completa, ver **Módulo 0 — Secção 2**.

---

## Tabela Comparativa Principal

| Dimensão | GTFS | NeTEx |
|----------|------|-------|
| **Formato** | CSV (ficheiros `.txt` num `.zip`) | XML (por norma um `.xml` por `Frame`) |
| **Modelo conceptual** | Pragmático, sem modelo formal | Baseado no Transmodel CEN EN 12896 [STD-04] |
| **Cobertura** | Horários, paragens, rotas, tarifas básicas | Tudo do GTFS + acessibilidade, multimodalidade, tarifas complexas, informação ao passageiro |
| **Identificadores** | Locais (sem formato obrigatório) | Globais e hierárquicos: `PT:EXEMPLO:Line:200:LOC` |
| **Versionamento** | Não existe | Nativo — cada objeto tem `version` e `validFrom/To` |
| **Complexidade** | Baixa — 6 ficheiros obrigatórios | Alta — schema XSD com ~500 tipos XML |
| **Adoção** | 10.000+ agências globalmente | Obrigatório na UE (Regulamento 2024/490) [REG-01] |
| **Extensibilidade** | Via extensões ad hoc (GTFS-Flex, GTFS-Fares v2) | Nativa — perfis nacionais definem subconjuntos |
| **Tempo real** | GTFS Realtime (Protocol Buffers) | SIRI (XML), baseado no mesmo Transmodel |
| **Regulação UE** | Aceite para dados básicos | Obrigatório para dados avançados (NeTEx) [REG-01] |

---

## Equivalências de Conceitos

| Conceito GTFS | Ficheiro | Equivalente NeTEx | Frame NeTEx | Notas |
|---------------|----------|-------------------|-------------|-------|
| Agency | `agency.txt` | `Operator` + `Authority` | `ResourceFrame` | NeTEx distingue operador (quem opera) de autoridade (quem regula) |
| Route | `routes.txt` | `Line` + `Route` + `Direction` | `ServiceFrame` | NeTEx separa a linha comercial da sua topologia física |
| Stop | `stops.txt` (coordenadas) | `ScheduledStopPoint` | `ServiceFrame` | Ponto lógico; o lugar físico é `StopPlace`/`Quay` em `SiteFrame` |
| Trip | `trips.txt` | `ServiceJourney` | `TimetableFrame` | NeTEx adiciona padrões reutilizáveis (`ServiceJourneyPattern`) |
| Stop Time | `stop_times.txt` | `TimetabledPassingTime` | `TimetableFrame` | Estrutura semelhante, mais campos de acessibilidade |
| Calendar | `calendar.txt` | `DayType` | `ServiceCalendarFrame` | Modelo mais expressivo para variações de serviço |
| Calendar Date | `calendar_dates.txt` | `DayTypeAssignment` + `OperatingDay` | `ServiceCalendarFrame` | Suporta regras complexas (ex: "2.º domingo de cada mês") |
| Shape | `shapes.txt` | `RouteLink` (geometria) | `ServiceFrame` | NeTEx integra geometria dentro dos elementos de rota |
| Fare Zone | `fare_zones.txt` | `TariffZone` | `FareFrame` | Estrutura semelhante; NeTEx tem modelo tarifário muito mais rico |
| Fare Rule | `fare_rules.txt` | `FareProduct` + `DistanceMatrixElement` | `FareFrame` | NeTEx suporta produtos complexos (passes, bilhetes combinados) |
| — | *(não existe)* | `StopPlace` / `Quay` | `SiteFrame` | Lugar físico onde os passageiros acedem ao veículo |
| — | *(não existe)* | `AccessibilityAssessment` | `SiteFrame` | Avaliação detalhada de acessibilidade |
| — | *(não existe)* | `ServiceJourneyPattern` | `ServiceFrame` | Template de viagem reutilizável por múltiplas `ServiceJourney` |

---

## O que o NeTEx Acrescenta ao GTFS

Capacidades que o GTFS não tem (ou tem de forma muito limitada):

1. **Versionamento**: cada objeto NeTEx tem `version`, `created` e `changed` — permite auditoria de alterações
2. **Hierarquia de paragens**: `StopPlace` → `Quay` → `BoardingPosition` — distingue o edifício/praça do ponto de embarque
3. **Multimodalidade**: um `StopPlace` pode ter `AccessSpace` e `PathLink` para modelar o percurso pedonal dentro da estação
4. **Tarifas avançadas**: produtos como passes mensais, bilhetes combinados, zonas variáveis (ex: Andante, Navegante)
5. **Identificadores globais**: os IDs NeTEx são únicos globalmente (`PT:EXEMPLO:Stop:BLRB1:LOC`), não apenas localmente
6. **Perfis nacionais**: cada país pode definir um subconjunto obrigatório (Portugal tem o Perfil Nacional NeTEx, ver Módulo 7)

---

## Referências

| ID | Referência |
|----|-----------|
| [STD-01] | CEN. "NeTEx – Network Timetable Exchange." CEN/TS 16614-1:2024. https://github.com/NeTEx-CEN/NeTEx |
| [STD-04] | CEN. "Transmodel – Reference Data Model for Public Transport." EN 12896. https://transmodel-cen.eu |
| [STD-07] | MobilityData. "General Transit Feed Specification (GTFS)." https://gtfs.org |
| [REG-01] | Regulamento Delegado (UE) 2024/490 da Comissão Europeia (MMTIS). |
