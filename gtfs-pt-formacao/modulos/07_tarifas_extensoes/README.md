# Módulo 7 — Tarifas e Extensões

## Objetivos de Aprendizagem

Após completar este módulo, o leitor será capaz de:

- Compreender a evolução do modelo tarifário no GTFS, do Fares v1 ao Fares v2
- Modelar tarifas baseadas em zonas usando `fare_attributes.txt` e `fare_rules.txt` (v1)
- Descrever os ficheiros e conceitos do Fares v2 (`fare_products`, `fare_media`, `fare_leg_rules`, `rider_categories`)
- Identificar as extensões GTFS-Flex e GTFS-ride, compreendendo os seus casos de uso em Portugal
- Relacionar os conceitos tarifários GTFS com os equivalentes NeTEx (FareFrame, FareProduct)

---

## 1. Tarifas no GTFS — Visão Geral

### 1.1 Evolução das Tarifas no GTFS

A modelação de tarifas no GTFS passou por duas fases distintas: [[STD-01]](#STD-01)

1. **Fares v1 (modelo original)** — dois ficheiros simples (`fare_attributes.txt` e `fare_rules.txt`) que cobrem cenários básicos como tarifas fixas e tarifas por zona
2. **Fares v2 (modelo moderno)** — um conjunto alargado de ficheiros que permite modelar cenários complexos como tarifas por distância, títulos multimodais, descontos por categoria e *fare capping*

O v1 continua a ser amplamente utilizado em feeds de produção, mas apresenta limitações significativas para sistemas tarifários complexos como o Andante do Porto. O v2, embora mais recente e ainda em evolução, resolve a maioria dessas limitações. [[STD-05]](#STD-05)

### 1.2 Porquê Estudar Ambos?

- Muitos feeds em produção (incluindo o da STCP) usam apenas v1 [[DATA-01]](#DATA-01)
- O v2 está a ser adotado progressivamente e é recomendado para novos feeds [[BP-01]](#BP-01)
- Compreender o v1 ajuda a entender as motivações por trás do design do v2

---

## 2. Fares v1 — O Modelo Original

### 2.1 `fare_attributes.txt`

O `fare_attributes.txt` define as propriedades de cada tarifa disponível no sistema: [[STD-01]](#STD-01)

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `fare_id` | ID | Sim | Identificador único da tarifa |
| `price` | Float | Sim | Preço da tarifa (ex: `1.55`) |
| `currency_type` | Código ISO 4217 | Sim | Moeda (ex: `EUR`) |
| `payment_method` | Inteiro | Sim | 0=a bordo, 1=antes de embarcar |
| `transfers` | Inteiro | Sim | Número de transbordos permitidos (vazio=ilimitados) |
| `transfer_duration` | Inteiro | Não | Tempo máximo de transbordo em segundos |

### 2.2 `fare_rules.txt`

O `fare_rules.txt` associa tarifas a rotas e/ou zonas: [[STD-01]](#STD-01)

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `fare_id` | ID referenciando `fare_attributes.fare_id` | Sim | Tarifa aplicável |
| `route_id` | ID referenciando `routes.route_id` | Não | Rota onde a tarifa se aplica |
| `origin_id` | ID referenciando `stops.zone_id` | Não | Zona de origem |
| `destination_id` | ID referenciando `stops.zone_id` | Não | Zona de destino |
| `contains_id` | ID referenciando `stops.zone_id` | Não | Zona que a viagem deve atravessar |

### 2.3 Exemplo: Andante Porto (Simplificado)

O sistema Andante da Área Metropolitana do Porto utiliza tarifas baseadas em zonas. Os títulos ocasionais (títulos unitários carregados no cartão Andante) têm preços que variam conforme o número de zonas atravessadas: [[DATA-01]](#DATA-01)

| Título | Zonas | Preço (2026) | Transbordos | Duração |
|--------|-------|-------------|-------------|---------|
| Andante Z2 | 2 zonas | 1,55 EUR | Até 3 | 1 hora |
| Andante Z3 | 3 zonas | 2,00 EUR | Até 3 | 1 hora |
| Andante Z4 | 4 zonas | 2,50 EUR | Até 3 | 1 hora |

> **Nota**: Os preços são ilustrativos e baseados na estrutura tarifária do Andante. Consultar o website do TIP (Transportes Intermodais do Porto) para valores atuais.

**`fare_attributes.txt`**:

```csv
fare_id,price,currency_type,payment_method,transfers,transfer_duration
ANDANTE_Z2,1.55,EUR,0,3,3600
ANDANTE_Z3,2.00,EUR,0,3,3600
ANDANTE_Z4,2.50,EUR,0,3,3600
```

**`fare_rules.txt`**:

```csv
fare_id,origin_id,destination_id
ANDANTE_Z2,Z_PRT1,Z_PRT1
ANDANTE_Z3,Z_PRT1,Z_PRT2
ANDANTE_Z3,Z_PRT2,Z_PRT1
ANDANTE_Z4,Z_PRT1,Z_PRT3
ANDANTE_Z4,Z_PRT3,Z_PRT1
```

Neste modelo simplificado, `Z_PRT1`, `Z_PRT2` e `Z_PRT3` representam zonas do sistema Andante. O `payment_method=0` indica que a validação pode ser feita a bordo (nos autocarros, por exemplo). Os 3 transbordos com duração de 3600 segundos (1 hora) refletem a política de transbordo livre dentro do período de validade do título.

Ver ficheiros: [`exemplos/07_01_fare_attributes.txt`](exemplos/07_01_fare_attributes.txt), [`exemplos/07_02_fare_rules.txt`](exemplos/07_02_fare_rules.txt)

### 2.4 Limitações do Fares v1

O modelo v1 não consegue representar: [[STD-05]](#STD-05)

- **Tarifas por distância percorrida** — apenas zona-a-zona ou rota fixa
- **Suportes tarifários diferentes** — o mesmo título no cartão Andante, na app ou em papel tem o mesmo preço, mas o v1 não distingue suportes
- **Categorias de passageiro** — não há como modelar descontos para crianças, idosos ou estudantes
- **Fare capping** — limites diários/semanais de gasto (ex: "nunca paga mais de X EUR por dia")
- **Tarifas multimodais complexas** — combinação de autocarro + metro com regras específicas
- **Transferências condicionais** — regras que dependem do modo, operador ou tempo

---

## 3. Fares v2 

### 3.1 Porque o v2?

O Fares v2 nasceu da necessidade de representar a complexidade real dos sistemas tarifários modernos. Sistemas como o Andante, que combinam múltiplos modos de transporte, categorias de passageiro e suportes tarifários, não podiam ser adequadamente representados no v1. [[STD-05]](#STD-05)

### 3.2 Ficheiros Principais

O Fares v2 introduz vários ficheiros novos: [[STD-05]](#STD-05)

| Ficheiro | Descrição |
|----------|-----------|
| `fare_products.txt` | Define os produtos tarifários (título Z2, passe mensal, etc.) |
| `fare_media.txt` | Define os suportes (cartão Andante, app, bilhete papel) |
| `fare_leg_rules.txt` | Associa produtos tarifários a segmentos de viagem |
| `fare_transfer_rules.txt` | Define regras de transbordo entre produtos/segmentos |
| `rider_categories.txt` | Define categorias de passageiro (adulto, criança, sénior) |
| `fare_capping.txt` | Define limites de gasto por período |
| `areas.txt` | Define áreas geográficas para regras tarifárias |
| `stop_areas.txt` | Associa paragens a áreas |

### 3.3 Conceitos-Chave

#### Suportes Tarifários (`fare_media`)

O conceito de suporte tarifário é central no v2. No caso do Porto: [[STD-05]](#STD-05)

| `fare_media_id` | `fare_media_name` | `fare_media_type` | Exemplo Porto |
|------------------|-------------------|-------------------|---------------|
| `ANDANTE_CARD` | Cartão Andante | 2 (cEMV / smartcard) | Cartão Andante físico |
| `ANDA_APP` | App ANDA | 4 (app móvel) | Aplicação ANDA Porto |
| `PAPER_TICKET` | Bilhete papel | 0 (papel) | Bilhete unitário (turistas) |

#### Categorias de Passageiro (`rider_categories`)

```
rider_category_id,rider_category_name,min_age,max_age
ADULT,Adulto,18,64
CHILD,Criança (4-12),4,12
SENIOR,Sénior (65+),65,
STUDENT,Estudante,,
```

#### Produtos Tarifários (`fare_products`)

Cada produto pode ser associado a um suporte e a uma categoria:

```
fare_product_id,fare_product_name,fare_media_id,rider_category_id,amount,currency
ANDANTE_Z2_ADULT,Andante Z2 Adulto,ANDANTE_CARD,ADULT,1.55,EUR
ANDANTE_Z2_CHILD,Andante Z2 Criança,ANDANTE_CARD,CHILD,0.80,EUR
PASSE_MENSAL_Z2,Passe Mensal Z2,ANDANTE_CARD,ADULT,30.00,EUR
```

### 3.4 Vantagem do v2: Exemplo Comparativo

**Cenário**: Modelar a tarifa Z2 para adulto e criança, com validação no cartão e na app.

No **v1**, seria necessário criar entradas separadas em `fare_attributes.txt` sem poder distinguir categorias nem suportes, uma limitação que obriga a soluções alternativas fora do standard.

No **v2**, cada combinação produto + suporte + categoria é explicitamente modelada, permitindo que as aplicações de planeamento de viagem mostrem o preço correto para cada utilizador.

---

## 4. GTFS-Flex — Transporte a Pedido

### 4.1 O que é

O GTFS-Flex é uma extensão do GTFS que permite modelar serviços de transporte flexível, onde pelo menos uma parte da viagem não segue um percurso ou horário fixo. Isto inclui: [[STD-06]](#STD-06)

- **Transporte a pedido** (*demand-responsive transport*, DRT) — o veículo só opera se houver pedido
- **Desvio de rota** (*route deviation*) — o veículo segue um percurso base mas pode desviar-se para recolher/deixar passageiros
- **Paragens por pedido** (*flag stops*) — o veículo para em qualquer ponto do percurso, não apenas em paragens fixas
- **Zonas de serviço** — áreas geográficas onde o serviço opera sem paragens definidas

### 4.2 Ficheiros

O GTFS-Flex introduz os seguintes ficheiros e campos adicionais: [[STD-06]](#STD-06)

| Ficheiro / Campo | Descrição |
|-------------------|-----------|
| `booking_rules.txt` | Regras de reserva (antecedência mínima, contacto, URL) |
| `location_groups.txt` | Agrupamentos de paragens ou áreas de serviço |
| `location_group_stops.txt` | Associa paragens a grupos de localização |
| `stop_times.txt` — campos adicionais | `start_pickup_drop_off_window`, `end_pickup_drop_off_window`, `pickup_booking_rule_id`, `drop_off_booking_rule_id` |

**Exemplo de `booking_rules.txt`**:

```csv
booking_rule_id,booking_type,prior_notice_duration_min,message,phone_number,info_url
DRT_RULE_01,1,60,Reserva com 1h de antecedência,+351 22 XXX XXXX,https://exemplo.pt/reserva
```

- `booking_type=1` — reserva no mesmo dia com antecedência mínima
- `prior_notice_duration_min=60` — pelo menos 60 minutos antes

### 4.3 Relevância para Portugal

O GTFS-Flex é particularmente relevante para o contexto português por causa dos serviços de transporte a pedido que servem zonas rurais e de baixa densidade: [[STD-06]](#STD-06)

- **Interior de Portugal** — muitas comunidades intermunicipais (CIM) operam serviços DRT para ligar aldeias a sedes de concelho
- **Complemento ao transporte regular** — horários noturnos ou de fim de semana com procura insuficiente para justificar serviço fixo
- **Transporte escolar flexível** — serviços que operam apenas em período letivo com percursos variáveis
- **Áreas metropolitanas** — serviços de "última milha" entre estações de metro/comboio e zonas residenciais

> **Nota**: Em Portugal, muitos destes serviços ainda não estão modelados em GTFS. O GTFS-Flex oferece o framework para o fazer, contribuindo para a completude dos dados no NAP (Ponto de Acesso Nacional). [[PT-01]](#PT-01)

---

## 5. GTFS-ride — Dados de Ridership

### 5.1 O que é

O GTFS-ride é uma extensão que permite associar dados de utilização real (contagem de passageiros) às viagens definidas no GTFS Schedule. Foi desenvolvido para facilitar a partilha de dados de ridership entre operadores, agências de financiamento e investigadores. [[STD-04]](#STD-04)

Os principais ficheiros são:

| Ficheiro | Descrição |
|----------|-----------|
| `board_alight.txt` | Contagens de embarques e desembarques por paragem e viagem |
| `rider_trip.txt` | Informação agregada sobre passageiros por viagem |
| `ridership.txt` | Resumos de utilização por rota, paragem ou período |

### 5.2 `board_alight.txt`

O ficheiro central do GTFS-ride regista, para cada paragem de cada viagem, quantos passageiros embarcaram e desembarcaram: [[STD-04]](#STD-04)

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `trip_id` | ID | Viagem (referência a `trips.txt`) |
| `stop_id` | ID | Paragem (referência a `stops.txt`) |
| `stop_sequence` | Inteiro | Ordem da paragem na viagem |
| `record_use` | Inteiro | 0=entrada, 1=saída |
| `schedule_relationship` | Inteiro | 0=programada, 1=adicionada, 2=cancelada |
| `boardings` | Inteiro | Número de embarques |
| `alightings` | Inteiro | Número de desembarques |
| `current_load` | Inteiro | Ocupação atual do veículo |

### 5.3 Exemplo

```csv
trip_id,stop_id,stop_sequence,record_use,schedule_relationship,boardings,alightings,current_load
200_DU_0700,BOLS1,1,0,0,12,0,12
200_DU_0700,CLRG2,2,0,0,8,2,18
200_DU_0700,ALID3,3,0,0,5,6,17
200_DU_0700,CAST4,4,0,0,3,10,10
200_DU_0700,HOSP5,5,0,0,1,8,3
200_DU_0700,SJOA6,6,0,0,0,3,0
```

Este exemplo mostra a viagem `200_DU_0700` (Linha 200, dias úteis, partida às 07:00). O veículo começa vazio em Bolhão (`BOLS1`), atinge o pico de ocupação na Praça da República (`CLRG2`) com 18 passageiros, e termina vazio em São João (`SJOA6`).

Ver ficheiro: [`exemplos/07_03_board_alight.txt`](exemplos/07_03_board_alight.txt)

### 5.4 Utilidade dos Dados de Ridership

Os dados de `board_alight.txt` permitem:

- **Planeamento** — identificar paragens com maior procura para otimizar frequências
- **Financiamento** — justificar investimentos com dados reais de utilização
- **Acessibilidade** — priorizar melhorias de acessibilidade em paragens com maior fluxo
- **Análise de rede** — detetar padrões de origem-destino e desequilíbrios de oferta/procura

---

## 6. Outras Extensões

### 6.1 GTFS Realtime

O GTFS Realtime complementa o GTFS Schedule (estático) com informação dinâmica: [[STD-07]](#STD-07)

| Entidade | Descrição | Exemplo |
|----------|-----------|---------|
| `TripUpdate` | Atrasos e cancelamentos em tempo real | "Linha 200 com 5 min de atraso" |
| `VehiclePosition` | Posição GPS dos veículos | Localização no mapa em tempo real |
| `Alert` | Alertas de serviço | "Paragem X temporariamente desativada" |

O GTFS Realtime usa Protocol Buffers (não CSV) e é transmitido via feeds HTTP. A maioria dos operadores portugueses ainda não publica GTFS Realtime, embora a STCP e o Metro do Porto tenham sistemas de informação ao passageiro em tempo real.

### 6.2 GTFS-Pathways (Referência ao Módulo 5)

O GTFS-Pathways modela a navegação pedonal dentro de estações e interfaces de transporte, incluindo caminhos, escadas, elevadores e passadeiras rolantes. Este tema foi abordado em detalhe no **Módulo 5 — Acessibilidade e Pathways**. Os `pathways` são particularmente relevantes para estações complexas como a Trindade (Metro do Porto) ou o Campanhã Intermodal.

---

## 7. Ponte NeTEx (Opcional)

Para profissionais que também trabalham com NeTEx (obrigatório para o NAP europeu sob o regulamento MMTIS), é útil compreender os equivalentes dos conceitos tarifários: [[REG-01]](#REG-01)

| Conceito GTFS | Equivalente NeTEx | Notas |
|---------------|-------------------|-------|
| `fare_attributes` / `fare_products` | `FareProduct` dentro de `FareFrame` | O NeTEx tem um modelo muito mais granular |
| `fare_rules` / `fare_leg_rules` | `FareStructureElement`, `DistanceMatrixElement` | Regras de aplicação de tarifas |
| `fare_media` | `TypeOfTravelDocument` | Cartão, app, papel, etc. |
| `rider_categories` | `UserProfile` | Perfis de utilizador com atributos detalhados |
| zonas em `stops.zone_id` | `FareZone` | Zonas tarifárias como entidade própria |
| `fare_capping` | `CappedDiscountRight`, `Capping` | Limites de gasto por período |

> **Nota**: O modelo tarifário NeTEx (parte 3 da norma EN 16614) é significativamente mais complexo que o GTFS, tanto no v1 como no v2. A decisão entre GTFS e NeTEx para modelação de tarifas depende dos requisitos regulatórios e da complexidade do sistema tarifário.

---

## Exemplos

- [`exemplos/07_01_fare_attributes.txt`](exemplos/07_01_fare_attributes.txt) — Tarifas Andante por zona (Fares v1)
- [`exemplos/07_02_fare_rules.txt`](exemplos/07_02_fare_rules.txt) — Regras tarifárias por zona de origem/destino (Fares v1)
- [`exemplos/07_03_board_alight.txt`](exemplos/07_03_board_alight.txt) — Contagens de passageiros por paragem (GTFS-ride)

---

## Exercícios

- [Exercício 7 — Modelar Tarifas e Analisar Dados de Utilização](exercicios/exercicio_07.md)

---

## Referências

| ID | Referência |
|----|-----------|
| <a id="STD-01"></a>[STD-01] | MobilityData. "GTFS Schedule Reference." https://gtfs.org/schedule/reference |
| <a id="STD-04"></a>[STD-04] | GTFS-ride. Extensão para dados de ridership. https://gtfsride.org |
| <a id="STD-05"></a>[STD-05] | MobilityData. "GTFS-Fares v2." https://gtfs.org/extensions/fares-v2 |
| <a id="STD-06"></a>[STD-06] | MobilityData. "GTFS-Flex." https://gtfs.org/extensions/flex |
| <a id="STD-07"></a>[STD-07] | MobilityData. "GTFS Realtime Reference." https://gtfs.org/realtime/reference |
| <a id="BP-01"></a>[BP-01] | MobilityData. "GTFS Schedule Best Practices." https://gtfs.org/documentation/schedule/schedule-best-practices |
| <a id="PT-01"></a>[PT-01] | IMT. "Ponto de Acesso Nacional (NAP) Portugal." https://nap-portugal.imt-ip.pt |
| <a id="REG-01"></a>[REG-01] | Regulamento Delegado (UE) 2024/490 (MMTIS). https://eur-lex.europa.eu/legal-content/PT/TXT/?uri=OJ:L_202400490 |
| <a id="DATA-01"></a>[DATA-01] | STCP. "Feed GTFS STCP Porto (versão Escolar 228, 2026-04-30)." Feed oficial STCP. |
