# Parte 12: O que pular, o que quebra e como manter a sanidade

Depois de onze partes falando de arquitetura, capacidade e operação, vale fazer a conversa mais honesta.

Hermes é poderoso. Também é um sistema com arestas.

Saber o que usar, o que adiar e o que evitar é parte da maturidade de uso. Sem isso, a ferramenta deixa de parecer ampliação de capacidade e começa a parecer fonte de atrito.

## Todo modelo tem uma janela de contexto

Esse é um limite estrutural, não um detalhe cosmético.

Tudo disputa espaço dentro da janela:

- instruções de sistema;
- memória;
- índice de skills;
- conversa recente;
- resultados de ferramentas;
- screenshots e outros anexos.

Sessões longas, investigações grandes e fluxos com muita ferramenta queimam contexto rápido. A compressão ajuda, mas compressão não faz milagre. Resumo sempre perde nuance.

O jeito saudável de pensar nisso é simples: contexto é orçamento. Gaste com intenção.

## A fragilidade das integrações é constante

Quanto mais o Hermes conversa com o mundo externo, mais você passa a depender de peças que podem falhar por motivos fora do seu controle.

API muda. Página muda. Bot cai. Sessão expira. Permissão quebra. Binário some. Isso não é exceção. É o custo normal de operar integrações.

Por isso, vale assumir desde o começo que observabilidade e recuperação fazem parte do sistema.

## O isolamento de perfis tem armadilhas

Separar perfis é ótimo. Também traz alguns tropeços previsíveis.

### Isolamento do diretório pessoal

Se o perfil não enxerga o que deveria enxergar, várias automações simplesmente não funcionam. Se enxerga demais, você perde a proteção que queria.

### Conflitos de tokens do gateway

Tokens e credenciais podem se atropelar se a separação entre perfis estiver mal feita. Isso costuma dar uma dor de cabeça chata porque o erro aparece na superfície errada.

### Persistência de sessões

Perfil isolado sem persistência confiável gera um agente que parece especializado, mas esquece tudo a cada reinício.

## O curador pode morder

Curadoria de skills é ótima até o momento em que uma limpeza agressiva remove algo que ainda importava.

Isso não é argumento contra curadoria. É argumento a favor de fazer curadoria com critério, snapshot e rollback.

## O que pular

Nem tudo precisa entrar na sua operação logo de cara.

### Servidores MCP

Podem ser úteis, mas não precisam estar no dia zero. Se a base ainda está sendo validada, adicionar mais uma camada de integração só aumenta a superfície de falha.

### Roteamento entre múltiplos provedores

É poderoso, mas também adiciona complexidade cedo demais para muita gente. Primeiro faça um modelo e um provedor funcionarem bem.

### Processamento em lote

Vale muito em alguns cenários, mas não é o melhor ponto de partida se o fluxo principal ainda está instável.

### Plugins personalizados

Entram melhor quando você já sabe exatamente o que precisa estender. Antes disso, o risco é construir uma solução sofisticada para um problema ainda mal definido.

### Integração ACP com editores

Pode ser excelente depois. Não precisa ser prioridade antes de validar o núcleo operacional.

## Quando não usar o Hermes

Essa é uma pergunta boa, e pouca gente faz cedo o suficiente.

### Geração de uso único

Se tudo que você quer é um texto pontual sem contexto, sem ferramenta e sem continuidade, talvez um chat comum já resolva.

### Tarefas que não precisam de ferramentas

Nem todo problema pede um agente operacional. Às vezes o valor está só numa resposta rápida.

### Saída determinística

Se você precisa de comportamento totalmente fixo, talvez um script dedicado seja melhor do que um loop de agente.

### Aplicações sensíveis à latência

O Hermes pensa, usa ferramenta, verifica, volta. Isso é ótimo para qualidade e péssimo para cenários que exigem resposta instantânea e mínima latência.

## O que se acumula

Vale insistir no lado bom também.

Quando a base está sólida, algumas coisas realmente se acumulam:

- memória útil;
- skills boas;
- sessões pesquisáveis;
- automações bem definidas;
- integrações estáveis;
- divisão clara de perfis.

Esse acúmulo é o que faz o Hermes sair do estágio de experimento interessante e virar infraestrutura pessoal ou de equipe.

## Ações recomendadas

Se você quer manter a sanidade, comece por aqui:

1. valide persistência de sessão;
2. confirme que as ferramentas essenciais funcionam;
3. separe perfis quando houver motivo real;
4. introduza cron e gateways aos poucos;
5. trate integração externa como algo que vai quebrar algum dia;
6. faça snapshot do que for importante;
7. não transforme toda vitória pontual em complexidade permanente.

## Fechamento

A promessa do Hermes não é ser mágico. É ser acumulativo.

Mas acúmulo útil depende de base boa, escopo bem escolhido e alguma disciplina para não montar uma máquina mais complicada do que o problema pede.

Se você respeitar esses limites, o Hermes cresce com você.
Se ignorar, ele vira exatamente o tipo de sistema que parece poderoso e cansa rápido.

Saber a diferença entre esses dois caminhos é, no fundo, o que mantém a sanidade.