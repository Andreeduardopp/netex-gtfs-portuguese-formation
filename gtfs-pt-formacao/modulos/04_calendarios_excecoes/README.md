# Módulo 4 — Calendários e Exceções

## Objetivos de Aprendizagem

Após completar este módulo, o leitor será capaz de:

- Compreender o papel do `service_id` como abstração central do calendário GTFS
- Criar padrões semanais com `calendar.txt` e exceções data a data com `calendar_dates.txt`
- Distinguir as duas abordagens ao calendário: baseada em regras vs baseada em exceções
- Modelar os três tipos de serviço da STCP (dias úteis, sábados, domingos/feriados)
- Aplicar as Best Practices para gestão de calendários em feeds de produção

---

## 1. O Problema do Calendário

### 1.1 O que Precisa de Ser Modelado

Um serviço de transporte público opera em padrões temporais complexos: [[STD-01]](#STD-01)

- **Padrões semanais** — serviço diferente em dias úteis, sábados e domingos
- **Feriados** — serviço de domingo em feriados nacionais (1 Janeiro, 25 Abril, 1 Maio, etc.)
- **Períodos letivos** — horários escolares vs período de férias
- **Eventos especiais** — serviço extra para jogos de futebol, festivais, etc.
- **Obras e perturbações** — supressão ou desvio temporário de serviço

O GTFS oferece dois mecanismos complementares para modelar esta complexidade.

### 1.2 O `service_id` — A Abstração Central

O `service_id` é o identificador que liga viagens (`trips.txt`) a padrões de calendário. Não é um atributo do calendário em si, é uma abstração que desacopla a definição da viagem da definição de quando opera. [[STD-01]](#STD-01)

```
trips.txt:
  trip_id=200_DU_0600  →  service_id=DIAS_UTEIS
  trip_id=200_SAB_0600 →  service_id=SABADOS
  trip_id=200_DF_0600  →  service_id=DOMINGOS_FERIADOS

calendar.txt / calendar_dates.txt:
  DIAS_UTEIS → opera segunda a sexta, exceto feriados
  SABADOS → opera aos sábados
  DOMINGOS_FERIADOS → opera aos domingos e feriados nacionais
```

Esta separação permite que muitas viagens partilhem o mesmo `service_id` sem redundância.

---

## 2. `calendar.txt` — Padrões Semanais

### 2.1 Estrutura

O `calendar.txt` define padrões regulares baseados nos dias da semana: [[STD-01]](#STD-01)

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `service_id` | ID | Sim | Identificador do padrão de serviço |
| `monday` | Inteiro | Sim | 1=ativo, 0=inativo |
| `tuesday` | Inteiro | Sim | 1=ativo, 0=inativo |
| `wednesday` | Inteiro | Sim | 1=ativo, 0=inativo |
| `thursday` | Inteiro | Sim | 1=ativo, 0=inativo |
| `friday` | Inteiro | Sim | 1=ativo, 0=inativo |
| `saturday` | Inteiro | Sim | 1=ativo, 0=inativo |
| `sunday` | Inteiro | Sim | 1=ativo, 0=inativo |
| `start_date` | Data | Sim | Início do período de validade (`YYYYMMDD`) |
| `end_date` | Data | Sim | Fim do período de validade (`YYYYMMDD`) |

### 2.2 Exemplo: Três Tipos de Serviço

```csv
service_id,monday,tuesday,wednesday,thursday,friday,saturday,sunday,start_date,end_date
DIAS_UTEIS,1,1,1,1,1,0,0,20260430,20261231
SABADOS,0,0,0,0,0,1,0,20260430,20261231
DOMINGOS_FERIADOS,0,0,0,0,0,0,1,20260430,20261231
```

Este exemplo define três padrões:
- `DIAS_UTEIS` — ativo de segunda a sexta
- `SABADOS` — ativo apenas ao sábado
- `DOMINGOS_FERIADOS` — ativo apenas ao domingo

> **Problema**: este padrão não captura feriados. O 25 de Abril (sábado em 2026) ou o 1 de Maio (sexta em 2026) precisam de tratamento especial. É aqui que entra o `calendar_dates.txt`.

---

## 3. `calendar_dates.txt` — Exceções Data a Data

### 3.1 Estrutura

O `calendar_dates.txt` define exceções, adições ou remoções de serviço em datas específicas: [[STD-01]](#STD-01)

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `service_id` | ID | Sim | Identificador do serviço |
| `date` | Data | Sim | Data da exceção (`YYYYMMDD`) |
| `exception_type` | Inteiro | Sim | 1=adição, 2=remoção |

### 3.2 Dois Tipos de Exceção

| `exception_type` | Significado | Caso de uso |
|-------------------|-------------|-------------|
| `1` (adição) | O serviço opera nesta data mesmo que o `calendar.txt` diga que não | Feriado com serviço especial |
| `2` (remoção) | O serviço não opera nesta data mesmo que o `calendar.txt` diga que sim | Dia útil que é feriado |

### 3.3 Exemplo:

Combinando `calendar.txt` com `calendar_dates.txt` para tratar feriados:

```csv
service_id,date,exception_type
DIAS_UTEIS,20260501,2
DOMINGOS_FERIADOS,20260501,1
DIAS_UTEIS,20260610,2
DOMINGOS_FERIADOS,20260610,1
```

Leitura:
- **1 de Maio (sexta-feira, Dia do Trabalhador)**: remover serviço de `DIAS_UTEIS` (exception_type=2), adicionar serviço de `DOMINGOS_FERIADOS` (exception_type=1)
- **10 de Junho (quarta-feira, Dia de Portugal)**: mesma lógica

---

## 4. A Abordagem da STCP: Só `calendar_dates.txt`

### 4.1 A Abordagem Baseada em Exceções

A STCP usa exclusivamente `calendar_dates.txt` com `exception_type=1`. Não existe `calendar.txt` no feed. [[DATA-01]](#DATA-01)

Isto significa que o feed lista explicitamente cada data em que cada `service_id` está ativo:

```csv
service_id,date,exception_type
DIAS UTEIS,20260430,1
DIAS UTEIS,20260504,1
DIAS UTEIS,20260505,1
DIAS UTEIS,20260506,1
...
SÁBADOS,20260502,1
SÁBADOS,20260509,1
...
DOMINGOS|FERIADOS,20260501,1
DOMINGOS|FERIADOS,20260503,1
...
```

### 4.2 Porque Esta Abordagem?

| Vantagem | Desvantagem |
|----------|-------------|
| Sem ambiguidade — cada data é explícita | Ficheiro maior (centenas de linhas) |
| Feriados e exceções tratados naturalmente | Mais difícil de ler manualmente |
| Simples de gerar programaticamente | Sem padrão semanal visível |
| Não requer `calendar.txt` | Não é auto-documentado |

Esta abordagem é comum entre operadores que geram feeds programaticamente a partir de software de planeamento.

### 4.3 Os Três `service_id` da STCP

| `service_id` GTFS | Descrição | Dias típicos |
|---|---|---|
| `DIAS UTEIS` | Dias úteis | Segunda a sexta (exceto feriados) |
| `SÁBADOS` | Sábados | Sábados |
| `DOMINGOS\|FERIADOS` | Domingos e feriados | Domingos + feriados nacionais |

> **Nota sobre os IDs reais da STCP**: os `service_id` do feed real diferem das boas práticas por razões históricas:
> - `DIAS UTEIS` contém um espaço (Best Practices recomendam `DIAS_UTEIS`)
> - `SÁBADOS` contém um acento (recomendado: `SABADOS`)
> - `DOMINGOS|FERIADOS` contém um pipe (recomendado: `DOMINGOS_FERIADOS`)
>
> Em feeds novos, use sempre IDs alfanuméricos simples (`[a-zA-Z0-9_-]`). [[BP-01]](#BP-01)

---

## 5. Quando utilizar `calendar.txt` e `calendar_dates.txt`

### 5.1 Três Estratégias Possíveis

| Estratégia | `calendar.txt` | `calendar_dates.txt` | Quando usar |
|-----------|----------------|----------------------|-------------|
| **Só regras** | Padrão semanal | Não usado | Serviço muito regular, sem exceções |
| **Regras + exceções** | Padrão semanal | Exceções (feriados, etc.) | Caso mais comum e recomendado |
| **Só exceções** | Não usado | Todas as datas (type=1) | Geração programática, serviço irregular |

### 5.2 As Best Practices Recomendam

1. Incluir ambos os ficheiros para legibilidade humana, `calendar.txt` para o padrão base, `calendar_dates.txt` para exceções
2. Usar `service_id` descritivos (ex: `WEEKDAY_SUMMER_2026` em vez de `SVC_001`)
3. Remover `service_id` expirados ou não utilizados do feed antes de publicar
4. Garantir que as datas `start_date` e `end_date` em `calendar.txt` cobrem todo o período de validade do feed


---

## 6. Gestão Temporal do Feed

### 6.1 Datas de Validade

Um feed GTFS bem gerido deve ter datas de validade claras: [[BP-01]](#BP-01) [[BP-02]](#BP-02)

- O feed deve ser válido para pelo menos os próximos 7 dias
- Idealmente, deve cobrir 30 dias ou todo o período de serviço
- Novos feeds devem ser publicados pelo menos 7 dias antes de alterações de horário entrarem em vigor

### 6.2 Publicação e Atualização

A frequência de atualização varia por operador, mas tipicamente está ligada a mudanças sazonais de horário (2-4 vezes por ano). Para alterações menores dentro de 7 dias, as GTFS Best Practices recomendam usar GTFS Realtime em vez de atualizar o feed estático. [[BP-01]](#BP-01)

---

## Exemplos

- [`exemplos/04_01_calendar_semanal.txt`](exemplos/04_01_calendar_semanal.txt) — Padrão semanal com 3 tipos de serviço usando `calendar.txt`
- [`exemplos/04_02_calendar_dates_stcp.txt`](exemplos/04_02_calendar_dates_stcp.txt) — Abordagem da STCP: só `calendar_dates.txt` com exception_type=1

---

## Exercícios

- [Exercício 4 — Modelar o Calendário de um Operador](exercicios/exercicio_04.md)

---

## Referências

| ID | Referência |
|----|-----------|
| <a id="STD-01"></a>[STD-01] | MobilityData. "GTFS Schedule Reference." https://gtfs.org/schedule/reference |
| <a id="BP-01"></a>[BP-01] | MobilityData. "GTFS Schedule Best Practices." https://gtfs.org/documentation/schedule/schedule-best-practices |
| <a id="BP-02"></a>[BP-02] | Transit App. "Guidelines for Producing GTFS Static Data." https://resources.transitapp.com/article/458-guidelines-for-producing-gtfs-static-data-for-transit |
| <a id="DATA-01"></a>[DATA-01] | STCP. "Feed GTFS STCP Porto (versão Escolar 228, 2026-04-30)." Feed oficial STCP. |
