# Worksheet rules
Worksheet rules are classes based on the `WorksheetRule` abstract class.
They represent rules which are applied in the context of the entire exported worksheet.

Following worksheet rules are available.

|Class name|Parameters|Description|
|---|---|---|
|`AdjustToContentsWorksheetRule`|None|Adjusts the size of used cells in the entire worksheet in order to accommodate their contents.|
|`StartAddressWorksheetRule`|`int row, int column`|Offsets the starting cell address of the export range.|
|`TableHeaderWorksheetRule`|`bool createTable`|Creates a header row with the option of creating an explicit Excel table range.|
|`WorksheetNameWorksheetRule`|`string worksheetName`|Applies a custom worksheet name.|
|`AnonymousWorksheetRule`*|`Action<IXLWorksheet> onWorksheetExporting,`<br/> `Action<IXLWorksheet, ExportData> onDataExporting,`<br/> `Action<IXLWorksheet, ExportData> onDataExported)`|A delegate-based `WorksheetRule`.|

*\* Marked rules implement `IStackableExcelExportRule` meaning that multiple instances can be specified simultaneously.*

### Extensibility points
The abstract `WorksheetRule` class contains multiple overridable virtual methods invoked during the export process.

|Method|Parameters|Description|
|---|---|---|
|`OnWorksheetExporting`|`IXLWorksheet worksheet`|Called before the worksheet is exported.|
|`OnDataExporting`|`IXLWorksheet worksheet, ExportData exportData`|Called before data is exported.|
|`OnDataExported`|`IXLWorksheet worksheet, ExportData exportData`|Called after data is exported.|

Custom worksheet rules can be created by extending the `WorksheetRule` class and overriding the methods above.

A delegate-based worksheet rule called `AnonymousWorksheetRule` can be used for simple rules without the need for a dedicated class. 
