# Exercício 7 — Modelar Tarifas e Analisar Dados de Utilização

## Contexto

A **TransPorto** é um operador fictício que serve a Área Metropolitana do Porto com uma rede de autocarros e um serviço de transporte a pedido para zonas rurais. O sistema tarifário é baseado em zonas, semelhante ao Andante:

- **3 zonas**: Centro (C), Norte (N), Sul (S)
- **Títulos unitários**: preço varia conforme o número de zonas
  - 1 zona: 1,40 EUR
  - 2 zonas: 1,85 EUR
  - 3 zonas: 2,30 EUR
- **Transbordos**: até 2, dentro de 1 hora
- **Validação**: a bordo do veículo (autocarros)

---

## Parte 1 — Modelar Tarifas com Fares v1

Crie os seguintes ficheiros:

### 1.1 `fare_attributes.txt`

Defina os três títulos unitários da TransPorto. Para cada um, inclua:
- `fare_id` descritivo
- `price` correto
- `currency_type` = EUR
- `payment_method` adequado
- `transfers` e `transfer_duration`

### 1.2 `fare_rules.txt`

Associe cada tarifa às combinações de zonas de origem/destino corretas:
- 1 zona: viagens dentro da mesma zona (C→C, N→N, S→S)
- 2 zonas: viagens entre zonas adjacentes (C→N, N→C, C→S, S→C)
- 3 zonas: viagens entre zonas extremas (N→S, S→N)

---

## Parte 2 — Limitações do v1 e Soluções do v2

Responda às seguintes questões:

1. A TransPorto quer oferecer um **desconto de 50% para crianças** (4-12 anos) em todos os títulos. Como faria isso no v1? Quais as limitações?

2. A TransPorto quer introduzir um **passe mensal de 30 EUR** válido para todas as zonas. O v1 consegue representar este produto? Porquê?

3. A TransPorto quer que os passageiros com a **app móvel** paguem menos 10% que os que usam cartão físico. Descreva como o v2 (`fare_media`, `fare_products`) resolveria este cenário. Não precisa de escrever CSV — descreva a estrutura conceptual.

4. A TransPorto quer implementar um **limite diário de 5 EUR** (*fare capping*) — o passageiro nunca paga mais de 5 EUR por dia. Este conceito existe no v2? Que ficheiro seria usado?

---

## Parte 3 — Criar um Registo `board_alight`

A Linha 100 da TransPorto (Centro → Norte) tem 4 paragens:

| Sequência | Paragem | stop_id |
|-----------|---------|---------|
| 1 | Praça Central | PRC01 |
| 2 | Avenida da República | AVR02 |
| 3 | Estação Norte | ETN03 |
| 4 | Parque Industrial | PIR04 |

Na viagem `100_DU_0830` (dias úteis, 08:30), os dados de contagem registaram:

- **Praça Central**: 15 embarques, 0 desembarques
- **Avenida da República**: 8 embarques, 3 desembarques
- **Estação Norte**: 2 embarques, 12 desembarques
- **Parque Industrial**: 0 embarques, 10 desembarques

Crie o ficheiro `board_alight.txt` com estes dados. Use `record_use=0` e `schedule_relationship=0`. Calcule o valor correto de `current_load` para cada paragem.

---

## Questões de Reflexão

1. O sistema Andante do Porto abrange múltiplos operadores (Operadora Exemplo, Metro do Porto, CP). Porque é que modelar tarifas intermodais é particularmente difícil no Fares v1?

2. Se Portugal quisesse publicar dados tarifários no NAP (Ponto de Acesso Nacional) seguindo o regulamento MMTIS, seria preferível GTFS Fares v2 ou NeTEx Part 3? Que fatores influenciam esta decisão?

3. Os dados de `board_alight.txt` podem ser usados para otimizar a oferta tarifária? Dê um exemplo concreto de como dados de ridership poderiam influenciar a política de preços.

---

## Checklist de Auto-avaliação

- [ ] Criei `fare_attributes.txt` com 3 tarifas e todos os campos corretos
- [ ] Criei `fare_rules.txt` com as associações zona-a-zona corretas (incluindo ambos os sentidos)
- [ ] Identifiquei pelo menos 3 limitações do Fares v1 para cenários reais
- [ ] Descrevi como o Fares v2 resolve os cenários de desconto por categoria e suporte
- [ ] Criei `board_alight.txt` com `current_load` calculado corretamente em cada paragem
- [ ] Refleti sobre a complexidade tarifária no contexto intermodal português
