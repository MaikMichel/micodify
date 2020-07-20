# Interactive Grid – Update current row only

In my current project I use an interactive grid on one page. When you click on a row, i. e. select it, a report is displayed below the grid that displays this data set. The editing of the data set is realized in a modal dialog. If it is closed, the report including the currently selected line in the grid should be updated.
<!--more-->
Unfortunately, the selection is not retained after a refresh of the grid. If you use JavaScript to update the grid, first memorize the selected data set and then want to mark it again, you get the problem that updating the grid is asynchronous. And the data record to be selected is not even there yet.

However, since I only want to edit one record, it is sufficient to update this row only. The IG offers a corresponding method via API.

```javascript
var myGrid = apex.region( "TEST_CASES_GRID" )
 .widget()
 .interactiveGrid("getViews")
 .grid;
myGrid.model.fetchRecords( myGrid.getSelectedRecords());
```

This only works if the IG has a RowID.

!["Refresh current row"](/images/refresh-current-row-in-ig/apex_ig_refresh_single_record.gif "Refresh current row")

