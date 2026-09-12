# Arquitetura

Como o interpretador funciona por dentro, e as decisões que valem
lembrar antes de mexer em qualquer coisa.

Este documento é mantido junto do código, em
[`CLAUDE.md`](https://github.com/estevam5s/DataForge/blob/main/CLAUDE.md),
e serve de contexto tanto para quem contribui quanto para ferramentas
de assistência. Ele é atualizado no mesmo commit da mudança que
descreve — por isso o tom é direto e cheio de "por que isto foi feito
assim", que é a parte que se perde primeiro.

---

## O que é este projeto

**DataForge** é uma linguagem de programação interpretada, de propósito geral,
implementada em Python 3.10+ **sem dependências externas no runtime**. Não é um
DSL nem um transpilador: tem lexer, parser recursivo descendente, AST tipada,
analisador estático e interpretador de árvore próprios.

- Versão atual: **1.0.0**
- Extensão dos arquivos: `.df`
- Entrypoints: `dataforge` e `df` (mesmo `main`)
- Licença: MIT

### Verificação rápida — rode antes e depois de mexer

```bash
python3 -m pytest tests/ -q                          # mais de 1850 testes
python3 exercicios/run_all.py                        # 227 exercícios
python3 tools/verificar_docs.py                      # os códigos do site compilam
for f in examples/*.df; do python3 -m dataforge run "$f" >/dev/null || echo "FALHOU $f"; done
```

**Cuidado com instalação velha no PATH.** Há três lugares onde o
DataForge pode estar instalado (`.venv/`, `~/.dataforge/`, o Python do
sistema), e uma cópia antiga produz erros que não existem no repositório
— foi assim que um `LexError: Unexpected character: '$'` apareceu num
arquivo que usava interpolação normalmente. O `.venv` deve estar em modo
editável (`pip install -e .`), que aponta para o repo e nunca envelhece.

O estado esperado é **tudo verde**. Se algo falhar antes da sua mudança, diga
isso ao usuário em vez de assumir que foi você.

---

## Mapa do código

```
dataforge/
  tokens.py        305   TokenType (enum) + KEYWORDS (81 palavras reservadas)
  lexer.py         556   texto → tokens. INDENT/DEDENT, interpolação, '//' vs '~/'
  parser.py       1941   recursivo descendente: tokens → AST
  ast_nodes.py     706   dataclasses dos nós
  interpreter.py  2703   interpretador de árvore — quase toda a semântica
  compilador.py    330   a árvore vira fechamentos, uma vez (1,5× a 1,8×)
  ponte.py         290   'adopt Python.numpy' — a ponte para o Python
  cauda.py         170   'yield f(…)' vira salto, e a recursão deixa de ter teto
  typechecker.py  1752   análise estática: nomes, aridade, tipos, alcance
  resolucao.py     190   onde mora o módulo de um 'adopt' — a única cópia
  superficie.py    300   o que um .df oferece, sem executá-lo
  formatter.py     280   dataforge fmt
  linter.py        394   dataforge lint
  testrunner.py    380   dataforge test, com cobertura de linha
  cobertura.py     170   quais linhas os testes executaram
  docgen.py        218   dataforge doc
  project.py       184   forge.toml
  environment.py    91   cadeia de escopos
  errors.py        167   hierarquia de erros, sinais de controle, stack traces
  builtins.py     1224   225 funções globais, sem import
  repl.py          409   console interativo
  cli.py          1055   CLI + templates de projeto
  stdlib/                38 módulos (1325 símbolos), incluindo:
    catalogo.py          o nome, o apelido e o "para quê" de cada módulo
    kiln.py              Kiln — o framework web (73 símbolos)
    kiln_tempo_real.py   upload multipart, SSE e WebSocket (RFC 6455)
    vitrine/             Vitrine — dashboards e data apps (113 símbolos)
    arcane_excel.py      planilhas .xlsx, sem dependência externa (29)
    arcane_arquivo_seguro.py  cofre de arquivo + zip/tar seguro (56)
    cifra.py             ChaCha20-Poly1305 puro (RFC 8439)

doc/               INSTALACAO, TUTORIAL, REFERENCIA, BIBLIOTECA_PADRAO,
                   KILN, ANALISE_E_ROADMAP (todos em pt-BR)
examples/          44 programas de demonstração
exercicios/        227 exercícios em 31 módulos + run_all.py
                   (os módulos 11-23 têm um .md explicativo por exercício)
projetos/          4 programas completos com forge.toml e testes
tools/             gerar_doc_stdlib, gerar_gramatica, gerar_ref_kiln
tests/             test_dataforge.py (legado), test_regressoes.py, test_dataforge4.py
tools/             gerar_doc_stdlib.py
editor/vscode/     extensão do VS Code — gramática **gerada** de tokens.py,
                   snippets, ícone. Instalada por 'dataforge editor'.
```

### Fluxo de execução

```
arquivo.df → tokenize() → parse() → [check_program()] → Interpreter().run(ast)
             lexer.py     parser.py  typechecker.py     interpreter.py
```

O interpretador despacha por nome de classe: um nó `GivenBlock` procura
`exec_GivenBlock`; uma expressão `BinaryOp` procura `eval_BinaryOp`.

**Para adicionar um recurso à linguagem, você mexe em cinco lugares**:
`tokens.py` (se houver palavra/símbolo novo) → `lexer.py` → `ast_nodes.py` +
`parser.py` → `interpreter.py` → `typechecker.py`.

---

## A linguagem em 90 segundos

```dataforge
// atribuição, constante, tipo opcional, saída
x := 10
steady PI := 3.14159
idade: Integer := 30
out $"x vale {x}, o dobro é {x * 2}"

// condicional e ternário
given x bigger 5:
    out "grande"
orif x is 5:
    out "cinco"
otherwise:
    out "pequeno"
rotulo := "par" given x % 2 is 0 otherwise "impar"

// laços
cycle i from 1 to 5 step 2:
    out i
cycle item in [1, 2, 3]:
    out item
persist x bigger 0:
    x -= 1

// coleções, compreensões, fatiamento, spread
nums := [1, 2, 3, 4, 5]
out [n * n cycle n in nums given n % 2 is 0]
out nums[1:3], nums[::-1]
out [...nums, 6]
primeiro, ...resto := nums

// ação com tipos
action somar(a: Integer, b: Integer) -> Integer:
    yield a + b

// record: imutável, igualdade estrutural
record Ponto:
    x: Integer
    y: Integer
    action norma():
        yield sqrt(self.x ** 2 + self.y ** 2)

p := Ponto(3, 4)
p2 := p with {"y": 0}

// enum
enum Status:
    Ativo
    Inativo := "off"

// blueprint: mutável, herança, traits
blueprint Forma:
    action area():
        yield 0

blueprint Quadrado(lado) extends Forma:
    action area():
        yield self.lado ** 2

// pattern matching
match valor:
    point Integer as n when n bigger 100:
        yield "grande"
    point [a, b]:
        yield "par"
    point Ponto(x, y):
        yield "ponto"
    point {"tipo": t}:
        yield "vault"
    point Status.Ativo:
        yield "ligado"
    default:
        yield "outro"

// erros
monitor:
    trigger "falhou"
handle RuntimeError as e:
    out e.type, e.message
ensure:
    out "sempre roda"

// pipeline
out [1, 2, 3, 4, 5, 6]
    >> sift n: n % 2 is 0
    >> morph n: n * 10
    >> distill acc, v: acc + v 0

// generator preguiçoso
stream action fib():
    a := 0
    b := 1
    persist yes:
        emit a
        a, b := b, a + b
out fib().take(8)

// módulos
adopt Arcane.Math as Math
adopt Arcane.Math.{sqrt, floor}
adopt {sqrt as raiz} from Arcane.Math
relay somar, Ponto
```

### Tabela de tradução

| Conceito | DataForge |
|----------|-----------|
| `=` | `:=` |
| `const` | `steady` |
| `print` | `out` |
| f-string | `$"texto {expr}"` |
| `if/elif/else` | `given/orif/otherwise` |
| ternário | `a given cond otherwise b` |
| `switch`/`match` | `match` / `point` / `when` / `default` |
| `for` | `cycle … from … to` / `cycle … in` |
| `while` | `persist` |
| `do..while` | `perform … persist` |
| `break`/`continue` | `halt`/`skip` |
| `def`/`return` | `action`/`yield` |
| generator | `stream action` / `emit` |
| `class`/`new` | `blueprint`/`spawn` |
| `@dataclass(frozen)` | `record` |
| `enum` | `enum` |
| `self`/`super` | `self`/`root` |
| `interface` | `trait` |
| `import`/`export` | `adopt`/`relay` |
| chamar biblioteca Python | `adopt Python.numpy as np` |
| `try/catch/finally` | `monitor/handle/ensure` |
| `throw` | `trigger` |
| `true/false/null` | `yes/no/void` |
| `??` / `?.` | iguais |
| `in` / `not in` | iguais |
| spread `...` | igual |
| list comprehension | `[expr cycle x in fonte given cond]` |
| `filter/map/reduce` | `>> sift` / `>> morph` / `>> distill` |
| lambda | `lambda x: expr` ou `lambda a, b => expr` |
| decorator | `mark @nome` |
| `//` (div. inteira) | **`~/`** |
| `Decimal` exato | `adopt Arcane.Decimal as Dec` · `Dec.de("0.1")` |

---

## Armadilhas — leia antes de escrever `.df`

Estas são as que mais custam tempo:

1. **`yield` retorna, `emit` produz.** `yield` encerra a ação. Para uma
   sequência, use `stream action` + `emit`.

2. **Divisão inteira: use `~/`.** `//` é **comentário por padrão** desde o 4.0.
   Só vira divisão quando seguido de dígito, `(`, ou chamada/índice/membro.
   `x // 2` é divisão; `x // nota` é comentário.

3. **Só espaços na indentação.** Tab é `SyncError`. 4 espaços por nível.

4. **`monitor` sem `handle` não engole o erro.** Ele só garante o `ensure`.

5. **`halt`, `skip` e `yield` atravessam `monitor`.** São `ControlSignal`
   (derivam de `BaseException`). Nunca capture `BaseException` no interpretador.

6. **Palavras reservadas não podem ser nomes.** As que mais pegam em português:
   `no`, `in`, `is`, `to`, `from`, `as`, `step`, `point`, `default`, `frame`,
   `stream`, `emit`, `forge`, `record`, `enum`, `when`. Já `range`, `cluster` e
   `vault` **são funções**, não reservadas.

7. **`self` dentro de métodos, sempre.** Escrever `x` em vez de `self.x` lê a
   variável do escopo externo.

8. **Records são imutáveis.** `p.x := 1` é erro; use `p with {"x": 1}`.

9. **Padrão de sequência não casa com vault**, e vice-versa. `[a, b]` só casa com
   `Cluster`; `{"k": v}` casa com `Vault`, record ou instância.

10. **A ordem dos `point` importa.** Do específico ao geral. Uma captura
    (`point n`) no topo torna tudo abaixo inalcançável.

11. **String simples não cruza linhas.** Para SQL multilinha, use `"""..."""`.

12. **`cycle from … to` é inclusivo** nos dois extremos.

13. **Generator infinito + `to_cluster()` trava.** Use `take(n)` ou garanta um
    `halt`.

14. **A linguagem não sincroniza sozinha.** Duas threads escrevendo na
    mesma variável perdem atualizações. `Arcane.Concurrent` tem `mutex`,
    `semaforo`, `contador` e canal bloqueante — mas usá-los é escolha de
    quem escreve. Vale para o Kiln, que atende **um pedido por thread**.

15. **Dentro de `$"{…}"`, aspas normais.** `$"item {v["id"]}"` funciona;
    `$"item {v[\"id\"]}"` não — o lexer copia strings aninhadas verbatim, e o
    escape quebra a leitura. A mensagem ("Unterminated interpolation") não
    aponta para a causa.

16. **`query["x"]` sem `??` dá 500 numa rota.** A query, o corpo e os
    cabeçalhos vêm de fora: a chave pode não vir, e indexar um vault sem a
    chave é erro. `params` é a exceção — se a rota casou, o parâmetro existe.

17. **O valor inicial do `distill` vem depois do corpo.**
    `>> distill a, v: a + v 0 / len(x)` divide o **zero**, não a soma. O
    resultado fica errado sem nada denunciar.

18. **`trigger` levanta `TriggerError`, não `RuntimeError`.**
    `handle RuntimeError` não pega um `trigger`. Para pegar qualquer
    coisa, `handle Error`.

19. **Campo declarado com padrão mutável é copiado no `spawn`.**
    `itens: Cluster := []` dá uma lista nova por instância — o literal é
    avaliado uma vez, na declaração, e sem a cópia todas compartilhariam
    a mesma. Padrões imutáveis (número, texto) não são copiados.

20. **`<T>` não é verificado em execução.** O parâmetro de tipo é aceito
    pelo analisador e documenta a relação entre entrada e saída, mas a
    linguagem é dinâmica: `action eco<T>(x: T) -> T` aceita qualquer
    valor. Um tipo **concreto** continua sendo cobrado.

21. **Um decorador que devolve `void` não substitui o alvo.** É o que
    permite `@Rota("/x")` só anotar. Se ele devolvesse `void` e isso
    virasse o novo valor, a ação decorada sumiria.

---

## Convenções ao mexer no interpretador

### Estilo

- Comentários e docstrings do runtime em **inglês** nos arquivos antigos,
  **português** nos módulos 4.0 (`typechecker`, `formatter`, `linter`,
  `testrunner`, `docgen`, `project`, e os `arcane_*` novos). Siga o arquivo.
- Documentação em `doc/`, exercícios e mensagens ao usuário final em
  **português**.
- Sem dependências externas em `dataforge/`. A stdlib usa apenas a stdlib do
  Python.
- Mensagens de erro devem dizer **o que fazer**. Compare:
  `"Invalid assignment target"` (ruim) com `"'no' is a reserved keyword and
  cannot be assigned to. Pick another name."` (bom). Quando houver um nome
  parecido, sugira: o `typechecker` usa `difflib` para isso.

### O compilador de fechamentos

`compilador.py` percorre a árvore **uma vez** e devolve, para cada nó, um
fechamento que faz o que aquele nó faz. Executar passa a ser chamar
fechamentos: sem tabela de despacho, sem `isinstance`, sem `node.campo`.

Três regras ao mexer nele:

1. **Cada construtor espelha um `eval_`/`exec_`, e delega aos mesmos
   auxiliares.** A semântica não é reimplementada — `_operar`, `_comparar`,
   `_chamar_metodo`, `_ler_membro` e `_escrever_membro` são os mesmos. Quando
   um `eval_X` avalia as partes e depois decide, extraia a decisão para um
   auxiliar que receba os valores prontos, como foi feito nos cinco acima.

2. **O que não estiver nas tabelas recua** para `interp.evaluate`/`execute`,
   que é o comportamento de hoje byte por byte. Um recurso novo na linguagem
   continua funcionando sem tocar aqui; só não fica mais rápido. Devolver
   `None` de um construtor também recua — é como os casos difíceis
   (`f(...xs)`, `v["k"] += 1`) ficam de fora sem duplicar regra.

3. **O depurador desliga tudo** (`interp.compilar_corpos = False`). Ele para
   em cada linha sombreando `execute`, e o corpo compilado passa por fora —
   um depurador que enxerga metade das instruções é pior que um interpretador
   mais lento.

`tests/test_desempenho.py` roda uma amostra dos exercícios com a compilação
ligada e desligada e compara a saída caractere por caractere. É esse teste que
pega um fechamento que divergiu do método que ele espelha.

### Otimizar: meça antes

Três gargalos já foram medidos e resolvidos (`cProfile`, não intuição):
despacho por string (virou tabela por classe), alocação de escopo por
volta de laço (reaproveitado quando o corpo não captura) e construção
de AST em tempo de execução (`x += 1` montava dois nós por volta).

A carga de referência está em `tests/test_desempenho.py`; use
`dataforge profile` num programa real antes de mexer em qualquer coisa.

**O reaproveitamento de escopo é a otimização mais perigosa do
interpretador.** Se o corpo do laço captura o escopo — uma ação, um
`lambda`, um `blueprint`, um `thread`, um `defer` — cada volta precisa
do seu, senão todas as closures veem o último valor.
`_corpo_captura_escopo` varre a árvore inteira, e não só as instruções:
um `lambda` vive dentro de uma expressão.

### Ao adicionar um recurso à linguagem

1. `tokens.py` — o `TokenType` e, se for palavra, a entrada em `KEYWORDS`.
2. `lexer.py` — reconhecer o símbolo (operadores de 2 chars vão no
   `two_char_map`; de 3, antes dele).
3. `ast_nodes.py` — o nó, como `@dataclass` com defaults.
4. `parser.py` — o método `parse_*`, ligado em `parse_statement` ou
   `parse_primary`.
5. `interpreter.py` — `exec_<Nó>` para instrução, `eval_<Nó>` para expressão.
6. `typechecker.py` — `st_<Nó>` ou `ex_<Nó>`, senão o analisador ignora o
   recurso novo.
7. **Teste em `tests/test_dataforge4.py`** e um exercício em `exercicios/`.
8. Atualize `doc/REFERENCIA.md` — inclusive a §1.6 e a gramática EBNF.

### Ao mexer em `KEYWORDS`

Toda palavra em `KEYWORDS` deixa de poder ser identificador. Antes de adicionar
uma, confirme que o parser realmente a consome:

```bash
grep -c "TokenType.NOVA\b" dataforge/parser.py    # precisa ser > 0
```

Se for 0, ela só quebra código de usuário sem entregar nada. Sete palavras já
foram removidas por esse motivo.

Depois, sincronize `doc/REFERENCIA.md` §1.6 — **há um teste que compara as duas**
(`test_referencia_lista_exatamente_as_palavras_reservadas`).

### Ao mexer na stdlib

Os módulos são dicionários criados em `__new__`. Cuidado: dentro de um
`@staticmethod` referenciado por outro método da classe, use o nome da classe
(`ArcaneCortex._softmax`), **nunca `cls`** — `cls` não existe ali e o módulo
inteiro deixa de carregar.

Funções que recebem "um campo" devem aceitar **vault, record e instância**. Há
um helper para isso em `arcane_collections.py` (`_campo_de`). Esse foi um bug
real: `sort_by_field` devolvia `None` para todos os records.

Confirme que todos carregam e regenere a doc:

```bash
python3 -c "
import sys; sys.path.insert(0,'.')
from dataforge.stdlib import get_module, list_modules
for m in sorted(set(list_modules())): assert get_module(m) is not None, m
print('ok')"

python3 tools/gerar_doc_stdlib.py
```

Ao criar um módulo novo, registre-o em `stdlib/__init__.py` **e** descreva-o no
`DESCRICOES` de `stdlib/catalogo.py` — daí saem a doc em Markdown, os dados do
site e a tabela de `/docs/biblioteca`. A tabela já esteve escrita em três lugares,
e os três divergiram.

Os apelidos (`Zip`, `Cor`, `Banco`) são traduzidos para o nome oficial por
`_CANONICO` antes do carimbo de `__name__`: o mesmo módulo precisa se chamar
`Arcane.Archive` venha por `Zip`, por `Archive` ou pelo nome inteiro.

---

## As ferramentas

| Comando | Arquivo | Faz |
|---------|---------|-----|
| `dataforge check` | `typechecker.py` | nomes, aridade, tipos, alcance (aceita arquivo, pasta ou padrão) |
| `dataforge test` | `testrunner.py` | descobre `*_test.df`, `tests/`; `--cobertura` e `--minimo=80` |
| `dataforge fmt` | `formatter.py` | formata (`--check` só verifica) |
| `dataforge lint` | `linter.py` | estilo e higiene |
| `dataforge doc` | `docgen.py` | Markdown a partir dos comentários |
| `dataforge init` | `project.py` | cria `forge.toml` e esqueleto |
| `dataforge info` | `project.py` | mostra o manifesto |
| `dataforge repl` | `repl.py` | console com `:type`, `:ast`, `:load` |
| `dataforge editor` | `cli.py` | instala a coloração no VS Code e derivados |
| `dataforge new` | `modelos.py` + `scaffold.py` | 9 modelos; todo projeto criado passa nos próprios testes |
| `dataforge vitrine` | `vitrine_cli.py` | `run`, `dev` (hot reload), `doctor`, `new`. Sem `build` nem `deploy` — e os dois explicam por quê |
| `dataforge stats` | `cli.py` | inventário: ações, blueprints, o arquivo e a ação mais longos |
| `dataforge profile` | `cli.py` | tempo **próprio** por ação (o acumulado somaria mais de 100%) |
| `dataforge fix` | `cli.py` | formata e aponta o que exige julgamento |
| `dataforge add/remove` | `packages.py` | instala e desinstala dependências |
| `dataforge install` | `packages.py` | resolve o `forge.toml` inteiro |
| `dataforge search` | `packages.py` | procura no registro |
| `dataforge pack/publish` | `packages.py` | empacota e publica |

### As ferramentas não podem morrer no meio da pasta

`check`, `fmt` e `lint` aceitam arquivo, pasta ou padrão, e resolvem a lista
com `_expandir()`. A leitura passa por `_ler()`, que devolve `(fonte, None)`
ou `(None, motivo)`: um `.df` fora de UTF-8 é reportado e os demais seguem.
Antes disso, um único arquivo mal codificado derrubava `fmt .` inteiro com um
`UnicodeDecodeError` cru — e `check` numa pasta estourava `IsADirectoryError`.

Há teste para os dois casos em `tests/test_regressoes.py`, mais um que proíbe
qualquer `.df` fora de UTF-8 no repositório.

### Cobertura de linha

`dataforge test --cobertura` diz quais linhas rodaram, e
`--minimo=80` reprova no CI. Os dois números têm um jeito próprio de
mentir, e o arquivo `cobertura.py` existe para que nenhum deles minta:

| Metade | De onde vem | Como mentiria |
|---|---|---|
| denominador | o parser: quais linhas são **executáveis** | contar comentário e linha vazia dá um número sempre pessimista |
| numerador | `execute` sombreado, como o depurador faz | com `compilar_corpos` ligado, o corpo das ações passa por fora e toda ação dá 0% |

**A definição de "instrução" é a existência de `exec_<Nó>` no
interpretador**, e não uma lista. A primeira versão era uma lista e
apodreceu antes de ser commitada: tinha `CycleLoop`, e o nó se chama
`CycleFromTo` — o laço inteiro ficava fora do denominador, e a cobertura
saía **otimista**, que é o pior defeito possível numa métrica.

A linha da declaração de `action` não conta; o corpo conta. Assim uma
ação nunca chamada aparece com **0%** e não 20%. E um arquivo que
nenhum teste toca aparece com 0% em vez de sumir do relatório — sumir é
o que faz uma cobertura de 95% conviver com metade do sistema sem teste.

`forge_modules/` ficou fora da descoberta: um projeto com 13 testes
relatava **89**, e a suíte ficava vermelha por falha de uma biblioteca
que ninguém escreveu.

### Toda ação sabe em que arquivo nasceu

`DFAction.arquivo` é carimbado no **construtor**, e `_corpo_da_acao`
troca `self.filename` enquanto o corpo roda. Duas coisas dependiam
disso e as duas estavam erradas:

1. Um `1 / 0` na linha 5 de `lib.df` era reportado como `main.df:5`,
   **com o trecho do outro arquivo desenhado embaixo da seta**. Em
   projeto grande isso manda a pessoa depurar o arquivo errado.
2. A cobertura de um módulo importado era contada no arquivo de teste.

O carimbo é no `__init__` porque há **nove** lugares que criam uma
`DFAction` — método de blueprint, de record, propriedade, operador,
lambda, método mágico. Carimbar em cada um deixaria de fora os que
vierem depois, e a falta não dá erro: só faz o arquivo errado aparecer.

`Error.render` também passou a **ler do disco o arquivo que ela nomeia**
quando o que recebeu não é dele.

### O analisador atravessa arquivos

`P.naoExiste()` e `P.criar(1, 2, 3)` são acusados **antes de rodar**,
mesmo quando `P` vem de outro `.df`. É a checagem que mais importa em
sistema grande: num arquivo de 40 linhas o erro aparece na primeira
execução; num de 200 arquivos, a maioria das chamadas é entre módulos, e
todas elas eram invisíveis.

Três arquivos sustentam isso:

| Onde | O quê |
|---|---|
| `resolucao.py` | onde mora o módulo que um `adopt` pede — **a única cópia** |
| `superficie.py` | o que um `.df` oferece, lido com lexer e parser, **sem executar** |
| `typechecker.py` | `st_AdoptStatement` guarda a superfície; `ex_MemberAccess` e `_conferir_chamada_de_modulo` cobram |

A superfície lê só o que `relay` exporta, quando há `relay` — um módulo
que declara o que exporta está dizendo que o resto é interno. Ela é
**conservadora**: devolve `aberta = yes`, e o analisador volta a calar,
quando o outro arquivo não compila, quando há ciclo de import, quando a
profundidade (4) acaba, ou quando o `relay` nomeia algo que só existe em
execução. Um falso alarme é pior que um silêncio.

O cache é por `(caminho, mtime)`. Sem ele, 200 arquivos importando três
vizinhos cada levariam o `check` de 0,7 s a mais de um minuto.

**A resolução de caminho estava escrita em dois lugares, e divergiu.**
O analisador fazia `nome.replace('.', os.sep)`, o que transforma
`'./mod'` em `'//mod'`: todo `adopt` relativo de todo projeto gerava um
aviso "Module not found" falso — 62 no repositório, e **795 de 795** num
projeto de 21 mil linhas. Cada aviso que o `check` emitia ali era
mentira, o que é pior que não avisar nada.

E um pacote não sabia se importar pelo **próprio nome**. O teste de uma
biblioteca escreve `adopt validador`, não `adopt ../src/main`, porque
precisa exercitá-la pelo caminho que um usuário usaria: as suítes dos
**vinte** pacotes do repositório falhavam, e a CI não apanhava — ela não
rodava `dataforge test` dentro de `packages/`.
E `dataforge deps` tinha a **terceira** cópia da regra — uma expressão
regular que começava em `[A-Za-z_]`, então `./vizinho` nunca casava. O
comando cuja única função é mostrar o grafo de imports dizia "0 arquivos
com imports próprios" em todo projeto do repositório, e a detecção de
ciclo nunca disparava. Hoje ele usa o parser e o `resolucao.py`.

**Hífen num caminho relativo não compilava.** `adopt ./minha-lib as L`
falhava: o lexer entrega o hífen como `MINUS`, o loop de segmento parava
ali, e o parser reclamava de um `as` inesperado. `_segmento_de_caminho`
cola `-`, `.` e dígitos ao nome, exigindo **adjacência de coluna** — sem
essa guarda, `a - b` viraria um arquivo chamado `a-b`.

`tests/test_resolucao.py` cobre os quatro, e proíbe a cópia voltar.

**Ciclo de import agora é erro do `check`.** Ele estourava só em
execução, no primeiro `adopt`, e o `check` passava limpo num projeto que
não sobe. `superficie.ciclo_a_partir_de` faz busca em **largura**, para
achar o ciclo mais curto — o mais fácil de quebrar — e a mensagem mostra
a cadeia inteira (`a.df → b.df → c.df → a.df`), porque um ciclo de
quatro arquivos é impossível de quebrar sem saber por onde ele passa.

### O analisador vê dentro dos objetos

`p.clientte` é acusado antes de rodar, com sugestão. É a checagem que
mais importa em projeto grande: num arquivo de 40 linhas o erro aparece
na primeira execução; num sistema de 200 arquivos, aparece em produção.

Ela só acusa quando consegue **provar**, e o que a faz calar é tão
importante quanto o que a faz falar:

| Cala quando | Porque |
|---|---|
| o membro vem da mãe ou de um trait | herdado é tão legítimo quanto declarado |
| o blueprint herda de algo não visto | ele pode ganhar qualquer membro |
| o campo nasceu de `self.x := …` | é como a maioria do código cria estado — **inclusive solto no corpo do blueprint**, que é o construtor inline |
| alguém fez `obj.x := …` de fora | quem faz isso abre mão da conferência ali |
| o nome começa com `__` | método mágico é chamado pelo runtime |

A primeira versão olhava `self.x := …` só dentro de métodos, e deu **32
falsos alarmes** num exemplo que funciona há meses. `test_membros_de_instancia.py`
roda o `check` sobre os 320 arquivos do repositório justamente por isso.

### O analisador estático é otimista de propósito

Quando não consegue **provar** que algo está errado, fica calado. Um falso alarme
ensina o usuário a ignorar mensagens. Dentro de `monitor`/`retry`, erros são
rebaixados a aviso — provocar falha ali é legítimo.

Calibragem atual: **0 erros** em 222 arquivos conhecidamente bons.

### O formatador precisa ser idempotente

`format_source(format_source(x)) == format_source(x)`. Há teste para isso. A
profundidade vem do INDENT/DEDENT do lexer, nunca da contagem de espaços do
original.

---

## O gerenciador de pacotes

`dataforge add` resolve, baixa e instala; `adopt` encontra o resultado. As
três peças:

| Onde | O quê |
|------|-------|
| `dataforge/packages.py` | `Versao`, `Requisito` (semver `^` `~` `>=`), `Registro`, `Dependencia`, `Lock`, `resolver()`, `instalar_pacote()`, `empacotar()` |
| `dataforge/cli.py` | `add_command`, `remove_command`, `install_command`, `list_command`, `search_command`, `pack_command`, `publish_command` |
| `dataforge/interpreter.py` | `_procurar_em_pacotes()` — o `adopt` olha `forge_modules/`, subindo até achar um `forge.toml` |

Layout de um projeto com dependências:

```
forge.toml          o que você pediu      (versionado)
forge.lock          o que foi instalado   (versionado)
forge_modules/      os pacotes            (NÃO versionado)
~/.dataforge/cache/ tarballs, entre projetos
```

**O registro é estático**: uma pasta com `index.json` e `pacotes/*.tar.gz`,
servida por qualquer host. Não há servidor a manter. O do projeto vive em
`site/public/registry/` e vai ao ar junto com o site.

Três decisões que valem lembrar:

1. **Conflito de versão é erro, não aviso.** Se dois pacotes pedem faixas
   incompatíveis do mesmo terceiro, `resolver()` falha dizendo quem pediu o
   quê. Instalar duas cópias em versões diferentes gera bug irreproduzível.
2. **O tarball é reprodutível** — `mtime=0`, uid/gid zerados. Sem isso o
   sha256 mudaria a cada empacotamento e a verificação de integridade não
   significaria nada.
3. **A extração recusa `../` e links simbólicos.** Um pacote não pode
   escrever fora da sua pasta.

### Os pacotes deste repositório

`packages/` tem quatro bibliotecas escritas em DataForge, publicadas no
registro do site: `validador` (CPF/CNPJ/e-mail e esquema de formulário),
`tabela` (saída para terminal), `datas` (datas em pt-BR com feriados) e
`cofre` (configuração em camadas). Somam 46 testes.

Elas servem de referência para quem for escrever um pacote — e de prova de
que o gerenciador funciona ponta a ponta.

```bash
cd packages/validador && dataforge pack
dataforge publish --registry=../../site/public/registry
```

## Kiln — o framework web

`dataforge/stdlib/kiln.py` é o runtime; as onze palavras da linguagem
(`server`, `route`, `respond`, `render`, `redirect`, `middleware`, `after`,
`mount`, `assets`, `views`, `ignite`) atravessam os cinco lugares de sempre e estão em
`CONTEXTUAIS_KILN`, não em `KEYWORDS`.

**Elas são contextuais pelo mesmo motivo de `get`/`set`/`final`**: `route`,
`render` e `server` são nomes bons demais para tirar de quem escreve. O parser
as reconhece pelo texto, e só onde fazem sentido — `_em_server` e `_em_rota`
controlam isso. `server` só abre bloco quando o que vem depois confirma
(`_abre_server`): um nome seguido de `:`, de `on` ou de `at`.

Três decisões que valem lembrar:

1. **`server` monta, `ignite` sobe.** Sem essa separação, um teste que
   importasse o módulo subiria o servidor e nunca terminaria. É por isso que
   `projetos/loja-web` tem `app.df` e `main.df` separados.
2. **`respond` e `render` levantam `YieldSignal`.** Encerram a rota como
   `yield` encerra uma ação — e o `typechecker` marca isso devolvendo `True`,
   senão o analisador acusaria "código inalcançável" logo abaixo.
3. **`Kiln.test` executa a rota sem socket.** É o que torna teste de rota
   barato. Mas ele roda tudo na mesma thread: bug de concorrência (como o do
   SQLite) **só aparece subindo o servidor de verdade**.

O `parse_render` precisa de `_no_with`: sem essa guarda,
`render "x" with {…}` seria lido como a expressão `record with {…}` e o
template comeria os dados.

## Vitrine — dashboards e aplicações de dados

`dataforge/stdlib/vitrine/` é o segundo framework web, e ele resolve um
problema diferente do Kiln. **Nenhuma palavra reservada nova**: tudo é
chamada de ação num módulo da biblioteca.

| Arquivo | O quê |
|---|---|
| `nucleo.py` | `No` (componente), `Sessao`, `Contexto` (a pilha de montagem) |
| `componentes.py` | os 40 componentes; cada um põe um nó **e devolve um valor** |
| `layout.py` | `Area` — coluna, aba, cartão; os componentes são métodos dela |
| `graficos.py` | sete tipos, montados como dado |
| `render.py` | a árvore vira HTML; o SVG e os ~4 KB de cliente moram aqui |
| `estado.py` | `V.estado` (sessão), `V.geral` (processo), `V.cache` (TTL + LRU) |
| `runtime.py` | `Aplicacao` — sessões, ciclo do pedido, servidor sobre o Kiln |
| `teste.py` | a `Sonda`: clica, digita e pergunta, sem navegador |
| `extras.py` | validação de campo, tradução por sessão, componentes por nome |
| `tema.py` | claro, escuro e o vault de variáveis CSS |
| `api.py` | o dicionário que o `adopt` entrega |

Nove decisões que valem lembrar:

1. **O programa inteiro roda de novo a cada interação**, e o estado da
   sessão sobrevive. É o que dispensa callback e diffing — e o que
   torna `V.cache` obrigatório, não opcional.

2. **A área de layout é um objeto, não um bloco de contexto.** A
   linguagem não tem `with`, e inventar uma palavra reservada para o
   layout de um módulo seria caro demais. `colunas[0].metrica(…)` lê
   melhor, aninha sem indentação e pode ser passado adiante.

3. **`_Parar`, `_Navegar` e `_Reexecutar` derivam de `BaseException`**,
   como `halt` e `skip`. O interpretador embrulha toda `Exception` que
   sai de função Python num `RuntimeError_` — com `Exception`,
   `V.exigir_login()` virava a mensagem "exigir_login: _Parar" no meio
   da página em vez de parar coisa alguma.

4. **O cookie de sessão é uma STRING.** O Kiln guarda cookie como a
   linha `Set-Cookie` pronta, não como vault. Passar um dicionário faz
   o navegador descartar o cookie e cada pedido abrir sessão nova — o
   sintoma é um contador que nunca passa de 1 e um login que nunca
   "pega", sem nenhum erro. `Kiln.test` **não devolve cookies**, então
   esse bug só aparece com socket de verdade.

5. **O login reexecuta a página do começo.** Continuar de onde parou
   deixaria a tela vazia para quem escreveu `given V.autenticado(): …`
   — esse teste já passou com a resposta antiga.

6. **O cache indexa o depósito por identidade, não por nome.** Duas
   ações `carregar` em arquivos diferentes dividiriam o mesmo cache, e
   o `teto` da primeira venceria calado sobre o da segunda.

7. **Acessibilidade é como os componentes são desenhados**, não uma
   camada por cima: `<fieldset>`/`<legend>` nos grupos, `role="alert"`
   no erro de campo com `aria-describedby` ligando ao campo, setas nas
   abas com só a ativa no Tab, e a variação da métrica com a palavra
   ("aumento de") ao lado da seta marcada `aria-hidden`. A primária do
   tema claro é `#B28600` e não o amarelo da marca — amarelo sobre
   branco dá contraste 1,3:1 onde a WCAG pede 4,5:1.

8. **O que não casa com página nenhuma cai no tratador de 404 do Kiln**,
   e não numa rota curinga. Uma curinga é casada na ORDEM do registro,
   então ela engolia toda rota acrescentada depois do `V.montar()` — que
   é exatamente o que se faz para servir uma API ao lado do painel. E a
   página de "não achei" responde **404**, não 200 com "404" no corpo.

9. **Zero dependência também no navegador.** O gráfico é SVG escrito no
   servidor; o cliente são ~4 KB sem build e sem CDN. Uma biblioteca de
   CDN quebra qualquer app em rede fechada — que é onde painel de dados
   costuma rodar. Há teste proibindo `http://`, `https://` e `cdn` no
   CSS e no JS.

`site/app/docs/vitrine/referencia/page.tsx` é **gerado** por
`tools/gerar_ref_vitrine.py`, que recusa rodar se um símbolo do módulo
ficar de fora — a mesma trava da gramática do editor.

## Kiln — upload, SSE e WebSocket

`kiln_tempo_real.py` traz as três coisas que o framework não tinha.

**Upload.** O corpo era interpretado como JSON ou formulário simples; um
`<input type="file">` chegava como texto ilegível. Agora os campos vão
para `req["body"]` e os arquivos para `req["files"]` — separados, para
que um `cycle` sobre `body` não tope com bytes onde espera texto.
`salvar_upload` **recusa** nome com `/` ou `..`, tamanho acima do limite
e extensão fora da lista, e o nome final leva prefixo aleatório.

**SSE.** `Kiln.sse(gerador)` marca a resposta como fluxo; o handler
chama o gerador com um `Fluxo` em vez de serializar um corpo.
`fluxo.aberto` vira `no` quando o cliente fecha a aba — sem conferir
isso no laço, um painel fechado deixa uma thread empurrando dado para
sempre. `Kiln.stream` é o mesmo mecanismo sem o formato de evento.

**WebSocket.** O handshake é HTTP com `Upgrade`, e `_atender_ws` o faz
antes de qualquer leitura de corpo. Depois dele, `self.connection` é o
socket. Ping, pong, máscara, continuação e quadro de 64 bits são
tratados dentro do `Soquete`.

Três decisões:

1. **A rota usa o método `WS`**, que não existe em HTTP. Assim ela não é
   alcançável por um GET comum, e um `GET /ws` continua livre para
   servir a página que abre a conexão.
2. **`_ler_exato` insiste até completar.** `recv` devolve menos do que
   se pediu com frequência num quadro que atravessa pacotes, e tratar o
   retorno curto como o quadro inteiro corrompe a mensagem seguinte — o
   sintoma é uma conexão que funciona e de repente para.
3. **`Sala.transmitir` remove o soquete morto** em vez de levantar: um
   cliente que fechou a aba não pode derrubar a mensagem dos outros.

## Arcane.Database — o que um CRUD exige

63 símbolos. O que foi acrescentado, e o problema de cada um:

| Símbolo | Sem ele |
|---|---|
| `transacao` · `savepoint` | a venda é gravada e o estoque não baixa |
| `upsert` · `upsert_many` · `insert_or_ignore` | o catálogo que chega por CSV duplica |
| `increment` | duas vendas ao mesmo tempo perdem uma baixa |
| `paginate` | a tela não sabe desenhar a paginação |
| `aggregate` · `group_count` | todo relatório é SQL escrito à mão |
| `create_search` · `search` | `LIKE %termo%` varre a tabela inteira |
| `explain` · `indexes` · `stats` | ninguém descobre o índice que falta |
| `rollback_migration` | desfazer exige editar o banco à mão |

**Toda escrita confirmava sozinha**, e por isso `transacao` era inútil:
o primeiro `insert` de dentro commitava, e o `rollback` não tinha o que
desfazer. `_confirmar(db)` respeita a profundidade da transação.

**Nome de coluna vai cru para o SQL** — o SQLite não aceita nome por
parâmetro. `_identificador`, `_conferir_colunas` e `_ordem_segura`
recusam o que não parece um nome; a lista de agregações é fechada pelo
mesmo motivo. Isso importa porque o `order_by` de uma listagem chega de
fora (`?ordenar=nome`).

Dois bugs que a busca textual custou, ambos silenciosos:

1. `"livr*"` — o `*` **dentro** das aspas é literal; a sintaxe de
   prefixo do FTS5 é `"livr"*`. A busca devolvia lista vazia, calada.
2. `apelido MATCH ?` é recusado pelo SQLite, e o nome da tabela junto de
   um apelido devolvia vazio — também sem erro. A consulta virou
   subconsulta, que também aplica o `LIMIT` antes do `JOIN`.

## Crucible — instantâneo, banco isolado, instável

`snapshot` usa a infraestrutura que **já existia e não estava ligada a
nada** (`_caminho_snapshot`, `carregar_snapshots`, `gravar_snapshots`,
`REGISTRO.snapshots`). Um JSON por arquivo de teste, ao lado dele.
Na primeira vez grava e passa; `DF_ATUALIZAR_SNAPSHOT=1` aceita a
mudança. Atualizar por padrão seria pior que não ter instantâneo.

`Crucible.banco(db)` abre transação e a desfaz no fim do trial. O
desfazer entra em `alvo.limpezas`, e **não** em `depois_de_cada`: essa
lista roda a cada teste, e acrescentar a ela dentro de um `before` a
faria crescer um item por teste.

`flaky` devolve o número de tentativas: um teste que precisa de três
toda vez não é instável, está quebrado.

## O que é gerado — não edite à mão

| Arquivo | Gerador | Guardado por |
|---------|---------|--------------|
| `editor/vscode/syntaxes/dataforge.tmLanguage.json` | `tools/gerar_gramatica.py` | `tests/test_editor.py` |
| `site/app/docs/kiln/referencia/page.tsx` | `tools/gerar_ref_kiln.py` | — |
| `site/app/docs/vitrine/referencia/page.tsx` | `tools/gerar_ref_vitrine.py` | `tests/test_vitrine.py` |
| `site/app/docs/biblioteca/page.tsx` | `tools/gerar_pagina_biblioteca.py` | `tests/test_regressoes.py` |
| `doc/BIBLIOTECA_PADRAO.md` | `tools/gerar_doc_stdlib.py` | — |
| `site/lib/dados-gerados.json` | `site/scripts/gerar_dados.py` | — |
| o `const headings` de cada `site/app/docs/**/page.tsx` | `site/scripts/gerar_indices.py` | `tests/test_api_e_marca.py` |
| **67 páginas** de `site/app/docs/` | `site/scripts/gerar_conteudo.py`, de `site/scripts/conteudo/*.py` | o job `gerado` do CI |
| `site/public/dist/*.tar.gz` | `scripts/gerar_tarball.py` | `tests/test_regressoes.py` |
| `site/lib/marca.ts`, favicon, ícones | `tools/vetorizar_logo.py` | `tests/test_api_e_marca.py` |
| `site/public/api/*.json` | `scripts/gerar_api.py` | `tests/test_api_e_marca.py` |
| `dataforge/marca.py` (arte ASCII) | `tools/vetorizar_logo.py` | — |

**Sessenta e sete das páginas de `/docs` são geradas.** Elas trazem o aviso na
primeira linha, com o caminho do arquivo de conteúdo que as origina — editar o
`.tsx` funciona até alguém rodar o gerador, e aí a correção some sem nada
explicando. Foi assim que uma contagem de símbolos voltou a ficar errada depois
de corrigida.

O índice lateral dessas páginas sai da **mesma** `slugify` do
`gerar_indices.py`. Já foram duas implementações, e elas se sobrescreviam a cada
geração: o estado final dependia da ordem em que os dois geradores rodassem.

A gramática do editor tem **duas** travas: o gerador recusa rodar se uma
palavra de `KEYWORDS` não estiver em nenhum grupo de cor, e um teste falha se o
arquivo versionado divergir do que o gerador produz. A versão anterior era
escrita à mão e por isso não conhecia `record` nem `enum` — exatamente o
problema que isso resolve.

O mesmo vale para `site/lib/highlight.ts`: há teste comparando com `tokens.py`.

## Instaladores

| Arquivo | Para |
|---------|------|
| `scripts/instalar.sh` | macOS e Linux, POSIX sh (roda em dash e busybox) |
| `scripts/instalar.ps1` | Windows, PowerShell |
| `Dockerfile` | imagem multi-estágio, usuário sem privilégio |

Ambos criam uma venv em `~/.dataforge` — não tocam no Python do sistema e
não pedem sudo. A origem do download é o site (`/dist/dataforge-X.tar.gz`),
com o GitHub apenas como alternativa.

Ao mudar a versão, regenere o tarball que o site serve:

```bash
python3 scripts/gerar_tarball.py
```

## Testes

| Arquivo | Como roda | Cobre |
|---------|-----------|-------|
| `tests/test_dataforge4.py` | `pytest` | recursos 4.0: interpolação, ternário, records, enums, padrões, generators, stack traces, checker, stdlib nova, ferramentas |
| `tests/test_regressoes.py` | `pytest` | bugs já corrigidos + sincronia da doc |
| `tests/test_dataforge.py` | `pytest` **e** script | 69 verificações da suíte original |
| `tests/test_kiln.py` | `pytest` | o framework web: rotas, respostas, templates, segurança, a sintaxe da linguagem e as palavras que continuam livres |
| `tests/test_resolucao.py` | `pytest` | onde mora o módulo de um `adopt`; ciclo no `check`; os 20 pacotes rodam; a cópia não volta |
| `tests/test_cobertura.py` | `pytest` | o denominador e o numerador da cobertura; a linha vai para o arquivo certo |
| `tests/test_banco.py` | `pytest` | transação que desfaz, `upsert`, `increment` sob 4 threads, FTS5, `explain`, migração com `down`, e o nome de coluna recusado |
| `tests/test_kiln_tempo_real.py` | `pytest` | multipart, SSE e WebSocket — o protocolo falado à mão, para pegar erro de enquadramento |
| `tests/test_vitrine.py` | `pytest` | a Vitrine: árvore, interação, estado, cache, autenticação, gráficos, escape, HTTP — e um ciclo completo por socket |
| `tests/test_excel.py` | `pytest` | `.xlsx`: o arquivo gerado é um ZIP válido, os tipos sobrevivem à ida e volta, `describe(frame)` |
| `tests/test_editor.py` | `pytest` | a gramática do VS Code está em dia com `tokens.py`; os snippets são DataForge válido |
| `exercicios/run_all.py` | script | 227 exercícios em 31 módulos, cada um com `assert` |
| `projetos/*/tests/` | `dataforge test` | 61 testes nos 4 projetos completos |
| `examples/*.df` | manual | 44 programas maiores |

**Ao corrigir um bug, escreva primeiro o teste que falha.** Todos os bugs
corrigidos no 3.1 e no 4.0 têm teste correspondente.

`tests/test_dataforge.py` roda nas duas formas — o `sys.exit` só dispara sob
`__main__`. Não renomeie `verificar()` para `test()`: o pytest tentaria coletá-la.

---

## Estado conhecido e limitações

O que **funciona e está testado**: tudo do 3.1 mais gerenciador de pacotes
(`add`, `install`, `remove`, `list`, `search`, `pack`, `publish`, com semver,
lockfile e verificação de integridade), tipos verificados, análise
estática com sugestões, stack traces, interpolação, ternário, `??`, `?.`, `in`,
spread/rest, desestruturação, compreensões, records imutáveis com `with`, enums
com valores, pattern matching estrutural completo, generators preguiçosos
(inclusive infinitos), imports seletivos, `relay` real, detecção de ciclos,
`forge.toml`, e as seis ferramentas de linha de comando.

O que **ainda não existe** (não invente que existe):

- **Generics com restrição** — `<T>` existe e o analisador o aceita,
  mas não há `<T extends Comparable>`: o parâmetro de tipo documenta a
  relação entre entrada e saída, e não é verificado em execução.
- **Exaustividade além do enum** — o `match` avisa quando um membro de
  enum fica de fora, mas não confere sequências nem records.
- **Debugger** — não há breakpoint nem passo a passo. O LSP existe
  (`dataforge lsp`, em `lsp.py`) e cobre autocompletar, hover, definição,
  referências, esquema, assinatura, renomear, formatar e correção rápida —
  servido do mesmo `typechecker`. Depuração continua sendo `out` e stack trace.
- **Bytecode** — continua sendo interpretador de árvore. O que existe é
  **compilação para fechamentos** (`compilador.py`): a árvore é percorrida
  uma vez e vira funções Python, o que tira o despacho do caminho quente.
  Medido: 1,5× a 1,8× conforme a carga. O teto dessa técnica, e o de uma VM
  de bytecode escrita em Python, é ~6,5× — o resto exigiria sair do Python.
- **Sincronização automática** — `Arcane.Concurrent` tem mutex, semáforo,
  barreira, contador atômico e canal bloqueante, mas nada é aplicado sozinho.
  O Kiln atende um pedido por thread: o `Arcane.Database` serializa o acesso
  à conexão (sem isso, a primeira consulta de qualquer servidor estoura), mas
  estado em memória compartilhado entre rotas não é protegido.
- **WebSocket, HTTP/2 e streaming de resposta** — o Kiln não tem. Ele roda
  sobre o `http.server` do Python; em produção pública, ponha um nginx ou
  Caddy na frente. A Vitrine herda isso: o "tempo real" dela é
  `V.atualizar_a_cada(n)`, que é por pergunta e não por empurrão.
- **Sessão da Vitrine vive na memória do processo.** Com mais de um
  processo, dois pedidos da mesma pessoa caem em memórias diferentes.
  Um processo por aplicação, com proxy na frente, é a forma testada.
- **Literal decimal exato** — não há sufixo nem sintaxe: `19.99` no código é
  `Float`, com o arredondamento binário de sempre. Para exatidão use
  `Arcane.Decimal`, e prefira a forma com aspas (`Dec.de("19.99")`), que não
  passa por float nenhum. Misturar `Decimal` com `Float` numa conta é
  **recusado** de propósito.
- **Cálculo de fórmula em planilha** — o `Arcane.Excel` grava a fórmula e o
  Excel a resolve ao abrir. Também não lê o `.xls` binário antigo.
- **`receive` bloqueante** — devolve `void` na hora se a fila está vazia.
- **`parallel`** roda **cada instrução** numa thread, não um bloco por thread.
- **`frame`, `train`, `predict`** são **açúcar fino** sobre `Arcane.Analytics`
  e `Arcane.Cortex` — não reimplementam nada. `frame` devolve o
  `AnalyticsFrame` de verdade; `train "floresta" using {…}` chama o treinador
  do Cortex com o vault de argumentos nomeados; `predict` chama
  `Cortex.prever`, ou a sua ação. Quem precisa de controle chama o Cortex
  direto e vê todos os parâmetros.
- **`async/await` é concorrente de verdade, mas só para entrada e saída.**
  Chamar uma ação `async` começa o trabalho numa thread e devolve uma
  tarefa; `await` espera. Rede, disco, banco e `sleep` se sobrepõem de
  fato. Trabalho de CPU não: o GIL continua no caminho, e a resposta ali
  é `Arcane.Concurrent`, que usa processos.

### Chamada de cauda

`yield` em DataForge **devolve e encerra**, então `yield f(…)` não tem
nada depois dele: o quadro existe só para repassar o resultado. `cauda.py`
marca esses `yield`, e `_corpo_com_salto` reusa **um** quadro em vez de
empilhar mil.

É um caminho **separado** de `_corpo_da_acao`, e não uma mudança nele: a
esmagadora maioria das ações não tem recursão de cauda e não pode pagar
por um laço, um `try` a mais e um estado por thread que nunca vai usar.

Quatro recusas, todas por análise, antes de rodar:

| Recusa quando | Porque |
|---|---|
| há `defer` na ação | ele roda na saída do quadro, e o salto reusa o quadro |
| o `yield` está dentro de `monitor` | um `handle` acima precisa ver o que a chamada levanta |
| a recursão é indireta (`f`→`g`→`f`) | a análise olha uma ação por vez |
| **todo** `yield` da ação é cauda | a ação nunca devolve; virar laço mudo seria pior que o `StackOverflowError` |

A última é a menos óbvia e a mais importante: sem ela,
`action r(n): yield r(n + 1)` deixaria de dar erro e passaria a travar.

A marca é por **nome**, e a identidade é conferida na hora — `f := outra`
dentro do corpo, ou um método substituído na filha, fariam o salto reusar
o quadro errado.

### A ponte para o Python

`adopt Python.numpy as np` traz qualquer biblioteca do Python. Quatro
coisas que valem lembrar antes de mexer em `ponte.py`:

1. **`Python` é espaço de nomes reservado**, resolvido em
   `_resolver_modulo` **antes** da stdlib e dos arquivos vizinhos. Um
   `Python.df` no disco não pode sequestrar o import.

2. **A ponte não converte.** Um `ndarray` continua um `ndarray` — é o
   que faz `a * 2` ser a conta vetorizada do numpy em vez de um laço
   sobre um milhão de posições. Isso só funciona porque o interpretador
   trata objeto estranho por **protocolo**, e não por tipo: membro,
   método, índice, `len`, iteração, aritmética, texto e verdade já
   passavam assim. Se alguém um dia trocar protocolo por `isinstance`,
   a ponte quebra inteira — `test_o_objeto_do_python_funciona_por_protocolo`
   existe para denunciar.

3. **Número é a exceção do `typeof`.** `np.int64` não é subclasse de
   `int`, mas faz conta de inteiro, então `_type_of` responde `Integer`
   por `numbers.Integral`. Não há nada de numpy no interpretador —
   `Fraction` e `Decimal` entram pela mesma porta.

4. **A mensagem de ausência nomeia o Python exato.** O instalador cria
   uma venv em `~/.dataforge`, e `pip install` no terminal instala em
   outro. E no executável único não há `pip` nenhum: ali a mensagem
   aponta `pip install dataforge-lang`, e não um comando que nunca
   funcionaria.

O roadmap completo está em `doc/ANALISE_E_ROADMAP.md`.

---

## Ao responder sobre o projeto

- **Não afirme sem rodar.** O interpretador está aqui; execute o `.df` antes de
  dizer o que ele faz.
- **Não invente sintaxe.** Se não está em `doc/REFERENCIA.md` ou nos exercícios
  que rodam, provavelmente não existe. `=>` só em lambda; `end` não existe; `->`
  só em tipo de retorno; `when` só em guarda de padrão.
- **Prefira citar um exercício** a inventar um exemplo: os 180 são verificados a
  cada execução, e os dos módulos 11-20 têm `.md` explicativo ao lado.
- Ao criar exemplos novos, termine com `assert` verificando o resultado.

---

## Comandos úteis

```bash
dataforge run arquivo.df --time      # com tempo de execução
dataforge run arquivo.df --debug     # tokens + AST + traceback
dataforge check src/ --strict        # avisos como erros
dataforge test tests/ -v --fail-fast
dataforge fmt . --check
dataforge lint src/
dataforge doc src/ --out=doc/API.md
dataforge repl

python3 exercicios/run_all.py 14     # só o módulo 14
python3 tools/gerar_doc_stdlib.py    # regenera BIBLIOTECA_PADRAO.md
```

Depurar o lexer numa linha específica:

```bash
python3 -c "
import sys; sys.path.insert(0,'.')
from dataforge.lexer import tokenize
for t in tokenize('x := 7 ~/ 2'): print(t)"
```

Rodar o analisador sobre tudo:

```bash
python3 -c "
import sys, glob; sys.path.insert(0,'.')
from dataforge.lexer import tokenize
from dataforge.parser import parse
from dataforge.typechecker import check_program
for f in glob.glob('examples/*.df') + glob.glob('exercicios/*/*.df'):
    for d in check_program(parse(tokenize(open(f,encoding='utf-8').read()), f), f):
        if d.severity == 'error': print(d.format(f, color=False))"
```
