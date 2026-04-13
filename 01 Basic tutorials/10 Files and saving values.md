
# Saving data is annoying
Unfortunately, it is also essential for most programs.
It's not particularely difficult, just soooo off-putting.

This tutorial explains how SwiftGUI makes saving data less annoying.

It also explains how to "collect"/"restore" all current element-values.\
This way, it's easy to save/load the user-entries of the whole layout.

# File-types
Before getting to the interesting part, you need to know these two common file-types: `json` and `pickle`.

## JSON
JSON lets you store complex object-structures by converting it to a so-called "json-string".
It's actually originated from Java (the programming language), not Python.

Such a string looks a lot like python-objects:
```json
{
    "Key1": [
        0,
        1,
        2
    ],
    "Other key": "Hello World",
    "Nested GUI": {
        "Swift": "Gui"
    }
}
```
You could just copy and paste it to the script to recreate the saved dict.

A big disadvantage is that **json only lets you save (most) builtin types**.\
That means, you can store `list`, `dict`, `set`, `int`, and so on, but not instances of classes you created yourself.
Same goes for most objects from other packages.

## Pickle
Pickle is the Python-equivalent of json.
It lets you save whatever type of object you want, even objects of self-created classes.\
However, it is not humanly-readable like json.
You can't open a pickle-file with a text editor.

Pickle has a couple of benefits that also pose a risk:

### Preserving references
All referenced objects from any object are saved together with that object.

Example: Let's say you want to save the value-dict of your window (Which doesn't work, don't do it!):
```py
my_values = w.value

pickle_save(my_values)    # Placeholder, doesn't work
```
But `my_values` contains a hidden reference to the window: `my_values._window`.\
That means, pickle will **also save the window-object**.

Guess what, the window contains references to every element in the layout.
These will also be collected and saved.

So, instead of saving some values to the file, you just saved your whole layout.

### Missing attributes
This is more advanced and only important when saving self-made classes (instances).

Objects loaded from a pickle file do not run `__init__` when being restored.\
That means, any attribute you create in `__init__` is either saved in the file or doesn't exist.

Consider this example:
```py
class Test:
    def __init__(self):
        self.hello = "World"

    def print_object(self):
        print(self.hello)
```
The user creates and saves an instance of that class:
```py
my_instance = Test()
pickle_save(my_instance)    # Placeholder, does not work
my_instance.print_object()
```
He can also load it later:
```py
my_instance = pickle_load() # Still a placeholder
my_instance.print_object()
```
It runs without any errors.

Now, you'd like to release an update for your class:
```py
class Test:
    def __init__(self):
        self.hello = "World"
        self.updated_hello = "New world"

    def print_object(self):
        print(self.hello, self.updated_hello)
```
What do you think happens, when a user loads his save from the prior version?
```py
my_instance = pickle_load() # Still a placeholder
my_instance.print_object()
```
`my_instance.print_object()` generates an `AttributeError`, because `my_instance` does not know `updated_hello`, since it was not saved in that file.

There are ways to work around this problem, but I'm not gonna cover those here.
Just keep in mind that this problem exists and can potentially kill old save-files.

# Dict-files
SwiftGUI's dict-files basically work like Python-dictionaries, but let you easily save its content to an actual file.

Currently (Version 0.11.0), there is a json- and a pickle-version of dict-files.

You can use them exactly the same way, even though they save in different formats.
It's also quite easy to create your own version of dict-files, but that's not covered by this tutorial.

Keep the advantages of json/pickle in mind when choosing which to use.\
Json is humanly-readable, while pickle lets you save non-basic types.

## Creating the file
Create the file like this:
```py
import SwiftGUI as sg

my_file = sg.Files.DictFileJSON(
    "My File.json",
)
```
Instead of a filename you can also pass a more complex path, like `assets/json/My File.json`.
Missing sub-folders are automatically created.

## Saving values
You can use `my_file` like a normal dict:
```py
import SwiftGUI as sg

my_file = sg.Files.DictFileJSON(
    "My File.json",
)

my_file["Hello"] = "World"
my_file["Foo"] = "Bar"
```
The file was instantly created and looks like this:
```json
{
    "Hello": "World",
    "Foo": "Bar"
}
```

You can also use `.set("Hello", "World")`, which has no advantage over the other way.

### Saving multiple values
(With auto-save enabled) Each value-change updates the file, which can cause lag-spikes.
File-operations are slow.

So if possible, save multiple values at once.

Like normal dicts, you may use `.update`:
```py
my_file.update({
    "Hello": "World",
    "Foo": "Bar"
})
```

Or, if you prefer a different notation, use `.set_many`:
```py
my_file.set_many(
    Hello= "World",
    Foo= "Bar",
)
```
...but that only lets you use strings as keys.

### Auto-save
By default, every "write-operation" overwrites the file.

You can disable this behavior when creating the file:
```py
my_file = sg.Files.DictFileJSON(
    "My File.json",
    auto_save= False,
)
```
However, you'll need to save manually now by calling `my_file.save()`.

## Loading values
Read values like you'd read them from a dictionary:
```py
print(my_file["Foo"])
```
Of course, if the key (`Foo` in this case) isn't found, a `KeyError` is thrown.

Alternatively, you may use `.get()`, which returns `None` instead of throwing a `KeyError`.

It also allows you to specify a default-return which returns instead of `None`:
```py
print(my_file.get("Foo", "Value not found"))
```
You might already know that `.get` works exactly the same as `dict.get`.

### Loading values from the file
When created, the file is read once and only written after that.
If an external source changes the file, your program won't notice by itself.

So, to read the file again, use `.load()`.

Keep in mind that this operation applies all values included in the file.
If you don't use auto-save and forgot to save before, this might overwrite changes you made.

## Default values
Instead of using `.get` with a default-return, you should define default values.

Pass these values to the `default`-parameter when creating the file:
```py
import SwiftGUI as sg

my_file = sg.Files.DictFileJSON(
    "My File.json",
    defaults={
        "Number": 15,
    }
)

my_file["Hello"] = "World"
my_file["Foo"] = "Bar"

print(my_file["Number"])    # 15
```
The file still looks like this:
```json
{
    "Hello": "World",
    "Foo": "Bar"
}
```

## Counters
I found that I often need to save "counted values".
These are numeric values that, well, count something.

You don't actually need to create a counter, just use `.increment()`.

Example: Let's print how often the python-script was executed:
```py
import SwiftGUI as sg

my_file = sg.Files.DictFileJSON(
    "My File.json",
    defaults={
        "times_executed": 0,
    }
)

print(my_file.increment("times_executed"))
```
`.increment` adds `1` to the current value and returns the new value.

You can also remove the default-value, just remember that `my_file["times_executed"]` might generate a key-error if `.increment` wasn't called before:
```py
import SwiftGUI as sg

my_file = sg.Files.DictFileJSON(
    "My File.json",
)

print(my_file.increment("times_executed"))
```
You may also specify how much the value should increment by passing `amount`:
```py
print(my_file.increment("money", amount=gained_money))
```

This doesn't only work with `int`, but with any value-types that can be added together.
The implementation of `.increment` is supendously simple:
```py
    def increment(self, key, amount = 1, initial = 0):
        new_val = self.get(key, default=initial) + amount   # Add the amount to the value
        self.set(key, new_val) # Save that value
        return new_val  # Return that value
```
As you can see, you can also pass `initial`, which does the same as defining the key in `default`.

## Saving/loading from different locations
Sometimes, it's necessary to save to a different file, like when creating a backup.

That's why you can pass a different path to `.save()`:
```py
my_file = sg.Files.DictFileJSON(
    "My File.json", # Default location
)

my_file.save("backups/file_backup.json")    # Saving to a different file
my_file.save()  # Saving to "My File.json"
```

Simmilarely, you can load from different files by passing a path to `.load()`.

## Changing the default path
Change the path like so:
```py
my_file.path = "New path.json"
```
Note that the values DO NOT automatically refresh like when using `.load()`.
If you also want to load the new file, you need to call `.load()` afterwards.

Alternatively, you may use `.set_path`:
```py
my_file.set_path("New path.json", reload= True)
```
If that file already exists, `reload=True` calls `.load()` after updating the path.

## Password-protected dict-files
Since SwiftGUI version 0.11.9, you can password-protect dict-files.

However, this is it's own package called `SwiftGUI_Encryption`, which you need to install seperately (via pip).
A lot of that package can be used without SwiftGUI, so you can use it to encrypt things without GUIs too.

`SwiftGUI_Encryption` contains a class called `PasswordJSONDictFile", which works exactly like a normal `JSONDictFile`, but can only be accessed if the correct password is passed:
```py
import SwiftGUI_Encryption as sge

my_file = sge.sg.PasswordJSONDictFile(
    "secrets/ConfidentialInformation", # Choose a file-ending if you like
    "Password", # This is the password to acces the file
)

my_file["SecretHello"] = "Secret world"
```
If you tried to open that file with a different password, a value-error occurs.

Asking the user for a password could look like this:
```py
while True:
    try:
        user_entry = input("Enter your password: ")

        if user_entry == "":
            # If the user entered nothing
            exit()
        
        my_file = sge.sg.PasswordJSONDictFile(  # "Try" to unlock the file
            "secrets/ConfidentialInformation",
            user_entry,
        )

        break   # Stop the loop

    except ValueError:  # If the password is wrong, a value-error happens
        print("Wrong password!")
```

### Not just "security-by-obscurity"
Keep in mind that the file is actually encrypted.
**If you forget your password, that file is useless.**

If you chose a decent password (not just 4 digits), probably not even the best supercomputers could break into that file (at the moment).

# Saving/restoring user-entries
Since SwiftGUI version 0.11.0, it's possible to "gather" and restore all values of full layout-parts at once.

**However, this only works for elements with a key.
Elements without keys are ignored.**

## Saving values
Value dicts are instances of custom classes, so they can't be saved in json files.
**Using pickle is not an option**, since it would save the whole objects instead of their values.

Using `.to_json()` on value-dicts returns all of its contained values as a dictionary that can be json-encoded.

So, saving values can be done like this:
```py
import SwiftGUI as sg

my_file = sg.Files.DictFileJSON("Values.json")

layout = ...

w = sg.Window(layout)

for e,v in w:
    ...
```
```py
my_file.update(w.value.to_json())
my_file.save()  # Only necessary if auto-save is disabled
```
... or inside the for-loop:
```py
my_file.update(v.to_json())
my_file.save()  # Only necessary if auto-save is disabled
```

Tipp: You can also use `.to_json()` on most elements, which returns its value in a format json-files support.\
Some elements return more information than the current value (E.g.: `sg.List`).
So when saving a value, you should stick to `.to_json()`.

## Restoring values
The opposite of `to_json` is `from_json`.

`from_json` requires a `dict` in the form like created by `to_json`:
```py
# Save:
save_state = w.value.to_json()

# Load:
w.value.from_json(save_state)
```

To save and load from a dict-file:
```py
# Save:
my_file.update(w.value.from_json())
my_file.save()

# Load:
w.value.from_json(my_file.to_dict())
```

Tipp: Like `to_json` can be used on most elements, `from_json` also works.
Just keep in mind that `from_json` always requires a format fitting to `to_json`.

## Sub-layouts
Remember that each sub-layout has its own value-dict.

You can use this to divide value-saves into parts.

Consider this example:
```py
layout = [
    [
        sg.In(key= "Input"),
        sg.Combobox(["Choice 1", "Choice 2"], key= "Combo"),
    ],[
        my_sublayout := sg.SubLayout([
            [
                sg.Checkbutton("Check!", key= "Check"),
            ]
        ])
    ]
]

w = sg.Window(layout)
```
`w.value.to_json()` does not contain the sublayout or its elements, because the sublayout has no key.

To include the sublayout in `w.value.to_json()`, you need to give it a key:
```py
    my_sublayout := sg.SubLayout([
        [
            sg.Checkbutton("Check!", key= "Check"),
        ]
    ], key= "Sublayout")
```

Or you can convert only the sublayout and its contained elements by calling `my_sublayout.to_json()`.

