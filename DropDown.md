A simple DropDown selection:

[[https://github.com/rivo/tview/blob/master/demos/dropdown/screenshot.png]]

Code:

```go
package main

import "github.com/rivo/tview"

func main() {
	app := tview.NewApplication()
	dropdown := tview.NewDropDown().
		SetLabel("Select an option (hit Enter): ").
		SetOptions([]string{"First", "Second", "Third", "Fourth", "Fifth"}, nil)
	if err := app.SetRoot(dropdown, true).SetFocus(dropdown).Run(); err != nil {
		panic(err)
	}
}
```

See also: https://pkg.go.dev/github.com/rivo/tview#DropDown