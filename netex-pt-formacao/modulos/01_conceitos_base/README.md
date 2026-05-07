# Módulo 1 — Conceitos Base

## Objetivos de Aprendizagem

Após completar este módulo, o leitor será capaz de:

- Identificar entidades, atributos e relações no modelo NeTEx/Transmodel
- Explicar o que é um `VersionFrame` e como organiza um ficheiro NeTEx
- Construir identificadores NeTEx correctos (formato hierárquico global)
- Usar referências (`ref`) entre objetos NeTEx
- Ler e navegar um ficheiro XML NeTEx real sem se perder no schema

---

## 1. Entidades, Atributos e Relações

### 1.1 NeTEx é Orientado a Objetos

O NeTEx herda do Transmodel uma abordagem orientada a objetos: tudo é uma entidade com atributos e relações com outras entidades. [[STD-04]](#STD-04)

A classe raiz de quase tudo em NeTEx é `EntityInVersion`, que garante que cada objeto tem:

- Um **identificador único** (`id`)
- Um **número de versão** (`version`)
- Uma **validade temporal** opcional (`validBetween`)

```
EntityInVersion
├── id              (obrigatório)
├── version         (obrigatório)
├── validBetween    (opcional)
│   ├── FromDate
│   └── ToDate
└── ...             (atributos específicos de cada subclasse)
```

> **Nota:** Em XML, `id` e `version` são atributos XML (`<Line id="..." version="...">`), enquanto `validBetween` é um  elemento filho (`<validBetween><FromDate>...</FromDate></validBetween>`).

Em GTFS, os objetos são linhas de CSV sem versão nem validade. Em NeTEx, cada objeto sabe quando foi criado, qual é a sua versão, e quando deixa de ser válido.

### 1.2 Hierarquia de Classes Simplificada

O diagrama seguinte mostra as classes mais importantes para os Módulos 1–3:

```
EntityInVersion
├── DataManagedObject
│   ├── Organisation
│   │   ├── Authority          ← entidade reguladora (IMT)
│   │   └── Operator           ← operador (STCP)
│   ├── Network                ← rede de linhas
│   ├── Line                   ← linha comercial (ex: Linha 200)
│   ├── Route                  ← sequência de pontos numa direção
│   ├── ScheduledStopPoint     ← ponto lógico de paragem
│   ├── StopPlace              ← lugar físico (edifício, praça)
│   ├── Quay                   ← plataforma/cais de embarque
│   └── ServiceJourney         ← viagem com horários concretos
└── VersionFrame               ← contentor de objetos (ver Secção 2)
```

### 1.3 Relações Entre Objetos

Em NeTEx, as relações entre objetos usam referências por ID (`ref`). Por exemplo:

- Uma `Route` referencia uma `Line` via `lineRef`
- Um `ServiceJourney` referencia um `ServiceJourneyPattern` via `journeyPatternRef`
- Um `ScheduledStopPoint` pode ser referenciado por múltiplas `Route` em múltiplas linhas

Isto é análogo às chaves estrangeiras de uma base de dados relacional.

---

## 2. VersionFrame — A Estrutura de um Ficheiro NeTEx

### 2.1 O que é um Frame?

Um **VersionFrame** é o contentor principal de um ficheiro NeTEx. Agrupa objetos do mesmo tipo e contexto temporal. [[STD-01]](#STD-01)

```xml
<PublicationDelivery>
  <dataObjects>
    <CompositeFrame id="PT:STCP:CompositeFrame:STCP_200:LOC" version="1">
      <frames>
        <ResourceFrame>    <!-- Operadores, autoridades -->
        <ServiceFrame>     <!-- Linhas, rotas, paragens -->
        <ServiceCalendarFrame>  <!-- Calendários, dias de serviço -->
        <TimetableFrame>   <!-- Viagens com horários -->
        <SiteFrame>        <!-- Lugares físicos, acessibilidade -->
        <FareFrame>        <!-- Tarifas, zonas, produtos -->
      </frames>
    </CompositeFrame>
  </dataObjects>
</PublicationDelivery>
```

> *Nota: os frames são mostrados de forma abreviada — cada um é um elemento XML completo com a respetiva tag de fecho (`</ResourceFrame>`, `</ServiceFrame>`, etc.).*

### 2.2 Tipos de Frame

| Frame | Conteúdo | Equivalente GTFS (aproximado) |
|-------|----------|-------------------------------|
| `ResourceFrame` | Operadores, autoridades, equipamentos | `agency.txt` |
| `ServiceFrame` | Linhas, rotas, paragens lógicas, padrões de viagem | `routes.txt`, `stops.txt` (parcial) |
| `SiteFrame` | Paragens físicas, acessibilidade | `stops.txt` (parcial) — muito mais rico |
| `ServiceCalendarFrame` | Tipos de dia, períodos de operação | `calendar.txt`, `calendar_dates.txt` |
| `TimetableFrame` | Viagens com horários | `trips.txt`, `stop_times.txt` |
| `FareFrame` | Zonas tarifárias, produtos, regras | `fare_*.txt` — muito mais rico |
| `CompositeFrame` | Contentor de múltiplos frames | *(não existe em GTFS)* |

### 2.3 PublicationDelivery 

O elemento raiz de qualquer ficheiro NeTEx é `PublicationDelivery`: [[STD-01]](#STD-01)
>Funciona como um contentor de mensagens que agrupa metadados de transação e os objetos de dados reais (dataObjects), permitindo o controle de versões e a rastreabilidade da troca de informações.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<PublicationDelivery
    xmlns="http://www.netex.org.uk/netex"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.netex.org.uk/netex NeTEx_publication.xsd"
    version="1.0">

  <!-- Metadados da publicação -->
  <PublicationTimestamp>2026-04-30T00:00:00</PublicationTimestamp>
  <ParticipantRef>PT:STCP</ParticipantRef>

  <!-- Os frames com os dados -->
  <dataObjects>
    ...
  </dataObjects>

</PublicationDelivery>
```

O `ParticipantRef` identifica quem publicou os dados, neste caso, a STCP.

---

## 3. Identificadores, Referências e Versionamento

### 3.1 Formato dos Identificadores NeTEx

Os IDs NeTEx seguem um formato hierárquico que os torna globalmente únicos: [[STD-01]](#STD-01)

```
<país>:<operador/autoridade>:<TipoDeObjeto>:<identificadorLocal>:<âmbito>
```

Exemplos:

| ID | Decomposição |
|----|-------------|
| `PT:STCP:Line:200:LOC` | País=PT, Operador=STCP, Tipo=Line, Local=200, Âmbito=LOC |
| `PT:IMT:Authority:IMT:LOC` | País=PT, Autoridade=IMT, Tipo=Authority, Local=IMT |
| `PT:STCP:StopPlace:BLRB:LOC` | País=PT, Operador=STCP, Tipo=StopPlace, Local=BLRB (Bolhão) |

O sufixo `:LOC` indica que o identificador é local (válido apenas no âmbito nacional). Para identificadores com validade europeia, usam-se outros sufixos:

| Sufixo | Âmbito | Quando usar |
|--------|--------|-------------|
| `:LOC` | Nacional | Identificadores para o NAP Portugal (caso mais comum) |
| `:GLO` | Europeu | Objetos partilhados entre países |
| `:TAP` | TAP-TSI | Dados ferroviários sob regulação TAP |

**Comparação com GTFS:**

No GTFS, o `stop_id` `BLRB1` é apenas local, não há forma de saber que pertence à STCP, que está em Portugal, ou que tipo de objecto é. No NeTEx, `PT:STCP:StopPlace:BLRB:LOC` contém toda essa informação.

### 3.2 Referências (`ref`)

Quando um objecto NeTEx precisa de referenciar outro, usa o atributo `ref`:

```xml
<!-- Definição da linha -->
<Line id="PT:STCP:Line:200:LOC" version="1">
  <Name>200</Name>
</Line>

<!-- A Route referencia a Line -->
<Route id="PT:STCP:Route:200_IDA:LOC" version="1">
  <lineRef ref="PT:STCP:Line:200:LOC" version="1"/>
  ...
</Route>
```

O atributo `version` na referência garante que se está a referenciar uma versão específica do objeto.

### 3.3 Versionamento

Cada objecto NeTEx tem um número de versão:

```xml
<Line id="PT:STCP:Line:200:LOC" version="2">
  <!-- version="2" → esta é a segunda versão desta linha -->
  <validBetween>
    <FromDate>2026-04-30</FromDate>
    <!-- Sem ToDate = válido indefinidamente -->
  </validBetween>
</Line>
```

Quando um objecto muda (ex: alteração de nome, adição de paragem), cria-se uma nova versão com um `version` incremental. As versões antigas ficam disponíveis para auditoria.

**Porque isto importa para Portugal**: o Perfil Nacional NeTEx Portugal exige que todos os objetos tenham `version` e que as datas de validade sejam preenchidas corretamente antes de submissão ao NAP. [[PT-01]](#PT-01)

---

## 4. Anatomia de um Ficheiro NeTEx Real

Veja o ficheiro de exemplo [`exemplos/01_frame_basico.xml`](exemplos/01_frame_basico.xml), que contém:

1. O envelope `PublicationDelivery` com metadados da STCP
2. Um `CompositeFrame` a agrupar os frames
3. Um `ResourceFrame` com a autoridade (IMT) e o operador (STCP)
4. Um `ServiceFrame` minimalista com a Linha 200 definida

Cada elemento está extensivamente comentado em português para explicar **o porquê** de cada escolha estrutural.

---

## Exemplos

- [`exemplos/01_frame_basico.xml`](exemplos/01_frame_basico.xml) — Anatomia completa de um ficheiro NeTEx com ResourceFrame e ServiceFrame para a STCP Linha 200

---

## Exercícios

- [Exercício 1 — Identificadores e Referências](exercicios/exercicio_01.md)

---

## Referências

| ID | Referência |
|----|-----------|
| <a id="STD-01"></a>[STD-01] | CEN. "NeTEx – Network Timetable Exchange." CEN/TS 16614-1:2024 (Parte 1: Topologia de Rede). https://github.com/NeTEx-CEN/NeTEx |
| <a id="STD-04"></a>[STD-04] | CEN. "Transmodel – Reference Data Model for Public Transport." EN 12896. https://transmodel-cen.eu |
| <a id="PT-01"></a>[PT-01] | IMT. "Perfil Nacional NeTEx Portugal." https://ptprofiles.azurewebsites.net |
| <a id="DATA-03"></a>[DATA-03] | STCP. "Feed GTFS STCP Porto (versão Escolar 228, 2026-04-30)." Feed oficial STCP. |
