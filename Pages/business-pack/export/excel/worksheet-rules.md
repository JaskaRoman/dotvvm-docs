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
|`AnonymousWorksheetRule`*|`Action<IXLWorksheet> onWorksheetExporting,`<br/> `Action<IXLWorksheet, ExportData> onDataExporting,`<br/> `Action<IXLWorksheet, ExportData> onDataExported)`|A delegate-based `WorksheetRule`. See [Sample 1](#sample-1).|

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

#### Sample 1
The `AnonymousWorksheetRule` can be used to quickly access the underlying ClosedXML objects. Multiple instances of this rule can be applied on the same column.
Alternatively, a custom column rule can be implemented by extending the `WorksheetRule` base class.

```CSHARP
ExportRules =
{
    new AnonymousWorksheetRule(
        onWorksheetExporting: (worksheet) => 
        {
            worksheet.Workbook.Properties.Author = "John Doe";
            worksheet.Workbook.Properties.Company = "Riganti";
        },
        onDataExported: (worksheet, exportData) => worksheet.Workbook.Properties.Created = DateTime.Now);
}
```

```CSHARP
public class CustomMetadataWorksheetRule : WorksheetRule
{
    public string CompanyName { get; }
    public string AuthorName { get; }

    public AuthorMetadataWorksheetRule(string companyName, string authorName)
    {
        CompanyName = companyName;
        AuthorName = authorName;
    }

    public override void OnWorksheetExporting(IXLWorksheet worksheet)
    {
        worksheet.Workbook.Properties.Company = CompanyName;
        worksheet.Workbook.Properties.Author = AuthorName;
    }

    public override void OnDataExported(IXLWorksheet worksheet, ExportData exportData)
    {
        worksheet.Workbook.Properties.Created = DateTime.Now;
    }
}
```
