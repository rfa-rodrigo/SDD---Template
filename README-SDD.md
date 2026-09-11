# Template SDD

Spec-driven development em seis estágios: **pdr → spec → plan → tasks → execute → archive**. O artefato escrito é autoritativo, o código é derivado dele, e cada estágio tem um portão de aprovação humana antes do próximo começar.

## Instalação

Antes de abrir a sessão, coloque o `CLAUDE.md` na raiz do projeto — são as diretrizes de comportamento que valem para todas as sessões. Ele é carregado na abertura, e só na abertura: arquivo posto depois não entra na sessão que já está rodando.

Com ele no lugar, coloque o `SETUP-SDD.md` na raiz, abra a sessão e diga:

```
Leia o SETUP-SDD.md na raiz deste projeto e execute-o do início ao fim, seguindo as regras de execução dele.
```

Nada é sobrescrito. Ao terminar, o agente relata o que criou e o que pulou — e nada além disso.

O setup não instala nada, não pede rede e não depende de ferramenta de terceiro. Ele cria pastas e escreve arquivos, e para na primeira coisa que já existir em vez de sobrescrever.

## O que o setup entrega

Uma skill é um arquivo: `.claude/skills/<nome>/SKILL.md`, com `name` e `description` no cabeçalho e as instruções no corpo. A seção 2.1 fixa esse contrato, e cada subseção de 2.2 a 2.7 traz a `description` literal e o corpo da sua skill. O agente copia — não redige, não parafraseia, não resume.

A `description` é o que decide quando a skill dispara, e as seis dizem por escrito que só disparam por invocação explícita do usuário. É o portão do fluxo: skill que dispara porque o assunto surgiu começa a entrevistar, planejar ou executar sem ninguém ter pedido. Por isso o texto vem pronto, em vez de ser redigido a cada instalação.

```
projeto/
├── .claude/
│   ├── skills/          pdr, spec, plan, tasks, execute, archive
│   └── agents/
│       ├── revisor.md   audita artefatos, só leitura, nunca corrige
│       └── executor.md  executa uma tarefa por vez, único que escreve código
├── .spec/
│   ├── specs/           os artefatos da feature em andamento
│   │   └── archive/     uma pasta por feature concluída
│   ├── templates/       pdr, spec, plan, tasks, exec
│   └── shared/
│       └── memory.md      o que atravessa features
└── resources/           a raiz do código
```

## O fluxo

Cada estágio roda, você confere, e só então o próximo começa. Nenhum dispara atrás do anterior, e nenhum se autoaprova.

    /pdr        conduz a conversa e registra a decisão      → PDR-<NNN>
    /spec       pesquisa, prova viabilidade, especifica      → SPEC-<NNN>
    /plan       lê o código e desenha a engenharia          → PLAN-<NNN>
    /tasks      decompõe em tarefas atômicas com checkbox   → TASKS-<NNN>
    /execute    executa, isolado, e registra o que houve    → EXEC-<NNN>
    /archive    arquiva os artefatos e registra a memória da feature

Os cinco artefatos vivem em `.spec/specs/` e carregam o mesmo `<NNN>` — o ID da feature, perguntado ao usuário no PDR e conferido contra as features ativas **e** arquivadas antes de ser aceito. O slug segue o mesmo caminho: nasce no PDR e é repetido, sem alteração, por todos os artefatos seguintes e pela pasta arquivada.

## As ideias que sustentam o fluxo

**Decidir e especificar são coisas diferentes.** O PDR fixa a decisão e o porquê; a spec fixa o como. Uma spec que decide o que o PDR não decidiu é achado bloqueante — isso é PDR novo, não spec maior.

**Viabilidade antes de proposta.** A `spec` pesquisa a internet antes de propor: repositório, documentação oficial, notas de versão, licença. Abrir a fonte é obrigatório — nada citado só pelo resultado de busca. Para cada abordagem ela lista o que precisa ser verdade para funcionar e confirma item por item numa fonte que abriu. O que não se confirma vira risco declarado ou derruba a abordagem.

**A verificação nasce antes do código.** Toda etapa do plano e toda tarefa carregam o comando que prova que funcionou e o resultado que aprova. Etapa cuja verificação você não sabe nomear está grande demais ou mal definida.

**Paralelismo é derivado, não desejado.** Duas tarefas só correm juntas se não houver dependência entre elas, se não escreverem no mesmo arquivo e se não usarem o mesmo recurso. Escrita no mesmo arquivo é dependência mesmo sem dependência lógica; e o que o comando de verificação escreve — lockfile, cache, `dist/`, cobertura — conta como escrita. É assim que se descobre que rodar `install` em paralelo nunca foi paralelo.

Derivado, e ainda assim opcional: o arquivo de tarefas carrega um campo `Paralelismo`, que nasce `não` e só vira `sim` se você liberar em palavras. Ele governa o despacho, nunca a análise — os blocos continuam sendo montados e conferidos dos dois jeitos, porque é a montagem do bloco que prova que as dependências estão certas. Rodar em série é sempre seguro; o campo existe para que paralelizar seja escolha sua, e não padrão do agente.

**Atômico não é minúsculo.** É indivisível sem perder valor: uma mudança que se verifica sozinha e caberia num commit. Oito passos que só valem juntos eram um.

**A cerimônia não se paga em tudo.** O ciclo vale para trabalho multi-sessão, com invariantes que não podem quebrar, ou que vai ser retomado daqui a meses. Trabalho trivial fica de fora.

## Contra a contaminação

O pesadelo do fluxo é o agente fazer o que ninguém pediu. A defesa é em quatro camadas, e é literal.

**Isolamento.** Toda tarefa roda no subagente `executor`, mesmo quando é a única do bloco. Ele recebe o bloco da tarefa e nada mais: não vê o PDR, a spec, o plano, as outras tarefas nem a conversa. O que não está escrito na tarefa não existe para ele. Quem orquestra não executa.

**Declaração de escopo literal.** Todo despacho termina com o mesmo bloco, copiado sem adaptar: apenas os arquivos listados, nada de consertar o que está errado ao lado, nada de teste, documentação, formatação ou refatoração não pedidos, e verificação que falha é parada — nunca ajuste da verificação. Isso não é enfeite: sem declaração explícita de escopo no prompt, a taxa de ação fora do pedido em agentes de código sobe de perto de zero para perto de um em seis.

**Prova por inventário.** Antes de começar, a execução registra caminho, tamanho e data de todo arquivo em `resources/`, e guarda essa lista num arquivo próprio. Ao fim de **cada bloco** — não só no fim da rodada — ela compara. Tudo que mudou tem de estar na união dos arquivos declarados nas tarefas executadas. O relatório do subagente é o que ele diz ter feito; o inventário é o que aconteceu.

**A válvula.** O executor vai ver coisa errada que não é dele. Não conserta: registra em **Achados**, com onde está e por que importa, marcado como não executado. É onde a vontade de consertar vira insumo para a próxima rodada em vez de estrago nesta. O `revisor` audita exatamente isso — achado descrevendo algo no mesmo arquivo que a rodada alterou fora da tarefa é conserto disfarçado, e é bloqueante.

## O revisor

Um subagente com `tools: Read, Grep, Glob` que lê o artefato e o modelo, procura defeitos numa **lista fechada** e reporta por severidade. Nunca corrige, nunca julga o mérito da decisão, nunca comenta estilo, e pode — deve — responder `Nenhum defeito encontrado.`

A lista fechada é o ponto. Um revisor solto, com a missão de "ver se dá para melhorar", sempre acha algo, porque melhoria é ilimitada; o retorno vira ruído genérico e por volta do terceiro uso você aprende a ignorá-lo. Cada etapa tem a sua lista, com cada defeito nomeado e os bloqueantes marcados como tais: PDR com 8 itens, SPEC com 14, PLAN com 15, TASKS com 22, EXEC com 20, ARCHIVE com 9. Etapa que não estiver na lista, ele recusa em vez de improvisar critério.

E ele serve porque é cego: quem escreveu o artefato tinha a conversa inteira no contexto, e por isso não enxerga o que falta. O revisor recebe só o arquivo — que é a condição de quem vai ler daqui a um ano.

## Os portões

Todo artefato carrega `Status` no topo, e é esse campo, não a memória da conversa, que a etapa seguinte lê. O PDR nasce `proposto` e a spec exige `aceito`; a spec nasce `rascunho` e o plano exige `aprovada`; e assim por diante.

Quem promove é a própria skill, **depois** de o usuário aprovar em palavras. Nunca por conta própria, nunca tratando silêncio como aprovação. Artefato que chega com o status já avançado sem registro da aprovação é achado bloqueante em todas as etapas.

## O que atravessa features

`.spec/shared/memory.md` é a memória do projeto e o único arquivo que não é clonado: é preenchido no lugar, pelo `archive`, entrada por entrada, e só com o seu sim. Vocabulário, stack e versões fixadas, convenções, invariantes, limites e políticas, decisões permanentes, **caminhos já tentados** e armadilhas conhecidas.

O PDR, a spec e o plano o leem antes de qualquer pergunta, e o que está lá não se rediscute a cada rodada — contradizê-lo é achado a reportar, não escolha a fazer. Seção com campos entre `<>` está vazia: placeholder não é convenção, não é invariante e não é decisão.

No arquivamento, o `archive` relê o que a feature ensinou e propõe o que sobe. O critério é um só: **vale para a próxima feature?** Se vale só para esta, fica no README da pasta arquivada. Você aprova entrada por entrada, e o `archive` escreve. Encher a memória de detalhe morto é a forma mais rápida de fazer todo mundo parar de lê-la.

## O arquivamento

O `archive` não toca em `resources/`. Ele confere o portão de fechamento — log `concluída`, todas as caixas marcadas, nenhuma contaminação em aberto —, **move** tudo que está na raiz de `.spec/specs/` para `.spec/specs/archive/<NNN>-<slug>/`, escreve ali o README com a memória da feature, e propõe o que dela sobe para a memória do projeto. Não apaga nada.

Feature abandonada também se arquiva, com menos artefatos do que os cinco, desde que o README da pasta registre por que foi abandonada e que estágios nunca existiram. Abandono com rastro é legítimo; abandono silencioso não.

## Recuperação

A execução se recusa a começar se já houver rodada em curso — dois orquestradores na mesma feature corrompem o trabalho um do outro. Mas log em `Status: em execução` sem rodada viva é rodada **morta**, não rodada em curso: a skill mostra o log ao usuário, pede confirmação de que nada está rodando, fecha aquela linha como `interrompida` e retoma, pulando as tarefas já marcadas. Rastro de rodada anterior nunca se apaga — a retomada acrescenta uma linha na tabela de Rodadas.

Por isso o `executor` confere estado antes de agir: antes de criar, se já existe; antes de alterar, se já não está como deveria. Uma tarefa pode estar sendo reexecutada depois de uma interrupção, e rodar duas vezes não pode duplicar nada.

## A regra que quase todo mundo quebra

Durante a execução você vai descobrir que uma etapa estava errada. Quando acontecer, **pare e suba**: conserte no artefato de origem e refaça o que dele depende. Consertar direto no código "só desta vez" é o ponto exato em que SDD vira documentação morta — e aí você pagou o overhead sem receber o benefício.
