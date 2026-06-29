---
name: eremex-controls-shared
description: Shared concepts for ALL Eremex Avalonia controls — the namespace gotcha, class naming, theming via DeltaDesign, pseudoclasses, localization, and where docs/demos live. This skill should be used when a question is about Eremex controls in general (not one specific control), when resolving "which namespace" or "which using", when styling/theming any Eremex control, when adding a localization language, or when locating Eremex control demos and documentation.
---

# Eremex controls — shared concepts

Cross-cutting facts that apply to every Eremex Avalonia control. Per-control skills (e.g. `mx-messagebox`) reference this instead of restating it.

## Namespace gotcha (read this first)

The **NuGet package name** and the **runtime namespace** do not match. This is the single most common mistake when writing `using` directives:

| What | Value |
|------|-------|
| NuGet package | `Eremex.Avalonia.Controls` |
| **Runtime namespace** | **`Eremex.AvaloniaUI.Controls`** ← note `AvaloniaUI`, one word |

So to use any control:

```csharp
using Eremex.AvaloniaUI.Controls;          // MxMessageBox, MxTabControl, … and most enums
```

Always confirm the exact namespace from the official docs (`https://eremexcontrols.net/`) or the NuGet package (`https://www.nuget.org/packages/Eremex.Avalonia.Controls`) before writing code. Enum types (`MessageBoxButtons`, `MessageBoxIcon`, `MessageBoxResult`, …) generally live in `Eremex.AvaloniaUI.Controls`.

## Class naming

Eremex control names are **not uniform** — do not assume the `Mx` prefix. Three patterns coexist, and which one a control uses is not predictable:

- **`Mx` prefix** — e.g. `MxMessageBox`, `MxWindow`, `MxTextBlock`, `MxTabControl`, `MxTabItem`, `MxSplitButton`.
- **Role suffix, no `Mx`** — `…Control` (`DataGridControl`, `ListViewControl`, `PropertyGridControl`, `ToolbarControl`, `RibbonControl`), `…Editor` (`TextEditor`, `SpinEditor`, `ComboBoxEditor`), `…Manager` (`DockManager`), and similar.
- **`…Base`** — abstract base classes.

When a user says "the Eremex message box" / "the grid" / "the docking manager", map to the real class name — `MxMessageBox`, `DataGridControl`, `DockManager` respectively (note that only the first one carries the `Mx` prefix). Always confirm the exact name from the docs (`https://eremexcontrols.net/`) or the NuGet package (`https://www.nuget.org/packages/Eremex.Avalonia.Controls`) before writing code.

## Control base & styling

- Controls derive from Avalonia primitives (typically `TemplatedControl` / `ContentControl` / `Window`). Behavior is in C#; visuals are in XAML control templates shipped by the theme assemblies.
- Styling/theme comes from theme assemblies:
  - `Eremex.Avalonia.Themes.DeltaDesign` — the primary Eremex theme (default look).
  - Reference the theme in `App.axaml` `<Application.Styles>` so control templates resolve. Without a theme, controls render with no template.
- State is exposed via **pseudoclasses** in styles (standard ones like `:pointerover`, `:pressed`, `:selected`, plus control-specific ones). When customizing appearance, target pseudoclasses rather than hard-coding brushes. The exact control-specific pseudoclasses live in the theme sources (`https://github.com/Eremex/controlthemes`) — confirm them there before relying on a name.
- To restyle a control, override its control template / styles in your `App.axaml` or a local `Styles.axaml`; use the theme's resources (brushes, sizes) to stay consistent across themes.

## Localization

- UI strings (button labels, built-in dialog text, etc.) come from `.resx` resource files shipped in the package, with culture-specific siblings (e.g. the Russian and Chinese Simplified resources).
- The active culture follows the thread/UI culture. To switch language, set the thread culture (`CultureInfo.CurrentUICulture`) at startup before UI is constructed.
- To add a language for your own resources, follow the standard .resx satellite convention (e.g. `Resources.fr.resx`) and translate the values; the strongly-typed accessor (`*.Designer.cs`) picks up keys automatically. For built-in control strings, see the docs (`https://eremexcontrols.net/`).
- The library ships several cultures out of the box (en, ru, zh-Hans, and more across the codebase).

## Where to look (public sources)

The control source itself is not public. Verify behavior and API against these instead:

- **Documentation:** `https://eremexcontrols.net/` — API reference and guides; ground truth for the public API.
- **NuGet packages:** `https://www.nuget.org/packages/Eremex.Avalonia.Controls` — published assemblies and the contracts/enums they expose.
- **Demo app:** `https://github.com/Eremex/controls-demo` — runnable examples of controls in a real app.
- **Themes:** `https://github.com/Eremex/controlthemes` — the Eremex themes sources.
- When unsure how a control behaves, check the docs or the NuGet package; prefer them over memory.

## Related skills

- `datagrid-control` — `DataGridControl` tabular grid: columns/bands, sorting/grouping/filtering, editing, selection, row drag-drop, export.
- `mx-messagebox` — `MxMessageBox` dialog API, examples, customization.
- `dock-manager` — `DockManager` dockable layout: tool windows, document hosts, float/auto-hide, save/restore layout.
