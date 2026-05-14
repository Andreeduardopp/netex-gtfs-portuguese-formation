# Análise do Projeto Francês — etalab/netex-france-formation

> **Task 1.1 do PLANO_IMPLEMENTACAO.md** — Concluída em Abril 2026.
>
> Fonte: [PROJ-01] Etalab. "Formation NeTEx France." https://github.com/etalab/netex-france-formation

---

## 1. Estrutura do Projeto Francês

O projeto francês utiliza **Livebook** (notebooks interativos Elixir) com 7 módulos:

| # | Ficheiro | Tema |
|---|----------|------|
| 1 | `1_intro/intro.livemd` | Introdução, apresentação dos formadores, instruções base |
| 2 | `2_bases/bases.livemd` | Conceitos fundamentais: modelos de dados, standards, perfis |
| 3 | `3_reseau/reseau_rochefort.livemd` | Modelação de rede — exemplo Rochefort Océan (R'Bus) |
| 4 | `4_reseau/Stop_place.livemd` | Representação multimodal — hierarquia de paragens |
| 5 | `5_accessibilite/accessibilite_transport.livemd` | Acessibilidade no transporte |
| 6 | `5_accessibilite/accessibilite_voirie.livemd` | Acessibilidade na via pública |
| 7 | `6_outils/outils.livemd` | Ferramentas para exploração aprofundada |

**Rede âncora usada**: R'Bus (Rochefort Océan), 14 linhas regulares, França.

**Licenças**: CC-BY (documentação), WTFPL (código), ODbL/LO (datasets).

---

## 2. Conceitos Cobertos pelo Projeto Francês

### Módulo 2 — Bases (mapeia para o nosso Módulo 1)

- Distinção entre modelos de dados, standards e perfis
- Definição de transporte público no contexto europeu
- NeTEx vs SIRI: dados planeados vs dados em tempo real
- Conceitos espaciais: `Point`, `Link`, `StopPlace`, `Quay`, `StopPlaceEntrance`
- Perfil francês: 7 secções (Elementos Comuns, Redes, Paragens, Horários, Tarifas, Acessibilidade, Estacionamento)

### Módulo 3 — Rede (mapeia para o nosso Módulo 2)

- `Network`, `Line`, `Route`, `Direction`
- `ScheduledStopPoint` (ponto lógico) vs `Quay` (lugar físico)
- `PassengerStopAssignment` — ligação lógico/físico
- `RouteLink` — segmentos direcionais com geometria
- `ServiceJourneyPattern` — template de viagem comercial
- `ServiceJourney` — viagem específica com horários
- `DayType` e `DayTypeAssignment` — calendários

### Módulo 4 — StopPlace (mapeia para o nosso Módulo 4)

- Hierarquia: `StopPlace` → `Quay` → `BoardingPosition`
- `AccessSpace`, `PathLink` — caminhos internos
- Identificação IFOPT

### Módulos 5a e 5b — Acessibilidade (mapeiam para o nosso Módulo 5)

- `AccessibilityAssessment` — avaliação de acessibilidade
- Atributos específicos: rampas, piso tátil, informação sonora
- Acessibilidade na via pública (complementar ao transporte)

### Módulo 7 — Ferramentas (mapeia para o nosso Módulo 8)

- Validador NeTEx da Entur
- Como navegar o schema XSD
- Ferramentas de exploração do perfil

---

## 3. O que o Projeto Francês NÃO cobre

| Tópico | Justificação para incluir no projeto PT |
|--------|----------------------------------------|
| **Módulo 6: Tarifas** | Não incluído no projeto francês (apenas mencionado no perfil). Essencial para operadores PT com o Navegante e o Andante. |
| **Módulo 7: Perfil Nacional** | O projeto francês não tem equivalente público detalhado do perfil nacional PT. Este módulo é **único globalmente**. |
| **Ponte GTFS → NeTEx** | O projeto francês não usa GTFS como ponto de partida. Para operadores PT que conhecem GTFS, esta é a entrada natural. |
| **Regulação EU 2024/490** | O contexto regulatório português está em fase de implementação ativa — mais urgente para o público PT do que foi para o público FR. |

---

## 4. O que Adaptar do Projeto Francês

| Elemento | Decisão |
|----------|---------|
| Estrutura pedagógica (conceito → exemplo → exercício) | ✅ Adaptar — funciona bem |
| Progressão modular (bases → rede → paragens → horários) | ✅ Adaptar — lógica clara |
| Exemplos XML comentados extensivamente | ✅ Adaptar — metodologia excelente |
| Rede âncora (Rochefort) | ❌ Substituir por **Operadora Exemplo** (dados reais, ver Task 1.2) |
| Notebooks Livebook/Elixir | ❌ Substituir por Markdown + XML estático (mais acessível ao público PT) |
| Referências ao perfil francês | ❌ Substituir por referências ao perfil nacional PT |

---

## 5. Diferenças Estruturais do Curso Português

```
Projeto Francês (7 módulos)          Projeto Português (9 módulos)
────────────────────────────         ──────────────────────────────────
1. Intro                             0. Antes de Começar (+ GTFS bridge)
2. Bases                             1. Conceitos Base (Transmodel/NeTEx)
3. Rede (Rochefort)        →         2. Rede de Transporte (Operadora Exemplo)
4. StopPlace               →         4. Multimodalidade
5a. Acessib. Transporte    →         5. Acessibilidade
5b. Acessib. Voirie        →         (integrado no Módulo 5)
                                     3. Horários e Calendários (novo)
                                     6. Tarifas Introdução (novo)
                                     7. Perfil NeTEx Portugal (único)
6. Outils                  →         8. Ferramentas Práticas
```

---

## Referências

- [PROJ-01] Etalab. "Formation NeTEx France." https://github.com/etalab/netex-france-formation
