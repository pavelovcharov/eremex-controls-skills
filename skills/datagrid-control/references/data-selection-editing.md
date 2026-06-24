# DataGridControl — data, selection, editing, drag-drop & export

The runtime data/row APIs: navigation & focus, single/multiple selection, the in-place editing lifecycle, row drag-and-drop reordering, batch updates, and export/clipboard. API reference in [../SKILL.md](../SKILL.md); columns in [columns-and-bands.md](columns-and-bands.md); sorting/grouping/filtering in [sorting-grouping-filtering.md](sorting-grouping-filtering.md).

```csharp
using Eremex.AvaloniaUI.Controls.DataGrid;
using Eremex.AvaloniaUI.Controls.DataControl;   // InvalidCellValueExceptionEventArgs, enums
```

## Navigation & focus

Keyboard navigation follows `grid.NavigationMode` — `Cell` (move through cells, the default) or `Row` (move through whole rows).

| Member | Notes |
|--------|-------|
| `FocusedRowIndex` | Two-way. The focused grid row index (`InvalidRowIndex` = none). |
| `FocusedColumn` | Two-way. The focused `GridColumn`. |
| `AutoScrollToFocusedRow` | Keep the focused row in view (default `true`). |
| `MoveFirstRow()` / `MoveLastRow()` | Move focus to the first/last row. |
| `MoveNextRow()` / `MovePrevRow(bool allowNavigateToAutoFilterRow = false)` | Move focus up/down. |
| `MoveNextPage()` / `MovePrevPage()` | Page up/down. |
| `MoveNextCell()` / `MovePrevCell()` | Cell-level navigation (Tab/Shift+Tab). |
| `IsTabStop` (column) | Whether a column participates in Tab navigation. |
| `AllowFocus` (column) | Whether a column can be focused at all. |

```csharp
grid.FocusedRowIndex = 5;
grid.FocusedColumn   = grid.Columns["Amount"];
grid.MoveNextCell();
```

## Selection

`grid.SelectionMode` is `Single` (default) or `Multiple`.

| Member | Notes |
|--------|-------|
| `SelectedItems` | Two-way `IList?`. In multiple mode, the selected source objects (bind to your VM). |
| `SelectRow(int)` / `UnselectRow(int)` | Select/deselect by row index. |
| `SelectRange(int start, int end)` | Select a contiguous range (multiple mode). |
| `ClearSelection()` / `SelectAll()` | Clear everything / select all rows. |
| `BeginSelection()` / `EndSelection()` | Batch selection changes (suppress intermediate `SelectionChanged`). |
| `IsRowSelected(int)` | `bool`. |
| `GetSelectedRowIndexes()` | `List<int>` of selected row indexes. |
| `KeepSelectedOnClick` | Preserve selection on a plain click. |

```csharp
grid.SelectionMode = RowSelectionMode.Multiple;
grid.BeginSelection();
grid.SelectRange(0, 9);
grid.EndSelection();

foreach (var idx in grid.GetSelectedRowIndexes())
    DoSomething(grid.GetSourceItemByRowIndex(idx));
```

`SelectionChanged` fires with `RowIndex` and `ChangeType` (`Refresh`, `Add`, `Remove`). Wrap bulk changes in `BeginSelection`/`EndSelection` to get a single refresh.

## Editing

Cells are editable when `grid.AllowEditing` and `column.AllowEditing` aren't `false`, and the column isn't `ReadOnly`.

| Member | Notes |
|--------|-------|
| `EditorShowMode` | When the editor appears: `PointerPressed` (default), `PointerPressedInFocusedCell`, `PointerReleased`, `PointerReleasedInFocusedCell`. |
| `EditorButtonShowMode` | When editor buttons appear: `ShowOnlyInEditor` (default), `ShowForFocusedCell`, `ShowForFocusedRow`, `ShowAlways`. |
| `column.EditorProperties` | A `BaseEditorProperties` configuring the editor (mask, display format, buttons, min/max, …). |
| `column.AllowImmediateEditorValuePosting` | Post the value as it changes vs on commit. |
| `ShowEditor()` / `HideEditor()` | Open/close the editor programmatically. |
| `PostEditor()` | Push the editor's current value into the cell (returns `bool`). |
| `CloseEditor()` | Commit & close the editor (returns `bool`). |
| `CommitEditing()` | Commit any pending editor (default == `CloseEditor`). |
| `ActiveEditor` | Read-only. The currently active editor `Control`. |

### Editor lifecycle events

```
user action → ShowingEditor (cancel) → editor opens → ShownEditor
            → user edits → CellValueChanging → (validate) → commit
            → CellValueChanged → editor closes → HiddenEditor
```

| Event | Use |
|-------|-----|
| `ShowingEditor` | Veto opening the editor for a specific cell (`e.Cancel = true`). |
| `ShownEditor` | Configure the just-opened editor (`e.Editor`). |
| `CellValueChanging` | Observe a value as it changes. |
| `ValidateCellValue` | Validate; set `e.ErrorContent` to reject and keep editing. |
| `CellValueChanged` | React to a committed change. |
| `HiddenEditor` | Tear down after the editor closes. |

```csharp
grid.ShowingEditor += (_, e) =>
{
    // Don't allow editing the Id column.
    if (e.Column.FieldName == "Id") e.Cancel = true;
};

grid.ValidateCellValue += (_, e) =>
{
    if (e.Column.FieldName == "Amount" && (decimal)e.Value! < 0)
        e.ErrorContent = "Amount must be non-negative.";
};

grid.CellValueChanged += (_, e) =>
{
    if (e.Column.FieldName == "Quantity" || e.Column.FieldName == "UnitPrice")
        RecalcTotal(e.RowIndex);
};
```

### Value validation via the row model

`grid.ValidateCellValuesOnShowAndUpdate` and `grid.ShowItemsSourceErrors` surface errors exposed by `IDataErrorInfo` / `INotifyDataErrorInfo` on the bound row objects. If an invalid value throws on commit, `InvalidCellValueException` fires with the exception.

## Batch data updates

When you're about to mutate `ItemsSource` heavily (or sort/group/filter in a chain), wrap it to avoid repeated refreshes:

```csharp
grid.BeginDataUpdate();
try
{
    // ... mutate source, change SortDirection/GroupCount/FilterString, etc.
}
finally
{
    grid.EndDataUpdate();   // single refresh; preserves focus/scroll/selection state
}
```

`BeginDataUpdate`/`EndDataUpdate` lock updates and restore row state; `CancelDataUpdate()` unlocks without refreshing. `grid.RefreshData()` forces a full re-sort/re-filter; `grid.RefreshRow(rowIndex)` repaints one row's cells.

## Row drag-and-drop

Enable row reordering with `grid.AllowDragDrop`. Two interaction modes:

| `RowDragMode` | Behavior |
|---------------|----------|
| `Row` (default) | The whole row is draggable. |
| `DragHandle` | Drag from the row indicator column (shown when `AllowDragDrop` + this mode). |

Related switches:

| Property | Notes |
|----------|-------|
| `UsePlatformRowDragDrop` | Use the platform drag-drop data flow. |
| `AllowDragDropSortedRows` | Allow D&D while the grid is sorted (off by default — reordering a sorted view is ambiguous). |
| `AllowScrollingOnDrag` | Auto-scroll near the edges while dragging. |
| `AutoExpandOnDrag` / `AutoExpandDelayOnDrag` | Auto-expand hovered group rows. |

### Drag events

| Event | Args | Notes |
|-------|------|-------|
| `StartDrag` | `Items`, `DragRowIndexes`, `Data`, `Effects` | A drag began; set `Effects`. |
| `DragOver` | `Items`, `TargetRowIndex`, `DropPosition`, `KeyModifiers`, `Effects` | Hovering; choose `DropPosition` (`Before`/`After`/`Inside`) and `Effects`. |
| `Drop` | `Items`, `TargetRowIndex`, `DropPosition`, `Data`, `Effects` | Rows dropped — perform the move in your source. |
| `CompleteDragDrop` | `Items`, `Effects` | Drag-drop finished. |

> The grid raises these events; **you** apply the reordering to your bound `ItemsSource` in the `Drop` handler (insert at the computed index using `TargetRowIndex`/`DropPosition`). `DataGridDragData` / `DataGridDropData` carry the drag/drop payloads.

```csharp
grid.AllowDragDrop = true;
grid.Drop += (_, e) =>
{
    var list = (ObservableCollection<Order>)grid.ItemsSource!;
    int target = e.TargetRowIndex;
    foreach (var item in e.Items!.Reverse())
    {
        int from = grid.GetRowIndexBySourceItemIndex(list.IndexOf((Order)item));
        if (from == target) continue;
        list.Move(list.IndexOf((Order)item), target);
    }
};
```

## Context menus

Set whole-grid header/cell context menus via properties; the active column/row is available through the focused state:

| Property | Notes |
|----------|-------|
| `ColumnMenu` | `PopupMenu` shown for column headers. |
| `RowCellMenu` | `PopupMenu` shown for data cells. |
| `ShowColumnMenuFixedItem` | Include the built-in "fixed" item in the column menu. |

## Export & clipboard

| Method | Notes |
|--------|-------|
| `ExportToXlsx(string path / Stream, XlsxExportOptions?)` | Excel `.xlsx`. |
| `ExportToPdf(string path / Stream, PageExportOptions?)` | PDF (paginated report). |
| `ExportToCsv(string path / Stream, TextExportMode, string separator, bool quoteStringsWithSeparators)` | CSV. |
| `ExportToImages(string directory, string fileNameFormat, ImageExportOptions?)` | Per-page image files. |
| `CopyToClipboardAsync()` | Copy rows/selection to the clipboard. |

Options types: `XlsxExportOptions`, `PageExportOptions`, `ImageExportOptions`, `TextExportMode` (`Value` / `Text`). Per column, `AllowExport` includes/excludes it and `TextExportMode` selects value vs display text. `grid.ClipboardCopyHeaders` controls whether headers are copied.

```csharp
grid.ExportToXlsx("report.xlsx", new XlsxExportOptions { /* … */ });
await grid.CopyToClipboardAsync();
```

## Styling notes

Behavior is in C#; the look comes from the theme assemblies (e.g. `Eremex.Avalonia.Themes.DeltaDesign`) via control templates and pseudoclasses. Target pseudoclasses (`:focused`, `:selected`, `:editing`, `:filtered`, `:grouped`, `:readonly`, etc.) rather than hard-coding brushes when restyling rows, cells, and headers. See the `_shared` skill for theming and pseudoclass conventions.
