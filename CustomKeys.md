Many of the widgets `tview` provides can be controlled using the keyboard. The keys are inspired by the keys in `vi` or `vim` so users familiar with this editor will have an easy time navigating the widgets. The documentation for a widget should list the keys and their functions.

For some applications, you may wish to handle additional keys or change the keys used by `tview`. For this, the [`Box` class](https://pkg.go.dev/github.com/rivo/tview#Box) (superclass of all primitives) provides the function [`SetInputCapture()`](https://pkg.go.dev/github.com/rivo/tview#Box.SetInputCapture). You give it your own callback which is invoked every time a key is pressed. It receives the associated key event and returns a key event.

## Adding New Keys

If you want a key that is currently not handled (or has a different function) to be used for a specific function, you map it to the default key of the primitive. For example, if you want <kbd>w</kbd> to navigate a `TextView` upwards in addition to its default <kbd>j</kbd>:

```go
textView.SetInputCapture(func(event *tcell.EventKey) *tcell.EventKey {
	if event.Rune() == 'w' {
		return tcell.NewEventKey(tcell.KeyRune, 'j', tcell.ModNone)
	}
	return event
})
```

## Deactivating Keys

In the previous example, the <kbd>j</kbd> key will still work (in addition to the <kbd>w</kbd> key). If you want to deactivate a key, you can return `nil`:

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

## Hierarchical Key Processing

If your primitive is part of a container widget such as [`Grid`](https://pkg.go.dev/github.com/rivo/tview#Grid) or [`Flex`](https://pkg.go.dev/github.com/rivo/tview#Flex), you can use `SetInputCapture()` in those container widgets to intercept key events for all contained primitives and their descendents.

Finally, this function also exists [in the `Application` class](https://pkg.go.dev/github.com/rivo/tview#Application.SetInputCapture) so you can assign functions to specific keys globally as well. For example, if you would like to stop you application with <kbd>Ctrl</kbd>-<kbd>Q</kbd> instead of the default <kbd>Ctrl</kbd>-<kbd>C</kbd>:

```go
app.SetInputCapture(func(event *tcell.EventKey) *tcell.EventKey {
	switch event.Key() {
	case tcell.KeyCtrlQ:
		app.Stop()
	case tcell.KeyCtrlC:
		return nil
	}
	return event
})
```
