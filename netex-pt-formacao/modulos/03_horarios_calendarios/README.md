# Módulo 3 — Horários e Calendários

## Objetivos de Aprendizagem

Após completar este módulo, o leitor será capaz de:

- Modelar uma `ServiceJourney` com tempos de passagem (`TimetabledPassingTime`) em NeTEx
- Construir um `TimetableFrame` completo para a Linha 200
- Definir `DayType` para os três tipos de serviço da STCP (dias úteis, sábados, domingos/feriados)
- Usar `DayTypeAssignment` e `OperatingDay` para mapear `calendar_dates.txt` para NeTEx
- Converter `trips.txt` + `stop_times.txt` + `calendar_dates.txt` para os equivalentes NeTEx

---

## 1. ServiceJourney — A Viagem Concreta

### 1.1 O que é uma ServiceJourney

Uma **`ServiceJourney`** é uma viagem específica de um veículo, com horários concretos de passagem em cada paragem. É a entidade NeTEx mais próxima de uma linha de `trips.txt` + os seus `stop_times.txt`. [[STD-02]](#STD-02)

```
GTFS                              NeTEx
────                              ─────
trips.txt (1 linha)           →   ServiceJourney
  + stop_times.txt (N linhas) →   + TimetabledPassingTime × N
```

A distinção fundamental: em GTFS, os horários de uma viagem estão numa tabela separada (`stop_times.txt`). Em NeTEx, cada `ServiceJourney` contém diretamente a lista de `TimetabledPassingTime`.

### 1.2 Estrutura de uma ServiceJourney

```xml
<ServiceJourney id="PT:STCP:ServiceJourney:200_DU_0600:LOC" version="1">
  <!-- Referência ao padrão de paragens (definido no Módulo 2) -->
  <JourneyPatternRef ref="PT:STCP:SJP:200_IDA_NORMAL:LOC" version="1"/>
  <!-- Referência ao tipo de dia (definido na Secção 2) -->
  <DayTypeRefs>
    <DayTypeRef ref="PT:STCP:DayType:DiasUteis:LOC" version="1"/>
  </DayTypeRefs>
  <passingTimes>
    <TimetabledPassingTime>
      <StopPointInJourneyPatternRef ref="PT:STCP:SPIJP:200_IDA_01:LOC" version="1"/>
      <DepartureTime>06:00:00</DepartureTime>
    </TimetabledPassingTime>
    <!-- ... outros tempos ... -->
    <TimetabledPassingTime>
      <StopPointInJourneyPatternRef ref="PT:STCP:SPIJP:200_IDA_30:LOC" version="1"/>
      <ArrivalTime>06:27:00</ArrivalTime>
    </TimetabledPassingTime>
  </passingTimes>
</ServiceJourney>
```

### 1.3 Campos de TimetabledPassingTime

| Campo NeTEx | Correspondência GTFS | Notas |
|-------------|---------------------|-------|
| `ArrivalTime` | `arrival_time` | Hora de chegada à paragem |
| `DepartureTime` | `departure_time` | Hora de partida da paragem |
| `StopPointInJourneyPatternRef` | posição implícita em `stop_times.txt` | Referência ao ponto no padrão de viagem |

Quando `arrival_time` = `departure_time` (sem espera), NeTEx aceita usar apenas `DepartureTime` (exceto na última paragem, onde se usa `ArrivalTime`).

### 1.4 JourneyPatternRef — Reutilização

Note que a `ServiceJourney` não repete a lista de paragens, apenas referencia o `ServiceJourneyPattern` definido no Módulo 2. Os `TimetabledPassingTime` usam `StopPointInJourneyPatternRef` para indicar a qual paragem do padrão corresponde cada tempo.

Isto evita duplicação: se 50 viagens usam o mesmo padrão de paragens (e.g., todas as viagens da Linha 200 IDA), o padrão é definido uma vez e referenciado 50 vezes.

### 1.5 Como construir uma ServiceJourney

Quando o operador precisa representar uma viagem real. Por exemplo, a partida das 06:00 da Linha 200 em dia útil, o processo é direto. Partindo da sua tabela de horários (seja um ficheiro Excel, uma base de dados ou um sistema de bordo), identifique:

1. **O padrão de percurso (JourneyPattern)** – Lista a sequência de paragens.
2. **O tipo de dia (DayType)** – por exemplo, “Dias Úteis”, “Sábados” ou “Feriados”.
3. **Os tempos de passagem** – para cada paragem do padrão, a hora de chegada e/ou partida.

Com esses três elementos, criar a `ServiceJourney` é uma tarefa de mapeamento direto: cada paragem do padrão recebe um `TimetabledPassingTime` com a respetiva hora. Se o seu negócio trabalha com viagens de sentido único (ex: ida sem regresso), preencha `DepartureTime` para todas as paragens exceto a última, onde utiliza `ArrivalTime`. Para viagens circulares ou com tempos de espera, distinga claramente `ArrivalTime` e `DepartureTime`.

**Exemplo prático (uma viagem de 4 paragens):**
Suponha que a Linha 100 tem um padrão `JP_100_IDA` com as paragens: `Paragem_A`, `Paragem_B`, `Paragem_C`, `Paragem_D`. Os horários operacionais da primeira viagem do dia são:
- Parte de A às 06:00
- Chega a B às 06:10 e parte às 06:10 (tempo de espera zero)
- Chega a C às 06:22 e parte às 06:23 (1 minuto de espera)
- Chega a D (terminal) às 06:35

O operador gera então o seguinte XML (as referências aos IDs do padrão e do tipo de dia são as que já definiu no sistema):

```xml
<ServiceJourney id="PT:OPERADOR:ServiceJourney:100_0600_V1" version="1">
  <JourneyPatternRef ref="PT:OPERADOR:JP:100_IDA" version="1"/>
  <DayTypeRefs>
    <DayTypeRef ref="PT:OPERADOR:DayType:DiasUteis" version="1"/>
  </DayTypeRefs>
  <passingTimes>
    <TimetabledPassingTime>
      <StopPointInJourneyPatternRef ref="PT:OPERADOR:SPIJP:100_IDA_A" version="1"/>
      <DepartureTime>06:00:00</DepartureTime>
    </TimetabledPassingTime>
    <TimetabledPassingTime>
      <StopPointInJourneyPatternRef ref="PT:OPERADOR:SPIJP:100_IDA_B" version="1"/>
      <ArrivalTime>06:10:00</ArrivalTime>
      <DepartureTime>06:10:00</DepartureTime>
    </TimetabledPassingTime>
    <TimetabledPassingTime>
      <StopPointInJourneyPatternRef ref="PT:OPERADOR:SPIJP:100_IDA_C" version="1"/>
      <ArrivalTime>06:22:00</ArrivalTime>
      <DepartureTime>06:23:00</DepartureTime>
    </TimetabledPassingTime>
    <TimetabledPassingTime>
      <StopPointInJourneyPatternRef ref="PT:OPERADOR:SPIJP:100_IDA_D" version="1"/>
      <ArrivalTime>06:35:00</ArrivalTime>
    </TimetabledPassingTime>
  </passingTimes>
</ServiceJourney>
```

Note que este processo pode ser automatizado, sempre que os seus dados de negócio tiverem uma tabela de horários, uma rotina simples de conversão gera o XML NeTEx, garantindo consistência e rastreabilidade.

---

## 2. DayType e Calendários

### 2.1 O Problema do Calendário

O GTFS usa dois mecanismos para calendários:
- `calendar.txt`: regras por dia da semana + datas de início/fim
- `calendar_dates.txt`: exceções data a data (additions ou removals)

A STCP usa **exclusivamente `calendar_dates.txt`** com `exception_type=1` (service added), ou seja, lista explicitamente cada data em que cada `service_id` está ativo. Três tipos de serviço:

| `service_id` GTFS | Descrição | Dias típicos |
|---|---|---|
| `DIAS UTEIS` | Dias úteis | Segunda a sexta (exceto feriados) |
| `SÁBADOS` | Sábados | Sábados |
| `DOMINGOS\|FERIADOS` | Domingos e feriados | Domingos + feriados nacionais |

### 2.2 DayType — O Tipo de Dia

Um **`DayType`** define uma categoria de dias de operação. [[STD-02]](#STD-02) Corresponde ao `service_id` do GTFS, mas é mais expressivo, pode incluir propriedades como dias da semana.

```xml
<DayType id="PT:STCP:DayType:DiasUteis:LOC" version="1">
  <Name>Dias Úteis</Name>
  <properties>
    <PropertyOfDay>
      <DaysOfWeek>Monday Tuesday Wednesday Thursday Friday</DaysOfWeek>
    </PropertyOfDay>
  </properties>
</DayType>
```

### 2.3 OperatingDay e DayTypeAssignment

Para mapear `calendar_dates.txt`, usa-se:

- **`OperatingDay`**: representa uma data específica de operação
- **`DayTypeAssignment`**: liga um `DayType` a um `OperatingDay`

```
calendar_dates.txt:
DIAS UTEIS,20260430,1   →   OperatingDay (2026-04-30)
                              + DayTypeAssignment → DiasUteis

DOMINGOS|FERIADOS,20260501,1  →  OperatingDay (2026-05-01)
                                   + DayTypeAssignment → DomingosFeriados
```

Visualmente:

```
DayType "DiasUteis"
    ↑
DayTypeAssignment ← OperatingDay 2026-04-30 (quinta-feira)
DayTypeAssignment ← OperatingDay 2026-05-04 (segunda-feira)
DayTypeAssignment ← OperatingDay 2026-05-05 (terça-feira)
... (todos os dias úteis do ano letivo)
```

### 2.4 ServiceCalendarFrame

Todos estes objetos vivem no **`ServiceCalendarFrame`**:

```xml
<ServiceCalendarFrame id="PT:STCP:ServiceCalendarFrame:2026-2027:LOC" version="1">
  <dayTypes>
    <DayType id="PT:STCP:DayType:DiasUteis:LOC" version="1"> ... </DayType>
    <DayType id="PT:STCP:DayType:Sabados:LOC" version="1"> ... </DayType>
    <DayType id="PT:STCP:DayType:DomingosFeriados:LOC" version="1"> ... </DayType>
  </dayTypes>
  <operatingDays>
    <OperatingDay id="PT:STCP:OperatingDay:20260430:LOC" version="1">
      <CalendarDate>2026-04-30</CalendarDate>
    </OperatingDay>
    <!-- ... um OperatingDay por data ... -->
  </operatingDays>
  <dayTypeAssignments>
    <DayTypeAssignment id="PT:STCP:DTA:DU_20260430:LOC" version="1" order="1">
      <DayTypeRef ref="PT:STCP:DayType:DiasUteis:LOC" version="1"/>
      <OperatingDayRef ref="PT:STCP:OperatingDay:20260430:LOC" version="1"/>
    </DayTypeAssignment>
    <!-- ... -->
  </dayTypeAssignments>
</ServiceCalendarFrame>
```

### 2.5 Como construir os DayType e o calendário

Na prática, o operador já sabe quais dias o serviço funciona. Se utiliza um sistema interno ou mesmo uma folha de cálculo com a lista de todas as datas do ano (ex: “2026-04-30 – Dia Útil”, “2026-05-01 – Feriado”, “2026-05-02 – Sábado”, etc.), criar o `DayType` e o calendário NeTEx é um processo simples:

1. **Defina os tipos de dia** que existem na sua operação (ex: “Dias Úteis”, “Sábados”, “Domingos e Feriados”).
2. **Para cada data** do seu calendário operacional, crie um `OperatingDay` e uma `DayTypeAssignment` que ligue essa data ao tipo de dia correspondente.
3. **Associe depois** cada `ServiceJourney` ao `DayType` adequado (via `DayTypeRef`).

Se o seu negócio já produz um ficheiro `calendar_dates.txt` no formato GTFS (com `service_id` e `date`), a conversão para NeTEx é direta: cada linha desse ficheiro origina um `OperatingDay` + uma `DayTypeAssignment`. O `service_id` GTFS vira o `DayType`, e a data vira o `OperatingDay`.

**Exemplo prático:**
O operador tem uma tabela de serviço para maio de 2026:

| Data | Tipo de serviço |
|------|-----------------|
| 2026-05-01 | DomingosFeriados |
| 2026-05-02 | Sabados |
| 2026-05-04 | DiasUteis |

O sistema gera:

```xml
<OperatingDay id="PT:OPERADOR:OperatingDay:20260501" version="1">
  <CalendarDate>2026-05-01</CalendarDate>
</OperatingDay>
<DayTypeAssignment id="PT:OPERADOR:DTA:20260501_DF" version="1" order="1">
  <DayTypeRef ref="PT:OPERADOR:DayType:DomingosFeriados" version="1"/>
  <OperatingDayRef ref="PT:OPERADOR:OperatingDay:20260501" version="1"/>
</DayTypeAssignment>
<!-- repetir para 2026-05-02 (Sabados) e 2026-05-04 (DiasUteis) ... -->
```

Este método é fácil de automatizar e garante que as `ServiceJourney` só correm nos dias corretos, tal como acontece atualmente com os `service_id` do GTFS. Ao manter a mesma lógica de “lista explícita de datas”, o operador não perde controle sobre exceções (pontes, feriados regionais, dias de greve ou de oferta especial).

---

## 3. TimetableFrame — O Frame de Horários

O **`TimetableFrame`** contém todas as `ServiceJourney` de um período. [[STD-02]](#STD-02)

```
ServiceCalendarFrame   ←── define QUANDO o serviço opera (DayType, OperatingDay)
ServiceFrame           ←── define ONDE (Line, Route, ScheduledStopPoint)
TimetableFrame         ←── define AS VIAGENS concretas (ServiceJourney + tempos)
```

Para a Linha 200 completa, o `TimetableFrame` conteria ~340 `ServiceJourney` (IDA + VOLTA, todos os tipos de dia). No exemplo do módulo, mostramos 3 viagens representativas — uma por tipo de dia — usando 5 paragens para manter a legibilidade.

---

## 4. Comparação GTFS ↔ NeTEx: Horários

| Conceito GTFS | Ficheiro | Equivalente NeTEx | Frame |
|---------------|----------|-------------------|-------|
| `service_id` | `trips.txt` / `calendar_dates.txt` | `DayType` | `ServiceCalendarFrame` |
| Data com exception_type=1 | `calendar_dates.txt` | `OperatingDay` + `DayTypeAssignment` | `ServiceCalendarFrame` |
| `trip_id` | `trips.txt` | `ServiceJourney` | `TimetableFrame` |
| `arrival_time` | `stop_times.txt` | `TimetabledPassingTime/ArrivalTime` | `TimetableFrame` |
| `departure_time` | `stop_times.txt` | `TimetabledPassingTime/DepartureTime` | `TimetableFrame` |
| `wheelchair_accessible=1` | `trips.txt` | `ServiceJourney/AccessibilityAssessment` | `TimetableFrame` |
| `trip_headsign` | `trips.txt` | `DestinationDisplayRef` (no `ServiceJourneyPattern`) | `ServiceFrame` |
| *(não existe)* | — | `ServiceJourneyPattern` | `ServiceFrame` |

---

## 5. Horários da Linha 200 — Dados Reais STCP

Tempos de passagem reais para a partida das 06:00 de Bolhão, comparando os 3 tipos de serviço: [[DATA-03]](#DATA-03)

| Paragem | Dias Úteis | Sábados | Dom./Feriados |
|---------|-----------|---------|---------------|
| Bolhão (dep.) | 06:00 | 06:00 | 06:00 |
| Carmo | 06:05 | 06:04 | 06:04 |
| Praça da Galiza | 06:10 | 06:10 | 06:10 |
| Mercado da Foz | 06:21 | 06:21 | 06:21 |
| Castelo do Queijo (arr.) | 06:27 | 06:27 | 06:27 |

Ligeira diferença no Carmo (+1 min em dias úteis) — provavelmente maior tráfego no centro em dias de trabalho.

---

## Exemplos

- [`exemplos/03_horarios.xml`](exemplos/03_horarios.xml) — `ServiceCalendarFrame` com 3 `DayType` + 7 `OperatingDay` de exemplo + `TimetableFrame` com 3 `ServiceJourney` (uma por tipo de dia, 5 paragens cada), extensivamente comentado

---

## Exercícios

- [Exercício 3 — Converter calendar_dates.txt e stop_times.txt para NeTEx](exercicios/exercicio_03.md)

---

## Referências

| ID | Referência |
|----|-----------|
| <a id="STD-01"></a>[STD-01] | CEN. "NeTEx – Network Timetable Exchange." CEN/TS 16614-1:2024 (Parte 1). https://github.com/NeTEx-CEN/NeTEx |
| <a id="STD-02"></a>[STD-02] | CEN. "NeTEx – Network Timetable Exchange." CEN/TS 16614-2:2024 (Parte 2: Horários). https://github.com/NeTEx-CEN/NeTEx |
| <a id="PT-01"></a>[PT-01] | IMT. "Perfil Nacional NeTEx Portugal." https://ptprofiles.azurewebsites.net |
| <a id="DATA-03"></a>[DATA-03] | STCP. "Feed GTFS STCP Porto (versão Escolar 228, 2026-04-30)." Feed oficial STCP. |
