# Exercício 4 — Modelar a Paragem Trindade em NeTEx

> *Módulo 4 — Multimodalidade e StopPlace*

---

## Contexto

A Trindade é um dos nós de transporte mais importantes do centro do Porto. É servida por:

- **Operadora Exemplo** — autocarro: múltiplas linhas passam na Rua de Camões / Rua 31 de Janeiro
- **Metro do Porto** — estação Trindade (Linhas A, B, C, E, F — todas exceto Linha D)

No feed GTFS da Operadora Exemplo, as paragens da Trindade incluem:

| stop_id | stop_name | stop_lat | stop_lon |
|---------|-----------|----------|----------|
| TRD1 | Trindade | 41.149540 | -8.612300 |
| TRD2 | Trindade | 41.149210 | -8.612080 |
| TRD3 | Trindade | 41.149700 | -8.612500 |

A estação de metro tem as seguintes coordenadas aproximadas:
- Entrada principal: lat=41.149380, lon=-8.611700
- Plataforma (subterrânea): lat=41.149380, lon=-8.611700

---

## Parte A — SiteFrame: StopPlace de Autocarro

Crie um `SiteFrame` com o `StopPlace` para as paragens de autocarro da Trindade.

**Requisitos:**
1. `StopPlace` com id `PT:EXEMPLO:StopPlace:Trindade:LOC`, nome "Trindade", `StopPlaceType=onstreetBus`
2. Três `Quay` dentro do StopPlace:
   - `PT:EXEMPLO:Quay:Trindade_TRD1:LOC` — nome "Trindade — TRD1", coords de TRD1
   - `PT:EXEMPLO:Quay:Trindade_TRD2:LOC` — nome "Trindade — TRD2", coords de TRD2
   - `PT:EXEMPLO:Quay:Trindade_TRD3:LOC` — nome "Trindade — TRD3", coords de TRD3
3. `StopPlace` para o metro adjacente:
   - Id: `PT:MetroPorto:StopPlace:Trindade_Metro:LOC`
   - Nome: "Trindade (Metro)"
   - `StopPlaceType=metroStation`
   - Um `Quay` com id `PT:MetroPorto:Quay:Trindade_Metro_Plat:LOC` e `QuayType=metroPlatform`

---

## Parte B — PathLink: Transferência Autocarro ↔ Metro

Adicione dois `PathLink` no `SiteFrame`:

1. **Da paragem TRD1 para a estação de metro** — tempo de transferência: 3 minutos
2. **Da paragem TRD2 para a estação de metro** — tempo de transferência: 4 minutos

Lembre-se:
- O `From` referencia o `Quay` ou `StopPlace` de partida
- O `To` referencia o destino
- Use `PT3M` e `PT4M` para os tempos em formato ISO 8601

---

## Parte C — ServiceFrame: ScheduledStopPoint e PassengerStopAssignment

Crie um `ServiceFrame` com:

1. Três `ScheduledStopPoint` (um por stop_id GTFS):
   - `PT:EXEMPLO:ScheduledStopPoint:TRD1:LOC` — nome "Trindade"
   - `PT:EXEMPLO:ScheduledStopPoint:TRD2:LOC` — nome "Trindade"
   - `PT:EXEMPLO:ScheduledStopPoint:TRD3:LOC` — nome "Trindade"

2. Três `PassengerStopAssignment` ligando cada SSP ao respetivo Quay:
   - TRD1 → StopPlace:Trindade + Quay:Trindade_TRD1
   - TRD2 → StopPlace:Trindade + Quay:Trindade_TRD2
   - TRD3 → StopPlace:Trindade + Quay:Trindade_TRD3

---

## Questões de Reflexão

**1. Centralização vs. distribuição de dados**
No GTFS de Portugal, cada operador tem o seu feed independente (Operadora Exemplo, Metro Porto, CP são feeds separados). Em NeTEx, um único `SiteFrame` pode descrever os `StopPlace` de vários operadores.

Quais são as vantagens e desvantagens desta centralização? Quem deveria ser o `ParticipantRef` de um ficheiro que descreve a infraestrutura partilhada?

**2. A Trindade tem 5 linhas de metro (A, B, C, E, F)**
O modelo que criou tem um único `Quay` para o metro. Como modelaria as duas plataformas separadas (sentido Estádio do Dragão vs. sentido Senhor de Matosinhos / Póvoa / Trofa / Maia / Aeroporto)?

**3. Versioning e obras**
Durante as obras de renovação da Rua de Camões em 2025, as paragens TRD1 e TRD2 foram temporariamente relocalizadas para a Rua de Passos Manuel (50m a sul).

Como usaria os mecanismos de versionamento do NeTEx (`validBetween`, nova `version`) para modelar esta alteração temporária sem apagar a informação original?

---

## Autoavaliação

- [ ] O `StopPlace` de autocarro tem `StopPlaceType=onstreetBus` e `TransportMode=bus`?
- [ ] Cada `Quay` tem coordenadas próprias (`Centroid/Location`) extraídas do GTFS?
- [ ] O `StopPlace` de metro tem `StopPlaceType=metroStation` e `TransportMode=metro`?
- [ ] Os `PathLink` têm `From`, `To` e `TransferDuration` com formato ISO 8601 (`PT3M`, `PT4M`)?
- [ ] Cada `PassengerStopAssignment` liga um `ScheduledStopPoint` a um `StopPlaceRef` **e** a um `QuayRef`?
- [ ] Os `id` seguem o formato `PT:<operador>:<ObjectType>:<local>:LOC`?

---

## Solução

Consulte [`solucoes/exercicio_04_solucao.xml`](solucoes/exercicio_04_solucao.xml)
