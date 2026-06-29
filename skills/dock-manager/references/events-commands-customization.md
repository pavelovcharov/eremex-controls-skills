# DockManager — events, commands & customization

Observing/vetoing dock operations, binding commands to UI, the behavior switches, and the styling hooks (pseudoclasses and template parts). API reference in [../SKILL.md](../SKILL.md); items in [layout-and-items.md](layout-and-items.md); save/restore in [serialization.md](serialization.md).

```csharp
using Eremex.AvaloniaUI.Controls.Docking;
```

## Events

| Event | Event-args (properties) | Cancelable? |
|-------|-------------------------|-------------|
| `DockItemActivated` | `OldItem`, `NewItem` (`DockItemBase?`) | — |
| `DockOperationStarting` | `DockOperation`, `Item`, **`Cancel`** | ✅ |
| `DockOperationCompleted` | `DockOperation`, `Item` | — |
| `DockItemStartFloatDragging` | `Item` | — |
| `DockItemEndFloatDragging` | `Item` | — |
| `DockItemContextMenuOpening` | `Item`, `PopupMenu`, **`Cancel`** | ✅ |
| `RegisterDockItem` | `Item` | — |

`DockOperation` (`Dock`, `Float`, `Close`, `Hide`) is the payload of the operation events.

### Veto an operation

`DockOperationStarting` fires before dock/float/close/hide; set `Cancel = true` to block it:

```csharp
dm.DockOperationStarting += (_, e) =>
{
    // Don't allow closing documents with unsaved changes:
    if (e.DockOperation == DockOperation.Close && e.Item is DocumentPane doc && HasUnsavedChanges(doc))
    {
        e.Cancel = true;
    }
};

dm.DockOperationCompleted += (_, e) =>
{
    if (e.DockOperation == DockOperation.Float)
        TrackFloated(e.Item);
};
```

### Track activation

```csharp
dm.DockItemActivated += (_, e) =>
{
    if (e.NewItem is DockPane pane)
        ActiveViewModel = pane.DataContext;   // keep VM selection in sync
};
```

### Float-drag lifecycle

```csharp
dm.DockItemStartFloatDragging += (_, e) => BeginFloatPreview(e.Item);
dm.DockItemEndFloatDragging   += (_, e) => EndFloatPreview(e.Item);
```

### Customize / suppress the context menu

The pane header context menu is cancelable, and you get the `PopupMenu` to mutate it. `PopupMenu.Items` is a collection of Eremex bar items (`ToolbarItem` and subclasses, namespace `Eremex.AvaloniaUI.Controls.Bars`) — not Avalonia `MenuItem`. Use `ToolbarButtonItem` for a clickable entry:

```csharp
using Eremex.AvaloniaUI.Controls.Bars;   // ToolbarButtonItem

dm.DockItemContextMenuOpening += (_, e) =>
{
    if (e.Item is DockPane { AllowClose: false })
        e.Cancel = true;            // no menu for non-closeable panes
    else
        e.PopupMenu.Items.Add(new ToolbarButtonItem { Header = "Reveal in tree" });
};
```

## Commands

`DockManager.Commands` (`DockManagerCommands`) exposes standard dock actions as `ICommand`. Pass the target item as the command parameter.

| Command | Action |
|---------|--------|
| `Close` | Close the item. |
| `Dock` | Dock back into the main layout. |
| `Float` | Float the item. |
| `FloatAll` | Float all items in the group. |
| `AutoHide` | Auto-hide the item. |
| `ToggleAutoHide` | Toggle auto-hide state. |
| `Pin` / `Unpin` | Pin (keep visible) / unpin (enable auto-hide). |
| `CloseAllButThis` | Close every sibling except this one. |
| `CloseAllTabs` | Close all tabs in the group. |
| `CloseAllDocuments` | Close all documents. |
| `ToggleMaximize` / `Maximize` / `Minimize` / `Restore` | Window/MDI state. |
| `Expand` | Expand an auto-hidden panel. |
| `NewHorizontalDocumentGroup` / `NewVerticalDocumentGroup` | Split the document host. |
| `MoveToNextDocumentGroup` / `MoveToPreviousDocumentGroup` | Move the active document between groups. |

### Bind a command to a button

```xml
<do:DockManager x:Name="dm">
  <do:DockManager.Root>
    <do:DockGroup>
      <do:DockPane x:Name="props" Header="Properties">
        <Button Content="Float me"
                Command="{Binding #dm.Commands.Float}"
                CommandParameter="{Binding #props}" />
      </do:DockPane>
    </do:DockGroup>
  </do:DockManager.Root>
</do:DockManager>
```

Invoke from code:

```csharp
if (dm.Commands.Close.CanExecute(props)) dm.Commands.Close.Execute(props);
dm.Commands.ToggleAutoHide.Execute(props);
```

> The per-pane `Allow*` flags (`AllowClose`, `AllowFloat`, `AllowAutoHide`, …) automatically disable the corresponding commands, so a command's `CanExecute` reflects what the pane is allowed to do.

## Behavior switches

| Property | Default | Effect |
|----------|---------|--------|
| `AutoHideMode` | `Default` | `Default` = auto-hide panes slide out as an **overlay**; `Inline` = they **push** the main layout aside. |
| `CloseOnlyActivePane` | `true` | Close targets only the active pane; `false` closes the whole group. |
| `AutoHideOnlyActivePane` | `false` | Restrict auto-hide to the active pane only. |
| `AllowDocumentSwitcher` | `true` | Enables the **Ctrl+Tab** document switcher window. |
| `OwnsFloatingDockPanes` | `true` | Manager owns the lifetime of floating *dock* panes. |
| `OwnsFloatingDocuments` | `true` | Manager owns the lifetime of floating *documents*. |
| `AllowFreeDocumentLayout` | `false` | Allow documents to live outside a document host. |
| `ShowAutoHideExpandButton` | `false` | Show an expand button on auto-hide trays. |

### Document switcher (Ctrl+Tab)

When `AllowDocumentSwitcher` is `true`, holding **Ctrl** and tapping **Tab** opens the switcher over panes where `ShowInDocumentSwitcher` is `true`. Each pane can also supply `DocumentSwitcherDescription` and `DocumentSwitcherFooterDescription` text shown in that window.

```xml
<do:DocumentPane Header="Program.cs"
                 ShowInDocumentSwitcher="True"
                 DocumentSwitcherDescription="C# source" />
```

## Restyling: pseudoclasses & template parts

As with all Eremex controls, visuals live in the theme assemblies (`Eremex.Avalonia.Themes.DeltaDesign` is the default). Target **pseudoclasses** rather than hard-coding brushes. Pseudoclasses used by the DeltaDesign dock themes include:

| Pseudoclass | On | Meaning |
|-------------|----|---------|
| `:active` | pane header | the pane is the active pane |
| `:glyph-before` / `:glyph-after` / `:glyph-hidden` | pane header | glyph placement (`ShowGlyphMode`) |
| `:top` / `:bottom` / `:left` / `:right` | `DockPane` | tab-strip placement side |
| `:no-system-chrome` | `FloatGroup` | custom-chrome float window |

> Pseudoclasses and template-part names below are taken from the current DeltaDesign theme and can change between theme versions. Before relying on one in a production style, confirm it against the live theme source (`https://github.com/Eremex/controlthemes`).

Selector examples (in `App.axaml` or a `Styles.axaml`; `do:` is the Docking XAML namespace):

```xml
<!-- highlight the active pane's border -->
<Style Selector="do|DockPane:active /template/ Border#PART_LayoutRoot">
    <Setter Property="BorderBrush" Value="{DynamicResource SystemAccentBrush}" />
</Style>

<!-- custom background for custom-chrome float windows -->
<Style Selector="do|FloatGroup:no-system-chrome">
    <Setter Property="Background" Value="..." />
</Style>
```

> The brush resource keys are illustrative — look up the real DeltaDesign keys in the themes sources (`https://github.com/Eremex/controlthemes`) and bind to those so your style tracks the active theme. Do not assume a specific resource key is stable across versions.

Template part names you may target when overriding a control template (verify each in the theme source before use):

| Part | Control | Purpose |
|------|---------|---------|
| `PART_Tray{Left,Top,Right,Bottom}` | `DockManager` | auto-hide trays per edge |
| `PART_Pane{Left,Top,Right,Bottom}` | `DockManager` | auto-hide panes per edge |
| `PART_LayoutRoot` | `DockPane` | root border (target of the `:top/:bottom/:left/:right` styles) |
| `PART_Grid` | `FloatGroup` | float window root grid (the caption is a `FloatWindowCaptionArea`-classed border, not a `PART_`) |
| `PART_ExpandButton`, `PART_TransformControl`, `PART_ItemsPresenter` | auto-hide tray | expand button, chevron rotation, items |

To fully retemplate a control, copy the theme's `DockManager.axaml` from the themes sources (`https://github.com/Eremex/controlthemes`) and adapt it; use the theme's resource keys (brushes/sizes) to stay consistent across themes. See the `_shared` skill for the broader theming story.

## Related skills

- `_shared` — theming via DeltaDesign, pseudoclasses, localization, the namespace gotcha.
