# Parte 06: Cron torna Hermes infraestrutura

Até aqui, toda a masterclass partiu do mesmo ponto: você chama, o Hermes responde.

Cron quebra essa lógica.

Com trabalhos agendados, o Hermes passa a agir sozinho em horários definidos. Isso muda o sistema de categoria. Ele deixa de ser apenas reativo e começa a funcionar como infraestrutura.

## Como o sistema cron funciona

A ideia é simples: existe um agendador verificando quando cada trabalho deve rodar. Quando chega a hora, o Hermes cria uma execução nova, com contexto próprio, e segue o prompt configurado para aquele job.

O detalhe importante é este: cada execução acontece em uma sessão fresca. Ou seja, cron não “continua a conversa atual”. Ele roda uma tarefa autônoma.

Isso obriga o prompt do job a ser mais completo e autocontido.

## Armazenamento dos trabalhos e resultados

Os trabalhos precisam existir em algum lugar estável, junto com seus metadados, horários e histórico de saída. Essa persistência é o que permite pausar, retomar, inspecionar e entender o que foi executado.

Se você usa cron de verdade, esse histórico deixa de ser detalhe e vira ferramenta de operação.

## Criação de trabalhos em linguagem natural

Uma das partes mais agradáveis do sistema é não precisar pensar primeiro em JSON, YAML ou cron cru.

Você pode pedir algo como:

- resumir um feed todo dia às 9h;
- verificar um site a cada 30 minutos;
- mandar um relatório em determinados horários;
- executar um script silencioso e só avisar quando houver mudança.

O Hermes traduz isso para a estrutura de job.

## Formatos de agendamento

O sistema aceita mais de um jeito de pensar tempo.

### Atrasos relativos

Bom para “daqui a 30 minutos” ou “a cada 2 horas”.

### Intervalos recorrentes

Úteis quando você quer um padrão simples e legível, sem entrar em cron expression.

### Expressões cron

Entram quando você quer precisão e já pensa nesse formato.

### Timestamps ISO

Ótimos para execuções pontuais e programadas no relógio.

## Gerenciamento do ciclo de vida

Job bom não é só job criado. Ele precisa poder ser:

- listado;
- atualizado;
- pausado;
- retomado;
- removido;
- executado manualmente para teste.

Essas operações parecem administrativas, mas são o que torna o cron confiável no dia a dia.

## Skills anexadas a trabalhos cron

Cron fica muito mais útil quando pode carregar procedimento junto com o prompt.

### Uma única skill

Ótimo para tarefas focadas, como um fluxo de checagem bem definido.

### Múltiplas skills

Entra quando o trabalho mistura mais de um procedimento, por exemplo coleta + formatação + entrega.

### Disponibilidade das skills

Aqui vale o mesmo princípio do resto do Hermes: o job precisa ser autossuficiente. Não adianta depender de contexto da conversa atual, porque ele não estará lá na execução futura.

## Entrega dos resultados

Executar é só metade do trabalho. O outro pedaço é entregar o que saiu dali.

### Entrega para a origem

Em plataformas com canal conectado, isso pode significar responder no mesmo lugar em que o job foi criado. No CLI local, porém, não existe essa conveniência automática. É importante não confundir os dois cenários.

### Entrega local

Boa quando você só quer guardar a saída para inspeção posterior.

### Distribuição para múltiplas plataformas

Faz sentido quando o Hermes já está integrado a canais de mensagem e o job precisa chegar em pessoas ou lugares diferentes.

## Entregas continuáveis

Alguns jobs são “fire and forget”. Outros pedem continuidade.

Relatórios diários, briefings e lembretes costumam funcionar melhor quando podem virar conversa depois da entrega. Essa capacidade faz diferença porque aproxima automação e colaboração, em vez de tratá-las como coisas separadas.

## Supressão silenciosa

Em jobs de vigilância, silêncio é comportamento esperado. Se nada mudou, nada precisa ser entregue.

Isso evita ruído e mantém o sistema útil. Alerta demais vira papel de parede.

## Trabalhos sem agente

Nem todo agendamento precisa passar pelo LLM.

Às vezes um script já produz exatamente a saída desejada.

### Comportamento do `stdout`

Nesses casos, o stdout pode virar a própria mensagem final. Se vier vazio, o job fica silencioso. Se vier conteúdo, ele é entregue. Se falhar, o erro precisa aparecer.

Esse modo é ótimo para watchdogs, checks simples e integrações muito mecânicas.

### Configuração em linguagem natural

Mesmo quando existe script por baixo, continuar pedindo a automação em linguagem natural ajuda bastante. Você mantém a ergonomia sem perder a precisão operacional.

## Encadeamento de trabalhos com `context_from`

Uma ideia poderosa é separar coleta e análise.

Um job pode buscar dados ou gerar uma saída bruta. Outro job, depois, usa esse resultado como contexto e produz algo mais útil para humanos. Isso reduz custo, melhora organização e cria pequenos pipelines em cima do Hermes.

## Gate `wakeAgent`

Em ambientes que hibernam, não basta ter job agendado. O backend precisa acordar quando necessário. Esse detalhe é fácil de esquecer e costuma ser o tipo de problema que só aparece na madrugada, quando ninguém quer depurar nada.

## Escrevendo prompts para sessões isoladas

Aqui mora um erro bem comum.

### Prompt inadequado

"Continue aquele trabalho de ontem".

Para um cron, isso é ruim porque ele não tem acesso implícito à sua conversa anterior.

### Prompt adequado

Um bom prompt de cron inclui:

- objetivo claro;
- fonte de dados, se existir;
- formato de saída esperado;
- regras de entrega;
- critérios do que fazer quando não houver novidade.

Quanto menos o job precisar adivinhar, mais confiável ele será.

## De ferramenta reativa para infraestrutura proativa

Esse é o ponto central.

Quando o Hermes passa a executar tarefas sozinho, em agenda, com contexto novo e entrega configurada, ele deixa de ser apenas algo que responde quando você lembra de abrir.

Ele passa a ocupar um lugar mais sério na sua operação.

Cron é justamente a peça que faz essa transição acontecer.