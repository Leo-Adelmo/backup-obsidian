---

## name: skill-file-authoring  
description: Use this skill when creating, reviewing, or improving SKILL.md files that define focused, reusable, and well-structured AI skills.

# SKILL.md Authoring

## Purpose

Define best practices for creating new `SKILL.md` files.

A `SKILL.md` must describe one reusable capability that can be applied across different projects, stacks, or repositories. It should not describe project-specific configuration, commands, tools, folder structures, or runtime assumptions.

## When to use this skill

Use this skill when:

- Creating a new `SKILL.md`.
    
- Reviewing an existing `SKILL.md`.
    
- Splitting a broad skill into smaller focused skills.
    
- Deciding whether content belongs in `SKILL.md` or `AGENTS.md`.
    
- Standardizing the format of reusable AI skills.
    

Do not use this skill when:

- Writing project-specific repository instructions.
    
- Defining commands for lint, test, build, or deploy.
    
- Documenting the concrete stack of a project.
    
- Creating README documentation for human onboarding.
    
- Writing long reference documentation for a specific technology.
    

## Core principle

A `SKILL.md` must define a reusable skill, not a project context.

Use this distinction:

```
SKILL.md  = reusable capability
AGENTS.md = project-specific agent instructions
README.md = human-facing project documentation
references/ = long supporting documentation
examples/ = long examples
```

## Scope rules

Each `SKILL.md` must focus on one responsibility.

Good examples:

```
c4-model-documentation/SKILL.md
python-docstring-review/SKILL.md
sql-query-review/SKILL.md
architecture-decision-record/SKILL.md
```

Bad examples:

```
data-engineering-platform/SKILL.md
backend-development/SKILL.md
project-standards/SKILL.md
spark-airflow-docker-hudi/SKILL.md
```

Avoid combining multiple unrelated skills in the same file.

## Recommended file size

Keep the file concise.

Recommended size:

```
Simple skill: 40-80 lines
Medium skill: 80-150 lines
Complex skill: 150-250 lines
Practical maximum: 300 lines
Hard maximum: 5,000 words
```

If the file exceeds the practical maximum, split the skill or move detailed material into:

```
references/
examples/
```

## Folder structure

Use this structure:

```
skill-name/
├── SKILL.md
├── references/
│   └── reference-file.md
└── examples/
    └── example-file.md
```

Rules:

- The folder name must use `kebab-case`.
    
- The main file must be named exactly `SKILL.md`.
    
- Long references must go inside `references/`.
    
- Long examples must go inside `examples/`.
    
- The `SKILL.md` must remain the concise operational entry point.
    

## Frontmatter

Every `SKILL.md` must start with YAML frontmatter.

Use this format:

```
---
name: skill-name-in-kebab-case
description: Use this skill when [specific condition].
---
```

Rules:

- `name` must use `kebab-case`.
    
- `description` must start with `Use this skill when`.
    
- `description` must explain the specific situation in which the skill should be used.
    
- Avoid vague descriptions.
    

Bad example:

```
description: Helps with documentation.
```

Good example:

```
---
name: c4-model-documentation
description: Use this skill when creating, reviewing, or improving architecture documentation using the C4 Model.
---
```

## Recommended structure

Use this structure by default:

```
# Skill Title

## Purpose

## When to use this skill

## Core principle

## Instructions

## Output format

## Examples

## Validation checklist

## References
```

Remove sections that are not useful for the specific skill.

Avoid deep heading nesting. Prefer only:

```
# Heading 1
## Heading 2
### Heading 3
```

## Writing rules

Write instructions as direct operational rules.

Prefer:

```
Start with the System Context view before describing Containers.
```

Avoid:

```
It may be useful to think about the system context before maybe describing containers.
```

Use:

- short sections;
    
- direct verbs;
    
- explicit steps;
    
- clear output expectations;
    
- small examples;
    
- objective language.
    

Avoid:

- vague advice;
    
- motivational language;
    
- excessive theory;
    
- long technology explanations;
    
- redundant instructions;
    
- conflicting rules.
    

## Content that belongs in SKILL.md

Include:

- the purpose of the skill;
    
- when to use the skill;
    
- when not to use the skill;
    
- step-by-step instructions;
    
- output format;
    
- concise examples;
    
- validation checklist;
    
- links or references to supporting files;
    
- rules for ambiguity or missing information.
    

## Content that does not belong in SKILL.md

Do not include:

- project-specific commands;
    
- repository paths;
    
- local environment setup;
    
- concrete service names from one project;
    
- required versions of tools;
    
- secrets, tokens, passwords, or credentials;
    
- long tutorials;
    
- broad engineering principles unrelated to the skill;
    
- lint, test, build, or deploy commands;
    
- instructions that should be enforced by automated tools.
    

Move project-specific content to `AGENTS.md`.

## Instructions for creating a new SKILL.md

When creating a new skill:

1. Identify the single reusable capability.
    
2. Confirm that the content is not project-specific.
    
3. Define a clear `name` in `kebab-case`.
    
4. Write a `description` starting with `Use this skill when`.
    
5. Add a short `Purpose` section.
    
6. Add clear usage and non-usage conditions.
    
7. Add direct step-by-step instructions.
    
8. Define the expected output format.
    
9. Add one or two concise examples.
    
10. Add a validation checklist.
    
11. Move long explanations to `references/`.
    
12. Move long examples to `examples/`.
    

## Instructions for reviewing an existing SKILL.md

When reviewing a skill, check:

1. Does it define only one reusable capability?
    
2. Is the scope independent of a specific project?
    
3. Is the folder name in `kebab-case`?
    
4. Is the file named exactly `SKILL.md`?
    
5. Does it contain valid YAML frontmatter?
    
6. Does the `description` start with `Use this skill when`?
    
7. Are the instructions direct and actionable?
    
8. Is the file concise?
    
9. Are long references externalized?
    
10. Are project-specific details removed?
    
11. Are examples short and relevant?
    
12. Is there a validation checklist?
    

## Output format for new skills

When producing a new `SKILL.md`, use this format:

```
---
name: example-skill-name
description: Use this skill when creating, reviewing, or improving a specific reusable capability.
---

# Example Skill Name

## Purpose

## When to use this skill

## Do not use this skill when

## Instructions

## Output format

## Examples

## Validation checklist

## References
```

## Ambiguity handling

If the requested skill is too broad, split it into smaller skills.

Example:

```
Bad: data-engineering-documentation

Good:
- c4-model-documentation
- data-contract-documentation
- pipeline-readme-documentation
- architecture-decision-record
```

If the requested content is project-specific, recommend `AGENTS.md` instead.

Example:

```
Belongs in AGENTS.md:
- Use Python 3.12.
- Run tests with pytest.
- Use Spark 3.5.6.
- The source code is in src/jobs/.
- Docker Compose services are master_node, worker_node, and client_node.
```

## Validation checklist

Before finalizing a `SKILL.md`, verify:

-  The skill has one clear responsibility.
    
-  The content is reusable across projects.
    
-  The folder name uses kebab-case.
    
-  The file name is exactly `SKILL.md`.
    
-  YAML frontmatter is present.
    
-  The description starts with `Use this skill when`.
    
-  The file is preferably under 300 lines.
    
-  The file is under 5,000 words.
    
-  Instructions are direct and actionable.
    
-  Long examples are moved to `examples/`.
    
-  Long references are moved to `references/`.
    
-  Project-specific content was not included.
    
-  No secrets or credentials are present.
    
-  The output format is explicit.
    
-  The examples are short and relevant.
    

## References

Use these references when validating or improving this skill:

- OpenAI Skills documentation.
    
- Anthropic Claude Skills documentation.
    
- Anthropic guide for building Skills.
    
- OpenAI Codex AGENTS.md guidance.
    
- Google Gemini custom instructions guidance.
    
- AGENTS.md community documentation.