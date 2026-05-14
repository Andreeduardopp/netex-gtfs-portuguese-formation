# Registo Central de Referências

> Todas as referências citadas nos módulos devem estar registadas aqui. Antes de citar uma nova fonte num módulo, adicioná-la primeiro a este ficheiro.

---

## Standards e Especificações

| ID | Referência | URL |
|----|-----------|-----|
| **STD-01** | MobilityData. "GTFS Schedule Reference." Especificação oficial do formato GTFS. | https://gtfs.org/schedule/reference |
| **STD-02** | MobilityData. "About GTFS." História e contexto do standard. | https://gtfs.org/about |
| **STD-03** | MobilityData. "Why use GTFS?" Benefícios da adoção do standard. | https://gtfs.org/getting-started/why-use-gtfs |
| **STD-04** | GTFS-ride. Extensão para dados de ridership (passageiros). | https://gtfsride.org |
| **STD-05** | MobilityData. "GTFS-Fares v2." Extensão para tarifas avançadas. | https://gtfs.org/extensions/fares-v2 |
| **STD-06** | MobilityData. "GTFS-Flex." Extensão para serviços de transporte flexível. | https://gtfs.org/extensions/flex |
| **STD-07** | MobilityData. "GTFS Realtime Reference." Dados em tempo real. | https://gtfs.org/realtime/reference |

---

## Boas Práticas

| ID | Referência | URL |
|----|-----------|-----|
| **BP-01** | MobilityData. "GTFS Schedule Best Practices." Recomendações oficiais para produção de feeds GTFS. | https://gtfs.org/documentation/schedule/schedule-best-practices |
| **BP-02** | Transit App. "Guidelines for Producing GTFS Static Data for Transit." | https://resources.transitapp.com/article/458-guidelines-for-producing-gtfs-static-data-for-transit |

---

## Regulação Europeia

| ID | Referência | URL |
|----|-----------|-----|
| **REG-01** | Regulamento Delegado (UE) 2024/490 da Comissão Europeia (MMTIS — Multimodal Travel Information Services). | https://eur-lex.europa.eu/legal-content/PT/TXT/?uri=OJ:L_202400490 |
| **REG-02** | Diretiva ITS 2010/40/UE do Parlamento Europeu e do Conselho (Sistemas de Transporte Inteligentes). | https://eur-lex.europa.eu/legal-content/PT/TXT/?uri=CELEX:32010L0040 |

---

## Portugal — Instituições e NAP

| ID | Referência | URL |
|----|-----------|-----|
| **PT-01** | IMT. "Ponto de Acesso Nacional (NAP) Portugal." Plataforma de dados de transporte. | https://nap-portugal.imt-ip.pt |
| **PT-02** | Instituto da Mobilidade e dos Transportes (IMT-IP). Entidade reguladora. | https://www.imt-ip.pt |

---

## Ferramentas

| ID | Referência | URL |
|----|-----------|-----|
| **TOOL-01** | MobilityData. "Canonical GTFS Validator." Validador oficial de feeds GTFS (Java). | https://github.com/MobilityData/gtfs-validator |
| **TOOL-02** | Remix / Conveyal. "partridge." Biblioteca Python para leitura rápida de GTFS. | https://github.com/remix/partridge |
| **TOOL-03** | PID-GTFS. Pipeline Python para validação, importação e exportação de feeds GTFS (este projeto). | `../PID-GTFS/` |

---

## Dados Abertos

| ID | Referência | URL |
|----|-----------|-----|
| **DATA-01** | Operadora Exemplo. "Feed GTFS Operadora Exemplo (versão Escolar 228, 2026-04-30)." Feed oficial Operadora Exemplo disponibilizado pela operadora. | — |
| **DATA-02** | MobilityData. "Mobility Database." Agregador global de feeds GTFS. | https://mobilitydatabase.org |
| **DATA-03** | "European transport feeds." EU National Access Points Feed Registry. | https://eu.data.public-transport.earth |
| **DATA-04** | Portal de Dados Abertos da Administração Pública Portuguesa. | https://dados.gov.pt |

---

## Como adicionar uma nova referência

1. Escolher o prefixo adequado: `STD-` (standards), `BP-` (boas práticas), `REG-` (regulação), `PT-` (Portugal), `TOOL-` (ferramentas), `DATA-` (dados)
2. Atribuir o número sequencial seguinte dentro da categoria
3. Preencher: **ID**, **Referência** (autor, título, ano), **URL** (se aplicável)
4. Citar no módulo usando o formato `[[STD-01]](#STD-01)` (com âncora) ou `[STD-01]` (sem âncora)

---

*Última atualização: Maio 2026 — criação do registo inicial.*
