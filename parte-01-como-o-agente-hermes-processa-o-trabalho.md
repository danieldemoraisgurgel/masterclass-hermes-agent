# Parte 01: Como o Agente Hermes Realmente Processa o Trabalho

Todo chatbot de IA que você já usou tem a mesma arquitetura: você digita uma mensagem, o modelo gera uma resposta.

Uma etapa. Pronto.

O Hermes é diferente de uma forma que não fica óbvia nas capturas de tela.

Quando você envia uma mensagem, ele passa por um processo estruturado — montagem de prompt, resolução de provedor, chamada de API, despacho de ferramentas, avaliação de resultado e persistência de contexto — antes de responder.

Algumas mensagens acionam dez chamadas de ferramentas, cada uma alimentando o modelo para outra rodada de raciocínio. Então a sessão é salva, a memória é gravada, e tudo fica pronto para ser retomado mais tarde.

Isso é um **loop de agente**.

Entender como ele funciona é a chave para compreender por que essa ferramenta se acumula de formas que chatbots básicos não conseguem.

Veja o que acontece nos bastidores toda vez que você envia uma mensagem.

## Chatbots têm uma arquitetura simples

```text
mensagem do usuário -> modelo prevê -> resposta
```

Cada entrada é uma nova inferência sobre dados estáticos de treinamento.

O modelo não executa nada. Ele não verifica nada. Ele faz suposições com base no que aprendeu durante o treinamento.

## O Hermes tem um loop

A classe `AIAgent`, localizada em `run_agent.py`, trata todo o ciclo de vida de um único turno:

- montagem de prompt;
- seleção de provedor;
- chamada de API;
- despacho de ferramentas;
- compressão;
- fallback;
- persistência.

Ele oferece suporte a três modos de execução de API:

1. conclusões de chat da OpenAI;
2. OpenAI Codex/Responses;
3. Messages nativo da Anthropic.

Todos eles convergem para o mesmo formato interno de mensagem.

## O que acontece em cada mensagem

### 1. Montagem de prompt

O sistema constrói o contexto a partir de mais de dez camadas:

- `SOUL.md`, para identidade;
- habilidades, para conhecimento procedimental;
- instantâneos de memória;
- perfil do usuário;
- arquivos de contexto do diretório do projeto;
- dicas da plataforma de onde você está conversando.

Tudo é montado em três níveis ordenados:

- **Estável:** identidade, ferramentas e habilidades;
- **Contexto:** arquivos do projeto;
- **Volátil:** memória, perfil e carimbo de data e hora.

### 2. Resolução de provedor

O Hermes mapeia a seleção de provedor e modelo para:

- endpoint de API;
- chave de API;
- modo correto de execução.

Ele trata mais de 18 provedores, fluxos de OAuth e pools de credenciais.

### 3. Compressão prévia

Se a conversa ultrapassa 50% da janela de contexto do modelo, o Hermes comprime o histórico antes de fazer a chamada de API.

Nesse processo:

- os turnos intermediários são resumidos;
- as últimas 20 mensagens são preservadas intactas;
- um novo ID de linhagem da sessão é gerado.

### 4. Chamada de API

O contexto montado é enviado ao modelo.

A solicitação HTTP é executada em uma thread em segundo plano, com um evento de interrupção monitorando sua execução.

O processamento pode ser cancelado por:

- um sinal;
- o comando `/stop`;
- o envio de uma nova mensagem.

Se o modelo falhar com um erro `429` ou `5xx`, o Hermes verifica a lista de provedores de fallback e tenta o próximo.

### 5. Análise da resposta

Se o modelo retorna texto, essa é a resposta final, que é persistida no armazenamento da sessão.

Se o modelo retorna chamadas de ferramentas, o loop continua.

## A diferença fundamental: chamadas de ferramentas

Quando um modelo de linguagem determina que precisa fazer algo — executar um comando, pesquisar na web, escrever um arquivo ou ler um documento — ele retorna uma `tool_call` em vez de texto.

O Hermes captura essa chamada e a encaminha por meio de um registro central localizado em:

```text
tools/registry.py
```

O registro possui mais de 70 ferramentas distribuídas em aproximadamente 28 conjuntos de ferramentas.

Cada arquivo de ferramenta chama `registry.register()` no momento da importação, informando:

- nome;
- esquema;
- função manipuladora;
- verificação de disponibilidade;
- metadados.

Quando uma `tool_call` chega:

- ferramentas de nível de agente, como memória, tarefas, busca de sessão e delegação, são interceptadas pelo próprio loop do agente, pois precisam de acesso direto ao estado;
- todo o restante passa por `registry.dispatch()`;
- o registro procura o manipulador;
- a disponibilidade é verificada por meio da `check_fn`;
- a ferramenta é executada;
- o resultado é retornado como uma string JSON.

Os erros são encapsulados em dois níveis:

1. `registry.dispatch()` captura exceções do manipulador;
2. `handle_function_call()` captura exceções de despacho.

O modelo sempre recebe um resultado bem-formado, nunca um erro não tratado.

## Execução simultânea de ferramentas

Múltiplas chamadas de ferramentas retornadas por uma única resposta do modelo são executadas simultaneamente por meio de um executor com pool de threads.

A exceção são as ferramentas marcadas como interativas, como `clarify`, que forçam execução sequencial.

Assim que todos os resultados retornam, eles são acrescentados ao histórico da conversa como mensagens de função de ferramenta.

Em seguida, o loop volta para a etapa de chamada de API com o novo contexto.

Esse processo continua até que:

- o modelo retorne uma resposta de texto; ou
- o orçamento de iterações seja atingido.

## Por que o Hermes melhora com o tempo

O motivo não é magia. É estrutural.

O prompt de sistema é construído em três níveis ordenados.

### Estável

Inclui:

- `SOUL.md`, que define a identidade do agente;
- orientação sobre ferramentas;
- habilidades;
- dicas de ambiente;
- dicas de plataforma.

Essa camada não muda no meio da conversa.

### Contexto

Inclui um dos arquivos de contexto do projeto:

- `.hermes.md`;
- `AGENTS.md`;
- `CLAUDE.md`.

Também pode incluir qualquer mensagem de sistema fornecida pelo chamador.

Apenas um tipo de contexto de projeto é carregado, seguindo uma ordem de prioridade a partir do diretório de trabalho.

### Volátil

Inclui:

- instantâneo de memória;
- instantâneo do perfil do usuário;
- bloco do provedor de memória externo;
- linha atual com carimbo de data e hora, sessão e modelo.

Esses dados são atualizados entre sessões, mas permanecem congelados durante uma única conversa.

A memória pode ser gravada em disco no meio da sessão, mas não altera imediatamente o prompt de sistema armazenado em cache.

A atualização acontece quando ocorre um caminho de reconstrução, como:

- nova sessão;
- compressão;
- invalidação explícita.

Isso mantém o prefixo do prompt estável e favorece o cache do lado do provedor.

## Por que essa separação é intencional

Essa arquitetura produz três efeitos importantes:

1. a primeira parte do prompt — identidade, ferramentas e habilidades — pode aproveitar o cache de prompt no nível da API;
2. alterações de memória não invalidam o cache no meio da conversa;
3. habilidades, arquivos de contexto e dicas de plataforma possuem posição e precedência definidas.

## Cinco princípios de design

A documentação destaca cinco princípios que explicam o comportamento do Hermes.

### 1. Estabilidade de prompt

O prompt de sistema não muda no meio da conversa.

Não há mutações que quebrem o cache, a menos que o modelo seja alterado explicitamente com:

```text
/model
```

### 2. Execução observável

Toda chamada de ferramenta é visível:

- indicador giratório na CLI;
- mensagens de progresso no Telegram;
- atualizações de callback no Discord.

É possível acompanhar o que o agente está fazendo durante a execução.

### 3. Interruptível

Chamadas de API e execução de ferramentas podem ser canceladas no meio do processamento.

Não se trata de forçar o encerramento do processo, mas de abandonar a solicitação anterior de forma limpa quando uma nova mensagem é enviada.

### 4. Núcleo independente de plataforma

Uma única classe `AIAgent` atende:

- CLI;
- gateways de mensagens;
- integração do editor ACP;
- processamento em lote;
- servidor de API.

As diferenças entre plataformas ficam nos pontos de entrada, não no agente.

### 5. Acoplamento fraco

Servidores MCP, plugins, provedores de memória e ambientes de RL utilizam padrões de registro e bloqueio por `check_fn`.

Subsistemas opcionais não criam dependências rígidas.

Se um plugin falhar ao carregar, o restante do agente continua funcionando.

# O que isso significa para o uso do Hermes

Entender o loop do agente muda a forma como você trabalha com o Hermes.

## Habilidades ficam no nível estável

As habilidades são carregadas no nível estável do prompt.

Elas permanecem disponíveis durante a sessão, mas não mudam no meio da conversa.

Para fazer o agente adotar uma nova especialidade, adicione ou substitua habilidades entre sessões, não durante uma sessão já iniciada.

## Arquivos de contexto seguem prioridade

Os arquivos de contexto são carregados a partir do diretório de trabalho seguindo esta prioridade:

1. `.hermes.md`;
2. `AGENTS.md`;
3. `CLAUDE.md`.

Um `.hermes.md` na raiz do projeto tem precedência sobre um `AGENTS.md` no mesmo diretório.

O `CLAUDE.md` só é carregado quando não existe `.hermes.md` nem `AGENTS.md`.

Estruture o contexto do projeto considerando essa ordem.

## Chamadas independentes podem ser paralelas

As chamadas de ferramentas são simultâneas por padrão.

Quando você pede quatro tarefas independentes, o Hermes pode executá-las em paralelo, e não necessariamente uma depois da outra.

Elabore os prompts para aproveitar essa capacidade.

## Existe um orçamento de iterações

O orçamento padrão é de 150 turnos, máximo 500. 

Em alguns casos, o construtor interno AIAgent() e alguns caminhos de inicialização podem referenciar 90 como valor padrão, quando nenhum valor era fornecido explicitamente - tudo depende de como você inicia o seu agente.

Cada chamada de ferramenta conta como um turno.

Uma tarefa complexa que exige 15 chamadas de ferramentas consome 15 dos 150 turnos disponíveis.

Os subagentes recebem orçamentos próprios e independentes, normalmente limitados a 50 turnos (Max tool-calling).

Em fluxos de trabalho longos, considere o orçamento necessário para chamadas de ferramentas, e não apenas para mensagens de conversa.

# A arquitetura que torna tudo possível

O loop do agente é a arquitetura que viabiliza todo o restante.

As habilidades se acumulam porque são carregadas como contexto estável que o modelo mantém disponível.

A memória se acumula porque é gravada em disco durante a sessão, mas registrada como instantâneo nos limites de sessão.

O sistema de ferramentas cresce porque novas ferramentas se autorregistram durante a importação, sem exigir conexão manual no núcleo do agente.

> Chatbots preveem o próximo token.  
> O Hermes executa um processo.

Essa é a diferença que se acumula.

O loop só funciona se conseguir alcançar suas ferramentas e persistir suas sessões.

Este artigo é a primeira parte da Master Class do Agente Hermes. Ele foi escrito partindo da suposição de que o Hermes já está instalado.

Se você ainda não tem o Hermes instalado, comece pelo guia de instalação.

Novos usuários também podem consultar o material introdutório enquanto aguardam a Parte 2 da Master Class.
