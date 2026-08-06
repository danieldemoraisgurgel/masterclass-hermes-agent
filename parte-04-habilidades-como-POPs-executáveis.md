# Parte 04: Habilidades como POPs Executáveis

Uma habilidade no Hermes é um arquivo Markdown. Só isso.

É um arquivo de texto simples com:

- um cabeçalho YAML;
- títulos;
- marcadores;
- etapas numeradas.

Nada o compila. Nada o transpila. O agente simplesmente lê o arquivo e segue as instruções.

Essa simplicidade é o ponto central. Você não precisa de:

- um SDK de plugins;
- um arquivo de manifesto;
- um pipeline de implantação.

Para criar uma habilidade, você precisa apenas de um editor de texto e de uma opinião clara sobre como determinado fluxo de trabalho deve ser executado.

O agente já entende esse modelo. Ele:

- escreve habilidades para si mesmo depois de concluir tarefas complexas;
- carrega as habilidades quando necessário;
- ignora as que não são relevantes;
- atualiza os procedimentos à medida que os fluxos evoluem.

Com o curador mantendo tudo organizado, a biblioteca não se degrada em uma pilha de relíquias semimúteis.

Este artigo explica como escrever habilidades que realmente funcionem — não como simples documentação, mas como procedimentos executáveis que o agente pode seguir sem orientação constante.

---

## A Anatomia de uma Boa Habilidade

Toda habilidade é armazenada em um arquivo Markdown no seguinte caminho:

```text
~/.hermes/skills/<category>/<name>/SKILL.md
```

O formato possui duas partes principais:

1. **Frontmatter**
2. **Corpo da habilidade**

---

### 1. Frontmatter

O frontmatter fornece metadados ao agente, como:

- nome;
- descrição de uma linha;
- versão;
- restrições opcionais de plataforma;
- tags;
- categoria.

Exemplo:

```yaml
---
name: deploy-runbook
description: Implanta e valida um serviço no ambiente de homologação.
version: 1.0.0
platforms:
  - linux
tags:
  - deploy
  - homologacao
  - infraestrutura
category: operations
---
```

A descrição é o campo mais importante.

É ela que aparece no índice de habilidades examinado pelo agente no início da sessão. Uma descrição fraca pode fazer com que o agente nunca perceba que deve carregar aquela habilidade.

Uma descrição específica é melhor do que uma descrição genérica.

**Descrição fraca:**

```text
Ajuda com infraestrutura.
```

**Descrição melhor:**

```text
Implanta um serviço no ambiente de homologação e verifica sua disponibilidade.
```

---

### 2. Corpo da habilidade

O corpo contém o procedimento que será executado.

A documentação recomenda quatro seções principais:

1. Quando usar;
2. Procedimento;
3. Armadilhas;
4. Verificação.

---

## Quando Usar

Esta seção define a condição de acionamento da habilidade.

É onde você informa ao agente quais solicitações devem carregar e executar aquele procedimento.

Seja específico.

**Menos adequado:**

> Quando o usuário precisar de ajuda com infraestrutura.

**Mais adequado:**

> Quando o usuário pedir para implantar um serviço.

Quanto mais clara for a condição, maior será a chance de o agente selecionar a habilidade corretamente.

---

## Procedimento

O procedimento deve conter etapas numeradas que o agente seguirá.

Cada etapa deve representar uma ação concreta.

Quando uma etapa exigir uma chamada de ferramenta, informe:

- qual ferramenta deve ser usada;
- quais argumentos devem ser fornecidos;
- quais resultados devem ser esperados.

Quando uma etapa exigir julgamento, explique como o agente deve decidir.

Exemplo:

```markdown
## Procedimento

1. Verifique se o repositório possui alterações não versionadas.

   Execute:

   ```bash
   git status --short
   ```

2. Caso existam alterações não relacionadas à implantação, interrompa o procedimento e informe o usuário.

3. Execute os testes automatizados:

   ```bash
   npm test
   ```

4. Prossiga com a implantação somente se todos os testes forem concluídos com sucesso.

5. Conecte-se ao servidor de homologação utilizando a porta `2222`:

   ```bash
   ssh -p 2222 deploy@homologacao.example.com
   ```

6. Execute o script de implantação:

   ```bash
   ./deploy.sh
   ```

---

## Armadilhas

Esta seção registra modos de falha conhecidos e explica como evitá-los.

É aqui que fica o conhecimento tribal da organização: detalhes que normalmente não aparecem na documentação genérica, mas que são essenciais para o procedimento funcionar.

Exemplos:

- O servidor de homologação usa a porta `2222`, e não a porta `22`.
- A API retorna o status HTTP `202` antes de o recurso estar realmente pronto.
- O script de implantação precisa ser executado pelo usuário `deploy`.
- O serviço pode demorar até alguns minutos para responder após a reinicialização.
- O ambiente de produção utiliza variáveis diferentes das usadas em homologação.

Exemplo em Markdown:

```markdown
## Armadilhas

- O servidor de homologação usa a porta `2222`, não a porta `22`.
- Não considere uma resposta HTTP `202` como conclusão da operação.
- Depois de receber `202`, consulte o endpoint de status até que o recurso esteja marcado como `ready`.
- Nunca execute o script de implantação como `root`.

---

## Verificação

Esta seção explica como confirmar que o procedimento funcionou.

A verificação pode incluir:

- um comando;
- uma URL;
- um arquivo;
- um log;
- uma consulta de API;
- um teste funcional.

Sem uma etapa de verificação, o agente pode concluir o procedimento sem saber se ele realmente teve êxito.

Exemplo:

```markdown
## Verificação

1. Verifique o estado do serviço:

   ```bash
   systemctl status minha-aplicacao
   ```

2. Confirme que o endpoint responde com HTTP `200`:

   ```bash
   curl --fail https://homologacao.example.com/health
   ```

3. Inspecione as últimas linhas do log:

   ```bash
   journalctl -u minha-aplicacao -n 50 --no-pager
   ```

4. Considere a implantação concluída somente quando:

   - o serviço estiver no estado `active`;
   - o endpoint retornar HTTP `200`;
   - não houver erros críticos nos logs.
```

---

## Exemplo de uma Habilidade Completa

```markdown
---
name: deploy-runbook
description: Implanta um serviço no ambiente de homologação e confirma que ele está operacional.
version: 1.0.0
platforms:
  - linux
tags:
  - deploy
  - homologacao
category: operations
---

# Deploy Runbook

## Quando Usar

Use esta habilidade quando o usuário solicitar a implantação de um serviço no ambiente de homologação.

Não use para produção, rollback ou criação de infraestrutura.

## Procedimento

1. Acesse o diretório do projeto.

2. Verifique o estado do repositório:

   ```bash
   git status --short
   ```

3. Interrompa o procedimento se existirem alterações locais não relacionadas à implantação.

4. Execute os testes:

   ```bash
   npm test
   ```

5. Prossiga somente se os testes forem concluídos com sucesso.

6. Conecte-se ao servidor de homologação:

   ```bash
   ssh -p 2222 deploy@homologacao.example.com
   ```

7. Execute o script:

   ```bash
   ./deploy.sh
   ```

8. Aguarde a conclusão do script antes de iniciar a verificação.

## Armadilhas

- O servidor usa a porta `2222`.
- Não execute o procedimento como `root`.
- A API pode retornar HTTP `202` antes de o recurso estar pronto.
- Não considere a implantação concluída apenas porque o script terminou sem erros.

## Verificação

1. Consulte o endpoint de saúde:

   ```bash
   curl --fail https://homologacao.example.com/health
   ```

2. Verifique o serviço:

   ```bash
   systemctl status minha-aplicacao
   ```

3. Consulte os logs:

   ```bash
   journalctl -u minha-aplicacao -n 50 --no-pager
   ```

4. Considere o procedimento concluído somente se:

   - o endpoint retornar HTTP `200`;
   - o serviço estiver `active`;
   - os logs não apresentarem erros críticos.
```

Quando a habilidade `deploy-runbook` é carregada, o agente lê o arquivo, segue as etapas em ordem, considera as armadilhas e verifica o resultado.

Ele não precisa ser conduzido manualmente pelo fluxo de trabalho todas as vezes.

---

# Divulgação Progressiva

A divulgação progressiva é o motivo pelo qual as skills conseguem escalar.

O agente não carrega todas as skills em todas as conversas. Isso consumiria tokens com fluxos de trabalho que talvez nunca fossem utilizados.

Em vez disso, o carregamento ocorre em níveis.

---

## Nível 1: Índice compacto

No início da sessão, o agente carrega um índice contendo:

- nome;
- descrição;
- categoria.

Uma biblioteca com dezenas de skills pode consumir aproximadamente 3.000 tokens nesse formato compacto.

O agente lê o índice e sabe o que está disponível sem precisar carregar o conteúdo completo de cada `SKILL.md`.

---

## Nível 2: Arquivo principal da skill

Quando uma solicitação corresponde à descrição de uma skill, o agente carrega o arquivo completo utilizando:

```text
skill_view(name)
```

Exemplo:

```text
skill_view("deploy-runbook")
```

Isso carrega o conteúdo do arquivo:

```text
~/.hermes/skills/operations/deploy-runbook/SKILL.md
```

---

## Nível 3: Arquivos de apoio

Quando uma skill faz referência a arquivos adicionais, eles são carregados sob demanda.

Esses arquivos podem incluir:

- modelos;
- scripts;
- documentos de referência;
- exemplos;
- configurações;
- checklists.

O carregamento é realizado com:

```text
skill_view(name, path)
```

Exemplo:

```text
skill_view("deploy-runbook", "scripts/deploy.sh")
```

Assim, o agente paga o custo de tokens somente quando realmente utiliza o conteúdo.

A maioria das skills nunca é carregada na maioria das sessões.

É dessa forma que o Hermes pode ser distribuído com dezenas de skills e ainda permanecer dentro de uma janela de contexto razoável.

---

# Executando Várias Skills

Várias skills podem ser executadas juntas em um único comando.

Exemplo:

```text
/github-pr-workflow /test-driven-development fix issue 123
```

Nesse caso, o agente carrega:

- `github-pr-workflow`;
- `test-driven-development`.

Depois, segue os dois conjuntos de instruções para corrigir a issue `123`.

Os comandos com barra posicionados mais à esquerda são analisados como invocações de skills.

A análise para no primeiro token que não corresponde ao nome de uma skill.

Isso evita que argumentos de caminho ou nomes de arquivos iniciados por `/` sejam interpretados incorretamente como skills.

---

# Pacotes de Skills

Pacotes agrupam skills relacionadas em um único atalho.

Por exemplo, um pacote chamado `backend-dev` poderia combinar:

- revisão de código;
- testes;
- fluxo de trabalho de pull request.

Ao executar:

```text
/backend-dev
```

o Hermes carrega as três skills de uma só vez.

Pacotes são apenas arquivos YAML que listam nomes de skills.

Exemplo:

```yaml
name: backend-dev
description: Carrega o fluxo padrão de desenvolvimento backend.
skills:
  - code-review
  - test-driven-development
  - github-pr-workflow
```

Os pacotes não substituem as skills individuais.

Eles funcionam como aliases para combinações utilizadas com frequência.

---

# Hub de Skills

O Hub de Skills é o local onde a comunidade publica e distribui habilidades.

Para navegar pelo catálogo:

```bash
hermes skills browse
```

Para pesquisar por palavra-chave:

```bash
hermes skills search <keyword>
```

Exemplo:

```bash
hermes skills search kubernetes
```

Para inspecionar uma skill antes de instalá-la:

```bash
hermes skills inspect <skill>
```

Para instalar:

```bash
hermes skills install <skill>
```

O hub pode consultar diferentes fontes, incluindo:

- skills opcionais oficiais do Hermes;
- o diretório `skills.sh` da Vercel;
- endpoints conhecidos de sites de documentação;
- repositórios GitHub diretos.

Cada instalação proveniente do hub passa por um scanner de segurança.

O scanner procura riscos como:

- exfiltração de dados;
- injeção de prompt;
- comandos destrutivos;
- comportamento suspeito.

---

# Compartilhamento Interno com Taps

Equipes podem publicar suas próprias skills internas utilizando taps.

Um tap pode apontar para um repositório GitHub contendo arquivos `SKILL.md`.

Para adicionar um tap:

```bash
hermes skills tap add <org/repo>
```

Exemplo:

```bash
hermes skills tap add minha-empresa/hermes-skills
```

Depois disso, os membros da equipe podem instalar skills individuais do repositório.

Esse modelo não exige:

- cadastro em um registro;
- servidor próprio;
- pipeline de publicação;
- infraestrutura adicional de distribuição.

---

# O Que Faz as Skills se Acumularem

O agente pode criar skills automaticamente depois de concluir tarefas complexas.

Quando ele resolve um problema novo que exige cinco ou mais chamadas de ferramenta, pode salvar o fluxo de trabalho como uma skill.

Na próxima vez, ele carrega essa skill e executa o procedimento com maior rapidez.

Em execuções futuras, ele também passa a saber que a habilidade existe sem precisar ser explicitamente lembrado.

Você também pode criar skills para procedimentos que já internalizou, como:

- runbooks de implantação;
- listas de verificação de resposta a incidentes;
- padrões de revisão de código;
- procedimentos de backup;
- fluxos de aprovação;
- processos de diagnóstico;
- rotinas de manutenção.

O agente não precisa reaprender esses procedimentos do zero. Ele os recebe na forma de um arquivo que pode ler e seguir.

---

# Curadoria da Biblioteca

O curador impede que a biblioteca cresça indefinidamente.

As skills são tratadas de acordo com seu tempo de inatividade:

- sem uso por 30 dias: tornam-se obsoletas;
- sem uso por 90 dias: são arquivadas.

O curador não exclui definitivamente as skills.

As habilidades arquivadas ficam armazenadas em:

```text
~/.hermes/skills/.archive/
```

Para restaurar uma skill:

```bash
hermes curator restore <name>
```

Exemplo:

```bash
hermes curator restore deploy-runbook
```

Skills consideradas críticas podem ser fixadas:

```bash
hermes curator pin <name>
```

Exemplo:

```bash
hermes curator pin incident-response
```

Uma skill fixada é removida da jurisdição normal do curador e não será arquivada automaticamente por falta de uso.

---

# O Efeito de Acumulação

O efeito de acumulação é simples:

Cada skill adicionada ou corrigida torna o agente mais rápido naquele fluxo de trabalho específico.

Não em tudo.

Naquela tarefa específica.

Uma biblioteca com 20 skills bem escritas, cobrindo fluxos realmente utilizados, pode tornar o agente muito mais útil do que uma biblioteca com 100 skills vagas que nunca são carregadas.

A qualidade da biblioteca depende de:

- descrições específicas;
- condições de acionamento claras;
- etapas concretas;
- armadilhas conhecidas;
- verificações objetivas;
- manutenção contínua.

---

# Skills como Camada de Formalização

Skills são a camada em que você molda diretamente como o agente se comporta.

Isso não ocorre por meio de uma configuração abstrata, mas por procedimentos escritos uma vez e executados repetidamente.

Essa é a diferença entre:

> dizer ao agente o que fazer todas as vezes;

e:

> ensiná-lo uma vez como determinado procedimento deve ser executado.

Skills são a camada de formalização.

A memória guarda os fatos.

As skills guardam os procedimentos.

Juntas, elas cobrem grande parte do que o agente precisa para ser útil.

Nenhuma delas, porém, é suficiente se o agente não conseguir acessar as ferramentas necessárias para executar o procedimento.
