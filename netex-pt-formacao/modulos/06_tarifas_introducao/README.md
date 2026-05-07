# Módulo 6 — Tarifas (Introdução)

## Objetivos de Aprendizagem

Após completar este módulo, o leitor será capaz de:

- Compreender a estrutura do `FareFrame` e porque o GTFS não tem equivalente direto
- Modelar zonas tarifárias com `TariffZone` e ligar paragens a zonas
- Criar um produto tarifário simples com `PreassignedFareProduct`
- Representar o passe Andante (Porto) em NeTEx
- Compreender as limitações deste módulo e onde está a especificação completa de tarifas

---

## 1. A Limitação:

O GTFS tem uma extensão opcional chamada `fare_attributes.txt` / `fare_rules.txt` (versão legada) e uma nova `fares v2` (ainda em discussão), mas nenhuma delas é universalmente adotada ou suficientemente expressiva para o sistema tarifário português.

| GTFS Fares | Limitações |
|-----------|-----------|
| `fare_attributes.txt` | Só suporta preços fixos por ligação ou zona simples |
| `fare_rules.txt` | Regras por `origin_id`/`destination_id` mas sem produtos tarifários |
| GTFS Fares v2 | Ainda extensão experimental, não no perfil PT |
| Passes/assinaturas | Não modeláveis em GTFS |
| Títulos com validade temporal | Não modeláveis em GTFS |
| Descontos (jovem, sénior, estudante) | Não modeláveis em GTFS |

O NeTEx (Parte 3 — Fares) tem um modelo de tarifas completo e muito expressivo. Este módulo cobre apenas os conceitos introdutórios, ficando a especificação completa de tarifas para uma versão futura do curso.

---

## 2. O FareFrame

O **`FareFrame`** é o frame NeTEx dedicado às tarifas. [[STD-03]](#STD-03)

```xml
<FareFrame id="PT:STCP:FareFrame:Andante_FF:LOC" version="1">
  <Name>Andante Porto — Tarifas Base</Name>

  <tariffZones> ... </tariffZones>   <!-- zonas geográficas de tarifa -->
  <fareProducts> ... </fareProducts> <!-- títulos de transporte (passes, bilhetes) -->
  <salesOfferPackages> ... </salesOfferPackages> <!-- como adquirir os títulos -->
</FareFrame>
```

---

## 3. TariffZone — Zonas Tarifárias

Uma **`TariffZone`** define uma região geográfica à qual pertence um conjunto de paragens, para efeitos de cálculo de tarifa. [[STD-03]](#STD-03)

O sistema Andante de Porto usa zonas concêntricas Z2, Z3, Z4... que representam o número mínimo de zonas para uma viagem. Cada paragem pertence a uma ou mais zonas.

```xml
<TariffZone id="PT:STCP:TariffZone:Z2:LOC" version="1">
  <Name>Andante Z2</Name>
</TariffZone>

<TariffZone id="PT:STCP:TariffZone:Z3:LOC" version="1">
  <Name>Andante Z3</Name>
</TariffZone>
```

### 3.1 Ligar Paragens a Zonas

As `ScheduledStopPoint` são ligadas às `TariffZone` diretamente no `ServiceFrame`:

```xml
<ScheduledStopPoint id="PT:STCP:ScheduledStopPoint:BLRB1:LOC" version="1">
  <Name>Bolhão</Name>
  <!-- Bolhão está na zona Z2 do Andante -->
  <tariffZones>
    <TariffZoneRef ref="PT:STCP:TariffZone:Z2:LOC" version="1"/>
  </tariffZones>
</ScheduledStopPoint>
```

### 3.2 Zonas Andante do Porto

A Linha 200 (Bolhão → Castelo do Queijo) atravessa várias zonas:

| Zona | Paragens (exemplos) | Preço de base Andante |
|------|--------------------|-----------------------|
| Z2 | Bolhão, Carmo, Trindade (centro do Porto) | 1,20 € |
| Z3 | Ramalde, Mercado da Foz (periferia imediata) | 1,60 € |
| Z4 | Castelo do Queijo (extremidade da linha) | 1,85 € |

*Nota: preços indicativos — verificar valores atuais em andante.net*

> **Nota pedagógica:** Os exemplos XML deste módulo omitem o elemento `FarePrice/Amount` por simplificação. Numa implementação de produção, cada `PreassignedFareProduct` teria um `FarePrice` com `Amount` e `Currency` (ex: `1.20 EUR` para Z2). A modelação completa de preços está na Parte 3 do NeTEx.

---

## 4. FareProduct e PreassignedFareProduct

Um **`FareProduct`** é o tipo de título de transporte. O subtipo mais comum é o **`PreassignedFareProduct`** — um título com valor fixo e condições pré-definidas (ao contrário de tarifas calculadas dinamicamente). [[STD-03]](#STD-03)

### 4.1 Tipos de FareProduct relevantes para Portugal

| Tipo | Descrição | Exemplos PT |
|------|-----------|-------------|
| `PreassignedFareProduct` | Título com preço e validade fixos | Andante 1 viagem, Andante 24h |
| `SaleDiscountRight` | Desconto sobre um produto base | Andante Jovem, Passe Social |
| `SupplementProduct` | Suplemento (ex: bagagem, bicicleta) | Bicicleta no comboio |

### 4.2 PreassignedFareProduct — Exemplo Andante Z2

```xml
<PreassignedFareProduct id="PT:Andante:FareProduct:Z2_Simples:LOC" version="1">
  <Name>Andante — 1 Viagem Z2</Name>

  <!-- ConditionSummary descreve as condições de uso do título -->
  <conditionSummary>
    <TravelDocumentType>electronicDocument</TravelDocumentType>
    <RequiresCard>true</RequiresCard>     <!-- cartão Andante obrigatório -->
    <RequiresReservation>false</RequiresReservation>
  </conditionSummary>
</PreassignedFareProduct>
```

### 4.3 Passe Mensal vs Bilhete Simples

Em NeTEx, a distinção entre um bilhete simples e um passe mensal é feita através de `UsageParameters`:

```
Bilhete simples (Andante):
  UsageValidityPeriod: StandardDuration=PT75M  (75 minutos após validação)
  MaximumNumberOfAccesses: 1 viagem (por validação)

Passe mensal:
  UsageValidityPeriod: StandardDuration=P1M  (1 mês após ativação)
  MaximumNumberOfAccesses: ilimitado no período
```

*Nota: a modelação completa de passes mensais e validações multi-viagem requer NeTEx Parte 3 (Fares), que está fora do âmbito introdutório deste módulo.*

---

## 5. SalesOfferPackage — Como o Título é Vendido

O **`SalesOfferPackage`** descreve como o passageiro pode adquirir um produto tarifário: em máquina automática, online, ou em bilheteira. [[STD-03]](#STD-03)

```xml
<SalesOfferPackage id="PT:Andante:SalesOfferPackage:Z2_Maquina:LOC" version="1">
  <Name>Andante Z2 — Máquina automática</Name>

  <salesOfferPackageElements>
    <SalesOfferPackageElement id="PT:Andante:SOPE:Z2_Maquina_01:LOC" version="1" order="1">
      <FareProductRef ref="PT:Andante:FareProduct:Z2_Simples:LOC" version="1"/>
    </SalesOfferPackageElement>
  </salesOfferPackageElements>
</SalesOfferPackage>
```

---

## 6. Comparação GTFS ↔ NeTEx para Tarifas

| GTFS Fares (legado) | NeTEx | Notas |
|--------------------|-------|-------|
| `fare_attributes.txt: fare_id` | `FareProduct/id` | O "título" de transporte |
| `fare_attributes.txt: price` | `FarePrice/Amount` | O preço |
| `fare_attributes.txt: currency_type` | `FarePrice/Currency` | A moeda |
| `fare_rules.txt: zone_id` | `TariffZone/id` | A zona tarifária |
| `stops.txt: zone_id` | `ScheduledStopPoint/tariffZones` | A zona a que a paragem pertence |
| *(não existe)* | `PreassignedFareProduct` | Produto tarifário com condições |
| *(não existe)* | `SalesOfferPackage` | Canal de venda |
| *(não existe)* | `SaleDiscountRight` | Desconto (jovem, sénior, etc.) |
| *(não existe)* | `UsageValidityPeriod` | Janela temporal de validade |

---

## 7. Âmbito e Limitações deste Módulo

O NeTEx Parte 3 (Fares) é o subsistema mais extenso e complexo do NeTEx. Este módulo cobre apenas os conceitos base. Para modelação tarifária completa, consultar: [[STD-03]](#STD-03)

| Conceito | Neste módulo | Módulo futuro |
|----------|-------------|---------------|
| `TariffZone` | ✅ Coberto | — |
| `PreassignedFareProduct` (simples) | ✅ Coberto | — |
| `SalesOfferPackage` (básico) | ✅ Coberto | — |
| Passes mensais / anuais | ⚠️ Introduzido | NeTEx Parte 3 |
| Descontos e concessions | ⚠️ Mencionado | NeTEx Parte 3 |
| Interoperabilidade Andante / Navegante | ❌ Fora do âmbito | NeTEx Parte 3 |
| Validação e controlo de títulos | ❌ Fora do âmbito | NeTEx Parte 3 |

---

## Exemplos

- [`exemplos/06_01_andante_porto.xml`](exemplos/06_01_andante_porto.xml) — FareFrame com TariffZone Andante (Z2/Z3/Z4), PreassignedFareProduct bilhete simples Z2, SalesOfferPackage, e ligação das paragens da Linha 200 às suas zonas tarifárias

---

## Exercícios

- [Exercício 6 — Modelar o Navegante Municipal Lisboa](exercicios/exercicio_06.md)

---

## Referências

| ID | Referência |
|----|-----------|
| <a id="STD-01"></a>[STD-01] | CEN. "NeTEx – Network Timetable Exchange." CEN/TS 16614-1:2024 (Parte 1). https://github.com/NeTEx-CEN/NeTEx |
| <a id="STD-03"></a>[STD-03] | CEN. "NeTEx – Network Timetable Exchange." CEN/TS 16614-3:2024 (Parte 3 — Fares). https://github.com/NeTEx-CEN/NeTEx |
| <a id="REG-01"></a>[REG-01] | Regulamento Delegado (UE) 2024/490 da Comissão, de 14 de Dezembro de 2023. JO L, 2024/490. https://eur-lex.europa.eu/legal-content/PT/TXT/?uri=OJ:L_202400490 |
| <a id="PT-01"></a>[PT-01] | IMT. "Perfil Nacional NeTEx Portugal." https://ptprofiles.azurewebsites.net |
| <a id="DATA-03"></a>[DATA-03] | STCP. "Feed GTFS STCP Porto (versão Escolar 228, 2026-04-30)." Feed oficial STCP. |
