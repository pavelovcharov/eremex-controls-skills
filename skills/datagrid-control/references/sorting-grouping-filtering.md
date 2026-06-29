# DataGridControl — sorting, grouping & filtering

How rows get ordered, grouped, and filtered in `DataGridControl`: the sorted-columns model, the group panel, the auto-filter row, column filter drop-downs, the active-filter string, search panel, and summaries. API reference in [../SKILL.md](../SKILL.md); columns in [columns-and-bands.md](columns-and-bands.md); data/selection/editing in [data-selection-editing.md](data-selection-editing.md).

```csharp
using Eremex.AvaloniaUI.Controls.DataGrid;
using Eremex.AvaloniaUI.Controls.DataControl;   // enums
```

## Sorting

A column is sorted via its `SortDirection` (`Ascending` / `Descending` / `null`) and its position in the order via `SortIndex` (`-1` = unsorted). Clicking a header (when `grid.AllowSorting` and `column.AllowSorting` aren't disabled) cycles the direction.

| Member | Notes |
|--------|-------|
| `grid.ClearSorting()` | Clear all sort criteria. |
| `grid.Commands.SortColumnAscending` / `SortColumnDescending` / `ClearColumnSorting` | Commands (param = column). |
| `column.SortMode` | `Value` (compare the raw value, default), `DisplayText` (compare formatted text), or `Custom` (raise `CustomColumnSort`). |
| `grid.TextSortMode` | `Alphabetical` or `Alphanumeric` (natural ordering of strings like "file2" < "file10"). |
| `grid.StartSorting` / `EndSorting` | Routed events around a sort operation. |

### Custom sort

Set `column.SortMode = SortMode.Custom` and compare rows yourself. Set `e.Result` to the comparison (`< 0`, `0`, `> 0`):

```csharp
grid.CustomColumnSort += (_, e) =>
{
    if (e.Column.FieldName != "Priority") return;
    e.Result = PriorityRank(e.Value1).CompareTo(PriorityRank(e.Value2));
};
```

> `CustomColumnSort` gives you `Value1`/`Value2` and source indexes; return an `int?` in `Result`. For grouping comparisons use `CustomColumnGroup` (same shape, `Result` is `bool?`).

## Grouping

Grouping is sorting *plus* a `GroupCount`: the first `GroupCount` sorted columns become group keys, and the grid inserts collapsible group rows. Enable grouping with `grid.AllowGrouping`; the user drags a column header into the group panel (`grid.ShowGroupPanel`) to group by it.

| Member | Notes |
|--------|-------|
| `grid.GroupCount` | How many of the leading sorted columns are group keys. |
| `grid.ShowGroupPanel` | Show the drag-to-group panel (default `true`). |
| `grid.ShowGroupedColumns` | Keep grouped columns visible in the row area instead of hiding them. |
| `grid.AutoExpandAllGroups` | Expand all group rows after grouping. |
| `grid.Commands.GroupByColumn` / `UngroupByColumn` | Group/ungroup a column (param = column). |
| `grid.Commands.ShowGroupPanel` / `HideGroupPanel` | Toggle the group panel. |
| `grid.Commands.ExpandAllGroups` / `CollapseAllGroups` | Expand/collapse every group row. |

### Group-row traversal

Group rows occupy their own slots in the row-index space (a `rowIndex` may be a group row — check `grid.IsGroupRow(rowIndex)`).

| Method | Returns |
|--------|---------|
| `ExpandGroupRow(int rowIndex)` / `CollapseGroupRow(int rowIndex)` | Expand/collapse one group row. |
| `IsGroupRowExpanded(int rowIndex)` | `bool`. |
| `GetGroupRowValue(int rowIndex)` | The group key value. |
| `GetGroupChildRowCount(int rowIndex)` / `GetGroupChildRowCount()` | Child data-row count of a group / of the whole grid. |
| `GetGroupChildRowIndex(int rowIndex, int childIndex)` / `GetGroupChildRowIndex(int childIndex)` | A child row's index. |
| `GetParentRowIndex(int rowIndex)` | The enclosing group row. |
| `GetRowLevel(int rowIndex)` | Nesting level of a row. |

```csharp
grid.GroupCount = 1;          // group by the first sorted column
grid.ExpandAllGroups();

for (int i = 0; i < grid.VisibleRowCount; i++)
{
    if (grid.IsGroupRow(i))
        Console.WriteLine($"group: {grid.GetGroupRowValue(i)} ({grid.GetGroupChildRowCount(i)} rows)");
}
```

### Custom group display text

Override the text shown for a group row:

```csharp
grid.CustomGroupValueDisplayText += (_, e) =>
{
    if (e.Column.FieldName == "Status")
        e.DisplayText = $"Status: {e.Value}";
};
```

Use `CustomColumnGroup` to define *which* rows belong together (e.g. group by value range).

## Filtering

Three filtering mechanisms coexist; they combine with AND.

### 1. Column filter drop-downs

When `grid.AllowColumnFiltering` (and `column.AllowColumnFiltering`) are on, each header shows a filter button (`ColumnFilterButtonDisplayMode`: `Auto`/`Always`) opening a popup.

| Member | Notes |
|--------|-------|
| `grid.ColumnFilterPopupMode` / `column.FilterPopupMode` | `List` (single-select) or `CheckedList` (multi-select). |
| `grid.ColumnFilterMode` / `column.ColumnFilterMode` | `Value` or `DisplayText` — what the list items represent. |
| `column.RoundDateTimeForColumnFilter` | Drop the time portion of `DateTime` values in the list. |
| `grid.ClearAllColumnsFilter()` | Clear every column's filter. |
| `column.IsFiltered` | Read-only. Is a filter active on this column? |

### 2. Auto-filter row

`grid.ShowAutoFilterRow = true` shows a top row with a cell per column; typing filters by that column. Per-column:

| Member | Notes |
|--------|-------|
| `column.AutoFilterValue` | The typed value (two-way bindable). |
| `column.AutoFilterCondition` | `Equals`, `Contains`, `StartsWith`, `Greater`, … (see `AutoFilterCondition`). |
| `grid.ShowConditionInAutoFilterRow` / `column.ShowConditionInAutoFilterRow` | Show a condition selector next to the value. |
| `column.AutoFilterRowCellTemplate` | Custom cell template for the auto-filter row. |

### 3. Filter string (whole-grid)

`grid.FilterString` is a string criteria applied to the whole view (build it from your UI, or bind it). `grid.IsFilterEnabled` toggles whether filtering is applied at all. When a filter is active, `IsFilterPanelVisible` reflects it and `FilterPanelText` describes it (`FilterPanelDisplayMode` controls the panel visibility). `grid.FilterChanged` fires whenever the active filter changes.

```xml
<dg:DataGridControl ItemsSource="{Binding Orders}"
                    FilterString="{Binding UserFilter}"
                    IsFilterEnabled="{Binding IsFilterOn}"/>
```

### Custom row filter

Take full control: for each source row decide whether it's visible, regardless of the other filters:

```csharp
grid.CustomRowFilter += (_, e) =>
{
    // Hide archived rows no matter what:
    var order = grid.GetSourceItem(e.SourceItemIndex) as Order;
    if (order is { Archived: true })
        e.Visible = false;
};
```

`e.SourceItemIndex` is the index in `ItemsSource`; set `e.Visible` to keep or drop the row.

## Search panel

A built-in find-as-you-type panel (`grid.SearchPanelDisplayMode`):

| Member | Notes |
|--------|-------|
| `SearchPanelDisplayMode` | `HotKey` (show on Ctrl+F, default), `Always`, `Never`. |
| `SearchText` | The current search term (two-way). |
| `SearchPanelHighlightResults` | Highlight matches. |
| `ShowSearchPanelCloseButton` | Show the close button. |
| `IsSearchPanelVisible` | Read-only visibility state. |
| `Commands.ShowSearchPanel` / `HideSearchPanel` | Toggle programmatically. |

## Summaries

The library ships public types for grid aggregates — `SummaryItem` (with `FieldName`, `SummaryType`, `ShowInColumn`, `IsCustom`) and the `SummaryItemType` enum (`None`, `Sum`, `Min`, `Max`, `Count`, `Average`, `Custom`). The exact way summaries are attached to a grid (the summary collections and the custom-aggregation event) is not part of the documented public surface in the current package version, so confirm the supported configuration path — and whether a custom-summary hook is exposed — against the official docs (`https://eremexcontrols.net/`) and the demo (`https://github.com/Eremex/controls-demo`) before relying on it.

## Save / restore (sorting, grouping, filtering)

`grid.SaveLayout(stream)` persists the user's column configuration — including visible index, width, visibility, `FixedMode`, **sort order, `GroupCount`, and the active filter** — so a restored layout reproduces the sorted/grouped/filtered view. `grid.RestoreLayout(stream)` reapplies it. Use JSON via `SerializationSettings` if preferred.

```csharp
grid.SaveLayout(ms);
// later
grid.RestoreLayout(ms);
```

> Layout keys columns by `FieldName`/`BandName`, so keep those stable across sessions for a reliable restore.
