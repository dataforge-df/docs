# Testes

Três ferramentas, e a ordem é de dentro para fora.

| Ferramenta | Para |
|---|---|
| `dataforge test` | descoberta simples: toda ação `test_` é um caso |
| `Arcane.Crucible` | suítes, matchers, dublês, propriedade, instantâneo |
| `--cobertura` | o que os testes **não** exercitaram |

---

## `dataforge test`

```bash
dataforge test                   # descobre *_test.df e tests/
dataforge test tests/ -v         # mostrando cada caso
dataforge test --filter=soma     # só os que casam
dataforge test --fail-fast       # para na primeira falha
dataforge test --cobertura       # com o relatório de cobertura
dataforge test --minimo=80       # reprova abaixo disso (saída 1)
```

Toda ação cujo nome começa com `test_` vira um caso. Um arquivo **sem**
ações `test_` é executado inteiro como um único caso, e os `assert` de
dentro valem como verificações.

Ganchos opcionais, chamados quando existem: `setup_all`, `teardown_all`,
`setup`, `teardown`.

```dataforge
adopt Arcane.Test as T
adopt ../src/conta as C

action test_deposito_soma():
    conta := spawn C.Conta("Ana")
    conta.depositar(100)
    T.assert_eq(conta.saldo, 100)

action test_saque_recusado():
    conta := spawn C.Conta("Ana")
    monitor:
        conta.sacar(999)
        T.fail("deveria ter recusado")
    handle TriggerError as e:
        T.assert_contains(e.message, "insuficiente")
```

`forge_modules/` fica **fora** da descoberta: os testes das suas
dependências não são os seus. Um projeto com 13 testes relatava 89, e a
suíte ficava vermelha por falha de uma biblioteca que ninguém escreveu.

---

## Crucible

```dataforge
adopt Crucible

crucible "a conta":
    Crucible.before(lambda suite: preparar())

    trial "deposito soma":
        Crucible.expect(depositar(100)).to_be(100)

    trial "saque acima do saldo e recusado":
        Crucible.expect(lambda => sacar(999)).to_raise("insuficiente")

Crucible.run()
```

```bash
dataforge crucible                 # roda as suítes
dataforge crucible -v              # cada trial
dataforge crucible --aleatorio     # embaralha a ordem
dataforge crucible --matchers      # lista o que se pode cobrar
dataforge crucible --formato=junit --saida=report.xml
dataforge crucible --cobertura --minimo=80
```

### O que o Crucible tem

| Grupo | Símbolos |
|---|---|
| suítes | `crucible`/`describe`/`suite`, `trial`/`test`, `only`, `pending`, `tag` |
| ganchos | `before`, `after`, `before_all`, `after_all`, `fixture` |
| expectativas | `expect` com ~40 matchers, `check`, `fail`, `approx`, `diff` |
| dublês | `mock`, `stub`, `spy`, `capture` |
| tabela | `table` — o mesmo trial com N casos |
| propriedade | `forall`, `integers`, `texts`, `floats`, `booleans`, `clusters`, `vaults`, `one_of` |
| tempo | `freeze_time`, `timed`, `benchmark` |
| arquivo | `temp_file`, `temp_dir` |
| instantâneo | `snapshot`, `snapshot_dir` |
| banco | `banco` — transação que se desfaz |
| instável | `flaky` |
| relatório | `report`, `summary`, `results`, `junit`, `tap`, `json` |

### Teste por propriedade

```dataforge
crucible "propriedades da lista":
    trial "inverter duas vezes devolve a original":
        Crucible.forall(Crucible.clusters(Crucible.integers(-100, 100)),
                        lambda xs: xs[::-1][::-1] is xs)
```

Em vez de escrever trinta casos à mão, descreve-se a regra que vale para
todos. Quando ela falha, o Crucible **encolhe** o contraexemplo até o
menor que ainda falha — um contraexemplo de quarenta elementos é
impossível de ler; o de dois diz qual é o bug.

---

## Instantâneo

```dataforge
crucible "o relatorio":
    trial "nao muda sem aviso":
        Crucible.snapshot("relatorio_mensal", gerar_relatorio())
```

Para o que é grande demais para escrever à mão: o HTML de uma página, o
relatório de trinta linhas, o JSON de uma rota. Escrever o esperado à
mão para isso dá um teste que ninguém mantém.

**Na primeira vez ele grava e passa.** É o único jeito de começar, e por
isso o arquivo (`__snapshots__/<teste>.snap.json`) vai no controle de
versão: é no diff do commit que alguém confere se o novo esperado está
certo.

```bash
DF_ATUALIZAR_SNAPSHOT=1 dataforge crucible     # aceita a mudança
```

Atualizar por padrão seria **pior que não ter instantâneo**: o teste
passaria sempre, gravando o errado por cima do certo.

Quando muda, a mensagem traz o **diff** — trezentas linhas lado a lado
num terminal são ilegíveis, e ter trezentas linhas é justamente o motivo
de usar instantâneo.

---

## Banco que se desfaz

```dataforge
crucible "cadastro":
    Crucible.before(lambda suite: Crucible.banco(db))

    trial "grava":
        Banco.insert(db, "livros", {"titulo": "Duna", "preco": 79.9})
        Crucible.expect(Banco.count(db, "livros")).to_be(1)

    trial "e o seguinte nao ve":
        Crucible.expect(Banco.count(db, "livros")).to_be(0)
```

Um teste que grava deixa a linha lá, e o seguinte a encontra. A suíte
passa **na ordem em que foi escrita** e falha em qualquer outra — e
`--aleatorio` expõe isso de um jeito que parece intermitente.

Apagar tudo entre testes seria a alternativa, e é mais lenta e mais
frágil: ela precisa saber a ordem das chaves estrangeiras.

---

## Teste instável

```dataforge
r := Crucible.flaky(lambda: Http.get(URL).json(), 3, 0.5)
Crucible.expect(r["ok"]).to_be(yes)
```

Para o que depende de rede, de relógio ou de escalonamento — e **não**
para esconder um bug. Ele devolve o número de tentativas: um teste que
precisa de três toda vez não é instável, **está quebrado**.

---

## Cobertura

```bash
dataforge test --cobertura --linhas
```

```
  src/main.df         ░░░░░░░░░░░░░░░░░░░░   0.0%  0/94
                      sem teste: montar_cli, mostrar, principal
                      linhas: 4-12, 18, 22-31
  src/repositorio.df  ████████████████████ 100.0%  38/38

  total  35.6%  52 de 146 linhas executáveis
```

`58% coberto` não diz o que fazer. **`sem teste: nunca_chamada`** diz.

### Os dois lados da fração

| Metade | De onde vem | Como mentiria |
|---|---|---|
| denominador | o parser: quais linhas são **executáveis** | contar comentário e linha vazia dá um número sempre pessimista |
| numerador | a execução, instrumentada | com a compilação de corpos ligada, toda ação daria 0% |

E duas escolhas que mudam o que se lê:

- **A linha do `action` não conta; o corpo conta.** Uma ação nunca
  chamada aparece com **0%**, e não com 20%.
- **Arquivo sem teste nenhum aparece com 0%**, em vez de sumir do
  relatório. Sumir é o que faz uma cobertura de 95% conviver com metade
  do sistema sem teste.

### O que ela não mede

É de **linha**, e não de ramo: `given a and b` conta como coberta mesmo
que `b` nunca tenha sido avaliado.

### Uma advertência

Cobertura alta não é qualidade. Um teste que chama tudo e não verifica
nada dá 100%:

```dataforge
action test_nao_verifica_nada():
    processar_pedido(pedido)      // coberto a 100%, zero garantido
```

O número serve para achar o que está a **zero**, e é aí que ele vale
quase tudo o que custa.

---

## Testar uma rota

```dataforge
r := Kiln.test(app, "GET", "/api/itens")
assert r["status"] is 200
assert len(r["body"]["itens"]) is 3
```

Executa a rota inteira, com middleware, **sem abrir socket**. É o que
torna teste de rota tão barato quanto teste de função.

Mas ele roda tudo na mesma thread e para antes do cabeçalho
`Set-Cookie`: bug de concorrência e de cookie só aparecem com
`Kiln.serve(app, 0)` e um cliente HTTP real. SSE e WebSocket também
precisam de socket.

---

## Testar uma página

```dataforge
t := V.testar(painel)
t.selecionar("Região", "Norte")
t.clicar("Exportar")
assert t.metrica("Receita") is "R$ 312.000"
assert not t.falhou()
```

A árvore de componentes da Vitrine é um **dado**, e conferir um dado é o
que um teste sabe fazer. Sem navegador.

---

## No CI

```yaml
- run: dataforge check . --strict
- run: dataforge test --minimo=80
- run: dataforge crucible --formato=junit --saida=report.xml
- run: dataforge lint src/
- run: dataforge fmt . --check
```

`dataforge check` acha o que não roda: nome errado, aridade errada,
campo inexistente, chamada entre arquivos que não existe, e ciclo de
import. Começar por ele economiza o tempo dos testes.

---

## Veja também

- [Instantâneos e isolamento](https://dataforge-lang.vercel.app/docs/tecnicas/instantaneos)
- [Cobertura](https://dataforge-lang.vercel.app/docs/tecnicas/cobertura)
- [Crucible](https://dataforge-lang.vercel.app/docs/tecnicas/testes)
