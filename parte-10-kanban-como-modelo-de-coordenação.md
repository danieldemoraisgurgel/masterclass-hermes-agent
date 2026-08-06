# Parte 10: Kanban como Modelo de Coordenação

Tudo nesta série até agora pressupôs:

- Um agente;
- Uma conversa;
- Uma tarefa por vez.

Você pede. O agente faz. Você recebe um resultado.

Esse é um modelo de **processamento serial**.

O Kanban rompe esse modelo. Ele oferece um quadro de tarefas durável, baseado em SQLite, que múltiplos agentes podem ler, reivindicar e atualizar.

O quadro:

- Sobrevive a reinicializações;
- Armazena tarefas com estados;
- Define responsáveis;
- Registra prioridades;
- Organiza tarefas por tags;
- Mantém o estado do trabalho entre sessões.

O agente não precisa se lembrar de qual trabalho está aberto. Ele consulta o quadro.

> O Kanban no Hermes não é apenas um recurso de gerenciamento de projetos. Ele é um modelo de coordenação de agentes para trabalhos que abrangem múltiplos perfis, múltiplas sessões e múltiplas instâncias de agentes.

---

## Workspace Kanban

Um workspace Kanban é armazenado como um banco de dados SQLite no seguinte diretório:

```text
~/.hermes/kanban/
```

O banco de dados armazena tarefas contendo:

- Título;
- Descrição;
- Estado;
- Responsável;
- Prioridade;
- Tags;
- Prazos;
- Dependências;
- Notas;
- Timestamps.

O agente interage com o quadro por meio da ferramenta `kanban`.

---

## Estados das Tarefas

As tarefas passam pelos seguintes estados:

| Estado | Descrição |
|---|---|
| `todo` | A tarefa foi criada, mas ainda não foi iniciada. |
| `in_progress` | A tarefa foi reivindicada e está sendo executada. |
| `review` | O trabalho foi concluído e aguarda revisão. |
| `done` | A tarefa foi concluída e aprovada. |
| `cancelled` | A tarefa foi cancelada ou deixou de fazer sentido. |

Uma nova tarefa normalmente é criada no estado `todo`.

Um perfil Hermes reivindica a tarefa, define-se como responsável e altera seu estado para `in_progress`.

Quando o trabalho está pronto para validação, a tarefa é movida para `review`.

Após a aprovação, ela é movida para `done`.

Caso o trabalho não seja mais necessário, a tarefa pode ser movida para `cancelled`.

---

## Responsáveis

Cada tarefa possui um campo de responsável que pode ser associado a um perfil Hermes.

Quando um perfil assume uma tarefa, ele reivindica sua propriedade.

Isso permite que outros perfis identifiquem:

- Quem está trabalhando na tarefa;
- Quais tarefas já foram reivindicadas;
- Quais tarefas ainda estão disponíveis;
- Onde existe risco de duplicação de esforços.

O responsável funciona como um mecanismo de coordenação entre agentes.

---

## Prioridades e Tags

Prioridades e tags ajudam os agentes a selecionar o trabalho correto no momento adequado.

### Prioridade

A prioridade classifica as tarefas dentro de um mesmo estado.

Por exemplo, entre várias tarefas no estado `todo`, o agente pode selecionar primeiro aquelas com prioridade mais alta.

### Tags

As tags agrupam tarefas relacionadas.

Exemplos:

```text
research
development
security
documentation
review
urgent
customer-a
project-x
```

O agente pode filtrar as tarefas por:

- Estado;
- Prioridade;
- Tag;
- Responsável;
- Prazo.

Isso permite localizar exatamente o trabalho que deve ser executado.

---

## Fluxo de Trabalho Kanban

O fluxo de trabalho Kanban possui três etapas principais:

1. Ler;
2. Reivindicar;
3. Executar.

---

### 1. Ler

O agente consulta o quadro em busca de tarefas que correspondam ao seu perfil e ao estado esperado.

Um perfil orquestrador pode, por exemplo, consultar todas as tarefas:

- No estado `todo`;
- Marcadas com a tag `research`.

Um perfil trabalhador pode consultar tarefas:

- No estado `in_progress`;
- Atribuídas ao próprio perfil.

O quadro retorna dados estruturados contendo o contexto necessário para compreender o trabalho.

Exemplo conceitual:

```json
{
  "id": 42,
  "title": "Pesquisar vulnerabilidades do componente X",
  "status": "todo",
  "priority": "high",
  "assignee": null,
  "tags": ["research", "security"],
  "blocked_by": [],
  "due_date": "2026-08-10"
}
```

---

### 2. Reivindicar

Após selecionar uma tarefa, o agente atribui a tarefa a si mesmo.

A tarefa passa de:

```text
todo
```

para:

```text
in_progress
```

O nome do perfil do agente é registrado no campo de responsável.

Outros perfis conseguem identificar que a tarefa já foi reivindicada e, portanto, devem ignorá-la.

Esse mecanismo reduz a duplicação de trabalho.

---

### 3. Executar

Depois de reivindicar a tarefa, o agente realiza o trabalho.

Ele pode utilizar qualquer ferramenta disponível em seu conjunto de ferramentas, como:

- Terminal;
- Sistema de arquivos;
- Busca na web;
- Navegador;
- Uso de computador;
- Delegação para outros agentes;
- Scripts;
- APIs;
- Ferramentas especializadas.

Quando o trabalho é concluído, o agente move a tarefa para:

```text
review
```

ou diretamente para:

```text
done
```

O agente também pode adicionar notas descrevendo:

- O que foi executado;
- O que foi encontrado;
- Quais arquivos foram modificados;
- Quais decisões foram tomadas;
- O que permanece pendente;
- Quais riscos foram identificados.

---

## Escalabilidade para Múltiplos Perfis

Esse padrão pode ser utilizado por múltiplos perfis executados simultaneamente.

Por exemplo:

- O **Perfil A** assume tarefas de pesquisa;
- O **Perfil B** assume tarefas de desenvolvimento;
- O **Perfil C** revisa o trabalho concluído.

Cada perfil:

1. Consulta sua faixa de trabalho;
2. Reivindica tarefas disponíveis;
3. Executa as atividades;
4. Atualiza o quadro;
5. Registra os resultados.

O quadro representa o estado compartilhado que coordena os agentes sem exigir comunicação direta entre eles.

---

# Dependências entre Tarefas

O quadro também pode gerenciar dependências.

Uma tarefa bloqueada armazena os identificadores das tarefas que precisam ser concluídas antes que ela possa avançar.

Exemplo:

```yaml
id: 18
title: Implantar aplicação em produção
status: todo
blocked_by:
  - 15
  - 16
```

Nesse exemplo, a tarefa `18` depende das tarefas `15` e `16`.

O agente não deve reivindicar a tarefa enquanto os bloqueadores não forem resolvidos.

Isso evita que múltiplos agentes desperdicem tempo tentando executar trabalhos que ainda não podem prosseguir.

---

## Exemplo de Dependência

Considere o seguinte fluxo:

```text
Tarefa 10: Pesquisar solução
        ↓
Tarefa 11: Definir arquitetura
        ↓
Tarefa 12: Desenvolver implementação
        ↓
Tarefa 13: Revisar implementação
        ↓
Tarefa 14: Implantar em produção
```

A tarefa de implantação não deve começar antes da conclusão da implementação e da revisão.

O quadro Kanban mantém essas dependências explicitamente registradas.

---

# Prazos

Prazos são opcionais, mas podem ajudar na priorização.

Uma tarefa com prazo próximo pode receber atenção antes de uma tarefa sem data definida.

Exemplo:

```yaml
title: Renovar certificado TLS
priority: high
due_date: 2026-08-08
```

Combinados, prazo e prioridade criam uma ordem natural de triagem.

O agente pode considerar:

1. Tarefas vencidas;
2. Tarefas com prazo próximo;
3. Tarefas de alta prioridade;
4. Tarefas sem bloqueadores;
5. Tarefas sem prazo definido.

---

# Limites de Trabalho em Progresso

O Kanban pode estabelecer limites para a quantidade de tarefas permitidas em determinado estado.

Esses limites são conhecidos como **Work in Progress Limits**, ou limites de WIP.

Exemplo:

```yaml
wip_limits:
  in_progress: 5
  review: 3
```

Nesse cenário:

- No máximo cinco tarefas podem permanecer em `in_progress`;
- No máximo três tarefas podem permanecer em `review`.

Se já existem três tarefas em revisão, isso indica que a etapa de revisão pode ser o gargalo do processo.

O perfil responsável pela revisão deve limpar sua fila antes que o sistema continue puxando novos trabalhos.

Os limites de WIP ajudam a revelar:

- Gargalos;
- Sobrecarga de perfis;
- Filas acumuladas;
- Etapas lentas;
- Problemas de capacidade.

---

# Notas de Ciclo de Vida

As notas de ciclo de vida documentam o que aconteceu durante a execução de uma tarefa.

Quando uma tarefa passa de `in_progress` para `review`, o agente pode registrar:

- O que foi feito;
- O que foi encontrado;
- Como o resultado foi validado;
- O que não foi concluído;
- Quais riscos permanecem;
- Quais decisões precisam ser tomadas pelo revisor.

Exemplo:

```markdown
## Notas da transição para review

- Implementada a validação de entrada.
- Adicionados testes unitários.
- Corrigida falha de autenticação.
- Testes executados com sucesso.
- Permanece pendente a validação de desempenho.
```

O perfil de revisão lê essas informações antes de iniciar sua análise.

As notas se acumulam ao longo das transições de estado, construindo uma trilha de auditoria completa.

---

# Coordenação Multiagente

O quadro Kanban se destaca quando múltiplos perfis Hermes compartilham o mesmo workspace.

Cada perfil representa um agente separado, com:

- Configuração própria;
- Memória própria;
- Habilidades próprias;
- Ferramentas próprias;
- Responsabilidades próprias;
- Ambiente de execução próprio.

O quadro funciona como a ponte entre esses agentes.

---

## Configuração Típica

Uma configuração comum utiliza três perfis:

1. Orquestrador;
2. Trabalhador;
3. Revisor.

---

### Perfil Orquestrador

O orquestrador cria, classifica e distribui tarefas.

Ele pode preencher o quadro a partir de fontes externas, como:

- Um trabalho cron que lê um feed RSS;
- Uma mensagem recebida por um gateway;
- Uma solicitação manual;
- Um webhook;
- Um formulário;
- Uma fila;
- Um sistema de tickets;
- Um repositório de código.

Exemplo de fluxo:

```text
Fonte externa
      ↓
Orquestrador
      ↓
Criação de tarefas em todo
```

---

### Perfil Trabalhador

O trabalhador consulta tarefas disponíveis, reivindica uma delas e executa o trabalho.

Fluxo típico:

```text
todo
  ↓
in_progress
  ↓
review
```

O trabalhador pode ser especializado em:

- Pesquisa;
- Desenvolvimento;
- Infraestrutura;
- Segurança;
- Documentação;
- Análise de dados;
- Automação;
- Operações.

---

### Perfil Revisor

O revisor valida tarefas que estão no estado `review`.

Ele pode:

- Aprovar o trabalho;
- Solicitar alterações;
- Registrar problemas;
- Devolver a tarefa para retrabalho;
- Marcar a tarefa como concluída.

Fluxo de aprovação:

```text
review
  ↓
done
```

Fluxo de retrabalho:

```text
review
  ↓
in_progress
```

---

## Execução Independente

Cada perfil é executado independentemente.

O orquestrador não precisa aguardar o trabalhador.

O trabalhador não precisa aguardar o revisor.

O quadro é o ponto de sincronização entre eles.

Se o trabalhador estiver offline, as tarefas se acumulam no estado `todo`.

Quando ele voltar a operar, consulta o quadro e retoma o trabalho de onde parou.

Da mesma forma, se o revisor estiver indisponível, as tarefas permanecem em `review` até que possam ser analisadas.

---

## Coordenação Distribuída

A documentação do Hermes apresenta um tutorial de Kanban no qual três perfis são conectados utilizando estado Raft compartilhado para consistência distribuída.

Nesse modelo, perfis podem estar em:

- Máquinas diferentes;
- Redes diferentes;
- Ambientes diferentes;
- Regiões diferentes;
- Instâncias independentes.

Mesmo sem um canal direto de comunicação, eles conseguem se coordenar por meio do quadro compartilhado.

O quadro representa o estado comum do sistema.

---

# Quando o Kanban Supera o Chat Direto

O trabalho com um único agente é mais simples.

O fluxo é direto:

```text
Usuário solicita
      ↓
Agente executa
      ↓
Usuário recebe o resultado
```

Não existe quadro, transição de estados ou coordenação entre perfis.

O Kanban adiciona sobrecarga operacional:

- Criação de tarefas;
- Gerenciamento de estados;
- Atribuição de responsáveis;
- Registro de notas;
- Coordenação entre perfis;
- Controle de dependências;
- Revisão do trabalho.

Cada etapa adiciona um nível de atrito que o chat direto não possui.

Essa sobrecarga é justificável quando o trabalho apresenta determinadas características.

---

## Trabalho de Longa Duração

Uma tarefa que leva horas ou dias se beneficia de um quadro durável.

O quadro sobrevive a:

- Reinicializações;
- Encerramento de sessões;
- Falhas do agente;
- Manutenções;
- Trocas de perfil;
- Interrupções de conectividade.

O chat direto pode perder parte do contexto quando a sessão termina.

O quadro mantém o estado do trabalho persistido.

---

## Trabalho com Múltiplas Etapas

Uma tarefa que exige pesquisa, desenvolvimento, revisão e aprovação se beneficia de transições explícitas de estado.

Exemplo:

```text
Pesquisa
   ↓
Desenvolvimento
   ↓
Revisão
   ↓
Aprovação
   ↓
Conclusão
```

Cada etapa possui:

- Um responsável;
- Um estado;
- Um critério de conclusão;
- Um registro do trabalho executado;
- Um ponto explícito de transferência.

---

## Trabalho Multiagente

Tarefas que exigem habilidades diferentes se beneficiam do roteamento por perfil.

Por exemplo:

| Perfil | Especialidade |
|---|---|
| Orquestrador | Planejamento e decomposição de trabalho |
| Pesquisador | Busca e análise de informações |
| Desenvolvedor | Implementação técnica |
| Revisor | Validação e controle de qualidade |
| Operador | Implantação e execução operacional |

O orquestrador não precisa possuir as mesmas habilidades ou ferramentas do trabalhador.

Cada perfil pode ser especializado em uma função.

---

## Trabalho Assíncrono

Uma tarefa criada à meia-noite pode ser reivindicada por um trabalhador às 8h.

O criador da tarefa e o executor não precisam estar ativos ao mesmo tempo.

O quadro permanece disponível como ponto de coordenação.

Esse modelo é adequado para:

- Processamento em lotes;
- Filas de trabalho;
- Operações distribuídas;
- Equipes em fusos horários diferentes;
- Agentes executados por cron;
- Tarefas que dependem de eventos externos.

---

# O Quadro como Memória do Sistema

Um quadro Kanban com histórico completo de tarefas representa um registro operacional de:

- O que foi feito;
- Quando foi feito;
- Quem executou;
- Quanto tempo levou;
- Quais resultados foram encontrados;
- Quais problemas ocorreram;
- Quais tarefas foram canceladas;
- Onde existem gargalos.

Cada movimentação, nota e alteração de estado é persistida no SQLite.

O quadro pode responder perguntas que o chat direto normalmente não consegue responder de forma confiável, como:

- Quantas tarefas foram concluídas nesta semana?
- Quais perfis estão sobrecarregados?
- Qual etapa do pipeline é o principal gargalo?
- Quantas tarefas foram canceladas?
- Quais tarefas estão bloqueadas?
- Quais tarefas estão próximas do prazo?
- Quanto tempo uma tarefa permanece em revisão?
- Qual perfil conclui mais tarefas?
- Quais tags concentram mais trabalho?

---

## Kanban, Memória e Habilidades

O quadro não substitui memória nem habilidades.

Ele complementa esses componentes.

| Componente | Função |
|---|---|
| Memória | Armazena fatos e informações duráveis. |
| Habilidades | Armazenam procedimentos e formas de executar tarefas. |
| Kanban | Armazena o estado do trabalho em andamento. |

Em conjunto, esses três sistemas dão ao agente consciência sobre:

1. O que ele sabe;
2. Como executar determinada atividade;
3. Em que tarefa deveria estar trabalhando.

---

# Separação de Responsabilidades

O quadro coordena o trabalho entre perfis, mas cada perfil ainda precisa de seu próprio ambiente de execução.

Cada agente pode possuir:

- Prompt de sistema específico;
- Conjunto de ferramentas;
- Credenciais;
- Permissões;
- Diretório de trabalho;
- Memória;
- Habilidades;
- Restrições de segurança;
- Recursos computacionais.

O Kanban não executa o trabalho.

Ele mantém o estado, distribui as responsabilidades e registra o progresso.

A execução continua sendo responsabilidade dos perfis Hermes.

---

# Exemplo de Arquitetura

```text
                   ┌──────────────────────┐
                   │    Fonte externa     │
                   │ RSS, cron, gateway,  │
                   │ usuário ou webhook   │
                   └──────────┬───────────┘
                              │
                              ▼
                   ┌──────────────────────┐
                   │    Orquestrador      │
                   │ Cria e classifica    │
                   │ tarefas              │
                   └──────────┬───────────┘
                              │
                              ▼
                 ┌──────────────────────────┐
                 │      Quadro Kanban       │
                 │ SQLite / estado comum    │
                 │                          │
                 │ todo                     │
                 │ in_progress              │
                 │ review                   │
                 │ done                     │
                 │ cancelled                │
                 └───────┬─────────┬────────┘
                         │         │
              ┌──────────┘         └──────────┐
              ▼                               ▼
    ┌─────────────────────┐       ┌─────────────────────┐
    │ Perfil Trabalhador  │       │   Perfil Revisor    │
    │ Reivindica e        │       │ Valida, aprova ou   │
    │ executa tarefas     │       │ solicita retrabalho │
    └─────────────────────┘       └─────────────────────┘
```

---

# Exemplo de Ciclo Completo

```text
1. O orquestrador cria a tarefa.

   Estado: todo
   Responsável: nenhum

2. O trabalhador consulta o quadro.

3. O trabalhador reivindica a tarefa.

   Estado: in_progress
   Responsável: worker-security

4. O trabalhador executa a atividade.

5. O trabalhador registra as notas de execução.

6. O trabalhador envia a tarefa para revisão.

   Estado: review

7. O revisor analisa o resultado.

8. O revisor aprova a tarefa.

   Estado: done
```

Caso o revisor encontre problemas:

```text
review
  ↓
in_progress
```

O revisor registra o motivo da devolução, e o trabalhador realiza o retrabalho.

---

# Conclusão

O Kanban transforma o Hermes de um agente que apenas responde a solicitações em um sistema capaz de coordenar trabalho durável, assíncrono e distribuído.

Ele é especialmente útil quando existem:

- Múltiplos agentes;
- Múltiplas etapas;
- Dependências;
- Revisões;
- Prazos;
- Tarefas de longa duração;
- Necessidade de auditoria;
- Execução em máquinas ou sessões diferentes.

O chat direto continua sendo a melhor opção para tarefas simples, rápidas e executadas por um único agente.

O Kanban passa a ser vantajoso quando o trabalho precisa sobreviver à conversa e ser coordenado como parte de um fluxo operacional.

> A memória guarda o que o agente sabe.  
> As habilidades guardam como o agente trabalha.  
> O Kanban guarda no que o agente está trabalhando.