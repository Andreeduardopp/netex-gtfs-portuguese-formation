# Módulo 8 — Ferramentas Práticas

## Objetivos de Aprendizagem

Após completar este módulo, o leitor será capaz de:

- Validar ficheiros NeTEx com o `netex-validator` da Entur (linha de comandos)
- Interpretar erros de validação e corrigi-los
- Usar conversores GTFS ↔ NeTEx e compreender as suas limitações
- Navegar o schema XSD do NeTEx para encontrar elementos e atributos
- Completar a checklist de implementação antes de submeter ao NAP Portugal

---

## 1. Validação — netex-validator (Entur)

O `netex-validator` é a ferramenta de validação NeTEx de referência, desenvolvida e mantida pela Entur (NAP da Noruega). É open-source e é usada como base por vários NAPs europeus, incluindo o sistema de validação do NAP Portugal. [[TOOL-01]](#TOOL-01)

### 1.1 Instalação

O `netex-validator` requer Java 17+.

```bash
# Descarregar a versão mais recente do GitHub
# https://github.com/entur/netex-validator-java

# Executar com o JAR
java -jar netex-validator-java.jar --input meu_ficheiro.xml

# Ou validar um ZIP
java -jar netex-validator-java.jar --input EXEMPLO_NeTEx_2026-04-30.zip
```

### 1.2 Interpretar os Resultados

O validador produz relatórios com três níveis:

| Nível | Significado | Ação |
|-------|-------------|------|
| `ERROR` | Violação que impede processamento | Corrigir obrigatoriamente |
| `WARNING` | Recomendação não seguida | Corrigir se possível |
| `INFO` | Informação sobre o ficheiro | Nenhuma ação necessária |

**Exemplo de saída:**

```
[ERROR] Line 42: Element 'Line' missing required attribute 'PublicCode'
[ERROR] Line 87: Reference 'PT:EXEMPLO:DayType:XYZ:LOC' not found in document
[WARNING] Line 156: StopPlace missing recommended field 'AccessibilityAssessment'
[INFO] Validated 341 ServiceJourney, 59 ScheduledStopPoint, 2 DayType
```

### 1.3 Erros Mais Frequentes

| Erro | Causa | Solução |
|------|-------|---------|
| `Reference not found` | `ref` aponta para um ID não definido | Verificar se todos os objetos referenciados existem |
| `Missing required attribute` | Campo M do Perfil PT em falta | Adicionar o campo — ver Módulo 7 |
| `Invalid version value` | `version="0"` ou ausente | Usar `version="1"` como base |
| `Encoding error` | Caracteres especiais (ã, ç, é) corrompidos | Garantir que o ficheiro está em UTF-8 sem BOM |
| `Duplicate ID` | Dois objetos com o mesmo `id` | IDs devem ser únicos no CompositeFrame |

---

## 2. Editores XML Recomendados

Para editar ficheiros NeTEx, os editores recomendados são:

### VS Code + Extensão Red Hat XML

```
Extensão: "XML" (Red Hat)
ID: redhat.vscode-xml
```

**Funcionalidades úteis:**
- Validação contra XSD em tempo real (erros sublinhados a vermelho)
- Auto-completar elementos e atributos NeTEx
- Formatação automática do XML
- Navegação de referências

**Configuração recomendada** (`.vscode/settings.json`):

```json
{
  "xml.validation.enabled": true,
  "xml.format.splitAttributes": true,
  "xml.format.joinContentLines": false,
  "[xml]": {
    "editor.formatOnSave": true
  }
}
```

### Oxygen XML Editor

Ferramenta profissional (licença paga). Recomendada para trabalho intensivo com NeTEx:
- Validação contra perfis NeTEx
- Visualização de schema como árvore
- Comparação de ficheiros XML

---

## 3. Conversão GTFS ↔ NeTEx

### 3.1 Estado atual (2026)

A conversão automática entre GTFS e NeTEx é um problema parcialmente resolvido:

| Direção | Ferramentas disponíveis | Completude |
|---------|------------------------|------------|
| GTFS → NeTEx | Chouette (France), Entur tools, MobilityData converters | ~70-80% dos dados |
| NeTEx → GTFS | Chouette, conversores ad-hoc | ~60-70% dos dados |

**O que se perde na conversão GTFS → NeTEx:**
- Acessibilidade (só `wheelchair_*` binário, não AccessibilityLimitation completo)
- StopPlace/Quay (sem hierarquia física no GTFS)
- FareFrame (GTFS Fares v1 limitado; v2 não universal)
- PathLink e transferências físicas

**O que se perde na conversão NeTEx → GTFS:**
- Versioning de objetos
- PassengerStopAssignment (GTFS não separa lógico/físico)
- FareFrame complexo (tarifas avançadas)
- AccessibilityLimitation detalhada

### 3.2 Chouette

O **Chouette** é a ferramenta open-source da AFIMB (França) para gestão e conversão de dados de transporte. Suporta GTFS, NeTEx, e outros formatos. [[TOOL-02]](#TOOL-02)

```
Repositório: https://github.com/entur/chouette
```

### 3.3 Recomendação Prática

Para operadores portugueses que partem de GTFS:

1. Usar o conversor como ponto de partida para os dados de rede e horários
2. Enriquecer manualmente os dados de acessibilidade, tarifas e infraestrutura física
3. Validar com o `netex-validator` após cada enriquecimento
4. Submeter ao NAP Portugal apenas após a checklist completa

---

## 4. Navegar o Schema XSD do NeTEx

O schema XSD completo do NeTEx está disponível no repositório oficial: [[STD-01]](#STD-01)

```
https://github.com/NeTEx-CEN/NeTEx/tree/master/xsd
```

### 4.1 Estrutura do Repositório XSD

```
NeTEx/xsd/
├── NeTEx_publication.xsd      ← Ponto de entrada principal
├── NeTEx_generic_framework/   ← Elementos base (EntityInVersion, etc.)
├── NeTEx_network/             ← Line, Route, ScheduledStopPoint, etc.
├── NeTEx_timetable/           ← ServiceJourney, TimetabledPassingTime, etc.
├── NeTEx_fares/               ← FareFrame, TariffZone, FareProduct, etc.
└── NeTEx_siri/                ← Elementos partilhados com SIRI
```

### 4.2 Como Encontrar um Elemento

**Exemplo: encontrar a definição de `PassengerStopAssignment`**

1. Abrir `NeTEx_publication.xsd` no VS Code
2. Ctrl+Shift+O (Navigate to symbol) → procurar "PassengerStopAssignment"
3. Ou pesquisar no GitHub: `complexType name="PassengerStopAssignment_VersionStructure"`

**Convenção de nomes no XSD:**
- Tipo de elemento: `<NomeDoElemento>_VersionStructure`
- Grupo de elementos: `<NomeDoElemento>Group`
- Tipo de enumeração: `<NomeDaEnumeração>Enumeration`

### 4.3 Verificar Campos Obrigatórios no XSD

No XSD, um campo obrigatório tem `minOccurs="1"` (ou sem `minOccurs`, que por defeito é 1):

```xml
<!-- Campo obrigatório -->
<xsd:element name="Name" type="MultilingualString" minOccurs="1"/>

<!-- Campo opcional -->
<xsd:element name="ShortName" type="MultilingualString" minOccurs="0"/>
```

---

## 5. Checklist de Implementação — Antes de Submeter ao NAP

Use esta checklist antes de cada submissão ao NAP Portugal:

### 5.1 Estrutura e Formato

- [ ] Ficheiro em UTF-8 sem BOM
- [ ] Declaração `<?xml version="1.0" encoding="UTF-8"?>` presente
- [ ] Namespace `xmlns="http://www.netex.org.uk/netex"` correto
- [ ] ZIP nomeado como `<Operador>_NeTEx_<YYYY-MM-DD>.zip`

### 5.2 PublicationDelivery

- [ ] `PublicationTimestamp` com data/hora UTC atual
- [ ] `ParticipantRef` no formato `PT:<org>`
- [ ] `CompositeFrame` com `validBetween` incluindo `FromDate` e `ToDate`

### 5.3 ResourceFrame (obrigatório)

- [ ] `Authority` presente com `id`, `version`, `Name`, `ContactDetails/Email`
- [ ] `Operator` presente com `id`, `version`, `Name`, `ContactDetails/Url`
- [ ] Todos os IDs no formato `PT:<org>:<tipo>:<id>:LOC`

### 5.4 SiteFrame (obrigatório se houver StopPlace)

- [ ] Todos os `StopPlace` com `Name`, `StopPlaceType`, `Centroid/Location`
- [ ] Todos os `Quay` com `Name`, `QuayType`, `Centroid/Location`
- [ ] Cada `Quay` dentro do `StopPlace` correto

### 5.5 ServiceFrame (obrigatório)

- [ ] `Line` com `Name`, `PublicCode`, `TransportMode`
- [ ] Todos os `ScheduledStopPoint` com `Name`
- [ ] `Route` com `LineRef` e `pointsInSequence`
- [ ] `ServiceJourneyPattern` com `RouteRef` e `pointsInSequence`
- [ ] `PassengerStopAssignment` para cada `ScheduledStopPoint`

### 5.6 ServiceCalendarFrame (obrigatório se houver horários)

- [ ] `DayType` com `Name` e `properties/PropertyOfDay`
- [ ] `OperatingDay` para cada data de operação
- [ ] `DayTypeAssignment` ligando cada data ao `DayType` correto

### 5.7 TimetableFrame (obrigatório se houver horários)

- [ ] `ServiceJourney` com `DayTypeRefs` e `JourneyPatternRef`
- [ ] Cada `TimetabledPassingTime` com `StopPointInJourneyPatternRef` e tempo
- [ ] Terminal de partida tem `DepartureTime`; terminal de chegada tem `ArrivalTime`

### 5.8 Referências

- [ ] Todos os `ref` apontam para IDs existentes no mesmo ficheiro ou pacote ZIP
- [ ] Sem IDs duplicados em todo o CompositeFrame
- [ ] Todos os `version` são inteiros positivos (≥1)

### 5.9 Validação Final

- [ ] Validado com `netex-validator` sem erros `[ERROR]`
- [ ] Avisos `[WARNING]` revistos e documentados se não corrigidos
- [ ] Ficheiro ZIP gerado e testado

---

## Exemplos

- [`exemplos/08_01_erros_frequentes.xml`](exemplos/08_01_erros_frequentes.xml) — Erros frequentes na submissão ao NAP Portugal, cada um com comentário explicativo e a correção correspondente

---

## Referências

| ID | Referência |
|----|-----------|
| <a id="STD-01"></a>[STD-01] | CEN. "NeTEx – Network Timetable Exchange." CEN/TS 16614-1:2024. https://github.com/NeTEx-CEN/NeTEx |
| <a id="PT-01"></a>[PT-01] | IMT. "Perfil Nacional NeTEx Portugal." https://ptprofiles.azurewebsites.net |
| <a id="PT-02"></a>[PT-02] | IMT. "NAP Portugal — Portal do Ponto de Acesso Nacional." https://nap-portugal.imt-ip.pt |
| <a id="TOOL-01"></a>[TOOL-01] | Entur AS. "netex-validator-java." https://github.com/entur/netex-validator-java |
| <a id="TOOL-02"></a>[TOOL-02] | Entur AS (fork de AFIMB). "Chouette." https://github.com/entur/chouette |
| <a id="TOOL-03"></a>[TOOL-03] | Red Hat. "VSCode XML Extension." https://marketplace.visualstudio.com/items?itemName=redhat.vscode-xml |
