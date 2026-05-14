# Como Contribuir

Obrigado pelo interesse em contribuir para o **GTFS PT Formação**!

## Reportar Erros

Se encontrou um erro num ficheiro de exemplo ou no texto de um módulo:

1. Abra uma **Issue** com o template `Bug / Erro`
2. Indique o módulo e ficheiro afetado
3. Descreva o erro e, se possível, sugira a correção

## Sugerir Novos Módulos

1. Abra uma **Issue** com o template `Sugestão de Módulo`
2. Descreva o tema, o público-alvo e a relevância prática

## Submeter Exemplos CSV/TXT

1. Faça fork do repositório
2. Crie os ficheiros CSV na pasta `exemplos/` do módulo relevante
3. Garanta que os ficheiros seguem o formato GTFS (cabeçalhos corretos, campos separados por vírgula, UTF-8)
4. Valide os dados com o [Canonical GTFS Validator](https://github.com/MobilityData/gtfs-validator) da MobilityData
5. Abra um Pull Request

## Citações e Referências (Obrigatório)

> **Regra fundamental**: Todo o conteúdo deste projeto deve citar as fontes originais. Nenhuma informação deve ser apresentada sem a devida atribuição.

### Regras de citação

1. **Toda afirmação técnica** (definições GTFS, especificações, regras do standard) deve incluir uma referência à fonte original (especificação GTFS, Best Practices, regulação EU, etc.)
2. **Texto adaptado ou traduzido** de outros projetos deve indicar explicitamente a origem e a natureza da adaptação
3. **Dados e exemplos** baseados em feeds reais de operadores devem creditar a fonte (ex: "Dados baseados no feed GTFS público da Operadora Exemplo, disponível em [URL]")
4. **Diagramas e figuras** reproduzidos ou adaptados de outras fontes devem incluir "Fonte:" ou "Adaptado de:" na legenda
5. **Citações diretas** (texto copiado ipsis verbis) devem usar aspas ou blocos de citação `>` e indicar autor/fonte/ano

### Formato de referências

Cada módulo deve incluir uma secção **Referências** no final do `README.md`, usando o formato:

```markdown
## Referências

[STD-01] MobilityData. "GTFS Schedule Reference." gtfs.org/schedule/reference
[BP-01] MobilityData. "GTFS Schedule Best Practices." gtfs.org/documentation/schedule/schedule-best-practices
[DATA-01] Operadora Exemplo. "Feed GTFS Operadora Exemplo." Feed oficial Operadora Exemplo.
```

### O que NÃO é aceitável

- Copiar texto de documentação oficial sem citar a fonte
- Apresentar exemplos baseados em dados reais sem creditar o operador
- Usar diagramas de terceiros sem atribuição

## Convenções

- Todos os ficheiros CSV/TXT devem seguir o formato da especificação GTFS
- Conteúdo dos módulos em português de Portugal (PT-PT)
- Nomes de ficheiro em snake_case com prefixo numérico do módulo
- UTF-8 sem BOM
- **Todas as fontes devem ser citadas** (ver secção Citações acima)
