# Módulo 1 — Conceitos Base

## Objetivos de Aprendizagem

Após completar este módulo, o leitor será capaz de:

- Descrever o modelo de dados relacional do GTFS e a hierarquia entre ficheiros
- Identificar os 6 ficheiros obrigatórios e o seu papel no feed
- Compreender as convenções de formato: CSV, cabeçalhos, tipos de dados, encoding
- Explicar o conceito de `service_id` como abstração de calendário
- Criar a estrutura de um feed GTFS mínimo válido

---

## 1. O Modelo de Dados GTFS

### 1.1 GTFS como Base de Dados Relacional

O GTFS é, na sua essência, uma base de dados relacional exportada como ficheiros CSV. [[STD-01]](#STD-01) Cada ficheiro `.txt` corresponde a uma tabela, com:

- Uma linha de cabeçalho que define os nomes dos campos (colunas)
- Linhas de dados com valores separados por vírgula
- Chaves primárias que identificam cada registo de forma única
- Chaves estrangeiras que ligam registos entre tabelas

A analogia certa: pense no GTFS como uma base de dados PostgreSQL ou SQLite onde cada tabela foi exportada para CSV e empacotada num ZIP.

### 1.2 Hierarquia Relacional

A hierarquia central do GTFS segue esta estrutura: [[STD-01]](#STD-01)

```
agency.txt (QUEM opera)
  │
  └── routes.txt (QUE linhas opera)
        │
        ├── trips.txt (QUE viagens concretas existem)
        │     │
        │     ├── stop_times.txt (A QUE HORAS passa em cada paragem)
        │     │     │
        │     │     └── stops.txt (ONDE ficam as paragens)
        │     │
        │     └── calendar.txt / calendar_dates.txt (QUANDO opera)
        │
        └── shapes.txt (POR ONDE passa no mapa)
```

Cada nível da hierarquia responde a uma pergunta fundamental sobre o serviço de transporte.

### 1.3 Chaves e Relações

| Tabela | Chave Primária | Chaves Estrangeiras |
|--------|----------------|---------------------|
| `agency` | `agency_id` | — |
| `routes` | `route_id` | `agency_id` → `agency` |
| `trips` | `trip_id` | `route_id` → `routes`, `service_id` → `calendar`/`calendar_dates` |
| `stop_times` | (`trip_id`, `stop_sequence`) | `trip_id` → `trips`, `stop_id` → `stops` |
| `stops` | `stop_id` | `parent_station` → `stops` (auto-referência) |
| `calendar` | `service_id` | — |
| `calendar_dates` | (`service_id`, `date`) | `service_id` → `calendar` (opcional) |
| `shapes` | (`shape_id`, `shape_pt_sequence`) | — |

> **Nota:** `stop_times` tem uma chave primária composta — a combinação de `trip_id` e `stop_sequence` é que identifica de forma única cada registo.

---

## 2. Formato e Convenções

### 2.1 Formato CSV

Todos os ficheiros GTFS seguem as mesmas convenções CSV: [[STD-01]](#STD-01)

- **Separador**: vírgula (`,`)
- **Encoding**: UTF-8 (suporta caracteres portugueses: ã, ç, é, ô, etc.)
- **Primeira linha**: cabeçalho com nomes dos campos
- **Valores com vírgula**: envolvidos em aspas duplas (`"valor, com vírgula"`)
- **Campos opcionais vazios**: representados por campo vazio entre vírgulas (`,,`)

```csv
stop_id,stop_name,stop_lat,stop_lon
BLRB1,Bolhão,41.151868,-8.607115
MCBL,Mercado do Bolhão,41.149509,-8.607564
```

### 2.2 Tipos de Dados

| Tipo | Formato | Exemplo |
|------|---------|---------|
| **ID** | String sem espaços (recomendado) | `BLRB1`, `200`, `Operadora Exemplo` |
| **Texto** | String UTF-8 | `Bolhão`, `Mercado do Bolhão` |
| **Latitude** | Decimal (graus, WGS 84) | `41.151868` |
| **Longitude** | Decimal (graus, WGS 84) | `-8.607115` |
| **Hora** | `HH:MM:SS` (pode exceder 24:00) | `06:00:00`, `25:30:00` |
| **Data** | `YYYYMMDD` | `20260430` |
| **Inteiro** | Número inteiro | `0`, `1`, `3` |
| **Cor** | Hexadecimal (6 dígitos, sem `#`) | `187EC2`, `FFFFFF` |
| **URL** | URL completo | `https://www.exemplo.pt` |
| **Timezone** | Fuso horário IANA | `Europe/Lisbon` |

> **Atenção ao formato de hora**: no GTFS, horas que passam da meia-noite usam valores superiores a `24:00:00`. Uma viagem que chega à 1:30 da manhã é representada como `25:30:00` se começou antes da meia-noite. Isto garante que a sequência temporal de uma viagem é sempre crescente.

### 2.3 Estrutura do ZIP

O feed GTFS é distribuído como um ficheiro `.zip` que contém diretamente os ficheiros `.txt` na raiz — sem pastas intermediárias: [[BP-01]](#BP-01)

```
feed_exemplo.zip
├── agency.txt
├── routes.txt
├── stops.txt
├── trips.txt
├── stop_times.txt
├── calendar_dates.txt
├── shapes.txt
└── feed_info.txt
```

> **Erro comum**: compactar a pasta que contém os ficheiros em vez dos ficheiros diretamente. Validadores rejeitam feeds onde os `.txt` estão dentro de um subdirectório.

---

## 3. Os 6 Ficheiros Obrigatórios

### 3.1 `agency.txt` — O Operador

Define a(s) empresa(s) de transporte. Cada feed pode ter múltiplos operadores.

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `agency_id` | ID | Condicional* | Identificador único do operador |
| `agency_name` | Texto | Sim | Nome do operador |
| `agency_url` | URL | Sim | Website do operador |
| `agency_timezone` | Timezone | Sim | Fuso horário (`Europe/Lisbon`) |
| `agency_lang` | Código | Não | Idioma (`pt`) |
| `agency_phone` | Texto | Não | Telefone de contacto |

*Obrigatório quando existem múltiplos operadores no feed.

### 3.2 `stops.txt` — As Paragens

Define os locais de embarque e desembarque.

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `stop_id` | ID | Sim | Identificador único da paragem |
| `stop_name` | Texto | Sim | Nome da paragem |
| `stop_lat` | Latitude | Sim | Latitude (WGS 84) |
| `stop_lon` | Longitude | Sim | Longitude (WGS 84) |
| `location_type` | Inteiro | Não | 0=paragem, 1=estação, 2=entrada |
| `parent_station` | ID | Não | Referência à estação-pai |

### 3.3 `routes.txt` — As Rotas

Define as linhas de transporte.

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `route_id` | ID | Sim | Identificador único da rota |
| `agency_id` | ID | Condicional | Referência ao operador |
| `route_short_name` | Texto | Condicional** | Código curto (`200`) |
| `route_long_name` | Texto | Condicional** | Nome completo (`Bolhão - Castelo do Queijo`) |
| `route_type` | Inteiro | Sim | Modo de transporte |

**Pelo menos um de `route_short_name` ou `route_long_name` é obrigatório.

Códigos `route_type` mais relevantes para Portugal:

| Código | Modo |
|--------|------|
| `0` | Eléctrico (Tram) |
| `1` | Metro |
| `2` | Comboio (Rail) |
| `3` | Autocarro (Bus) |
| `4` | Ferry |
| `7` | Funicular |

### 3.4 `trips.txt` — As Viagens

Define cada viagem concreta de um veículo.

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `route_id` | ID | Sim | A que rota pertence esta viagem |
| `service_id` | ID | Sim | Quando esta viagem opera (liga ao calendário) |
| `trip_id` | ID | Sim | Identificador único da viagem |
| `direction_id` | Inteiro | Não | 0=ida, 1=volta |
| `shape_id` | ID | Não | Geometria da rota |

### 3.5 `stop_times.txt` — Os Horários

Define a hora de passagem de cada viagem em cada paragem.

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `trip_id` | ID | Sim | A que viagem pertence |
| `arrival_time` | Hora | Sim | Hora de chegada (`HH:MM:SS`) |
| `departure_time` | Hora | Sim | Hora de partida (`HH:MM:SS`) |
| `stop_id` | ID | Sim | Em que paragem |
| `stop_sequence` | Inteiro | Sim | Ordem de passagem (crescente) |

> Este é tipicamente o **ficheiro mais pesado** do feed — a Operadora Exemplo Linha 200 tem ~10.000 registos só para `stop_times.txt`.

### 3.6 `calendar.txt` / `calendar_dates.txt` — O Calendário

Definem **quando** cada `service_id` está ativo.

**`calendar.txt`** — padrão semanal:

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `service_id` | ID | Sim | Identificador do padrão de serviço |
| `monday`...`sunday` | Inteiro | Sim | 1=ativo, 0=inativo |
| `start_date` | Data | Sim | Data de início do padrão |
| `end_date` | Data | Sim | Data de fim do padrão |

**`calendar_dates.txt`** — exceções data a data:

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `service_id` | ID | Sim | Identificador do serviço |
| `date` | Data | Sim | Data da exceção |
| `exception_type` | Inteiro | Sim | 1=adição, 2=remoção |

> A Operadora Exemplo usa **exclusivamente `calendar_dates.txt`** com `exception_type=1`, listando explicitamente cada data ativa. Esta abordagem é comum entre operadores portugueses. Os dois ficheiros serão explorados em detalhe no Módulo 4.

---

## 4. Ficheiros Recomendados e Opcionais

### 4.1 `feed_info.txt` — Metadados (Recomendado)

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `feed_publisher_name` | Texto | Sim | Nome do publicador |
| `feed_publisher_url` | URL | Sim | Website do publicador |
| `feed_lang` | Código | Sim | Idioma do feed (`pt`) |
| `feed_start_date` | Data | Não | Início da validade |
| `feed_end_date` | Data | Não | Fim da validade |

### 4.2 `shapes.txt` — Geometria (Recomendado)

Define o traçado geográfico das rotas no mapa. Cada shape é uma sequência de pontos GPS. Explorado no Módulo 3.

### 4.3 Outros Ficheiros Opcionais

| Ficheiro | Conteúdo | Módulo |
|----------|----------|--------|
| `fare_attributes.txt` | Atributos tarifários | 7 |
| `fare_rules.txt` | Regras de tarifa | 7 |
| `transfers.txt` | Transferências entre paragens | 5 |
| `pathways.txt` | Percursos pedonais em estações | 5 |
| `levels.txt` | Pisos de estações | 5 |
| `frequencies.txt` | Serviço baseado em frequência | 3 |

---

## 5. Anatomia de um Feed GTFS Real

O feed GTFS da Operadora Exemplo (Linha 200) disponível em `exemplos/rede_exemplo_portugal/gtfs/` contém: [[DATA-01]](#DATA-01)

| Ficheiro | Registos | Descrição |
|----------|----------|-----------|
| `agency.txt` | 1 | Operadora Exemplo |
| `routes.txt` | 1 | Linha 200 |
| `stops.txt` | ~59 | Paragens IDA + VOLTA |
| `trips.txt` | ~340 | Viagens (3 tipos de dia × ~57 viagens × 2 direções) |
| `stop_times.txt` | ~10.000 | Horários de passagem |
| `calendar_dates.txt` | ~900 | Datas ativas por service_id |
| `feed_info.txt` | 1 | Metadados da Operadora Exemplo |

Nos módulos seguintes, vamos explorar cada um destes ficheiros em detalhe, usando os dados reais da Linha 200 como base.

---

## Exemplos

Os ficheiros de exemplo deste módulo estão disponíveis na pasta principal de dados:
- [`exemplos/rede_exemplo_portugal/gtfs/`](../../exemplos/rede_exemplo_portugal/gtfs/) — Feed GTFS completo da Operadora Exemplo Linha 200

---

## Exercícios

- [Exercício 1 — Explorar um Feed GTFS](exercicios/exercicio_01.md)

---

## Referências

| ID | Referência |
|----|-----------|
| <a id="STD-01"></a>[STD-01] | MobilityData. "GTFS Schedule Reference." https://gtfs.org/schedule/reference |
| <a id="BP-01"></a>[BP-01] | MobilityData. "GTFS Schedule Best Practices." https://gtfs.org/documentation/schedule/schedule-best-practices |
| <a id="DATA-01"></a>[DATA-01] | Operadora Exemplo. "Feed GTFS Operadora Exemplo (versão Escolar 228, 2026-04-30)." Feed oficial Operadora Exemplo. |
