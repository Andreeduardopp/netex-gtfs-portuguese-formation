# Módulo 0 — Antes de Começar

## Objetivos de Aprendizagem

Após completar este módulo, o leitor será capaz de:

- Explicar o que é o GTFS, a sua origem e importância para o ecossistema de transporte público
- Identificar os componentes de um feed GTFS e o seu papel no ciclo de informação ao passageiro
- Compreender porque a adoção do GTFS é essencial para operadores portugueses
- Situar o GTFS no contexto europeu (NAPs, Regulamento 2024/490, relação com NeTEx)
- Configurar o ambiente de trabalho para os módulos seguintes

---

## 1. O que é o GTFS e Porque Importa

### 1.1 Definição

**GTFS** (*General Transit Feed Specification*) é o standard global de facto para a troca de dados de transporte público. [[STD-02]](#STD-02) Define um formato aberto, baseado em ficheiros CSV empacotados num arquivo ZIP, para descrever:

- **Operadores** — identificação das empresas de transporte
- **Rotas** — linhas de transporte com modo e identidade comercial
- **Paragens** — locais geográficos de embarque/desembarque
- **Viagens e horários** — sequências de passagem com tempos concretos
- **Calendários** — padrões de serviço e exceções por data
- **Geometria** — traçado geográfico das rotas no mapa

### 1.2 Origem e Contexto

O GTFS nasceu em 2005 de uma colaboração entre a **TriMet** (operador de transporte público de Portland, Oregon) e a **Google**. Bibiana McHugh (TriMet) liderou a iniciativa de criar um formato simples para importar dados de transporte para o Google Maps. [[STD-02]](#STD-02)

O formato foi originalmente designado **Google Transit Feed Specification**. Em 2010, foi renomeado para **General Transit Feed Specification**, marcando a transição de uma iniciativa proprietária da Google para um standard aberto mantido pela comunidade. [[STD-02]](#STD-02)

Os problemas que o GTFS foi desenhado para resolver eram simultaneamente técnicos e organizacionais:

- **Técnicos**: agências armazenavam dados em sistemas incompatíveis — desde horários em papel a software de planeamento proprietário — sem modelo de dados comum
- **Organizacionais**: não existia mecanismo para partilhar dados com programadores terceiros

O GTFS resolveu ambos ao definir um formato simples, aberto e baseado em ficheiros que qualquer agência pode produzir e qualquer programador pode consumir. [[STD-02]](#STD-02)

### 1.3 O GTFS Hoje

Atualmente, o GTFS é utilizado por mais de **10.000 agências de transporte** em mais de **100 países**. [[STD-03]](#STD-03) O standard é mantido pela **MobilityData**, uma organização sem fins lucrativos independente, com refinamento contínuo impulsionado por uma comunidade global de operadores, programadores, consultores e organizações académicas. [[STD-02]](#STD-02) [[STD-03]](#STD-03)

O GTFS alimenta uma vasta gama de aplicações:
- **Planeamento de viagens** — Google Maps, Apple Maps, Transit App, Moovit
- **Criação de horários** — geração automática de quadros horários
- **Visualização de dados** — mapas interativos de redes de transporte
- **Ferramentas de acessibilidade** — informação para passageiros com mobilidade reduzida
- **Análise de transporte** — planeamento de rede, previsão de procura

### 1.4 Porque Importa para Operadores Portugueses

Para operadores de transporte público em Portugal, a adoção do GTFS é relevante por quatro razões:

1. **Visibilidade nas plataformas de mobilidade.** Publicar dados GTFS permite que serviços como Google Maps, Moovit e Transit App incluam o operador nos seus resultados de pesquisa, tornando a rede acessível a milhões de utilizadores.

2. **Cumprimento regulatório europeu.** O Regulamento Delegado (UE) 2024/490 (MMTIS) obriga os estados-membros a publicar dados de transporte em formatos interoperáveis nos Pontos de Acesso Nacionais (NAPs). O GTFS é aceite para dados básicos (paragens, rotas, horários). Para dados avançados (acessibilidade detalhada, tarifas complexas), o NeTEx é obrigatório. [[REG-01]](#REG-01)

3. **Base para a transição para NeTEx.** Muitos operadores portugueses já publicam dados em GTFS. Compreender bem o GTFS é o primeiro passo para a transição para NeTEx, que o regulamento europeu exige para dados avançados.

Para empresas privadas de transporte (rodoviário, ferroviário, serviços de transporte a pedido), adotar GTFS (e posteriormente NeTEx) não é apenas uma questão regulatória, mas uma vantagem competitiva. Operadores privados que disponibilizam dados estruturados e interoperáveis conseguem:

1. **Participar em concursos públicos** com maior qualificação técnica, uma vez que muitos cadernos de encargos já exigem a produção de dados em GTFS/NeTEx.
2. **Integrar-se em plataformas de Mobilidade como Serviço (MaaS)** como a PickMe, a Moovit ou o Google Maps, aumentando a sua visibilidade e potencialmente o número de passageiros.
3. **Aceder a fundos comunitários** (por exemplo, CEEAC, POSEUR, PT2020) que frequentemente condicionam financiamentos à publicação de dados abertos e normalizados.
4. **Reduzir custos de manutenção** dos seus sistemas ao substituir formatos proprietários por padrões abertos, facilitando a interoperabilidade com sistemas de bilhética, informação ao cliente e planeamento de frota.

Em Portugal, a entidade responsável pelo NAP é o **IMT-IP** (Instituto da Mobilidade e dos Transportes). [[PT-01]](#PT-01) [[PT-02]](#PT-02)

---

## 2. Estrutura de um Feed GTFS

### 2.1 Visão Geral

Um feed GTFS é um arquivo ZIP contendo um conjunto de ficheiros de texto (.txt), cada um formatado como CSV (valores separados por vírgula) com uma linha de cabeçalho definida. [[STD-01]](#STD-01)

```
feed_gtfs.zip
├── agency.txt          (obrigatório)
├── stops.txt           (obrigatório)
├── routes.txt          (obrigatório)
├── trips.txt           (obrigatório)
├── stop_times.txt      (obrigatório)
├── calendar.txt        (obrigatório*)
├── calendar_dates.txt  (obrigatório*)
├── shapes.txt          (recomendado)
├── fare_attributes.txt (opcional)
├── fare_rules.txt      (opcional)
├── transfers.txt       (opcional)
├── pathways.txt        (opcional)
├── levels.txt          (opcional)
├── feed_info.txt       (recomendado)
└── frequencies.txt     (opcional)
```

> *Pelo menos um de `calendar.txt` ou `calendar_dates.txt` deve estar presente.

### 2.2 A Natureza Relacional

O GTFS é, na sua essência, uma base de dados relacional exportada como série de ficheiros CSV. [[BP-01]](#BP-01) As relações entre ficheiros funcionam como chaves estrangeiras:

```
agency.txt
  └── routes.txt (agency_id)
        └── trips.txt (route_id)
              ├── stop_times.txt (trip_id)
              │     └── stops.txt (stop_id)
              └── calendar.txt / calendar_dates.txt (service_id)
```

Compreender esta hierarquia relacional, e possuir um banco compativel com essa arquitetura, é fundamental para produzir feeds corretos, visto que uma viagem (`trip`) sem rota (`route`) associada, ou um horário (`stop_time`) sem paragem (`stop`) válida, são erros que invalidam o feed.

### 2.3 Ficheiros Obrigatórios vs Recomendados vs Opcionais

| Categoria | Ficheiros | Notas |
|-----------|-----------|-------|
| **Obrigatório** | `agency`, `stops`, `routes`, `trips`, `stop_times`, `calendar`/`calendar_dates` | Mínimo para um feed funcional |
| **Recomendado** | `shapes`, `feed_info` | Best Practices recomendam fortemente [[BP-01]](#BP-01) |
| **Opcional** | `fare_attributes`, `fare_rules`, `transfers`, `pathways`, `levels`, `frequencies` | Enriquecem o feed com informação adicional |
| **Extensões** | `board_alight` (GTFS-ride), GTFS-Fares v2, GTFS-Flex | Standards complementares em desenvolvimento |

---

## 3. GTFS no Contexto Europeu

### 3.1 O Regulamento MMTIS (2024/490)

O Regulamento Delegado (UE) 2024/490, conhecido como MMTIS (Multimodal Travel Information Services), é a principal obrigação legal europeia para operadores de transporte público. [[REG-01]](#REG-01) Substitui o regulamento anterior (2017/1926).

O regulamento obriga os estados-membros a:

1. Tornar disponíveis dados de transporte público em formatos interoperáveis
2. Publicar esses dados num Ponto de Acesso Nacional (NAP)
3. Garantir qualidade e atualização regular dos dados

### 3.2 GTFS e NeTEx: Papéis Complementares

| Tipo de Dado | GTFS | NeTEx | Obrigação |
|-------------|------|-------|-----------|
| Paragens e redes | Aceite | Obrigatório | Básico |
| Horários | Aceite | Obrigatório | Básico |
| Tarifas básicas | Aceite (limitado) | Obrigatório | Recomendado |
| Acessibilidade detalhada | Limitado (binário) | Obrigatório (EPIAP) | Recomendado |
| Tempo real | GTFS Realtime | SIRI | Obrigatório |

Para a maioria dos operadores portugueses, a estratégia prática é:
1. **Começar com GTFS** — publicar dados básicos de rede e horários
2. **Enriquecer progressivamente** — adicionar shapes, acessibilidade, tarifas
3. **Transitar para NeTEx** — para dados avançados exigidos pelo regulamento

### 3.3 O Papel do NAP Portugal

O Ponto de Acesso Nacional (NAP) é a plataforma onde os operadores publicam os seus dados para consumo por aplicações de mobilidade, investigadores e outros stakeholders. [[PT-01]](#PT-01)

Em Portugal, o NAP, gerido pelo IMT-IP, aceita dados nos formatos GTFS (dados básicos) e NeTEx (dados avançados).

---

## 4. A Importância da Adoção

### 4.1 Benefícios da Standardização

Quando os operadores publicam dados em formato GTFS, garantem: [[STD-03]](#STD-03)

- **Consistência entre regiões** — simplifica o planeamento de viagens para passageiros que usam múltiplos operadores
- **Menor barreira de entrada para programadores** — permite que aplicações de terceiros integrem os dados sem integrações customizadas
- **Base para análise e planeamento** — investigadores e planeadores usam dados GTFS para prever tempos de viagem, analisar padrões espaciais e temporais, e avaliar fiabilidade do serviço

> "GTFS provides a standardized format for transit data, ensuring consistency across agencies and regions, simplifying travel planning for riders." [[STD-03]](#STD-03)

### 4.2 Boas Práticas como Requisito

O GTFS Best Practices Working Group, convocado pelo Rocky Mountain Institute e composto por 17 organizações, estabeleceu que boas práticas coordenadas são essenciais para prevenir requisitos divergentes e datasets incompatíveis. [[BP-01]](#BP-01)

Operadores que adotam GTFS e seguem as suas boas práticas contribuem para um ecossistema de dados de transporte mais saudável e interoperável.

---

## 5. Como Usar Este Curso

### 5.1 Pré-requisitos

Este curso destina-se a quem precisa de produzir, gerir ou consumir feeds GTFS. O público-alvo inclui:

- Técnicos de operadores de transporte público portugueses
- Profissionais de sistemas de informação de mobilidade
- Estudantes de engenharia de transportes
- Programadores que desenvolvem aplicações de mobilidade

O que deve saber antes de começar:
- Noções básicas de ficheiros CSV (abrir, editar, compreender cabeçalhos)
- Familiaridade com folhas de cálculo (Excel, Google Sheets)
- Conceitos básicos de bases de dados relacionais (chave primária, chave estrangeira)

### 5.2 Ferramentas Recomendadas

**Editor de código**: [Visual Studio Code](https://code.visualstudio.com/) (gratuito)

Extensões VS Code recomendadas:
- **Rainbow CSV** — coloração e alinhamento de ficheiros CSV
- **Edit CSV** — edição tabular interativa de CSV
- **GitLens** — navegação do histórico do repositório

Validação de feeds: O Canonical GTFS Validator da MobilityData permite validar feeds GTFS contra a especificação oficial. [[TOOL-01]](#TOOL-01) Será introduzido no Módulo 8.

### 5.3 A Rede Âncora: STCP Porto

Todos os exemplos deste curso são baseados em dados reais da STCP (Sociedade de Transportes Colectivos do Porto). [[DATA-01]](#DATA-01) Usamos a Linha 200 (Bolhão → Castelo do Queijo) como caso principal.

A escolha de dados reais (vs. uma rede fictícia) tem uma razão pedagógica: os exemplos são mais credíveis quando podemos verificar que correspondem a uma realidade que o leitor conhece ou pode consultar.

Os ficheiros GTFS de base estão disponíveis em `gtfs-pt-formacao/exemplos/rede_exemplo_portugal/gtfs/` (raiz do repositório).

### 5.4 Estrutura de Cada Módulo

Cada módulo segue esta estrutura:

1. **Objetivos** — o que vai aprender
2. **Conteúdo** — explicação dos conceitos, com tabelas e exemplos CSV
3. **Exemplos** — ficheiros CSV na pasta `exemplos/` do módulo
4. **Exercícios** — exercícios práticos com solução
5. **Referências** — todas as fontes citadas no módulo

### 5.5 Como Navegar os Módulos

O curso é **sequencial**: cada módulo constrói sobre o anterior. A progressão recomendada é:

```
Módulo 0 (este)
    ↓
Módulo 1 — Conceitos Base (modelo de dados, ZIP, 6 ficheiros obrigatórios)
    ↓
Módulo 2 — Agência, Paragens e Rotas (agency.txt, stops.txt, routes.txt)
    ↓
Módulo 3 — Viagens, Horários e Geometria (trips.txt, stop_times.txt, shapes.txt)
    ↓
Módulo 4 — Calendários e Exceções (calendar.txt, calendar_dates.txt)
    ↓
Módulo 5 — Acessibilidade e Transferências (wheelchair_*, pathways, transfers)
    ↓
Módulo 6 — Ciclo de Vida dos Dados (recolha, armazenamento, gestão)
    ↓
Módulo 7 — Tarifas e Extensões (fares, GTFS-ride, GTFS-Flex)
    ↓
Módulo 8 — Ferramentas Práticas (validação, bibliotecas Python, publicação)
```

---

## Referências

| ID | Referência |
|----|-----------|
| <a id="STD-01"></a>[STD-01] | MobilityData. "GTFS Schedule Reference." https://gtfs.org/schedule/reference |
| <a id="STD-02"></a>[STD-02] | MobilityData. "About GTFS." https://gtfs.org/about |
| <a id="STD-03"></a>[STD-03] | MobilityData. "Why use GTFS?" https://gtfs.org/getting-started/why-use-gtfs |
| <a id="BP-01"></a>[BP-01] | MobilityData. "GTFS Schedule Best Practices." https://gtfs.org/documentation/schedule/schedule-best-practices |
| <a id="REG-01"></a>[REG-01] | Regulamento Delegado (UE) 2024/490 da Comissão Europeia (MMTIS). |
| <a id="PT-01"></a>[PT-01] | IMT. "Ponto de Acesso Nacional (NAP) Portugal." https://nap-portugal.imt-ip.pt |
| <a id="PT-02"></a>[PT-02] | Instituto da Mobilidade e dos Transportes (IMT-IP). https://www.imt-ip.pt |
| <a id="TOOL-01"></a>[TOOL-01] | MobilityData. "Canonical GTFS Validator." https://github.com/MobilityData/gtfs-validator |
| <a id="DATA-01"></a>[DATA-01] | STCP. "Feed GTFS STCP Porto (versão Escolar 228, 2026-04-30)." Feed oficial STCP. |
