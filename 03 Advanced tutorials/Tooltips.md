Written for SwiftGUI 0.11.20

# Tooltips
Since SwiftGUI version 0.11.20, it is possible to create tooltips.

Tooltips are those texts popping up when you hold your mouse over an element for some time:\
![](../assets/images/2026-09-02-15-50-03.png)

The tooltip will disappear if the mouse is moved away from the element.

# Basic usage
Creating tooltips is as simple as it gets.

Just call `.set_tooltip` on the element when creating it:
```py
layout = [
    [
        sg.T(
            "Hover over me",
        ).set_tooltip("I'm a tooltip")
    ]
]
```

## Frames
You may also add tooltips to frames.

That way, every contained element will show that tooltip.

If an element inside a "tooltipped" frame also has a tooltip, the inner one shows over that element:
```py
inner_layout = [
    [
        sg.T(
            "Hover over me!",
        )
    ],[
        sg.T(
            "No, hover over me!"
        ).set_tooltip(
            "I take priority :D"
        )
    ]
]

layout = [
    [
        sg.Frame(inner_layout).set_tooltip("You are hovering over the frame!")
    ]
]
```
![](../assets/images/2026-09-02-15-59-34.png)\
![](../assets/images/2026-09-02-15-57-38.png)

Same with combined elements.

# Customization
## Colors / Look
Customize tooltips by modifying the global options `sg.GlobalOptions.Tooltip`:
```py
sg.Themes.FourColors.TransgressionTown()
sg.GlobalOptions.Tooltip.background_color = "navy"

layout = [
    [
        sg.T(
            "Hover over me!"
        ).set_tooltip(
            "Thanks!"
        )
    ]
]
```
![](../assets/images/2026-09-02-16-06-07.png)

Most options of `sg.Text` can be modified through that go-class.

Right now, there is no way of customizing only a single tooltip, all of them look the same for all elements.

## Delay
You may specify how long it takes until a tooltip opens by setting `sg.Tooltips.tooltip_open_delay` to your desired delay (in seconds):
```py
sg.Tooltips.tooltip_open_delay = 0.1
```

# Overwriting the tooltip-class
It is possible to redefine the whole tooltip-popup.

However, it is not documented yet.


