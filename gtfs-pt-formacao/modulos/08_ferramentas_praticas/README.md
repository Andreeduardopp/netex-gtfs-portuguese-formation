# Módulo 8 — Ferramentas Práticas

## Objetivos de Aprendizagem

Após completar este módulo, o leitor será capaz de:

- Utilizar o MobilityData Canonical GTFS Validator para validar feeds
- Interpretar relatórios de validação e corrigir erros comuns
- Usar Python (com `partridge` ou `pandas`) para explorar e manipular feeds GTFS
- Conhecer as principais ferramentas de visualização e edição de feeds
- Construir um pipeline básico de ETL para produção de feeds GTFS

---

## 1. Validação de Feeds

### 1.1 MobilityData Canonical GTFS Validator

O Canonical GTFS Validator é a ferramenta oficial de referência para validação de feeds GTFS. É desenvolvido pela MobilityData e usado pelo Google Transit como critério de aceitação. [[TOOL-01]](#TOOL-01)

**Instalação e execução:**

```bash
# Descarregar o .jar mais recente
java -jar gtfs-validator-cli.jar -i feed.zip -o relatorio/
```

O validador produz um relatório HTML com três níveis de severidade:

| Nível | Significado | Ação |
|-------|-------------|------|
| **ERROR** | Violação da especificação — o feed é inválido | Corrigir antes de publicar |
| **WARNING** | Desvio das Best Practices — funcional mas subótimo | Corrigir se possível |
| **INFO** | Sugestão de melhoria | Avaliar caso a caso |

### 1.2 Validação Online

Para feeds pequenos ou testes rápidos, existe uma versão web do validador:

- **GTFS Validator Web**: permite carregar um ZIP e obter o relatório diretamente no browser
- **Mobility Database**: ao submeter um feed, a validação é executada automaticamente [[DATA-02]](#DATA-02)

### 1.3 Erros Mais Frequentes em Feeds Portugueses

Com base na experiência de validação de feeds de operadores portugueses: [[DATA-01]](#DATA-01)

| Erro | Severidade | Causa Típica |
|------|-----------|--------------|
| `foreign_key_violation` | ERROR | `stop_id` em `stop_times.txt` que não existe em `stops.txt` |
| `decreasing_stop_time_distance` | WARNING | `shape_dist_traveled` não crescente |
| `fast_travel_between_stops` | WARNING | Velocidade implausível entre paragens consecutivas |
| `unused_shape` | WARNING | `shape_id` definido mas não referenciado por nenhum trip |
| `missing_feed_info_date` | WARNING | `feed_info.txt` sem `feed_start_date` ou `feed_end_date` |
| `stop_too_far_from_trip_shape` | WARNING | Paragem a mais de 100m da shape |
| `duplicate_route_name` | WARNING | Duas rotas com o mesmo `route_long_name` |

### 1.4 Exemplo de Relatório

Ao validar o feed Operadora Exemplo (versão Escolar 228), o validador tipicamente reporta:

```
Validation Report
=================
Validated feed: exemplo_gtfs.zip
GTFS files: 7 files
Errors: 0
Warnings: 3
  - missing_feed_info_date (1 occurrence)
  - fast_travel_between_stops (2 occurrences)
```

---

## 2. Exploração com Python

### 2.1 `partridge` — Leitura Rápida de Feeds

O `partridge` é uma biblioteca Python que carrega feeds GTFS em DataFrames do pandas, filtrando automaticamente por data de serviço: [[TOOL-02]](#TOOL-02)

```python
import partridge as ptg

# Carregar feed filtrando por data
feed = ptg.load_geo_feed("exemplo_gtfs.zip", view={"trips.txt": {"service_id": "DIAS UTEIS"}})

# Explorar rotas
print(feed.routes[["route_id", "route_short_name", "route_long_name"]])

# Explorar paragens de uma viagem
trip_stops = feed.stop_times[feed.stop_times.trip_id == "200_DU_0600"]
print(trip_stops[["stop_id", "arrival_time", "departure_time", "stop_sequence"]])
```

### 2.2 `pandas` — Manipulação Direta

Para quem prefere controlo total, os ficheiros GTFS são CSV simples que o `pandas` lê nativamente:

```python
import pandas as pd
import zipfile

with zipfile.ZipFile("exemplo_gtfs.zip") as z:
    stops = pd.read_csv(z.open("stops.txt"), dtype=str)
    routes = pd.read_csv(z.open("routes.txt"), dtype=str)
    trips = pd.read_csv(z.open("trips.txt"), dtype=str)
    stop_times = pd.read_csv(z.open("stop_times.txt"), dtype=str)

# Quantas viagens por rota?
trips_per_route = trips.groupby("route_id").size().sort_values(ascending=False)
print(trips_per_route.head(10))

# Quantas paragens únicas?
print(f"Total de paragens: {len(stops)}")

# Paragens da Linha 200
trips_200 = trips[trips.route_id == "200"]
st_200 = stop_times[stop_times.trip_id.isin(trips_200.trip_id)]
stops_200 = stops[stops.stop_id.isin(st_200.stop_id)]
print(f"Paragens da Linha 200: {len(stops_200)}")
```

### 2.3 Análises Úteis

Exemplos de análises que se podem fazer com Python sobre um feed GTFS:

| Análise | Descrição |
|---------|-----------|
| Cobertura temporal | Quantas viagens por hora do dia? |
| Cobertura espacial | Mapa de paragens (com `folium` ou `matplotlib`) |
| Velocidade comercial | Tempo entre primeira e última paragem / distância da shape |
| Frequência efetiva | Headway médio por rota e faixa horária |
| Consistência | Comparar dois feeds para detetar alterações |

```python
import matplotlib.pyplot as plt

# Histograma de partidas por hora
stop_times["hour"] = stop_times.departure_time.str[:2].astype(int)
stop_times.groupby("hour").size().plot(kind="bar", title="Partidas por Hora")
plt.xlabel("Hora")
plt.ylabel("Número de partidas")
plt.show()
```

---

## 3. Ferramentas de Visualização

### 3.1 Ferramentas Web

| Ferramenta | Descrição | Utilidade |
|-----------|-----------|-----------|
| **GTFS-to-HTML** | Gera horários HTML a partir de um feed | Publicação de horários no website do operador |
| **Transit Map** | Gera mapas de rede a partir de shapes | Visualização da cobertura geográfica |
| **Transitland** | Plataforma de dados de trânsito abertos | Exploração e comparação de feeds |

### 3.2 Ferramentas Desktop

| Ferramenta | Plataforma | Descrição |
|-----------|-----------|-----------|
| **QGIS** | Windows/Mac/Linux | SIG open-source — importar stops.txt e shapes.txt como camadas |
| **GTFS Editor** | Web | Editor visual de feeds GTFS |

### 3.3 Visualização em QGIS

Para visualizar um feed GTFS no QGIS:

1. Extrair o ZIP do feed
2. Importar `stops.txt` como camada CSV (longitude/latitude)
3. Importar `shapes.txt` como camada CSV e converter para linhas
4. Estilizar por `route_id` ou `route_type`

---

## 4. Pipeline ETL para Produção de Feeds

### 4.1 Arquitetura Típica

Um pipeline de produção de feeds GTFS segue tipicamente estas etapas:

```
┌─────────────┐    ┌──────────┐    ┌───────────┐    ┌──────────┐    ┌───────────┐
│  Fontes de  │───>│  Extração │───>│Transformação│──>│ Validação │──>│Publicação │
│   Dados     │    │  (E)     │    │   (T/L)    │    │          │    │           │
└─────────────┘    └──────────┘    └───────────┘    └──────────┘    └───────────┘
     AVL              CSV/API         Mapeamento       Validator      NAP / URL
     SAE              JSON/XML        Limpeza          Relatório      feed_info
     SIG              BD              Geração ZIP      Aprovação      Mobility DB
```

### 4.2 Etapas do Pipeline

| Etapa | Entrada | Saída | Ferramentas |
|-------|---------|-------|-------------|
| **Extração** | BD operacional, AVL, SIG | Dados brutos (CSV/JSON) | SQL, APIs, scripts Python |
| **Transformação** | Dados brutos | Ficheiros GTFS normalizados | `pandas`, `partridge`, scripts custom |
| **Validação** | Feed ZIP | Relatório de erros | MobilityData Validator [[TOOL-01]](#TOOL-01) |
| **Publicação** | Feed validado | URL pública | NAP Portugal [[PT-01]](#PT-01), servidor HTTP |

### 4.3 Exemplo de Script de Produção

```python
import subprocess
import shutil
import zipfile
from pathlib import Path

def build_feed(output_dir: Path, zip_path: Path):
    """Empacota ficheiros GTFS num ZIP e valida."""
    gtfs_files = [
        "agency.txt", "stops.txt", "routes.txt",
        "trips.txt", "stop_times.txt", "calendar_dates.txt",
        "feed_info.txt"
    ]

    with zipfile.ZipFile(zip_path, "w", zipfile.ZIP_DEFLATED) as zf:
        for fname in gtfs_files:
            fpath = output_dir / fname
            if fpath.exists():
                zf.write(fpath, fname)

    result = subprocess.run(
        ["java", "-jar", "gtfs-validator-cli.jar", "-i", str(zip_path), "-o", "report/"],
        capture_output=True, text=True
    )
    print(result.stdout)
    if "ERROR" in result.stdout:
        raise RuntimeError("Feed contém erros — corrigir antes de publicar")
```

### 4.4 Automatização

Para operadores com feeds que mudam regularmente:

- **Cron job / GitHub Actions**: executar o pipeline diariamente ou semanalmente
- **Monitorização**: alertas quando a validação falha
- **Versionamento**: guardar cada versão do feed com data no nome (`exemplo_gtfs_20260503.zip`)
- **Diff**: comparar feed anterior com novo para detetar alterações inesperadas

---

## 5. `feed_info.txt` — Metadados do Feed

O `feed_info.txt` contém metadados sobre o feed como um todo: [[STD-01]](#STD-01)

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `feed_publisher_name` | Texto | Sim | Nome do publicador |
| `feed_publisher_url` | URL | Sim | Website do publicador |
| `feed_lang` | Código | Sim | Idioma principal (`pt`) |
| `default_lang` | Código | Não | Idioma padrão quando `feed_lang` não é especificado |
| `feed_start_date` | Data | Não | Início da validade (`YYYYMMDD`) |
| `feed_end_date` | Data | Não | Fim da validade (`YYYYMMDD`) |
| `feed_version` | Texto | Não | Versão do feed |
| `feed_contact_email` | Email | Não | Contacto técnico |
| `feed_contact_url` | URL | Não | URL de contacto |

```csv
feed_publisher_name,feed_publisher_url,feed_lang,feed_start_date,feed_end_date,feed_version
Operadora Exemplo,https://www.exemplo.pt,pt,20260430,20261231,Escolar_228
```

As Best Practices recomendam que `feed_start_date` e `feed_end_date` estejam sempre preenchidos. [[BP-01]](#BP-01)

---

## 6. Ponte NeTEx (Opcional)

| GTFS | NeTEx | Notas |
|------|-------|-------|
| `feed_info.txt` | `PublicationDelivery` | NeTEx inclui metadados de publicação no envelope XML |
| Validator (MobilityData) | Validator (greenlight / Entur) | Cada standard tem o seu validador oficial |
| Pipeline ETL | SIRI → NeTEx / NeTEx → GTFS | Conversão bidirecional possível |
| `partridge` (Python) | `netex-python` / `chouette` | Ferramentas de parsing específicas |

---

## Exemplos

- [`exemplos/08_01_erros_frequentes.md`](exemplos/08_01_erros_frequentes.md) — Catálogo de erros frequentes em feeds portugueses com causas e soluções

---

## Referências

| ID | Referência |
|----|-----------|
| <a id="STD-01"></a>[STD-01] | MobilityData. "GTFS Schedule Reference." https://gtfs.org/schedule/reference |
| <a id="BP-01"></a>[BP-01] | MobilityData. "GTFS Schedule Best Practices." https://gtfs.org/documentation/schedule/schedule-best-practices |
| <a id="TOOL-01"></a>[TOOL-01] | MobilityData. "Canonical GTFS Validator." https://github.com/MobilityData/gtfs-validator |
| <a id="TOOL-02"></a>[TOOL-02] | Remix. "partridge — Python GTFS library." https://github.com/remix/partridge |
| <a id="DATA-01"></a>[DATA-01] | Operadora Exemplo. "Feed GTFS Operadora Exemplo (versão Escolar 228, 2026-04-30)." Feed oficial Operadora Exemplo. |
| <a id="DATA-02"></a>[DATA-02] | MobilityData. "Mobility Database." https://mobilitydatabase.org |
| <a id="PT-01"></a>[PT-01] | IMT. "NAP Portugal." https://nap-portugal.imt-ip.pt |
