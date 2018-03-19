# WARNING: This is work in progress and therefore incomplete!

# How to Implement Your Own Primitive

In some rare cases, you may need functionality that `tview` does not offer. If it's something that you believe may be needed by others as well, consider opening an [issue](https://github.com/rivo/tview/issues) with a feature request. (Definitely do that before sending me pull requests.) If it's a reasonable request, it's likely that I will add it to `tview`.

If your request is very specific to your application, however, and existing primitives such as `TextView` are not sufficient to achieve what you want to do, you may need to write your own [`Primitive`](https://godoc.org/github.com/rivo/tview#Primitive).

## Alternatives

Before you start writing your own `Primitive`, you should know that there are alternatives. It is likely that you only need direct access to `tcell`'s [`Screen`](https://godoc.org/github.com/gdamore/tcell#Screen) during certain points of a primitive's lifecycle. This can be achieved without having to fully implement the `Primitive` interface.

### Hooking Into a Primitive's `Draw()` Function

All of `tview`'s primitives inherit from [`Box`](https://godoc.org/github.com/rivo/tview#Box). And `Box` provides a function [`SetDrawFunc()`](https://godoc.org/github.com/rivo/tview#Box.SetDrawFunc) which lets you install a callback function that is called after the `Box` has been drawn.

`SetDrawFunc()` gives you a `tcell.Screen` object and the coordinates of the `Box`. In the following example, we want our box to have a horizontal line and some text in its center:

```go
tview.NewBox().
  SetBorder(true).
  SetDrawFunc(func(screen tcell.Screen, x int, y int, width int, height int) (int, int, int, int) {
    // Draw a horizontal line across the middle of the box.
    centerY := y + height/2
    for cx := x + 1; cx < x+width-1; cx++ {
      screen.SetContent(cx, centerY, tview.GraphicsHoriBar, nil, tcell.StyleDefault.Foreground(tcell.ColorWhite))
    }

    // Write some text along the horizontal line.
    tview.Print(screen, " Center Line ", x+1, centerY, width-2, tview.AlignCenter, tcell.ColorYellow)

    // Space for other content.
    return x + 1, centerY + 1, width - 2, height - (centerY + 1 - y)
  })
```

> TODO: Insert screenshot (boxwithcenterline.png)

This also works for other primitives, and that's where the return values come into play. They dictate where the other content of the primitive will be placed. Here is the same example but for a `TextView`:

```go
tview.NewTextView().
  SetText("This is the text view's content").
  SetTextAlign(tview.AlignCenter)
textView.SetBorder(true).
  SetDrawFunc(func(screen tcell.Screen, x int, y int, width int, height int) (int, int, int, int) {
    // Draw a horizontal line across the middle of the box.
    centerY := y + height/2
    for cx := x + 1; cx < x+width-1; cx++ {
      screen.SetContent(cx, centerY, tview.GraphicsHoriBar, nil, tcell.StyleDefault.Foreground(tcell.ColorWhite))
    }

    // Write som text along the horizontal line.
    tview.Print(screen, " Center Line ", x+1, centerY, width-2, tview.AlignCenter, tcell.ColorYellow)

    // Space for other content.
    return x + 1, centerY + 1, width - 2, height - (centerY + 1 - y)
  })
```

> TODO: Insert screenshot (textviewwithcenterline.png)

### Hooking Into the Application's `Draw()` Function

If what you need to do affects all primitives, you can also get access to `tcell.Screen` when the entire `Application` is redrawn. There are two functions for this:

- [`SetBeforeDrawFunc()`](https://godoc.org/github.com/rivo/tview#Application.SetBeforeDrawFunc): The provided callback function is invoked just *before* the `Application` draws its root `Primitive`. This allows you to draw onto the screen "underneath" any primitives.
- [`SetAfterDrawFunc()`](https://godoc.org/github.com/rivo/tview#Application.SetAfterDrawFunc): The provided callback function is invoked *after* the `Application` has drawn its root `Primitive`. This is useful if you need to draw something on top of all primitives.

### Hooking Into Key Events

In addition to drawing, you can also intercept all key events. Again, this can be done on the `Box` level or on the `Application` level:

- [`Application.SetInputCapture()`](https://godoc.org/github.com/rivo/tview#Application.SetInputCapture): The provided callback function is invoked when a key is pressed. Whatever key event you return will be passed on to the default Application handling.
- [`Box.SetInputCapture()`](https://godoc.org/github.com/rivo/tview#Box.SetInputCapture): Same as `Application.SetInputCapture()` but on a primitive level. The callback function is invoked when the primitive has focus and a key is pressed.

For example, if you want to cause the application to shut down upon `Ctrl-Q` in addition to `tview`'s default `Ctrl-C`, you can achieve that in the following way:

```go
app.SetInputCapture(func(event *tcell.EventKey) *tcell.EventKey {
  if event.Key() == tcell.KeyCtrlQ {
    app.Stop()
  }
  return event
})
```

## Writing Primitives

If none of this is enough to achieve your goals, you can write your own `Primitive`. Please note that the `Primitive` interface may be subject to changes (e.g. I might add mouse interaction in the future).

One could say that all you need to do is implement the [`Primitive` interface](https://godoc.org/github.com/rivo/tview#Primitive). And this is true. But your life will be much easier if you subclass from the [`Box` primitive](https://godoc.org/github.com/rivo/tview#Box), as all other primitives do, because `Box` already provides default implementations for many of `Primitive`'s functions.

Let's say we want to implement a simple radio buttons primitive. The definition could look like this:

```go
type RadioButtons struct {
	*tview.Box
	options       []string
	currentOption int
}

func NewRadioButtons(options []string) *RadioButtons {
	return &RadioButtons{
		Box:     tview.NewBox(),
		options: options,
	}
}
```

Because we subclass from the `Box` primitive, we automatically inherit all of its functions such as [`SetBorder()`](https://godoc.org/github.com/rivo/tview#Box.SetBorder) and [`SetTitle()`](https://godoc.org/github.com/rivo/tview#Box.SetTitle).

`Box`es are empty per default so we add our own `Draw()` function to draw the radio button options:

```go
func (r *RadioButtons) Draw(screen tcell.Screen) {
	r.Box.Draw(screen)
	x, y, width, height := r.GetInnerRect()

	for index, option := range r.options {
		if index >= height {
			break
		}
		radioButton := "\u25ef" // Unchecked.
		if index == r.currentOption {
			radioButton = "\u25c9" // Checked.
		}
		line := fmt.Sprintf(`%s[white]  %s`, radioButton, option)
		tview.Print(screen, line, x, y+index, width, tview.AlignLeft, tcell.ColorYellow)
	}
}
```

At first, we call the `Draw()` function of `Box`. This will clear the space for us and possibly draw the border and title. We then determine the area that we need to draw into by calling `GetInnerRect()`. After that, we draw the options, consisting of a radio button (checked or unchecked) and the option text. We need to ensure that we don't draw outside the allowed space so we break out of the loop when we reach the maximum height. Horizontally, the `tview.Print()` function will take care of that.

The `tview.Print()` function is quite powerful and it is used throughout the entire `tview` package. It takes a screen coordinate and a maximum width. Text will not be written outside that box. In addition, you can align the text to the left, right, or center, and give it a color. It also uses color tags (described [here](https://godoc.org/github.com/rivo/tview)) so you can give different parts of the text different colors.

The result may look like this:

> TODO: Insert screenshot (screenshot.png)

