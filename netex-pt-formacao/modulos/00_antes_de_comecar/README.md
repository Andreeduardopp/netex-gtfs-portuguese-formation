# Módulo 0 — Antes de Começar

> *Tempo estimado: 1–2 horas*

## Objetivos de Aprendizagem

Após completar este módulo, o leitor será capaz de:

- Explicar o que é o NeTEx e a sua importância para operadores públicos portugueses 
- Distinguir GTFS e NeTEx em termos de cobertura, profundidade e modelo conceptual
- Situar o NeTEx no ecossistema: Transmodel → NeTEx → SIRI → NAP
- Identificar as obrigações regulatórias europeias aplicáveis em Portugal
- Configurar o ambiente de trabalho para os módulos seguintes

---

## 1. O que é o NeTEx e Porque Importa

### 1.1 Definição

**NeTEx** (*Network Timetable Exchange*) é um standard europeu de troca de dados de transporte público, publicado pelo CEN (Comité Europeu de Normalização) como série de normas técnicas CEN/TS 16614. [[STD-01]](#STD-01)

O NeTEx define um formato XML para descrever:

- **Redes de transporte** — linhas, rotas, paragens, topologia
- **Horários e calendários** — viagens, padrões de serviço, variações sazonais
- **Tarifas** — zonas, produtos, títulos de transporte
- **Acessibilidade** — atributos de paragens, veículos e percursos
- **Multimodalidade** — interação entre modos (metro, autocarro, comboio, bicicleta)

### 1.2 Origem e Contexto

O NeTEx foi desenvolvido a partir do **Transmodel** (CEN EN 12896), o modelo conceptual de referência para transporte público europeu. [[STD-04]](#STD-04) A relação é direta:

> Transmodel define *o quê* (os conceitos). NeTEx define *como* (a troca de dados XML).

A primeira versão do NeTEx foi publicada em 2014. A versão atual (2024) inclui três partes principais e uma parte dedicada à informação ao passageiro (**EPIAP** — *European Passenger Information and Access Profile*).

### 1.3 Porque Importa para Operadores (Públicos e Privados) em Portugal

Até recentemente, a maioria dos operadores portugueses publicava os seus dados em **GTFS** (Google Transit Feed Specification). [[STD-07]](#STD-07) O GTFS é prático e amplamente adotado para visualização em mapas, mas:

- **É insuficiente para cumprir a regulação europeia.** O Regulamento Delegado (UE) 2024/490 obriga os operadores a partilhar dados no formato NeTEx/SIRI com o Ponto de Acesso Nacional (NAP). [[REG-01]](#REG-01)
- **Não modela toda a complexidade do serviço.** Tarifas avançadas (como o Navegante Metropolitano), acessibilidade detalhada da infraestrutura e regras de intermodalidade ficam fora do alcance do GTFS.
- **Não é interoperável a nível europeu.** O NeTEx permite que sistemas de diferentes países "falem a mesma língua", o que é essencial no espaço Schengen.
- **É limitativo para redes privadas e serviços especializados.** Para operadores de vaivéns corporativos, redes universitárias ou serviços a pedido, o GTFS falha ao assumir que o acesso é universal. O NeTEx permite modelar regras restritas de acesso (ex: "apenas funcionários com crachá"), contratos de financiamento corporativo (B2B) e facilita a integração destas frotas privadas em plataformas empresariais de MaaS (*Mobility as a Service*). Além disso, garante a granularidade necessária para reportar SLAs (Níveis de Serviço) rigorosos em regimes de subconcessão.

Em Portugal, o **IMT-IP** (Instituto da Mobilidade e dos Transportes) é a entidade responsável pela implementação do NAP e pela definição do Perfil Nacional NeTEx Portugal. [[PT-03]](#PT-03)

---

## 2. Comparação com GTFS

Se já trabalha com GTFS, esta tabela mostra onde cada standard se posiciona:

| Dimensão | GTFS | NeTEx |
|----------|------|-------|
| **Formato** | CSV (ficheiros `.txt` num `.zip`) | XML (por norma um `.xml` por `Frame`) |
| **Modelo conceptual** | Pragmático, sem modelo formal | Baseado no Transmodel (CEN EN 12896) [[STD-04]](#STD-04) |
| **Cobertura** | Horários, paragens, rotas, tarifas básicas | Tudo do GTFS + acessibilidade, multimodalidade, tarifas complexas, informação ao passageiro |
| **Identificadores** | Locais (sem formato obrigatório) | Globais, hierárquicos: `PT:EXEMPLO:Line:200:LOC` |
| **Versionamento** | Não existe | Nativo — cada objeto tem `version` e `validFrom/To` |
| **Complexidade** | Baixa — 6 ficheiros obrigatórios | Alta — schema XSD extenso (~500 tipos XML) |
| **Adoção** | 10.000+ agências globalmente | Obrigatório na UE (Regulamento 2024/490) [[REG-01]](#REG-01) |
| **Extensibilidade** | Via extensões ad hoc (GTFS-Flex, GTFS-Fares v2) | Nativa — perfis nacionais definem subconjuntos |
| **Tempo real** | GTFS Realtime (Protocol Buffers) | SIRI (XML), baseado no mesmo Transmodel |

### 2.1 A Analogia Certa

Pense no GTFS como o Menu do Restaurante e no NeTEx como o Sistema ERP da Cozinha:

- O GTFS (O Menu): Diz ao cliente exatamente o que está disponível, quando pode ser consumido e a que preço. Resolve rapidamente o problema da informação ao público.

- O NeTEx (O Sistema da Cozinha): Modela os ingredientes, as receitas, as escalas de turno dos cozinheiros, as regras de substituição de pratos e a logística de entrega. Resolve o problema de gerir e interoperar o negócio.

Transitar do GTFS para o NeTEx significa deixar de olhar apenas para o produto final que o passageiro consome e passar a modelar toda a linha de montagem do serviço de transporte público, com rigor e profundidade.

### 2.2 Equivalências Diretas (Prévia)

Esta tabela será desenvolvida em detalhe em cada módulo. Aqui fica uma prévia:

| Conceito GTFS | Equivalente NeTEx | Diferença Principal |
|---------------|-------------------|---------------------|
| `agency` | `Operator` + `Authority` | NeTEx distingue quem opera de quem tem autoridade |
| `route` | `Line` + `Route` + `Direction` | NeTEx separa a linha comercial da sua topologia |
| `stop` (coordenadas) | `ScheduledStopPoint` | Ponto lógico; o lugar físico é `StopPlace`/`Quay` |
| `trip` | `ServiceJourney` | NeTEx adiciona padrões reutilizáveis (`JourneyPattern`) |
| `calendar_dates` | `DayType` + `DayTypeAssignment` | Modelo mais expressivo para variações de serviço |
| `fare_zone` | `TariffZone` | Estrutura semelhante, NeTEx mais completo |

---

## 3. O Ecossistema: Transmodel → NeTEx → SIRI → NAP

O NeTEx não existe isolado. Faz parte de um ecossistema de standards europeus construídos sobre o mesmo modelo conceptual:

```
┌─────────────────────────────────────────────────────┐
│                    TRANSMODEL                        │
│         (CEN EN 12896 — Modelo Conceptual)          │
│  Define os conceitos: Line, StopPlace, Journey...   │
└───────────────────┬────────────────┬────────────────┘
                    │                │
          ┌─────────▼──────┐  ┌──────▼──────────┐
          │     NeTEx      │  │      SIRI        │
          │  (CEN/TS 16614)│  │  (CEN/TS 15531) │
          │  Dados planeados│  │  Dados em tempo │
          │  (horários,     │  │  real (posições,│
          │   tarifas...)   │  │   avarias...)   │
          └────────┬────────┘  └──────┬───────────┘
                   │                  │
          ┌────────▼──────────────────▼───────────┐
          │           NAP Portugal                 │
          │  (nap-portugal.imt-ip.pt)              │
          │  Ponto de Acesso Nacional — agregador  │
          │  de dados de todos os operadores PT    │
          └────────────────────────────────────────┘
```

### 3.1 Transmodel — A Base de Tudo

O **Transmodel** é o modelo de dados de referência para transporte público europeu, mantido pelo CEN. [[STD-04]](#STD-04) Define os conceitos abstratos: o que é uma linha, uma paragem, uma viagem.

O NeTEx e o SIRI são implementações do Transmodel para diferentes casos de uso:
- **NeTEx** = dados estáticos/planeados (horários, tarifas, topologia)
- **SIRI** = dados dinâmicos/tempo real (posição de veículos, perturbações)

Isto significa que quem aprende NeTEx também está a aprender Transmodel, os conceitos são os mesmos.

### 3.2 O Papel do NAP

O **Ponto de Acesso Nacional (NAP)** é a plataforma onde os operadores publicam os seus dados para consumo por aplicações de mobilidade, investigadores e outros stakeholders. [[PT-02]](#PT-02)

Em Portugal, o NAP é gerido pelo **IMT-IP** e aceita dados no formato NeTEx. O Perfil Nacional NeTEx Portugal (ver Módulo 7) define exatamente que subconjunto do NeTEx é obrigatório para submissão ao NAP português.

---

## 4. Regulação Europeia Relevante para Portugal

### 4.1 O Regulamento MMTIS (2024/490)

O **Regulamento Delegado (UE) 2024/490**, conhecido como MMTIS (Multimodal Travel Information Services), é a principal obrigação legal para operadores portugueses. [[REG-01]](#REG-01) Substitui o regulamento anterior (2017/1926).

O regulamento obriga os estados-membros a:

1. Tornar disponíveis dados de transporte público em formatos interoperáveis
2. Publicar esses dados num Ponto de Acesso Nacional (NAP)
3. Garantir qualidade e atualização regular dos dados

Os formatos exigidos para dados estáticos são NeTEx (obrigatório para dados avançados) e GTFS (aceite para dados básicos). Para dados em tempo real, o SIRI é o standard de referência.

### 4.2 Que Dados São Obrigatórios?

O regulamento define obrigações por tipo de dado e prazo. Para operadores de transporte público em Portugal:

| Tipo de Dado | Standard | Obrigação |
|-------------|----------|-----------|
| Paragens e redes | NeTEx Parte 1 | Obrigatório |
| Horários | NeTEx Parte 2 | Obrigatório |
| Tarifas básicas | NeTEx Parte 3 | Recomendado |
| Acessibilidade | Perfil EPIAP | Recomendado |
| Tempo real — posições | SIRI ET/VM | Obrigatório |
| Perturbações | SIRI SX | Recomendado |

### 4.3 A Diretiva ITS como Base Legal

O MMTIS surge no âmbito da **Diretiva ITS 2010/40/UE** (Sistemas de Transporte Inteligentes), que criou o quadro legal para a digitalização do transporte na UE. [[REG-02]](#REG-02) Os regulamentos delegados (como o MMTIS) são os instrumentos concretos de implementação desta diretiva.

### 4.4 Impacto Prático em Portugal

Para um técnico de um operador português, as implicações práticas são:

1. O feed GTFS existente não é suficiente para cumprir a regulação, é preciso converter ou criar dados NeTEx
2. Os dados devem ser submetidos ao NAP Portugal com um perfil validado
3. O Perfil Nacional NeTEx Portugal (definido pelo IMT) especifica exatamente o que é obrigatório vs opcional para Portugal (ver Módulo 7)

---

## 5. Como Usar Este Curso

### 5.1 Pré-requisitos

Este módulo não ensina GTFS, parte do princípio que o leitor já conhece o standard e, caso ainda não o conheça, recomendamos a consulta prévia do [GTFS PT Formação](../../gtfs-pt-formacao/README.md). O público-alvo é:

- Técnicos de operadores de transporte público portugueses
- Profissionais de sistemas de informação de mobilidade
- Estudantes de engenharia de transportes com conhecimento de GTFS

O que deve saber antes de começar:
- ✅ Estrutura do GTFS (`agency.txt`, `routes.txt`, `stops.txt`, `trips.txt`, `stop_times.txt`)
- ✅ Noções básicas de XML (elementos, atributos, namespaces)
- ✅ Como usar um editor de texto com suporte a XML

### 5.2 Ferramentas Recomendadas

**Editor de código**: [Visual Studio Code](https://code.visualstudio.com/) (gratuito)

Extensões VS Code recomendadas:
- **XML** (Red Hat) — validação, formatação e navegação de ficheiros XML
- **GitLens** — para navegar o histórico do repositório

**Validação de XML**: O **NeTEx Validator da Entur** permite validar ficheiros NeTEx contra o schema XSD e os perfis nacionais. [[TOOL-01]](#TOOL-01) Será introduzido no Módulo 8.

### 5.3 A Rede Âncora: Operadora Exemplo

Todos os exemplos XML deste curso são baseados em dados reais da **Operadora Exemplo (Sociedade de Transportes Colectivos do Porto)**. [[DATA-03]](#DATA-03) Usamos a Linha 200 (Bolhão → Castelo do Queijo) como caso principal.

A escolha de dados reais (vs. uma rede fictícia) tem uma razão pedagógica: os exemplos são mais credíveis quando podemos verificar que correspondem a uma realidade que o leitor conhece ou pode consultar.

Os ficheiros GTFS de base estão disponíveis em [`netex-pt-formacao/exemplos/rede_exemplo_portugal/gtfs/`](../../exemplos/rede_exemplo_portugal/gtfs/) (na raiz deste repositório).

### 5.4 Estrutura de Cada Módulo

Cada módulo segue esta estrutura:

1. **Objetivos** — o que vai aprender
2. **Conteúdo** — explicação dos conceitos, com a ponte GTFS ↔ NeTEx
3. **Exemplos** — ficheiros XML comentados na pasta `exemplos/` do módulo
4. **Exercícios** — exercícios práticos com solução comentada
5. **Referências** — todas as fontes citadas no módulo

Os ficheiros XML de exemplo têm comentários extensivos que explicam o porquê de cada elemento, não apenas o que faz, mas porque existe e que problema resolve.

### 5.5 Como Navegar os Módulos

O curso é sequencial. Cada módulo constrói sobre o anterior. A progressão recomendada é:

```
Módulo 0 (este)
    ↓
Módulo 1 — Conceitos Base (estrutura NeTEx, VersionFrame, identificadores)
    ↓
Módulo 2 — Rede de Transporte (Line, Route, StopPlace)
    ↓
Módulo 3 — Horários e Calendários (ServiceJourney, DayType)
    ↓
Módulos 4–6 — Tópicos Avançados (multimodalidade, acessibilidade, tarifas)
    ↓
Módulo 7 — Perfil NeTEx Portugal (aplicação ao contexto nacional)
    ↓
Módulo 8 — Ferramentas Práticas (validação, submissão ao NAP)
```

**Dica de navegação**: Para cada conceito NeTEx que encontrar, procure o ficheiro XML de exemplo do módulo correspondente. Os exemplos são a melhor forma de consolidar os conceitos.

---

## Referências

| ID | Referência |
|----|-----------|
| <a id="STD-01"></a>[STD-01] | CEN. "NeTEx – Network Timetable Exchange." CEN/TS 16614-1:2024 (Parte 1: Topologia de Rede). https://github.com/NeTEx-CEN/NeTEx |
| <a id="STD-04"></a>[STD-04] | CEN. "Transmodel – Reference Data Model for Public Transport." EN 12896. https://transmodel-cen.eu |
| <a id="STD-07"></a>[STD-07] | MobilityData. "General Transit Feed Specification (GTFS)." https://gtfs.org |
| <a id="REG-01"></a>[REG-01] | Regulamento Delegado (UE) 2024/490 da Comissão Europeia (MMTIS — Multimodal Travel Information Services). |
| <a id="REG-02"></a>[REG-02] | Diretiva ITS 2010/40/UE do Parlamento Europeu e do Conselho. |
| <a id="PT-02"></a>[PT-02] | IMT. "Ponto de Acesso Nacional (NAP) Portugal." https://nap-portugal.imt-ip.pt |
| <a id="PT-03"></a>[PT-03] | Instituto da Mobilidade e dos Transportes (IMT-IP). https://www.imt-ip.pt |
| <a id="TOOL-01"></a>[TOOL-01] | Entur. "NeTEx Validator (Java)." https://github.com/entur/netex-validator-java |
| <a id="DATA-03"></a>[DATA-03] | Operadora Exemplo. "Feed GTFS Operadora Exemplo (versão Escolar 228, 2026-04-30)." Feed oficial Operadora Exemplo disponibilizado pela operadora. |
