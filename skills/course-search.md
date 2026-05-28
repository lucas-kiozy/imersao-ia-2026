# Fluxo de Busca de Cursos (Curator)

## Objetivo
Definir o fluxo que o agente Curator deve seguir para buscar cursos faltantes usando o CLI `firecrawl`, analisar os resultados e produzir um envelope de resposta estruturado.

## Entrada
- `data/job-search-results.md` – contém as vagas e, para cada vaga, a lista `habilidades_faltantes`.
- `data/user-profile.md` – perfil do usuário (não usado diretamente, mas pode ser útil para filtrar nível).

## Saída
Um envelope de resposta gravado em `data/course-recommendations.md` com o formato:
```
## RESPOSTA: CURATOR
### estado
[sucesso | erro]
### resumo
[texto curto]

### dados
1. titulo: ...
   link: ...
   preço: ...
   descricao_curta: ...
   habilidades_cobertas: ...
   nivel: ...
   carga_horaria: ...
2. ...
### erros
[se houver]
```

## Passos Detalhados

1. **Carregar habilidades faltantes**
   - Ler `data/job-search-results.md`.
   - Extrair todas as linhas que começam com `habilidades_faltantes:`.
   - Separar por vírgula, remover espaços e normalizar (minúsculas).
   - Consolidar em um `HashSet` para eliminar duplicatas.

2. **Construir queries**
   - Para cada habilidade, montar a string de busca: `"curso {habilidade}"`.
   - Agrupar habilidades relacionadas (ex.: `docker` e `containers` → usar apenas `docker`).
   - Manter uma lista de termos de busca.

3. **Buscar na Web**
   - Para cada termo, executar via terminal:
     ```
     firecrawl search {term} --json
     ```
   - Capturar a saída JSON (array de objetos com `url`, `title`, `description`).
   - Limitar a **10** resultados por termo.
   - Acumular resultados únicos (por URL).

4. **Raspar detalhes dos cursos**
   - Para cada URL obtida, executar:
     ```
     firecrawl scrape <url> --format markdown
     ```
   - Analisar o markdown para extrair:
     - **Título** – geralmente a primeira linha `#`.
     - **Preço** – procurar por preço (como `R$ <valor>` ou `\d+\s*reais?`) ou gratuito (como `Gratuito` ou `\d+\s*gratis?`).
     - **Descrição curta** – primeiro parágrafo ou texto até a primeira quebra dupla.
     - **Carga horária** – procurar padrão `\d+\s*horas?` ou `\d+\s*h`.
     - **Nível** – buscar palavras `Iniciante`, `Intermediário`, `Avançado` (case‑insensitive).
     - **Habilidades cobertas** – seção típica "O que você vai aprender"; extrair itens de lista e normalizar.
   - Se o `scrape` falhar, usar apenas os metadados da busca (`title` e `url`) e marcar `descricao_curta` como "Detalhes não disponíveis (falha ao raspar)."

5. **Filtrar e ordenar**
   - Manter somente cursos onde `habilidades_cobertas` intersecta o conjunto de habilidades faltantes.
   - Calcular `num_cobertas = |interseção|`.
   - Ordenar por `num_cobertas` decrescente e, em caso de empate, por nível (Avançado > Intermediário > Iniciante) e por carga horária (menor primeiro).
   - Selecionar até **5** cursos.

6. **Montar envelope de resposta**
   - Se nenhum curso for encontrado, definir `estado: erro` e `erros: "Nenhum curso encontrado para as habilidades faltantes"`.
   - Caso contrário, `estado: sucesso`.
   - Preencher `resumo` com algo como "Encontrei X cursos que cobrem Y das habilidades faltantes."
   - Listar cada curso no formato especificado.

7. **Persistir**
   - Gravar o envelope completo em `data/course-recommendations.md` usando `write_file`, conforme definido em `plano-3.md`.

## Tratamento de Erros
- **Falha na leitura de arquivos** – registrar erro e abortar com `estado: erro`.
- **Falha no `firecrawl search`** – registrar a mensagem de erro retornada e abortar.
- **Falha no `firecrawl scrape`** – registrar a falha apenas para aquele curso, mas ainda incluir o curso usando os metadados disponíveis.
- **Nenhuma habilidade faltante** – abortar imediatamente com `estado: erro` e mensagem "Nenhuma habilidade faltante identificada".

## Observações
- Todos os caminhos são relativos à raiz do projeto (`imersao-ia-2026`).
- Não usar tabelas markdown; usar listas numeradas com pares chave‑valor.
- O agente não gera código Python ou scripts; ele age através das ferramentas `terminal` e `write_file`.
