# Princípios em detalhe

Cada seção tem a regra, um exemplo de antes e depois e os casos em que ela **não** se aplica. Os exemplos alternam entre TypeScript, PHP e Python de propósito: a regra vale para qualquer linguagem, mas a forma segue o idioma de cada uma (snake_case em Python, tipos em TypeScript, e assim por diante).

## Índice

1. Nomes que revelam a intenção
2. Variável explicativa
3. Uma ideia por linha
4. Clareza antes de esperteza
5. Retorno antecipado
6. Funções pequenas que fazem uma coisa
7. Mesmo nível de abstração
8. Sem efeitos colaterais escondidos
9. Erros explícitos
10. Comentários explicam o porquê
11. DRY e a regra de três
12. KISS e YAGNI
13. SOLID na medida
14. Baixo acoplamento, alta coesão
15. Formatação, testes e Regra do Escoteiro

---

## 1. Nomes que revelam a intenção

O nome responde "o que é isto?" ou "o que isto faz?" sem o leitor precisar abrir a implementação.

```ts
// Antes
const d = getData(u.id);
const list = d.filter(x => x.s === 1);

// Depois
const pedidosDoCliente = buscarPedidos(cliente.id);
const pedidosPagos = pedidosDoCliente.filter(pedido => pedido.status === STATUS_PAGO);
```

Guias práticos:

- Booleanos leem como pergunta de sim ou não: `estaAtivo`, `temEstoque`, `podeCancelar`.
- Funções começam com verbo: `calcularFrete`, `enviarNotificacao`.
- Coleções no plural, item no singular: `for (const pedido of pedidos)`.
- Unidade no nome quando não é óbvia: `tempoLimiteEmSegundos`, `precoEmCentavos`.
- Evite `data`, `info`, `item`, `temp`, `aux`, `obj`, `result`, `handle`, `process`, `manager` sozinhos.
- O tamanho do nome acompanha o tamanho do escopo: quanto mais longe do uso, mais descritivo.

**Quando não aplicar**

- Escopos de uma linha: `i` num laço simples e `(a, b) => a - b` numa ordenação são mais claros que nomes longos.
- Termos do negócio ou do framework: se a empresa chama de "SKU" ou o framework espera `req` e `res`, mantenha.
- Não troque o idioma dos nomes de um arquivo que já existe. Siga o que está lá.

## 2. Variável explicativa

Dê um nome ao que o leitor teria de decifrar: condição composta, cálculo intermediário, número ou texto solto.

```python
# Antes
def oferecer_upgrade(usuario, agora):
    if usuario.ultimo_acesso and (agora - usuario.ultimo_acesso).days < 30 and usuario.plano != "gratis":
        enviar_oferta(usuario)

# Depois
DIAS_PARA_CONSIDERAR_ATIVO = 30

def oferecer_upgrade(usuario, agora):
    if not usuario.ultimo_acesso:
        return

    dias_desde_ultimo_acesso = (agora - usuario.ultimo_acesso).days
    acessou_recentemente = dias_desde_ultimo_acesso < DIAS_PARA_CONSIDERAR_ATIVO
    e_assinante = usuario.plano != "gratis"

    if acessou_recentemente and e_assinante:
        enviar_oferta(usuario)
```

**Quando não aplicar**

- Quando a expressão já se explica: `if (pedido.estaPago)` não precisa de `const pedidoEstaPago = pedido.estaPago`.
- Quando a variável seria usada uma vez e o nome só repetiria a chamada: `const usuario = buscarUsuario(id); return usuario;`.

## 3. Uma ideia por linha

O leitor processa uma linha por vez. Se a linha busca, filtra, transforma e formata, ele precisa desmontá-la de cabeça.

```php
// Antes
return number_format(array_sum(array_map(fn($i) => $i->preco * $i->qtd, array_filter($itens, fn($i) => !$i->cancelado))) * (1 - $desconto), 2, ',', '.');

// Depois
$itensAtivos = array_filter($itens, fn($item) => !$item->cancelado);
$subtotaisPorItem = array_map(fn($item) => $item->preco * $item->qtd, $itensAtivos);
$subtotal = array_sum($subtotaisPorItem);
$totalComDesconto = $subtotal * (1 - $desconto);

return number_format($totalComDesconto, 2, ',', '.');
```

Sinais de mais de uma ideia na linha: ternário dentro de ternário, chamada dentro de chamada dentro de chamada, atribuição dentro de condição, `&&` e `||` usados para controlar fluxo.

**Quando não aplicar**

- Cadeias curtas em que cada elo é óbvio: `nomes.map(limpar).filter(Boolean)` é uma ideia só.
- Não quebre tanto que a função triplique de tamanho com variáveis que não acrescentam nome útil.

## 4. Clareza antes de esperteza

Código esperto impressiona quem escreve e custa caro para quem lê.

```ts
// Antes
const ehPar = n => !(n & 1);
const total = itens.reduce((soma, { preco, qtd }) => (soma += preco * qtd, soma), 0);

// Depois
const ehPar = (numero: number) => numero % 2 === 0;

let total = 0;
for (const item of itens) {
  total += item.preco * item.quantidade;
}
```

Prefira o recurso que um desenvolvedor júnior da linguagem reconhece. Truques de bit, regex densa, metaprogramação, `reduce` que monta estruturas complexas e comprehensions aninhadas pedem uma versão mais direta.

**Quando não aplicar**

- Quando a forma "esperta" é o idioma padrão da linguagem (uma list comprehension simples em Python, desestruturação em TypeScript).
- Quando há um motivo de desempenho medido. Nesse caso mantenha e escreva um comentário explicando o porquê.

## 5. Retorno antecipado

Resolva os casos de exceção no topo e saia. O caminho principal fica reto, sem indentação.

```ts
// Antes
function finalizarPedido(pedido) {
  if (pedido) {
    if (pedido.itens.length > 0) {
      if (pedido.pagamentoAprovado) {
        enviarParaCozinha(pedido);
      } else {
        throw new Error("Pagamento não aprovado");
      }
    } else {
      throw new Error("Pedido vazio");
    }
  } else {
    throw new Error("Pedido não encontrado");
  }
}

// Depois
function finalizarPedido(pedido) {
  if (!pedido) throw new Error("Pedido não encontrado");
  if (pedido.itens.length === 0) throw new Error("Pedido vazio");
  if (!pedido.pagamentoAprovado) throw new Error("Pagamento não aprovado");

  enviarParaCozinha(pedido);
}
```

Vale o mesmo dentro de laços, com `continue`.

**Quando não aplicar**

- Quando os dois ramos têm o mesmo peso (não há "exceção" e "caminho principal"), um `if/else` simples é mais honesto.
- Quando a função precisa liberar um recurso no final, cuide para que as saídas antecipadas não pulem a liberação (`finally`, `defer`, `with`).

## 6. Funções pequenas que fazem uma coisa

Teste: descreva a função numa frase. Se a frase precisa de "e", são duas funções.

```python
# Antes
def cadastrar_usuario(dados):
    if "@" not in dados["email"]:
        raise ValueError("E-mail inválido")
    if len(dados["senha"]) < 8:
        raise ValueError("Senha curta")
    hash_senha = bcrypt.hashpw(dados["senha"].encode(), bcrypt.gensalt())
    usuario = db.execute("INSERT INTO usuarios ...", (dados["email"], hash_senha))
    smtp.send(dados["email"], "Bem-vindo!", TEMPLATE_BOAS_VINDAS)
    return usuario

# Depois
def cadastrar_usuario(dados):
    validar_dados_de_cadastro(dados)
    usuario = salvar_usuario(dados["email"], dados["senha"])
    enviar_email_de_boas_vindas(usuario.email)
    return usuario
```

Parâmetros: até 3. Acima disso, agrupe os que andam juntos num objeto com nome (`endereco` em vez de `rua, numero, cidade, cep`). Evite parâmetro booleano que troca o comportamento (`salvar(usuario, true)`); prefira duas funções com nomes claros.

**Quando não aplicar**

- Não extraia uma função usada uma única vez se o nome dela não disser mais do que as 2 ou 3 linhas que ela esconde. Pular para outro lugar do arquivo também custa leitura.
- Não fatie uma sequência linear e simples em dez funções de uma linha.

## 7. Mesmo nível de abstração

Dentro de uma função, todas as linhas falam na mesma "altura". A função de cima narra os passos; as de baixo fazem o trabalho.

```ts
// Antes: mistura regra de negócio com detalhe de HTTP e de SQL
async function processarPedido(req, res) {
  const pedido = req.body;
  if (pedido.itens.length === 0) return res.status(400).json({ erro: "vazio" });
  const total = pedido.itens.reduce((soma, item) => soma + item.preco * item.qtd, 0);
  await db.query(`INSERT INTO pedidos (cliente_id, total) VALUES (${pedido.clienteId}, ${total})`);
  res.status(201).json({ total });
}

// Depois: o handler cuida de HTTP, o serviço cuida da regra, o repositório cuida do banco
async function criarPedidoHandler(req, res) {
  const pedidoCriado = await criarPedido(req.body);
  res.status(201).json(pedidoCriado);
}

async function criarPedido(dadosDoPedido) {
  validarPedido(dadosDoPedido);
  const total = calcularTotal(dadosDoPedido.itens);
  return repositorioDePedidos.salvar({ ...dadosDoPedido, total });
}
```

Organize o arquivo como um jornal: a função principal em cima, os detalhes embaixo, na ordem em que são chamados.

**Quando não aplicar**

- Scripts curtos e endpoints triviais. Um CRUD de dez linhas não precisa de três camadas (veja KISS).

## 8. Sem efeitos colaterais escondidos

O nome é uma promessa. Se a função faz algo além do que o nome diz, o leitor é enganado.

```php
// Antes: "verificar" também cria sessão
function verificarSenha(Usuario $usuario, string $senha): bool {
    if (password_verify($senha, $usuario->hashSenha)) {
        Sessao::iniciar($usuario);
        return true;
    }
    return false;
}

// Depois
function senhaEstaCorreta(Usuario $usuario, string $senha): bool {
    return password_verify($senha, $usuario->hashSenha);
}

function autenticar(Usuario $usuario, string $senha): void {
    if (!senhaEstaCorreta($usuario, $senha)) {
        throw new CredenciaisInvalidasException();
    }
    Sessao::iniciar($usuario);
}
```

Separe funções que **respondem** (retornam um valor, não mudam nada) das que **agem** (mudam estado). Não altere os argumentos recebidos; devolva um valor novo.

**Atenção na revisão:** separar um efeito colateral muda quem chama o quê. Trate como risco médio ou alto e confira todos os usos.

## 9. Erros explícitos

```python
# Antes
def buscar_cliente(cliente_id):
    try:
        return api.get(f"/clientes/{cliente_id}")
    except Exception:
        return None  # quem chama não sabe se não existe ou se a API caiu

# Depois
def buscar_cliente(cliente_id):
    try:
        return api.get(f"/clientes/{cliente_id}")
    except NaoEncontrado as erro:
        raise ClienteNaoEncontrado(cliente_id) from erro
```

Os outros erros sobem sem tratamento aqui, e quem chama decide o que fazer com uma API fora do ar.

- Exceções ou tipos de resultado em vez de `-1`, `false` ou `"erro"` como retorno.
- Nenhum `catch` vazio. Capture só o que sabe tratar; o resto sobe.
- A mensagem de erro diz o que aconteceu e com qual dado: `"Pedido 123 não encontrado"`.
- Evite devolver `null` para "deu errado". Para "não achei", prefira uma lista vazia, um tipo opcional ou uma exceção específica, conforme o costume da linguagem e do projeto.

**Quando não aplicar**

- Siga o padrão de erro que o projeto já usa. Não introduza um estilo novo num arquivo só.
- Trocar retorno `null` por exceção muda o comportamento para quem chama. Na revisão, é achado com proposta, não mudança automática.

## 10. Comentários explicam o porquê

```ts
// Ruim: repete o código
// incrementa o contador
contador++;

// Bom: explica uma decisão que o código não consegue mostrar
// A API do banco rejeita mais de 50 itens por requisição (limite de 2024, chamado #812).
const TAMANHO_DO_LOTE = 50;
```

Bons motivos para comentar: decisão de negócio que não é óbvia, contorno de bug de terceiros, aviso de consequência ("alterar isto exige rodar a migration X"), documentação de API pública.

Apague: comentário que descreve o que a linha faz, comentário desatualizado, código comentado, separadores decorativos, histórico de alterações.

Antes de escrever um comentário para explicar um trecho confuso, tente renomear ou extrair uma variável. Se o código ficar claro, o comentário não é necessário.

**Quando não aplicar**

- Não apague comentários de licença, anotações usadas por ferramentas (`@param`, `# type:`, `// eslint-disable`, `TODO` com dono) nem um comentário de "porquê" só por estar longo.

## 11. DRY e a regra de três

DRY é sobre **conhecimento**, não sobre texto. Duas funções com o mesmo código representam a mesma regra? Se a regra mudar, as duas mudam juntas?

- **Sim** (a mesma regra de cálculo de frete em dois lugares): unifique.
- **Não** (validação de cadastro de cliente e de fornecedor que hoje coincidem): deixe separadas. Juntar acopla coisas que vão evoluir de formas diferentes, e a função única acaba cheia de `if (tipo === ...)`.

Regra de três: na primeira vez, escreva. Na segunda, tolere a duplicação. Na terceira, extraia. Com três casos reais dá para ver qual é a abstração correta.

Uma abstração errada custa mais que a duplicação. Se a função compartilhada precisa de flags para atender cada chamador, ela provavelmente deveria ser duas.

## 12. KISS e YAGNI

- **KISS**: entre duas soluções que funcionam, escolha a que tem menos partes.
- **YAGNI**: não construa para um futuro que ninguém pediu.

Sinais de excesso que a skill deve evitar criar, e apontar na revisão:

- Interface ou classe abstrata com uma única implementação.
- Parâmetro de configuração que só tem um valor em uso.
- Camada que só repassa a chamada para a próxima.
- Factory, strategy ou builder para um caso só.
- Função genérica com parâmetros que ninguém passa.
- Código "para quando precisarmos de...".

```ts
// Excesso
interface CalculadoraDeFrete { calcular(pedido: Pedido): number }
class CalculadoraDeFretePadrao implements CalculadoraDeFrete { /* ... */ }
class CalculadoraDeFreteFactory { static criar(): CalculadoraDeFrete { return new CalculadoraDeFretePadrao(); } }

// Suficiente hoje
function calcularFrete(pedido: Pedido): number { /* ... */ }
```

Quando aparecer a segunda forma de calcular frete, a interface passa a ter motivo para existir.

## 13. SOLID na medida

Use cada princípio como diagnóstico de um problema que já existe, não como modelo a seguir desde a primeira linha.

- **Responsabilidade única.** Sintoma: a mesma classe muda por motivos diferentes (regra de negócio, formato do relatório, banco). Remédio: separar por motivo de mudança. É o princípio mais útil no dia a dia e se aplica também a funções e módulos.
- **Aberto/fechado.** Sintoma: toda variação nova exige mexer num `switch` ou `if/else` grande, em vários lugares. Remédio: polimorfismo ou um mapa de estratégias. Com duas ou três variações estáveis, o `switch` é mais legível.
- **Substituição de Liskov.** Sintoma: a subclasse lança "não suportado" ou ignora um método da classe base, ou o código pergunta `instanceof` para decidir. Remédio: rever a hierarquia; muitas vezes composição serve melhor que herança.
- **Segregação de interfaces.** Sintoma: quem implementa a interface deixa métodos vazios. Remédio: dividir em interfaces menores.
- **Inversão de dependência.** Sintoma: não dá para testar a regra de negócio sem banco, rede ou relógio reais. Remédio: receber a dependência por parâmetro ou construtor. Injetar a dependência resolve na maioria dos casos; a interface só é necessária quando há mais de uma implementação (contando o dublê de teste, se a linguagem exigir).

Em código funcional ou em scripts, os mesmos princípios valem em outra forma: funções com um motivo para mudar, dependências passadas como argumento, comportamento variável passado como função.

Na revisão, só proponha uma mudança de SOLID quando conseguir apontar o sintoma concreto no código. "Viola o princípio X" sozinho não é um achado.

## 14. Baixo acoplamento, alta coesão

- **Coesão**: o que está no mesmo módulo trata do mesmo assunto e muda junto. Um arquivo `utils` com funções de data, de texto e de preço tem coesão baixa.
- **Acoplamento**: quanto um módulo precisa saber do outro. `pedido.cliente.endereco.cidade.nome` acopla quem lê à estrutura inteira; `pedido.cidadeDeEntrega()` não.

Sinais de acoplamento alto: mudar um arquivo obriga a mudar vários outros; um módulo importa detalhes internos de outro; estado global compartilhado; dependência circular.

Na revisão, mover código entre módulos é mudança de risco alto e diff grande. Proponha, explique o ganho e deixe o usuário decidir.

## 15. Formatação, testes e Regra do Escoteiro

**Formatação.** Indentação, aspas, ponto e vírgula e ordem de imports são trabalho do formatador e do linter do projeto. Rode a ferramenta em vez de ajustar à mão. Se o projeto não tem uma, siga o estilo do arquivo e mencione isso como oportunidade, sem instalar nada por conta própria.

**Testes.** Código limpo é fácil de testar, e dificuldade para testar costuma indicar acoplamento ou função que faz demais. A skill usa os testes existentes para confirmar que o comportamento não mudou e não cria testes sem pedido. Quando o usuário pedir testes, eles seguem as mesmas regras de legibilidade e o FIRST: rápidos, independentes, repetíveis, com resultado claro de passou ou falhou, escritos junto com o código.

**Regra do Escoteiro.** "Deixe o código mais limpo do que encontrou" vale dentro do escopo da tarefa: as funções que você já está alterando saem melhores. Fora desse escopo, a melhoria vira item da lista "Oportunidades que vi e não mexi" ou um achado de revisão. Assim o diff continua pequeno e o usuário consegue revisar o que mudou.
