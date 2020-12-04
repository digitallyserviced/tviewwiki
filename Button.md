A simple Button with a frame:

[[https://github.com/rivo/tview/blob/master/demos/button/screenshot.png]]

Code:

```go
package main

import "github.com/rivo/tview"

func main() {
	app := tview.NewApplication()
	button := tview.NewButton("Hit Enter to close").SetSelectedFunc(func() {
		app.Stop()
	})
	button.SetBorder(true).SetRect(0, 0, 22, 3)
	if err := app.SetRoot(button, false).SetFocus(button).Run(); err != nil {
		panic(err)
	}
}
```

See also: https://pkg.go.dev/github.com/rivo/tview#Button