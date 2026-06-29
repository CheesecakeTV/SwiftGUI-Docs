
Written in SwiftGUI version 0.11.14, if nothing else stated.

# Extending elements
Extending elements means taking an existing element and modifying it.

There already are some extended elements, like the `sg.ColorChooserButton`.
The underlying element is just a button that has some builtin functionality.

In my opinion, creating extended elements is harder than creating combined elements.
You should learn about combined elements first.

## Why extend elements and not just use combined ones?
You could implement all of these features using combined elements, true.

However, the combined element is a segregated container.
You'll need to manually implement a way to transfer information from/to the element.

Extending an element makes sure all of the unmodified functionalty still does what it's supposed to do.
You only need to change what you want to change and not worry about the rest.

# Basics
To extend an element, derive its class.

For this tutorial, we'll extend input-elements as an example:
```py
import SwiftGUI as sg

class ModifiedInput(sg.Input):
    ...
```

We'll extend/modify the element by overwriting methods defined in `sg.Input` or its parent-classes.

You can use that element like any other element:
```py
import SwiftGUI as sg

class ModifiedInput(sg.Input):
    ...

sg.Themes.FourColors.Emerald()

layout = [
    [
        ModifiedInput(
            key="MyInput",
            default_event=True,
        )
    ]
]

w = sg.Window(layout)

for e,v in w:
    print(e,v)
```

# Modifying the value
As you know, the value of an input-element is whatever text it contains.

You can change that by overwriting `set_value` and `_get_value`.

Let's say, our element should only accept and return `int`-values.
If the entered value can't be converted to `int`, the most recent acceptable value should be returned:
```py
from typing import Any
import SwiftGUI as sg
from SwiftGUI.Compat import Self    # Version of typing.Self that is compatible with Python 3.10

class ModifiedInput(sg.Input):
    def set_value(self, val: int, throw_event: bool = False) -> Self:   # This is not necessary, but I've added it for the typehints
        super().set_value(str(val), throw_event)
        return self # In SwiftGUI, methods that don't return anything usually return self instead

    _last_correct_value: int = 0
    def _get_value(self) -> int:
        value = super()._get_value()    # Get the actual value

        try:    # Try to convert it to int
            value = int(value)
        except ValueError:  # If conversion fails return the last correct value
            return self._last_correct_value

        # Otherwise, save that value and return it
        self._last_correct_value = value
        return value
```
Now, the value of that element is always an integer.

## Saving and loading values
As you might remember, you can "collect" the values of every keyed element by calling `.to_json()` on the window, or its value-dict.

The values are collected through every element's `to_json`-method.
This way, you can save additional information when using this functionality.

Take a look at `to_json` of `sg.Listbox`:
```py
    def to_json(self) -> dict:
        return {
            "index": self.index,
            "rows": self.all_elements
        }
```
As you can see, besides its current selection (index), it returns the whole list.

On the other side is `to_json`, which re-applies the saved values:
```py
    def from_json(self, val: dict) -> Self:
        self.clear_list()
        self.append(*val.get("rows", tuple()))  # Add all rows

        if val.get("index") is not None:
            self.index = val.get("index")   # Select the prior selection

        return self
```

By default, `to_json` just returns `_get_value()`, while `from_json` uses `set_value(...)`:
```py
    def to_json(self) -> Any:
        val = self._get_value()
        return val

    def from_json(self, val: Any) -> Self:
        self.set_value(val)
        return self
```
If you do overwrite them, make sure `to_json` returns something that can actually be converted to a json-string.

# Default event
## Brief introduction to how events are handled
Any element with a default event has the following methods:
- `throw_event`: Throws the default event, no matter if it was enabled or not
- `throw_default_event`: Throws the default event, if enabled
- `_event_callback`: Gets called when the underlying tkinter-event triggers (see below)

The implementation is pretty straight-forward for "non-textual-elements" (explained below):
```py
    def throw_event(self) -> Self:
        self._event_function()  # This is the actual event-function that calls the key-functions and event-loop

        return self

    def throw_default_event(self) -> Self:
        if self._default_event:
            self.throw_event()

        return self

    def _event_callback(self, *_):
        self.throw_default_event()
```
As you can see, the actual event is just passed to the next function, until `throw_event` finally generates the SwiftGUI event.

For elements with an input-field, tkinter (the underlying GUI-package) may sometimes generate an event, even when the text doesn't change.
That's why there is a different `_event_callback` for these elements:
```py
    def _event_callback(self, *_):
        if self.value == self._prev_value:  # Make sure the value actually changed
            return

        self._prev_value = self.value

        self.throw_default_event()
```

You can find out which version is used by an element when looking at its class (Hold `Ctrl` and left-click on `sg.Input` to get there in PyCharm):
```py
class Input(MixinElementWithValue, BaseWidget):
```
As you can see, `sg.Input` derives from `MixinElementWithValue`, meaning it uses the second implementation of `_event_callback`.

For the other implementation, `MixinElementWithDefaultEvent` is inherited:
```py
class Checkbox(MixinElementWithDefaultEvent, BaseWidget):
```

## "intercepting" default events
So, what to make of that?

First, you can always use `throw_event` and `throw_default_event` to generate the default event.

But even better, you can add inherent functionality to the default event.

Let's say, our example element is supposed to change its color if the user-entry is valid.
We can implement this easily by modifying `_event_callback`:
```py
    def _event_callback(self, *_):  # Don't forget *_
        value = super()._get_value()    # We modified self._get_value earlier, so use super()._get_value() for the acutal value

        try:    # Try to convert to int
            int(value)
        except ValueError:  # If conversion fails
            self.update(background_color = "red")
        else:   # If conversion succeeds
            self.update(background_color = "green")
        
        super()._event_callback(*_)   # Make sure the event isn't totally ignored
```

But why modify `_event_callback` instead of `throw_event`?

Remember that earlier, we modified `_get_value` so it can only return proper `int`-values.
`_event_callback` of `sg.Input` checks, if `_get_value` changed before generating the event.
That means, `throw_event` is only called if the value is correct anyways.

As a rule of thumb:
- Overwrite `_event_callback` to handle ALL default events
- Overwrite `throw_event` to prevent certain events from happening
- Don't overwrite `throw_default_event`.

# Updating / adding options
The full update-routine is way to difficult to explain in full here.
So I'll just show you how to implement it properly and be done with it.

## _update_special_key
First, copy the following method:
```py
    def _update_special_key(self, key: str, new_val: Any) -> bool|None:
        match key:
            case _:
                return super()._update_special_key(key, new_val)
        
        return True
```
Add a case for every option you want to modify.
When that option is being updated, `new_val` contains the new value for that option.

Let's implement two new options for our prior example-element.
The new values should just be saved in the element, so nothing too complicated:
```py
    _correct_color: str = "green"   # Just add some value so the attribute exists
    _wrong_color: str = "red"
    def _update_special_key(self, key: str, new_val: Any) -> bool|None:
        match key:
            case "correct_color":
                self._correct_color = new_val
            case "wrong_color":
                self._wrong_color = new_val
            case _:
                return super()._update_special_key(key, new_val)

        return True
```
Let's also update `_event_callback` to reflect those changes:
```py
    def _event_callback(self, *_):  # Don't forget *_
        value = super()._get_value()

        try:
            int(value)
        except ValueError:  # If conversion fails
            self.update(background_color = self._wrong_color)
        else:   # If conversion succeeds
            self.update(background_color = self._correct_color)

        super()._event_callback(*_)
```

You just implemented all of the following methods for these option:
- `update`
- `update_after_window_creation`
- `get_option`
- `_update_initial` (explained later)

## Difference between update and _update_initial
`.update` and `._update_initial` do basically the same thing, but with one defining difference:
`.update` ignores options passed with the value `None`, while `._update_initial` applies its default value (from the global-option-class).

That's why most of the parameter of `__init__` of SwiftGUI-elements have a default value of `None`.
They get passed to `_update_initial`, which applies the default values.

Let's add the new options to `__init__`, so the user can enter them when creating the element:
```py
    def __init__(
            self,
            *args,
            correct_color: str = "green",   # We don't have default values for these options yet, so let's just add the default here
            wrong_color: str = "red",
            **kwargs,
    ):
        super().__init__(*args, **kwargs)

        self._update_initial(
            correct_color = correct_color,
            wrong_color = wrong_color,
        )
```

## Global options
In SwiftGUI, most elements have their own global-option-class (go-class).
Let's create such a class for the example element too:
```py
class ModifiedInputOptions(sg.GlobalOptions.Input):
    correct_color: str = "green"
    wrong_color: str = "red"
```
The name of the attributes has to match the key used in `_update_initial`.

To connect the example element to this go-class, overwrite the `defaults`-attribute:
```py
class ModifiedInput(sg.Input):
    defaults = ModifiedInputOptions
```
Also, set default parameter-values of `__init__` to None:
```py
    def __init__(
            self,
            *args,
            correct_color: str = None,
            wrong_color: str = None,
            **kwargs,
    ):
        super().__init__(*args, **kwargs)

        self._update_initial(
            correct_color = correct_color,
            wrong_color = wrong_color,
        )
```

That's all.

Besides implementing default values, you also implemented `.update_to_default_value` for these options.

# Doing things right after the window-creation
For some actions, you want to be sure the window exists before you do them.
This turned out to be a very annoying problem, so I spent a considerable amount of time to find a solution.
You're welcome.

## Testing if the window exists
This is very easy.

Just test for `self.window`:
```py
if self.window:     # Same as if bool(self.window)
    ...
```
`bool(self.window)` is only `True`, if the window exists.

## Updating
Besides `.update`, there is `.update_after_window_creation`.
Calling the latter will save that update-call and execute it when the window got created.

If you want to prevent certain update-calls before the window exists, do it like so:
```py
    def _update_special_key(self, key: str, new_val: Any) -> bool|None:
        match key:
            case "someKey":
                if self.window:
                    ... # Some action
                else:
                    self.update_after_window_creation(someKey = new_val)    # Save the call for later
            case _:
                return super()._update_special_key(key, new_val)

        return True
```

For multiple keys, you could use this approach to avoid copying code:
```py
    def _update_special_key(self, key: str, new_val: Any) -> bool|None:
        if key in ["someKey", "someOtherKey"]:  # Protect these two keys
            self.update_after_window_creation(**{key: new_val}) # Save the call for later
            return True # Return True to signal that this key was handled by _update_special_key. Very important.

        match key:
            case "someKey":
                ... # Some action
            case "someOtherKey":
                ... # Some other action
            case _:
                return super()._update_special_key(key, new_val)
        
        return True
```

## _run_after_window_creation
This functionality is extremely useful.

For whole methods that can't run before the window exists, use the decorator `_run_after_window_creation`.
Any call to the decorated method will be saved and ran once the window exists.
If the method is called when the window already exists, the decorator does nothing.

The decorator is an attribute of every SwiftGUI-element, so just pick any class your IDE doesn't complain about:
```py
class ModifiedInput(sg.Input):
    
    @sg.Input._run_after_window_creation
    def has_to_run_later(self) -> Self:
        ...
```

One small hickup:\
Obviously, this doesn't work for methods that return something.
So these functions always return `self`.

## init_window_creation_done
VERY IMPORTANT: NEVER, NEVER, NEVER FORGET TO CALL `super().init_window_creation_done` WHEN OVERWRITING THIS METHOD!

The method `init_window_creation_done` is called once, when the window is created.

Any initialization related to the actual GUI should be done inside `init_window_creation_done`.

# (Interacting with the SwiftGUI event-system)
This will get its own tutorial soon.

# Full code of the example
In case you were curious how `ModifiedInput` looks after all of these examples, here you go:
```py
from typing import Any
import SwiftGUI as sg
from SwiftGUI.Compat import Self    # Version of typing.Self that is compatible with Python 3.10

class ModifiedInputOptions(sg.GlobalOptions.Input):
    correct_color: str = "green"
    wrong_color: str = "red"

class ModifiedInput(sg.Input):
    defaults = ModifiedInputOptions

    def __init__(
            self,
            *args,
            correct_color: str = None,
            wrong_color: str = None,
            **kwargs,
    ):
        super().__init__(*args, **kwargs)

        self._update_initial(
            correct_color = correct_color,
            wrong_color = wrong_color,
        )

    def set_value(self, val: int, throw_event: bool = False) -> Self:   # This is not really necessary, but I've added it for the typehints
        super().set_value(str(val), throw_event)
        return self

    _last_correct_value: int = 0
    def _get_value(self) -> int:
        value = super()._get_value()    # Get the actual value

        try:    # Try to convert it to int
            value = int(value)
        except ValueError:  # If conversion fails return the last correct value
            return self._last_correct_value

        # Otherwise, save that value and return it
        self._last_correct_value = value
        return value

    def _event_callback(self, *_):  # Don't forget *_
        value = super()._get_value()    # We modified self._get_value earlier, so use super()._get_value() for the acutal value

        try:
            int(value)
        except ValueError:  # If conversion fails
            self.update(background_color = self._wrong_color)
        else:   # If conversion succeeds
            self.update(background_color = self._correct_color)

        super()._event_callback(*_)   # Make sure the event isn't totally ignored

    _correct_color: str = "green"
    _wrong_color: str = "red"
    def _update_special_key(self, key: str, new_val: Any) -> bool|None:
        if key in ["someKey", "someOtherKey"]:
            self.update_after_window_creation(**{key: new_val})
            return True

        match key:
            case "correct_color":
                self._correct_color = new_val
            case "wrong_color":
                self._wrong_color = new_val
            case _:
                return super()._update_special_key(key, new_val)

        return True

sg.Themes.FourColors.Emerald()

layout = [
    [
        ModifiedInput(
            key="MyInput",
            default_event=True,
        )
    ]
]

w = sg.Window(layout)

for e,v in w:
    print(e,v)
```
