# Referência da linguagem DataForge

Documento normativo: gramática, palavras-chave, operadores, tipos e semântica.
Para aprender a linguagem, comece pelo [`TUTORIAL.md`](TUTORIAL.md).

Versão descrita: **DataForge 3.1**

---

## 1. Estrutura léxica

### 1.1 Codificação e linhas

Arquivos `.df` são UTF-8. O fim de linha encerra uma instrução; não existe `;`.

### 1.2 Comentários

```dataforge
// linha
# linha
/* bloco
   de várias linhas */
```

### 1.3 Indentação

- Blocos abrem com `:` e são delimitados por indentação.
- **Tabs são erro** (`SyncError`). Use espaços — 4 por nível, por convenção.
- Dentro de `(`, `[`, `{` a indentação é ignorada.
- Uma linha começando com `.` continua o encadeamento de métodos anterior.
- Uma linha começando com `>>` continua o pipeline anterior.

### 1.4 Identificadores

Começam com letra ou `_`, seguidos de letras, dígitos ou `_`. Sensíveis à caixa.

### 1.5 Literais

| Forma | Exemplos |
|-------|----------|
| Inteiro decimal | `42`, `1_000_000` |
| Inteiro hexadecimal | `0xFF`, `0Xff` |
| Inteiro octal | `0o755` |
| Inteiro binário | `0b1010` |
| Decimal | `3.14`, `0.5` |
| Texto | `"a"`, `'a'` |
| Texto multilinha | `"""..."""`, `'''...'''` |
| Texto interpolado | `$"Ola {nome}"` |
| Booleano | `yes`, `no` |
| Nulo | `void` |
| Lista | `[1, 2, 3]` |
| Dicionário | `{"k": v}` |

Escapes em texto: `\n`, `\t`, `\r`, `\\`, `\'`, `\"`, `\0`.

### 1.6 Palavras reservadas (81)

```
action     adopt      and        as         assert     async      await      bigger
bigger_eq  blueprint  cast       channel    cycle      default    defer      delete
distill    emit       ensure     enum       extends    forge      frame      from
given      guard      halt       handle     in         inspect    is         isnt
lambda     mark       match      monitor    morph      no         not        observe
or         orif       otherwise  out        parallel   perform    persist    point
predict    propagate  pulse      record     recover    relay      retry      root
self       shadow     sift       skip       smaller    smaller_eq spawn      static
steady     step       stream     thread     to         train      trait      trigger
typeof     using      validate   void       wait       when       with       yes
yield
```

`cluster`, `vault` e `range` **não** são reservadas: são funções embutidas e
podem ser usadas como nome.

**Novas no 4.0:** `record`, `enum`, `when`.

### 1.7 Palavras contextuais

`abstract`, `final`, `get`, `operator`, `private`, `protected`, `set` têm significado **apenas dentro do corpo de um blueprint**.
Em qualquer outro lugar seguem sendo identificadores comuns:

```dataforge
final := 10                    // uma variável chamada 'final'
action get(chave):             // uma ação chamada 'get'
    yield chave

blueprint Conta:
    private saldo: Float := 0.0    // aqui 'private' é modificador
    get extrato():                 // e 'get' abre uma propriedade
        yield self.saldo
```

Foi uma decisão deliberada: `get`, `set` e `final` são nomes bons demais
para tirar de quem escreve. O parser só os trata como palavra-chave quando
o que vem em seguida confirma a intenção — `get nome(` é propriedade,
`get := 1` é variável.

`after`, `assets`, `ignite`, `middleware`, `mount`, `redirect`, `render`,
`respond`, `route`, `server` e `views` seguem a mesma regra, mas dentro de um bloco
**`server`** — são as palavras do [Kiln](KILN.md), o framework web. Fora
dali continuam livres:

```dataforge
render := 42                   // uma variável chamada 'render'
action route(x):               // uma ação chamada 'route'
    yield x + 1

server api on 8080:            // aqui 'server' abre a aplicação
    route GET "/":             // e 'route' declara uma rota
        respond html "<h1>oi</h1>"
```

`server` só abre um bloco quando o que vem em seguida confirma: um nome
seguido de `:`, de `on` ou de `at`. `server := "10.0.0.1"` continua sendo
uma atribuição comum.

---

## 2. Operadores

### 2.1 Tabela completa

| Operador | Categoria | Significado |
|----------|-----------|-------------|
| `:=` | atribuição | atribui/declara |
| `+=` `-=` `*=` `/=` `%=` | atribuição | composta |
| `+` `-` `*` `/` `%` | aritmética | soma, subtração, produto, divisão, resto |
| `**` | aritmética | potência (associa à direita) |
| `~/` | aritmética | divisão inteira (**preferido**) |
| `??` | coalescência | valor alternativo quando o esquerdo é `void` |
| `?.` | acesso seguro | membro/método, ou `void` se o objeto for `void` |
| `...` | spread / rest | expande ou coleta em coleções e chamadas |
| `in` / `not in` | pertinência | o elemento está na coleção? |
| `//` | aritmética | divisão inteira (ambíguo — ver 2.3) |
| `is` / `==` | comparação | igual |
| `isnt` / `is not` / `!=` | comparação | diferente |
| `bigger` / `>` | comparação | maior |
| `smaller` / `<` | comparação | menor |
| `bigger_eq` / `>=` | comparação | maior ou igual |
| `smaller_eq` / `<=` | comparação | menor ou igual |
| `and` `or` `not` | lógica | conjunção, disjunção, negação |
| `>>` | pipeline | encadeia `sift` / `morph` / `distill` |
| `.` | acesso | membro |
| `[]` | acesso | índice ou fatia |
| `()` | chamada | invocação |
| `->` | tipo | tipo de retorno de uma ação |
| `=>` | lambda | corpo de lambda (alternativa a `:`) |
| `@` | metadado | nome do decorador após `mark` |

### 2.2 Precedência

Da mais alta para a mais baixa:

| Nível | Operadores | Associatividade |
|-------|-----------|-----------------|
| 1 | `()` `[]` `.` | esquerda |
| 2 | `**` | direita |
| 3 | `-` `+` `not` (unários) | direita |
| 4 | `*` `/` `%` `~/` `//` | esquerda |
| 5 | `+` `-` | esquerda |
| 6 | comparações | encadeável |
| 7 | `not` | direita |
| 8 | `and` | esquerda |
| 9 | `or` | esquerda |
| 10 | `??` | esquerda |
| 11 | `given … otherwise` (ternário) | direita |
| 12 | `>>` | esquerda |

Consequências:

```dataforge
out 2 ** 3 ** 2    # 512, e não 64
out -2 ** 2        # -4, e não 4
out 2 + 3 * 4      # 14
```

> Repare que os comentários acima usam `#`. Com `//`, a linha
> `2 ** 3 ** 2  // 512, não 64` seria lida como divisão inteira — exatamente a
> ambiguidade descrita a seguir.

### 2.3 A ambiguidade de `//`

`//` abre comentário **e** é divisão inteira. O lexer decide pelo contexto:

É divisão inteira quando **todas** valerem:
1. o token anterior pode terminar uma expressão (valor, `)`, `]`, `}`);
2. há no máximo 2 espaços antes do `//`;
3. o que vem depois começa como operando (dígito, `(`, `-`, `_` ou letra);
4. o restante da linha não é prosa (duas palavras seguidas sem operador = prosa).

```dataforge
x := 7 // 2                     // divisão → 3
x := 3  // marcar como caminho  // comentário
```

**Recomendação: use `~/`.** É inequívoco e não depende de heurística.

### 2.4 Comparações encadeadas

```dataforge
nota := 7.5
out 0 <= nota <= 10        // equivale a (0 <= nota) and (nota <= 10)
out 1 smaller 5 smaller 10
```

O termo do meio é avaliado uma única vez.

### 2.5 Formato na interpolação

Depois de `:` vem o formato, com a mini-linguagem do `format`:

```dataforge
out $"{3.14159:.2f}"      // 3.14
out $"[{"ab":<10}]"       // [ab        ]
out $"{1234567:,}"        // 1,234,567
```

O `:` que abre o corpo de um `lambda` ou de um `morph` **não** é
formato: ele é distinguido por não ter espaço depois e por o que vem
em seguida parecer formato.

### 2.6 `is not` e `not in`

`is not` e `not in` são **um operador cada**, escritos com dois tokens:

```dataforge
out 5 is not 3         // yes — o mesmo que  5 isnt 3
out 3 not in [1, 2]    // yes
```

Ler `is not` como `is` aplicado a `(not x)` daria `5 is no`, que é `no`
para qualquer número — compila, roda e responde errado sem avisar. Por
isso o par é reconhecido junto, como em Python.

Para negar de fato uma expressão depois de `is`, ponha parênteses:

```dataforge
out 5 is (not 3)       // no — 'not 3' é 'no', e 5 não é 'no'
```

---

## 3. Tipos

### 3.1 Tipos internos

| Nome DataForge | Representação | Literal |
|----------------|---------------|---------|
| `Integer` | inteiro de precisão arbitrária | `42` |
| `Float` | ponto flutuante 64 bits | `3.14` |
| `String` | texto imutável | `"a"` |
| `Boolean` | `yes` / `no` | `yes` |
| `Void` | ausência de valor | `void` |
| `Cluster` | lista mutável ordenada | `[1, 2]` |
| `Vault` | dicionário chave→valor | `{"k": 1}` |
| `Action` | ação (função) | `action f(): ...` |
| `Blueprint` | classe | `blueprint P: ...` |
| *nome do blueprint* | instância | `spawn P()` |

`typeof(x)` devolve esses nomes como texto.

### 3.2 Anotações de tipo

Opcionais e **verificadas em tempo de execução**:

```dataforge
idade: Integer := 30                       // variável
action f(n: Integer, s: String) -> Float:  // parâmetros e retorno
    yield n * 1.0
```

Nomes aceitos, com sinônimos:

| Canônico | Sinônimos aceitos |
|----------|-------------------|
| `Integer` | `integer`, `int` |
| `Float` | `float` |
| `Number` | `number` (aceita `Integer` ou `Float`) |
| `String` | `string`, `str`, `text` |
| `Boolean` | `boolean`, `bool` |
| `Cluster` | `cluster`, `list`, `array` |
| `Vault` | `vault`, `dict`, `map` |
| `Void` | `void`, `none` |
| `Action` | `action`, `function` |
| `Any` | `any` (desliga a checagem) |

Um nome não listado é tratado como nome de blueprint, e a checagem percorre a
cadeia de herança.

Regra de conveniência: **um `Integer` é aceito onde se espera `Float`**.

### 3.3 Conversão

Formas disponíveis (`valor` e `x` são espaços reservados):

```
cast valor as Integer        // Integer, Float, String, Boolean, Cluster
str(x)  int(x)  float(x)  bool(x)  cluster(x)  vault(x)
```

---

## 4. Declarações

### 4.1 Variável

```ebnf
identificador [":" tipo] ":=" expressão
```

`:=` cria no escopo atual se o nome não existir em nenhum escopo visível;
caso contrário, atualiza onde ele foi encontrado.

### 4.2 Constante

```ebnf
steady NOME := valor
```

Reatribuir ou apagar dispara `RuntimeError_`.

### 4.3 Shadow

```ebnf
shadow nome := valor
```

Cria uma variável **estritamente local**, mesmo que o nome exista fora.

### 4.4 Estático

```ebnf
static nome := valor
```

Dentro de um `blueprint`, cria um membro compartilhado por todas as instâncias.

---

## 5. Controle de fluxo

### 5.1 Condicional

```
given <condição> ":" bloco
{ orif <condição> ":" bloco }
[ otherwise ":" bloco ]
```

### 5.2 Seleção múltipla

```
match <expressão> ":"
    { point <valor> ":" bloco }
    [ default ":" bloco ]
```

Compara por igualdade; não há fall-through.

### 5.3 Laços

```
cycle <var> from <início> to <fim> [ step <passo> ] ":" bloco
cycle <var> in <coleção> ":" bloco
persist <condição> ":" bloco
perform ":" bloco persist <condição>
```

O `to` de `cycle from` é **inclusivo**. O `step` padrão é `1` e pode ser negativo.

### 5.4 Interrupção

| Palavra | Efeito |
|---------|--------|
| `halt` | sai do laço |
| `skip` | vai para a próxima iteração |

`halt` e `skip` atravessam blocos `monitor` sem serem capturados: são controle de
fluxo, não erros.

---

## 6. Ações

### 6.1 Declaração

```
[ mark "@" decorador [ "(" args ")" ] ]
[ async ] action nome "(" [ params ] ")" [ "->" tipo ] ":" bloco
```

Um parâmetro é `nome [":" tipo] [":=" padrão]`.

### 6.2 Retorno

`yield <expressão>` devolve o valor e encerra a ação. Sem `yield`, devolve `void`.

`yield` atravessa `monitor` sem ser capturado.

### 6.3 Aridade

Verificada na chamada. Argumentos faltando, sobrando ou com nome desconhecido
disparam `TypeError_` com a mensagem apontando o problema.

### 6.4 Lambdas

```
lambda [ params ] ( ":" | "=>" ) <expressão>
lambda "(" params-tipados ")" ( ":" | "=>" ) <expressão>
```

Anotações de tipo em lambda exigem parênteses (sem eles, `:` inicia o corpo).

```dataforge
lambda x: x * 2
lambda a, b => a + b
lambda: 42
lambda (n: Integer): n + 1
```

### 6.5 Decoradores

```ebnf
mark @envolver
action alvo(x):
    yield x
```

O decorador recebe a ação e devolve a substituta. Com argumentos
(`mark @cache(60)`), o decorador é chamado primeiro com os argumentos e o
resultado recebe a ação.

Vários `mark` empilham; o mais próximo da ação é aplicado primeiro.

### 6.6 defer

```ebnf
defer:
    bloco
```

Agenda o bloco para rodar na saída da ação, em ordem LIFO. Erros dentro de um
bloco `defer` são descartados.

### 6.7 Limite de recursão

O interpretador aborta em **1000 quadros** com `StackOverflowError_`, nomeando a
ação. Recursões legítimas de até ~900 níveis funcionam.

---

## 7. Blueprints

### 7.1 Declaração

```
blueprint Nome [ "(" params ")" ] [ extends Pai {"," Pai} ] [ with Trait {"," Trait} ] ":"
    corpo
```

### 7.2 Construção

Duas formas, que podem coexistir:

**Parâmetros no cabeçalho** — viram campos automaticamente:

```dataforge
blueprint Ponto(x, y):
    action mostrar():
        yield self.x
```

**Método `setup`** — para campos derivados:

```dataforge
blueprint Lista:
    action setup():
        self.itens := []
```

Se ambos existirem, os parâmetros são atribuídos primeiro e depois `setup` roda.
`initiate` é aceito como sinônimo de `setup`.

Instanciar: `spawn Nome(args)` ou `forge Nome(args)`. Chamar o blueprint
diretamente (`Nome(args)`) também instancia.

### 7.3 self e root

- `self` (ou `this`) é a instância atual.
- `root` resolve métodos do primeiro blueprint pai.

### 7.4 Traits

```ebnf
trait Nome:
    action assinatura()
```

Métodos do trait só são copiados para o blueprint se ele **não** definir o
próprio. Não há verificação de que o contrato foi cumprido — use
`has_method(x, "nome")` quando isso importar.

### 7.5 Métodos especiais

Os nomes curtos, herdados das primeiras versões:

| Método | Chamado por |
|--------|-------------|
| `setup` / `initiate` | `spawn` |
| `toString` | `out`, `str()` |
| `add` `sub` `mul` `div` `mod` `pow` `floordiv` | `+` `-` `*` `/` `%` `**` `~/` |

E os **95 métodos mágicos** no estilo Python, em 16 grupos.
Quando o blueprint declara os dois para a mesma operação, o `operator` vence.
A lista completa está em [Métodos mágicos](https://dataforge-lang.vercel.app/docs/oop/magicos);
os mais usados:

| Método | Chamado por |
|--------|-------------|
| `__init__` | roda no spawn; sinonimo de 'setup' |
| `__new__` | cria a instancia antes de '__init__' |
| `__del__` | roda quando o objeto e descartado |
| `__copy__` | copia rasa |
| `__deepcopy__` | copia profunda |
| `__clone__` | copia, no vocabulario do DataForge |
| `__str__` | o texto que 'out' imprime |
| `__repr__` | o texto para quem depura |
| `__format__` | formatacao com especificador |
| `__bytes__` | a representacao em bytes |
| `__doc__` | a documentacao do objeto |
| `__eq__` | a == b  (e 'a is b') |
| `__ne__` | a != b  (e 'a isnt b') |
| `__lt__` | a < b   (e 'a smaller b') |
| `__le__` | a <= b |
| `__gt__` | a > b   (e 'a bigger b') |
| `__ge__` | a >= b |
| `__cmp__` | -1, 0 ou 1; cobre os seis de uma vez |
| `__add__` | a + b |
| `__sub__` | a - b |
| `__mul__` | a * b |
| `__truediv__` | a / b |
| `__floordiv__` | a ~/ b |
| `__mod__` | a % b |
| `__pow__` | a ** b |
| `__divmod__` | quociente e resto de uma vez |
| `__matmul__` | a @ b — multiplicacao de matriz |
| `__len__` | len(obj) |
| `__getitem__` | obj[chave] |
| `__setitem__` | obj[chave] := valor |
| `__delitem__` | delete obj[chave] |
| `__contains__` | item in obj |
| `__iter__` | 'cycle x in obj' |
| `__next__` | o proximo item do percurso |
| `__reversed__` | percurso de tras para frente |
| `__missing__` | chave ausente, antes de dar erro |
| `__length_hint__` | tamanho aproximado, para alocar antes |
| `__bool__` | o que 'given obj:' decide |
| `__int__` | int(obj) |
| `__float__` | float(obj) |
| `__complex__` | complex(obj) |
| `__index__` | o objeto como indice de colecao |
| `__hash__` | a chave de vault que este objeto vira |
| `__call__` | obj(argumentos) |
| `__enter__` | 'with obj as x:' — o que 'x' recebe |
| `__exit__` | o fim do bloco, mesmo com erro |

Ver também `slots` (§7.6) e a MRO por linearização C3 (§7.7).

### 7.6 `slots`

`slots` declara **todos** os campos que a instância pode ter; qualquer outro é
recusado na escrita e na leitura.

```dataforge
blueprint Ponto:
    slots x, y
```

Os valores passam a ser guardados numa lista, e não num vault: **64% menos
memória por objeto**, medido. Herdar e acrescentar slots soma os campos — mas
se **qualquer** ancestral não declara `slots`, a restrição cai por terra, porque
ele aceita campo livre.

`slots` é contextual, não reservada: `slots := 3` fora de um blueprint continua
sendo uma variável.

### 7.7 Resolução de nomes e MRO

A busca em uma instância segue: campos → métodos do blueprint → estáticos →
ancestrais **na ordem da MRO**.

A MRO (*method resolution order*) é calculada por **linearização C3**, a mesma
do Python, e garante três coisas: a classe vem antes das mães; a ordem em que
as mães foram escritas é respeitada; e uma mãe só aparece depois de todas as
filhas dela. Quando não existe ordem que satisfaça as três, a hierarquia é
ambígua e o C3 **recusa**, em vez de escolher em silêncio.

`root` segue a MRO — vai ao **próximo** na lista a partir de onde a chamada
está, e não ao "primeiro pai". É o que faz uma cadeia de `root` percorrer cada
blueprint exatamente uma vez, mesmo em diamante.

---

## 8. Erros

### 8.1 Blocos

```
monitor ":" bloco
{ handle [ Tipo [ as nome ] | nome ] ":" bloco }
[ ensure ":" bloco ]
```

- **Sem `handle`**, o erro **não é capturado** — só o `ensure` roda antes de o
  erro subir.
- Com `handle Tipo as e`, só erros daquele tipo são capturados; os demais sobem.
- `Error`, `Exception` e `Any` capturam qualquer erro.
- **Vários `handle` são permitidos**, um por tipo. Vence o **primeiro que
  casar** — a ordem importa, como nos `point` de um `match`:

```dataforge
monitor:
    valor := vault[chave] / divisor
handle KeyError:
    valor := 0
handle DivisionByZeroError as e:
    out e.message
    valor := 0
handle Error as e:
    propagate
```

  Um `handle` que captura tudo torna inalcançável todo `handle` abaixo dele;
  `dataforge check` avisa.

### 8.2 Objeto de erro

O nome ligado pelo `handle` expõe:

| Campo | Conteúdo |
|-------|----------|
| `.type` | nome do tipo, ex. `"RuntimeError"` |
| `.message` | a mensagem |
| `.line`, `.column` | posição de origem |

Ele se comporta como texto ao ser concatenado ou comparado com uma `String`.

### 8.3 Hierarquia de erros

| Tipo | Quando ocorre |
|------|---------------|
| `SyncError` | indentação inconsistente ou tab |
| `LexError` | caractere inválido, texto não terminado |
| `ParseError` | sintaxe inválida |
| `RuntimeError_` | divisão por zero, `assert` falso, operação inválida |
| `TypeError_` | tipo incompatível, aridade errada |
| `NameError_` | nome não definido |
| `IndexError_` | índice ou chave fora de alcance |
| `ImportError_` | módulo não encontrado |
| `TriggerError` | lançado por `trigger`, `guard`, `validate` |
| `StackOverflowError_` | recursão além de 1000 quadros |

### 8.4 Lançar e propagar

| Forma | Efeito |
|-------|--------|
| `trigger <expr>` | lança `TriggerError` com a mensagem |
| `guard <cond>, <msg>` | lança se a condição for falsa |
| `guard <cond> otherwise: bloco` | roda o bloco e **sai da ação** |
| `validate <expr>, <msg>` | lança se o valor for falso |
| `validate <expr> otherwise: bloco` | roda o bloco e sai da ação |
| `propagate <expr>` | relança |
| `assert <cond>, <msg>` | lança `RuntimeError_` se falso |

### 8.5 retry

```
retry <n> ":" bloco [ ( handle | recover ) [nome] ":" bloco ]
```

Tenta o bloco até `n` vezes. Se todas falharem, o handler roda com o último erro.

---

## 9. Pipelines

```
<expressão> { ">>" <operação> }
```

| Operação | Forma inline | Forma por referência |
|----------|--------------|----------------------|
| `sift` | `sift p: condição` | `sift nome_da_ação` |
| `morph` | `morph p: expressão` | `morph nome_da_ação` |
| `distill` | `distill acc, v: expr [inicial]` | `distill nome inicial` |

```dataforge
dados := [50, 120, 300, 90]
out dados
    >> sift v: v bigger 100
    >> morph v: v * 1.1
    >> distill acc, v: acc + v 0
```

Sem valor inicial, `distill` usa o primeiro elemento como acumulador.

---

## 10. Módulos

```
adopt <Nome>[.<Sub>] [ as <alias> ]
relay <nome> {"," <nome>}
```

Resolução, nesta ordem:
1. módulo já carregado (cache);
2. biblioteca padrão (`Arcane.*` e os nomes curtos);
3. arquivo `<caminho>.df` ou `<caminho>/main.df` relativo ao diretório atual.

Não encontrando, dispara `ImportError_` listando os módulos disponíveis.

`relay` documenta o que o módulo exporta. Na implementação atual, todas as
variáveis de nível superior do arquivo ficam acessíveis pelo alias.

---

## 11. Concorrência

| Construção | Semântica |
|------------|-----------|
| `async action f():` | chamá-la **começa** o trabalho numa thread e devolve uma tarefa |
| `await <expr>` | espera a tarefa terminar e entrega o valor |
| `await [t1, t2, …]` | espera **todas**; elas já corriam desde a chamada |
| `thread: bloco` | roda o bloco em uma thread daemon |
| `parallel: bloco` | roda **cada instrução** do bloco em uma thread, com join de 30 s |
| `channel nome` | cria uma fila FIFO com trava |
| `nome.send(v)` / `nome.receive()` | escreve / lê (devolve `void` se vazio) |
| `wait <ms>` | dorme pelo número de milissegundos |
| `stream <expr>` | cria um stream a partir de uma coleção |
| `observe v in fonte: bloco` | itera a fonte; aceita `halt` e `skip` |
| `pulse <evento>[, <dado>]` | emite um evento |

> **Limitação:** não há sincronização automática de variáveis compartilhadas
> entre threads. `channel` é a via segura.

---

## 11.5 Recursos do DataForge 4.0

### 11.5.1 Interpolação de strings

```ebnf
$"texto {expressao} texto"
$"""multilinha com {valor}"""
```

Prefixo `$` liga a interpolação; `{{` e `}}` escapam chaves literais. Qualquer
expressão cabe dentro das chaves, e a conversão usa as mesmas regras de `out`.

### 11.5.2 Expressão condicional (ternário)

```
<valor> "given" <condição> "otherwise" <alternativa>
```

```ebnf
rotulo := "par" given n % 2 is 0 otherwise "impar"
```

Associa à direita, então encadeia:

```ebnf
x := "a" given c1 otherwise "b" given c2 otherwise "c"
```

Não se aplica em contexto de padrão — lá as guardas usam `when`.

### 11.5.3 Coalescência e acesso seguro

```ebnf
a ?? b            # b apenas se a for void
obj?.membro       # void se obj for void
obj?.metodo()     # idem
```

Só `void` dispara a alternativa. `no`, `0` e `""` são valores, e passam.

### 11.5.4 Pertinência

```ebnf
x in colecao
x not in colecao
```

Funciona em `Cluster` (elemento), `Vault` (chave), `String` (subtexto), `Record`
(nome de campo) e `Stream` (consome).

### 11.5.5 Spread e rest

| Posição | Significa | Exemplo |
|---------|-----------|---------|
| esquerda do `:=` | coleta | `a, ...resto := lista` |
| dentro de `[…]` | expande | `[...a, ...b]` |
| dentro de `{…}` | expande (último vence) | `{...padrao, ...usuario}` |
| numa chamada | expande argumentos | `f(...args)` |

Só um `...rest` por desestruturação.

### 11.5.6 Desestruturação

```
<nome> {"," <nome>} ":=" <expressão>          // por posição
"{" <nome> {"," <nome>} "}" ":=" <expressão>  // por nome
```

```ebnf
a, b := [1, 2]
a, ...resto := lista
a, b := b, a                       # troca
{nome, idade} := registro          # record, vault ou instância
```

Quantidade incompatível dispara `RuntimeError_`; chave ausente, `NameError_`.

### 11.5.7 Compreensões

```
"[" <expr> {"cycle" <nome> "in" <fonte> ["given" <cond>]} "]"
"{" <chave> ":" <valor> {"cycle" …} "}"
```

```ebnf
[n * 2 cycle n in nums given n bigger 0]
{k: v cycle k in chaves}
[a + b cycle a in xs cycle b in ys]     # o da direita gira mais rápido
```

A variável do `cycle` não vaza para fora da compreensão.

### 11.5.8 Records

```
"record" Nome ":"
    <campo> ":" <Tipo> [":=" <padrão>]
    ["action" …]
```

Características:

- **imutável** — atribuir a um campo é erro
- **igualdade estrutural** — mesmos valores, mesmo record
- construção posicional ou nomeada, com tipos verificados
- `registro with {"campo": valor}` produz uma cópia alterada
- `toString` é usado por `out` e `str()`
- `typeof` devolve o nome do record

### 11.5.9 Enums

```
"enum" Nome ":"
    <MEMBRO> [":=" <valor>]
    ["action" …]
```

Cada membro expõe `.name`, `.value` (padrão: o nome) e `.index`.

O enum expõe `names()`, `values()`, `members()`, `count()`, `has(n)`,
`from_name(n)` e `from_value(v)` — as duas últimas devolvem `void` se não achar.

### 11.5.10 Pattern matching

```
"point" <padrão> ["when" <guarda>] ":" bloco
```

| Padrão | Casa com | Exemplo |
|--------|----------|---------|
| literal | igualdade | `point 0` |
| captura (minúscula) | qualquer coisa, liga o nome | `point n` |
| tipo (Maiúscula) | o tipo | `point Integer` |
| curinga | qualquer coisa | `point _` |
| valor nomeado | igualdade | `point Status.Ativo` |
| sequência | lista, por comprimento | `point [a, b]`, `point [x, ...r]` |
| mapa | vault/record, **parcial** | `point {"k": v}` |
| record posicional | tipo + campos na ordem | `point Ponto(x, y)` |
| record nomeado | tipo + campos por nome | `point Ponto(y := 0)` |
| alternativa | qualquer uma | `point 1 or 2` |
| apelido | liga o valor inteiro | `point [a, b] as tudo` |

Regras:

- testados **de cima para baixo**; o primeiro que casa vence
- `when` falso faz o `match` **continuar** para o próximo `point`
- padrão de sequência não casa com `Vault`, e vice-versa
- padrão de mapa é parcial: chaves extras não impedem o casamento

### 11.5.11 Generators

```
"stream" "action" nome "(" params ")" ":" bloco
"emit" <expressão>
```

Um `stream action` devolve um `Stream` **preguiçoso**: nada executa até alguém
pedir um item. Por isso um `persist yes:` com `emit` dentro é legítimo.

| Método | Devolve |
|--------|---------|
| `to_cluster()` | tudo, como lista |
| `take(n)` | os `n` primeiros |
| `next()` | o próximo, ou `void` |
| `count()` | quantos itens |
| `first()` | o primeiro, ou `void` |
| `map(f)` / `filter(f)` | lista |
| `reset()` | reinicia o `next()` |

`yield` dentro de um `stream action` encerra a produção. Fora de um
`stream action`, `emit` é um alias histórico de `out`.

### 11.5.12 Módulos

```
"adopt" <Caminho> ["as" <alias>]
"adopt" <Caminho> "." "{" <nome> ["as" <apelido>] {"," …} "}"
"adopt" "{" <nome> ["as" <apelido>] {"," …} "}" "from" <Caminho>
"relay" <nome> {"," <nome>}
```

Resolução, nesta ordem: cache → biblioteca padrão → arquivo `.df` **relativo ao
arquivo que importa**.

`relay` controla o que sai: sem nenhum, o módulo exporta tudo do nível superior;
com pelo menos um, só os nomes listados. Importes circulares são detectados e
disparam `ImportError_`.

---

## 12. Gramática (EBNF)

```ebnf
programa       = { instrução } ;

instrução      = decl_var | decl_destr | decl_steady | decl_shadow | decl_static
               | decl_ação | decl_blueprint | decl_trait
               | decl_record | decl_enum
               | adopt | relay
               | condicional | seleção_match | laço
               | bloco_erro | concorrência
               | out | emit | yield | halt | skip | trigger | assert
               | delete | wait | inspect | expressão ;

decl_var       = identificador [ ":" tipo ] ":=" expressão
               | alvo ( "+=" | "-=" | "*=" | "/=" | "%=" ) expressão ;
decl_destr     = alvos_pos ":=" expressão { "," expressão }
               | "{" alvos_nom "}" ":=" expressão ;
alvos_pos      = alvo_destr { "," alvo_destr } ;
alvos_nom      = alvo_destr { "," alvo_destr } ;
alvo_destr     = [ "..." ] identificador ;

decl_record    = "record" identificador ":" NEWLINE INDENT
                 { campo_record | decl_ação } DEDENT ;
campo_record   = identificador ":" tipo [ ":=" expressão ] NEWLINE ;
decl_enum      = "enum" identificador ":" NEWLINE INDENT
                 { membro_enum | decl_ação } DEDENT ;
membro_enum    = identificador [ ":=" expressão ] NEWLINE ;
decl_steady    = "steady" identificador ":=" expressão ;
decl_shadow    = "shadow" identificador ":=" expressão ;
decl_static    = "static" identificador ":=" expressão ;

decl_ação      = { "mark" "@" identificador [ "(" args ")" ] }
                 [ "async" | "stream" ] "action" identificador
                 "(" [ params ] ")" [ "->" tipo ] ":" bloco ;
params         = param { "," param } ;
param          = identificador [ ":" tipo ] [ ":=" expressão ] ;

decl_blueprint = "blueprint" identificador [ "(" nomes ")" ]
                 [ "extends" nomes ] [ "with" nomes ] ":" bloco ;
decl_trait     = "trait" identificador ":" bloco ;

adopt          = "adopt" caminho [ "as" identificador ]
               | "adopt" caminho "." seleção
               | "adopt" seleção "from" caminho ;
seleção        = "{" sel_item { "," sel_item } "}" ;
sel_item       = identificador [ "as" identificador ] ;
relay          = "relay" nomes ;
emit           = "emit" expressão { "," expressão } ;

condicional    = "given" expressão ":" bloco
                 { "orif" expressão ":" bloco }
                 [ "otherwise" ":" bloco ] ;
seleção_match  = "match" expressão ":" NEWLINE INDENT
                 { "point" padrão [ "when" expressão ] ":" bloco }
                 [ "default" ":" bloco ] DEDENT ;

padrão         = padrão_alt [ "as" identificador ] ;
padrão_alt     = padrão_base { "or" padrão_base } ;
padrão_base    = literal
               | "_"
               | identificador                          (* minúscula: captura *)
               | Identificador                          (* Maiúscula: tipo *)
               | Identificador "(" [ sub_padrões ] ")"  (* record por posição/nome *)
               | Identificador { "." nome }             (* valor nomeado *)
               | "[" [ padrão_seq ] "]"
               | "{" [ padrão_mapa ] "}" ;
sub_padrões    = ( padrão | identificador ":=" padrão )
                 { "," ( padrão | identificador ":=" padrão ) } ;
padrão_seq     = ( padrão | "..." [ identificador ] )
                 { "," ( padrão | "..." [ identificador ] ) } ;
padrão_mapa    = ( primário ":" padrão | "..." [ identificador ] )
                 { "," ( primário ":" padrão | "..." [ identificador ] ) } ;

laço           = "cycle" identificador "from" expressão "to" expressão
                     [ "step" expressão ] ":" bloco
               | "cycle" identificador "in" expressão ":" bloco
               | "persist" expressão ":" bloco
               | "perform" ":" bloco "persist" expressão ;

bloco_erro     = "monitor" ":" bloco
                 { "handle" [ tipo [ "as" identificador ] | identificador ]
                   ":" bloco }
                 [ "ensure" ":" bloco ]
               | "retry" expressão ":" bloco
                 [ ( "handle" | "recover" ) [ identificador ] ":" bloco ]
               | "guard" expressão ( [ "," expressão ] | "otherwise" ":" bloco )
               | "validate" expressão ( [ "," expressão ] | "otherwise" ":" bloco )
               | "propagate" [ expressão ]
               | "defer" ":" bloco ;

concorrência   = "thread" ":" bloco
               | "parallel" ":" bloco
               | "channel" identificador
               | "observe" identificador "in" expressão ":" bloco
               | "pulse" expressão [ "," expressão ] ;

expressão      = pipeline ;
pipeline       = ternário { ">>" op_pipeline } ;
ternário       = coalescência [ "given" coalescência "otherwise" ternário ] ;
coalescência   = ou { "??" ou } ;
op_pipeline    = "sift"    ( identificador ":" ou | identificador )
               | "morph"   ( identificador ":" ou | identificador )
               | "distill" ( identificador [ "," ] identificador ":" ou [ ou ]
                           | identificador ou ) ;
ou             = e { "or" e } ;
e              = negação { "and" negação } ;
negação        = [ "not" ] comparação ;
comparação     = adição ( [ "not" ] "in" adição
                        | { op_comp adição } ) ;
adição         = multiplicação { ( "+" | "-" ) multiplicação } ;
multiplicação  = unário { ( "*" | "/" | "%" | "~/" | "//" ) unário } ;
unário         = ( "+" | "-" | "not" ) unário | potência ;
potência       = posfixo [ "**" unário ] ;
posfixo        = primário { "." nome | "?." nome
                          | "[" índice "]" | "(" args ")"
                          | "with" dicionário } ;
índice         = expressão | [expressão] ":" [expressão] [ ":" [expressão] ] ;

primário       = literal | interpolada | identificador | "(" expressão ")"
               | lista | dicionário | lambda
               | compreensão_lista | compreensão_vault
               | "self" | "root"
               | ( "spawn" | "forge" ) posfixo
               | "typeof" unário
               | "cast" unário "as" identificador
               | "await" expressão
               | "stream" ou
               | "in" [ texto ]
               | "frame" primário
               | "train" posfixo "using" expressão
               | "predict" posfixo "using" expressão ;

lambda         = "lambda" [ params_lambda ] ( ":" | "=>" ) ou ;
interpolada    = "$" '"' { texto | "{" expressão "}" } '"' ;
lista          = "[" [ elemento { "," elemento } ] "]" ;
elemento       = expressão | "..." expressão ;
dicionário     = "{" [ par_ou_spread { "," par_ou_spread } ] "}" ;
par_ou_spread  = expressão ":" expressão | "..." expressão ;

compreensão_lista = "[" expressão cláusulas "]" ;
compreensão_vault = "{" expressão ":" expressão cláusulas "}" ;
cláusulas      = cláusula { cláusula } ;
cláusula       = "cycle" identificador { "," identificador }
                 "in" expressão [ "given" expressão ] ;

args           = ( expressão | "..." expressão
                 | identificador ":=" expressão )
                 { "," ( expressão | "..." expressão
                       | identificador ":=" expressão ) } ;

bloco          = NEWLINE INDENT { instrução } DEDENT ;
```

---

## 13. Funções embutidas

Disponíveis sem `adopt`. São 225 nomes, agrupados por tema:

**Tipos e conversão** — `len` `type` `str` `int` `float` `bool` `cluster` `vault`
`linhagem` `e_um`
`range` `cast`

**Texto** — `join` `split` `strip` `lstrip` `rstrip` `upper` `lower` `title`
`capitalize` `swapcase` `center` `ljust` `rjust` `zfill` `replace` `startswith`
`endswith` `find` `rfind` `index_of` `last_index_of` `char_at` `substring`
`isalpha` `isdigit` `isalnum` `isspace` `isupper` `islower` `istitle`
`isnumeric` `isascii` `repeat` `reverse_str` `trim` `pad_start` `pad_end`
`includes` `concat` `char` `ord` `encode` `decode` `format` `count_str`
`expandtabs` `partition` `rpartition` `splitlines` `removeprefix` `removesuffix`
`words` `lines` `template`

**Regex** — `regex_match` `regex_search` `regex_findall` `regex_sub`
`regex_split` `regex_test` `regex_count` `regex_extract`

**Matemática** — `abs` `min` `max` `sum` `round` `floor` `ceil` `sqrt` `cbrt`
`pow` `log` `log2` `log10` `exp` `sin` `cos` `tan` `asin` `acos` `atan` `atan2`
`degrees` `radians` `hypot` `factorial` `gcd` `lcm` `comb` `perm` `clamp` `lerp`
`sign` `is_nan` `is_inf` `is_finite`

**Estatística** — `mean` `median` `stdev` `variance` `mode` `percentile`
`random` `randint` `choice` `shuffle` `sample`

**Coleções** — `sorted` `reversed` `enumerate` `zip` `append` `pop` `insert`
`remove` `keys` `values` `items` `contains` `flatten` `unique` `count` `index`
`slice` `first` `last` `take` `drop` `chunk` `interleave` `rotate` `unzip`
`merge_dicts` `invert_dict` `pick` `omit` `deep_copy` `frequencies`

**Funcional** — `map` `filter` `reduce` `compose` `pipe_fn` `partial` `curry`
`memoize` `once` `tap` `identity` `constantly` `complement` `every` `some`
`none_of` `find_first` `find_last` `flat_map` `scan` `zip_with` `chain`
`product` `permutations` `combinations` `repeat_val` `accumulate`

**Sistema** — `time` `sleep` `timestamp` `exists` `freeze` `thaw` `hash` `id`
`uuid` `to_json` `from_json` `hash_md5` `hash_sha256` `base64_encode`
`base64_decode` `env_var`

**Predicados** — `is_empty` `is_string` `is_number` `is_integer` `is_float`
`is_boolean` `is_list` `is_dict` `is_void` `is_callable` `coalesce` `default`
`default_val` `assert_type` `validate` `instanceof`

**Introspecção** — `has_method` `has_field` `get_fields` `get_methods` `get_mro`
`get_parent` `class_name`

**Constantes globais** — `PI` `E` `TAU` `INF` `NAN` `MAX_INT` `MIN_INT`
`NEWLINE` `TAB` `EMPTY`

---

## 14. Semântica de valores

- **Inteiros** têm precisão arbitrária.
- **Textos** são imutáveis; métodos devolvem cópias.
- **Clusters e vaults** são mutáveis e passados por referência.
- `/` sempre devolve `Float`; `~/` devolve `Integer` para operandos inteiros.
- Divisão por zero (`/`, `~/`) dispara `RuntimeError_`.
- `+` com um operando texto converte o outro para texto.
- Índices negativos contam a partir do fim.
- Em contexto booleano: `void`, `no`, `0`, `""`, `[]` e `{}` são falsos.

---

## 15. Notas de implementação

- Interpretador de árvore (tree-walking), sem bytecode. O corpo das ações e
  do programa é compilado para fechamentos Python na primeira execução, o
  que tira o despacho do caminho quente (1,5× a 1,8× conforme a carga).
- Python 3.10+, sem dependências externas.
- Ordem: lexer (`lexer.py`) → parser recursivo descendente (`parser.py`) →
  interpretador (`interpreter.py`).
- Escopos formam uma cadeia (`environment.py`); a busca sobe até o global.
- Não há verificação estática antes da execução: `dataforge check` valida apenas
  a sintaxe.
