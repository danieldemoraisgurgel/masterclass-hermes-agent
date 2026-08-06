# Parte 11: A Camada de Administração

> Um agente é uma ferramenta. Dois agentes são um sistema. Três ou mais são uma operação.

Assim que você executa múltiplos perfis Hermes — um para programação, um para pesquisa, um para o gateway e outro como trabalhador Kanban —, precisa de uma camada que os gerencie.

O **dashboard**, o **servidor de API** e o **sistema de perfis** formam essa camada:

- **Perfis** são o limite de execução.
- **Dashboard** é a superfície de controle.
- **Servidor de API** é o ponto de integração.

---

## Perfis São o Limite de Execução

Cada perfil Hermes é um agente completamente independente, com:

- Seu próprio `config.yaml`;
- Seu próprio `HERMES_HOME`;
- Sua própria memória;
- Suas próprias sessões;
- Suas próprias habilidades;
- Seus próprios trabalhos cron;
- Seu próprio estado de gateway;
- Suas próprias credenciais.

Por exemplo:

```bash
hermes -p coder chat
```

Esse comando inicia uma sessão no perfil `coder`.

Já o comando:

```bash
hermes -p researcher chat
```

inicia uma sessão completamente separada no perfil `researcher`.

Os perfis não compartilham nada por padrão.

### Criando Perfis

Criar um perfil requer apenas um comando.

Para criar um perfil vazio, com uma configuração nova:

```bash
hermes profile create coder
```

Para criar um perfil copiando a configuração, as habilidades e o `SOUL` do perfil atual:

```bash
hermes profile create work --clone-from default
```

Para criar um snapshot completo, incluindo memórias, sessões e trabalhos cron:

```bash
hermes profile create backup --clone-all
```

### Aliases de Comando

Cada perfil recebe automaticamente seu próprio alias de comando.

Ao criar um perfil chamado `coder`, o seguinte comando passa a iniciar o Hermes diretamente nesse perfil:

```bash
coder chat
```

O prefixo da CLI passa a ser o nome do perfil. Portanto, não é necessário digitar:

```bash
hermes -p coder
```

todas as vezes.

### Gateways por Perfil

Os gateways também são configurados por perfil.

Cada perfil executa seu próprio gateway como um processo separado, com seu próprio token de bot.

Por exemplo:

- O perfil A pode se comunicar com o Telegram;
- O perfil B pode se comunicar com o Discord.

Caso dois perfis utilizem acidentalmente o mesmo token de bot, o segundo gateway se recusará a iniciar e apresentará uma mensagem clara informando o conflito.

### Isolamento do Diretório Home

A configuração de modo *home* controla o nível de isolamento da execução de ferramentas de um perfil.

Por padrão, todos os perfis compartilham o diretório *home* real do usuário. Isso significa que recursos como os seguintes também são compartilhados:

- Configuração do Git;
- Estado do npm;
- Chaves SSH;
- Outros arquivos e configurações do usuário.

Para restringir a execução de ferramentas ao diretório *home* específico do perfil, configure:

```yaml
terminal:
  home_mode: profile
```

Essa opção é útil quando, por exemplo, um trabalhador Kanban precisa possuir sua própria identidade Git.

### Exportação e Distribuição de Perfis

Perfis podem ser empacotados como distribuições.

Para exportar o perfil `coder`:

```bash
hermes profile export coder
```

Esse comando cria um arquivo compartilhável contendo:

- Configuração;
- `SOUL`;
- Habilidades;
- Trabalhos cron.

As credenciais e o histórico de sessões permanecem armazenados localmente e não são incluídos na distribuição.

Para instalar a distribuição em outra máquina:

```bash
hermes profile install
```

---

## Página 39

## O Dashboard É a Superfície de Controle

O dashboard web é executado como um servidor web opcional.

Ele fornece uma visão gráfica da instalação do Hermes, incluindo:

- Perfil atual;
- Sessões ativas;
- Configuração de modelo;
- Configurações de ferramentas;
- Status do gateway;
- Biblioteca de habilidades;
- Trabalhos cron.

### Alternância entre Perfis

O principal valor do dashboard é o alternador de perfis.

Por meio dele, é possível:

- Alternar entre perfis;
- Inspecionar a configuração de cada perfil;
- Editar habilidades;
- Ajustar configurações de modelo;
- Iniciar sessões de chat;
- Acompanhar sessões de diferentes perfis.

As sessões de todos os perfis aparecem no mesmo dashboard.

Assim, é possível visualizar no que o perfil `coder` está trabalhando enquanto você conversa com o perfil `researcher`.

### Interface de Chat

O dashboard também disponibiliza uma interface de chat.

A partir do navegador, você pode:

1. Iniciar uma sessão;
2. Escolher um perfil;
3. Conversar com o agente.

O histórico de sessões é compartilhado entre:

- CLI;
- Gateway;
- Dashboard.

Dessa forma, é possível alternar entre diferentes superfícies sem perder o contexto da conversa.

---

## O Servidor de API É o Ponto de Integração

O Hermes disponibiliza um endpoint HTTP compatível com o formato da API da OpenAI.

Qualquer frontend que utilize esse formato pode controlar o Hermes, incluindo:

- Open WebUI;
- LobeChat;
- LibreChat;
- Ferramentas semelhantes.

O servidor de API encaminha as solicitações pelo loop do agente associado ao perfil ativo.

Ele utiliza os mesmos componentes empregados pela CLI:

- Montagem de prompt;
- Distribuição de ferramentas;
- Persistência de sessões;
- Memória;
- Habilidades;
- Superfície de ferramentas.

A única diferença é o meio de transporte:

- Na CLI, a interação ocorre pelo terminal;
- No servidor de API, a interação ocorre por HTTP.

### Casos de Uso

O servidor de API é útil para:

- Construir frontends personalizados;
- Integrar o Hermes a fluxos de trabalho existentes;
- Disponibilizar o agente para uma equipe;
- Criar uma interface compartilhada;
- Integrar aplicações compatíveis com a API da OpenAI.

O servidor de API também lida com:

- Autenticação;
- Limitação de taxa;
- Gerenciamento de sessões.

Múltiplos usuários podem se conectar a uma única instância do Hermes, cada um com sua própria sessão.

---

## O Dashboard Web É a Superfície de Controle Administrativo

O dashboard web permite operar o Hermes sem depender continuamente do terminal.

A partir de uma aba do navegador, você pode:

- Alternar entre perfis e visualizar a configuração, o modelo, as ferramentas e o status do gateway de cada um;
- Editar habilidades diretamente no navegador, com destaque de sintaxe e pré-visualização ao vivo;
- Gerenciar trabalhos cron, incluindo criar, editar, pausar, retomar e remover;
- Inspecionar o histórico de sessões entre perfis e superfícies;
- Monitorar atividades em andamento.

Cada sessão iniciada pela CLI, pelo gateway ou pelo dashboard utiliza o mesmo armazenamento de sessões.

As visualizações do dashboard também apresentam:

- Sessões ativas;
- Chamadas de ferramentas em execução;
- Status dos gateways;
- Conclusões recentes.

O dashboard é opcional.

---

## Página 40

O dashboard é uma camada de conveniência para gerenciamento visual.

Tudo o que pode ser feito pelo dashboard também pode ser realizado pela CLI. Entretanto, para operadores que executam múltiplos perfis e múltiplos gateways, a visualização centralizada pode economizar tempo.

---

## O Que Muda com Múltiplos Perfis

Uma instalação Hermes com perfil único é simples.

Ela possui:

- Um agente;
- Uma configuração;
- Um conjunto de habilidades;
- Um gateway.

Tudo fica concentrado em um único local.

Com múltiplos perfis, o modelo operacional muda. Cada perfil passa a funcionar como seu próprio agente, com uma finalidade específica.

### Exemplo de Especialização

Um ambiente pode possuir os seguintes perfis:

#### Perfil de Programação

- Acesso ao terminal;
- Modelo específico para programação;
- Ferramentas de desenvolvimento;
- Acesso a repositórios e arquivos.

#### Perfil de Pesquisa

- Ferramentas de pesquisa web;
- Modelo mais barato para processamento em volume;
- Habilidades voltadas para análise e síntese de informações.

#### Perfil Trabalhador Kanban

- Sem gateway;
- Conjunto mínimo de ferramentas;
- Execução como serviço em segundo plano;
- Supervisão contínua de tarefas do quadro Kanban.

### Coordenação entre Perfis

Os perfis não compartilham nada por padrão. Esse é o comportamento mais seguro.

Entretanto, eles podem ser coordenados por meio de:

1. Um quadro Kanban;
2. Um sistema de arquivos compartilhado configurado explicitamente.

O quadro Kanban apresentado no Artigo 10 é o mecanismo de coordenação recomendado, pois não exige acesso compartilhado ao sistema de arquivos.

### Sobrecarga Operacional

A utilização de múltiplos perfis aumenta a sobrecarga operacional.

Mais perfis significam:

- Mais arquivos de configuração;
- Mais gateways para administrar;
- Mais processos em execução;
- Mais credenciais;
- Mais pontos de falha;
- Mais componentes que podem apresentar problemas.

Por esse motivo, recomenda-se começar com um único perfil e adicionar perfis especializados somente quando existir um caso de uso claro.

Para a maioria dos ambientes, geralmente é suficiente utilizar:

1. Um perfil de propósito geral, com todas as ferramentas necessárias;
2. Um perfil de pesquisa, configurado com um modelo mais econômico para tarefas em volume.

---

## Conclusão

Perfis, dashboard e servidor de API fornecem uma camada de controle para ambientes com múltiplos agentes.

Cada componente possui uma função específica:

| Componente | Função |
|---|---|
| Perfis | Isolam agentes, configurações, memórias, sessões e credenciais |
| Dashboard | Centraliza o gerenciamento visual e administrativo |
| Servidor de API | Integra o Hermes a frontends, aplicações e fluxos externos |
| Quadro Kanban | Coordena o trabalho entre agentes independentes |

O próximo e último artigo desta série aborda os limites do Hermes, incluindo:

- Janelas de contexto;
- Orçamentos de tokens;
- Isolamento de perfis;
- Situações em que é mais inteligente não utilizar o Hermes.