# Documentação do DataForge

**DataForge** é uma linguagem de programação interpretada, de propósito
geral, escrita em Python 3.10+ **sem dependências externas no runtime**.
Não é um DSL nem um transpilador: tem lexer, parser recursivo
descendente, AST tipada, analisador estático e interpretador próprios.

| | |
|---|---|
| Versão | 1.0.0 |
| Extensão | `.df` |
| Licença | MIT |
| Site | <https://dataforge-lang.vercel.app> |
| Código | <https://github.com/estevam5s/DataForge> |

```bash
curl -fsSL https://dataforge-lang.vercel.app/instalar.sh | sh
dataforge repl
```

> Os arquivos desta pasta são **espelhados** de `doc/` no
> [repositório principal](https://github.com/estevam5s/DataForge)
> por `scripts/sincronizar_docs_org.py`. Uma correção feita aqui
> é perdida na próxima sincronização — mande o PR para lá.

---

## Por onde começar

| Se você quer | Leia |
|---|---|
| instalar | [instalacao.md](instalacao.md) |
| aprender a linguagem | [tutorial.md](tutorial.md) |
| consultar a sintaxe | [referencia.md](referencia.md) |
| ver o que a biblioteca traz | [biblioteca.md](biblioteca.md) |
| fazer um site ou uma API | [kiln.md](kiln.md) |
| fazer um painel de dados | [vitrine.md](vitrine.md) |
| guardar dado | [banco-de-dados.md](banco-de-dados.md) |
| testar | [testes.md](testes.md) |
| entender como o interpretador funciona | [arquitetura.md](arquitetura.md) |
| saber o que falta | [roadmap.md](roadmap.md) |
| saber o que nao vai quebrar | [estabilidade.md](estabilidade.md) |

---

## A linguagem em trinta segundos

```dataforge
// atribuição, constante, tipo opcional, saída
x := 10
steady PI := 3.14159
idade: Integer := 30
out $"x vale {x}, o dobro é {x * 2}"

// condicional
given x bigger 5:
    out "grande"
orif x is 5:
    out "cinco"
otherwise:
    out "pequeno"

// laços
cycle i from 1 to 5:
    out i
cycle item in [1, 2, 3]:
    out item
persist x bigger 0:
    x -= 1

// ação com tipos
action somar(a: Integer, b: Integer) -> Integer:
    yield a + b

// record imutável, com igualdade estrutural
record Ponto:
    x: Integer
    y: Integer

p := Ponto(3, 4)
p2 := p with {"y": 0}

// blueprint mutável, com herança
blueprint Quadrado(lado) extends Forma:
    action area():
        yield self.lado ** 2

// pattern matching
match valor:
    point Integer as n when n bigger 100:
        yield "grande"
    point [a, b]:
        yield "par"
    point {"tipo": t}:
        yield "vault"
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
```

### Tabela de tradução

| Conceito | DataForge |
|---|---|
| `=` | `:=` |
| `const` | `steady` |
| `print` | `out` |
| f-string | `$"texto {expr}"` |
| `if/elif/else` | `given/orif/otherwise` |
| `switch`/`match` | `match` / `point` / `when` / `default` |
| `for` | `cycle … from … to` / `cycle … in` |
| `while` | `persist` |
| `break`/`continue` | `halt`/`skip` |
| `def`/`return` | `action`/`yield` |
| generator | `stream action` / `emit` |
| `class`/`new` | `blueprint`/`spawn` |
| `@dataclass(frozen)` | `record` |
| `interface` | `trait` |
| `self`/`super` | `self`/`root` |
| `import`/`export` | `adopt`/`relay` |
| `try/catch/finally` | `monitor/handle/ensure` |
| `throw` | `trigger` |
| `true/false/null` | `yes/no/void` |
| `filter/map/reduce` | `>> sift` / `>> morph` / `>> distill` |
| decorator | `mark @nome` |
| `//` (divisão inteira) | **`~/`** |
| chamar biblioteca Python | `adopt Python.numpy as np` |

---

## O que vem na caixa

**38 módulos, 1325 símbolos**, e nada a instalar.

| Área | Módulos |
|---|---|
| Núcleo | `Math`, `Text`, `IO`, `Regex`, `Collections`, `Functional`, `Iter`, `Decimal` |
| Frameworks | `Kiln` (web), `Vitrine` (painéis), `Crucible` (testes), `Forge` (banco), `API` |
| Dados | `Data`, `Analytics`, `Cortex` (ML), `Lago` (Parquet), `Pipeline`, `Qualidade`, `Stream` |
| Formatos | `Serialization`, `Excel`, `Archive`, `Database` |
| Sistema | `OS`, `Process`, `Time`, `Http`, `Web`, `Async`, `Concurrent`, `Ponte` |
| Qualidade | `Test`, `Logging`, `Crypto`, `Observar`, `Color`, `Meta` |

Mais **228 funções globais** sem nenhum `adopt`.

E, quando a biblioteca não alcança, `adopt Python.<pacote>` alcança
qualquer biblioteca do Python — sem conversão: um `ndarray` continua um
`ndarray`, e `a * 2` é a conta vetorizada do numpy.

---

## As ferramentas

```bash
dataforge run arquivo.df          # executa
dataforge check src/ --strict     # o que não vai rodar, antes de rodar
dataforge test --cobertura        # testes, com o que falta cobrir
dataforge crucible -v             # as suítes do Crucible
dataforge fmt .                   # formata
dataforge lint src/               # estilo e higiene
dataforge repl                    # console
dataforge new meuapp --modelo=api # 9 modelos, todos com testes
dataforge vitrine dev             # sobe um painel, recarregando ao salvar
dataforge debug arquivo.df        # breakpoint e passo a passo
dataforge lsp                     # servidor de linguagem (VS Code)
dataforge editor                  # instala a extensão
dataforge add validador           # gerenciador de pacotes
dataforge converter script.py     # Python → DataForge
```

São 43 comandos. `dataforge help` lista todos; `dataforge help <cmd>`
detalha um.

---

## Armadilhas

As que mais custam tempo a quem começa:

1. **`yield` retorna, `emit` produz.** Para uma sequência, `stream
   action` + `emit`.
2. **Divisão inteira é `~/`.** `//` é comentário, e só vira divisão
   quando seguido de dígito ou `(`.
3. **Só espaços na indentação.** Tab é erro; 4 espaços por nível.
4. **`monitor` sem `handle` não engole o erro.** Ele só garante o
   `ensure`.
5. **`self` dentro de métodos, sempre.** `x` em vez de `self.x` lê a
   variável de fora.
6. **Records são imutáveis.** `p.x := 1` é erro; use `p with {"x": 1}`.
7. **`trigger` levanta `TriggerError`**, não `RuntimeError`. Para pegar
   qualquer coisa, `handle Error`.
8. **O valor inicial do `distill` vem depois do corpo.**
   `>> distill a, v: a + v 0 / len(x)` divide o **zero**.
9. **`query["x"]` sem `??` dá 500 numa rota.** A query vem de fora.
10. **A linguagem não sincroniza sozinha.** `Arcane.Concurrent` tem
    mutex, semáforo e canal — mas usá-los é escolha de quem escreve.

A lista completa está em [referencia.md](referencia.md).

---

## O que ainda não existe

Dito para não ser descoberto no meio do trabalho:

- **Generics com restrição.** `<T>` existe e documenta a relação entre
  entrada e saída, mas não é verificado em execução.
- **Bytecode.** É interpretador de árvore com compilação para
  fechamentos (1,5× a 1,8× mais rápido que a versão anterior). Contra o
  CPython, a mediana é 58× mais lento; recursão pesada chega a 160×.
- **HTTP/2 e TLS no Kiln.** Ele roda sobre o `http.server`; em produção
  pública, ponha um nginx ou Caddy na frente. WebSocket e SSE **existem**.
- **Detecção de corrida.** Duas threads escrevendo na mesma variável
  perdem atualizações, e o analisador não avisa.
- **Sessão compartilhada entre processos** na Vitrine — um processo por
  aplicação.
- **Cobertura de ramo.** A que existe é de linha.

O [roadmap.md](roadmap.md) tem a lista inteira, com o que já foi feito.

---

## Licença

MIT.
