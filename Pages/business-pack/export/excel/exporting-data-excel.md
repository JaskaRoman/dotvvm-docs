# Excel export
`GridViewExportExcel<T>` from the `DotVVM.BusinessPack.Export.Excel` package allows you to export `GridView` data into an [OpenXML SpreadsheetML](TODO) format document.

DotVVM Excel exporter uses the [ClosedXML](TODO) library for document creation and manipulation. The ClosedXML objects are exposed and can be accessed using extensibility points described below.

While the basic process remains as described in [Exporting data](TODO), additional features are supported by the Excel exporter.
These features are controlled by the `GridViewExportExcelSettings<T>` class.

## Excel export settings
The Excel-specific class `GridViewExportExcelSettings<T>` consists of following instance properties:

|Type|Property|Description|
|---|---|---|
|`ColumnValueProviderHandlers<T>`|`ColumnValueProviderHandlers`|Value providers used to retrieve the data from exported data-context.|
|`ICollection<IExcelExportRule>`|`ExportRules`|Rules applied during the export process.|

## Export rules
Following abstract implementations of `IExcelExportRule` are available. All built-in rules are based on these classes.

|Type|Description|
|---|---|
|`WorksheetRule`|An `IExcelExportRule` targeting the entire worksheet.|
|`ColumnRule`|An `IExcelExportRule` targeting a single column. <br /> Internal column name must be specified. |

Only the last instance of each concrete rule type is applied, unless `IStackableExcelExportRule` is implemented. See **Sample 2**.

Two rule presets are available as static get-only properties of the `GridViewExportExcelSettings<T>` class.
- `Empty` - Creates an instance of `GridViewExportExcelSettings<T>` with an empty rule set.
- `Default` - Creates an instance of `GridViewExportExcelSettings<T>` with an [`AdjustToContentsWorksheetRule`](TODO) and a [`TableHeaderWorksheetRule`](TODO).

When no rules are passed to the `GridViewExportExcelSettings<T>` constructor, the `Default` rule preset is used. This is the case in **Sample 1**.

A **Fluent API** for population of the `ExportRules` collection is available. See **Sample 3**.

##### Sample 1

A simple set of export rules, which will achieve the following results:
- The exported worksheet will be named *“Phone numbers”*.
- The exported phone number column will be stored as `Text`.
- The phone number column header text will be set to *“Phone number 📞”*.

```CSHARP
public class CustomerData
{
    public int PhoneNumber { get; set; }
}
```
```CSHARP
new GridViewExportExcelSettings<CustomerData>
{
    ExportRules =
    {
        new WorksheetNameWorksheetRule("Phone numbers"),
        new DataTypeColumnRule(nameof(CustomerData.PhoneNumber), XLDataType.Text),
        new HeaderTextColumnRule(nameof(CustomerData.PhoneNumber), "Phone number 📞")
    }
}
```

##### Sample 2
This example illustrates the behavior described above. When multiple instances of the same rule type are present, only the last is applied. The resulting phone number column will be stored as a `Number`, because the last instance of `DataTypeColumnRule` for given column name takes precedence over all previous.

```CSHARP
ExportRules =
{
    new DataTypeColumnRule(nameof(CustomerData.PhoneNumber), XLDataType.Text),
    new DataTypeColumnRule(nameof(CustomerData.PhoneNumber), XLDataType.Number)
}
```

##### Sample 3
This sample illustrates the usage of included **Fluent API** for rule construction.
The produced rule set is functionally equal to the rule set constructed in **Sample 1**.

```CSHARP
GridViewExportExcelSettings<CustomerData>.Default
    .WithWorkSheeName("Phone numbers")
    .ForColumn(c => c.PhoneNumber, r => r
        .WithDataType(XLDataType.Text)
        .WithHeaderText("Phone number 📞"));
```
    

### Extensibility points
Each abstract rule class contains multiple overridable virtual methods invoked during the export process.
#### WorksheetRule
|Method|Parameters|Description|
|---|---|---|
|`OnWorksheetExporting`|`IXLWorksheet worksheet`|Called before the worksheet is exported.|
|`OnDataExporting`|`IXLWorksheet worksheet, ExportData exportData`|Called before data is exported.|
|`OnDataExported`|`IXLWorksheet worksheet, ExportData exportData`|Called after data is exported.|

Custom worksheet rules can be created by extending the `Worksheet` class and overriding the methods above.

A delegate-based worksheet rule called `AnonymousWorksheetRule` can be used for simple rules without the need for a dedicated class. 

#### ColumnRule
|Method|Parameters|Description|
|---|---|---|
|`OnDataTransforming`|`ExportData exportData, ColumnData columnData`|Called during the data-transformation part of the export process.|
|`OnColumnExporting`|`IXLColumn column`|Called before the column is exported.|
|`OnColumnExported`|`IXLColumn column`|Called after the column is exported.|
|`OnCellExporting`|`IXLCell cell, ColumnValue columnValue`|Called before a cell in the column is exported.|
|`OnCellExported`|`IXLCell cell, ColumnValue columnValue`|Called after a cell in the column is exported.|

