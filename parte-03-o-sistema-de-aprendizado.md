# Parte 03: O sistema de aprendizado

Depois de entender o loop do agente e a base de implantação, vem a pergunta mais importante de todas:

isso melhora com o tempo ou não?

Na maioria das ferramentas de IA, a resposta honesta é “quase não”. O modelo até pode receber uma atualização do fornecedor, mas a ferramenta em si não vai aprendendo com você.

No Hermes, a proposta é diferente.

Ele não treina o modelo com seus dados. O que ele faz é construir um sistema em volta do modelo para acumular contexto útil sobre:

- você;
- seus projetos;
- seu ambiente;
- seus fluxos de trabalho.

É isso que transforma uso repetido em vantagem real.

## Memória: um conjunto selecionado de fatos

A memória do Hermes não é um diário completo da sua vida nem um dump da sessão anterior.

Ela funciona mais como um bloco pequeno de fatos que vale a pena manter sempre por perto. Coisas do tipo:

- preferências de resposta;
- convenções do projeto;
- detalhes do ambiente;
- correções recorrentes feitas por você.

Em geral, o sistema separa isso em dois espaços:

- `MEMORY.md`, com notas do ambiente e do trabalho;
- `USER.md`, com informações duráveis sobre você e seu jeito de trabalhar.

A limitação de tamanho não é um defeito. É uma proteção. Memória boa é memória selecionada.

### Como a memória se comporta

Quando o Hermes aprende algo que tende a continuar valendo, ele pode salvar aquilo.

Por exemplo:

- você prefere respostas curtas e diretas;
- determinado projeto usa uma convenção específica;
- um workflow sempre falha por um detalhe conhecido;
- certo diretório é o lugar certo para guardar saídas temporárias.

Esse bloco entra no contexto logo no começo de novas sessões. É isso que evita ter de reexplicar as mesmas coisas toda hora.

## Persistência e atualização da memória

Uma nuance importante: a memória pode ser atualizada no meio da sessão, mas o efeito completo costuma aparecer na próxima. Isso mantém o contexto mais estável e evita bagunçar o prefixo do prompt a cada pequena mudança.

Quando a memória enche, o Hermes não deveria sair apagando fatos aleatoriamente. O comportamento esperado é consolidar:

1. encurtar entradas redundantes;
2. remover o que ficou obsoleto;
3. abrir espaço para o que ainda importa.

Isso é importante porque memória custa tokens em todo turno. Se você guardar qualquer coisa, a conta cresce e a qualidade cai.

## Memória versus busca de sessões

Nem tudo que o agente precisa lembrar merece memória permanente.

Sessões antigas existem justamente para isso.

Quando a pergunta é algo como:

> o que decidimos sobre aquele projeto semana passada?

não faz sentido carregar essa informação em todo prompt para sempre. Faz mais sentido buscar quando necessário.

Daí a divisão útil:

- memória guarda fatos duráveis;
- sessões guardam histórico recuperável.

Essa distinção parece técnica, mas muda bastante a qualidade do sistema.

## Habilidades: a memória procedimental do agente

Se memória guarda fatos, skills guardam procedimento.

Quando o Hermes resolve uma tarefa mais complexa e percebe que aquilo pode voltar, ele pode transformar a abordagem em uma skill. Normalmente isso faz sentido quando houve:

- várias chamadas de ferramenta;
- tentativa e erro real;
- uma correção importante do usuário;
- um fluxo que claramente vai se repetir.

A skill vira um arquivo Markdown com instruções executáveis. Não é só documentação bonita. É um jeito de formalizar trabalho que já provou funcionar.

## Divulgação progressiva das habilidades

Aqui existe uma sacada boa de arquitetura.

O Hermes não carrega todas as skills completas em toda sessão. Primeiro ele carrega só um índice enxuto com nome, descrição e categoria. O conteúdo completo entra apenas quando necessário.

Isso reduz custo e evita encher o contexto com procedimento que provavelmente não será usado.

## Combinação de habilidades

Outra vantagem é que skills podem ser compostas.

Você pode combinar, por exemplo, uma skill de fluxo GitHub com outra de testes. O agente passa a trabalhar com as duas lentes ao mesmo tempo, o que é muito mais útil do que tentar enfiar tudo numa única skill gigante e genérica.

## Origem das habilidades

As skills costumam vir de três lugares.

### 1. Habilidades incluídas

São as que já acompanham o Hermes. Elas cobrem operações comuns e ajudam bastante no começo.

### 2. Habilidades do Hub

São contribuições externas instaláveis. Entram bem quando você quer ampliar a biblioteca sem partir do zero.

### 3. Habilidades criadas pelo agente

Essas são as mais interessantes no longo prazo, porque capturam o seu jeito de trabalhar. É aqui que o Hermes começa a ficar menos genérico e mais seu.

# A revisão em segundo plano é uma infraestrutura invisível

O aprendizado do Hermes não depende só do que acontece na conversa principal.

Depois de um turno, ele pode fazer uma revisão paralela do que aconteceu para identificar se existe algo que vale transformar em memória ou skill.

Essa revisão é valiosa porque separa duas tarefas diferentes:

- resolver o problema agora;
- decidir o que merece ser preservado depois.

Misturar as duas coisas no mesmo fluxo deixaria a conversa mais lenta e mais barulhenta.

## Aprovação das gravações

Em geral, o que faz mais sentido guardar sem pedir permissão explícita são fatos pequenos e duráveis, como preferência de linguagem ou detalhe estável do ambiente.

Já mudanças mais estruturais, como criar uma nova skill, costumam merecer confirmação. Isso evita transformar qualquer sucesso ocasional em regra permanente.

## Uso de um modelo mais barato para revisão

Essa parte também abre espaço para otimização.

Nem toda tarefa de revisão precisa usar o modelo mais caro. Muitas vezes um modelo menor já dá conta de inspecionar o turno e sugerir o que vale preservar. Isso reduz custo sem derrubar valor.

# O curador impede a degradação das habilidades

Criar skill é só metade do problema. A outra metade é impedir que a biblioteca apodreça.

Sem manutenção, skills viram uma coleção de instruções antigas, redundantes ou meio certas. E uma skill meio certa é pior do que nenhuma, porque dá falsa confiança.

## Ciclo de vida das habilidades

O curador existe para revisar, consolidar e podar a biblioteca.

### Habilidade não utilizada por 30 dias

Esse costuma ser um bom momento para perguntar se ela ainda faz sentido ou se foi só um workaround pontual.

### Habilidade não utilizada por 90 dias

Aqui já vale considerar arquivar, fundir com outra ou remover, se não houver motivo real para mantê-la viva.

## Consolidação opcional com LLM

Quando várias skills começam a se sobrepor, um processo de consolidação ajuda a reduzir ruído. O objetivo não é deixar tudo uniforme por estética. É facilitar descoberta e evitar duplicação desnecessária.

## Proteção de habilidades críticas

Nem tudo deve ficar sujeito a poda automática. Algumas skills viram infraestrutura do seu jeito de operar e precisam de proteção extra.

## Instantâneos e rollback

Se você vai deixar o sistema mexer em skills, snapshot e rollback deixam de ser luxo e viram requisito. Sem isso, uma "limpeza" ruim pode custar conhecimento útil.

# Como o ciclo de aprendizado funciona

Se você juntar todas as peças, o ciclo fica assim:

1. você trabalha com o agente;
2. o Hermes usa ferramentas e resolve a tarefa;
3. a sessão é salva;
4. uma revisão identifica o que foi aprendido;
5. fatos duráveis podem virar memória;
6. procedimentos reutilizáveis podem virar skill;
7. sessões futuras passam a começar mais fortes.

É esse acúmulo que faz o sistema melhorar no uso real.

# Por que a persistência das sessões importa

Sem sessão persistente, o aprendizado perde profundidade.

Memória sozinha não resolve tudo. Skill sozinha também não. O histórico de sessões é o que permite ao agente revisitar decisões, recuperar contexto e conectar trabalhos espalhados no tempo.

Quando essa base falha, o Hermes até continua conversando, mas deixa de acumular.

# O sistema de aprendizado é o produto

Esse ponto é central.

Muita gente olha para o Hermes e enxerga modelo, terminal ou automação. Tudo isso importa, mas o que realmente diferencia o sistema é a capacidade de aprender estrutura com o uso.

Memória captura o que vale carregar.
Skills capturam o que vale repetir.
Sessões capturam o caminho que levou até ali.

Quando essas três camadas funcionam juntas, o Hermes deixa de ser só uma interface esperta e vira um sistema que melhora no seu contexto.