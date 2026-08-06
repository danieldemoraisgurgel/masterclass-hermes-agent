# Parte 04: Habilidades como POPs executáveis

Uma skill do Hermes é, no fundo, uma ideia simples com consequências grandes: transformar um jeito de fazer as coisas em procedimento reutilizável.

E o formato disso é quase humilde.

Não tem SDK complexo. Não tem plugin empacotado. Não tem cerimônia demais. Na maior parte do tempo, uma skill é um arquivo Markdown com frontmatter e instruções claras.

Essa simplicidade é uma vantagem enorme. Ela reduz atrito para escrever, revisar, corrigir e evoluir procedimentos.

## A anatomia de uma boa habilidade

Uma boa skill não é um texto bonito. É um texto executável por um agente.

Isso muda o padrão de qualidade. O importante não é soar impressionante. É funcionar quando carregada no meio de uma tarefa real.

### Quando usar

A descrição de gatilho é a primeira parte crítica.

Ela precisa deixar claro em que situação a skill faz sentido. Se essa condição estiver vaga, o agente pode nunca carregar a skill ou carregar na hora errada.

Uma descrição boa costuma responder:

- qual problema essa skill resolve;
- quando ela deve ser usada;
- que tipo de tarefa ela acelera ou estabiliza.

### Procedimento

Aqui entra o coração da skill.

O procedimento precisa ser:

- ordenado;
- específico;
- verificável;
- curto o suficiente para ser seguido;
- completo o suficiente para não depender de adivinhação.

Se a etapa pede um comando, coloque o comando.
Se pede verificação, diga o que confirmar.
Se existe uma decisão importante, explicite o critério.

Skill boa reduz improviso.

### Armadilhas

Essa seção vale ouro.

Os maiores ganhos não costumam vir da parte feliz do fluxo, mas do que costuma quebrar:

- pré-requisito esquecido;
- sistema operacional com diferença importante;
- arquivo que vive em lugar não óbvio;
- comportamento que parece sucesso, mas não é.

Sem essa parte, a skill vira só um resumo otimista do processo.

### Verificação

Toda skill útil deveria terminar com alguma forma de checagem.

Não basta dizer “feito”. O agente precisa ter como confirmar se aquilo realmente funcionou. Em prática, isso pode ser:

- um comando de leitura;
- um teste;
- uma URL respondendo;
- um arquivo gerado;
- um diff aplicado corretamente.

Se não dá para verificar, a skill ainda está crua.

## A divulgação progressiva é o motivo de as skills escalarem

Um dos riscos de qualquer sistema baseado em procedimento é a biblioteca crescer demais e virar peso morto.

O Hermes evita isso carregando primeiro só o índice das skills. O conteúdo completo entra no contexto apenas quando necessário.

Esse desenho resolve dois problemas ao mesmo tempo:

- preserva contexto para o que importa agora;
- permite ter uma biblioteca rica sem pagar o custo total em toda sessão.

É por isso que dá para acumular bastante conhecimento sem transformar o prompt num depósito de instruções.

## Execução de várias skills

Outro ponto forte é a composição.

Na vida real, uma tarefa quase nunca é “só GitHub” ou “só testes” ou “só debug”. Normalmente ela mistura peças. O Hermes consegue carregar várias skills ao mesmo tempo e seguir instruções complementares.

Isso é importante porque permite montar workflows por combinação, em vez de tentar escrever uma skill monstruosa para cada cenário possível.

## Pacotes de skills

Quando certos conjuntos aparecem juntos o tempo todo, faz sentido empacotar a convenção.

Pacotes ajudam quando você já sabe que determinado contexto quase sempre pede a mesma combinação de procedimentos. Eles reduzem digitação, evitam esquecimento e deixam o uso mais fluido.

## Hub de skills

O hub amplia o sistema para além do que veio instalado por padrão.

Isso é útil por dois motivos:

- acelera adoção, porque você não precisa escrever tudo do zero;
- cria um ecossistema de práticas compartilhadas.

Mas vale um cuidado: skill de terceiros não deve ser tratada como verdade revelada. Leia, teste e ajuste para o seu contexto.

## O que faz as skills se acumularem

Uma skill boa faz três coisas ao mesmo tempo:

1. economiza contexto;
2. reduz retrabalho;
3. captura um jeito de operar que já provou valor.

No longo prazo, isso pesa muito.

Sem skill, você continua ensinando o agente repetidas vezes.
Com skill, você ensina uma vez, melhora quando necessário e reaproveita depois.

Esse é o momento em que o Hermes começa a sair do modo “assistente que ajuda” e entra no modo “sistema que aprendeu como você trabalha”.

A memória guarda fatos.
As skills guardam procedimentos.
Juntas, elas formam a base prática do acúmulo.