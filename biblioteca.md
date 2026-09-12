# Biblioteca padrão DataForge — módulos `Arcane.*`

Referência gerada a partir das assinaturas reais do código
(`python3 tools/gerar_doc_stdlib.py`). Todo módulo é carregado com `adopt`:

```dataforge
adopt Arcane.Math as Math
out Math.sqrt(16)
```

Cada módulo tem um **nome curto** equivalente (`adopt Math as M` funciona igual).

## Índice

| Módulo | Nome curto | Símbolos | Para quê |
|--------|-----------|----------|----------|
| [`Arcane.Math`](#arcanemath) | `Math` | 51 | Matemática, álgebra linear e estatística básica. |
| [`Arcane.Text`](#arcanetext) | `Text` | 58 | Manipulação de texto, formatação, tabelas e conversão de caixa. |
| [`Arcane.Analytics`](#arcaneanalytics) | `Analytics` | 65 | Análise de dados: estatística, regressão, clustering e gráficos ASCII. |
| [`Arcane.Functional`](#arcanefunctional) | `Functional` | 56 | Utilitários funcionais: composição, lentes, Maybe/Either, transdutores. |
| [`Arcane.Database`](#arcanedatabase) | `Database / DB` | 64 | Banco de dados SQLite: tabelas, consultas, migrações e importação. |
| [`Arcane.Excel`](#arcaneexcel) | `Excel / Xlsx` | 29 | Planilhas .xlsx: ler, gravar, fórmulas e conversão para CSV e frame. |
| [`Arcane.Meta`](#arcanemeta) | `Meta` | 12 | Metadados de decorador: ler @Nome em tempo de execução. |
| [`Kiln`](#kiln) | `Kiln` | 73 | Framework web: rotas, middleware, templates, sessão e arquivos estáticos. |
| [`Arcane.Test`](#arcanetest) | `Test` | 34 | Asserções e organização de suítes de teste. |
| [`Arcane.Regex`](#arcaneregex) | `Regex` | 32 | Expressões regulares e validadores brasileiros (CPF, CNPJ, telefone). |
| [`Arcane.IO`](#arcaneio) | `IO` | 30 | Arquivos, diretórios, JSON, CSV e shell. |
| [`Arcane.Http`](#arcanehttp) | `Http / Server` | 17 | Servidor HTTP: rotas, middleware, JSON, arquivos estáticos. |
| [`Arcane.Async`](#arcaneasync) | `Async` | 46 | Promessas, filas, agendamento e execução concorrente. |
| [`Arcane.Data`](#arcanedata) | `Data` | 13 | DataFrames, séries e transformações tabulares. |
| [`Arcane.Web`](#arcaneweb) | `Web / Network` | 11 | Cliente HTTP, URL encoding e JSON. |
| [`Arcane.Cortex`](#arcanecortex) | `Cortex` | 25 | Aprendizado de máquina: regressão, árvore, floresta, k-NN, Naive Bayes, k-médias e PCA. |
| [`Arcane.Time`](#arcanetime) | `Time` | 54 | Datas, horas, durações e cronometragem. |
| [`Arcane.OS`](#arcaneos) | `OS` | 42 | Sistema operacional, ambiente, disco e processo atual. |
| [`Arcane.Process`](#arcaneprocess) | `Process` | 15 | Execução de processos externos, com stdout, stderr e código de saída. |
| [`Arcane.Logging`](#arcanelogging) | `Logging / Log` | 14 | Registro estruturado de eventos, com níveis e destinos. |
| [`Arcane.Crypto`](#arcanecrypto) | `Crypto` | 48 | Hashes, HMAC, senhas, codificações, aleatoriedade segura e cifragem de arquivo (ChaCha20-Poly1305). |
| [`Arcane.Collections`](#arcanecollections) | `Collections` | 63 | Estruturas de dados e algoritmos: pilha, fila, grafo, união-busca. |
| [`Arcane.Serialization`](#arcaneserialization) | `Serialization / Serde` | 26 | JSON, CSV, INI, TOML, XML e conversões entre eles. |
| [`Arcane.Forge`](#arcaneforge) | `Forge / Banco` | 28 | Banco de dados: SQLite, Postgres, MySQL, Redis e MongoDB pela mesma interface. |
| [`Arcane.Crucible`](#arcanecrucible) | `Crucible` | 50 | Framework de testes: suítes, matchers, fixtures, dublês e benchmark. |
| [`Arcane.Iter`](#arcaneiter) | `Iter` | 44 | Iteradores preguiçosos e composição de ações: janelas, combinatória, memoize. |
| [`Arcane.Color`](#arcanecolor) | `Color / Cor` | 66 | Cor de 24 bits no terminal, tabela, moldura, barra de progresso e árvore. |
| [`Arcane.Concurrent`](#arcaneconcurrent) | `Concurrent / Paralelo` | 25 | Threads, processos, canal bloqueante, grupo de tarefas e prazo. |
| [`Arcane.Archive`](#arcanearchive) | `Archive / Zip` | 8 | Zip e tar: compactar, listar, conferir e extrair recusando Zip Slip e zip bomb. |
| [`Arcane.Pipeline`](#arcanepipeline) | `Pipeline / Fluxo` | 11 | Orquestração de ETL/ELT: DAG, dependências, retry, incremental e relatório. |
| [`Arcane.Stream`](#arcanestream) | `Stream / Corrente` | 17 | Streaming: tópicos, partições, offsets, grupos de consumo e janelas de tempo. |
| [`Arcane.Observar`](#arcaneobservar) | `Observar / Observe` | 19 | Observabilidade: métricas com percentil, tracing aninhado e linhagem de dados. |
| [`Arcane.Lago`](#arcanelago) | `Lago / Parquet` | 18 | Data Lake: Parquet nativo, partições Hive, camadas bronze/prata/ouro e compactação. |
| [`Arcane.Malha`](#arcanemalha) | `Malha` | 23 | Chamada entre serviços que não mente: cliente HTTP com prazo, retry com recuo e tremor, disjuntor de três estados, descoberta por nome e propagação automática do rastro do pedido. |
| [`Arcane.Vitrine`](#arcanevitrine) | `Vitrine` | 113 | O framework de dashboards e aplicações de dados: você escreve um programa de cima para baixo e ele vira uma página web, com componentes, layout, gráficos em SVG, estado por sessão e cache — servido pelo Kiln. |
| [`Arcane.API`](#arcaneapi) | `API` | 7 | A API do Kiln vista de fora: OpenAPI, coleção do Insomnia e do Postman, curl e a tabela em Markdown — tudo derivado das rotas registradas. |
| [`Arcane.Decimal`](#arcanedecimal) | `Decimal / Exato` | 16 | Número decimal exato, para quando 0,1 + 0,2 precisa dar 0,3 — dinheiro, imposto, e todo número que alguém confere na mão. |
| [`Arcane.Ponte`](#arcaneponte) | `Ponte / Bridge` | 12 | A ponte para o Python: perguntar se um pacote existe, explorar o que ele oferece e converter o que ele devolve. |
| [`Arcane.Qualidade`](#arcanequalidade) | `Qualidade / Quality` | 13 | Qualidade de dados: as seis dimensões, perfil, validação e limpeza. |

> Os nomes curtos e os aliases (`DB`, `Server`, `Network`) apontam para o mesmo
> módulo — use o que ficar mais legível.

## Exemplos rápidos

```dataforge
adopt Arcane.Math as Math
adopt Arcane.Text as Text
adopt Arcane.Analytics as An

out Math.sqrt(16)                       // 4.0
out Math.is_prime(97)                   // yes
out Text.slug("Ola Mundo")              // ola-mundo
out Text.box("Relatorio")               // caixa desenhada
out An.correlation([1,2,3], [2,4,6])    // 1.0
```


---

## Arcane.Math

Matemática, álgebra linear e estatística básica.

```dataforge
adopt Arcane.Math as Math
```

**Constantes**

| Nome | Valor |
|------|-------|
| `E` | `2.718281828459045` |
| `INF` | `inf` |
| `NAN` | `nan` |
| `PI` | `3.141592653589793` |
| `TAU` | `6.283185307179586` |
| `random` | `{'random': <built-in method random of Random objec…` |

**Funções (45)**

| Assinatura |
|------------|
| `abs(x)` |
| `acos(x)` |
| `asin(x)` |
| `atan(x)` |
| `atan2(y, x)` |
| `cbrt(x)` |
| `ceil(x)` |
| `clamp(value, min_val, max_val)` |
| `comb(n, k)` |
| `cos(x)` |
| `degrees(x)` |
| `determinant(matrix)` |
| `dot(a, b)` |
| `exp(x)` |
| `factorial(n)` |
| `fibonacci(n)` |
| `floor(x)` |
| `gcd(*inteiros)` |
| `hypot(*coordenadas)` |
| `identity(n)` |
| `is_prime(n)` |
| `lcm(a, b)` |
| `lerp(a, b, t)` |
| `log(x, base=e)` |
| `log10(x)` |
| `log2(x)` |
| `map_range(value, in_min, in_max, out_min, out_max)` |
| `matrix(data)` |
| `max(*valores)` |
| `mean(data)` |
| `median(data)` |
| `min(*valores)` |
| `ones(rows, cols=None)` |
| `perm(n, k=void)` |
| `pow(x, y)` |
| `radians(x)` |
| `round(numero, casas=void)` |
| `sin(x)` |
| `sqrt(x)` |
| `stdev(data)` |
| `sum(data)` |
| `tan(x)` |
| `transpose(matrix)` |
| `variance(data)` |
| `zeros(rows, cols=None)` |


---

## Arcane.Text

Manipulação de texto, formatação, tabelas e conversão de caixa.

```dataforge
adopt Arcane.Text as Text
```

**Funções (58)**

| Assinatura |
|------------|
| `align(lines, alignment='left', width=None)` |
| `box(text, style='single')` |
| `camel_case(text)` |
| `char_count(text, include_spaces=True)` |
| `closest(query, candidates, n=3)` |
| `constant_case(text)` |
| `currency(amount, symbol='$', decimals=2)` |
| `dedent(text)` |
| `diff(a, b)` |
| `distance(a, b)` |
| `dot_case(text)` |
| `escape_html(text)` |
| `escape_regex(text)` |
| `extract_emails(text)` |
| `extract_hashtags(text)` |
| `extract_links(text)` |
| `extract_mentions(text)` |
| `extract_numbers(text)` |
| `frequency(text)` |
| `fuzzy_match(query, text, threshold=0.6)` |
| `highlight(text, word, start='\x1b[1;33m', end='\x1b[0m')` |
| `indent(text, prefix='    ')` |
| `kebab_case(text)` |
| `line_count(text)` |
| `lorem(sentences=3)` |
| `ngrams(text, n=2)` |
| `normalize_whitespace(text)` |
| `number_format(n, decimals=2, thousands_sep=',', decimal_sep='.')` |
| `pad(text, width, fill=' ', align='left')` |
| `paragraph_count(text)` |
| `parse_csv(text, delimiter=',')` |
| `parse_ini(text)` |
| `parse_query(query_string)` |
| `pascal_case(text)` |
| `path_case(text)` |
| `random_string(length=16, charset='alphanumeric')` |
| `reading_time(text, wpm=200)` |
| `remove_accents(text)` |
| `render(template, context)` |
| `repeat_str(text, n, separator='')` |
| `sentence_case(text)` |
| `sentence_count(text)` |
| `similarity(a, b)` |
| `slug(text)` |
| `snake_case(text)` |
| `strip_ansi(text)` |
| `strip_html(text)` |
| `table(headers, rows, style='simple')` |
| `template(text)` |
| `title_case(text)` |
| `to_csv(data, delimiter=',')` |
| `to_query(params)` |
| `transliterate(text)` |
| `truncate(text, length=50, suffix='...')` |
| `unescape_html(text)` |
| `unified_diff(a, b, a_name='original', b_name='modified')` |
| `word_count(text)` |
| `wrap(text, width=80)` |


---

## Arcane.Analytics

Análise de dados: estatística, regressão, clustering e gráficos ASCII.

```dataforge
adopt Arcane.Analytics as Analytics
```

**Funções (65)**

| Assinatura |
|------------|
| `DataFrame(data=None, columns=None)` |
| `autocorrelation(data, lag=1)` |
| `bar_chart(data, labels=None, width=40, char='█')` |
| `bin_data(data, bins=5)` |
| `bootstrap(data, n_samples=1000, stat_fn=None)` |
| `box_plot(data, width=40)` |
| `correlation(x, y)` |
| `correlation_matrix(data_dict)` |
| `cosine_similarity(a, b)` |
| `covariance(x, y)` |
| `create_frame(data, columns=None)` |
| `cross_tab(data, row_fn, col_fn)` |
| `cumulative_sum(data)` |
| `data_types(data)` |
| `describe(data)` |
| `diff(data, periods=1)` |
| `euclidean_distance(a, b)` |
| `exponential_smoothing(data, alpha=0.3)` |
| `frequency_table(data)` |
| `from_csv(path, delimiter=',', has_header=True)` |
| `from_dict(d)` |
| `from_json(path)` |
| `from_records(records)` |
| `group_by(data, key_fn)` |
| `heatmap(matrix, row_labels=None, col_labels=None)` |
| `histogram(data, bins=10, width=40, char='█')` |
| `iqr(data)` |
| `kmeans(data, k=3, max_iter=100)` |
| `kurtosis(data)` |
| `lag(data, k=1)` |
| `line_chart(data, width=60, height=15)` |
| `linear_regression(x, y)` |
| `log_transform(data, base=None)` |
| `manhattan_distance(a, b)` |
| `mean(data)` |
| `median(data)` |
| `min_max_scale(data, feature_range=(0, 1))` |
| `missing_values(data)` |
| `mode(data)` |
| `moving_average(data, window=3)` |
| `normalize(data, low=0, high=1)` |
| `outliers(data, threshold=1.5)` |
| `percentile(data, p)` |
| `pivot_table(data, index_fn, value_fn, agg='sum')` |
| `predict_linear(model, x_val)` |
| `profile(data)` |
| `quartiles(data)` |
| `r_squared(x, y)` |
| `rank(data, method='average')` |
| `running_average(data)` |
| `sample(data, n=5, replace=False)` |
| `scatter_plot(x, y, width=40, height=20)` |
| `seasonality(data, period=7)` |
| `silhouette_score(data, labels)` |
| `skewness(data)` |
| `sparkline(data)` |
| `standardize(data)` |
| `stdev(data)` |
| `stratified_sample(data, labels, n_per_group=2)` |
| `summary(data)` |
| `trend(data)` |
| `unique_counts(data)` |
| `value_counts(data)` |
| `variance(data)` |
| `zscore(data)` |


---

## Arcane.Functional

Utilitários funcionais: composição, lentes, Maybe/Either, transdutores.

```dataforge
adopt Arcane.Functional as Functional
```

**Funções (56)**

| Assinatura |
|------------|
| `all_pass(*preds)` |
| `any_pass(*preds)` |
| `both(f, g)` |
| `chunk(n, collection)` |
| `complement(fn)` |
| `compose(*fns)` |
| `constantly(x)` |
| `curry(fn, arity=None)` |
| `drop_while(fn, collection)` |
| `either(f, g)` |
| `filter(fn, collection)` |
| `flat_map(fn, collection)` |
| `flip(fn)` |
| `frequencies(collection)` |
| `from_either(left_fn, right_fn, e)` |
| `from_maybe(default, m)` |
| `group_by(fn, collection)` |
| `identity(x)` |
| `index_by(fn, collection)` |
| `interleave(*collections)` |
| `into(target_type, xform, collection)` |
| `is_just(m)` |
| `is_left(e)` |
| `is_nothing(m)` |
| `is_right(e)` |
| `just(value)` |
| `juxt(*fns)` |
| `left(value)` |
| `lens(*keys)` |
| `map(fn, collection)` |
| `match(value, *cases)` |
| `maybe(value)` |
| `memoize(fn)` |
| `nothing()` |
| `once(fn)` |
| `over(lens, fn, obj)` |
| `partial(fn, *args)` |
| `partition_by(fn, collection)` |
| `pipe(*fns)` |
| `reduce(fn, collection, initial=None)` |
| `right(value)` |
| `scan(fn, collection, initial)` |
| `set_lens(lens, value, obj)` |
| `sort_by(fn, collection)` |
| `spread(fn)` |
| `take_while(fn, collection)` |
| `tap(fn, value)` |
| `thread_first(value, *fns)` |
| `thread_last(value, *fns)` |
| `trampoline(fn, *args)` |
| `transduce(xform, reducer, initial, collection)` |
| `try_catch(fn)` |
| `unique_by(fn, collection)` |
| `view(lens, obj)` |
| `when(*conditions)` |
| `zip_with(fn, *collections)` |


---

## Arcane.Database

Banco de dados SQLite: tabelas, consultas, migrações e importação.

```dataforge
adopt Arcane.Database as Database
```

**Funções (64)**

| Assinatura |
|------------|
| `Model(db, table, schema=None)` |
| `QueryBuilder(db, table)` |
| `add_column(db, table, name, col_type='TEXT')` |
| `aggregate(db, table, agregados, group_by=None, where=None, order_by=None, limit=None)` |
| `backup(db, dest_path)` |
| `begin(db)` |
| `builder(db, table)` |
| `check_foreign_keys(db)` |
| `close(db)` |
| `columns(db, table)` |
| `commit(db)` |
| `connect(path)` |
| `count(db, table, where=None)` |
| `create_index(db, table, columns, unique=False, name=None)` |
| `create_model(db, table, schema)` |
| `create_search(db, table, columns, nome=None)` |
| `create_table(db, name, schema)` |
| `database_size(db)` |
| `delete(db, table, where=None)` |
| `drop_index(db, nome)` |
| `drop_table(db, name)` |
| `execute(db, sql, params=None)` |
| `execute_many(db, sql, params_list)` |
| `execute_script(db, script)` |
| `exists(db, table, where)` |
| `explain(db, sql, params=None)` |
| `export_csv(db, table, path)` |
| `export_json(db, table, path)` |
| `foreign_keys(db, table)` |
| `group_count(db, table, column, where=None, order_by='quantidade DESC', limit=None)` |
| `import_csv(db, table, path, has_header=True)` |
| `import_json(db, table, path)` |
| `in_transaction(db)` |
| `increment(db, table, column, delta=1, where=None)` |
| `indexes(db, table=None)` |
| `insert(db, table, data)` |
| `insert_many(db, table, records)` |
| `insert_or_ignore(db, table, data)` |
| `integrity(db)` |
| `memory()` |
| `migrate(db, migrations)` |
| `migrations_applied(db)` |
| `paginate(db, table, pagina=1, por_pagina=20, where=None, order_by=None, columns=None)` |
| `query(db, sql, params=None)` |
| `query_one(db, sql, params=None)` |
| `rollback(db)` |
| `rollback_migration(db, migrations, ate=None)` |
| `savepoint(db, nome, acao)` |
| `schema_sql(db, table=None)` |
| `search(db, table, termo, limit=20, nome=None, columns=None)` |
| `seed(db, table, records)` |
| `select(db, table, where=None, order_by=None, limit=None, columns=None)` |
| `slow_log(db)` |
| `stats(db)` |
| `table_exists(db, name)` |
| `table_info(db, table)` |
| `tables(db)` |
| `transacao(db, acao)` |
| `transaction(db, acao)` |
| `update(db, table, data, where)` |
| `upsert(db, table, data, chaves)` |
| `upsert_many(db, table, records, chaves)` |
| `vacuum(db)` |
| `watch_slow(db, acima_de_ms=50)` |


---

## Arcane.Excel

Planilhas .xlsx: ler, gravar, fórmulas e conversão para CSV e frame.

```dataforge
adopt Arcane.Excel as Excel
```

**Funções (29)**

| Assinatura |
|------------|
| `addr(linha, coluna)` |
| `append(aba, linha_valores)` |
| `autofit(aba)` |
| `bold_row(aba, linha)` |
| `cell(aba, linha, coluna, valor=None)` |
| `col_letter(indice)` |
| `column(livro, nome_aba, coluna)` |
| `dims(livro, nome=None)` |
| `drop_sheet(livro, nome)` |
| `formula(aba, ref, expressao)` |
| `formulas(aba)` |
| `freeze(aba, ref='A2')` |
| `from_csv(caminho, separador=',', nome='Planilha1')` |
| `from_frame(frame, nome='Dados', livro=None)` |
| `get(aba, ref)` |
| `get_formula(aba, ref)` |
| `new()` |
| `parse_addr(ref)` |
| `quick(caminho, dados, nome='Planilha1', cabecalho=None)` |
| `read(caminho)` |
| `records(livro, nome=None)` |
| `rows(livro, nome=None)` |
| `save(livro, caminho)` |
| `set(aba, ref, valor)` |
| `sheet(livro, nome, dados=None, cabecalho=None)` |
| `sheets(livro)` |
| `to_csv(livro, caminho, nome=None, separador=',')` |
| `to_frame(livro, nome=None)` |
| `width(aba, coluna, largura)` |


---

## Arcane.Meta

Metadados de decorador: ler @Nome em tempo de execução.

```dataforge
adopt Arcane.Meta as Meta
```

**Funções (12)**

| Assinatura |
|------------|
| `arg(alvo, nome, indice=0, padrao=None)` |
| `descrever(alvo)` |
| `filtrar(valores, nome)` |
| `ler(alvo, nome)` |
| `limpar(alvo)` |
| `marcar(alvo, nome, *args, **kwargs)` |
| `metodos_com(alvo, nome)` |
| `nomes(alvo)` |
| `opcao(alvo, nome, chave, padrao=None)` |
| `tem(alvo, nome)` |
| `todos(alvo)` |
| `todos_de(alvo, nome)` |


---

## Kiln

Framework web: rotas, middleware, templates, sessão e arquivos estáticos.

```dataforge
adopt Kiln as Kiln
```

**Funções (73)**

| Assinatura |
|------------|
| `Sala(nome='sala')` |
| `after(app, funcao)` |
| `any(app, padrao, handler)` |
| `app(nome='kiln', **config)` |
| `audit(escrever=None, metodos=('POST', 'PUT', 'PATCH', 'DELETE'))` |
| `auditoria(escrever=None, metodos=('POST', 'PUT', 'PATCH', 'DELETE'))` |
| `auth(verificador, esquema='Bearer')` |
| `body_limit(bytes_maximos=1048576)` |
| `buscar(itens, req=None, campos=(), parametro='q')` |
| `cabecalhos_seguros(csp="default-src 'self'", hsts=False, frame='DENY', referrer='strict-origin-when-cross-origin', permissoes='geolocation=(), microphone=(), camera=()')` |
| `cache(segundos=60, privado=False)` |
| `comprimir(minimo=1024)` |
| `conferir(dados, esquema)` |
| `config(app, chave, valor)` |
| `cookie(resp, nome, valor, dias=None, http_only=True, caminho='/', same_site='Lax', seguro=False)` |
| `cors(origens='*', metodos=None, cabecalhos=None)` |
| `csrf(segredo, campo='_csrf', cabecalho='X-CSRF-Token')` |
| `csrf_token(req, segredo=None)` |
| `delete(app, padrao, handler)` |
| `escape(texto)` |
| `evento(dados, tipo='', identificador='', reconectar=0)` |
| `file(caminho, tipo=None, baixar=None)` |
| `forge(nome='kiln', **config)` |
| `get(app, padrao, handler)` |
| `group(app, prefixo, meio=None)` |
| `guard(condicao, status=403, mensagem='sem permissão')` |
| `head(app, padrao, handler)` |
| `header(resp, chave, valor)` |
| `html(texto, status=200, cabecalhos=None)` |
| `idempotente(janela=86400)` |
| `json(dados, status=200, cabecalhos=None)` |
| `limite_de_corpo(bytes_maximos=1048576)` |
| `listen(app, porta=8080, host='127.0.0.1', silencioso=False)` |
| `logger(formato='dev')` |
| `mount(app, prefixo, outro)` |
| `on_error(app, status, handler)` |
| `options(app, padrao, handler)` |
| `ordenar(itens, req=None, campos=None, padrao='')` |
| `paginar(itens, req=None, por_pagina=20, teto=100)` |
| `patch(app, padrao, handler)` |
| `post(app, padrao, handler)` |
| `put(app, padrao, handler)` |
| `rate_limit(maximo=60, janela=60)` |
| `redirect(destino, status=302)` |
| `render(app, nome, dados=None, status=200)` |
| `render_string(texto, dados=None)` |
| `request_id(cabecalho='X-Request-Id')` |
| `resource(app, base, controlador)` |
| `route(app, metodo, padrao, handler)` |
| `routes(app)` |
| `sala(nome='sala')` |
| `salvar_upload(arquivo, pasta, nome=None, limite=0, tipos=None)` |
| `secure_headers(csp="default-src 'self'", hsts=False, frame='DENY', referrer='strict-origin-when-cross-origin', permissoes='geolocation=(), microphone=(), camera=()')` |
| `serve(app, porta=8080, host='127.0.0.1')` |
| `session_end(app, req, resp)` |
| `session_start(app, req, resp, dados=None)` |
| `sign(dados, segredo)` |
| `sse(gerador, cabecalhos=None)` |
| `static(app, prefixo, pasta)` |
| `stats(app)` |
| `status(codigo, mensagem=None)` |
| `stop(app)` |
| `stream(gerador, tipo='text/plain; charset=utf-8', cabecalhos=None)` |
| `templates(app, pasta)` |
| `test(app, metodo, caminho, corpo=None, cabecalhos=None)` |
| `text(texto, status=200, cabecalhos=None)` |
| `unsign(token, segredo)` |
| `upload(req, campo)` |
| `uploads(req)` |
| `use(app, funcao)` |
| `validar(esquema, alvo='body')` |
| `validate(esquema, alvo='body')` |
| `ws(app, padrao, handler)` |


---

## Arcane.Test

Asserções e organização de suítes de teste.

```dataforge
adopt Arcane.Test as Test
```

**Funções (34)**

| Assinatura |
|------------|
| `add_test(suite, name, spec)` |
| `assert_all(collection, predicate, msg=None)` |
| `assert_any(collection, predicate, msg=None)` |
| `assert_between(value, low, high, msg=None)` |
| `assert_close(actual, expected, tolerance=0.001, msg=None)` |
| `assert_contains(collection, item, msg=None)` |
| `assert_deep_eq(a, b, msg=None)` |
| `assert_empty(collection, msg=None)` |
| `assert_eq(actual, expected, msg=None)` |
| `assert_false(value, msg=None)` |
| `assert_greater(a, b, msg=None)` |
| `assert_instance(obj, blueprint_name, msg=None)` |
| `assert_keys(d, *expected_keys, msg=None)` |
| `assert_length(collection, expected, msg=None)` |
| `assert_less(a, b, msg=None)` |
| `assert_match(string, pattern, msg=None)` |
| `assert_neq(actual, expected, msg=None)` |
| `assert_not_contains(collection, item, msg=None)` |
| `assert_not_empty(collection, msg=None)` |
| `assert_not_void(value, msg=None)` |
| `assert_sorted(collection, reverse=False, msg=None)` |
| `assert_throws(func, msg=None)` |
| `assert_true(value, msg=None)` |
| `assert_type(value, expected_type, msg=None)` |
| `assert_unique(collection, msg=None)` |
| `assert_void(value, msg=None)` |
| `benchmark(func, iterations=1000)` |
| `describe(name, tests)` |
| `it(description, test_func)` |
| `mock(return_value=None)` |
| `run(suite, tests)` |
| `run_suite(suite)` |
| `spy(func)` |
| `suite(name='Test Suite')` |


---

## Arcane.Regex

Expressões regulares e validadores brasileiros (CPF, CNPJ, telefone).

```dataforge
adopt Arcane.Regex as Regex
```

**Constantes**

| Nome | Valor |
|------|-------|
| `DOTALL` | `re.DOTALL` |
| `IGNORECASE` | `re.IGNORECASE` |
| `MULTILINE` | `re.MULTILINE` |
| `patterns` | `{'email': '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-z…` |

**Funções (28)**

| Assinatura |
|------------|
| `clean_whitespace(string)` |
| `compile(pattern, flags=0)` |
| `count(pattern, string, flags=0)` |
| `escape(string)` |
| `extract(pattern, string, flags=0)` |
| `extract_emails(string)` |
| `extract_numbers(string)` |
| `extract_urls(string)` |
| `extract_words(string)` |
| `findall(pattern, string, flags=0)` |
| `finditer(pattern, string, flags=0)` |
| `is_cnpj(string)` |
| `is_cpf(string)` |
| `is_date(string)` |
| `is_email(string)` |
| `is_ipv4(string)` |
| `is_phone(string)` |
| `is_url(string)` |
| `mask(string, pattern, mask_char='*')` |
| `match(pattern, string, flags=0)` |
| `remove_html(string)` |
| `replace_all(pattern, repl, string)` |
| `search(pattern, string, flags=0)` |
| `split(pattern, string, maxsplit=0, flags=0)` |
| `sub(pattern, repl, string, count=0, flags=0)` |
| `subn(pattern, repl, string, count=0, flags=0)` |
| `test(pattern, string, flags=0)` |
| `word_count(string)` |


---

## Arcane.IO

Arquivos, diretórios, JSON, CSV e shell.

```dataforge
adopt Arcane.IO as IO
```

**Funções (30)**

| Assinatura |
|------------|
| `abs(path)` |
| `append(path, content)` |
| `basename(path)` |
| `copy(src, dst)` |
| `copy_tree(origem, destino)` |
| `cwd()` |
| `delete(path)` |
| `dirname(path)` |
| `exists(path)` |
| `ext(path)` |
| `file_exists(path)` |
| `join(*parts)` |
| `list_dir(path='.')` |
| `listdir(path='.')` |
| `mkdir(path)` |
| `open(path, mode='r')` |
| `path(path)` |
| `read(path)` |
| `read_csv(path)` |
| `read_file(path)` |
| `read_json(path)` |
| `remove_tree(path)` |
| `rename(old, new)` |
| `rmdir(path)` |
| `shell(command)` |
| `size(path)` |
| `write(path, content)` |
| `write_csv(path, data)` |
| `write_file(path, content)` |
| `write_json(path, data, indent=2)` |


---

## Arcane.Http

Servidor HTTP: rotas, middleware, JSON, arquivos estáticos.

```dataforge
adopt Arcane.Http as Http
```

**Funções (17)**

| Assinatura |
|------------|
| `cors(app)` |
| `create(name='DataForge App')` |
| `delete(app, path, handler)` |
| `get(app, path, handler)` |
| `html_response(html, status=200)` |
| `json_parser(app)` |
| `json_response(data, status=200)` |
| `listen(app, port=3000, host='0.0.0.0')` |
| `logger(app)` |
| `patch(app, path, handler)` |
| `post(app, path, handler)` |
| `put(app, path, handler)` |
| `route(app, method, path, handler)` |
| `static(app, directory)` |
| `stop(app)` |
| `templates(app, directory)` |
| `use(app, middleware)` |


---

## Arcane.Async

Promessas, filas, agendamento e execução concorrente.

```dataforge
adopt Arcane.Async as Async
```

**Funções (46)**

| Assinatura |
|------------|
| `all(promises)` |
| `any(promises)` |
| `batch(fn)` |
| `buffer_op(size)` |
| `catch(promise, callback)` |
| `channel(buffer_size=0)` |
| `combine_latest(*observables)` |
| `computed(signals, fn)` |
| `debounce_op(ms)` |
| `delay(ms, fn=None)` |
| `distinct_op()` |
| `effect(signals, fn)` |
| `emit(emitter, event, *data)` |
| `emit_event(emitter, event, *data)` |
| `emitter()` |
| `event_emitter()` |
| `filter_op(fn)` |
| `from_list(lst)` |
| `interval(ms, count=10)` |
| `map_op(fn)` |
| `merge(*observables)` |
| `observable(producer=None)` |
| `of(*values)` |
| `off(emitter, event, callback=None)` |
| `on(emitter, event, callback)` |
| `once_event(emitter, event, callback)` |
| `parallel(*fns)` |
| `pipe_stream(observable, *operators)` |
| `promise(executor)` |
| `race(promises)` |
| `receive(ch)` |
| `reject(error)` |
| `resolve(value)` |
| `retry_task(fn, max_retries=3, delay_ms=100)` |
| `scan_op(fn, initial)` |
| `send(ch, value)` |
| `sequential(*fns)` |
| `settled(promises)` |
| `signal(initial_value)` |
| `skip_op(n)` |
| `subject()` |
| `subscribe(observable, callback)` |
| `take_op(n)` |
| `task(fn)` |
| `then(promise, callback)` |
| `timeout(ms, fn)` |


---

## Arcane.Data

DataFrames, séries e transformações tabulares.

```dataforge
adopt Arcane.Data as Data
```

**Funções (13)**

| Assinatura |
|------------|
| `Frame(data=None, columns=None)` |
| `correlate(x, y)` |
| `describe(data)` |
| `group_by(data, key_func)` |
| `merge(left, right, on)` |
| `normalize(data, min_val=0, max_val=1)` |
| `one_hot(labels)` |
| `pivot(data, index_col, value_col, agg='sum')` |
| `read_csv(path, header=True)` |
| `read_json(path)` |
| `series(data, name='series')` |
| `split(data, ratio=0.8)` |
| `standardize(data)` |


---

## Arcane.Web

Cliente HTTP, URL encoding e JSON.

```dataforge
adopt Arcane.Web as Web
```

**Funções (11)**

| Assinatura |
|------------|
| `check(user_id=None)` |
| `close()` |
| `decode_url(text)` |
| `encode_url(text)` |
| `get(url, headers=None)` |
| `json_parse(text)` |
| `json_stringify(obj, indent=None)` |
| `post(url, data=None, headers=None)` |
| `request(url, method='GET', data=None, headers=None)` |
| `serve(port=8080, host='0.0.0.0')` |
| `socket(host, port)` |


---

## Arcane.Cortex

Aprendizado de máquina: regressão, árvore, floresta, k-NN, Naive Bayes, k-médias e PCA.

```dataforge
adopt Arcane.Cortex as Cortex
```

**Funções (25)**

| Assinatura |
|------------|
| `acuracia(reais, previstos)` |
| `aplicar_escala(linhas, escala)` |
| `arvore(linhas, alvo, colunas, profundidade=6, minimo=2)` |
| `avaliar(modelo, linhas)` |
| `bayes_texto(linhas, alvo, coluna)` |
| `carregar(caminho)` |
| `categorico(linhas, coluna, prefixo='')` |
| `dividir(linhas, proporcao=0.2, semente=42, estratificar='')` |
| `embaralhar(linhas, semente=42)` |
| `erro(reais, previstos)` |
| `escalonar(linhas, colunas)` |
| `floresta(linhas, alvo, colunas, arvores=20, profundidade=8, minimo=2, semente=42)` |
| `importancia(modelo)` |
| `kmedias(linhas, colunas, grupos=3, voltas=50, semente=42)` |
| `linear(linhas, alvo, colunas)` |
| `logistica(linhas, alvo, colunas, voltas=300, taxa=0.1)` |
| `matriz(modelo, linhas)` |
| `pca(linhas, colunas, componentes=2)` |
| `prever(modelo, linhas)` |
| `prever_um(modelo, linha)` |
| `probabilidade(modelo, linha)` |
| `resumo(modelo)` |
| `salvar(modelo, caminho)` |
| `validacao_cruzada(linhas, alvo, colunas, especie='floresta', dobras=5, semente=42, **extras)` |
| `vizinhos(linhas, alvo, colunas, k=5)` |


---

## Arcane.Time

Datas, horas, durações e cronometragem.

```dataforge
adopt Arcane.Time as Time
```

**Funções (54)**

| Assinatura |
|------------|
| `add_days(d, n)` |
| `add_hours(d, n)` |
| `add_minutes(d, n)` |
| `add_months(d, n)` |
| `add_seconds(d, n)` |
| `add_weeks(d, n)` |
| `add_years(d, n)` |
| `age(nascimento, referencia=None)` |
| `date(ano, mes, dia)` |
| `datetime(ano, mes, dia, hora=0, minuto=0, segundo=0)` |
| `day(d)` |
| `day_of_year(d)` |
| `days_between(a, b)` |
| `days_in_month(a, m)` |
| `diff(a, b)` |
| `duration(dias=0, horas=0, minutos=0, segundos=0)` |
| `end_of_day(d)` |
| `end_of_month(d)` |
| `format(d, formato='%Y-%m-%d %H:%M:%S')` |
| `from_iso(t)` |
| `from_timestamp(ts)` |
| `hour(d)` |
| `humanize(s)` |
| `is_after(a, b)` |
| `is_before(a, b)` |
| `is_leap_year(a)` |
| `is_same_day(a, b)` |
| `is_weekend(d)` |
| `measure(acao)` |
| `minute(d)` |
| `monotonic()` |
| `month(d)` |
| `month_name(d, curto=False)` |
| `now()` |
| `parse(texto, formato=None)` |
| `quarter(d)` |
| `second(d)` |
| `sleep(s)` |
| `start_of_day(d)` |
| `start_of_month(d)` |
| `stopwatch()` |
| `timestamp()` |
| `timezone_offset()` |
| `to_br(d)` |
| `to_date_string(d)` |
| `to_iso(d)` |
| `to_time_string(d)` |
| `to_utc(d)` |
| `today()` |
| `utcnow()` |
| `week_of_year(d)` |
| `weekday(d)` |
| `weekday_name(d, curto=False)` |
| `year(d)` |


---

## Arcane.OS

Sistema operacional, ambiente, disco e processo atual.

```dataforge
adopt Arcane.OS as OS
```

**Funções (42)**

| Assinatura |
|------------|
| `arch()` |
| `argv()` |
| `argv_completo()` |
| `beside(*partes)` |
| `chdir(caminho)` |
| `cpu_count()` |
| `cwd()` |
| `disk_usage(caminho='.')` |
| `env()` |
| `env_names()` |
| `executable()` |
| `exit(codigo=0)` |
| `get_env(nome, padrao=None)` |
| `has_env(nome)` |
| `home()` |
| `hostname()` |
| `info()` |
| `is_linux()` |
| `is_mac()` |
| `is_posix()` |
| `is_tty()` |
| `is_windows()` |
| `line_separator()` |
| `machine()` |
| `memory_info()` |
| `name()` |
| `parent_pid()` |
| `path_separator()` |
| `pid()` |
| `platform()` |
| `processor()` |
| `python_version()` |
| `release()` |
| `script()` |
| `script_dir()` |
| `separator()` |
| `set_env(nome, valor)` |
| `temp_dir()` |
| `terminal_size()` |
| `user()` |
| `version()` |
| `which(prog)` |


---

## Arcane.Process

Execução de processos externos, com stdout, stderr e código de saída.

```dataforge
adopt Arcane.Process as Process
```

**Funções (15)**

| Assinatura |
|------------|
| `capture(comando, shell=False, timeout=None)` |
| `check(comando, shell=False, timeout=None)` |
| `exists(prog)` |
| `exit_code(comando, shell=False, timeout=None)` |
| `is_running(processo)` |
| `kill(processo)` |
| `pid()` |
| `pipeline(comandos, timeout=None)` |
| `python()` |
| `run(comando, shell=False, timeout=None, cwd=None, env=None, input_text=None)` |
| `run_shell(cmd, timeout=None)` |
| `spawn(comando, shell=False, cwd=None)` |
| `terminate(processo)` |
| `wait(processo, timeout=None)` |
| `which(programa)` |


---

## Arcane.Logging

Registro estruturado de eventos, com níveis e destinos.

```dataforge
adopt Arcane.Logging as Logging
```

**Funções (14)**

| Assinatura |
|------------|
| `as_json(a=True)` |
| `debug(m, campos=None)` |
| `default()` |
| `error(m, campos=None)` |
| `fatal(m, campos=None)` |
| `info(m, campos=None)` |
| `levels()` |
| `log(n, m, campos=None)` |
| `logger(nome='app', nivel='INFO')` |
| `set_level(n)` |
| `stats()` |
| `to_file(c, anexar=True)` |
| `trace(m, campos=None)` |
| `warn(m, campos=None)` |


---

## Arcane.Crypto

Hashes, HMAC, senhas, codificações, aleatoriedade segura e cifragem de arquivo (ChaCha20-Poly1305).

```dataforge
adopt Arcane.Crypto as Crypto
```

**Funções (48)**

| Assinatura |
|------------|
| `algorithms()` |
| `apagar_seguro(caminho, passadas=1)` |
| `base32_decode(v)` |
| `base32_encode(v)` |
| `base64_decode(texto)` |
| `base64_encode(v)` |
| `base64url_decode(texto)` |
| `base64url_encode(v)` |
| `blake2b(v)` |
| `blake2s(v)` |
| `chave_nova(tamanho=32)` |
| `cifrar(dados, senha, iteracoes=None)` |
| `cifrar_arquivo(origem, destino='', senha='', iteracoes=None)` |
| `cifrar_pasta(pasta, destino='', senha='')` |
| `constant_time_equals(a, b)` |
| `decifrar(pacote, senha)` |
| `decifrar_arquivo(origem, destino='', senha='')` |
| `derivar_chave(senha, sal='', iteracoes=None)` |
| `e_cifrado(caminho)` |
| `hash(valor, algoritmo='sha256')` |
| `hash_file(caminho, algoritmo='sha256')` |
| `hash_password(senha, iteracoes=200000)` |
| `hex_decode(v)` |
| `hex_encode(v)` |
| `hmac(chave, mensagem, algoritmo='sha256')` |
| `hmac_verify(chave, mensagem, assinatura, algoritmo='sha256')` |
| `informacao_do_cofre(caminho)` |
| `mask(texto, visiveis=4, caractere='*')` |
| `md5(v)` |
| `pbkdf2(senha, sal, iteracoes=200000, algoritmo='sha256')` |
| `random_bytes(n=32)` |
| `random_choice(itens)` |
| `random_hex(n=32)` |
| `random_int(a, b)` |
| `random_password(tamanho=16, simbolos=True)` |
| `random_token(n=32)` |
| `rot13(t)` |
| `sha1(v)` |
| `sha224(v)` |
| `sha256(v)` |
| `sha384(v)` |
| `sha512(v)` |
| `short_id(n=12)` |
| `uuid()` |
| `uuid4()` |
| `uuid_hex()` |
| `verify_password(senha, guardada)` |
| `xor_cipher(texto, chave)` |


---

## Arcane.Collections

Estruturas de dados e algoritmos: pilha, fila, grafo, união-busca.

```dataforge
adopt Arcane.Collections as Collections
```

**Funções (63)**

| Assinatura |
|------------|
| `add(conjunto, item)` |
| `batched(itens, n)` |
| `binary_search(ordenado, alvo)` |
| `bottom_n(itens, n, chave=None)` |
| `cartesian(a, b)` |
| `chain_vaults(*vaults)` |
| `chunk_evenly(itens, partes)` |
| `counter(itens=None)` |
| `deep_merge(a, b)` |
| `default_vault(padrao=None)` |
| `deque(itens=None, maximo=0)` |
| `difference(a, b)` |
| `discard(conjunto, item)` |
| `elements(contagem)` |
| `extend_left(fila, itens)` |
| `first_key(v)` |
| `flatten_deep(itens, profundidade=-1)` |
| `frozen(itens)` |
| `graph(dirigido=False)` |
| `group(itens, chave)` |
| `group_by(itens, chave)` |
| `heap(itens=None)` |
| `heap_peek(h)` |
| `heap_pop(h)` |
| `heap_push(h, item)` |
| `index_by(itens, chave)` |
| `intersection(a, b)` |
| `is_disjoint(a, b)` |
| `is_subset(a, b)` |
| `is_superset(a, b)` |
| `last_key(v)` |
| `merge_sorted(a, b)` |
| `most_common(contagem, n=0)` |
| `move_to_end(v, chave, para_o_fim=True)` |
| `n_largest(itens, n, chave=None)` |
| `n_smallest(itens, n, chave=None)` |
| `named(nome, campos, valores)` |
| `ordered(pares=None)` |
| `ordered_vault(pares=None)` |
| `pairwise(itens)` |
| `partition(itens, predicado)` |
| `peek(fila)` |
| `peek_left(fila)` |
| `pop(fila)` |
| `pop_left(fila)` |
| `priority_queue()` |
| `push(fila, item)` |
| `push_left(fila, item)` |
| `queue(itens=None)` |
| `rotate(fila, n=1)` |
| `set(itens=None)` |
| `sliding_window(itens, tamanho)` |
| `sort_by(itens, chave)` |
| `sort_by_field(itens, campo, reverso=False)` |
| `stack(itens=None)` |
| `subtract(a, b)` |
| `symmetric_difference(a, b)` |
| `top_n(itens, n, chave=None)` |
| `total(contagem)` |
| `union(a, b)` |
| `union_find(itens=None)` |
| `unique_by(itens, chave)` |
| `zip_longest(a, b, preencher=None)` |


---

## Arcane.Serialization

JSON, CSV, INI, TOML, XML e conversões entre eles.

```dataforge
adopt Arcane.Serialization as Serialization
```

**Funções (26)**

| Assinatura |
|------------|
| `csv_to_records(texto, delimitador=',')` |
| `deep_copy(d)` |
| `flatten(dados, separador='.', prefixo='')` |
| `formats()` |
| `from_base64(t)` |
| `from_bytes(b)` |
| `from_csv(texto, delimitador=',', tem_cabecalho=False)` |
| `from_ini(texto)` |
| `from_json(texto)` |
| `from_json_lines(texto)` |
| `from_json_safe(texto, padrao=None)` |
| `from_toml(texto)` |
| `from_xml(texto)` |
| `is_valid_json(texto)` |
| `json_lines(registros)` |
| `json_path(dados, caminho, padrao=None)` |
| `json_pretty(d, indent=2)` |
| `records_to_csv(registros, delimitador=',')` |
| `to_base64(d)` |
| `to_bytes(d)` |
| `to_csv(linhas, delimitador=',', cabecalho=None)` |
| `to_ini(dados)` |
| `to_json(dados, indent=None, ordenar=False)` |
| `to_toml(dados)` |
| `to_xml(dados, raiz='root')` |
| `unflatten(plano, separador='.')` |


---

## Arcane.Forge

Banco de dados: SQLite, Postgres, MySQL, Redis e MongoDB pela mesma interface.

```dataforge
adopt Arcane.Forge as Forge
```

**Funções (28)**

| Assinatura |
|------------|
| `buscar_modelo(nome)` |
| `colunas(db, t)` |
| `conectar(url, **opcoes)` |
| `conexao(pool)` |
| `consultar(conexao, sql, parametros=None)` |
| `de(conexao, tabela)` |
| `de_csv(caminho_ou_texto)` |
| `dialetos()` |
| `executar(conexao, sql, parametros=None)` |
| `fechar(db)` |
| `ligar(modelo, conexao)` |
| `limpar_modelos()` |
| `memoria()` |
| `migracoes(conexao)` |
| `migrar_tudo(conexao)` |
| `modelo(nome, campos=None, opcoes=None)` |
| `modelos()` |
| `motores()` |
| `para_csv(linhas, caminho='')` |
| `ping(db)` |
| `pool(url, tamanho=5, **o)` |
| `primeiro(conexao, sql, parametros=None)` |
| `tabela(conexao, tabela)` |
| `tabelas(db)` |
| `tipos()` |
| `transacao(conexao, corpo)` |
| `url(url)` |
| `versao(db)` |


---

## Arcane.Crucible

Framework de testes: suítes, matchers, fixtures, dublês e benchmark.

```dataforge
adopt Arcane.Crucible as Crucible
```

**Funções (50)**

| Assinatura |
|------------|
| `after(corpo)` |
| `after_all(corpo)` |
| `approx(valor, casas=7)` |
| `banco(db)` |
| `before(corpo)` |
| `before_all(corpo)` |
| `benchmark(nome, acao, vezes=1000, aquecimento=10)` |
| `booleans()` |
| `capture(acao)` |
| `check(condicao, mensagem='a condicao nao se cumpriu')` |
| `clusters(item=None, tamanho_max=10)` |
| `database(db)` |
| `describe(nome, corpo=None)` |
| `diff(esperado, obtido)` |
| `expect(valor, rotulo='')` |
| `fail(mensagem='falhou por decisao do teste')` |
| `fixture(nome, corpo)` |
| `flaky(acao, tentativas=3, espera=0.0)` |
| `floats(minimo=-1000.0, maximo=1000.0)` |
| `forall(gerador, propriedade, casos=100, semente=None)` |
| `freeze_time(instante)` |
| `instantaneo(nome, valor, atualizar=None)` |
| `instavel(acao, tentativas=3, espera=0.0)` |
| `integers(minimo=-1000, maximo=1000)` |
| `json()` |
| `junit()` |
| `mock(nome='mock', alvo=None)` |
| `one_of(valores)` |
| `only(nome, corpo, tags=None)` |
| `pending(nome, motivo='', corpo=None)` |
| `report(colorir=True, verboso=False)` |
| `reset()` |
| `results()` |
| `run(opcoes=None)` |
| `snapshot(nome, valor, atualizar=None)` |
| `snapshot_dir(arquivo)` |
| `spy(alvo, nome='spy')` |
| `stub(respostas=None, nome='stub')` |
| `suite(nome, corpo=None)` |
| `summary()` |
| `table(nome, casos, corpo, tags=None)` |
| `tag(*nomes)` |
| `tap()` |
| `temp_dir()` |
| `temp_file(conteudo='', sufixo='.txt')` |
| `test(nome, corpo, tags=None, prazo=0, repetir=1, dados=None)` |
| `texts(tamanho_max=20, alfabeto=None)` |
| `timed(acao, vezes=1)` |
| `trial(nome, corpo, tags=None, prazo=0, repetir=1, dados=None)` |
| `vaults(valor=None, tamanho_max=6)` |


---

## Arcane.Iter

Iteradores preguiçosos e composição de ações: janelas, combinatória, memoize.

```dataforge
adopt Arcane.Iter as Iter
```

**Funções (44)**

| Assinatura |
|------------|
| `accumulate(fonte, funcao=None, inicial=None)` |
| `attr(nome)` |
| `batched(fonte, tamanho)` |
| `cache_info(memoizada)` |
| `chain(*fontes)` |
| `chunk(fonte, tamanho)` |
| `combinations(fonte, tamanho)` |
| `combinations_with_repetition(fonte, tamanho)` |
| `compose(*acoes)` |
| `compress(fonte, marcas)` |
| `constant(valor)` |
| `count(inicio=0, passo=1)` |
| `curry(funcao, aridade=2)` |
| `cycle_forever(itens)` |
| `drop(fonte, n)` |
| `drop_while(fonte, condicao)` |
| `filter_false(fonte, condicao)` |
| `flat_map(fonte, funcao)` |
| `flatten(fonte, profundidade=1)` |
| `flip(funcao)` |
| `group_runs(fonte, chave=None)` |
| `identity(x)` |
| `item(indice)` |
| `memoize(funcao, tamanho=128)` |
| `once(funcao)` |
| `op(simbolo)` |
| `pairwise(fonte)` |
| `partial(funcao, *fixos, **nomeados)` |
| `permutations(fonte, tamanho=0)` |
| `pipe(*acoes)` |
| `powerset(fonte)` |
| `product(*fontes, repetir=1)` |
| `reduce(fonte, funcao, inicial=None)` |
| `repeat(valor, vezes=0)` |
| `running_max(fonte)` |
| `running_sum(fonte)` |
| `slice(fonte, inicio, fim=None, passo=1)` |
| `take(fonte, n)` |
| `take_while(fonte, condicao)` |
| `to_cluster(fonte)` |
| `unique(fonte)` |
| `unique_by(fonte, chave)` |
| `window(fonte, tamanho)` |
| `zip_longest(a, b, preencher=None)` |


---

## Arcane.Color

Cor de 24 bits no terminal, tabela, moldura, barra de progresso e árvore.

```dataforge
adopt Arcane.Color as Color
```

**Funções (66)**

| Assinatura |
|------------|
| `auto()` |
| `badge(texto, cor='blue')` |
| `bar(valor, total, largura=30, cor='green', mostrar_numero=True)` |
| `bg_rgb(texto, r, g, b)` |
| `black(texto)` |
| `blink(texto)` |
| `blue(texto)` |
| `bold(texto)` |
| `box(texto, titulo='', cor='cyan', largura=0)` |
| `bright_blue(texto)` |
| `bright_cyan(texto)` |
| `bright_green(texto)` |
| `bright_magenta(texto)` |
| `bright_red(texto)` |
| `bright_white(texto)` |
| `bright_yellow(texto)` |
| `cyan(texto)` |
| `dim(texto)` |
| `error(texto)` |
| `force(ligado=True)` |
| `gradient(texto, de, para)` |
| `gray(texto)` |
| `green(texto)` |
| `grey(texto)` |
| `hex(texto, cor)` |
| `hidden(texto)` |
| `info()` |
| `info_msg(texto)` |
| `italic(texto)` |
| `magenta(texto)` |
| `muted(texto)` |
| `on_black(texto)` |
| `on_blue(texto)` |
| `on_bright_blue(texto)` |
| `on_bright_cyan(texto)` |
| `on_bright_green(texto)` |
| `on_bright_magenta(texto)` |
| `on_bright_red(texto)` |
| `on_bright_white(texto)` |
| `on_bright_yellow(texto)` |
| `on_cyan(texto)` |
| `on_gray(texto)` |
| `on_green(texto)` |
| `on_grey(texto)` |
| `on_magenta(texto)` |
| `on_red(texto)` |
| `on_white(texto)` |
| `on_yellow(texto)` |
| `paint(texto, *nomes)` |
| `rainbow(texto)` |
| `red(texto)` |
| `reverse(texto)` |
| `rgb(texto, r, g, b)` |
| `rule(titulo='', cor='gray', largura=0)` |
| `spinner_frames(estilo='pontos')` |
| `strike(texto)` |
| `strip(texto)` |
| `success(texto)` |
| `supports()` |
| `table(linhas, cabecalho=True, cor='cyan')` |
| `tree(no, prefixo='', ultimo=True)` |
| `underline(texto)` |
| `warning(texto)` |
| `white(texto)` |
| `width(padrao=80)` |
| `yellow(texto)` |


---

## Arcane.Concurrent

Threads, processos, canal bloqueante, grupo de tarefas e prazo.

```dataforge
adopt Arcane.Concurrent as Concurrent
```

**Funções (25)**

| Assinatura |
|------------|
| `barreira(quantas)` |
| `canal(capacidade=0)` |
| `com_prazo(acao, segundos)` |
| `com_trava(trava, acao)` |
| `condicao()` |
| `contador(inicial=0)` |
| `dormir(segundos)` |
| `esperar(tarefa, prazo=None)` |
| `esperar_primeira(tarefas, prazo=None)` |
| `esperar_todas(tarefas, prazo=None)` |
| `evento()` |
| `grupo(trabalhadores=None, nome='')` |
| `lotes(acao, itens, tamanho=10, trabalhadores=None)` |
| `map(acao, itens, trabalhadores=None, prazo=None)` |
| `map_processos(acao, itens, trabalhadores=None)` |
| `mutex()` |
| `nucleos()` |
| `para_cada(acao, itens, trabalhadores=None)` |
| `repetir_a_cada(acao, segundos, vezes=0)` |
| `rodar(acao, *args)` |
| `semaforo(quantos=1)` |
| `sou_principal()` |
| `thread_atual()` |
| `threads_vivas()` |
| `trava_leitura_escrita()` |


---

## Arcane.Archive

Zip e tar: compactar, listar, conferir e extrair recusando Zip Slip e zip bomb.

```dataforge
adopt Arcane.Archive as Archive
```

**Funções (8)**

| Assinatura |
|------------|
| `acrescentar(arquivo, caminho, nome='')` |
| `compactar(origem, destino, nivel=6)` |
| `compactar_tar(origem, destino, compressao='gz')` |
| `conferir(arquivo)` |
| `extrair(arquivo, destino='.', senha='')` |
| `extrair_tar(arquivo, destino='.')` |
| `ler_de(arquivo, nome, senha='')` |
| `listar(arquivo)` |


---

## Arcane.Pipeline

Orquestração de ETL/ELT: DAG, dependências, retry, incremental e relatório.

```dataforge
adopt Arcane.Pipeline as Pipeline
```

**Funções (11)**

| Assinatura |
|------------|
| `esquecer_marca(fluxo, chave='')` |
| `etapa(fluxo, nome, acao, depende_de=None, tentativas=1, espera=0, quando=None, opcional=False, descricao='')` |
| `fluxo(nome, estado='')` |
| `grafico(fluxo)` |
| `historico(fluxo, quantos=10)` |
| `marca(fluxo, chave, padrao=None)` |
| `marcar(fluxo, chave, valor)` |
| `ordem(fluxo)` |
| `rodar(fluxo, contexto=None, ate=None)` |
| `rodar_ate(fluxo, etapa, contexto=None)` |
| `ultima_execucao(fluxo)` |


---

## Arcane.Stream

Streaming: tópicos, partições, offsets, grupos de consumo e janelas de tempo.

```dataforge
adopt Arcane.Stream as Stream
```

**Funções (17)**

| Assinatura |
|------------|
| `atraso(corrente, topico, grupo)` |
| `confirmar(corrente, topico, grupo, evento)` |
| `confirmar_ate(corrente, topico, grupo, eventos)` |
| `consumir(corrente, topico, grupo, quantos=0, particao=None)` |
| `corrente(raiz)` |
| `grupos(corrente, topico)` |
| `informacao(corrente, topico)` |
| `janela(eventos, segundos=60, campo='quando')` |
| `ler_de(corrente, topico, desde=0, quantos=0, particao=None)` |
| `offset(corrente, topico, grupo)` |
| `publicar(corrente, topico, valor, chave=None)` |
| `publicar_lote(corrente, topico, eventos)` |
| `remover_topico(corrente, nome)` |
| `reter(corrente, topico, segmentos=10)` |
| `topico(corrente, nome, particoes=1)` |
| `topicos(corrente)` |
| `voltar(corrente, topico, grupo, para=0)` |


---

## Arcane.Observar

Observabilidade: métricas com percentil, tracing aninhado e linhagem de dados.

```dataforge
adopt Arcane.Observar as Observar
```

**Funções (19)**

| Assinatura |
|------------|
| `abrir(painel, nome, dentro_de=None)` |
| `alertar(painel, regras)` |
| `arvore(painel)` |
| `contar(painel, nome, quanto=1)` |
| `cronometrar(painel, nome, acao)` |
| `derivar(painel, saida, entradas, como='')` |
| `fechar(painel, ident, estado='ok', detalhe=None)` |
| `grafo(painel)` |
| `impacto(painel, nome)` |
| `marcar(painel, nome, valor)` |
| `medir(painel, nome, valor)` |
| `origem(painel, nome, profundidade=20)` |
| `painel(nome, versao='', arquivo='')` |
| `prometheus(painel)` |
| `relatorio(painel)` |
| `resumo(painel)` |
| `salvar(painel, caminho='')` |
| `trechos(painel)` |
| `valor(painel, nome)` |


---

## Arcane.Lago

Data Lake: Parquet nativo, partições Hive, camadas bronze/prata/ouro e compactação.

```dataforge
adopt Arcane.Lago as Lago
```

**Funções (18)**

| Assinatura |
|------------|
| `acrescentar(lago, tabela, linhas, particoes=None, compressao='gzip')` |
| `arquivos(lago, tabela, filtro=None)` |
| `camada(lago, nome)` |
| `compactar(lago, tabela, minimo=2)` |
| `esquema(lago, tabela)` |
| `esquema_parquet(caminho)` |
| `eventos(lago, quantos=20)` |
| `gravar(lago, tabela, linhas, particoes=None, compressao='gzip')` |
| `gravar_parquet(caminho, linhas, compressao='gzip')` |
| `lago(raiz)` |
| `ler(lago, tabela, filtro=None, colunas=None, limite=0)` |
| `ler_parquet(caminho, colunas=None)` |
| `particoes(lago, tabela)` |
| `promover(lago, tabela, de, para, transformar=None, particoes=None)` |
| `remover_particao(lago, tabela, filtro)` |
| `tabelas(lago)` |
| `tamanho(lago, tabela='')` |
| `vacuo(lago)` |


---

## Arcane.Malha

Chamada entre serviços que não mente: cliente HTTP com prazo, retry com recuo e tremor, disjuntor de três estados, descoberta por nome e propagação automática do rastro do pedido.

```dataforge
adopt Arcane.Malha as Malha
```

**Constantes**

| Nome | Valor |
|------|-------|
| `RETENTAVEIS` | `[408, 425, 429, 500, 502, 503, 504]` |

**Funções (22)**

| Assinatura |
|------------|
| `Cliente(base, opcoes=None)` |
| `Disjuntor(falhas=5, espera=30.0, nome='')` |
| `Passo(nome, fazer, desfazer=None, escreve=True, chave=None)` |
| `Saga(nome='saga', identificador=None, registro=None)` |
| `cabecalhos_de_contexto()` |
| `cliente(base, opcoes=None)` |
| `comecar_contexto(rastro=None, origem='', extra=None)` |
| `contexto()` |
| `de(nome, opcoes=None)` |
| `disjuntor(falhas=5, espera=30.0, nome='')` |
| `onde(nome)` |
| `padrao(opcoes)` |
| `propagar(req, origem='')` |
| `rastro()` |
| `recuo(tentativa, base=0.2, teto=10.0, tremor=True)` |
| `registrar(nome, base, opcoes=None)` |
| `resumo()` |
| `saga(nome='saga', identificador=None, registro=None)` |
| `saude()` |
| `servicos()` |
| `terminar_contexto()` |
| `vale_repetir(resposta)` |


---

## Arcane.Vitrine

O framework de dashboards e aplicações de dados: você escreve um programa de cima para baixo e ele vira uma página web, com componentes, layout, gráficos em SVG, estado por sessão e cache — servido pelo Kiln.

```dataforge
adopt Arcane.Vitrine as Vitrine
```

**Constantes**

| Nome | Valor |
|------|-------|
| `estado` | `<dataforge.stdlib.vitrine.estado.Estado object at …` |
| `geral` | `<dataforge.stdlib.vitrine.estado.Geral object at 0…` |
| `i18n` | `<dataforge.stdlib.vitrine.extras.Traducao object a…` |
| `paleta` | `['#FED403', '#0F62FE', '#24A148', '#FA4D56', '#8A3…` |

**Funções (109)**

| Assinatura |
|------------|
| `abas(rotulos)` |
| `agendar(acao, a_cada, *args)` |
| `antes(funcao)` |
| `app(titulo='Vitrine', **config)` |
| `area_de_texto(rotulo, valor='', linhas=4, dica='', chave=None)` |
| `arquivo(rotulo, tipos=None, varios=False, chave=None)` |
| `atualizar_a_cada(segundos)` |
| `audio(origem, formato='audio/mpeg')` |
| `autenticacao(verificador, papeis=None)` |
| `autenticado()` |
| `aviso(mensagem)` |
| `baixar(rotulo, conteudo, nome='dados.txt', tipo='text/plain')` |
| `botao(rotulo, tipo='primario', chave=None, largura='')` |
| `cabecalho(conteudo, nivel=3)` |
| `cache(*args, **kwargs)` |
| `caixa(rotulo, valor=False, chave=None)` |
| `caminho()` |
| `campo_validado(rotulo, regra, mensagem='', valor='', tipo='texto', dica='', chave=None)` |
| `carregando(mensagem='Carregando…')` |
| `cartao(titulo='', subtitulo='')` |
| `codigo(conteudo, linguagem='dataforge')` |
| `colunas(quantidade, larguras=None, espacamento='medio')` |
| `componente(nome, acao=None)` |
| `componentes()` |
| `configurar(chave=None, valor=None, **pares)` |
| `configurar_pagina(titulo='', icone='', **pares)` |
| `container(borda=False, altura=None)` |
| `cor(rotulo, valor='#FED403', chave=None)` |
| `data(rotulo, valor='', chave=None)` |
| `depois(funcao)` |
| `desenhar(g)` |
| `deslizante(rotulo, minimo=0, maximo=100, valor=None, passo=1, chave=None)` |
| `divisor()` |
| `encerrar_sessao()` |
| `entrada(rotulo, valor='', dica='', tipo='texto', chave=None)` |
| `entrar(usuario, senha)` |
| `erro(mensagem)` |
| `escolha(rotulo, opcoes, indice=0, chave=None)` |
| `escolhas(rotulo, opcoes, padrao=None, chave=None)` |
| `espacador()` |
| `espaco(altura=16)` |
| `exigir_login(mensagem='Entre para continuar.')` |
| `exigir_permissao(permissao, mensagem='')` |
| `expandir(rotulo, aberto=False)` |
| `exportar_csv(dados, nome='dados.csv', rotulo='Baixar CSV', separador=',')` |
| `exportar_json(dados, nome='dados.json', rotulo='Baixar JSON')` |
| `formulario(nome, limpar=False)` |
| `frame(dados, colunas=None, altura=None)` |
| `grafico(tipo='linha', dados=None)` |
| `grafico_area(dados, x='', y='', titulo='', altura=None, **kw)` |
| `grafico_barras(dados, x='', y='', titulo='', altura=None, **kw)` |
| `grafico_barras_h(dados, x='', y='', titulo='', altura=None, **kw)` |
| `grafico_dispersao(dados, x='', y='', titulo='', altura=None, **kw)` |
| `grafico_linha(dados, x='', y='', titulo='', altura=None, **kw)` |
| `grafico_pizza(dados, x='', y='', titulo='', altura=None, **kw)` |
| `grafico_rosca(dados, x='', y='', titulo='', altura=None, **kw)` |
| `histograma(dados, campo='', faixas=10, titulo='', altura=None)` |
| `html(conteudo)` |
| `html_da_pagina()` |
| `imagem(origem, legenda='', largura=None)` |
| `informacao(mensagem)` |
| `interruptor(rotulo, valor=False, chave=None)` |
| `json(dados, expandido=True)` |
| `lateral()` |
| `linha(alinhar='inicio', espacamento='medio')` |
| `link(rotulo, destino, nova_aba=False)` |
| `logs(quantos=100, nivel='')` |
| `markdown(conteudo)` |
| `markdown_para_html(texto)` |
| `menu(rotulo='Páginas')` |
| `metrica(rotulo, valor, variacao=None, ajuda='')` |
| `metricas()` |
| `montar()` |
| `navegar(destino)` |
| `numero(rotulo, valor=0, minimo=None, maximo=None, passo=1, chave=None)` |
| `opcao(rotulo, opcoes, indice=0, chave=None)` |
| `pagina(caminho, acao=None, titulo='', icone='', oculta=False)` |
| `paginas()` |
| `parametro(nome, padrao='')` |
| `parametros()` |
| `parar()` |
| `parar_servidor()` |
| `pedir(app, metodo, caminho, corpo=None, cabecalhos=None)` |
| `plugin(nome, instalar)` |
| `pode(permissao)` |
| `progresso(fracao, rotulo='')` |
| `recarregar()` |
| `registrar(mensagem, nivel='info', extra=None)` |
| `rodar(acao=None, porta=8501, host='127.0.0.1', recarregar=False, silencioso=False)` |
| `sair()` |
| `saude()` |
| `servir(porta=0, host='127.0.0.1')` |
| `sessoes()` |
| `subir(porta=8501, host='127.0.0.1', recarregar=False, silencioso=False)` |
| `subtitulo(conteudo)` |
| `sucesso(mensagem)` |
| `t(chave, **valores)` |
| `tabela(dados, colunas=None, altura=None)` |
| `tarefa(acao, *args)` |
| `testar(pagina_ou_app, caminho='/')` |
| `texto(*partes)` |
| `titulo(conteudo, icone='')` |
| `traduzir(chave, **valores)` |
| `usar(nome, *args, **kwargs)` |
| `usuario()` |
| `validar(valor, regra, mensagem='')` |
| `vault(dados)` |
| `vazio()` |
| `video(origem, formato='video/mp4')` |


---

## Arcane.API

A API do Kiln vista de fora: OpenAPI, coleção do Insomnia e do Postman, curl e a tabela em Markdown — tudo derivado das rotas registradas.

```dataforge
adopt Arcane.API as API
```

**Funções (7)**

| Assinatura |
|------------|
| `curl(app, config=None)` |
| `insomnia(app, config=None)` |
| `markdown(app, config=None)` |
| `openapi(app, config=None)` |
| `postman(app, config=None)` |
| `resumo(app)` |
| `rotas(app)` |


---

## Arcane.Decimal

Número decimal exato, para quando 0,1 + 0,2 precisa dar 0,3 — dinheiro, imposto, e todo número que alguém confere na mão.

```dataforge
adopt Arcane.Decimal as Decimal
```

**Funções (16)**

| Assinatura |
|------------|
| `abs(d)` |
| `arredondar(valor, casas=0, modo='MEIO_PARA_CIMA')` |
| `casas(valor)` |
| `centavos(valor)` |
| `de(valor)` |
| `de_centavos(centavos)` |
| `e_decimal(x)` |
| `float(valor)` |
| `inteiro(valor)` |
| `media(valores)` |
| `modos()` |
| `repartir(valor, partes, casas=2)` |
| `sinal(d)` |
| `soma(valores)` |
| `texto(valor, casas=None)` |
| `zero()` |


---

## Arcane.Ponte

A ponte para o Python: perguntar se um pacote existe, explorar o que ele oferece e converter o que ele devolve.

```dataforge
adopt Arcane.Ponte as Ponte
```

**Funções (12)**

| Assinatura |
|------------|
| `assinatura(valor)` |
| `atributos(valor)` |
| `chamavel(x)` |
| `cluster(valor)` |
| `doc(valor)` |
| `empacotado() -> bool` |
| `importar(nome)` |
| `onde() -> str` |
| `tem(nome)` |
| `tipo(valor) -> str` |
| `vault(valor)` |
| `versao(nome)` |


---

## Arcane.Qualidade

Qualidade de dados: as seis dimensões, perfil, validação e limpeza.

```dataforge
adopt Arcane.Qualidade as Qualidade
```

**Funções (13)**

| Assinatura |
|------------|
| `atualidade(linhas, campo, dias=1)` |
| `completude(linhas, campos=None)` |
| `conferir(linhas, regras, parar_em=0)` |
| `duplicadas(linhas, campos)` |
| `esperar(linhas, regras, minimo=1.0)` |
| `fora_da_faixa(linhas, campo, minimo=None, maximo=None)` |
| `formatos()` |
| `perfil(linhas, amostra=0)` |
| `preencher(linhas, padroes)` |
| `relatorio(resultado, largura=72)` |
| `sem_duplicadas(linhas, campos=None)` |
| `so_validas(linhas, regras)` |
| `unicidade(linhas, campo)` |
