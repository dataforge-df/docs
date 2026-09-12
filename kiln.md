# Kiln — o framework web do DataForge

No forno (*kiln*) a peça ganha a forma final. Aqui a requisição entra crua e sai
como resposta.

O Kiln não é um módulo como os outros: ele tem **sintaxe própria na linguagem**,
para que uma rota se leia como uma rota — e não como uma chamada de função com
um `lambda` dentro.

```dataforge
adopt Kiln

produtos := [{"id": 1, "nome": "Martelo", "preco": 89.9}]

server loja on 8080:
    middleware Kiln.logger()
    middleware Kiln.cors()

    route GET "/":
        respond html "<h1>Forja</h1>"

    route GET "/produtos":
        respond json {"itens": produtos, "total": len(produtos)}

    route GET "/produtos/:id":
        p := achar(int(params["id"]))
        given p is void:
            respond 404 json {"erro": "não achei"}
        respond json p

    route POST "/produtos":
        produtos.append(body)
        respond 201 json body

ignite loja
```

Sem dependência externa: `http.server` da biblioteca padrão do Python, com
roteamento, middleware, sessão e templates escritos aqui.

---

## As onze palavras

| Palavra | Faz | Equivale a |
|---------|-----|------------|
| `server nome on porta:` | declara a aplicação | `Kiln.forge(nome)` |
| `route VERBO "caminho":` | registra uma rota | `Kiln.get(app, …)` |
| `respond [status] [tipo] valor` | responde e **encerra a rota** | `yield Kiln.json(…)` |
| `render "arquivo" with dados` | renderiza um template | `yield Kiln.render(…)` |
| `redirect "/destino"` | 302 com `Location` | `yield Kiln.redirect(…)` |
| `middleware expr` | roda antes de toda rota | `Kiln.use(app, …)` |
| `after expr` | roda depois, com a resposta | `Kiln.after(app, …)` |
| `mount outro at "/prefixo"` | junta outro server | `Kiln.mount(…)` |
| `assets "/p" from "pasta"` | serve arquivos do disco | `Kiln.static(…)` |
| `views "pasta"` | onde ficam os templates | `Kiln.templates(…)` |
| `ignite nome [on porta]` | sobe e bloqueia | `Kiln.listen(…)` |

Todas são **contextuais**: só valem dentro de um bloco `server`. Fora dali,
`render := 42` e `action route(x)` continuam válidos — nenhum programa escrito
antes do Kiln parou de compilar.

`server` só abre um bloco quando o que vem em seguida confirma: um nome seguido
de `:`, de `on` ou de `at`.

---

## Declarar não é subir

`server` monta a aplicação. Quem acende o forno é `ignite`. A separação parece
pedante até o primeiro teste:

```dataforge
r := Kiln.test(loja, "GET", "/produtos/2")   // sem abrir socket
out r["status"], r["body"]["nome"]
```

Testar uma rota fica tão barato quanto testar uma ação. Num projeto de verdade,
o `app.df` monta e o `main.df` acende — senão um teste que importasse o `main`
subiria o servidor e nunca terminaria.

Três formas de rodar:

| Forma | Faz | Quando |
|-------|-----|--------|
| `Kiln.test(app, verbo, caminho)` | executa a rota, sem socket | teste |
| `Kiln.serve(app, porta)` | sobe em segundo plano, devolve a porta | script |
| `ignite app on 8080` | sobe e bloqueia até Ctrl-C | produção |

`Kiln.serve(app, 0)` deixa o sistema escolher uma porta livre.

---

## Dentro de uma rota

Seis nomes já existem: `req`, `params`, `query`, `body`, `headers` e `session`.

```dataforge
route POST "/itens/:id":
    out params["id"]              // do caminho — sempre existe
    out query["f"] ?? "padrao"    // da query — pode não vir
    out body["nome"] ?? void      // do corpo — pode não vir
```

| Padrão | Casa | Resultado |
|--------|------|-----------|
| `/itens/:id` | `/itens/42` | `{"id": "42"}` |
| `/files/*resto` | `/files/a/b.txt` | `{"resto": "a/b.txt"}` |

---

## O que vem de graça

| Situação | O Kiln faz |
|----------|------------|
| caminho não registrado | 404 |
| caminho existe, verbo não | **405** com `Allow` |
| `OPTIONS` com `Kiln.cors()` | responde o preflight |
| erro na rota | 500, detalhe no terminal, servidor de pé |
| JSON quebrado no corpo | chega como texto — a rota decide se é 400 |
| corpo grande demais | 413 antes de ler tudo na memória |
| `../` num caminho estático | 403, antes de abrir o arquivo |

---

## Três armadilhas

**1. `query["x"]` sem `??` dá 500.** A query é do visitante: a chave pode não
vir, e indexar um vault sem a chave é erro. Vale para `body` e `headers`
também; `params` é a exceção, porque se a rota casou o parâmetro existe.

**2. Cada pedido roda numa thread.** O `Arcane.Database` serializa o acesso para
que SQLite funcione, mas estado compartilhado em memória não é sincronizado —
duas threads escrevendo na mesma variável podem perder atualizações. Um teste
que roda tudo na mesma thread **não** pega isso.

**3. A ordem do middleware importa.** `rate_limit` antes de `auth`: na ordem
inversa, um pedido barrado pelo `auth` nunca é contado — e quem está martelando
a porta com credenciais inválidas é justamente quem você quer limitar.

---

## Templates

```html
<h1>{{titulo}}</h1>
{{#produtos}}<article>{{nome}} — {{preco}}</article>{{/produtos}}
{{^produtos}}<p>Nada na forja.</p>{{/produtos}}
```

| Marca | Faz |
|-------|-----|
| `{{x}}` | escreve, **escapando HTML** |
| `{{&x}}` | escreve sem escapar |
| `{{#lista}}…{{/lista}}` | repete |
| `{{^lista}}…{{/lista}}` | mostra quando vazio |

O escape é o padrão: um produto chamado `Bigorna <de aço>` sai
`Bigorna &lt;de aço&gt;`, e isso fecha a porta para XSS por acidente. A sintaxe
é pequena de propósito — template que vira linguagem é código escondido onde
ninguém procura.

---

## Onde ver mais

| Onde | O quê |
|------|-------|
| `exercicios/22-web-kiln/` | 7 exercícios, cada um com `.md` explicativo |
| `projetos/loja-web/` | um site completo: páginas, API, sessão, Excel — 29 testes |
| `tests/test_kiln.py` | 49 testes do framework |
| [dataforge-lang.vercel.app/docs/kiln](https://dataforge-lang.vercel.app/docs/kiln) | a documentação completa, com a referência das 46 funções |

---

## O que ele não é

O Kiln roda sobre o `http.server` do Python. É sólido para uma aplicação
interna, um painel, uma API de time — e **não é um servidor de borda**. Em
produção pública, ponha um nginx ou um Caddy na frente para cuidar de TLS,
compressão e clientes lentos.

Não há WebSocket, não há HTTP/2, não há streaming de resposta, e a sessão vive
na memória do processo (com dois processos, o visitante desloga a cada pedido —
guarde a sessão no banco e use `Kiln.sign`).
