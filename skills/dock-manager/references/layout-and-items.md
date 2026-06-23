# DockManager — items & layout

The dock item model in depth: every item type, its properties, who contains whom, and how to express layouts in XAML. API reference in [../SKILL.md](../SKILL.md); serialization in [serialization.md](serialization.md); events/commands/styling in [events-commands-customization.md](events-commands-customization.md).

Namespace for everything below:

```csharp
using Eremex.AvaloniaUI.Controls.Docking;
```

All types derive (directly or indirectly) from Avalonia `Control`. Defaults shown come from the property registrations.

## Hierarchy recap

```
DockItemBase                         (abstract: sizing, parent, roots, Name)
├── DockPane                         (leaf: Content + Header + control-box flags)
│   └── DocumentPane                 (marker: a DockPane that is a document)
└── DockGroup                        (container: Items + Orientation)
    ├── TabbedGroup                  (tabs: SelectedIndex, strip at Bottom)
    │   └── DocumentGroup            (document host: strip Top, close on all tabs)
    ├── FloatGroup                   (root: a floating window)
    └── AutoHideGroup                (root: a slide-out auto-hide panel)
```

- `DockGroup`, `TabbedGroup`, `DocumentGroup` are *containers* — they hold `Items`.
- `DockPane` / `DocumentPane` are *leaves* — they hold `Content`.
- `FloatGroup` and `AutoHideGroup` are *roots* — they are not nested inside `DockManager.Root`; they appear in `DockManager.FloatGroups` / `DockManager.AutoHideGroups`.

## `DockItemBase` (abstract base)

Every node in the tree.

| Property | Type | Default | Notes |
|----------|------|---------|-------|
| `DockWidth` | `GridLength` | `*` | Width share within the parent group's split. |
| `DockHeight` | `GridLength` | `*` | Height share within the parent group's split. |
| `DockParent` | `DockGroup?` | `null` | Immediate containing group (getter public). |
| `FloatGroup` | `FloatGroup?` | — | The floating root this item lives in, else `null`. |
| `AutoHideGroup` | `AutoHideGroup?` | — | The auto-hide root this item lives in, else `null`. |
| `SerializationInfo` | `object?` | `null` | Serialization payload while a (de)serialization is in flight. |
| `Name` | `string?` | — | Inherited from Avalonia `Control`. **Used as the stable id for layout serialization** — set it on items you want re-bound on restore. |

Inherited Avalonia sizing (`MinWidth`/`MaxWidth`/`MinHeight`/`MaxHeight`, `IsVisible`) also applies.

## `DockPane` — a dockable panel (leaf)

| Property | Type | Default | Notes |
|----------|------|---------|-------|
| `Content` | `object?` | `null` | `[Content]`. The pane body. |
| `ContentTemplate` | `IDataTemplate?` | `null` | Template for `Content`. |
| `Header` | `object?` | `null` | Caption content shown in the pane's title bar. |
| `HeaderTemplate` | `IDataTemplate?` | `null` | Template for `Header`. |
| `IsActive` | `bool` | `false` | Two-way. Whether this pane is the active one; setting `true` activates it. |
| `IsHeaderVisible` | `bool` | `true` | Read-only. Forced `false` when the parent is a `DocumentGroup`. |
| `Glyph` | `IImage?` | `null` | Icon shown in the header. |
| `GlyphSize` | `Size` | `NaN, NaN` | Size for `Glyph`. |
| `ShowGlyphMode` | `ShowGlyphMode` | `BeforeText` | Where the header glyph sits: `BeforeText` / `AfterText` / `Hidden`. |
| `AllowClose` | `bool` | `true` | Whether the user can close the pane. |
| `AllowAutoHide` | `bool` | `true` | Whether the pane can be auto-hidden. |
| `AllowFloat` | `bool` | `true` | Whether the pane can be floated. |
| `AllowMinimize` | `bool` | `true` | Whether the pane can be minimized (floating/MDI). |
| `AllowMaximize` | `bool` | `true` | Whether the pane can be maximized. |
| `CloseCommand` | `ICommand?` | `null` | Custom command invoked on close. |
| `CloseCommandParameter` | `object?` | `null` | Parameter for `CloseCommand`. |
| `TabHeader` | `object?` | `null` | Header shown on the tab when inside a `TabbedGroup` (falls back to `Header`). |
| `TabHeaderTemplate` | `IDataTemplate?` | `null` | Template for `TabHeader`. |
| `TabGlyph` | `IImage?` | `null` | Icon shown on the tab. |
| `TabGlyphSize` | `Size` | `NaN, NaN` | Size for `TabGlyph`. |
| `ShowTabGlyphMode` | `ShowGlyphMode?` | `null` | Overrides glyph mode for the tab; `null` falls back to `ShowGlyphMode`. |
| `ShowInDocumentSwitcher` | `bool` | `true` | Whether this pane appears in the Ctrl+Tab document switcher. |
| `DocumentSwitcherDescription` | `string?` | `null` | Description shown for this pane in the switcher. |
| `DocumentSwitcherFooterDescription` | `string?` | `null` | Footer description in the switcher. |

> The `Allow*` flags drive the pane's control box (close/auto-hide/float/maximize buttons): disabling one hides/disables the corresponding action. `CloseCommand` lets you intercept close for MVVM (e.g. prompt "save changes?").

## `DocumentPane` — a document (leaf, `: DockPane`)

A marker subclass with **no additional API**. It behaves like a `DockPane` but is treated as a *document*: the default item adapter places it in a `DocumentGroup`, and it participates in the document switcher. Use it for editor-style tabs.

## `DockGroup` — a split container

| Property / member | Type | Default | Notes |
|-------------------|------|---------|-------|
| `Items` | `ObservableCollection<DockItemBase>` | empty | `[Content]`. Child items. |
| `Orientation` | `Orientation` | `Horizontal` | `Horizontal` or `Vertical` split. |
| `IsDocumentHost` | `bool` | `false` | Marks this subtree as a document host (drop target for documents). |
| `ItemCount` | `int` | — | Number of items. |
| `this[int]` | `DockItemBase` | — | Indexer into `Items`. |

Methods: `Add(item)`, `Insert(index, item)`, `AddRange(params items)`, `Remove(item)`, `Clear()`.

## `TabbedGroup` — a tabbed container (`: DockGroup`)

Displays one child at a time, as a tab.

| Property | Type | Default | Notes |
|----------|------|---------|-------|
| `SelectedIndex` | `int` | `-1` | Two-way. Selected tab; `-1` = none. |
| `TabStripPlacement` | `Dock` | `Bottom` | Edge of the tab strip: `Left` / `Top` / `Right` / `Bottom`. |
| `ShowTabStripForSingleChild` | `bool` | `false` | Show the strip even with a single child. |
| `TabStripLayoutType` | `TabStripLayoutType` | `Scroll` | `MultiLine` / `Scroll` / `Stretch` overflow behavior. |
| `TabHeaderOrientation` | `TabHeaderOrientation` | `Auto` | `Auto` / `Horizontal` / `Vertical`. |
| `CloseButtonShowMode` | `TabControlCloseButtonShowMode` | `None` | Where per-tab close buttons appear (`None`, `InAllTabs`, `InActiveTab`, `InHeaderPanel`). |
| `AllowFloat` | `bool` | `true` | Whether the group can be floated as a unit. |
| `SelectTabOnClose` | `SelectTabOnClose` | `MostRecentlyOpened` | Which tab is selected after the selected tab closes. |

Inherits `Items`, `Orientation` (ignored — tabs are not a split), `IsDocumentHost`, and the `Add/Insert/...` methods from `DockGroup`.

## `DocumentGroup` — the document host (`: TabbedGroup`)

A `TabbedGroup` specialized for documents. It adds no new members but **overrides these defaults**:

| Property | Overridden default |
|----------|--------------------|
| `TabStripPlacement` | `Top` |
| `ShowTabStripForSingleChild` | `true` |
| `CloseButtonShowMode` | `InAllTabs` |
| `IsDocumentHost` | `true` |

When a `DockPane`'s parent is a `DocumentGroup`, the pane's `IsHeaderVisible` is forced `false` (the tab is the header). With multiple empty `DocumentGroup`s, only one is shown until documents are present.

## `FloatGroup` — a floating window (root, `: DockGroup`)

A root group rendered in its own top-level window.

| Property | Type | Default | Notes |
|----------|------|---------|-------|
| `Header` / `HeaderTemplate` | `object?` / `IDataTemplate?` | `null` | Window caption. |
| `Glyph` / `GlyphSize` | `IImage?` / `Size` | `null` / `NaN, NaN` | Caption icon. |
| `ShowGlyphMode` | `ShowGlyphMode` | `BeforeText` | Caption glyph placement. |
| `FloatWidth` / `FloatHeight` | `double` | `200` / `200` | Floating window size. |
| `FloatLocation` | `PixelPoint` | default | Screen-space position. |
| `WindowState` | `WindowState` | `Normal` | `Normal` / `Minimized` / `Maximized`. |
| `ShowSystemChrome` | `bool` | `false` | Use native OS window chrome vs. custom. |
| `WindowTitle` | `string?` | `null` | Native window title. |
| `WindowIcon` | `WindowIcon?` | `null` | Native window icon. |

> `FloatGroup` requires a desktop/windowing lifetime — constructing one on a platform without windowing throws. Floating windows are normally created for you by the `Float` operation; you rarely instantiate them directly.

**Attached properties** (set on any `AvaloniaObject` — typically a `DockPane` — to predetermine float geometry before it is floated):

| Attached prop | Default | Get / Set |
|---------------|---------|-----------|
| `FloatWidth` | `NaN` | `FloatGroup.GetFloatWidth(obj)` / `SetFloatWidth(obj, value)` |
| `FloatHeight` | `NaN` | `FloatGroup.GetFloatHeight(obj)` / `SetFloatHeight(obj, value)` |
| `FloatLocation` | default | `FloatGroup.GetFloatLocation(obj)` / `SetFloatLocation(obj, value)` |

## `AutoHideGroup` — a slide-out panel (root, `: DockGroup`)

| Property | Type | Default | Notes |
|----------|------|---------|-------|
| `Dock` | `Dock` | `Left` | Edge the auto-hide panel is attached to. |

**Attached properties** (set on a `DockPane` to size its slide-out):

| Attached prop | Default | Get / Set |
|---------------|---------|-----------|
| `AutoHideWidth` | `150` | `AutoHideGroup.GetAutoHideWidth(obj)` / `SetAutoHideWidth(obj, value)` |
| `AutoHideHeight` | `150` | `AutoHideGroup.GetAutoHideHeight(obj)` / `SetAutoHideHeight(obj, value)` |

Auto-hide groups are created by the `AutoHide` operation; `DockManager.AutoHideGroups` tracks them, and `AutoHideMode` (`Default` overlay vs `Inline`) controls how they expand.

## More XAML layouts

### Nested splits (Orientation per group)

```xml
<do:DockManager>
  <do:DockGroup Orientation="Horizontal">
    <!-- Left column is itself a vertical split -->
    <do:DockGroup Orientation="Vertical" DockWidth="300">
      <do:DockPane Header="Toolbox" DockHeight="2*"><TextBlock Text="…" /></do:DockPane>
      <do:DockPane Header="Output"  DockHeight="*"><TextBlock Text="…" /></do:DockPane>
    </do:DockGroup>

    <!-- Center: documents -->
    <do:DocumentGroup>
      <do:DocumentPane Header="file1.cs"><TextBlock Text="//" /></do:DocumentPane>
    </do:DocumentGroup>
  </do:DockGroup>
</do:DockManager>
```

### Pre-set auto-hide size and float geometry on a pane

```xml
<do:DockPane Header="Properties"
             Name="propertiesPane"
             do:AutoHideGroup.AutoHideWidth="220"
             do:FloatGroup.FloatWidth="360"
             do:FloatGroup.FloatHeight="480">
  <TextBlock Text="…" />
</do:DockPane>
```

These attached values are used when the pane later enters auto-hide or is floated.

### Configure a float window

Float groups are usually auto-created, but if you declare one (or restyle it), these are the caption/window knobs:

```xml
<do:FloatGroup FloatWidth="400" FloatHeight="300"
               WindowTitle="Floating Tools" ShowSystemChrome="True">
  <do:DockPane Header="Properties"><TextBlock Text="…" /></do:DockPane>
</do:FloatGroup>
```

## MVVM: ItemsSource & item adapter

Bind `ItemsSource` to a collection and each item is auto-wrapped into a dock container:

```csharp
dm.ItemsSource = viewModels;   // each becomes a DockPane added to Root
```

Default placement rules (the built-in `DefaultItemAdapter`):
- A generated **document** container (`DocumentPane`) → the first *visible* `DocumentGroup`; if none exists, a `DocumentGroup` is auto-created and added to `Root`.
- Everything else → added directly to `Root`.

Customize with three properties:

| Property | Purpose |
|----------|---------|
| `ItemTemplate` | `IDataTemplate` that builds the **container** (`DockPane` / `DocumentPane`) for a view-model. |
| `ItemContentTemplate` | `IDataTemplate` that builds the **content** placed inside the container. |
| `ItemAdapter` | An `IDockManagerItemAdapter` that decides *where* each generated item goes. |

The adapter interface is a single method:

```csharp
public interface IDockManagerItemAdapter
{
    void Adapt(DockManager dockManager, DockItemBase dockItem, object item);
}
```

Custom adapter — route view-models to specific groups:

```csharp
public sealed class MyAdapter : IDockManagerItemAdapter
{
    public void Adapt(DockManager dockManager, DockItemBase dockItem, object item)
    {
        var root = dockManager.Root ??= new DockGroup();

        switch (item)
        {
            case ToolViewModel:
                root.Add(dockItem);                       // tools → root split
                break;

            case DocumentViewModel:
                var docs = dockManager.GetItems()
                    .OfType<DocumentGroup>().FirstOrDefault(g => g.ActualIsVisible)
                    ?? new DocumentGroup();
                if (docs.DockParent == null) root.Add(docs);
                docs.Add(dockItem);                       // documents → document host
                break;
        }
    }
}

dm.ItemAdapter = new MyAdapter();
```

## Item traversal helpers

`DockItemExtensions` (on `DockGroup`) — recurse the subtree:

```csharp
public static IEnumerable<T> GetNestedItems<T>(this DockGroup group)        where T : DockItemBase;
public static IEnumerable<T> GetNestedVisibleItems<T>(this DockGroup group) where T : DockItemBase;
```

`DockManagerExtensions` (on `DockManager`) — search across *all* roots:

```csharp
var pane  = dm.FindItem<DockPane>("propertiesPane");   // by Name
var found = dm.FindItem(item => /* predicate */);
var all   = dm.GetItems();                             // Root + Float + AutoHide + Closed
dm.OptimizeLayout();                                  // drop redundant empty groups
```

## Nuances & gotchas

- **`IsHeaderVisible` is read-only** — it tracks the parent type (`false` under a `DocumentGroup`). Set `Header` instead.
- **`DockPane.TabHeader` falls back to `Header`** when unset; same for `TabGlyph`/`Glyph`. Set the tab-specific ones only when the tab should differ from the docked caption.
- **Sizing is star-based** — `DockWidth`/`DockHeight` are `GridLength`; use `*`/`2*`/fixed values like `"240"`. Auto-hide/float sizing uses the attached `double` properties instead.
- **`DocumentPane` vs `DockPane`** — pick by intent. A pane you want tabbed as a *document* (closeable tabs, document host, switcher) should be a `DocumentPane`; a *tool window* should be a `DockPane`.
- **`FloatGroup` needs a desktop lifetime** — it opens a real OS window; don't construct one in headless/non-desktop scenarios.
