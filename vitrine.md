# Vitrine — dashboards e aplicações de dados

Na vitrine a peça pronta é exposta. O programa entra como um roteiro de
cima para baixo e sai como uma página web.

```dataforge
adopt Arcane.Vitrine as V

action painel():
    V.titulo("Vendas")
    regiao := V.escolha("Região", ["Sul", "Sudeste", "Norte"])
    V.metrica("Receita", "R$ 128.400", variacao := 12.5)
    V.grafico_barras(vendas_de(regiao), x := "mes")

V.rodar(painel, porta := 8501)
```

Isso é a aplicação inteira. Sem HTML, sem CSS, sem JavaScript, sem build,
e sem separar o que calcula do que mostra.

---

## O modelo de execução

A cada interação, **o programa inteiro roda de novo** — e o estado da
sessão sobrevive.

```
clique  →  o programa roda do começo  →  a árvore vira HTML  →  a tela troca
              ↑                                                      │
              └──────────  o estado da sessão continua  ─────────────┘
```

Parece desperdício e é o contrário. Quem escreve nunca pensa em callback,
em diffing, nem em qual pedaço da tela atualizar: a linha de cima sempre
aconteceu antes da linha de baixo.

É por isso que um componente **devolve** o que quem escreve precisa:

```dataforge
given V.botao("Salvar"):
    salvar(V.entrada("Nome"))
    V.sucesso("Pronto.")
```

O preço é que a página precisa ser rápida a cada clique. Daí `V.cache`,
que existe desde o primeiro dia e não como otimização posterior — e
`V.formulario`, para campos ligados a consulta pesada.

---

## Vitrine ou Kiln?

|  | Kiln | Vitrine |
|---|---|---|
| Para | sites e APIs | painéis e aplicações de dados |
| Você escreve | rotas que devolvem o que quiser | um programa de cima para baixo |
| A página é | um template que você controla | a árvore que o programa montou |
| Sintaxe | onze palavras contextuais | nenhuma palavra nova |
| Cliente | o que você puser lá | ~4 KB, prontos |

A Vitrine **roda sobre o Kiln**: HTTP, rotas, arquivos estáticos, sessão,
cabeçalhos de segurança e REST já existiam lá, testados. Reimplementá-los
criaria duas implementações do mesmo protocolo para divergirem.
`V.montar()` devolve o app Kiln por baixo.

### Por que não se chama "Stream"

`Arcane.Stream` já é o módulo de streaming de dados — tópicos, partições,
offsets, grupos de consumo. Duas coisas chamadas Stream no mesmo `adopt`
seria exatamente a ambiguidade que este projeto passa o tempo todo
evitando.

---

## Componentes

Todo componente faz duas coisas: põe um nó na árvore e devolve um valor.

### Texto

```dataforge
V.titulo("Vendas", icone := "📊")
V.subtitulo("Primeiro semestre")
V.cabecalho("Por região", nivel := 3)
V.texto("Um parágrafo.")
V.markdown("**negrito**, `código`, [link](/docs) e tabelas.")
V.codigo("x := 10", linguagem := "dataforge")
V.divisor()
V.espaco(24)
```

`V.html(…)` insere HTML **cru**. É a única porta de XSS da Vitrine, e ela
existe porque às vezes não há alternativa. Nunca passe por ali algo que
veio do usuário.

### Entrada

| Componente | Devolve |
|---|---|
| `V.botao(rótulo)` | `yes` no ciclo do clique |
| `V.entrada(rótulo)` | o texto digitado |
| `V.area_de_texto(rótulo)` | o texto digitado |
| `V.numero(rótulo, valor, minimo, maximo)` | o número |
| `V.deslizante(rótulo, min, max)` | o número |
| `V.caixa(rótulo)` · `V.interruptor(rótulo)` | `yes`/`no` |
| `V.opcao(rótulo, opções)` | a escolhida, em rádio |
| `V.escolha(rótulo, opções)` | a escolhida, em lista |
| `V.escolhas(rótulo, opções)` | um cluster |
| `V.data(rótulo)` · `V.cor(rótulo)` | texto |
| `V.arquivo(rótulo)` | `void`, ou o vault do arquivo |

**O botão vale por uma execução.** Se fosse permanente, a ação
dispararia de novo no próximo carregamento da página — e duplicar um
pagamento é o tipo de bug que ninguém perdoa.

### Formulário

Sem formulário, **cada tecla digitada roda o programa inteiro**.

```dataforge
forma := V.formulario("cadastro")
nome  := forma.entrada("Nome")
email := forma.entrada("E-mail", tipo := "email")

given forma.enviar("Cadastrar"):
    criar_usuario(nome, email)
    V.sucesso("Usuário cadastrado.")
```

### Validação

O erro aparece **sob o campo**, e não num alerta no topo. Num formulário
de doze campos, um alerta dizendo "há erros" obriga a pessoa a caçar qual
deles.

```dataforge
adopt Arcane.Regex as Regex

email := V.entrada("E-mail")
V.validar(email, Regex.is_email, "Digite um e-mail válido.")

// ou com a regra junto, devolvendo (valor, esta_bom)
senha, ok := V.campo_validado("Senha", lambda s: len(s) bigger 7,
                              "Mínimo de 8 caracteres.", tipo := "senha")
```

A regra devolve `yes`/`no` — e aí vale a mensagem que você passou — ou um
**texto**, que vira a mensagem (vazio significa que passou).

Um campo vazio e nunca tocado não é acusado: reclamar antes de a pessoa
digitar é ruído. E uma regra que **dispara** vira "a regra de validação
falhou", porque dizer "E-mail inválido" ali esconderia o bug real.

### Componentes próprios

Uma ação já é um componente:

```dataforge
action cartao_de_usuario(nome, email):
    caixa := V.cartao(nome)
    caixa.texto(email)

cartao_de_usuario("Ana", "ana@exemplo.br")
```

O registro por nome existe para quando o nome precisa atravessar
módulos — um plugin que acrescenta componentes, um tema que substitui um
deles sem que quem chama saiba:

```dataforge
V.componente("usuario", cartao_de_usuario)
V.usar("usuario", "Ana", "ana@exemplo.br")
```

### Dados

```dataforge
V.tabela(linhas)        // estática
V.frame(linhas)         // com busca e ordenação
V.metrica("Receita", "R$ 850 mil", variacao := 18.0)
V.json(vault)
V.vault(vault)
```

`V.tabela` e `V.frame` aceitam cluster de vaults, vault de colunas,
matriz e o `frame` do `Arcane.Analytics` — que são as formas que o resto
da linguagem devolve. A ordem das colunas é a de **aparição**.

A variação da métrica é um **número**, e não um texto, porque a Vitrine
precisa saber o sinal: ela sobe em verde e desce em vermelho.

### Retorno ao usuário

```dataforge
V.sucesso("Salvo.")      V.erro("Falhou.")
V.aviso("Cuidado.")      V.informacao("Dados de ontem.")
V.progresso(0.62, "62%")  V.carregando("Consultando…")
V.imagem(origem)  V.audio(origem)  V.video(origem)
V.link("Docs", "/docs")  V.baixar("Relatório", texto, "r.txt")
V.exportar_csv(linhas)   V.exportar_json(dados)
```

---

## Layout

A DataForge não tem bloco de contexto, e inventar uma palavra reservada
para o layout de um módulo seria caro demais. A saída é o idioma que a
linguagem já tem: **a área é um objeto, e os componentes são métodos**.

```dataforge
colunas := V.colunas(3)
colunas[0].metrica("Vendas", "R$ 100K")
colunas[1].metrica("Clientes", "2.500")
colunas[2].metrica("Pedidos", "8.400")
```

Aninha sem indentação extra, e a área pode ser guardada numa variável e
passada adiante — o que um `with` não permite.

```dataforge
V.colunas([2, 1])                      // pesos
V.lateral()                            // a barra lateral
V.abas(["Resumo", "Detalhe"])          // todas montadas, uma visível
V.cartao("Vendas", subtitulo := "…")
V.expandir("Detalhes", aberto := no)   // o estado sobrevive
V.container(borda := yes, altura := 320)
V.linha(alinhar := "entre")            // horizontal
V.vazio()                              // lugar reservado
V.espacador()                          // empurra para a outra ponta
```

Toda área oferece os mesmos componentes que `V`. Aprender um lugar ensina
todos. Abaixo de 860 px cada coluna ocupa a largura inteira.

**Todas as abas são montadas**, e só a escolhida aparece. Montar apenas a
visível deixaria o programa com um caminho diferente por aba, e um erro
escondido atrás de um clique.

---

## Estado e cache

Três lugares, e a diferença entre eles é quem enxerga:

| Onde | Quem vê | Some quando |
|---|---|---|
| `V.estado` | uma sessão | a sessão expira |
| `V.geral` | **todas** as sessões | o processo termina |
| `V.cache` | todas, por argumento | o TTL vence ou é invalidado |

```dataforge
V.estado.padrao("contador", 0)
given V.botao("Incrementar"):
    V.estado.somar("contador")
V.texto($"Valor: {V.estado.obter("contador")}")
```

`V.estado.somar` e `V.geral.somar` são **atômicos**: duas abas clicando ao
mesmo tempo não perdem uma das somas, o que o ler-somar-escrever à mão
perderia.

### Cache

```dataforge
mark @V.cache
action vendas(mes):
    yield Banco.consultar("SELECT … WHERE mes = ?", [mes])

mark @V.cache(validade := 300, teto := 32)
action cotacao(moeda):
    yield Http.get($"https://…/{moeda}").json()

mark @V.cache(pasta := ".cache/ibge")
action municipios():
    yield Http.get("https://…/municipios").json()
```

| Opção | Faz |
|---|---|
| `validade` | segundos até o valor vencer (TTL) |
| `teto` | quantos guardar; ao encher sai o menos usado (LRU) |
| `pasta` | grava em disco e sobrevive a reiniciar |

O teto existe porque um cache sem limite é um vazamento com outro nome.
`V.cache.invalidar(vendas)` esvazia; `vendas.sem_cache(mes)` chama a ação
original.

O cache em disco só guarda o que vira JSON. Guardar objeto arbitrário
exigiria `pickle`, e ler `pickle` de um arquivo que outro processo
escreveu é execução de código — num framework web, é a porta aberta.

---

## Gráficos

```dataforge
V.grafico_linha(dados, x := "mes", y := "receita")
V.grafico_barras(dados, x := "mes", y := ["receita", "meta"])
V.grafico_area(…)  V.grafico_dispersao(…)
V.grafico_pizza(…) V.grafico_rosca(…) V.grafico_barras_h(…)
V.histograma(dados, campo := "idade", faixas := 12)
```

Sem `x` e `y`, a Vitrine adivinha: a primeira coluna não numérica vira o
eixo, e as numéricas viram as séries.

A forma construída, quando há mais a dizer:

```dataforge
g := V.grafico("barras", vendas)
g.eixo_x("mes")
g.eixo_y(["receita", "meta"])
g.titulo("Vendas por período")
g.cores(["#FED403", "#0F62FE"])
g.empilhar(yes)
g.limite_y(0, 1000)
V.desenhar(g)
```

**O gráfico é SVG escrito no servidor.** Uma dependência de JavaScript
obrigaria a página a buscar centenas de kilobytes de uma CDN, o que quebra
qualquer aplicação em rede fechada — que é exatamente onde painel de dados
costuma rodar. SVG também imprime, escala e é legível por leitor de tela.

---

## Páginas e segurança

```dataforge
V.app("Painel")
V.pagina("/", inicio, titulo := "Início", icone := "🏠")
V.pagina("/vendas", vendas, titulo := "Vendas")
V.pagina("/produto/:id", produto, oculta := yes)
V.rodar(porta := 8501)
```

`V.menu()` desenha o menu na barra lateral. `V.parametro("id")` lê tanto
o parâmetro da rota quanto o da query.

```dataforge
V.navegar("/entrar")   // vai para outra página, e para aqui
V.parar()              // acaba a página neste ponto, sem erro
V.recarregar()         // roda de novo, jogando fora a árvore montada
```

### Autenticação

```dataforge
action conferir(usuario, senha):
    linha := Banco.um("SELECT * FROM usuarios WHERE email = ?", [usuario])
    given linha is not void and Crypto.conferir_senha(senha, linha["hash"]):
        yield {"nome": linha["nome"], "papel": linha["papel"]}
    yield void

V.autenticacao(conferir, {
    "admin":  ["ver", "editar", "apagar"],
    "leitor": ["ver"]
})

action relatorio():
    V.exigir_login()
    V.titulo($"Olá, {V.usuario()["nome"]}")

action edicao():
    V.exigir_permissao("editar")
```

`V.exigir_login()` desenha o formulário e **para** a página. Quando já há
alguém logado, devolve o usuário e não desenha nada — o que permite
chamá-la sempre na primeira linha.

Ao acertar a senha, a página **roda de novo do começo**. Continuar de onde
parou deixaria a tela vazia para quem escreveu `given V.autenticado(): …`:
esse teste já passou com a resposta antiga.

### O que vem de graça

| Cabeçalho | Contra |
|---|---|
| `X-Content-Type-Options: nosniff` | o navegador adivinhar o tipo |
| `X-Frame-Options: SAMEORIGIN` | clickjacking |
| `Referrer-Policy` | vazar a URL para terceiros |
| `Content-Security-Policy` | script de outra origem |

O cookie de sessão é `HttpOnly` e `SameSite=Lax`. Todo texto é escapado —
`V.markdown` escapa **antes** de reconhecer a marcação, e recusa link
`javascript:`.

---

## Acessibilidade e idioma

Nada disto precisa ser ligado — é como os componentes são desenhados:
HTML semântico, link para pular a navegação, foco sempre visível, rótulo
ligado ao campo, erro anunciado com `role="alert"`, abas navegáveis pelas
setas, `prefers-reduced-motion` e `prefers-color-scheme` respeitados.

A variação de uma métrica sai com a seta marcada `aria-hidden` e a
palavra ao lado, visível só para leitor de tela: cor e seta sozinhas
excluem quem não vê a tela **e** quem não separa vermelho de verde.

A paleta clara usa `#B28600` como primária, e não o amarelo `#FED403` da
marca: amarelo sobre branco dá contraste 1,3:1, e a WCAG pede 4,5:1.

O que fica com você: o texto alternativo das imagens (`legenda`), rótulos
que descrevem (`"Excluir pedido 42"` diz mais que `"Excluir"`), o
contraste do seu tema, e o que você puser em `V.html`, que passa cru.

### Idioma

```dataforge
V.i18n.carregar("pt-BR", {"painel.titulo": "Painel", "ola": "Olá, {nome}"})
V.i18n.carregar("en-US", {"painel.titulo": "Dashboard", "ola": "Hello, {nome}"})

action painel():
    V.titulo(V.t("painel.titulo"))
    V.texto(V.t("ola", nome := "Ana"))
```

O idioma é **por sessão**: dois visitantes podem estar lendo a mesma
página em línguas diferentes. `V.i18n.seletor("Idioma")` desenha a troca
e reexecuta a página — mudá-lo no meio deixaria a metade de cima na
língua anterior.

Uma chave sem tradução aparece **crua** (`painel.titulo`), e não vazia:
feio o bastante para alguém corrigir, e informativo o bastante para dizer
qual falta.

---

## De onde vêm os dados

De lugar nenhum especial. A Vitrine desenha; quem lê os dados é o módulo
que você já usaria num script.

```dataforge
adopt Arcane.Database as DB
adopt Arcane.Analytics as An
adopt Arcane.Cortex as ML

banco := DB.connect("vendas.db")

mark @V.cache
action vendas():
    yield DB.query(banco, "SELECT mes, numero, receita FROM v")

action painel():
    V.frame(vendas())                                    // do banco
    V.grafico_barras(An.from_records(vendas()), x := "mes")
    V.vault(An.describe(An.from_records(vendas())))      // estatística
    modelo := ML.linear(vendas(), "receita", ["numero"])
    V.metrica("Previsão", round(ML.prever(modelo, [{"numero": 4}])[0], 1))
```

| Para | Use |
|---|---|
| banco relacional | `Arcane.Database`, `Arcane.Forge` |
| CSV, JSON, Excel | `Arcane.IO`, `Arcane.Excel` |
| estatística e DataFrame | `Arcane.Analytics` (um `Frame` entra direto) |
| Parquet e Data Lake | `Arcane.Lago` |
| streaming | `Arcane.Stream`, com `V.atualizar_a_cada(n)` |
| aprendizado de máquina | `Arcane.Cortex` |
| uma API de fora | `Arcane.Http`, sob `mark @V.cache` sempre |

### A mesma aplicação serve uma API

`V.montar()` devolve o app **Kiln** por baixo. O painel e a API vivem no
mesmo processo, na mesma porta, lendo das mesmas ações em cache:

```dataforge
adopt Kiln

kiln := V.montar()
Kiln.get(kiln, "/api/vendas", lambda req: {"itens": vendas()})
Kiln.use(kiln, Kiln.cors())
Kiln.use(kiln, Kiln.rate_limit(120))

V.subir(porta := 8501)
```

As suas rotas vencem sempre: as páginas viram rotas declaradas, e o que
não casa com nenhuma cai no **tratador de 404** — que roda depois de
todas. Uma rota curinga seria casada na ordem do registro e engoliria
tudo o que você acrescentasse depois do `V.montar()`.

Uma URL que não existe responde **404 de verdade**, e não um 200 com
"404" escrito no corpo.

---

## Testar sem navegador

A árvore de componentes é um **dado**, e conferir um dado é o que um teste
sabe fazer.

```dataforge
crucible "o painel":
    trial "o botão soma":
        t := V.testar(painel)
        t.clicar("Somar")
        assert t.metrica("Total") is "1"
```

| Agir | Perguntar |
|---|---|
| `t.clicar(rótulo)` | `t.texto()` |
| `t.digitar(rótulo, valor)` | `t.achar(tipo)` · `t.primeiro(tipo)` |
| `t.marcar(rótulo, ligado)` | `t.quantos(tipo)` · `t.existe(…)` |
| `t.selecionar(rótulo, valor)` | `t.metrica(rótulo)` · `t.valor(rótulo)` |
| `t.abrir_aba(rótulo)` | `t.alertas(nível)` |
| `t.enviar(formulário)` | `t.estado(chave)` |
| `t.enviar_arquivo(…)` | `t.falhou()` · `t.falhas()` |
| `t.ir_para(caminho)` · `t.rodar()` | `t.html()` · `t.arvore()` |

Quando um rótulo não existe, a mensagem lista os que existem na página.

Para status, cabeçalho e redirecionamento, `V.pedir(app, "GET", "/")` —
o `Kiln.test` por baixo, sem abrir porta nenhuma.

**Mas ele para antes do `Set-Cookie`.** Bug de cookie e de concorrência só
aparecem com `V.servir(0)` e um cliente HTTP real — foi assim que se
descobriu um cookie malformado que fazia cada pedido abrir uma sessão
nova, com o sintoma de um contador que nunca passava de 1.

---

## Produção

### A linha de comando

```bash
dataforge vitrine new meupainel   # cria o projeto
dataforge vitrine dev             # sobe recarregando ao salvar
dataforge vitrine run             # sobe, sem recarregar
dataforge vitrine doctor          # diz por que ela não sobe

dataforge vitrine dev --porta=8600 --host=0.0.0.0
```

Sem argumento, ele procura `main.df`, `app.df`, `painel.df` e
`src/main.df`, nessa ordem, e depois a entrada do `forge.toml`. A porta e
o host da linha de comando **vencem** o que está escrito no arquivo.

O `doctor` responde as perguntas de quem vê uma tela em branco, da causa
mais provável para a menos: o módulo carrega, o Kiln está lá, existe um
arquivo que sobe, ele compila, ele adota a Vitrine, ele chama `V.subir`,
a porta está livre.

Não há `build` nem `deploy`. Não existe etapa de build — sem bundler, sem
transpilação, sem `node_modules`; o que roda é o próprio `.df`. E
`deploy` seria inventar uma opinião sobre Docker, systemd ou nuvem que o
projeto não tem. Os dois comandos existem só para **explicar isso** a
quem veio de outro framework.

### Programaticamente

```dataforge
V.subir(porta := 8501, recarregar := yes)   // hot reload
porta := V.servir(0)                         // em segundo plano
```

Com `recarregar := yes`, salvar um `.df` **reinicia o processo**.
Reiniciar, e não recarregar o módulo: o estado de um módulo recarregado
pela metade produz erros que não existem no código.

```dataforge
V.registrar("consulta lenta", "aviso", {"ms": 1840})
V.logs(50, "erro")
V.metricas()   // execuções, erros, média em ms, sessões, cache
V.saude()
```

Duas rotas vêm prontas: `GET /__vitrine__/saude` e
`GET /__vitrine__/metricas`.

```dataforge
V.antes(so_de_dia)                      // 'no' interrompe a página
V.depois(anotar)                        // com o contexto montado
V.tarefa(enviar_email, destinatario)    // thread, não espera
V.agendar(recalcular, 3600)             // de hora em hora
V.plugin("tema-empresa", instalar)      // o mesmo nome duas vezes é erro
V.atualizar_a_cada(15)                  // a página se recarrega sozinha
```

`V.atualizar_a_cada` **não** dispara quando a aba está escondida: cobrar
do servidor por uma página que ninguém está vendo é desperdício puro.

### O que colocar na frente

Em produção pública, ponha um nginx ou Caddy na frente. A Vitrine roda
sobre o Kiln, que roda sobre o `http.server` do Python: não há HTTP/2, TLS
nem streaming de resposta.

E a sessão vive **na memória do processo**. Com mais de um processo, dois
pedidos da mesma pessoa caem em memórias diferentes. Um processo por
aplicação, com o proxy na frente, é a forma testada.

---

## Referência

Os 113 símbolos, com assinatura extraída do código-fonte:
<https://dataforge-lang.vercel.app/docs/vitrine/referencia>.

Um painel completo que roda e se testa sozinho está em
`examples/vitrine_dashboard.df`; o exercício comentado, em
`exercicios/28-vitrine/`.
