# Este fork, e por que ele existe

Fork de [`mattpocock/skills`](https://github.com/mattpocock/skills) (MIT). O upstream é a
fonte; aqui só existem **duas** customizações, ambas no frontmatter, ambas de uma linha por
arquivo. Isso é deliberado: quanto menor o diff, mais barato o `git merge upstream/main`.

## As duas mudanças

### 1. `when_to_use` em português (**11 skills**)

O agente decide invocar uma skill lendo a `description` e o `when_to_use`. Com eles em inglês
genérico, as frases reais do usuário (*"vamos fazer uma triagem das issues"*, *"faça um
handoff"*, *"alinhe comigo"*) não casavam.

**Alcance:** isto vale para o harness do **Claude Code**, onde `when_to_use` entra na
descrição da tool (*"Becomes part of the tool description"*, no schema de frontmatter do
binário). O formato de skill do **Codex** lê apenas `name` e `description`, então lá estes
gatilhos não participam da descoberta. Duplicá-los na `description` significaria reescrever o
texto em inglês do upstream em 11 arquivos, que é exatamente o tipo de diff que encarece o
merge, e o Codex não é o harness em uso aqui. Fica registrado como limite conhecido, não como
esquecimento.

São as 11 citadas nominalmente nas regras do usuário: `triage`, `to-tickets`, `to-spec`,
`handoff`, `grill-me`, `grill-with-docs`, `improve-codebase-architecture`, `diagnosing-bugs`,
`prototype`, `tdd`, `writing-for-agents`.

Os gatilhos vieram de `docs/skills/overlay-when-to-use.json` no repo `claude-config`, extraído
das cópias locais em 2026-08-15. Enquanto viveram só naquele arquivo de documentação eles não
disparavam: gatilho fora do frontmatter não entra na decisão de invocar. Esse era o custo
declarado da Fase 2, e este fork existe para zerá-lo.

### 2. `disable-model-invocation` removido em 7 skills

O upstream marca **15** skills como só-do-usuário. O critério dele é coerente e foi preservado
onde não há razão para contrariá-lo: são as que **tomam a conversa** (`teach`, `wait-what`,
`loop-me`), **fazem setup** (`setup-*`, `wayfinder`) ou **escrevem em modo de redação**
(`writing-*`).

As 7 liberadas foram escolhidas **por medição**, não por preferência. A fonte é
`~/.claude/bin/medir-ativacao.py`, janela de 60 dias:

| Skill | Oportunidades | Atendidas | |
|---|---|---|---|
| `grill-me` | **40** | 6 | **33%**, a maior lacuna do conjunto |
| `handoff` | **14** | 3 | **30%** |
| `triage` | 96 (âncora `gh issue create`) | 69 | funcionava, e a migração quebrou |
| `to-tickets`, `to-spec` | 1 | 0 | estão na tabela de pré-condição do `CLAUDE.md` |
| `grill-with-docs`, `improve-codebase-architecture` | n/d | n/d | citadas nas regras |

**O que NÃO muda:** `triage`, `to-spec` e `to-tickets` escrevem no issue tracker. A regra do
`CLAUDE.md` continua valendo: o agente abre e propõe; criar, rotular e fechar em massa passa
pelo usuário. A skill volta a ser invocável; a disciplina de não escrever fora sem aval é
regra do repo, não do frontmatter.

### O que anda junto com o flag, e o que não anda

O `CLAUDE.md` deste repo trata **dois** campos como par: `disable-model-invocation: true` no
`SKILL.md` e `policy.allow_implicit_invocation: false` no `agents/openai.yaml` da skill. Skill
model-invoked simplesmente **omite** o bloco `policy:`. As 7 tiveram os dois removidos.

O que **não** foi acompanhado, por decisão explicita:

| Invariante do repo | Situação neste fork |
|---|---|
| `agents/openai.yaml` casado com o flag | **acompanhado** (7 blocos `policy:` removidos) |
| `docs/` afirma quem alcança a skill | **acompanhado** (a cláusula falsa foi trocada nas 7 páginas) |
| `README.md` agrupa em User-invoked / Model-invoked | **nota de desvio**, sem mover entradas |
| `README.md` de cada bucket, idem | **nota de desvio**, sem mover entradas |
| `ask-matt` rotula quem invoca o quê | **não acompanhado**, de propósito |

A distinção entre as duas primeiras linhas e as três últimas é a diferença entre **afirmar um
fato** e **organizar uma lista**. As páginas de `docs/` diziam, com todas as letras, que o agente
não alcança a skill sozinho: neste fork isso é falso, então a cláusula foi trocada, uma por
página, em posição estável. Já o agrupamento dos `README.md` é organização do upstream, e movê-lo
criaria conflito de apagar-e-inserir em duas seções de três arquivos que o Matt edita a cada skill
nova. A nota de desvio cobre o leitor pelo mesmo preço, com uma linha em posição fixa.

O `ask-matt` fica como está porque ele roteia por **fluxo de trabalho**, não por permissão: tudo
que ele diz (`/triage` para issues que chegaram cruas) continua verdadeiro. O fork **acrescenta**
um caminho, não retira o que o roteador descreve.

A razão é a mesma que justifica o fork inteiro: reescrever três `README.md` e o roteador
transformaria cada `git merge upstream/main` numa negociação de conflitos em arquivos que o Matt
edita com frequência. O ganho funcional seria zero, porque quem lê o flag é o harness, não o
README. Em troca, o `README.md` ganhou **uma linha** no topo da seção Reference dizendo que o
agrupamento abaixo é o do upstream e nomeando as 7 que este fork move. Assim o documento para de
mentir sem virar superfície de conflito.

## O manifesto do marketplace

`.claude-plugin/marketplace.json` declara `name: mattpocock-skills-fork`, nao o `mattpocock`
herdado do upstream. Isso nao e cosmetico: o nome do manifesto e a **identidade pela qual o
Claude Code resolve o marketplace**, e ele precisa bater com a chave em
`extraKnownMarketplaces` do `settings.json` no `claude-config`. Com o nome do upstream, tanto
`claude plugin marketplace add` quanto a declaracao pinada registram um marketplace com nome
diferente do esperado, e o `plugin install mattpocock-skills@mattpocock-skills-fork` falha com
*"not found in marketplace"*.

O `plugins[0].name` continua `mattpocock-skills` de proposito: e o nome do **plugin**, e e o
prefixo das skills (`mattpocock-skills:triage`). Trocar isso quebraria todas as invocacoes
escritas no `CLAUDE.md`.

**Num merge do upstream, se este arquivo conflitar, mantenha o nome do fork.**

## Como atualizar

```bash
git fetch upstream
git merge upstream/main
```

Conflito só acontece se o Matt mexer nas **mesmas linhas de frontmatter**: `description`,
`when_to_use` ou o flag. Nesse caso, resolva mantendo a `description` nova dele e o
`when_to_use` seu.

Depois do merge, **reaplique e confira as duas coisas**:

```bash
for nome in triage to-tickets to-spec handoff grill-me grill-with-docs \
            improve-codebase-architecture diagnosing-bugs prototype tdd writing-for-agents
do
  f=$(printf '%s\n' skills/*/"$nome"/SKILL.md)
  [ -f "$f" ] || { echo "AUSENTE: $nome"; continue; }
  grep -q '^when_to_use:' "$f" || echo "SEM GATILHO: $nome"
  grep -q '^disable-model-invocation: true' "$f" && echo "FLAG VOLTOU: $nome"
done
```

**Silêncio é o resultado bom.** Qualquer linha impressa é customização desfeita pelo merge:
reaplique. O laço testa `-f` antes de gravar, e por isso distingue *"a skill sumiu"* de *"a
skill está lá e perdeu a linha"*. Uma versão anterior deste comando usava
`grep -L ... 2>/dev/null`, que engolia justamente o primeiro caso e saía vazia, ou seja,
dizia "tudo certo" quando a skill nem existia mais.

⚠ **Depois de mexer em frontmatter, rode um parser YAML.** Trocar o travessão por
dois-pontos nas 11 linhas `when_to_use:` quebrou **todas elas**: em YAML, um `: ` dentro de
escalar simples fecha o escalar, e `chave: texto: mais` é erro de sintaxe
(*mapping values are not allowed here*). Por isso o valor vai entre **aspas simples**, onde `:` e
`"` são literais. Nenhum gatilho contém apóstrofo, então não há escape a fazer. A conferência:

```bash
python - <<'PY'
import glob, io, re, sys, yaml
mau = []
for f in sorted(glob.glob("skills/*/*/SKILL.md")):
    m = re.match(r"^---\n(.*?)\n---\n", io.open(f, encoding="utf-8").read(), re.S)
    if not m:
        mau.append((f, "sem frontmatter"))
        continue
    try:
        yaml.safe_load(m.group(1))
    except Exception as e:
        mau.append((f, str(e).split("\n")[0]))
print("quebradas:", mau or "nenhuma")
sys.exit(1 if mau else 0)
PY
```

Ele **diz qual arquivo** quebrou e sai com código 1, em vez de estourar uma exceção opaca no
primeiro erro.

O travessão **não** quebrava nada: ele saiu por causa da regra de prosa, não por sintaxe.

⚠ **Ao recopiar um gatilho da fonte, tire o travessão.** O `CLAUDE.md:25` deste repo proíbe
travessão em toda a prosa, inclusive em `SKILL.md`. A fonte (`gatilhos-ptbr.md`, no
`claude-config`) usa travessão livremente, porque lá a regra não existe. Nas linhas
`when_to_use:` o travessão introduzia a enumeração de frases, então a pontuação certa é
dois-pontos. Reescreva, não substitua caractere por caractere: a própria regra pede isso.

E então, no `claude-config`, bump do `sha` em `home-claude/settings.json` →
`extraKnownMarketplaces` → `mattpocock-skills-fork`. **O pin é deliberado**: o conteúdo é de
terceiro e registra hooks e fluxos que rodam como você, mesma razão pela qual o `i-have-adhd`
e o `openai-codex` são pinados lá.

## O que este fork NÃO é

Não é um lugar para escrever skills novas. Skill autoral vive em `agents-skills/` no
`claude-config`. Aqui só entra o que é derivação do upstream: senão o merge deixa de ser
barato, que é a única coisa que faz este fork valer a pena.
