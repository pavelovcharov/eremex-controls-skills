# DockManager — save & restore layout

How to persist a dock layout (structure, sizes, tab order, float windows, auto-hide state) and restore it across sessions. API reference in [../SKILL.md](../SKILL.md); the item model in [layout-and-items.md](layout-and-items.md).

```csharp
using System.IO;
using System.Text;
using Eremex.AvaloniaUI.Controls.Docking;
using Eremex.AvaloniaUI.Controls.Serialization;   // SerializationSettings, SerializationMode
```

## The API (stream-based)

Save and restore live on `DockManager` and take a `Stream`:

```csharp
public void SaveLayout(Stream stream);
public void SaveLayout(Stream stream, SerializationSettings settings);

public void RestoreLayout(Stream stream);
public void RestoreLayout(Stream stream, SerializationSettings settings);
```

`SerializationSettings` selects the format:

```csharp
public class SerializationSettings
{
    public SerializationMode Mode { get; set; }   // Xml (default) | Json
}
public enum SerializationMode { Xml, Json }
```

`SerializationSettings.Default` gives the default (XML). There is **no built-in string/file overload** — wrap a `MemoryStream` / `FileStream` yourself (snippets below).

## Save to / restore from a string

```csharp
// --- SAVE to an XML string ---
string xml;
using (var ms = new MemoryStream())
{
    dm.SaveLayout(ms);
    xml = Encoding.UTF8.GetString(ms.ToArray());
}

// --- RESTORE from an XML string ---
using (var ms = new MemoryStream(Encoding.UTF8.GetBytes(xml), writable: false))
{
    dm.RestoreLayout(ms);
}
```

## Save to / restore from a file

```csharp
using (var fs = File.Create("layout.xml"))
    dm.SaveLayout(fs);

using (var fs = File.OpenRead("layout.xml"))
    dm.RestoreLayout(fs);
```

## JSON instead of XML

```csharp
dm.SaveLayout(stream, new SerializationSettings { Mode = SerializationMode.Json });
dm.RestoreLayout(stream, new SerializationSettings { Mode = SerializationMode.Json });
```

## Item identity — give items a `Name` ⚠️

Restore matches serialized entries back onto **live items by `Name`** (the property inherited from Avalonia `Control`):

- On **save**, each item's id is its `Name`; if `Name` is empty, a GUID like `Item_<guid>` is generated.
- On **restore**, the engine first looks for an existing live item with that `Name`. If found, the persisted dock state is applied **to your existing instance**. If not found, a new item of the serialized type is created (an empty container) and placed.

So: **give every item you want re-bound a stable `Name`.**

```xml
<do:DockPane x:Name="propertiesPane" Header="Properties" />
<do:DockPane x:Name="toolboxPane"    Header="Toolbox" />
```

Without those names, restore rebuilds the *structure* but will not reattach your existing pane instances (they get GUIDs on save and are recreated as empty typed containers on restore).

## What gets persisted

| Persisted | Notes |
|-----------|-------|
| Full item tree | parent/child relations, which item is `Root`. |
| Item sizes | `DockWidth` / `DockHeight` per item. |
| Group orientation | `DockGroup.Orientation`. |
| Tab order + active tab | `TabbedGroup.SelectedIndex`. |
| Float windows | `FloatLocation`, `FloatWidth`/`FloatHeight`, `WindowState`, and the screen it was on. |
| Auto-hide state | `AutoHideGroup.Dock` (edge) — and the auto-hide groups themselves. |
| Closed panes | items in `ClosedPanes`. |
| Per-item dock history | previous dock positions, so an item restores to where it was docked. |
| Screens | screen bounds at save time, used to reposition float windows that would otherwise fall off-screen. |

Not persisted (only layout is): your `Content`/view-model, header text, styling, and custom properties — re-supply those from your own state (or via `ItemsSource`).

## Restore behavior — clear vs. keep old items

By default, restore **clears the current layout** (root, float groups, auto-hide groups, closed panes) and rebuilds it from the stream, wrapped in a single layout-update batch.

To preserve items that exist before restore but **aren't** part of the saved layout, set the `RestoreBehavior` attached property to `KeepOldItems`:

```csharp
public enum DockManagerRestoreBehavior { Default, KeepOldItems }
```

```csharp
// Keep items not present in the saved layout (e.g. newly opened since last save):
DockManagerSerializer.SetRestoreBehavior(dm, DockManagerRestoreBehavior.KeepOldItems);
dm.RestoreLayout(stream);
```

| Behavior | What happens to current items |
|----------|-------------------------------|
| `Default` | The layout is cleared, then rebuilt entirely from the stream. |
| `KeepOldItems` | Items present before restore but absent from the saved layout are preserved (float/auto-hide groups kept; other items re-appended to their parents). |

## Typical app lifecycle

Save on close/shutdown, restore on startup:

```csharp
// startup
dm.Root = BuildInitialLayout();          // include stable Name on each item
if (File.Exists("layout.xml"))
{
    using var fs = File.OpenRead("layout.xml");
    dm.RestoreLayout(fs);                 // rebinds onto named items
}

// shutdown
using (var fs = File.Create("layout.xml"))
    dm.SaveLayout(fs);
```

> Call `dm.OptimizeLayout()` after bulk structural changes (including a restore) to collapse any redundant empty groups left behind.
