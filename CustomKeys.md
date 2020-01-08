Many of the widgets `tview` provides can be controlled using the keyboard. The keys are inspired by the keys in `vi` or `vim` so users familiar with this editor will have an easy time navigating the widgets. The documentation for a widget should list the keys and their functions.

For some applications, you may wish to handle additional keys or change the keys used by `tview`. For this, the [`Box` class](https://godoc.org/github.com/rivo/tview#Box) (superclass of all primitives) provides the function [`SetInputCapture()`](https://godoc.org/github.com/rivo/tview#Box.SetInputCapture). You give it your own callback which is invoked every time a key is pressed. It receives the associated key event and returns a key event.

## Using Different Keys

If you want a key that is currently not handled (or has a different function) to be used for a specific function, you map it to the default key of the primitive. For example, if you want `w` to navigate a `TextView` upwards instead of its default `j`:

```go
textView.SetInputCapture(func(event *tcell.EventKey) *tcell.EventKey {
	if event.Rune() == 'w' {
		return tcell.NewEventKey(tcell.KeyRune, 'j', tcell.ModNone)
	}
	return event
})
```

## Deactivating Keys

In the previous example, the `j` key will still work (in addition to the `w` key). If you want to deactivate a key, you can return `nil`:

```go
textView.SetInputCapture(func(event *tcell.EventKey) *tcell.EventKey {
	switch event.Rune() {
	case 'w':
		return tcell.NewEventKey(tcell.KeyRune, 'j', tcell.ModNone)
	case 'j':
		return nil
	}
	return event
})
```

## Handling Additional Keys

Of course, you can assign your own functionality to any key. Note that your `SetInputCapture()` callback is [invoked in the main goroutine and a redraw will always follow](https://github.com/rivo/tview/wiki/Concurrency#event-handlers). So it's quite easy to make changes to your widget:

```go
textView.SetInputCapture(func(event *tcell.EventKey) *tcell.EventKey {
	if event.Key() == tcell.KeyEscape {
		textView.Clear()
		return nil
	}
	return event
})
```