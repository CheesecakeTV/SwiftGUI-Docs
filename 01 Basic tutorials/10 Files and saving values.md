
# Saving data is annoying
Unfortunately, saving data is essential for most programs.

It's not particularely difficult, just soooo off-putting.

This tutorial explains how SwiftGUI makes saving data less annoying.

It also explains how to "collect"/"restore" all current element-values.\
This way, it's easy to let the user save what their doing and restore it later.

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
You can not save instances of classes you created yourself.
Same goes for objects from external packages.

## Pickle
Pickle is the Python-equivalent of json.
It lets you save whatever you want, even objects of self-created classes.\
However, it is not humanly-readable like json.

The basic usage of pickle is the same as json, but it has room for a couple of major mistakes:

### Preserving references
All referenced objects from any object are saved with the object.

Let's say you want to save the value-dict of your window (Which doesn't work, don't do it):
```py
my_values = w.value

pickle_save(my_values)    # Placeholder, doesn't work
```
But `my_values` contains a hidden reference to the window:`my_values._window`.\
That means, pickle will **also save the window-object**.

Guess what, the window contains references to every element in the layout.
These will also be collected and saved.

So instead of saving some values to the file, you just saved your whole layout.

### Missing attributes
This is more advanced and only important for anyone creating their own classes.

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
`my_instance.print_object()` generates and `AttributeError`, because `my_instance` does not know `updated_hello`.

There are ways around this problem, but I'm not gonna cover those here.
Just keep in mind that this problem exists and can kill old save-files if you're not careful.

# Dict-files
SwiftGUI's dict-files work like Python-dictionaries, but let you easily save its content to an actual file.

Currently (Version 0.11.0), there is a json- and a pickle-version of dict-files.

They save the values differently, but you can use them almost exactly the same way.
It's also quite easy to create a dict-file-version for your own encoder, but that's not covered by this tutorial.

I'll only show the json-version.
Just remember that Pickle works the same.

Keep the advantages of json/pickle in mind when choosing the type.
Json is humanly-readable while pickle lets you save non-basic types.

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
(With auto-save on) Each value-change updates the file, which can cause lag-spikes.
File-operations are slow.

So if possible, save multiple values at once.

Like normal dicts, you can use `.update`:
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

Your decision.

### auto-save
By default, every "write-operation" writes over the file.

Disable this behavior when creating the file:
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

### Loading values from the file
When created, the file is read once and only written after that.
If an external source changes the file, your program won't notice by itself.

So, to read the file again, use `.load()`.

Keep in mind that this operation applies all values included in the file.
If you don't use auto-save and forget to save before, this might overwrite changes you made.

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

## Saving/loading to different locations


