# Parte 01: Como o agente Hermes realmente processa o trabalho

Quase todo chatbot de IA funciona do mesmo jeito: você manda uma mensagem, o modelo responde, fim.

Com o Hermes, a história é outra.

Quando você pede alguma coisa, ele não sai cuspindo texto imediatamente. Antes disso, ele monta contexto, escolhe provedor, decide se precisa compactar histórico, chama ferramentas, avalia o que voltou e só então responde. Em muitos casos, uma única tarefa passa por várias rodadas desse ciclo.

É isso que faz o Hermes parecer menos um chat e mais um agente de verdade.

## Chatbots têm uma arquitetura simples

No modelo mais básico, o fluxo é assim:

```text
mensagem do usuário -> modelo prevê a próxima sequência -> resposta
```

Isso resolve conversa. Não resolve execução.

O modelo não verifica se um arquivo existe. Não abre um terminal. Não consulta uma API. Não volta para conferir se a primeira tentativa deu errado. Ele só prevê texto.

Por isso tanta ferramenta de IA parece impressionante nos primeiros cinco minutos e limitada no uso real.

## O Hermes tem um loop

O Hermes funciona em ciclos.

Cada turno passa, mais ou menos, por esta sequência:

1. montar o prompt com instruções, memória e contexto atual;
2. resolver qual modelo e qual provedor serão usados;
3. comprimir histórico, se necessário;
4. chamar o modelo;
5. interpretar a resposta;
6. executar ferramentas, quando o modelo pedir;
7. alimentar o resultado de volta no loop;
8. repetir até chegar a uma resposta final.

Na prática, isso significa que uma tarefa pode envolver várias idas e voltas entre raciocínio e execução. É essa alternância que dá ao Hermes uma sensação de continuidade e trabalho real.

## O que acontece em cada mensagem

### 1. Montagem de prompt

Antes de responder, o Hermes junta as peças que precisa para pensar direito.

Esse pacote costuma incluir:

- instruções de sistema;
- memória permanente;
- perfil do usuário;
- mensagens recentes da sessão;
- índice de skills;
- contexto do projeto, quando existe;
- resultados de ferramentas já usados no turno.

Essa etapa importa mais do que parece. Um agente bom não depende só do modelo; depende de como o contexto é montado.

### 2. Resolução de provedor

O Hermes também precisa decidir quem vai executar o turno.

Dependendo da configuração, isso pode envolver:

- escolher o modelo principal;
- aplicar fallback para outro provedor;
- usar um modelo diferente em tarefas auxiliares;
- respeitar regras do perfil ativo.

Isso separa o “pensar” da infraestrutura de inferência. Você consegue trocar o motor sem reescrever a máquina inteira.

### 3. Compressão prévia

Se o histórico crescer demais, o Hermes não segue empilhando mensagem até quebrar.

Ele compacta o que ficou para trás, preserva o que ainda importa e continua a sessão em uma nova linhagem. Não é perfeito, porque resumo nunca substitui o original, mas é muito melhor do que simplesmente explodir a janela de contexto.

Em uso real, isso faz diferença. Sessões longas acontecem. Sem compressão, elas morrem cedo.

### 4. Chamada de API

Com o prompt pronto, o Hermes chama o modelo.

Até aqui, ele ainda está no território de qualquer app de chat. A diferença aparece no momento em que a resposta do modelo não é texto final, mas um pedido de ação.

### 5. Análise da resposta

Quando o modelo responde, o Hermes decide: isso já é a resposta final ou é uma instrução para continuar trabalhando?

Se vier uma chamada de ferramenta, o agente executa. Se vier um lote de chamadas independentes, ele pode paralelizar. Se vier texto final, encerra o turno.

Essa análise é o que transforma o modelo em operador.

## A diferença fundamental: chamadas de ferramentas

É aqui que o Hermes muda de categoria.

Quando o modelo pede uma ferramenta, o sistema sai da camada de linguagem e entra na camada de ação. Isso pode significar:

- rodar um comando no terminal;
- ler ou editar arquivos;
- pesquisar na web;
- usar navegador;
- controlar desktop;
- criar cron jobs;
- chamar subagentes;
- consultar sessões antigas;
- gravar memória.

Um chatbot comum descreve o que faria.
O Hermes faz, verifica e volta com o resultado.

## Execução simultânea de ferramentas

Outro detalhe importante: o Hermes não precisa tratar todo trabalho como uma fila única.

Se duas leituras são independentes, ele pode executá-las em paralelo. Isso reduz latência e evita desperdício de contexto. Em tarefas maiores, essa diferença acumula rápido.

Na prática, isso significa menos tempo esperando e menos voltas desnecessárias no loop.

## Por que o Hermes melhora com o tempo

O Hermes funciona em camadas diferentes de estabilidade.

### Estável

Aqui fica o que muda pouco e vale a pena carregar sempre:

- memória durável;
- perfil do usuário;
- skills;
- convenções do ambiente.

### Contexto

Aqui fica o que pertence à sessão atual:

- mensagens recentes;
- arquivos lidos;
- resultados de ferramentas;
- decisões tomadas ao longo do turno.

### Volátil

Aqui fica o ruído operacional:

- logs temporários;
- comandos de verificação;
- tentativas descartadas;
- saídas intermediárias que não merecem ser persistidas.

Essa separação evita dois extremos ruins: esquecer tudo ou tentar guardar tudo.

## Por que essa separação é intencional

Se tudo fosse permanente, o agente viraria uma bagunça muito rápido.
Se nada fosse permanente, cada sessão começaria do zero.

O desenho do Hermes tenta segurar o meio-termo útil: lembrar o que realmente se acumula e deixar passar o que só serviu para aquele momento.

É por isso que o sistema de memória, skills e busca de sessões existe como peças separadas. Cada uma guarda um tipo de valor diferente.

## Cinco princípios de design

### 1. Estabilidade de prompt

Quanto mais estável o prefixo do prompt, melhor o cache funciona e menor o custo recorrente. Isso parece detalhe de implementação, mas impacta desempenho e preço.

### 2. Execução observável

O Hermes não deveria agir no escuro. Sempre que possível, ele executa, lê o retorno e decide com base no que realmente aconteceu.

### 3. Interruptível

Um agente útil não pode ser uma caixa-preta impossível de interromper. O loop precisa tolerar pausas, retomadas e revisões.

### 4. Núcleo independente de plataforma

CLI, desktop, gateway e automação não são produtos separados. São superfícies diferentes sobre o mesmo motor.

### 5. Acoplamento fraco

Modelo, provedor, ferramentas, perfil e interface podem mudar sem derrubar a arquitetura inteira. Isso dá longevidade ao sistema.

# O que isso significa para o uso do Hermes

Na prática, entender o loop muda a forma como você usa o agente.

Você para de pensar em “uma pergunta, uma resposta” e começa a pensar em “um objetivo, várias etapas, com verificação no meio”.

## Habilidades ficam no nível estável

Skills fazem sentido porque o Hermes tem uma camada estável onde procedimentos podem viver. Quando uma tarefa se repete, o melhor caminho não é reexplicar tudo. É formalizar o processo.

## Arquivos de contexto seguem prioridade

Nem todo arquivo merece entrar no prompt. O Hermes precisa escolher o que vale carregar agora e o que pode ser buscado depois. Esse filtro é parte da qualidade do trabalho.

## Chamadas independentes podem ser paralelas

Sempre que duas leituras ou consultas não dependem uma da outra, o Hermes pode ganhar tempo executando em lote. Isso vale especialmente para inspeção de repositório, leitura de arquivos e pesquisas auxiliares.

## Existe um orçamento de iterações

O loop não é infinito. Cada chamada de ferramenta consome orçamento. Isso protege contra espirais inúteis, runaway loops e tarefas que queimam contexto sem sair do lugar.

# A arquitetura que torna tudo possível

No fim das contas, o valor do Hermes não está em “responder bonito”. Está em conseguir atravessar o ciclo inteiro:

- entender o pedido;
- montar contexto útil;
- agir com ferramentas;
- verificar o resultado;
- persistir o que importa;
- deixar a sessão pronta para continuar depois.

É esse loop que sustenta tudo o que vem a seguir na masterclass.

Sem ele, o Hermes seria só mais uma interface para um LLM.
Com ele, vira uma base operacional para trabalho assistido por agentes.
