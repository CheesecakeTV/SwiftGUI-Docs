
# sg.Text
Displays a text:\
![](../assets/images/2025-11-14-15-44-37.png)
```py
import SwiftGUI as sg

sg.Themes.FourColors.GarnetFlair()

layout = [
    [
        sg.Text("Amazing!")
    ]
]

w = sg.Window(layout, padx=30, pady=30)

for e, v in w:
    print(e, v)
```

However, a lot of elements have an integrated text, so they share a lot of options with `sg.Text`.

That's why this is one of the most important tutorials.

# Options
## relief
Select the type of relief around the text.

The relief can be used to distinguish the element from the background:\
![](../assets/images/2025-11-14-15-50-19.png)
```py
sg.Themes.FourColors.ArcticGlow()   # Used going further

layout = [
    [sg.Text('relief = raised', relief= "raised")],
    [sg.Spacer(height=5)],  # Some space between texts
    [sg.Text('relief = sunken', relief= "sunken")],
    [sg.Spacer(height=5)],
    [sg.Text('relief = flat', relief= "flat")],
    [sg.Spacer(height=5)],
    [sg.Text('relief = ridge', relief= "ridge")],
    [sg.Spacer(height=5)],
    [sg.Text('relief = solid', relief= "solid")],
    [sg.Spacer(height=5)],
    [sg.Text('relief = groove', relief= "groove")],
]
```

I'll show this option first because it makes many others soo much easier to explain.

## width
The width in digits of the text-area:\
![](../assets/images/2025-11-14-15-53-16.png)
```py
layout = [
    [
        sg.Text('width = 15', relief= "solid", width= 15),
        sg.Text('width = 30', relief= "solid", width= 30),
    ]
]
```
If the width is too small for the text, exceeding text is cut off:\
![](../assets/images/2025-11-14-15-54-57.png)
```py
    sg.Text("Hello_World", relief= "solid", width= 5),
```
As you can see, the width in characters might vary (6 shown instead of 5), because characters are not all of the same width.

## cursor
The cursor-type when the mouse is over this element.




