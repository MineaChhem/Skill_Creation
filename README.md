# claude-skills

Custom skills for [Claude](https://claude.ai) and Claude Code. Each skill is a
self-contained folder under [`skills/`](skills/).

## Skills

| Skill | What it does |
|---|---|
| [`ooad-engineer`](skills/ooad-engineer/) | Evidence-based object-oriented analysis and design: understand the real-world process, then produce requirements, use cases, domain model, design, traceability, and a vertical-slice build order — scaled to what was actually asked for |

## Install

### Claude Code

Clone once, then copy the skills you want.

```bash
git clone --depth 1 https://github.com/MineaChhem/claude-skills.git
mkdir -p ~/.claude/skills
```

One skill, available in all your projects:

```bash
cp -r claude-skills/skills/ooad-engineer ~/.claude/skills/
```

Every skill:

```bash
cp -r claude-skills/skills/* ~/.claude/skills/
```

Project only (committed with the project, so your team gets it too): copy into
`.claude/skills/` at the project root instead of `~/.claude/skills/`.

### Claude.ai / Claude desktop

Zip a skill folder (the folder itself, not just its contents), for example `ooad-engineer/`,
and upload it in the Skills section of Claude's settings.

---

## ooad-engineer

Makes Claude do proper object-oriented analysis and design before writing code. It works even
for domains Claude doesn't know well, because it learns how the real process works before
modeling it.

Most AI-generated system designs fail in one of two ways. Either every system turns into the
same `User / Product / Order / Payment` template, or the code gets written first and a class
diagram is reverse-engineered afterwards. This skill blocks both. Every element of the model
has to be justified by domain evidence, an explicit requirement, or a labelled assumption.

Two things are decided before any phase runs.

**Entry point** — greenfield, a feature added to an existing system, an existing codebase to
redesign, or a single named artifact. Each starts somewhere different. A pasted class gets
diagnosed before anything is proposed; a feature request gets the delta, not a remodelled
system.

**Depth** — Quick, Standard, or Full. "Draw me a class diagram" gets a class diagram and the
rules its multiplicities encode, not a software requirements specification. Full depth is
staged across turns: domain brief and requirements land first and get corrected before the
design is built on them.

| Phase | Output |
|---|---|
| **0. Understand the domain** | Learn the real process, extract practitioners' vocabulary, find rules and edge cases, then a short domain brief with every rule labelled Known / Inferred / Assumed / Open Question |
| **1. Requirements** | Problem statement, scope and out-of-scope, classified actors, numbered business rules with IDs, testable functional requirements, non-functional requirements, assumption and open-question log |
| **2. Analysis** | Use case diagram plus fully dressed use cases (extensions, exceptions, BR citations), conceptual domain model with multiplicities, system sequence diagrams |
| **3. Design** | GRASP-justified responsibility assignment, design class diagram, interaction diagrams, state machines, layering, patterns only where earned, decision log |
| **Traceability** | `FR → use case → business rule → responsibility → class/operation → test`, checked in both directions |
| **Validation** | Self-review gate across domain, requirements, analysis, design, and traceability before anything is presented as final |
| **4. Implementation** | Vertical slices, one feature end to end, ordered by risk rather than ease |

Anything with more than one role also gets a role × action permission matrix and a data
classification pass in Phase 1 — sensitive data turns the audit trail into a functional
requirement with a domain object behind it, rather than a non-functional aspiration. Money is
a value object carrying its currency; multi-currency systems store the exchange rate used at
transaction time, which is what makes reconciliation possible later.

Evidence labels are the part that matters most in practice. A design built on three unlabelled
guesses looks identical to one built on three confirmed rules — the labels are what tell the
reader which one they're holding. Inferred and assumed rules carry an H/M/L confidence grade so
the shakiest load-bearing assumption is the one raised first, and a regulatory number is never
graded `[Known]` without a citation.

Diagrams default to **Mermaid**. **PlantUML** is used for true UML use case notation and
**draw.io XML** for diagrams you'll rearrange by hand.

Reference files, loaded per phase rather than all at once:

| File | Loaded |
|---|---|
| [`references/artifact-templates.md`](skills/ooad-engineer/references/artifact-templates.md) | Phase 1 — domain brief, fully dressed use case, requirement and rule formats, assumption log, decision log, traceability table, slice definition-of-done |
| [`references/design-heuristics.md`](skills/ooad-engineer/references/design-heuristics.md) | Phase 3 — GRASP, SOLID, entity vs. value object, aggregate rules, pattern selection, design smells |
| [`references/diagram-syntax.md`](skills/ooad-engineer/references/diagram-syntax.md) | Any diagram — Mermaid, PlantUML, draw.io mxGraphModel |
| [`references/validation-checklist.md`](skills/ooad-engineer/references/validation-checklist.md) | Before finalizing |

It triggers on requests like:

- "Analyze, design and develop a house rent management system"
- "Draw the use case and class diagrams for a clinic booking system"
- "I have an idea for a POS for small coffee shops in Phnom Penh"
- "Our order module is all in OrderService — can you redesign the domain model?"

It won't trigger for single functions, bug fixes, layout and UX work with no domain-model
question behind it, or OOP theory questions.

A six-case eval suite covering both the positive and the negative triggers lives in
[`evals/evals.json`](skills/ooad-engineer/evals/evals.json).

---

## Adding a skill

1. Put the skill in `skills/<skill-name>/`. The folder name must match the `name:` field in
   its `SKILL.md` frontmatter.
2. Keep it self-contained. Anything the skill references (`references/`, `scripts/`,
   `assets/`) goes inside its own folder.
3. Check it before committing:
   - the frontmatter has `name` and `description`
   - the description is under 1024 characters and says when the skill should trigger
   - every file the `SKILL.md` points to exists
4. Add a row to the **Skills** table above and a short section describing the skill.

```bash
git add skills/<skill-name> README.md
git commit -m "Add <skill-name> skill"
git push
```

## Structure

```
claude-skills/
├── README.md
├── LICENSE
└── skills/
    └── ooad-engineer/
        ├── SKILL.md
        ├── LICENSE.txt
        ├── evals/
        │   └── evals.json
        └── references/
            ├── artifact-templates.md
            ├── design-heuristics.md
            ├── diagram-syntax.md
            └── validation-checklist.md
```

## License

[MIT](LICENSE), unless a skill folder has its own `LICENSE.txt` that says otherwise.
