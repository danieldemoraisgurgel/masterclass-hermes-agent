# Parte 09: Navegador e Computer Use

Até aqui, quase tudo na masterclass operou em cima de texto.

Terminal devolve texto. Arquivo devolve texto. Busca devolve texto. Até cron, na maior parte do tempo, vive em prompts e saídas textuais.

Browser automation e Computer Use quebram esse limite.

De repente, o agente não lida só com comandos ou documentos. Ele passa a interagir com interfaces reais.

## Automação de navegador

A automação de navegador permite que o Hermes faça o tipo de coisa que normalmente exigiria um humano clicando:

- abrir páginas;
- navegar por fluxos web;
- preencher campos;
- clicar em botões;
- capturar o estado da tela;
- ler o DOM e a estrutura da página.

Isso é muito útil em sistemas que até têm interface web, mas não oferecem uma API boa ou não oferecem API nenhuma.

## Backends disponíveis

O backend exato pode mudar, mas a lógica geral é a mesma: dar ao agente uma forma controlada de operar o navegador.

### Navegadorbase

Entra como uma camada de automação mais estruturada para páginas web.

### Navegador Use

Ajuda quando a tarefa pede interação mais direta com o fluxo da página.

### Firecrawl

É útil em cenários de coleta, extração e leitura web mais focada em conteúdo.

### Chromium local via CDP

Ótimo quando você quer conectar o Hermes a um navegador real já rodando localmente, com mais visibilidade do estado da página.

### Chromium local padrão

Serve bem para automações locais em ambiente controlado, sem depender de uma conexão externa com navegador separado.

## Roteamento híbrido

Na prática, o Hermes pode escolher entre abordagens diferentes conforme o tipo de trabalho.

Às vezes faz mais sentido operar a página por estrutura semântica.
Às vezes é melhor clicar visualmente.
Às vezes basta extrair conteúdo.

Essa flexibilidade evita forçar um único modelo de automação em tudo.

## Exemplo de preenchimento de formulário

Esse é o caso mais fácil de imaginar.

O agente abre a página, encontra os campos, digita, clica no envio e depois verifica se a ação realmente aconteceu.

O ponto importante aqui não é o glamour do clique. É a verificação depois do clique. Sem isso, automação vira aposta.

# Computer Use

Se browser automation resolve páginas, Computer Use amplia isso para o desktop inteiro.

O Hermes pode operar uma área de trabalho real em macOS, Windows ou Linux, com captura da interface, leitura de elementos e ações como:

- clicar;
- digitar;
- rolar;
- arrastar;
- usar atalhos de teclado.

Isso abre a porta para aplicações que nunca foram pensadas para integração programática.

## Execução em segundo plano

Uma das melhores características aqui é a execução em segundo plano. O agente consegue agir sem roubar o seu cursor e sem tomar o foco da sua janela.

Ou seja: você continua usando a máquina enquanto ele trabalha.

## Cursores independentes por sessão

Essa separação visual e operacional ajuda bastante. Fica claro o que é ação do agente e o que é ação sua, sem virar uma guerra pelo mouse.

## Compatibilidade com modelos

Nem todo modelo lida com visão e interface do mesmo jeito. Por isso, a camada de automação precisa conversar bem com a capacidade visual disponível e com as ferramentas de captura e navegação do sistema.

# Navegador versus Computer Use

As duas coisas parecem próximas, mas não são iguais.

## Automação de navegador

É melhor quando o alvo é uma página web, com estrutura DOM acessível e fluxo relativamente previsível.

## Computer Use

É melhor quando o alvo está fora do browser ou quando a interface depende de elementos nativos, canvas, apps desktop, diálogos do sistema ou fluxos visuais sem API.

## Qual ferramenta utilizar

A pergunta prática é: o que exatamente você está tentando operar?

Se a tarefa cabe num navegador com estrutura legível, comece pelo navegador.
Se depende da tela como um todo, vá para Computer Use.

# Proteções de segurança

Esse tipo de poder precisa vir com cuidado redobrado.

## Segurança da automação de navegador

O agente não deveria sair confirmando compras, aceitando permissões sensíveis ou fazendo ações não solicitadas só porque um botão existe.

## Segurança do Computer Use

No desktop, o cuidado é ainda maior. Prompt de senha, caixa de pagamento, diálogo de privilégio e confirmação crítica não são coisas para clicar no automático.

## Proteção contra injeção de prompt pela interface

Toda interface pode carregar texto malicioso. O agente precisa seguir a instrução do usuário, não o que aparecer numa página tentando redirecionar o comportamento.

## Aprovação de ações

Aprovação continua sendo a linha de defesa quando a ação tem impacto relevante. Em automação visual, isso importa muito.

# Consumo de tokens por screenshots

A parte menos glamourosa, mas muito real, é custo.

Captura de tela consome contexto. Fluxos longos com muitas imagens podem inflar o orçamento rápido.

## Otimizações aplicadas pelo Hermes

Por isso entram várias otimizações.

### 1. Retenção das screenshots mais recentes

O sistema tenta manter o que ainda é útil para a próxima decisão, sem carregar um museu completo de telas antigas.

### 2. Compressão de contexto

Quando necessário, a sessão é compactada para não estourar a janela.

### 3. Contabilização fixa de imagens

Tratar imagens com custo previsível ajuda a manter o orçamento sob controle.

### 4. Limpeza no lado do servidor

Descartar o que não é mais necessário também faz parte da higiene operacional.

# Modo de árvore de acessibilidade

Quando existe uma árvore de acessibilidade rica, o Hermes ganha um mapa muito melhor da interface. Isso costuma deixar a automação mais robusta do que depender apenas de pixels.

# Conclusão

Browser automation e Computer Use expandem o alcance do Hermes para onde APIs não chegam.

Se algo está numa página, numa janela, num formulário ou numa interface legada, há uma boa chance de o agente conseguir interagir com aquilo.

Esse é um passo importante porque leva o Hermes para o território das interfaces reais, com toda a utilidade e toda a responsabilidade que isso traz.