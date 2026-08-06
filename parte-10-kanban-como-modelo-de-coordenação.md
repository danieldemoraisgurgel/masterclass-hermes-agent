# Parte 10: Kanban como modelo de coordenação

Até aqui, a masterclass mostrou um Hermes que conversa, executa, aprende, agenda e delega.

Mas ainda existe uma limitação implícita nesse modelo: muita coisa continua dependente da conversa atual.

Kanban entra para resolver exatamente isso.

Ele tira o trabalho do fluxo efêmero do chat e coloca esse trabalho num quadro durável, visível e coordenável.

## Workspace Kanban

O quadro funciona como um espaço persistente onde tarefas existem independentemente da sessão aberta no momento.

Isso muda tudo em operações maiores, porque o trabalho deixa de depender da memória humana ou da sorte de alguém lembrar onde a conversa parou.

## Estados das tarefas

Uma tarefa precisa dizer em que ponto está.

Pendente, em andamento, em revisão, concluída: esses estados parecem simples, mas são o mínimo para coordenação não virar caos.

## Responsáveis

Quando mais de um agente ou mais de uma pessoa participa do fluxo, o responsável deixa de ser detalhe. É ele que transforma “alguém deveria fazer isso” em “isso está com alguém”.

## Prioridades e tags

Essas duas camadas ajudam muito na leitura do quadro.

### Prioridade

Prioridade existe para separar o urgente do importante e o importante do que pode esperar.

### Tags

Tags ajudam a agrupar por tipo de trabalho, área, sistema ou contexto. Elas melhoram busca, filtragem e visão operacional.

## Fluxo de trabalho Kanban

A lógica básica costuma seguir um caminho simples.

### 1. Ler

O agente ou operador olha o quadro e entende o que existe, o que bloqueia e o que já está em andamento.

### 2. Reivindicar

Alguém assume a tarefa. Isso evita duplicação e conflito de execução.

### 3. Executar

A tarefa é feita e depois atualizada no quadro com o novo estado, notas ou evidências.

## Escalabilidade para múltiplos perfis

Aqui o Kanban começa a ficar realmente interessante.

Como cada perfil Hermes pode ter função diferente, o quadro vira um ponto comum de coordenação entre especializações. Um perfil pesquisa. Outro executa. Outro revisa. Todos conseguem se alinhar pelo estado compartilhado das tarefas.

# Dependências entre tarefas

Nem todo trabalho pode começar na mesma hora.

Às vezes uma tarefa depende da conclusão de outra. Sem essa noção de dependência, o quadro fica bonito, mas mente sobre a realidade.

## Exemplo de dependência

Uma automação não deveria entrar em produção antes de o ambiente estar configurado.
Um artigo não deveria ser publicado antes da revisão.
Um deploy não deveria acontecer antes dos testes.

Parece óbvio. O valor do Kanban é tornar esse óbvio explícito.

# Prazos

Prazo não é só pressão. É contexto.

Saber quando algo vence ajuda a decidir ordem, urgência e capacidade real do sistema.

# Limites de trabalho em progresso

WIP limit existe para impedir uma ilusão comum: parecer produtivo por começar muitas coisas e terminar poucas.

Isso vale tanto para humanos quanto para agentes. Um quadro saudável não é o que tem mais itens em andamento. É o que consegue mover itens até conclusão com consistência.

# Notas de ciclo de vida

Estados sozinhos não bastam. Em tarefas mais sérias, você também quer saber o que aconteceu na transição.

## Notas da transição para review

Esse é um bom exemplo. Não basta mudar de “doing” para “review”. Vale registrar:

- o que foi feito;
- o que ainda preocupa;
- onde está a evidência;
- o que o revisor precisa observar.

# Coordenação multiagente

Aqui está o grande salto da parte 10.

Kanban não é apenas gestão de tarefas. No contexto do Hermes, ele pode funcionar como camada de coordenação entre agentes independentes.

## Configuração típica

### Perfil orquestrador

Cria, prioriza e distribui trabalho.

### Perfil trabalhador

Executa as tarefas atribuídas.

### Perfil revisor

Valida, comenta e fecha o loop.

## Execução independente

Cada perfil trabalha no seu contexto, com suas credenciais, memória e ferramentas. O quadro é o ponto de sincronização entre eles.

## Coordenação distribuída

Essa separação é valiosa porque evita misturar tudo numa única sessão monstruosa. Em vez disso, cada parte faz o seu pedaço e o quadro guarda o estado compartilhado.

# Quando o Kanban supera o chat direto

Nem toda tarefa precisa de quadro.

## Trabalho de longa duração

Se vai atravessar várias sessões, o chat sozinho já começa a sofrer.

## Trabalho com múltiplas etapas

Quando existe sequência, handoff ou revisão, o quadro ajuda muito.

## Trabalho multiagente

Se mais de um agente participa, Kanban deixa de ser opcional e começa a parecer infraestrutura.

## Trabalho assíncrono

Quando as pessoas ou agentes não estão sincronizados no tempo, o quadro preserva continuidade.

# O quadro como memória do sistema

Memória guarda fatos.
Skills guardam procedimento.
Kanban guarda trabalho em andamento.

Essa distinção é poderosa porque cada camada resolve um problema diferente.

## Kanban, memória e habilidades

Juntos, esses três sistemas formam uma base de operação bem sólida:

- memória diz o que é estável;
- skill diz como executar;
- Kanban diz o que está aberto agora.

# Separação de responsabilidades

Uma das maiores vantagens do quadro é obrigar clareza.

Quem pensa?
Quem faz?
Quem revisa?
O que depende de quê?
O que está parado?

Quando isso fica visível, a operação melhora.

# Exemplo de arquitetura

Imagine um perfil monitorando demandas, outro executando automações e um terceiro revisando saídas críticas. Sem um quadro, a coordenação vira conversa espalhada. Com um quadro, o estado do trabalho fica centralizado.

# Exemplo de ciclo completo

1. tarefa entra no quadro;
2. alguém reivindica;
3. execução acontece;
4. a tarefa vai para revisão;
5. ajustes são feitos, se necessário;
6. conclusão é registrada.

Simples. E justamente por isso funciona.

# Conclusão

O chat direto continua ótimo para coisa rápida.

Mas, quando o trabalho precisa sobreviver à conversa, envolver múltiplos agentes ou passar por etapas reais de execução e revisão, Kanban oferece algo que a sessão sozinha não consegue: coordenação durável.

É nesse ponto que o Hermes deixa de operar apenas por conversa e passa a operar por fluxo.