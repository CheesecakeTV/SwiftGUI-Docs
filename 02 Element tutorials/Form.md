
Written for SwiftGUI version 0.11.20 if not stated further.


# sg.Form
A combined element that combines multiple `sg.Input` and their descriptive texts into a single element:\
![](../assets/images/2026-08-28-11-34-21.png)

```py
sg.Form(
    ("Name", "Address", "Job", "Salary"),
    submit_button=True,
    big_clear_button=True,
)
```

# Basic functionality
The form can be specified in two different ways:
- "List-mode": Field-Texts are given as an iterable like `list` or `tuple`
- "Mapping-mode": Field-Texts are given as a mapping like `dict`

I recommend the mapping-mode, since it is easier to change later on.

In list-mode, you provide field texts using an iterable:
```py
sg.Form(
    ("Name", "Address", "Job", "Salary"),
)
```
This way, the form-element acts similar to a list.

In mapping-mode, you provide a mapping:
```py
my_form := sg.Form(
    {"name":"Name", "add": "Address", "job":"Job", "sal":"Salary"},
)
```
This way, the form-element acts simmilar to a dictionary.

## Submit-button
You may add a submit-button below the entries:
![](../assets/images/2026-08-28-12-02-43.png)

# Default event


# Value


# Options


## Options of other elements


# Methods


# Tips


