# Módulo 7 — Perfil NeTEx Portugal


## Objetivos de Aprendizagem

Após completar este módulo, o leitor será capaz de:

- Compreender o que é um perfil nacional NeTEx e porque Portugal tem o seu
- Consultar o Perfil Nacional em `ptprofiles.azurewebsites.net`
- Identificar os campos obrigatórios vs opcionais no contexto português
- Aplicar as convenções de identificadores do perfil PT (formato IFOPT)
- Preparar um ficheiro NeTEx conforme com o Perfil PT para submissão ao NAP
- Compreender o processo de submissão ao NAP Portugal (`nap-portugal.imt-ip.pt`)

---

## 1. O que é um Perfil Nacional NeTEx?

O NeTEx (CEN/TS 16614) é uma especificação muito vasta, cobre desde paragens simples até tarifas complexas, acessibilidade detalhada e gestão operacional. Um ficheiro NeTEx "completo" pode ter centenas de elementos possíveis.

Um perfil nacional define um subconjunto obrigatório e recomendado desses elementos para um país específico. O perfil responde às seguintes questões:

- Quais os elementos que o NAP nacional exige obrigatoriamente?
- Quais os campos que são recomendados mas opcionais?
- Que convenções de nomenclatura e identificadores devem ser usadas?
- Qual a estrutura de ficheiros esperada na submissão?

Sem um perfil, cada operador poderia produzir NeTEx válido mas incompatível com o sistema de receção nacional.

### Perfis Nacionais Europeus

| País | Perfil | NAP |
|------|--------|-----|
| Portugal | PT-EPIP (ptprofiles.azurewebsites.net) | nap-portugal.imt-ip.pt |
| França | NETEX_FRANCE (etalab/gtfs-rt-france, AFIMB) | transport.data.gouv.fr |
| Noruega | NeTEx Nordic (Entur) | entur.no |
| Alemanha | DELFI-Profile (VDV) | mdv.de / MCloud |
| Reino Unido | NeTEx UK (DfT) | data.bus-data.dft.gov.uk |

---

## 2. Perfil Nacional NeTEx Portugal (PT-EPIP)

O Perfil Nacional NeTEx Portugal, também referido como **PT-EPIP** (European Passenger Information Profile — adaptação nacional), está publicado em: [[PT-01]](#PT-01)

```
https://ptprofiles.azurewebsites.net
```

É mantido pelo IMT (Instituto da Mobilidade e dos Transportes) e define os requisitos para a submissão de dados ao NAP Portugal.

### 2.1 Estrutura do Perfil PT

O perfil PT organiza os requisitos em três níveis:

| Nível | Símbolo | Significado |
|-------|---------|-------------|
| Obrigatório | **M** (Mandatory) | O elemento DEVE estar presente; sem ele, a submissão falha validação |
| Recomendado | **R** (Recommended) | O elemento DEVERIA estar presente; ausência gera aviso |
| Opcional | **O** (Optional) | O elemento PODE estar presente; sem penalidade |

### 2.2 Campos Obrigatórios 
- #### ResourceFrame (Authority e Operator)

| Elemento | Campo | Nível |
|---------|-------|-------|
| `Authority` | `id`, `version`, `Name` | **M** |
| `Authority` | `ContactDetails/Email` | **M** |
| `Authority` | `OrganisationType=authority` | **M** |
| `Operator` | `id`, `version`, `Name` | **M** |
| `Operator` | `ContactDetails/Url` | **M** |
| `Operator` | `OrganisationType=operator` | **M** |

- #### ServiceFrame (Rede)

| Elemento | Campo | Nível |
|---------|-------|-------|
| `Network` | `id`, `version`, `Name` | **M** |
| `Line` | `id`, `version`, `Name`, `TransportMode` | **M** |
| `Line` | `PublicCode` | **M** |
| `ScheduledStopPoint` | `id`, `version`, `Name` | **M** |
| `Route` | `id`, `version`, `LineRef` | **M** |
| `ServiceJourneyPattern` | `id`, `version`, `RouteRef` | **M** |
| `PassengerStopAssignment` | `ScheduledStopPointRef`, `StopPlaceRef`, `QuayRef` | **M** |

- #### SiteFrame (Infraestrutura)

| Elemento | Campo | Nível |
|---------|-------|-------|
| `StopPlace` | `id`, `version`, `Name`, `StopPlaceType` | **M** |
| `StopPlace` | `Centroid/Location` | **M** |
| `Quay` | `id`, `version`, `Name`, `QuayType` | **M** |
| `Quay` | `Centroid/Location` | **M** |

- #### TimetableFrame (Horários)

| Elemento | Campo | Nível |
|---------|-------|-------|
| `ServiceJourney` | `id`, `version`, `DayTypeRefs`, `JourneyPatternRef` | **M** |
| `TimetabledPassingTime` | `StopPointInJourneyPatternRef`, `DepartureTime` ou `ArrivalTime` | **M** |
| `DayType` | `id`, `version` | **M** |
| `DayTypeAssignment` | `DayTypeRef`, `OperatingDayRef` ou `Date` | **M** |

---

## 3. Convenções de Identificadores no Perfil PT

O Perfil PT adota o formato **IFOPT** (CEN/TS 28701) para identificadores: [[STD-05]](#STD-05)

```
PT:<CodigoOrganizacao>:<TipoDeObjecto>:<IdLocal>:LOC
```

### 3.1 Código de Organização

| Organização | Código |
|------------|--------|
| IMT (autoridade nacional) | `IMT` |
| STCP (Porto) | `STCP` |
| Carris (Lisboa) | `Carris` |
| Metro do Porto | `MetroPorto` |
| Metro de Lisboa | `MetroLisboa` |
| CP | `CP` |
| Transtejo | `Transtejo` |
| Fertagus | `Fertagus` |
| Sistema Andante | `Andante` |
| AML (Área Metropolitana Lisboa) | `AML` |

*Nota: os códigos exatos são definidos pelo IMT; os acima são convenções deste curso.*

### 3.2 Tipo de Objecto

| Tipo | Exemplo de ID |
|------|---------------|
| `Authority` | `PT:IMT:Authority:IMT:LOC` |
| `Operator` | `PT:STCP:Operator:STCP:LOC` |
| `Line` | `PT:STCP:Line:200:LOC` |
| `StopPlace` | `PT:STCP:StopPlace:Bolhao:LOC` |
| `Quay` | `PT:STCP:Quay:Bolhao_IDA:LOC` |
| `ScheduledStopPoint` | `PT:STCP:ScheduledStopPoint:BLRB1:LOC` |
| `ServiceJourney` | `PT:STCP:ServiceJourney:200_DU_0600:LOC` |

### 3.3 O sufixo `:LOC`

O sufixo `:LOC` indica que o identificador é **local** (definido pelo operador), em oposição a `:NAT` (nacional, atribuído pelo NAP) ou `:GEN` (genérico).

Na submissão ao NAP, o sistema pode reescrever IDs `:LOC` para IDs `:NAT` após validação.

---

## 4. Estrutura de Ficheiros para o NAP Portugal

O NAP Portugal aceita ficheiros NeTEx como **pacote ZIP** com a seguinte estrutura recomendada: [[PT-01]](#PT-01)

```
<operador>_<tipo>_<versao>.zip
│
├── _network.xml          — ResourceFrame + ServiceFrame (rede, linhas, paragens)
├── _timetable.xml        — TimetableFrame + ServiceCalendarFrame (horários)
├── _stops.xml            — SiteFrame (infraestrutura física)
└── _fares.xml            — FareFrame (tarifas, opcional)
```

**Convenção de nomenclatura do ZIP:**

```
STCP_NeTEx_2026-04-30.zip
```

### 4.1 PublicationDelivery obrigatório em todos os ficheiros

```xml
<PublicationDelivery version="1.0">
  <PublicationTimestamp>2026-04-30T00:00:00Z</PublicationTimestamp>
  <ParticipantRef>PT:STCP</ParticipantRef>
  ...
</PublicationDelivery>
```

O `PublicationTimestamp` deve ser a data de geração do ficheiro. O `ParticipantRef` deve ser o código do operador submetente.

---

## 5. Comparação Perfil PT vs Perfil Francês

Para quem vier do projeto `etalab/netex-france-formation`, as diferenças principais são: [[PROJ-01]](#PROJ-01)

| Aspeto | Perfil PT | Perfil França |
|--------|-----------|---------------|
| Formato ID | `PT:<org>:<tipo>:<id>:LOC` | `FR:<org>:<tipo>:<id>:LOC` |
| NAP | nap-portugal.imt-ip.pt | transport.data.gouv.fr |
| Formato de entrega | ZIP com múltiplos XML | ZIP ou XML único |
| Ferramenta de validação | IMT validator (beta) | Entur netex-validator |
| Perfil base | EPIP (CEN TS 16614) | NeTEx Nordic + EPIP |
| Documentação | ptprofiles.azurewebsites.net | normes.transport.data.gouv.fr |
| Status obrigatoriedade | Regulamento 2024/490 (Em vigor) | Décret 2023 (Em vigor) |

---

## 6. Processo de Submissão ao NAP Portugal

### Passo a passo

1. **Preparar os dados** — Gerar ficheiros NeTEx conformes com o Perfil PT
2. **Validar localmente** — Usar o validador disponível em ptprofiles.azurewebsites.net
3. **Criar conta no NAP** — Registar em nap-portugal.imt-ip.pt como operador
4. **Submeter o ZIP** — Upload na interface web do NAP ou via API
5. **Verificar validação** — O NAP valida automaticamente e reporta erros
6. **Publicar** — Após validação, os dados ficam públicos no NAP

### Erros Comuns na Submissão

| Erro | Causa | Solução |
|------|-------|---------|
| "ID inválido" | Formato ID não segue convenção IFOPT | Verificar formato `PT:<org>:<tipo>:<id>:LOC` |
| "Campo obrigatório ausente" | Element M em falta | Consultar tabelas da Secção 2 deste módulo |
| "Referência inválida" | `ref` aponta para ID inexistente | Verificar se todos os objetos referenciados estão definidos |
| "Versão inválida" | `version` não é inteiro positivo | Usar `version="1"` como base |
| "Encoding inválido" | Ficheiro não é UTF-8 | Guardar como UTF-8 sem BOM |

---

## Exemplos

- [`exemplos/07_01_checklist_submissao.xml`](exemplos/07_01_checklist_submissao.xml) — Ficheiro NeTEx mínimo conforme com o Perfil PT: todos os campos obrigatórios presentes, IDs no formato correto, pronto para submissão ao NAP

---

## Exercícios

- [Exercício 7 — Auditar e corrigir um ficheiro NeTEx com erros de conformidade](exercicios/exercicio_07.md)

---

## Referências

| ID | Referência |
|----|-----------|
| <a id="STD-01"></a>[STD-01] | CEN. "NeTEx – Network Timetable Exchange." CEN/TS 16614-1:2024 (Parte 1). https://github.com/NeTEx-CEN/NeTEx |
| <a id="STD-05"></a>[STD-05] | CEN. "IFOPT – Identification of Fixed Objects in Public Transport." CEN/TS 28701. |
| <a id="REG-01"></a>[REG-01] | Regulamento Delegado (UE) 2024/490 da Comissão, de 14 de Dezembro de 2023. JO L, 2024/490. https://eur-lex.europa.eu/legal-content/PT/TXT/?uri=OJ:L_202400490 |
| <a id="PT-01"></a>[PT-01] | IMT. "Perfil Nacional NeTEx Portugal." https://ptprofiles.azurewebsites.net |
| <a id="PT-02"></a>[PT-02] | IMT. "NAP Portugal — Portal do Ponto de Acesso Nacional." https://nap-portugal.imt-ip.pt |
| <a id="PROJ-01"></a>[PROJ-01] | etalab / Direction interministérielle du numérique. "netex-france-formation." https://github.com/etalab/netex-france-formation |
| <a id="DATA-03"></a>[DATA-03] | STCP. "Feed GTFS STCP Porto (versão Escolar 228, 2026-04-30)." Feed oficial STCP. |
