# ARTEL Skills Agent Instructions

## Skill catalog maintenance

- Review `knowledge/catalog.json` whenever you add, edit, rename, move, or delete a `SKILL.md`
  or a file under a skill's `references/` directory. Update affected entries in the same change.
- Keep one topic per skill. Match the topic `id` to the skill directory and the `name` in its
  frontmatter, keep the `title` accurate, and set `path` to `skills/<id>/SKILL.md`.
- List every file under the skill's `references/` directory by its skill-relative path, including
  examples with extensions other than `.md`. Remove entries for deleted files and update paths
  for renamed or moved files. Keep an empty `references` array when the skill has no references.
- Register shared references under the skill that owns the file. When another skill links to one,
  include the owning topic and reference path so CLI and MCP users can retrieve it.
- Before handing off, compare the catalog with the skill tree. Check for missing or duplicate
  entries and paths that no longer resolve. Leave the JSON unchanged for content-only edits that
  do not change any catalog fields.
