# Módulo 5 — Acessibilidade e Transferências

## Objetivos de Aprendizagem

Após completar este módulo, o leitor será capaz de:

- Utilizar os campos de acessibilidade em `stops.txt` e `trips.txt` para descrever a acessibilidade física do serviço
- Compreender o mecanismo de herança de `wheelchair_boarding` entre `parent_station` e paragens-filhas
- Modelar transferências entre serviços com `transfers.txt`, distinguindo os quatro tipos de transferência
- Descrever caminhos pedonais dentro de estações com `pathways.txt` e `levels.txt`
- Reconhecer a importância da acessibilidade no contexto da regulação europeia e da inclusão social

---

## 1. Acessibilidade no GTFS

A acessibilidade é um dos aspetos mais impactantes de um feed GTFS. Quando corretamente preenchidos, os campos de acessibilidade permitem que aplicações como Google Maps, Moovit e CityMapper ofereçam rotas adaptadas a utilizadores com mobilidade reduzida, cadeiras de rodas ou necessidades especiais. [[STD-01]](#STD-01)

### 1.1 Campos de Acessibilidade em `stops.txt`

O campo `wheelchair_boarding` em `stops.txt` indica se é possível embarcar numa paragem usando cadeira de rodas: [[STD-01]](#STD-01)

| Valor | Significado |
|-------|-------------|
| `0` (ou vazio) | Sem informação de acessibilidade |
| `1` | Embarque em cadeira de rodas é possível |
| `2` | Embarque em cadeira de rodas **não** é possível |

**Herança de `parent_station`**:

Quando uma paragem (`location_type=0`) está associada a uma estação (`location_type=1`) via `parent_station`, o campo `wheelchair_boarding` segue regras de herança: [[STD-01]](#STD-01)

- Se a **paragem-filha** tem `wheelchair_boarding=0` (ou vazio), herda o valor da estação-pai
- Se a **paragem-filha** tem valor `1` ou `2`, esse valor prevalece sobre o da estação-pai
- Se a **estação-pai** tem `wheelchair_boarding=1`, todas as plataformas sem valor explícito são consideradas acessíveis

Isto é particularmente relevante para estações intermodais como o Bolhão, onde diferentes plataformas podem ter níveis de acessibilidade distintos.

```
Estação Bolhão (parent_station, wheelchair_boarding=1)
  ├── Plataforma Metro Linha D (wheelchair_boarding=0 → herda 1 da estação)
  ├── Plataforma Operadora Exemplo Linha 200 (wheelchair_boarding=1 → explícito)
  └── Acesso provisório obras (wheelchair_boarding=2 → sem acesso cadeira rodas)
```

### 1.2 Campos de Acessibilidade em `trips.txt`

Cada viagem pode também indicar a acessibilidade do veículo que a realiza: [[STD-01]](#STD-01)

**`wheelchair_accessible`**:

| Valor | Significado |
|-------|-------------|
| `0` (ou vazio) | Sem informação |
| `1` | Pelo menos um lugar para cadeira de rodas |
| `2` | Nenhum lugar para cadeira de rodas |

**`bikes_allowed`**:

| Valor | Significado |
|-------|-------------|
| `0` (ou vazio) | Sem informação |
| `1` | Bicicletas permitidas |
| `2` | Bicicletas não permitidas |

Para que um utilizador em cadeira de rodas receba uma rota acessível, ambas as condições devem ser satisfeitas:
1. A paragem deve ter `wheelchair_boarding=1`
2. A viagem deve ter `wheelchair_accessible=1`

Se um dos dois campos for `0` (desconhecido), as aplicações tipicamente não filtram, mas também não garantem acessibilidade. Se um dos dois for `2`, a rota é marcada como inacessível.

### 1.3 Importância para Portugal

A acessibilidade nos transportes públicos é uma obrigação legal e um imperativo social: [[REG-01]](#REG-01) [[DL-01]](#DL-01)

- **Regulamento Delegado (UE) 2024/490** (MMTIS) — exige que os dados de acessibilidade sejam publicados em formatos abertos, incluindo informação sobre acessibilidade em cadeira de rodas nas paragens e nos veículos
- **Decreto-Lei n.º 163/2006** — regulamenta as normas técnicas de acessibilidade em edifícios e espaços públicos em Portugal, incluindo interfaces de transporte
- **Inclusão social** — cerca de 1 milhão de portugueses tem algum tipo de deficiência; informação de acessibilidade fiável é essencial para a sua mobilidade autónoma

Na prática, muitos feeds GTFS portugueses ainda deixam os campos de acessibilidade em `0` (desconhecido), o que equivale a não fornecer informação útil. O preenchimento correto destes campos é uma das melhorias de maior impacto que um operador pode fazer no seu feed.

---

## 2. `transfers.txt` — Transferências entre Serviços

### 2.1 Finalidade

O `transfers.txt` permite definir regras especiais de transferência entre paragens ou rotas, sobrepondo-se ao comportamento por defeito dos planificadores de viagem. Sem este ficheiro, as aplicações calculam as transferências apenas com base na proximidade geográfica e nos tempos de passagem. [[STD-01]](#STD-01)

### 2.2 Campos

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `from_stop_id` | ID | Sim | Paragem de origem da transferência |
| `to_stop_id` | ID | Sim | Paragem de destino da transferência |
| `from_route_id` | ID | Não | Rota de origem (filtragem adicional) |
| `to_route_id` | ID | Não | Rota de destino (filtragem adicional) |
| `from_trip_id` | ID | Não | Viagem de origem (filtragem adicional) |
| `to_trip_id` | ID | Não | Viagem de destino (filtragem adicional) |
| `transfer_type` | Inteiro | Sim | Tipo de transferência (ver abaixo) |
| `min_transfer_time` | Inteiro | Não | Tempo mínimo de transferência (segundos) |

### 2.3 Tipos de Transferência

| `transfer_type` | Significado | Caso de uso |
|------------------|-------------|-------------|
| `0` | Ponto de transferência recomendado | Paragens próximas onde a transferência é fácil |
| `1` | Transferência temporizada (timed) | O veículo de destino espera pelo de origem |
| `2` | Transferência com tempo mínimo | Necessário um tempo mínimo entre chegada e partida |
| `3` | Transferência não possível | Proibir transferência entre estas paragens |

### 2.4 Exemplo: Bolhão (Metro ↔ Operadora Exemplo)

O interface intermodal do Bolhão, no Porto, é um ponto de transferência crítico entre o Metro do Porto (Linha D — Amarela) e várias linhas de autocarro da Operadora Exemplo, incluindo a Linha 200. [[DATA-01]](#DATA-01)

```csv
from_stop_id,to_stop_id,transfer_type,min_transfer_time
BLH_METRO_D,BLRB1,2,180
BLRB1,BLH_METRO_D,2,240
BLH_METRO_D,BLH_EXEMPLO_301,2,180
BLH_EXEMPLO_301,BLH_METRO_D,2,240
```

Leitura:
- **Metro → Operadora Exemplo Linha 200**: `transfer_type=2` com `min_transfer_time=180` (3 minutos), o passageiro precisa de subir da plataforma do metro até à paragem de autocarro à superfície
- **Operadora Exemplo Linha 200 → Metro**: `transfer_type=2` com `min_transfer_time=240` (4 minutos), sentido inverso demora mais porque inclui descer ao piso subterrâneo e validar o título de transporte
- As transferências entre a Linha 301 e o Metro seguem a mesma lógica

O tempo assimétrico (3 min vs 4 min) reflete a realidade física: descer é tipicamente mais rápido do que subir num interface com desníveis.

Ver ficheiro: [`exemplos/05_02_transfers_bolhao.txt`](exemplos/05_02_transfers_bolhao.txt)

### 2.5 Boas Práticas para Transferências

As Best Practices recomendam: [[BP-01]](#BP-01)

1. **Definir transferências explícitas** para interfaces intermodais importantes, em vez de confiar apenas na proximidade geográfica
2. **Usar `transfer_type=3`** para proibir transferências impossíveis (ex: paragens no mesmo edifício mas sem acesso pedonal entre elas)
3. **Tempos realistas** — medir o tempo de caminhada real, incluindo escadas, elevadores e validação tarifária
4. **Considerar acessibilidade** — o tempo de transferência para utilizadores com mobilidade reduzida pode ser significativamente superior

---

## 3. `pathways.txt` — Caminhos Pedonais

### 3.1 Finalidade

O `pathways.txt` descreve os caminhos físicos dentro de uma estação — corredores, escadas, elevadores, escadas rolantes e portões de validação. Combinado com `levels.txt`, permite modelar a navegação interna de estações complexas. [[STD-01]](#STD-01)

Esta informação é crítica para:
- Cálculo preciso de tempos de transferência
- Navegação acessível (rotas sem escadas)
- Representação realista de estações intermodais em aplicações de navegação

### 3.2 Campos

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `pathway_id` | ID | Sim | Identificador único do caminho |
| `from_stop_id` | ID | Sim | Nó de origem (`location_type=0/2/3/4`) |
| `to_stop_id` | ID | Sim | Nó de destino (`location_type=0/2/3/4`) |
| `pathway_mode` | Inteiro | Sim | Tipo de caminho (ver abaixo) |
| `is_bidirectional` | Inteiro | Sim | 0=unidirecional, 1=bidirecional |
| `length` | Float | Não | Comprimento em metros |
| `traversal_time` | Inteiro | Não | Tempo de travessia em segundos |
| `stair_count` | Inteiro | Não | Número de degraus (positivo=subir, negativo=descer) |
| `max_slope` | Float | Não | Inclinação máxima (ratio) |
| `min_width` | Float | Não | Largura mínima em metros |
| `signposted_as` | Texto | Não | Sinalização visível (ex: "Saída", "Linha D") |
| `reversed_signposted_as` | Texto | Não | Sinalização no sentido inverso |

### 3.3 Tipos de Caminho (`pathway_mode`)

| `pathway_mode` | Tipo | Acessível cadeira de rodas? |
|-----------------|------|----------------------------|
| `1` | Corredor/passadiço (walkway) | Sim (se largura suficiente) |
| `2` | Escadas (stairs) | Não |
| `3` | Passadeira rolante (moving sidewalk) | Sim |
| `4` | Escada rolante (escalator) | Não |
| `5` | Elevador (elevator) | Sim |
| `6` | Portão de entrada tarifário (fare gate) | Depende da largura |
| `7` | Portão de saída (exit gate) | Depende da largura |

Para garantir acessibilidade, uma estação deve ter **pelo menos um caminho completo** entre cada entrada e cada plataforma que não inclua `pathway_mode` 2 (escadas) ou 4 (escada rolante).

### 3.4 Nós de Navegação (`location_type`)

O `pathways.txt` utiliza nós (`nodes`) definidos em `stops.txt` com tipos específicos: [[STD-01]](#STD-01)

| `location_type` | Tipo | Papel no `pathways.txt` |
|------------------|------|------------------------|
| `0` | Paragem/Plataforma | Ponto de embarque (origem/destino do passageiro) |
| `2` | Entrada/Saída | Acesso ao exterior da estação |
| `3` | Nó genérico | Ponto intermédio (cruzamento de corredores, etc.) |
| `4` | Área de embarque | Ponto específico dentro de uma plataforma |

### 3.5 Exemplo: Estação de Bolhão

Modelação simplificada dos caminhos dentro da Estação de Bolhão, com dois pisos:

```
Nível 0 (Rua): Entrada principal → Átrio → Portões tarifários
Nível -1 (Subterrâneo): Plataforma Metro Linha D
```

```csv
pathway_id,from_stop_id,to_stop_id,pathway_mode,is_bidirectional,length,traversal_time,stair_count
pw_blh_01,BLH_ENTRADA,BLH_ATRIO,1,1,15,20,
pw_blh_02,BLH_ATRIO,BLH_GATE_IN,6,0,3,5,
pw_blh_03,BLH_GATE_IN,BLH_ATRIO,7,0,3,5,
pw_blh_04,BLH_GATE_IN,BLH_ESCADAS_TOPO,1,1,10,15,
pw_blh_05,BLH_ESCADAS_TOPO,BLH_PLAT_D,2,1,20,45,-24
pw_blh_06,BLH_ESCADAS_TOPO,BLH_ELEVADOR_TOPO,1,1,8,10,
pw_blh_07,BLH_ELEVADOR_TOPO,BLH_ELEVADOR_BASE,5,1,,30,
pw_blh_08,BLH_ELEVADOR_BASE,BLH_PLAT_D,1,1,5,8,
```

Leitura:
- **pw_blh_01**: corredor de 15 m da entrada ao átrio (20 segundos, bidirecional)
- **pw_blh_02**: portão de entrada tarifário (unidirecional — só entrada)
- **pw_blh_03**: portão de saída (unidirecional — só saída)
- **pw_blh_04**: corredor do portão até ao topo das escadas
- **pw_blh_05**: escadas com 24 degraus (negativo = descer) até à plataforma — **não acessível**
- **pw_blh_06 a pw_blh_08**: percurso alternativo via elevador — **acessível**

Um utilizador em cadeira de rodas seguiria o percurso: `BLH_ENTRADA → BLH_ATRIO → BLH_GATE_IN → BLH_ESCADAS_TOPO → BLH_ELEVADOR_TOPO → BLH_ELEVADOR_BASE → BLH_PLAT_D` (total: ~88 segundos), evitando as escadas.

---

## 4. `levels.txt` — Níveis de uma Estação

### 4.1 Finalidade

O `levels.txt` define os pisos/níveis físicos de uma estação. Cada nó em `stops.txt` pode referenciar um nível, o que permite representar a estrutura vertical de estações complexas. [[STD-01]](#STD-01)

### 4.2 Campos

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `level_id` | ID | Sim | Identificador único do nível |
| `level_index` | Float | Sim | Índice numérico do nível (0=rua, negativo=subterrâneo, positivo=elevado) |
| `level_name` | Texto | Não | Nome legível do nível |

### 4.3 Exemplo: Estação de Bolhão

```csv
level_id,level_index,level_name
BLH_L0,0,Rua / Átrio
BLH_L-1,-1,Plataforma Metro
```

Cada nó (`stop`) associado a esta estação referencia o nível correspondente via o campo `level_id` em `stops.txt`:

```csv
stop_id,stop_name,location_type,parent_station,level_id
BLH_ENTRADA,Bolhão - Entrada Principal,2,BLH_STATION,BLH_L0
BLH_ATRIO,Bolhão - Átrio,3,BLH_STATION,BLH_L0
BLH_GATE_IN,Bolhão - Portão Entrada,3,BLH_STATION,BLH_L0
BLH_ESCADAS_TOPO,Bolhão - Topo Escadas,3,BLH_STATION,BLH_L0
BLH_ELEVADOR_TOPO,Bolhão - Elevador (Rua),3,BLH_STATION,BLH_L0
BLH_ELEVADOR_BASE,Bolhão - Elevador (Metro),3,BLH_STATION,BLH_L-1
BLH_PLAT_D,Bolhão - Plataforma Linha D,0,BLH_STATION,BLH_L-1
```

### 4.4 Relação entre `levels.txt`, `stops.txt` e `pathways.txt`

A interação entre os três ficheiros permite uma modelação completa:

```
levels.txt          stops.txt (nodes)           pathways.txt
+----------+       +-------------------+       +------------------+
| BLH_L0   |<──────| BLH_ENTRADA (L0)  |──────>| pw_blh_01        |
| (Rua)    |       | BLH_ATRIO (L0)    |       | pw_blh_02        |
+----------+       | BLH_GATE_IN (L0)  |       | pw_blh_04        |
                    +-------------------+       +------------------+
+----------+       +-------------------+       +------------------+
| BLH_L-1  |<──────| BLH_PLAT_D (L-1)  |──────>| pw_blh_05        |
| (Metro)  |       | BLH_ELEV_BASE(-1) |       | pw_blh_07        |
+----------+       +-------------------+       +------------------+
```

---

## 5. Ponte NeTEx (Opcional)

Para quem seguirá o curso NeTEx, as equivalências são:

| GTFS | NeTEx | Notas |
|------|-------|-------|
| `wheelchair_boarding` / `wheelchair_accessible` | `AccessibilityAssessment` | NeTEx modela acessibilidade como uma avaliação com múltiplas dimensões (cadeira de rodas, audição, visão) |
| `transfers.txt` | `SiteConnection` / `DefaultConnection` | NeTEx distingue conexões entre locais (`SiteConnection`) de conexões entre serviços (`Connection`) |
| `pathways.txt` | `NavigationPath` / `PathLink` | NeTEx usa `PathLink` para segmentos e `NavigationPath` para percursos completos |
| `levels.txt` | `Level` (dentro de `StopPlace`) | Estrutura semelhante; NeTEx integra-a diretamente na hierarquia do `StopPlace` |
| `pathway_mode` (elevator, stairs, etc.) | `AccessFeature` / `StairSpecification` / `LiftEquipment` | NeTEx modela cada equipamento como entidade separada com atributos detalhados |

O NeTEx oferece uma modelação de acessibilidade significativamente mais rica, incluindo tipos de deficiência (visual, auditiva, cognitiva), equipamentos específicos (pisos táteis, anúncios sonoros) e avaliações por percurso.

---

## Exemplos

- [`exemplos/05_01_stops_acessibilidade.txt`](exemplos/05_01_stops_acessibilidade.txt) — Paragens com campos de acessibilidade (`wheelchair_boarding`) para a Estação de Bolhão
- [`exemplos/05_02_transfers_bolhao.txt`](exemplos/05_02_transfers_bolhao.txt) — Transferências no interface intermodal do Bolhão (Metro ↔ Operadora Exemplo)

---

## Exercícios

- [Exercício 5 — Modelar Acessibilidade e Transferências](exercicios/exercicio_05.md)

---

## Referências

| ID | Referência |
|----|-----------|
| <a id="STD-01"></a>[STD-01] | MobilityData. "GTFS Schedule Reference." https://gtfs.org/schedule/reference |
| <a id="BP-01"></a>[BP-01] | MobilityData. "GTFS Schedule Best Practices." https://gtfs.org/documentation/schedule/schedule-best-practices |
| <a id="REG-01"></a>[REG-01] | Regulamento Delegado (UE) 2024/490 relativo ao quadro europeu para serviços de informação sobre viagens multimodais (MMTIS). |
| <a id="DATA-01"></a>[DATA-01] | Operadora Exemplo. "Feed GTFS Operadora Exemplo (versão Escolar 228, 2026-04-30)." Feed oficial Operadora Exemplo. |
| <a id="DL-01"></a>[DL-01] | Decreto-Lei n.º 163/2006 — regulamenta as normas técnicas de acessibilidade em edifícios e espaços públicos em Portugal, incluindo interfaces de transporte