
Written in SwiftGUI version 0.11.17.

# Hotkeys
This tutorial talks about causing events when certain keys, or key-combinations are pressed.


# Element- and Window-events
When you bind a "hotkey-event" to a single element, it can only trigger while the event has __focus__.

Think of focus like a selection.
The selected element has the focus.
Different elements behave differently regarding the focus, but usually the last clicked element has the focus.

Window-events can trigger when any element in that window has the focus.
This is pretty similar to having the focus.
The last selected (clicked, or `alt+tab`bed) window has the "window-focus" (not its real name).

Element-events are bound to elements and window-events to the window:
```py
import SwiftGUI as sg

sg.Themes.FourColors.RoyalBlue()

layout = [
    [
        sg.Input(
            "I am an input"
        ).bind_event(
            sg.Event.KeyEnter,
            key= "Element-event",
        )
    ],[
        sg.Input(
            "I am another input"
        )
    ]
]

w = sg.Window(layout, padx=50, pady=50).bind_event(sg.Event.Control_Q, key="Window-event")

for e, v in w:
    print(e)
```
Running the code, you will see that pressing `Enter` only causes an event when the first `sg.Input` is selected, but `Ctrl+Q` works with any selection.

# sg.Event
`sg.Event` is a small helper-class that stores some useful Tkinter event-strings, so you don't need to memorize them all.

Here is a look at its current form:
```py
class Event:
    ### Focus ###
    FocusIn = "FocusIn" # Element gained the focus
    FocusOut = "FocusOut"   # Element lost the focus

    ### Mouse ###
    MouseWheel = "MouseWheel"   # Scrolled with scroll wheel
    MouseMove = "Motion"    # Mouse has been moved

    MouseHoldAndMoveLeft = "B1-Motion"  # The mouse-button is held down and moved over this element
    MouseHoldAndMoveMiddle = "B2-Motion"
    MouseHoldAndMoveRight = "B3-Motion"

    MouseEnter = "Enter"    # Mouse starts hovering over the element
    MouseExit = "Leave"     # Mouse stopped hovering over the element

    ClickAny = "Button" # Any mousebutton clicked on this element
    ClickLeft = "Button-1"
    ClickMiddle = "Button-2"
    ClickRight = "Button-3"

    ClickDoubleAny = "Double-Button"
    ClickDoubleLeft = "Double-Button-1"
    ClickDoubleMiddle = "Double-Button-2"
    ClickDoubleRight = "Double-Button-3"

    ### Special keys ###
    KeyDelete = "Delete"
    KeyEnter = "Return"
    KeySpace = "space"
    KeyTab = "Tab"
    KeyEscape = "Escape"
    KeyPageDown = "Next"
    KeyPageUp = "Prior"
    KeyBackspace = "BackSpace"

    LeftShift = "Shift_L"
    LeftAlt = "Alt_L"
    LeftControl = "Control_L"
    RightShift = "Shift_R"
    RightAlt = "Alt_R"
    RightControl = "Control_R"

    ### Arrow-keys ###
    ArrowDown = "Down"
    ArrowLeft = "Left"
    ArrowUp = "Up"
    ArrowRight = "Right"

    ### Hotkeys ###
    Control_Q = "Control-q"     # Used for quitting applications in many software
    Control_C = "Control-c"
    Control_V = "Control-v"
    Control_X = "Control-x"
    Control_Enter = "Control-Return"
    Shift_Enter = "Shift-Return"

    ### Others ###
    AnyKey = "Any-KeyPress"

    ### Methods ###
    # You can use these methods like this: sg.Event.Control_("e")       # for ctrl+e
    # Or like this: sg.Event.Shift_(sg.Event.Enter)     # for Shift+Enter
    Control_ = _event_modifier("Control")
    Alt_ = _event_modifier("Alt")
    Shift_ = _event_modifier("Shift")   # Doesn't work with letters. For letters, pass the capital version instead of the normal letter.

    Double_ = _event_modifier("Double") # Same event twice in a row (like double-click)
    Triple_ = _event_modifier("Triple") # Same event 3 times in a row
    
    # (Two methods are missing down here)
```
As you can see, most attributes are just a constant string.

So, you could just pass the string itself to `.bind_event`:
```py
    sg.Input(
        "I am an input"
    ).bind_event(
        "Return",   # Event-string
        key= "Element-event",
    )
```

## Methods on sg.Event
`sg.Event` also offers some "Modifier-functions".

For example, if your event is supposed to trigger when `Ctrl+e` is pressed, create the event-string like this:
```py
sg.Event.Control_("e")
```
Note that the shift-modifier is applied automatically if the passed letter is uppercase.
This will result in the hotkey `Ctrl+Shift+e`:
```py
sg.Event.Control_("E")
```

There is also `.Double_` and `.Triple_` which require the key-presses to be caused two, or three times in a row for the event to trigger.

These modifications also work with most non-keyboard events:
```py
sg.Event.Double_(sg.Event.ClickLeft)
```
(This does the same as `sg.Event.ClickDoubleLeft`)

## Chaining events
There is a way to chain events.
That means, all "parts of the chain" must trigger one after the other for the event to occur.

For example, this event will trigger when the mouse enters the input-element and the user presses `Esc` shortly after:
```py
    sg.Input(
        "I am an input"
    ).bind_event(
        sg.Event.chain(sg.Event.MouseEnter, sg.Event.KeyEscape),
        key= "The user knew what this event does",
    )
```

Since this method can be annoying for many characters, there is also `sg.Event.string`, which treats every letter as its own event.
This can be useful when you want to control your applications through "commands":
```py
w = sg.Window(layout).bind_event(sg.Event.string("close"), key_function=lambda w:w.close())
```
The above example closes the window whenever the user types "close".



