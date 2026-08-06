# Parte 11: A camada de administração

Um agente isolado já consegue fazer bastante coisa.

Dois agentes começam a formar um sistema.
Três ou mais já pedem administração de verdade.

É aqui que entram perfis, dashboard e servidor de API.

Essas peças não são enfeite. Elas são o que permite operar o Hermes com menos improviso quando a estrutura cresce.

## Perfis são o limite de execução

Perfil é, na prática, a fronteira de isolamento do Hermes.

Cada perfil pode ter:

- configuração própria;
- memória própria;
- sessões próprias;
- skills próprias;
- cron próprio;
- gateways próprios;
- credenciais próprias.

Isso é valioso porque evita misturar contextos que não deveriam se contaminar.

### Criando perfis

Criar perfis separados faz sentido quando você quer dividir áreas de atuação, ambientes ou responsabilidades. Um perfil para programação, outro para pesquisa, outro para operação contínua: esse tipo de separação costuma melhorar segurança e organização.

### Aliases de comando

Aliases ajudam a tornar o uso diário menos chato e menos sujeito a erro. Se você troca de perfil o tempo todo, ergonomia conta bastante.

### Gateways por perfil

Esse ponto é importante. Se cada perfil pode ter seus próprios gateways, você evita um acoplamento perigoso entre contextos que deveriam ficar isolados.

### Isolamento do diretório home

Quanto melhor o isolamento do diretório e dos dados, menor a chance de um perfil enxergar ou afetar o que pertence a outro sem querer.

### Exportação e distribuição de perfis

Quando o perfil vira uma unidade bem definida, fica mais fácil mover, replicar ou compartilhar uma configuração operacional inteira.

## O dashboard é a superfície de controle

À medida que o sistema cresce, operar tudo só por linha de comando pode continuar possível, mas deixa de ser confortável.

O dashboard entra como visão de controle.

### Alternância entre perfis

Trocar de perfil visualmente ajuda bastante quando você está gerenciando ambientes diferentes ao longo do dia.

### Interface de chat

Ter chat ali não substitui o CLI. Ele complementa. Dependendo do momento, uma superfície visual agiliza inspeção, acompanhamento e navegação entre sessões.

## O servidor de API é o ponto de integração

Se o dashboard é a superfície humana, a API é a superfície programática.

Ela permite conectar o Hermes a aplicações, frontends, serviços internos e fluxos externos sem depender sempre da interface manual.

### Casos de uso

A API faz sentido quando você quer:

- acionar o Hermes por outro sistema;
- consultar estado;
- integrar com interfaces próprias;
- construir automações em volta do agente.

## O dashboard web é a superfície de controle administrativo

Mais do que conversar, o dashboard ajuda a administrar. Isso inclui visualizar perfis, acompanhar sessões, entender estado do sistema e navegar pela operação com menos atrito.

## O que muda com múltiplos perfis

Quando você sai de um único perfil e entra em vários, o Hermes muda de escala.

### Exemplo de especialização

#### Perfil de programação

Focado em repositório, terminal, testes e revisão de código.

#### Perfil de pesquisa

Voltado para busca, síntese, leitura e monitoramento de informação.

#### Perfil trabalhador Kanban

Especializado em executar tarefas do quadro com escopo delimitado.

### Coordenação entre perfis

A especialização ajuda, mas também cria um novo problema: coordenação. A partir daqui, não basta que cada perfil seja bom sozinho. Eles precisam cooperar sem se atropelar.

### Sobrecarga operacional

Esse é o preço da maturidade.

Mais perfis significam mais poder e mais organização, mas também mais pontos para configurar, observar e manter. Se a estrutura crescer sem disciplina, a operação começa a pesar.

## Conclusão

Perfis, dashboard e API formam a camada administrativa do Hermes.

Eles ajudam a separar contextos, centralizar controle e abrir integração com o resto do ambiente. Quando você está operando mais de um agente, essa camada deixa de ser opcional.

Ela é o que mantém o sistema utilizável conforme a complexidade sobe.