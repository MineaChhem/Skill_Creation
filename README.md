# claude-skills

Custom skills for [Claude](https://claude.ai) and Claude Code. Each skill is a
self-contained folder under [`skills/`](skills/).

## Skills

| Skill | What it does |
|---|---|
| [`ooad-engineer`](skills/ooad-engineer/) | Domain-first object-oriented analysis and design: learn the real-world process, then produce requirements, use cases, domain model, design, and vertical-slice build order |

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

Makes Claude do proper object-oriented analysis and design before writing code. It works
even for domains Claude doesn't know well, because it learns how the real process works
before modeling it.

Most AI-generated system designs fail in one of two ways. Either every system turns into the
same `User / Product / Order / Payment` template, or the code gets written first and a class
diagram is reverse-engineered afterwards. This skill blocks both. The model has to come from
the domain.

| Phase | Output |
|---|---|
| **0. Learn the domain** | Research the real-world process, extract the practitioners' vocabulary, find the rules and edge cases, then report back a short domain brief for correction |
| **1. Requirements** | Problem statement, scope and non-scope, actors, numbered testable functional requirements, non-functional requirements, assumptions and open questions |
| **2. Analysis** | Use case diagram plus fully dressed use cases (with alternate flows and exceptions), a conceptual domain model with multiplicities, system sequence diagrams |
| **3. Design** | GRASP-justified responsibility assignment, design class diagram, interaction diagrams, state machines, layering, patterns only where they're needed |
| **4. Build** | Vertical slices, one feature end to end, starting with the riskiest assumption |

Diagrams default to **Mermaid**. **PlantUML** is used for true UML use case notation and
**draw.io XML** for diagrams you'll rearrange by hand. See
[`references/diagram-syntax.md`](skills/ooad-engineer/references/diagram-syntax.md).

It triggers on requests like:

- "Analyze, design and develop a house rent management system"
- "Draw the use case and class diagrams for a clinic booking system"
- "I have an idea for a POS for small coffee shops in Phnom Penh"

It won't trigger for single functions, bug fixes, pure UI work, or OOP theory questions.

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
        └── references/
            └── diagram-syntax.md
```

## License

[MIT](LICENSE), unless a skill folder has its own `LICENSE.txt` that says otherwise.
