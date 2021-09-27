The [`Table`](https://pkg.go.dev/github.com/rivo/tview#Table) widget in its basic form allows you to place text in "cells" located by their rows and columns. The [[Table]] wiki page contains a simple example for this. In some cases, however, you may wish to use the `Table` widget as a view on your data without copying all of your data to the `Table` class. For example, if your data set is very large and you can only keep a part of it in memory, using the `Table` widget in its traditional form is not feasible. In those cases, you can implement the [`TableContent`](https://pkg.go.dev/github.com/rivo/tview#TableContent) interface and plug it into the `Table` with [`SetContent()`](https://pkg.go.dev/github.com/rivo/tview#Table.SetContent) to create a "virtual table":

```go
type TableContent interface {
	// Return the cell at the given position or nil if there is no cell. The
	// row and column arguments start at 0 and end at what GetRowCount() and
	// GetColumnCount() return, minus 1.
	GetCell(row, column int) *TableCell

	// Return the total number of rows in the table.
	GetRowCount() int

	// Return the total number of columns in the table.
	GetColumnCount() int

	// Set the cell at the given position to the provided cell.
	SetCell(row, column int, cell *TableCell)

	// Remove the row at the given position by shifting all following rows up
	// by one. Out of range positions may be ignored.
	RemoveRow(row int)

	// Remove the column at the given position by shifting all following columns
	// left by one. Out of range positions may be ignored.
	RemoveColumn(column int)

	// Insert a new empty row at the given position by shifting all rows at that
	// position and below down by one. Implementers may decide what to do with
	// out of range positions.
	InsertRow(row int)

	// Insert a new empty column at the given position by shifting all columns
	// at that position and to the right by one to the right. Implementers may
	// decide what to do with out of range positions.
	InsertColumn(column int)

	// Remove all table data.
	Clear()
}
```

To display the data, the `Table` class really only needs the `GetCell()`, the `GetRowCount()`, and `GetColumnCount()` functions. But since the non-virtual, basic `Table` class also provides functions to modify the data (e.g. to delete a row or to remove all data), this interface also passes these functions on to your implementation.

But if you don't need this, that is, if your data does not get modified and is read-only, these write functions can remain empty. And to make it easier to implement a read-only data structure, there is also the [`TableContentReadOnly`](https://pkg.go.dev/github.com/rivo/tview#TableContentReadOnly) struct, which already implements the write functions (which do nothing). So your implementation of `TableContent` may look like this:

```go
type MyData struct {
	tview.TableContentReadOnly
}

func (d *MyData) GetCell(row, column int) *tview.TableCell {
	// Return content at (row,column).
}

func (d *MyData) GetRowCount() int {
	// Return total number of rows.
}

func (d *MyData) GetColumnCount() int {
	// Return total number of columns.
}
```

The `Table` widget will do everything you would expect when interacting with it, such as scrolling the data and selecting cells, rows, or columns. And whenever it is drawn, your functions are called on the visible part of the table (unless [`SetEvaluateAllRows()`](https://pkg.go.dev/github.com/rivo/tview#Table.SetEvaluateAllRows) is set to `true`, which is not advisable for virtual tables).

Since it is difficult to know in advance which cells exactly will be visible, your code should be able to provide the content of any cell at any time. This is also important because the user can navigate to any part of the `Table` with the keyboard or the mouse: for example, pressing <kbd>g</kbd> will send the user to the beginning of the table. If this poses a problem to your implementation, you may want to intercept the keyboard/mouse events for more control over them. See [[CustomKeys]] for details.

Alternatively, you may opt to implement your `TableContent` interface as a small window into your larger data structure. For example, if you have a large, growing log file where you always want to display only the last 200 rows, the table can be 200 rows high and everything before is discarded. Here's an example that uses a rolling window:

```go
type MyLog struct {
	tview.TableContentReadOnly

	// Nevermind the hard-coded values, this is just an example.
	data       [200][5]string
	startIndex int
}

func (d *MyLog) GetCell(row, column int) *tview.TableCell {
	return tview.NewTableCell(d.data[(row+d.startIndex)%200][column])
}

func (d *MyLog) GetRowCount() int {
	return 200
}

func (d *MyLog) GetColumnCount() int {
	return 5
}

func (d *MyLog) AppendRow(row [5]string) {
	d.data[d.startIndex] = row
	d.startIndex = (d.startIndex + 1) % 200
}
```

Also check out the virtual table [demo application](https://github.com/rivo/tview/blob/master/demos/table/virtualtable/main.go) which implements a virtual table over the entire (positive) `int64` range for rows and columns.