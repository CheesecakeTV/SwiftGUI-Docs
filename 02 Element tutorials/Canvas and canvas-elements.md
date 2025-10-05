
# sg.Canvas
The canvas is a very unique element added in SwiftGUI version 0.10.0.

You can place "canvas elements" onto it freely.
You can also bind events to these canvas-elements, so interactions are easily possible.

The canvas-space can be scrolled.
This way, you can have a big canvas showing only a part of it at once.

## Creating the canvas
Create the canvas like any other element:
```py
import SwiftGUI as sg
import SwiftGUI.Canvas_Elements as sgc

sg.Themes.FourColors.RoyalBlue()

layout = [
    [
        sg.Canvas(
            
        )
    ]
]

w = sg.Window(layout)

for e,v in w:
    print(e, v)
```
![](../assets/images/2025-10-05-13-30-22.png)

The height and width of the canvas can be defined using `height` and `width` options.

## Coordinates
The canvas has an underlying coordinate-grid.

By default, the upper left corner equals `(x, y) = (0, 0)`.\
Going to the right, `x` increases, going down, `y` increases.

# Canvas-elements
Instead of the normal import of SwiftGUI, it's intended to import the canvas-elements too:
```py
import SwiftGUI as sg
import SwiftGUI.Canvas_Elements as sgc
```
Add sgc-elements directly when creating the canvas, or later:
```py
import SwiftGUI as sg
import SwiftGUI.Canvas_Elements as sgc

sg.Themes.FourColors.RoyalBlue()

layout = [
    [
        canv := sg.Canvas(
            sgc.Line(   # Add an element directly
                (10, 10),
                (50, 30),
            )
        )
    ]
]

# Add one or more elements later
canv.add_canvas_element(
    sgc.Text(
        (50, 50),
        "Hello World",
    )
)

w = sg.Window(layout)

# Create an element, then add it to the canvas later
canv_element = sgc.Oval(
    (10, 50),
    (20, 60),
)

canv_element.attach_to_canvas(canv)

for e,v in w:
    print(e, v)
```
![](../assets/images/2025-10-05-13-57-18.png)

You can use all of those variants before and after creating the window.

## Element-states
Most sgc-elements have 3 different states: "normal", "hidden" and "disabled".

The normal mode is pretty self-explanitory.

In "disabled" state, the element does not respond to events.

In "hidden" state, the element is hidden and unresponsive.

## Normal - active- disabled
When the mouse is over a canvas-element, that element is considered "active".
Setting its state to "disabled", will prevent it from being "active".

Some options can have different values for when the element is active or disabled.
These options will automatically change accordingly.

E.g.: `sgc.Line` can take the options `width`, `width_active` and `width_disabled`.

Setting `width_active` differently than `width`, will make the line change its width when the mouse is over it.

### Closeenough
Especially when dealing with smaller sgc-elements, it might be hard for the user to hover directly over an element.

`closeenough` defines how close the mouse has to be to an element for it to turn "active".
Default is 1.

It is defined for all sgc-elements inside the canvas:
```py
canv = sg.Canvas(
    closeenough= 5,
)
```

## sgc.Line
An `sgc.Line` connects two or more points together:
```py
canv = sg.Canvas(
    sgc.Line(  # Add an element directly
        (10, 10),
        (50, 30),
        (50, 10),
        (70, 30)
    )
)
```
![](../assets/images/2025-10-05-14-04-24.png)

You can configure `width` and `color` to your liking:
```py
canv = sg.Canvas(
    sgc.Line(  # Add an element directly
        (10, 10),
        (50, 30),
        (50, 10),
        (70, 30),
        width= 5,
        color= "red",
    )
)
```
![](../assets/images/2025-10-05-14-06-54.png)

### Capstyle and joinstyle
The style of the line-endings can be changed setting `capstyle` to either "butt" (default), "projecting", or "round".

You may also change the style of the edges (joins) by setting `joinstyle` to "round" (default), "bevel", or "miter".

A great comparison between all those styles can be found here: https://anzeljg.github.io/rin2/book2/2405/docs/tkinter/cap-join-styles.html

### Spline
Turn the line into a spline by smoothing it:
```py
canv = sg.Canvas(
    sgc.Line(  # Add an element directly
        (10, 10),
        (50, 30),
        (50, 10),
        (70, 30),
        width= 3,
        smooth= True,
    )
)
```
![](../assets/images/2025-10-05-14-08-44.png)

If this doesn't look good, try increasing `splinesteps` (default is 12).

### Arrow
Turn the line into an arrow by setting `arrow` to either `first`, `last`, or `both`:
```py
canv = sg.Canvas(
    sgc.Line(  # Add an element directly
        (10, 10),
        (50, 30),
        (50, 10),
        (70, 30),
        arrow= "last",
    )
)
```
![](../assets/images/2025-10-05-14-14-36.png)

If you set it to `both`, both ends of the line have an arrowhead:\
![](../assets/images/2025-10-05-14-16-22.png)

Change the shape of the arrowhead itself by adjusting `arrowshape`:
```py
canv = sg.Canvas(
    sgc.Line(  # Add an element directly
        (20, 20),
        (50, 20),
        arrow= "last",
        arrowshape=(-5, -10, 5),
    )
)
```
![](../assets/images/2025-10-05-14-19-49.png)

The exact meaning of the 3 values is explained excellently here: https://anzeljg.github.io/rin2/book2/2405/docs/tkinter/create_line.html

### Dash
There are a couple of pre-made dash-patterns to choose from:
```py
[".", "-", "-.", "-..", ","]
```
![](../assets/images/2025-10-05-15-29-53.png)

Example for dottet line:
```py
canv = sg.Canvas(
    sgc.Line(  # Add an element directly
        (20, 20),
        (150, 20),
        width= 4,
        dash= ".",
    )
)
```
![](../assets/images/2025-10-05-15-26-33.png)

#### Custom dash-patterns
This feature does not seem to work properly on windows.

Instead of using pre-made dash-patterns, you may define your own one using a tuple:\
`(5, 2)` means 5 pixels of color followed by 2 pixels of transparency.\
`(5, 2, 1, 2)` means 5 px of color, followed by 2 px of transparency, followed by 1 px of color, followed by 2 px of transparency.

#### Dashoffset
This feature does not seem to work properly on windows.

Remember that dot-pattern from before?\
![](../assets/images/2025-10-05-15-26-33.png)

It's a bit annoying that it doesn't look symmetrical.

That's where `dashoffset` comes in.
If you define `dashoffset`, you can skip a number of pixels in the beginning of the pattern.

This will most likely not be necessary anytime soon, but now you know how to fix this issue.

### Stipple
You can use a bitmap to "stipple" the line.

Not all bitmaps are suitable for this, you should only use "gray12", "gray25", "gray50", or "gray75":
```py
canv = sg.Canvas(
    sgc.Line(  # Add an element directly
        (20, 20),
        (150, 20),
        width= 6,
        stipple= "gray25",
    )
)
```
![](../assets/images/2025-10-05-15-57-11.png)

#### Stippleoffset
You may offset the start of the stipple:
```py
canv = sg.Canvas(
    sgc.Line(  # Add an element directly
        (20, 20),
        (150, 20),
        width= 6,
        stipple= "gray25",
        stippleoffset= "n"
    )
)
```
![](../assets/images/2025-10-05-16-02-04.png)

Notice how the corners are different than before.

This feature can come in handy when you two stippled patterns are right next to each other.
A wront stippleoffset might cause visual errors.

It is explained very well here: https://anzeljg.github.io/rin2/book2/2405/docs/tkinter/stipple-offset.html



