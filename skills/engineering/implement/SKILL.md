---
name: implement
description: "Implement a piece of work based on a spec or set of tickets."
when_to_use: 'Gatilhos em PT-BR: "implementa a #123", "implemente a issue", "pode implementar o que combinamos", "constrói o ticket", ou uma issue ready-for-agent a executar. Só para trabalho já decidido: decidir o que fazer é grill-me, to-spec ou to-tickets.'
---

Implement the work described by the user in the spec or tickets.

Use /tdd where possible, at pre-agreed seams.

Run typechecking regularly, single test files regularly, and the full test suite once at the end.

Once done, use /code-review to review the work.

Commit your work to the current branch.
