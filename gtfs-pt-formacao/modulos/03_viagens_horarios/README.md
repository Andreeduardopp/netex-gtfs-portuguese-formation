# Módulo 3 — Viagens, Horários e Geometria

## Objetivos de Aprendizagem

Após completar este módulo, o leitor será capaz de:

- Modelar viagens concretas em `trips.txt` com `direction_id`, `shape_id` e `wheelchair_accessible`
- Construir sequências de horários em `stop_times.txt` com `stop_sequence` crescente
- Compreender o formato de hora GTFS, incluindo tempos superiores a `24:00:00`
- Definir geometria de rotas em `shapes.txt`
- Distinguir serviço baseado em horários (`stop_times`) de serviço baseado em frequência (`frequencies`)

---

## 1. `trips.txt` — As Viagens Concretas

### 1.1 O que é uma Viagem

Uma **viagem** (`trip`) é uma instância concreta de um veículo a percorrer uma rota num determinado dia. Enquanto a `route` é a abstração (a Linha 200), o `trip` é o evento físico (o autocarro das 06:00 da Linha 200, direção Castelo do Queijo, nos dias úteis). [[STD-01]](#STD-01)

### 1.2 Campos Completos

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `route_id` | ID | Sim | A que rota pertence |
| `service_id` | ID | Sim | Quando opera (referência ao calendário) |
| `trip_id` | ID | Sim | Identificador único da viagem |
| `trip_headsign` | Texto | Não | Destino mostrado no veículo |
| `trip_short_name` | Texto | Não | Nome curto (ex: número do comboio) |
| `direction_id` | Inteiro | Não | 0=ida, 1=volta |
| `block_id` | ID | Não | Bloco operacional do veículo |
| `shape_id` | ID | Não | Referência à geometria |
| `wheelchair_accessible` | Inteiro | Não | 0=desconhecido, 1=sim, 2=não |
| `bikes_allowed` | Inteiro | Não | 0=desconhecido, 1=sim, 2=não |

### 1.3 `direction_id` — Ida e Volta

O campo `direction_id` distingue as duas direções de uma rota: [[STD-01]](#STD-01)

| Valor | Significado | Exemplo Linha 200 |
|-------|-------------|-------------------|
| `0` | Ida (outbound) | Bolhão → Castelo do Queijo |
| `1` | Volta (inbound) | Castelo do Queijo → Bolhão |

### 1.4 `block_id` — Blocos Operacionais

O `block_id` identifica o bloco, a sequência de viagens que o mesmo veículo físico realiza consecutivamente. Se duas viagens têm o mesmo `block_id`, significa que o passageiro pode permanecer no veículo durante a transição: [[STD-01]](#STD-01)

```csv
route_id,service_id,trip_id,block_id,direction_id
200,DIAS UTEIS,200_DU_0600,BLK_01,0
200,DIAS UTEIS,200_DU_0700_V,BLK_01,1
```

Neste exemplo, o mesmo autocarro (bloco `BLK_01`) faz a IDA às 06:00 e a VOLTA às 07:00.

### 1.5 Exemplo: Três Viagens da Linha 200

```csv
route_id,service_id,trip_id,trip_headsign,direction_id,shape_id,wheelchair_accessible
200,DIAS_UTEIS,200_DU_0600,Castelo do Queijo,0,shape_200_ida,1
200,SABADOS,200_SAB_0600,Castelo do Queijo,0,shape_200_ida,1
200,DOMINGOS_FERIADOS,200_DF_0600,Castelo do Queijo,0,shape_200_ida,1
```

> **Nota**: os dados reais da STCP usam IDs como `DIAS UTEIS`, `SÁBADOS` e `DOMINGOS|FERIADOS` por razões históricas. Neste curso usamos IDs alfanuméricos simples (`[a-zA-Z0-9_-]`), conforme recomendado pelas Best Practices. [[BP-01]](#BP-01)

Ver ficheiro: [`exemplos/03_01_trips_200.txt`](exemplos/03_01_trips_200.txt)

Cada viagem referencia:
- `route_id=200` → a Linha 200 definida no Módulo 2
- `service_id` → o padrão de calendário definido no Módulo 4
- `shape_id=shape_200_ida` → a geometria da rota
- `wheelchair_accessible=1` → autocarro acessível

---

## 2. `stop_times.txt` — Os Horários de Passagem

### 2.1 Campos Completos

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `trip_id` | ID | Sim | A que viagem pertence |
| `arrival_time` | Hora | Condicional | Hora de chegada (`HH:MM:SS`) |
| `departure_time` | Hora | Condicional | Hora de partida (`HH:MM:SS`) |
| `stop_id` | ID | Sim | Em que paragem |
| `stop_sequence` | Inteiro | Sim | Ordem de passagem (crescente) |
| `stop_headsign` | Texto | Não | Destino a partir desta paragem |
| `pickup_type` | Inteiro | Não | 0=normal, 1=não recolhe, 2=contactar operador, 3=coordenar com motorista |
| `drop_off_type` | Inteiro | Não | 0=normal, 1=não deixa sair, 2=contactar operador, 3=coordenar com motorista |
| `shape_dist_traveled` | Float | Não | Distância percorrida desde o início da shape (metros) |
| `timepoint` | Inteiro | Não | 1=exato, 0=aproximado |

### 2.2 Formato de Hora e o Problema da Meia-Noite

No GTFS, os tempos usam o formato `HH:MM:SS` e podem exceder `24:00:00`. [[STD-01]](#STD-01)

```csv
trip_id,arrival_time,departure_time,stop_id,stop_sequence
200_DU_2330,23:30:00,23:30:00,BLRB1,1
200_DU_2330,23:45:00,23:45:00,CMO,2
200_DU_2330,24:05:00,24:05:00,MFZ1,3
200_DU_2330,24:15:00,24:15:00,CQ10,4
```

A viagem das 23:30 chega ao Castelo do Queijo às 00:15 do dia seguinte, representado como `24:15:00`. Isto garante que a sequência temporal é sempre crescente dentro da mesma viagem.

### 2.3 `timepoint` — Tempos Exatos vs Aproximados

O campo `timepoint` distingue paragens onde o tempo é exato (horário publicado) de paragens onde o tempo é interpolado: [[BP-01]](#BP-01)

| Valor | Significado |
|-------|-------------|
| `1` (ou vazio) | Tempo exato — o veículo deve respeitar este horário |
| `0` | Tempo aproximado — interpolado entre timepoints |

As Best Practices recomendam que os tempos sejam fornecidos para todas as paragens, mesmo que interpolados, em vez de deixar os campos vazios. [[BP-01]](#BP-01)

### 2.4 Exemplo: Partida das 06:00 (Dias Úteis)

Tempos de passagem reais para 5 paragens-chave da Linha 200: [[DATA-01]](#DATA-01)

```csv
trip_id,arrival_time,departure_time,stop_id,stop_sequence,timepoint
200_DU_0600,06:00:00,06:00:00,BLRB1,1,1
200_DU_0600,06:05:00,06:05:00,CMO,2,1
200_DU_0600,06:10:00,06:10:00,PRG4,3,1
200_DU_0600,06:21:00,06:21:00,MFZ1,4,1
200_DU_0600,06:27:00,06:27:00,CQ10,5,1
```

Ver ficheiro: [`exemplos/03_02_stop_times_200.txt`](exemplos/03_02_stop_times_200.txt)

### 2.5 Comparação entre Tipos de Dia

| Paragem | Dias Úteis | Sábados | Dom./Feriados |
|---------|-----------|---------|---------------|
| Bolhão (dep.) | 06:00 | 06:00 | 06:00 |
| Carmo | 06:05 | 06:04 | 06:04 |
| Praça da Galiza | 06:10 | 06:10 | 06:10 |
| Mercado da Foz | 06:21 | 06:21 | 06:21 |
| Castelo do Queijo (arr.) | 06:27 | 06:27 | 06:27 |

A ligeira diferença no Carmo (+1 min em dias úteis) reflete maior tráfego no centro em dias de trabalho. [[DATA-01]](#DATA-01)

---

## 3. `shapes.txt` — Geometria das Rotas

### 3.1 Campos

O `shapes.txt` define o traçado geográfico de cada rota no mapa: [[STD-01]](#STD-01)

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `shape_id` | ID | Sim | Identificador da shape |
| `shape_pt_lat` | Latitude | Sim | Latitude do ponto |
| `shape_pt_lon` | Longitude | Sim | Longitude do ponto |
| `shape_pt_sequence` | Inteiro | Sim | Ordem dos pontos (crescente) |
| `shape_dist_traveled` | Float | Não | Distância acumulada (metros) |

### 3.2 Como Funciona

Uma shape é uma sequência de pontos GPS que desenha o percurso da rota no mapa:

```csv
shape_id,shape_pt_lat,shape_pt_lon,shape_pt_sequence,shape_dist_traveled
shape_200_ida,41.151868,-8.607115,1,0
shape_200_ida,41.151200,-8.607300,2,78
shape_200_ida,41.149509,-8.607564,3,340
...
shape_200_ida,41.167422,-8.689127,250,8500
```

Cada viagem em `trips.txt` referencia uma shape via `shape_id`. Múltiplas viagens podem partilhar a mesma shape se seguem o mesmo percurso.

### 3.3 Porque São Importantes

Embora tecnicamente opcionais, as shapes são fortemente recomendadas: [[BP-01]](#BP-01)

- Permitem que Google Maps, Moovit e outras apps desenhem a rota no mapa
- Sem shapes, as apps traçam linhas retas entre paragens, o que é visualmente incorreto
- A fonte primária para shapes é tipicamente dados AVL (Automatic Vehicle Location)

---

## 4. `frequencies.txt` — Serviço Baseado em Frequência

### 4.1 Quando Usar

Alguns serviços de transporte não têm horários fixos, operam com uma frequência (headway). Por exemplo: "um autocarro a cada 10 minutos entre as 07:00 e as 20:00". [[STD-01]](#STD-01)

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `trip_id` | ID | Sim | Viagem modelo |
| `start_time` | Hora | Sim | Início do período |
| `end_time` | Hora | Sim | Fim do período |
| `headway_secs` | Inteiro | Sim | Intervalo entre veículos (segundos) |
| `exact_times` | Inteiro | Não | 0=frequência, 1=horários calculados |

```csv
trip_id,start_time,end_time,headway_secs,exact_times
200_DU_FREQ,07:00:00,20:00:00,600,0
```

Neste exemplo, o autocarro passa a cada 600 segundos (10 minutos) entre as 07:00 e as 20:00.

### 4.2 `frequencies.txt` vs `stop_times.txt`

| Aspeto | `stop_times.txt` | `frequencies.txt` |
|--------|-------------------|-------------------|
| Quando usar | Horários fixos e publicados | Serviço de alta frequência |
| Precisão | Hora exata de passagem | Intervalo aproximado |
| Volume de dados | Uma linha por paragem por viagem | Uma linha por período |
| Exemplo | Linha 200 STCP (horários fixos) | Metro do Porto (cada 6 min) |

A maioria dos operadores de autocarro em Portugal usa `stop_times.txt` com horários explícitos. O `frequencies.txt` é mais comum em metros e serviços de alta frequência.

---

## 5. Ponte NeTEx (Opcional)

Para quem seguirá o curso NeTEx, as equivalências são:

| GTFS | NeTEx | Notas |
|------|-------|-------|
| `trip` | `ServiceJourney` | NeTEx adiciona padrões reutilizáveis (`JourneyPattern`) |
| `stop_time` | `TimetabledPassingTime` | Tempos dentro de uma `ServiceJourney` |
| `shape` | `RouteLink` (geometria) | NeTEx inclui geometria nos segmentos entre pontos |
| `trip_headsign` | `DestinationDisplay` | NeTEx modela o letreiro como entidade separada |

---

## Exemplos

- [`exemplos/03_01_trips_200.txt`](exemplos/03_01_trips_200.txt) — 3 viagens da Linha 200 (uma por tipo de dia)
- [`exemplos/03_02_stop_times_200.txt`](exemplos/03_02_stop_times_200.txt) — Horários de passagem da viagem das 06:00 (dias úteis, 5 paragens-chave)

---

## Exercícios

- [Exercício 3 — Criar Viagens e Horários](exercicios/exercicio_03.md)

---

## Referências

| ID | Referência |
|----|-----------|
| <a id="STD-01"></a>[STD-01] | MobilityData. "GTFS Schedule Reference." https://gtfs.org/schedule/reference |
| <a id="BP-01"></a>[BP-01] | MobilityData. "GTFS Schedule Best Practices." https://gtfs.org/documentation/schedule/schedule-best-practices |
| <a id="DATA-01"></a>[DATA-01] | STCP. "Feed GTFS STCP Porto (versão Escolar 228, 2026-04-30)." Feed oficial STCP. |
