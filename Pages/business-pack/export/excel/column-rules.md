# Column rules
Column rules are classes based on the `ColumnRule` abstract class.
They represent rules which are applied in the context of a single column or its cells.

Following column rules are available.

|Class name|Parameters|Description|
|---|---|---|
|`AdjustToContentsColumnRule`|None|Adjusts the size of used cells in the entire column in order to accommodate their contents. A worksheet-wide version of this rule is available. See [`AdjustToContentsWorksheetRule`](TODO).|
|`IgnoreColumnRule`|None|Ignores target column during the entire export process.|
|`DataTypeColumnRule`|`XLDataType dataType`|Overrides the column's [data-type](https://github.com/ClosedXML/ClosedXML/wiki/Data-Types).|
|`HeaderTextColumnRule`|`string headerText`|Applies a custom header text.|
|`NumberFormatColumnRule`|`string numberFormat`/`int numberFormatId`|Applies specified number format using an Excel number/date format string or a culture-dependent OpenXML [number format id](https://github.com/ClosedXML/ClosedXML/wiki/NumberFormatId-Lookup-Table).|
|`ValueTransformColumnRule<TSrc, TDst>`*|`Func<TSrc, TDst> transform`|Transforms a value from source type to target type using a delegate. See [Sample 2](#sample-2).|
|`AnonymousColumnRule`*|`Action<ExportData, ColumnData> onDataTransforming,`<br/> `Action<IXLColumn> onColumnExporting,`<br/> `Action<IXLColumn> onColumnExported,`<br/>`Action<IXLCell, ColumnValue> onCellExporting,`<br/>`Action<IXLCell, ColumnValue> onCellExported`|A delegate-based `ColumnRule`. See [Sample 3](#sample-3).|

*\* Marked rules implement `IStackableExcelExportRule` meaning that multiple instances can be specified simultaneously.*

**All column rules must specify column name using the `ColumnName` property.**

When a column is being exported, rules matching its `ColumnName` are looked up and applied. This means that rules with incorrect `ColumnName` values will be quietly ignored.

### Sample 1
The column name of a `GridViewColumn` can be set explicitly using the `ColumnName` DotVVM property. When not set, a column name is generated. For columns based on the `GridViewValueColumn<TValue, TProperty>` base class, bound directly to a property (“*Name*” in case of this sample), the name of the property is used.\
If you intend to adress a column with non-trivial binding from your code (“*PriceWithCurrencyCode*” in case of this sample), it is recommended to set the column name explicitly.

```CSHARP
public class ProductData
{
    public string Name { get; set; }
    public decimal Price { get; set; }
    public bool IsAvailable { get; set; }
}
```
```CSHARP
public string CurrencyCode { get; set; } = "EUR";
public BusinessPackDataSet<ProductData> ProductDataSet { get; set; } = new BusinessPackDataSet<ProductData>();

public void OnExportButtonClicked()
{
    var exportSettings = new GridViewExportExcelSettings<ProductData>
    {
        ExportRules =
        {
            new HeaderTextColumnRule(nameof(ProductData.Name), "Product name"),
            new HeaderTextColumnRule("PriceWithCurrencyCode", "Price and currency")
        }
    };
    var export = new GridViewExportExcel<ProductData>(exportSettings);
    var gridView = Context.View.FindControlByClientId<BusinessPack.Controls.GridView>("ProductGrid");
    var stream = export.Export(gridView, ProductDataSet);
    Context.ReturnFile(stream, "Products.xlsx", MimeTypeConstants.OpenXmlExcel);
}
```
```DOTHTML
<bp:GridView DataSource="{value: ProductDataSet}" ID="ProductGrid">
    <Columns>
        <bp:GridViewTextColumn Value="{value: Name}" />
        <bp:GridViewTextColumn Value="{value: Price + " " + _parent.CurrencyCode}" ColumnName="PriceWithCurrencyCode" />
    </Columns>
</bp:GridView>

<bp:Button Click="{command: OnExportButtonClicked()}" Text="Export to Excel" />
```

### Sample 2
`ValueTransformColumnRule<TSrc, TDst>` can be used to transform the exported value before the actual export. Source type and target type must be specified. Multiple instances of this rule can be applied on the same column.\
In this example, a boolean value is transformed into a user-friendly string.
```CSHARP
ExportRules =
{
    new ValueTransformColumnRule<bool, string>(nameof(ProductData.IsAvailable), 
        b => b ? "Available" : "Unavailable"),
}
```

## Extensibility points
The abstract `ColumnRule` class contains multiple overridable virtual methods invoked during the export process.

|Method|Parameters|Description|
|---|---|---|
|`OnDataTransforming`|`ExportData exportData, ColumnData columnData`|Called during the data-transformation part of the export process.|
|`OnColumnExporting`|`IXLColumn column`|Called before the column is exported.|
|`OnColumnExported`|`IXLColumn column`|Called after the column is exported.|
|`OnCellExporting`|`IXLCell cell, ColumnValue columnValue`|Called before a cell in the column is exported.|
|`OnCellExported`|`IXLCell cell, ColumnValue columnValue`|Called after a cell in the column is exported.|

Custom worksheet rules can be created by extending the `ColumnRule` class and overriding the methods above.

A delegate-based worksheet rule called `AnonymousColumnRule` can be used for simple rules without the need for a dedicated class. 



### Sample 3
The `AnonymousColumnRule` can be used to quickly access the underlying ClosedXML objects. Multiple instances of this rule can be applied on the same column.
Alternatively, a custom column rule can be implemented by extending the `ColumnRule` base class.

```CSHARP
ExportRules =
{
    new AnonymousColumnRule(nameof(ProductData.Name), 
        onCellExported: (cell, columnValue) => cell.Style.Fill.BackgroundColor = XLColor.LightBlue);
    new AnonymousColumnRule(nameof(ProductData.Name), 
        onCellExported: (cell, columnValue) => cell.Style.Font.FontColor = XLColor.DarkBlue);
}
```

```CSHARP
public class BlueColorColumnRule : ColumnRule
{
    public BlueColorColumnRule(string columnName) : base(columnName) { }

    public override void OnCellExported(IXLCell cell, ColumnValue columnValue)
    {
        cell.Style.Fill.BackgroundColor = XLColor.LightBlue;
        cell.Style.Font.FontColor = XLColor.DarkBlue;
    }
}
```

## Fluent API
A Fluent API is available for construction of column rules.

### Sample 4
Column rule creation chain is implemented as an extension method `ForColumn` extending the `GridViewExportExcelSettings<T>` class.\
This method has two parameters - first determines the target column name while the second defines a `Func` which transforms the current `ICollection<ColumnRule>`. These transformations can either directly modify the passed `ICollection<ColumnRule>` instance using regular collection manipulation operations, or use the additional Fluent methods described in [Sample 5](#sample-5).

Two overloads of the `ForColumn` method are available, differing only in the way the column name is specified.\
The first option is to specify the column name directly using a string. This is ideal for columns with custom names, as described in [Sample 1](#sample-1).\
The second option is to pass a member selector expression, which will be used to retrieve the member name. This will only work with column bound directly to a property.
```CSHARP
GridViewExportExcelSettings<ProductData>.Empty
    .ForColumn("Name", r => r.WithAdjustToContent())
```
```CSHARP
GridViewExportExcelSettings<ProductData>.Empty
    .ForColumn(c => c.Name, r => r.WithAdjustToContent())
```

Multi-propery paths in the member selector expression are supported, while indexed access is not.
```CSHARP
GridViewExportExcelSettings<ProductData>.Empty
    // Supported
    .ForColumn(c => c.ProductType.Name, r => r.WithAdjustToContent())
    // Not supported
    
```

### Sample 5
Once the target column is specified using the first parameter of the `ForColumn` method described in [Sample 4](#sample-4), the current collection of rules for given column can be modified.
All built-in column rules are accessible using the prefix `With` in the context of an `ICollection<ColumnRule>`.
These methods return a transformed collection and can be chained.

*Please note, that for technical reasons, the rule column names set from within the rule transformation `Func` of the `ForColumn` method will be overwritten by the column name set in the first parameter of the `ForColumn` method. This explains why `null` is used as the column name in the second code block of this sample.*
```CSHARP
GridViewExportExcelSettings<ProductData>.Empty
    .ForColumn("Name", r => r
        .WithAdjustToContent()
        .WithHeaderText("Product name"));
```
The code above is functionally equal to:
```CSHARP
GridViewExportExcelSettings<ProductData>.Empty
    .ForColumn("Name", r => 
        {
            r.Add(new AdjustToContentsColumnRule(null));
            r.Add(new HeaderTextColumnRule(null, "Product name"));
            return r;
        });
```
Which in turn is equal to:
```CSHARP
new GridViewExportExcelSettings<ProductData>(Enumerable.Empty<IExcelExportRule>())
{
    ExportRules = 
    {
        new AdjustToContentsColumnRule("Name"),
        new HeaderTextColumnRule("Name", "Product name")
    }
}
```
