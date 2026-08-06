# Parte 12: O que Pular, o que Quebra e Como Manter a Sanidade

Onze artigos de recursos, ferramentas e possibilidades. Agora, a conversa real.

Hermes é o agente de código aberto mais capaz que já usei. Também é um sistema complexo, com arestas afiadas.

Entender o que quebra, o que pular e quando usar outra coisa é a diferença entre um agente que se acumula e um que frustra.

Aqui estão os modos de falha que encontrei, as coisas que pulo e as regras que sigo.

## Toda Modelo Tem uma Janela de Contexto

O Hermes é executado dentro dessa janela. O prompt de sistema, o snapshot de memória, o índice de habilidades, o histórico da conversa e os resultados de chamadas de ferramentas disputam o mesmo espaço limitado.

A compressão de pré-voo entra em ação em 50% da janela de contexto. O gateway faz compressão automática em 85%.

Os turnos do meio são resumidos. As últimas 20 mensagens são preservadas. Uma nova linhagem de sessão é criada.

Isso é automático e geralmente invisível. Também é destrutivo.

> Conteúdo comprimido é um resumo, não o original.

Cadeias longas de chamadas de ferramentas aceleram a pressão no contexto. Uma única tarefa de pesquisa pode produzir:

- Cinco resultados de busca;
- Três extrações de página;
- Um resumo.

Isso representa nove pares de chamada de ferramenta e resultado alimentando o histórico da conversa. Se o modelo executar outro lote, o histórico dobra.

A compressão de pré-voo lida com isso, mas o agente perde nuance a cada compressão.

O orçamento de iterações é a válvula de segurança. O padrão é de 90 turnos. Cada chamada de ferramenta conta como um turno.

Uma tarefa complexa com 20 chamadas de ferramentas usa 20 dos 90 turnos.

Subagentes têm seus próprios orçamentos. Portanto, a delegação é uma solução alternativa para o contexto limitado. As iterações do agente filho não contam contra o orçamento do agente pai.

> **Regra prática:** se uma tarefa precisar de mais de 30 chamadas de ferramentas, delegue-a a um subagente ou divida-a em múltiplas sessões.

Não deixe uma única cadeia de conversa crescer até ser comprimida e se tornar ilegível.

## A Fragilidade das Integrações é Constante

Toda ferramenta que depende de um serviço externo é um ponto de falha.

- A ferramenta de busca na web precisa de uma chave de API.
- A ferramenta de navegador precisa de saldo de crédito de um provedor de nuvem ou de uma instância local do Chromium em execução.
- A ferramenta de geração de imagens precisa ter o cliente FAL instalado.
- A ferramenta de TTS precisa de uma chave do ElevenLabs ou de um serviço semelhante.

Credenciais expiram. Limites de taxa são atingidos. Planos gratuitos acabam.

O `check_fn` da ferramenta informa que ela está indisponível, e o modelo perde silenciosamente o acesso.

Sem erro. Sem alerta. A ferramenta simplesmente não aparece no esquema.

---

**Página 42**

---

O sistema de provedor alternativo lida com falhas do modelo. Quando o modelo primário retorna um erro `429` ou `5xx`, o Hermes tenta a lista de alternativas.

Entretanto, as alternativas cobrem somente o modelo, não as ferramentas.

Se sua chave de API de busca na web expirar, a ferramenta de busca desaparece independentemente de qual modelo você esteja usando.

> **Regra prática:** execute `hermes tools` periodicamente para verificar se sua superfície de ferramentas está intacta.

Configure um trabalho cron semanal que teste as três ferramentas do Artigo 2:

1. Terminal;
2. Web;
3. Arquivo.

O trabalho deve alertar você caso alguma delas falhe.

> Uma ferramenta que desaparece silenciosamente é pior que uma ferramenta que nunca funcionou.

## O Isolamento de Perfis Tem Armadilhas

Perfis oferecem agentes independentes. Eles também compartilham seu sistema real por padrão.

O valor padrão de `terminal.home_mode` dá a todo perfil acesso ao seu diretório pessoal real.

Isso significa que:

- A configuração Git de um perfil de programação é a mesma configuração Git do perfil de pesquisa;
- Um perfil de pesquisa pode ler arquivos do diretório do projeto de programação;
- Credenciais, arquivos e estados locais podem ser compartilhados involuntariamente.

Perfis são isolados por configuração, não pelo sistema operacional.

### Isolamento do Diretório Pessoal

A configuração do modo de diretório pessoal resolve o problema mais comum:

```yaml
terminal:
  home_mode: profile
```

Com `terminal.home_mode: profile`, a execução das ferramentas fica restrita a:

```text
{HERMES_HOME}/home
```

Cada perfil recebe:

- Seu próprio `.gitconfig`;
- Suas próprias chaves SSH;
- Seu próprio estado do npm;
- Seu próprio ambiente de execução local.

A desvantagem é que ferramentas como `git` e `gh` precisam ser autenticadas novamente em cada perfil.

### Conflitos de Tokens do Gateway

Conflitos de tokens do gateway são a segunda armadilha.

O gateway de cada perfil precisa de seu próprio token de bot.

Se dois perfis compartilharem um token de bot do Telegram, o segundo gateway recusará a inicialização e apresentará um erro claro.

A correção é criar bots separados para perfis separados.

### Persistência de Sessões

A persistência de sessões é a terceira armadilha.

Um contêiner Docker sem uma montagem de volume perde todas as sessões quando é reiniciado.

A hibernação de um backend serverless perde as sessões ativas até que o ambiente desperte novamente.

Teste a persistência das sessões antes de construir qualquer coisa sobre ela:

1. Inicie uma conversa.
2. Pare o agente.
3. Reinicie o agente.
4. Execute:

```bash
hermes -c
```

Se a conversa retornar, a persistência está funcionando.

## O Curador Pode Morder

O curador foi projetado para evitar a deterioração e o acúmulo de habilidades.

Ele arquiva habilidades que não foram usadas durante 90 dias e é executado automaticamente a cada sete dias.

Isso é ótimo para habilidades criadas por agentes que se acumularam a partir de uma dúzia de fluxos de trabalho diferentes.

Porém, é inconveniente quando o curador arquiva uma habilidade que você escreveu e utiliza apenas uma vez por trimestre.

A correção é simples:

```bash
hermes curator pin <name>
```

Habilidades fixadas são removidas completamente da jurisdição do curador.

O agente ainda pode modificá-las, mas o curador não pode arquivá-las.

> Fixe qualquer habilidade que você escreveu pessoalmente e deseja manter.

O curador também nunca exclui habilidades automaticamente.

As habilidades arquivadas são movidas para:

```text
~/.hermes/skills/.archive/
```

Elas podem ser recuperadas com:

```bash
hermes curator restore <name>
```

Se alguma habilidade desaparecer, verifique primeiro o diretório de arquivamento.

## O que Pular

---

**Página 43**

---

Nem todo recurso do Hermes merece seu tempo no primeiro dia.

Alguns recursos são realmente úteis para casos de uso avançados. Outros são distrações até que você tenha uma necessidade específica.

### Servidores MCP

Pule servidores MCP até precisar de uma integração específica.

As ferramentas principais já cobrem:

- Web;
- Terminal;
- Arquivos;
- Navegador;
- Mídia.

MCP adiciona integrações de nicho, como:

- Jira;
- Notion;
- Bancos de dados;
- Outros serviços especializados.

Adicione servidores MCP quando tiver um fluxo de trabalho que realmente exija essas integrações, não antes.

### Roteamento entre Múltiplos Provedores

Pule o roteamento entre múltiplos provedores até que seu modelo primário comece a falhar regularmente.

Um único provedor com um único modelo pode funcionar bem durante meses.

Adicione provedores alternativos quando limites de taxa, indisponibilidade ou erros recorrentes se tornarem um problema real.

### Processamento em Lote

Pule o processamento em lote, a menos que esteja:

- Treinando modelos;
- Fazendo pesquisa em escala;
- Processando grandes volumes de tarefas semelhantes.

O loop padrão do agente lida bem com tarefas individuais.

O processamento em lote adiciona complexidade para obter uma vazão de que a maioria dos operadores não precisa.

### Plugins Personalizados

Pule o desenvolvimento de plugins personalizados até que as ferramentas integradas não cubram seu caso de uso.

O sistema de plugins é poderoso, mas também exige mais trabalho do que escrever uma habilidade.

Habilidades cobrem a maioria das necessidades procedimentais sem exigir código.

### Integração ACP com Editores

Pule a integração do editor ACP, a menos que você trabalhe constantemente no VS Code ou no Zed e queira utilizar o Hermes dentro do editor.

A CLI e o gateway cobrem a mesma funcionalidade.

## Quando Não Usar o Hermes

Hermes é um agente com uma superfície de ferramentas. Não é a ferramenta certa para todas as tarefas.

### Geração de Uso Único

Não use Hermes para geração de uso único.

Se você precisa de uma geração de texto avulsa, como:

- Escrever um e-mail;
- Redigir um tweet;
- Explicar um conceito;
- Reformular um parágrafo.

Uma chamada direta a um LLM é mais rápida e mais barata.

O loop do agente adiciona uma sobrecarga desnecessária para uma tarefa que exige apenas uma inferência.

### Tarefas que Não Precisam de Ferramentas

Não use Hermes para tarefas que não precisam de ferramentas.

Por exemplo:

> “Traduza este parágrafo.”

Essa tarefa não precisa de:

- Busca na web;
- Acesso ao terminal;
- Ferramentas de arquivo;
- Navegador;
- Delegação.

Um chat do ChatGPT ou Claude realiza esse trabalho em menos tempo e com menos sobrecarga.

### Saída Determinística

Não use Hermes quando precisar de uma saída determinística.

O loop do agente é inerentemente não determinístico.

O modelo pode escolher:

- Ferramentas diferentes;
- Ordens diferentes;
- Estratégias diferentes;
- Abordagens diferentes em cada execução.

Para fluxos de trabalho que precisam de resultados reproduzíveis, crie scripts que chamem as ferramentas diretamente em vez de roteá-las pelo agente.

### Aplicações Sensíveis à Latência

Não use Hermes quando a latência for mais importante que a capacidade.

Cada chamada de ferramenta adiciona tempo de ida e volta.

Uma chamada direta de API pode retornar em centenas de milissegundos.

Uma chamada de ferramenta do Hermes adiciona:

1. Tempo de raciocínio do modelo;
2. Tempo de seleção da ferramenta;
3. Tempo de execução da ferramenta;
4. Tempo de interpretação do resultado;
5. Possíveis chamadas adicionais.

Para aplicações em tempo real, chame a API diretamente.

## O que se Acumula

Os onze artigos anteriores descrevem um sistema que melhora ao longo do tempo:

- A arquitetura do loop do agente;
- A persistência de sessões;
- A memória e as habilidades;
- A superfície de ferramentas;
- O cron;
- Os gateways;
- A delegação;
- A automação de navegador;
- O Kanban;
- Os perfis.

---

**Página 44**

---

Nada disso funciona se a base estiver errada.

- Sessões que não persistem quebram o ciclo de aprendizado.
- Chaves de API ausentes quebram a superfície de ferramentas.
- Tokens de gateway no perfil errado quebram a entrega de mensagens.
- O curador arquivando uma habilidade que você escreveu quebra uma automação da qual você dependia.

O loop do agente é o motor.

O sistema de aprendizado é o combustível.

As ferramentas são a saída.

Os perfis são o ambiente de execução.

E os limites são o que mantém tudo real.

## Ações Recomendadas

Execute nesta semana:

```bash
hermes tools
```

Verifique se toda a sua superfície de ferramentas está disponível.

Fixe as habilidades que você escreveu:

```bash
hermes curator pin <name>
```

Teste a persistência das sessões:

```bash
hermes -c
```

Antes de adicionar mais recursos, confirme que a base está funcionando.