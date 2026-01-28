
# Config-files
Even though you like your program in a certain way, your users might not feel the same.

E.g.: If my grandma used programs I wrote, she'd not be able to read anything, because the texts were too small.
Without configuration-options, I'd have to change the program itself, making the font bigger for all users.\
Not ideal.

A configuration-file is the easiest (proper) way to make programs configurable.
Using configuration-files, you can add a ton of options with little to no additional effort.

SwiftGUI offers a way to implement them easily.
It looks complicated at first, but is much easier than the stuff explained in most other basic SwiftGUI tutorials.

To be honest, the heavy lifting is all done by the builtin package `configparser`: https://docs.python.org/3/library/configparser.html#configparser.ConfigParser \
SwiftGUI only adapts the package and makes it more suited for its usecase.

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
A different name means a different (or new) file, so all user-configurations in the old file are ignored.

**It is crucial that each configuration-file only has one corresponding `ConfigFile`-object.**
If more `ConfigFile`-objects access the same file, they could overwrite each other.

# Creating sections
Remember that `ini`-files are divided into sections.
Create a section like so:
```py
import SwiftGUI as sg

config_file = sg.Files.ConfigFile("Main config.ini")

my_section = config_file.section("Some section")
```
You can create multiple sections for the same file:
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

my_section.set("my_value", "Hello") # Changing "my_value" to "Hello"
my_section["another_value"] = "World"   # Changing "another_value" to "World"
```
The changes are saved to the file automatically.\
This is what "Main config.ini" looks like afterwards:
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

For naming consistency, there is also `.update`, which requires a dict as argument:
```py
my_section.set_many({
    "my_value": "Hello",
    "another_value": "World",
})
```

## Auto-save
Alternatively, you can disable auto-save to prevent lags when writing a lot of values.
```py
config_file = sg.Files.ConfigFile("Main config.ini", auto_save= False)
```

However, now you have to save manually using `.save()` on any object related to `config_file`:
```py
config_file.save()  # Save to file
my_section.save()   # Does the same as config_file.save()
```
Note that `.save()` on any section saves the whole file, not just that particular section.

# Reading values
Read values like with dictionaires:
```py
import SwiftGUI as sg

config_file = sg.Files.ConfigFile("Main config.ini")

my_section = config_file.section("Some section")

print(my_section["my_value"])   # Hello
```
Make sure that your key is lowercase only and the value is included in the config-file (or its defaults, which is discussed later).\
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
Aparrently, the program expects you to look up all of the parameter names.
Very inconvenient.

This is what happens when you only use `.get(...)` to handle default-values.

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
If any of these values are missing, they are added instantly:
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
This has no benefit and should be avoided.

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

# Bools as values
Bools are absolutely not compatible with this format.\
Any non-empty string is considered as `True` by Python.

So when calling `my_section.get("some_bool", to_type= bool)`, any value except for an empty string would be `True`.

SwiftGUI offers a workaround: Using `.get_bool` instead of `.get`.

This way, the following strings represent `True` (Not case-sensitive):
- `1`
- `True`
- `Yes`
- `Y`

Anything else is `False`.

# More complex values (JSON)
For structures like lists, dicts and sets, SwiftGUI offers another workaround:
Saving the values as a JSON-string.

Note that this is very unclean and should only be utilized in small scales.
It works well, but feels wrong.

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
The following json-methods are available:
- `set_json`
- `set_json_many`
- `get_json`
- `add_json_defaults`

I know this isn't ideal.
A different pure file-format, like TOML might be better suited, but that would require an additional package to be installed.

## Pure json-configuration
If you need to include a lot of json-objects in your configuration, consider using a json-dict-file instead.
Setting `add_defaults_to_values=True`, the file functions almost exactly like a configuration-section:
```py
sg.Files.DictFileJSON("My configuration.json", defaults=..., add_defaults_to_values=True)
```

# Configuration-elements

# Conclusion: The proper way of using config-files
I know the concept of user-configurability might sound a bit strange at first.
But trust me on this, if you want people to use your program, you need to make it configurable. 
Especially if those people are computer-literate, or even, god forbid, use Linux.




