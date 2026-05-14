# Exercício 1 — Identificadores e Referências NeTEx

> **Módulo 1 — Conceitos Base**

---

## Contexto

Neste exercício vai praticar a construção de identificadores NeTEx e a criação de referências entre objetos, usando dados reais da Operadora Exemplo.

---

## Parte A — Construir Identificadores (10 min)

O formato de um identificador NeTEx em Portugal é:

```
PT:<operador>:<TipoDeObjeto>:<idLocal>:LOC
```

**Tarefa:** Converta os seguintes dados GTFS da Operadora Exemplo em identificadores NeTEx correctos.

| Dado GTFS | Tipo NeTEx | Identificador NeTEx correto |
|-----------|-----------|----------------------------|
| `agency_id = Operadora Exemplo` | `Operator` | ? |
| `route_id = 201` (linha Aliados-Viso) | `Line` | ? |
| `stop_id = 0AAL2` (Av. Aliados) | `ScheduledStopPoint` | ? |
| `route_id = 300` (circular Hospital S. João) | `Line` | ? |
| Autoridade reguladora: IMT | `Authority` | ? |

> **Dica:** O `<operador>` no ID do `Authority` é o código da própria autoridade, não da Operadora Exemplo.

---

## Parte B — Identificar Problemas (10 min)

Os seguintes fragmentos XML têm erros. Identifique e corrija cada um:

**Fragmento 1:**
```xml
<Line id="200" version="1">
  <Name>Bolhão - Castelo do Queijo</Name>
</Line>
```

**Fragmento 2:**
```xml
<Route id="PT:EXEMPLO:Route:200_IDA:LOC" version="1">
  <lineRef ref="200"/>  <!-- referência à linha -->
</Route>
```

**Fragmento 3:** O fragmento seguinte tem um atributo obrigatório em falta. Identifique-o e acrescente-o.
```xml
<Operator id="PT:EXEMPLO:Operator:Operadora Exemplo:LOC">
  <Name>Operadora Exemplo</Name>
</Operator>
```

---

## Parte C — Criar um ResourceFrame (20 min)

O ficheiro GTFS da Operadora Exemplo tem os seguintes dados em `agency.txt`:

```
agency_id,agency_name,agency_url,agency_timezone,agency_phone,agency_lang
Operadora Exemplo,"Sociedade de Transportes Colectivos do Porto, E.I.M., S.A",http://www.exemplo.pt,Europe/Lisbon,225071000,pt
```

**Tarefa:** Crie um `ResourceFrame` NeTEx completo com:
1. Um `Authority` para o IMT (entidade reguladora)
2. Um `Operator` para a Operadora Exemplo com todos os campos equivalentes ao GTFS
3. Identificadores corretos para ambos
4. `validBetween` com `FromDate` de 2026-04-30

---

## Solução

A solução comentada encontra-se em [`solucoes/exercicio_01_solucao.xml`](solucoes/exercicio_01_solucao.xml).

Antes de consultar a solução, tente resolver cada parte de forma independente.

---

## Autoavaliação

Após consultar a solução, verifique:

- [ ] Os identificadores seguem o formato `PT:<org>:<Tipo>:<local>:LOC`?
- [ ] O `Operator` tem o atributo `version`?
- [ ] A referência `lineRef` usa o ID completo do objecto referenciado?
- [ ] O `ResourceFrame` tem `id` e `version`?
- [ ] O `validBetween` usa elementos `<FromDate>` e `<ToDate>`, não atributos XML?
- [ ] O `validBetween` tem o formato de data correto (`YYYY-MM-DD`)?
