# Eremex Controls Skills

A collection of [Claude Code](https://claude.com/claude-code) skills that help Claude work correctly with **Eremex Avalonia controls from an application/consumer perspective** — how to call and customize them in an app that references the library.

> Audience: developers **using** the Eremex control library in their apps. These skills are *not* about modifying the control source. Point them at the consumer-facing public API, examples, and customization options.

## Repository layout

```
eremex-controls-skills/
├── README.md                  ← you are here
└── skills/
    ├── _shared/               ← concepts common to ALL Eremex controls
    │   └── SKILL.md
    └── mx-messagebox/         ← one folder per control
        ├── SKILL.md           ← lean entry point (API + quick examples)
        └── references/        ← depth loaded on demand
            ├── usage-examples.md
            └── customization.md
```

Each control gets its own folder under `skills/`. The `_shared` skill holds cross-cutting knowledge (namespaces, theming, localization) so per-control skills don't repeat it.

## Enabling the skills in Claude Code

This repo is **structure-only** — pick whichever loading mechanism fits your setup:

- **Per-project (recommended):** symlink or copy each `skills/<name>` folder into your project's `.claude/skills/` directory. On Windows (admin shell): `mklink /D .claude\skills\mx-messagebox d:\work\eremex\eremex-controls-skills\skills\mx-messagebox`.
- **User-wide:** copy or symlink into `C:\Users\<you>\.claude\skills\`.
- **As a plugin:** wrap `skills/` in a plugin layout (`.claude-plugin/marketplace.json`) and install via `/plugin marketplace add` + `/plugin install`.

After enabling, restart Claude Code and mention a control by name — its skill activates automatically via the `description` triggers in its `SKILL.md`.

## Adding a skill for a new control

Keep future skills uniform. The recipe:

1. **Folder.** Create `skills/<control-kebab-name>/` where the kebab name matches the class (`MxDataGrid` → `mx-datagrid`, `MxTabControl` → `mx-tab-control`). Underscore-prefixed names (`_shared`) are reserved for cross-cutting skills.
2. **`SKILL.md` frontmatter.** Must contain `name` (the kebab id) and `description`. Write the description in **third person** ("This skill should be used when…") and list concrete trigger phrases: the class name, common synonyms, and the main API method names (e.g. "MxMessageBox", "message box", "show a dialog", "Show", "ShowAsync"). The description is what Claude matches against user intent — be specific.
3. **Keep `SKILL.md` lean** (target ≤ ~2000 words): identity, namespace/`using`, the public API signatures, enum/value tables, a property table, and 2–3 quick examples. Push everything else into `references/*.md` (progressive disclosure — `SKILL.md` is always loaded when the skill fires; references load only when read).
4. **Don't duplicate `_shared`.** Reference it ("see the `_shared` skill for theming/localization") instead of restating.
5. **Verify the namespace.** Eremex source lives under `Source/Eremex.Avalonia.Controls/...` but the runtime namespace is `Eremex.AvaloniaUI.Controls` (`AvaloniaUI`, one word). Confirm the real `using` for each control from its source file before documenting.
6. **Cite source paths.** Back every API claim with a path into the `controls` repo (e.g. `Source/Eremex.Avalonia.Controls/Common/MessageBox/MxMessageBox.cs`) so Claude can re-verify against ground truth.

## Conventions

- Language: **English**.
- Description voice: **third person**, present tense.
- One control = one skill folder; depth in `references/`.
- Source of truth is always the `controls` repo at `d:\work\eremex\controls\`.

## Available skills

| Skill | Control | Covers |
|-------|---------|--------|
| `_shared` | (all) | Namespace gotcha, theming, localization, where source/demos live |
| `mx-messagebox` | `MxMessageBox` | Show/ShowAsync API, enums, examples, customization & nuances |
