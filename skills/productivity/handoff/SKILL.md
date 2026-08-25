---
name: handoff
description: Compact the current conversation into a handoff document for another agent to pick up.
argument-hint: "What will the next session be used for?"
when_to_use: 'Gatilhos em PT-BR (frases reais do usuário): "faça um handoff", "compacte a janela", "para não poluir o contexto", "vamos continuar depois/em outra sessão", "farei o merge e continuaremos depois". Também ao concluir tarefa longa ou antes de compactação iminente, proponha sem esperar o pedido. Para recapitular DENTRO da sessão, sem escrever arquivo, use landing.'
---

Write a handoff document summarising the current conversation so a fresh agent can continue the work. Save to the temporary directory of the user's OS - not the current workspace.

Include a "suggested skills" section in the document, naming which skills the next agent should call the Skill tool for.

Do not duplicate content already captured in other artifacts (specs, plans, ADRs, issues, commits, diffs). Reference them by path or URL instead.

Redact any sensitive information, such as API keys, passwords, or personally identifiable information.

If the user passed arguments, treat them as a description of what the next session will focus on and tailor the doc accordingly.
