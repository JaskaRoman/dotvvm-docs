# Exporting data
[Business Pack](/landing/business-pack) provides support for customizable [`GridView`](TODO) data export.

Export-related classes can be found in the `DotVVM.BusinessPack.Export` namespace provided by the optional NuGet packages.

Following export formats are available as separate packages:

||Extension|MIME type|Exporter class|Export settings class|Namespace|
|---|---|---|---|---|
|**Excel**|xlsx|`application/vnd.openxmlformats-officedocument.spreadsheetml.sheet`|`GridViewExportExcel<T>`|`GridViewExportExcelSettings<T>`|`DotVVM.BusinessPack.Export.Excel`|
|**Csv**|csv|`text/csv`|`GridViewExportCsv<T>`|`GridViewExportCsvSettings<T>`|`DotVVM.BusinessPack.Export.Csv`|
|**Pdf**|pdf|`application/pdf`|`GridViewExportPdf<T>`|`GridViewExportPdfSettings<T>`|`DotVVM.BusinessPack.Export.Pdf`|

Support for additional export formats can be added by implementing the `IGridViewExport<T>` interface.
See [Custom export formats](TODO) for more information.

## Export process
This section describes the general export pipeline, providing a code sample.

### Prerequisites
- The export process was designed to be invoked from within an actual [DotVVM ViewModel](TODO), after the `Load` phase of its life-cycle.
- A [`GridView`](TODO) instance to export: 
  - Target instance can be retrieved during a request using one of the `FindControlBy...<T>` methods located on the `View` property of the `IDotvvmRequestContext`.
  - If the `GridView.UserSettings` property is set, the currently bound instance of `GridViewUserSettings` will be taken into account during the export.
- A populated [`IBusinessPackDataSet<T>`]().
  - The exported data-set can differ from the data-set bound in the `DataSource` property of the `GridView`.
  - Only data currently present in the data-set will be exported regardless of `IPagingOptions`. Data needs to be loaded manually if you intend to export a different range of rows than the one currently loaded.
  

### Basic export sample

The basic export process itself consists of three steps:

1. Exporter initialization - an instance of the export class must be created by the user using the `new` keyword.
   - The built-in `IGridViewExport<T>` implementations allow specification of an optional parameter containing export-specific settings.
2. The actual export - invocation of `IGridViewExport<T>.Export(gridView, dataSet)`.
   - An instance of `GridView` and `IBusinessPackDataSet<T>` must be provided.
3. Consumption of the resulting `Stream` representing the exported file.

#### Sample
```DOTHTML
    <bp:GridView DataSource="{value: CustomerDataSet}" ID="CustomerGrid">
        <Columns>
            <bp:GridViewNumericColumn Value="{value: CustomerId}" />
            <bp:GridViewTextColumn Value="{value: Name}" />
        </Columns>
    </bp:GridView>

    <bp:Button Click="{command: OnExportButtonClicked()}" Text="Export to Excel" />
```
```CSHARP
    public BusinessPackDataSet<CustomerData> CustomerDataSet { get; set; } = new BusinessPackDataSet<CustomerData>();

    public void OnExportButtonClicked()
    {
        // locate the GridView using an HTML ID
        var gridView = Context.View.FindControlByClientId<BusinessPack.Controls.GridView>("CustomerGrid");
        // create an instance of an Excel exporter for type CustomerData
        var export = new GridViewExportExcel<CustomerData>();
        // execute the export using the provided gridView and dataSet
        var stream = export.Export(gridView, CustomerDataSet);
        // return the exported file
        Context.ReturnFile(stream, "Customers.xlsx", MimeTypeConstants.OpenXmlExcel);
    }
``` 
