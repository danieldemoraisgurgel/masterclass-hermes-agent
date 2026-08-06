# Parte 05: Ferramentas e conjuntos de ferramentas

Fazer uma pergunta ao Hermes e receber uma resposta coerente é o teste mais fácil do mundo.

Também é o mais enganoso.

Esse teste prova que o chat funciona. Não prova que o agente trabalha.

A diferença prática entre o Hermes e um chatbot comum está na superfície de ferramentas. É ela que permite sair da conversa e entrar em execução.

## Registro central de ferramentas

No Hermes, ferramentas não ficam espalhadas como improviso. Elas passam por um registro central, com nome, descrição, parâmetros, regras de disponibilidade e, quando necessário, verificação prévia.

Isso parece detalhe interno, mas é o que dá consistência ao agente. Ferramenta boa não é só algo que existe. É algo que pode ser descoberto, chamado, validado e controlado.

## Categorias de ferramentas

A biblioteca é ampla, mas ela fica mais fácil de entender quando você pensa em categorias.

### Web

Ferramentas de busca, leitura de páginas e coleta de informação atual.

São o antídoto contra um problema clássico de LLM: responder sobre o presente usando só o passado do treino.

### Terminal e arquivos

Aqui mora uma parte grande do poder real do Hermes.

Essas ferramentas permitem:

- executar comandos;
- ler arquivos;
- editar conteúdo;
- rodar testes;
- inspecionar repositórios;
- iniciar processos longos.

Sem isso, o agente continua preso na descrição. Com isso, ele passa a tocar o sistema.

### Navegador

Automação de navegador entra quando a informação ou a ação está numa interface web real. Isso inclui navegação, clique, preenchimento e captura de estado.

### Mídia

Imagem, voz, OCR e outros fluxos multimodais entram aqui. Não são o centro de toda operação, mas ampliam bastante o alcance do agente.

### Orquestração de agentes

Subagentes, delegação e tarefas paralelas vivem nessa categoria. É o que permite quebrar um trabalho grande em unidades menores sem entupir o contexto principal.

### Memória e recuperação

Ferramentas para salvar fatos duráveis, consultar sessões anteriores e recuperar contexto histórico quando isso faz sentido.

### Automação

Cron, rotinas recorrentes e outros mecanismos que fazem o Hermes trabalhar sem depender de uma nova mensagem sua.

### Integrações

Gateways, APIs e serviços externos entram aqui. São as pontes entre o agente e o resto do ambiente.

# Conjuntos de ferramentas controlam o alcance do agente

Nem toda sessão precisa de tudo.

Por isso, o Hermes trabalha com conjuntos de ferramentas. Eles servem para limitar o escopo operacional do agente de acordo com a tarefa.

Isso traz alguns ganhos claros:

- menos ruído no contexto;
- menor risco operacional;
- menor custo de prompt;
- comportamento mais previsível.

Um agente restrito às ferramentas certas costuma trabalhar melhor do que um agente com acesso irrestrito a tudo.

## Configuração dos conjuntos de ferramentas

Na prática, isso significa que você pode montar perfis ou execuções especializadas. Um fluxo de pesquisa pode carregar web e arquivos. Um fluxo de desenvolvimento pode precisar de terminal, arquivos e GitHub. Um cron simples talvez precise de quase nada além do essencial.

Esse controle fino faz o Hermes escalar melhor, porque evita o padrão “um agente genérico tentando fazer tudo do mesmo jeito”.

## Verificação de disponibilidade com `check_fn`

Nem toda ferramenta está sempre pronta para uso.

Algumas dependem de binários instalados, variáveis de ambiente, permissões ou conectividade. A checagem de disponibilidade existe para evitar falsas promessas. Melhor descobrir cedo que uma ferramenta não está pronta do que deixar o agente descobrir isso no pior momento possível.

# Ambientes de execução do terminal

O terminal é uma das peças mais poderosas do Hermes, mas o comportamento dele muda de acordo com o backend.

## Local

No local, o agente enxerga a mesma máquina em que você está trabalhando. Isso é ótimo para tarefas rápidas, inspeção do sistema e iteração direta.

## Docker

No Docker, o terminal vive dentro do contêiner. Isso é mais previsível, mas também significa que o ambiente disponível é o do contêiner, não o da máquina host. Se algo precisa existir ali, você precisa colocar ali.

## SSH

Com SSH, o Hermes executa em outra máquina. É uma forma elegante de levar o agente até onde os recursos estão.

## Singularity

Em ambientes científicos ou institucionais, Singularity pode ser o caminho natural para ganhar isolamento sem depender do ecossistema Docker.

## Modal e Daytona

Aqui o terminal pode existir em workspaces efêmeros ou sob demanda. Funciona muito bem para certos fluxos, mas exige que você pense melhor em persistência e disponibilidade.

# Persistência do backend Docker

Esse ponto merece reforço: contêiner descartável com volume persistente é ótimo. Contêiner descartável sem volume persistente é uma armadilha.

Quando o volume está certo, o Hermes mantém estado.
Quando não está, parece que o sistema sofre amnésia a cada reinício.

# Gerenciamento de processos em segundo plano

Outra capacidade importante é iniciar processos longos e continuar trabalhando enquanto eles rodam.

Isso vale para:

- servidores locais;
- builds demorados;
- testes longos;
- watchers;
- pipelines auxiliares.

O ganho aqui não é só conforto. É continuidade operacional. O agente não precisa ficar travado esperando tudo acabar para só então seguir.

# Comandos perigosos exigem aprovação

Poder de execução sem controle vira risco rápido.

Por isso, comandos destrutivos ou potencialmente perigosos passam por aprovação. Essa camada não existe para atrapalhar o fluxo. Ela existe para impedir que o agente aja além do que deveria.

É uma das diferenças entre “automação útil” e “automação irresponsável”.

# Por que o acesso a ferramentas muda tudo

Sem ferramentas, o agente interpreta o mundo por descrição.
Com ferramentas, ele interage com o mundo por observação e ação.

Isso muda tudo porque troca plausibilidade por verificação.

Um exemplo simples:

- sem terminal, o modelo diz como checaria a versão do Python;
- com terminal, ele checa de fato;
- com arquivo, ele pode registrar resultado;
- com processo em segundo plano, ele ainda acompanha o que ficou rodando.

É isso que faz o Hermes ser operacional, e não apenas eloquente.

# Teste de fumaça essencial

Se você quer saber se a superfície principal está viva, faça um teste curto e honesto:

1. rode um comando simples no terminal;
2. leia ou escreva um arquivo;
3. faça uma consulta externa quando aplicável.

Se essas peças funcionam, o Hermes já passou do nível “conversa convincente” e entrou no nível “agente que consegue trabalhar”.

E é aí que a ferramenta começa a valer mesmo.