

# Elements overview
Tipp: Preview every element by calling `sg.Examples.preview_all_elements()`.

This tutorial gives you a brief overview over all the available SwiftGUI-elements.

Most of these elements can do much more than I show in this tutorial, so it pays off to read the corresponding element-tutorials.

SwiftGUI-elements are categorized into 3 groups:
- Widget elements: Elements containing only a single tkinter-widget that is used as tkinter intends to
- Extended elements: Elements containing only a single tkinter-widget that is used significantly different to tkinters intention
- Combined elements: (Exclusive to SwiftGUI) Elements containing multiple tkinter-widgets

All screenshots in this tutorial use the theme `sg.Themes.FourColors.Emerald`.

# Widget-elements
Tkinter, the package behind SwiftGUI already offers a lot of elements (called widgets).

Widget-elements are basically just a raw tkinter-widget, maybe with some custom, higher-level functionality.

These elements should be simmilar for most GUI-libraries based on tkinter.

## Text
Aliases: T, Label

Displays a text. Nothing more, nothing less:\
![](../assets/images/2025-11-12-10-58-44.png)

Something I like to do is to set the relief to `groove`, so it looks like a value-display:\
![](../assets/images/2025-11-12-11-01-06.png)
```py
    sg.T("14.2 °C", relief= "groove", width= 10, anchor= "center")
```

## Button
A button that can be clicked:\
![](../assets/images/2025-11-12-11-03-19.png)

The only element with its default-event enabled without specifying.

There is a little known functionality:
Setting `repeatdelay` and `repeatinterval`, the button will repeat its press when the user holds it down.

Try this out:
```py
import SwiftGUI as sg
import time

sg.Themes.FourColors.Emerald()

layout = [
    [
        sg.Button(
            "Press me",
            key= "Button",
            repeatdelay= 400,
            repeatinterval= 200,
        )
    ]
]

w = sg.Window(layout, padx=30, pady=30)

for e, v in w:
    print(e, v)
```
If you hold down the button, it will start to repeat after 400 ms and simulate a press each 200 ms.

## Checkbox
Aliases: Check, Checkbutton

Something the user can "check" and "uncheck" by clicking on it:\
![](../assets/images/2025-11-12-11-14-32.png)

You can also make it look like a button setting `check_type = "button"`:\
![](../assets/images/2025-11-12-11-16-35.png)
```py
layout = [
    [
        sg.Checkbox("Check", check_type= "button", width= 8)
    ],[
        sg.Checkbox("Checked", default_value= True, check_type= "button", width= 8)
    ]
]
```

It's a bit hard to see, but the lower button looks like it is held down, because it is "checked".

## Radiobutton
Aliases: Radio

The radiobutton is somewhat simmilar to a checkbox, but only one radio of a group can be checked:\
![](../assets/images/2025-11-12-13-35-25.png)

When `Radio 2` is clicked, `Radio 1` unchecks.

## Scale
The "opposite" of `sg.Progressbar`:
Lets the user select a number by sliding a slider:\
![](../assets/images/2025-11-12-13-38-12.png)

## Frame
Aliases: Column

The frame can combine multiple elements into one.

It's crucial for building well-looking GUIs and explained in-depth in basic tutorial 06 (bigger layouts).

## GridFrame
Usually, elements are ordered in rows.
Inside a `sg.GridFrame`, elements are also aligned in columns.

It's also explained more detailed in basic tutorial 04 (bigger layouts).

## LabelFrame
Same as `sg.Frame`, but has a border and a descriptive text:\
![](../assets/images/2025-11-12-13-28-31.png)

## Input
Aliases: In, Entry

The input lets the user enter a text.\
![](../assets/images/2025-11-12-12-06-06.png)

As you already know, the default-event of an input is thrown whenever the input-text changes.\
An alternative to this is to only throw an event when `return` is pressed:
```py
    sg.Input(
        "Enter your text",
        key= "Input"
    ).bind_event(
        sg.Event.KeyEnter
    ),
```

## Listbox
The listbox displays a list of texts:\
![](../assets/images/2025-11-12-12-22-33.png)
```py
    sg.Listbox(
        range(15)
    )
```
The user can select rows by clicking on them, causing the default event, if enabled.

A little known functionality is that you can color specific rows.
```py
    sg.Listbox(
        range(15)
    ).color_rows(
        (3,4,7),
        background_color= "red",
    )
```
![](../assets/images/2025-11-12-12-27-48.png)

However, the coloring disappears when overwriting a colored row.

## TKContainer
Aliases: TKWidget

If you know tkinter, you can use this element to add a tkinter widget into the SwiftGUI layout with not too much hassle.

## TextField
Aliases: Multiline

The Multiline is basically an input but with multiple lines of text:\
![](../assets/images/2025-11-12-12-43-25.png)

The tkinter-widget behind this has a loooooot of additional functionality.
For now (version 0.10.13), these are not implemented in SwiftGUI yet.

One cool thing that is implemented already is the undo-stack.
When setting `undo = True`, the user can reset previous changes by pressing `Ctrl + z` inside the element.

## Treeview (WIP)
The element that gives me the most headache, so I put it off for now (version 0.10.13).
It will be implemented until version 1.0.0, I promise.

## Notebook
Aliases: TabGroup

Notebooks let you divide your layout into tabs:\
![](../assets/images/2025-11-12-13-07-47.png)

## Combobox
Aliases: Combo

The combobox features a drop-down menu to select a string form a list.
The selected string is displayed even when the drop-down is closed:\
![](../assets/images/2025-11-12-13-18-10.png)

![](../assets/images/2025-11-12-13-21-55.png)

## Canvas
The canvas lets you "draw" structures and layouts very freely:\
![](../assets/images/2025-11-12-13-26-33.png)

This element is huge and not explained in detail here, but there is an element-tutorial for it.

## Progressbar, ProgressbarVertical
(Usually) displays a percentage in a bar-graph:\
![](../assets/images/2025-11-12-13-31-17.png)\
(3 `sg.Progressbar`)

The vertical version is called `sg.ProgressbarVertical`:\
![](../assets/images/2025-11-12-13-32-27.png)

# Extended elements
SwiftGUI offers a couple of elements that contain a single tkinter-widget, but change/extend its functionality drastically.

The definition is up to interpretation.
E.g. sg.Table is actually made from a ttk.Treeview but modified to something completely different, so it should be categorized as extended.
It's still closer to a widget-element in my opinion.

## Table
A table with one or more columns.\
Probably the best element of them all.

Also actually an extended Element, but it fits better in this category in my opinion.

![](../assets/images/2025-08-06-12-16-53.png)

I do have to admit that PySimpleGUI's Table has a lower startup-time when adding a lot of rows, but SwiftGUI's Table easily has the better performance at runtime.

You should really check out the detailed documentation on Tables, this element is incredibly useful.

## Spacer
Aliases: S

To be exact, this is an extended element, but it's so basic, let's keep it here.

## Separators (HorizontalSeparator, VerticalSeparator)
Aliases: HSep, VSep

## FileBrowseButton
A button that opens a file-dialogue when clicked.

## ColorChooserButton
A button that opens a color-chooser-menu when clicked.

## Image
Simple widget to display an image.

## ImageButton
Button with an image instead of a text.


# Combined elements
SwiftGUI offeres a couple of "combined" elements that consist of multiple SwiftGUI-elements.

## Form (WIP)

