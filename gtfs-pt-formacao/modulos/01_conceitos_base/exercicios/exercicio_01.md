# Exercício 1 — Explorar um Feed GTFS

## Contexto

Neste exercício, vai explorar o feed GTFS real da Operadora Exemplo (Linha 200) disponível em `exemplos/rede_exemplo_portugal/gtfs/`.

---

## Parte 1 — Identificar a Estrutura

Abra cada ficheiro `.txt` do feed e responda:

1. Quantos ficheiros existem no feed? Quais são obrigatórios e quais são recomendados/opcionais?
2. Qual é o `agency_id` da Operadora Exemplo?
3. Quantas rotas existem no feed?
4. Qual é o `route_type` da Linha 200? O que significa?

---

## Parte 2 — Seguir as Relações

Usando os ficheiros do feed, trace o caminho relacional para a primeira viagem da Linha 200:

1. Em `routes.txt`, encontre o `route_id` da Linha 200
2. Em `trips.txt`, encontre uma viagem (`trip_id`) que pertença a essa rota
3. Em `stop_times.txt`, encontre os horários de passagem dessa viagem
4. Em `stops.txt`, identifique a primeira e a última paragem dessa viagem

---

## Parte 3 — Identificar o Calendário

1. O feed usa `calendar.txt`, `calendar_dates.txt`, ou ambos?
2. Quantos `service_id` distintos existem?
3. Que tipos de dia de serviço representam (dias úteis, sábados, domingos)?

---

## Questões de Reflexão

1. Porque é que a Operadora Exemplo usa apenas `calendar_dates.txt` em vez de `calendar.txt`? Que vantagens e desvantagens tem esta abordagem?
2. Se quisesse adicionar uma segunda rota (ex: Linha 201) ao feed, que ficheiros precisaria de modificar?
3. Qual é o ficheiro mais pesado do feed? Porque é expectável que assim seja?

---

## Checklist de Auto-avaliação

- [ ] Identifiquei todos os ficheiros obrigatórios e recomendados no feed
- [ ] Tracei as relações entre `routes` → `trips` → `stop_times` → `stops`
- [ ] Compreendi o papel do `service_id` como elo de ligação ao calendário
- [ ] Refleti sobre as vantagens/desvantagens de `calendar.txt` vs `calendar_dates.txt`
