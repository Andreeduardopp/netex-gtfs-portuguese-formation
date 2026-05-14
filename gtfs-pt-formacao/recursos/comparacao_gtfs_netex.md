# Comparação GTFS vs NeTEx — Referência Rápida

> Este documento serve de ponte para quem transita entre os dois standards. Para o curso NeTEx completo, consultar [`netex-pt-formacao/`](../../netex-pt-formacao/).

---

## Tabela Comparativa Principal

| Dimensão | GTFS | NeTEx |
|----------|------|-------|
| **Formato** | CSV (ficheiros `.txt` num `.zip`) | XML (por norma um `.xml` por `Frame`) |
| **Modelo conceptual** | Pragmático, sem modelo formal | Baseado no Transmodel (CEN EN 12896) |
| **Cobertura** | Horários, paragens, rotas, tarifas básicas | Tudo do GTFS + acessibilidade detalhada, multimodalidade, tarifas complexas |
| **Identificadores** | Locais (sem formato obrigatório) | Globais, hierárquicos: `PT:EXEMPLO:Line:200:LOC` |
| **Versionamento** | Não existe nativamente | Nativo — cada objeto tem `version` e `validFrom/To` |
| **Complexidade** | Baixa — 6 ficheiros obrigatórios | Alta — schema XSD extenso (~500 tipos XML) |
| **Adoção** | 10.000+ agências globalmente | Obrigatório na UE (Regulamento 2024/490) |
| **Extensibilidade** | Via extensões ad hoc (GTFS-Flex, GTFS-Fares v2) | Nativa — perfis nacionais definem subconjuntos |
| **Tempo real** | GTFS Realtime (Protocol Buffers) | SIRI (XML), baseado no mesmo Transmodel |
| **Regulação UE** | Aceite para dados básicos | Obrigatório para dados avançados |

---

## Equivalências de Conceitos

| Conceito GTFS | Ficheiro | Equivalente NeTEx | Notas |
|---------------|----------|-------------------|-------|
| `agency` | `agency.txt` | `Operator` + `Authority` | NeTEx distingue quem opera de quem tem autoridade regulatória |
| `route` | `routes.txt` | `Line` + `Route` + `Direction` | NeTEx separa a linha comercial da topologia da rota |
| `stop` (paragem) | `stops.txt` | `ScheduledStopPoint` | Ponto lógico no NeTEx; o lugar físico é `StopPlace`/`Quay` |
| `stop` (estação) | `stops.txt` (`location_type=1`) | `StopPlace` | NeTEx modela a hierarquia física completa |
| `trip` | `trips.txt` | `ServiceJourney` | NeTEx adiciona padrões reutilizáveis (`JourneyPattern`) |
| `stop_time` | `stop_times.txt` | `TimetabledPassingTime` | Tempos de passagem dentro de uma `ServiceJourney` |
| `calendar` / `calendar_dates` | `calendar.txt`, `calendar_dates.txt` | `DayType` + `DayTypeAssignment` | Modelo mais expressivo para variações de serviço |
| `shape` | `shapes.txt` | `RouteLink` (geometria) | NeTEx inclui geometria nos segmentos entre pontos da rota |
| `fare_attributes` / `fare_rules` | `fare_*.txt` | `FareFrame` (tarifas completas) | NeTEx Parte 3 modela passes, descontos, zonas complexas |
| `wheelchair_accessible` | `trips.txt` | `AccessibilityAssessment` | NeTEx avalia 6 dimensões com 4 estados (yes/no/partial/unknown) |
| `transfers` | `transfers.txt` | `PathLink` + `TransferDuration` | NeTEx modela percursos pedonais completos |
| `pathways` | `pathways.txt` | `AccessSpace` + `PathLink` | NeTEx modela a infraestrutura física da estação |
| *(não existe)* | — | `VersionFrame` | Contentor versionado — GTFS não tem equivalente |

---

## O que o NeTEx Adiciona ao GTFS

1. **Versionamento nativo** — cada objeto tem `version`, `created`, `changed`
2. **Hierarquia de paragens** — `StopPlace` → `Quay` → `BoardingPosition`
3. **Multimodalidade** — `AccessSpace` e `PathLink` para modelar nós intermodais
4. **Tarifas avançadas** — passes mensais, bilhetes combinados, zonas variáveis
5. **Identificadores globais** — formato `PT:EXEMPLO:StopPlace:BLRB:LOC`
6. **Perfis nacionais** — subconjuntos obrigatórios por país

## O que o GTFS Oferece que o NeTEx Não

1. **Simplicidade** — 6 ficheiros CSV para um feed funcional
2. **Adoção universal** — suportado nativamente por Google Maps, Apple Maps, Transit App
3. **Baixa barreira de entrada** — editável numa folha de cálculo
4. **Ecossistema de ferramentas** — validadores, bibliotecas, conversores amplamente disponíveis
5. **GTFS Realtime** — protocolo eficiente (Protocol Buffers) para dados em tempo real

---

## Referências

| ID | Referência |
|----|-----------|
| [STD-01] | MobilityData. "GTFS Schedule Reference." https://gtfs.org/schedule/reference |
| [STD-02] | MobilityData. "About GTFS." https://gtfs.org/about |

*Para referências NeTEx, consultar o registo central em [`netex-pt-formacao/referencias/README.md`](../../netex-pt-formacao/referencias/README.md).*
