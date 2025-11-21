
This tutorial explains a way of calling functions periodically without any knowledge of threading.


# Basic principle
Use the decorator `sg.call_periodically()` to make a function be called once every second.

For example, let's implement a simple counter:
```py
import SwiftGUI as sg

sg.Themes.FourColors.DeepSea()

counter = 0

@sg.call_periodically()
def count():
    global counter
    counter += 1
    my_text.value = str(counter)

layout = [
    [
        my_text := sg.Text("0")
    ]
]

w = sg.Window(layout, padx=30, pady=30)

for e, v in w:
    print(e, v)
```

## Delay
To change the delay between function-calls, modify the `delay`-parameter:
```py
@sg.call_periodically(delay= 0.5)
```

The minimum delay possible is `0.001`.

# Counter
Since the counting-mechanism is quite common, SwiftGUI offers a builtin way.

If you set `counter_reset` to a positive integer, the first argument of the decorated function will receive a counter-value:
```py
@sg.call_periodically(counter_reset= 3)
def count(counter):
    my_text.value = str(counter)
```
The counter resets at `counter_reset`, restarting at `0`.
The value of `counter_reset` will NEVER be passed.

## No reset
To make the counter count indefinetly, set `counter_reset = 0`:
```py
@sg.call_periodically(counter_reset= 0)
```

# Manually starting the periodic calls
If you don't want to make the function execute whenever any window is opened, you have the option to disable `autostart`:
```py
@sg.call_periodically(counter_reset= 0, autostart= False)
```

To start the execution, call the function once and WITHOUT ARGUMENTS:
```py
import SwiftGUI as sg

sg.Themes.FourColors.DeepSea()

@sg.call_periodically(counter_reset= 0, autostart= False)
def count(counter):
    my_text.value = str(counter)

layout = [
    [
        my_text := sg.Text("0")
    ]
]

w = sg.Window(layout, padx=30, pady=30)
count() # Start periodic execution

for e, v in w:
    print(e, v)
```

The function stops executing once its window doesn't exist anymore.

# Dangers
## High execution time
This decorator is NOT THREADED (there will be a threaded one in a later version, currently 0.10.14)!

It's the same as if you'd call the function from the main event-loop.

While the function executes, the window doesn't respond.
Keep execution-time as short as possible.

## Non-accessible references
If `autostart = True`, the function executes as soon as any window exists.

If that window is something else, like a popup you opened before the main window, the function executes anyhow.

This can cause references to elements that don't exist (yet), like shown in this example:
```py
import SwiftGUI as sg

sg.Themes.FourColors.DeepSea()

counter = 0

@sg.call_periodically(delay= 0.5)
def count():
    global counter
    counter += 1
    my_text.value = str(counter)

sg.Popups.show_text("The code will start now!") # NameError: name 'my_text' is not defined

layout = [
    [
        my_text := sg.Text("0")
    ]
]

w = sg.Window(layout, padx=30, pady=30)

for e, v in w:
    print(e, v)
```

## Calling the function manually
Every time you call the function manually, a periodic execution is started.

That means, calling it 5 times, will make it execute 5 (or 6) times per delay instead of 1.

## A bug I wasn't able to fix yet (version 0.10.14)
When a periodic function (with `autostart = True`) executes on one window, the window is closed and another one is opened, an exception is thrown.

You can ignore that exception and know that I am going to fix it as soon as I know how.

