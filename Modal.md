A Modal message box:

[[https://github.com/rivo/tview/blob/master/demos/modal/screenshot.png]]

Code:

```go
package main

import (
	"github.com/rivo/tview"
)

func main() {
	app := tview.NewApplication()
	modal := tview.NewModal().
		SetText("Do you want to quit the application?").
		AddButtons([]string{"Quit", "Cancel"}).
		SetDoneFunc(func(buttonIndex int, buttonLabel string) {
			if buttonLabel == "Quit" {
				app.Stop()
			}
		})
	if err := app.SetRoot(modal, false).SetFocus(modal).Run(); err != nil {
		panic(err)
	}
}
```