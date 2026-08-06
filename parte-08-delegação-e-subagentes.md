# Parte 08: Delegação e Subagentes

Uma única sessão de agente é um processo serial:

1. Uma mensagem entra.
2. O modelo pensa.
3. O modelo chama ferramentas.
4. Os resultados são avaliados.
5. Uma resposta é produzida.

Tudo acontece em sequência.

Para a maioria das tarefas, isso é suficiente. Porém, em atividades como pesquisa, revisão de código, refatoração e outros fluxos de trabalho paralelizáveis, o processamento serial se torna um gargalo.

A delegação resolve esse problema iniciando agentes filhos que trabalham de forma independente e em paralelo.

Cada agente filho recebe:

- Sua própria conversa;
- Sua própria sessão de terminal;
- Seu próprio conjunto de ferramentas;
- Seu próprio orçamento de execução.

Apenas o resumo final retorna ao agente pai.

O trabalho intermediário dos filhos — chamadas de ferramentas, erros, tentativas malsucedidas, becos sem saída e ruído de depuração — permanece no contexto de cada subagente e nunca entra no contexto do agente pai.

---

## Como a delegação funciona

A ferramenta `delegate_task` inicia uma nova instância de `AIAgent`.

O agente filho começa com uma conversa completamente limpa e não possui nenhum conhecimento sobre o histórico do agente pai.

O único contexto recebido pelo subagente é aquilo que o pai fornece nos campos:

- `goal`;
- `context`.

Esse é o aspecto mais importante da delegação:

> Subagentes não sabem nada além do que lhes é explicitamente fornecido.

Se você delegar uma tarefa usando apenas a instrução:

```text
Corrija o erro.
```

o subagente não saberá a qual erro você está se referindo.

O objetivo precisa ser autossuficiente. Todos os detalhes necessários devem ser informados nos campos `goal` e `context`, incluindo:

- Caminhos de arquivos;
- Mensagens de erro;
- Convenções do projeto;
- Restrições técnicas;
- Resultados esperados;
- Critérios de validação;
- Comandos que devem ser executados;
- Informações relevantes sobre o ambiente.

Uma tarefa individual cria um único agente filho.

Um lote pode criar até três agentes filhos por padrão, executados simultaneamente em um pool de threads.

O agente pai permanece bloqueado até que todos os filhos concluam suas tarefas.

Os resultados retornam na ordem em que as tarefas foram enviadas, independentemente da ordem em que foram concluídas.

---

## Quando delegar

A delegação é indicada para tarefas que exigem raciocínio, julgamento e iteração, como:

- Depuração complexa;
- Refatoração de vários arquivos;
- Pesquisa comparativa;
- Revisão de código;
- Auditoria de segurança;
- Análise arquitetural;
- Avaliação de diferentes abordagens;
- Investigação de problemas com múltiplas hipóteses.

O subagente recebe o mesmo ciclo de raciocínio baseado em LLM disponível para o agente pai.

Ele pode:

- Chamar ferramentas;
- Avaliar resultados;
- Iterar sobre uma solução;
- Recuar quando uma abordagem falhar;
- Testar hipóteses;
- Produzir um resultado estruturado.

### Quando usar `execute_code`

A ferramenta `execute_code` é a alternativa para tarefas que não exigem raciocínio.

Ela é adequada para atividades como:

- Escrever e executar um script Python;
- Processar arquivos em lote;
- Analisar dados;
- Converter formatos;
- Executar uma sequência determinística de comandos;
- Extrair informações estruturadas;
- Automatizar operações repetitivas.

Esse modelo é mais barato e mais rápido porque não existe um ciclo de raciocínio baseado em LLM. Há apenas a execução do script.

A regra prática é:

- Use **subagentes** para trabalhos que exigem julgamento.
- Use **`execute_code`** para trabalhos determinísticos.

---

## Padrões comuns de delegação

### Pesquisa paralela

Envie três subagentes para pesquisar três tópicos diferentes simultaneamente.

Cada agente:

1. Executa pesquisas independentes na web;
2. Analisa suas próprias fontes;
3. Organiza as descobertas;
4. Retorna um resumo estruturado.

O agente pai recebe os três resultados e produz uma síntese final.

Esse é um dos casos de uso de maior retorno sobre investimento para a maioria dos usuários.

### Revisão de código e correção

Delegue a um subagente uma auditoria de segurança do módulo de autenticação.

O agente filho pode:

1. Revisar o código;
2. Encontrar vulnerabilidades;
3. Corrigir os problemas;
4. Executar testes;
5. Validar o comportamento;
6. Retornar um resumo das alterações.

O agente pai não precisa receber todo o processo de depuração. Ele recebe apenas o resultado final e, quando aplicável, o conjunto de alterações.

### Refatoração de vários arquivos

Uma refatoração que modifica 20 arquivos pode gerar uma quantidade muito grande de contexto intermediário.

Ao delegar a tarefa:

1. O subagente trabalha em cada arquivo;
2. Realiza as modificações;
3. Executa verificações;
4. Corrige falhas;
5. Retorna um resumo;
6. Entrega o `diff` produzido.

O agente pai carrega apenas o resultado consolidado, mantendo sua sessão principal limpa.

---

## Seleção do conjunto de ferramentas

Cada subagente recebe seu próprio parâmetro `toolset`.

O conjunto de ferramentas deve ser limitado ao necessário para a tarefa.

### Subagentes de pesquisa

```json
["web"]
```

### Subagentes de código

```json
["terminal", "file"]
```

### Tarefas full-stack

```json
["terminal", "file", "web"]
```

Restringir o conjunto de ferramentas reduz o consumo de tokens.

Por exemplo, um subagente de pesquisa configurado apenas com ferramentas web não precisa carregar em seu prompt os esquemas das ferramentas de:

- Terminal;
- Arquivos;
- Navegador;
- Imagem;
- Outras integrações não utilizadas.

Com menos ferramentas disponíveis:

- O prompt fica menor;
- O índice de ferramentas é reduzido;
- O modelo possui menos opções para avaliar;
- A seleção de ferramentas tende a ser mais objetiva;
- O custo de execução diminui.

### Ferramentas bloqueadas

Algumas ferramentas são bloqueadas para subagentes independentemente do que for especificado no `toolset`.

#### `clarify`

A ferramenta `clarify` é bloqueada porque subagentes não podem interagir diretamente com o usuário.

Todas as informações necessárias precisam ser fornecidas pelo agente pai.

#### `memory`

A ferramenta `memory` é bloqueada porque subagentes não devem gravar em um estado persistente compartilhado.

#### `delegation`

A ferramenta de delegação é bloqueada para subagentes folha, evitando a criação recursiva e descontrolada de novos agentes.

---

## O modelo assíncrono

A ferramenta original de delegação bloqueava o chat do agente pai enquanto os filhos eram executados.

Ao iniciar três subagentes de pesquisa, o usuário precisava aguardar a conclusão de todos. Se um dos agentes travasse, era necessário esperar até que ele terminasse ou cancelar o lote inteiro.

Os subagentes assíncronos corrigiram essa limitação.

A ferramenta `delegate_task_async` inicia um subagente e retorna imediatamente.

O agente pai pode continuar trabalhando enquanto o filho executa sua tarefa.

Um novo conjunto de ações permite controlar os subagentes assíncronos:

| Ação | Finalidade |
|---|---|
| `check` | Verificar o progresso de uma tarefa |
| `steer` | Redirecionar um subagente durante a execução |
| `collect` | Coletar os resultados concluídos |
| `cancel` | Cancelar uma tarefa que não é mais necessária |
| `list` | Listar os subagentes e seus estados |

O fluxo passa a ser:

1. Iniciar as tarefas;
2. Continuar trabalhando;
3. Verificar o progresso quando necessário;
4. Redirecionar agentes que estejam seguindo uma abordagem inadequada;
5. Coletar os resultados quando estiverem prontos;
6. Cancelar tarefas que perderam relevância.

Esse é o modelo que faz a delegação se comportar como paralelismo real, em vez de apenas trabalho sequencial executado por processos separados.

> Dispare e continue. Verifique depois. Colete quando estiver pronto.

---

## Visualização dos agentes

O comando com barra `/agents`, disponível na TUI, transforma a distribuição dos subagentes em uma visualização de árvore ao vivo.

A interface pode exibir:

- Agentes filhos em execução;
- Resultados concluídos;
- Custos por ramificação;
- Consumo de tokens;
- Totais agregados;
- Controles para encerrar tarefas;
- Controles para pausar tarefas;
- Histórico turno a turno.

A CLI clássica imprime um resumo em texto.

A TUI apresenta essas informações como uma sobreposição interativa.

---

## Orçamento de iteração e timeout

Cada subagente possui um limite de iterações.

Esse limite controla quantos turnos de chamadas de ferramentas o agente poderá realizar.

O valor padrão é:

```text
50 iterações
```

Uma tarefa simples de verificação de arquivo pode consumir apenas cinco iterações.

Uma revisão de código aprofundada pode utilizar todas as 50.

### Timeout

Por padrão, o timeout rígido do agente filho foi removido.

Versões anteriores possuíam um limite fixo de 300 segundos, equivalente a cinco minutos. Esse limite encerrava tarefas legítimas de longa duração.

Atividades como estas frequentemente precisam de mais de dez minutos:

- Revisões aprofundadas;
- Grandes lotes de pesquisa;
- Refatorações extensas;
- Modelos de raciocínio mais lentos;
- Auditorias com muitos arquivos;
- Investigações que exigem várias rodadas de testes.

Subagentes travados são detectados por meio de um monitor de desatualização de `heartbeat`.

Quando for necessário impor um limite rígido para controle de custos — especialmente em delegações não supervisionadas acionadas por `cron` — o timeout pode ser habilitado explicitamente por configuração.

---

## Profundidade de orquestração

Por padrão, a delegação é plana.

Nesse modelo:

```text
Agente pai
├── Agente filho 1
├── Agente filho 2
└── Agente filho 3
```

O agente pai cria filhos, mas esses filhos não podem criar seus próprios subagentes.

Essa restrição evita árvores descontroladas de delegação recursiva.

### Orquestração em múltiplos estágios

Em fluxos de trabalho mais complexos, um agente filho pode atuar como orquestrador e manter acesso ao conjunto de ferramentas de delegação.

Esse agente orquestrador pode criar seus próprios trabalhadores folha.

Exemplo:

```text
Agente pai
└── Agente orquestrador
    ├── Trabalhador 1
    ├── Trabalhador 2
    └── Trabalhador 3
```

O limite de profundidade padrão é:

```text
1
```

Esse valor representa uma estrutura plana.

Para permitir um nível adicional de aninhamento, aumente a profundidade para:

```text
2
```

Cada nível adicional multiplica rapidamente o número potencial de agentes.

Com três filhos simultâneos por agente, uma profundidade 3 pode produzir até:

```text
3 × 3 × 3 = 27 agentes folha
```

A profundidade deve ser aumentada intencionalmente, considerando:

- Custo;
- Consumo de tokens;
- Uso de ferramentas;
- Concorrência;
- Complexidade operacional;
- Dificuldade de depuração;
- Risco de duplicação de trabalho.

---

## Quando o paralelismo muda o fluxo de trabalho

### Sessões com um único agente

Sessões de agente único são:

- Simples;
- Previsíveis;
- Fáceis de acompanhar;
- Fáceis de depurar.

Você visualiza cada chamada de ferramenta e sabe exatamente o que o agente está fazendo.

### Delegação paralela

A delegação paralela introduz mais incerteza operacional.

Três subagentes podem estar sendo executados simultaneamente. O agente pai não visualiza automaticamente todo o trabalho intermediário, a menos que consulte o estado das tarefas.

Um dos agentes pode:

- Travar;
- Seguir uma hipótese incorreta;
- Duplicar o trabalho de outro agente;
- Consumir mais iterações do que o esperado;
- Retornar um resultado incompleto.

No modelo síncrono, o pai permanece bloqueado até que todos os subagentes terminem.

A troca vale a pena quando o paralelismo economiza um tempo significativo.

Três pesquisas executadas em sequência podem levar aproximadamente três vezes o tempo de uma única pesquisa.

Três pesquisas executadas em paralelo levam aproximadamente o tempo da tarefa mais lenta.

Essa abordagem é especialmente útil para trabalhos comparativos, como:

- Testar três abordagens;
- Pesquisar três fornecedores;
- Revisar três módulos;
- Analisar três arquiteturas;
- Investigar três hipóteses;
- Avaliar três implementações.

Uma espera total de 15 minutos pode ser reduzida para aproximadamente cinco minutos, dependendo da duração da tarefa mais lenta.

---

## Delegação assíncrona como colaboração

O modelo assíncrono reduz ou elimina o tempo ocioso do agente pai.

Em vez de aguardar passivamente:

1. O agente inicia os subagentes;
2. Continua executando outras partes do trabalho;
3. Consulta o progresso quando necessário;
4. Coleta os resultados concluídos;
5. Integra as descobertas ao resultado final.

O agente deixa de ser algo que você observa enquanto processa uma tarefa e passa a trabalhar ao seu lado em diferentes frentes simultaneamente.

---

## Consideração final

Agentes paralelos trabalham principalmente com texto.

A qualidade do resultado depende diretamente da qualidade do contexto fornecido a cada subagente.

Uma boa delegação deve possuir:

- Objetivo autossuficiente;
- Contexto completo;
- Escopo bem delimitado;
- Ferramentas mínimas necessárias;
- Critérios claros de conclusão;
- Formato de saída definido;
- Orçamento proporcional à complexidade;
- Profundidade de orquestração controlada.

Delegar não significa apenas dividir uma tarefa. Significa criar unidades independentes de trabalho, cada uma com contexto, ferramentas, orçamento e responsabilidades claramente definidos.