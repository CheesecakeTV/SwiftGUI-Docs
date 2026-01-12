
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

