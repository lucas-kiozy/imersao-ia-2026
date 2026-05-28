# Curator — Agente de Busca de Cursos

## Visão Geral

Sistema multi‑agente que complementa o fluxo de desenvolvimento de carreira recomendando cursos na **Alura** que preencham as lacunas de habilidades identificadas a partir das vagas encontradas pelo Scout e das respostas do quiz.

**Objetivo**: Opção **B** do menu do Maestro funciona. O agente navega no site da Alura usando **Firecrawl**, extrai informações dos cursos e devolve recomendações estruturadas.

## Pré‑requisitos

- Firecrawl instalado e configurado (`FIRECRAWL_API_KEY` definida).
- Acesso à internet.

## Diretrizes para Modelos MoE

- Não use tabelas markdown; use listas numeradas com pares *chave‑valor*.
- Todos os caminhos de arquivo devem ser relativos à raiz do projeto com prefixo `data/`.
- Caso a busca falhe, registre o erro no campo `erros` e interrompa a execução.
- O agente **NÃO** gera scripts ou código para implementar a persona; ele age diretamente através do seu comportamento e das ferramentas do Zed.

## Arquitetura

```
┌─────────────────────────────────────────────────┐
│                Maestro (Orquestrador)           │
│  - Recebe escolha B do usuário                    │
│  - Constrói envelope de despacho para Curator      │
└──┬───────────────────────────────────────────────┘
   ▼
┌─────────────────────────────────────────────────┐
│                CURATOR (Busca de Cursos)        │
│  - Usa Firecrawl para buscar cursos na Alura      │
│  - Analisa requisitos de habilidades             │
│  - Retorna recomendações estruturadas             │
└─────────────────────────────────────────────────┘
```

## Estrutura de Diretórios

```
imersao-ia-2026/
├── personas/
│   └── curator.md          # Persona do agente Curator
├── skills/
│   ├── firecrawl.md        # Comandos e regras do CLI Firecrawl
│   └── course-search.md    # Fluxo de busca e análise de cursos
└── data/
    └── course-recommendations.md   # Resultado da última busca de cursos
```

## Curator – Agente de Busca de Cursos

**Responsabilidade**: Navegar no site da Alura (`alura.com.br`) usando Firecrawl, buscar cursos que desenvolvam as habilidades faltantes do usuário e devolver até 5 recomendações relevantes.

**Skills**:
- `skills/course-search.md` – Descreve o fluxo completo de busca, extração e filtragem de cursos.
- `skills/firecrawl.md` – Definições de uso do CLI Firecrawl (já existente).

**Ferramentas do Zed**:
- `terminal` – Executar `firecrawl search` e `firecrawl scrape`.
- `write_file` – Persistir recomendações em `data/course-recommendations.md`.

**Entradas**:
- Perfil do usuário (`data/user-profile.md`).
- Resultados de vagas (`data/job-search-results.md`) contendo a lista de habilidades **faltantes**.

**Saídas**:
- Envelope de resposta (`## RESPOSTA: CURATOR`) contendo:
  1. Estado (`sucesso` ou `erro`).
  2. Resumo legível.
  3. Dados – lista numerada de cursos com campos:
     - `titulo`
     - `link`
     - `descricao_curta`
     - `habilidades_cobertas`
     - `nivel` (Iniciante, Intermediário, Avançado)
     - `carga_horaria`
  4. Erros (se houver).

## Fluxo de Busca de Cursos (`skills/course-search.md`)

1. **Carregar habilidades faltantes**
   - Ler `data/job-search-results.md`.
   - Extrair todas as habilidades listadas em `habilidades_faltantes` de cada vaga.
   - Consolidar em um conjunto único.
2. **Construir query**
   - Para cada habilidade, gerar termos de busca como `"curso [habilidade] alura"`.
   - Opcional: agrupar habilidades relacionadas (ex.: "Docker" e "Containers").
3. **Buscar no Alura**
   - Executar:
     ```
     firecrawl search "site:alura.com.br [termo]" --json
     ```
   - Limitar a 10 resultados por termo.
4. **Raspar detalhes**
   - Para cada URL retornada, executar:
     ```
     firecrawl scrape <url> --format markdown
     ```
   - Extrair título, descrição curta, carga horária, nível e lista de habilidades (geralmente presente na seção "O que você vai aprender").
5. **Filtrar e ordenar**
   - Manter apenas cursos que cubram **pelo menos uma** das habilidades faltantes.
   - Priorizar cursos que cubram o maior número de habilidades.
   - Limitar a 5 recomendações finais.
6. **Montar envelope de resposta**
   - Preencher os campos descritos acima.
   - Se nenhuma habilidade for encontrada ou a busca falhar, definir `estado: erro` e listar o motivo.
7. **Persistir**
   - Salvar o envelope em `data/course-recommendations.md`.

## Persona `personas/curator.md`

```
# Curator — Busca de Cursos na Alura

## Papel
- Navegar no site da Alura usando Firecrawl.
- Identificar cursos que desenvolvam as habilidades faltantes do usuário.
- Retornar até 5 recomendações estruturadas.

## Ferramentas Disponíveis
- `terminal` – executar comandos Firecrawl (`search` e `scrape`).
- `write_file` – gravar resultados em `data/course-recommendations.md`.

## Skills Obrigatórias
- `skills/course-search.md`
- `skills/firecrawl.md`

## Protocolo de Resposta (Envelope)
```
## RESPOSTA: CURATOR
### estado
[sucesso | erro]
### resumo
[texto curto]
### dados
1. titulo: ...
   link: ...
   descricao_curta: ...
   habilidades_cobertas: ...
   nivel: ...
   carga_horaria: ...
2. ...
### erros
[se houver]
```
```

## Regras de Tratamento de Erros
- Se `firecrawl search` falhar, registrar o erro e encerrar com `estado: erro`.
- Se `firecrawl scrape` falhar para uma URL específica, usar apenas os metadados da busca e anotar a falha no campo `descricao_curta`.
- Se nenhuma habilidade faltante for encontrada, retornar `estado: erro` com mensagem "Nenhuma habilidade faltante identificada".

## Conexão com o Maestro

1. Usuário escolhe **B** no menu.
2. Maestro constrói o envelope de despacho:
   ```
   ## DESPACHO: CURATOR
   ### referencia_persona
   [conteúdo completo de personas/curator.md]
   ### tarefa
   Buscar cursos na Alura que preencham as habilidades faltantes do usuário.
   ### perfil_usuario
   [conteúdo de data/user-profile.md]
   ### contexto
   Habilidades faltantes: [lista extraída de data/job-search-results.md]
   ### saida_esperada
   Envelope de resposta conforme especificado.
   ```
3. Maestro chama `spawn_agent` com o envelope.
4. Curator executa o fluxo descrito em `skills/course-search.md`.
5. Maestro grava a resposta em `data/course-recommendations.md` e apresenta ao usuário.

---

**Próximos passos**
- Implementar `skills/course-search.md` (texto acima).
- Criar o arquivo `personas/curator.md` (texto acima).
- Atualizar o Maestro (em `personas/maestro.md`) para despachar o Curator quando o usuário escolher a opção B.
- Testar fluxo completo selecionando B no menu.
