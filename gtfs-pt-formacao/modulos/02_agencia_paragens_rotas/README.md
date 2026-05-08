# Módulo 2 — Agência, Paragens e Rotas

## Objetivos de Aprendizagem

Após completar este módulo, o leitor será capaz de:

- Configurar `agency.txt` com todos os campos obrigatórios e recomendados
- Modelar paragens em `stops.txt` com hierarquia `parent_station` e `location_type`
- Definir rotas em `routes.txt` com códigos `route_type` corretos
- Compreender as relações entre os três ficheiros
- Aplicar as Best Practices para coordenadas, IDs persistentes e nomes descritivos

---

## 1. `agency.txt` — O Operador

### 1.1 Campos Completos

O `agency.txt` identifica a(s) empresa(s) que operam o serviço: [[STD-01]](#STD-01)

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `agency_id` | ID | Condicional* | Identificador único do operador |
| `agency_name` | Texto | Sim | Nome oficial do operador |
| `agency_url` | URL | Sim | Website do operador |
| `agency_timezone` | Timezone | Sim | Fuso horário IANA |
| `agency_lang` | Código ISO 639-1 | Não | Idioma principal (`pt`) |
| `agency_phone` | Texto | Não | Telefone de contacto |
| `agency_fare_url` | URL | Não | URL da página de tarifas |
| `agency_email` | Email | Não | Email de contacto |

*`agency_id` é obrigatório quando o feed contém dados de múltiplos operadores.

### 1.2 STCP — Exemplo

```csv
agency_id,agency_name,agency_url,agency_timezone,agency_lang
STCP,STCP,https://www.stcp.pt,Europe/Lisbon,pt
```

Ver ficheiro: [`exemplos/02_01_agency_stcp.txt`](exemplos/02_01_agency_stcp.txt)

### 1.3 Feeds Multi-Operador

Alguns feeds agregam dados de múltiplos operadores. Nesse caso, `agency_id` é obrigatório e cada rota em `routes.txt` deve referenciar o seu operador:

```csv
agency_id,agency_name,agency_url,agency_timezone,agency_lang
STCP,STCP,https://www.stcp.pt,Europe/Lisbon,pt
MetroPorto,Metro do Porto,https://www.metrodoporto.pt,Europe/Lisbon,pt
CP,Comboios de Portugal,https://www.cp.pt,Europe/Lisbon,pt
```

O NAP Portugal poderá, no futuro, publicar feeds agregados com múltiplos operadores da mesma região metropolitana. [[PT-01]](#PT-01)

---

## 2. `stops.txt` — As Paragens

### 2.1 Campos Completos

O `stops.txt` define todos os locais de embarque e desembarque: [[STD-01]](#STD-01)

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `stop_id` | ID | Sim | Identificador único |
| `stop_code` | Texto | Não | Código público visível na paragem |
| `stop_name` | Texto | Condicional* | Nome da paragem |
| `stop_desc` | Texto | Não | Descrição adicional |
| `stop_lat` | Latitude | Condicional* | Latitude (WGS 84) |
| `stop_lon` | Longitude | Condicional* | Longitude (WGS 84) |
| `zone_id` | ID | Não | Zona tarifária |
| `stop_url` | URL | Não | URL com informação da paragem |
| `location_type` | Inteiro | Não | Tipo de local (ver 2.2) |
| `parent_station` | ID | Não | Referência à estação-pai |
| `stop_timezone` | Timezone | Não | Fuso horário (se diferente da agência) |
| `wheelchair_boarding` | Inteiro | Não | 0=desconhecido, 1=sim, 2=não |
| `platform_code` | Texto | Não | Código da plataforma |

*`stop_name`, `stop_lat` e `stop_lon` são obrigatórios para `location_type` 0, 1 e 2 (paragens, estações, entradas/saídas). Opcionais apenas para tipos 3 e 4 (nós genéricos, zonas de embarque).

### 2.2 Hierarquia location_type

O campo `location_type` define o tipo de local na hierarquia física: [[STD-01]](#STD-01)

| Valor | Tipo | Descrição | Exemplo Porto |
|-------|------|-----------|---------------|
| `0` (ou vazio) | Paragem | Ponto de embarque/desembarque | Bolhão (paragem de autocarro) |
| `1` | Estação | Agrupa múltiplas paragens | Estação de Campanhã |
| `2` | Entrada/saída | Porta de acesso à estação | Entrada norte de Campanhã |
| `3` | Nó genérico | Ponto de decisão dentro da estação | Cruzamento de corredores |
| `4` | Zona de embarque | Posição exata de embarque | Plataforma 2A |

A maioria das paragens de autocarro em Portugal usa `location_type=0` (ou campo vazio, que é equivalente). A hierarquia completa é mais relevante para estações de metro e comboio.

### 2.3 parent_station — Agrupar Paragens

Quando uma estação tem múltiplas plataformas, cada plataforma é uma paragem (`location_type=0`) e a estação é o `parent_station` (`location_type=1`):

```csv
stop_id,stop_name,stop_lat,stop_lon,location_type,parent_station
EST_CAMP,Estação de Campanhã,41.148904,-8.585407,1,
CAMP_P1,Campanhã — Plataforma 1,41.148950,-8.585300,0,EST_CAMP
CAMP_P2,Campanhã — Plataforma 2,41.148860,-8.585500,0,EST_CAMP
```

### 2.4 Paragens da Linha 200 (Direção Castelo do Queijo)

Primeiras 10 paragens extraídas do feed STCP: [[DATA-01]](#DATA-01)

| Seq | `stop_id` | `stop_name` | `stop_lat` | `stop_lon` | `zone_id` |
|-----|-----------|-------------|-----------|-----------|-----------|
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

Ver ficheiro completo: [`exemplos/02_02_stops_200.txt`](exemplos/02_02_stops_200.txt)

A Linha 200 tem 30 paragens na direção IDA (Bolhão → Castelo do Queijo) e 31 na direção VOLTA. Todas são paragens simples (`location_type=0`) sem `parent_station`.

### 2.5 Precisão das Coordenadas

As boas práticas especificam que as coordenadas devem ter um erro máximo de 4 metros em relação à posição real da paragem, e devem ser colocadas junto ao passeio/cais de embarque onde o passageiro espera. [[BP-01]](#BP-01)

```
Correto: Coordenadas no passeio junto ao poste de paragem
Errado: Coordenadas no centro da estrada ou no edifício adjacente
```

---

## 3. `routes.txt` — As Rotas

### 3.1 Campos Completos

O `routes.txt` define as linhas de transporte: [[STD-01]](#STD-01)

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `route_id` | ID | Sim | Identificador único |
| `agency_id` | ID | Condicional | Referência ao operador |
| `route_short_name` | Texto | Condicional* | Código curto (`200`) |
| `route_long_name` | Texto | Condicional* | Nome completo |
| `route_desc` | Texto | Não | Descrição |
| `route_type` | Inteiro | Sim | Modo de transporte |
| `route_url` | URL | Não | URL da página da rota |
| `route_color` | Cor hex | Não | Cor de fundo (sem `#`) |
| `route_text_color` | Cor hex | Não | Cor do texto |
| `route_sort_order` | Inteiro | Não | Ordem de apresentação |

*Pelo menos um de `route_short_name` ou `route_long_name` é obrigatório.

### 3.2 Códigos `route_type`

Os códigos mais relevantes para operadores portugueses: [[STD-01]](#STD-01)

| Código | Modo | Operadores PT |
|--------|------|---------------|
| `0` | Eléctrico / Tram | Carris (elétricos históricos) |
| `1` | Metro | Metro de Lisboa, Metro do Porto |
| `2` | Comboio / Rail | CP, Fertagus |
| `3` | Autocarro / Bus | STCP, Carris Metropolitana, Transdev |
| `4` | Ferry | Transtejo/Soflusa |
| `7` | Funicular | Elevador da Glória, Elevador da Bica |
| `11` | Trolleybus | — (não existe atualmente em PT) |
| `12` | Monorail | — (não existe atualmente em PT) |

### 3.3 STCP Linha 200 

```csv
route_id,agency_id,route_short_name,route_long_name,route_type,route_color,route_text_color
200,STCP,200,Bolhão - Castelo do Queijo,3,187EC2,FFFFFF
```

Ver ficheiro: [`exemplos/02_03_routes_200.txt`](exemplos/02_03_routes_200.txt)

Leitura campo a campo:
- `route_id=200` — identificador da rota
- `agency_id=STCP` — referência ao operador em `agency.txt`
- `route_short_name=200` — código que o passageiro vê no autocarro
- `route_long_name=Bolhão - Castelo do Queijo` — nome completo com terminais
- `route_type=3` — autocarro
- `route_color=187EC2` — azul (cor da Linha 200 no mapa STCP)
- `route_text_color=FFFFFF` — texto branco sobre fundo azul

---

## 4. Relações Entre os Três Ficheiros

A ligação entre os três ficheiros é feita pelo `agency_id`:

```
agency.txt                routes.txt
┌──────────────┐          ┌───────────────────────┐
│agency_id=STCP│ ◄─────── │ agency_id=STCP        │
│agency_name=  │          │ route_id=200          │
│  STCP        │          │ route_short_name=200  │
└──────────────┘          └───────────────────────┘

stops.txt (independente nesta fase — ligado via stop_times.txt no Módulo 3)
┌────────────────────────┐
│ stop_id=BLRB1          │
│ stop_name=Bolhão       │
│ stop_lat=41.151868     │
│ stop_lon=-8.607115     │
└────────────────────────┘
```

> **Nota**: `stops.txt` não referencia diretamente `routes.txt`. A ligação entre paragens e rotas é estabelecida **indiretamente** através de `stop_times.txt` → `trips.txt` → `routes.txt`.

---

## 5. Best Practices

### 5.1 Identificadores Persistentes

Os `stop_id`, `route_id` e `agency_id` devem permanecer consistentes entre atualizações do feed. [[BP-01]](#BP-01) Alterar identificadores quebra integrações com aplicações que dependem de IDs estáveis para GTFS Realtime, histórico de performance, e favoritos dos utilizadores.

### 5.2 Nomes de Paragem

As Best Practices recomendam: [[BP-01]](#BP-01)

- Usar o nome real visível na paragem (poste, abrigo)
- Evitar abreviaturas quando possível (`Hospital Santo António` em vez de `Hosp. St. António`)
- Não incluir o nome da rota no nome da paragem
- Não incluir prefixos genéricos como "Paragem" ou "Estação" quando o nome é suficiente

### 5.3 Cores de Rota

As cores `route_color` e `route_text_color` devem garantir contraste suficiente para legibilidade. [[BP-01]](#BP-01) Usar as cores oficiais do operador quando disponíveis.

---

## Exemplos

- [`exemplos/02_01_agency_stcp.txt`](exemplos/02_01_agency_stcp.txt) — `agency.txt` da STCP
- [`exemplos/02_02_stops_200.txt`](exemplos/02_02_stops_200.txt) — Primeiras 10 paragens da Linha 200 (direção Castelo do Queijo)
- [`exemplos/02_03_routes_200.txt`](exemplos/02_03_routes_200.txt) — Rota da Linha 200

---

## Exercícios

- [Exercício 2 — Criar Agency, Stops e Routes para um Operador](exercicios/exercicio_02.md)

---

## Referências

| ID | Referência |
|----|-----------|
| <a id="STD-01"></a>[STD-01] | MobilityData. "GTFS Schedule Reference." https://gtfs.org/schedule/reference |
| <a id="BP-01"></a>[BP-01] | MobilityData. "GTFS Schedule Best Practices." https://gtfs.org/documentation/schedule/schedule-best-practices |
| <a id="PT-01"></a>[PT-01] | IMT. "Ponto de Acesso Nacional (NAP) Portugal." https://nap-portugal.imt-ip.pt |
| <a id="DATA-01"></a>[DATA-01] | STCP. "Feed GTFS STCP Porto (versão Escolar 228, 2026-04-30)." Feed oficial STCP. |
