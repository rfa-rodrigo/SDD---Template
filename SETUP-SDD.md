# SETUP — estrutura de spec-driven development
Autor: Rodrigo Figueiredo Abdo

Instruções para o agente. Execute na raiz do projeto atual.

**Regras de execução:**

1. Copie o conteúdo dos blocos **literalmente**. Não parafraseie, não resuma, não traduza, não "melhore" o texto. Os placeholders entre `<>` ficam como estão — quem preenche é o usuário.
2. **Nunca sobrescreva** arquivo que já exista. Se existir, pule e registre.
3. Ao final, relate o que foi criado e o que foi pulado, e nada além disso.

---

## 1. Estrutura de pastas e Fluxo

Crie as pastas abaixo, se ainda não existirem. Não crie nenhum arquivo ainda — isso vem nas seções seguintes.

```
projeto/
├── .claude/
│   ├── skills/
│   └── agents/
├── .spec/
│   ├── specs/
│   │   └── archive/
│   ├── shared/
│   └── templates/
└── resources/
```

`resources/` é a raiz do código, é onde as informações externas serão armazenadas.

Não crie pasta que ninguém use — toda pasta acima tem dono declarado neste documento.

### Fluxo

`PDR -> Spec -> Plan -> Task -> Execute -> Archive`

---

## 2. Skills

A 2.1 instala skills de terceiros. As demais serão criadas seguindo o seguinte fluxo: /skill-creator + prompt

### 2.1 Skills básicas

Peça permissão antes de instalar qualquer coisa e não prossiga sem resposta afirmativa.

- skill-creator — `npx -y skills add anthropics/skills --skill skill-creator --agent claude-code`

Se a permissão for negada, ou a instalação falhar, **pare o setup aqui** e registre a dependência insatisfeita. As seções 2.2 a 2.7 criam as skills do fluxo entregando prompts à `skill-creator`; sem ela, não há como executá-las. Não escreva as skills à mão a partir dos prompts: prompt não é skill, e o que sairia daí não tem a forma que o fluxo espera.

---

### 2.2 Skill do PDR

Primeira etapa do fluxo. Invoque a `skill-creator` e entregue a ela o prompt abaixo.

```
Crie uma skill chamada `pdr`.

**O que ela faz.** Conduz uma conversa com o usuário e, no fim, escreve um Product Decision Record — o registro da decisão de produto que está sendo tomada. Ela não escreve código, não altera nada em `resources/`, e não produz nenhum artefato além do PDR.

**Quando dispara.** Só quando o usuário a invoca explicitamente. A `description` deve dizer isso com todas as letras, para o modelo não começar a entrevistar ninguém porque o assunto surgiu na conversa.

**A conversa.**

- Uma pergunta por mensagem. Nunca empilhe perguntas.
- Prefira alternativas fechadas quando o assunto permitir; pergunta aberta quando não permitir.
- Antes da primeira pergunta, leia o estado do projeto: `.spec/shared/`, os PDRs já existentes em `.spec/specs/` e o que houver em `resources/`. Não pergunte o que o repositório já responde.
- Leia `.spec/shared/` antes de qualquer outra coisa. O que está ali vale para todas as features — vocabulário, convenções, invariantes, limites, versões fixadas, caminhos já tentados e armadilhas conhecidas — e não se rediscute a cada rodada. Contradizer o que está lá é achado a reportar, não escolha a fazer.
- Investigue nesta ordem: que problema motiva a decisão, quem sente esse problema, o que já foi tentado, que restrições existem (prazo, técnica, gente, dinheiro), o que está fora de escopo, e como se saberá que a decisão foi acertada.
- Não trate resposta vaga como resposta. Se o usuário disser "precisa ser rápido", pergunte rápido em relação a quê e medido como.

**As alternativas.**

Depois de entender o problema, apresente de duas a três alternativas reais — cada uma com o que custa e o que entrega — e diga qual você recomenda e por quê. Alternativa que ninguém escolheria não conta: se só existe um caminho viável, diga isso, em vez de inventar duas opções de enfeite.

**O portão.**

Não escreva o PDR antes da aprovação. Apresente na conversa, em texto, o que vai ser registrado — decisão, alternativas descartadas, consequências — e espere um sim explícito. Concordar com um trecho não é aprovar o documento.

**A escrita.**

Recebido o sim: copie `.spec/templates/pdr.md` para `.spec/specs/PDR-<NNN>-<slug>.md` e preencha a cópia. `<NNN>` é o ID da feature, sempre pergunte ao usuário. Antes de aceitar o número, liste `.spec/specs/` **e** `.spec/specs/archive/`: número de feature não se reaproveita, e a pasta de specs fica vazia depois de cada arquivamento — quem só olha ali não enxerga nenhuma feature já fechada. Se o número pedido aparecer em qualquer um dos dois, recuse e proponha o próximo livre. `<slug>` é a decisão em três a cinco palavras, minúsculas, separadas por hífen. O modelo em `.spec/templates/` nunca é editado.

Preencha todos os campos do modelo. Campo que você não consegue preencher é pergunta que faltou fazer: volte e pergunte, não escreva "a definir".

**Régua do conteúdo.**

- Contexto descreve as forças em jogo no momento da decisão, não a solução. Quem ler daqui a um ano precisa entender por que aquilo fazia sentido então.
- Decisão é uma frase afirmativa, no presente, dizendo o que será feito. "Avaliar se vale a pena" não é decisão.
- Toda alternativa descartada carrega o motivo da recusa. Sem motivo, ela não entra na tabela.
- Consequências incluem o que piora. PDR que só lista benefício não foi pensado, foi vendido.
- Gatilho de revisão é observável: uma data, um número que se cruza, uma premissa que pode cair.

**Antes de entregar.**

Releia o que escreveu procurando quatro defeitos: campo com placeholder ou "a definir"; seção que contradiz outra; frase que admite duas leituras; e decisão que na verdade são duas disfarçadas de uma. Corrija os três primeiros na hora. No quarto caso, pare e proponha ao usuário dois PDRs separados.

**A revisão.**

Escrito o arquivo e feita a autorrevisão, invoque o subagente `revisor`, informando o caminho do PDR e a etapa `PDR`. Ele lê só o arquivo e o modelo, procura defeitos numa lista fechada e devolve achados por severidade — corrigir não é função dele. Achado bloqueante você corrige antes de entregar; o resto você apresenta e o usuário decide.

Ao terminar, informe o caminho do arquivo, o que o `revisor` apontou e o que foi corrigido, e peça a revisão do usuário.

**A aprovação.**

O PDR nasce `proposto` e é a spec que exige `aceito`. Quem promove é você, e só depois de o usuário aprovar em palavras: mude o `Status` para `aceito`, ou para `rejeitado` se ele recusar. Nunca promova por conta própria, nunca trate silêncio como aprovação, e nunca deixe o arquivo em `proposto` depois de um sim — é esse campo, e não a memória da conversa, que a etapa seguinte lê.
```

### 2.3 Skill das especificações

Segunda etapa do fluxo. Invoque a `skill-creator` e entregue a ela o prompt abaixo.

```
Crie uma skill chamada `spec`.

**O que ela faz.** Parte de um PDR aprovado e produz a especificação técnica da funcionalidade: entradas, saídas, comportamento esperado e critérios de aceitação objetivos. O PDR decidiu o quê e por quê; a spec fixa o como, com rigor suficiente para que quem for implementar não precise decidir nada sozinho. Ela não escreve código e não altera `resources/`.

**Quando dispara.** Só quando o usuário a invoca explicitamente, passando o número do PDR de origem. A `description` deve dizer isso.

**Entrada.**

Leia `.spec/shared/` antes de qualquer outra coisa. O que está ali vale para todas as features — vocabulário, convenções, invariantes, limites, versões fixadas, caminhos já tentados e armadilhas conhecidas — e não se rediscute a cada rodada. Contradizer o que está lá é achado a reportar, não escolha a fazer.

Leia o PDR informado em `.spec/specs/`. Se ele não estiver com `Status: aceito`, pare e diga por quê — não se especifica em cima de decisão que ainda não foi tomada. Leia também `resources/` para saber o que já existe: linguagem, versões, bibliotecas em uso, convenções. A spec tem que caber no projeto real, não num projeto ideal.

**A pesquisa.**

Esta etapa pesquisa a internet antes de propor qualquer coisa, e pesquisa amplo: repositórios, bibliotecas, documentação oficial, notas de versão, artigos, discussões de issue, comparações. O objetivo é achar solução que já existe e funciona, em vez de projetar do zero o que alguém já resolveu.

Regras da pesquisa:

- Pesquise antes de propor, nunca depois para justificar o que já escolheu.
- Abra a fonte. Não cite repositório, biblioteca ou API que você só viu no resultado de busca.
- Confira data e versão. Biblioteca abandonada, API removida e tutorial de cinco anos atrás são armadilhas comuns aqui: registre a versão que você conferiu e a data em que conferiu.
- Confira a licença de tudo que virar dependência.
- Confira compatibilidade com o que já está em `resources/` — versão de linguagem, runtime, dependências que já existem e podem conflitar.
- Registre o que encontrou. Toda fonte que sustentou uma escolha entra na seção Referências da spec, com o que foi tirado dela e a data de consulta. Fonte que você leu e descartou também vale registrar quando o descarte foi informativo.

**A prova de viabilidade.**

Antes de propor uma abordagem, tenha certeza de que ela é executável. Para cada abordagem, liste o que precisa ser verdade para ela funcionar — a biblioteca faz mesmo isso, a API expõe mesmo esse campo, o limite de requisição comporta o volume, a licença permite o uso, a versão instalada suporta a chamada — e confirme cada item numa fonte que você abriu.

O que você não conseguir confirmar não vira premissa silenciosa: ou vira risco declarado, com o gatilho que o resolve, ou derruba a abordagem. Abordagem que depende de algo não confirmado não é apresentada como viável.

**A conversa.**

- Uma pergunta por mensagem. Nunca empilhe perguntas.
- Não pergunte o que o PDR, o `resources/` ou a pesquisa já respondem.
- Investigue o que a spec precisa e o PDR não fixa: volume e frequência esperados, formato exato das entradas e saídas, o que acontece em cada erro, o que é aceitável degradar, o que não pode quebrar de jeito nenhum.
- Traduza resposta vaga em número ou em condição observável antes de seguir. "Tem que aguentar bastante acesso" não é requisito; "mil requisições por minuto, com resposta abaixo de 300 ms no percentil 95" é.

**As abordagens.**

Apresente de duas a três abordagens viáveis — viáveis no sentido acima, cada uma com a prova feita. Para cada uma: como funciona, o que ela custa (dependências, complexidade, manutenção), o que ela entrega, e o que ela impede no futuro. Diga qual você recomenda e por quê. Se a pesquisa mostrou que só existe um caminho executável, diga isso e mostre por que os outros caíram.

**O portão.**

Não escreva a spec antes da aprovação. Apresente na conversa a abordagem recomendada, as entradas e saídas principais e os critérios de aceitação, e espere um sim explícito.

**A escrita.**

Recebido o sim: copie `.spec/templates/spec.md` para `.spec/specs/SPEC-<NNN>-<slug>.md` e preencha a cópia. `<NNN>` é o ID da feature — o mesmo do PDR de origem, sem número novo: é ele que amarra os artefatos da feature. `<slug>` é a funcionalidade em três a cinco palavras, minúsculas, separadas por hífen. O cabeçalho aponta o PDR de origem.

**Régua do conteúdo.**

- Toda entrada tem nome, tipo, origem, se é obrigatória, a validação que sofre e o que acontece quando ela chega inválida. Entrada sem o caso inválido está pela metade.
- Toda saída tem nome, tipo, destino, formato e quando é produzida.
- O comportamento esperado é descrito do lado de fora: o que se observa, não o que acontece internamente. Inclui o fluxo principal, um caso de erro por condição que pode falhar, e os invariantes que precisam valer em qualquer execução.
- Todo critério de aceitação é objetivo: traz como verificar e qual resultado torna o critério satisfeito. "Rápido", "estável", "intuitivo", "funciona bem" não são critérios. Critério que você não sabe verificar é requisito mal definido — volte e feche com o usuário.
- Nenhum critério de aceitação existe sem entrada, saída ou comportamento que o sustente, e nada que o PDR decidiu fica sem cobertura na spec.
- Lista fechada é lista fechada: nada de "etc.", "entre outros", "e assim por diante".

**Antes de entregar.**

Releia procurando: campo com placeholder; critério subjetivo; entrada sem caso inválido; afirmação técnica sem linha na tabela de viabilidade; contradição entre o comportamento e os critérios; e escopo que cresceu além do PDR. Corrija na hora. Se a spec passou a cobrir decisão que o PDR não tomou, pare e diga ao usuário — isso é PDR novo, não spec maior.

**A revisão.**

Escrita a spec e feita a autorrevisão, invoque o subagente `revisor`, informando o caminho do arquivo e a etapa `SPEC`. Corrija os achados bloqueantes antes de entregar; o resto você apresenta e o usuário decide.

Ao terminar, informe o caminho do arquivo, o que o `revisor` apontou e o que foi corrigido, e peça a revisão do usuário.

**A aprovação.**

A spec nasce `rascunho` e o plano exige `aprovada`. Depois de o usuário aprovar em palavras, mude o `Status` para `aprovada`. Nunca promova por conta própria nem trate silêncio como aprovação.
```

### 2.4 Skill do planejamento

Terceira etapa do fluxo. Invoque a `skill-creator` e entregue a ela o prompt abaixo.

```
Crie uma skill chamada `plan`.

**O que ela faz.** Cruza uma spec aprovada com o código que existe em `resources/` e produz a estratégia de engenharia: que arquivos serão criados, alterados ou removidos; que dependências serão instaladas; que efeitos colaterais a mudança provoca na arquitetura; e quais etapas precisam ser feitas, em que ordem, quais dependem de quais e quais podem correr juntas. Ela não escreve código e não altera `resources/`.

**Quando dispara.** Só quando o usuário a invoca explicitamente, passando o ID da feature. A `description` deve dizer isso.

**Entrada.**

Leia `.spec/shared/` antes de qualquer outra coisa. O que está ali vale para todas as features — vocabulário, convenções, invariantes, limites, versões fixadas, caminhos já tentados e armadilhas conhecidas — e não se rediscute a cada rodada. Contradizer o que está lá é achado a reportar, não escolha a fazer.

Leia a spec da feature em `.spec/specs/`. Se ela não estiver com `Status: aprovada`, pare e diga por quê. Leia também o PDR de origem, para não replanejar o que já foi decidido.

**O levantamento.**

Planejar aqui é ler o código, não imaginá-lo. Antes de escrever qualquer etapa:

- Localize em `resources/` cada ponto que a spec toca e abra os arquivos. "Provavelmente está em" não é levantamento.
- Para cada arquivo que será alterado ou removido, levante o raio de alcance: quem o importa, quem o chama, que testes o cobrem, que contrato público muda. Esse conjunto é o efeito colateral, e ele se descobre com busca no código — não é estimativa.
- Levante as dependências a instalar com nome, versão, licença, gerenciador e o comando exato, e confira conflito com o que já está instalado.
- Se a spec pede algo que o código atual impede, isso é achado: reporte ao usuário antes de planejar por cima.

**A conversa.**

Esta etapa não entrevista ninguém. Ela lê, levanta e planeja. Mas na dúvida, pergunte — sempre, e antes de escrever, não depois.

Conta como dúvida: existem dois lugares plausíveis para a mesma mudança; a convenção do projeto é ambígua ou tem exceções; a spec admite duas implementações com consequências diferentes; não dá para saber, lendo o repositório, se alguém de fora depende do contrato que vai mudar; a ordem entre duas etapas não é óbvia.

O motivo de perguntar em vez de adivinhar é econômico: erro de sequenciamento não aparece no plano, aparece na execução, que é o lugar mais caro de consertar. Uma pergunta custa uma mensagem; um plano com ordem errada custa etapas refeitas.

**As etapas.**

- Cada etapa é uma unidade com começo e fim observáveis e uma verificação própria — o comando ou a observação que prova que ela funcionou, com o resultado que a aprova.
- Etapa cuja verificação você não sabe nomear está grande demais ou mal definida. Quebre.
- Etapa entrega uma coisa. "E" ligando dois trabalhos que se verificam separadamente são duas etapas.
- Cada etapa declara: o que faz, os arquivos exatos, a verificação, de que etapas depende, com quais pode correr em paralelo, que critérios da spec atende, e como se desfaz.
- Ordem lógica significa que nada depende do que ainda não existe. Percorra o grafo e confirme: para toda etapa, tudo de que ela precisa foi produzido antes.
- O grafo não tem ciclo. Duas etapas que dependem uma da outra são um recorte errado — refaça o recorte, não invente uma ordem.

**Paralelismo.**

Paralelismo se deriva das dependências, não do desejo. Duas etapas correm juntas quando nenhuma depende da outra e nenhuma escreve no arquivo em que a outra escreve. Escrita no mesmo arquivo é dependência, mesmo quando não há dependência lógica.

Agrupe as etapas em ondas: a onda 1 são as etapas sem dependência; a onda N são as etapas cujas dependências foram todas satisfeitas até a onda anterior. Dentro de uma onda, tudo corre junto.

Nomeie o caminho crítico: a cadeia mais longa de etapas dependentes. É ela que define o piso do prazo — encurtar etapa fora dela não adianta nada.

**Cobertura.**

Nos dois sentidos: todo critério de aceitação da spec é atendido por pelo menos uma etapa, e toda etapa serve a pelo menos um critério — ou a uma etapa que sirva. Etapa que não serve a nada sai do plano.

**Reversão.**

Toda etapa que mexe em estado difícil de desfazer — migração, dado existente, contrato já publicado, configuração de ambiente — diz como se desfaz. A volta se planeja agora, no desenho, não quando quebrar.

**O portão.**

Antes de escrever o arquivo, apresente na conversa só a lista de etapas com suas dependências e as ondas, e espere um sim. É a decomposição que precisa ser conferida, não a prosa — um sim basta.

**A escrita.**

Recebido o sim: copie `.spec/templates/plan.md` para `.spec/specs/PLAN-<NNN>-<slug>.md` e preencha a cópia. `<NNN>` é o ID da feature, o mesmo da spec de origem. `<slug>` acompanha o da spec.

**Antes de entregar.**

Releia procurando: etapa sem verificação observável; ciclo no grafo; etapa que usa o que só existe depois; paralelismo declarado entre etapas que escrevem no mesmo arquivo; critério da spec sem etapa; arquivo citado numa etapa e ausente do mapa; mudança irreversível sem volta. Corrija na hora.

**A revisão.**

Escrito o plano e feita a autorrevisão, invoque o subagente `revisor`, informando o caminho do arquivo e a etapa `PLAN`. Corrija os achados bloqueantes antes de entregar; o resto você apresenta e o usuário decide.

Ao terminar, informe o caminho do arquivo, o que o `revisor` apontou e o que foi corrigido, e peça a revisão do usuário.

**A aprovação.**

O plano nasce `rascunho` e as tarefas exigem `aprovado`. Depois de o usuário aprovar em palavras, mude o `Status` para `aprovado`. Nunca promova por conta própria nem trate silêncio como aprovação.
```

### 2.5 Skill das tarefas

Quarta etapa do fluxo. Invoque a `skill-creator` e entregue a ela o prompt abaixo.

```
Crie uma skill chamada `tasks`.

**O que ela faz.** Traduz um plano aprovado numa lista cirúrgica de tarefas atômicas, encadeadas e marcáveis. Cada tarefa é uma instrução isolada de execução: quem for executá-la recebe aquele bloco e nada mais. Ela não escreve código e não altera `resources/`.

**Não gere uma skill por tarefa.** A entrada da tarefa já é o prompt do executor; uma skill em cima dela seria uma segunda descrição do mesmo trabalho, que diverge com o tempo e polui `.claude/skills/` com arquivos de uso único disputando atenção com os permanentes. O isolamento vem do subagente, não do arquivo. Skill só se justifica para procedimento que se repete entre tarefas e entre features — e essa é permanente, escrita uma vez, referenciada pelas tarefas.

**Quando dispara.** Só quando o usuário a invoca explicitamente, passando o ID da feature. A `description` deve dizer isso.

**Entrada.**

Leia o plano da feature em `.spec/specs/`. Se ele não estiver com `Status: aprovado`, pare e diga por quê. Leia a spec, para os critérios de aceitação, e o código em `resources/`, para os caminhos exatos — a tarefa cita caminho que existe, não caminho que deveria existir.

**A tradução.**

Cada etapa do plano vira uma ou mais tarefas, e nenhuma tarefa nasce fora do plano. Se algo necessário não está lá, o plano é que está incompleto: pare e diga, não conserte por conta própria. A ordem das tarefas respeita o grafo do plano, e as ondas do plano viram blocos de tarefas.

**O tamanho.**

Atômico não é minúsculo — é indivisível sem perder valor. A régua: a tarefa entrega uma mudança que se verifica sozinha e caberia num commit. Se a verificação precisa de dois resultados para aprovar, são duas tarefas. Se oito tarefas só valem juntas e nenhuma delas deixa o sistema num estado verificável, era uma tarefa só.

A outra régua é o contexto: a tarefa, somada aos arquivos que ela manda abrir, tem que caber no contexto de um agente que começa do zero. Se para entender o que fazer ele precisa ler meia dúzia de arquivos grandes, ou a tarefa está mal recortada ou falta instrução nela.

**A régua da autonomia.**

Escreva cada tarefa para um executor que não viu o PDR, não viu a spec, não viu o plano e não viu esta conversa. É a régua que sustenta todas as outras:

- Caminho exato e completo. Nunca "o arquivo de configuração", "o módulo de autenticação".
- Comando literal, copiável e colável. Nunca "rode os testes", "valide o resultado".
- Interface que a tarefa cria ou altera vem com nome e tipo concretos, escritos ali.
- Zero placeholder, zero "conforme necessário", "se aplicável", "ajuste ao projeto".
- Tarefa que exige uma decisão não está pronta. Decidir é do plano; executar é da tarefa.
- Tarefa que só se entende depois de ler o plano está mal escrita, mesmo que o plano esteja certo.

**O encadeamento.**

Quem executa não vai reconstruir o grafo: vai ler o que você declarou e conferir. Então declare de forma redundante e literal, para que erro apareça como contradição em vez de passar batido.

Cada tarefa declara de que tarefas depende, com quais pode correr em paralelo, a pré-condição observável que precisa valer antes de começar e a pós-condição que passa a valer depois. Numeração contínua `T1`, `T2`, ... , sem buraco e sem reaproveitamento.

Sete regras fecham a corrente:

- **"Arquivos" lista só o que a tarefa escreve.** Arquivo que ela apenas lê fica no "O que fazer". É por essa lista que a execução detecta colisão entre tarefas paralelas — poluí-la com leitura cega a checagem.
- **"Depende de" e "Paralelizável com" não se cruzam.** Nenhuma tarefa aparece nos dois campos da mesma tarefa, e os dois campos concordam com o bloco. A redundância é proposital: é ela que torna o erro visível.
- **A pré-condição é copiada, não reescrita.** Ou é o estado inicial do repositório, ou é a pós-condição de uma das tarefas em "Depende de", palavra por palavra. Quem executa compara os dois textos; sinônimo quebra a comparação.
- **Tarefas no mesmo bloco não se cruzam nem se sobrepõem.** Só entram juntas tarefas sem dependência entre si, direta ou por cadeia, e sem nenhum arquivo em comum em "Arquivos".
- **Caminho é arquivo, nunca pasta nem padrão.** "Arquivos" lista caminhos de arquivo, um a um. Pasta, curinga ou "todos os arquivos de" cegam a comparação: duas tarefas parecem disjuntas e escrevem no mesmo lugar. Tarefa que precisa escrever numa pasta inteira lista os arquivos, ou não é paralelizável com ninguém.
- **A verificação também escreve.** Comando de teste, build, instalação, formatação ou geração de código quase sempre escreve fora dos arquivos da tarefa: lockfile, cache, `dist/`, cobertura, `__pycache__`, migração. Levante o que o comando escreve e inclua em "Arquivos" como qualquer outra escrita. Duas tarefas cujas verificações escrevem no mesmo lugar não vão para o mesmo bloco — e é assim que se descobre que rodar `install` em paralelo nunca foi paralelo.
- **Recurso que não é arquivo colide igual.** Banco de dados, porta, serviço em execução, variável de ambiente, credencial de uso exclusivo, diretório temporário fixo. Declare no campo "Recursos". Duas tarefas que usam o mesmo recurso não correm juntas, mesmo sem compartilhar um único caminho.

Antes de fechar um bloco, faça a checagem cruzada que revela dependência escondida: a pré-condição de cada tarefa do bloco não pode ser a pós-condição de nenhuma outra tarefa do mesmo bloco. Se for, existe dependência real que o campo "Depende de" não declarou — separe as duas em blocos diferentes.

Feche o arquivo com o Mapa de execução: uma linha por bloco, com as tarefas do bloco, de que bloco ele depende, todos os arquivos escritos ali e todos os recursos usados. Arquivo repetido em duas tarefas do mesmo bloco é erro de recorte, não detalhe. É esse mapa que a execução usa para conferir o paralelismo antes de despachar qualquer coisa.

**Os checkboxes.**

Cada tarefa começa com `- [ ]`. Quem executa marca `- [x]` assim que a verificação daquela tarefa passar, no próprio arquivo, uma por uma — nunca em lote no fim. Tarefa cuja verificação não rodou, ou rodou e falhou, não é marcada.

O arquivo é o estado da execução: quem retomar a feature semanas depois descobre onde ela parou lendo só ele. Mantenha também a linha de Progresso coerente com as caixas marcadas.

**A conversa.**

Esta etapa não entrevista. Mas na dúvida, pergunte antes de escrever. Conta como dúvida: o plano cita caminho que não existe em `resources/` e nenhuma etapa anterior cria; a mesma mudança caberia em dois arquivos e o plano não escolheu; o comando de verificação depende de ferramenta que você não confirmou instalada.

**O portão.**

Antes de escrever o arquivo, apresente só a lista numerada dos títulos das tarefas, com as dependências e os blocos, e espere um sim. O que precisa ser conferido é o recorte.

**A escrita.**

Recebido o sim: copie `.spec/templates/tasks.md` para `.spec/specs/TASKS-<NNN>-<slug>.md` e preencha a cópia. `<NNN>` é o ID da feature e `<slug>` é o slug, os dois iguais aos do plano de origem. Entregue todas as caixas desmarcadas.

**Antes de entregar.**

Releia procurando: tarefa que deixa decisão para o executor; caminho inexato; comando que não dá para copiar e rodar; verificação sem resultado esperado; dependência apontando para tarefa de número maior; etapa do plano que ficou sem tarefa; caixa já marcada. Corrija na hora.

**A revisão.**

Escrita a lista e feita a autorrevisão, invoque o subagente `revisor`, informando o caminho do arquivo e a etapa `TASKS`. Corrija os achados bloqueantes antes de entregar; o resto você apresenta e o usuário decide.

Ao terminar, informe o caminho do arquivo, o que o `revisor` apontou e o que foi corrigido, e peça a revisão do usuário.

**A aprovação.**

O arquivo de tarefas nasce `rascunho` e a execução exige `aprovado`. Depois de o usuário aprovar em palavras, mude o `Status` para `aprovado`. Nunca promova por conta própria nem trate silêncio como aprovação.
```

### 2.6 Skill da execução

Quinta etapa do fluxo. Invoque a `skill-creator` e entregue a ela o prompt abaixo.

```
Crie uma skill chamada `execute`.

**O que ela faz.** Executa a lista de tarefas de uma feature, respeitando os blocos e o paralelismo declarados, e escreve o log da execução. É a única etapa do fluxo que altera `resources/`.

**Quando dispara.** Só quando o usuário a invoca explicitamente, passando o ID da feature. Nunca porque o assunto surgiu na conversa, nunca porque o usuário comentou que quer executar. A `description` precisa dizer isso com todas as letras: esta skill escreve código, e o portão é o usuário digitar o comando.

**Entrada.**

Leia o arquivo de tarefas da feature em `.spec/specs/`. Se ele não estiver com `Status: aprovado`, pare e diga por quê. Tarefas já marcadas com `- [x]` não são reexecutadas.

Recuse-se a começar se já houver rodada em curso para esta feature — arquivo de tarefas com `Status: em execução`, ou log `EXEC` da feature com `Status: em execução`. Dois orquestradores na mesma feature marcam o mesmo arquivo e escrevem o mesmo log; o segundo corrompe o trabalho do primeiro. Marque `em execução` ao começar e devolva o status ao terminar ou ao interromper. A exceção é a rodada morta, tratada em Retomada.

Antes de tocar em qualquer coisa, registre o inventário de `resources/`: todo caminho de arquivo com seu tamanho e data de modificação. Salve a saída em `.spec/specs/EXEC-<NNN>-<slug>.inventario.txt` e anote o caminho na seção Inventário inicial do log — o log guarda o ponteiro, o arquivo guarda a prova. É esse inventário que prova o que mudou.

**A orquestração.**

Execute bloco a bloco, na ordem. Dentro de um bloco, as tarefas correm em paralelo, uma por subagente. Espere o bloco inteiro terminar antes de abrir o próximo — bloco seguinte não começa com o anterior incompleto.

**O paralelismo, conferido por você.**

Duas tarefas com dependência entre si nunca vão para agentes diferentes ao mesmo tempo. Nunca. Não confie no bloco: antes de despachar, confira você mesmo, tarefa a tarefa do bloco, que nenhuma delas aparece no "Depende de" de outra do mesmo bloco, direta ou por cadeia, e que duas não escrevem no mesmo arquivo.

Confira quatro coisas em cada bloco, antes de despachar:

1. **Dependência.** Nenhuma tarefa do bloco aparece no "Depende de" de outra do mesmo bloco, direta ou por cadeia.
2. **Dependência escondida.** A pré-condição de nenhuma tarefa do bloco é a pós-condição de outra tarefa do mesmo bloco. Se for, existe dependência que o campo não declarou — vale mais que o campo.
3. **Arquivos.** Nenhum caminho aparece em "Arquivos" de duas tarefas do bloco. Caminho que for pasta ou curinga não dá para comparar: trate a tarefa como não paralelizável.
4. **Recursos.** Nenhum recurso — banco, porta, serviço, variável de ambiente — aparece em duas tarefas do bloco.

Se qualquer uma das quatro falhar, não paralelize: execute aquelas tarefas em série, na ordem da dependência, registre o desvio no log e avise o usuário — o bloco está errado no arquivo de tarefas, e isso é defeito a corrigir lá.

Uma tarefa só é despachada quando todas as tarefas de que ela depende já estão marcadas concluídas e suas verificações passaram. Dependência pendente segura a tarefa, mesmo que o bloco diga que ela pode correr.

Toda tarefa roda no subagente `executor`, inclusive quando o bloco tem uma só. O contrato é sempre o mesmo, e é o isolamento que impede quem executa de arrastar contexto que não é dele.

Despache uma tarefa por invocação do `executor`, mandando o bloco daquela tarefa na íntegra mais a declaração de escopo abaixo, e nada além disso.

Cada subagente recebe o bloco daquela tarefa na íntegra e mais nada: não recebe o PDR, a spec, o plano, as outras tarefas nem esta conversa. Se a tarefa não basta sozinha, o defeito é da tarefa — pare e reporte, não complemente.

**Você não executa, você orquestra.** Não escreva código por conta própria, não conserte o que um subagente deixou pela metade, não decida o que uma tarefa não decidiu. Seu trabalho é despachar, ler relatório, marcar e registrar.

**A declaração de escopo.**

Todo prompt de subagente termina com este bloco, literalmente, sem adaptar:

    Escopo desta tarefa: apenas os arquivos listados em Arquivos. Não altere nenhum outro arquivo. Não conserte nada fora do que a tarefa pede, mesmo que esteja visivelmente errado. Não adicione teste, documentação, comentário, formatação, renomeação, refatoração ou dependência que a tarefa não peça. Não complete o que faltar na tarefa e não decida o que ela não decidiu. Se faltar informação, pare e reporte. Se a verificação falhar, pare e reporte: não altere a verificação nem a tarefa para fazê-la passar. Ao terminar, entregue o relatório completo na ordem do seu prompt de agente: arquivos tocados, comando, código de saída, saída, resultado e achados. A lista de achados é obrigatória — é o que você viu de errado e deliberadamente não consertou.

Esse bloco não é enfeite. Sem declaração explícita de escopo no prompt, a taxa de ação fora do pedido em agentes de código sobe de perto de zero para perto de um em seis.

**As regras de comando.**

- Rode o comando exatamente como está escrito na tarefa. Precisou mudar para funcionar? Isso é desvio: registre no log e reporte, não corrija em silêncio.
- Capture sempre o código de saída e a saída. Os dois vão para o log, mesmo quando dá certo.
- Comando destrutivo não roda automaticamente, ainda que escrito na tarefa: apagar arquivo ou diretório, sobrescrever em massa, migração de dados, limpeza de banco, publicação. Pare, mostre o comando ao usuário e espere confirmação explícita.
- Nunca `sudo`.
- **Instalação é sua, nunca do executor.** Tarefa que precise instalar dependência faz o `executor` parar e reportar. Quem instala é você, conferindo o pacote e a versão contra as Dependências a instalar do plano, e registrando o comando na trilha. É por isso que o executor não precisa do plano para obedecer à regra: ele simplesmente não instala nada.
- Não repita comando que já teve sucesso nesta rodada.
- Antes de criar, confira se já existe; antes de alterar, confira se já não está no estado esperado. Rodar a mesma tarefa duas vezes não pode duplicar nada.

**As paradas obrigatórias.**

Pare a tarefa, não marque a caixa, registre e reporte quando:

- a verificação falhar, qualquer que seja o motivo;
- a tarefa mandar tocar arquivo que não existe e que nenhuma tarefa anterior criou;
- um arquivo fora da lista da tarefa precisaria mudar para a verificação passar;
- a tarefa for ambígua, incompleta ou exigir uma decisão;
- o comando pedir confirmação, credencial ou acesso de rede não previsto.

As demais tarefas do mesmo bloco seguem até o fim. Os blocos seguintes não começam. Quem decide continuar é o usuário.

**A marcação.**

Quem marca `- [x]` no arquivo de tarefas é você, nunca o subagente. Duas razões: vários subagentes escrevendo no mesmo arquivo ao mesmo tempo o corrompem, e marcar é decisão de quem leu o relatório e a saída da verificação. Marque uma tarefa por vez, só depois da verificação ter passado, e mantenha a linha de Progresso coerente.

**A verificação de contaminação.**

Ao fim de cada bloco, antes de abrir o próximo, levante de novo o inventário de `resources/` e compare com o do começo. Todo arquivo que mudou, foi criado ou sumiu tem que estar na união dos "Arquivos" das tarefas executadas. O que estiver fora dessa união é contaminação: registre no log com o que mudou, **pare a rodada** e reporte ao usuário. Não desfaça por conta própria — desfazer também é alterar o que ninguém pediu.

A conferência é por bloco, não só no fim, e a razão é prática: contaminação no bloco 1 descoberta só no fim significa que todos os blocos seguintes rodaram em cima de estado sujo. O relatório do subagente sobre os arquivos que ele tocou é o que ele diz ter feito; o inventário é o que aconteceu.

**Os achados.**

Você vai encontrar coisa errada que não é da sua tarefa: código quebrado ao lado, premissa do plano que não se sustentou, dívida óbvia. Não conserte. Registre em Achados, com onde está e por que importa, e siga. O log é o lugar onde a vontade de consertar vira informação para a próxima rodada, em vez de virar contaminação nesta.

**O log.**

Copie `.spec/templates/exec.md` para `.spec/specs/EXEC-<NNN>-<slug>.md` no começo da rodada e preencha à medida que executa, não no fim — se a rodada for interrompida, o que já foi escrito precisa valer. `<NNN>` é o ID da feature e `<slug>` é o slug, os dois iguais aos do arquivo de tarefas — o mesmo par nomeia o arquivo de inventário.

Preencha à medida que executa, e não só o resultado: o inventário inicial na seção própria, uma linha na Conferência por bloco ao fechar cada bloco, e uma linha em Confirmações destrutivas a cada confirmação que você pedir. São esses três registros que tornam a rodada auditável depois — sem eles o `revisor` não tem como conferir o que você diz ter feito.

**Retomada.** Se já existe log da feature com `Status: interrompida`, não crie outro nem sobrescreva aquele: reabra o mesmo arquivo, acrescente uma linha na tabela de Rodadas, marque `em execução` e continue de onde parou, pulando as tarefas já marcadas. Rastro de rodada anterior não se apaga.

Log com `Status: em execução` e nenhuma rodada viva é rodada **morta**, não rodada em curso — a sessão acabou antes do fechamento. Não confunda com concorrência, e não destrave sozinho: mostre ao usuário o log e a última linha da tabela de Rodadas, peça confirmação explícita de que nada está em curso, e só com o sim feche aquela linha como `interrompida`, ponha o `Status` do log em `interrompida` e siga a retomada acima. Se o arquivo de tarefas também estiver em `em execução`, devolva-o a `aprovado` antes de começar. Sem o sim, não comece.

**A revisão e o fechamento.**

Terminada a última tarefa possível, invoque o subagente `revisor`, informando o caminho do log e a etapa `EXEC`. Depois trate os achados assim:

- **Bloqueante que é defeito do registro** — número errado no Resumo, comando ausente da trilha, bloco sem linha de conferência: corrija no log e siga.
- **Bloqueante que exigiria mexer em `resources/`** — contaminação, tarefa aprovada com verificação que falhou: não corrija. Registre em Desvios, deixe o Status como `interrompida` e reporte. Consertar aqui é exatamente a contaminação que esta etapa existe para impedir.
- **Não bloqueante:** apresente ao usuário e siga; quem decide é ele.

Fechados os achados, atualize o `Status` do log: `concluída` só quando todas as tarefas estão marcadas, sem contaminação em aberto e sem tarefa interrompida; `interrompida` em qualquer outro caso, dizendo onde parou. Devolva o `Status` do arquivo de tarefas: `concluído` quando todas as caixas estão marcadas, `aprovado` quando sobrou tarefa por fazer. As caixas e o Progresso ficam como ficaram.

Só então informe ao usuário: quantas tarefas concluíram, onde parou e por quê, os desvios, a contaminação encontrada, o que o `revisor` apontou e o que foi corrigido, e o caminho do log. A rodada está entregue nesse ponto, e não antes.
```

### 2.7 Skill do arquivamento

Sexta e última etapa do fluxo. Invoque a `skill-creator` e entregue a ela o prompt abaixo.

```
Crie uma skill chamada `archive`.

**O que ela faz.** Fecha uma feature: colhe o que sobrevive dela para `.spec/shared/` e move os artefatos de `.spec/specs/` para uma pasta própria dentro de `.spec/specs/archive/`, deixando `.spec/specs/` limpo para a próxima rodada.

**Quando dispara.** Só quando o usuário a invoca explicitamente, passando o ID da feature. Esta skill move arquivos: nunca dispare porque o assunto surgiu, porque a execução terminou ou porque o usuário comentou que a feature acabou. A `description` precisa dizer isso.

**Não toque em `resources/`.** Nem para ler o resultado, nem para limpar nada. O código já foi entregue pela execução; aqui se mexe apenas em `.spec/`.

**Entrada.**

Localize os artefatos da feature em `.spec/specs/`: o PDR, a spec, o plano, o arquivo de tarefas e o log de execução.

**O portão de fechamento.**

Não arquive feature em aberto. Confira, e pare se qualquer uma falhar:

- existe log de execução, e ele está com `Status: concluída` — não `em execução`, não `interrompida`;
- todas as caixas do arquivo de tarefas estão marcadas, e a linha de Progresso bate com elas;
- o log não tem contaminação em aberto nem tarefa interrompida sem resolução.

Falhando alguma, diga exatamente o que falta e pare. Arquivar feature incompleta é perder o rastro dela junto com o trabalho.

A exceção é a feature abandonada: se o usuário mandar arquivar mesmo assim, arquive, mas registre no README da pasta arquivada por que foi abandonada e em que ponto parou. Abandono com rastro é legítimo; abandono silencioso não.

**A colheita.**

Antes de mover qualquer coisa, leia os cinco artefatos e proponha o que merece sobreviver em `.spec/shared/`:

- do PDR, alternativa descartada cujo motivo vale para o projeto inteiro, não só para aquela decisão;
- da spec, restrição técnica confirmada na pesquisa — versão, licença, limite de serviço — que vai valer na próxima feature também;
- do plano, efeito colateral estrutural que qualquer mudança futura naquela área vai encontrar de novo;
- do log, os Achados não consertados e os desvios que revelaram algo sobre o projeto, não sobre a feature.

O critério é um só: **vale para a próxima feature?** Se vale só para esta, fica na pasta arquivada e não sobe para `.spec/shared/`. Encher o compartilhado de detalhe morto é a forma mais rápida de fazer todo mundo parar de lê-lo.

Proponha as entradas ao usuário, com a seção de destino de cada uma, e espere o sim. Só então escreva em `.spec/shared/`. Nunca reescreva entrada que já existe lá sem apontar o conflito e perguntar.

**A mudança.**

Recebido o sim:

1. Crie `.spec/specs/archive/<NNN>-<slug>/`. O `<slug>` é o da spec — o do PDR nomeia a decisão, o da spec nomeia a funcionalidade, e é pela funcionalidade que alguém procura a feature depois. Se a pasta já existir, pare — número de feature não se reaproveita, e destino ocupado é sinal de erro.
2. Mova para dentro dela os cinco artefatos — na feature abandonada, os que chegaram a existir. Mova, não copie: no fim, `.spec/specs/` não pode ter nenhum arquivo daquela feature.
3. Escreva `README.md` dentro da pasta arquivada: o que a feature era em uma linha, o que ela entregou, quando abriu e quando fechou, quantas tarefas rodaram, o que foi para `.spec/shared/`, os achados de alcance além da feature que você propôs e o usuário recusou promover — com o motivo da recusa — e, se for o caso, por que a feature foi abandonada e que estágios nunca chegaram a existir.
4. Não apague nada. Nada mesmo. Se algo parece sobrar, reporte em vez de remover.

**Antes de entregar.**

Confira: a pasta de destino tem os cinco artefatos mais o README — na feature abandonada, os artefatos que existiram mais o README nomeando os estágios que nunca existiram; `.spec/specs/` não tem mais nenhum arquivo da feature; o que subiu para `.spec/shared/` é o que o usuário aprovou, sem acréscimo; `resources/` está intocado.

**A revisão.**

Feita a conferência, invoque o subagente `revisor`, informando o caminho da pasta arquivada e a etapa `ARCHIVE`. Corrija os achados bloqueantes antes de entregar.

Ao terminar, informe a pasta criada, o que foi movido, o que subiu para `.spec/shared/` e o que o `revisor` apontou.
```

---

## 3. Templates

Crie cada arquivo abaixo no caminho que dá título à sua subseção. São dois destinos diferentes, e a distinção importa:

- **3.1 a 3.5 vão para `.spec/templates/`.** São modelos, e nascem vazios: quem preenche é a skill que os clona, durante o uso. O original nunca é editado depois — cada uso trabalha sobre uma cópia salva em `.spec/specs/`.
- **3.6 vai para `.spec/shared/`.** Não é modelo e não é clonado: é o arquivo de trabalho do projeto, criado direto no lugar e editado ali mesmo.

### 3.1 `.spec/templates/pdr.md`

Modelo do Product Decision Record, clonado pela skill `pdr`.

```markdown
# PDR-<NNN> — <a decisão em uma linha>

- **Status:** proposto | aceito | rejeitado | substituído por PDR-<NNN>
- **Data:** <AAAA-MM-DD>
- **Escopo:** <versão, período ou área do produto que esta decisão alcança>
- **Autor(es):** <quem decidiu>

## Contexto

<As forças em jogo no momento da decisão: o problema, quem sente, o que já foi tentado, as restrições. Descreve a situação, não a solução.>

## Decisão

<Uma frase afirmativa, no presente, dizendo o que será feito. Em seguida, o detalhe necessário para alguém executar sem precisar perguntar.>

## Alternativas consideradas

| Alternativa | Por que foi descartada |
| --- | --- |
| <alternativa> | <motivo da recusa> |

## Consequências

**Esperadas:** <o que melhora com esta decisão>

**Aceitas:** <o que piora, o que fica mais caro, o que se perde>

## Gatilho de revisão

<O que traz este PDR de volta à mesa: uma data, um número ultrapassado, uma premissa que caiu.>

## Fora de escopo

<O que este PDR não decide, para não ser lido como se decidisse.>
```

### 3.2 `.spec/templates/spec.md`

Modelo da especificação técnica, clonado pela skill `spec`.

```markdown
# SPEC-<NNN> — <a funcionalidade em uma linha>

- **Status:** rascunho | aprovada | substituída por SPEC-<NNN>
- **PDR de origem:** PDR-<NNN> — <título do PDR>
- **Data:** <AAAA-MM-DD>
- **Autor(es):** <quem especificou>

## Objetivo

<O que esta funcionalidade entrega, em uma frase, amarrada à decisão do PDR de origem.>

## Fora de escopo

<O que esta spec não cobre, para não ser lida como se cobrisse.>

## Solução escolhida

<A abordagem técnica em prosa curta: que peças existem, como conversam, onde ficam em `resources/`.>

### Viabilidade verificada

| O que precisa ser verdade | Como foi confirmado | Versão | Data |
| --- | --- | --- | --- |
| <afirmação técnica> | <fonte aberta e o que ela diz> | <versão conferida> | <AAAA-MM-DD> |

## Dependências externas

| Dependência | Versão | Licença | Para quê |
| --- | --- | --- | --- |
| <nome> | <versão> | <licença> | <o que ela resolve aqui> |

## Entradas

| Nome | Tipo | Origem | Obrigatória | Validação | Se inválida |
| --- | --- | --- | --- | --- | --- |
| <nome> | <tipo> | <quem envia> | sim / não | <regra> | <comportamento> |

## Saídas

| Nome | Tipo | Destino | Formato | Quando é produzida |
| --- | --- | --- | --- | --- |
| <nome> | <tipo> | <quem recebe> | <formato> | <condição> |

## Comportamento esperado

### Fluxo principal

<Passo a passo do que se observa de fora, do acionamento ao resultado.>

### Casos de erro

| Condição | Comportamento | Sinal para quem chamou |
| --- | --- | --- |
| <o que dá errado> | <o que o sistema faz> | <erro, código, mensagem> |

### Invariantes

<O que precisa continuar verdadeiro em qualquer execução, inclusive nas que falham.>

## Critérios de aceitação

| # | Critério | Como verificar | Resultado esperado |
| --- | --- | --- | --- |
| 1 | <o que precisa ser verdade> | <comando, chamada ou observação> | <resultado que satisfaz> |

## Riscos

| Risco | Impacto | O que fazer se ocorrer |
| --- | --- | --- |
| <o que pode dar errado no plano> | <consequência> | <mitigação ou gatilho> |

## Referências

| Fonte | URL | O que foi tirado dela | Consulta |
| --- | --- | --- | --- |
| <título> | <url> | <o que sustentou> | <AAAA-MM-DD> |
```

### 3.3 `.spec/templates/plan.md`

Modelo do plano de engenharia, clonado pela skill `plan`.

```markdown
# PLAN-<NNN> — <a funcionalidade em uma linha>

- **Status:** rascunho | aprovado | substituído por PLAN-<NNN>
- **SPEC de origem:** SPEC-<NNN> — <título da spec>
- **Data:** <AAAA-MM-DD>
- **Autor(es):** <quem planejou>

## Estratégia

<Como o trabalho será atacado, em prosa curta: por onde começa, por que nessa ordem, o que fica para o fim, o que foi deliberadamente deixado de lado.>

## Mapa de arquivos

| Caminho em `resources/` | Ação | O que muda | Etapa |
| --- | --- | --- | --- |
| <caminho exato> | criar / alterar / remover | <o que muda ali> | E<N> |

## Dependências a instalar

| Dependência | Versão | Licença | Gerenciador | Comando | Conflita com |
| --- | --- | --- | --- | --- | --- |
| <nome> | <versão> | <licença> | <npm, pip, ...> | <comando exato> | <o que já existe e conflita, ou nada> |

## Efeitos colaterais

| Arquivo alterado | Quem depende dele | Impacto | Etapa que trata |
| --- | --- | --- | --- |
| <caminho> | <quem importa, chama ou testa> | <o que quebra ou precisa mudar> | E<N> |

## Etapas

### E<N> — <nome curto e imperativo>

- **Faz:** <o que entrega, em uma frase>
- **Arquivos:** <caminhos exatos>
- **Verificação:** <comando ou observação> → <resultado que aprova>
- **Depende de:** <E<N>, E<N>> ou nenhuma
- **Paralelizável com:** <E<N>> ou nenhuma
- **Critérios da spec que atende:** <#>
- **Como desfazer:** <passo concreto, ou "reverter o arquivo">

## Ordem de execução

| Onda | Etapas | Correm juntas |
| --- | --- | --- |
| 1 | <E1, E2> | sim |
| 2 | <E3> | — |

**Caminho crítico:** <E1 → E3 → E6>

## Riscos do plano

| Risco | Etapa | O que fazer se ocorrer |
| --- | --- | --- |
| <o que pode furar o plano> | E<N> | <ação ou gatilho> |
```

### 3.4 `.spec/templates/tasks.md`

Modelo da lista de tarefas, clonado pela skill `tasks`.

```markdown
# TASKS-<NNN> — <a funcionalidade em uma linha>

- **Status:** rascunho | aprovado | em execução | concluído
- **PLAN de origem:** PLAN-<NNN> — <título do plano>
- **Data:** <AAAA-MM-DD>
- **Autor(es):** <quem decompôs>

## Como executar

Execute na ordem dos blocos. Dentro de um bloco, as tarefas correm juntas. Duas tarefas com dependência entre si nunca rodam ao mesmo tempo, nem em agentes diferentes nem no mesmo — confira o Mapa de execução antes de despachar. Marque `- [x]` assim que a verificação da tarefa passar — uma por uma, nunca em lote — e atualize a linha de Progresso. Tarefa cuja verificação falhou não é marcada. Se algo no repositório não bater com o que está escrito aqui, pare e reporte: não improvise nem conserte por fora.

## Progresso

<0> de <total> tarefas concluídas.

## Mapa de execução

| Bloco | Tarefas | Depende do bloco | Arquivos escritos no bloco | Recursos usados |
| --- | --- | --- | --- | --- |
| 1 | <T1, T2> | nenhum | <caminhos, sem repetição entre as tarefas do bloco> | <sem repetição entre as tarefas do bloco> |
| 2 | <T3> | 1 | <caminhos> | <recursos> |

## Bloco 1 — <o que este bloco entrega>

- [ ] **T1 — <título imperativo e curto>**
  - **Etapa do plano:** E<N>
  - **Depende de:** <T<N>> ou nenhuma
  - **Paralelizável com:** <T<N>> ou nenhuma
  - **Pré-condição:** <o que precisa estar verdadeiro antes de começar, observável>
  - **Arquivos:** <caminho de arquivo, um a um, nunca pasta ou curinga> (criar / alterar / remover) — inclui o que o comando de verificação escreve
  - **Recursos:** <banco, porta, serviço, variável de ambiente> ou nenhum
  - **O que fazer:** <a instrução completa, com interfaces nomeadas e tipadas; nada que exija decisão de quem executa>
  - **Verificação:** `<comando literal>` → <o resultado exato que aprova a tarefa>
  - **Pós-condição:** <o que passa a ser verdadeiro depois>
  - **Se falhar:** <pare e reporte, ou o passo concreto de desfazer>

- [ ] **T2 — <título imperativo e curto>**
  - **Etapa do plano:** E<N>
  - **Depende de:** <T<N>> ou nenhuma
  - **Paralelizável com:** <T<N>> ou nenhuma
  - **Pré-condição:** <...>
  - **Arquivos:** <...>
  - **Recursos:** <...> ou nenhum
  - **O que fazer:** <...>
  - **Verificação:** `<comando literal>` → <resultado esperado>
  - **Pós-condição:** <...>
  - **Se falhar:** <...>

## Bloco 2 — <o que este bloco entrega>

- [ ] **T3 — <título imperativo e curto>**
  - **Etapa do plano:** E<N>
  - **Depende de:** T1
  - **Paralelizável com:** nenhuma
  - **Pré-condição:** <a pós-condição de T1, copiada palavra por palavra>
  - **Arquivos:** <...>
  - **Recursos:** <...> ou nenhum
  - **O que fazer:** <...>
  - **Verificação:** `<comando literal>` → <resultado esperado>
  - **Pós-condição:** <...>
  - **Se falhar:** <...>
```

### 3.5 `.spec/templates/exec.md`

Modelo do log de execução, clonado pela skill `execute`.

```markdown
# EXEC-<NNN> — <a funcionalidade em uma linha>

- **Status:** em execução | concluída | interrompida
- **TASKS de origem:** TASKS-<NNN> — <título>
- **Início:** <AAAA-MM-DD HH:MM> — **Fim:** <AAAA-MM-DD HH:MM>
- **Orquestrador:** <quem conduziu a rodada>

## Resumo

| Tarefas | Concluídas | Interrompidas | Não iniciadas |
| --- | --- | --- | --- |
| <total> | <n> | <n> | <n> |

**Blocos executados:** <1, 2> — **Parou em:** <T<N>, ou nenhuma>

## Registro por tarefa

### T<N> — <título da tarefa>

- **Bloco:** <n>
- **Arquivos tocados:** <caminhos reais, como relatados pelo executor>
- **Comando de verificação:** `<comando exatamente como rodou>`
- **Código de saída:** <n>
- **Saída:** <o trecho que importa; corte o resto>
- **Resultado:** concluída | interrompida
- **Observação:** <o que o executor relatou de relevante, ou nada>

## Desvios

| Tarefa | O que estava previsto | O que aconteceu | O que foi feito |
| --- | --- | --- | --- |
| T<N> | <o que a tarefa dizia> | <o que se encontrou> | <parou e reportou / seguiu assim> |

## Rodadas

| Rodada | Início | Fim | Status | Parou em |
| --- | --- | --- | --- | --- |
| 1 | <AAAA-MM-DD HH:MM> | <AAAA-MM-DD HH:MM> | concluída / interrompida | <T<N>, ou nenhuma> |

## Inventário inicial

Levantado antes de tocar em qualquer coisa. É a base de toda comparação posterior.

| Levantado em | Arquivos em `resources/` | Como foi levantado | Inventário completo em |
| --- | --- | --- | --- |
| <AAAA-MM-DD HH:MM> | <n> | `<comando>` | `<caminho do arquivo salvo>` |

## Conferência por bloco

Uma linha por bloco, escrita ao fechá-lo e antes de abrir o próximo.

| Bloco | Conferido em | Arquivos alterados no bloco | Todos declarados? | Resultado |
| --- | --- | --- | --- | --- |
| 1 | <AAAA-MM-DD HH:MM> | <caminhos> | sim / não | seguiu / parou |

## Confirmações destrutivas

| Tarefa | Comando | Pedida em | Resposta do usuário |
| --- | --- | --- | --- |
| T<N> | `<comando>` | <AAAA-MM-DD HH:MM> | autorizado / recusado |

## Contaminação verificada

Consolidação das conferências por bloco, ao fim da rodada.

| Arquivo alterado | Declarado na tarefa? | Tarefa | Situação |
| --- | --- | --- | --- |
| <caminho> | sim / não | T<N> | previsto / fora de escopo |

## Achados para a próxima rodada

Coisas encontradas durante a execução e deliberadamente **não** consertadas.

| Achado | Onde | Por que importa | Sugestão (não executada) |
| --- | --- | --- | --- |
| <o que se viu> | <caminho> | <consequência> | <o que caberia fazer> |

## Comandos executados

| # | Tarefa | Comando | Código de saída |
| --- | --- | --- | --- |
| 1 | T<N> | `<comando>` | <n> |
```

### 3.6 `.spec/shared/orientacoes.md`

Crie este arquivo em `.spec/shared/orientacoes.md`, **não** em `.spec/templates/`. Ele não é modelo: é preenchido no lugar e cresce a cada arquivamento. O PDR, a spec e o plano o leem antes de começar.

```markdown
# Orientações do projeto

Este arquivo vale para todas as features. O PDR, a spec e o plano o leem antes de qualquer pergunta, e o que está aqui não se rediscute a cada rodada — mudou, muda aqui, e passa a valer da próxima feature em diante.

Só entra o que serve a mais de uma feature. Aprendizado específico de uma feature fica na pasta arquivada dela.

**Seção com campos entre `<>` está vazia.** Placeholder não é convenção, não é invariante e não é decisão: só vale como orientação o que estiver preenchido. Um arquivo recém-criado não orienta nada.

## Vocabulário

| Termo | O que significa neste projeto |
| --- | --- |
| <termo> | <definição, na forma como o time usa> |

## Stack e versões fixadas

| O quê | Versão | Por que está fixada |
| --- | --- | --- |
| <linguagem, runtime, biblioteca> | <versão> | <o que quebra se mudar> |

## Convenções

<Como se nomeia, onde cada tipo de arquivo fica, que padrão o código segue, o que se faz sempre do mesmo jeito.>

## Invariantes

<O que não pode quebrar em nenhuma hipótese, e o que acontece se quebrar. Isto é o que uma spec nunca tem permissão de contradizer.>

## Limites e políticas

<Segurança, dados pessoais, licenças aceitas e recusadas, o que não pode sair do ambiente, o que exige aprovação humana.>

## Decisões permanentes

| Decisão | PDR de origem | Por que ainda vale |
| --- | --- | --- |
| <o que ficou decidido> | PDR-<NNN> | <a razão, ainda de pé> |

## Caminhos já tentados

| O que se tentou | Por que não funcionou | Feature |
| --- | --- | --- |
| <abordagem> | <o que a derrubou> | <NNN> |

## Armadilhas conhecidas

| Onde | O que acontece | Como evitar |
| --- | --- | --- |
| <arquivo, área, ferramenta> | <o sintoma> | <o que fazer em vez disso> |
```

---

## 4. Agentes

Subagentes rodam em contexto isolado: não enxergam a conversa que os chamou. São dois, e a mesma cegueira serve a propósitos opostos. No `revisor`, ela audita: o que não se entende lendo só o arquivo é defeito do arquivo. No `executor`, ela contém: o que não está escrito na tarefa não existe, e por isso não há como ele fazer o que ninguém pediu.


### 4.1 `.claude/agents/revisor.md`

Auditor dos artefatos do fluxo. Uma lista de defeitos por etapa: PDR, SPEC, PLAN, TASKS, EXEC e ARCHIVE.

```markdown
---
name: revisor
description: Audita um artefato do fluxo SDD. Recebe o caminho do artefato e a etapa que o produziu, procura defeitos numa lista fechada e reporta por severidade. Só leitura, nunca corrige. Invocado pelas skills do fluxo.
tools: Read, Grep, Glob
---

# Revisor

Você audita um artefato do fluxo. Quem o escreveu tinha a conversa inteira no contexto; você não tem, e é exatamente por isso que você serve. O que você não entender lendo apenas o arquivo é defeito do arquivo, não limitação sua.

## O que você recebe

O caminho do artefato e o nome da etapa que o produziu. Leia o artefato e o modelo correspondente em `.spec/templates/`. Quando a etapa for `SPEC`, leia também o PDR de origem citado no cabeçalho; quando for `PLAN`, leia também a spec de origem e o código em `resources/` que o plano diz tocar; quando for `TASKS`, leia também o plano de origem; quando for `EXEC`, leia também o arquivo de tarefas de origem; quando for `ARCHIVE`, leia a pasta arquivada inteira, o `.spec/shared/` e o conteúdo de `.spec/specs/`. Não peça contexto, não pergunte nada, não tente reconstruir a conversa: se faltou informação, isso é um achado.

## O que você nunca faz

- Não edita arquivo nenhum, e não entrega texto de substituição pronto.
- Não julga se a decisão está certa. Você audita o registro, não o mérito.
- Não comenta estilo, tom, tamanho ou organização. Você procura os defeitos da lista, e só eles.
- Não inventa achado para justificar a chamada. "Nenhum defeito encontrado" é resposta legítima, e é a resposta esperada na maioria das vezes.

## Como você reporta

Uma linha por achado, agrupadas por severidade:

- **Bloqueante** — o artefato não serve para a etapa seguinte enquanto estiver assim.
- **Grave** — vai custar caro depois, mas não trava agora.
- **Observação** — vale corrigir se for barato.

Cada achado cita a seção onde está e diz qual item da lista foi violado. Não havendo achado, responda exatamente: `Nenhum defeito encontrado.`

## Listas por etapa

Use apenas a lista da etapa que lhe foi informada. Se a etapa não estiver abaixo, diga isso e pare — não improvise critério.

Antes da lista da etapa, confira sempre este item, que vale para todas:

- **Status promovido sem aprovação.** O artefato chega com o `Status` já avançado — `aceito`, `aprovada`, `aprovado`, `concluído` — sem que o texto entregue registre a aprovação do usuário. Bloqueante: é assim que uma etapa passa pela porta da seguinte sem ninguém ter dito sim.

### PDR

1. **Decisão que não decide.** A seção Decisão não é uma frase afirmativa, no presente, dizendo o que será feito. "Avaliar", "estudar", "considerar" caem aqui. Bloqueante.
2. **Duas decisões num PDR.** A Decisão traz conjunção que esconde uma segunda escolha independente — dá para recusar uma sem derrubar a outra. Bloqueante.
3. **Alternativa sem motivo.** Linha da tabela de Alternativas consideradas sem motivo de recusa, ou com motivo que não é motivo ("não era o ideal", "não fazia sentido").
4. **Consequência só positiva.** A seção Consequências não registra nada que piora, encarece ou se perde.
5. **Gatilho não observável.** O Gatilho de revisão não é uma data, um número que se cruza, nem uma premissa nomeada que pode cair.
6. **Contexto preso à conversa.** O Contexto só faz sentido para quem participou da entrevista: fala em "o problema que discutimos", cita pessoa ou sistema nunca apresentado, ou pressupõe fato que o arquivo não registra.
7. **Campo vazio ou placeholder.** Campo do modelo mantido com `<...>`, "a definir" ou "TBD".
8. **Fora de escopo ausente ou falso.** A seção não existe, ou repete a Decisão em vez de delimitar o que não foi decidido.

### SPEC

1. **Critério de aceitação subjetivo.** O critério não diz como verificar, ou o resultado esperado depende de julgamento: "rápido", "estável", "intuitivo", "funciona bem". Bloqueante.
2. **Afirmação técnica sem prova.** A Solução escolhida ou as Dependências afirmam algo sobre biblioteca, API ou limite externo que não tem linha correspondente na tabela de Viabilidade verificada, ou cuja linha não cita fonte e data. Bloqueante.
3. **Decisão do PDR sem cobertura.** Algo que a Decisão do PDR de origem determina não aparece em nenhuma entrada, saída, comportamento ou critério. Bloqueante.
4. **Spec que decide sozinha.** A spec fixa escolha de produto que o PDR não tomou. Bloqueante — isso é PDR novo.
5. **Entrada pela metade.** Entrada sem tipo, sem validação, ou sem o que acontece quando chega inválida.
6. **Saída pela metade.** Saída sem destino, sem formato, ou sem a condição que a produz.
7. **Só o caminho feliz.** A seção Casos de erro está vazia, ou não cobre condição de falha que as Entradas e Dependências tornam possível.
8. **Critério solto.** Critério de aceitação que nenhuma entrada, saída ou comportamento sustenta.
9. **Dependência incompleta.** Dependência externa sem versão ou sem licença.
10. **Referência não consultada.** URL na tabela de Referências sem o que foi tirado dela ou sem data de consulta.
11. **Lista aberta.** "Etc.", "entre outros", "e assim por diante" ou reticências em lista que precisa ser fechada.
12. **Campo vazio ou placeholder.** Campo do modelo mantido com `<...>`, "a definir" ou "TBD".
13. **ID divergente.** O `<NNN>` do nome do arquivo ou do título não é o mesmo do PDR de origem citado no cabeçalho. Bloqueante — é esse número que amarra os artefatos da feature.

### PLAN

1. **Etapa sem verificação.** Etapa sem verificação, ou com verificação que ninguém consegue executar e ler o resultado: "conferir se ficou bom", "validar o comportamento". Bloqueante.
2. **Ciclo no grafo.** Duas ou mais etapas que dependem uma da outra, direta ou por cadeia. Bloqueante.
3. **Ordem impossível.** Etapa que usa arquivo, função ou dependência produzida por etapa posterior, ou por etapa da mesma onda. Bloqueante.
4. **Critério da spec sem etapa.** Critério de aceitação da spec de origem que nenhuma etapa atende. Bloqueante.
5. **Paralelismo falso.** Etapas declaradas paralelizáveis que escrevem no mesmo arquivo, ou entre as quais existe dependência declarada.
6. **Efeito colateral não levantado.** Arquivo com ação `alterar` ou `remover` no Mapa de arquivos sem linha correspondente em Efeitos colaterais, e sem afirmação de que nada depende dele.
7. **Mapa furado.** Caminho citado numa etapa e ausente do Mapa de arquivos, ou linha do mapa que nenhuma etapa realiza.
8. **Etapa sem destino.** Etapa que não atende critério nenhum da spec nem habilita outra etapa que atenda.
9. **Etapa grande demais.** A etapa entrega mais de uma coisa — "e" ligando trabalhos que se verificam separadamente, ou verificação que precisa de mais de um resultado para aprovar.
10. **Mudança sem volta.** Etapa que altera migração, dado existente, contrato publicado ou configuração de ambiente sem "Como desfazer" concreto.
11. **Dependência incompleta.** Dependência a instalar sem versão, sem licença ou sem o comando exato.
12. **Ondas inconsistentes.** A tabela de Ordem de execução contradiz os campos "Depende de", ou o caminho crítico declarado não é a cadeia mais longa do grafo.
13. **Campo vazio ou placeholder.** Campo do modelo mantido com `<...>`, "a definir" ou "TBD".
14. **ID divergente.** O `<NNN>` do arquivo ou do título não é o mesmo da spec de origem. Bloqueante.
15. **Slug divergente.** O `<slug>` do nome do arquivo não é o mesmo da spec de origem.

### TASKS

1. **Tarefa que decide.** A tarefa devolve uma escolha ao executor: "escolha a melhor abordagem", "ajuste conforme o projeto", "se aplicável", "conforme necessário". Bloqueante.
2. **Tarefa presa ao contexto.** Só se entende tendo lido o plano, a spec ou a conversa: fala em "o arquivo de configuração", "o serviço que criamos", "conforme decidido", sem nomear. Bloqueante.
3. **Verificação não executável.** Sem comando literal que dê para copiar e rodar, ou sem o resultado exato que aprova. Bloqueante.
4. **Dependência para frente.** "Depende de" apontando tarefa de número maior, ou pré-condição que só uma tarefa posterior produz. Bloqueante.
5. **Etapa do plano sem tarefa.** Etapa do plano de origem que nenhuma tarefa realiza. Bloqueante.
6. **Declarações em conflito.** A mesma tarefa aparece no "Depende de" e no "Paralelizável com" de outra, ou esses campos contradizem o bloco em que as tarefas estão. Bloqueante.
7. **Bloco mal formado.** Tarefas no mesmo bloco com dependência entre si, direta ou por cadeia, com o mesmo arquivo em "Arquivos", ou com o mesmo recurso em "Recursos". Bloqueante.
8. **Dependência escondida entre irmãs.** A pré-condição de uma tarefa é a pós-condição de outra tarefa do mesmo bloco. Existe dependência real que "Depende de" não declarou. Bloqueante.
9. **Mapa de execução ausente ou incoerente.** O mapa não existe, contradiz os campos das tarefas, ou repete arquivo ou recurso entre duas tarefas do mesmo bloco. Bloqueante.
10. **Caminho que não é arquivo.** "Arquivos" traz pasta, curinga ou "todos os arquivos de". Isso cega a comparação de colisão entre tarefas paralelas. Bloqueante.
11. **Escrita da verificação não declarada.** O comando de verificação escreve lockfile, cache, diretório de build, cobertura, migração ou artefato equivalente, e esses caminhos não estão em "Arquivos".
12. **Recurso compartilhado não declarado.** A tarefa usa banco, porta, serviço, variável de ambiente ou diretório temporário fixo, e o campo "Recursos" está vazio.
13. **Pré-condição reescrita.** A pré-condição não é o estado inicial do repositório nem a pós-condição, palavra por palavra, de uma das tarefas em "Depende de".
14. **Leitura na lista de escrita.** "Arquivos" cita arquivo que a tarefa apenas lê — isso cega a checagem de colisão entre tarefas paralelas.
15. **Tarefa órfã.** Tarefa que não aponta nenhuma etapa do plano.
16. **Tarefa composta.** Entrega mais de uma coisa, ou a verificação precisa de mais de um resultado para aprovar.
17. **Caminho inexato.** Arquivo citado sem caminho completo, ou caminho que não existe em `resources/` e que nenhuma tarefa anterior cria.
18. **Caixa marcada cedo.** Tarefa entregue com `- [x]` antes da execução, ou linha de Progresso incoerente com as caixas.
19. **Tarefa sem volta.** Tarefa que mexe em estado difícil de desfazer sem "Se falhar" concreto.
20. **Numeração furada.** Buraco na sequência `T<N>`, número repetido ou reaproveitado.
21. **Campo vazio ou placeholder.** Campo do modelo mantido com `<...>`, "a definir" ou "TBD".
22. **ID divergente.** O `<NNN>` do arquivo ou do título não é o mesmo do plano de origem. Bloqueante.
23. **Slug divergente.** O `<slug>` do nome do arquivo não é o mesmo do plano de origem.
### EXEC

1. **Conclusão sem prova.** Tarefa com resultado `concluída` sem comando de verificação, sem código de saída ou sem saída registrada. Bloqueante.
2. **Contaminação não declarada.** Arquivo tocado que não consta dos "Arquivos" da tarefa e não aparece na tabela de Contaminação verificada. Bloqueante.
3. **Conserto disfarçado de achado.** Um item de Achados descreve algo no mesmo arquivo que a rodada alterou fora da tarefa — foi consertado em vez de registrado. Bloqueante.
4. **Ordem violada.** Tarefa executada antes daquela de que depende, ou bloco iniciado com o anterior incompleto. Bloqueante.
5. **Paralelismo indevido.** Duas tarefas com dependência entre si, com arquivo em comum ou com recurso em comum, despachadas ao mesmo tempo para agentes diferentes. Bloqueante.
6. **Rodadas concorrentes.** O log registra execução iniciada com outra rodada da mesma feature em curso, ou com o arquivo de tarefas já em `Status: em execução`. Bloqueante.
7. **Bloco sem conferência.** Bloco encerrado e o seguinte aberto sem linha correspondente na tabela de Conferência por bloco. Bloqueante — é o que permite o resto da rodada correr sobre estado sujo.
8. **Inventário inicial ausente.** A seção Inventário inicial está vazia, sem o comando que a levantou, ou sem o caminho do inventário completo: sem base de comparação, nenhuma conferência posterior prova coisa alguma. Bloqueante.
9. **Rodada sobrescrita.** A tabela de Rodadas tem uma linha só num log que registra retomada, ou o registro anterior sumiu. Bloqueante — retomada acrescenta, nunca apaga.
10. **Instalação pelo executor.** A trilha registra instalação de pacote ou ferramenta dentro de uma tarefa, em vez de feita pelo orquestrador contra as Dependências do plano.
11. **Verificação trocada.** O comando registrado difere do comando escrito na tarefa e não há linha correspondente em Desvios.
12. **Aprovação com falha.** Tarefa `concluída` com código de saída diferente do que a tarefa define como aprovação.
13. **Tarefa sumida.** Tarefa do arquivo de origem ausente do Registro por tarefa e não contabilizada como não iniciada.
14. **Resumo incoerente.** Os números do Resumo não batem com o Registro por tarefa nem com as caixas marcadas no arquivo de tarefas.
15. **Destrutivo sem confirmação.** Comando destrutivo na trilha sem linha correspondente na tabela de Confirmações destrutivas.
16. **Sugestão executada.** Item de Achados cuja "sugestão não executada" aparece realizada na rodada.
17. **Trilha incompleta.** Comando citado no Registro por tarefa e ausente da tabela de Comandos executados.
18. **Campo vazio ou placeholder.** Campo do modelo mantido com `<...>`, "a definir" ou "TBD".
19. **ID divergente.** O `<NNN>` do arquivo ou do título não é o mesmo do arquivo de tarefas de origem. Bloqueante.
20. **Slug divergente.** O `<slug>` do log, ou o do arquivo de inventário que ele aponta, não é o mesmo do arquivo de tarefas de origem.

### ARCHIVE

1. **Feature em aberto.** A pasta foi arquivada com log de execução `em execução` ou `interrompida`, ou com caixa desmarcada no arquivo de tarefas, e o README não registra abandono nem o ponto de parada. Bloqueante.
2. **Artefato faltando.** A pasta arquivada não tem os cinco artefatos — PDR, spec, plano, tarefas e log — mais o README, e o README não registra o abandono nomeando os estágios que nunca existiram. Bloqueante.
3. **Sobra na origem.** Ainda existe arquivo daquela feature em `.spec/specs/`. Bloqueante — foi cópia, não mudança.
4. **Entrada não aprovada.** O `.spec/shared/` ganhou entrada que o README da feature não registra como colhida.
5. **Colheita específica demais.** Entrada em `.spec/shared/` que só vale para esta feature: cita tarefa, arquivo ou decisão sem alcance além dela.
6. **Conflito silencioso.** Entrada nova em `.spec/shared/` que contradiz entrada existente sem que o conflito esteja apontado.
7. **Achado perdido.** Achado do log com alcance além da feature que não aparece nem em `.spec/shared/` nem no README como descartado.
8. **README raso.** O README não diz o que a feature entregou, quando abriu e fechou, ou o que subiu para o compartilhado.
9. **Código tocado.** Há sinal de alteração em `resources/` nesta etapa. Bloqueante.
10. **Numeração reaproveitada.** Já existe outra pasta arquivada com o mesmo `<NNN>`. Bloqueante.
11. **Slug da pasta errado.** O `<slug>` da pasta arquivada não é o da spec de origem.
```

### 4.2 `.claude/agents/executor.md`

Executa uma tarefa por vez, isolado. É o único agente do fluxo que escreve em `resources/`.

```markdown
---
name: executor
description: Executa uma única tarefa de uma lista de tarefas do fluxo SDD, exatamente como escrita, e devolve relatório. Invocado uma vez por tarefa pela skill execute, nunca pelo usuário.
tools: Read, Grep, Glob, Edit, Write, Bash
---

# Executor

Você recebe uma tarefa e nada mais: não viu o PDR, a spec, o plano, as outras tarefas nem a conversa que gerou tudo isso. Isso é proposital. O que não estiver escrito na tarefa não existe para você.

## O que você faz

Exatamente o que a tarefa manda, nos arquivos que ela lista, e depois roda o comando de verificação que ela traz, exatamente como está escrito.

Antes de criar, confira se já existe; antes de alterar, confira se já não está no estado esperado. A tarefa pode estar sendo reexecutada depois de uma rodada interrompida, e rodar duas vezes não pode duplicar nada.

## O que você nunca faz

- Não altera arquivo fora da lista "Arquivos" da tarefa. Nenhum, por nenhum motivo.
- Não conserta o que está errado ao lado. Não formata, não renomeia, não refatora, não remove código morto, não atualiza dependência.
- Não acrescenta teste, documentação, comentário ou registro que a tarefa não peça.
- Não instala nada. Nem pacote, nem ferramenta, nem extensão. Tarefa que precise de instalação faz você parar e reportar: quem instala é quem te chamou.
- Não toca em `.spec/`. Não marca checkbox, não edita a lista de tarefas, não escreve no log.
- Não completa o que falta na tarefa e não decide o que ela não decidiu.
- Não altera o comando de verificação, nem o resultado esperado, para fazer a tarefa passar.
- Nunca `sudo`.

## Quando parar

Pare e reporte, sem tentar contornar, se: a verificação falhar; um arquivo fora da lista precisar mudar; um arquivo que a tarefa manda alterar não existir; a tarefa for ambígua, incompleta ou exigir uma decisão; o comando pedir confirmação, credencial ou acesso de rede não previsto; ou a tarefa exigir instalação.

Parar não é falhar. Seguir adivinhando, sim.

## O relatório

Termine sempre com, nesta ordem:

1. **Arquivos tocados** — todo caminho que você criou, alterou ou removeu. Liste tudo, inclusive o que o comando de verificação escreveu.
2. **Comando** — o que você rodou, literalmente.
3. **Código de saída** — o número.
4. **Saída** — o trecho que importa.
5. **Resultado** — concluída ou interrompida, e por quê.
6. **Achados** — o que você viu de errado e deliberadamente não consertou, com o caminho. Esta lista é o valor que você entrega além do trabalho; não a deixe vazia por preguiça nem a encha de trivialidade.
```
