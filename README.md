# Hermes Masterclass

Um guia em português, direto ao ponto, para entender como o Hermes Agent funciona de verdade — não só como chat, mas como sistema operacional de agentes.

Este repositório organiza a masterclass em 12 partes independentes, mas pensadas para serem lidas em sequência. A ideia é sair da visão superficial de “um chatbot com ferramentas” e enxergar o Hermes como aquilo que ele realmente é: um loop de agente com memória, habilidades, ferramentas, automação, gateways, delegação, coordenação e administração.

Repositório original do projeto:
https://github.com/danieldemoraisgurgel/masterclass-hermes-agent

## Para quem este material é

Este conteúdo é especialmente útil para quem quer:

- entender a arquitetura real do Hermes Agent;
- usar Hermes além do terminal, com automação e execução contínua;
- criar agentes que acumulam contexto, memória e processos;
- estruturar fluxos com skills, cron, gateways e subagentes;
- operar múltiplos perfis com mais controle e menos improviso.

## O que você vai encontrar aqui

A masterclass cobre o Hermes em camadas.

1. Fundamentos do loop do agente
2. Decisões de infraestrutura e ambiente
3. Sistema de aprendizado contínuo
4. Skills como procedimentos executáveis
5. Ferramentas e toolsets
6. Cron como infraestrutura autônoma
7. Gateways de mensagens
8. Delegação e subagentes
9. Browser e Computer Use
10. Kanban como coordenação
11. Camada de administração
12. Limites, falhas e sanidade operacional

## Ordem de leitura recomendada

Se você está começando agora, esta é a sequência ideal:

### Bloco 1 — Como o Hermes pensa e opera
- [Parte 01: Como o Agente Hermes Realmente Processa o Trabalho](./parte-01-como-o-agente-hermes-processa-o-trabalho.md)
- [Parte 02: As Escolhas Que se Acumulam](./parte-02-as-escolhas-que-se-acumulam.md)
- [Parte 03: O Sistema de Aprendizado](./parte-03-o-sistema-de-aprendizado.md)

### Bloco 2 — Como o Hermes ganha capacidade
- [Parte 04: Habilidades como POPs Executáveis](./parte-04-habilidades-como-POPs-executáveis.md)
- [Parte 05: Ferramentas e Conjuntos de Ferramentas](./parte-05-ferramentas-e-conjuntos-de-ferramentas.md)
- [Parte 06: Cron Torna Hermes Infraestrutura](./parte-06-cron-torna-hermes-nfraestrutura.md)

### Bloco 3 — Como o Hermes sai do chat e vira ambiente
- [Parte 07: Gateways de Mensagens Criam o Ambiente Hermes](./parte-07-gateways-de-mensagens-tornam-hermes-ambiente.md)
- [Parte 08: Delegação e Subagentes](./parte-08-delegação-e-subagentes.md)
- [Parte 09: Navegador e Computer Use](./parte-09-browser-e-computer-use.md)

### Bloco 4 — Como operar o sistema em escala
- [Parte 10: Kanban como Modelo de Coordenação](./parte-10-kanban-como-modelo-de-coordenação.md)
- [Parte 11: A Camada de Administração](./parte-11-a-camada-de-administração.md)
- [Parte 12: O que Pular, o que Quebra e Como Manter a Sanidade](./parte-12-o-que-pular-o-que-quebra-e-como-manter-a-sanidade.md)

## Resumo de cada parte

### Parte 01 — Como o agente trabalha por dentro
Explica o loop interno do Hermes: montagem de prompt, despacho de ferramentas, avaliação dos resultados, persistência de contexto e retomada de sessão. É a melhor porta de entrada para entender por que o Hermes não se comporta como um chatbot tradicional.

### Parte 02 — As decisões que continuam importando
Mostra quais escolhas realmente se acumulam com o tempo: backend, persistência e acesso real a ferramentas. É o texto que ajuda a evitar uma instalação “funcional”, mas frágil.

### Parte 03 — Como o Hermes aprende
Apresenta o sistema de aprendizado do Hermes na prática: memória, sessões e habilidades. Em vez de “treinar o modelo”, o Hermes melhora formalizando o que aprende sobre você, seus projetos e seus fluxos.

### Parte 04 — Skills que o agente consegue executar
Mostra como escrever habilidades úteis de verdade: boas descrições, gatilhos corretos, etapas verificáveis e manutenção contínua. É leitura essencial para transformar know-how em procedimento reutilizável.

### Parte 05 — A superfície real de poder
Explora as ferramentas e os conjuntos de ferramentas que tornam o Hermes operacional: terminal, arquivos, web, navegador, imagem, voz, subagentes e mais. O ponto central aqui é simples: a conversa entra, mas o trabalho sai pelas ferramentas.

### Parte 06 — Quando o Hermes deixa de ser reativo
Explica como o cron transforma o Hermes em infraestrutura autônoma. Em vez de depender de uma mensagem sua, o agente passa a executar tarefas recorrentes com agenda, contexto próprio e entrega programada.

### Parte 07 — Quando o Hermes vira ambiente
Mostra como gateways de mensagens tiram o Hermes da sua máquina e colocam o agente no seu fluxo diário, via Telegram, WhatsApp, Discord, Slack, e-mail e outras superfícies.

### Parte 08 — Paralelismo com subagentes
Detalha como delegação resolve gargalos de tarefas seriais. O Hermes pode iniciar agentes filhos com contexto e ferramentas próprios, trabalhando em paralelo sem poluir o contexto do agente principal.

### Parte 09 — Interface gráfica sem API
Apresenta browser automation e computer use como formas de operar interfaces reais — páginas, botões, formulários e desktop — mesmo quando o sistema não foi feito para integração programática.

### Parte 10 — Coordenação com Kanban
Mostra o Kanban não apenas como quadro de tarefas, mas como mecanismo de coordenação entre agentes, sessões e perfis. É onde o trabalho deixa de depender da memória da conversa.

### Parte 11 — Administração de múltiplos agentes
Explica perfis, dashboard e API server como a camada de operação do Hermes. Fundamental para quem quer separar contextos, credenciais, memórias e rotinas sem virar refém de configuração manual.

### Parte 12 — O que quebra e o que vale a pena
Fecha a série com uma visão madura: limites de contexto, compressão, ruído de ferramentas, complexidade operacional e critérios práticos para decidir o que usar, o que evitar e como manter a sanidade.

## Como usar este repositório

Você pode usar este material de três formas:

1. Leitura sequencial, como uma trilha completa de masterclass.
2. Consulta pontual, indo direto ao tema que você precisa.
3. Base de estudo para montar sua própria operação com Hermes.

Se a sua intenção é implementar de verdade, a melhor abordagem é:

- ler as Partes 01 a 03 para construir fundamento;
- estudar as Partes 04 a 09 para ganhar capacidade operacional;
- usar as Partes 10 a 12 para coordenação, administração e limites reais.

## Sugestão de leitura por perfil

### Se você é iniciante no Hermes
Comece por 01, 02 e 03.

### Se você quer produtividade prática rápido
Priorize 04, 05, 06 e 07.

### Se você quer automação avançada
Vá direto para 08, 09, 10 e 11.

### Se você já apanhou de contexto, compressão ou complexidade
Leia a 12 cedo.

## Sobre o foco desta masterclass

Esta masterclass não trata o Hermes como “mais um app de IA”.
Ela trata o Hermes como uma arquitetura viva de trabalho assistido por agentes.

Por isso, o valor do material está menos em comandos isolados e mais em entender:

- como o sistema se acumula com o tempo;
- onde ficam os limites práticos;
- quais peças realmente compõem uma operação útil;
- como sair da conversa e entrar em execução.

## Licença

Este repositório inclui o arquivo [LICENSE](./LICENSE). Consulte o texto da licença para os detalhes de uso.

---

Se este material te ajudar, vale a pena começar pela Parte 01 e seguir em ordem até a Parte 12. A série foi construída para ir do entendimento do núcleo do Hermes até sua operação real em ambiente, automação e escala.
