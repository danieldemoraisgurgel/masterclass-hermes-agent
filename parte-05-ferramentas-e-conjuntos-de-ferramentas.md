# Parte 05: Ferramentas e Conjuntos de Ferramentas

O teste mais simples para saber se o Hermes está funcionando é fazer uma pergunta e receber uma resposta.

O teste funciona. Também é enganoso, porque testa o chat e deixa de fora a máquina.

O que torna o Hermes diferente de todos os chatbots que você já usou é a **superfície de ferramentas**, não a conversa.

O agente não apenas gera texto. Ele:

- executa comandos;
- pesquisa na web;
- lê e escreve arquivos;
- navega em páginas;
- gera imagens;
- transcreve fala;
- inicia subagentes.

O chat é a camada de entrada. As ferramentas são a saída.

---

## Registro Central de Ferramentas

Toda ferramenta disponível ao agente fica em um registro central:

```text
tools/registry.py
```

Quando um módulo de ferramenta é importado, ele chama `registry.register()` no nível do módulo, informando:

- nome da ferramenta;
- esquema;
- função manipuladora;
- verificação opcional de disponibilidade.

Essa chamada autorregistra a ferramenta em um dicionário indexado por nome.

O registro também descobre ferramentas automaticamente.

Durante a inicialização, a função `discover_builtin_tools()` examina cada arquivo Python existente no diretório:

```text
tools/
```

Ela executa uma verificação de AST em busca de chamadas de nível superior para:

```python
registry.register()
```

Em seguida, importa os módulos correspondentes.

Dessa forma, novos arquivos de ferramentas são detectados sem necessidade de conexão ou registro manual.

Depois da varredura principal, ferramentas MCP e ferramentas de plugins são descobertas por seus próprios mecanismos de descoberta.

O registro possui mais de **70 ferramentas**, distribuídas em aproximadamente **28 conjuntos de ferramentas**.

Essa contagem não inclui as ferramentas MCP conectadas pelo próprio usuário, que são adicionadas dinamicamente ao total.

---

## Categorias de Ferramentas

As ferramentas são agrupadas em categorias de capacidades de alto nível.

### Web

Responsável por pesquisar na internet e extrair conteúdo de páginas.

Existem duas ferramentas principais:

- `web_search`: pesquisa na web;
- `web_extract`: extração de conteúdo de páginas.

Essas são as ferramentas mais usadas em fluxos de pesquisa.

São também o que separa um agente que inventa informações de um agente que verifica os próprios fatos.

---

### Terminal e Arquivos

Essa categoria permite:

- executar comandos;
- gerenciar processos em segundo plano;
- ler arquivos;
- escrever arquivos;
- aplicar patches;
- pesquisar conteúdo dentro de arquivos.

A ferramenta de terminal oferece suporte a seis backends:

1. Local;
2. Docker;
3. SSH;
4. Singularity;
5. Modal;
6. Daytona.

Cada backend altera:

- onde os comandos são executados;
- o nível de isolamento;
- o nível de persistência;
- o grau de acesso à máquina do usuário.

---

### Navegador

As ferramentas de navegador permitem:

- abrir e navegar por páginas;
- clicar em elementos;
- digitar textos;
- preencher formulários;
- enviar formulários;
- fazer capturas de tela;
- executar JavaScript;
- rolar páginas.

O Hermes pode operar um navegador completo por meio de cinco backends:

1. Browserbase na nuvem;
2. Browser Use na nuvem;
3. Chrome local via CDP;
4. Chromium local gerenciado;
5. Supervisor CDP para trabalhos com `iframes` de origem cruzada.

A ferramenta de navegador não serve apenas para pesquisar informações. Ela pode efetivamente interagir com aplicações web, preencher formulários e enviá-los.

---

### Mídia

As ferramentas de mídia permitem:

- analisar imagens por meio de visão computacional;
- gerar imagens com base em prompts;
- converter texto em fala.

Essas ferramentas se conectam a backends específicos de modelos.

Dependendo da configuração, elas exigem:

- acesso ao Nous Portal Tool Gateway;
- chaves de API individuais;
- bibliotecas ou clientes adicionais instalados.

---

### Orquestração de Agentes

Essa categoria inclui recursos para:

- acompanhar tarefas;
- solicitar esclarecimentos;
- executar código;
- delegar tarefas a subagentes.

A ferramenta `execute_code` permite que o agente escreva scripts Python capazes de chamar outras ferramentas programaticamente.

Isso pode reduzir fluxos de trabalho com várias etapas a um único turno de inferência.

A ferramenta `delegate_task` inicia agentes filhos com:

- contexto isolado;
- sessões próprias de terminal;
- execução independente da sessão principal.

---

### Memória e Recuperação

Essa categoria contém recursos de persistência e recuperação de contexto.

As principais ferramentas são:

- `memory`: grava informações persistentes;
- `session_search`: pesquisa conversas e sessões anteriores.

A memória é o que se acumula entre sessões.

A busca de sessões é o que impede o agente de esquecer o trabalho realizado anteriormente, inclusive tarefas executadas dias ou semanas antes.

---

### Automação

A ferramenta `cronjob` permite:

- criar tarefas agendadas;
- atualizar tarefas existentes;
- consultar tarefas;
- gerenciar o ciclo de vida de trabalhos recorrentes;
- remover ou desabilitar tarefas.

Uma única ferramenta gerencia todo o ciclo de vida das automações recorrentes do agente.

---

### Integrações

As integrações incluem ferramentas específicas, como as do Home Assistant, além de qualquer servidor MCP conectado ao Hermes.

As ferramentas MCP são registradas dinamicamente durante a execução, com base nos servidores configurados.

Cada servidor MCP expõe seu próprio conjunto de ferramentas usando um namespace semelhante a:

```text
mcp-<nome-do-servidor>
```

---

# Conjuntos de Ferramentas Controlam o Alcance do Agente

Um conjunto de ferramentas é um pacote nomeado de ferramentas.

Esses conjuntos existem para permitir que diferentes superfícies ou plataformas recebam diferentes níveis de capacidade.

Por exemplo, o perfil da CLI pode carregar um conjunto amplo, contendo:

- web;
- terminal;
- arquivos;
- navegador;
- visão;
- geração de imagens;
- delegação;
- automação.

Já um perfil de bot do Telegram pode carregar um conjunto mais restrito, contendo apenas:

- mensagens;
- busca de sessões;
- tarefas cron.

Nesse cenário, o bot não teria acesso ao terminal nem ao navegador.

O sistema de predefinições de plataforma gerencia esses mapeamentos automaticamente.

---

## Configuração dos Conjuntos de Ferramentas

Os conjuntos de ferramentas podem ser configurados por meio do comando:

```bash
hermes tools
```

Também podem ser definidos no arquivo:

```text
config.yaml
```

É possível habilitar ou desabilitar conjuntos de ferramentas individualmente para cada plataforma.

Um detalhe importante é que o atalho:

```text
all
```

habilita a maioria dos conjuntos de ferramentas, mas não todos.

Conjuntos especializados, como `kanban`, exigem adesão explícita e precisam ser adicionados separadamente, mesmo quando `all` já estiver habilitado.

Por isso, é importante executar:

```bash
hermes tools
```

Esse comando permite verificar o que está realmente carregado no perfil atual, evitando presumir que determinada ferramenta está disponível.

---

## Verificação de Disponibilidade com `check_fn`

Toda ferramenta pode fornecer uma função chamada:

```python
check_fn
```

Essa função retorna:

- `True`, quando a ferramenta pode ser utilizada;
- `False`, quando a ferramenta está indisponível.

Por exemplo:

- a ferramenta de pesquisa na web verifica se uma chave de API de pesquisa foi configurada;
- a ferramenta de navegador verifica se um backend de navegador está acessível;
- a ferramenta de geração de imagens verifica se o cliente FAL AI está instalado.

Quando o agente cria a lista de esquemas que será apresentada ao modelo, ele executa a `check_fn` de cada ferramenta.

Ferramentas indisponíveis são excluídas do esquema.

Isso significa que o modelo nunca recebe definições de ferramentas que não pode utilizar de fato.

Caso uma chave de API seja adicionada posteriormente e o Hermes seja reiniciado, a ferramenta correspondente passa a aparecer.

A superfície de capacidades do agente pode, portanto, mudar com base apenas na configuração, sem qualquer alteração de código.

Por exemplo:

- instale um servidor MCP e reinicie o Hermes: novas ferramentas serão adicionadas;
- remova uma credencial e reinicie: as ferramentas dependentes dessa credencial desaparecerão.

O agente não sabe o que não pode fazer, porque as ferramentas indisponíveis não são apresentadas a ele.

---

# Ambientes de Execução do Terminal

A ferramenta de terminal oferece suporte a seis ambientes de execução, cada um com características distintas de segurança e persistência.

## Local

Executa comandos diretamente na máquina do usuário.

Características:

- acesso amplo ao sistema;
- baixa isolação;
- acesso aos arquivos e recursos locais;
- maior risco em caso de execução de comandos destrutivos.

---

## Docker

Executa comandos dentro de um contêiner isolado.

Características:

- sistema de arquivos raiz somente leitura;
- capacidades do sistema removidas;
- maior isolamento em relação à máquina hospedeira;
- ambiente controlado para execução de comandos.

---

## SSH

Delega a execução para um servidor remoto.

Esse modelo mantém o agente afastado da máquina e do código locais.

É útil para:

- administração remota;
- execução em servidores dedicados;
- separação do ambiente de controle e do ambiente de trabalho.

---

## Singularity

Oferece suporte a ambientes de computação de alto desempenho, especialmente clusters HPC.

É adequado para cenários nos quais Docker não está disponível ou não é permitido.

---

## Modal e Daytona

Executam comandos em ambientes de nuvem sem servidor.

Esses ambientes podem hibernar quando estão ociosos, reduzindo o consumo de recursos.

São úteis para:

- execução temporária;
- isolamento;
- tarefas elásticas;
- ambientes descartáveis ou sob demanda.

---

# Persistência do Backend Docker

O backend Docker merece atenção especial.

No primeiro uso, o Hermes inicia um único contêiner de longa duração.

Todas as chamadas das seguintes ferramentas são encaminhadas para esse mesmo contêiner:

- terminal;
- arquivos;
- `execute_code`.

Dentro da vida útil do processo, permanecem disponíveis:

- alterações no diretório de trabalho;
- pacotes instalados;
- arquivos criados;
- configurações de ambiente;
- mudanças realizadas por subagentes.

Essas alterações persistem entre chamadas de ferramentas e delegações de subagentes.

Por padrão, quando o Hermes é encerrado, o contêiner é interrompido e removido.

Para preservar o espaço de trabalho entre reinicializações, é possível utilizar:

```yaml
container_persistent: true
```

Com essa configuração, o ambiente de trabalho sobrevive às reinicializações do Hermes.

---

# Gerenciamento de Processos em Segundo Plano

O gerenciamento de processos em segundo plano está integrado à superfície de ferramentas.

Um comando pode ser iniciado em segundo plano por meio de uma chamada semelhante a:

```text
terminal(background=true)
```

A ferramenta `process` pode então:

- listar processos;
- consultar o estado de um processo;
- aguardar sua conclusão;
- enviar dados para a entrada padrão;
- encerrar processos;
- fechar processos concluídos.

O modo PTY permite utilizar ferramentas de linha de comando interativas, como:

- Codex;
- Claude Code;
- outras aplicações que exigem um terminal interativo.

---

# Comandos Perigosos Exigem Aprovação

A ferramenta de terminal possui um sistema de detecção de comandos perigosos.

A variável:

```python
DANGEROUS_PATTERNS
```

contém uma lista de expressões regulares que identifica operações potencialmente destrutivas.

Entre os padrões detectados estão:

- exclusões recursivas;
- formatação de sistemas de arquivos;
- operações SQL destrutivas;
- sobrescrita de configurações do sistema;
- manipulação de serviços;
- execução remota de código;
- fork bombs.

Quando um comando corresponde a um desses padrões, o agente solicita aprovação antes de executá-lo.

Na CLI, o usuário recebe um prompt interativo.

Em plataformas de mensagens, o agente envia uma solicitação de aprovação pelo próprio chat.

As aprovações são rastreadas por sessão.

Também é possível configurar uma lista permanente de permissões no arquivo:

```text
config.yaml
```

---

# Por Que o Acesso a Ferramentas Muda Tudo

Todos os recursos apresentados no restante desta série dependem das ferramentas.

- Skills são procedimentos executados por meio de ferramentas.
- Trabalhos cron agendam chamadas de ferramentas.
- A delegação inicia subagentes que utilizam ferramentas.
- A automação de navegador é uma categoria de ferramenta.
- O uso de computador depende de ferramentas.
- O loop do agente despacha e acompanha chamadas de ferramentas.

Sem uma superfície de ferramentas funcional, o Hermes é apenas um chatbot com memória persistente.

Com uma superfície de ferramentas funcional, o agente pode agir sobre o mundo.

---

# Teste de Fumaça Essencial

O primeiro teste de fumaça apresentado no Artigo 2 continua sendo a etapa de verificação mais importante para qualquer instalação do Hermes.

O teste utiliza três chamadas de ferramentas, cobrindo:

1. terminal;
2. web;
3. arquivos.

Se essas três categorias funcionarem, a superfície principal de ferramentas está acessível.

Se uma delas falhar, o problema deve ser corrigido antes da implementação de qualquer recurso adicional, porque todo o restante depende dessas capacidades.

A superfície de ferramentas torna o agente capaz.

Porém, uma capacidade que funciona apenas enquanto você está sentado diante do teclado representa somente metade da história.
