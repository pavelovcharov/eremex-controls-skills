---
name: datagrid-control
description: The Eremex DataGridControl — a high-performance tabular data grid for Avalonia apps (data binding, auto-generated or explicit columns, bands, sorting, grouping with a group panel, column/auto-filter row filtering, cell/row editing with in-place editors, single/multiple row selection, row drag-and-drop reordering, fixed (frozen) columns, best-fit, column chooser, search panel, and export to XLSX/PDF/CSV/images). This skill should be used when the user asks about DataGridControl, a data grid / table control, grid columns, grid bands, grouping/sorting/filtering in a grid, the auto-filter row, the group panel, row selection in a grid, cell editing in a grid, row drag-and-drop, fixed/frozen grid columns, the column chooser, or exporting grid data. Covers the public API (properties, methods, routed events, commands), the GridColumn / GridBand model, the relevant enums, XAML and C# usage, MVVM binding via ItemsSource/ColumnsSource/BandsSource, and layout serialization/export.
---

# DataGridControl

`DataGridControl` is Eremex's tabular data grid: bind a collection, get auto-generated columns (or define your own `GridColumn`s), and the user can sort, group (via a drag group panel), filter (column filter drop-downs and an optional auto-filter row), edit cells with in-place editors, select rows, reorder rows by drag-and-drop, freeze columns, best-fit, and export to XLSX/PDF/CSV. (Docs: `https://eremexcontrols.net/`; package: `https://www.nuget.org/packages/Eremex.Avalonia.Controls`; runnable examples: `https://github.com/Eremex/controls-demo`.)

> This skill documents the **public** API. Behavioral descriptions are derived from the public surface; confirm exact nuances against the docs or NuGet package when in doubt.

## Namespace

```csharp
using Eremex.AvaloniaUI.Controls.DataGrid;      // DataGridControl, GridColumn, GridBand, GridColumnCollection, GridBandCollection, DataGrid*EventArgs
using Eremex.AvaloniaUI.Controls.DataControl;   // base members: ColumnBase props, enums, DataControlCommands, InvalidCellValueExceptionEventArgs
```

In XAML:

```xml
xmlns:dg="clr-namespace:Eremex.AvaloniaUI.Controls.DataGrid;assembly=Eremex.Avalonia.Controls"
```

> The runtime namespace is `Eremex.AvaloniaUI.Controls.DataGrid` (one word: `AvaloniaUI`), **not** the package name `Eremex.Avalonia.Controls`. See the `_shared` skill for why. The `dg:` prefix is conventional but arbitrary.

## Minimal XAML

Bind `ItemsSource` and let columns auto-generate, or declare them explicitly. `DataGridControl.Columns` is the `[Content]` property:

```xml
<UserControl xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:dg="clr-namespace:Eremex.AvaloniaUI.Controls.DataGrid;assembly=Eremex.Avalonia.Controls">

  <!-- Explicit columns, multi-select, auto-filter row, grouping enabled -->
  <dg:DataGridControl x:Name="grid"
                      ItemsSource="{Binding Orders}"
                      SelectionMode="Multiple"
                      ShowAutoFilterRow="True"
                      AllowSorting="True"
                      AllowGrouping="True"
                      AutoGenerateColumns="False">

    <dg:GridColumn FieldName="Id"       Header="ID"     Width="60" ReadOnly="True"/>
    <dg:GridColumn FieldName="Customer" Header="Customer"/>
    <dg:GridColumn FieldName="Date"     Header="Date"/>
    <dg:GridColumn FieldName="Amount"   Header="Amount"/>
  </dg:DataGridControl>
</UserControl>
```

Just bind `ItemsSource` and leave `AutoGenerateColumns="True"` (the default is `False` — set it explicitly) for a zero-config grid.

Per-column and per-band property tables, bands, fixed columns, and templates are in [columns-and-bands.md](references/columns-and-bands.md).

## Rows, indexes, and the data model

The grid tracks **three** row coordinates — keep them straight:

- **`sourceItemIndex`** — the object's position in the bound `ItemsSource`. Stable across sorting/filtering/grouping.
- **`rowIndex`** — the row's coordinate inside the grid. Data rows are `0`-based; **group rows use negative indexes** (`-1`, `-2`, …). Shifts when rows are sorted/filtered.
- **`visibleRowIndex`** — the row's on-screen position among the currently visible rows (also changes when groups are collapsed/expanded).

Group rows have a `rowIndex` but **no** `sourceItemIndex` (they aren't data rows, so the source-index converters return `-1` for them). `DataGridControl.InvalidRowIndex` (`int.MinValue`) means "no row".

Convert between the three, and read/write cells, with these public methods:

| Method | Returns | Notes |
|--------|---------|-------|
| `GetSourceItemIndexByRowIndex(int rowIndex)` | `int` | `rowIndex` → `sourceItemIndex` (`-1` for group rows). |
| `GetRowIndexBySourceItemIndex(int sourceItemIndex)` | `int` | `sourceItemIndex` → `rowIndex`. |
| `GetSourceItemIndexByVisibleRowIndex(int visibleRowIndex)` | `int` | `visibleRowIndex` → `sourceItemIndex`. |
| `GetVisibleRowIndexBySourceItemIndex(int sourceItemIndex)` | `int` | `sourceItemIndex` → `visibleRowIndex`. |
| `GetVisibleRowIndexByRowIndex(int rowIndex)` | `int` | `rowIndex` → `visibleRowIndex`. |
| `GetRowIndexByVisibleRowIndex(int visibleRowIndex)` | `int` | `visibleRowIndex` → `rowIndex`. |
| `GetSourceItem(int sourceItemIndex)` | `object?` | The bound object at a source index. |
| `GetSourceItemByRowIndex(int rowIndex)` / `GetSourceItemByVisibleRowIndex(int)` | `object?` | Bound object by row/visible index. |
| `GetCellValue(int rowIndex, GridColumn | string fieldName)` | `object?` | Read a cell value. |
| `SetCellValue(int rowIndex, GridColumn | string fieldName, object? value)` | `void` | Write a cell value. |
| `GetCellDisplayText(int rowIndex, GridColumn | string fieldName)` | `string?` | Formatted display text of a cell. |
| `GetSourceItemValue(int sourceItemIndex, GridColumn | string fieldName)` | `object?` | Value straight from the source object. |
| `FindRowByItem(object? item)` | `int` | `rowIndex` holding this source object (`-1` if none). |
| `FindRowByValue(GridColumn | string fieldName, object? value)` | `int` | First `rowIndex` whose cell equals `value`. |
| `FindRowByDisplayText(GridColumn | string fieldName, string? displayText)` | `int` | First `rowIndex` whose display text matches. |
| `IsGroupRow(int rowIndex)` | `bool` | Is this `rowIndex` a group header row? |
| `RefreshData()` / `RefreshRow(int rowIndex)` | `void` | Force a data refresh (re-sort/filter) or refresh one row's cells. |
| `PopulateColumns()` | `void` | (Re)generate columns from the current `ItemsSource` type. |

Group-row traversal (`ExpandGroupRow`, `GetGroupRowValue`, `GetGroupChildRowCount`, `GetGroupChildRowIndex`, …) is in [sorting-grouping-filtering.md](references/sorting-grouping-filtering.md#grouping).

## DataGridControl properties

### Columns, bands & headers

| Property | Type | Default | Notes |
|----------|------|---------|-------|
| `Columns` | `GridColumnCollection` | empty | `[Content]`. Explicit columns. |
| `Bands` | `GridBandCollection` | empty | Column-grouping bands. |
| `ColumnsSource` | `IEnumerable?` | — | MVVM: generate columns from a collection (`ColumnTemplate` builds each). |
| `ColumnTemplate` | `ITemplate<object, GridColumn>?` | — | Factory for `ColumnsSource` items. |
| `BandsSource` | `IEnumerable?` | — | MVVM: generate bands from a collection (`BandTemplate` builds each). |
| `BandTemplate` | `ITemplate<object, GridBand>?` | — | Factory for `BandsSource` items. |
| `AutoGenerateColumns` | `bool` | `false` | Generate columns from the source type. |
| `AutoGenerateBands` | `bool` | `true` | Auto-wrap auto-generated columns in a band. |
| `ShowColumnHeaders` | `bool` | `true` | Show the column header panel. |
| `ShowBands` | `bool` | `true` | Show the band header panel. |
| `ShowBandSeparators` | `bool` | `false` | Draw separators between bands. |
| `AllowBandResizing` | `bool` | `true` | Let the user resize bands. |
| `HeaderPanelMinHeight` | `double` | theme default | Minimum height of the header panel. |
| `CellTemplate` | `IDataTemplate?` | — | Default cell template applied to all columns (column `CellTemplate` wins). |
| `ColumnMenu` | `PopupMenu?` | — | Context menu for column headers. |
| `RowCellMenu` | `PopupMenu?` | — | Context menu for data cells. |
| `ShowColumnMenuFixedItem` | `bool` | `false` | Show the "fixed" item in the column menu. |

### Sorting, grouping, filtering

| Property | Type | Default | Notes |
|----------|------|---------|-------|
| `AllowSorting` | `bool` | `true` | Allow click/header sorting. |
| `AllowGrouping` | `bool` | `true` | Allow dragging columns to the group panel. |
| `AllowColumnFiltering` | `bool` | `true` | Show column filter drop-downs. |
| `ColumnFilterButtonDisplayMode` | `ColumnFilterButtonDisplayMode` | `Auto` | `Auto` / `Always` — filter button visibility. |
| `ColumnFilterPopupMode` | `FilterPopupMode` | `List` | `List` (single-select) / `CheckedList` (multi-select) filter popups. |
| `ShowAutoFilterRow` | `bool` | `false` | Show the per-column auto-filter row. |
| `ShowConditionInAutoFilterRow` | `bool` | `false` | Show the condition selector in the auto-filter row. |
| `ShowGroupPanel` | `bool` | `true` | Show the drag-to-group panel. |
| `ShowGroupedColumns` | `bool` | `false` | Keep grouped columns visible in the row area (instead of hiding them). |
| `GroupCount` | `int` | `0` | Number of grouped columns (top of sort order). |
| `AutoExpandAllGroups` | `bool` | `false` | Expand all group rows after grouping. |
| `FilterPanelDisplayMode` | `FilterPanelDisplayMode` | `Auto` | `Auto` / `Never` — the active-filter panel. |
| `IsFilterPanelVisible` | `bool` | — | Read-only. Whether the filter panel is currently shown. |
| `FilterPanelText` | `string?` | — | Read-only. Text describing the active filter. |

Filtering details (the auto-filter row, filter conditions, `ClearAllColumnsFilter`, `CustomRowFilter`) are in [sorting-grouping-filtering.md](references/sorting-grouping-filtering.md).

### Editing & navigation

| Property | Type | Default | Notes |
|----------|------|---------|-------|
| `AllowEditing` | `bool` | `true` | Master switch for in-place editing (columns can override). |
| `EditorShowMode` | `EditorShowMode` | `PointerPressed` | When the editor activates. |
| `EditorButtonShowMode` | `EditorButtonShowMode` | `ShowOnlyInEditor` | When editor buttons show. |
| `NavigationMode` | `NavigationMode` | `Cell` | `Cell` or `Row` keyboard navigation. |
| `FocusedColumn` | `GridColumn?` | — | Two-way. The focused column. |
| `FocusedRowIndex` | `int` | `InvalidRowIndex` | Two-way. The focused (grid) row index. |
| `AutoScrollToFocusedRow` | `bool` | `true` | Keep the focused row scrolled into view. |

Editor lifecycle (`ShowingEditor`/`ShownEditor`/`HiddenEditor`, `ShowEditor`/`HideEditor`/`PostEditor`/`CommitEditing`, value validation) is in [data-selection-editing.md](references/data-selection-editing.md#editing).

### Selection

| Property | Type | Default | Notes |
|----------|------|---------|-------|
| `SelectionMode` | `RowSelectionMode` | `Single` | `Single` / `Multiple`. |
| `SelectedItems` | `IList?` | — | Two-way. Bound selected-rows list (multiple mode). |
| `KeepSelectedOnClick` | `bool` | `false` | Preserve selection on click. |

Selection methods (`SelectRow`, `UnselectRow`, `SelectRange`, `ClearSelection`, `SelectAll`, `GetSelectedRowIndexes`) are in [data-selection-editing.md](references/data-selection-editing.md#selection).

### Drag-and-drop, fixed columns, appearance

| Property | Type | Default | Notes |
|----------|------|---------|-------|
| `AllowDragDrop` | `bool` | `false` | Enable row drag-and-drop reordering. |
| `RowDragMode` | `RowDragMode` | `Row` | `Row` (whole row) / `DragHandle` (drag from the row indicator). |
| `UsePlatformRowDragDrop` | `bool` | `false` | Use the platform drag-drop machinery. |
| `AllowDragDropSortedRows` | `bool` | `false` | Allow D&D when rows are sorted. |
| `AllowScrollingOnDrag` | `bool` | `true` | Auto-scroll while dragging. |
| `AutoExpandOnDrag` | `bool` | `true` | Auto-expand group rows while dragging. |
| `AutoExpandDelayOnDrag` | `double` | theme default | Delay before auto-expand on drag. |
| `RowIndicatorWidth` | `double` | `17` | Width of the row indicator/handle column. |
| `ShowRowIndicator` | `bool` | — | Read-only. Visible only with `RowDragMode=DragHandle` + `AllowDragDrop`. |
| `FixedColumnSeparatorWidth` | `double` | `3` | Width of the fixed/non-fixed separator. |
| `FixedLeftColumnWidth` / `FixedRightColumnWidth` | `double` | — | Read-only. Total width of left/right fixed columns. |
| `ExtendScrollbarToFixedColumns` | `bool` | `false` | Let the horizontal scrollbar span fixed columns. |
| `ShowHorizontalLines` / `ShowVerticalLines` | `bool` | `true` | Grid lines. |
| `AllowColumnResizing` | `bool` | `true` | Resize columns by dragging separators. |
| `AllowColumnMoving` | `bool` | `true` | Reorder columns by dragging headers. |
| `AllowHorizontalVirtualization` | `bool` | `true` | Virtualize columns horizontally. |
| `RowLevelIndent` | `double` | theme default | Indent per group/hierarchy level. |
| `RowMinHeight` | `double` | theme default | Minimum data-row height. |
| `ClipboardCopyHeaders` | `bool` | `true` | Include column headers when copying to clipboard. |

Fixed columns are set per-column via `GridColumn.FixedMode` (`None` / `Left` / `Right`) — see [columns-and-bands.md](references/columns-and-bands.md#fixed-columns--best-fit).

## DataGridControl events

| Event | Args (properties) | Cancelable? | Notes |
|-------|-------------------|-------------|-------|
| `SelectionChanged` | `RowIndex`, `ChangeType` (`CollectionChangeAction`) | — | Row selection changed. |
| `ShowingEditor` | `RowIndex`, `Column`, **`Cancel`** | ✅ | Before an editor opens; veto per cell. |
| `ShownEditor` | `RowIndex`, `Column`, `Editor` | — | Editor just opened. |
| `HiddenEditor` | `RowIndex`, `Column`, `Editor` | — | Editor closed. |
| `CellValueChanging` | `RowIndex`, `Column`, `Value`, `OldValue` | — | A cell value is being changed by the editor. |
| `CellValueChanged` | `RowIndex`, `Column`, `Value`, `OldValue` | — | A cell value changed. |
| `ValidateCellValue` | `RowIndex`, `Column`, `Value`, **`ErrorContent`** | ✅ | Validate an editor value; set `ErrorContent` to reject. |
| `InvalidCellValueException` | exception info | — | An invalid value threw during commit. |
| `RowClick` | `RowIndex`, `ClickCount`, `IsLeftButtonPressed` | — | A row was clicked. |
| `StartSorting` / `EndSorting` | `RoutedEventArgs` | — | A sort operation started/ended. |
| `FilterChanged` | `RoutedEventArgs` | — | The active filter changed. |
| `AutoGeneratingColumn` | `Column`, **`Cancel`** | ✅ | For each auto-generated column; mutate `Column` or cancel. |
| `AutoGeneratedColumns` | `EventArgs` | — | All auto-generated columns are ready. |
| `CustomUnboundColumnData` | `Item`, `Column`, `Value`, `IsGettingData` | — | Provide/read values for `UnboundDataType` columns. |
| `CustomColumnDisplayText` | `SourceItemIndex`, `Column`, `Value`, `DisplayText` | — | Override a cell's display text. |
| `CustomColumnSort` | `Column`, `SourceItemIndex1/2`, `Value1/2`, **`Result`** (`int?`) | — | Custom sort comparison. |
| `CustomColumnGroup` | …same, **`Result`** (`bool?`) | — | Custom grouping comparison. |
| `CustomGroupValueDisplayText` | `RowIndex`, `Column`, `Value`, `DisplayText` | — | Text for a group row. |
| `CustomRowFilter` | `SourceItemIndex`, **`Visible`** | — | Decide per row whether it passes the filter. |
| `StartDrag` | `Items`, `DragRowIndexes`, `Data`, `Effects` | — | A row drag started. |
| `DragOver` | `Items`, `TargetRowIndex`, `DropPosition`, `KeyModifiers`, `Effects` | — | Drag hovering over a drop target. |
| `Drop` | `Items`, `TargetRowIndex`, `DropPosition`, `Data`, `Effects` | — | Rows dropped. |
| `CompleteDragDrop` | `Items`, `Effects` | — | Drag-and-drop finished. |

Event handler patterns (veto an editor, validate a value, custom sort/filter) are in [data-selection-editing.md](references/data-selection-editing.md#events).

## Enums (quick reference)

| Enum | Members |
|------|---------|
| `NavigationMode` | `Cell`, `Row` |
| `RowSelectionMode` | `Single`, `Multiple` |
| `SortMode` | `Value`, `DisplayText`, `Custom` |
| `ColumnFilterMode` | `Value`, `DisplayText` |
| `AutoFilterCondition` | `Default`, `Equals`, `DoesNotEqual`, `Contains`, `DoesNotContain`, `StartsWith`, `EndsWith`, `Greater`, `GreaterOrEqual`, `Less`, `LessOrEqual` |
| `SearchPanelDisplayMode` | `HotKey`, `Always`, `Never` |
| `EditorShowMode` | `PointerPressed`, `PointerPressedInFocusedCell`, `PointerReleased`, `PointerReleasedInFocusedCell` |
| `EditorButtonShowMode` | `ShowOnlyInEditor`, `ShowForFocusedCell`, `ShowForFocusedRow`, `ShowAlways` |
| `FilterPopupMode` | `List`, `CheckedList` |
| `FilterPanelDisplayMode` | `Auto`, `Never` |
| `ColumnFilterButtonDisplayMode` | `Auto`, `Always` |
| `TextSortMode` | `Alphabetical`, `Alphanumeric` |
| `TextExportMode` | `Value`, `Text` |
| `FixedMode` | `None`, `Left`, `Right` |
| `BestFitMode` | `Full`, `Fast` |
| `RowDragMode` | `Row`, `DragHandle` |
| `DropPosition` | `Before`, `After`, `Inside` |
| `ColumnHeaderPlacement` | `HeaderPanel`, `GroupPanel`, `ColumnChooser` |

## Commands

`grid.Commands` (a `DataGridControlCommands`) exposes standard grid actions as `ICommand` — bindable to buttons/menu items. Most column-targeted commands take the target `GridColumn` as the command parameter.

| Command | Target | Action |
|---------|--------|--------|
| `SortColumnAscending` / `SortColumnDescending` / `ClearColumnSorting` | column | Sort / clear sort. |
| `SetFixedModeLeft` / `SetFixedModeNone` / `SetFixedModeRight` | column | Change fixed mode. |
| `BestFitColumn` / `BestFitAllColumns` | column / — | Best-fit a column / all columns. |
| `ResetColumnWidth` | — | Restore column widths. |
| `ShowSearchPanel` / `HideSearchPanel` | — | Toggle the search panel. |
| `ShowColumnChooser` / `HideColumnChooser` | — | Toggle the column chooser. |
| `ShowGroupPanel` / `HideGroupPanel` | — | Toggle the group panel. |
| `GroupByColumn` / `UngroupByColumn` | column | Group / ungroup by a column. |
| `ExpandAllGroups` / `CollapseAllGroups` | — | Expand / collapse all group rows. |

```xml
<Menu>
  <MenuItem Header="Sort ↑"  Command="{Binding #grid.Commands.SortColumnAscending}"
            CommandParameter="{Binding #grid.FocusedColumn}"/>
  <MenuItem Header="Best Fit" Command="{Binding #grid.Commands.BestFitColumn}"
            CommandParameter="{Binding #grid.FocusedColumn}"/>
  <MenuItem Header="Chooser"  Command="{Binding #grid.Commands.ShowColumnChooser}"/>
</Menu>
```

## Quick C# examples

Bind data, sort/group programmatically, react to selection and value changes:

```csharp
using Eremex.AvaloniaUI.Controls.DataGrid;

grid.ItemsSource = orders;

// Focus a specific row by source object:
grid.FocusedRowIndex = grid.GetRowIndexBySourceItemIndex(42);

// Read & write cell values:
var name = grid.GetCellValue(grid.FocusedRowIndex, "Customer");
grid.SetCellValue(grid.FocusedRowIndex, "Amount", 0m);

// Selection (multiple mode):
grid.SelectRange(0, 9);
var selected = grid.GetSelectedRowIndexes();   // List<int>

// Events:
grid.SelectionChanged += (_, e) => Console.WriteLine($"Row {e.RowIndex} {e.ChangeType}");
grid.ValidateCellValue += (_, e) =>
{
    if (e.Column.FieldName == "Amount" && (decimal)e.Value! < 0)
        e.ErrorContent = "Amount must be non-negative.";
};
grid.CellValueChanged += (_, e) => Save(e.RowIndex, e.Column.FieldName, e.Value);
```

Group programmatically and walk group rows:

```csharp
grid.GroupCount = 1;                       // group by the first sorted column
grid.ExpandAllGroups();

bool expanded = grid.IsGroupRowExpanded(rowIndex);
var groupValue = grid.GetGroupRowValue(rowIndex);
int children  = grid.GetGroupChildRowCount(rowIndex);
```

## Save / restore layout (quick)

```csharp
using System.IO;
using System.Text;
using Eremex.AvaloniaUI.Controls.Serialization;

// Save column layout (widths, order, visibility, sort, group, filter) to XML:
string xml;
using (var ms = new MemoryStream()) { grid.SaveLayout(ms); xml = Encoding.UTF8.GetString(ms.ToArray()); }

// Restore later:
using (var ms = new MemoryStream(Encoding.UTF8.GetBytes(xml))) grid.RestoreLayout(ms);
```

> `SaveLayout(Stream)` / `RestoreLayout(Stream)` persist the user's column configuration. JSON mode is available via `SerializationSettings`. See [sorting-grouping-filtering.md](references/sorting-grouping-filtering.md) for what is persisted.

## Export (quick)

```csharp
grid.ExportToXlsx("orders.xlsx");                       // or a Stream
grid.ExportToPdf("orders.pdf");                         // or a Stream + PageExportOptions
grid.ExportToCsv("orders.csv", textExportNode: TextExportMode.Text, separator: ",");
grid.ExportToImages(dir, "page-{0}", options);          // per-page images
await grid.CopyToClipboardAsync();                      // copy selection/rows to clipboard
```

`ExportOptions` (`XlsxExportOptions`, `PageExportOptions`, `ImageExportOptions`) and per-column `AllowExport` / `TextExportMode` control the output.

## MVVM

- **Data:** bind `ItemsSource` to an `ObservableCollection`/`IList`.
- **Selection:** bind `SelectedItems` (multiple mode) for a selected-rows list in your VM.
- **Focus:** `FocusedRowIndex` and `FocusedColumn` are two-way bindable.
- **Columns/Bands from a VM:** set `ColumnsSource` + `ColumnTemplate` (and `BandsSource` + `BandTemplate`) instead of declaring `Columns` inline. `PopulateColumns()` regenerates columns from the source type.
- **Editing & validation:** handle `ValidateCellValue` / `CellValueChanged`, or `IDataErrorInfo`/`INotifyDataErrorInfo` on your row objects (the grid surfaces errors via `ShowItemsSourceErrors`).

## Related skills

- `_shared` — namespace gotcha, theming via DeltaDesign, pseudoclasses, localization.
- `dock-manager` — `DockManager` if you need to embed the grid in a dockable tool/document pane.
