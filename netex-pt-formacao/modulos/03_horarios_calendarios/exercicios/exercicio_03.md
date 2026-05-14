# Exercício 3 — Converter calendar_dates.txt e stop_times.txt para NeTEx

> **Módulo 3 — Horários e Calendários**

---

## Contexto

Vai criar um `ServiceCalendarFrame` e um `TimetableFrame` completos para uma segunda partida da Linha 200 — a das **06:20** em Dias Úteis — e para a **primeira partida de Sábado** (também às 06:00 de Bolhão).

Os dados vêm diretamente do feed GTFS da Operadora Exemplo [DATA-03].

---

## Dados de Entrada

### calendar_dates.txt (extrato — primeira semana)

```
service_id,date,exception_type
DIAS UTEIS,20260430,1
DOMINGOS|FERIADOS,20260501,1
SÁBADOS,20260502,1
DOMINGOS|FERIADOS,20260503,1
DIAS UTEIS,20260504,1
DIAS UTEIS,20260505,1
SÁBADOS,20260509,1
```

### trips.txt (as viagens a modelar)

```
route_id,service_id,trip_id,trip_headsign,wheelchair_accessible,direction_id
200,DIAS UTEIS,200_DU_0620,Cast. Queijo,1,0
200,SÁBADOS,200_SAB_0600,Cast. Queijo,1,0
```

*(IDs simplificados para o exercício; o feed real usa IDs mais longos)*

### stop_times.txt (5 paragens para cada viagem)

**Viagem dias úteis 06:20 (200_DU_0620):**
```
trip_id,arrival_time,departure_time,stop_id,stop_sequence
200_DU_0620,06:20:00,06:20:00,BLRB1,1
200_DU_0620,06:25:00,06:25:00,CMO,6
200_DU_0620,06:30:00,06:30:00,PRG4,9
200_DU_0620,06:41:00,06:41:00,MFZ1,23
200_DU_0620,06:47:00,06:47:00,CQ10,30
```

**Viagem sábado 06:00 (200_SAB_0600):**
```
trip_id,arrival_time,departure_time,stop_id,stop_sequence
200_SAB_0600,06:00:00,06:00:00,BLRB1,1
200_SAB_0600,06:04:00,06:04:00,CMO,6
200_SAB_0600,06:10:00,06:10:00,PRG4,9
200_SAB_0600,06:21:00,06:21:00,MFZ1,23
200_SAB_0600,06:27:00,06:27:00,CQ10,30
```

---

## Tarefas

### Parte A — ServiceCalendarFrame (25 min)

Crie um `ServiceCalendarFrame` com:

1. Os 3 `DayType` da Operadora Exemplo (já viu no exemplo `03_horarios.xml` — podem ser copiados)
2. `OperatingDay` para as 7 datas do extrato acima
3. `DayTypeAssignment` corretos para cada data:
   - 20260430 (quinta-feira) → ?
   - 20260501 (feriado) → ?
   - 20260502 (sábado) → ?
   - 20260503 (domingo) → ?
   - 20260504 (segunda) → ?
   - 20260505 (terça) → ?
   - 20260509 (sábado) → ?

### Parte B — TimetableFrame: viagem dias úteis 06:20 (20 min)

Crie a `ServiceJourney` para a viagem `200_DU_0620`:
- `DayTypeRef` → `DiasUteis`
- `JourneyPatternRef` → o mesmo padrão usado no exemplo (`200_IDA_NORMAL`)
- 5 `TimetabledPassingTime` com os horários corretos
- `AccessibilityAssessment` com `WheelchairAccess=yes`

### Parte C — TimetableFrame: viagem sábado 06:00 (20 min)

Crie a `ServiceJourney` para a viagem `200_SAB_0600`:
- Estrutura igual à da Parte B, mas com `DayTypeRef` → `Sabados`
- Horários do sábado (ligeiramente diferentes dos dias úteis)

---

## Questões de Reflexão

1. A viagem `200_DU_0600` (do exemplo `03_horarios.xml`) e a viagem `200_DU_0620` (deste exercício) partilham o mesmo `ServiceJourneyPattern`. Que partes do XML são **exatamente iguais** entre as duas viagens? E o que muda?

2. Em GTFS, o campo `wheelchair_accessible=1` está em `trips.txt`. Em NeTEx, o equivalente (`AccessibilityAssessment`) pode estar na `ServiceJourney` (como neste módulo) ou na `StopPlace`/`Quay` (Módulo 4). O que modela cada um? São mutuamente exclusivos?

3. O feed GTFS da Operadora Exemplo tem 341 viagens para a Linha 200. Numa implementação real em NeTEx, todas elas teriam de ser representadas como `ServiceJourney`. Que estratégias existem para gerir esta escala?

---

## Solução

A solução está em [`solucoes/exercicio_03_solucao.xml`](solucoes/exercicio_03_solucao.xml).

---

## Autoavaliação

- [ ] Cada `OperatingDay` tem `CalendarDate` no formato `YYYY-MM-DD`?
- [ ] O feriado 20260501 está associado a `DomingosFeriados`, não a `DiasUteis`?
- [ ] Os `DayTypeAssignment` têm atributo `order`?
- [ ] Os `TimetabledPassingTime` usam `ArrivalTime` na última paragem e `DepartureTime` na primeira?
- [ ] As viagens de dias úteis e de sábado têm `DayTypeRef` diferentes?
- [ ] O `id` das `ServiceJourney` segue o formato `PT:EXEMPLO:ServiceJourney:...:LOC`?
