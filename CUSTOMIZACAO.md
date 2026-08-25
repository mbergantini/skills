# Este fork, e por que ele existe

Fork de [`mattpocock/skills`](https://github.com/mattpocock/skills) (MIT). O upstream é a
fonte; aqui só existem **duas** customizações, ambas no frontmatter, ambas de uma linha por
arquivo. Isso é deliberado: quanto menor o diff, mais barato o `git merge upstream/main`.

## As duas mudanças

### 1. `when_to_use` em português

O agente decide invocar uma skill lendo a `description` e o `when_to_use`. Com eles em inglês
genérico, as frases reais do usuário — *"vamos fazer uma triagem das issues"*, *"faça um
handoff"*, *"alinhe comigo"* — não casavam.

Os gatilhos vieram de `docs/skills/overlay-when-to-use.json` no repo `claude-config`, extraído
das cópias locais em 2026-08-15.

### 2. `disable-model-invocation` removido em 7 skills

O upstream marca **15** skills como só-do-usuário. O critério dele é coerente e foi preservado
onde não há razão para contrariá-lo: são as que **tomam a conversa** (`teach`, `wait-what`,
`loop-me`), **fazem setup** (`setup-*`, `wayfinder`) ou **escrevem em modo de redação**
(`writing-*`).

As 7 liberadas foram escolhidas **por medição**, não por preferência —
`~/.claude/bin/medir-ativacao.py`, janela de 60 dias:

| Skill | Oportunidades | Atendidas | |
|---|---|---|---|
| `grill-me` | **40** | 6 | **33%** — a maior lacuna do conjunto |
| `handoff` | **14** | 3 | **30%** |
| `triage` | 96 (âncora `gh issue create`) | 69 | funcionava, e a migração quebrou |
| `to-tickets`, `to-spec` | 1 | 0 | estão na tabela de pré-condição do `CLAUDE.md` |
| `grill-with-docs`, `improve-codebase-architecture` | — | — | citadas nas regras |

**O que NÃO muda:** `triage`, `to-spec` e `to-tickets` escrevem no issue tracker. A regra do
`CLAUDE.md` continua valendo — o agente abre e propõe; criar, rotular e fechar em massa passa
pelo usuário. A skill volta a ser invocável; a disciplina de não escrever fora sem aval é
regra do repo, não do frontmatter.

## Como atualizar

```bash
git fetch upstream
git merge upstream/main
```

Conflito só acontece se o Matt mexer nas **mesmas linhas de frontmatter** — `description`,
`when_to_use` ou o flag. Nesse caso, resolva mantendo a `description` nova dele e o
`when_to_use` seu.

Depois do merge, **reaplique e confira**:

```bash
grep -l "disable-model-invocation: true" skills/*/*/SKILL.md
```

Se alguma das 7 reaparecer na lista, o merge trouxe o flag de volta — remova de novo.

E então, no `claude-config`, bump do `sha` em `home-claude/settings.json` →
`extraKnownMarketplaces` → `mattpocock-skills-fork`. **O pin é deliberado**: o conteúdo é de
terceiro e registra hooks e fluxos que rodam como você, mesma razão pela qual o `i-have-adhd`
e o `openai-codex` são pinados lá.

## O que este fork NÃO é

Não é um lugar para escrever skills novas. Skill autoral vive em `agents-skills/` no
`claude-config`. Aqui só entra o que é derivação do upstream — senão o merge deixa de ser
barato, que é a única coisa que faz este fork valer a pena.
