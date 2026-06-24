# DataGridControl — columns & bands

The column/band model in depth: `GridColumn` and `GridBand`, the shared base members, cell/header templates, fixed columns, best-fit, and the column chooser. API reference in [../SKILL.md](../SKILL.md); sorting/grouping/filtering in [sorting-grouping-filtering.md](sorting-grouping-filtering.md); data/selection/editing in [data-selection-editing.md](data-selection-editing.md).

Namespace for everything below:

```csharp
using Eremex.AvaloniaUI.Controls.DataGrid;      // GridColumn, GridBand, GridColumnCollection, GridBandCollection
using Eremex.AvaloniaUI.Controls.DataControl;   // ColumnBase, DataControlColumnBase, DataControlBandBase (base members)
```

All column/band types derive (directly or indirectly) from Avalonia `StyledElement` — they are **logical** elements, not visuals. Defaults shown come from the property registrations; many column flags are `bool?` (tri-state) so `null` means "inherit the grid's setting".

## Type hierarchy

```
StyledElement
└── DataControlColumnBase        (Header, header template/alignment, tooltip, ActualCaption)
    ├── ColumnBase               (FieldName, Width, sorting/filtering/editing/fixed, templates, …)
    │   └── GridColumn           (+ AllowGrouping)   ← the column you declare
    └── DataControlBandBase      (BandName)
        └── GridBand             (+ Bands / BandsSource)  ← the band you declare
```

Collections: `GridColumnCollection` (the grid's `Columns`) and `GridBandCollection` (a band's `Bands`).

## `GridColumn`

The cell-defining element. Set `FieldName` to bind to a property on the row object; leave it unset for an **unbound** column (set `UnboundDataType` and supply values via `CustomUnboundColumnData`).

### Identity & content

| Property | Type | Default | Notes |
|----------|------|---------|-------|
| `FieldName` | `string?` | — | Bound property/path on the row object. The data key for this column. |
| `Header` | `object` | — | Header caption (string or any content). |
| `HeaderTemplate` | `IDataTemplate` | — | Template for `Header`. |
| `HeaderToolTip` | `object` | — | Tooltip on the header. |
| `HeaderHorizontalAlignment` / `HeaderVerticalAlignment` | `HorizontalAlignment` / `VerticalAlignment` | `Left` / `Center` | Header alignment. |
| `ActualCaption` | `object` | — | Read-only. Resolved header (`Header`, or `FieldName` when auto-generated). |
| `CellTemplate` | `IDataTemplate?` | — | Per-column cell template (overrides the grid's `CellTemplate`). |
| `AutoFilterRowCellTemplate` | `IDataTemplate?` | — | Cell template used in the auto-filter row. |

### Layout

| Property | Type | Default | Notes |
|----------|------|---------|-------|
| `Width` | `GridLength` | `*` (theme default px) | Absolute, Auto, or star sizing. |
| `MinWidth` | `double` | `20` | Coerced to ≥ 20. |
| `MaxWidth` | `double` | `+∞` | |
| `VisibleIndex` | `int` | `-1` | Display order; reassigned automatically as columns move. |
| `IsVisible` | `bool` | `true` | Show/hide without removing from `Columns`. |
| `FixedMode` | `FixedMode` | `None` | `None` / `Left` / `Right` — freeze the column. |
| `ActualVisibleIndex` | `int` | `-1` | Read-only. Final visible position. |
| `ActualWidth` | `double` | — | Read-only. Rendered width. |
| `HeaderWidth` / `CellWidth` | `double` | — | Resolved header / cell width. |
| `IsResizeGripVisible` | `bool` | — | Read-only. Whether the resize grip shows. |
| `HasBandSeparator` | `bool` | — | Read-only. Separator visibility at the band edge. |

### Behavior flags (tri-state `bool?` ⇒ `null` = inherit from grid)

| Property | Type | Default | Notes |
|----------|------|---------|-------|
| `AllowSorting` | `bool?` | `null` | Can this column be sorted? |
| `AllowGrouping` | `bool?` | `null` | Can this column be dragged to the group panel? (`GridColumn`-only override.) |
| `AllowColumnFiltering` | `bool?` | `null` | Show the column filter drop-down? |
| `AllowEditing` | `bool?` | `null` | Can cells be edited? |
| `AllowResizing` | `bool?` | `null` | Can the user resize this column? |
| `AllowMoving` | `bool?` | `null` | Can the user reorder this column? |
| `AllowBestFit` | `bool?` | `null` | Can this column be best-fit? |
| `AllowImmediateEditorValuePosting` | `bool?` | `null` | Post editor value as it changes (vs on commit). |
| `ShowConditionInAutoFilterRow` | `bool?` | `null` | Show the condition selector for this column. |
| `ReadOnly` | `bool` | `false` | Read-only display (forces no editing for this column). |
| `AllowFocus` | `bool` | `true` | Can the column receive keyboard focus? |
| `IsTabStop` | `bool` | — | Part of Tab navigation. |

### Sorting & filtering state

| Property | Type | Default | Notes |
|----------|------|---------|-------|
| `SortDirection` | `ListSortDirection?` | `null` | `Ascending` / `Descending` / `null` (unsorted). Two-way. |
| `SortIndex` | `int` | `-1` | Position in the sort order (`-1` = not sorted). |
| `SortMode` | `SortMode?` | `null` | `Value` / `DisplayText` / `Custom`. |
| `ColumnFilterMode` | `ColumnFilterMode` | `Value` | `Value` / `DisplayText`. |
| `FilterPopupMode` | `FilterPopupMode?` | `null` | `List` / `CheckedList` per column. |
| `AutoFilterValue` | `object?` | — | Value typed in the auto-filter row (two-way). |
| `AutoFilterCondition` | `AutoFilterCondition` | `Default` | Condition for the auto-filter value. |
| `IsFiltered` | `bool` | — | Read-only. Whether a filter is active on this column. |
| `RoundDateTimeForColumnFilter` | `bool` | `true` | Ignore time portion when building date filter items. |

> Sorting/grouping are driven by the grid's sorted-columns collection: setting `SortDirection`/`SortIndex` participates in it. See [sorting-grouping-filtering.md](sorting-grouping-filtering.md).

### Editing & unbound data

| Property | Type | Default | Notes |
|----------|------|---------|-------|
| `EditorProperties` | `BaseEditorProperties?` | — | Configure the in-place editor (mask, format, buttons, …). |
| `ActualEditorProperties` | `BaseEditorProperties?` | — | Read-only. Merged effective editor settings. |
| `UnboundDataType` | `Type?` | — | Declare an unbound column; supply values via `CustomUnboundColumnData`. |
| `DataType` | `Type` | `object` | Read-only. Resolved data type of the field. |

### Column chooser & export

| Property | Type | Default | Notes |
|----------|------|---------|-------|
| `ShowInColumnChooser` | `bool` | `true` | List this column in the column chooser dialog. |
| `ColumnChooserHeader` | `object?` | — | Caption shown in the column chooser. |
| `BandName` | `string?` | — | Place this column under the band with this `BandName`. |
| `AllowExport` | `bool` | `true` | Include in exports. |
| `TextExportMode` | `TextExportMode?` | — | `Value` / `Text` for this column's export. |

### Best-fit

| Property | Type | Default | Notes |
|----------|------|---------|-------|
| `BestFitMode` | `BestFitMode?` | `null` | `Full` (measure all rows) / `Fast` (visible rows). Overrides grid `BestFitMode`. |

## `GridBand`

A band groups one or more columns (and/or nested bands) under a shared header, forming a multi-row header hierarchy.

| Property / member | Type | Default | Notes |
|-------------------|------|---------|-------|
| `Header` | `object` | — | Band caption (inherited from `DataControlColumnBase`). |
| `BandName` | `string` | — | Stable name; columns reference a band via their `BandName`. |
| `Bands` | `GridBandCollection` | empty | `[Content]`. Nested child bands. |
| `BandsSource` | `IEnumerable?` | — | MVVM: generate child bands from a collection. |
| `HeaderTemplate` / `HeaderHorizontalAlignment` / `HeaderVerticalAlignment` | — | — | Inherited header styling. |

> Place a column in a band either by setting the column's `BandName` to the band's `BandName`, or by nesting. Bands can contain bands, so headers can span multiple rows.

## Declaring columns in XAML

```xml
<dg:DataGridControl ItemsSource="{Binding Orders}" AutoGenerateColumns="False">

  <!-- Bound column with a numeric editor and right alignment -->
  <dg:GridColumn FieldName="Amount" Header="Amount">
    <dg:GridColumn.EditorProperties>
      <editors:SpinEditorProperties Mask="c" />
    </dg:GridColumn.EditorProperties>
  </dg:GridColumn>

  <!-- Unbound column: computed in code via CustomUnboundColumnData -->
  <dg:GridColumn FieldName="Surcharge" Header="Surcharge"
                 UnboundDataType="x:Type sys:Decimal" ReadOnly="True"/>

  <!-- A templated column -->
  <dg:GridColumn FieldName="Status" Header="Status">
    <dg:GridColumn.CellTemplate>
      <DataTemplate>
        <Border Background="{Binding Status, Converter={StaticResource StatusToBrush}}"
                CornerRadius="8" Padding="6,2">
          <TextBlock Text="{Binding Status}"/>
        </Border>
      </DataTemplate>
    </dg:GridColumn.CellTemplate>
  </dg:GridColumn>
</dg:DataGridControl>
```

```csharp
grid.CustomUnboundColumnData += (_, e) =>
{
    if (e.IsGettingData && e.Column.FieldName == "Surcharge")
    {
        var amount = (decimal)e.Item.GetType().GetProperty("Amount")!.GetValue(e.Item)!;
        e.Value = amount * 0.1m;
    }
};
```

## Declaring bands in XAML

```xml
<dg:DataGridControl ItemsSource="{Binding Orders}" AutoGenerateColumns="False" ShowBands="True">
  <dg:DataGridControl.Bands>
    <dg:GridBand Header="Customer">
      <!-- columns can sit here too, or reference the band by name -->
      <dg:GridBand Header="Contact">
        <!-- nested band -->
      </dg:GridBand>
    </dg:GridBand>
    <dg:GridBand Header="Financial"/>
  </dg:DataGridControl.Bands>

  <dg:GridColumn FieldName="Name"    Header="Name"    BandName="Customer"/>
  <dg:GridColumn FieldName="Email"   Header="Email"   BandName="Contact"/>
  <dg:GridColumn FieldName="Amount"  Header="Amount"  BandName="Financial"/>
</dg:DataGridControl>
```

## Auto-generation

When `AutoGenerateColumns="True"`, the grid creates a `GridColumn` per public property of the source row type and raises `AutoGeneratingColumn` for each — cancel it (`e.Cancel = true`) or tweak `e.Column` (rename, hide, mark read-only, attach an editor). After all are created it raises `AutoGeneratedColumns`. Call `PopulateColumns()` to regenerate at runtime (e.g. after the source type changes).

```csharp
grid.AutoGeneratingColumn += (_, e) =>
{
    if (e.Column.FieldName == "Secret")
        e.Cancel = true;
    else if (e.Column.FieldName == "Amount")
        e.Column.Header = "Total";
};
```

## Fixed columns & best-fit

- **Freeze** a column with `FixedMode`: `Left` pins it to the left edge, `Right` to the right. Fixed columns stay visible while the rest scroll horizontally. The fixed/non-fixed divider width is `grid.FixedColumnSeparatorWidth`; `grid.ExtendScrollbarToFixedColumns` lets the scrollbar cover fixed columns.
- **Best-fit** sizes a column to its content:
  - `grid.BestFit(column)` / `grid.BestFitAllColumns()`.
  - Or the commands `grid.Commands.BestFitColumn` (param = column) / `BestFitAllColumns`.
  - `grid.BestFitMode` / `column.BestFitMode` choose `Full` (all rows, slower) vs `Fast` (visible rows only). `grid.AllowBestFit` / `column.AllowBestFit` gate the feature.
- `grid.ResetColumnWidth()` (or `grid.Commands.ResetColumnWidth`) restores columns to their initial widths.

## Column chooser

A built-in dialog lets users show/hide columns:

```csharp
grid.ShowColumnChooser();   // open
grid.HideColumnChooser();   // close
// or: grid.Commands.ShowColumnChooser / HideColumnChooser
```

Control per-column visibility in the dialog with `ShowInColumnChooser` and the displayed text with `ColumnChooserHeader`.

## MVVM columns/bands

Generate columns/bands from view-model collections instead of declaring them:

```xml
<dg:DataGridControl ItemsSource="{Binding Rows}"
                    ColumnsSource="{Binding ColumnDescriptors}"
                    ColumnTemplate="{StaticResource ColumnTemplate}"
                    BandsSource="{Binding BandDescriptors}"
                    BandTemplate="{StaticResource BandTemplate}"/>
```

`ColumnTemplate` is an `ITemplate<object, GridColumn>` — build and return a configured `GridColumn` from each descriptor; `BandTemplate` is the `GridBand` equivalent. The grid calls them as the source collections change.
