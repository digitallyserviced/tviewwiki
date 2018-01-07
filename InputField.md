A one line InputField:

[[https://github.com/rivo/tview/blob/master/demos/inputfield/screenshot.png]]

Code:

```go
package main

import (
	"github.com/gdamore/tcell"
	"github.com/rivo/tview"
)

func main() {
	app := tview.NewApplication()
	inputField := tview.NewInputField().
		SetLabel("Enter a number: ").
		SetFieldLength(10).
		SetAcceptanceFunc(tview.InputFieldInteger).
		SetDoneFunc(func(key tcell.Key) {
			app.Stop()
		})
	if err := app.SetRoot(inputField, true).SetFocus(inputField).Run(); err != nil {
		panic(err)
	}
}
```