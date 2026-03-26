# nnecec's skills

Personal repository for curated skills.

## Usage

```bash
npx skills add https://github.com/nnecec/skills
```

## Structure

```text
skills/
  <skill-name>/
    SKILL.md
```

`SKILL.md` is the required entrypoint for each skill. Optional supporting files
can live beside it in `references/`, `scripts/`, or `assets/`.

## Conventions

- Use kebab-case for directory names.
- Keep each skill self-contained.
- Put trigger guidance in the `description` field.
- Add external sources in the skill body when a skill is adapted from an
  article, gist, or thread.

## Current Skills

- `no-use-effect`

## Adding More

1. Create a new folder under `skills/`.
2. Add `SKILL.md` with YAML frontmatter containing `name` and `description`.
3. Add references or scripts only when they materially improve reuse.
