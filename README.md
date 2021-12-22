# Power BI Calendar

Basic copy paste into a Power BI Power Query Editor for ease of use. You need to use three tables (so far). One calendar table, two sort (by year by month) which will help your charts month year sort.

Go into Power BI Desktop and open Transform data. Create new tables with the three queries below. Sort columns by numeric values and connect relationship date values to calendar. Calendar date column should have a relationship with raw data. 

Date are from DateTime.LocalNow() to Date.AddYears(Date.From(DateTime.LocalNow()), -2). (3 years)

- [x] Added fiscal year map for Jun-July
- [x] Added fiscal year month sort
- [x] Added fiscal year quarter
- [x] Added fiscal year quarter sort
- [x] Added fiscal year to years (i.e. 2020-2021)
- [x] Added month sort

## Power Query M
1. calendar source

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
    #"Added Custom4" = Table.AddColumn(#"Added Custom3", "Month Name Short", each Date.ToText([Date],"MMM")),
    #"Added Custom5" = Table.AddColumn(#"Added Custom4", "Month Name Long", each Date.ToText([Date],"MMMM")),
    #"Reordered Columns" = Table.ReorderColumns(#"Added Custom5",{"Date", "Index", "Year", "Month Number", "Month Name Short", "Month Name Long", "Day", "Day Name"}),
    #"Added Custom6" = Table.AddColumn(#"Reordered Columns", "Quarter Number", each Date.QuarterOfYear([Date])),
    #"Duplicated Column" = Table.DuplicateColumn(#"Added Custom6", "Year", "Copy of Year"),
    #"Renamed Columns1" = Table.RenameColumns(#"Duplicated Column",{{"Copy of Year", "Short Year"}}),
    #"Changed Type" = Table.TransformColumnTypes(#"Renamed Columns1",{{"Short Year", type text}}),
    #"Split Column by Position" = Table.SplitColumn(#"Changed Type","Short Year",Splitter.SplitTextByRepeatedLengths(2),{"Short Year.1", "Short Year.2"}),
    #"Changed Type1" = Table.TransformColumnTypes(#"Split Column by Position",{{"Short Year.1", Int64.Type}, {"Short Year.2", Int64.Type}}),
    #"Removed Columns" = Table.RemoveColumns(#"Changed Type1",{"Short Year.1"}),
    #"Renamed Columns2" = Table.RenameColumns(#"Removed Columns",{{"Short Year.2", "Short Year"}}),
    #"Added Custom7" = Table.AddColumn(#"Renamed Columns2", "Quarter Year", each Number.ToText([Short Year]) & "Q" & Number.ToText([Quarter Number],"00")),
    #"Reordered Columns1" = Table.ReorderColumns(#"Added Custom7",{"Index", "Date", "Day", "Day Name", "Month Number", "Month Name Short", "Month Name Long", "Quarter Number", "Quarter Year", "Short Year", "Year"}),
    #"Changed Type2" = Table.TransformColumnTypes(#"Reordered Columns1",{{"Date", type date}, {"Day", Int64.Type}, {"Index", Int64.Type}, {"Month Number", Int64.Type}, {"Quarter Number", Int64.Type}, {"Month Name Short", type text}, {"Month Name Long", type text}, {"Quarter Year", type text}, {"Year", Int64.Type}}),
    #"Added Custom8" = Table.AddColumn(#"Changed Type2", "Fiscal Year", each if Date.Month([Date])>6 then Date.Year([Date])+1 else Date.Year([Date])),
    #"Added Custom9" = Table.AddColumn(#"Added Custom8", "Fiscal Quarter", each Quarters{Date.Month([Date])-1}),
    #"Added Custom10" = Table.AddColumn(#"Added Custom9", "Fiscal Period", each Period{Date.Month([Date])-1}),
    #"Added Custom11" = Table.AddColumn(#"Added Custom10", "Fiscal Year Quarter ", each Number.ToText([Fiscal Year]) & " Q" & Number.ToText([Fiscal Quarter])),
    #"Added Custom12" = Table.AddColumn(#"Added Custom11", "Fiscal Year To Years", each Number.ToText([Fiscal Year]-1) & "-" & Number.ToText([Fiscal Year]))
in
    #"Added Custom12"
```
2. yearSort
```
let
  Quarters = {3,3,3,4,4,4,1,1,1,2,2,2},
  Period = {7,8,9,10,11,12,1,2,3,4,5,6},
  Source = List.Generate( () => Number.From(Date.Year(DateTime.LocalNow())), each _ >= Number.From(Date.Year(Date.AddYears(Date.From(DateTime.LocalNow()), -2))), each _ - 1 ),
  #"Table from List" = Table.FromList(Source, Splitter.SplitByNothing(), null, null, ExtraValues.Error),
  #"Added Index" = Table.AddIndexColumn(#"Table from List", "Index", 1, 1),
  #"Renamed Columns" = Table.RenameColumns(#"Added Index",{{"Column1", "Year"}}),
    #"Added Custom" = Table.AddColumn(#"Renamed Columns", "Fiscal Year To Years", each Number.ToText([Year]) & "-" & Number.ToText([Year]+1))
 in
   #"Added Custom"
```
3. monthSort
```
let
    Source = Table.FromRows(Json.Document(Binary.Decompress(Binary.FromText("i45WMlTSUfIqzalUitWJVjICchxL00uLS8BcYyA3OLWgJDU3KbUILGICFPFPLsmH8U2BfL/8MoQCM6CAS2oyQsAcZH5iXmliEcQKCyDfLTWpCC5gCRTwTSxKzgDzDA1ATigoysyBcA3BshCVhkZgt+alKsXGAgA=", BinaryEncoding.Base64), Compression.Deflate)), let _t = ((type nullable text) meta [Serialized.Text = true]) in type table [Index = _t, Month = _t]),
    #"Changed Type" = Table.TransformColumnTypes(Source,{{"Index", Int64.Type}, {"Month", type text}})
in
    #"Changed Type"
```
4. fiscalYearQuarterSort
```
let
    Quarters = {3,3,3,4,4,4,1,1,1,2,2,2},
    Source = List.Dates,
    #"Invoked FunctionSource" = Source(Date.AddYears(Date.From(DateTime.LocalNow()), -2), Duration.Days(DateTime.Date(DateTime.FixedLocalNow()) - Date.AddYears(Date.From(DateTime.LocalNow()), -2)), #duration(1, 0, 0, 0)),
    #"Table from List1" = Table.FromList(#"Invoked FunctionSource", Splitter.SplitByNothing(), null, null, ExtraValues.Error),
    #"Renamed Columns1" = Table.RenameColumns(#"Table from List1",{{"Column1", "Date"}}),
    #"Added Conditional Column" = Table.AddColumn(#"Renamed Columns1", "End of Quarter", each Date.EndOfQuarter([Date])),
    #"End of Quarter" = #"Added Conditional Column"[End of Quarter],
    #"Remove Nulls" = List.RemoveNulls(#"End of Quarter"),
    #"Table from List2" = Table.FromList(#"Remove Nulls", Splitter.SplitByNothing(), null, null, ExtraValues.Error),
    #"Renamed Columns2" = Table.RenameColumns(#"Table from List2",{{"Column1", "End of Quarter"}}),
    #"Added Custom1" = Table.AddColumn(#"Renamed Columns2", "Fiscal Year", each if Date.Month([End of Quarter])>6 then Date.Year([End of Quarter])+1 else Date.Year([End of Quarter])),
    #"Added Custom2" = Table.AddColumn(#"Added Custom1", "Fiscal Quarter", each Quarters{Date.Month([End of Quarter])-1}),
    #"Added Custom3" = Table.AddColumn(#"Added Custom2", "Fiscal Year Quarter ", each Number.ToText([Fiscal Year]) & " Q" & Number.ToText([Fiscal Quarter])),
    #"Added Custom4" = Table.AddColumn(#"Added Custom3", "Fiscal Year Quarter Sort", each Number.FromText(Number.ToText([Fiscal Year]) & "0" & Number.ToText([Fiscal Quarter]))),
    #"Removed Duplicates" = Table.Distinct(#"Added Custom4", {"End of Quarter"}),
    #"Added Index" = Table.AddIndexColumn(#"Removed Duplicates", "Index", 1, 1)
in
    #"Added Index"
```
