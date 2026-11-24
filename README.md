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
        i1["Olha as convenções do projeto<br/>e procura o que já existe"] --> i2["Escreve aplicando as regras,<br/>só no escopo da tarefa"]
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

Clone o repositório e ligue a skill e o subagente à pasta do Claude Code com um link, em vez de copiar. Assim, um `git pull` no clone atualiza a skill em todos os projetos de uma vez. Use `~/.claude/` para ter em todos os projetos, ou `.claude/` dentro de um projeto para ter só nele.

**macOS / Linux**

```bash
git clone https://github.com/GuilhermeOliveira0/gh-codigo-limpo.git ~/gh-codigo-limpo
mkdir -p ~/.claude/skills ~/.claude/agents
ln -s ~/gh-codigo-limpo/skills/gh-codigo-limpo ~/.claude/skills/gh-codigo-limpo
ln -s ~/gh-codigo-limpo/agents/revisor-codigo-limpo.md ~/.claude/agents/revisor-codigo-limpo.md
```

**Windows (PowerShell)**

```powershell
git clone https://github.com/GuilhermeOliveira0/gh-codigo-limpo.git "$HOME\gh-codigo-limpo"
New-Item -ItemType Directory -Force "$HOME\.claude\skills", "$HOME\.claude\agents" | Out-Null
New-Item -ItemType Junction -Path "$HOME\.claude\skills\gh-codigo-limpo" -Target "$HOME\gh-codigo-limpo\skills\gh-codigo-limpo"
Copy-Item "$HOME\gh-codigo-limpo\agents\revisor-codigo-limpo.md" "$HOME\.claude\agents\"
```

No Windows, a pasta da skill entra por junction, que não exige administrador. O arquivo do subagente é copiado porque link de arquivo no Windows exige administrador ou o Modo de Desenvolvedor ligado. Com um dos dois, troque o `Copy-Item` por:

```powershell
New-Item -ItemType SymbolicLink -Path "$HOME\.claude\agents\revisor-codigo-limpo.md" -Target "$HOME\gh-codigo-limpo\agents\revisor-codigo-limpo.md"
```

**Para atualizar**, rode `git pull` no clone. Se o subagente foi copiado, copie o arquivo de novo depois do pull.

O link aponta para a pasta clonada, então a skill instalada é a da branch que estiver ativa nela. Se você trocar de branch no clone, a skill muda junto.

**Permissão para o revisor ler as regras.** O revisor lê o `SKILL.md` e o `principios.md` direto da pasta da skill, que fica fora do projeto, e o Claude Code pede permissão para qualquer leitura fora do projeto. Para não ter de aprovar isso a cada revisão, libere a leitura dessa pasta em `~/.claude/settings.json`. Com a skill instalada por link, a regra precisa cobrir os dois caminhos, o do link e o do clone; só um deles não basta.

```json
{
  "permissions": {
    "allow": [
      "Read(~/.claude/skills/gh-codigo-limpo/**)",
      "Read(~/gh-codigo-limpo/skills/gh-codigo-limpo/**)"
    ]
  }
}
```

Troque o segundo caminho se você clonou o repositório em outro lugar. Se instalou por cópia, a primeira regra basta. Sem a permissão, o revisor avisa que não conseguiu ler as regras e revisa só com as instruções do próprio arquivo.

Abra uma nova sessão do Claude Code depois de instalar.

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
- **Reaproveitar antes de criar.** Antes de escrever um helper, hook ou componente, a skill procura se o projeto já tem um que faz a mesma coisa.
- **Escopo fechado.** Na implementação, só o que a tarefa pede; o resto vira sugestão no final.
- **Honestidade.** A entrega diz o que foi verificado, o que foi suposto e o que ficou de fora.
