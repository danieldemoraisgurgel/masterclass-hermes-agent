# Parte 09: Navegador e Computer Use
Todas as ferramentas desta série até agora foram baseadas em texto:

- O terminal executa comandos e retorna texto.
- As ferramentas de arquivo leem e escrevem texto.
- A busca na web retorna texto.
- Até mesmo o sistema de cron funciona com entrada e saída em texto.

A automação de Navegador e o Computer Use rompem esse padrão.

A ferramenta de navegador navega em sites reais, clica em botões, preenche formulários e interage com páginas web. A ferramenta de Computer Use controla uma área de trabalho real, clicando, digitando, rolando e arrastando em sistemas macOS, Windows e Linux.

O cursor do usuário não se move, o foco da janela não muda e o agente trabalha ao lado do usuário na mesma máquina.

Essas são as ferramentas que permitem ao Hermes operar em interfaces que nunca foram projetadas para APIs.

---

## Automação de navegador

O conjunto de ferramentas de navegador transforma o Hermes em um navegador web completo.

Ele pode:

- Navegar por páginas;
- Clicar em elementos;
- Digitar texto;
- Capturar screenshots;
- Executar JavaScript;
- Rolar páginas;
- Consultar erros no console do navegador.

O agente visualiza a página por meio de uma **árvore de acessibilidade**, ou seja, um instantâneo textual com IDs de referência numerados para cada elemento interativo.

Por exemplo:

- Clicar em `@e5` para pressionar um botão;
- Digitar em `@e3` para preencher um campo de busca;
- Ler o console para identificar erros de JavaScript.

---

## Backends disponíveis

Cinco backends atendem a diferentes casos de uso.

### Navegadorbase

O modo em nuvem do Navegadorbase oferece navegadores gerenciados com:

- Proxies residenciais;
- Resolução de CAPTCHA;
- Recursos adequados para sites que combatem automações e bots.

### Navegador Use

O Navegador Use funciona como um provedor de nuvem alternativo, com recursos próprios de automação e antidetecção.

### Firecrawl

O Firecrawl também atua como provedor de nuvem alternativo, oferecendo mecanismos próprios para navegação, extração e interação com páginas web.

### Chromium local via CDP

O modo Chromium local via **Chrome DevTools Protocol — CDP** conecta o Hermes a uma instalação existente do Chrome ou Brave.

### Chromium local padrão

O modo local padrão utiliza uma instalação do Chromium controlada pela CLI `agent-Navegador`.

---

## Roteamento híbrido

O recurso de roteamento híbrido merece destaque.

Quando um provedor de nuvem está configurado:

- URLs públicas são acessadas pelo navegador em nuvem;
- Endereços locais ou privados são automaticamente direcionados para um Chromium local.

Isso inclui endereços como:

```text
localhost
127.0.0.1
192.168.x.x
10.x.x.x
172.16.x.x
```

O servidor local de desenvolvimento nunca precisa trafegar pela rede do provedor de nuvem.

Na mesma conversa, o agente pode:

```text
Capturar uma screenshot de http://localhost:3000
```

e também:

```text
Extrair dados de https://github.com
```

Tudo isso sem trocar manualmente de provedor.

---

## Exemplo de preenchimento de formulário

O exemplo de formulário web da documentação demonstra bem essa capacidade.

O usuário solicita:

> Cadastre-se para criar uma conta.

O agente então:

1. Navega até a página de cadastro;
2. Captura um instantâneo da página;
3. Identifica os campos do formulário por meio dos IDs de referência;
4. Digita o e-mail e a senha nos campos corretos;
5. Clica no botão de criação da conta;
6. Captura um novo instantâneo;
7. Confirma se a operação foi concluída com sucesso.

Esse tipo de interação era impossível para agentes limitados exclusivamente a texto.

---

# Computer Use

O computer use estende o mesmo princípio para toda a área de trabalho.

O agente pode capturar qualquer janela visível como uma screenshot, adicionando sobreposições numeradas sobre os elementos interativos.

Ele consegue:

- Clicar em elementos;
- Digitar textos;
- Pressionar combinações de teclas;
- Rolar interfaces;
- Arrastar objetos;
- Interagir com aplicações nativas;
- Trabalhar com diálogos do sistema operacional.

O agente clica utilizando o índice de cada elemento, e não coordenadas fixas de pixel.

Esse método é consideravelmente mais confiável, especialmente quando:

- A janela muda de tamanho;
- A resolução da tela é alterada;
- Um elemento se desloca;
- A interface possui layouts responsivos.

---

## Execução em segundo plano

O modelo de execução em segundo plano é uma das principais decisões de design.

Quando o agente clica em alguma coisa:

- O cursor real do sistema operacional permanece no lugar;
- A janela utilizada pelo agente não é trazida para a frente;
- As áreas de trabalho virtuais não são alteradas;
- O foco atual do usuário é preservado.

O agente trabalha na mesma máquina enquanto o usuário continua executando suas próprias atividades.

Uma sobreposição colorida de cursor mostra onde o agente está atuando. Dessa forma, o usuário pode acompanhar suas ações sem perder o contexto da própria atividade.

---

## Cursores independentes por sessão

O cursor do agente possui escopo de sessão.

Cada sessão do Hermes e cada subagente recebem uma identidade própria de cursor.

Isso permite que múltiplos agentes trabalhem simultaneamente sem produzir comportamentos confusos, como movimentos concorrentes ou um cursor interferindo no outro.

---

## Compatibilidade com modelos

O Computer Use funciona com qualquer modelo capaz de utilizar ferramentas, incluindo:

- Claude;
- GPT;
- Gemini;
- Modelos abertos;
- Modelos executados em endpoints locais.

Não existe dependência de um esquema específico de fornecedor.

O conjunto de ferramentas utiliza **MCP por stdio** para se comunicar com o `cua-driver`.

O `cua-driver` é um driver de código aberto executado em segundo plano, responsável por lidar com a pilha de acessibilidade específica de cada plataforma.

---

# Navegador versus Computer Use

A automação de navegador e o Computer Use possuem capacidades semelhantes, mas atendem a propósitos diferentes.

## Automação de navegador

A automação de navegador opera dentro de uma instância isolada do Chromium.

Características:

- Fica isolada da máquina do usuário;
- Não consegue abrir acidentalmente aplicações externas;
- Não altera configurações do sistema operacional;
- Utiliza cookies, cache e fingerprint próprios;
- Limpa automaticamente a sessão após um período de inatividade;
- É indicada para tarefas exclusivamente web.

## Computer Use

O Computer Use opera diretamente sobre a área de trabalho real.

O agente pode:

- Abrir aplicativos;
- Interagir com diálogos nativos;
- Navegar por interfaces sem equivalente web;
- Consultar mensagens em aplicativos;
- Abrir documentos;
- Alterar configurações do sistema;
- Interagir com ferramentas empresariais legadas.

Exemplos:

- Localizar o e-mail mais recente da Stripe;
- Abrir um documento em um editor de texto;
- Configurar uma definição do sistema operacional;
- Interagir com uma aplicação sem API.

---

## Qual ferramenta utilizar

Para tarefas exclusivamente web, utilize o conjunto de ferramentas de navegador.

Ele tende a ser:

- Mais rápido;
- Mais barato;
- Mais isolado;
- Mais previsível;
- Independente de permissões específicas da plataforma.

Para tarefas nativas de área de trabalho, especialmente aquelas que envolvem aplicações sem interface web, utilize o Computer Use.

---

# Proteções de segurança

As duas ferramentas possuem múltiplas camadas de proteção.

## Segurança da automação de navegador

O conjunto de ferramentas de navegador possui menos preocupações relacionadas a comandos perigosos, pois as ações são executadas dentro de uma sessão isolada.

As principais limitações são:

- Não pode baixar arquivos diretamente pelo navegador;
- Depende da árvore de acessibilidade em vez de coordenadas de pixel;
- As sessões expiram conforme o plano do provedor;
- Alguns sites podem limitar ou bloquear automações.

## Segurança do Computer Use

O Computer Use possui um modelo de segurança mais abrangente.

Toda captura que apresenta um diálogo de permissão é sinalizada.

Algumas ações são rigidamente bloqueadas no nível da ferramenta, incluindo:

- Esvaziar a lixeira;
- Efetuar logout;
- Bloquear a tela;
- Forçar exclusões;
- Executar determinadas combinações perigosas de teclas;
- Digitar padrões perigosos de shell.

Combinações de teclas como a tecla Windows também podem ser filtradas.

O prompt de sistema do agente impede explicitamente que ele:

- Clique em diálogos de permissão;
- Digite senhas;
- Contorne mecanismos de segurança;
- Execute ações destrutivas sem aprovação.

---

## Proteção contra injeção de prompt pela interface

Screenshots são tratadas como dados, não como instruções.

O agente é orientado a não obedecer a diretivas incorporadas em imagens ou interfaces.

Essa abordagem reduz o risco de ataques de **prompt injection via interface gráfica**, nos quais uma página ou aplicação exibe instruções maliciosas tentando controlar o agente.

---

## Aprovação de ações

O bloqueio por aprovação permite que o usuário visualize uma ação antes de sua execução.

Na CLI, o usuário recebe um prompt interativo.

Em plataformas de mensagens, podem ser exibidos botões de aprovação.

A proteção também pode ser configurada em modo manual, exigindo confirmação para cada ação executada pelo agente.

---

# Consumo de tokens por screenshots

Screenshots possuem um custo elevado de contexto.

Uma única screenshot com resolução aproximada de:

```text
1568 × 900
```

pode consumir cerca de:

```text
1.500 tokens
```

Uma sessão com 20 ações e sem mecanismos de otimização poderia consumir rapidamente a janela de contexto do modelo.

---

## Otimizações aplicadas pelo Hermes

O Hermes utiliza quatro mecanismos principais de otimização.

### 1. Retenção das screenshots mais recentes

O adaptador mantém somente as três screenshots mais recentes no contexto.

As imagens mais antigas são substituídas por marcadores de texto.

### 2. Compressão de contexto

O compressor de contexto remove partes antigas de imagens presentes nos resultados das ferramentas.

### 3. Contabilização fixa de imagens

Cada imagem é contabilizada com uma taxa fixa de aproximadamente 1.500 tokens na Anthropic, independentemente do tamanho do conteúdo em Base64.

### 4. Limpeza no lado do servidor

Na Anthropic, resultados antigos de ferramentas são removidos no lado do servidor.

Com essas otimizações, uma sessão completa normalmente consome cerca de:

```text
30.000 tokens
```

em contexto de screenshots, em vez de aproximadamente:

```text
600.000 tokens
```

---

# Modo de árvore de acessibilidade

O modo de árvore de acessibilidade é uma alternativa adequada para modelos que trabalham somente com texto ou para cenários em que se deseja economizar tokens.

Nesse modo, o agente recebe a estrutura textual da interface, sem a screenshot.

Ele ainda pode:

- Navegar;
- Clicar;
- Digitar;
- Identificar campos;
- Acionar botões;
- Ler textos;
- Interagir com elementos acessíveis.

A principal limitação é que o agente não consegue visualizar o layout gráfico da interface.

---

# Conclusão

A automação de navegador permite que o agente interaja com praticamente qualquer site, inclusive aqueles que:

- Utilizam renderização JavaScript;
- Possuem formulários complexos;
- Dependem de interfaces dinâmicas;
- Tentam detectar ou bloquear bots.

O Computer Use permite que o agente interaja com praticamente qualquer aplicação na área de trabalho, inclusive aquelas que:

- Não possuem API;
- Não possuem versão web;
- Nunca foram projetadas para automação;
- Dependem de diálogos nativos;
- Utilizam interfaces gráficas legadas.

Ambas as ferramentas estendem o Hermes para ambientes que agentes exclusivamente textuais não conseguem alcançar.

> Se algo pode ser visto na tela, provavelmente o agente consegue interagir com esse elemento.

O agente já consegue ver, clicar, digitar e navegar. No entanto, gerenciar múltiplos agentes, cada um com suas próprias tarefas, sessões e contextos, exige uma camada adicional de coordenação.
