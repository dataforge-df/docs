# DataForge — análise técnica e o que falta

Auditoria completa do repositório, feita rodando o código, não lendo apenas.
Auditoria inicial em 2026-09-06 (3.0.0 → 3.1.0); segunda rodada em
2026-09-07 implementando o roadmap 4.x (3.1.0 → 4.0.0).

---

## 1. Veredito: é uma linguagem de programação?

**Sim, sem ressalva.** DataForge tem os cinco componentes que definem uma
implementação de linguagem, todos escritos do zero e todos funcionando:

| Componente | Arquivo | Linhas | Estado |
|------------|---------|--------|--------|
| Lexer com INDENT/DEDENT | `lexer.py` | 556 | completo |
| Parser recursivo descendente | `parser.py` | 1.941 | completo |
| AST tipada (dataclasses) | `ast_nodes.py` | 706 | completo |
| Interpretador de árvore | `interpreter.py` | 2.703 | completo |
| Analisador estático | `typechecker.py` | 1.193 | completo (4.0) |
| Cadeia de escopos | `environment.py` | 91 | completo |
| Funções embutidas | `builtins.py` | 1.224 | 225 funções |
| Biblioteca padrão | `stdlib/` | 7.282 | 20 módulos, 674 símbolos |
| Formatter | `formatter.py` | 280 | completo (4.0) |
| Linter | `linter.py` | 394 | completo (4.0) |
| Test runner | `testrunner.py` | 194 | completo (4.0) |
| Doc generator | `docgen.py` | 218 | completo (4.0) |
| Manifesto de projeto | `project.py` | 184 | completo (4.0) |
| REPL | `repl.py` | 409 | avançado (4.0) |
| CLI | `cli.py` | 1.055 | 13 comandos |

Não é um wrapper sobre `eval` do Python: cada construção tem seu nó de AST e sua
regra de avaliação. A prova prática está em `exercicios/`, onde um dos exercícios
é um **interpretador de notação polonesa reversa escrito em DataForge** — a
linguagem é expressiva o bastante para implementar outra.

### O que a torna avançada

- Sistema de tipos opcional com checagem em tempo de execução, incluindo
  parâmetros e retorno.
- Herança múltipla com resolução em profundidade, traits, sobrecarga de
  operadores e introspecção (`get_mro`, `has_method`, `get_fields`).
- Tratamento de erros com `handle` tipado, `retry`, `guard`, `validate`,
  `propagate`, `defer` (LIFO na saída do escopo) e `ensure`.
- Pipelines como sintaxe de primeira classe.
- Closures, ações de alta ordem, lambdas em três grafias, decoradores.
- Concorrência: `async/await`, threads, canais com trava, `parallel`, streams.
- Biblioteca padrão com servidor HTTP real (testado com requisições) e SQLite.

### O que ainda a separa de uma linguagem madura

Sem análise estática, sem gerenciador de pacotes, sem compilação para bytecode e
sem verificação de contrato de traits. Detalhes na seção 5.

---

## 2. Bugs encontrados e corrigidos

Todos foram reproduzidos com um `.df` mínimo, corrigidos e cobertos por teste de
regressão em `tests/test_regressoes.py`.

### Críticos — corrompiam a semântica

**1. `monitor/handle` capturava `yield`, `halt` e `skip`**

`HaltSignal`, `SkipSignal` e `YieldSignal` derivavam de `Exception`, e o
`except Exception` do `exec_MonitorBlock` os engolia. Consequência: um `yield`
dentro de um `monitor` nunca retornava — caía no `handle` como se fosse erro.

```dataforge
action f():
    monitor:
        yield 42          // devolvia void e rodava o handle
    handle e:
        out "erro"
```

**Correção:** os três passaram a derivar de `ControlSignal(BaseException)`.
Controle de fluxo deixou de ser confundível com erro.

**2. `monitor` sem `handle` engolia o erro silenciosamente**

Um `monitor:` seguido apenas de `ensure:` capturava tudo e seguia em frente. Erro
sumia sem rastro.

**Correção:** sem `handle`, o erro é relançado depois do `ensure`.

**3. `2 ** 3 ** 2` dava 64**

`parse_power` era não-associativo e ficava abaixo da multiplicação, então
`-2 ** 2` também dava `4` em vez de `-4`.

**Correção:** `**` associa à direita e liga mais forte que o unário.

**4. Aridade nunca era verificada**

`f(1)` numa ação de dois parâmetros ligava o segundo a `void`, e o erro só
aparecia depois, como `unsupported operand type(s) for +`. Argumentos a mais eram
descartados em silêncio.

**Correção:** `_check_arity` reporta o que falta, o que sobra e nomes
desconhecidos, no ponto da chamada.

**5. Recursão infinita derrubava o interpretador**

`RecursionError` do Python vazava como `Internal Error: maximum recursion depth
exceeded`, sem indicar a ação culpada.

**Correção:** contador de profundidade com limite de 1000 quadros e
`StackOverflowError_` nomeando a ação. O limite do Python foi elevado para 20.000
para que a checagem própria dispare antes.

**6. `adopt` de módulo inexistente criava um dicionário vazio**

Um erro de digitação em `adopt Arcane.Mathh` passava despercebido; o erro
aparecia páginas depois como `NameError`.

**Correção:** `ImportError_` listando os módulos disponíveis.

**7. `Arcane.Cortex` não carregava**

`cls._softmax` dentro de um `@staticmethod` — `cls` não existe ali. O módulo
inteiro quebrava ao ser importado.

**Correção:** referência pela classe. Todos os 30 nomes de módulo carregam.

**8. `setup` era ignorado quando havia parâmetros de construtor**

Um blueprint com `blueprint Pedido(cliente)` **e** um `action setup` nunca
executava o `setup`, então campos como `self.itens := []` nunca existiam.

**Correção:** os parâmetros são atribuídos e depois o `setup` roda.

**9. `str()` não respeitava `toString()`**

`out obj` chamava `toString`, mas `str(obj)` devolvia
`<Blueprint instance>`. As duas formas discordavam.

**Correção:** `str()` delega ao formatador do interpretador.

**10. `defer` não rodava quando a ação saía por erro**

O `_run_deferred` só era chamado no caminho de sucesso e no `yield`. Se a ação
falhasse, a limpeza agendada nunca acontecia — exatamente o cenário para o qual
`defer` existe. Um arquivo temporário ficava para trás no disco.

**Correção:** o `_run_deferred` passou para o `finally`, em `_call_action` e em
`_call_func`.

**11. Erros do Python vazavam como "Internal Error"**

`cycle x in 5`, `x[0]` sobre um inteiro e `observe` sobre um número produziam
mensagens Python cruas.

**Correção:** viram `TypeError_` com mensagem em vocabulário DataForge.

### Sérios — travavam uso legítimo

**12. `//` interpretado errado**

`lab[r][c] := 3  // marcar como caminho` era lido como divisão inteira, porque a
heurística só olhava o primeiro caractere depois de `//`. Um exemplo do próprio
repositório (`24_games_simulations.df`) falhava por isso.

**Correção:** heurística mais rigorosa (analisa se o resto da linha é prosa) e,
principalmente, o operador **`~/`**, inequívoco.

**13. Sete palavras reservadas sem uso**

`each`, `link`, `unlink`, `listen`, `claim`, `release` e `abstract` estavam em
`KEYWORDS` mas o parser nunca as consumia: só impediam o usuário de usá-las como
nome, sem entregar recurso nenhum.

**Correção:** removidas de `KEYWORDS` (os `TokenType` ficaram para uso futuro).

**14. `cluster`, `vault` e `range` eram funções impossíveis de chamar**

Estavam registradas em `builtins.py` **e** em `KEYWORDS`. `cluster([1,2])` era
erro de sintaxe.

**Correção:** removidas de `KEYWORDS`. As três funcionam.

**15. `pip install .` falhava**

`setup.py` apontava para `public/DataForge.png`, caminho inexistente. A
instalação abortava no build do wheel.

**Correção:** migração para `pyproject.toml`, sem `data_files` quebrado.

**16. `pytest` não coletava nada**

`tests/test_dataforge.py` chamava `sys.exit()` no nível do módulo; o pytest
abortava com `INTERNALERROR`. Além disso, a função auxiliar `test()` e a classe
`TestResults` eram coletadas como testes e falhavam por fixture ausente.

**Correção:** `sys.exit` sob `if __name__ == "__main__"`, auxiliares renomeadas
para `verificar()` e `Resultados`, e um `test_suite_legada()` expõe o resultado
ao pytest. Funciona nas duas formas.

**17. Todos os templates de `dataforge new` geravam código inválido**

Os seis templates usavam sintaxe que a linguagem não tem: `adopt X from "Y"`,
lambdas `=>` sem `lambda`, `end` para fechar blocos, `show()`, `isnot`,
`to_number()`, `push()`. Nenhum passava nem no `dataforge check`.

**Correção:** os seis foram reescritos em sintaxe real e verificados — os oito
arquivos `.df` gerados passam no `check`, e os executáveis rodam. O servidor
HTTP do template `api` foi testado com requisições reais (GET, POST, 404).

**18. `Text.render` só aceitava `{{chave}}`**

A documentação e o uso natural pediam `{chave}`.

**Correção:** aceita as duas formas.

**19. `Analytics.group_by` exigia uma função**

Agrupar por campo — o caso comum — obrigava a escrever uma lambda.

**Correção:** aceita também o nome do campo como texto.

**20. `Runtime_` — nome inexistente**

`_call_func` referenciava `Runtime_(...)`, que não existe. Qualquer valor não
chamável num pipeline geraria `NameError` do Python em vez de erro da linguagem.

**Correção:** `TypeError_` com mensagem adequada.

**21. `eval_BinaryOp` devolvia `None` em silêncio**

Um operador desconhecido caía fora do `if/elif` e a função devolvia `None`.

**Correção:** `RuntimeError_` explícito.

---

## 3. Recursos adicionados

Além das correções, a linguagem ganhou o que faltava para escrever código
idiomático sem contorcionismo:

| Recurso | Sintaxe | Por que |
|---------|---------|---------|
| Divisão inteira inequívoca | `a ~/ b` | remove a ambiguidade com comentário |
| Fatiamento | `l[1:3]`, `l[::2]`, `l[::-1]` | não existia; só índice simples |
| Comparação por símbolo | `<` `>` `<=` `>=` | antes só `smaller`/`bigger` |
| Comparação encadeada | `0 <= n <= 10` | evita repetir o termo do meio |
| Atribuição composta | `+=` `-=` `*=` `/=` `%=` | `x := x + 1` em todo lugar |
| Lambdas | `lambda x: e`, `lambda a, b => e` | não havia função anônima |
| Anotação de tipo em variável | `n: Integer := 5` | documentada, não implementada |
| Parâmetro e retorno tipados | `action f(n: Integer) -> Float:` | idem |
| Decoradores funcionais | `mark @nome` | o parser lia e descartava |
| Chamada encadeada | `f(1)(2)` | só era possível em identificador |
| `handle` tipado | `handle RuntimeError as e:` | documentado, não implementado |
| Objeto de erro | `e.type`, `e.message`, `e.line` | antes o erro virava só texto |
| Pipeline multilinha | linha iniciada por `>>` | quebrava a expressão |
| Unário `+` | `+5` | assimetria com `-5` |

---

## 4. Estado verificado

Comandos e resultados desta auditoria:

| Verificação | Comando | Resultado |
|-------------|---------|-----------|
| Testes unitários | `python3 -m pytest tests/ -q` | **240 passando** |
| Exercícios | `python3 exercicios/run_all.py` | **180/180** |
| Exemplos | `examples/*.df` | **42/42** |
| Blocos de documentação | executados um a um | **93/94** (o restante é lista de assinaturas) |
| Templates de projeto | `dataforge check` em cada `.df` gerado | **8/8** |
| Módulos da stdlib | import de todos | **20 módulos, 674 símbolos** |
| Instalação | `pip install .` em venv limpo | **funciona** |
| Servidor HTTP | GET, POST, GET por id, 404 | **funciona** |

### Os 120 exercícios

Cada um traz enunciado comentado e verifica o próprio resultado com `assert`.

| Módulo | Exercícios | Cobre |
|--------|-----------|-------|
| 01 fundamentos | 12 | tipos, operadores, precedência, conversão, anotações |
| 02 controle de fluxo | 12 | `given`, `match`, os quatro laços, `halt`/`skip`, `guard` |
| 03 coleções | 14 | clusters, fatiamento, vaults, matrizes, busca, ordenação |
| 04 strings | 10 | métodos, regex, templates, palíndromo, cifra de César |
| 05 ações | 14 | padrões, nomeados, tipos, aridade, recursão, closures, lambdas, decoradores, `defer` |
| 06 blueprints | 14 | construtores, herança, `root`, traits, polimorfismo, estáticos, operadores, padrões de projeto |
| 07 erros | 10 | `monitor`/`handle`/`ensure`, tipado, `guard`, `retry`, `propagate` |
| 08 pipelines | 12 | `sift`/`morph`/`distill`, composição, currying, streams |
| 09 módulos | 12 | `adopt` local e da stdlib, Math, Analytics, IO, JSON, SQLite, Text, Test |
| 10 avançado | 10 | async, threads, canais, `parallel`, lista ligada, árvore binária, máquina de estados, interpretador RPN |

---

## 5. O que faltava na 3.1 (histórico)

> Esta era a análise feita na versão 3.1. **A maior parte foi implementada no
> 4.0** — veja a seção 5.5 logo abaixo para o que saiu do papel, e a seção 6
> para o que ainda resta. Mantida aqui como registro do diagnóstico original.


Em ordem de impacto. As três primeiras são o que separa DataForge de uma
linguagem que dá para usar em produção.

### Prioridade alta

**1. Análise estática (`dataforge check` de verdade)**

Hoje o `check` só valida sintaxe. Um nome inexistente, uma aridade errada ou um
tipo incompatível só aparecem quando aquela linha executa. Falta um passe sobre a
AST que verifique, antes de rodar: nomes não definidos, aridade das chamadas a
ações conhecidas, tipos nas anotações declaradas, `yield` fora de ação, e código
inalcançável depois de `yield`.

Isso é o que mais custa ao usuário hoje: um erro de digitação num ramo raro do
programa só aparece em produção.

**2. Rastreamento de pilha nos erros**

O erro atual dá linha e coluna do ponto de falha, mas não o caminho de chamadas.
Numa recursão ou numa cadeia de ações, saber apenas "linha 47" não localiza a
origem. Falta empilhar os quadros e imprimir algo como:

```
TriggerError [linha 47]: saldo insuficiente
  em sacar()        linha 47
  em processar()    linha 88
  em main           linha 120
```

**3. Sistema de módulos com escopo real**

`relay` hoje é decorativo: todas as variáveis de topo de um arquivo importado
ficam visíveis, exportadas ou não. Falta respeitar o `relay`, permitir import
seletivo (`adopt Arcane.Math.{sqrt, floor}`), resolver caminhos relativos ao
arquivo importador (e não ao diretório de trabalho) e detectar import circular.

### Prioridade média

**4. Açúcares que já se sente falta**

Ao escrever os 120 exercícios, estes foram os que fizeram falta:

| Falta | Sintaxe sugerida |
|-------|------------------|
| Compreensão de lista | `[n * 2 cycle n in lista given n bigger 0]` |
| Desestruturação | `a, b := [1, 2]` e `nome, idade := pessoa` |
| `cycle` com índice | `cycle i, v in enumerate(lista):` |
| Operador de pertinência | `x in lista` como expressão |
| Interpolação | `f"total: {x}"` |
| Ternário | `a given cond otherwise b` |

**5. Verificação de contrato de traits**

Um blueprint pode declarar `with Serializavel` e não implementar `serializar`.
Ninguém reclama até alguém chamar. Deveria falhar na declaração.

**6. Casamento de padrão em `match`**

Hoje `match` compara por igualdade. Falta desestruturar
(`point [a, b]:`), casar por tipo (`point Integer:`) e ter guardas
(`point n given n bigger 10:`).

**7. Geradores**

`yield` retorna e encerra. Não há como produzir uma sequência preguiçosa —
o que é uma lacuna sentida numa linguagem que se apresenta como voltada a dados.
Precisaria de outra palavra (`produce`, por exemplo) para não colidir.

**8. Concorrência com garantias**

`thread` e `parallel` não sincronizam variáveis compartilhadas; só `channel` é
seguro. Falta `mutex`/`lock`, e `parallel` deveria tratar cada **bloco** como
uma tarefa, não cada instrução.

`await` sobre várias tarefas ao mesmo tempo **existe** desde que `async` deixou
de ser decoração: `await [f(x) cycle x in fonte]` espera todas, e elas já
corriam desde a chamada.

### Prioridade baixa

**9. Ferramental**

- Formatador (`dataforge fmt`) — indentação obrigatória sem formatador convida a
  divergência de estilo.
- *Language server* (LSP) para autocompletar, ir-para-definição e erros em tempo
  real. A gramática TextMate existe, mas só colore.
- Depurador com breakpoints — hoje só `inspect`.
- Cobertura de testes.

**10. Gerenciador de pacotes** — *feito no 4.0*

`dataforge add/remove/install/list/search/pack/publish`, com semver (`^`, `~`,
comparadores), lockfile com sha256, resolução de dependências transitivas e
quatro origens: registro, pasta local, git e URL. O registro é um índice
estático — `index.json` mais tarballs — servido junto com o site.

**11. Desempenho**

Interpretador de árvore **com compilação para fechamentos** (`compilador.py`):
a árvore é percorrida uma vez e vira funções Python, o que tira o despacho do
caminho quente. Medido, melhor de três:

| Carga | Antes | Depois | |
|---|---|---|---|
| `fib(24)`, recursão | 0,735 s | 0,459 s | 1,60× |
| laço de 300 mil | 0,270 s | 0,174 s | 1,55× |
| 200 mil chamadas de método | 0,962 s | 0,535 s | 1,80× |
| 200 mil `append` + `distill` | 1,032 s | 0,838 s | 1,23× |

O que **não** se resolve daqui: um `fib(25)` leva 1,28 s onde o CPython leva
0,03 s, e o CPython já é lento perto de uma linguagem compilada. Um protótipo
de VM de bytecode escrita em Python deu 7,9×, e um de fechamentos preservando a
semântica inteira deu 6,5× — mesma ordem de grandeza, nenhum dos dois fecha uma
diferença de 40×. Fechá-la exigiria sair do Python, e isso custaria a promessa
de zero dependências.

O que ainda cabe sem sair: compilar mais tipos de nó (a lista está ordenada por
frequência real em `compilador.py`), e encurtar a máquina de chamada —
`_check_arity` e `_run_deferred` rodam em toda chamada de ação.

**11-B. Recursão com teto de mil** — ~~limite~~ **resolvido para o caso direto**

`yield f(…)` onde `f` é a própria ação virou salto: um quadro em vez de
mil. Medido: um milhão de chamadas em ~4 s, onde mil já era
`StackOverflowError`.

Continua empilhando, e por escolha: recursão indireta (`f`→`g`→`f`),
ação com `defer`, `yield` dentro de `monitor`, e a ação que nunca
devolve — esta última de propósito, porque virar laço mudo seria pior
que a mensagem de erro.

**11-A. Interoperabilidade** — ~~não existe~~ **feita**

`adopt Python.numpy as np` traz qualquer biblioteca do Python, sem cópia
na fronteira. `Arcane.Ponte` responde se o pacote existe, explora o que
ele oferece e converte quando se pede.

A promessa de zero dependências continua inteira: nada em `dataforge/`
importa nada de fora. O que mudou é que o **programa de quem escreve**
passa a poder escolher as suas, e a palavra `Python` na linha do `adopt`
marca onde a fronteira foi cruzada.

**12. `frame`, `train`, `predict`** — ~~marcadores vazios~~ **ligadas**

As três devolviam um vault com `__type__` e nada acontecia. Eram o pior tipo de
lacuna: **pareciam** implementadas, e quem lia a gramática não tinha como saber.

E a tabela e os algoritmos já existiam um módulo ao lado. `Arcane.Analytics` tem
um Frame com 25 métodos; `Arcane.Cortex` tem 25 algoritmos de verdade. As
palavras não precisavam ser implementadas — precisavam ser **ligadas**.

```dataforge
t := frame registros                  // AnalyticsFrame, com select/filter/group_by

modelo := train "floresta" using {
    "linhas": treino, "alvo": "especie",
    "colunas": ["largura", "altura"], "arvores": 50
}
out predict modelo using teste
```

Elas continuam sendo açúcar fino, e isso é deliberado: quem precisa de controle
chama `Cortex.floresta(…)` direto e vê todos os parâmetros.

---

## 5.5 O que foi implementado no DataForge 4.0

Esta seção registra o que saiu do "falta" e virou realidade, seguindo o roadmap
de `DataForge_4_0_5_0_Roadmap_Melhorias.docx`.

### Fase 1 — Núcleo (roadmap §3)

| Item | Estado | Onde |
|------|--------|------|
| Type checker estático | **feito** | `typechecker.py`, `dataforge check` |
| Stack traces | **feito** | `errors.py`, com linha, coluna e cadeia |
| Records | **feito** | imutáveis, igualdade estrutural, `with` |
| Enums | **feito** | valores associados, integração com `match` |
| Desestruturação | **feito** | listas, records, vaults, `...resto` |
| Spread | **feito** | literais, vaults, chamadas |
| Compreensões | **feito** | de lista e de vault, múltiplas cláusulas |
| Null safety | **feito** | `??` e `?.` |
| Pattern matching avançado | **feito** | tipo, sequência, mapa, record, enum, guardas |
| Generics | **pendente** | `Cluster<T>` continua no roadmap |

### Fase 2 — Developer experience (roadmap §4)

| Item | Estado | Comando |
|------|--------|---------|
| Formatter | **feito** | `dataforge fmt [--check]` |
| Linter | **feito** | `dataforge lint [--strict]` |
| REPL avançado | **feito** | `:type`, `:ast`, `:check`, `:load`, `:save`, `:doc`, histórico |
| Test runner | **feito** | `dataforge test [-v] [--filter=] [--fail-fast]` |
| Doc generator | **feito** | `dataforge doc [--out=]` |
| LSP | **pendente** | |
| Debugger | **pendente** | |

### Fase 3 — Módulos (roadmap §5)

| Item | Estado |
|------|--------|
| `forge.toml` | **feito** — `dataforge init` / `info`, scripts nomeados |
| Escopo real entre módulos | **feito** — `relay` controla o que sai |
| Imports seletivos | **feito** — `adopt M.{a, b}` e `adopt {a as b} from M` |
| Caminhos relativos ao importador | **feito** |
| Detecção de ciclos | **feito** — `ImportError_` com a cadeia |
| Package manager | **pendente** |

### Fase 5 — Biblioteca padrão (roadmap §6)

Sete módulos novos, 220 símbolos, mantendo a política de zero dependências:

| Módulo | Símbolos | Conteúdo |
|--------|----------|----------|
| `Arcane.Time` | 54 | datas, durações, cronômetro, idade, fusos |
| `Arcane.OS` | 38 | sistema, ambiente, disco, terminal, processo |
| `Arcane.Crypto` | 38 | hashes, HMAC, PBKDF2, base64/32/hex, aleatoriedade |
| `Arcane.Collections` | 35 | pilha, fila, deque, heap, conjunto, união-busca, grafo |
| `Arcane.Serialization` | 26 | JSON, JSONL, CSV, INI, TOML, XML, flatten |
| `Arcane.Process` | 15 | execução, pipeline, spawn, timeout |
| `Arcane.Logging` | 14 | seis níveis, campos, arquivo, JSON |

Total da stdlib: **20 módulos, 674 símbolos**.

### Streams e generators (roadmap §7)

Implementado exatamente como o documento propõe: `emit` produz, `yield` retorna.
Um `stream action` devolve um `Stream` **verdadeiramente preguiçoso** — o corpo é
percorrido por um executor paralelo (`_lazy_block` em `interpreter.py`) que
entrega cada valor no instante em que `emit` o produz. Por isso um `persist yes:`
com `emit` dentro não trava: `take(n)` simplesmente para de pedir.

### Bugs encontrados e corrigidos no 4.0

O trabalho de escrever os 60 exercícios novos revelou cinco defeitos reais:

**22. `observe` recusava um `Stream`**
`exec_ObserveBlock` só aceitava lista e o vault `{"__type__": "Stream"}`, não o
`DFStream` que um `stream action` devolve. Consumir um generator com `observe`
era impossível.

**23. Transações do SQLite eram decorativas**
`DB.execute` fazia `commit()` incondicional. Com isso, `DB.rollback` nunca tinha
o que desfazer: `begin` / `insert` / `rollback` deixava a linha inserida.
Corrigido com um marcador `_in_transaction` respeitado por `execute` e
`execute_many`.

**24. `Collections.sort_by_field` ignorava records**
Lia campos com `item.get()` ou `getattr`, e um `DFRecordInstance` guarda os
valores em `.values`. O resultado era `None` para todos, e ordenar estourava com
`'<' not supported between instances of 'NoneType'`. Vale para `group_by` e
`index_by` também, em `Arcane.Collections` e `Arcane.Analytics`.

**25. Ordenação estourava com tipos misturados**
`sorted` com `void` ou tipos diferentes na mesma lista levantava `TypeError` do
Python. Agora há uma chave de ordenação que agrupa por categoria antes de
comparar.

**26. A heurística do `//` era imprevisível**
A regra anterior classificava `lab[r][c] := 3  // marcar como caminho` como
divisão. A regra do 4.0 é conservadora e explicável: `//` é **comentário**, salvo
quando seguido de dígito, `(`, ou um identificador que abre chamada, índice ou
membro. Para divisão inteira sem ambiguidade existe **`~/`**.

---

## 6. Roadmap sugerido

O que resta, em ordem de impacto.

### 4.1 — Confiança

- **Verificação de exaustividade** em `match` sobre enum: avisar quando um membro
  ficou de fora. É o item de melhor relação custo/benefício que sobrou.
- **Contrato de trait**: falhar na declaração quando o blueprint não implementa
  os métodos do trait, em vez de só na chamada.
- **Generics** — `Cluster<T>`, `Vault<K,V>`, ações genéricas.

### 4.2 — Ferramental

- **LSP**: autocomplete, ir-para-definição, renomear, hover com tipos. O
  `typechecker` já produz diagnósticos com linha e coluna — falta o servidor.
- **Debugger**: breakpoints, passo a passo, inspeção de variáveis.
- **Cobertura de testes** no `dataforge test`.

### 4.3 — Ecossistema

- ~~**Gerenciador de pacotes**~~ — **feito**. `dataforge add/remove/install`,
  lockfile com integridade, registro estático.
- ~~**Publicação**~~ — **feito**. `dataforge pack` e `dataforge publish`, com
  tarball reprodutível e versionamento semântico.
- **Registro hospedado com autenticação** — hoje publicar é abrir um PR no
  repositório do registro. Basta para começar; não escala para milhares de
  pacotes nem permite revogar uma versão comprometida.

### 5.0 — Runtime

- **IR e VM de bytecode**: hoje é interpretador de árvore com compilação
  para fechamentos (1,23× a 1,80×). Um protótipo de VM escrita em Python deu
  7,9× e um de fechamentos com a semântica inteira deu 6,5× — mesma ordem de
  grandeza. Medir de novo antes de investir.
- **Cache de compilação** — hoje os fechamentos são montados a cada processo.
- ~~**Empacotamento**: gerar um executável com runtime embutido.~~ **Feito**:
  `scripts/gerar_binario.py` e o fluxo `binarios` produzem um executável por
  sistema, e o instalador o usa quando não há Python na máquina.

### Concorrência (roadmap §8)

- `Mutex`, `Semaphore`, `Atomic` — hoje só `channel` é seguro.
- `receive` bloqueante.
- `TaskGroup` e cancelamento.
- `parallel` tratando **blocos** em vez de instruções.

### Web

- ~~**Framework de dashboards**~~ — **feito**. `Arcane.Vitrine`: um programa
  de cima para baixo vira uma página web, com 105 símbolos, gráficos em SVG
  escritos no servidor, estado por sessão, cache com TTL e LRU, autenticação,
  temas e uma sonda que testa sem navegador. Roda sobre o Kiln.
- **WebSocket no Kiln** — o `http.server` do Python não tem. Enquanto isso, o
  "tempo real" da Vitrine é `V.atualizar_a_cada(n)`, por pergunta e não por
  empurrão. É suficiente para painel, e insuficiente para chat.
- **Sessão compartilhada entre processos** — hoje a sessão da Vitrine vive na
  memória do processo, o que limita a aplicação a um processo com proxy na
  frente. Escalar horizontalmente exige um armazenamento comum primeiro.

### Decisão pendente

`frame`, `train` e `predict` estão **ligadas** ao `Arcane.Analytics` e ao `Arcane.Cortex`: elas não reimplementam nada, apenas nomeiam o que já existe.

---

## 7. Resumo em uma frase

DataForge é uma linguagem de programação real e agora razoavelmente madura: os
vinte e um defeitos do 3.1 e os cinco encontrados no 4.0 foram corrigidos e
cobertos por teste, e o núcleo ganhou tipos verificados, análise estática,
records, enums, pattern matching estrutural, generators preguiçosos, módulos com
escopo real e seis ferramentas de linha de comando; o que falta agora não é fundação nem ferramenta
básica, é ecossistema: generics, LSP e debugger. O gerenciador de pacotes
saiu no 4.0.
