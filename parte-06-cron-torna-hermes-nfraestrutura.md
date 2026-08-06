# Parte 06: Cron Torna Hermes Infraestrutura

Cada artigo desta série, até agora, foi sobre um sistema reativo:

1. Você envia uma mensagem.
2. O agente a processa.
3. Você recebe uma resposta.

O ciclo é iniciado por você.

**Cron quebra esse padrão.**

Um trabalho agendado é algo que o agente executa sem ser solicitado, em uma agenda definida por você, utilizando um contexto novo a cada execução e entregando os resultados para qualquer plataforma configurada.

O agente deixa de ser algo com que você conversa e passa a ser algo que executa.

---

## Como o sistema cron funciona

O sistema cron fica dentro do daemon do gateway.

A cada 60 segundos, o agendador é acionado. Ele:

1. Carrega todos os trabalhos armazenados em:

   ```text
   ~/.hermes/cron/jobs.json
   ```

2. Compara o campo `next_run_at` de cada trabalho com a hora atual.
3. Dispara todos os trabalhos vencidos.

Cada trabalho vencido recebe uma nova sessão `AIAgent`.

- Sem histórico de conversa.
- Sem memória de execuções anteriores.
- Uma folha em branco.

As skills anexadas são injetadas como contexto. O prompt é executado até a conclusão e a resposta final é entregue ao destino configurado.

Os metadados do trabalho também são atualizados no armazenamento, incluindo:

- Horário da última execução.
- Próximo horário agendado.

Um bloqueio de arquivo em:

```text
~/.hermes/cron/.tick.lock
```

impede que acionamentos sobrepostos executem duas vezes o mesmo lote.

Se um acionamento levar mais de 60 segundos, o próximo aguardará sua conclusão.

---

## Armazenamento dos trabalhos e resultados

Os trabalhos cron são armazenados como JSON.

A saída de cada execução é salva em:

```text
~/.hermes/cron/output/{job_id}/{timestamp}.md
```

Gravações atômicas de arquivo evitam que os dados dos trabalhos sejam corrompidos por uma gravação interrompida.

---

## Criação de trabalhos em linguagem natural

Você pode criar um trabalho cron usando linguagem natural:

> Todas as manhãs, às 9h, verifique o Hacker News em busca de histórias sobre IA e resuma-as no Telegram.

Uma única frase é suficiente.

O agente:

1. Analisa a intenção.
2. Cria o trabalho.
3. Agenda a próxima execução.
4. Dispara o trabalho no horário configurado.

---

## Formatos de agendamento

Os formatos de agenda cobrem diferentes tipos de execução.

### Atrasos relativos

Utilizados para tarefas únicas.

```text
30m
```

Executa uma vez, após 30 minutos.

```text
2h
```

Executa uma vez, após 2 horas.

### Intervalos recorrentes

Utilizados para tarefas que devem se repetir até serem removidas.

```text
every 30m
```

Executa a cada 30 minutos.

```text
every 2h
```

Executa a cada 2 horas.

```text
every 1d
```

Executa uma vez por dia.

### Expressões cron

Disponíveis para quem prefere a sintaxe cron tradicional.

```cron
0 9 * * 1-5
```

Executa às 9h, de segunda a sexta-feira.

```cron
0 */6 * * *
```

Executa a cada 6 horas.

### Timestamps ISO

Utilizados para tarefas únicas em uma data e hora exatas.

```text
2026-07-15T09:00:00
```

---

## Gerenciamento do ciclo de vida

O ciclo de vida completo é gerenciado por uma única ferramenta: `cronjob`.

As ações disponíveis incluem:

- Criar.
- Listar.
- Pausar.
- Retomar.
- Executar imediatamente.
- Editar.
- Remover.

Tudo isso pode ser feito por meio de uma mensagem de chat, sem necessidade de utilizar a CLI.

O agente também aceita nomes de trabalhos no lugar de identificadores hexadecimais.

Por exemplo:

> Pause o briefing matinal.

Isso elimina a necessidade de localizar e copiar o ID interno do trabalho.

---

## Skills anexadas a trabalhos cron

Um trabalho cron pode carregar uma ou mais skills antes de executar seu prompt.

Essa é a diferença entre uma tarefa agendada que improvisa um fluxo de trabalho e outra que executa um procedimento conhecido.

### Uma única skill

Uma única skill anexada ao trabalho cron funciona como um playbook reutilizável.

O arquivo `SKILL.md` da skill fornece o contexto operacional da tarefa, incluindo:

- Condições de disparo.
- Procedimento numerado.
- Armadilhas conhecidas.
- Etapas de verificação.

### Múltiplas skills

Várias skills podem ser carregadas em uma ordem definida.

Um trabalho de resumo matinal, por exemplo, poderia carregar as skills `blogwatcher` e `maps`:

> Verifique os feeds configurados e resuma qualquer novidade. Em seguida, procure eventos locais perto de mim.

O agente terá os contextos das duas skills disponíveis durante a execução.

### Disponibilidade das skills

Skills anexadas a trabalhos cron precisam estar disponíveis no perfil que executa o gateway.

Essa é uma consideração operacional importante.

Caso existam skills em um subperfil das quais os trabalhos cron dependem, elas deverão ser copiadas para o diretório de skills do perfil padrão.

Os trabalhos cron são executados no processo do gateway, e o gateway utiliza o catálogo de skills do perfil padrão.

---

## Entrega dos resultados

Um trabalho cron que executa, mas nunca informa seu resultado, é um trabalho que efetivamente não aconteceu.

O sistema de entrega oferece suporte a mais de 20 plataformas.

### Entrega para a origem

A opção mais simples é entregar o resultado de volta à plataforma em que o trabalho foi criado.

Esse é o comportamento padrão para plataformas de mensagens.

### Entrega local

Para trabalhos criados pela CLI, o destino `local` salva a saída em arquivos dentro de:

```text
~/.hermes/cron/output/
```

### Distribuição para múltiplas plataformas

Um único trabalho cron pode entregar resultados para várias plataformas simultaneamente.

```text
all
```

Envia para todos os canais domésticos conectados.

```text
telegram,discord
```

Envia exclusivamente para Telegram e Discord.

```text
origin,all
```

Envia para o chat de origem e também para todos os outros canais conectados.

O token `all` é resolvido no momento do disparo.

Isso significa que um trabalho criado antes da configuração do Discord passará a incluir o Discord automaticamente depois que o canal for conectado.

---

## Entregas continuáveis

A opção `continuable` transforma uma entrega cron em uma conversa.

Por padrão, cron opera no modelo **disparar e esquecer**:

- A mensagem é entregue.
- O usuário pode lê-la.
- Uma resposta posterior não possui, automaticamente, o contexto da execução.

Com:

```yaml
attach_to_session: true
```

a entrega é inserida em uma thread de sessão.

Quando o usuário responde, o agente já possui o resumo entregue no contexto da conversa.

Em plataformas com suporte a threads, como Telegram e Discord, cada entrega abre sua própria thread dedicada.

---

## Supressão silenciosa

O padrão de supressão silenciosa é especialmente útil para tarefas de monitoramento.

Se a resposta final do agente contiver:

```text
[SILENT]
```

a entrega será completamente suprimida.

A saída ainda será salva localmente para fins de auditoria, mas nenhuma mensagem será enviada.

Exemplo:

> Verifique se o nginx está em execução. Se tudo estiver saudável, responda apenas com `[SILENT]`. Caso contrário, informe o problema.

Esse padrão evita mensagens desnecessárias quando nenhuma intervenção é exigida.

---

## Trabalhos sem agente

Nem todo trabalho agendado precisa de uma LLM.

Algumas verificações clássicas de watchdog devem ser baratas, previsíveis e confiáveis, como:

- Verificação de uso de memória.
- Alerta de espaço em disco.
- Ping de heartbeat.
- Verificação de processos.
- Testes simples de disponibilidade.

O modo sem agente ignora completamente a LLM.

O agendador:

1. Executa o script na agenda configurada.
2. Captura o conteúdo de `stdout`.
3. Entrega o resultado diretamente.

Nesse modo, existem:

- Zero tokens consumidos.
- Zero chamadas ao provedor.
- Zero fallback de modelo.

### Comportamento do `stdout`

Se o script produzir um `stdout` não vazio, o conteúdo será entregue literalmente.

Se o `stdout` estiver vazio:

- Nenhuma mensagem será enviada.
- A execução não será registrada como falha.

Um código de saída diferente de zero ou um timeout dispara um alerta de erro.

Isso impede que um watchdog defeituoso falhe silenciosamente.

### Configuração em linguagem natural

O agente pode configurar esse tipo de trabalho para você.

Por exemplo:

> Avise-me no Telegram se a RAM estiver acima de 85%, verificando a cada 5 minutos.

O Hermes:

1. Escreve o script de verificação.
2. Cria o trabalho cron sem agente.
3. Configura a entrega.
4. Gerencia o ciclo de vida da tarefa.

Tudo isso sem a necessidade de utilizar a CLI.

---

## Encadeamento de trabalhos com `context_from`

Trabalhos cron são executados em sessões isoladas, sem memória de execuções anteriores.

Entretanto, em alguns fluxos, a saída de um trabalho é exatamente o que o próximo trabalho precisa.

O parâmetro `context_from` cria essa ligação automaticamente.

O Trabalho B recebe a saída mais recente do Trabalho A anexada como contexto em tempo de execução.

A cadeia pode ter qualquer tamanho.

Exemplo de pipeline:

1. Coletar dados.
2. Filtrar e classificar.
3. Formatar os resultados.
4. Entregar o relatório.

Cada trabalho executa em sua própria agenda e lê a saída mais recente do trabalho anterior.

---

## Gate `wakeAgent`

O gate `wakeAgent` permite que um script de pré-verificação decida, em tempo de execução, se a LLM deve ser invocada.

Um script pode consultar:

- Um feed.
- Uma API.
- Um banco de dados.
- Um diretório.
- Um endpoint.
- Um sistema de monitoramento.

Caso nada tenha mudado, o script pode emitir:

```json
{
  "wakeAgent": false
}
```

Nesse caso, o turno do agente é ignorado completamente.

Para consultas frequentes, executadas a cada 1 a 5 minutos, essa abordagem representa a diferença entre:

- Pagar por turnos de agente sem conteúdo novo.
- Não pagar por nenhuma chamada desnecessária.

---

## Escrevendo prompts para sessões isoladas

Trabalhos cron são executados em uma sessão de agente completamente nova.

- Sem histórico.
- Sem memória.
- Sem contexto de execuções anteriores.

Isso significa que o prompt precisa conter tudo de que o agente necessita.

### Prompt inadequado

> Verifique aquele problema no servidor.

Esse prompt falha porque o agente não sabe:

- Qual servidor deve ser verificado.
- Qual era o problema.
- Como se conectar.
- Qual serviço deve ser analisado.
- Qual resultado é esperado.

### Prompt adequado

> Conecte-se por SSH ao servidor `192.168.1.100` com o usuário `deploy`, verifique se o nginx está em execução usando `systemctl status nginx` e confirme que `https://example.com` retorna HTTP 200.

Esse prompt funciona porque fornece:

- Endereço do servidor.
- Protocolo de acesso.
- Usuário.
- Serviço a ser verificado.
- Comando de validação.
- Endpoint de teste.
- Critério de sucesso esperado.

Se a tarefa mudar, utilize a ação `edit` do cron para alterar o prompt.

Não é necessário excluir e recriar o trabalho.

---

## De ferramenta reativa para infraestrutura proativa

Antes do cron, Hermes é uma ferramenta utilizada sob demanda.

Você tem uma pergunta e a faz.

Você tem uma tarefa e a delega.

O agente é reativo por definição.

Depois do cron, Hermes passa a executar em seu nome.

Ele:

- Verifica coisas que você poderia esquecer de verificar.
- Resume informações que você talvez não percebesse.
- Monitora sistemas continuamente.
- Entrega resultados nos mesmos chats utilizados para outras interações.
- Executa procedimentos conhecidos por meio de skills.
- Reduz chamadas desnecessárias por meio de scripts e gates.

O agente se torna uma infraestrutura da qual você depende, e não apenas um parceiro de conversa.

A transição do comportamento reativo para o proativo é o que faz Hermes parecer menos um chatbot e mais um operador.

Cron é o mecanismo que impulsiona essa transformação.

Um trabalho cron executado no gateway é um agente sem ninguém com quem falar.

O próximo passo é dar a ele uma voz que possa ser acessada de qualquer lugar: gateways de mensagens que transformam Hermes de algo que você precisa parar para utilizar em algo que simplesmente está presente.