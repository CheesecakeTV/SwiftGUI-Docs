
# sg.Notebook
The notebook (tabview in PySimpleGUI) lets you organize layouts in "tabs".

Only the selected tab is visible, so it's great for avoiding clutter in your layout.

`sg.TabFrame` is a related element that makes implemending notebooks more convenient.

A general introduction to both elements is given in basic tutorial 06 (bigger layouts).

![](../assets/images/2025-10-20-12-46-01.png)

This code acts as a starting point and will be modified through the tutorial:
```py
import SwiftGUI as sg

sg.Themes.FourColors.HotAsh()

tab1 = sg.Frame([
    [
        sg.T("This is Tab 1!")
    ],[
        sg.Button("Some elements")
    ],[
        sg.Listbox(range(10)).set_index(3)
    ]
], key= "Tab 1")

tab2 = sg.Frame([
    [
        sg.T("This is Tab 2!")
    ],[
        sg.Button("Some elements")
    ]
], key= "Tab 2")

tab3 = sg.Frame([
    [
        sg.T("This is Tab 3!")
    ],[
        sg.Button("Some elements")
    ]
], key= "Tab 3")

layout = [
    [
        my_nb := sg.Notebook(
            tab1, tab2, tab3,
        )
    ]
]

w = sg.Window(layout, padx=30, pady=30)

for e,v in w:
    ...

```

# Basic usage
In the above code, you can see the basic usage of notebooks.

Just take some frames, put them into the notebook and you are done.

The key of each frame acts as the title of that tab.


# Events
Enabling the default event, an `sg.Notebook` throws an event each time the user changes the tab:
```py
    my_nb := sg.Notebook(
        tab1, tab2, tab3,
        default_event= True,
        key_function= lambda: print("Tab changed!"),
    )
```

## Events for specific tabs
In many cases, you don't need to know if the tab has changed, but if it changed to a specific tab.

Thats why, `sg.Notebook` let's you bind events to single tabs.
When that tab is selected, that special event is called instead of the default one.

If `default_event = False`, the other tabs won't cause events.

Define a specific event like this:
```py
    my_nb := sg.Notebook(
        tab1, tab2, tab3,
    ).bind_event_to_tab(
        tab_key = "Tab 1",
        key_function = lambda: print("Tab 1 was opened"),
    )
```
`tab_key` references the key of that tab's included frame.
Instead, you may pass `tab_index`, which references the index of the tab (counted from left to right, starting at 0).

Other than that, `.bind_event_to_tab` works the same like `.bind_event`.


## Events from code-side tab-selection
Like this, changing the tab via code (explained later) does not throw an event.

What I sometimes like to do is to "link" two notebooks.
When one of the notebooks changes, the other should too.

In cases like this, it can be helpful to throw an event on any tab-change.
To do so, enable `event_on_backend_selection`:
```py
    my_nb := sg.Notebook(
        tab1, tab2, tab3,
        default_event= True,
        key_function= lambda: print("Tab changed!"),
        event_on_backend_selection= True,
    )
```
You'll notice that the initialization of the notebook also counts as a tab-change and causes an event.



