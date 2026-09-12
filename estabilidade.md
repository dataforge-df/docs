# O que pode quebrar, e quando

DataForge está em **1.0.0**. Este documento diz o que você pode
construir em cima sem medo, e o que ainda vai mudar.

Ele existe porque uma linguagem sem essa resposta não é uma linguagem em
que alguém investe tempo — e porque prometer estabilidade em prosa não
impede ninguém de renomear um símbolo. Aqui a promessa é **verificada**:
`doc/superficie.json` guarda a lista de tudo o que é público, e
`tests/test_estabilidade.py` falha quando alguma coisa some dela.

---

## A regra, em uma linha

**Acrescentar é livre. Tirar e renomear exigem uma versão maior.**

Um símbolo novo não quebra ninguém. Um símbolo que some quebra todo
programa que o usava.

## O que a versão significa

| | Quer dizer | Exemplo |
|---|---|---|
| **1.x.y → 2.0.0** | um programa válido pode parar de funcionar | uma palavra reservada some; um módulo muda de nome |
| **1.2.0 → 1.3.0** | há capacidade nova, e o que existia continua | um módulo novo; um parâmetro opcional novo |
| **1.2.3 → 1.2.4** | correção, sem interface nova | um bug de arredondamento; uma mensagem melhor |

## O que está coberto

Tudo o que aparece em `doc/superficie.json`:

- as **81 palavras reservadas** e a gramática que elas formam
- as **228 funções embutidas** — nome e ordem dos parâmetros
- os **39 módulos** `Arcane.*`, seus **apelidos** (`Zip`, `Cor`, `Banco`)
  e cada símbolo público deles
- os **57 comandos** da CLI e suas opções
- os **177 códigos de erro** (`DF0101` e companhia) — o código, não o texto
- o formato do `forge.toml` e do `forge.lock`

## O que **não** está coberto

Estas coisas mudam em qualquer versão, e depender delas é por sua conta:

- **O texto das mensagens de erro.** O código (`DF0301`) é estável; a
  frase melhora. Case pelo código, nunca pela mensagem.
- **A representação em texto de um valor.** `out` de um record ou de uma
  instância pode mudar de formato.
- **Qualquer coisa dentro de `dataforge/`.** As classes do interpretador,
  a AST, o compilador de fechamentos — são detalhe de implementação, e
  este projeto os reescreve quando medir que vale a pena.
- **Desempenho.** Tempo e memória são resultado de medição, não promessa.
- **A ordem de saída** onde ela não foi declarada — `Ponte.atributos`
  ordena; um `Vault` preserva inserção; o resto não promete nada.
- **Módulos marcados como experimentais** na própria documentação.

## A ponte para o Python é um caso à parte

`adopt Python.numpy` depende de um pacote que **não é nosso**. A
estabilidade ali é a do pacote que você escolheu, e a promessa deste
documento cobre apenas a ponte em si: a sintaxe, `Arcane.Ponte`, e o
fato de que valores atravessam sem conversão.

## Como uma remoção acontece

Quando tirar for mesmo o certo:

1. o símbolo ganha um aviso do `lint` numa versão menor, dizendo o que
   usar no lugar;
2. ele continua funcionando por **toda** a série `1.x`;
3. some só na `2.0.0`, e o `CHANGELOG.md` diz o nome e o motivo.

Sete palavras reservadas já foram removidas assim, antes da 1.0 —
todas porque o parser não as consumia e elas só tiravam nomes de quem
escreve.

## Se algo quebrar sem estar aqui

É um bug, não uma mudança planejada. Abra uma questão com o programa
mínimo que reproduz: o objetivo deste documento é que isso não aconteça,
e cada vez que acontecer o guarda ganha um caso a mais.
