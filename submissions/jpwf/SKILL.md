---
name: pipefy-software-pipe
description: >
  Use this skill when the user wants to build, review, or improve a Pipefy pipe
  for a software development workflow (backlog, sprint, dev, QA, release). It
  guarantees that cards capture everything a team needs to actually build the
  software — clear requirements, acceptance criteria, Definition of Ready/Done,
  and links to code standards. Pairs with pipefy-ai-agents (agent that validates
  intake) and the workspace steerings (code-style, security, error-handling,
  testing-guide, git-conventions, documentation-standards).
tags: [pipefy, software, backlog, requirements, definition-of-ready, quality-gate]
---

# Software Development Pipe

Design a Pipefy pipe so that a card cannot advance into development without carrying the information a team needs to build the software correctly the first time. The goal is not "more fields" — it is **the right fields, gated at the right phase**, so ambiguity is caught before code is written.

Pairs with:
- [pipefy-ai-agents/SKILL.md](../pipefy-ai-agents/SKILL.md) — attach an AI agent that scores intake completeness and flags gaps (see [Optional: AI intake gate](#optional-ai-intake-gate)).
- Workspace steerings: `code-style`, `security`, `error-handling`, `testing-guide`, `git-conventions`, `documentation-standards`, `development-standards` — the pipe should make these standards *checklists on the card*, not tribal knowledge.

---

## Principle: capture at intake, gate at transitions

Two mistakes to avoid:

1. **Everything on the start form.** Requesters abandon 20-field forms and fill them with junk. Only ask upfront what a requester can genuinely answer.
2. **Nothing gated.** If any card can move to "Em Desenvolvimento" regardless of content, the form is theater.

The fix: split information across phases and enforce a **Definition of Ready (DoR)** and **Definition of Done (DoD)** as required fields on the *entry* to the phase that needs them.

| Information | Who provides it | Where it lives |
|---|---|---|
| Problem / value / requester | Requester | Start form |
| Refined requirements + acceptance criteria | Product/analyst | "Refinamento" phase (DoR gate) |
| Technical approach, risks, estimate | Dev/tech lead | "Refinamento" / "Pronto p/ Dev" |
| Test evidence, review, docs | Dev/QA | "Em Desenvolvimento" → "Revisão" (DoD gate) |

---

## Reference flow

```
[Start form] ──► Backlog ──► Refinamento ──► Pronto p/ Dev ──► Em Desenvolvimento ──► Code Review ──► QA / Testes ──► Pronto p/ Release ──► Concluído
                                 │ (DoR gate)                        │                                  │ (DoD gate)
                                 ▼                                   ▼                                  ▼
                            Bloqueado ◄──────────────── (de qualquer fase de trabalho) ────────────────┘
```

- **Backlog** — item recebido, ainda não priorizado.
- **Refinamento** — requisitos detalhados, critérios de aceite escritos, estimativa. **DoR é validada na saída daqui.**
- **Pronto p/ Dev** — fila priorizada, tudo que o dev precisa está no card.
- **Em Desenvolvimento** — implementação.
- **Code Review** — revisão de código (gancho com `code-style`, `security`).
- **QA / Testes** — validação funcional e de qualidade (gancho com `testing-guide`).
- **Pronto p/ Release** — **DoD validada**, aguardando deploy.
- **Concluído** — entregue.
- **Bloqueado** — fase lateral para impedimentos; card volta para a fase de origem quando desbloqueia.

> As **regras de transição** (quais fases um card alcança a partir de outra, incluindo retornos e o lateral Bloqueado) são configuradas na **UI do Pipefy** (Settings → Phases → *cards can be moved to*). As ferramentas de pipe/campos criam fases e campos; as setas do fluxo são de UI.

---

## Start form — o mínimo que um requester consegue responder

Peça só o que agrega no recebimento. Campos sugeridos (crie na UI ou confirme com `get_start_form_fields`, guardando o `internal_id`):

| Campo | Tipo | Obrigatório | Por quê |
|---|---|---|---|
| Título da demanda | short_text | Sim | Identidade do card |
| Tipo | select (`Feature`, `Bug`, `Melhoria`, `Débito técnico`, `Spike`) | Sim | Roteamento e template de campos |
| Descrição do problema / objetivo | long_text | Sim | O "porquê", não o "como" |
| Valor esperado / impacto | long_text | Sim | Prioização honesta |
| Solicitante (e-mail) | email | Sim | Notificações e follow-up |
| Prioridade sugerida | select (`Baixa`, `Média`, `Alta`, `Urgente`) | Não | Sinal, decisão é do PO |
| Prazo desejado | date | Não | Expectativa, não compromisso |
| Anexos / evidências | attachment | Não | Prints, logs, mockups |

Para **Bug**, um formulário condicional (ou fase de triagem) deve exigir também: passos para reproduzir, comportamento esperado vs atual, ambiente/versão, e evidência (log/print). Um bug sem repro não é acionável.

---

## Definition of Ready (DoR) — gate de saída do Refinamento

Antes de um card entrar em desenvolvimento, estes campos devem estar preenchidos (torne-os obrigatórios na fase **Refinamento** ou como checklist obrigatório na transição para **Pronto p/ Dev**):

| Campo | Tipo | Conteúdo |
|---|---|---|
| Requisitos detalhados | long_text | O que deve ser feito, escopo e não-escopo explícitos |
| Critérios de aceite | long_text / checklist | Formato **Given/When/Then** ou lista verificável |
| Regras de negócio | long_text | Restrições, cálculos, casos especiais |
| Dependências | long_text / connection | Outros cards, times, serviços, APIs |
| Abordagem técnica | long_text | Arquitetura/decisão de design (link para ADR se relevante — ver `documentation-standards`) |
| Impacto de segurança | select + long_text | Toca auth/dados sensíveis/PII? (gancho com `security`) |
| Estimativa | select (Fibonacci: 1,2,3,5,8,13) ou horas | Tamanho relativo |
| Definição de "pronto" desta demanda | checklist | O DoD específico do card |

**Checklist de DoR (todos verdadeiros para sair do Refinamento):**

- [ ] O problema e o valor estão claros e não ambíguos.
- [ ] Critérios de aceite são testáveis e escritos.
- [ ] Dependências identificadas e endereçadas (ou marcadas como bloqueio).
- [ ] Impacto de segurança avaliado; se toca dados sensíveis, plano definido (`security`).
- [ ] Abordagem técnica esboçada e cabe em uma sprint (ou foi quebrada).
- [ ] Estimativa registrada.
- [ ] Critérios de teste conhecidos (`testing-guide`).

---

## Definition of Done (DoD) — gate de saída para Release

Campos/checklist obrigatórios na entrada de **Pronto p/ Release** (ou saída de QA):

| Item DoD | Gancho de steering |
|---|---|
| Código segue convenções (naming, imports, funções pequenas, sem código comentado) | `code-style` |
| Sem hardcoding de config/secrets; valores via env/settings | `development-standards`, `security` |
| Tratamento de erros tipado; sem `catch {}` vazio; logs estruturados | `error-handling` |
| Input validado/sanitizado; sem SQL por concatenação; headers de segurança | `security` |
| Testes escritos e passando; cobertura ≥ 70% no código novo | `testing-guide` |
| Code review aprovado (≥ 1 revisor) | `git-conventions` |
| Commits seguem Conventional Commits; PR ≤ 400 linhas, 1 responsabilidade | `git-conventions` |
| Documentação atualizada no mesmo PR (API.md, ARCHITECTURE.md, steerings) | `documentation-standards` |
| Sem dependências não usadas; versões fixas | `development-standards` |
| Funciona em qualquer ordem / sem estado global de teste | `testing-guide` |

**Checklist de DoD (todos verdadeiros para ir a Release):**

- [ ] Critérios de aceite atendidos e demonstrados.
- [ ] Testes unit/integração passando; cobertura do código novo ≥ 70%.
- [ ] Code review aprovado; PR dentro do padrão de tamanho e commits.
- [ ] Sem vulnerabilidades conhecidas introduzidas; segredos fora do código.
- [ ] Documentação e ADRs atualizados quando a mudança exige (`documentation-standards`).
- [ ] Sem regressões conhecidas; feature flags/rollback definidos se aplicável.

---

## Field-type cheat sheet (Pipefy)

Ao criar os campos via ferramentas de pipe (`create_pipe_field` / equivalentes) ou UI, use tipos que forçam qualidade de entrada:

| Necessidade | Tipo Pipefy | Nota |
|---|---|---|
| Texto curto controlado | `select` / `radio_vertical` | Prefira enum a texto livre para taxonomia (Tipo, Prioridade) |
| Texto estruturado | `long_text` | Para requisitos/critérios; use template no help text |
| Checklist verificável | `checklist_vertical` | DoR/DoD como itens marcáveis |
| Ligação a outro card | `connector` (pipe relation) | Dependências, épico ↔ história |
| E-mail do solicitante | `email` | Necessário para notificações |
| Estimativa | `select` (Fibonacci) | Evita números arbitrários |
| Evidência | `attachment` | Bug repro, mockups, logs |
| Datas | `date` / `due_date` | Prazo desejado ≠ compromisso |

**Regra:** todo campo que gateia uma transição deve ser **required na fase** (Phase field settings), não apenas no start form. Campos de start form obrigatórios só cobrem o recebimento.

---

## Build workflow (discover → structure → gate → verify)

Nunca chute IDs de fase, campo ou pipe — descubra sempre.

1. **Metadata** — `get_pipe(pipe_id)` para `uuid`, `phases[].id`, `phases[].name`.
2. **Start form** — `get_start_form_fields(pipe_id)`; crie/ajuste os campos de recebimento.
3. **Phase fields** — `get_phase_fields(phase_id)` por fase; adicione os campos de DoR na fase Refinamento e os de DoD antes de Release, marcando os obrigatórios.
4. **Transições** — configure na UI as setas do fluxo (avanços, retornos, Bloqueado). Documente o mapa de transições no card de setup ou no README do time.
5. **Gates** — marque required-on-phase os campos de DoR/DoD; opcionalmente adicione a AI intake gate (abaixo).
6. **Verifique** — mova um card de teste ponta a ponta; confirme que ele **não** avança sem os campos obrigatórios e que os checklists aparecem nas fases certas.

---

## Optional: AI intake gate

Use a skill [pipefy-ai-agents](../pipefy-ai-agents/SKILL.md) para adicionar um agente que reforça a qualidade do card sem depender de disciplina humana:

- **Behavior 1 — Triagem de completude (`card_created`):** o agente lê os campos do start form (`%{field:<internal_id>}` de descrição, valor, tipo) e, via `update_card` com `inputMode: fill_with_ai`, preenche um campo "Análise de completude" apontando o que falta (ex.: "sem critérios de aceite", "bug sem passos de repro"). Não deixe o agente aprovar/mover sozinho — ele **sinaliza**, humano decide.
- **Behavior 2 — Rascunho de critérios de aceite (`manually_triggered`):** ao clicar num botão no card, o agente propõe critérios Given/When/Then a partir da descrição, preenchendo um campo de rascunho para o analista revisar.
- **Behavior 3 — Aviso de segurança (`field_updated` no campo Tipo/Descrição):** se detectar menção a auth, PII ou dados sensíveis, preenche o campo "Impacto de segurança" com um alerta e link para o steering `security`.

Regras de modelagem (detalhes na skill de agents): `send_email_template` para notificar; `update_card` com `inputMode` obrigatório; máximo 5 behaviors; validar com `validate_ai_agent_behaviors` antes de criar. **Consentimento:** só adicione o agente se o usuário pediu IA — caso contrário, sugira e pergunte.

---

## Type-specific card templates

Ajuste os campos exigidos por **Tipo** (via campos condicionais ou fases distintas):

| Tipo | Campos extras exigidos |
|---|---|
| Feature | Critérios de aceite, mockup/UX, impacto em API (`API.md`), plano de teste |
| Bug | Passos de repro, esperado vs atual, ambiente/versão, evidência, severidade |
| Melhoria | Baseline atual, meta mensurável, critério de sucesso |
| Débito técnico | Risco de não fazer, área afetada, plano de refatoração, ADR se muda arquitetura |
| Spike | Pergunta a responder, timebox, entregável (documento/decisão) |

---

## Success criteria

- Um card **não avança** para desenvolvimento sem os campos de DoR preenchidos.
- Critérios de aceite existem e são testáveis antes de codar.
- DoD é uma checklist visível no card, alinhada aos steerings do repo.
- Card de teste percorre o fluxo inteiro e os gates bloqueiam quando devem.
- (Se houver agente) a análise de completude aparece no card e reflete lacunas reais.

## Failure modes

- **Start form gigante.** Requesters preenchem lixo. Mova campos de "como" para o Refinamento; deixe no form só o "o quê/porquê".
- **Campos obrigatórios só no start form.** Não gateiam transições. Torne-os required **na fase** que precisa deles.
- **Critérios de aceite em texto livre solto.** Sem formato, não são testáveis. Padronize Given/When/Then ou checklist.
- **DoD como documento externo.** Ninguém abre. Traga como `checklist_vertical` na fase de Release.
- **Agente que aprova sozinho.** Perde o julgamento humano e pode alucinar em campos vazios. Agente sinaliza; pessoa decide.
- **Transições sem retorno/bloqueio.** Cards travam ou pulam etapas. Configure retornos e o lateral Bloqueado na UI.

---

## Suggestions to evolve this skill

Ideias para aumentar o valor, se você quiser expandir depois:

1. **SLA por fase + escalonamento.** Campo de data de entrada por fase + automação/agente que notifica quando um card fica parado além do SLA (gancho com pipefy-automations e observability).
2. **Sincronização com Git/PR.** Registrar no card o link do PR e status de CI (via automação/webhook), fechando o loop entre o pipe e o `git-conventions`.
3. **Métricas ágeis.** Campos que alimentam lead time, cycle time e throughput; um dashboard (Grafana, dado o contexto do ambiente) lendo esses dados para o time.
4. **Templates versionados de card por Tipo.** Manter os templates de campos como código/config para recriar o pipe de forma reprodutível em novos times.
5. **DoR/DoD como steering dedicado.** Extrair as checklists para um steering próprio (`definition-of-ready.md` / `definition-of-done.md`) e referenciá-lo aqui e no code review, evitando divergência.
6. **Pré-preenchimento por RAG.** Se houver base de conhecimento (docs, ADRs), um data lookup/knowledge base no agente para sugerir requisitos e riscos a partir de demandas similares passadas.

## See also

- [pipefy-ai-agents/SKILL.md](../pipefy-ai-agents/SKILL.md) — agente de intake/notificação.
- Steerings: `.kiro/steering/code-style.md`, `security.md`, `error-handling.md`, `testing`/`testing-guide`, `git-conventions.md`, `documentation-standards.md`, `development-standards.md`.
