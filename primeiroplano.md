# Plano de Orquestrador de Busca de Empregos

## Visão Geral
Criar um **maestro** que será o ponto de contato principal com o usuário e delegará tarefas a agentes especializados. O primeiro agente será responsável por buscar vagas de emprego na área de tecnologia (implementação futura).

## Componentes
1. **Maestro**
   - Agente central que interage com o usuário.
   - Possui habilidade (skill) de delegar trabalho usando a ferramenta nativa de despacho de agentes.
   - Segue o playbook abaixo.
2. **Agente de Busca de Vagas** (a ser criado futuramente)
   - Responsável por pesquisar vagas de tecnologia em fontes externas.
   - Recebe demandas do maestro.

## Skill do Maestro: Delegar Trabalho
- Nome da skill: `dispatch_agent`
- Descrição: Permite ao maestro criar e comunicar agentes auxiliares, passando parâmetros de contexto e recebendo resultados.
- Uso típico:
  ```
  result = dispatch_agent(agent_name, task_description, params)
  ```
- O maestro deve validar a resposta e encaminhar ao usuário ou continuar o fluxo.

## Playbook do Maestro
1. **Saudação ao usuário**
   - Ex.: "Olá! Eu sou o Maestro, seu assistente para encontrar oportunidades de carreira."
2. **Verificar existência de quiz**
   - Checar se já há respostas armazenadas (arquivo `quiz_respostas.json` ou similar).
3. **Caso não exista quiz**
   - Enviar **8 perguntas** para conhecer habilidades e preferências do usuário. **Perguntas a serem anotadas:**
     1. Quais são as suas principais linguagens de programação?
     2. Você prefere trabalho remoto, presencial ou híbrido?
     3. Qual tipo de contrato busca (CLT, PJ, estágio, trainee, etc.)?
     4. Qual nível de senioridade você busca (Júnior, Pleno, Sênior)?
     5. Com quais tecnologias ou stacks você gostaria de trabalhar?
     6. Qual a faixa salarial esperada (em reais)?
     7. Você tem disponibilidade para mudança de cidade ou país?
     8. Qual o tipo de empresa que mais lhe atrai (startup, grande corporação, consultoria, etc.)?
   - Armazenar respostas em `quiz_respostas.json`.
4. **Disponibilizar menu de opções**
   - Opções apresentadas ao usuário:
     - **Responder o quiz** (caso já exista ou queira atualizar).
     - **Buscar vagas de emprego** (funcionalidade futura, delegada ao agente de busca).
   - O maestro deve aguardar a escolha e agir de acordo.

## Próximos Passos
- Implementar o agente **Maestro** com a skill `dispatch_agent`.
- Criar a estrutura de armazenamento do quiz (`quiz_respostas.json`).
- Definir a interface de comunicação entre Maestro e o futuro agente de busca de vagas.
- Quando o agente de busca estiver pronto, integrar ao menu de opções do Maestro.
