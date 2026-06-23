---
name: dock-manager
description: The Eremex DockManager — a dockable-layout control for Avalonia apps (tool windows, tabbed/document hosts, floating windows, auto-hide/pin, drag-docking, and save/restore layout). This skill should be used when the user asks about DockManager, a docking manager / dock layout, tool windows or tool panels, document tabs / document host, floating/undocked windows, auto-hide or pinning panels, docking operations (Float/Dock/AutoHide/Close/Restore), the Ctrl+Tab document switcher, or saving/restoring a dock layout. Covers the public API (properties, methods, events, attached properties, commands), the dock item hierarchy (DockManager.Root → DockGroup / TabbedGroup / DocumentGroup / DockPane / DocumentPane / FloatGroup / AutoHideGroup), the relevant enums, XAML and C# usage, MVVM binding via ItemsSource, and layout serialization.
---

# DockManager

`DockManager` is Eremex's dockable-layout control: it hosts a tree of dock items — tool panels, tabbed groups, and document hosts — that the user can drag, dock to edges, float into separate windows, auto-hide (pin), close, and restore. Layouts can be serialized and restored across sessions. (Docs: `https://eremexcontrols.net/`; package: `https://www.nuget.org/packages/Eremex.Avalonia.Controls`; runnable examples: `https://github.com/Eremex/controls-demo`.)

> This skill documents the **public** API. Behavioral descriptions are derived from the public surface; confirm exact nuances against the docs or NuGet package when in doubt.

## Namespace

```csharp
using Eremex.AvaloniaUI.Controls.Docking;          // DockManager + all dock items + enums
using Eremex.AvaloniaUI.Controls.Serialization;     // SerializationSettings, SerializationMode (save/restore)
```

In XAML:

```xml
xmlns:do="clr-namespace:Eremex.AvaloniaUI.Controls.Docking;assembly=Eremex.Avalonia.Controls"
```

> The runtime namespace is `Eremex.AvaloniaUI.Controls.Docking` (one word: `AvaloniaUI`), **not** the package name `Eremex.Avalonia.Controls`. See the `_shared` skill for why. The `do:` prefix is conventional but arbitrary.

## Minimal XAML

`DockManager.Root` is the `[Content]` property, and `DockGroup.Items` is also `[Content]`, so the layout tree reads naturally inline:

```xml
<UserControl xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:do="clr-namespace:Eremex.AvaloniaUI.Controls.Docking;assembly=Eremex.Avalonia.Controls">

  <do:DockManager AllowDocumentSwitcher="True">

    <!-- Root split group laid out horizontally -->
    <do:DockGroup Orientation="Horizontal">

      <!-- A docked tool panel -->
      <do:DockPane Header="Toolbox" DockWidth="240">
        <TextBlock Text="Toolbox content" />
      </do:DockPane>

      <!-- Tabbed tool panels (tab strip defaults to Bottom) -->
      <do:TabbedGroup DockWidth="320">
        <do:DockPane Header="Properties"><TextBlock Text="…" /></do:DockPane>
        <do:DockPane Header="Errors"><TextBlock Text="…" /></do:DockPane>
      </do:TabbedGroup>

      <!-- Document host: tabs on top, close button on every tab -->
      <do:DocumentGroup>
        <do:DocumentPane Header="Program.cs"><TextBlock Text="// code" /></do:DocumentPane>
        <do:DocumentPane Header="MainWindow.axaml"><TextBlock Text="&lt;Window/&gt;" /></do:DocumentPane>
      </do:DocumentGroup>

    </do:DockGroup>
  </do:DockManager>
</UserControl>
```

More layouts (nested splits, auto-hide sizing, float-window config) are in [layout-and-items.md](references/layout-and-items.md).

## The dock item hierarchy

The layout is a **composite tree**. Groups contain items; panes are leaves. `FloatGroup` and `AutoHideGroup` are *roots* that live outside `DockManager.Root` (created on demand by dock operations):

```
DockManager.Root ──► DockGroup            (split: Orientation = Horizontal | Vertical)
                       ├── DockPane        (docked tool panel, leaf)
                       ├── DockGroup       (nested split — recursion allowed)
                       ├── TabbedGroup     (tabs; one child visible at a time; strip Bottom)
                       │      └── DockPane / DocumentPane
                       └── DocumentGroup   (IsDocumentHost=true; tabs on Top, close on all tabs)
                              └── DocumentPane / DockPane
                                 (parent = DocumentGroup ⇒ pane header hidden)

FloatGroup      (root — its own floating window; not inside Root)
   └── DockPane | TabbedGroup | DocumentGroup | nested DockGroup

AutoHideGroup   (root — slide-out panel pinned to an edge)
   ├── Dock   (Left | Top | Right | Bottom)
   └── DockPane
```

Key rules:
- **`DockPane` / `DocumentPane` are always leaves** — they carry `Content`, never children.
- **`DocumentPane`** is a `DockPane` marker subclass — same API, but treated as a *document* (lives in a `DocumentGroup`, appears in the document switcher).
- A `DockPane` whose parent is a `DocumentGroup` automatically hides its own header (`IsHeaderVisible = false`).
- **`FloatGroup`** requires a desktop/windowing lifetime (it opens a real window).

Full per-type property tables are in [layout-and-items.md](references/layout-and-items.md).

## DockManager properties

| Property | Type | Default | Notes |
|----------|------|---------|-------|
| `Root` | `DockGroup?` | — | `[Content]`. The main dock layout tree. |
| `ActiveDockItem` | `DockItemBase?` | `null` | Read-only. The currently active item. |
| `AutoHideMode` | `AutoHideMode` | `Default` | `Default` (overlay) or `Inline` (pushes layout). |
| `AutoHideOnlyActivePane` | `bool` | `false` | Restrict auto-hide to the active pane only. |
| `CloseOnlyActivePane` | `bool` | `true` | Close affects only the active pane (vs. the whole group). |
| `ShowAutoHideExpandButton` | `bool` | `false` | Show the expand button on auto-hide trays. |
| `AllowDocumentSwitcher` | `bool` | `true` | Enable the Ctrl+Tab document switcher. |
| `AllowFreeDocumentLayout` | `bool` | `false` | Allow documents outside a document host. |
| `OwnsFloatingDockPanes` | `bool` | `true` | Manager owns lifetime of floating *dock* panes. |
| `OwnsFloatingDocuments` | `bool` | `true` | Manager owns lifetime of floating *documents*. |
| `ItemsSource` | `IList?` | — | MVVM source; items auto-wrap into dock items (see below). |
| `ItemAdapter` | `IDockManagerItemAdapter?` | — | Custom placement logic for generated items. |
| `ItemTemplate` | `IDataTemplate?` | — | Container template for `ItemsSource` items. |
| `ItemContentTemplate` | `IDataTemplate?` | — | Content template for generated items. |
| `FloatGroups` | `ObservableCollection<FloatGroup>` | empty | Read-only collection of floating windows. |
| `AutoHideGroups` | `ObservableCollection<AutoHideGroup>` | empty | Read-only collection of auto-hide groups. |
| `ClosedPanes` | `ObservableCollection<DockPane>` | empty | Read-only collection of closed panes. |
| `Commands` | `DockManagerCommands` | — | Standard dock commands exposed as `ICommand` (see below). |

**Attached properties** (inherited down the dock tree):

| Property | Get / Set |
|----------|-----------|
| `DockManager` | `GetDockManager(obj)` / `SetDockManager(obj, value)` — back-reference to the owning manager. |
| `DockItem` | `GetDockItem(obj)` / `SetDockItem(obj, value)` — back-reference to the dock item a visual belongs to. |

## DockManager methods

All dock-operation methods return `bool` (`true` if the operation was applied):

```csharp
public bool Float(DockItemBase item);                                  // make it a floating window
public bool AutoHide(DockItemBase item);                               // pin to an edge as auto-hide
public bool Dock(DockItemBase item);                                   // dock back into the main layout
public bool Dock(DockItemBase item, DockItemBase target, DockType dockType); // dock next to a target
public bool Close(DockItemBase item);                                  // close (moves to ClosedPanes)
public bool Restore(DockItemBase item);                                // restore a closed item
public bool Remove(DockItemBase item);                                 // remove entirely
public bool ExpandAutoHidePanel(DockPane item);                        // expand an auto-hide panel
public bool CollapseAutoHidePanel(DockPane item);                      // collapse an auto-hide panel
```

Layout serialization (full details in [serialization.md](references/serialization.md)):

```csharp
public void SaveLayout(Stream stream);
public void SaveLayout(Stream stream, SerializationSettings settings);
public void RestoreLayout(Stream stream);
public void RestoreLayout(Stream stream, SerializationSettings settings);
```

**Extension methods** (`DockManagerExtensions`): `FindItem(name)`, `FindItem(predicate)`, `FindItem<T>(name)`, `GetItems()` (every item across Root/Float/AutoHide/Closed), `OptimizeLayout()`.

## DockManager events

| Event | Args | Cancelable? | Notes |
|-------|------|-------------|-------|
| `DockItemActivated` | `DockItemActivatedEventArgs` (`OldItem`, `NewItem`) | — | Active item changed (plain CLR event). |
| `DockOperationStarting` | `DockOperationStartingEventArgs` (`DockOperation`, `Item`, **`Cancel`**) | ✅ | Raised before dock/float/close/hide; set `Cancel = true` to veto. |
| `DockOperationCompleted` | `DockOperationCompletedEventArgs` (`DockOperation`, `Item`) | — | Raised after an operation completed. |
| `DockItemStartFloatDragging` | (`Item`) | — | A float drag started. |
| `DockItemEndFloatDragging` | (`Item`) | — | A float drag ended. |
| `DockItemContextMenuOpening` | (`Item`, `PopupMenu`, **`Cancel`**) | ✅ | Pane header context menu about to open. |
| `RegisterDockItem` | (`Item`) | — | A dock item was registered with the manager. |

Event/cancellation patterns and the context-menu handling are in [events-commands-customization.md](references/events-commands-customization.md).

## Enums (quick reference)

| Enum | Members |
|------|---------|
| `DockType` | `Left`, `Bottom`, `Right`, `Top`, `Fill` — target edge/zone for `Dock(item, target, dockType)`. |
| `AutoHideMode` | `Default` (overlay slide-out), `Inline` (pushes the layout). |
| `DockOperation` | `Dock`, `Float`, `Close`, `Hide` — payload of operation events. |
| `SelectTabOnClose` | `MostRecentlyOpened`, `Previous`, `Next` — which tab is selected after one closes. |
| `ShowGlyphMode` | `BeforeText`, `AfterText`, `Hidden` — icon placement in a header/tab. |
| `SerializationMode` | `Xml` (default), `Json` — `SerializationSettings.Mode`. |

## Item types at a glance

| Type | Is | Role |
|------|----|------|
| `DockItemBase` | abstract base | Common base: `DockWidth`/`DockHeight` (star sizing), `DockParent`, `FloatGroup`, `AutoHideGroup`, and `Name` (= serialization id). |
| `DockPane` | leaf | A dockable panel: `Content`, `Header`, `Glyph`, `IsActive`, `Allow*` flags, `CloseCommand`, tab overrides, document-switcher text. |
| `DocumentPane` | leaf (`: DockPane`) | A `DockPane` used as a document; no extra API. |
| `DockGroup` | container | Split layout (`Orientation`); holds `Items`. The usual `Root`. |
| `TabbedGroup` | container (`: DockGroup`) | Tabs (strip defaults to `Bottom`); `SelectedIndex`. |
| `DocumentGroup` | container (`: TabbedGroup`) | Document host: strip on `Top`, close button on all tabs, `IsDocumentHost = true`. |
| `FloatGroup` | root (`: DockGroup`) | A floating window: `FloatWidth/Height`, `FloatLocation`, `WindowState`, chrome/title/icon. |
| `AutoHideGroup` | root (`: DockGroup`) | An auto-hide panel: `Dock` (edge) + attached `AutoHideWidth/Height`. |

## Quick C# example

Build a tree imperatively, run dock operations, and react to events:

```csharp
using Eremex.AvaloniaUI.Controls.Docking;

var dm = new DockManager();

var properties = new DockPane { Header = "Properties", Name = "propertiesPane" };
var errors     = new DockPane { Header = "Errors",     Name = "errorsPane" };

var tools = new TabbedGroup();
tools.Add(properties);
tools.Add(errors);

var root = new DockGroup { Orientation = Avalonia.Layout.Orientation.Horizontal };
root.Add(tools);
dm.Root = root;

// Dock operations (each returns bool):
dm.Float(properties);                       // pop Properties out into a floating window
dm.AutoHide(errors);                        // pin Errors to an edge
dm.Dock(properties);                        // dock it back in
dm.Close(properties);                       // close → moves to dm.ClosedPanes
dm.Restore(properties);                     // bring it back

// Events — veto operations, observe completion, track activation:
dm.DockOperationStarting += (_, e) =>
{
    // e.DockOperation is Dock/Float/Close/Hide; e.Item is the target.
    // e.Cancel = true; // uncomment to veto
};
dm.DockOperationCompleted += (_, e) => Console.WriteLine($"{e.DockOperation} done.");
dm.DockItemActivated += (_, e) => Console.WriteLine($"Active: {e.NewItem}");
```

Finding items and tidying the layout:

```csharp
var pane = dm.FindItem<DockPane>("propertiesPane");   // by Name
var all  = dm.GetItems();                              // Root + Float + AutoHide + Closed
dm.OptimizeLayout();                                   // collapse redundant empty groups
```

## Save / restore layout (quick)

```csharp
using System.IO;
using System.Text;
using Eremex.AvaloniaUI.Controls.Serialization;

// Save to a string (XML by default):
string xml;
using (var ms = new MemoryStream())
{
    dm.SaveLayout(ms);
    xml = Encoding.UTF8.GetString(ms.ToArray());
}

// Restore later:
using (var ms = new MemoryStream(Encoding.UTF8.GetBytes(xml)))
{
    dm.RestoreLayout(ms);
}

// JSON instead of XML:
dm.SaveLayout(stream, new SerializationSettings { Mode = SerializationMode.Json });
```

> Give dock items a stable `Name` so a restored layout rebinds to your existing instances (otherwise items get a GUID and are recreated as empty containers). Full details — what is persisted, `RestoreBehavior` (`Default` vs `KeepOldItems`), file/string helpers — are in [serialization.md](references/serialization.md).

## MVVM with ItemsSource

Bind a collection and items are auto-wrapped into dock containers:

```csharp
dm.ItemsSource = myToolViewModels;   // each item becomes a DockPane added to Root
```

By default, generated **documents** (`DocumentPane`) go into the first visible `DocumentGroup` (one is auto-created if absent); everything else goes into `Root`. Customize placement with `ItemAdapter` (an `IDockManagerItemAdapter`), and the container/content templates with `ItemTemplate` / `ItemContentTemplate`. See [layout-and-items.md](references/layout-and-items.md#mvvm-itemssource--item-adapter).

## Commands

`DockManager.Commands` exposes standard dock actions as `ICommand` — bindable to buttons/menu items: `Close`, `Float`, `Dock`, `AutoHide`, `ToggleAutoHide`, `Pin`, `Unpin`, `CloseAllButThis`, `CloseAllTabs`, `CloseAllDocuments`, `FloatAll`, `ToggleMaximize`, `Maximize`, `Minimize`, `Restore`, `Expand`, `NewHorizontalDocumentGroup`, `NewVerticalDocumentGroup`, `MoveToNextDocumentGroup`, `MoveToPreviousDocumentGroup`. Binding examples and the full list are in [events-commands-customization.md](references/events-commands-customization.md#commands).

## Related skills

- `_shared` — namespace gotcha, theming via DeltaDesign, pseudoclasses, localization.
- `mx-messagebox` — `MxMessageBox` dialog for confirmations (e.g. "close this document?").
