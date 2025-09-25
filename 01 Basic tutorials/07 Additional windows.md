
Since SwiftGUI version 0.9.0, it is possible to create and use multiple windows at once.

# Popups and such
To not "take a sledgehammer to crack a nut", I'll explain the easiest way to handle additional windows first.

By "easiest", I mean windows for a maximum of a single keyed event.
Sounds useless, but it's perfect for something like a small popup.

Create another window using `sg.SubWindow` instead of `sg.Window`:
```py
import SwiftGUI as sg

sg.Themes.FourColors.Froggy()

main_layout = [
    [
        sg.Button(
            "Some Button",
            width= 30,
            height= 5,
        )
    ]
]

another_layout = [
    [
        sg.Button(
            "Another Button",
            width= 30,
            height= 5,
        )
    ]
]

w = sg.Window(main_layout)
sw = sg.SubWindow(another_layout)

for e,v in w:
    print(e,v)
```
![](../assets/images/2025-09-24-23-36-05.png)

SubWindows have almost all of the same options normal windows have.\
You can set an icon, title, background_color, etc.\
You can use `.update` the same way you use it on normal windows.

## Single events
Subwindows can't have an event-loop (yet).
You can do something very simmilar to an event-loop, which I'll explain later.

Using key-functions is the best way to handle events.

For smaller popups, I recommend using `loop_close`.
It will wait for a single keyed event, then close the sub-window, returning the event-key and the last state of values:
```py
another_layout = [
    [
        sg.Input(key= "Input"),
        sg.Button("Button 1", key="B1"),
        sg.Button("Button 2", key="B2"),
        sg.Button("Button 3", key="B3"),
    ]
]

w = sg.Window(main_layout)
sw = sg.SubWindow(another_layout)

e,v = sw.loop_close()
print("Button clicked:", e)
print("Input:", v["Input"])
```
`sw.loop_close()` will wait for a keyed event.
The only possible keyed events are button presses.
The `sg.Input` has a key, but no events enabled by default, so it won't cause a keyed event.

Key-function-events don't cause the window to close, so you may use these for functionality inside the subwindow.

## "Blocking" windows
Executing the above code, you'll notice that it is not possible to click on the button in the main window, until the subwindow is closed.
That's because `sg.loop_close()` "blocks" all the other windows.

Blocked windows won't cause (almost any) events, until unblocked.
Disable this behavior by setting `block_others = False`:
```py
e,v = sw.loop_close(block_others= False)
```
Just remember that the event-loop won't run if the code stops at loop_close, before reaching the event-loop.

There are other ways to block other windows:
- `sg.block_others` to block all other windows, `sg.unblock_others` to unblock them again. The methods won't stop the code from executing.
- `sw.block_others_until_closed()` will block other windows, until the calling window is closed. It won't close until it is actually closed by the user or by calling `sw.close()`.

# Full, additional windows
Small windows are easy, what about actual, functional, additional windows?

In SwiftGUI, sub-windows can do anything normal windows can (except an event-loop that uses `for`).

## Somewhat-event-loop
You can still have a fully functional event-loop for sub-windows, if you don't loop, but use a function:
```py
import SwiftGUI as sg

sg.Themes.FourColors.Froggy()

main_layout = [
    [
        sg.Button(
            "Some Button",
            width= 30,
            height= 5,
            key= "Main Button",
        )
    ]
]

another_layout = [
    [
        sg.Input(key= "Input"),
        sg.Button("Button 1", key="B1"),
        sg.Button("Button 2", key="B2"),
        sg.Button("Button 3", key="B3"),
    ]
]

def sw_loop(e,v):
    # Some example-loop
    print("Button-press:", e)
    print("Input-value:", v["Input"])
    sw["Input"].value = e

w = sg.Window(main_layout)
sw = sg.SubWindow(another_layout, event_loop_function=sw_loop)

for e,v in w:
    print(e,v)
```
It was already mentioned at the end of the previous tutorial.

## Window-order
You can't have multiple `sg.Window`s at once.
If you try, an error will occur.

However, for a sub-window to exist, it needs an active (non-closed) `sg.Window`.
If the window is closed, all the sub-windows will close too.

Pro-tipp: `sg.main_window()` always returns the currently active window.

Sometimes, you can't be sure if a sub-window is really opened after the main window is created.\
E.g.: If you create a function that opens a popup. 
What if the user wants to open the popup without a window?

For that reason, `sg.SubWindow` **will create an actual window if none is present**.

So, if you are unsure, create a sub-window.

### Important!
This can cause funny problems.

Remember, if the window is closed, all the sub-windows close too.

So, if you open multiple popups at once without a pre-made main window, closing the first popup closes all the others too.


