# Exercício 4 — Modelar o Calendário de um Operador

## Contexto

A **Carris Metropolitana** (operador da Área Metropolitana de Lisboa) tem os seguintes padrões de serviço para a sua Linha 1523:

- **Dias úteis (período escolar)**: segunda a sexta, exceto feriados, de Setembro a Junho
- **Dias úteis (período de férias)**: segunda a sexta, exceto feriados, em Julho e Agosto (frequência reduzida)
- **Sábados**: todos os sábados do ano
- **Domingos e feriados**: todos os domingos + feriados nacionais

---

## Parte 1 — Abordagem com `calendar.txt`

Crie um ficheiro `calendar.txt` que defina os 4 tipos de serviço acima para o ano de 2026. Use `service_id` descritivos.

---

## Parte 2 — Exceções com `calendar_dates.txt`

Crie um ficheiro `calendar_dates.txt` que trate os seguintes feriados de 2026:

1. **1 de Maio (sexta-feira)**: remover serviço de dias úteis, adicionar serviço de domingos/feriados
2. **10 de Junho (quarta-feira)**: mesma lógica
3. **24 de Junho (quarta-feira, feriado municipal de Lisboa)**: mesma lógica

---

## Parte 3 — Abordagem Alternativa (só `calendar_dates.txt`)

Reescreva o calendário de Maio de 2026 usando **apenas** `calendar_dates.txt` com `exception_type=1` (sem `calendar.txt`). Liste todas as datas do mês com o `service_id` correspondente.

---

## Questões de Reflexão

1. Qual das duas abordagens (regras+exceções vs só exceções) é mais fácil de manter quando se adicionam novos feriados?
2. A Operadora Exemplo usa `DOMINGOS|FERIADOS` como `service_id`. Porque é que as Best Practices desaconselham caracteres especiais nos IDs?
3. Se a Carris Metropolitana precisasse de adicionar um quarto tipo de serviço (ex: vésperas de feriado com horário reduzido), que alterações seriam necessárias em cada abordagem?

---

## Checklist de Auto-avaliação

- [ ] Criei 4 `service_id` descritivos em `calendar.txt`
- [ ] Tratei corretamente os feriados com `exception_type=1` e `exception_type=2`
- [ ] Compreendi a diferença entre adição (1) e remoção (2) de serviço
- [ ] Modelei Maio de 2026 usando apenas `calendar_dates.txt`
- [ ] Refleti sobre as vantagens e desvantagens de cada abordagem
