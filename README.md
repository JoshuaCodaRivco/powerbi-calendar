# Power BI Calendar

Basic copy paste into a Power BI Power Query Editor for ease of use. 

Date are from DateTime.LocalNow() to Date.AddYears(Date.From(DateTime.LocalNow()), -2). (3 years)

- [x] Added fiscal year map for Jun-July

```
let
    Quarters = {3,3,3,4,4,4,1,1,1,2,2,2},
    Period = {7,8,9,10,11,12,1,2,3,4,5,6},
    Source = List.Dates,
    #"Invoked FunctionSource" = Source(Date.AddYears(Date.From(DateTime.LocalNow()), -2), Duration.Days(DateTime.Date(DateTime.FixedLocalNow()) - Date.AddYears(Date.From(DateTime.LocalNow()), -2)), #duration(1, 0, 0, 0)),
    #"Table from List" = Table.FromList(#"Invoked FunctionSource", Splitter.SplitByNothing(), null, null, ExtraValues.Error),
    #"Added Index" = Table.AddIndexColumn(#"Table from List", "Index", 1, 1),
    #"Renamed Columns" = Table.RenameColumns(#"Added Index",{{"Column1", "Date"}}),
    #"Added Custom" = Table.AddColumn(#"Renamed Columns", "Year", each Date.Year([Date])),
    #"Added Custom1" = Table.AddColumn(#"Added Custom", "Month Number", each Date.Month([Date])),
    #"Added Custom2" = Table.AddColumn(#"Added Custom1", "Day", each Date.Day([Date])),
    #"Added Custom3" = Table.AddColumn(#"Added Custom2", "Day Name", each Date.ToText([Date],"ddd")),
    #"Added Custom4" = Table.AddColumn(#"Added Custom3", "Month Name", each Date.ToText([Date],"MMM")),
    #"Reordered Columns" = Table.ReorderColumns(#"Added Custom4",{"Date", "Index", "Year", "Month Number", "Month Name", "Day", "Day Name"}),
    #"Added Custom5" = Table.AddColumn(#"Reordered Columns", "Quarter Number", each Date.QuarterOfYear([Date])),
    #"Duplicated Column" = Table.DuplicateColumn(#"Added Custom5", "Year", "Copy of Year"),
    #"Renamed Columns1" = Table.RenameColumns(#"Duplicated Column",{{"Copy of Year", "Short Year"}}),
    #"Changed Type" = Table.TransformColumnTypes(#"Renamed Columns1",{{"Short Year", type text}}),
    #"Split Column by Position" = Table.SplitColumn(#"Changed Type","Short Year",Splitter.SplitTextByRepeatedLengths(2),{"Short Year.1", "Short Year.2"}),
    #"Changed Type1" = Table.TransformColumnTypes(#"Split Column by Position",{{"Short Year.1", Int64.Type}, {"Short Year.2", Int64.Type}}),
    #"Removed Columns" = Table.RemoveColumns(#"Changed Type1",{"Short Year.1"}),
    #"Renamed Columns2" = Table.RenameColumns(#"Removed Columns",{{"Short Year.2", "Short Year"}}),
    #"Added Custom6" = Table.AddColumn(#"Renamed Columns2", "Quarter Year", each Number.ToText([Short Year]) & "Q" & Number.ToText([Quarter Number],"00")),
    #"Reordered Columns1" = Table.ReorderColumns(#"Added Custom6",{"Index", "Date", "Day", "Day Name", "Month Number", "Month Name", "Quarter Number", "Quarter Year", "Short Year", "Year"}),
    #"Changed Type2" = Table.TransformColumnTypes(#"Reordered Columns1",{{"Date", type date}, {"Day", Int64.Type}, {"Index", Int64.Type}, {"Month Number", Int64.Type}, {"Quarter Number", Int64.Type}, {"Month Name", type text}, {"Quarter Year", type text}, {"Year", Int64.Type}}),
    #"Added Custom7" = Table.AddColumn(#"Changed Type2", "Fiscal Year", each if Date.Month([Date])>6 then Date.Year([Date])+1 else Date.Year([Date])),
    #"Added Custom8" = Table.AddColumn(#"Added Custom7", "Fiscal Quarter", each Quarters{Date.Month([Date])-1}),
    #"Added Custom9" = Table.AddColumn(#"Added Custom8", "Fiscal Period", each Period{Date.Month([Date])-1})
in
    #"Added Custom9"
    
```
