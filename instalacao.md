# Instalação e preparação do ambiente — DataForge

Guia passo a passo, do zero até rodar seu primeiro programa. Tempo estimado: 5 minutos.

---

## 0. O caminho mais curto: o executável

Se você só quer **usar** a linguagem, não precisa de Python nem de mais nada.
Há um executável pronto por sistema:

```bash
curl -fsSL https://dataforge-lang.vercel.app/instalar.sh | DATAFORGE_BINARIO=1 sh
```

Ele baixa, confere que o programa **roda de verdade**, e configura o `PATH`.
No Windows:

```powershell
irm https://dataforge-lang.vercel.app/instalar.ps1 | iex
```

Ou pegue o arquivo à mão em
[releases](https://github.com/estevam5s/DataForge/releases) — há um para
Linux x64, macOS Intel, macOS Apple Silicon e Windows x64, com `SHA256SUMS.txt`
ao lado para conferir.

| | executável | Python + pip |
|---|---|---|
| precisa de Python | **não** | 3.10+ |
| download | ~12 MB comprimido, ~28 MB em disco | ~2 MB |
| `dataforge run` de um "olá" | 80 ms | 67 ms |
| extensão do VS Code com LSP | **vem junto** | precisa de `npm install` |
| `pip install` de pacote Python | não | sim |
| mexer no interpretador | não | sim |

Os 80 ms contra 67 ms são medidos, não estimados: o executável carrega o mesmo
interpretador, só que de uma cópia própria do Python. Ele vem como pasta
comprimida em vez de arquivo único **de propósito** — em arquivo único, o
pacote inteiro é descompactado num diretório temporário a cada chamada, e o
mesmo comando passa a levar 3,5 segundos.

**Use o executável para escrever programas; use o Python para contribuir com a
linguagem.** O resto deste guia é o segundo caminho.

---

## 1. Requisitos

| Item | Versão mínima | Como conferir |
|------|---------------|---------------|
| Python | 3.10 | `python3 --version` |
| pip | qualquer recente | `python3 -m pip --version` |
| git | opcional (para clonar) | `git --version` |

DataForge é um interpretador escrito em Python puro. **Não há dependências
externas obrigatórias** — nada de compilar, nada de baixar toolchain.

> Nada disto vale para o executável da seção 0, que já traz o Python dentro.

### Se você ainda não tem Python

**macOS**

```bash
brew install python@3.12
```

**Ubuntu / Debian**

```bash
sudo apt update && sudo apt install -y python3 python3-pip python3-venv
```

**Windows**

Baixe em [python.org/downloads](https://www.python.org/downloads/) e marque
**"Add Python to PATH"** durante a instalação.

Confirme:

```bash
python3 --version
# Python 3.12.x   (qualquer 3.10+ serve)
```

---

## 2. Obter o DataForge

### Opção A — clonar o repositório (recomendado para estudar/contribuir)

```bash
git clone https://github.com/estevam5s/DataForge.git
cd DataForge
```

### Opção B — baixar o ZIP

Baixe pelo botão **Code → Download ZIP** no GitHub, extraia e entre na pasta.

---

## 3. Criar um ambiente virtual

O ambiente virtual isola o DataForge do Python do sistema. É opcional, mas evita
conflitos.

**macOS / Linux**

```bash
python3 -m venv .venv
source .venv/bin/activate
```

**Windows (PowerShell)**

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
```

O prompt passa a exibir `(.venv)`. Para sair depois: `deactivate`.

---

## 4. Instalar

### Instalação normal

```bash
pip install .
```

### Instalação editável (para quem vai mexer no interpretador)

```bash
pip install -e ".[dev]"
```

O modo editável faz suas alterações em `dataforge/*.py` valerem imediatamente,
sem reinstalar. O extra `[dev]` traz o `pytest`.

### Verificar

```bash
dataforge version
```

Saída esperada:

```
DataForge v3.1.0
Python 3.12.x ...
```

Os dois comandos abaixo são equivalentes — `df` é o atalho:

```bash
dataforge run programa.df
df run programa.df
```

### Rodar sem instalar

Dentro da pasta do projeto:

```bash
python3 -m dataforge run programa.df
```

---

## 5. Primeiro programa

Crie `ola.df`:

```dataforge
// ola.df — primeiro programa em DataForge

nome := "mundo"
out "Ola, " + nome + "!"

cycle i from 1 to 3:
    out "contando:", i
```

Execute:

```bash
dataforge run ola.df
```

```
Ola, mundo!
contando: 1
contando: 2
contando: 3
```

---

## 6. Comandos da CLI

| Comando | O que faz |
|---------|-----------|
| `dataforge run arquivo.df` | Executa um programa |
| `dataforge arquivo.df` | Idem (a palavra `run` é opcional) |
| `dataforge repl` | Abre o console interativo |
| `dataforge check arquivo.df` | Analisa sem executar (aceita pasta: `dataforge check .`) |
| `dataforge tokens arquivo.df` | Mostra o fluxo de tokens (lexer) |
| `dataforge ast arquivo.df` | Mostra a árvore sintática (parser) |
| `dataforge new` | Cria um projeto a partir de um template |
| `dataforge version` | Mostra a versão |
| `dataforge help` | Mostra a ajuda |

### Flags

| Flag | Efeito |
|------|--------|
| `--time` | Mostra o tempo de execução |
| `--debug` | Imprime tokens, AST e o traceback completo em caso de erro |
| `--no-color` | Desliga as cores ANSI (útil em CI e logs) |

```bash
dataforge run ola.df --time
dataforge run ola.df --debug
```

---

## 7. Console interativo (REPL)

```bash
dataforge repl
```

```
forge> x := 10
forge> x * 2
20
forge> action dobro(n):
...        yield n * 2
...
forge> dobro(21)
42
forge> exit
```

Comandos do REPL:

| Comando | O que faz |
|---------|-----------|
| `help` | Lista os comandos |
| `exit` / `quit` | Sai do REPL |
| `clear` | Limpa a tela |
| `env` | Mostra as variáveis definidas |
| `reset` | Zera o estado do interpretador |
| `tokens <codigo>` | Mostra os tokens do trecho |
| `ast <codigo>` | Mostra a árvore sintática do trecho |
| `version` | Mostra a versão |

Blocos são detectados pelo `:` no fim da linha: o REPL continua lendo até você
enviar uma linha em branco.

---

## 8. Criar um projeto pronto

```bash
dataforge new
```

Escolha um template pelo número e informe o nome. Os templates disponíveis:

| Template | Gera |
|----------|------|
| **CLI Tool** | Ferramenta de linha de comando |
| **Data Analytics** | Análise estatística com pipelines |
| **Orientado a Objetos** | Blueprints, herança, traits |
| **Suíte de Testes** | Módulo + testes automatizados |
| **API REST** | Servidor HTTP com CRUD |
| **Web App** | Servidor com HTML e estáticos |

Depois:

```bash
cd nome-do-projeto
dataforge run main.df
```

---

## 9. Editor com realce de sintaxe (VS Code)

A gramática TextMate está em `editor/vscode/`.

```bash
mkdir -p ~/.vscode/extensions/dataforge
cp -r editor/vscode/* ~/.vscode/extensions/dataforge/
```

Reinicie o VS Code. Arquivos `.df` passam a ter realce.

---

## 10. Rodar os testes e exercícios

```bash
# Suíte de testes do interpretador
python3 -m pytest tests/ -q

# 120 exercícios comentados, todos com verificações
python3 exercicios/run_all.py

# Apenas um módulo de exercícios
python3 exercicios/run_all.py 03

# Os 43 exemplos
for f in examples/*.df; do dataforge run "$f" > /dev/null || echo "FALHOU: $f"; done
```

---

## 11. Problemas comuns

### `command not found: dataforge`

O ambiente virtual não está ativo, ou o `pip install` não rodou.

```bash
source .venv/bin/activate    # macOS/Linux
pip install .
```

Alternativa que sempre funciona dentro da pasta do projeto:

```bash
python3 -m dataforge run arquivo.df
```

### `No module named dataforge`

Você está fora da pasta do projeto e não instalou o pacote. Instale com
`pip install .` ou volte para a raiz do repositório.

### `SyncError: Tab character detected`

DataForge exige **espaços**, nunca tabs. No VS Code: abra a paleta de comandos →
"Convert Indentation to Spaces".

### `SyncError: Indentation mismatch`

Um bloco tem recuo inconsistente. Use sempre 4 espaços por nível.

### Erro de build no `pip install`

Atualize as ferramentas de empacotamento:

```bash
pip install --upgrade pip setuptools wheel
```

---

## 12. Desinstalar

```bash
pip uninstall dataforge-lang
```

Se usou ambiente virtual, apagar a pasta `.venv` também resolve.

---

## Próximos passos

- [`doc/TUTORIAL.md`](TUTORIAL.md) — a linguagem do zero, com exemplos
- [`doc/REFERENCIA.md`](REFERENCIA.md) — referência completa da sintaxe
- [`doc/BIBLIOTECA_PADRAO.md`](BIBLIOTECA_PADRAO.md) — os 13 módulos Arcane
- [`exercicios/`](../exercicios) — 120 exercícios resolvidos e comentados
