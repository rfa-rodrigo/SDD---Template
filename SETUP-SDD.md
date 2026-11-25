# SETUP — estrutura de spec-driven development
Autor: Rodrigo Figueiredo Abdo

Instruções para o agente. Execute na raiz do projeto atual.

**Regras de execução:**

1. Copie o conteúdo dos blocos **literalmente**. Não parafraseie, não resuma, não traduza, não "melhore" o texto. Os placeholders entre `<>` ficam como estão — quem preenche é o usuário.
2. **Nunca sobrescreva** arquivo que já exista. Se existir, pule e registre.
3. Ao final, relate o que foi criado e o que foi pulado, e nada além disso.

---

## 0. Pré-fase — conferir o `CLAUDE.md`

Antes de qualquer outra coisa, verifique se existe um `CLAUDE.md` na raiz do projeto.

Se existir, siga para a seção 1.

Se não existir, pare e informe o usuário: o `CLAUDE.md` carrega as diretrizes de comportamento e é lido na abertura da sessão, e só nela — colocá-lo agora não tem efeito sobre esta sessão, seria preciso encerrar e reabrir. Pergunte se ele prefere parar o setup para colocar o arquivo e reabrir a sessão, ou continuar mesmo assim. Espere a resposta: não escolha por ele e não trate silêncio como resposta.

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
    └── archive/        só em projeto com trabalho cíclico
```

Antes de criar as pastas, pergunte ao usuário se o projeto tem trabalho cíclico: a mesma tarefa repetida de tempos em tempos sobre arquivos diferentes. Espere a resposta: não escolha por ele e não trate silêncio como resposta. Se tiver, crie também `resources/archive/`; se não tiver, não crie.

`.spec/specs/` guarda **uma feature por vez**. Concluída, ela é arquivada e a raiz volta a ficar vazia. Duas features abertas ali é erro, não paralelismo.

`resources/` é a raiz do código, é onde as informações externas serão armazenadas.

`resources/archive/` só existe em projeto com trabalho cíclico. É onde ficam guardados os arquivos dos ciclos já feitos: terminado um ciclo, o usuário move para lá os arquivos daquele ciclo, e `resources/` fica pronto para o próximo. Quem move é sempre o usuário — nenhuma skill do fluxo move ou arquiva nada ali. Em projeto sem trabalho cíclico, `resources/` é fixo e a pasta não existe. Não confundir com `.spec/specs/archive/`, que guarda os artefatos das features.

Não crie pasta que ninguém use — toda pasta acima tem dono declarado neste documento.

### Fluxo

`PDR -> Spec -> Plan -> Task -> Execute -> Archive`

---

## 2. Skills

As seis skills do fluxo são escritas por você, aqui, a partir dos blocos das subseções 2.2 a 2.7. Não instale nada e não dependa de nenhuma ferramenta de terceiro: uma skill é um arquivo, e a 2.1 diz exatamente qual.

### 2.1 O contrato do arquivo de skill

Cada uma das seis skills é um único arquivo, em `.claude/skills/<nome>/SKILL.md`, com esta forma:

```
---
name: <nome>
description: <o texto literal dado na subseção da skill>
---

<o bloco da subseção, copiado literalmente>
```

Três regras, e nenhuma delas é opcional:

1. O `name` é o nome da skill, igual ao da pasta: `pdr`, `spec`, `plan`, `tasks`, `execute`, `archive`.
2. A `description` é o texto literal que cada subseção fornece. Não redija a sua. É a `description` que decide quando a skill dispara, e todas as seis dizem a mesma coisa por escrito: só por invocação explícita do usuário. Esse é o portão do fluxo — uma skill que dispara porque o assunto surgiu na conversa começa a entrevistar, planejar ou executar sem ninguém ter pedido.
3. O corpo é o bloco da subseção, copiado literalmente, pela regra de execução 1 deste documento. Não parafraseie, não resuma, não reorganize.

Se você conhece alguma ferramenta de criação de skills, não a use aqui. Elas costumam orientar descrições agressivas, para a skill disparar sempre que puder — o oposto exato do que estas seis precisam.

### 2.2 Skill do PDR

Primeira etapa do fluxo. Crie `.claude/skills/pdr/SKILL.md` conforme o contrato da 2.1, com esta `description`:

```
Conduz a entrevista de decisão de produto e escreve o Product Decision Record (PDR) da feature em `.spec/specs/`. Use SOMENTE quando o usuário invocar esta skill explicitamente. Nunca dispare porque uma decisão de produto surgiu na conversa, porque o usuário mencionou que precisa decidir algo, ou porque parece útil: o portão é o usuário invocar o comando.
```

E com o bloco abaixo como corpo, copiado literalmente:

```
**O que ela faz.** Conduz uma conversa com o usuário e, no fim, escreve um Product Decision Record — o registro da decisão de produto que está sendo tomada. Ela não escreve código, não altera nada em `resources/`, e não produz nenhum artefato além do PDR.

**Quando dispara.** Só quando o usuário a invoca explicitamente. A `description` deve dizer isso com todas as letras, para o modelo não começar a entrevistar ninguém porque o assunto surgiu na conversa.

**Reabertura.** Se já existe PDR em `.spec/specs/`, há feature aberta, e só há uma por vez. Pergunte ao usuário se a invocação é correção desse PDR. Se não for, não abra outra feature: diga que a aberta precisa ser arquivada antes — pela `archive`, como abandonada se for o caso — e pare. Se for, a invocação é correção, não criação: não copie o modelo de novo nem peça outro número. Se a feature já tem log de execução, não reabra: há código escrito sobre esta decisão, e mudá-la agora é feature nova — diga isso ao usuário e pare; o caminho é fechar esta feature pela `archive`, como abandonada se for o caso, e abrir outra. Sem log, devolva o `Status` a `proposto`, aplique a correção que o usuário apontar, rode a revisão até zero bloqueantes e zero graves e peça a aprovação outra vez. A spec, o plano e o arquivo de tarefas que já existirem voltam a `rascunho`: foram escritos sobre o que mudou, e cada um se realinha reabrindo a sua etapa, na ordem do fluxo.

**A conversa.**

- Uma pergunta por mensagem. Nunca empilhe perguntas.
- Prefira alternativas fechadas quando o assunto permitir; pergunta aberta quando não permitir.
- Antes da primeira pergunta, leia o estado do projeto em `.spec/shared/` e o que houver em `resources/`. Não pergunte o que o repositório já responde.
- Leia `.spec/shared/` antes de qualquer outra coisa. O que está ali vale para todas as features — vocabulário, convenções, invariantes, limites, versões fixadas, caminhos já tentados e armadilhas conhecidas — e não se rediscute a cada rodada. Contradizer o que está lá é achado a reportar, não escolha a fazer.
- Investigue nesta ordem: que problema motiva a decisão, quem sente esse problema, o que já foi tentado, que restrições existem (prazo, técnica, gente, dinheiro), o que está fora de escopo, e como se saberá que a decisão foi acertada.
- Não trate resposta vaga como resposta. Se o usuário disser "precisa ser rápido", pergunte rápido em relação a quê e medido como.

**As alternativas.**

Depois de entender o problema, apresente de duas a três alternativas reais — cada uma com o que custa e o que entrega — e diga qual você recomenda e por quê. Alternativa que ninguém escolheria não conta: se só existe um caminho viável, diga isso, em vez de inventar duas opções de enfeite.

**O portão.**

Não escreva o PDR antes da aprovação. Apresente na conversa, em texto, o que vai ser registrado — decisão, alternativas descartadas, consequências — e espere um sim explícito. Concordar com um trecho não é aprovar o documento.

**A escrita.**

Recebido o sim: copie `.spec/templates/pdr.md` para `.spec/specs/PDR-<NNN>-<slug>.md` e preencha a cópia. `<NNN>` é o ID da feature, sempre pergunte ao usuário. Antes de aceitar o número, liste `.spec/specs/` **e** `.spec/specs/archive/`: número de feature não se reaproveita, e a pasta de specs fica vazia depois de cada arquivamento — quem só olha ali não enxerga nenhuma feature já fechada. Se o número pedido aparecer em qualquer um dos dois, recuse e proponha o próximo livre. `<slug>` é a feature em três a cinco palavras, minúsculas, separadas por hífen. É ele que todos os artefatos seguintes e a pasta arquivada repetem, sem alteração — escolha pensando em quem vai procurar a feature depois. O modelo em `.spec/templates/` nunca é editado.

Preencha todos os campos do modelo. Campo que você não consegue preencher é pergunta que faltou fazer: volte e pergunte, não escreva "a definir". Tabela ou seção do modelo sem o que registrar leva `nenhum` escrito, nunca o placeholder.

**Régua do conteúdo.**

- Contexto descreve as forças em jogo no momento da decisão, não a solução. Quem ler daqui a um ano precisa entender por que aquilo fazia sentido então.
- Decisão é uma frase afirmativa, no presente, dizendo o que será feito. "Avaliar se vale a pena" não é decisão.
- Toda alternativa descartada carrega o motivo da recusa. Sem motivo, ela não entra na tabela.
- Consequências incluem o que piora. PDR que só lista benefício não foi pensado, foi vendido.
- Gatilho de revisão é observável: uma data, um número que se cruza, uma premissa que pode cair.

**Antes de entregar.**

Releia o que escreveu procurando quatro defeitos: campo com placeholder ou "a definir"; seção que contradiz outra; frase que admite duas leituras; e decisão que na verdade são duas disfarçadas de uma. Corrija os três primeiros na hora. No quarto caso, pare e proponha ao usuário dois PDRs separados — duas features, uma de cada vez.

**A revisão.**

Escrito o arquivo e feita a autorrevisão, invoque o subagente `revisor`, informando o caminho do PDR e a etapa `PDR`. Ele lê o arquivo, o modelo e a memória do projeto, procura defeitos numa lista fechada e devolve achados por severidade — corrigir não é função dele. A revisão é um laço, e ele só termina de um jeito: o `revisor` devolvendo zero bloqueantes e zero graves. Corrija os bloqueantes e os graves e invoque o `revisor` de novo — cada volta é uma invocação nova; ele não lembra da anterior, e é isso que a faz valer. Não há número máximo de voltas.

Enquanto houver bloqueante ou grave, o artefato não é entregue: não peça a revisão do usuário e não promova o `Status`. Não adianta apresentar ao autor um artefato que se sabe estar com defeito.

Dois casos você não corrige sozinho, e nenhum deles encerra o laço com defeito dentro:

- **O achado depende de uma resposta do usuário.** Pergunte, corrija com a resposta e invoque o `revisor` de novo.
- **O achado voltou depois de corrigido.** Não repita a mesma correção: mostre ao usuário o achado e o que você tentou, corrija como ele decidir e invoque o `revisor` de novo. O `revisor` só lê o arquivo; o que ele lê errado é trecho que admite a leitura errada, e se resolve reescrevendo o trecho, nunca descartando o achado.

Observação não entra no laço: você apresenta e o usuário decide.

Com zero bloqueantes e zero graves, informe o caminho do arquivo, quantas voltas houve, o que o `revisor` apontou e o que foi corrigido em cada uma, e peça a revisão do usuário.

**A aprovação.**

O PDR nasce `proposto` e é a spec que exige `aceito`. Quem promove é você, e só depois de o usuário aprovar em palavras: mude o `Status` para `aceito`, ou para `rejeitado` se ele recusar — PDR `rejeitado` encerra a feature, e o que resta é arquivá-la como abandonada. Nunca promova por conta própria, nunca trate silêncio como aprovação, e nunca deixe o arquivo em `proposto` depois de um sim — é esse campo, e não a memória da conversa, que a etapa seguinte lê. Se, em vez de aprovar, o usuário pedir mudança, faça a mudança e rode a revisão de novo, até zero bloqueantes e zero graves, antes de pedir a aprovação outra vez.
```

### 2.3 Skill das especificações

Segunda etapa do fluxo. Crie `.claude/skills/spec/SKILL.md` conforme o contrato da 2.1, com esta `description`:

```
Produz a especificação técnica (SPEC) de uma feature a partir de um PDR com Status aceito, pesquisando a internet e provando viabilidade antes de propor. Use SOMENTE quando o usuário invocar esta skill explicitamente, passando o ID da feature. Nunca dispare por conta própria depois de um PDR aprovado.
```

E com o bloco abaixo como corpo, copiado literalmente:

```
**O que ela faz.** Parte de um PDR aprovado e produz a especificação técnica da funcionalidade: entradas, saídas, comportamento esperado e critérios de aceitação objetivos. O PDR decidiu o quê e por quê; a spec fixa o como, com rigor suficiente para que quem for implementar não precise decidir nada sozinho. Ela não escreve código e não altera `resources/`.

**Quando dispara.** Só quando o usuário a invoca explicitamente, passando o número do PDR de origem. A `description` deve dizer isso.

**Reabertura.** Se já existe SPEC desta feature em `.spec/specs/`, a invocação é correção, não criação: não copie o modelo de novo. Se a feature já tem log de execução, não reabra: há código escrito sobre esta spec, e mudá-la agora é feature nova — diga isso ao usuário e pare; o caminho é fechar esta feature pela `archive`, como abandonada se for o caso, e abrir outra. Sem log, devolva o `Status` a `rascunho`, aplique a correção — a que o usuário apontar, ou a que o PDR corrigido exigir —, rode a revisão até zero bloqueantes e zero graves e peça a aprovação outra vez. O plano e o arquivo de tarefas que já existirem voltam a `rascunho`: foram escritos sobre o que mudou, e cada um se realinha reabrindo a sua etapa, na ordem do fluxo.

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
- Verifique se a solução já é consolidada ou se ainda é incerta.
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

Recebido o sim: copie `.spec/templates/spec.md` para `.spec/specs/SPEC-<NNN>-<slug>.md` e preencha a cópia. Tanto o `<NNN>` quanto o `<slug>` encontram-se no PDR, se precisar, leia-o — o slug não se reescreve aqui. Tabela ou seção do modelo sem o que registrar leva `nenhum` escrito, nunca o placeholder.

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

Escrita a spec e feita a autorrevisão, invoque o subagente `revisor`, informando o caminho do arquivo e a etapa `SPEC`. A revisão é um laço, e ele só termina de um jeito: o `revisor` devolvendo zero bloqueantes e zero graves. Corrija os bloqueantes e os graves e invoque o `revisor` de novo — cada volta é uma invocação nova; ele não lembra da anterior, e é isso que a faz valer. Não há número máximo de voltas.

Enquanto houver bloqueante ou grave, o artefato não é entregue: não peça a revisão do usuário e não promova o `Status`. Não adianta apresentar ao autor um artefato que se sabe estar com defeito.

Três casos você não corrige sozinho, e nenhum deles encerra o laço com defeito dentro:

- **O achado depende de uma resposta do usuário.** Pergunte, corrija com a resposta e invoque o `revisor` de novo.
- **O achado voltou depois de corrigido.** Não repita a mesma correção: mostre ao usuário o achado e o que você tentou, corrija como ele decidir e invoque o `revisor` de novo. O `revisor` só lê o arquivo; o que ele lê errado é trecho que admite a leitura errada, e se resolve reescrevendo o trecho, nunca descartando o achado.
- **O achado nasce no artefato de origem.** Ele se conserta lá, e você não edita artefato já aprovado. A etapa fica bloqueada: deixe este arquivo em `rascunho`, não peça aprovação, e diga ao usuário qual artefato de origem, qual seção e qual defeito. Corrigida a origem, a etapa é invocada de novo, retoma este mesmo arquivo e a auditoria recomeça do zero.

Observação não entra no laço: você apresenta e o usuário decide.

Com zero bloqueantes e zero graves, informe o caminho do arquivo, quantas voltas houve, o que o `revisor` apontou e o que foi corrigido em cada uma, e peça a revisão do usuário.

**A aprovação.**

A spec nasce `rascunho` e o plano exige `aprovada`. Depois de o usuário aprovar em palavras, mude o `Status` para `aprovada`. Nunca promova por conta própria nem trate silêncio como aprovação. Se, em vez de aprovar, o usuário pedir mudança, faça a mudança e rode a revisão de novo, até zero bloqueantes e zero graves, antes de pedir a aprovação outra vez.
```

### 2.4 Skill do planejamento

Terceira etapa do fluxo. Crie `.claude/skills/plan/SKILL.md` conforme o contrato da 2.1, com esta `description`:

```
Cruza uma SPEC aprovada com o código em `resources/` e escreve o plano de engenharia (PLAN): mapa de arquivos, dependências, efeitos colaterais, etapas, ondas e caminho crítico. Use SOMENTE quando o usuário invocar esta skill explicitamente, passando o ID da feature. Nunca dispare por conta própria depois de uma spec aprovada.
```

E com o bloco abaixo como corpo, copiado literalmente:

```
**O que ela faz.** Cruza uma spec aprovada com o código que existe em `resources/` e produz a estratégia de engenharia: que arquivos serão criados, alterados ou removidos; que dependências serão instaladas; que efeitos colaterais a mudança provoca na arquitetura; e quais etapas precisam ser feitas, em que ordem, quais dependem de quais e quais podem correr juntas. Ela não escreve código e não altera `resources/`.

**Quando dispara.** Só quando o usuário a invoca explicitamente, passando o ID da feature. A `description` deve dizer isso.

**Reabertura.** Se já existe PLAN desta feature em `.spec/specs/`, a invocação é correção, não criação: não copie o modelo de novo. Se a feature tem log de execução com `Status: em execução`, não reabra: feche a rodada pela `execute` primeiro. Com log `interrompida` ou `concluída`, reabra, mas o que já foi executado não se reescreve: leia o arquivo de tarefas, e etapa realizada por tarefa já marcada fica como está — a correção entra como etapa nova. Devolva o `Status` a `rascunho`, aplique a correção — a que o usuário apontar, ou a que a spec corrigida exigir —, rode a revisão até zero bloqueantes e zero graves e peça a aprovação outra vez. O arquivo de tarefas, se já existir, volta a `rascunho`: foi escrito sobre o que mudou, e se realinha reabrindo a etapa das tarefas.

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
- A verificação exercita o que a etapa entrega, não só o arquivo existir ou compilar. Etapa que cria lógica e verifica só sintaxe tem verificação que não verifica: o defeito aparece na etapa seguinte, ou nunca.
- Etapa entrega uma coisa. "E" ligando dois trabalhos que se verificam separadamente são duas etapas.
- Cada etapa declara: o que faz, os arquivos exatos, os recursos que usa — banco, porta, serviço, variável de ambiente —, a verificação, de que etapas depende, com quais pode correr em paralelo, que critérios da spec atende, e como se desfaz.
- Ordem lógica significa que nada depende do que ainda não existe. Percorra o grafo e confirme: para toda etapa, tudo de que ela precisa foi produzido antes.
- O grafo não tem ciclo. Duas etapas que dependem uma da outra são um recorte errado — refaça o recorte, não invente uma ordem.

**Paralelismo.**

Paralelismo se deriva das dependências, não do desejo. Duas etapas correm juntas quando nenhuma depende da outra, nenhuma escreve no arquivo em que a outra escreve, e nenhuma usa o recurso que a outra usa — banco, porta, serviço em execução, variável de ambiente, diretório temporário fixo. Escrita no mesmo arquivo é dependência, mesmo quando não há dependência lógica. E o que o comando de verificação escreve — lockfile, cache, `dist/`, cobertura — conta como escrita da etapa: é assim que se descobre que rodar `install` em paralelo nunca foi paralelo.

Esse é o mesmo critério que define um bloco de tarefas. Onda frouxa aqui vira bloco errado lá, e bloco errado quebra a execução.

Agrupe as etapas em ondas: a onda 1 são as etapas sem dependência; a onda N são as etapas cujas dependências foram todas satisfeitas até a onda anterior. Dentro de uma onda, tudo corre junto.

Nomeie o caminho crítico: a cadeia mais longa de etapas dependentes. É ela que define o piso do prazo — encurtar etapa fora dela não adianta nada.

**Cobertura.**

Nos dois sentidos: todo critério de aceitação da spec é atendido por pelo menos uma etapa, e toda etapa serve a pelo menos um critério — ou a uma etapa que sirva. Etapa que não serve a nada sai do plano.

**Reversão.**

Toda etapa que mexe em estado difícil de desfazer — migração, dado existente, contrato já publicado, configuração de ambiente — diz como se desfaz. A volta se planeja agora, no desenho, não quando quebrar.

**O portão.**

Antes de escrever o arquivo, apresente na conversa só a lista de etapas com suas dependências e as ondas, e espere um sim. É a decomposição que precisa ser conferida, não a prosa — um sim basta.

**A escrita.**

Recebido o sim: copie `.spec/templates/plan.md` para `.spec/specs/PLAN-<NNN>-<slug>.md` e preencha a cópia. Tanto o `<NNN>` quanto o `<slug>` encontram-se no PDR, se precisar, leia-o. Tabela ou seção do modelo sem o que registrar leva `nenhum` escrito, nunca o placeholder.

**Antes de entregar.**

Releia procurando: etapa sem verificação observável; ciclo no grafo; etapa que usa o que só existe depois; paralelismo declarado entre etapas que escrevem no mesmo arquivo; critério da spec sem etapa; arquivo citado numa etapa e ausente do mapa; mudança irreversível sem volta. Corrija na hora.

**A revisão.**

Escrito o plano e feita a autorrevisão, invoque o subagente `revisor`, informando o caminho do arquivo e a etapa `PLAN`. A revisão é um laço, e ele só termina de um jeito: o `revisor` devolvendo zero bloqueantes e zero graves. Corrija os bloqueantes e os graves e invoque o `revisor` de novo — cada volta é uma invocação nova; ele não lembra da anterior, e é isso que a faz valer. Não há número máximo de voltas.

Enquanto houver bloqueante ou grave, o artefato não é entregue: não peça a revisão do usuário e não promova o `Status`. Não adianta apresentar ao autor um artefato que se sabe estar com defeito.

Três casos você não corrige sozinho, e nenhum deles encerra o laço com defeito dentro:

- **O achado depende de uma resposta do usuário.** Pergunte, corrija com a resposta e invoque o `revisor` de novo.
- **O achado voltou depois de corrigido.** Não repita a mesma correção: mostre ao usuário o achado e o que você tentou, corrija como ele decidir e invoque o `revisor` de novo. O `revisor` só lê o arquivo; o que ele lê errado é trecho que admite a leitura errada, e se resolve reescrevendo o trecho, nunca descartando o achado.
- **O achado nasce no artefato de origem.** Ele se conserta lá, e você não edita artefato já aprovado. A etapa fica bloqueada: deixe este arquivo em `rascunho`, não peça aprovação, e diga ao usuário qual artefato de origem, qual seção e qual defeito. Corrigida a origem, a etapa é invocada de novo, retoma este mesmo arquivo e a auditoria recomeça do zero.

Observação não entra no laço: você apresenta e o usuário decide.

Com zero bloqueantes e zero graves, informe o caminho do arquivo, quantas voltas houve, o que o `revisor` apontou e o que foi corrigido em cada uma, e peça a revisão do usuário.

**A aprovação.**

O plano nasce `rascunho` e as tarefas exigem `aprovado`. Depois de o usuário aprovar em palavras, mude o `Status` para `aprovado`. Nunca promova por conta própria nem trate silêncio como aprovação. Se, em vez de aprovar, o usuário pedir mudança, faça a mudança e rode a revisão de novo, até zero bloqueantes e zero graves, antes de pedir a aprovação outra vez.
```

### 2.5 Skill das tarefas

Quarta etapa do fluxo. Crie `.claude/skills/tasks/SKILL.md` conforme o contrato da 2.1, com esta `description`:

```
Traduz um PLAN aprovado numa lista de tarefas atômicas com checkbox (TASKS), cada uma isolada e executável por um agente que não viu mais nada. Use SOMENTE quando o usuário invocar esta skill explicitamente, passando o ID da feature. Nunca dispare por conta própria depois de um plano aprovado.
```

E com o bloco abaixo como corpo, copiado literalmente:

```
**O que ela faz.** Traduz um plano aprovado numa lista cirúrgica de tarefas atômicas, encadeadas e marcáveis. Cada tarefa é uma instrução isolada de execução: quem for executá-la recebe aquele bloco e nada mais. Ela não escreve código e não altera `resources/`.

**Não gere uma skill por tarefa.** A entrada da tarefa já é o prompt do executor; uma skill em cima dela seria uma segunda descrição do mesmo trabalho, que diverge com o tempo e polui `.claude/skills/` com arquivos de uso único disputando atenção com os permanentes. O isolamento vem do subagente, não do arquivo. Skill só se justifica para procedimento que se repete entre tarefas e entre features — e essa é permanente, escrita uma vez, referenciada pelas tarefas. Caso seja necessária a criação de uma skill, coloque isso como tarefa.

**Quando dispara.** Só quando o usuário a invoca explicitamente, passando o ID da feature. A `description` deve dizer isso.

**Reabertura.** Se já existe TASKS desta feature em `.spec/specs/`, a invocação é correção, não criação: não copie o modelo de novo. Se a feature tem log de execução com `Status: em execução`, não reabra: feche a rodada pela `execute` primeiro. Com log `interrompida` ou `concluída`, reabra, mas o que já foi executado não se reescreve: tarefa marcada `- [x]` não se edita nem se desmarca, e a correção altera tarefa ainda não marcada ou entra como tarefa nova, com o próximo número de tarefa, em bloco novo, com o próximo número de bloco. O bloco novo entra no arquivo e no Mapa de execução antes dos blocos que dependem dele, ou ao final se nenhum depende: a ordem de execução é a ordem das linhas do mapa, não o número do bloco. As caixas marcadas ficam marcadas, e a linha de Progresso mantém as concluídas e atualiza o total. Devolva o `Status` a `rascunho`, aplique a correção — a que o usuário apontar, ou a que o plano corrigido exigir —, rode a revisão até zero bloqueantes e zero graves e peça a aprovação outra vez.

**Entrada.**

Leia o plano da feature em `.spec/specs/`. Se ele não estiver com `Status: aprovado`, pare e diga por quê. Leia a spec, para os critérios de aceitação, e o código em `resources/`, para os caminhos exatos — a tarefa cita caminho que existe, não caminho que deveria existir. Leia também o PDR da feature: é dele que vêm o `<NNN>` e o `<slug>`.

**A tradução.**

Cada etapa do plano vira uma ou mais tarefas, e nenhuma tarefa nasce fora do plano. Se algo necessário não está lá, o plano é que está incompleto: pare e diga, não conserte por conta própria. A ordem das tarefas respeita o grafo do plano, e as ondas do plano viram blocos de tarefas.

Um bloco é o conjunto de tarefas que podem correr ao mesmo tempo. É essa a propriedade que o define — não o tema, não o tamanho.

Uma onda do plano pode virar mais de um bloco: se as tarefas dela colidem em arquivo ou em recurso, quebre a onda em quantos blocos forem necessários até a propriedade valer. Bloco a mais custa tempo; bloco errado quebra a execução.

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

Cada tarefa declara de que tarefas depende e com quais pode correr em paralelo. Numeração contínua `T1`, `T2`, ... , sem buraco e sem reaproveitamento.

Seis regras fecham a corrente:

- **"Arquivos" lista só o que a tarefa escreve.** Arquivo que ela apenas lê fica no "O que fazer". É por essa lista que a execução detecta colisão entre tarefas paralelas — poluí-la com leitura cega a checagem.
- **"Depende de" e "Paralelizável com" não se cruzam.** Nenhuma tarefa aparece nos dois campos da mesma tarefa, e os dois campos concordam com o bloco. A redundância é proposital: é ela que torna o erro visível.
- **Tarefas no mesmo bloco não se cruzam nem se sobrepõem.** Só entram juntas tarefas sem dependência entre si, direta ou por cadeia, e sem nenhum arquivo em comum em "Arquivos".
- **Caminho é arquivo, nunca pasta nem padrão.** "Arquivos" lista caminhos de arquivo, um a um. Pasta, curinga ou "todos os arquivos de" cegam a comparação: duas tarefas parecem disjuntas e escrevem no mesmo lugar. Tarefa que precisa escrever ou remover uma pasta inteira lista os arquivos; se não dá para listar, declara a pasta, não é paralelizável com ninguém e fica sozinha no bloco.
- **A verificação também escreve.** Comando de teste, build, instalação, formatação ou geração de código quase sempre escreve fora dos arquivos da tarefa: lockfile, cache, `dist/`, cobertura, `__pycache__`, migração. Levante o que o comando escreve e inclua em "Arquivos" como qualquer outra escrita. Duas tarefas cujas verificações escrevem no mesmo lugar não vão para o mesmo bloco — e é assim que se descobre que rodar `install` em paralelo nunca foi paralelo.
- **Recurso que não é arquivo colide igual.** Banco de dados, porta, serviço em execução, variável de ambiente, credencial de uso exclusivo, diretório temporário fixo. Declare no campo "Recursos". Duas tarefas que usam o mesmo recurso não correm juntas, mesmo sem compartilhar um único caminho.

Feche o arquivo com o Mapa de execução: uma linha por bloco, com as tarefas do bloco, de que bloco ele depende, todos os arquivos escritos ali e todos os recursos usados. Arquivo repetido em duas tarefas do mesmo bloco é erro de recorte, não detalhe. É esse mapa que a execução usa para conferir o paralelismo antes de despachar qualquer coisa, e a ordem das linhas dele é a ordem de execução dos blocos.

**Os checkboxes.**

Cada tarefa começa com `- [ ]`. Quem executa marca `- [x]` assim que a verificação daquela tarefa passar, no próprio arquivo, uma por uma — nunca em lote no fim. Tarefa cuja verificação não rodou, ou rodou e falhou, não é marcada.

O arquivo é o estado da execução: quem retomar a feature semanas depois descobre onde ela parou lendo só ele. Mantenha também a linha de Progresso coerente com as caixas marcadas.

**A conversa.**

Esta etapa não entrevista. Mas na dúvida, pergunte antes de escrever. Conta como dúvida: o plano cita caminho que não existe em `resources/` e nenhuma etapa anterior cria; a mesma mudança caberia em dois arquivos e o plano não escolheu; o comando de verificação depende de ferramenta que você não confirmou instalada.

**O portão.**

Antes de escrever o arquivo, apresente só a lista numerada dos títulos das tarefas, com as dependências e os blocos, e espere um sim. O que precisa ser conferido é o recorte.

No mesmo momento, pergunte como ele quer executar: em série, uma tarefa por vez, ou em paralelo dentro dos blocos. Pergunte sempre, mesmo quando o paralelismo for óbvio, e nunca trate silêncio como liberação — sem um sim explícito, o campo fica em `não`.

**A escrita.**

Recebido o sim: copie `.spec/templates/tasks.md` para `.spec/specs/TASKS-<NNN>-<slug>.md` e preencha a cópia. Tanto o `<NNN>` quanto o `<slug>` encontram-se no PDR, se precisar, leia-o. Preencha o campo `Paralelismo` com a resposta do usuário — `não` se ele não liberou em palavras; se liberou, registre no campo `Aprovado por` quem liberou e quando. Entregue todas as caixas desmarcadas. Tabela ou seção do modelo sem o que registrar leva `nenhum` escrito, nunca o placeholder.

**Antes de entregar.**

Releia procurando: tarefa que deixa decisão para o executor; caminho inexato; comando que não dá para copiar e rodar; verificação sem resultado esperado; dependência apontando para tarefa de bloco posterior no mapa; etapa do plano que ficou sem tarefa; caixa marcada sem execução que a registre. Corrija na hora.

Depois percorra bloco a bloco e confira as três, uma de cada vez: nenhuma tarefa do bloco aparece no "Depende de" de outra do mesmo bloco, direta ou por cadeia; nenhum arquivo aparece em duas tarefas do bloco; nenhum recurso aparece em duas. Falhando qualquer uma, quebre o bloco antes de entregar — é a última vez que isso sai barato.

**A revisão.**

Escrita a lista e feita a autorrevisão, invoque o subagente `revisor`, informando o caminho do arquivo e a etapa `TASKS`. A revisão é um laço, e ele só termina de um jeito: o `revisor` devolvendo zero bloqueantes e zero graves. Corrija os bloqueantes e os graves e invoque o `revisor` de novo — cada volta é uma invocação nova; ele não lembra da anterior, e é isso que a faz valer. Não há número máximo de voltas.

Enquanto houver bloqueante ou grave, o artefato não é entregue: não peça a revisão do usuário e não promova o `Status`. Não adianta apresentar ao autor um artefato que se sabe estar com defeito.

Três casos você não corrige sozinho, e nenhum deles encerra o laço com defeito dentro:

- **O achado depende de uma resposta do usuário.** Pergunte, corrija com a resposta e invoque o `revisor` de novo.
- **O achado voltou depois de corrigido.** Não repita a mesma correção: mostre ao usuário o achado e o que você tentou, corrija como ele decidir e invoque o `revisor` de novo. O `revisor` só lê o arquivo; o que ele lê errado é trecho que admite a leitura errada, e se resolve reescrevendo o trecho, nunca descartando o achado.
- **O achado nasce no artefato de origem.** Ele se conserta lá, e você não edita artefato já aprovado. A etapa fica bloqueada: deixe este arquivo em `rascunho`, não peça aprovação, e diga ao usuário qual artefato de origem, qual seção e qual defeito. Corrigida a origem, a etapa é invocada de novo, retoma este mesmo arquivo e a auditoria recomeça do zero.

Observação não entra no laço: você apresenta e o usuário decide.

Com zero bloqueantes e zero graves, informe o caminho do arquivo, quantas voltas houve, o que o `revisor` apontou e o que foi corrigido em cada uma, e peça a revisão do usuário.

**A aprovação.**

O arquivo de tarefas nasce `rascunho` e a execução exige `aprovado`. Depois de o usuário aprovar em palavras, mude o `Status` para `aprovado`. Nunca promova por conta própria nem trate silêncio como aprovação. Se, em vez de aprovar, o usuário pedir mudança, faça a mudança e rode a revisão de novo, até zero bloqueantes e zero graves, antes de pedir a aprovação outra vez.
```

### 2.6 Skill da execução

Quinta etapa do fluxo. Crie `.claude/skills/execute/SKILL.md` conforme o contrato da 2.1, com esta `description`:

```
Executa a lista de tarefas de uma feature despachando um subagente executor por tarefa, confere contaminação por inventário e escreve o log EXEC. Esta skill escreve código em `resources/`. Use SOMENTE quando o usuário invocar esta skill explicitamente, passando o ID da feature. Nunca dispare porque o assunto surgiu, porque as tarefas foram aprovadas, ou porque o usuário comentou que quer executar.
```

E com o bloco abaixo como corpo, copiado literalmente:

```
**O que ela faz.** Executa a lista de tarefas de uma feature, respeitando os blocos e o paralelismo declarados, e escreve o log da execução. É a única etapa do fluxo que altera `resources/`.

**Quando dispara.** Só quando o usuário a invoca explicitamente, passando o ID da feature. Nunca porque o assunto surgiu na conversa, nunca porque o usuário comentou que quer executar. A `description` precisa dizer isso com todas as letras: esta skill escreve código, e o portão é o usuário digitar o comando.

**Entrada.**

Leia o arquivo de tarefas da feature em `.spec/specs/`. Se ele não estiver com `Status: aprovado`, pare e diga por quê. Tarefas já marcadas com `- [x]` não são reexecutadas. Leia também o PDR da feature: é dele que vêm o `<NNN>` e o `<slug>` que nomeiam o log e o arquivo de inventário.

Recuse-se a começar se já houver rodada em curso para esta feature — arquivo de tarefas com `Status: em execução`, ou log `EXEC` da feature com `Status: em execução`. Dois orquestradores na mesma feature marcam o mesmo arquivo e escrevem o mesmo log; o segundo corrompe o trabalho do primeiro. Marque `em execução` ao começar e devolva o status ao terminar ou ao interromper. A exceção é a rodada morta, tratada em Retomada.

Antes de tocar em qualquer coisa, registre o inventário de `resources/`: todo caminho de arquivo com seu tamanho e data de modificação. Salve a saída em `.spec/specs/EXEC-<NNN>-<slug>.inventario-r<N>.txt`, onde `<N>` é o número da rodada, e anote o caminho na seção Inventário inicial do log — uma linha por rodada. O log guarda o ponteiro, o arquivo guarda a prova. É esse inventário que prova o que mudou — dentro de `resources/`, e só. Escrita fora dali, em diretório temporário, banco ou serviço, não aparece em nenhuma conferência de contaminação: o que a registra é a declaração de `Recursos` da tarefa e o relatório do executor. Saber o que a prova não cobre é parte de confiar nela.

**A orquestração.**

Execute bloco a bloco, na ordem das linhas do Mapa de execução. Leia o campo `Paralelismo` no cabeçalho do arquivo de tarefas: em `sim`, as tarefas de um bloco correm em paralelo, uma por subagente; em `não`, despache uma por vez, na ordem, mesmo onde o bloco permitiria. Campo ausente ou ilegível vale como `não`. Espere o bloco inteiro terminar antes de abrir o próximo — bloco seguinte não começa com o anterior incompleto.

Duas tarefas com dependência entre si nunca vão para agentes diferentes ao mesmo tempo. Nunca. Não confie no bloco: antes de despachar, confira você mesmo, tarefa a tarefa do bloco, que nenhuma delas aparece no "Depende de" de outra do mesmo bloco, direta ou por cadeia, e que duas não escrevem no mesmo arquivo.

Confira três coisas em cada bloco, antes de despachar — inclusive com `Paralelismo: não`, porque elas provam que o recorte está certo, e o que elas acham é defeito do arquivo de tarefas:

1. **Dependência.** Nenhuma tarefa do bloco aparece no "Depende de" de outra do mesmo bloco, direta ou por cadeia.
2. **Arquivos.** Nenhum caminho aparece em "Arquivos" de duas tarefas do bloco. Caminho que for pasta ou curinga não dá para comparar: trate a tarefa como não paralelizável.
3. **Recursos.** Nenhum recurso — banco, porta, serviço, variável de ambiente — aparece em duas tarefas do bloco.

Se qualquer uma das três falhar, não paralelize: execute aquelas tarefas em série, na ordem da dependência, registre o desvio no log e avise o usuário — o bloco está errado no arquivo de tarefas, e isso é defeito a corrigir lá.

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
- **Instalação é sua, nunca do executor.** Tarefa que precise instalar dependência faz o `executor` parar e reportar. Quem instala é você, conferindo o pacote e a versão contra as Dependências a instalar do plano, e registrando o comando na trilha, com `orquestrador` na coluna Tarefa. É por isso que o executor não precisa do plano para obedecer à regra: ele simplesmente não instala nada. Feita a instalação, redespache a mesma tarefa, com o mesmo bloco: ele confere o estado antes de agir, encontra a dependência instalada e segue. O que a instalação escreve em `resources/` tem de estar em "Arquivos" da tarefa que a pediu, como qualquer outra escrita. Pacote que não consta nas Dependências a instalar do plano, ou instalação que falha, é parada obrigatória da tarefa.
- Não repita comando que já teve sucesso nesta rodada.
- Antes de criar, confira se já existe; antes de alterar, confira se já não está no estado esperado. Rodar a mesma tarefa duas vezes não pode duplicar nada.

**As paradas obrigatórias.**

Pare a tarefa, não marque a caixa, registre e reporte quando:

- a verificação falhar, qualquer que seja o motivo;
- a tarefa mandar tocar arquivo que não existe e que nenhuma tarefa anterior criou;
- um arquivo fora da lista da tarefa precisaria mudar para a verificação passar;
- a tarefa for ambígua, incompleta ou exigir uma decisão;
- o comando pedir confirmação, credencial ou acesso de rede não previsto.

Parada do `executor` por instalação não entra nesta lista: você instala e redespacha a tarefa, como está nas regras de comando.

As demais tarefas do mesmo bloco seguem até o fim. Os blocos seguintes não começam: a rodada vai para a revisão e fecha `interrompida`. Continuar é decisão do usuário, numa retomada.

**A marcação.**

Quem marca `- [x]` no arquivo de tarefas é você, nunca o subagente. Duas razões: vários subagentes escrevendo no mesmo arquivo ao mesmo tempo o corrompem, e marcar é decisão de quem leu o relatório e a saída da verificação. Marque uma tarefa por vez, só depois da verificação ter passado, e mantenha a linha de Progresso coerente.

**A verificação de contaminação.**

Ao fim de cada bloco, antes de abrir o próximo, levante de novo o inventário de `resources/` e compare com o do começo. Todo arquivo que mudou, foi criado ou sumiu tem que estar na união dos "Arquivos" das tarefas executadas — arquivo dentro de pasta declarada conta como declarado. O que estiver fora dessa união é contaminação: registre no log com o que mudou, **pare a rodada** e reporte ao usuário. Não desfaça por conta própria — desfazer também é alterar o que ninguém pediu.

A conferência é por bloco, não só no fim, e a razão é prática: contaminação no bloco 1 descoberta só no fim significa que todos os blocos seguintes rodaram em cima de estado sujo. O relatório do subagente sobre os arquivos que ele tocou é o que ele diz ter feito; o inventário é o que aconteceu.

**Os achados.**

Você vai encontrar coisa errada que não é da sua tarefa: código quebrado ao lado, premissa do plano que não se sustentou, dívida óbvia. Não conserte. Registre em Achados, com onde está e por que importa, e siga. O log é o lugar onde a vontade de consertar vira informação para a próxima rodada, em vez de virar contaminação nesta.

**O log.**

Copie `.spec/templates/exec.md` para `.spec/specs/EXEC-<NNN>-<slug>.md` no começo da rodada e preencha à medida que executa, não no fim — se a rodada for interrompida, o que já foi escrito precisa valer. Tanto o `<NNN>` quanto o `<slug>` encontram-se no PDR, se precisar, leia-o. Tabela ou seção do modelo sem o que registrar leva `nenhum` escrito, nunca o placeholder.

Preencha à medida que executa, e não só o resultado: o inventário inicial na seção própria, uma linha na Conferência por bloco ao fechar cada bloco, e uma linha em Confirmações destrutivas a cada confirmação que você pedir. São esses três registros que tornam a rodada auditável depois — sem eles o `revisor` não tem como conferir o que você diz ter feito.

**Retomada.** Se já existe log da feature com `Status: interrompida` ou `concluída`, não crie outro nem sobrescreva aquele: reabra o mesmo arquivo, acrescente uma linha na tabela de Rodadas, marque `em execução` e continue de onde parou, pulando as tarefas já marcadas. Rastro de rodada anterior não se apaga. Levante o inventário de `resources/` de novo, como base desta rodada, e salve-o com o número da rodada no nome; o da rodada anterior fica onde está.

Antes de despachar qualquer tarefa na retomada, percorra Desvios: para cada fato recusado ainda sem o registro de que o que o usuário mandou fazer foi feito, pergunte a ele e registre a resposta na linha. Quem faz o que ele mandou não é você, e não é o `executor`: é o usuário, por fora do fluxo. A você cabe perguntar e registrar. Se não sobrou tarefa por fazer, a rodada se resume a esses registros e segue direto para a revisão e o fechamento.

Log com `Status: em execução` e nenhuma rodada viva é rodada **morta**, não rodada em curso — a sessão acabou antes do fechamento. Não confunda com concorrência, e não destrave sozinho: mostre ao usuário o log e a última linha da tabela de Rodadas, peça confirmação explícita de que nada está em curso, e só com o sim feche aquela linha como `interrompida`, ponha o `Status` do log em `interrompida` e siga a retomada acima. Se o arquivo de tarefas também estiver em `em execução`, devolva-o a `aprovado` antes de começar. Sem o sim, não comece.

**A revisão e o fechamento.**

Terminada a última tarefa possível, feche o registro da rodada — o Fim, o Resumo e a linha dela na tabela de Rodadas, com o desfecho que ela tem neste momento — e invoque o subagente `revisor`, informando o caminho do log e a etapa `EXEC`. A revisão é um laço, e ele só termina de um jeito: zero bloqueantes e zero graves. Não há número máximo de voltas, e a rodada não é entregue antes disso.

O `revisor` audita o log, não a rodada: log que registra uma falha com fidelidade está certo; errado é o log que a esconde. Por isso todo achado se resolve no log:

- **Erro do registro** — número errado no Resumo, comando ausente da trilha, bloco sem linha de conferência: corrija no log e invoque o `revisor` de novo.
- **Fato da rodada sem decisão registrada** — contaminação, tarefa marcada concluída com verificação que falhou, ordem violada, paralelismo indevido, instalação feita pelo executor, desvio da proposta original, inventário ou conferência de bloco que não foram feitos: nada disso se desfaz aqui, e você não toca em `resources/` para desfazer nem altera o log para o fato sumir. Registre o fato em Desvios, mostre ao usuário e escreva na mesma linha a decisão dele: aceito como está, ou recusado, com o que ele mandou fazer. Só então invoque o `revisor` de novo. Nunca preencha a decisão por ele. Fato recusado continua pendente até a linha registrar que o que ele mandou fazer foi feito — na retomada, pergunte; não presuma.
- **Achado que voltou depois de corrigido:** não repita a mesma correção. Mostre ao usuário o achado e o que você tentou, corrija como ele decidir e invoque o `revisor` de novo.
- **Observação:** apresente ao usuário e siga; quem decide é ele.

Fechado o laço, atualize o `Status` do log: `concluída` só quando todas as tarefas estão marcadas, sem tarefa interrompida e sem fato pendente em Desvios — pendente é o fato sem decisão do usuário, ou recusado sem o registro de que o que ele mandou fazer foi feito; `interrompida` em qualquer outro caso, dizendo onde parou. Devolva o `Status` do arquivo de tarefas: `concluído` quando o log fechou `concluída`, `aprovado` em qualquer outro caso — é o que deixa a rodada ser retomada. As caixas e o Progresso ficam como ficaram. Se o laço mudou o desfecho da rodada, atualize também o Fim, o Resumo e a linha dela em Rodadas.

Só então informe ao usuário: quantas tarefas concluíram, onde parou e por quê, os desvios, a contaminação encontrada, quantas voltas de revisão houve, o que o `revisor` apontou e o que foi corrigido em cada uma, e o caminho do log. A rodada está entregue nesse ponto, e não antes.
```

### 2.7 Skill do arquivamento

Sexta e última etapa do fluxo. Crie `.claude/skills/archive/SKILL.md` conforme o contrato da 2.1, com esta `description`:

```
Fecha uma feature: move os artefatos dela da raiz de `.spec/specs/` para `.spec/specs/archive/<NNN>-<slug>/` e escreve ali o README com a memória da feature. Esta skill move arquivos. Use SOMENTE quando o usuário invocar esta skill explicitamente, passando o ID da feature. Nunca dispare porque a execução terminou ou porque o usuário comentou que a feature acabou.
```

E com o bloco abaixo como corpo, copiado literalmente:

```
**O que ela faz.** Fecha uma feature: move os artefatos de `.spec/specs/` para uma pasta própria dentro de `.spec/specs/archive/`, deixando `.spec/specs/` limpo para a próxima rodada, e escreve ali o registro do que a feature foi e do que ela ensinou.

**Quando dispara.** Só quando o usuário a invoca explicitamente, passando o ID da feature. Esta skill move arquivos: nunca dispare porque o assunto surgiu, porque a execução terminou ou porque o usuário comentou que a feature acabou. A `description` precisa dizer isso.

**Não toque em `resources/`.** Nem para ler o resultado, nem para limpar nada. O código já foi entregue pela execução; aqui se mexe apenas em `.spec/`.

**Entrada.**

Localize na raiz de `.spec/specs/` os artefatos da feature informada: os arquivos com o `<NNN>` dela no nome. Leia o PDR: é dele que vêm o `<NNN>` e o `<slug>` que nomeiam a pasta de destino. Arquivo de outro `<NNN>` na raiz não é desta feature: não o mova, e avise o usuário de que ele está ali.

**Retomada.** Se já existe em `.spec/specs/archive/` a pasta da feature informada, o arquivamento já começou. Não refaça o portão e não crie outra pasta: mova para ela o que ainda estiver na raiz de `.spec/specs/` com o mesmo `<NNN>` no nome, leia o PDR dentro dela, complete o que faltar — o README, a proposta de memória e o registro dela — e siga para a conferência e a revisão. Se nada faltar, diga que a feature já está arquivada e pare.

**O portão de fechamento.**

Não arquive feature em aberto. Confira, e pare se qualquer uma falhar:

- existe log de execução, e ele está com `Status: concluída` — não `em execução`, não `interrompida`;
- o PDR está `aceito`, a spec `aprovada`, o plano `aprovado` e o arquivo de tarefas `concluído`;
- todas as caixas do arquivo de tarefas estão marcadas, e a linha de Progresso bate com elas;
- o log não tem fato pendente em Desvios nem tarefa interrompida sem resolução.

Falhando alguma, diga exatamente o que falta e pare. Arquivar feature incompleta é perder o rastro dela junto com o trabalho.

A exceção é a feature abandonada: se o usuário mandar arquivar mesmo assim, arquive, mas registre no README da pasta arquivada por que foi abandonada e em que ponto parou. Abandono com rastro é legítimo; abandono silencioso não.

Passando o portão, ou tendo o usuário mandado arquivar como abandonada, diga a ele o que será movido e para onde, e espere um sim explícito antes de mover.

**A mudança.**

Recebido o sim:

1. Crie `.spec/specs/archive/<NNN>-<slug>/`. Tanto o `<NNN>` quanto o `<slug>` encontram-se no PDR, se precisar, leia-o.
2. Mova para dentro dela os artefatos da feature que estão na raiz de `.spec/specs/`: todo arquivo com o `<NNN>` dela no nome. Mova, não copie: no fim, não sobra na raiz nada desta feature.
3. Escreva `README.md` dentro da pasta arquivada: o que a feature era em uma linha, o que ela entregou, quando abriu e quando fechou, quantas tarefas rodaram, os achados do log que não foram consertados, uma seção "O que a feature ensinou" — o que funcionou, o que não funcionou e por quê —, e, se for o caso, por que a feature foi abandonada e que estágios nunca chegaram a existir. O que foi promovido para a memória, e o que foi proposto e recusado, entra depois, no passo da memória. É a memória desta feature — o que vale para as próximas, quem promove é o usuário.
4. Não apague nada. Nada mesmo. Se algo parece sobrar, reporte em vez de remover.

**A memória.** Escrito o README, releia a seção "O que a feature ensinou" e proponha ao usuário quais entradas sobem para `.spec/shared/memory.md`, cada uma com a seção de destino. O critério é um só: **vale para a próxima feature?** Se vale só para esta, fica no README. Espere o sim, entrada por entrada, e só então escreva. Entrada que contradiz uma existente não substitui em silêncio: aponte o conflito e pergunte. Depois complete o README com o que subiu e com o que foi proposto e recusado, com o motivo.

**Antes de entregar.**

Confira: a pasta de destino tem os cinco artefatos mais o README — na feature abandonada, os artefatos que existiram mais o README nomeando os estágios que nunca existiram; na raiz de `.spec/specs/` não sobrou nada desta feature; o que subiu para `.spec/shared/` é o que o usuário aprovou, sem acréscimo; você não alterou nada em `resources/` nesta etapa.

**A revisão.**

Feita a conferência, invoque o subagente `revisor`, informando o caminho da pasta arquivada e a etapa `ARCHIVE`. A revisão é um laço, e ele só termina de um jeito: o `revisor` devolvendo zero bloqueantes e zero graves. Corrija os bloqueantes e os graves e invoque o `revisor` de novo — cada volta é uma invocação nova; ele não lembra da anterior, e é isso que a faz valer. Não há número máximo de voltas. Corrija apenas dentro de `.spec/`, e sem apagar nada: sobra na raiz se corrige movendo para a pasta arquivada — se lá já houver arquivo de mesmo nome, com o sufixo `-duplicata` no nome e o registro disso no README.

Enquanto houver bloqueante ou grave, o arquivamento não é entregue: não informe a feature como arquivada. Não adianta apresentar ao autor um resultado que se sabe estar com defeito.

Três casos você não corrige sozinho, e nenhum deles encerra o laço com defeito dentro:

- **O achado depende de uma resposta do usuário.** Pergunte, corrija com a resposta e invoque o `revisor` de novo.
- **O achado voltou depois de corrigido.** Não repita a mesma correção: mostre ao usuário o achado e o que você tentou, corrija como ele decidir e invoque o `revisor` de novo. O `revisor` só lê o arquivo; o que ele lê errado é trecho que admite a leitura errada, e se resolve reescrevendo o trecho, nunca descartando o achado.
- **O achado é alteração em `resources/` feita nesta etapa.** Não se desfaz aqui. Registre no README o que foi alterado, mostre ao usuário e escreva ali a decisão dele: aceito como está, ou recusado, com o que ele mandou fazer. Só então invoque o `revisor` de novo. Nunca preencha a decisão por ele.

Observação não entra no laço: você apresenta e o usuário decide.

Com zero bloqueantes e zero graves, informe a pasta criada, o que foi movido, o que subiu para a memória, quantas voltas houve, o que o `revisor` apontou e o que foi corrigido em cada uma.
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
- **Aprovado por:** <quem aprovou, quando, e o que foi apresentado antes da escrita>
- **ID da feature:** <NNN>
- **Slug adotado**: <slug>

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
- **Aprovado por:** <quem aprovou, quando, e o que foi apresentado antes da escrita>

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
- **Aprovado por:** <quem aprovou, quando, e o que foi apresentado antes da escrita>

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
- **Arquivos:** <caminhos exatos> — inclui o que o comando de verificação escreve
- **Recursos:** <banco, porta, serviço, variável de ambiente> ou nenhum
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
- **Paralelismo:** não | sim
- **PLAN de origem:** PLAN-<NNN> — <título do plano>
- **Data:** <AAAA-MM-DD>
- **Autor(es):** <quem decompôs>
- **Aprovado por:** <quem aprovou, quando, e o que foi apresentado antes da escrita; se o paralelismo foi liberado, quem liberou e quando>

## Como executar

Execute os blocos na ordem das linhas do Mapa de execução. Dentro de um bloco, as tarefas correm juntas se o campo `Paralelismo` do cabeçalho estiver em `sim`; em `não`, execute uma tarefa por vez, na ordem, mesmo onde daria para paralelizar. Duas tarefas com dependência entre si nunca rodam ao mesmo tempo, nem em agentes diferentes nem no mesmo — confira o Mapa de execução antes de despachar. Marque `- [x]` assim que a verificação da tarefa passar — uma por uma, nunca em lote — e atualize a linha de Progresso. Tarefa cuja verificação falhou não é marcada. Se algo no repositório não bater com o que está escrito aqui, pare e reporte: não improvise nem conserte por fora.

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
  - **Arquivos:** <caminho de arquivo, um a um; pasta só em tarefa não paralelizável com ninguém; nunca curinga> (criar / alterar / remover) — inclui o que o comando de verificação escreve
  - **Recursos:** <banco, porta, serviço, variável de ambiente> ou nenhum
  - **O que fazer:** <a instrução completa, com interfaces nomeadas e tipadas; nada que exija decisão de quem executa>
  - **Verificação:** `<comando literal>` → <o resultado exato que aprova a tarefa>
  - **Se falhar:** <pare e reporte, ou o passo concreto de desfazer>

- [ ] **T2 — <título imperativo e curto>**
  - **Etapa do plano:** E<N>
  - **Depende de:** <T<N>> ou nenhuma
  - **Paralelizável com:** <T<N>> ou nenhuma
  - **Arquivos:** <...>
  - **Recursos:** <...> ou nenhum
  - **O que fazer:** <...>
  - **Verificação:** `<comando literal>` → <resultado esperado>
  - **Se falhar:** <...>

## Bloco 2 — <o que este bloco entrega>

- [ ] **T3 — <título imperativo e curto>**
  - **Etapa do plano:** E<N>
  - **Depende de:** T1
  - **Paralelizável com:** nenhuma
  - **Arquivos:** <...>
  - **Recursos:** <...> ou nenhum
  - **O que fazer:** <...>
  - **Verificação:** `<comando literal>` → <resultado esperado>
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

| Tarefa | O que estava previsto | O que aconteceu | O que foi feito | Decisão do usuário |
| --- | --- | --- | --- | --- |
| T<N> | <o que a tarefa dizia> | <o que se encontrou> | <parou e reportou / seguiu assim> | <aceito; ou recusado, com o que ele mandou fazer e, quando feito, o registro disso; "não se aplica" em desvio que não pede decisão> |

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
| 1 | T<N> ou `orquestrador` | `<comando>` | <n> |
```

### 3.6 `.spec/shared/memory.md`

Crie este arquivo em `.spec/shared/memory.md`, **não** em `.spec/templates/`. Ele não é modelo: é a memória do projeto, preenchida no lugar pelo `archive`, entrada por entrada, com aprovação do usuário. O PDR, a spec e o plano o leem antes de começar.

```markdown
# Memória do projeto

Este arquivo vale para todas as features. O PDR, a spec e o plano o leem antes de qualquer pergunta, e o que está aqui não se rediscute a cada rodada — mudou, muda aqui, e passa a valer da próxima feature em diante.

Só entra o que serve a mais de uma feature, e quem escreve aqui é o `archive`, entrada por entrada, depois de o usuário aprovar cada uma. Aprendizado específico de uma feature fica na pasta arquivada dela.

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

O caminho do artefato e o nome da etapa que o produziu. Leia o artefato e o modelo correspondente em `.spec/templates/` — a etapa `ARCHIVE` não tem modelo. Quando a etapa for `SPEC`, leia também o PDR de origem citado no cabeçalho; quando for `PLAN`, leia também a spec de origem, o código em `resources/` que o plano diz tocar e, se existirem, o arquivo de tarefas e o log de execução da feature; quando for `TASKS`, leia também o plano de origem, o código em `resources/` que as tarefas citam e, se existir, o log de execução da feature; quando for `EXEC`, leia também o arquivo de tarefas de origem; quando for `ARCHIVE`, leia a pasta arquivada inteira, o conteúdo de `.spec/specs/` e `.spec/shared/memory.md`. Quando a etapa for `PDR`, `SPEC` ou `PLAN`, leia também `.spec/shared/memory.md` — são as três etapas que o consultam antes de decidir. Quando a etapa for `PLAN`, `TASKS` ou `EXEC`, leia também o PDR e a spec da feature em `.spec/specs/` — é contra eles que se confere o desvio da proposta original. Não peça contexto, não pergunte nada, não tente reconstruir a conversa: se faltou informação para conferir um item, isso é achado bloqueante: diga o que faltou e que item ficou sem conferência.

## O que você nunca faz

- Não edita arquivo nenhum, e não entrega texto de substituição pronto.
- Não julga se a decisão está certa. Você audita o registro, não o mérito.
- Não comenta estilo, tom, tamanho ou organização. Você procura os defeitos da lista, e só eles.
- Não inventa achado para justificar a chamada. "Nenhum defeito encontrado" é resposta legítima, e é a resposta esperada na maioria das vezes.

## Como você reporta

Uma linha por achado, agrupadas por severidade:

- **Bloqueante** — o artefato não serve para a etapa seguinte enquanto estiver assim.
- **Grave** — vai custar caro depois; também se corrige antes da entrega.
- **Observação** — vale corrigir se for barato.

A severidade não é escolha sua: cada item da lista traz a dele. Item marcado Bloqueante é bloqueante, item marcado Grave é grave, e item sem marca é observação.

Cada achado cita a seção onde está e diz qual item da lista foi violado. Não havendo achado, responda exatamente: `Nenhum defeito encontrado.` A resposta não leva seção alguma além dos achados — nada de "o que foi conferido", "o que passou" ou resumo do método. Quem lê precisa achar o defeito, não a prova de diligência.

## Listas por etapa

Use apenas a lista da etapa que lhe foi informada. Se a etapa não estiver abaixo, diga isso e pare — não improvise critério.

Antes da lista da etapa, confira sempre estes três itens — o primeiro em todas as etapas, os outros dois nas etapas que nomeiam:

- **Status promovido sem aprovação.** O artefato chega com o `Status` já avançado — `aceito`, `aprovada`, `aprovado`, `concluído` — sem que o texto entregue registre a aprovação do usuário. Bloqueante: é assim que uma etapa passa pela porta da seguinte sem ninguém ter dito sim.
- **Contradiz a memória do projeto.** Nas etapas `PDR`, `SPEC` e `PLAN`: o artefato adota o que `.spec/shared/memory.md` veta, ignora uma versão fixada, repete um caminho já tentado e recusado, ou contraria um invariante — sem que o próprio artefato registre que está mudando aquela orientação. Bloqueante: o que está na memória não se rediscute por omissão, e decisão que a contraria sem dizer que a contraria é como a orientação morre.
- **Desvio da proposta original.** Nas etapas `PLAN`, `TASKS` e `EXEC`: o artefato contraria a Decisão, o Escopo ou o Fora de escopo do PDR da feature, ou contraria uma linha da tabela Viabilidade verificada da spec — usa versão diferente da conferida, ultrapassa o limite assumido, apoia-se no que a spec registrou como risco não confirmado — sem que o próprio artefato registre o desvio. Bloqueante: cada etapa confere só a anterior, e é de etapa em etapa que a feature vira outra coisa sem ninguém ter decidido.

Nos itens "Campo vazio ou placeholder", campo ou tabela com `nenhum` escrito não é campo vazio: é o registro de que não havia o que registrar.

Nas etapas `PLAN` e `TASKS`, quando a feature já tem log de execução, o que foi executado é registro e não se audita de novo: tarefa marcada `- [x]` que o log registra como concluída, e etapa do plano realizada por tarefa assim. Os itens da lista valem para as etapas e tarefas ainda não executadas e para o conjunto — numeração, ordem, ondas, blocos, mapa, cobertura e progresso. Defeito que só existe no conteúdo de etapa ou tarefa já executada não é achado.

### PDR

1. **Decisão que não decide.** A seção Decisão não é uma frase afirmativa, no presente, dizendo o que será feito. "Avaliar", "estudar", "considerar" caem aqui. Bloqueante.
2. **Duas decisões num PDR.** A Decisão traz conjunção que esconde uma segunda escolha independente — dá para recusar uma sem derrubar a outra. Bloqueante.
3. **Alternativa sem motivo.** Linha da tabela de Alternativas consideradas sem motivo de recusa, ou com motivo que não é motivo ("não era o ideal", "não fazia sentido"). Grave.
4. **Consequência só positiva.** A seção Consequências não registra nada que piora, encarece ou se perde. Grave.
5. **Gatilho não observável.** O Gatilho de revisão não é uma data, um número que se cruza, nem uma premissa nomeada que pode cair. Grave.
6. **Contexto preso à conversa.** O Contexto só faz sentido para quem participou da entrevista: fala em "o problema que discutimos", cita pessoa ou sistema nunca apresentado, ou pressupõe fato que o arquivo não registra. Grave.
7. **Campo vazio ou placeholder.** Campo do modelo mantido com `<...>`, "a definir" ou "TBD". Grave.
8. **Fora de escopo ausente ou falso.** A seção não existe, ou repete a Decisão em vez de delimitar o que não foi decidido. Grave.

### SPEC

1. **Critério de aceitação subjetivo.** O critério não diz como verificar, ou o resultado esperado depende de julgamento: "rápido", "estável", "intuitivo", "funciona bem". Bloqueante.
2. **Afirmação técnica sem prova.** A Solução escolhida ou as Dependências afirmam algo sobre biblioteca, API ou limite externo que não tem linha correspondente na tabela de Viabilidade verificada, ou cuja linha não cita fonte e data. Bloqueante.
3. **Decisão do PDR sem cobertura.** Algo que a Decisão do PDR de origem determina não aparece em nenhuma entrada, saída, comportamento ou critério. Bloqueante.
4. **Spec que decide sozinha.** A spec fixa escolha de produto que o PDR não tomou. Bloqueante — isso é PDR novo.
5. **Entrada pela metade.** Entrada sem tipo, sem validação, ou sem o que acontece quando chega inválida. Grave.
6. **Saída pela metade.** Saída sem destino, sem formato, ou sem a condição que a produz. Grave.
7. **Só o caminho feliz.** A seção Casos de erro está vazia, ou não cobre condição de falha que as Entradas e Dependências tornam possível. Grave.
8. **Critério solto.** Critério de aceitação que nenhuma entrada, saída ou comportamento sustenta. Grave.
9. **Dependência incompleta.** Dependência externa sem versão ou sem licença. Grave.
10. **Referência não consultada.** URL na tabela de Referências sem o que foi tirado dela ou sem data de consulta.
11. **Lista aberta.** "Etc.", "entre outros", "e assim por diante" ou reticências em lista que precisa ser fechada. Grave.
12. **Campo vazio ou placeholder.** Campo do modelo mantido com `<...>`, "a definir" ou "TBD". Grave.
13. **ID divergente.** O `<NNN>` do nome do arquivo ou do título não é o mesmo do PDR de origem citado no cabeçalho. Bloqueante — é esse número que amarra os artefatos da feature.
14. **Slug divergente.** O `<slug>` do nome do arquivo não é o mesmo do PDR de origem.

### PLAN

1. **Etapa sem verificação.** Etapa sem verificação, ou com verificação que ninguém consegue executar e ler o resultado: "conferir se ficou bom", "validar o comportamento". Bloqueante.
2. **Ciclo no grafo.** Duas ou mais etapas que dependem uma da outra, direta ou por cadeia. Bloqueante.
3. **Ordem impossível.** Etapa que usa arquivo, função ou dependência produzida por etapa posterior, ou por etapa da mesma onda. Bloqueante.
4. **Critério da spec sem etapa.** Critério de aceitação da spec de origem que nenhuma etapa atende. Bloqueante.
5. **Paralelismo falso.** Etapas declaradas paralelizáveis que escrevem no mesmo arquivo, que usam o mesmo recurso, ou entre as quais existe dependência declarada. Bloqueante — onda errada vira bloco errado, e bloco errado quebra a execução.
6. **Efeito colateral não levantado.** Arquivo com ação `alterar` ou `remover` no Mapa de arquivos sem linha correspondente em Efeitos colaterais, e sem afirmação de que nada depende dele. Grave.
7. **Mapa furado.** Caminho citado numa etapa e ausente do Mapa de arquivos, ou linha do mapa que nenhuma etapa realiza. Grave.
8. **Etapa sem destino.** Etapa que não atende critério nenhum da spec nem habilita outra etapa que atenda. Grave.
9. **Etapa grande demais.** A etapa entrega mais de uma coisa — "e" ligando trabalhos que se verificam separadamente, ou verificação que precisa de mais de um resultado para aprovar. Grave.
10. **Mudança sem volta.** Etapa que altera migração, dado existente, contrato publicado ou configuração de ambiente sem "Como desfazer" concreto. Grave.
11. **Dependência incompleta.** Dependência a instalar sem versão, sem licença ou sem o comando exato. Grave.
12. **Ondas inconsistentes.** A tabela de Ordem de execução contradiz os campos "Depende de", ou o caminho crítico declarado não é a cadeia mais longa do grafo. Grave.
13. **Campo vazio ou placeholder.** Campo do modelo mantido com `<...>`, "a definir" ou "TBD". Grave.
14. **ID divergente.** O `<NNN>` do arquivo ou do título não é o mesmo da spec de origem. Bloqueante.
15. **Slug divergente.** O `<slug>` do nome do arquivo não é o mesmo do PDR da feature.

### TASKS

1. **Tarefa que decide.** A tarefa devolve uma escolha ao executor: "escolha a melhor abordagem", "ajuste conforme o projeto", "se aplicável", "conforme necessário". Bloqueante.
2. **Tarefa presa ao contexto.** Só se entende tendo lido o plano, a spec ou a conversa: fala em "o arquivo de configuração", "o serviço que criamos", "conforme decidido", sem nomear. Bloqueante.
3. **Verificação não executável.** Sem comando literal que dê para copiar e rodar, ou sem o resultado exato que aprova. Bloqueante.
4. **Dependência para frente.** "Depende de" apontando tarefa de bloco que vem depois no Mapa de execução. Bloqueante.
5. **Etapa do plano sem tarefa.** Etapa do plano de origem que nenhuma tarefa realiza. Bloqueante.
6. **Declarações em conflito.** A mesma tarefa aparece no "Depende de" e no "Paralelizável com" de outra, ou esses campos contradizem o bloco em que as tarefas estão. Bloqueante.
7. **Bloco mal formado.** Tarefas no mesmo bloco com dependência entre si, direta ou por cadeia, com o mesmo arquivo em "Arquivos", ou com o mesmo recurso em "Recursos". Bloqueante.
8. **Mapa de execução ausente ou incoerente.** O mapa não existe, contradiz os campos das tarefas, ou repete arquivo ou recurso entre duas tarefas do mesmo bloco. Bloqueante.
9. **Caminho que não é arquivo.** "Arquivos" traz pasta, curinga ou "todos os arquivos de". Isso cega a comparação de colisão entre tarefas paralelas. Bloqueante — salvo pasta em tarefa que não é paralelizável com ninguém e está sozinha no bloco.
10. **Escrita da verificação não declarada.** O comando de verificação escreve lockfile, cache, diretório de build, cobertura, migração ou artefato equivalente, e esses caminhos não estão em "Arquivos". Grave.
11. **Recurso compartilhado não declarado.** A tarefa usa banco, porta, serviço, variável de ambiente ou diretório temporário fixo, e o campo "Recursos" está vazio. Grave.
12. **Leitura na lista de escrita.** "Arquivos" cita arquivo que a tarefa apenas lê — isso cega a checagem de colisão entre tarefas paralelas.
13. **Tarefa órfã.** Tarefa que não aponta nenhuma etapa do plano. Grave.
14. **Tarefa composta.** Entrega mais de uma coisa, ou a verificação precisa de mais de um resultado para aprovar. Grave.
15. **Caminho inexato.** Arquivo citado sem caminho completo, ou caminho que não existe em `resources/`, que a própria tarefa não cria e que nenhuma tarefa anterior cria. Grave.
16. **Caixa marcada sem execução.** Tarefa com `- [x]` que o log de execução da feature não registra como concluída — sem log, qualquer caixa marcada —, ou linha de Progresso incoerente com as caixas. Grave.
17. **Tarefa sem volta.** Tarefa que mexe em estado difícil de desfazer sem "Se falhar" concreto. Grave.
18. **Numeração furada.** Buraco na sequência `T<N>`, número repetido ou reaproveitado.
19. **Campo vazio ou placeholder.** Campo do modelo mantido com `<...>`, "a definir" ou "TBD". Grave.
20. **ID divergente.** O `<NNN>` do arquivo ou do título não é o mesmo do plano de origem. Bloqueante.
21. **Slug divergente.** O `<slug>` do nome do arquivo não é o mesmo do PDR da feature.
22. **Paralelismo sem liberação.** O campo `Paralelismo` do cabeçalho está ausente, com placeholder, ou em `sim` sem que o campo `Aprovado por` registre quem liberou e quando. Bloqueante — paralelizar é decisão do usuário, nunca padrão do agente.

### EXEC

Nesta lista, fato da rodada que o log não desfaz — algo que aconteceu e não devia, ou que devia ter acontecido e não aconteceu — não é achado quando tem linha em Desvios com a decisão do usuário escrita nela. Sem a linha, com linha sem decisão, ou com decisão que o log não atribui ao usuário, é achado do item correspondente. Erro do registro — o log dizendo diferente do que houve, ou deixando de dizer — é sempre achado.

1. **Conclusão sem prova.** Tarefa com resultado `concluída` sem comando de verificação, sem código de saída ou sem saída registrada. Bloqueante.
2. **Contaminação não declarada ou sem decisão.** Arquivo tocado que não consta dos "Arquivos" da tarefa e não aparece na tabela de Contaminação verificada, ou aparece ali como fora de escopo sem linha em Desvios com a decisão do usuário. Bloqueante.
3. **Conserto disfarçado de achado.** Um item de Achados descreve algo no mesmo arquivo que a rodada alterou fora da tarefa — foi consertado em vez de registrado. Bloqueante.
4. **Ordem violada.** Tarefa executada antes daquela de que depende, ou bloco iniciado com o anterior incompleto. Bloqueante.
5. **Paralelismo indevido.** Duas tarefas com dependência entre si, com arquivo em comum ou com recurso em comum, despachadas ao mesmo tempo para agentes diferentes. Bloqueante.
6. **Rodadas concorrentes.** O log registra execução iniciada com outra rodada da mesma feature em curso, ou com o arquivo de tarefas já em `Status: em execução`. Bloqueante.
7. **Bloco sem conferência.** Bloco encerrado e o seguinte aberto sem linha correspondente na tabela de Conferência por bloco. Bloqueante — é o que permite o resto da rodada correr sobre estado sujo.
8. **Inventário inicial ausente.** A seção Inventário inicial está vazia, sem o comando que a levantou, sem o caminho do inventário completo, ou — num log com mais de uma rodada — sem uma linha para cada rodada da tabela de Rodadas: sem base de comparação, nenhuma conferência posterior prova coisa alguma. Bloqueante.
9. **Rodada sobrescrita.** A tabela de Rodadas tem uma linha só num log que registra retomada, ou o registro anterior sumiu. Bloqueante — retomada acrescenta, nunca apaga.
10. **Instalação pelo executor.** A trilha registra instalação de pacote ou ferramenta com `T<N>` na coluna Tarefa — feita dentro de uma tarefa —, em vez de `orquestrador`, que é como se registra a instalação feita por ele contra as Dependências do plano. Grave.
11. **Verificação trocada.** O comando registrado difere do comando escrito na tarefa e não há linha correspondente em Desvios. Grave.
12. **Aprovação com falha.** Tarefa `concluída` com código de saída diferente do que a tarefa define como aprovação. Grave.
13. **Tarefa sumida.** Tarefa do arquivo de origem ausente do Registro por tarefa e não contabilizada como não iniciada. Grave.
14. **Resumo incoerente.** Os números do Resumo não batem com o Registro por tarefa nem com as caixas marcadas no arquivo de tarefas. Grave.
15. **Destrutivo sem confirmação.** Comando destrutivo na trilha sem linha correspondente na tabela de Confirmações destrutivas. Grave.
16. **Sugestão executada.** Item de Achados cuja "sugestão não executada" aparece realizada na rodada. Grave.
17. **Trilha incompleta.** Comando citado no Registro por tarefa e ausente da tabela de Comandos executados. Grave.
18. **Campo vazio ou placeholder.** Campo do modelo mantido com `<...>`, "a definir" ou "TBD". Grave.
19. **ID divergente.** O `<NNN>` do arquivo ou do título não é o mesmo do arquivo de tarefas de origem. Bloqueante.
20. **Slug divergente.** O `<slug>` do log, ou o do arquivo de inventário que ele aponta, não é o mesmo do PDR da feature.

### ARCHIVE

1. **Feature em aberto.** A pasta foi arquivada com log de execução `em execução` ou `interrompida`, com caixa desmarcada no arquivo de tarefas, ou com PDR, spec, plano ou arquivo de tarefas fora do `Status` final — `aceito`, `aprovada`, `aprovado`, `concluído` —, e o README não registra abandono nem o ponto de parada. Bloqueante.
2. **Artefato faltando.** A pasta arquivada não tem os cinco artefatos — PDR, spec, plano, tarefas e log — mais o README, e o README não registra o abandono nomeando os estágios que nunca existiram. Bloqueante.
3. **Sobra na origem.** Sobrou na raiz de `.spec/specs/` arquivo desta feature — com o `<NNN>` dela no nome. Bloqueante — foi cópia, não mudança.
4. **Achado perdido.** Achado do log que não foi consertado e não aparece no README da pasta arquivada. Grave.
5. **Memória escrita sem aprovação.** Entrada que o README registra como promovida e que não está em `.spec/shared/memory.md`; entrada da memória atribuída a esta feature — pelo número, em Caminhos já tentados ou em Decisões permanentes — que o README não registra como promovida e aprovada; ou entrada promovida que contradiz outra da memória sem o conflito apontado no README. Bloqueante — é a memória do projeto; entrada que ninguém aprovou vira regra que ninguém decidiu.
6. **README raso.** O README não diz o que a feature entregou, quando abriu e fechou, o que ela ensinou, nem o que foi promovido para a memória. Grave.
7. **Código tocado.** O README da pasta arquivada registra alteração em `resources/` feita nesta etapa, sem a decisão do usuário sobre ela escrita ali. Bloqueante — esta etapa mexe apenas em `.spec/`.
8. **Numeração reaproveitada.** Já existe outra pasta arquivada com o mesmo `<NNN>`. Bloqueante.
9. **Slug da pasta errado.** O `<slug>` da pasta arquivada não é o do PDR da feature.
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
- Não instala nada. Nem pacote, nem ferramenta, nem extensão. Tarefa que precise de instalação faz você parar e reportar: quem instala é quem te chamou. Se a dependência que a tarefa pede já está instalada, não há o que instalar: siga.
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
