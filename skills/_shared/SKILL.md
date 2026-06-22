---
name: eremex-controls-shared
description: Shared concepts for ALL Eremex Avalonia controls — the namespace gotcha, class naming, theming via DeltaDesign/Fluent, pseudoclasses, localization, and where source/demos live. This skill should be used when a question is about Eremex controls in general (not one specific control), when resolving "which namespace" or "which using", when styling/theming any Eremex control, when adding a localization language, or when locating Eremex control source code or demo projects.
---

# Eremex controls — shared concepts

Cross-cutting facts that apply to every Eremex Avalonia control. Per-control skills (e.g. `mx-messagebox`) reference this instead of restating it.

## Namespace gotcha (read this first)

The **on-disk project path** and the **runtime namespace** do not match. This is the single most common mistake when writing `using` directives:

| What | Value |
|------|-------|
| Project / source folder | `Source/Eremex.Avalonia.Controls/...` |
| NuGet package | `Eremex.Avalonia.Controls` |
| **Runtime namespace** | **`Eremex.AvaloniaUI.Controls`** ← note `AvaloniaUI`, one word |

So to use any control:

```csharp
using Eremex.AvaloniaUI.Controls;          // MxMessageBox, MxTabControl, … and most enums
using Eremex.AvaloniaUI.Controls.Internal; // rare — only internal types (avoid)
```

Always confirm the exact namespace from the control's source file before writing code. Enum types (`MessageBoxButtons`, `MessageBoxIcon`, `MessageBoxResult`, …) generally live in `Eremex.AvaloniaUI.Controls`.

## Class naming

- Public controls use the **`Mx` prefix**: `MxMessageBox`, `MxTabControl`, `MxTabItem`, `MxTextBlock`, `MxSplitButton`, `MxWindow`, …
- Suffixes follow role: `…Control` (`DataGridControl`, `ListViewControl`, `PropertyGridControl`, `ToolbarControl`, `RibbonControl`), `…Editor` (`TextEditor`, `SpinEditor`, `ComboBoxEditor`, …), `…Base` for abstract bases.
- When a user says "the Eremex message box" / "the grid" / "the docking manager", map to the `Mx`-prefixed class name.

## Control base & styling

- Controls derive from Avalonia primitives (typically `TemplatedControl` / `ContentControl` / `Window`). Behavior is in C#; visuals are in XAML control templates shipped by the theme assemblies.
- Styling/theme comes from theme assemblies:
  - `Eremex.Avalonia.Themes.DeltaDesign` — the primary Eremex theme (default look).
  - `Eremex.Avalonia.Themes.Fluent` — Fluent-style alternative.
  - Reference the theme in `App.axaml` `<Application.Styles>` so control templates resolve. Without a theme, controls render with no template.
- State is exposed via **pseudoclasses** in styles (e.g. `:active`, `:readonly`, `:error`, `:pointerover`, `:pressed`, control-specific ones). When customizing appearance, target pseudoclasses rather than hard-coding brushes.
- To restyle a control, override its control template / styles in your `App.axaml` or a local `Styles.axaml`; use the theme's resources (brushes, sizes) to stay consistent across themes.

## Localization

- UI strings (button labels, built-in dialog text, etc.) come from `.resx` resource files, e.g. `Source/Eremex.Avalonia.Controls/Common/MessageBox/MessageWindow.resx` with culture-specific siblings `MessageWindow.ru.resx`, `MessageWindow.zh-Hans.resx`.
- The active culture follows the thread/UI culture. To switch language, set the thread culture (`CultureInfo.CurrentUICulture`) at startup before UI is constructed.
- To add a language: copy the base `.resx`, name it `<File>.<culture>.resx` (e.g. `MessageWindow.fr.resx`), translate the values. The strongly-typed accessor (`*.Designer.cs`) picks up keys automatically.
- The library ships several cultures out of the box (en, ru, zh-Hans, and more across the codebase).

## Where to find source & demos

- **Control source:** `d:\work\eremex\controls\Source\Eremex.Avalonia.Controls\` — organized by area: `Common/`, `Editors/`, `DataGrid/`, `Docking/`, `Bars/`, `PropertyGrid/`, `ListView/`, `TabControl/`, `Themes/`, …
- **Contracts/enums:** `d:\work\eremex\controls\Source\Eremex.Common.Contracts\` (e.g. `ApplicationServices/` for MessageBox enums).
- **Demo app:** `d:\work\eremex\controls-demo\` — runnable examples of controls in an app.
- When unsure how a control behaves, read its source in `controls/` — that is ground truth. Prefer it over memory.

## Related skills

- `mx-messagebox` — `MxMessageBox` dialog API, examples, customization.
