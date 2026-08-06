# Parte 03: O Sistema de Aprendizado

Os dois primeiros artigos abordaram o **motor** — o loop do agente — e o **chassi** — implantação e configuração.

O motor dispara. O chassi é estável.

Agora a pergunta é:

> Ele realmente melhora com o tempo?

Essa não é uma pergunta retórica.

A maioria das ferramentas de IA é lançada com um nível fixo de capacidade e permanece assim. O ChatGPT de hoje sabe aproximadamente o que sabia na semana passada. O Claude Code é o mesmo Claude Code que era no dia da instalação.

O fornecedor do modelo pode lançar uma atualização, mas a ferramenta em si não aprende com seu trabalho.

O **Agente Hermes aprende**.

Não porque o modelo treina com seus dados, mas por causa de três sistemas que trabalham juntos para acumular, formalizar e manter o que o agente aprende sobre:

- Você;
- Seus projetos;
- Seus fluxos de trabalho.

---

## Memória: um conjunto selecionado de fatos

A memória no Hermes não é um registro de tudo que aconteceu.

É um conjunto pequeno e selecionado de fatos que o agente mantém em contexto o tempo todo.

O sistema mantém dois arquivos:

- `MEMORY.md`: contém as anotações pessoais do agente, como fatos do ambiente, convenções do projeto, peculiaridades de ferramentas e trabalho concluído.
- `USER.md`: contém seu perfil, como preferências, estilo de comunicação, nível técnico e coisas que o irritam.

Juntos, eles são limitados a aproximadamente **1.300 tokens**.

Isso é intencionalmente restrito.

### Como a memória se comporta

O agente grava na memória automaticamente quando aprende algo duradouro.

Você o corrige sobre uma convenção?

Ele salva isso.

Ele descobre que seu projeto usa Go 1.22 e `sqlc`?

Ele salva isso.

Você pede que ele se lembre da sua programação de rotação de chave de API?

Ele salva isso.

A memória é um instantâneo congelado no início da sessão.

Tudo de que o agente se lembra é carregado em seu prompt de sistema como um bloco de texto, disponível imediatamente a partir da primeira mensagem.

---

## Persistência e atualização da memória

Alterações de memória realizadas no meio da sessão são persistidas no disco, mas não aparecem no prompt até a próxima sessão.

Isso mantém o prefixo do prompt estável para o cache do lado do provedor.

Quando a memória fica cheia — e, aos **2.200 caracteres para o `MEMORY.md`**, ela ficará — o agente não descarta entradas silenciosamente.

A ferramenta retorna um erro contendo:

- As entradas atuais;
- A contagem de uso.

O agente então consolida a memória:

1. Mescla entradas relacionadas em outras mais curtas;
2. Remove fatos obsoletos;
3. Abre espaço para as novas informações.

A mensagem de erro mostra exatamente o que está na memória, permitindo que o agente decida o que deve manter e o que deve remover.

A memória custa tokens em cada prompt. Por isso, o agente precisa decidir se um fato é importante o suficiente para armazenamento permanente.

---

## Memória versus busca de sessões

A busca de sessão não custa nada até que você execute uma consulta.

O agente usa a busca de sessão para recuperação de informações, por exemplo:

> O que aprendi sobre o projeto X na semana passada?

Na memória permanente, ele salva apenas aquilo que deveria estar sempre em contexto.

A distinção central é:

> **A memória armazena fatos. As habilidades armazenam procedimentos.**

---

## Habilidades: a memória procedimental do agente

Quando o agente resolve um problema novo em um fluxo de trabalho de múltiplas etapas, ele pode salvar a abordagem como uma habilidade.

Isso pode acontecer quando a tarefa envolve:

- Cinco ou mais chamadas de ferramentas;
- Troca significativa de mensagens;
- Uma correção feita pelo usuário;
- Um procedimento que provavelmente será reutilizado.

A habilidade se torna um arquivo Markdown armazenado em:

```text
~/.hermes/skills/
```

Esse arquivo contém:

- Nome;
- Descrição;
- Procedimento passo a passo;
- Seção de armadilhas;
- Etapas de verificação.

---

## Divulgação progressiva das habilidades

As habilidades usam divulgação progressiva para minimizar a sobrecarga de tokens.

No início da sessão, apenas o índice de habilidades é carregado.

Esse índice contém uma lista compacta de:

- Nomes;
- Descrições;
- Categorias.

O agente carrega o conteúdo completo de uma habilidade somente quando realmente precisa dela:

```text
skill_view(name)
```

Para carregar arquivos de apoio, como modelos ou scripts:

```text
skill_view(name, path)
```

A maioria das habilidades nunca é carregada na maioria das sessões.

O custo de tokens é, portanto, limitado principalmente ao índice.

---

## Combinação de habilidades

Várias habilidades podem ser combinadas em um único comando.

Exemplo:

```text
/github-pr-workflow /test-driven-development fix issue #123
```

Nesse caso, o Hermes carrega ambas as habilidades no contexto e segue os dois conjuntos de instruções durante a execução da mesma tarefa.

Para fluxos de trabalho repetitivos, pacotes de habilidades permitem agrupar várias habilidades sob um único comando de barra.

---

## Origem das habilidades

As habilidades vêm de três lugares.

### 1. Habilidades incluídas

Acompanham o Hermes e cobrem fluxos de trabalho comuns, como:

- Revisão de código;
- Gerenciamento de pull requests;
- Pesquisa.

### 2. Habilidades do Hub

São contribuídas pela comunidade e podem ser instaladas a partir do **Skills Hub**.

### 3. Habilidades criadas pelo agente

São habilidades que o próprio agente escreve.

Elas funcionam como a memória procedimental dos seus padrões específicos de trabalho.

---

# A Revisão em Segundo Plano é uma Infraestrutura Invisível

Memória e habilidades não apenas se acumulam passivamente.

Após cada turno de conversa, o Hermes cria uma bifurcação de revisão em segundo plano que examina o que aconteceu.

A bifurcação é executada como um agente de IA separado, usando seu próprio cache de prompt.

Ela nunca interfere na conversa ativa.

Esse agente de revisão procura elementos que valha a pena lembrar, como:

- Correções feitas por você;
- Fluxos de trabalho executados;
- Fatos sobre seu ambiente;
- Procedimentos que foram refinados.

Quando encontra algo relevante, ele pode propor:

- Uma gravação de memória;
- Um patch de habilidade.

Essa é a parte do sistema que faz o ciclo de aprendizado parecer mágico.

Você corrige o agente uma vez sobre como seu projeto está estruturado.

A revisão em segundo plano captura a correção e a salva na memória.

Na próxima sessão, o agente já possui esse fato em contexto desde a primeira mensagem.

Você não precisa informá-lo novamente.

---

## Aprovação das gravações

O mecanismo de aprovação permite controlar o que o Hermes aprende e persiste.

Com as seguintes configurações:

```yaml
memory.write_approval: true
skills.write_approval: true
```

Toda proposta gerada pela revisão em segundo plano é preparada, mas não confirmada automaticamente.

Você pode revisar as propostas de memória com:

```text
/memory pending
```

E revisar as propostas de habilidades com:

```text
/skills pending
```

Depois, você pode:

- Aprovar as propostas corretas;
- Rejeitar as propostas incorretas;
- Auditar exatamente o que o agente pretende aprender.

Essa funcionalidade atua como uma válvula de segurança para ambientes em que é necessário controlar e auditar o aprendizado do agente.

---

## Uso de um modelo mais barato para revisão

A revisão em segundo plano também pode ser executada por um modelo mais barato.

Por padrão, ela utiliza o modelo principal de chat.

Como a conversa já está aquecida no cache de prompt, essas operações podem funcionar como leituras de cache efetivamente gratuitas.

Caso você esteja usando um modelo caro, pode direcionar a revisão para um modelo menor, sem diferença significativa de qualidade.

Em benchmarks:

- A captura de memória foi idêntica;
- A captura de habilidades foi quase idêntica.

---

# O Curador Impede a Degradação das Habilidades

O curador funciona como o coletor de lixo das habilidades.

Ele é executado por um agendador:

- A cada sete dias, por padrão;
- Quando o agente está ocioso há pelo menos duas horas.

---

## Ciclo de vida das habilidades

A fase determinística move as habilidades por um ciclo de vida.

### Habilidade não utilizada por 30 dias

Torna-se obsoleta.

### Habilidade não utilizada por 90 dias

É arquivada em:

```text
~/.hermes/skills/.archive/
```

Nada é apagado permanentemente.

O arquivamento pode ser revertido com:

```bash
hermes curator restore <name>
```

---

## Consolidação opcional com LLM

A fase opcional de LLM executa uma revisão usando um modelo auxiliar.

O agente bifurcado examina a biblioteca de habilidades e pode:

- Identificar habilidades sobrepostas;
- Propor habilidades mais abrangentes;
- Consolidar habilidades muito específicas;
- Corrigir divergências entre procedimentos.

Essa fase fica desativada por padrão porque consome tokens.

Para habilitá-la permanentemente:

```yaml
curator.consolidate: true
```

Para executá-la sob demanda:

```bash
hermes curator run --consolidate
```

---

## Proteção de habilidades críticas

Habilidades críticas podem ser fixadas com:

```bash
hermes curator pin <name>
```

Fixar uma habilidade impede:

- Transições automáticas;
- Arquivamento;
- Exclusão.

Patches e edições ainda podem ser aplicados.

Isso significa que o agente pode melhorar o conteúdo de uma habilidade fixada ao longo do tempo, mas a habilidade em si nunca será removida automaticamente.

---

## Instantâneos e rollback

Antes de cada execução do curador, um instantâneo em formato `tar.gz` de todo o diretório de habilidades é salvo.

Qualquer execução pode ser revertida com:

```bash
hermes curator rollback
```

A própria reversão também é reversível.

---

# Como o Ciclo de Aprendizado Funciona

Memória e habilidades são injetadas no prompt de maneiras diferentes:

- A memória é carregada no nível volátil, congelada durante a sessão;
- As habilidades são carregadas no nível estável, sempre presentes como um índice.

O agente pode consultar ambas durante a execução.

Quando encontra um problema, ele:

1. Verifica a memória em busca de fatos relevantes;
2. Identifica as habilidades correspondentes;
3. Carrega o conteúdo completo das habilidades necessárias;
4. Executa o procedimento;
5. Chama as ferramentas necessárias;
6. Retorna os resultados para a conversa.

Depois disso, a revisão em segundo plano examina o turno e identifica o que funcionou.

Ela pode salvar:

- Uma nova entrada de memória para um fato descoberto;
- Um patch de habilidade para um procedimento refinado;
- Uma nova habilidade para um fluxo de trabalho reutilizável.

Ao longo de várias sessões, o ciclo se repete.

A memória é consolidada à medida que fica cheia.

As habilidades são curadas à medida que envelhecem.

O curador remove ou arquiva aquilo que deixou de ser útil.

Como resultado, o agente passa a trabalhar com menos entradas, porém com entradas melhores.

---

# Por Que a Persistência das Sessões Importa

É por isso que as decisões de implantação e sessão apresentadas na Parte 2 desta Master Class são importantes.

O ciclo de aprendizado só funciona se as sessões persistirem.

Quando cada conversa começa do zero, sem memória, habilidades ou contexto persistente, o agente não consegue acumular aprendizado.

Ele aprende durante a sessão, esquece após a reinicialização e nunca melhora de forma contínua.

---

# O Sistema de Aprendizado é o Produto

O sistema de aprendizado é o produto.

O loop do agente é o motor que o executa.

Sem o sistema de aprendizado, você tem apenas um chatbot com acesso a ferramentas.

Com ele, você tem algo que realmente melhora no seu trabalho, na sua máquina e ao longo do tempo.

> **A memória captura o quê.**

> **As habilidades codificam o como.**

Ambas permanecem estáticas até que você ou o agente as criem.