# Módulo 2 — Rede de Transporte

## Objetivos de Aprendizagem

Após completar este módulo, o leitor será capaz de:

- Distinguir `Line`, `Route`, `RoutePoint` e `ScheduledStopPoint` e as suas relações
- Explicar a diferença entre ponto lógico (`ScheduledStopPoint`) e lugar físico (`StopPlace`/`Quay`)
- Converter `routes.txt` e `stops.txt` do GTFS para os elementos NeTEx equivalentes
- Criar um ficheiro NeTEx com a Linha 200 da Operadora Exemplo, do simples para o completo

---

## 1. Entidades da Rede de Transporte

### 1.1 Visão Geral

O modelo de rede no NeTEx separa claramente três camadas: [[STD-01]](#STD-01)

```
┌─────────────────────────────────────────────┐
│  CAMADA COMERCIAL (o que o passageiro vê)   │
│                                             │
│  Network ──── Line                          │
│                │                            │
│              Direction                      │
└──────────────────────────────────────────── ┘
                    │
┌───────────────────▼─────────────────────────┐
│  CAMADA TOPOLÓGICA (a geometria da linha)   │
│                                             │
│  Route ──── RoutePoint ──── RouteLink       │
│                                             │
│  ServiceJourneyPattern ──── StopPointInJourneyPattern │
└─────────────────────────────────────────────┘
                    │
┌───────────────────▼─────────────────────────┐
│  CAMADA FÍSICA (os lugares no terreno)       │
│                                             │
│  StopPlace ──── Quay ──── BoardingPosition  │
│       │                                     │
│  PassengerStopAssignment (liga camadas)     │
└─────────────────────────────────────────────┘
```

**Comparação com GTFS**: no GTFS, estas três camadas estão misturadas em `routes.txt` e `stops.txt`. O NeTEx separa-as para permitir maior flexibilidade (ex: a mesma paragem física serve várias linhas; uma linha muda de rota mas mantém a identidade).

### 1.2 Line — A Linha Comercial

Uma `Line` representa a linha de transporte como é conhecida pelos passageiros — o "Linha 200" ou "Metro Azul". Contém apenas informação comercial: nome, código público, cor, modo de transporte. [[STD-01]](#STD-01)

A `Line` NÃO contém a sequência de paragens. Isso é responsabilidade da `Route`.

```xml
<!-- Equivalente GTFS: uma linha de routes.txt -->
<Line id="PT:EXEMPLO:Line:200:LOC" version="1">
  <Name>Bolhão — Castelo do Queijo</Name>
  <PublicCode>200</PublicCode>
  <TransportMode>bus</TransportMode>
</Line>
```

> **Nota sobre nomes:** Nos exemplos XML deste curso, os nomes de paragens são expandidos para a forma completa (ex: `Hosp. St. António` → `Hospital Santo António`, `Univ. Católica` → `Universidade Católica`). O NeTEx encoraja nomes legíveis e completos no campo `Name`, ao contrário do GTFS que frequentemente usa abreviaturas por limitações de espaço.

**Comparação GTFS ↔ NeTEx para a Linha 200:**

| Campo GTFS (`routes.txt`) | Valor | Equivalente NeTEx | Elemento |
|---------------------------|-------|-------------------|---------|
| `route_id` | `200` | `id` | `Line` |
| `route_short_name` | `200` | `PublicCode` | `Line` |
| `route_long_name` | `Bolhão - Cast.queijo` | `Name` | `Line` |
| `route_type` | `3` (bus) | `TransportMode` = `bus` | `Line` |
| `route_color` | `187EC2` | `Presentation/Colour` | `Line` |
| `route_text_color` | `FFFFFF` | `Presentation/TextColour` | `Line` |
| `agency_id` | `Operadora Exemplo` | `OperatorRef` | `Line` |

### 1.3 Direction — A Direção da Linha

Uma linha tem tipicamente duas direções (ida e volta). No NeTEx, cada direção é representada por um objeto `Direction`: [[STD-01]](#STD-01)

```xml
<Direction id="PT:EXEMPLO:Direction:200_CQ:LOC" version="1">
  <!-- Direção → Castelo do Queijo -->
  <Name>Castelo do Queijo</Name>
</Direction>

<Direction id="PT:EXEMPLO:Direction:200_BL:LOC" version="1">
  <!-- Direção → Bolhão -->
  <Name>Bolhão</Name>
</Direction>
```

No GTFS, a direção é apenas um campo binário `direction_id` (0 ou 1) com o `trip_headsign`. O NeTEx representa a direção como uma entidade com identidade própria, permitindo nomes mais expressivos.

### 1.4 Route — A Topologia

Uma `Route` é uma sequência ordenada de `RoutePoint` numa determinada direção. Uma `Line` tem pelo menos duas `Route` (uma por direção), mas pode ter mais (ex: variantes de percurso). [[STD-01]](#STD-01)

```
Line 200
├── Route 200_IDA  (Bolhão → Castelo do Queijo, percurso normal)
├── Route 200_IDA_CURTO (Bolhão → Mercado da Foz, percurso curto)
└── Route 200_VOLTA (Castelo do Queijo → Bolhão)
```

**Correspondência GTFS**: no GTFS, `trips.txt` liga cada viagem ao seu percurso via `shape_id`. Em NeTEx, a `Route` é uma entidade reutilizável, muitas viagens podem usar a mesma `Route`.

### 1.5 RoutePoint e ScheduledStopPoint

Um `RoutePoint` é um ponto na geometria da rota (pode ser apenas um ponto de inflexão, não necessariamente uma paragem). [[STD-01]](#STD-01)

Um `ScheduledStopPoint` é um ponto específico onde o veículo pára e os passageiros podem embarcar/desembarcar. É um conceito lógico (associado ao horário), não físico.

A relação entre os dois: um `RoutePoint` pode ou não corresponder a um `ScheduledStopPoint`. Nos casos mais simples (que é o caso deste curso), todos os `RoutePoint` correspondem a `ScheduledStopPoint`.

**Para a maioria dos operadores de autocarro**, a distinção prática é:

| Objeto | O que representa | Comparação GTFS |
|--------|-----------------|-----------------|
| `ScheduledStopPoint` | Onde o veículo para (lógico) | `stop_id` em `stop_times.txt` |
| `StopPlace` | O edifício/praça onde fica a paragem (físico) | `parent_station` em `stops.txt` |
| `Quay` | A plataforma específica de embarque | `stop_id` com `location_type=0` |

### 1.6 StopPointInJourneyPattern

Uma `ServiceJourneyPattern` lista os `ScheduledStopPoint` na ordem de uma viagem, através de objetos `StopPointInJourneyPattern`. Esta camada permite reutilizar o padrão de paragens em múltiplas viagens.

```
ServiceJourneyPattern 200_IDA
├── StopPointInJourneyPattern 1 → ref: BLRB1 (Bolhão)
├── StopPointInJourneyPattern 2 → ref: MCBL (Mercado do Bolhão)
├── ...
└── StopPointInJourneyPattern 30 → ref: CQ10 (Castelo do Queijo)
```

### 1.7 Como construir Linhas, Rotas e Padrões de Viagem

O operador já conhece as suas linhas comerciais, os sentidos de circulação e a ordem das paragens em cada percurso. A conversão para NeTEx segue uma sequência lógica que espelha a realidade operacional:

1. **Para cada linha comercial** (ex: “Linha 200”), crie uma `Line` com o código público e o nome que aparece nas paragens e nos autocarros.
2. **Para cada sentido** (ida/volta), crie uma `Direction` com o nome do destino (ex: “Castelo do Queijo”).
3. **Para cada variante de percurso** (normal, curta, desvio), crie uma `Route` associada à linha e à direção, que contenha a sequência de pontos da rota.
4. **Para cada paragem** onde o veículo efetivamente pára, crie um `ScheduledStopPoint`. Se várias rotas partilham a mesma paragem, usam o mesmo `ScheduledStopPoint`.
5. **Para cada rota**, crie um `ServiceJourneyPattern` que lista os `StopPointInJourneyPattern` – cada um destes aponta para um `ScheduledStopPoint` e define a ordem da paragem no percurso.

Este processo é facilmente automatizado a partir das bases de dados internas do operador. Um ficheiro CSV com a lista de percursos (ex: `route_id`, `stop_sequence`, `stop_id`, `stop_name`) é tudo o que precisa para gerar os elementos NeTEx correspondentes.

**Exemplo prático (Linha 200, sentido Castelo do Queijo, primeiras três paragens):**

```xml
<!-- Linha comercial -->
<Line id="PT:OPERADOR:Line:200" version="1">
  <PublicCode>200</PublicCode>
  <Name>Bolhão — Castelo do Queijo</Name>
  <TransportMode>bus</TransportMode>
</Line>

<!-- Direção -->
<Direction id="PT:OPERADOR:Direction:200_CQ" version="1">
  <Name>Castelo do Queijo</Name>
</Direction>

<!-- Rota (sequência de RoutePoints, neste exemplo apenas os pontos que são paragens) -->
<Route id="PT:OPERADOR:Route:200_IDA_NORMAL" version="1">
  <LineRef ref="PT:OPERADOR:Line:200" version="1"/>
  <DirectionRef ref="PT:OPERADOR:Direction:200_CQ" version="1"/>
  <pointsInSequence>
    <RoutePoint id="PT:OPERADOR:RoutePoint:BLRB1" version="1">
      <ProjectedPointRef ref="PT:OPERADOR:ScheduledStopPoint:BLRB1" version="1"/>
    </RoutePoint>
    <RoutePoint id="PT:OPERADOR:RoutePoint:MCBL" version="1">
      <ProjectedPointRef ref="PT:OPERADOR:ScheduledStopPoint:MCBL" version="1"/>
    </RoutePoint>
    <RoutePoint id="PT:OPERADOR:RoutePoint:CQ10" version="1">
      <ProjectedPointRef ref="PT:OPERADOR:ScheduledStopPoint:CQ10" version="1"/>
    </RoutePoint>
    <!-- ... mais RoutePoints ... -->
  </pointsInSequence>
</Route>

<!-- Padrão de viagem (reutilizável por todas as viagens da mesma rota) -->
<ServiceJourneyPattern id="PT:OPERADOR:SJP:200_IDA_NORMAL" version="1">
  <RouteRef ref="PT:OPERADOR:Route:200_IDA_NORMAL" version="1"/>
  <pointsInSequence>
    <StopPointInJourneyPattern id="PT:OPERADOR:SPIJP:200_IDA_01" version="1" order="1">
      <ScheduledStopPointRef ref="PT:OPERADOR:ScheduledStopPoint:BLRB1" version="1"/>
    </StopPointInJourneyPattern>
    <StopPointInJourneyPattern id="PT:OPERADOR:SPIJP:200_IDA_02" version="1" order="2">
      <ScheduledStopPointRef ref="PT:OPERADOR:ScheduledStopPoint:MCBL" version="1"/>
    </StopPointInJourneyPattern>
    <!-- ... até à última paragem ... -->
    <StopPointInJourneyPattern id="PT:OPERADOR:SPIJP:200_IDA_30" version="1" order="30">
      <ScheduledStopPointRef ref="PT:OPERADOR:ScheduledStopPoint:CQ10" version="1"/>
    </StopPointInJourneyPattern>
  </pointsInSequence>
</ServiceJourneyPattern>
```

Uma vez definidos estes objetos, a criação de cada `ServiceJourney` resume-se a referenciar o `ServiceJourneyPattern` e a adicionar os horários de passagem para cada `StopPointInJourneyPattern`. Desta forma, a topologia fica separada dos horários, permitindo grande reutilização e manutenção simplificada.

---

## 2. Da Linha à Rota: o Processo de Conversão GTFS → NeTEx

Para converter a Linha 200 do GTFS para NeTEx, o processo é:

```
GTFS                          NeTEx
────                          ─────
routes.txt (linha 200)    →   Line + Direction
stops.txt (59 paragens)   →   ScheduledStopPoint × 59
trips.txt (direction_id)  →   Route + RoutePoint × 30
                          →   ServiceJourneyPattern + StopPointInJourneyPattern × 30
stop_times.txt            →   (Módulo 3 — ServiceJourney + TimetabledPassingTime)
```

O que o GTFS **não tem** e o NeTEx **acrescenta**:
- `RouteLink` — segmento entre dois `RoutePoint` com a geometria da rua
- `PassengerStopAssignment` — ligação entre `ScheduledStopPoint` e `Quay` físico
- `StopPlace`/`Quay` — representação do lugar físico (ver Módulo 4)

---

## 3. Paragens da Linha 200 (Direção → Castelo do Queijo)

Dados extraídos do feed GTFS da Operadora Exemplo [[DATA-03]](#DATA-03):

| Seq | `stop_id` | Nome | Lat | Lon | Zona |
|-----|-----------|------|-----|-----|------|
| 1 | BLRB1 | Bolhão | 41.151868 | -8.607115 | PRT1 |
| 2 | MCBL | Mercado do Bolhão | 41.149509 | -8.607564 | PRT1 |
| 3 | PRDJ | Pr. D. João I | 41.147581 | -8.609126 | PRT1 |
| 4 | PRFL | Pr. Filipa de Lencastre | 41.148261 | -8.612768 | PRT1 |
| 5 | GGF | Guil. G. Fernandes | 41.147560 | -8.614767 | PRT1 |
| 6 | CMO | Carmo | 41.147223 | -8.616926 | PRT1 |
| 7 | HSA5 | Hosp. St. António | 41.147766 | -8.622450 | PRT1 |
| 8 | PAL2 | Palácio | 41.149145 | -8.625447 | PRT1 |
| 9 | PRG4 | Praça da Galiza | 41.152531 | -8.627041 | PRT1 |
| 10 | JM1 | Junta Massarelos | 41.152694 | -8.631221 | PRT1 |
| 11 | GGT1 | Gólgota | 41.152639 | -8.633694 | PRT1 |
| 12 | PLNT1 | Planetário | 41.153094 | -8.638289 | PRT1 |
| 13 | FCUP1 | Faculdade de Ciências | 41.153838 | -8.641309 | PRT1 |
| 14 | JB1 | Jardim Botânico | 41.154528 | -8.644222 | PRT1 |
| 15 | LRD1 | Lordelo | 41.154389 | -8.649028 | PRT2 |
| 16 | PLM1 | Palmeiras | 41.152356 | -8.652107 | PRT2 |
| 17 | FLUN1 | Fluvial (norte) | 41.152139 | -8.655194 | PRT2 |
| 18 | PG1 | Paulo da Gama | 41.150856 | -8.658707 | PRT2 |
| 19 | PT3 | Pasteleira | 41.150486 | -8.661488 | PRT2 |
| 20 | TRR3 | Torres | 41.150749 | -8.664619 | PRT2 |
| 21 | PCL1 | Padre Luís Cabral | 41.151806 | -8.668056 | PRT2 |
| 22 | UC1 | Univ. Católica | 41.153538 | -8.670916 | PRT2 |
| 23 | MFZ1 | Mercado da Foz | 41.155048 | -8.673343 | PRT2 |
| 24 | LIEG1 | Praça de Liége | 41.155089 | -8.677168 | PRT2 |
| 25 | CRTO3 | Crasto | 41.157713 | -8.679766 | PRT2 |
| 26 | MLH5 | Molhe | 41.159880 | -8.681660 | PRT2 |
| 27 | FCH1 | Funchal | 41.163190 | -8.683490 | PRT2 |
| 28 | HMLM3 | Homem do Leme | 41.163454 | -8.685965 | PRT2 |
| 29 | TIM1 | Timor | 41.165587 | -8.687170 | PRT2 |
| 30 | CQ10 | Castelo do Queijo | 41.167422 | -8.689127 | PRT2 |

---

## Exemplos

- [`exemplos/02_01_linha_simples.xml`](exemplos/02_01_linha_simples.xml) — Exemplo mínimo: Line + 5 paragens principais + Route simples com `ServiceJourneyPattern`
- [`exemplos/02_02_linha_completa.xml`](exemplos/02_02_linha_completa.xml) — Exemplo completo: todas as 30 paragens, 2 direções, `RouteLink` com geometria

---

## Exercícios

- [Exercício 2 — Converter GTFS para NeTEx: Rede](exercicios/exercicio_02.md)

---

## Referências

| ID | Referência |
|----|-----------|
| <a id="STD-01"></a>[STD-01] | CEN. "NeTEx – Network Timetable Exchange." CEN/TS 16614-1:2024 (Parte 1: Topologia de Rede). https://github.com/NeTEx-CEN/NeTEx |
| <a id="STD-04"></a>[STD-04] | CEN. "Transmodel – Reference Data Model for Public Transport." EN 12896. https://transmodel-cen.eu |
| <a id="PT-01"></a>[PT-01] | IMT. "Perfil Nacional NeTEx Portugal." https://ptprofiles.azurewebsites.net |
| <a id="DATA-03"></a>[DATA-03] | Operadora Exemplo. "Feed GTFS Operadora Exemplo (versão Escolar 228, 2026-04-30)." Feed oficial Operadora Exemplo. |
