# gh-codigo-limpo

Skill para o [Claude Code](https://claude.com/claude-code) que faz a IA escrever e revisar código pensando em quem vai ler depois: nomes que revelam a intenção, variáveis explicativas, uma ideia por linha, retorno antecipado e funções pequenas, com SOLID, DRY, KISS e YAGNI na medida certa, sem inflar o código.

Ela vem com um subagente, o `revisor-codigo-limpo`, que lê cada mudança com o contexto limpo, como alguém que nunca viu o projeto, e aponta onde a leitura trava.

A skill só age quando você a chama com `/gh-codigo-limpo`. Fora disso, o Claude Code trabalha normalmente e não aplica o fluxo dela.

## O que tem aqui

```
skills/gh-codigo-limpo/
├── SKILL.md                    # regras, os dois modos de uso, honestidade e checklist
└── references/principios.md    # cada regra com exemplo de antes e depois e quando não se aplica
agents/
└── revisor-codigo-limpo.md     # subagente revisor, somente leitura
```

## Como funciona

A skill tem dois modos. No **modo implementação**, ela aplica as regras enquanto escreve e chama o revisor em toda mudança. No **modo revisão**, ela mostra um relatório numerado e só altera alguma coisa depois do seu ok.

```mermaid
flowchart TD
    inicio(["Você digita /gh-codigo-limpo e o pedido"]) --> tipo{"Que tipo de pedido?"}

    tipo -->|"Criar ou alterar código"| i1
    tipo -->|"Revisar código que já existe"| r1

    subgraph implementacao ["Modo implementação"]
        i1["Olha as convenções do projeto"] --> i2["Escreve aplicando as regras,<br/>só no escopo da tarefa"]
        i2 --> i3["Roda formatador, linter e testes"]
        i3 --> v1["Chama o revisor-codigo-limpo<br/>com os trechos alterados<br/>(em toda mudança, pequena ou grande)"]
        v1 --> v2{"Veredito do revisor"}
        v2 -->|"AJUSTAR"| v3["A sessão principal corrige os achados"]
        v3 -->|"Havia achado ALTO<br/>e foi a 1ª rodada"| v1
    end

    subgraph revisao ["Modo revisão"]
        r1["Lê o código, os testes e as convenções"] --> r2["Monta o relatório numerado"]
        r2 --> r3(["PARA e espera o seu ok"])
        r3 --> r4{"Sua resposta"}
        r4 -->|"Todos ou alguns números"| r5["Aplica um achado por vez,<br/>do menor risco ao maior"]
        r5 --> r6["Roda os testes entre as mudanças"]
    end

    v2 -->|"APROVADO"| checklist
    v3 -->|"Nos outros casos"| checklist
    r6 --> checklist
    r4 -->|"Nenhum"| semMudanca(["Nada é alterado"])

    checklist["Checklist final<br/>(inclui: tudo o que vou afirmar foi verificado?)"] --> entrega(["Entrega honesta: o que mudou, o que ficou de fora<br/>e o que não foi verificado"])
```

> Este README e o diagrama servem só para explicar a skill. Eles não fazem parte do que é instalado: a skill é apenas a pasta `skills/gh-codigo-limpo/` e o subagente é o arquivo `agents/revisor-codigo-limpo.md`.

## Instalação

Copie a skill e o subagente para a pasta do Claude Code. Use `~/.claude/` para ter em todos os projetos, ou `.claude/` dentro de um projeto para ter só nele.

**macOS / Linux**

```bash
git clone https://github.com/GuilhermeOliveira0/gh-codigo-limpo.git
mkdir -p ~/.claude/skills ~/.claude/agents
cp -r gh-codigo-limpo/skills/gh-codigo-limpo ~/.claude/skills/
cp gh-codigo-limpo/agents/revisor-codigo-limpo.md ~/.claude/agents/
```

**Windows (PowerShell)**

```powershell
git clone https://github.com/GuilhermeOliveira0/gh-codigo-limpo.git
New-Item -ItemType Directory -Force "$HOME\.claude\skills", "$HOME\.claude\agents" | Out-Null
Copy-Item -Recurse gh-codigo-limpo\skills\gh-codigo-limpo "$HOME\.claude\skills\"
Copy-Item gh-codigo-limpo\agents\revisor-codigo-limpo.md "$HOME\.claude\agents\"
```

Abra uma nova sessão do Claude Code depois de copiar.

## Como usar

Digite `/gh-codigo-limpo` seguido do que você quer. O texto depois da barra decide o modo.

**Implementar ou alterar código**

```
/gh-codigo-limpo cria um endpoint que lista os pedidos pagos do cliente
/gh-codigo-limpo corrige o cálculo de frete em src/frete.ts
```

**Revisar código que já existe**

```
/gh-codigo-limpo revisa src/pedidos.ts
/gh-codigo-limpo limpa a pasta src/pagamentos sem mudar o comportamento
```

O relatório da revisão vem numerado. Responda com os números que quer aplicar, "todos" ou "nenhum".

Se você digitar só `/gh-codigo-limpo`, sem nada depois, ela pergunta o que fazer antes de mexer em qualquer arquivo.

A skill vale só para o pedido feito junto com a barra (e para as respostas que fazem parte dele, como a aprovação do relatório). Os pedidos seguintes que vierem sem `/gh-codigo-limpo` são atendidos normalmente, sem o fluxo da skill.

## Princípios que guiam a skill

- **Comportamento preservado vem primeiro.** Refatorar é mudar a forma, não o resultado.
- **Convenções do projeto valem mais que preferência.** Idioma dos nomes, estilo e padrões que já existem são seguidos.
- **Sem inflar o código.** Abstração só quando resolve um problema que existe hoje.
- **Escopo fechado.** Na implementação, só o que a tarefa pede; o resto vira sugestão no final.
- **Honestidade.** A entrega diz o que foi verificado, o que foi suposto e o que ficou de fora.
