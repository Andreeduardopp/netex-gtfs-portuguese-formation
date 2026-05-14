# Como Contribuir

Obrigado pelo interesse em contribuir para o **NeTEx PT Formação**!

## Reportar Erros

Se encontrou um erro num ficheiro XML de exemplo ou no texto de um módulo:

1. Abra uma **Issue** com o template `Bug / Erro`
2. Indique o módulo e ficheiro afetado
3. Descreva o erro e, se possível, sugira a correção

## Sugerir Novos Módulos

1. Abra uma **Issue** com o template `Sugestão de Módulo`
2. Descreva o tema, o público-alvo e a relevância prática

## Submeter Exemplos XML

1. Faça fork do repositório
2. Crie os ficheiros XML na pasta `exemplos/` do módulo relevante
3. Adicione comentários `<!-- ... -->` extensivos a explicar cada elemento
4. Valide o XML com o `netex-validator` da Entur
5. Abra um Pull Request

## Citações e Referências (Obrigatório)

> **Regra fundamental**: Todo o conteúdo deste projeto deve citar as fontes originais. Nenhuma informação deve ser apresentada sem a devida atribuição.

### Regras de citação

1. **Toda afirmação técnica** (definições NeTEx, especificações, regras do standard) deve incluir uma referência à fonte original (spec NeTEx, Transmodel, regulação EU, perfil PT, etc.)
2. **Texto adaptado ou traduzido** de outros projetos (ex: `etalab/netex-france-formation`) deve indicar explicitamente a origem e a natureza da adaptação
3. **Dados e exemplos** baseados em feeds reais de operadores devem creditar a fonte (ex: "Dados baseados no feed GTFS público da Operadora Exemplo, disponível em [URL]")
4. **Diagramas e figuras** reproduzidos ou adaptados de outras fontes devem incluir "Fonte:" ou "Adaptado de:" na legenda
5. **Citações diretas** (texto copiado ipsis verbis) devem usar aspas ou blocos de citação `>` e indicar autor/fonte/ano

### Formato de referências

Cada módulo deve incluir uma secção **Referências** no final do `README.md`, usando o formato:

```markdown
## Referências

[1] CEN. "NeTEx – Network Timetable Exchange." CEN/TS 16614, 2024.
[2] Regulamento Delegado (UE) 2024/490 da Comissão Europeia.
[3] IMT. "Perfil Nacional NeTEx Portugal." ptprofiles.azurewebsites.net, 2024.
```

### O que NÃO é aceitável

- ❌ Copiar texto de documentação oficial sem citar a fonte
- ❌ Traduzir conteúdo do projeto francês sem mencionar que é uma adaptação
- ❌ Apresentar exemplos XML baseados em dados reais sem creditar o operador
- ❌ Usar diagramas de terceiros sem atribuição

### Convenções

- Todos os ficheiros XML devem ser válidos contra o schema NeTEx XSD
- Comentários em português
- Nomes de ficheiro em snake_case com prefixo numérico do módulo
- UTF-8 sem BOM
- **Todas as fontes devem ser citadas** (ver secção Citações acima)
