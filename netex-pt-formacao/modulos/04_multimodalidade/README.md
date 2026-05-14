# Módulo 4 — Multimodalidade e StopPlace

## Objetivos de Aprendizagem

Após completar este módulo, o leitor será capaz de:

- Distinguir `ScheduledStopPoint` (lógico) de `StopPlace`/`Quay` (físico) e ligar os dois com `PassengerStopAssignment`
- Construir a hierarquia de paragem completa: `StopPlace` → `Quay` → `BoardingPosition`
- Modelar uma paragem multimodal (autocarro + metro) com `AccessSpace` e `PathLink`
- Compreender o que é o IFOPT e como se aplica em Portugal

---

## 1. O Problema: GTFS Mistura Físico e Lógico

No GTFS, `stops.txt` descreve ao mesmo tempo:

- **A identidade lógica**: `stop_id`, `stop_name` — o ponto onde o veículo pára
- **A localização física**: `stop_lat`, `stop_lon` — as coordenadas GPS do local
- **A hierarquia física**: `parent_station` + `location_type` — qual é o edifício pai

Esta mistura é prática mas limitante. No caso do Bolhão em Porto, o GTFS tem quatro entradas separadas para o mesmo cruzamento:

```
BLRB1  Bolhão   41.151868  -8.607115   (paragem IDA de várias linhas)
BLRB2  Bolhão   41.151672  -8.607192   (paragem VOLTA de outras linhas)
BLRB3  Bolhão   41.151416  -8.607310   (paragem adicional)
BLRB4  Bolhão   41.150976  -8.606997   (paragem adicional)
BLM    Bolhão (metro)  41.149637  -8.604933  (estação de metro próxima)
```

São cinco entidades distintas no GTFS sem relação estrutural entre si. Para saber que estas paragens estão no mesmo nó de transferência, é necessário inferir pela proximidade geográfica ou pela presença de `parent_station`.

O NeTEx resolve isto com separação clara entre as três camadas:

```
Camada 1 — LÓGICA (ServiceFrame):
  ScheduledStopPoint  → onde o veículo para (conceito de horário)

Camada 2 — FÍSICA (SiteFrame):
  StopPlace           → o local físico (edifício, praça, largo)
  Quay                → a plataforma específica de embarque
  BoardingPosition    → posição exacta de subida (porta do veículo)

Camada 3 — LIGAÇÃO (ServiceFrame):
  PassengerStopAssignment → liga ScheduledStopPoint a Quay
```

---

## 2. SiteFrame — O Frame dos Lugares Físicos

Todos os objetos físicos vivem no **`SiteFrame`**: [[STD-01]](#STD-01)

```xml
<SiteFrame id="PT:EXEMPLO:SiteFrame:Porto_SF:LOC" version="1">
  <stopPlaces>
    <StopPlace> ... </StopPlace>
    <StopPlace> ... </StopPlace>
  </stopPlaces>
</SiteFrame>
```

O `SiteFrame` é o único frame que não tem equivalente direto em GTFS, é a grande adição do NeTEx para modelar a infraestrutura física de transporte.

---

## 3. Hierarquia StopPlace → Quay → BoardingPosition

### 3.1 StopPlace

Um **`StopPlace`** é o local físico como o cruzamento, a praça, o edifício ou a estação onde os passageiros acedem ao transporte. [[STD-01]](#STD-01)

Um `StopPlace` pode ter:
- **Múltiplos `Quay`** — uma plataforma por direção/modo
- **`AccessSpace`** — espaços de acesso (corredores, átrios, cais cobertos)
- **`StopPlaceEntrance`** — portas de entrada/saída

```xml
<StopPlace id="PT:EXEMPLO:StopPlace:Bolhao:LOC" version="1">
  <Name>Bolhão</Name>
  <Centroid>                 <!-- coordenadas do centro do StopPlace -->
    <Location>
      <Longitude>-8.607115</Longitude>
      <Latitude>41.151868</Latitude>
    </Location>
  </Centroid>
  <StopPlaceType>onstreetBus</StopPlaceType>   <!-- tipo de paragem -->
  <TransportMode>bus</TransportMode>
  <quays> ... </quays>
</StopPlace>
```

### 3.2 Quay

Um **`Quay`** é a plataforma específica onde os passageiros embarcam/desembarcam de um veículo. [[STD-01]](#STD-01) Cada `Quay` tem coordenadas próprias (o ponto exato na rua) e pode ter informação de acessibilidade.

```xml
<Quay id="PT:EXEMPLO:Quay:Bolhao_IDA:LOC" version="1">
  <Name>Bolhão — Sentido Castelo do Queijo</Name>
  <Centroid>
    <Location>
      <Longitude>-8.607115</Longitude>
      <Latitude>41.151868</Latitude>
    </Location>
  </Centroid>
  <QuayType>busStop</QuayType>
</Quay>
```

### 3.3 Comparação GTFS ↔ NeTEx para Paragens Físicas

| GTFS (`stops.txt`) | NeTEx | Notas |
|-------------------|-------|-------|
| `stop_id` (location_type=1, parent) | `StopPlace/id` | O lugar físico agregador |
| `stop_id` (location_type=0, stop) | `Quay/id` | A plataforma de embarque |
| `stop_lat`, `stop_lon` | `Quay/Centroid/Location` | Coordenadas da plataforma |
| `stop_name` | `StopPlace/Name` | Nome do lugar |
| `parent_station` | implícito (Quay está dentro de StopPlace) | NeTEx usa hierarquia XML |
| *(não existe)* | `StopPlaceType` | Tipo físico da paragem |
| *(não existe)* | `AccessibilityAssessment` | Acessibilidade da infraestrutura |

> **Nota:** No Módulo 2, os `ScheduledStopPoint` incluíam `Location` por conveniência pedagógica. Na arquitectura NeTEx correcta, o `ScheduledStopPoint` é uma abstracção **puramente lógica** — as coordenadas físicas pertencem ao `Quay`, ligado via `PassengerStopAssignment`. O campo `stop_code` do GTFS mapeia para `Quay/PublicCode`.

---

## 4. PassengerStopAssignment — A Ponte Lógico/Físico

O **`PassengerStopAssignment`** é o elemento que liga a camada lógica (`ScheduledStopPoint`, definida nos Módulos 2 e 3) à camada física (`Quay`). [[STD-01]](#STD-01)

```xml
<!-- No ServiceFrame, junto aos scheduledStopPoints -->
<stopAssignments>
  <PassengerStopAssignment id="PT:EXEMPLO:PSA:BLRB1_Bolhao_IDA:LOC" version="1"
                           order="1">
    <!-- O ponto lógico (do horário) -->
    <ScheduledStopPointRef ref="PT:EXEMPLO:ScheduledStopPoint:BLRB1:LOC" version="1"/>
    <!-- O lugar físico -->
    <StopPlaceRef ref="PT:EXEMPLO:StopPlace:Bolhao:LOC" version="1"/>
    <!-- A plataforma específica -->
    <QuayRef ref="PT:EXEMPLO:Quay:Bolhao_IDA:LOC" version="1"/>
  </PassengerStopAssignment>
</stopAssignments>
```

**Porque é necessário este elemento separado?**
Porque a relação lógico/físico pode ser complexa:
- O mesmo `ScheduledStopPoint` pode ser assignado a `Quay` diferentes em datas diferentes (ex: obras)
- O mesmo `Quay` pode servir múltiplos `ScheduledStopPoint` (ex: várias linhas usam a mesma plataforma)

---

## 5. Multimodalidade — Bolhão Porto

O nó Bolhão em Porto é servido por autocarro (Operadora Exemplo, múltiplas linhas) e metro (Metro do Porto, Linha D Amarela). No GTFS, estes modos estão em feeds separados sem ligação estrutural.

Em NeTEx, um único `StopPlace` do tipo `monomodalStopPlace` (apenas autocarros) pode ter adjacentes/dentro de si outros StopPlace para outros modos, ligados por `PathLink`.

Para Bolhão, a estrutura NeTEx seria:

```
StopPlace "Bolhão" (autocarro — onstreetBus)
├── Quay "Bolhão IDA" (stop_id BLRB1 — sentido Foz)
├── Quay "Bolhão VOLTA" (stop_id BLRB2 — sentido Aliados)
└── PathLink → StopPlace "Bolhão (Metro)" (estação de metro adjacente)

StopPlace "Bolhão (Metro)" (metro — metroStation)
└── Quay "Linha D — Bolhão" (plataforma do metro)
```

Para modelar a transferência entre autocarro e metro, o `PathLink` descreve o percurso pedonal que o passageiro faz entre as duas plataformas.

---

## 6. StopPlaceType — Tipos de Paragem

O campo `StopPlaceType` classifica o lugar físico: [[STD-01]](#STD-01)

| Valor | Descrição | Exemplo Porto |
|-------|-----------|---------------|
| `onstreetBus` | Paragem de autocarro à beira da estrada | Bolhão (Operadora Exemplo) |
| `onstreetTram` | Paragem de eléctrico à beira da estrada | Eléctricos históricos |
| `metroStation` | Estação de metro | Bolhão (Metro Porto) |
| `railStation` | Estação ferroviária | Campanhã (CP) |
| `busStation` | Terminal/estação de autocarros | Casa da Música |
| `ferryTerminal` | Terminal de ferry | Cacilhas (Transtejo) |
| `multimodal` | Nó com múltiplos modos | Campanhã |

---

## 7. IFOPT — Identificação de Objectos Fixos

O **IFOPT** (Identification of Fixed Objects in Public Transport — CEN/TS 28701) é o standard europeu para identificação única de paragens. [[STD-05]](#STD-05)

O formato de ID IFOPT para Portugal:

```
PT:NomeLocalidade:TipoDeObjecto:IdentificadorLocal
```

O Perfil Nacional NeTEx Portugal adopta o formato IFOPT para os `StopPlace` e `Quay`, alinhando com o sistema europeu de identificação de paragens (STOP.DB europeia). [[PT-01]](#PT-01)

Em termos práticos, no Perfil PT o `id` de um `StopPlace` deve seguir a convenção:
```
PT:EXEMPLO:StopPlace:Bolhao:LOC
```
Onde `Bolhao` é o identificador local reconhecido pelo operador/autoridade.

---

## Exemplos

- [`exemplos/04_01_paragem_bolhao.xml`](exemplos/04_01_paragem_bolhao.xml) — SiteFrame com StopPlace Bolhão + 2 Quay (IDA/VOLTA) + PassengerStopAssignment para a Linha 200
- [`exemplos/04_02_no_intermodal.xml`](exemplos/04_02_no_intermodal.xml) — Nó intermodal Campanhã: StopPlace multimodal com Quay de comboio, bus e metro + PathLink pedonal

---

## Exercícios

- [Exercício 4 — Modelar a paragem Trindade em NeTEx](exercicios/exercicio_04.md)

---

## Referências

| ID | Referência |
|----|-----------|
| <a id="STD-01"></a>[STD-01] | CEN. "NeTEx – Network Timetable Exchange." CEN/TS 16614-1:2024 (Parte 1). https://github.com/NeTEx-CEN/NeTEx |
| <a id="STD-05"></a>[STD-05] | CEN. "IFOPT – Identification of Fixed Objects in Public Transport." CEN/TS 28701. |
| <a id="PT-01"></a>[PT-01] | IMT. "Perfil Nacional NeTEx Portugal." https://ptprofiles.azurewebsites.net |
| <a id="DATA-03"></a>[DATA-03] | Operadora Exemplo. "Feed GTFS Operadora Exemplo (versão Escolar 228, 2026-04-30)." Feed oficial Operadora Exemplo. |
