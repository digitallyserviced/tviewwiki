Drawing images:

![Screenshot of Image demo](https://github.com/rivo/tview/blob/master/demos/image/screenshot.jpg)

Code:

```go
// Demo code for the Image primitive.
package main

import (
	"bytes"
	"encoding/base64"

	"image/jpeg"
	"image/png"

	"github.com/gdamore/tcell/v2"
	"github.com/rivo/tview"
)

const (
	beach = `see link below for the entire Base64 payload`
	chart = `see link below for the entire Base64 payload`
)

func main() {
	app := tview.NewApplication()

	image := tview.NewImage()
	b, _ := base64.StdEncoding.DecodeString(beach)
	photo, _ := jpeg.Decode(bytes.NewReader(b))
	b, _ = base64.StdEncoding.DecodeString(chart)
	graphics, _ := png.Decode(bytes.NewReader(b))
	image.SetImage(photo)

	imgType := tview.NewList().
		ShowSecondaryText(false).
		AddItem("Photo", "", 0, func() { image.SetImage(photo) }).
		AddItem("Graphics", "", 0, func() { image.SetImage(graphics) })
	imgType.SetTitle("Image Type").SetBorder(true)

	colors := tview.NewList().
		ShowSecondaryText(false).
		AddItem("2 colors", "", 0, func() { image.SetColors(2) }).
		AddItem("8 colors", "", 0, func() { image.SetColors(8) }).
		AddItem("256 colors", "", 0, func() { image.SetColors(256) }).
		AddItem("True-color", "", 0, func() { image.SetColors(tview.TrueColor) })
	colors.SetTitle("Colors").SetBorder(true)
	for i, c := range []int{2, 8, 256, tview.TrueColor} {
		if c == image.GetColors() {
			colors.SetCurrentItem(i)
			break
		}
	}

	dithering := tview.NewList().
		ShowSecondaryText(false).
		AddItem("None", "", 0, func() { image.SetDithering(tview.DitheringNone) }).
		AddItem("Floyd-Steinberg", "", 0, func() { image.SetDithering(tview.DitheringFloydSteinberg) }).
		SetCurrentItem(1)
	dithering.SetTitle("Dithering").SetBorder(true)

	selections := []*tview.Box{imgType.Box, colors.Box, dithering.Box}
	for i, box := range selections {
		(func(index int) {
			box.SetInputCapture(func(event *tcell.EventKey) *tcell.EventKey {
				switch event.Key() {
				case tcell.KeyTab:
					app.SetFocus(selections[(index+1)%len(selections)])
					return nil
				case tcell.KeyBacktab:
					app.SetFocus(selections[(index+len(selections)-1)%len(selections)])
					return nil
				}
				return event
			})
		})(i)
	}

	grid := tview.NewGrid().
		SetBorders(false).
		SetColumns(18, -1).
		SetRows(4, 6, 4, -1).
		AddItem(imgType, 0, 0, 1, 1, 0, 0, true).
		AddItem(colors, 1, 0, 1, 1, 0, 0, false).
		AddItem(dithering, 2, 0, 1, 1, 0, 0, false).
		AddItem(image, 0, 1, 4, 1, 0, 0, false)

	if err := app.SetRoot(grid, true).EnableMouse(true).Run(); err != nil {
		panic(err)
	}
}
```

(The [code in the repo](https://github.com/rivo/tview/blob/master/demos/image/main.go) contains the full Base64-encoded image data.)