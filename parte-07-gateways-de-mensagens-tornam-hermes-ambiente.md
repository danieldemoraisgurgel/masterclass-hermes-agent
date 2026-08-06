# Parte 07: Gateways de Mensagens Criam o Ambiente Hermes

Tudo nesta série, até agora, pressupôs que você está sentado diante de um teclado:

- CLI;
- TUI;
- aplicativo de desktop.

Você digita uma mensagem e recebe uma resposta. Funciona, mas também representa uma limitação, porque significa que o Hermes só é útil quando você está na sua máquina.

**Gateways de mensagens quebram essa restrição.**

O Hermes pode ser integrado a diferentes plataformas:

- Telegram;
- Discord;
- Slack;
- WhatsApp;
- Signal;
- iMessage;
- SMS;
- e-mail;
- Matrix;
- Microsoft Teams;
- e aproximadamente uma dúzia de outras plataformas.

Você pode conversar com o Hermes a partir de qualquer uma delas. O agente não sabe — e não precisa saber — qual plataforma está sendo utilizada.

As sessões podem atravessar os limites das plataformas:

1. inicie uma atividade na CLI;
2. verifique o resultado no Telegram;
3. retome a conversa no Discord.

O gateway é um único processo executado em segundo plano. Ele funciona junto com o agente, conecta-se a todas as plataformas configuradas, gerencia o roteamento de sessões, executa trabalhos cron e entrega mensagens de voz.

É isso que faz o Hermes parecer uma infraestrutura, em vez de apenas um aplicativo.

## Como o Gateway Processa as Mensagens

Quando uma mensagem chega por qualquer plataforma, o respectivo adaptador a converte em um evento de mensagem padronizado.

O fluxo ocorre da seguinte maneira:

1. o adaptador da plataforma recebe a mensagem;
2. a mensagem é convertida em um evento padronizado;
3. o roteador do gateway procura a sessão associada ao chat;
4. caso a sessão não exista, uma nova sessão é criada;
5. a mensagem e o histórico da sessão são entregues a uma nova instância do `AIAgent`;
6. o agente processa a solicitação;
7. o agente gera uma resposta;
8. o gateway devolve a resposta pelo mesmo adaptador de origem.

Em termos simplificados:

```text
Plataforma
    ↓
Adaptador
    ↓
Evento de mensagem padronizado
    ↓
Roteador de sessões
    ↓
AIAgent
    ↓
Resposta
    ↓
Adaptador
    ↓
Plataforma
```

## Sessões Portáteis Entre Plataformas

As sessões são portáteis entre plataformas.

Uma sessão iniciada no Telegram possui o mesmo histórico e a mesma memória de uma sessão acessada pela CLI ou pelo Discord. Isso é possível porque o armazenamento de sessões é compartilhado.

Você pode iniciar uma tarefa de pesquisa pelo celular, caminhar até sua mesa e continuar a mesma conversa no terminal.

O canal de acesso muda, mas a sessão permanece a mesma.

## Integração com o Agendador Cron

O gateway também executa o agendador cron.

A cada 60 segundos, o gateway consulta o armazenamento de trabalhos e dispara aqueles que estiverem vencidos.

A entrega dos resultados utiliza o mesmo sistema de adaptadores das mensagens comuns. Assim, o resultado de um trabalho cron chega diretamente ao chat configurado.

```text
Agendador cron
    ↓
Trabalho vencido
    ↓
Execução pelo agente
    ↓
Resultado
    ↓
Roteador do gateway
    ↓
Adaptador da plataforma
    ↓
Chat configurado
```

## A Configuração É Feita com um Único Comando

O caminho mais rápido para configurar um gateway funcional é executar o assistente interativo de configuração.

> O comando exato não foi incluído no trecho original.

O assistente conduz a configuração de cada plataforma por meio de uma interface com seleção pelas teclas de seta.

Ele permite:

- visualizar quais plataformas já estão configuradas;
- configurar novas plataformas;
- validar credenciais;
- revisar as configurações;
- iniciar o gateway ao final do processo.

## Telegram

O Telegram é o caminho mais simples para a maioria das pessoas.

O processo básico consiste em:

1. criar um bot por meio do **BotFather**;
2. copiar o token fornecido;
3. informar o token ao assistente de configuração;
4. iniciar o gateway.

Não é necessário:

- possuir um endereço IP estático;
- configurar um webhook;
- expor uma porta pública;
- publicar diretamente o gateway na internet.

O gateway utiliza **long polling**. Por isso, ele funciona mesmo quando está:

- atrás de NAT;
- em um laptop que muda frequentemente de rede;
- em uma VPS com endereço IP dinâmico.

## Outras Plataformas

Cada plataforma possui requisitos específicos.

### Discord

O Discord precisa de:

- um token de bot;
- um ID de aplicativo obtido no Discord Developer Portal.

### WhatsApp

O WhatsApp utiliza a ponte integrada baseada em **Baileys**.

### Signal

O Signal depende de um daemon do `signal-cli`.

Apesar das diferenças, o assistente interativo cobre o fluxo mais comum de configuração de cada plataforma.

## Personalidade da Plataforma

Plataformas diferentes recebem configurações de ferramentas diferentes por padrão.

Isso permite que o agente ajuste o comportamento, o formato das respostas e o volume de informações de acordo com o ambiente em que está sendo utilizado.

### Telegram

O Telegram é tratado como uma caixa de entrada móvel.

Por padrão:

- mensagens automáticas de progresso ficam desativadas;
- atualizações intermediárias são reduzidas;
- as respostas tendem a ser mais curtas;
- o agente prefere texto simples;
- o uso de Markdown é reduzido.

O objetivo é diminuir o ruído de notificações e melhorar a leitura em dispositivos móveis.

### CLI

A CLI recebe um conjunto mais amplo de ferramentas, incluindo:

- acesso ao terminal;
- acesso a arquivos;
- respostas mais detalhadas;
- tabelas;
- blocos de código;
- Markdown mais complexo;
- atualizações de progresso.

### Discord

O Discord fica em uma posição intermediária entre a experiência móvel do Telegram e o ambiente completo da CLI.

## Por Que as Dicas de Plataforma Importam

Uma resposta com cinco parágrafos e várias tabelas em Markdown pode funcionar muito bem em um terminal, mas ficar ruim na tela de um telefone.

O sistema de dicas de plataforma informa ao agente em qual ambiente ele está respondendo. A partir disso, o agente pode ajustar:

- o tamanho da resposta;
- o uso de Markdown;
- a frequência das atualizações;
- a exibição de progresso;
- o nível de detalhamento;
- a forma de apresentar comandos e tabelas.

As configurações padrão podem ser substituídas individualmente por plataforma.

É possível controlar, de forma independente:

- mensagens de progresso de ferramentas;
- *heartbeats* indicando que o agente ainda está trabalhando;
- atualizações de status;
- tamanho e estilo das respostas;
- orientações adicionais para o agente.

A chave de configuração `platform_hints` permite acrescentar ou substituir as orientações que o agente recebe em cada plataforma.

Exemplo conceitual:

```yaml
platform_hints:
  telegram:
    prefer_short_responses: true
    prefer_plain_text: true
    tool_progress: false

  cli:
    prefer_detailed_responses: true
    markdown: true
    tool_progress: true

  discord:
    prefer_short_responses: false
    markdown: true
    tool_progress: true
```

> O exemplo acima é apenas ilustrativo. A estrutura exata deve seguir o formato de configuração suportado pela versão instalada do Hermes.

## A Autorização Mantém a Segurança

Por padrão, o gateway nega mensagens de usuários que não tenham sido explicitamente autorizados.

Ninguém pode conversar com o agente até que sua identidade seja aprovada.

A autorização funciona principalmente por meio de dois mecanismos:

1. listas de permissões;
2. pareamento de mensagens diretas.

## Listas de Permissões

A lista de permissões contém os IDs dos usuários autorizados a conversar com o agente.

Uma mensagem recebida de um usuário que não esteja na lista é rejeitada ou ignorada, conforme a configuração da plataforma.

Exemplo conceitual:

```yaml
telegram:
  allowed_users:
    - "123456789"
    - "987654321"
```

## Pareamento por Mensagem Direta

O pareamento de mensagem direta, ou **DM pairing**, utiliza um código de uso único.

O fluxo é semelhante ao seguinte:

1. uma conta autorizada inicia o processo de pareamento;
2. o gateway gera um código temporário;
3. o novo usuário envia esse código em uma mensagem direta;
4. o gateway verifica o código;
5. a conta é vinculada e autorizada.

Esse processo comprova que o usuário controla a conta da plataforma que está tentando conectar.

## Níveis de Acesso

Os usuários podem pertencer a dois níveis por plataforma.

### Administrador

O administrador possui acesso completo e pode:

- executar todos os comandos com barra;
- administrar o gateway;
- inspecionar adaptadores;
- pausar e retomar plataformas;
- gerenciar configurações permitidas;
- executar operações administrativas.

### Usuário

O usuário possui acesso normal à conversa com o agente, sem os privilégios administrativos do gateway.

As permissões são definidas separadamente para cada plataforma.

Ser administrador em uma conversa direta não significa, necessariamente, ser administrador em um grupo. Da mesma forma, uma permissão concedida no Telegram não precisa existir no Discord.

## Primeiro Contato e Pareamento

O fluxo de pareamento protege a experiência do primeiro contato.

Quando um usuário ainda não pareado envia uma mensagem, o agente ignora a solicitação, a menos que o processo de pareamento tenha sido previamente iniciado por uma conta autorizada.

Isso reduz o risco de:

- acesso não autorizado;
- abuso de ferramentas;
- execução de comandos administrativos;
- consumo indevido de recursos;
- exposição acidental de informações;
- interação pública não controlada com o agente.

## Disjuntores Mantêm o Serviço em Execução

Cada adaptador de plataforma é encapsulado em um **disjuntor**, ou *circuit breaker*.

O disjuntor monitora falhas repetidas que podem permitir uma nova tentativa, como:

- instabilidades de rede;
- limites de taxa;
- respostas HTTP `5xx`;
- desconexões de WebSocket;
- falhas temporárias de autenticação;
- indisponibilidade da API da plataforma.

Quando o número de falhas ultrapassa o limite configurado, o disjuntor é acionado.

## O Que Acontece Quando o Disjuntor É Acionado

Quando um adaptador falha repetidamente:

1. o disjuntor é acionado;
2. o adaptador com problema é pausado automaticamente;
3. uma notificação é enviada ao operador;
4. a notificação utiliza o canal doméstico de alguma plataforma que ainda esteja ativa;
5. o gateway continua funcionando;
6. apenas a plataforma com falha fica temporariamente silenciosa.

Isso evita que a falha de uma única integração derrube todo o gateway.

```text
Telegram ───── ativo
Discord ────── ativo
WhatsApp ───── falha repetida
                  ↓
             disjuntor acionado
                  ↓
          WhatsApp é pausado

Gateway continua em execução
```

## Retomando um Adaptador

Retomar um adaptador cujo disjuntor foi acionado requer apenas um comando.

> O comando de terminal exato não foi incluído no trecho original.

A retomada também pode ser feita a partir de qualquer chat ativo:

```text
/platform resume <name>
```

Exemplo:

```text
/platform resume whatsapp
```

Esse comando:

- limpa o estado do disjuntor;
- rearma o adaptador;
- permite que a plataforma volte a processar mensagens.

## Gerenciamento de Plataformas

O comando com barra `/platform` permite inspecionar e controlar adaptadores individualmente, sem reiniciar todo o gateway.

Entre as operações possíveis estão:

```text
/platform list
/platform pause <name>
/platform resume <name>
```

Exemplos:

```text
/platform list
/platform pause telegram
/platform resume telegram
```

Ao pausar uma plataforma, o adaptador permanece carregado e seus loops em segundo plano continuam ativos.

Entretanto, as mensagens recebidas são descartadas silenciosamente.

Como a conexão continua aberta, a retomada pode ser praticamente instantânea.

## A Mudança para o Ambiente

O gateway é o componente que transforma o Hermes de algo que você abre quando precisa em algo que simplesmente está disponível.

Você pode:

- enviar mensagens pelo telefone enquanto se desloca;
- continuar a mesma conversa no terminal ao chegar à mesa;
- receber resultados de trabalhos cron no mesmo chat;
- receber alertas operacionais;
- acompanhar chamadas de ferramentas demoradas;
- receber notificações quando o gateway for reiniciado.

Quando uma chamada de ferramenta leva mais tempo, o gateway pode enviar atualizações de progresso.

Quando o gateway reinicia após uma atualização, ele envia uma notificação única ao canal doméstico de cada plataforma, informando que o serviço voltou a funcionar.

## O Gateway e o Ciclo de Aprendizagem

O ciclo de aprendizagem apresentado no Artigo 3 só se acumula se o agente estiver acessível.

Uma skill de escrita criada pelo agente tem pouca utilidade se ela só puder ser utilizada em um terminal diante do qual você não está sentado.

O gateway garante que as skills, ferramentas, memórias e sessões do agente estejam disponíveis onde quer que você esteja.

Ele conecta:

- o agente;
- as plataformas;
- as sessões;
- a memória;
- as skills;
- o agendador cron;
- as notificações;
- os mecanismos de autorização;
- os controles operacionais.

## De Aplicativo para Infraestrutura

Sem o gateway, o Hermes é um aplicativo que você abre para iniciar uma conversa.

Com o gateway, o Hermes passa a ser um serviço persistente:

- sempre ativo;
- acessível por diferentes plataformas;
- capaz de executar tarefas agendadas;
- capaz de entregar resultados automaticamente;
- protegido por controles de autorização;
- resiliente a falhas individuais de plataforma;
- integrado à rotina do usuário.

Um agente em uma plataforma é útil.

**Vários agentes trabalhando em paralelo, em tarefas diferentes, é onde o valor começa a se acumular.**
