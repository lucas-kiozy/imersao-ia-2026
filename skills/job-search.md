## Job Search Skill (Scout)

**Objetivo**: Definir o fluxo que o agente Scout deve seguir para buscar vagas de emprego usando o CLI `firecrawl`, analisar os resultados e produzir um envelope de resposta estruturado.

### 1. Entrada
- **Área de interesse** (ex.: "frontend", "data science").
- **Localização** (ex.: "São Paulo", "Remoto").
- **Nível de experiência** (Júnior, Pleno, Sênior) – extraído de `data/user-profile.md`.
- **Habilidades atuais** – lista de completa de habilidades do usuário, também de `data/user-profile.md`.

### 2. Busca de vagas
1. Executar no terminal:
   ```
   firecrawl search "vagas {area} {localizacao}" --json
   ```
   - O comando devolve um JSON contendo, para cada resultado, `url`, `title`, `description` e, se disponível, `company`.
2. Se o comando falhar, registrar o erro e encerrar com `estado: erro`.

### 3. Extração de detalhes da vaga
Para cada URL retornada (mínimo de 8 vagas até um máximo de 15 vagas):
1. Executar:
   ```
   firecrawl scrape {url} --format markdown
   ```
2. Caso a extração falhe, usar apenas o `title` e `description` obtidos na busca e anotar que a extração falhou.

### 4. Análise de requisitos
- **Extrair habilidades requeridas**: identificar no texto (markdown) palavras‑chave que correspondam a habilidades típicas esperadas (ex.: "Power Platform", "Python", "SQL").
- **Empresa**: tentar inferir a partir do título ou da URL (domínio).
- **Localização**: usar a informação presente no título da vaga ou na sua descrição.
- **Nível de experiência**: se o texto mencionar "Junior", "Pleno" ou "Sênior", usar essa indicação.

### 5. Correspondência de habilidades
- Recolher todas as habilidades do usuário em `data/user_profile.md`.
- Normalizar ambas as listas (minúsculas, remover acentos).
- `habilidades_correspondentes` = interseção entre habilidades do usuário e habilidades requeridas pela vaga.
- `habilidades_faltantes` = diferença de habilidades (habilidades requeridas pela vaga – habilidades do usuário).
- `contagem_correspondencia` = "X de Y habilidades correspondem" onde X = tamanho da interseção e Y = total de habilidades requeridas.

### 6. Filtragem por nível
- Priorizar vagas cujo nível coincida com o nível do usuário.
- Se nas primeiras 8 vagas não houver coincidência, expandir a busca (remover filtro de nível) e incluir vagas adjacentes, anotando a discrepância no campo `nivel_discrepancia`.

### 7. Formato da resposta (Envelope de Resposta)
```
## RESPOSTA: SCOUT
### estado
sucesso | erro

### resumo
[2‑3 frases resumindo o resultado]

### dados
1. titulo: …
   empresa: …
   localizacao: …
   link da vaga: …
   habilidades_correspondentes: [h1, h2, h3,...]
   habilidades_faltantes: [h4, h5,...]
   contagem_correspondencia: X de Y habilidades correspondem
   (nivel_discrepancia: "Nível diferente" – opcional)
2. …

### erros
[se estado = erro]
```

### 8. Tratamento de erros
- Falha no `firecrawl search`: preencher `estado: erro`, `erros` com a mensagem do terminal e encerrar.
- Falha no `firecrawl scrape` de uma URL específica: continuar com as demais vagas, usar título/descrição da busca e acrescentar a observação "extração falhou" ao registro da vaga.
- Qualquer outra exceção deve ser capturada, registrada em `erros` e resultar em `estado: erro`.

### 9. Persistência
- O Maestro salvará o envelope completo em `data/job-search-results.md` conforme o esquema definido em `plano-2.md`.

---
*Esta skill deve ser referenciada por `personas/scout.md` usando a diretiva **referência obrigatória**.*
