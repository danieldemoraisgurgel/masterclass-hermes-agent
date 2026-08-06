# Parte 02: As escolhas que se acumulam

Quando alguém instala o Hermes pela primeira vez, costuma gastar energia demais em coisas que parecem grandes e energia de menos nas coisas que realmente vão cobrar a conta depois.

Escolher o provedor importa. Rodar o instalador certo também.

Mas, passado o entusiasmo do primeiro dia, as decisões que continuam pesando são outras:

- onde o agente vive;
- se as sessões persistem de verdade;
- se ele consegue usar as ferramentas que você espera.

Se essa base estiver certa, o resto encaixa. Se estiver errada, o Hermes até parece funcionar, mas nunca engrena de verdade.

## Onde o agente fica?

O Hermes pode rodar em vários backends. Cada um muda a experiência prática de uso.

### Ambiente local

Para muita gente, o ambiente local é o melhor começo.

Você instala, conversa com o agente, testa as ferramentas e entende o sistema sem depender de infraestrutura extra. É o jeito mais rápido de pegar intimidade com o fluxo.

As vantagens são claras:

- setup simples;
- resposta imediata;
- fácil de depurar;
- sessões e memória gravadas na própria máquina.

A limitação também é clara: se seu laptop dorme, o agente dorme junto.

Isso é ótimo para aprendizado e uso pessoal. É ruim para qualquer cenário em que você queira disponibilidade contínua.

### Docker

Docker entra quando você quer estabilidade operacional.

Nesse modelo, o Hermes deixa de morar no seu laptop e passa a viver em um ambiente mais previsível, normalmente com um volume persistente mapeado para guardar:

- configuração;
- sessões;
- skills;
- memória;
- credenciais de integração.

A grande vantagem aqui é simples: você pode atualizar a imagem, recriar o contêiner e continuar de onde parou, desde que o diretório de dados esteja bem montado.

Se você pretende usar gateway, cron e acesso remoto com consistência, Docker costuma ser o primeiro salto realmente útil.

### Daytona e Modal

Daytona e Modal funcionam bem quando o objetivo é elasticidade, custo sob controle e execução sob demanda.

O ambiente sobe, faz o trabalho e pode hibernar depois. Isso é excelente para:

- tarefas em lote;
- pesquisas pontuais;
- automações que não exigem presença contínua;
- experimentos com backend remoto.

O que eles não resolvem tão bem é a ideia de “meu agente está sempre de pé esperando mensagem”. Se o ambiente dorme, o Hermes fica indisponível até acordar.

### SSH e Singularity

Essas opções atendem casos mais específicos.

SSH faz sentido quando você já tem uma máquina remota forte e quer executar o Hermes nela sem muita cerimônia.

Singularity aparece mais em ambientes de HPC, laboratório e computação científica, onde a forma de empacotar e isolar software é diferente do universo Docker tradicional.

## Padrão recomendado de adoção

Se eu fosse resumir uma rota sensata para a maioria das pessoas, seria esta:

1. comece localmente para entender o comportamento do agente;
2. valide memória, sessões e ferramentas;
3. mova para Docker ou um backend remoto quando quiser persistência e disponibilidade melhores;
4. só depois conecte gateways e automações mais sérias.

Pular direto para a configuração “mais completa” parece produtivo, mas costuma gerar uma pilha de problemas difíceis de diagnosticar.

## Persistência de sessões

Toda conversa útil com o Hermes precisa poder continuar depois.

As sessões normalmente vivem em um banco SQLite, em algo como:

```text
~/.hermes/state.db
```

É esse banco que permite:

- retomar conversas;
- pesquisar sessões antigas;
- manter linhagem após compressão;
- transformar histórico em contexto reutilizável.

Sem persistência, o Hermes vira um ótimo improvisador com amnésia.

### Persistência no ambiente local

No ambiente local, isso costuma vir pronto.

Você conversa hoje, fecha tudo, volta depois e retoma a sessão. É simples e funciona bem, desde que o diretório do perfil continue existindo e você não trate o ambiente como descartável.

### Persistência no Docker

No Docker, persistência depende de configuração correta.

Se o volume estiver bem montado, tudo bem. Se não estiver, cada reinício pode parecer um “novo Hermes” sem lembrança nenhuma do que aconteceu antes.

Esse é um erro comum porque o sistema pode até responder normalmente. O problema só aparece quando você tenta voltar para uma sessão antiga e descobre que ela sumiu.

### Persistência no Daytona e no Modal

Nesses ambientes, o estado pode sobreviver enquanto o workspace ou volume estiver preservado. O ponto não é se o backend é “bom” ou “ruim”, e sim se ele combina com sua expectativa de disponibilidade.

Se você quer cron, lote e execução periódica, ótimo.
Se quer um agente acessível a qualquer momento via mensagem, pense duas vezes.

## Teste de persistência

Persistência boa é invisível. Persistência ruim só aparece quando já atrapalhou.

Então vale testar cedo:

1. inicie uma conversa real;
2. feche o agente;
3. abra novamente;
4. rode:

```bash
hermes -c
```

Se a conversa reaparecer, ótimo.
Se não reaparecer, pare tudo e corrija isso antes de continuar.

## O erro que quase todo mundo comete no primeiro dia

Quase todo mundo faz um teste fácil demais.

Pergunta algo como:

> escreva um poema sobre IA

ou:

> me explique como funciona um banco vetorial

O agente responde. A pessoa conclui que está tudo certo.

Só que isso testa conversa, não operação.

Você ainda não sabe se o Hermes consegue usar terminal, ler arquivos, navegar, persistir sessão ou recuperar contexto. E essas são justamente as partes que diferenciam o sistema.

## Testando a superfície de ferramentas

Se você quer um teste honesto, ele precisa tocar a máquina.

### 1. Teste da ferramenta de terminal

Peça algo verificável, por exemplo:

> rode `pwd` e me diga em que diretório você está

ou:

> liste a versão do Python disponível

Se o agente conseguir executar e devolver o resultado real, o terminal está vivo.

### 2. Teste da ferramenta de pesquisa na web

Peça uma consulta factual e atual.

Se a ferramenta existir e estiver configurada, o Hermes deve conseguir buscar e citar o que encontrou, não apenas inventar uma resposta plausível.

### 3. Teste da ferramenta de arquivos

Crie um arquivo simples, leia de volta e confirme o conteúdo.

Esse teste parece banal, mas diz muito. Um agente que não consegue tocar no sistema de arquivos fica preso na camada de descrição.

## Critério de aprovação

Seu Hermes está realmente pronto quando consegue fazer três coisas ao mesmo tempo:

- manter sessão entre reinicializações;
- executar ferramentas que mexem no mundo real;
- voltar com o resultado certo, não com uma aproximação verbal.

Se um desses pontos falhar, a fundação ainda não está pronta.

## Orçamento de iterações

Outra escolha que se acumula é o orçamento de iterações.

Um agente com orçamento curto demais desiste cedo.
Um agente com orçamento longo demais pode gastar contexto e tokens em espirais bobas.

O ponto não é buscar o “máximo”. É achar um limite compatível com o tipo de trabalho que você espera: tarefas simples pedem menos; investigação, debug e automação longa pedem mais fôlego.

## Conclusão

Se você lembrar de uma coisa desta parte, que seja esta:

o Hermes não começa a ficar realmente útil quando responde bem. Ele começa a ficar útil quando persiste, executa e volta.

Sessões persistentes dão continuidade.
Ferramentas funcionais dão capacidade.
A escolha de ambiente dá sustentação.

É isso que faz o sistema se acumular de verdade.