# CSV export
`GridViewExportCsv<T>` from the `DotVVM.BusinessPack.Export.Csv` package allows you to export `GridView` data into a [Comma-separated values](TODO) file.

While the basic process remains as described in [Exporting data](TODO), additional features are supported by the Csv exporter.
These features are controlled by the `GridViewExportCsvSettings<T>` class.

## CSV export settings
The csv-specific class `GridViewExportCsvSettings<T>` consists of following instance properties:

|Type|Property|Description|
|---|---|---|
|`ColumnValueProviderHandlers<T>`|`ColumnValueProviderHandlers`|Value providers used to retrieve the data from exported data-context.|
|`bool`|`CreateHeader`|Determines whether a header row is created. Defaults to `true`.|
|`string`|`Separator`|Determines what string is used to separate column values. Defaults to `","`.|
|`QuotedValue`|`QuotedValue`|Determines whether the column values should be wrapped in quotation marks. Defaults to `QuotedValue.IfNeeded`, which only adds quotation marks when the `Separator` character is present within the column value. See [Sample 1](#sample-1).|

**Export settings are not serializable and should not be sent to the client.**

#### Sample 1

A simple instance of export settings, which will achieve the following results:
- The exported file will contain a header row, because `CreateHeader` is set to `true`.
- The exported column values will be separated by a `","` character, defined by the `Separator` property.
- The value “*FooBar, Ltd.*” will be sorrounded by quotation marks because it contains a `","` while the `Separator` property is also set to `","` and the `QuotedValue` property is set to `IfNeeded`.

```CSHARP
public class CustomerData
{
    public int PhoneNumber { get; set; } = 123456789
    public string FullName { get; set; } = "FooBar, Ltd."
}
```
```CSHARP
var exportSettings = new GridViewExportCsvSettings<CustomerData>
{
    CreateHeader = true,
    QuotedValue = QuotedValue.IfNeeded,
    Separator = ","
};

var export = new GridViewExportCsv<CustomerData>(exportSettings);
```

The resulting CSV file:
```
PhoneNumber,FullName
123456789,"FooBar, Ltd."
```


