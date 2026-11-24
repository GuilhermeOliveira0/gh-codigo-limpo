---
name: gh-codigo-limpo
description: Escreve ou revisa código para ser fácil de ler por humanos, aplicando Clean Code (nomes que revelam a intenção, variáveis explicativas, uma ideia por linha, retorno antecipado, funções pequenas) com SOLID, DRY, KISS e YAGNI na medida certa. Só roda quando o usuário chama /gh-codigo-limpo. Na implementação, aplica as regras enquanto escreve. Na revisão de código existente, mostra o que achou e espera aprovação antes de alterar qualquer coisa.
disable-model-invocation: true
argument-hint: "[tarefa a implementar, ou arquivo/pasta para revisar]"
---

# Código Limpo

Quem usa esta skill lê todo o código que a IA gera e precisa entendê-lo rápido. O objetivo é um só: uma pessoa que não conhece o código consegue lê-lo de cima para baixo, sem voltar, e entender o que ele faz e por quê.

Tudo abaixo serve a esse objetivo. Quando uma regra deixar o código mais difícil de ler num caso concreto, a regra perde.

## Quando esta skill vale

O usuário chamou esta skill com `/gh-codigo-limpo`. O pedido dele é:

> $ARGUMENTS

- **Vale para este pedido**, incluindo as respostas do usuário que fazem parte dele (por exemplo, os números aprovados no relatório de revisão).
- **Não vale para os pedidos seguintes** que vierem sem `/gh-codigo-limpo`. Neles, trabalhe normalmente, sem aplicar este fluxo nem chamar o `revisor-codigo-limpo`.
- **Se o pedido acima estiver vazio**, pergunte em uma linha o que fazer (implementar algo ou revisar qual arquivo ou pasta) e não altere nada até a resposta.

## Qual modo usar

**Implementação**: o pedido é criar ou alterar uma funcionalidade, corrigir um bug, gerar código novo. Aplique as regras enquanto escreve, sem pedir permissão para isso.

**Revisão**: o pedido é olhar código que já existe (revisar, limpar, refatorar, melhorar, "passar a skill"). Mostre o que achou e espere o ok antes de alterar.

Pedido misto ("corrige esse bug e aproveita para limpar o arquivo"): faça a implementação primeiro e trate a limpeza do resto como revisão, com relatório e aprovação.

## Ordem de prioridade

As regras entram em conflito com frequência. Resolva nesta ordem:

1. **Comportamento preservado.** Refatorar é mudar a forma sem mudar o que o código faz. Um código bonito que quebrou algo é pior que o original. Cuidado com as mudanças que parecem só de estilo e alteram o resultado: trocar `==` por `===`, mudar a ordem de avaliação de uma condição, trocar retorno `null` por exceção, mexer em arredondamento ou remover um efeito colateral do qual alguém depende.
2. **Convenções do projeto.** Idioma dos nomes, estilo, padrões de pastas e de tratamento de erro que já existem. Consistência também é legibilidade, e um arquivo "limpo" num estilo diferente do resto do projeto atrapalha.
3. **Legibilidade.** As regras da seção seguinte.
4. **Simplicidade (KISS, YAGNI).** A solução mais simples que resolve o problema de hoje.
5. **DRY.** Só extraia duplicação na terceira ocorrência, ou quando as cópias representam a mesma regra de negócio. Duas cópias parecidas por coincidência ficam separadas. Reaproveitar não espera a terceira ocorrência: se o projeto já tem um helper, hook ou componente que faz a mesma coisa, use o que existe em vez de escrever outro.
6. **SOLID.** Só quando resolve um problema que existe agora. Uma interface com uma única implementação, uma factory para um único tipo ou uma camada que só repassa chamadas são abstrações que custam leitura e não entregam nada.

O erro mais comum de uma IA aplicando "boas práticas" é inflar o código: mais arquivos, mais camadas, mais indireção. Se a versão nova tem mais conceitos para o leitor guardar na cabeça do que a antiga, ela piorou.

## As regras

Resumo de cada uma. Detalhes, exemplos de antes e depois e os casos em que a regra não se aplica estão em `references/principios.md`. Leia esse arquivo antes da primeira revisão da sessão e sempre que tiver dúvida sobre aplicar ou não uma regra.

**As quatro centrais**

1. **Nomes que revelam a intenção.** O nome diz o que a coisa é ou faz, sem precisar de comentário. `calcularTotalPedido()` em vez de `calc()`, `diasDesdeUltimoAcesso` em vez de `d`. Evite `data`, `info`, `temp`, `aux` e abreviações que só o autor entende.
2. **Variável explicativa.** Dê nome a condições compostas, cálculos intermediários e números mágicos. `if (clienteEstaAtivo)` em vez de `if (u.lc > Date.now() - 2592000000)`.
3. **Uma ideia por linha.** Cada linha faz uma coisa. Quebre cadeias longas e expressões aninhadas em passos nomeados.
4. **Clareza antes de esperteza.** Entre a versão curta e engenhosa e a versão óbvia, escolha a óbvia.

**As que sustentam as centrais**

5. **Retorno antecipado.** Trate os casos de exceção no topo da função com guard clauses e deixe o caminho principal sem indentação.
6. **Funções pequenas que fazem uma coisa.** Se a descrição precisa de "e" ("valida e salva e envia e-mail"), divida. Até 3 parâmetros; acima disso, agrupe num objeto.
7. **Mesmo nível de abstração.** Uma função não mistura "processar pedido" com "concatenar string SQL". A função de cima conta a história; as de baixo cuidam dos detalhes.
8. **Sem efeitos colaterais escondidos.** O que a função faz bate com o nome dela. `verificarSenha()` não inicia sessão.
9. **Erros explícitos.** Exceções em vez de códigos mágicos, nenhum `catch` vazio, evite retornar ou passar `null` quando houver alternativa clara.
10. **Comentários explicam o porquê.** O código diz o quê. Apague comentário que repete o código e código comentado (o Git guarda o histórico).
11. **Baixo acoplamento, alta coesão.** O que muda junto fica junto. Um módulo conhece o mínimo possível dos outros.

## Modo implementação

1. **Olhe em volta antes de escrever.** Veja como o projeto nomeia as coisas (português ou inglês, camelCase ou snake_case), como trata erros e como organiza arquivos. Siga o que já existe e não misture idiomas nos nomes.
2. **Procure se já existe antes de criar.** Antes de escrever uma função, hook, componente ou constante, busque no projeto pelo nome provável e pelo que ela faz: pastas como `lib/`, `utils/` e `components/`, e o que o `CLAUDE.md` do projeto lista. Se já existe, use. Se há várias versões, use a do módulo compartilhado e cite as outras em "Oportunidades que vi e não mexi". Se a que existe não atende o caso, não crie uma segunda versão em silêncio: diga na entrega por que ela não serviu.
3. **Escreva aplicando as regras.** O código já nasce limpo; não entregue uma primeira versão para limpar depois.
4. **Fique no escopo.** Altere só o que a tarefa pede. Limpar o código vizinho gera um diff grande, que esconde a mudança real e dificulta a revisão. Se viu algo que vale melhorar fora do escopo, liste no final em "Oportunidades que vi e não mexi", com no máximo 5 itens.
5. **Rode o que o projeto tem.** Formatador, linter e testes, se existirem. Formatação é trabalho da ferramenta (Prettier, ESLint, PHP-CS-Fixer, Black, Ruff), então não reformate à mão.
6. **Chame o revisor**, em toda mudança de código, pequena ou grande (veja a seção seguinte).
7. **Passe o checklist final** antes de entregar.

## Revisor independente

Quem escreveu o código não consegue julgar se ele está claro para quem nunca o viu. O subagente `revisor-codigo-limpo` lê a mudança com o contexto limpo, só com ferramentas de leitura, e devolve achados. Ele aponta; quem corrige é você, que conhece o motivo de cada decisão.

**Quando chamar.** Sempre que você escrever ou alterar código, seja uma linha ou vários arquivos. Mudança pequena segue o mesmo fluxo. Pule só quando o usuário pedir para pular.

**O que passar.** Os arquivos e os trechos alterados (com números de linha), uma frase com o objetivo da tarefa e o caminho da pasta desta skill, exatamente como está aqui: `${CLAUDE_SKILL_DIR}`. O revisor lê as regras direto desse caminho; sem ele, pode não achar a skill quando ela foi instalada por link. Não explique as suas decisões nem o que cada nome significa: o valor do revisor é ler sem esse contexto.

**Espere a resposta** antes de entregar. Não relate o resultado do revisor antes de ele terminar.

**O que fazer com os achados.**

- ALTO e MÉDIO dentro do escopo: corrija.
- Achado com o qual você discorda, porque o revisor não conhece o contexto: não aplique e diga o motivo na entrega. Se o motivo só existe na sua cabeça, considere um comentário de "porquê" no código.
- BAIXO: aplique se for barato; senão, ignore.
- Possível bug em código que você acabou de escrever: corrija e avise. Em código antigo: relate ao usuário e não mexa.
- Proposta que muda comportamento ou contrato público: não aplique; leve ao usuário.

**No máximo duas rodadas.** Depois de corrigir, chame o revisor de novo só se havia achado ALTO. Não há terceira rodada: o que sobrar vai para a entrega como pendência.

**Na entrega**, resuma em uma ou duas linhas: quantos achados, quantos corrigidos e quais ficaram de fora, com o motivo.

**Se o subagente não estiver instalado**, releia o diff inteiro de cima para baixo antes de entregar, anotando onde um leitor novo travaria, e avise o usuário de que o revisor não foi encontrado. Não diga que o revisor aprovou se ele não rodou.

## Modo revisão

1. **Entenda antes de julgar.** Leia o código alvo e quem o usa. Descubra se há testes cobrindo o trecho e quais convenções o projeto segue. Um nome que parece ruim pode ser o termo que o negócio usa.
2. **Monte o relatório** no formato abaixo. Não altere nenhum arquivo nesta etapa.
3. **Pare e espere.** Termine perguntando o que aplicar. Só continue depois da resposta.
4. **Aplique só o que foi aprovado**, um achado por vez, do menor risco para o maior. Rode os testes entre as mudanças quando existirem.
5. **Confirme e resuma.** Rode formatador, linter e testes. Diga o que mudou, o que ficou de fora e o que não deu para verificar.

### Formato do relatório

Use exatamente esta estrutura, porque ela é lida no terminal e o usuário responde pelos números:

```
## Revisão: <arquivo ou módulo>

Estado geral: <uma ou duas frases>
Testes cobrindo este trecho: <sim | parcial | não>

### Achados

1. [ALTO] caminho/arquivo.ts:42 — `calc()`
   Problema: o nome não diz o que calcula.
   Proposta: renomear para `calcularTotalPedido()`.
   Risco da mudança: baixo (função privada, 3 usos).

2. [MÉDIO] ...

### O que está bom e eu não mexeria
- <itens curtos>

Aplico todos, só alguns (diga os números) ou nenhum?
```

**Impacto do achado**

- **ALTO**: atrapalha entender o que o código faz, esconde um bug provável (efeito colateral escondido, `catch` vazio, função que faz cinco coisas) ou refaz algo que o projeto já tem num módulo compartilhado (as cópias divergem e o leitor não sabe qual é a certa).
- **MÉDIO**: exige esforço extra de leitura (nome genérico, `if` aninhado em três níveis, número mágico).
- **BAIXO**: polimento (comentário redundante, variável explicativa que ajudaria um pouco).

**Risco da mudança** é outra coisa: a chance de quebrar algo ao aplicar. Depende de quantos lugares usam o trecho, se há testes e se a mudança toca um contrato público.

Ordene por impacto. Se houver mais de 15 achados, mostre os 15 mais importantes e diga quantos ficaram de fora. Se encontrar um bug de verdade durante a revisão, relate-o separado dos achados de legibilidade, porque corrigi-lo muda o comportamento e precisa de uma decisão própria.

## Honestidade

O usuário decide o que aprovar e o que mandar para produção com base no que você relata. Um relato errado custa mais que um código feio, porque ele confia e não confere. Por isso, diga só o que você fez ou viu de fato.

- **Não invente nada.** Arquivo, linha, nome de função, achado, resultado de teste, saída de comando, resposta do revisor: cite só o que você leu ou executou nesta sessão.
- **Separe o que foi verificado do que foi suposto.** "Rodei os testes e passaram" só se você rodou e viu passar. Caso contrário: "não rodei os testes" ou "não há testes para este trecho".
- **Não afirme que o comportamento foi preservado sem dizer com que evidência** (testes que passaram, comparação de saídas, ou "mudança mecânica, não verifiquei").
- **Diga quando não sabe.** "Não sei", "não encontrei" e "não consegui verificar" são respostas válidas. Não preencha a lacuna com um palpite escrito como fato.
- **Não esconda o que deu errado.** Teste que falhou, linter com erro, achado do revisor que você não aplicou, parte da tarefa que ficou por fazer: tudo isso vai na entrega.
- **Não invente achados nem elogios.** Se o código está bom, diga que está. Se a revisão não encontrou nada relevante, o relatório fica curto.
- **Corrija-se.** Se perceber que disse algo errado antes, avise assim que notar.

## Checklist final

Antes de entregar qualquer código, nos dois modos:

- Alguém que não conhece o código entende cada nome?
- Cada linha faz uma coisa só?
- Dá para ler de cima para baixo sem voltar?
- O comportamento é o mesmo de antes, fora o que a tarefa pediu para mudar?
- Os nomes novos estão no mesmo idioma e estilo do resto do arquivo?
- Usei o que o projeto já tem, em vez de criar outra versão da mesma coisa?
- O diff contém só o que foi pedido ou aprovado?
- A versão nova tem menos coisas para o leitor guardar na cabeça, e não mais?
- Tudo o que você vai afirmar na entrega foi de fato feito ou verificado nesta sessão?

Se alguma resposta for "não", corrija antes de entregar. Se não der para corrigir, avise.

## Limites

- **Contratos públicos.** Não renomeie nem mude a assinatura de funções exportadas, rotas, campos de API, colunas de banco, variáveis de ambiente ou eventos sem aprovação explícita, mesmo no modo implementação. Quem usa esse contrato pode estar fora do repositório. Nomes de variáveis locais e de parâmetros posicionais não são contrato e podem ser renomeados. Em linguagens com argumentos nomeados (Python, PHP 8, Kotlin), o nome do parâmetro faz parte do contrato: confira quem chama antes.
- **Testes.** Use os testes existentes como rede de segurança. Não crie testes novos sem o usuário pedir. Se o trecho não tem testes e a mudança é arriscada, diga isso no relatório e sugira criá-los antes.
- **Sem testes e sem como verificar.** Prefira mudanças mecânicas e seguras (renomear variável local, extrair variável explicativa, guard clause) e marque as demais como risco alto.
- **Código gerado ou de terceiros.** Não revise migrations já aplicadas, arquivos gerados, `vendor/`, `node_modules/` nem lockfiles.

## Revisões grandes

Para revisar vários módulos de uma vez, use o `revisor-codigo-limpo` em paralelo na **fase de leitura**: uma chamada por módulo independente, cada uma devolvendo achados sem editar nada. Depois, junte tudo num relatório único no formato do modo revisão e padronize os nomes propostos, para que o mesmo conceito não receba nomes diferentes em módulos diferentes.

A **fase de aplicação** fica com você, um achado por vez, depois do ok do usuário. Edições paralelas no mesmo arquivo geram conflito e nomes inconsistentes.

Na dúvida, comece por um módulo, mostre o resultado e só então amplie.
