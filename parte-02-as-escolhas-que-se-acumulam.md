# Parte 02: As Escolhas Que se Acumulam

A maioria das pessoas passa sua primeira hora com o Hermes escolhendo um provedor e executando o instalador.

Ambas as coisas importam. Nenhuma delas, porém, é a decisão que continuará importando daqui a um mês.

As escolhas que se acumulam são três:

- **Onde seu agente fica?**
- **Ele persiste sessões entre reinicializações?**
- **Ele realmente consegue usar suas ferramentas?**

Acertar isso no primeiro dia torna tudo depois mais fácil.

Errar isso significa passar o próximo mês se perguntando por que o ciclo de aprendizado nunca parece entrar em ação.

## Onde o agente fica?

O Hermes é executado em seis backends de terminal:

- Local
- Docker
- SSH
- Daytona
- Modal
- Singularity

Cada um muda a forma como você interage com o agente.

### Ambiente local

A área de trabalho local é o ponto de partida mais comum.

O Hermes Desktop é instalado como um aplicativo nativo no macOS e no Windows, gerencia sua própria configuração, sessões e ambiente Python, e simplesmente funciona.

As sessões persistem no SQLite. A memória é gravada no disco. O agente está sempre disponível quando sua máquina está ligada.

A contrapartida é que seu agente morre com seu laptop.

Feche a tampa e o Hermes fica em silêncio.

### Docker

Docker é para pessoas que querem que o agente sobreviva a reinicializações e trocas de laptop.

A imagem oficial mapeia:

```text
~/.hermes
```

para:

```text
/opt/data
```

dentro do contêiner.

Esse único volume contém:

- Configuração
- Sessões
- Habilidades
- Memórias
- Credenciais do gateway

Você pode baixar uma nova imagem, destruir o contêiner e iniciar outro usando o mesmo diretório de dados sem perder nada.

A configuração Docker executa o Hermes como um serviço de gateway supervisionado.

O agente vive no seu servidor, não no seu laptop.

Esse é o modelo certo quando você quer que ele converse com Telegram ou Discord 24 horas por dia, sete dias por semana.

### Daytona e Modal

Daytona e Modal são as opções sem servidor.

Seu ambiente hiberna quando está ocioso e quase não custa nada.

O ciclo é simples:

1. Inicie o ambiente.
2. Execute o trabalho.
3. Coloque o ambiente em hibernação.

Essas opções são ótimas para trabalhos em lote e cargas de trabalho de pesquisa.

Não são ideais para um agente conversacional que precisa estar acessível pelo Telegram a qualquer hora.

### SSH e Singularity

SSH e Singularity preenchem lacunas específicas.

O SSH é a escolha certa quando você tem uma máquina remota potente e quer que o agente execute comandos nela.

O Singularity atende a ambientes de HPC e computação científica.

## Padrão recomendado de adoção

O padrão que vi funcionar para a maioria das pessoas é:

1. Começar localmente para aprender o sistema.
2. Adicionar Docker ou um backend de nuvem quando quiser que o agente funcione de modo independente.
3. Conectar gateways para poder acessá-lo de qualquer lugar.

Onde o agente vai executar determina o quanto o Hermes estará disponível para você.

---

## Persistência de sessões

Toda conversa com o Hermes é salva como uma sessão.

As sessões são armazenadas em um banco de dados SQLite localizado em:

```text
~/.hermes/state.db
```

As sessões:

- Recebem títulos.
- Possuem busca de texto completo por meio do FTS5.
- Acompanham a linhagem entre eventos de compressão.
- Podem ser pesquisadas pelo agente.
- Podem ser retomadas.
- Podem ser transferidas entre plataformas.

Essa é a infraestrutura que faz o ciclo de aprendizado funcionar.

Sem persistência de sessão, toda conversa começa do zero e o agente não tem como construir sobre trabalhos anteriores.

É aqui que a escolha de implantação importa.

### Persistência no ambiente local

Na área de trabalho local, as sessões persistem por padrão.

O banco de dados SQLite fica no seu diretório inicial e sobrevive a reinicializações.

Você pode se afastar de uma conversa, voltar uma semana depois e retomá-la de onde parou usando:

```bash
hermes -c
```

Também é possível retomar uma sessão pelo seu identificador.

### Persistência no Docker

No Docker, o banco de dados de sessão fica dentro do volume montado.

Desde que você monte:

```text
~/.hermes
```

em:

```text
/opt/data
```

as sessões sobreviverão a:

- Reinicializações do contêiner
- Atualizações da imagem
- Substituições completas do contêiner

Esse comportamento não é automático.

Se você esquecer a montagem do volume, o agente começará do zero todas as vezes, com um armazenamento de sessão vazio.

A documentação do Docker deixa isso claro, mas é o tipo de coisa que muitas pessoas só percebem quando o agente esquece a conversa do dia anterior.

### Persistência no Daytona e no Modal

Em backends sem servidor, como Daytona e Modal, o ambiente hiberna quando está ocioso.

O estado do agente hiberna junto com o ambiente.

As sessões são retomadas quando o ambiente desperta, mas o agente não fica acessível durante a hibernação.

Isso é adequado para trabalhos cron agendados.

Não é adequado quando você quer chamar o agente pelo Telegram e obter uma resposta em tempo real.

## Teste de persistência

A persistência de sessão é invisível quando funciona e catastrófica quando não funciona.

Teste sua configuração da seguinte maneira:

1. Inicie uma conversa.
2. Pare o agente.
3. Reinicie o agente.
4. Execute:

```bash
hermes -c
```

Se a conversa voltar, está tudo certo.

Se o agente começar do zero, corrija o armazenamento antes de construir qualquer coisa sobre ele.

---

## O erro que quase todo mundo comete no primeiro dia

Aqui está o erro que quase todo mundo comete no primeiro dia.

As pessoas fazem ao Hermes uma pergunta conversacional, como:

> Escreva um poema sobre IA.

ou:

> Explique computação quântica.

O agente responde, a conversa parece boa e elas assumem que tudo está funcionando.

Esse teste prova que o modelo funciona.

Ele não prova que o agente funciona.

---

## Testando a superfície de ferramentas

O que torna o Hermes diferente do ChatGPT é a superfície de ferramentas.

Se as ferramentas não estiverem conectadas, você pagou por um chatbot com etapas extras.

O teste que realmente importa é pedir ao agente para fazer algo que exija uma ferramenta.

Eu recomendaria um teste de fumaça dividido em três partes.

### 1. Teste da ferramenta de terminal

Peça para o agente executar um comando de terminal.

Por exemplo:

> Qual é o uso do meu disco?

ou:

> Mostre-me os últimos cinco arquivos modificados no meu projeto.

Se ele executar o comando e retornar uma saída real, a ferramenta de terminal está funcionando.

### 2. Teste da ferramenta de pesquisa na web

Peça para o agente pesquisar na web.

Por exemplo:

> Encontre as últimas notícias sobre um assunto atual e resuma-as em três itens.

Se ele realizar a pesquisa e retornar resultados ao vivo, a ferramenta da web está funcionando.

### 3. Teste da ferramenta de arquivos

Peça para o agente salvar algo em um arquivo.

Por exemplo:

> Escreva a data de hoje em um arquivo chamado `test.txt` e diga-me o caminho absoluto.

Se o arquivo aparecer no disco, as ferramentas de arquivo estão funcionando.

## Critério de aprovação

Se os três testes passarem, o agente está funcional.

Você pode seguir para:

- Memória
- Habilidades
- Cron
- Gateways
- Automações
- Demais recursos

Se algum teste falhar, corrija-o antes de fazer qualquer outra coisa.

Uma chave de API ausente ou um backend mal configurado poderá frustrá-lo por semanas se você não perceber o problema cedo.

---

## Orçamento de iterações

O orçamento de iterações existe por um motivo.

O padrão é de **90 turnos**.

Cada chamada de ferramenta conta como um turno.

Se sua primeira tarefa real exigir dez chamadas de ferramentas, isso representará dez dos seus 90 turnos.

Portanto, elabore o teste de fumaça para comprovar que a superfície de ferramentas funciona, não para esgotar o orçamento.

---

## Conclusão

As três escolhas são:

1. Morada de implantação
2. Persistência de sessão
3. Verificação das ferramentas

Elas não são a parte mais empolgante da configuração do Hermes.

São a parte mais importante.

Um agente local com sessões persistentes e ferramentas funcionais é uma plataforma sobre a qual você pode construir.

Cada habilidade adicionada, cada trabalho cron agendado e cada memória acumulada pelo agente passa a se apoiar em uma base estável.

Um agente que perde sessões ao reiniciar, não consegue acessar a web ou vive em um laptop que é fechado todas as noites nunca desenvolverá o efeito cumulativo.

Ele sempre parecerá apenas um chatbot.

Porque, sem essas três coisas se mantendo estáveis, é exatamente isso que ele é.

Acerte a base e o restante da série fará sentido.

Ignore essa base e tudo o que for descrito daqui em diante parecerá que quase funciona, mas nunca se encaixa completamente.

Sessões e ferramentas dão memória ao agente.

Mas memória sem estrutura é apenas uma pilha de fatos.

Se você perdeu a Parte 1, pode encontrá-la aqui.

A próxima parte desta Master Class do Hermes mostrará como o sistema de aprendizado realmente funciona: como memória, habilidades e contexto fazem as informações se acumularem de maneira útil, em vez de apenas formarem uma pilha de dados.

**Fique ligado!**
