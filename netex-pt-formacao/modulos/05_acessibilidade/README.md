# Módulo 5 — Acessibilidade

## Objetivos de Aprendizagem

Após completar este módulo, o leitor será capaz de:

- Compreender a diferença entre acessibilidade do veículo e acessibilidade da infraestrutura em NeTEx
- Aplicar `AccessibilityAssessment` e `AccessibilityLimitation` a `ServiceJourney`, `StopPlace` e `Quay`
- Usar correctamente os valores de `LimitationStatusEnumeration`: `yes`, `no`, `partial`, `unknown`
- Modelar rampas, elevadores, piso tátil, informação sonora e informação visual
- Compreender o perfil europeu EPIAP (NeTEx Parte 6) e as obrigações regulatórias
- Interpretar o mapeamento de `wheelchair_accessible` e `wheelchair_boarding` do GTFS para NeTEx

---

## 1. O Problema: GTFS Tem Acessibilidade Binária

No GTFS, a acessibilidade para pessoas com mobilidade condicionada é representada com um campo simples:

```
trips.txt:    wheelchair_accessible = 0 (desconhecido) | 1 (acessível) | 2 (não acessível)
stops.txt:    wheelchair_boarding   = 0 (desconhecido) | 1 (acessível) | 2 (não acessível)
```

Esta codificação binária é insuficiente para vários cenários reais:

| Situação | GTFS | Limitação |
|----------|------|-----------|
| Rampa disponível mas só com assistência | `wheelchair_accessible=1` | Não distingue independente de com ajuda |
| Paragem parcialmente acessível (rampa existe mas elevação excessiva) | Não representável | Sem valor parcial |
| Informação sonora no veículo | Não existe campo | Passageiros invisuais não contemplados |
| Piso tátil na plataforma | Não existe campo | Deficiência visual não modelada |
| Elevador avariado temporariamente | Não representável | Sem mecanismo de estado temporal |

O NeTEx resolve isto com um modelo de acessibilidade multi-dimensional que cobre:
- Utilizadores de cadeira de rodas
- Utilizadores de bengala / mobilidade reduzida
- Pessoas com deficiência visual
- Pessoas com deficiência auditiva
- Grávidas e passageiros com crianças (buggy)
- Passageiros com bagagem volumosa

---

## 2. AccessibilityAssessment — O Elemento Central

O `AccessibilityAssessment` é o container que agrega todas as informações de acessibilidade de um objeto. [[STD-01]](#STD-01)

Pode ser associado a:
- `ServiceJourney` — acessibilidade do veículo nesta viagem concreta
- `StopPlace` / `Quay` — acessibilidade da infraestrutura física
- `Line` — acessibilidade geral da linha

### 2.1 Estrutura Básica

```xml
<AccessibilityAssessment id="PT:EXEMPLO:AA:200_DU_0600:LOC" version="1">
  <!-- MobilityImpairedAccess = avaliação geral (yes/no/partial/unknown) -->
  <MobilityImpairedAccess>yes</MobilityImpairedAccess>

  <limitations>
    <AccessibilityLimitation>
      <!-- Cada dimensão de acessibilidade é avaliada separadamente -->
      <WheelchairAccess>yes</WheelchairAccess>
      <StepFreeAccess>yes</StepFreeAccess>
      <EscalatorFreeAccess>yes</EscalatorFreeAccess>
      <LiftFreeAccess>yes</LiftFreeAccess>
      <AudibleSignalsAvailable>yes</AudibleSignalsAvailable>
      <VisualSignsAvailable>yes</VisualSignsAvailable>
    </AccessibilityLimitation>
  </limitations>
</AccessibilityAssessment>
```

### 2.2 LimitationStatusEnumeration — Os Quatro Valores

Cada campo de limitação aceita um de quatro valores: [[STD-01]](#STD-01)

| Valor | Significado | Quando usar |
|-------|-------------|-------------|
| `yes` | Totalmente acessível | Sem restrições para este critério |
| `no` | Não acessível | Impossível para este utilizador |
| `partial` | Parcialmente acessível | Possível com condicionalismos (ex: rampa com inclinação elevada, assistência necessária) |
| `unknown` | Desconhecido | Informação não disponível |

**Importante**: `unknown` é diferente de `no`. Um operador que não avaliou uma paragem deve usar `unknown`, não `no`. Sistemas de pesquisa de percursos podem filtrar `no` mas apresentar `unknown` com aviso.

---

## 3. Campos de AccessibilityLimitation

### 3.1 Relacionados com Mobilidade Física

| Campo NeTEx | Descrição | Equivalente GTFS |
|-------------|-----------|------------------|
| `WheelchairAccess` | Acesso para cadeira de rodas (veículo ou paragem) | `wheelchair_accessible` / `wheelchair_boarding` |
| `StepFreeAccess` | Percurso sem degraus desde a entrada até ao lugar/plataforma | *(não existe)* |
| `EscalatorFreeAccess` | Existe alternativa às escadas rolantes (rampas/elevadores) | *(não existe)* |
| `LiftFreeAccess` | Existe alternativa ao elevador (rampas) | *(não existe)* |

### 3.2 Relacionados com Informação

| Campo NeTEx | Descrição | Equivalente GTFS |
|-------------|-----------|------------------|
| `AudibleSignalsAvailable` | Sinais sonoros disponíveis (anúncios de paragem, etc.) | *(não existe)* |
| `VisualSignsAvailable` | Sinais visuais disponíveis (painéis, informação escrita) | *(não existe)* |

### 3.3 Comparação de Cenários Reais

```
Autocarro de piso baixo com rampa automática:
  WheelchairAccess=yes, StepFreeAccess=yes

Autocarro com plataforma elevatória (necessita assistência):
  WheelchairAccess=partial, StepFreeAccess=partial

Paragem com lancil rebaixado mas sem piso tátil:
  WheelchairAccess=yes, StepFreeAccess=yes, VisualSignsAvailable=no

Estação de metro com elevador (pode estar avariado):
  LiftFreeAccess=no  (não existe alternativa ao elevador para subir à plataforma)
  WheelchairAccess=yes  (quando o elevador funciona)
```

---

## 4. AccessibilityAssessment na ServiceJourney vs StopPlace

A distinção entre acessibilidade do veículo e da infraestrutura é uma contribuição valiosa do NeTEx para a acessibilidade real dos sistemas de transporte. [[STD-01]](#STD-01)

```
Um percurso verdadeiramente acessível requer DOIS componentes:

  Veículo acessível              E              Paragem acessível
  ─────────────────────               ────────────────────────────
  ServiceJourney →                     StopPlace / Quay →
  AccessibilityAssessment              AccessibilityAssessment
  (piso baixo, rampa, sinais)          (lancil, piso tátil, rampa)
```

Se apenas o veículo for acessível mas a paragem não tiver lancil rebaixado, o passageiro de cadeira de rodas não consegue embarcar. O NeTEx modela ambos separadamente para que os sistemas possam verificar a cadeia completa.

### Mapeamento do GTFS

| GTFS | NeTEx | Onde |
|------|-------|------|
| `trips.txt: wheelchair_accessible=1` | `AccessibilityAssessment/WheelchairAccess=yes` | `ServiceJourney` |
| `trips.txt: wheelchair_accessible=2` | `AccessibilityAssessment/WheelchairAccess=no` | `ServiceJourney` |
| `trips.txt: wheelchair_accessible=0` | `AccessibilityAssessment/WheelchairAccess=unknown` | `ServiceJourney` |
| `stops.txt: wheelchair_boarding=1` | `AccessibilityAssessment/WheelchairAccess=yes` | `StopPlace` / `Quay` |
| `stops.txt: wheelchair_boarding=2` | `AccessibilityAssessment/WheelchairAccess=no` | `StopPlace` / `Quay` |
| `stops.txt: wheelchair_boarding=0` | `AccessibilityAssessment/WheelchairAccess=unknown` | `StopPlace` / `Quay` |

---

## 5. EPIAP — Perfil Europeu de Acessibilidade

O **EPIAP** (European Passenger Information for Accessibility Profile) é o perfil NeTEx definido na Parte 6 da especificação CEN/TS 16614. [[STD-06]](#STD-06)

É o perfil que os operadores devem seguir para conformidade com o Regulamento (UE) 2024/490 (MMTIS) no que respeita a dados de acessibilidade. [[REG-01]](#REG-01)

### Campos Obrigatórios no EPIAP

Para um `StopPlace` submeter ao NAP com dados de acessibilidade:

| Campo | Obrigatoriedade | Notas |
|-------|----------------|-------|
| `AccessibilityAssessment/MobilityImpairedAccess` | **Obrigatório** | Avaliação geral |
| `AccessibilityLimitation/WheelchairAccess` | **Obrigatório** | Pelo menos `unknown` |
| `AccessibilityLimitation/StepFreeAccess` | Recomendado | — |
| `AccessibilityLimitation/AudibleSignalsAvailable` | Recomendado | — |
| `AccessibilityLimitation/VisualSignsAvailable` | Recomendado | — |

### Contexto Português

O Perfil Nacional NeTEx Portugal (PT-EPIP) alinha com o EPIAP europeu e acrescenta requisitos específicos para submissão ao NAP Portugal. [[PT-01]](#PT-01)

A IMT publicou orientações sobre os campos de acessibilidade obrigatórios para operadores portugueses. Para dados rigorosos, os operadores devem fazer levantamento de campo das suas paragens.

---

## 6. Exemplo: Paragem Acessível vs Paragem com Limitações

### Cenário A — Quay totalmente acessível (nova paragem renovada)

```xml
<Quay id="PT:EXEMPLO:Quay:Bolhao_IDA:LOC" version="2">
  <Name>Bolhão — Sentido Castelo do Queijo</Name>
  <Centroid>...</Centroid>
  <QuayType>busStop</QuayType>

  <!-- AccessibilityAssessment da INFRAESTRUTURA (não do veículo) -->
  <AccessibilityAssessment id="PT:EXEMPLO:AA:Quay_Bolhao_IDA:LOC" version="1">
    <MobilityImpairedAccess>yes</MobilityImpairedAccess>
    <limitations>
      <AccessibilityLimitation>
        <WheelchairAccess>yes</WheelchairAccess>        <!-- lancil rebaixado -->
        <StepFreeAccess>yes</StepFreeAccess>             <!-- sem degraus -->
        <EscalatorFreeAccess>yes</EscalatorFreeAccess>  <!-- sem escadas rolantes -->
        <LiftFreeAccess>yes</LiftFreeAccess>             <!-- sem necessidade de elevador -->
        <AudibleSignalsAvailable>yes</AudibleSignalsAvailable>  <!-- sinais sonoros no poste -->
        <VisualSignsAvailable>yes</VisualSignsAvailable>        <!-- painel de informação -->
      </AccessibilityLimitation>
    </limitations>
  </AccessibilityAssessment>
</Quay>
```

### Cenário B — Quay com limitações (paragem antiga sem obras)

```xml
<Quay id="PT:EXEMPLO:Quay:Exemplo_Antigo:LOC" version="1">
  <Name>Exemplo — Paragem Antiga</Name>
  <QuayType>busStop</QuayType>

  <AccessibilityAssessment id="PT:EXEMPLO:AA:Quay_Antigo:LOC" version="1">
    <!-- MobilityImpairedAccess=partial → possível mas com condicionalismos -->
    <MobilityImpairedAccess>partial</MobilityImpairedAccess>
    <limitations>
      <AccessibilityLimitation>
        <WheelchairAccess>partial</WheelchairAccess>   <!-- lancil alto, difícil mas possível -->
        <StepFreeAccess>yes</StepFreeAccess>            <!-- não há degraus na aproximação -->
        <AudibleSignalsAvailable>no</AudibleSignalsAvailable>   <!-- sem sinais sonoros -->
        <VisualSignsAvailable>partial</VisualSignsAvailable>    <!-- só nome da paragem -->
      </AccessibilityLimitation>
    </limitations>
  </AccessibilityAssessment>
</Quay>
```

---

## Exemplos

- [`exemplos/05_01_paragem_acessivel.xml`](exemplos/05_01_paragem_acessivel.xml) — Comparação lado-a-lado: paragem Bolhão IDA (totalmente acessível) vs paragem com limitações. Inclui acessibilidade na ServiceJourney e na infraestrutura.

---

## Exercícios

- [Exercício 5 — Auditar a acessibilidade da Linha 200](exercicios/exercicio_05.md)

---

## Referências

| ID | Referência |
|----|-----------|
| <a id="STD-01"></a>[STD-01] | CEN. "NeTEx – Network Timetable Exchange." CEN/TS 16614-1:2024 (Parte 1). https://github.com/NeTEx-CEN/NeTEx |
| <a id="STD-06"></a>[STD-06] | CEN. "NeTEx – Network Timetable Exchange." CEN/TS 16614-6:2024 (Parte 6 — EPIAP). https://github.com/NeTEx-CEN/NeTEx |
| <a id="REG-01"></a>[REG-01] | Regulamento Delegado (UE) 2024/490 da Comissão, de 14 de Dezembro de 2023. JO L, 2024/490. https://eur-lex.europa.eu/legal-content/PT/TXT/?uri=OJ:L_202400490 |
| <a id="PT-01"></a>[PT-01] | IMT. "Perfil Nacional NeTEx Portugal." https://ptprofiles.azurewebsites.net |
| <a id="DATA-03"></a>[DATA-03] | Operadora Exemplo. "Feed GTFS Operadora Exemplo (versão Escolar 228, 2026-04-30)." Feed oficial Operadora Exemplo. |
