Writte in SwiftGUI version 0.11.14.

# The event-system
This tutorial takes a deeper dive into the SwiftGUI event-system.

It's very useful to understand when creating custom elements, or implementing some fancy additional functionality to SwiftGUI.

# tkinter-events
As you might know by now, behind SwiftGUI is a python-package called tkinter.

Tkinter generates all user-side (frontend) events.
For example:
- Changing the text in `sg.Input`
- Clicking somewhere on the window
- Moving the mouse over some element

Tkinter offers many different events, some of which are listed in the [TKinter reference](https://anzeljg.github.io/rin2/book2/2405/docs/tkinter/event-types.html).

When a tkinter-event occurs, tkinter simply calls a function.
No key-systems, no key-functions, only a single function-call.

Tkinter does pass some arguments to the called function, but these depend on the type of event and are therefore inconsistent.

SwiftGUI doesn't hide the tkinter-widget (element) away, so if you know tkinter, feel free to bind events in the tkinter-way.

# Event-functions
At the heart of SwiftGUI's event-system are so called event-functions.

An event-function stores all necessary information about the connected event:
- The linked element
- The linked window
- The event-key (The key of the element in most cases)
- All key-functions that are to be called when this event occurs

Calling the event-function (without arguments) triggers the event.

The event-function is designed so it can be bound to tkinter-events.
So when a tkinter-event occurs, the associated event-function is called.

## Creating event-functions
Event-functions are generated on the connected key-handler (Window, SubWindow, SubLayout).

As an example, let's generate an event-function and manually bind it to the tkinter-widget:
```py
import SwiftGUI as sg


layout = [
    [
        my_input := sg.Input()
    ]
]

w = sg.Window(layout)

my_event_function = w.get_event_function(
    me= my_input,   # The event creating this event-function
    key= "MouseOverInput",  # Let's throw it to the loop
    key_function= None, # No key-function should be called
)

my_input.tk_widget.bind("<Enter>", my_event_function)   # tkinter event-binding

for e,v in w:
    print(e, v)
```
This is very close to how events are actually handled.

You may leave `me` empty, which is useful for events not related to SwiftGUI-elements (Like the event happening when the window closes, or timeout-events).
`me` is only used to pass `elem` and `val` to key-functions.

# Throw_event
Manually thrown events through `.throw_event` skip the whole event-function part and mess with the connected key-handler directly.

`throw_event` is actually thread-safe, so you can use it from any thread.

I could go more into detail on `throw_event`, but won't.

# Timeout
Every key-handler has its own, seperate timeout-functionality.
The timeout triggers when no event occured on that key-handler for a specified time.
This includes events without keys.

When any event occurs, its key-handler saves the current time.
A separate thread checks, how long ago that most recent event occured and causes an event if necessary.

You can actually change the attributes `timeout_seconds` (float) and `timeout_active` (bool) while the timeout is active:
```py
import SwiftGUI as sg

sg.Themes.FourColors.Emerald()

layout = [
    [
        sg.Button(
            "Turn off timeout",
            key="Off",
        ),
    ],[
        sg.Button(
            "Set to 1 seconds",
            key="1s"
        ),
    ],[
        sg.Button(
            "Set to 3 seconds",
            key="3s",
        )
    ]
]

w = sg.Window(layout).init_timeout("WindowTimeout", seconds=2)

for e,v in w:
    print(e, v)

    if e == "Off":
        w.timeout_active = False

    if e == "1s":
        w.timeout_active = True
        w.timeout_seconds = 1

    if e == "3s":
        w.timeout_active = True
        w.timeout_seconds = 3
```


