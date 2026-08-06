# Parte 08: Delegação e subagentes

Uma sessão única de agente é, por natureza, serial.

Mensagem entra. Modelo pensa. Ferramenta roda. Resultado volta. Nova decisão acontece.

Isso funciona bem por bastante tempo. Mas chega uma hora em que vira gargalo.

Pesquisa grande, revisão ampla, refatoração distribuída, coleta em múltiplas frentes: tudo isso fica melhor quando o trabalho pode ser dividido.

É aí que entram subagentes.

## Como a delegação funciona

Quando o Hermes delega, ele cria agentes filhos com contexto próprio, ferramentas próprias e objetivo bem delimitado. Cada um trabalha isoladamente e devolve apenas um resumo final ao agente principal.

Essa separação é importante por dois motivos:

- evita poluir o contexto do agente pai com ruído intermediário;
- permite trabalho paralelo de verdade.

## Quando delegar

Delegação faz mais sentido quando a tarefa é pesada em raciocínio, grande em escopo ou naturalmente paralelizável.

Alguns sinais clássicos:

- há várias frentes independentes de investigação;
- a tarefa geraria muito contexto intermediário;
- diferentes arquivos ou áreas podem ser analisados em separado;
- você quer acelerar uma revisão ampla.

Se for só uma sequência mecânica de comandos, delegar costuma ser exagero.

### Quando usar `execute_code`

Se o problema for repetitivo, com loop, filtro ou processamento estruturado sobre muitas saídas, `execute_code` costuma ser melhor. Ele é ótimo para lógica programática. Delegação é melhor para subtarefas que ainda exigem julgamento.

## Padrões comuns de delegação

### Pesquisa paralela

Você divide fontes, hipóteses ou tópicos entre filhos. Cada um investiga uma frente e devolve o essencial.

### Revisão de código e correção

Um subagente pode focar em testes, outro em segurança, outro em arquitetura. Em vez de um agente tentando pensar em tudo ao mesmo tempo, você distribui lentes diferentes.

### Refatoração de vários arquivos

Quando diferentes partes do código podem ser tratadas em paralelo, delegar reduz tempo total e organiza melhor o raciocínio.

## Seleção do conjunto de ferramentas

Subagente bom não recebe tudo. Recebe o mínimo necessário.

### Subagentes de pesquisa

Normalmente precisam de web, leitura de arquivos e talvez algum suporte de síntese.

### Subagentes de código

Costumam precisar de terminal, arquivos, testes e inspeção do repositório.

### Tarefas full-stack

Podem exigir mistura de web, terminal, arquivos e talvez navegador, mas ainda assim vale restringir ao necessário.

### Ferramentas bloqueadas

Algumas ferramentas simplesmente não devem ficar na mão de filhos.

#### `clarify`

Subagente não deveria depender de diálogo com o usuário para funcionar. O objetivo precisa vir completo.

#### `memory`

Persistência durável é responsabilidade do fluxo principal, não de um filho isolado.

#### `delegation`

Sem controle de profundidade, delegação vira uma árvore que cresce sem necessidade.

## O modelo assíncrono

Um detalhe importante: subagentes trabalham em segundo plano. O pai não precisa travar esperando cada movimento. O resumo volta quando estiver pronto.

Isso muda o jeito de usar o sistema. Você passa a orquestrar trabalho, não só a executar uma trilha única de pensamento.

## Visualização dos agentes

Ter visibilidade de quem está fazendo o quê ajuda muito. Em tarefas maiores, isso deixa de ser curiosidade e vira gestão.

## Orçamento de iteração e timeout

Delegação também precisa de limites.

### Timeout

Se um filho não termina, você precisa saber quando encerrar, revisar escopo ou tentar outra abordagem. Paralelismo sem controle também desperdiça tempo.

## Profundidade de orquestração

Nem toda operação precisa de filhos dos filhos dos filhos.

Quanto maior a profundidade, maior o custo de coordenação. Em muitos cenários, um nível bem feito já entrega quase todo o benefício sem trazer caos adicional.

### Orquestração em múltiplos estágios

Quando fizer sentido, dá para pensar em coleta num primeiro estágio e síntese num segundo. Mas isso só vale a pena se cada camada tiver uma função clara.

## Quando o paralelismo muda o fluxo de trabalho

### Sessões com um único agente

Funcionam bem para tarefas diretas e lineares.

### Delegação paralela

Brilha quando há independência real entre as frentes. É nesse ponto que o Hermes começa a parecer menos uma conversa longa e mais uma pequena operação coordenada.

## Delegação assíncrona como colaboração

Na prática, usar subagentes é mais parecido com distribuir trabalho para pessoas competentes do que com rodar um único script gigante.

Você precisa definir:

- objetivo;
- contexto;
- ferramentas;
- critério de conclusão;
- formato de retorno.

Quanto melhor esse briefing, melhor o resultado.

## Consideração final

Delegar não é só “dividir tarefa”.

É desenhar unidades independentes de trabalho com contexto suficiente para agir sem supervisão constante. Quando isso é bem feito, o Hermes ganha velocidade, organização e escala sem entupir a sessão principal.