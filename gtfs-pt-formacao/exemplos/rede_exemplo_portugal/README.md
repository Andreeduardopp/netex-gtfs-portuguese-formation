# Rede Exemplo: Operadora Exemplo — Linha 200

## Decisão Âncora

Todos os exemplos deste curso utilizam dados reais da **Operadora Exemplo (Sociedade de Transportes Colectivos do Porto)**, especificamente a **Linha 200 (Bolhão → Castelo do Queijo)** como caso de estudo principal. [[DATA-01]](#DATA-01)

### Porquê dados reais?

- Exemplos mais credíveis e verificáveis pelo leitor
- Demonstra complexidades que dados fictícios não capturariam (zonas tarifárias, variações de calendário, múltiplas direções)
- Alinhamento com o curso NeTEx PT Formação, que usa a mesma rede âncora

### Porquê a Operadora Exemplo e a Linha 200?

- Operador de grande dimensão com dados GTFS publicados
- Linha que atravessa zonas tarifárias diferentes (Andante Z2, Z3, Z4)
- Percurso com 30 paragens por direção — suficiente para demonstrar complexidade sem ser excessivo
- Bolhão é um nó multimodal (autocarro + metro) — permite demonstrar transferências

## Ficheiros GTFS Incluídos

| Ficheiro | Conteúdo | Registos |
|----------|----------|----------|
| `agency.txt` | Operador Operadora Exemplo | 1 |
| `routes.txt` | Linha 200 | 1 |
| `stops.txt` | Paragens da Linha 200 | ~59 |
| `trips.txt` | Viagens da Linha 200 | ~340 |
| `stop_times.txt` | Horários de passagem | ~10.000 |
| `calendar_dates.txt` | Datas de serviço (exception_type=1) | ~900 |
| `feed_info.txt` | Metadados do feed | 1 |

## Nota sobre ordem de colunas

A spec GTFS permite que as colunas apareçam em qualquer ordem dentro de cada ficheiro. [[STD-01]](#STD-01) A ordem usada nestes ficheiros reais pode diferir da ordem pedagógica apresentada nos módulos (ex: `stops.txt` começa com `stop_lat` em vez de `stop_id`). Ambas são igualmente válidas.

## Fonte

Dados extraídos do feed GTFS oficial da Operadora Exemplo (versão Escolar 228, 2026-04-30). [[DATA-01]](#DATA-01)

---

## Referências

| ID | Referência |
|----|-----------|
| <a id="DATA-01"></a>[DATA-01] | Operadora Exemplo. "Feed GTFS Operadora Exemplo (versão Escolar 228, 2026-04-30)." Feed oficial Operadora Exemplo disponibilizado pela operadora. |
