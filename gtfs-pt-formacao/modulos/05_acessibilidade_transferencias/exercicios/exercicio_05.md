# Exercício 5 — Modelar Acessibilidade e Transferências

## Contexto

A **Estação de Campanhã** é o principal interface intermodal do Porto, ligando serviços ferroviários (CP — Comboios de Portugal), Metro do Porto (Linha F — Laranja) e autocarros urbanos da Operadora Exemplo. A estação foi recentemente renovada com um novo terminal intermodal.

Assuma os seguintes dados simplificados:

| Serviço | Paragem/Plataforma | Acessível? |
|---------|-------------------|------------|
| CP — Linha do Norte (Plataforma 1) | Piso 0 (térreo) | Sim (via rampa) |
| CP — Linha do Norte (Plataforma 2) | Piso 0 (térreo) | Sim (via rampa) |
| CP — Linha do Douro (Plataforma 5) | Piso 0 (térreo) | Não (sem rampa, degrau alto) |
| Metro Linha F | Piso -1 (subterrâneo) | Sim (elevador) |
| Operadora Exemplo Linha 205 | Piso 0 (terminal rodoviário) | Sim |
| Operadora Exemplo Linha 400 | Piso 0 (terminal rodoviário) | Sim |

---

## Parte 1 — Acessibilidade em `stops.txt`

Crie um ficheiro `stops.txt` para a Estação de Campanhã que inclua:

1. A estação-pai (`location_type=1`) com `wheelchair_boarding` adequado
2. Todas as plataformas listadas acima como paragens-filhas (`location_type=0`)
3. Pelo menos uma entrada/saída (`location_type=2`)
4. O campo `wheelchair_boarding` correto para cada elemento

**Dica**: lembre-se do mecanismo de herança — se a estação-pai tem `wheelchair_boarding=1`, as paragens-filhas com valor `0` herdam esse valor. Use `wheelchair_boarding=2` explicitamente apenas onde a acessibilidade **não** existe.

Coordenadas de referência para Campanhã: latitude `41.148652`, longitude `-8.585363`.

---

## Parte 2 — Transferências com `transfers.txt`

Crie um ficheiro `transfers.txt` que modele as seguintes transferências na Estação de Campanhã:

1. **CP → Metro** (descida de piso): `transfer_type=2`, tempo mínimo de 5 minutos (300 s)
2. **Metro → CP** (subida de piso): `transfer_type=2`, tempo mínimo de 6 minutos (360 s)
3. **CP → Operadora Exemplo** (mesmo piso, terminal rodoviário adjacente): `transfer_type=2`, tempo mínimo de 3 minutos (180 s)
4. **Metro → Operadora Exemplo** (subida de piso + caminhada): `transfer_type=2`, tempo mínimo de 7 minutos (420 s)
5. **Entre linhas Operadora Exemplo** (mesma zona do terminal): `transfer_type=0` (recomendado, sem tempo mínimo)

Modele as transferências nos **dois sentidos** (ida e volta), considerando que o tempo pode ser assimétrico.

---

## Parte 3 — Caminhos Pedonais com `pathways.txt`

Modele os caminhos pedonais dentro da Estação de Campanhã. Deve incluir:

1. **Nós necessários** em `stops.txt`:
   - Entrada principal (`location_type=2`)
   - Nós genéricos para átrio, topo de escadas, etc. (`location_type=3`)
   - Plataformas (`location_type=0`)

2. **Caminhos** em `pathways.txt`:
   - Corredor da entrada ao átrio (`pathway_mode=1`)
   - Escadas do átrio para a plataforma do Metro (`pathway_mode=2`)
   - Elevador como alternativa acessível (`pathway_mode=5`)
   - Portões de validação tarifária do Metro (`pathway_mode=6` e `7`)
   - Corredor do átrio ao terminal rodoviário (`pathway_mode=1`)

3. **Níveis** em `levels.txt`:
   - Nível 0 (Térreo / Plataformas CP / Terminal rodoviário)
   - Nível -1 (Plataforma Metro)

Verifique que existe **pelo menos um percurso completo acessível** (sem escadas) entre a entrada e cada plataforma com `wheelchair_boarding=1`.

---

## Questões de Reflexão

1. Porque é que o valor `0` (sem informação) em `wheelchair_boarding` é **pior** do que o valor `2` (não acessível) para o utilizador final? Pense no impacto nas decisões de navegação das aplicações.

2. Na Estação de Campanhã, o tempo de transferência Metro → CP (subida) é maior do que CP → Metro (descida). Que outros fatores, para além do desnível, podem influenciar o tempo real de transferência?

3. Se o elevador da Estação de Campanhã estivesse avariado durante duas semanas, que alterações faria ao feed GTFS para refletir esta situação? Considere tanto a abordagem estática como a dinâmica (GTFS Realtime).

4. Um operador argumenta que é demasiado trabalhoso preencher `pathways.txt` para todas as estações. Que critérios usaria para priorizar quais as estações que devem ter `pathways.txt` modelado primeiro?

---

## Checklist de Auto-avaliação

- [ ] Criei uma hierarquia `parent_station` correta com `location_type` adequado para cada nó
- [ ] O campo `wheelchair_boarding` está preenchido para todas as paragens, sem usar `0` quando a informação é conhecida
- [ ] Compreendo o mecanismo de herança entre estação-pai e paragens-filhas
- [ ] Modelei transferências nos dois sentidos com tempos assimétricos quando apropriado
- [ ] Distingui os quatro tipos de `transfer_type` e usei-os corretamente
- [ ] O meu `pathways.txt` inclui pelo menos um percurso acessível (sem escadas) para cada plataforma acessível
- [ ] Defini `levels.txt` com índices coerentes (0=rua, negativo=subterrâneo)
- [ ] Refleti sobre a importância prática da acessibilidade nos dados de transporte
