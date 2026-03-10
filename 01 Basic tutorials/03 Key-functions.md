
# Key-functions (motivation)
This is one of the most useful feature of `SwiftGUI` I really missed in `PySimpleGUI`.

It's actually the feature which first inspired me to create SwiftGUI.

Don't get me wrong, the event-loop-system is great for smaller windows, but for bigger ones, you don't want to set and handle each and every tiniest tiny action by running through the whole event-loop.
In PySimpleGUI, every event needs its own entry inside the single, giant event-loop, so everything gets really messy real quick.

Example: You'd like to create an input-field with a small button on the side which clears that input.
The only purpose of that button is to clear the input, yet it enlarges the event-loop.\
It also needs its own unique key, which is quite annoying. 
To have a proper key-structure, you'd need to create very long keys, like `"LeftColumn.TablePart.NameInput.ClearButton"`.\
Don't like that.

The solution: Skip the whole event-loop-tingy and tell the element directly what to do.
No extra key and no going through the event-loop.

Since version 0.8.0, there is a very elegant way to reduce clutter in the event-loop.
This is explained in basic tutorial 07 (Seperate key-systems).\
However, its usecase is a lot different from key-functions.
Both are incredibly useful.

# Basic concept
When creating elements, or binding events, you don't actually need to specify a key to throw events.

If you define a "key-function", that function will be called when the event occurs.

Example: Let's create a button that prints "hello world" when pressed:
```py
import SwiftGUI as sg

def hello_world():
    print("Hello World")

### Global options ###

### Layout ###
layout:list[list[sg.BaseElement]] = [
    [
        sg.Button(
            "Print hello World!",
            key_function= hello_world   # This gets called when the event occurs
        )
    ]
]

w = sg.Window(layout)

### Additional configurations/actions ###

### Main loop ###
for e,v in w:
    ...

### After window was closed ###
```
That's all you need.\
The main loop is empty, yet `hello_world` is called when the button is pressed.

For a small functionality like this, you might want to use a lambda-function instead:
```py
layout:list[list[sg.BaseElement]] = [
    [
        sg.Button(
            "Print hello World!",
            key_function=lambda: print("Hello World")
        )
    ]
]
```

You can also add more than one key-function by putting them inside a list/tuple.
All of them are called when the event occurs:
```py
sg.Button(
    "Print hello World!",
    key_function=[
        lambda: print("Hello World"),   # This gets called
        lambda: print("Another key-function!"), # This also gets called
    ]
)
```
You may add key-functions to custom events calling `.bind_event(..., key_function= ...)` on the element.

# Parameters
This feature looks pretty basic so far.
The key-functions only seem to be able to handle the most basic, unchanging operations.

Here's where the best part about key-functions comes in:\
You can "request" certain information about the GUI by using parameters.

If you create parameters with the following names, they will be passed accordingly:
- `w`     - Window (the window that event was called from, usually called `w`)
- `e`     - Event (Event-key, if you did define it. Same as `e` in the for-loop)
- `v`     - Value-"dict" (Value-dict, Same as `v` in the for-loop)
- `val`   - Value (Value of the event-throwing element, same as `w[e].value`)
- `elem`  - Element (Element that caused the event, same as `w[e]`)
- `args`  - The tkinter-event-arguments (if you don't know what this is, ignore it.), its type is always `tuple`.

Example: Let's create an input-element that prints its value to the console when being changed:
```py
### Layout ###
layout:list[list[sg.BaseElement]] = [
    [
        sg.In(
            default_event=True,
            key_function=lambda val: print("New value:",val),
        )
    ]
]
```
Because the parameter of the lambda-function is called `val`, the current value (`element.value`) is passed and can be printed.

For anyone not so firm with lambda-functions, this is the exact same functionality without lambdas:
```py
def my_fct(val):
    print("New value:", val)

### Layout ###
layout:list[list[sg.BaseElement]] = [
    [
        sg.In(
            default_event=True,
            key_function=my_fct,
        )
    ]
]
```

Another example: Let's change the inputs background-color depending on weather an input number is odd or even:
```py
### Global options ###
def change_if_value_is_even(val, elem): # val: current value of the input. elem: the sg.Input-element causing the event
    try:
        if int(val) % 2 == 0:   # Even
            elem.update(background_color = sg.Color.sea_green)
        else:   # Odd
            elem.update(background_color = sg.Color.light_blue)
    except ValueError: # Not a number
        elem.update(background_color = sg.Color.orange)


### Layout ###
layout:list[list[sg.BaseElement]] = [
    [
        sg.In(
            default_event=True,
            key_function=change_if_value_is_even,
        )
    ]
]
```
Try it yourself, it works perfectly.

This way, we can easily duplicate that functionality without additional effort:
```py
### Layout ###
layout:list[list[sg.BaseElement]] = [
    [sg.In(default_event=True, key_function=change_if_value_is_even)],
    [sg.In(default_event=True, key_function=change_if_value_is_even)],
    [sg.In(default_event=True, key_function=change_if_value_is_even)],
    [sg.In(default_event=True, key_function=change_if_value_is_even)],
    [sg.In(default_event=True, key_function=change_if_value_is_even)],
]
```
![](../assets/images/2025-08-05-14-52-03.png)

With keys, you would need to define a unique key for every copy and change the event-loop accordingly.

Keep in mind that the actual event-loop is still empty.\
Very powerful feature.

# Pre-made KeyFunctions
Some key-functions are pretty common (at least for me).
You will likely write them over and over again for different projects.

That's why SwiftGUI includes a bunch of common key-functions and "key-function-templates".

Every one of them has its own docstring, you can find them [here](https://github.com/CheesecakeTV/SwiftGUI/blob/main/src/SwiftGUI/KeyFunctions.py).

Example: We can use a pre-made key-function to implement the example mentioned at the beginning of this tutorial.\
Let's create an input-element that can be cleared by pressing a button:
```py
### Layout ###
layout:list[list[sg.BaseElement]] = [
    [
        sg.In(
            key="Input"
        ),
        sg.Button(
            "Clear",
            key_function=sg.KeyFunctions.set_value_to("", elem_key="Input") # Set w["Input"].value = ""
        )
    ]
]
```
Looking at the definition of `sg.KeyFunctions.set_value_to`, we can see that it is a function returning another function.
This might be confusing for beginners, so read the docstring of the specific function if you are unsure.\
The returned function is the actual key-function and looks like this:
```py
def temp(v):
    v[elem_key] = new_value
```
Simple, yet useful.

# Decorating functions to turn them into a key-functions
Thanks to [yunlo](https://github.com/yunluo) for this idea!

I tried really hard to find any advantage of this method over standard key-functions, but couldn't.
It's just a different way to structure your code, which you might like better.

Using the decorator `sg.attach_function_to_key(key)`, you may assign key-functions to a key in the event-loop.
When the event-loop receives that key in an event, the key function will be called **instead of running through the loop**:
```py
import SwiftGUI as sg

@sg.attach_function_to_key("Button1")
def do_something():
    print("Button 1 was pressed")

@sg.attach_function_to_key("Button2")
def do_something_else(v):
    print("Button 2 was pressed.")
    v["Button2"] = "Pressed"

layout = [
    [
        sg.Button(" 1 ", key="Button1"),
        sg.Button(" 2 ", key="Button2"),
        sg.Button(" 3 ", key="Button3"),
    ]
]

w = sg.Window(layout)

for e,v in w:
    print("Loop:",e, v)
```
Best to copy the code and try yourself:
- When b1 is pressed, `do_something` is called.
The loop does not run.
- When b2 is pressed, `do_something_else` is called and the value-dict is passed to `v`.
The loop does not run here either.
- The key of b3 isn't used in any of the decorators, so the event-loop runs normally.

Other than actual key-functions, these "decorated functions" should only take `e`, `v`, or `w` as parameters.
**`elem` and `val` will always be `None`.**

It is also possible to attach multiple keys:
```py
@sg.attach_function_to_key("Button1", "Button2")
def do_something(e):
    print(f"{e} was pressed")
    v[e] = "Pressed"
```

Some important things to keep in mind when using this feature though:
- All "decorations" must be done before creating the window (`w = sg.Window(layout)`)
- The decorator only applies to the main event-loop.
It is possible to create multiple event-loops, as explained in basic tutorial 06 (bigger layouts).
**Keys for these loops are not "captured" by the decorator.**
- If you create multiple windows using `sg.SubWindow`, the decorator only works for the main window.
- Don't decorate multiple functions with the same key. Only the last decorated function will be called.


