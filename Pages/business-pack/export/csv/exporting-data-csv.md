# CSV export
`GridViewExportCsv<T>` from the `DotVVM.BusinessPack.Export.Csv` package allows you to export `GridView` data into an [Comma-separated values](TODO) file.

While the basic process remains as described in [Exporting data](TODO), additional features are supported by the Csv exporter.
These features are controlled by the `GridViewExportCsvSettings<T>` class.

## CSV export settings
The csv-specific class `GridViewExportCsvSettings<T>` consists of following instance properties:

|Type|Property|Description|
|---|---|---|
|`ColumnValueProviderHandlers<T>`|`ColumnValueProviderHandlers`|Value providers used to retrieve the data from exported data-context.|
|`bool`|`CreateHeader`|Determines whether a header row is created. Defaults to `true`.|
|`string`|`Separator`|Determines what string is used to separate column values. Defaults to `","`.|

**Export settings are not serializable and should not be sent to the client.**

#### Sample 1
TODO