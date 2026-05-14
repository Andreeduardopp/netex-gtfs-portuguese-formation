# Referências do Projeto NeTEx PT Formação

Este diretório centraliza todas as referências bibliográficas e fontes utilizadas no projeto.

---

## 📖 Índice de Referências

### Standards e Especificações

| ID | Referência | URL | Usado em |
|----|-----------|-----|----------|
| STD-01 | CEN. "NeTEx – Network Timetable Exchange." CEN/TS 16614-1:2024 (Parte 1: Topologia de Rede) | [NeTEx-CEN/NeTEx](https://github.com/NeTEx-CEN/NeTEx) | Módulos 1–8 |
| STD-02 | CEN. "NeTEx – Network Timetable Exchange." CEN/TS 16614-2:2024 (Parte 2: Horários) | [NeTEx-CEN/NeTEx](https://github.com/NeTEx-CEN/NeTEx) | Módulos 3, 4 |
| STD-03 | CEN. "NeTEx – Network Timetable Exchange." CEN/TS 16614-3:2024 (Parte 3: Tarifas) | [NeTEx-CEN/NeTEx](https://github.com/NeTEx-CEN/NeTEx) | Módulo 6 |
| STD-04 | CEN. "Transmodel – Reference Data Model for Public Transport." EN 12896 | [transmodel-cen.eu](https://transmodel-cen.eu) | Módulos 0, 1 |
| STD-05 | CEN. "IFOPT – Identification of Fixed Objects in Public Transport." CEN/TS 28701 | — | Módulo 4 |
| STD-06 | CEN. "NeTEx – Part 6: Passenger Information (EPIAP)." 2024 | — | Módulo 5 |
| STD-07 | MobilityData. "General Transit Feed Specification (GTFS)." | [gtfs.org](https://gtfs.org) | Módulos 0, 2, 3 |

### Regulação Europeia

| ID | Referência | Notas | Usado em |
|----|-----------|-------|----------|
| REG-01 | Regulamento Delegado (UE) 2024/490 da Comissão Europeia (MMTIS) | Obrigação de partilha de dados de transporte | Módulos 0, 7 |
| REG-02 | Diretiva ITS 2010/40/UE | Base legal para NeTEx/SIRI na Europa | Módulo 0 |

### Portugal — Perfil Nacional e Instituições

| ID | Referência | URL | Usado em |
|----|-----------|-----|----------|
| PT-01 | IMT. "Perfil Nacional NeTEx Portugal." | [ptprofiles.azurewebsites.net](https://ptprofiles.azurewebsites.net) | Módulo 7 |
| PT-02 | IMT. "Ponto de Acesso Nacional (NAP) Portugal." | [nap-portugal.imt-ip.pt](https://nap-portugal.imt-ip.pt) | Módulo 7 |
| PT-03 | Instituto da Mobilidade e dos Transportes (IMT-IP) | [imt-ip.pt](https://www.imt-ip.pt) | Módulos 0, 7 |

### Projetos de Referência

| ID | Referência | URL | Relação |
|----|-----------|-----|---------|
| PROJ-01 | Etalab. "Formation NeTEx France." | [github.com/etalab/netex-france-formation](https://github.com/etalab/netex-france-formation) | Projeto base — estrutura pedagógica adaptada |
| PROJ-02 | Open Transport Data Switzerland. "NeTEx Cookbook." | [opentransportdata.swiss](https://opentransportdata.swiss/en/cookbook/timetable-cookbook/netex/) | Referência técnica complementar |

### Ferramentas

| ID | Referência | URL | Usado em |
|----|-----------|-----|----------|
| TOOL-01 | Entur. "NeTEx Validator (Java)." | [github.com/entur/netex-validator-java](https://github.com/entur/netex-validator-java) | Módulo 8 |
| TOOL-02 | MobilityData. "Canonical GTFS Validator." | [github.com/MobilityData/gtfs-validator](https://github.com/MobilityData/gtfs-validator) | Módulo 8 |

### Dados Abertos

| ID | Referência | URL | Usado em |
|----|-----------|-----|----------|
| DATA-01 | Portal de Dados Abertos do Governo Português | [dados.gov.pt](https://dados.gov.pt) | Exemplos |
| DATA-02 | MobilityData. "Mobility Database." | [mobilitydatabase.org](https://mobilitydatabase.org) | Exemplos |
| DATA-03 | Operadora Exemplo. "Feed GTFS Operadora Exemplo (versão Escolar 228, 2026-04-30)." Feed oficial disponibilizado pela operadora. | [exemplo.pt](https://www.exemplo.pt) | Módulos 0–6 |

---

## Como Adicionar Referências

1. Atribua um **ID único** seguindo o prefixo da categoria (`STD-`, `REG-`, `PT-`, `PROJ-`, `TOOL-`, `DATA-`)
2. Preencha todos os campos da tabela
3. No módulo onde a referência é usada, cite-a no formato: `[STD-01]` ou `[REG-01]`
4. Mantenha a secção **Referências** no final de cada módulo com a lista completa de IDs citados

---

*Última atualização: Abril 2026 — adicionado DATA-03 (Operadora Exemplo), usado em Módulos 0–6.*
