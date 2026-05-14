# Exercício 2 — Converter GTFS para NeTEx: Rede de Transporte

> **Módulo 2 — Rede de Transporte**

---

## Contexto

Vai converter dados reais da Operadora Exemplo para NeTEx. Desta vez, o sujeito é a **Linha 201 (Aliados → Viso)**, uma linha diferente da âncora do curso (Linha 200), para praticar a generalização dos conceitos.

Os ficheiros GTFS de comparação estão em `../exemplos/comparacao_gtfs/`.

---

## Dados de Entrada (GTFS)

### routes.txt (extrato)
```
agency_id,route_id,route_short_name,route_type,route_long_name,route_url,route_color,route_text_color
Operadora Exemplo,201,201,3,Aliados-viso,http://www.exemplo.pt/pt/viajar/linhas/?linha=201,187EC2,FFFFFF
```

### stops.txt (6 paragens da direção IDA)
```
stop_lat,stop_lon,stop_id,stop_code,stop_name,zone_id
41.147208,-8.610983,AL1,AL1,Av. Aliados,PRT1
41.151033,-8.610854,TRD6,TRD6,Trindade,PRT1
41.148261,-8.612768,PRFL,PRFL,Pr.filipa De Lencastre,PRT1
41.147560,-8.614767,GGF,GGF,Guil. G. Fernandes,PRT1
41.147223,-8.616926,CMO,CMO,Carmo,PRT1
41.147766,-8.622450,HSA5,HSA5,Hosp. St. António,PRT1
```

### trips.txt (extrato)
```
route_id,service_id,trip_id,trip_headsign,direction_id
201,DIAS UTEIS,201_0_1|228|D1|T3|N11,Viso,0
```

---

## Tarefa

Crie um ficheiro NeTEx (`exercicio_02_solucao.xml`) com:

### Parte A — ResourceFrame (10 min)
- Operador Operadora Exemplo (já visto no Módulo 1 e Exercício 1)

### Parte B — ServiceFrame: Line e Direction (15 min)
1. `Line` para a Linha 201 com todos os campos mapeados do GTFS
2. `Direction` para a direção "Viso" (IDA)

### Parte C — ServiceFrame: ScheduledStopPoint (20 min)
- 6 `ScheduledStopPoint` com `id`, `Name` e `Location` corretos
- Atenção ao formato do ID: `PT:EXEMPLO:ScheduledStopPoint:<stop_id>:LOC`
- Nota: PRFL, GGF, CMO e HSA5 já foram definidos no Módulo 2 para a Linha 200 — os IDs devem ser **idênticos** (partilhados entre linhas)

### Parte D — ServiceFrame: Route e ServiceJourneyPattern (20 min)
1. Uma `Route` referenciando a `Line` e a `Direction`
2. Um `ServiceJourneyPattern` com os 6 `StopPointInJourneyPattern` na ordem correta
3. Um `DestinationDisplay` com `FrontText` = "Viso"

---

## Questões de Reflexão

Responda (em comentários XML ou num ficheiro `.md` separado):

1. As paragens PRFL, GGF, CMO e HSA5 são usadas pela Linha 200 e pela Linha 201. Como o NeTEx trata este cenário — os `ScheduledStopPoint` são duplicados ou partilhados?

2. Na Linha 200, a paragem "Bolhão" tem `stop_id=BLRB1`. Na Linha 201, a paragem "Av. Aliados" tem `stop_id=AL1`. São paragens próximas mas distintas. Em GTFS, como sabemos que são da mesma zona? Em NeTEx, que elemento modelaria essa relação de proximidade?

3. O campo `route_type=3` do GTFS mapeia para `TransportMode=bus` em NeTEx. Que outros valores de `route_type` existem no GTFS e quais seriam os `TransportMode` correspondentes no NeTEx?

---

## Solução

A solução está em [`solucoes/exercicio_02_solucao.xml`](solucoes/exercicio_02_solucao.xml).

---

## Autoavaliação

- [ ] O `id` da `Line` usa o formato `PT:EXEMPLO:Line:201:LOC`?
- [ ] Os `ScheduledStopPoint` partilhados com a Linha 200 têm o mesmo `id`?
- [ ] A `Route` tem `lineRef` e `DirectionRef` com os IDs corretos?
- [ ] O `ServiceJourneyPattern` lista as 6 paragens em ordem crescente de `order`?
- [ ] O `DestinationDisplay` tem `FrontText` = "Viso"?
- [ ] Todos os elementos têm `version="1"`?
