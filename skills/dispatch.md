## Protocolo de Despacho e Handoff de Agentes

### Tabela de Roteamento
| Código | Agente   |
|--------|----------|
| A      | Scout    |
| B      | Curator  |
| C      | Coach    |
| D      | Maestro (lida com o quiz) |

### Envelope de Despacho (Maestro constrói este prompt para `spawn_agent`)
```
## DESPACHO: [NOME_DO_AGENTE]
### referencia_persona
[Conteúdo completo de personas/<nome_do_agente_minusculo>.md]

### tarefa
[Uma frase descrevendo o que o agente deve fazer]

### perfil_usuario
[Conteúdo de data/user-profile.md]

### contexto
[Contexto específico: ex: qual vaga para entrevistar, quais habilidades buscar cursos]

### saida_esperada
[Exatamente em que formato o agente deve retornar]
```

### Envelope de Resposta (agente despachado retorna isto)
```
## RESPOSTA: [NOME_DO_AGENTE]
### estado
[sucesso | erro]

### resumo
[Resumo legível de 2-3 frases para o usuário]

### dados
[Resultados como listas numeradas com pares chave-valor. Sem tabelas markdown.]

### erros
[Apenas se estado for erro: o que deu errado]
```

### Especificações de Handoff por Agente
- **Scout**: Recebe `query` de busca de vagas. Retorna lista de vagas em `data/scout-results.md` e resumo no envelope.
- **Curator**: Recebe `skill_gap` e retorna cursos recomendados em `data/curator-results.md`.
- **Coach**: Recebe `perfil_usuario` e conduz entrevista simulada em 6 passos, retornando `data/coach-interview.md`.

### Despacho Sequencial do Coach (6 despachos)
1. Pergunta inicial de entrevista.
2. Avalia resposta e pede detalhes.
3. Explora competências técnicas.
4. Simula pergunta comportamental.
5. Fornece feedback.
6. Conclui e gera relatório.

### Regras de Tratamento de Erros
- Cada agente deve preencher o campo `erros` no envelope de resposta caso encontre falha.
- O Maestro deve registrar erros em `data/agent-errors.log` e apresentar resumo ao usuário.
- Se o agente retornar `estado: erro`, o Maestro pode oferecer opção de retry ou fallback.
