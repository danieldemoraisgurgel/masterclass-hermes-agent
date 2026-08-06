# Parte 04: Habilidades como POPs Executáveis

Uma habilidade no Hermes é um arquivo Markdown. Só isso.

Um arquivo de texto simples com um cabeçalho YAML e um corpo composto de títulos, marcadores e etapas numeradas. Nada o compila. Nada o transpila. O agente o lê e segue as instruções.

Essa simplicidade é o ponto. Você não precisa de um SDK de plugin, um arquivo de manifesto ou um pipeline de implantação para criar uma habilidade. Você precisa de um editor de texto e de uma opinião sobre como um fluxo de trabalho deve ser executado.

O agente já sabe disso. Ele escreve habilidades para si mesmo depois de concluir tarefas complexas. Ele as carrega quando necessário e as ignora quando não. Ele as atualiza à medida que os fluxos de trabalho evoluem. E, com o curador mantendo tudo organizado, a biblioteca não se degrada em uma pilha de relíquias semimúteis.

Este artigo é sobre como escrever habilidades que realmente funcionem. Não como documentação, mas como procedimentos executáveis que o agente pode executar sem orientação constante.

## A Anatomia de uma Boa Habilidade

Toda habilidade é um arquivo Markdown localizado em:

```text
~/.hermes/skills/<category>/<name>/SKILL.md
```

O formato tem duas partes:

- O **frontmatter** fornece metadados ao agente: um nome, uma descrição de uma linha, uma versão, restrições opcionais de plataforma, tags e categoria.
- O **corpo** é onde o procedimento fica.

A descrição é o campo mais importante. É ela que aparece no índice de habilidades que o agente examina no início da sessão. Uma descrição fraca significa que o agente nunca sabe que deve carregar a habilidade.

O corpo é onde o procedimento fica. A documentação recomenda quatro seções:

### Quando Usar

Esta é a condição de acionamento. É onde você diz ao agente que tipo de solicitação deve acionar esta habilidade.

Seja específico.

> “Quando o usuário pedir para implantar um serviço” é melhor do que “Quando o usuário precisar de ajuda com infraestrutura.”

### Procedimento

Etapas numeradas que o agente segue.

Cada etapa deve ser uma ação concreta. Se uma etapa exige uma chamada de ferramenta, diga qual ferramenta e quais argumentos. Se uma etapa exige julgamento, diga como o agente deverá decidir.

### Armadilhas

Modos de falha conhecidos e como evitá-los.

É aqui que vive o conhecimento tribal.

Exemplos:

- “O servidor de homologação usa a porta 2222, não a 22.”
- “A API retorna um 202 antes de o recurso estar pronto.”

### Verificação

Como confirmar que o procedimento funcionou.

Pode ser:

- um comando para executar;
- uma URL para verificar;
- um arquivo para inspecionar.

Sem isso, o agente conclui o procedimento sem saber se teve êxito.

Aqui está um exemplo concreto de uma habilidade bem estruturada:

O agente lê este arquivo quando a habilidade `deploy-runbook` é carregada. Ele segue as etapas em ordem. Ele verifica as armadilhas antes de prosseguir. Ele verifica o resultado. Ele não precisa que você o conduza pelo fluxo de trabalho todas as vezes.

## A Divulgação Progressiva É o Motivo de as Skills Escalarem

O agente não carrega todas as skills em todas as conversas. Isso consumiria tokens em fluxos de trabalho que ele nunca usa.

No início da sessão, ele carrega um índice compacto:

- o nome;
- a descrição;
- a categoria de cada skill.

São cerca de 3.000 tokens para uma biblioteca de dezenas de skills. O agente lê o índice e sabe o que está disponível sem carregar o conteúdo completo.

Quando uma solicitação do usuário corresponde à descrição de uma skill, o agente chama:

```text
skill_view(name)
```

Isso carrega o arquivo `SKILL.md` completo.

Se a skill faz referência a arquivos de apoio, como modelos, scripts ou documentos de referência, eles são carregados sob demanda com:

```text
skill_view(name, path)
```

São três níveis. O agente paga o custo de tokens apenas quando realmente usa a skill. A maioria das skills nunca é carregada na maioria das sessões.

É assim que Hermes pode ser fornecido com dezenas de skills incluídas e ainda caber em uma janela de contexto razoável.

## Execução de Várias Skills

Várias skills podem ser executadas juntas em um único comando.

Digite:

```text
/github-pr-workflow /test-driven-development fix issue 123
```

O agente carrega ambos os arquivos de skill e segue os dois conjuntos de instruções.

Os comandos com barra mais à esquerda são analisados como invocações de skill. A análise para no primeiro token que não é um nome de skill. Assim, argumentos de caminho e nomes de arquivo que por acaso começam com `/` nunca são absorvidos.

## Pacotes de Skills

Pacotes de skills agrupam skills relacionadas em um único atalho.

Um pacote `backend-dev` poderia combinar as skills de:

- revisão de código;
- testes;
- fluxo de trabalho de PR.

Executar:

```text
/backend-dev
```

carrega as três de uma vez.

Pacotes são apenas arquivos YAML que listam nomes de skills. Eles não substituem skills individuais. Em vez disso, são aliases para combinações que você usa constantemente.

## Hub de Skills

O Hub de Skills é onde a comunidade publica skills.

Você navega com:

```bash
hermes skills browse
```

Pesquisa com:

```bash
hermes skills search <keyword>
```

Inspeciona antes de instalar com:

```bash
hermes skills inspect
```

E instala com:

```bash
hermes skills install
```

O hub cobre várias fontes:

- skills opcionais oficiais do Hermes;
- o diretório `skills.sh` da Vercel;
- endpoints conhecidos de sites de documentação;
- repositórios GitHub diretos.

Cada instalação do hub passa por um scanner de segurança que verifica:

- exfiltração de dados;
- injeção de prompt;
- comandos destrutivos.

Para equipes que desejam compartilhar skills internas, os taps permitem publicar um repositório GitHub de arquivos `SKILL.md`.

Os membros da equipe adicionam o tap com:

```bash
hermes skills tap add <org/repo>
```

Depois, instalam skills individuais a partir dele.

Sem cadastro em registro, sem servidor e sem pipeline.

## O Que Faz as Skills se Acumularem

O agente cria skills automaticamente após concluir tarefas complexas.

Quando resolve um problema novo com cinco ou mais chamadas de ferramenta, ele salva o fluxo de trabalho como uma skill. Na próxima vez, carrega essa skill e executa mais rápido. Na vez seguinte, lembra que a skill existe sem ser solicitado.

Você cria skills para os fluxos de trabalho que já internalizou:

- o runbook de implantação que você escreveu;
- a lista de verificação de resposta a incidentes;
- os padrões de revisão de código.

O agente não precisa aprender isso do zero. Ele os tem como um arquivo que lê e segue.

O curador impede que a biblioteca cresça para sempre.

- Skills não usadas por 30 dias tornam-se obsoletas.
- Sem uso por 90 dias, são arquivadas.

O curador nunca exclui. Skills arquivadas ficam em:

```text
~/.hermes/skills/.archive/
```

Elas podem ser recuperadas com:

```bash
hermes curator restore <name>
```

Skills críticas são fixadas com:

```bash
hermes curator pin
```

Assim, são removidas inteiramente da jurisdição do curador.

O efeito de acumulação é simples: cada skill que você adiciona ou corrige torna o agente mais rápido naquele fluxo de trabalho específico.

Não em tudo. Naquela única coisa.

Uma biblioteca de 20 skills bem escritas, cada uma cobrindo um fluxo de trabalho que você realmente executa, torna o agente dramaticamente mais útil do que uma biblioteca de 100 vagas que nunca são carregadas.

Skills são a camada em que você molda diretamente como o agente se comporta.

Não por meio de configuração, mas por meio de procedimentos que você escreve uma vez e o agente executa daí em diante.

Essa é a diferença entre dizer ao agente o que fazer todas as vezes e ensiná-lo o que fazer uma vez.

Skills são a camada de formalização.

A memória guarda os fatos. As skills guardam os procedimentos. Juntas, cobrem a maior parte do que o agente precisa para ser útil.

Nenhuma delas importa se o agente não consegue alcançar as ferramentas necessárias para executar o procedimento.
