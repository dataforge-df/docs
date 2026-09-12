# Banco de dados

Dois módulos, e a escolha entre eles é o motor.

| Módulo | Para | Símbolos |
|---|---|---|
| `Arcane.Database` | SQLite — um arquivo, zero configuração | 63 |
| `Arcane.Forge` | PostgreSQL, MySQL, MariaDB, MongoDB, Redis, SQLite | 28 |

Nenhum dos dois tem dependência. O Forge implementa o protocolo
publicado de cada servidor, um arquivo por protocolo — não há
`psycopg2`, `PyMySQL`, `redis-py` nem `pymongo` no caminho.

---

## Começar

```dataforge
adopt Arcane.Database as Banco

db := Banco.connect("loja.db")      // ou Banco.memory(), para teste

Banco.create_table(db, "produtos", {
    "id": "INTEGER PRIMARY KEY AUTOINCREMENT",
    "sku": "TEXT NOT NULL UNIQUE",
    "nome": "TEXT NOT NULL",
    "preco": "REAL NOT NULL CHECK (preco >= 0)",
    "estoque": "INTEGER NOT NULL DEFAULT 0 CHECK (estoque >= 0)"
})

Banco.upsert(db, "produtos", {"sku": "CAF-500", "nome": "Café 500g",
                              "preco": 32.9, "estoque": 20}, "sku")
```

---

## Parâmetros, sempre

```dataforge
// certo
Banco.query(db, "SELECT * FROM produtos WHERE sku = ?", [sku])

// errado, e é assim que um banco é apagado
Banco.query(db, $"SELECT * FROM produtos WHERE sku = '{sku}'")
```

Nome de **coluna**, de tabela e de índice não pode ir por parâmetro — o
SQLite não aceita — e por isso vai concatenado. Para esses, o módulo
**recusa** o que não parece um nome:

```dataforge
Banco.aggregate(db, "vendas", {"n": ["count", "*"]},
                order_by := "valor; DROP TABLE vendas")
// erro: 'valor; DROP TABLE vendas' nao e um nome valido
```

Isso importa porque o `order_by` de uma listagem vem de fora
(`?ordenar=nome`).

---

## CRUD

| Chamada | Faz |
|---|---|
| `Banco.insert(db, tabela, vault)` | insere; devolve o `id` |
| `Banco.insert_many(db, tabela, cluster)` | insere muitos |
| `Banco.insert_or_ignore(db, tabela, vault)` | não reclama se a chave existe |
| `Banco.upsert(db, tabela, vault, chaves)` | insere **ou** atualiza |
| `Banco.upsert_many(db, tabela, cluster, chaves)` | o mesmo, numa transação só |
| `Banco.select(db, tabela, where, order_by, limit)` | lê |
| `Banco.query(db, sql, params)` | SQL livre |
| `Banco.query_one(db, sql, params)` | a primeira linha, ou `void` |
| `Banco.update(db, tabela, vault, where)` | altera |
| `Banco.increment(db, tabela, coluna, delta, where)` | `coluna = coluna + ?` |
| `Banco.delete(db, tabela, where)` | apaga |
| `Banco.count` · `exists` | conta, confere |
| `Banco.paginate(db, tabela, pagina, por_pagina, …)` | fatia com total e páginas |

### A condição

```
{"id": 7}                  →  id = 7
{"id": [1, 2, 3]}          →  id IN (1, 2, 3)
{"preco": {"gte": 10}}     →  preco >= 10
{"nome": {"like": "caf%"}} →  nome LIKE 'caf%'
{"nota": void}             →  nota IS NULL
```

Operadores: `eq`, `ne`, `gt`, `gte`, `lt`, `lte`, `like`.

Um cluster **vazio** não casa com nada — `IN ()` é erro de sintaxe no
SQLite, e a resposta certa para "nenhum dos valores" é não casar com
nada. E `void` vira `IS NULL`, porque `coluna = NULL` nunca é verdadeiro
em SQL.

---

## Transações

```dataforge
action vender(sku, quantos):
    action corpo():
        p := Banco.query_one(db, "SELECT * FROM produtos WHERE sku = ?", [sku])
        given p["estoque"] smaller quantos:
            trigger $"estoque insuficiente de {p["nome"]}"
        Banco.insert(db, "vendas", {"produto_id": p["id"],
                                    "quantidade": quantos,
                                    "valor": p["preco"] * quantos})
        Banco.increment(db, "produtos", "estoque", -quantos, {"id": p["id"]})
        yield p["preco"] * quantos

    yield Banco.transacao(db, corpo)
```

Erro **desfaz tudo**. É a peça que falta num PDV: gravar a venda, baixar
o estoque e lançar o pagamento são três escritas que precisam valer
juntas.

```
       grava a venda   ✓
       grava o item 1  ✓
       baixa estoque 1 ✓
       grava o item 2  ✗  ← sem estoque
       ────────────────────
       sem transação:   a venda existe, com um item, e o estoque do
                        primeiro item foi baixado
       com transação:   nada aconteceu
```

Ninguém descobre o primeiro caso até o inventário.

| Chamada | Faz |
|---|---|
| `Banco.transacao(db, acao)` | roda; erro desfaz; devolve o que ela devolveu |
| `Banco.savepoint(db, nome, acao)` | uma transação **dentro** de outra |
| `Banco.in_transaction(db)` | estamos dentro de uma agora? |
| `Banco.begin` · `commit` · `rollback` | à mão — prefira `transacao` |

Uma `transacao` dentro de outra vira savepoint sozinha. O savepoint
desfaz **só a parte dele**: um item sem estoque não precisa derrubar a
venda inteira.

### `increment`, e não ler-somar-escrever

```dataforge
// errado, e o erro é silencioso
p := Banco.query_one(db, "SELECT estoque FROM produtos WHERE id = ?", [7])
Banco.update(db, "produtos", {"estoque": p["estoque"] - 1}, {"id": 7})

// certo: a soma é do banco, sob a trava da linha
Banco.increment(db, "produtos", "estoque", -1, {"id": 7})
```

Dois caixas vendendo o mesmo item ao mesmo tempo leem 10, os dois
escrevem 9, e uma unidade desaparece **sem nenhum erro aparecer**. Foi
medido: quatro threads com 200 incrementos cada perdem cerca de um terço
na forma ingênua.

---

## Relatório

```dataforge
Banco.aggregate(db, "vendas", {
    "receita": ["sum", "valor"],
    "vendas":  ["count", "*"],
    "ticket":  ["avg", "valor"]
}, group_by := "vendedor", order_by := "receita DESC")
```

```
[{"vendedor": "ana",   "receita": 4820.0, "vendas": 30, "ticket": 160.6},
 {"vendedor": "bruno", "receita": 3910.0, "vendas": 27, "ticket": 144.8}]
```

As colunas do `group_by` saem junto — que é o que um gráfico precisa.
Aceita `count`, `sum`, `avg`, `min`, `max` e `total`; a lista é fechada
porque o nome vai cru para o SQL.

`Banco.group_count(db, tabela, coluna)` é o atalho que mais se pede.

---

## Busca textual

```sql
-- o que quase todo mundo escreve
SELECT * FROM produtos WHERE nome LIKE '%cafe%'
```

`LIKE` com `%` na frente **não usa índice nenhum**. O FTS5 usa índice
invertido e ordena por relevância:

```dataforge
Banco.create_search(db, "produtos", ["nome", "categoria"])
Banco.search(db, "produtos", "merce")     // acha "mercearia"
```

| Detalhe | Por quê |
|---|---|
| prefixo na última palavra | quem digita `livr` espera achar `livro` antes de terminar |
| o índice se mantém em dia, por gatilhos | sem eles ele envelhece em silêncio e a busca deixa de achar o que foi cadastrado depois |
| devolve a linha da tabela **original** | quem busca quer o produto, não o índice |
| o `LIMIT` é aplicado antes do `JOIN` | com um milhão de linhas, junta-se vinte |

---

## Diagnóstico

```dataforge
Banco.explain(db, "SELECT * FROM vendas WHERE vendedor = ?", ["ana"])
// {"varre_tabela": yes, "aviso": "le a tabela inteira: SCAN vendas"}

Banco.create_index(db, "vendas", ["vendedor"])
// depois: {"varre_tabela": no, "aviso": ""}

Banco.indexes(db, "vendas")   // com as colunas de cada um
Banco.stats(db)               // tabelas, linhas, índices, bytes
Banco.integrity(db)           // o 'integrity_check' do SQLite
```

A linha que importa é a que diz **SCAN** em vez de SEARCH: SCAN lê a
tabela inteira, e num cadastro de 200 mil linhas é a diferença entre
2 ms e 2 s.

### Onde pôr índice

| Situação | Índice |
|---|---|
| toda chave estrangeira | sempre — o SQLite **não** cria |
| a coluna do `WHERE` de uma listagem | sim |
| a coluna do `ORDER BY` de uma listagem grande | sim; evita a ordenação |
| as duas juntas, na mesma consulta | um composto, na ordem `WHERE` → `ORDER BY` |
| uma coluna com três valores possíveis | não — não separa nada |
| uma tabela de cem linhas | não — varrer é mais rápido |

Cada índice é uma árvore a atualizar em todo `INSERT`. Numa tabela de
log com dez índices, gravar fica mais lento que ler.

---

## Migrações

```dataforge
steady MIGRACOES := [
    {"version": 1, "description": "clientes",
     "up":   "CREATE TABLE clientes (id INTEGER PRIMARY KEY, nome TEXT);",
     "down": "DROP TABLE clientes;"},
    {"version": 2, "description": "e-mail",
     "up":   "ALTER TABLE clientes ADD COLUMN email TEXT;",
     "down": "ALTER TABLE clientes DROP COLUMN email;"}
]

Banco.migrate(db, MIGRACOES)                 // aplica o que falta
Banco.migrations_applied(db)                 // o que já foi
Banco.rollback_migration(db, MIGRACOES)      // desfaz UMA
Banco.rollback_migration(db, MIGRACOES, ate := 1)
Banco.schema_sql(db)                         // o schema como está
```

`migrate` é **idempotente**: grava numa tabela `_migrations` o que já
aplicou. É o que permite chamá-lo no começo de todo processo.

Duas escolhas deliberadas:

- **Por padrão desfaz uma.** Desfazer em cascata por acidente é perda de
  dado — é a diferença entre "corrigi a última" e "apaguei o banco".
- **Sem `down`, o rollback para com erro.** Pular em silêncio deixaria o
  banco num estado que nenhuma versão descreve.

Antes de migrar em produção: `Banco.backup(db, "antes-da-v7.db")` faz
uma cópia consistente **com o banco em uso** — é a API de backup do
SQLite, não um `cp`.

---

## Concorrência

A conexão é utilizável de várias threads: o acesso é serializado por uma
trava, e o modo `WAL` fica ligado. Sem isso, a primeira consulta de
qualquer servidor estoura com `SQLite objects created in a thread can
only be used in that same thread`.

O que a trava **não** protege é a lógica de quem lê-e-depois-escreve.
Para isso, `increment`, `upsert` e `transacao`.

---

## Forge — os outros cinco motores

```dataforge
adopt Forge

db := Forge.conectar("postgres://usuario:senha@localhost:5432/app")
// ou "mysql://root@localhost/app"
// ou "mongodb://localhost/app"
// ou "redis://localhost"
// ou "dados.db"                    — SQLite

out Forge.versao(db)
```

Trocar o motor troca a URL, e mais nada.

| Motor | Autenticação | Testado contra |
|---|---|---|
| PostgreSQL | SCRAM-SHA-256, MD5, trust | 16 |
| MySQL | caching_sha2_password (com RSA), mysql_native_password | 8.4 |
| MariaDB | mysql_native_password | 11 |
| Redis | AUTH com usuário e senha | 7.4 |
| MongoDB | SCRAM-SHA-256 e SHA-1 | 7.0 |
| SQLite | — | o do Python |

---

## Dinheiro não é `Float`

`0.1 + 0.2` dá `0.30000000000000004`, e um centavo que soma errado numa
linha soma errado num milhão. Para valor conferido por pessoa,
`Arcane.Decimal` — e no banco, guarde como **inteiro de centavos** ou
como `TEXT`, nunca como `REAL`.

---

## Veja também

- [Exemplos completos](https://dataforge-lang.vercel.app/docs/tecnicas/banco-de-dados/crud) — livraria, biblioteca, comércio, estoque, PDV
- [Relatórios e busca](https://dataforge-lang.vercel.app/docs/tecnicas/banco-de-dados/relatorios)
- [Um painel sobre o banco](https://dataforge-lang.vercel.app/docs/vitrine/projeto)
- `Arcane.Forge` — [/docs/banco-de-dados](https://dataforge-lang.vercel.app/docs/banco-de-dados)
