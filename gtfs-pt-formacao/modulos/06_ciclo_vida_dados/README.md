# Módulo 6 — Ciclo de Vida dos Dados

## Objetivos de Aprendizagem

Após completar este módulo, o leitor será capaz de:

- Identificar as principais fontes de dados que alimentam um feed GTFS (AVL, APC, planeamento, GIS)
- Descrever o processo completo de criação de um feed GTFS, desde a recolha até à publicação
- Utilizar o Canonical GTFS Validator para detetar e classificar erros num feed
- Compreender as obrigações de publicação no NAP Portugal e no contexto regulatório europeu (MMTIS)
- Aplicar boas práticas de versionamento, atualização e controlo de qualidade a feeds de produção

---

## 1. Fontes de Dados

Os dados que compõem um feed GTFS vêm de múltiplos sistemas dentro de um operador de transporte. Compreender estas fontes é essencial para desenhar um pipeline de dados robusto. [[STD-01]](#STD-01)

### 1.1 Sistemas AVL (Automatic Vehicle Location)

Os sistemas AVL rastreiam a posição dos veículos em tempo real via GPS:

- **Dados produzidos**: posição geográfica, velocidade, aderência ao horário
- **Relevância para GTFS**: alimentam o GTFS Realtime (posições de veículos, previsões de chegada) e permitem validar se os `shapes.txt` e `stop_times.txt` do feed estático correspondem à realidade operacional [[STD-07]](#STD-07)
- **Protocolos comuns**: SIRI (Service Interface for Real Time Information), GTFS Realtime
- **Exemplo STCP**: a STCP disponibiliza dados AVL que alimentam os painéis de informação ao passageiro nas paragens [[DATA-01]](#DATA-01)

### 1.2 Sistemas APC (Automatic Passenger Counting)

Os sistemas APC contam automaticamente os passageiros que entram e saem dos veículos:

- **Dados produzidos**: contagens de embarque/desembarque por paragem, ocupação do veículo
- **Relevância para GTFS**: a extensão GTFS-ride permite publicar dados de ridership agregados, úteis para planeamento e transparência [[STD-04]](#STD-04)
- **Tecnologias**: sensores infravermelhos, câmaras estereoscópicas, pisómetros

> **Nota**: a maioria dos operadores portugueses ainda não publica dados APC em formato GTFS-ride. Este é um campo em evolução.

### 1.3 Sistemas de Planeamento

O software de planeamento de serviço é, na prática, a fonte primária de um feed GTFS estático:

- **Dados produzidos**: linhas, percursos, horários, escalas de serviço, calendários
- **Ferramentas comuns**: HASTUS (Giro), REMIX, Optibus, Trapeze, IVU
- **Fluxo típico**: o planeador define horários no software → o software exporta (ou um script extrai) os dados → transformação para formato GTFS [[BP-02]](#BP-02)

A STCP, por exemplo, gera o seu feed GTFS a partir do sistema interno de planeamento, o que explica a abordagem de usar exclusivamente `calendar_dates.txt` com `exception_type=1`, os dados são gerados programaticamente, data a data. [[DATA-01]](#DATA-01)

### 1.4 Dados Geoespaciais

As coordenadas das paragens e os traçados das linhas requerem dados geoespaciais de qualidade:

- **Fontes**: levantamento GPS em campo, OpenStreetMap (OSM), cartografia municipal, ortofotomapas
- **Campos GTFS afetados**: `stop_lat`/`stop_lon` em `stops.txt`, pontos em `shapes.txt`
- **Desafio português**: garantir que as coordenadas WGS84 no feed correspondem à localização real das paragens, especialmente em zonas urbanas densas como o centro do Porto [[BP-01]](#BP-01)

> **Boas práticas**: validar coordenadas contra uma fonte independente (ex: OSM) e verificar que nenhuma paragem está localizada em locais impossíveis (no meio de um rio, fora do concelho).

---

## 2. Processo de Criação de um Feed GTFS

### 2.1 Recolha e Integração

O primeiro passo é reunir dados das várias fontes num formato intermédio:

```
┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐
│  Software de    │   │  Dados GIS      │   │  Dados AVL/APC  │
│  Planeamento    │   │  (paragens,     │   │  (validação)    │
│  (horários,     │   │   shapes)       │   │                 │
│   calendários)  │   │                 │   │                 │
└────────┬────────┘   └────────┬────────┘   └────────┬────────┘
         │                     │                      │
         └─────────────────────┼──────────────────────┘
                               ▼
                    ┌─────────────────────┐
                    │  Base de Dados      │
                    │  Intermédia         │
                    │  (staging)          │
                    └──────────┬──────────┘
                               ▼
                    ┌─────────────────────┐
                    │  Pipeline ETL       │
                    │  (transformação)    │
                    └──────────┬──────────┘
                               ▼
                    ┌─────────────────────┐
                    │  Feed GTFS (.zip)   │
                    └─────────────────────┘
```

Desafios comuns na integração:

- **Inconsistência de IDs** — o mesmo ponto de paragem pode ter IDs diferentes no sistema de planeamento e no GIS
- **Dados desatualizados** — coordenadas de paragens que foram fisicamente relocalizadas
- **Formatos heterogéneos** — cada sistema exporta num formato proprietário diferente

### 2.2 Transformação (ETL)

O processo ETL (Extract, Transform, Load) converte os dados brutos em ficheiros GTFS conformes: [[STD-01]](#STD-01)

1. **Extract** — extrair dados do software de planeamento, GIS e outros sistemas
2. **Transform** — mapear campos internos para campos GTFS, normalizar IDs, converter coordenadas, gerar `shapes.txt` a partir de traçados
3. **Load** — escrever os ficheiros CSV com cabeçalhos corretos e empacotar num arquivo `.zip`

Exemplo de transformação para `agency.txt`:

```
Sistema interno:                    GTFS:
  operador_codigo = "STCP"   →     agency_id = "STCP"
  operador_nome = "STCP, SA"  →     agency_name = "STCP"
  operador_url = "..."        →     agency_url = "https://www.stcp.pt"
  fuso_horario = "WET"        →     agency_timezone = "Europe/Lisbon"
```

### 2.3 Validação

A validação é um passo **obrigatório** antes da publicação. O validador de referência é o **Canonical GTFS Validator** da MobilityData: [[TOOL-01]](#TOOL-01)

```bash
java -jar gtfs-validator-cli.jar -i feed_stcp.zip -o relatorio/
```

O validador produz um relatório HTML com três níveis de severidade:

| Nível | Significado | Ação requerida |
|-------|-------------|----------------|
| **ERROR** | Violação da especificação GTFS — o feed é inválido | Corrigir antes de publicar |
| **WARNING** | Desvio das boas práticas — o feed funciona mas pode causar problemas | Corrigir se possível |
| **INFO** | Informação ou sugestão de melhoria | Avaliar caso a caso |

Exemplos de erros comuns detetados pelo validador:

| Código | Nível | Descrição |
|--------|-------|-----------|
| `foreign_key_violation` | ERROR | Um `stop_id` referenciado em `stop_times.txt` não existe em `stops.txt` |
| `duplicate_key` | ERROR | Dois registos com o mesmo ID primário |
| `stop_too_far_from_trip_shape` | WARNING | Paragem a mais de 100m do traçado (`shapes.txt`) |
| `fast_travel_between_stops` | WARNING | Velocidade implícita entre paragens superior a 150 km/h |
| `feed_expiration_date` | WARNING | O feed expira nos próximos 7 dias |
| `missing_feed_info_date` | WARNING | `feed_info.txt` sem `feed_start_date` ou `feed_end_date` |

> **Recomendação**: executar o validador como parte de um pipeline automático (CI/CD). Qualquer ERROR deve bloquear a publicação. [[BP-01]](#BP-01)

### 2.4 Publicação

Após a validação, o feed deve ser publicado em locais acessíveis aos consumidores:

#### NAP Portugal

O Ponto de Acesso Nacional (NAP) português disponibiliza feeds de operadores nacionais: [[PT-01]](#PT-01)

- **URL**: https://nap-portugal.imt-ip.pt
- **Obrigação legal**: o Regulamento Delegado (UE) 2024/490 (MMTIS) exige que os operadores disponibilizem dados de transporte em formatos normalizados através dos NAP nacionais [[REG-01]](#REG-01)
- **Formatos aceites**: GTFS, NeTEx (CEN)

#### Mobility Database

A Mobility Database da MobilityData é um catálogo global de feeds GTFS: [[DATA-02]](#DATA-02)

- **URL**: https://mobilitydatabase.org
- Os operadores (ou terceiros) registam o URL do feed `.zip`
- A base de dados verifica periodicamente se o feed está atualizado
- Aplicações como Google Maps, Moovit e Transit App utilizam esta base para descobrir feeds

#### `feed_info.txt`

O ficheiro `feed_info.txt` é essencial para a gestão do ciclo de vida: [[STD-01]](#STD-01) [[BP-01]](#BP-01)

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `feed_publisher_name` | Texto | Sim | Nome da entidade que publica o feed |
| `feed_publisher_url` | URL | Sim | URL da entidade que publica |
| `feed_lang` | Código | Sim | Idioma principal do feed (ex: `pt`) |
| `default_lang` | Código | Não | Idioma por defeito quando `feed_lang` é multilíngue |
| `feed_start_date` | Data | Não* | Data de início da validade (`YYYYMMDD`) |
| `feed_end_date` | Data | Não* | Data de fim da validade (`YYYYMMDD`) |
| `feed_version` | Texto | Não* | Identificador da versão do feed |
| `feed_contact_email` | Email | Não | Email de contacto técnico |
| `feed_contact_url` | URL | Não | URL para reporte de problemas |

*Campos recomendados pelas Best Practices.

Exemplo para a STCP:

```csv
feed_publisher_name,feed_publisher_url,feed_lang,feed_start_date,feed_end_date,feed_version,feed_contact_email
STCP,https://www.stcp.pt,pt,20260430,20260731,Escolar_228,dados@stcp.pt
```

---

## 3. Manutenção e Atualização

### 3.1 Frequência de Atualização

A frequência de atualização de um feed GTFS depende da dinâmica do serviço: [[BP-01]](#BP-01)

| Tipo de alteração | Frequência típica | Exemplo |
|-------------------|-------------------|---------|
| Mudança sazonal de horários | 2-4 vezes/ano | Horário de Verão vs Inverno |
| Ajuste pontual de percurso | Conforme necessário | Obras na Via de Cintura Interna |
| Atualização de paragens | Conforme necessário | Nova paragem na linha 200 |
| Correção de erros | Imediato após deteção | Coordenadas erradas |

As Best Practices recomendam: [[BP-01]](#BP-01) [[BP-02]](#BP-02)

- Publicar o novo feed pelo menos 7 dias antes da data de entrada em vigor
- Manter o feed anterior acessível durante um período de transição
- Para alterações menores dentro de 7 dias, usar GTFS Realtime (service alerts) em vez de republicar o feed estático

### 3.2 Versionamento

Um sistema de versionamento claro facilita a rastreabilidade e a resolução de problemas:

- Usar o campo `feed_version` em `feed_info.txt` com um identificador único por versão
- Convenções comuns: data (`20260430`), nome de período (`Escolar_228`), semver (`2.3.1`)
- Manter um registo (changelog) das alterações entre versões

Exemplo de registo de versões:

```
Versão          Data        Alterações
Escolar_228     2026-04-30  Horário escolar 2o semestre; 3 novas paragens L200
Escolar_227     2026-01-15  Horário escolar 1o semestre; ajuste Linha 502
Ferias_226      2025-07-01  Horário de Verão 2025
```

### 3.3 Compatibilidade com Consumidores

O feed GTFS é consumido por múltiplas aplicações, cada uma com os seus requisitos: [[BP-01]](#BP-01)

| Consumidor | Requisitos específicos |
|------------|----------------------|
| **Google Transit** | Validação rigorosa; exige `feed_info.txt`; rejeita feeds com ERRORs |
| **Moovit** | Aceita feeds com WARNINGs; prefere `shapes.txt` para visualização |
| **Transit App** | Requer coordenadas precisas para cálculo de rotas a pé |
| **Planeadores internos** | Podem consumir campos opcionais (ex: `bikes_allowed`, `wheelchair_accessible`) |
| **NAP Portugal** | Aceita GTFS e NeTEx; verifica conformidade MMTIS [[REG-01]](#REG-01) |

> **Boa prática**: antes de publicar uma nova versão, testar o feed nos principais consumidores. Uma alteração de `route_id` pode quebrar integrações existentes no Google Transit.

---

## 4. Qualidade dos Dados

### 4.1 Métricas de Qualidade

A qualidade de um feed GTFS pode ser medida em várias dimensões: [[BP-01]](#BP-01)

| Dimensão | Métrica | Como medir |
|----------|---------|------------|
| **Completude** | % de campos opcionais preenchidos | Verificar presença de `shapes.txt`, `feed_info.txt`, `wheelchair_accessible` |
| **Exatidão** | Distância média entre paragens GTFS e localização real | Comparar `stop_lat`/`stop_lon` com GPS de campo ou OSM |
| **Atualidade** | Dias até expiração do feed | `feed_end_date` menos data atual |
| **Consistência** | N.o de ERRORs/WARNINGs no validador | Executar o Canonical GTFS Validator [[TOOL-01]](#TOOL-01) |
| **Acessibilidade** | Disponibilidade do URL do feed | Monitorizar uptime do endpoint `.zip` |

### 4.2 Erros Comuns em Feeds Portugueses

Com base na experiência de validação de feeds de operadores portugueses: [[DATA-01]](#DATA-01)

| Erro | Causa frequente | Impacto |
|------|----------------|---------|
| Paragens sem coordenadas ou com coordenadas `0,0` | Paragem nova não georreferenciada | Paragem não aparece no mapa |
| `shapes.txt` ausente | Operador não exporta traçados | Linha reta entre paragens nas apps |
| `feed_info.txt` ausente ou incompleto | Omissão na geração do feed | Google Transit pode rejeitar o feed |
| Caracteres especiais em IDs | Ex: `DOMINGOS\|FERIADOS` da STCP | Problemas de parsing em alguns sistemas |
| Datas expiradas no calendário | Feed não atualizado | Nenhum serviço ativo — apps mostram 0 resultados |
| Tempos de viagem irreais | Erro no `stop_times.txt` | Velocidade >150 km/h entre paragens |

### 4.3 Boas Práticas de Gestão

Para manter um feed GTFS de alta qualidade ao longo do tempo: [[BP-01]](#BP-01) [[BP-02]](#BP-02)

1. **Automatizar o pipeline** — da extração à validação, minimizar passos manuais
2. **Validar sempre antes de publicar** — Buscar um arquivo sem error
3. **Monitorizar a expiração** — alertas automáticos quando `feed_end_date` está a menos de 14 dias
4. **Documentar o processo** — quem gera o feed, com que ferramenta, com que frequência
5. **Designar um responsável** — uma pessoa ou equipa com responsabilidade clara pelo feed
6. **Recolher feedback dos consumidores** — apps como o Google Transit enviam relatórios de problemas
7. **Manter um ambiente de teste** — validar novas versões antes de publicar em produção

---

## 5. Ponte NeTEx (Opcional)

Para operadores no contexto europeu, o NeTEx (Network Timetable Exchange) é o formato de referência CEN: [[REG-01]](#REG-01)

### GTFS vs NeTEx no Ciclo de Publicação

| Aspeto | GTFS | NeTEx |
|--------|------|-------|
| **Formato** | CSV em ZIP | XML |
| **Complexidade** | Baixa (13 ficheiros base) | Alta (centenas de elementos) |
| **Adoção** | Global (apps, Google) | Europeia (regulação, NAPs) |
| **Validação** | Canonical GTFS Validator | Validadores NeTEx nacionais |
| **NAP Portugal** | Aceite | Aceite (formato preferido pela regulação EU) |

### Pipeline de Publicação Dual

Alguns operadores mantêm um pipeline que gera ambos os formatos a partir da mesma fonte:

```
Software de Planeamento
         │
         ▼
  Base de Dados Interna
         │
    ┌────┴────┐
    ▼         ▼
  GTFS      NeTEx
  (.zip)    (.xml)
    │         │
    ▼         ▼
  Mobility   NAP
  Database   Portugal
```

O Regulamento (UE) 2024/490 não impõe um formato específico, mas refere o NeTEx e o SIRI como standards de referência. Na prática, o GTFS é aceite pelos NAP nacionais, incluindo o português. [[REG-01]](#REG-01) [[PT-01]](#PT-01)

---

## Exemplos

> Este módulo é de natureza operacional e processual. Os exemplos práticos são trabalhados nos exercícios abaixo.

---

## Exercícios

- [Exercício 6 — Ciclo de Vida de um Feed GTFS](exercicios/exercicio_06.md)

---

## Referências

| ID | Referência |
|----|-----------|
| <a id="STD-01"></a>[STD-01] | MobilityData. "GTFS Schedule Reference." https://gtfs.org/schedule/reference |
| <a id="STD-04"></a>[STD-04] | GTFS-ride. Extensão para dados de ridership (passageiros). https://gtfsride.org |
| <a id="STD-07"></a>[STD-07] | MobilityData. "GTFS Realtime Reference." https://gtfs.org/realtime/reference |
| <a id="BP-01"></a>[BP-01] | MobilityData. "GTFS Schedule Best Practices." https://gtfs.org/documentation/schedule/schedule-best-practices |
| <a id="BP-02"></a>[BP-02] | Transit App. "Guidelines for Producing GTFS Static Data." https://resources.transitapp.com/article/458-guidelines-for-producing-gtfs-static-data-for-transit |
| <a id="TOOL-01"></a>[TOOL-01] | MobilityData. "Canonical GTFS Validator." https://github.com/MobilityData/gtfs-validator |
| <a id="DATA-01"></a>[DATA-01] | STCP. "Feed GTFS STCP Porto (versão Escolar 228, 2026-04-30)." Feed oficial STCP. |
| <a id="DATA-02"></a>[DATA-02] | MobilityData. "Mobility Database." https://mobilitydatabase.org |
| <a id="PT-01"></a>[PT-01] | IMT. "NAP Portugal — Ponto de Acesso Nacional." https://nap-portugal.imt-ip.pt |
| <a id="REG-01"></a>[REG-01] | Regulamento Delegado (UE) 2024/490 da Comissão Europeia (MMTIS). https://eur-lex.europa.eu/legal-content/PT/TXT/?uri=OJ:L_202400490 |
