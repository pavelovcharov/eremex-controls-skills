# Eremex Controls Skills

A collection of AI-coding-agent skills that help agents (Claude Code, Codex, and others) work correctly with **Eremex Avalonia controls from an application/consumer perspective** — how to call and customize them in an app that references the library.

> Audience: developers **using** the Eremex control library in their apps. These skills are *not* about modifying the control source. Point them at the consumer-facing public API, examples, and customization options.

## Repository layout

```
eremex-controls-skills/
├── README.md                  ← you are here
└── skills/
    ├── _shared/               ← concepts common to ALL Eremex controls
    │   └── SKILL.md
    ├── datagrid-control/      ← one folder per control
    │   ├── SKILL.md           ← lean entry point (API + quick examples)
    │   └── references/        ← depth loaded on demand
    │       ├── columns-and-bands.md
    │       ├── sorting-grouping-filtering.md
    │       └── data-selection-editing.md
    ├── dock-manager/
    │   ├── SKILL.md
    │   └── references/
    │       ├── events-commands-customization.md
    │       ├── layout-and-items.md
    │       └── serialization.md
    └── mx-messagebox/
        ├── SKILL.md
        └── references/
            ├── usage-examples.md
            └── customization.md
```

Each control gets its own folder under `skills/`. The `_shared` skill holds cross-cutting knowledge (namespaces, theming, localization) so per-control skills don't repeat it.

## Using these skills with AI coding agents

This repo is **structure-only** — there is no build step. Each `skills/<name>` folder is a self-contained skill (a `SKILL.md` plus a `references/` folder). Import them into whatever skills/custom-knowledge mechanism your agent uses:

| Agent | How to load |
|-------|-------------|
| **Claude Code** | Copy or symlink each `skills/<name>` folder into `.claude/skills/` (per-project) or `~/.claude/skills/` (user-wide), then restart Claude Code. Example (Windows, admin shell): `mklink /D .claude\skills\mx-messagebox d:\work\eremex\eremex-controls-skills\skills\mx-messagebox`. |
| **Codex** | Place the folders in your Codex skills directory (commonly `$CODEX_HOME/skills/`, or a project-level equivalent). The exact path depends on your Codex configuration — check your agent's docs for where custom skills/instructions are discovered. |
| **Other agents** | Import each folder's `SKILL.md` (and its `references/*.md`) into the agent's custom-knowledge / instructions / rules mechanism. |

Two things apply regardless of agent:

- **Include `_shared`.** The `_shared` skill (namespace gotcha, theming, localization) is referenced by every control skill — install it alongside the control-specific ones, or the per-control skills will reference concepts that aren't loaded.
- **Progressive disclosure.** `SKILL.md` is the always-loaded entry point; the `references/*.md` files load only when the agent reads them. If your agent does **not** support on-demand/progressive loading, don't import just `SKILL.md` — import `SKILL.md` **plus** the `references/*.md` files the task needs, or depth will be missing.

### Claude Code: install as a plugin (optional)

Claude Code can also install the whole set as a plugin: wrap `skills/` in a plugin layout (`.claude-plugin/marketplace.json`) and run `/plugin marketplace add <path-or-url>` then `/plugin install`. This `.claude-plugin` / `/plugin` flow is Claude-Code-specific — ignore it for other agents.

After enabling, mention a control by name and its skill activates automatically via the `description` triggers in its `SKILL.md`.

## Adding a skill for a new control

Keep future skills uniform. The recipe:

1. **Folder.** Create `skills/<control-kebab-name>/` where the kebab name is a readable slug of the class (`MxTabControl` → `mx-tab-control`, `DataGridControl` → `datagrid-control`). Underscore-prefixed names (`_shared`) are reserved for cross-cutting skills.
2. **`SKILL.md` frontmatter.** Must contain `name` (the kebab id) and `description`. Write the description in **third person** ("This skill should be used when…") and list concrete trigger phrases: the class name, common synonyms, and the main API method names (e.g. "MxMessageBox", "message box", "show a dialog", "Show", "ShowAsync"). The description is what Claude matches against user intent — be specific.
3. **Keep `SKILL.md` lean** (target ≤ ~2000 words): identity, namespace/`using`, the public API signatures, enum/value tables, a property table, and 2–3 quick examples. Push everything else into `references/*.md` (progressive disclosure — `SKILL.md` is always loaded when the skill fires; references load only when read).
4. **Don't duplicate `_shared`.** Reference it ("see the `_shared` skill for theming/localization") instead of restating.
5. **Verify the namespace.** The runtime namespace is `Eremex.AvaloniaUI.Controls` (`AvaloniaUI`, one word) — not the NuGet package name `Eremex.Avalonia.Controls`. Confirm the real `using` for each control from the official docs (`https://eremexcontrols.net/`) or the NuGet package (`https://www.nuget.org/packages/Eremex.Avalonia.Controls`) before documenting.
6. **Cite public sources only.** Back every API claim with a link to the official docs (`https://eremexcontrols.net/`), the NuGet package (`https://www.nuget.org/packages/Eremex.Avalonia.Controls`), or the demo (`https://github.com/Eremex/controls-demo`). Never use internal source paths.

### Verification checklist (before publishing a skill)

Run through this for every new or changed skill:

- [ ] **Public API exists.** Every property, method, event, command, and enum member you document is present on the **published** assembly. Verify against the released NuGet DLL for the current tag — not a development source branch, which can diverge from what consumers actually reference. (One way: reflect over the package DLL from the NuGet cache.)
- [ ] **Links resolve.** Any docs/demo/themes URLs you cite exist and point at the real page.
- [ ] **No internal/source leakage.** No references to `internal` types/members, source file paths, `DataController`-style plumbing, or other non-public surface. Search the skill text for the usual offenders.
- [ ] **Snippets compile.** `using` namespaces, type names, and XAML (e.g. markup extensions like `{x:Type ...}`) are copy-paste correct against the public API.
- [ ] **Hedge unconfirmed behavior.** Pseudoclasses, template part names (`PART_*`), and resource keys are verified against the themes sources (`https://github.com/Eremex/controlthemes`) or hedged ("confirm in the theme source") — never asserted as stable unless confirmed.

## Conventions

- Language: **English**.
- Description voice: **third person**, present tense.
- One control = one skill folder; depth in `references/`.
- Source of truth: official docs (`https://eremexcontrols.net/`) and NuGet packages (`https://www.nuget.org/packages/Eremex.Avalonia.Controls`). Demos: `https://github.com/Eremex/controls-demo`. Themes: `https://github.com/Eremex/controlthemes`.

## Available skills

| Skill | Control | Covers |
|-------|---------|--------|
| `_shared` | (all) | Namespace gotcha, theming, localization, where docs/demos live |
| `datagrid-control` | `DataGridControl` | Tabular grid: GridColumn/GridBand model, sorting/grouping/filtering, in-place editing, single/multiple selection, row drag-drop, fixed columns, column chooser, search panel, export (XLSX/PDF/CSV), layout serialization |
| `dock-manager` | `DockManager` | Dockable layout: dock item hierarchy (DockGroup/TabbedGroup/DocumentGroup/DockPane/DocumentPane/FloatGroup/AutoHideGroup), docking operations, document switcher, events/commands, MVVM via ItemsSource, layout serialization |
| `mx-messagebox` | `MxMessageBox` | Show/ShowAsync API, enums, examples, customization & nuances |
