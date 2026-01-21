
# Config-files
Even though you might like your program a certain way, your users might not feel the same.

E.g.: If my grandma used programs I wrote, she'd not be able to read anything, because the text is too small.
Without configuration-options, I'd have to change the program itself, making the font bigger for all users.\
Not ideal.

A configuration-file is the easiest (proper) option to make programs configurable.
This way, you can add a ton of options with little to no additional effort.

SwiftGUI offers a way to implement these fairly easily.\
It looks complicated at first, but is honestly a lot easier than the stuff explained in other basic tutorials.

To be honest, the heavy lifting is all done by the builtin package `configparser`: https://docs.python.org/3/library/configparser.html#configparser.ConfigParser \
SwiftGUI only made it easier to use.

# .ini files
In SwiftGUI, configuration-files have the `ini`-format.

These files are divided in sections.
Each section has their own parameters:
```ini
[Some section]
parameter1 = hi
parameter2 = hi2

[Another section]
hello = world
```
A major disadvantage of this format is that values can only be strings.
We'll get to that later.

Also, note that the parameter-names can only be set in lowercase.

# Creating the config-file
Config-files are handled through `ConfigFile`-objects:
```py
import SwiftGUI as sg

config_file = sg.Files.ConfigFile("Main config.ini")
```
Make sure to choose the name of that file very carefully.
If users already made configuration, using a different name would make the prior file useless.

**It is crucial that each configuration-file only has one corresponding `ConfigFile`-object.**
If more `ConfigFile`-objects access the same file, they could overwrite themselves.

# Creating sections
Remember that `ini`-files are divided into sections.
Create a section like so:
```py
import SwiftGUI as sg

config_file = sg.Files.ConfigFile("Main config.ini")

my_section = config_file.section("Some section")
```
You can create multiple sections in the same file:
```py
import SwiftGUI as sg

config_file = sg.Files.ConfigFile("Main config.ini")

my_section = config_file.section("Some section")
my_other_section = config_file.section("Another section")
last_section = config_file.section("Last one")
```

# Writing values
Write single values by using `.set(...)`, or using the section-object like a dictionary:
```py
import SwiftGUI as sg

config_file = sg.Files.ConfigFile("Main config.ini")

my_section = config_file.section("Some section")
my_other_section = config_file.section("Another section")
last_section = config_file.section("Last one")

my_section.set("my_value", "Hello")
my_section["another_value"] = "World"
```
The changes are saved automatically.
This is what the file looks like afterwards:
```ini
[Some section]
my_value = Hello
another_value = World

[Another section]

[Last one]
```
I'll only keep the first section for this tutorial.

## Writing multiple values
Writing any value updates the file, which may slow down your program when writing a lot of values.

So, instead of `set`, you may use `set_many`:
```py
my_section.set_many(
    my_value = "Hello",
    another_value = "World",
)
```
Sets multiple values, but saves only once.

## Auto-save
...or you can disable auto-save:
```py
config_file = sg.Files.ConfigFile("Main config.ini", auto_save= False)
```

However, now you have to save manually using `.save()` on any object related to `config_file`:
```py
config_file.save()  # Save to file
my_section.save()   # Does the same
```

# Reading values
Read values like you'd do with dictionaires:
```py
import SwiftGUI as sg

config_file = sg.Files.ConfigFile("Main config.ini")

my_section = config_file.section("Some section")

print(my_section["my_value"])   # Hello
```
Make sure that your key is lowercase only and the value is included in the config-file.
If it's not included, a `KeyError` occurs, so same as with dictionaries.

Also like dictionaries, you can use `.get()` instead, which doesn't cause a `KeyError`:
```py
print(my_section.get("something else"))   # None
```
This way, you can even add a default return-value:
```py
print(my_section.get("something else", "Default"))   # Default
```

# Default values
This part is important.

Imagine trying to configure a program and being greeted by this configuration-file:
```ini
[Appearance]

[Text settings]

[Translations settings]
```
Aparrently, the program expects you to know all of the parameter names, because the programmer only used `.get(...)`.

A proper way to set default values looks like this:
```py
import SwiftGUI as sg

config_file = sg.Files.ConfigFile("Main config.ini")

my_section = config_file.section("Some section", defaults={
    "text_color": "red",
    "text_size": "14",
    "title": "SwiftGUI Config test",
})
```
If any of these values are missing in the config-file, they are added instantly:
```ini
[Some section]
text_color = red
text_size = 14
title = SwiftGUI Config test
```
You can also add default values by calling `add_defaults(...)` on the section:
```py
my_section.add_defaults(
    text_color = "red",
    text_size = "14",
    title = "SwiftGUI Config test"
)
```

# Bools as values
Bools are absolutely not compatible with this format.
Any non-empty string is considered as `True` by Python.

SwiftGUI offers a workaround: Using `.get_bool` instead of `.get`.

This way, the following strings represent `True` (Not case-sensitive):
- `1`
- `True`
- `Yes`
- `Y`

# Numbers as values
As discussed earlier, only strings are permitted as values.

That's why SwiftGUI offers a way of converting the type when using `.get`:
```py
print(my_section.get("text_size", to_type=int))
```
(Of course, you can use `float` instead)

If the conversion fails, the default value is returned.\
Keep in mind that the user can access the file...

Since a lot of values are gonna be ints/floats, the methods `.get_int` and `.get_float` can be used to save a bit of typing.

# More complex values (JSON)
For structures like lists, dicts and sets, SwiftGUI offers another workaround:
Saving the values as a JSON-string.\
Note that I don't fully trust this feature, but none of my tests failed so far, so you'll probably be fine.

For most methods of the section-object, there is a json-version:
```py
my_section.set_json("my_list", [1, 2, 3, 4, 5])
print(my_section.get_json("my_list"))   # [1, 2, 3, 4, 5]
```
```bash
[Some section]
text_color = red
text_size = 14
title = SwiftGUI Config test
my_list = [
	    1,
	    2,
	    3,
	    4,
	    5
	]
```
It could look better, but does get the job done.

The following json-methods are available:
- `set_json`
- `set_json_many`
- `get_json`
- `add_json_defaults`

I know this isn't ideal.
A different pure file-format, like TOML might be better suited, but I don't want to rely on external libraries too much.\
However, there will be a simmilar implementation for json-files in the future.

# Conclusion: The proper way of using config-files
Since this tutorial is very theoretical, here's an example combining some of the concepts.

I know it looks very messy, since I included some higher concepts too.
```py
import SwiftGUI as sg

# Init the file
main_config = sg.Files.ConfigFile("Config.ini")

general_section = main_config.section("General", defaults={
    "four_colors_theme": "Emerald",
    "title": "SwiftGUI example",
})

layout_section = main_config.section("Layout", defaults={
    "font_size": 12,    # Will be converted to string automatically
    "submit_button": True,
}, json_defaults={
    "form_texts": [
        "Name",
        "Age",
        "Occupation",
    ]
})

theme = getattr(sg.Themes.FourColors, general_section["four_colors_theme"]) # Get a reference to the theme
theme() # Apply the theme

sg.GlobalOptions.Common_Textual.fontsize = layout_section.get_int("font_size")  # Apply font-size
sg.GlobalOptions.Button.fontsize = None # Remove the overwritten value

layout = [
    [
        sg.T("Please enter your information:"),
    ], [
        sg.Form(
            layout_section.get_json("form_texts"),  # Ideally, this is a list.
        )
    ]
]

if layout_section.get_bool("submit_button"):
    layout.append([ # Add a row with the button to the end
        sg.Button("Submit")
    ])

w = sg.Window(layout, title= general_section["title"])  # Apply the title

for e,v in w:
    ...
```

After starting the script, the config-file looks like this:
```ini
[General]
four_colors_theme = Emerald
title = SwiftGUI example

[Layout]
font_size = 12
submit_button = True
form_texts = [
	    "Name",
	    "Age",
	    "Occupation"
	]
```
Run the script yourself and play around with the values to see what they do.



